---
category: MyBatis
tags: [MyBatis, 마이바티스]
title: "SpringBoot에서 마이바티스 사용하기"
date:   2021-09-01 17:40:00 
lastmod : 2021-09-01 17:40:00
sitemap :
  changefreq : daily
  priority : 1.0
---

<br/><br/>

# MyBatis와 SpringBoot

[예제 프로젝트 다운로드](https://github.com/TaegyunWoo/TaegyunWoo.github.io/blob/master/ExampleProjects/mybatis.zip)

<br/><br/>

## 개요

- 스프링부트, 마이바티스, H2 DB 를 이용한다.
- 예시 프로젝트는 어노테이션 기반으로 작성되었다.

## MyBatis 사용

### Mapper란?

- sql과 매핑하는 기능이다.
- 인터페이스에 `@Mapper` 로 명시한다.

<br/>

### `@Insert` 와 `@Param`

- `@Param("쿼리로 전달할 파라미터 이름")`
    - `쿼리로 전달할 파라미터 이름`: 해당 이름의 파라미터를 `#{}` 으로 사용할 수 있다.
    - 메서드의 파라미터 레벨에 작성한다.
    - 해당 파라미터를 쿼리문에서 사용할 수 있게 해준다.

<br/>

- `@Insert("INSERT 쿼리")`
    - 반환값은 int이어야한다.
    - 반환값: 삽입에 성공한 튜플의 개수

<br/>

- 사용 예시

    ```java

    @Insert("INSERT INTO user(email, name, password) VALUES (#{user.email}, #{user.name}, #{user.password})")
    int insert(@Param("user") User user);


    ```

<br/>

### `@Select()` 와 `@Results`

- `@Select("SELECT 쿼리")`

<br/>

- `@Result( id="맵id", value={ @Result(property="매핑할_필드명", column="매핑할_칼럼명"), ... } )`
    - id 속성을 통해, 다른 메서드에서 매핑정보를 재활용할 수 있다.
    - `@Select` 경우, `@Results` 와 `@Result` 를 통해 칼럼과 필드 간의 매핑을 진행해야한다.
    - `@Insert` 경우, `#{}` 를 통해 매핑한다.
    - **칼럼명과 필드명이 같다면 `@Results` 를 생략할 수 있다.**

<br/>

- 사용 예시

    ```java


    //-----------조건 없이 SELECT-------------
    @Select("SELECT * FROM user")
    @Results(id="myResultId", value = {
            @Result(property = "dbid", column = "dbid"),
            @Result(property = "email", column = "email"),
            @Result(property = "name", column = "name"),
            @Result(property = "password", column = "password")
    }) //필드명과 컬럼명이 같으면 @Results 는 필요없다.
    List<User> getAll();

    //-----------조건과 함께 SELECT-------------
    @Select("SELECT * FROM user WHERE dbid=#{id}")
    @ResultMap("myResultId") //매핑정보 재활용
    User getById(@Param("id") Long id);


    ```

<br/>

### `@Update()`

- `@Update("UPDATE 쿼리")`
    - 반환값은 int형이다.
    - 수정된 튜플의 개수를 반환한다.
- 사용 예시

    ```java


    @Update("update user set name=#{user.name}, password=#{user.password} where dbid=#{user.dbid}")
    int updateById(@Param("user") User user);


    ```

<br/>

### `@Delete`

- `@Delete("DELETE 쿼리")`
    - 반환값은 int형이다.
    - 삭제된 튜플의 개수를 반환한다.
- 사용 예시

    ```java


    @Delete("delete from membertab where dbid=#{dbid}")
    int deleteById(@Param("dbid") Long id);


    ```

<br/>

### `@Options`

- `@Options( userGeneratedKeys=true, keyProperty="키_필드" )`
    - 위와 같이 작성하면, **SQL에서 생성된 값(AUTO_INCREMENT 등)이 매핑된 객체의 필드에 역으로 바인딩된다.**
    - 사용 예시

        ```java


        @Insert("INSERT INTO user(email, name, password) VALUES (#{user.email}, #{user.name}, #{user.password})")
        @Options(useGeneratedKeys = true, keyProperty = "dbid") //SQL이 생성한 KEY 값을 매핑된 객체의 dbid 필드에도 담아주겠다.
        int insert(@Param("user") User user);

        
        ```

<br/>

### `@Transactional`

- 해당 메서드를 하나의 DB 작업단위(트랜잭션)로 지정한다.
- **해당 애너테이션이 붙은 메서드 (보통 서비스 클래스의 메서드)에서 예외가 발생하면, 해당 메서드가 작업한 트랜잭션 (매퍼를 통해 수행한 DB 작업)은 모두 롤백된다.**
- 롤백할 예외 설정
    - `@Transactional(rollbackFor=설정할_예외.class)`
    - 해당 예외와 자식 예외까지 처리한다.
    - 처리할 예외의 Default값 = `RuntimeException.class`

<br/><br/>

## 1대다 관계 처리: `@Many`

### 예시 ERD

![Untitled](/assets/img/2021-09-01-MyBatis_MyBatisFunction/Untitled%203.png)

<br/>

### User 클래스

```java
import lombok.Data;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.util.List;

/**
 * 사용자 정보 도메인
 */
@Data
public class User {

  private Long dbid;
  private String email;
  private String name;
  private String password;

  //1대다 관계 설정 (1: User, 다: Phone)
  //테이블에 해당 컬럼은 존재하지 않는다.
  private List<Phone> phoneList;
}
```

<br/>

### Phone 클래스

```java
import lombok.Data;

/**
 * 핸드폰 정보 도메인
 */
@Data
public class Phone {
    private Long id;
    private Long dbid;
    private String number;
}
```

<br/>

### PhoneMapper 인터페이스

```java
@Select("SELECT * FROM phone WHERE id=#{id}")
//@Results 생략
List<Phone> getByUserDbId(@Param("id") Long id);
```

<br/>

### UserMapper 인터페이스

```java
@Select("SELECT * FROM user")
@Results(@Result(property="phoneList", column="dbid", many=@Many(select="패키지명.Phone.getByUserDbId")))
List<User> getAll();
```

- `property="phoneList"`
    - User.phoneList 필드에 저장하겠다.
- `column="dbid"`
    - "User 테이블의 dbid 칼럼의 값"을 "`@Many` 로 지정한 메서드의 파라미터"로 전달하겠다.
- `many=@Many(select="패키지명.매퍼_인터페이스명.메서드명")`
    - 해당 매퍼의 메서드를 호출하겠다.
    - 그리고 그 결과를 User.phoneList 필드에 저장하겠다.

<br/><br/>

## 활용 예시

### ERD

![Untitled](/assets/img/2021-09-01-MyBatis_MyBatisFunction/Untitled%203.png)

<br/>

### Data 객체: `User`

```java
import lombok.Data;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.util.List;

/**
 * 사용자 정보 도메인
 */
@Data
public class User {

  private Long dbid;
  private String email;
  private String name;
  private String password;

  //1대다 관계 설정 (1: User, 다: Phone)
  //테이블에 해당 컬럼은 존재하지 않는다.
  private List<Phone> phoneList;
}
```

<br/>

### Mapper: `UserMapper`

```java
import org.apache.ibatis.annotations.*;
import java.util.List;

@Mapper
public interface UserMapper {

  //Create
  @Insert("INSERT INTO user(email, name, password) VALUES (#{user.email}, #{user.name}, #{user.password})")
  @Options(useGeneratedKeys = true, keyProperty = "dbid") //SQL이 생성한 KEY 값을 매핑된 객체의 dbid 필드에도 담아주겠다.
  int insert(@Param("user") User user);

  //Read - 1
  @Select("SELECT * FROM user")
  @Results(id="myResultId", value = {
          @Result(property = "dbid", column = "dbid"),
          @Result(property = "email", column = "email"),
          @Result(property = "name", column = "name"),
          @Result(property = "password", column = "password"),
          @Result(property = "phoneList", column = "dbid", many = @Many(select = "test.mybatis.domain.PhoneMapper.getByUserId"))
  }) //필드명과 컬럼명이 같으면 @Results 는 필요없다.
  List<User> getAll();

  //Read - 2
  //@Results 재활용
  @Select("SELECT * FROM user WHERE dbid=#{id}")
  @ResultMap("myResultId")
  User getById(@Param("id") Long id);
}
```

<br/>

### Data 객체: `Phone`

```java
import lombok.Data;

/**
 * 핸드폰 정보 도메인
 */
@Data
public class Phone {
    private Long id;
    private Long dbid;
    private String number;
}
```

<br/>

### Mapper: `PhoneMapper`

```java
import org.apache.ibatis.annotations.*;

import java.util.List;

@Mapper
public interface PhoneMapper {
    //Create
    @Insert("INSERT INTO phone(number, dbid) VALUES (#{phone.number}, #{phone.dbid})")
    @Options(useGeneratedKeys = true, keyProperty = "id") //SQL이 생성한 KEY 값을 매핑된 객체의 dbid 필드에도 담아주겠다.
    int insert(@Param("phone") Phone phone);

    //Read - 1
    @Select("SELECT * FROM phone")
    @Results(id="phoneId", value = {
            @Result(property = "id", column = "id"),
            @Result(property = "number", column = "number"),
            @Result(property = "dbid", column = "dbid")
    }) //필드명과 컬럼명이 같으면 @Results 는 필요없다.
    List<Phone> getAll();

    //Read - 2
    //@Results 재활용
    @Select("SELECT * FROM phone WHERE id=#{id}")
    @ResultMap("phoneId")
    Phone getById(@Param("id") Long id);

    @Select("SELECT * FROM phone WHERE dbid=#{dbid}")
    @ResultMap("phoneId")
    List<Phone> getByUserId(@Param("dbid") Long dbid);
}
```

<br/>

### 컨트롤러: `UserController`

```java
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import test.mybatis.domain.User;
import test.mybatis.domain.UserMapper;
import test.mybatis.service.UserService;

import java.util.List;

@RequiredArgsConstructor
@RestController
@RequestMapping("/user")
public class UserController {

    private final UserMapper userMapper;

    @PostMapping("/insert")
    public User insertUser(@RequestBody User user) {
        userMapper.insert(user);
        return user;
    }

    @GetMapping("/selectAll")
    public List<User> selectAllUser() {
        return userMapper.getAll();
    }

    @GetMapping("/selectById/{id}")
    public User selectById(@PathVariable Long id) {
        return userMapper.getById(id);
    }

}
```

<br/>

### 컨트롤러: `PhoneController`

```java
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import test.mybatis.domain.Phone;
import test.mybatis.domain.PhoneMapper;
import test.mybatis.domain.User;
import test.mybatis.domain.UserMapper;

import java.util.List;

@RequiredArgsConstructor
@RestController
@RequestMapping("/phone")
public class PhoneController {
    private final PhoneMapper phoneMapper;

    @PostMapping("/insert")
    public Phone insertPhone(@RequestBody Phone phone) {
        phoneMapper.insert(phone);
        return phone;
    }

    @GetMapping("/selectAll")
    public List<Phone> selectAllPhone() {
        return phoneMapper.getAll();
    }

    @GetMapping("/selectById/{id}")
    public Phone selectById(@PathVariable Long id) {
        return phoneMapper.getById(id);
    }

}
```

<br/><br/>

## 로깅하기

### `application.properties` 파일

```java
logging.level.패키지명.매퍼_인터페이스명=TRACE
```

- SQL 쿼리문과 결과값이 로깅된다.