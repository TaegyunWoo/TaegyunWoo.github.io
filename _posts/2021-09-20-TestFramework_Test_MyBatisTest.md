---
category: Test-Framework
tags: [Test-Framework]
title: "[테스트] MyBatis 테스트"
date:   2021-09-20 10:30:00 
lastmod : 2021-09-20 10:30:00
sitemap :
  changefreq : daily
  priority : 1.0
---

# MyBatis 테스트

## 개요

### 테스트 종류

MyBatis를 테스트하는 방법에는 총 2가지가 존재한다.

- `@MyBatisTest` 애너테이션을 사용하는 방법
- `@AutoConfigureMyBatis` 애너테이션을 사용하는 방법

<br/><br/>

## MyBatis Test 의존성 추가

MyBatis를 테스트하려면, 먼저 관련 라이브러리를 추가해야한다.

<br/>

### `build.gradle` 파일

```groovy
dependencies {
	implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0' //mybatis
	implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter-test:2.2.0' //mybatis test
}
```

<br/><br/>

## `@MyBatisTest` 애너테이션

### 특징

- **`@MyBatisTest` 는 `@WebMvcTest`, `@SpringBootTest` 와 같은 다른 `@~Test` 애너테이션과 함께 사용할 수 없다.**

    > 이때는 `@AutoConfigureMyBatis` 를 사용해야한다. 이후에, 자세히 다룬다.

- **즉, `@MyBatisTest` 는 오직 Mapper에 대한 유닛 테스트를 수행할 때 사용한다.**

<br/>

### 예시

- `ArticleMapper` 인터페이스

    ```java
    //import 생략

    @Mapper
    public interface ArticleMapper {

      @Insert("INSERT INTO article (title, content, writer, datetime) VALUES (#{article.title}, #{article.content}, #{article.writer}, #{article.dateTime})")
      @Options(useGeneratedKeys = true, keyProperty = "id")
      int insertArticle(@Param("article") Article article);

      @Select("SELECT * FROM article")
      @Results(id = "ArticleMap", value = {
              @Result(property = "id", column = "id"),
              @Result(property = "title", column = "title"),
              @Result(property = "content", column = "content"),
              @Result(property = "writer", column = "writer"),
              @Result(property = "dateTime", column = "datetime")
      })
      List<Article> getAll();

      @Select("SELECT * FROM article WHERE id=#{id}")
      @ResultMap("ArticleMap")
      Article getById(@Param("id") Long id);

      @Select("SELECT * FROM article WHERE writer=#{userId}")
      @ResultMap("ArticleMap")
      Article getByWriter(@Param("userId") String userId);

      @Update("UPDATE article SET title=#{article.title}, datetime=#{article.dateTime}, content=#{article.content} WHERE id=#{article.id}")
      @ResultMap("ArticleMap")
      int update(@Param("article") Article article);

      @Delete("DELETE FROM article WHERE id=#{id}")
      int delete(@Param("id") Long id);
    }
    ```

<br/>

- `ArticleMapperTest` 테스트 클래스

    ```java
    //import 생략

    @MybatisTest
    class ArticleMapperTest {

      @Autowired
      private ArticleMapper articleMapper;

      @Transactional
      @DisplayName("Insert & Select 테스트")
      @Test
      void test() {
        Article article = new Article();
        article.setTitle("Test");
        article.setDateTime(LocalDateTime.now());
        article.setWriter(1L);
        article.setContent("Test Content");
        articleMapper.insertArticle(article);

        Article selectArticle = articleMapper.getById(article.getId());

        assertEquals(article.getId(), selectArticle.getId());
      }

    }
    ```

<br/><br/>

## `@AutoConfigureMyBatis` 애너테이션

### 특징

- **`@AutoConfigureMyBatis` 는 `@WebMvcTest`, `@SpringBootTest` 와 같은 다른 `@~Test` 애너테이션과 함께 사용할 때 사용한다.**

<br/>

### 예시

- `ArticleMapperTest` 테스트 클래스

    ```java
    //import 생략

    @AutoConfigureMybatis // 마이바티스
    @SpringBootTest
    class BoardControllerTest {
        @Autowired
        private ArticleMapper articleMapper;
        
        //기타 테스트 코드
    }
    ```