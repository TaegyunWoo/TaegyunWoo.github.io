---
category: Tech
tags: [Tech]
title: "다중 DB 환경에서의 UPDATE/DELETE 무시 이슈. 그리고 Transaction Manager, Entity Manager 간의 관계"
date: 2025-09-24 23:15:00
lastmod: 2025-09-24 23:15:00
sitemap:
  changefreq: daily
  priority: 1.0
---

안녕하세요. 다중 DB 환경에서 `@Transactional` 애너테이션 사용 시, `SELECT` 는 정상적으로 동작하지만 `UPDATE` 와 `DELETE` 가 실행되지 않는 문제를 겪어, 이 경험을 공유드리려 합니다. Kotlin과 JPA 기반 프로젝트에서 제가 직접 마주했던 이슈의 원인과 해결책을 설명하고, JPA 내부 코드까지 살펴보려고 합니다. 이번 경험이 JPA에 대한 동작을 이해하는 데 조금이나마 도움이 됐으면 합니다!

# 왜 SELECT 는 되는데, UPDATE 와 DELETE 는 안되지?

프로젝트 개발 중 특정 데이터를 초기화하는 기능을 구현해야 했습니다. 로직은 간단했습니다. 주문번호로 결제( `Payment` ) 데이터를 조회하고, 상태를 업데이트한 뒤, 연관된 세금계산서 정보들을 삭제하는 것이었죠.

당연히 데이터 정합성이 중요하므로 서비스 메서드에 `@Transactional` 애너테이션을 붙였습니다.

## 예시코드

### `PaymentService.kt`

```kotlin
@Transactional
fun initTax(requestDto: TaxInitRequestDto): TaxInitResponseDto {
  // 1. 데이터 조회 (SELECT)
  val paymentList = paymentRepository.findByOrderNumIn(requestDto.orderNumList)

  // 2. 데이터 변경 (UPDATE)
  paymentList.forEach {
    it.initTaxStatus(); // Dirty Checking 에 의한 UPDATE 유도
  }

  // 중략...

  // 3. 데이터 삭제 (DELETE)
  taxInfoRepository.deleteAllBySeqNoIn(taxInfoSeqNoList)

  // 중략...
}
```

## 당황스러운 문제…

위처럼 코드를 작성했고, 테스트를 해보니 예상치 못한 문제가 발생했습니다. **SELECT 쿼리는 정상적으로 실행되는 반면에, UPDATE 와 DELETE 쿼리는 아예 실행되지 않고 메서드가 종료되어 반환된 것입니다.** 아무런 예외나 경고 로그를 찍지도 않은 상태로 말이죠.

분명 `@Transactional` 을 선언했고 `payment.initTaxStatus()` 메서드는 엔티티의 필드를 변경하고 있었습니다. 따라서 Dirty Checking 에 의해 UPDATE 쿼리가 실행되는 것이 당연했으며, delete 역시 마찬가지였습니다.

### 디버깅 여정

개발과정에서 이와 같은 문제는 처음이었기에, 정말 당황스러웠습니다. 이런 마음은 잠시 미뤄두고… 우선 Dirty Check가 왜 되지 않는지 확인해보았습니다.

Dirty Check 를 자동으로 해주지 않는다면 수동으로 flush 해보자라는 생각으로 아래처럼 실행해봤습니다.

```kotlin

@PersistenceContext(unitName = "oracleEntityManager")
private lateinit var oracleEntityManager: EntityManager

@Transactional
fun initTax(requestDto: TaxInitRequestDto): TaxInitResponseDto {
  // 1. 데이터 조회 (SELECT)
  val paymentList = paymentRepository.findByOrderNumIn(requestDto.orderNumList))

  // 2. 데이터 변경 (UPDATE)
  paymentList.forEach {
    it.initTaxStatus(); // Dirty Checking 에 의한 UPDATE 유도
  }

  // 3. 수동 flush
  oracleEntityManager.flush()

  //이하 중략
}
```

그리고 이때 예상치 못한 예외가 발생했습니다.

```
jakarta.persistence.TransactionRequiredException: no transaction is in progress
```

이 예외는 **DB의 상태를 변경하는 작업(INSERT, UPDATE, DELETE)를 트랜잭션 없이 시도할 때 발생**합니다. 다만 의아한 부분은 분명히 `@Transactional` 을 선언했기에 트랜잭션 안에서 코드가 실행되었다는 것입니다.

