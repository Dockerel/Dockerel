<div align="center"><h1>Opensource Contribution</h1></div>

## 기여한 기능
* [Spring Data JPA](1-Spring-Data-JPA)
* [Spring Boot](2-Spring-Boot)
* [Spring Framework](3-Spring-Framework)

## 1. Spring Data JPA
### [[Released - 3.5.2]](https://github.com/spring-projects/spring-data-jpa/releases/tag/3.5.2) Cache query strings in SimpleJpaRepository to improve performance - [PR](https://github.com/spring-projects/spring-data-jpa/pull/3920)
### 개요
* SimpleJpaRepository는 Spring Data JPA의 내부 구현체로, 개발자가 정의한 JpaRepository 같은 인터페이스에 대한 CRUD 기능 제공
* 개발자가 직접 이 클래스를 사용하지는 않지만, JpaRepository를 상속받으면 이 클래스가 제공하는 기본 기능들을 활용할 수 있게 됨

### 문제 및 의사결정 과정
```java
public static String getQueryString(String template, String entityName) {
  Assert.hasText(entityName, "Entity name must not be null or empty");

  return String.format(template, entityName);
}
```
* 성능 개선을 위해 deleteAllInBatch의 코드를 분석하던 중 SimpleJpaRepository에서 deleteAll과 count쿼리 실행 시 불필요하게 쿼리 문자열이 반복적으로 생성되고 있다는 것을 발견
* 배치 삭제 또는 카운트 작업이 잦은 애플리케이션에서 성능 저하가 발생할 수 있음
```java
private String getDeleteAllQueryString() {
  return getQueryString(DELETE_ALL_QUERY_STRING, entityInformation.getEntityName());
}

private String getCountQueryString() {
  String countQuery = String.format(COUNT_QUERY_STRING, provider.getCountQueryPlaceholder(), "%s");
  return getQueryString(countQuery, entityInformation.getEntityName());
}
```
* 생성되는 쿼리 문자열들은 불변인 엔티티 메타데이터에 의해 생성됨
```java
private final Lazy<String> deleteAllQueryString;
private final Lazy<String> countQueryString;

this.deleteAllQueryString = Lazy
				.of(() -> getQueryString(DELETE_ALL_QUERY_STRING, entityInformation.getEntityName()));
this.countQueryString = Lazy
        .of(() -> getQueryString(String.format(COUNT_QUERY_STRING, provider.getCountQueryPlaceholder(), "%s"),
            entityInformation.getEntityName()));
```
* 인스턴스 필드 지연 초기화로 최초 1회만 생성 후 재사용하는 방식으로 개선

### 성과
* deleteAll / count 메서드 호출 시 문자열 생성 비용 제거 (매 호출마다 발생하던 오버헤드 제거)
* Spring Data JPA 3.5.2 버전 릴리즈에 성능 최적화 기여
* Spring Data JPA 공식 Contributor 등록

### [[Released - 3.5.2]](https://github.com/spring-projects/spring-data-jpa/releases/tag/3.5.2) Replace regex with startsWith / endsWith check for LIKE pattern detection - [PR](https://github.com/spring-projects/spring-data-jpa/pull/3932)
### 개요
* QueryUtils는 주로 데이터베이스 쿼리를 생성, 조작 또는 처리하는 데 사용되는 유틸리티 클래스
* getLikeTypeFrom 메서드를 통해 LIKE 쿼리를 생성할 때 검색 조건의 유형을 결정

### 문제 및 의사결정 과정
```java
static Type getLikeTypeFrom(String expression) {

	Assert.hasText(expression, "Expression must not be null or empty");

	if (expression.matches("%.*%")) {
		return Type.CONTAINING;
	}

	if (expression.startsWith("%")) {
		return Type.ENDING_WITH;
	}

	if (expression.endsWith("%")) {
		return Type.STARTING_WITH;
	}

	return Type.LIKE;
}
```
* LIKE 패턴 탐지에 복잡한 정규표현식이 사용되어 매 쿼리 파싱 시 불필요한 연산 발생
* expression.matches에서 내부적으로 비용이 높은 작업인 정규식 Pattern.compile()을 수행
```java
static Type getLikeTypeFrom(String expression) {

	Assert.hasText(expression, "Expression must not be null or empty");

	if (expression.startsWith(PERCENT) && expression.endsWith(PERCENT)) {
		return Type.CONTAINING;
	}

	if (expression.startsWith(PERCENT)) {
		return Type.ENDING_WITH;
	}

	if (expression.endsWith(PERCENT)) {
		return Type.STARTING_WITH;
	}

	return Type.LIKE;
}
```
* 단순 문자열 메서드 기반 검사로 전환

### 성과
* LIKE 패턴 탐지 성능 개선
* Spring Data JPA 3.5.2 버전 릴리즈에 성능 최적화 기여
* Spring Data JPA 공식 Contributor 등록

## 2. Spring Boot
### [[Released - 3.4.8]](https://github.com/spring-projects/spring-boot/releases/tag/v3.4.8) Use ThreadLocal.remove() instead of set(null) - [PR](https://github.com/spring-projects/spring-boot/pull/46256)

## 3. Spring Framework
### [[Released - 6.2.11]](https://github.com/spring-projects/spring-framework/releases/tag/v6.2.11) Correct the default value of nestedTransactionAllowed in JpaTransactionManager javadoc - [PR](https://github.com/spring-projects/spring-framework/pull/35212)