이때 제 머리를 스쳐지나간 것이 바로, 다중 DB 설정입니다. 해당 프로젝트에서는 MySQL과 오라클DB를 모두 사용하고 있었는데요. 그래서 아래처럼 설정되어있던 상태였습니다. (예시 코드입니다.)

```kotlin
// ---- MySQLConfig.kt ----
@Configuration
@EnableJpaRepositories(/* 중략 */)
class MySQLConfig {
    @Primary
    @Bean(name = ["mysqlDataSource"])
    @ConfigurationProperties(prefix = "spring.datasource.mysql")
    fun mysqlDataSource(): DataSource {
        return DataSourceBuilder.create().build()
    }

    @Primary
    @Bean(name = ["mysqlEntityManagerFactory"])
    fun mysqlEntityManagerFactory(
        builder: EntityManagerFactoryBuilder,
        dataSource: DataSource
    ): LocalContainerEntityManagerFactoryBean {
        return builder
            .dataSource(dataSource)
            .packages("com.example.mysql.domain")
            .persistenceUnit("mysqlEntityManager")
            .build()
    }

    @Primary
    @Bean(name = ["mysqlTransactionManager"])
    fun mysqlTransactionManager(
        entityManagerFactory: LocalContainerEntityManagerFactoryBean
    ): PlatformTransactionManager {
        return JpaTransactionManager(entityManagerFactory.getObject()!!)
    }
}
```

```kotlin
// ---- OracleConfig.kt ----
@Configuration
@EnableJpaRepositories(/* 중략 */)
class OracleConfig {
    @Bean(name = ["oracleDataSource"])
    @ConfigurationProperties(prefix = "spring.datasource.oracle")
    fun oracleDataSource(): DataSource {
        return DataSourceBuilder.create().build()
    }

    @Bean(name = ["oracleEntityManagerFactory"])
    fun oracleEntityManagerFactory(
        builder: EntityManagerFactoryBuilder,
        dataSource: DataSource
    ): LocalContainerEntityManagerFactoryBean {
        return builder
            .dataSource(dataSource)
            .packages("com.example.oracle.domain")
            .persistenceUnit("oracleEntityManager")
            .build()
    }

    @Bean(name = ["oracleTransactionManager"])
    fun oracleTransactionManager(
        entityManagerFactory: LocalContainerEntityManagerFactoryBean
    ): PlatformTransactionManager {
        return JpaTransactionManager(entityManagerFactory.getObject()!!)
    }
}
```

두 설정간의 차이가 보이실까요? 맞습니다. 바로 `@Primary` 가 있냐 없냐의 차이입니다. MySQLConfig 의 경우, `@Primary` 가 각 엔티티매니저팩토리, 트랜잭션매니저 Bean 에 붙어있고, OracleConfig 의 경우 그렇지 않습니다.

**스프링에서 `@Transactional` 애너테이션을 사용할 때 `transactionManager` 속성을 지정하지 않으면, 스프링 컨테이너에 등록된 트랜잭션매니저 중 `@Primary` 로 지정된 Bean 을 찾아서 사용합니다.**

제 경우, `SELECT`는 되지만 `UPDATE`와 `DELETE`는 실행되지 않았던 이유는 **트랜잭션이 관리하는 영속성 컨텍스트**와 **실제 데이터가 로드된 영속성 컨텍스트**가 달랐기 때문입니다.

1. `@Primary`로 인한 기본 트랜잭션 매니저 지정
   1. `@Transactional` 애너테이션은 별도의 설정을 하지 않으면 `@Primary`가 지정된 `mysqlTransactionManager`를 사용합니다.
   2. 따라서 제가 작성한 서비스 메서드는 **MySQL DB에 대한 트랜잭션**을 시작합니다.
2. 영속성 컨텍스트의 동작
   1. JPA의 `EntityManager`는 각자 독립적인 **영속성 컨텍스트**를 가집니다.
   2. 제 프로젝트에서는 `mysqlEntityManager`와 `oracleEntityManager`가 각각 별개의 영속성 컨텍스트를 생성 및 관리하고 있었습니다.
3. Oracle 엔티티 조회 및 변경
   1. `paymentRepository.findByOrderNumIn(...)`가 호출될 때, `OracleConfig`에 설정된 `oracleEntityManager`가 동작하여 **Oracle 영속성 컨텍스트**에 조회된 `Payment` 엔티티들을 영속화합니다.
   2. 이후 `it.initTaxStatus()` 호출로 상태 변경이 일어납니다. 이 변경 내역은 **Oracle 영속성 컨텍스트**에만 기록됩니다.
4. 잘못된 컨텍스트에 대한 트랜잭션 커밋
   1. 메서드가 종료될 때, `@Transactional`은 1번에서 지정된 `mysqlTransactionManager`에게 트랜잭션 커밋을 요청합니다.
   2. 이때 `mysqlTransactionManager`는 자신이 관리하는 **MySQL 영속성 컨텍스트**를 확인하며 변경된 엔티티(Dirty Checking)를 찾습니다. 하지만 정작 변경이 일어난 `Payment` 엔티티들은 **Oracle 영속성 컨텍스트**에 있으므로, MySQL 영속성 컨텍스트에서는 아무런 변경사항도 감지할 수 없었습니다.
   3. DELETE 의 경우, `oracleEntityManager`가 **자신을 위한 활성화된 트랜잭션이 없다고 판단했기 때문에** DB에 대한 `DELETE` 쿼리 발행을 시도조차 하지 않았을 수 있습니다.
   4. 최종적으로 아무런 UPDATE, DELETE 쿼리도 발생하지 않고 조용히 트랜잭션이 종료됩니다.

# TransactionManager 설정을 통한 문제 해결

정리하자면 트랜잭션 매니저와 엔티티 매니저가 서로 맞지 않아서 발생한 문제라고 할 수 있습니다.

따라서 아래처럼 설정을 하여 문제를 해결할 수 있었습니다.

```kotlin
@Transactional("oracleTransactionManager") //트랜잭션 매니저 명시
fun initTax(requestDto: TaxInitRequestDto): TaxInitResponseDto {
  // 메서드 내용은 모두 동일
}
```

하지만 주로 write 작업을 수행하는 DB는 오라클이였기에, MySQL 설정에 붙은 `@Primary` 를 제거하고, 아래처럼 Oracle 설정으로 `@Primary` 를 붙였습니다.

```kotlin
// ---- MySQLConfig.kt ----
@Configuration
@EnableJpaRepositories(/* 중략 */)
class MySQLConfig {
    //@Primary 제거
    @Bean(name = ["mysqlDataSource"])
    @ConfigurationProperties(prefix = "spring.datasource.mysql")
    fun mysqlDataSource(): DataSource {
        //중략
    }

    //@Primary 제거
    @Bean(name = ["mysqlEntityManagerFactory"])
    fun mysqlEntityManagerFactory(
        builder: EntityManagerFactoryBuilder,
        dataSource: DataSource
    ): LocalContainerEntityManagerFactoryBean {
        //중략
    }

    //@Primary 제거
    @Bean(name = ["mysqlTransactionManager"])
    fun mysqlTransactionManager(
        entityManagerFactory: LocalContainerEntityManagerFactoryBean
    ): PlatformTransactionManager {
        //중략
    }
}
```

```kotlin
// ---- OracleConfig.kt ----
@Configuration
@EnableJpaRepositories(/* 중략 */)
class OracleConfig {
    @Primary //추가
    @Bean(name = ["oracleDataSource"])
    @ConfigurationProperties(prefix = "spring.datasource.oracle")
    fun oracleDataSource(): DataSource {
        return DataSourceBuilder.create().build()
    }

    @Primary //추가
    @Bean(name = ["oracleEntityManagerFactory"])
    fun oracleEntityManagerFactory(
        builder: EntityManagerFactoryBuilder,
        dataSource: DataSource
    ): LocalContainerEntityManagerFactoryBean {
        return builder
            .dataSource(dataSource)
            .packages("com.example.oracle.domain")
            .persistenceUnit("oracleEntityManager")
            .build()
    }

    @Primary //추가
    @Bean(name = ["oracleTransactionManager"])
    fun oracleTransactionManager(
        entityManagerFactory: LocalContainerEntityManagerFactoryBean
    ): PlatformTransactionManager {
        return JpaTransactionManager(entityManagerFactory.getObject()!!)
    }
}
```

위와 같이 설정함으로써, `@Transactional` 애너테이션을 사용하는 것만으로도 제가 원하는 오라클 DB에 대한 트랜잭션을 생성할 수 있었습니다.

# EntityManager 와 TransactionManager 란 무엇일까?

그렇다면 도대체 TransactionManager 와 EntityManager 가 무엇이길래 절 괴롭혔던 것일까요? [SpringDocs](docs.spring.io) 와 [JPA 공식문서](https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html) 를 기반으로 각각의 역할을 정리해봤습니다.

## EntityManager

> 엔티티에 대한 작업을 처리하기 위한 API

Oracle의 Java EE 7 API 문서에 따르면, `EntityManager` 인터페이스는 **"엔티티의 생명주기를 관리하는 영속성 컨텍스트(persistence context)와 상호작용하기 위한 API"**라고 정의됩니다.

쉽게 말해, 애플리케이션과 영속성 컨텍스트 사이의 **중개자** 역할을 합니다. 개발자는 `EntityManager` 가 제공하는 메서드를 통해 데이터베이스 작업을 간접적으로 수행합니다.

- 핵심 기능:
  - `persist(entity)` : 새로운 엔티티를 영속성 컨텍스트에 등록하여 데이터베이스에 저장될 수 있도록 합니다.
  - `find(entityClass, primaryKey)` : 기본 키를 사용해 엔티티 인스턴스를 찾습니다. 먼저 영속성 컨텍스트의 1차 캐시를 확인하고, 없으면 데이터베이스에서 조회합니다.
  - `remove(entity)` : 영속성 컨텍스트에서 엔티티를 제거 대상으로 표시합니다. 트랜잭션이 커밋될 때 데이터베이스에서 삭제됩니다.
  - `merge(entity)` : 분리된(detached) 상태의 엔티티를 받아, 그 내용을 영속성 컨텍스트의 관리되는 엔티티에 병합합니다.
  - `flush()` : 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화합니다. `UPDATE`, `DELETE`, `INSERT` SQL이 이 시점에 실행될 수 있습니다.
- `EntityManager` VS `EntityManagerFactory`
  - 모든 `EntityManager` 객체는 `EntityManagerFactory` 에 의해 생성됩니다.
  - `EntityManagerFactory` 는 Thread-Safe 하며 싱글톤 객체로 관리됩니다. `EntityManager` 는 Thread-Safe 하지 않기에, 한번에 하나의 쓰레드(하나의 트랜잭션)에서만 사용되어야 합니다.
  - `EntityManagerFactory` 는 데이터베이스 연결 풀, JPA 캐시, SQL dialect 등 영속성 계층에 필요한 **공통적인 자원들을 관리**합니다. 매번 `EntityManager`가 생성될 때마다 이러한 자원들을 새로 초기화하는 대신, `EntityManagerFactory`가 한 번 초기화한 자원들을 `EntityManager`들이 공유하고 재사용할 수 있게 합니다.
  - `@PersistenceContext` 으로 `EntityManager` 를 주입받을 경우, 실제 `EntityManager` 대신 프록시 객체가 주입됩니다. 해당 `EntityManager` 프록시 객체 호출시, 현재 트랜잭션과 관련된 실제 `EntityManager` 를 호출해줍니다.
  - 정리하자면, `EntityManagerFactory` 덕분에 DB/JPA 관련 자원을 효율적이고 편리하게 관리할 수 있게 됩니다. 개발자가 `EntityManager` 를 직접 사용한다면, 동시성 관리나 최적화 등에 신경을 써야 합니다.

`EntityManager` 는 이처럼 개별적인 데이터 조작을 담당하지만, 이 작업들이 **언제 데이터베이스에 최종 반영될지를 결정하지는 않습니다.** 그 결정은 `TransactionManager` 이 수행합니다.

## JpaTransactionManager

> 단일 JPA EntityManagerFactory에 대한 트랜잭션을 관리

Spring Framework 공식 문서에서는 `JpaTransactionManager`를 **"지정된 `EntityManagerFactory`에 대한 트랜잭션을 노출시키는 `PlatformTransactionManager` 구현체"**라고 설명합니다.

`JpaTransactionManager`는 JPA 기반의 데이터 접근 코드에 Spring의 선언적 트랜잭션 관리(`@Transactional`) 기능을 적용할 수 있게 해주는 핵심 클래스입니다.

- 핵심 기능:
  - 트랜잭션 경계 설정: `@Transactional` 애너테이션을 감지하여 메서드 시작 시 트랜잭션을 시작하고, 종료 시 커밋(commit) 또는 롤백(rollback)을 수행합니다.
  - `EntityManager` 와 트랜잭션 동기화: 트랜잭션 매니저는 시작된 트랜잭션의 생명주기와 `EntityManager`의 생명주기를 일치시킵니다. 즉, 하나의 트랜잭션은 하나의 `EntityManager`와 연결되어 동작하도록 보장합니다. 제 문제 상황에서 `mysqlTransactionManager` 는 `mysqlEntityManager` 와만 연결되었던 것이 바로 이 원리 때문입니다.
  - 예외 처리: 메서드 실행 중 확인되지 않은 예외(Unchecked Exception)가 발생하면, 트랜잭션 전체를 롤백하여 데이터 일관성을 유지합니다.

## 두 매니저간의 관계

EntityManager 와 JpaTransactionManager 간의 관계를 아주 잘 나타내는 그림이 있어서 가져와봤습니다.

![Untitled](/assets/img/2025-09-25-MultiDB_JPA/image.png)

_출처: [채마스의 개발창고 - JPA에서 여러 종류의 영속성 관리하기](https://hyunwook.dev/220)_

그림을 보면 엔티티 매니저가 영속성 컨텍스트를 관리하고, 해당 엔티티 매니저 (팩토리) 를 트랜잭션 매니저가 의존하고 있습니다.

실제 코드로도 확인해볼까요?

## 트랜잭션 시작 과정을 통해 알아보는 TransactionManager 와 EntityManager 간의 관계

잘 알고계시듯 `@Transactional` 은 AOP를 통해, 즉 프록시 객체를 통해서 트랜잭션이 관리됩니다. 아래 `TransactionAspectSupport` 클래스의 코드이고, 프록시 객체에 의해서 호출됩니다. 이를 보면 어떻게 트랜잭션을 처리하는지 알 수 있습니다.

![Untitled](/assets/img/2025-09-25-MultiDB_JPA/image%201.png)

1. 트랜잭션을 시작합니다. 이때 트랜잭션매니저 객체인 `ptm` 가 파라미터로 전달됩니다. `ptm` 객체는 `@Transactional` 애너테이션으로부터 얻은 메타데이터를 기반으로 선택된 적절한 트랜잭션 매니저입니다. 여기에 위에서 제가 겪은 문제의 원흉이 있겠죠?
2. 실행하려고 했던 AOP의 실제 객체 메서드를 실행합니다. 즉, `@Transactional` 애너테이션이 붙은 실제 메서드를 실행합니다.
3. catch 문 안에 존재하여 예외가 발생한 경우, 롤백 등의 처리를 수행합니다.
4. 작업의 성공/실패 여부를 떠나서, 사용했던 자원들을 초기화합니다.
5. 작업이 모두 성공했다면 최종적으로 커밋합니다.

여기서 중요한 부분은 **1번에서 트랜잭션 매니저가 파라미터로 전달**된다는 것입니다. 그리고 해당 트랜잭션 매니저 객체를 통해서, 실제 트랜잭션 시작 로직을 수행하게 됩니다.

아래는 `JpaTransactionManager` 에서 실제 트랜잭션을 시작하는 코드입니다.

![Untitled](/assets/img/2025-09-25-MultiDB_JPA/image%202.png)

트랜잭션을 시작할 때, `EntityManager em` 을 파라미터로 넘겨주는 것을 확인할 수 있는데요. 해당 `EntityManager` 를 생성할 EntityManagerFactory 는 아래처럼 `JpaTransactionManger` 객체가 이미 알고 있는 상태입니다.

![Untitled](/assets/img/2025-09-25-MultiDB_JPA/image%203.png)

### 정리해보면…

지금까지 살펴본 내용을 정리해보면 아래와 같습니다.

1. `@Transactional` 은 AOP 에 의해 프록시 객체로 실행되며, 프록시 객체에서 트랜잭션 처리를 위해 `TransactionAspectSupport` 를 호출한다.
2. `TransactionAspectSupport` 는 트랜잭션 매니저를 통해 트랜잭션을 시작한다.
3. 트랜잭션 매니저는 트랜잭션을 시작하며 특정 엔티티 매니저를 사용한다.

즉, 코드를 살펴보았을 때도 트랜잭션 매니저가 특정한 엔티티 매니저를 사용(의존)하는 관계라는 것을 알 수 있습니다.

# 마치며

결론적으로 이번 문제는 `@Transactional`이 단순히 'AOP로 동작한다~'가 아니라, '특정 영속성 유닛(DB)에 묶인 트랜잭션 매니저'와 함께 작동한다는 JPA의 근본적인 원리를 놓쳤기 때문이었습니다. 문제를 해결하는 과정에서 JPA 의 내부 코드까지 살펴볼 수 있는 좋은 기회였습니다.

혹시 여러분도 다중 DB 환경에서 JPA의 `UPDATE`, `DELETE` 쿼리가 말없이 사라지는 경험을 하셨다면, 이 글이 여러분에게 작은 도움이 되었으면 하네요.

긴 글 읽어주셔서 감사합니다.
