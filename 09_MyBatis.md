![](./assets/spring-logo.png)

***

# 6. MyBatis

마이바티스는 데이터베이스와 대화할 때 사용할 수 있는 기술 중에 하나입니다. SQL쿼리를 DAO클래스로부터 분리함으로써 SQL관리성을 증대시켜주는 데이터베이스 연동기술 입니다.

Spring JDBC 기술을 사용하는 경우 개발자는 2가지 종류의 로직만 기술합니다. 첫 번째는 SQL쿼리작성이고 두 번째는 결과를 처리하는 로직입니다. 

그런데 변하는 것과 변하지 않는 것이 같이 있다면 분리하는 것이 좋습니다. SQL 쿼리는 시간이 지남에 따라 변경될 가능성이 높습니다. 변하는 부분만 따로 떼서 분리해 놓으면 그 만큼 관리성이 증대됩니다. 이 개념을 멋지게 구현한 기술이 마이바티스라고 할 수 있습니다.

마이바티스는 학습비용도 높지 않아서 국내외에서 널리 사용되고 있습니다. 프로젝트를 개발단계에만 초점을 맞추지 않고 운영 및 업그레이드 단계에도 초점을 맞추어 보면 Spring JDBC 기술보다 마이바티스를 선호하게 됩니다. 

또한 쿼리 작성자와 이를 이용해서 DAO 로직을 만드는 개발자를 분리할 수 있다는 점에서 분업화에 유리하기 때문에 국내의 대형 SI프로젝트에서 많이 사용되고 있습니다. 마이바티스는 자바 표준 데이터베이스 처리기술인 JDBC를 바탕으로 동작한다는 점에서 Spring JDBC 기술과 비슷합니다. 마치 뿌리는 하나인데 가지가 두개인 모습입니다.

* 마이바티스는 자바 코드와 SQL 문자열을 분리합니다. Spring JDBC는 SQL 문자열을 자바 코드와 같은 메소드 안에서 작성합니다. 마이바티스에서는 별도의 매퍼 파일에 SQL 구문을 작성하고 DAO에서 필요할 때 가져와서 사용합니다.

* 개발자가 작성 할 코드가 줄어들어 생산성이 향상됩니다.

* SQL 문자열을 변경할 때 매퍼 파일만 변경하면 되므로 유지보수성이 향상됩니다. DAO 파일은 변경할 필요가 없습니다.

## 마이바티스 표현식

매퍼 XML에서 파라미터를 쿼리문에 매핑할 때 마이바티스 표현식을 사용한다. 마이바티스 표현식에는 2가지 종류가 있으면 쿼리문을 캐싱하는 방식에서 약간의 차이점이 있다. 자주 사용하는 쿼리문을 캐싱하여 재활용을 극대화하기 위한 선택을 하면 된다.

**1. ${} : EAGER 매핑**

```xml
  select * from emp where empno=${empno}
  // select * from emp where empno=1 형태로 쿼리문이 캐싱된다.
```

**2. #{} : LAZY 매핑**

```xml
  select * from emp where empno=#{empno}
  // select * from emp where empno=? 형태로 쿼리문이 캐싱된다.
```

마이바티스 표현식은 다음처럼 파라미터 값을 SQL 구문에 매핑할 수 있다.

1. #{단일 값인 파라미터의 이름} 
2. #{객체형인 파라미터의 멤버변수명} 
3. #{Map 자료형인 객체가 가진 요소의 키값} 
4. #{`@Param` 애노테이션으로 설정한 이름}

***

# 첫 번째 프로젝트

## 프로젝트 생성

```
STS >> New >> Spring Starter Project

Project Name : spring-mybatis-1
Type : Maven
Packaging : Jar
Java : 8
Package : com.example.demo

Project Dependencies : Web, Lombok, MyBatis, H2
```

마이바티스 개발자들의 노력으로 스프링 부트 디펜던시 목록에 마이바티스가 추가되었습니다. 간단하게 디펜던시를 선택하는 것 만으로 설정작업이 자동으로 처리됩니다. 이전에는 마이바티스를 위한 환경설정을 개발자가 직접 처리해야 스프링 부트 프로젝트에서 사용할 수 있었습니다.

보다 자세한 내용은 다음 사이트를 참고하세요.
http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

## 환경설정

#### application.properties

```properties
# DataSource(JDBC Connection) Configuration
spring.datasource.url=jdbc:h2:mem:TESTDB;DB_CLOSE_DELAY=-1;
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

#spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Use schema.sql, data.sql
spring.datasource.initialize=true

# SQL Holder File
mybatis.mapper-locations=/mybatis/mapper/*-mapper.xml
```

#### schema.sql

```sql
drop table if exists emp;

create table emp (
	empno bigint identity not null primary key,
	ename varchar(100),
	job varchar(100),
	sal bigint
);
```

#### data.sql

```sql
insert into emp(ename, job, sal) values('홍길동', '도적', 800);
insert into emp(ename, job, sal) values('일지매', '의적', 700);
insert into emp(ename, job, sal) values('임꺽정', '산적', 900);
```

## 도메인 클래스

#### Emp.java

```java
package com.example.demo.domain;

import lombok.Data;

@Data
public class Emp {
	private int empno;
	private String ename;
	private String job;
	private double sal;
}
```

## DAO 클래스 : Annotation으로 SQL 설정 

`@Mapper` 어노테이션을 인터페이스 클래스에 붙이면 마이바티스가 인터페이스 구현체를 만들고 그 객체를 빈 컨테이너에 등록합니다. 인터페이스를 작성한 것 만으로 개발자가 해야 할 작업은 끝났습니다. 나머지는 마이바티스와 스프링이 협력해서 개발자 대신 처리해 줄 것입니다. 마이바티스가 만드는 인터페이스 구현체의 로직은 정적이지 않고 동적입니다. 이 말에 뜻은 메소드가 사용하는 쿼리는 실제로 메소드가 호출되는 시점에 구해서 사용된다는 얘기입니다. 따라서, 객체를 빈 컨테이너에 등록하는 시점에 메소드가 사용하는 쿼리문이 없어도 예외는 발생하지 않습니다.

#### EmpDao.java

```java
package com.example.demo.dao;

import java.util.List;

import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

import com.example.demo.domain.Emp;

// 인터페이스의 구현체 역할을 수행할 수 있는 객체(일종의 프록시)를 
// 마이바티스가 빈 컨테이너에 등록합니다.
@Mapper
public interface EmpDao {
	// 리턴자료형 int: 작업결과로 영향받은 row의 개수를 의미합니다.
//	@Insert("insert into EMP (ename, job, sal) values (#{ename}, #{job}, #{sal})")
//	@SelectKey(statement = "select LAST_INSERT_ID()", 
//		before = false, keyProperty = "empno", resultType = Integer.class)
	public int insert(Emp emp);

//	@Update("update EMP set ename=#{ename}, job=#{job}, sal=#{sal} where empno=#{empno}")
	public int update(Emp emp);

//	@Delete("delete from EMP where empno=#{empno}")
	public int delete(int empno);
	
	// 마이바티스가 제공하는 애노테이션으로 사용할 SQL을 설정할 수 있다.
	@Select("select * from emp order by empno asc")
/*
	- emp-mapper.xml에 정의한 <resultMap id="empRowMapper">을 사용할 수 있다.
	- 클래스내에서 정의한 @Results(id="empRowMapper")를 사용할 수 있다.
	@ResultMap("empRowMapper")
 */
	public List<Emp> findAll();

//	@Select("select count(*) from emp")
	public int count();

	/*
	 * 1. 칼럼명들 == 멤버변수명들, 마이바티스 알아서 처리
	 * 2. 칼럼명 != 멤버변수명, 마이바티스에게 매핑로직을 알려주어야 한다.
	 */
//	@Select("select * from emp where empno=#{empno}")

//	// #1: 생략가능, 생략하면 이 방식을 사용한다.
//	//@ResultType(Emp.class) 

//	// #2: 칼럼명과 멤버변수명이 상이할 때, 명시적으로 매핑방법을 설정한다.
//	@Results(id="empRowMapper", value={
//		@Result(property="empno", column="empno"),
//		@Result(property="ename", column="ename"),
//		@Result(property="job", column="job"),
//		@Result(property="sal", column="sal"),
//	})
	public Emp findOne(int empno);
}
```

findAll() 메소드가 사용해야 할 SQL 구문을 `@Select`으로 설정한다. `@Mapper` 애노테이션을 설정했기 때문에 빈 컨테이너에 EmpDao 자료형의 객체가 존재하고 이를 주입받아서 사용할 수 있다. findAll() 메소드는 제대로 작동하지만 다른 메소드들은 사용해야 할 SQL 구문을 설정하지 않았기 때문에 제대로 작동하지 않을 것이다.

일반적으로 SQL 쿼리는 예제처럼 짧지 않기 때문에 별도의 Mapper XML에 쿼리를 작성해 놓고 해당 인터페이스와 연계되어 사용되도록 설정한 후 사용하는 것을 선호합니다. SQL 문자열을 자바코드와 분리하기 위해서라도 애노테이션 방식보다는 Mapper XML 파일을 사용하는 것이 유리하다.

#### TEST : EmpDaoTest.java

```java
package com.example.demo.dao;

import static org.junit.Assert.fail;

import java.util.List;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.example.demo.domain.Emp;

@RunWith(SpringRunner.class)
@SpringBootTest
public class EmpDaoTest {
//	@Autowired
	@Resource(name="empDao")
	private EmpDao dao;

	@Test
	public void testFindAll() {
		List<Emp> emps = dao.findAll();
		System.out.println(emps.size());
		
		for (Emp emp : emps) {
			System.out.println(emp);
		}
	}
}
```

testFindAll() 메소드는 성공할 것이다. 하지만, testCount() 메소드는 실패할 것이다. 왜냐하면 해당 메소드 처리 시 사용해야 할 SQL 구문을 설정하지 않았기 때문이다.

EmpDao 인터페이스의 메소드들이 사용해야 할 SQL 구문을 애노테이션으로 설정해도 되지만 별도의 XML파일에 설정할 수도 있다. 이어서 매퍼 파일을 작성해 보자.

## Mapper XML로 SQL 설정

#### emp-mapper.xml

`src\main\resources\mybatis\mapper\emp-mapper.xml` 위치에 만든다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.dao.EmpDao">

<!-- 
	EmpDao 인터페이스에 정의된 추상메소드 정보를 바탕으로 매퍼설정을 진행한다.
	public int insert(Emp emp);
	public int update(Emp emp);
	public int delete(int empno);
	
	인터페이스의 풀패스 정보를 namespace 속성의 값으로 설정한다.
	메소드명을 id 속성의 값으로 설정한다.
	메소드가 받는 파라미터의 자료형을 parameterType 속성의 값으로 설정한다.
 -->

	<!-- 
		insert/update/delete 수행결과인 영향받은 로우의 개수 값은 자동으로 리턴된다.
	 -->
	<insert id="insert" parameterType="com.example.demo.domain.Emp">
		insert into emp(ename, job, sal) values(#{ename}, #{job}, #{sal})
		<!-- 
			insert 작업을 수행하는 본 쿼리가 수행된 후에 
			디비가 제너레이트 한 키 값이 무엇인지 알고 싶다.
			이 때, selectKey 태그로 제너레이트 된 키 값을 구하고
			그 값을 파라미터로 받은 객체의 멤버변수에 저장한다.
			파라미터로 받은 객체는 호출 측에서 넘겨준 객체이기 때문에
			호출 측에서 키 값을 꺼내어 사용할 수 있게 된다.
		 -->
		<selectKey resultType="int" order="AFTER" keyProperty="empno">
			select LAST_INSERT_ID() as empno
		</selectKey>
	</insert>

	<update id="update" parameterType="com.example.demo.domain.Emp">
		update emp set ename=#{ename}, job=#{job}, sal=#{sal} where empno=#{empno}
	</update>
	
	<delete id="delete" parameterType="int">
		delete from emp where empno=#{empno}
	</delete>

<!-- 
	public List<Emp> findAll();
	
	- 칼럼명들 == 멤버변수명들, 마이바티스가 알아서 처리
	- 칼럼명 != 멤버변수명, 별도로 매핑 로직을 명시적으로 제공해야 함

	<select id="findAll" resultMap="empRowMapper">
		select * from emp order by empno asc
	</select>
 -->
 
<!-- 
	public int count();
	public Emp findOne(int empno);
	메소드의 리턴타입을 resultType 속성에 설정한다.
 -->

	<!-- 조회 쿼리는 결과를 어떻게 처리해서 리턴할 지 설정해야 한다. -->
	<select id="count" resultType="int">
		select count(*) from emp
	</select>

	<!-- 
		하나의 값이 아니라 복수의 값을 묶어서 리턴하기 위해서 
		도메인 클래스를 설정한다. 칼럼명과 멤버변수명이 같은 경우 
		resultType 속성에 Emp 도메인 클래스를 설정하면 된다.
	 -->
	<select id="findOne" parameterType="int" resultType="com.example.demo.domain.Emp">
		select * from emp where empno=#{empno}
	</select>

</mapper>
```

`<mapper namespace="com.example.demo.dao.EmpDao">`

매퍼 XML을 마이바티스가 인식하도록 알려줄 필요가 있습니다. 네임 스페이스에 인터페이스
를 풀패스로 지정하면 마이바티스는 이 파일 안에 담긴 쿼리들을 어떤 인터페이스의 추상메
소드들이 사용해야 하는지 알게 됩니다.

`<select id="count" resultType="int">`

id로 추상 메소드명을 그대로 사용해서 설정하면 해당 메소드가 사용해야 하는 쿼리가 무엇
인지 파악하게 됩니다. 더불어서 select 태그와 resultType의 자료형으로 마이바티스는 쿼리의 종류를 파악하고 쿼리 결과를 어떻게 처리해야 하는지 판단할 수 있습니다.

`<select id="findOne" parameterType="int" resultType="Emp">`

resultType의 자료형으로 모델클래스를 사용할 수 있습니다. 클래스의 풀패스를 써야 합니다. 풀패스를 클래스명만 짧게 써도 인식되게 만들려면 마이바티스에게 미리 알려주어야 합니다. 패키지명까지 써야 하는 모델클래스 이름을 클래스명만으로 간단하게 줄여 쓰기 위한 앨리어스 설정을 스프링 부트 설정파일 application.properties에 추가합니다.

```properties
mybatis.type-aliases-package=com.example.demo.domain
```

type-aliases-package 설정으로 해당 패키지에 있는 모든 클래스들은 매퍼 XML에서 클래스명만으로 설정해도 마이바티스는 풀패스가 무엇인지 파악할 수 있게 됩니다. 어노테이션 방식으로 SQL 쿼리를 알려주고 사용할 때는 mybatis.type-aliases-package 설정은 필요하지 않습니다.

* 앞서서 작성한 매퍼파일을 마이바티스가 인식할 수 있도록 `application.properties` 파일에 `mybatis.mapper-locations=/mybatis/mapper/*-mapper.xml` 설정을 추가한다.
* `/mybatis/mapper/*-mapper.xml` 패스 앞에 존재하는 `spring-mybatis/src/main/resources` 패스는 생략한다.
* `*` 기호를 사용하여 다수의 파일을 인식하도록 할 수 있다.
* `**` 기호를 사용하여 다수의 폴더를 인식하도록 할 수 있다.

EmpDao 인터페이스 안에 정의된 모든 메소드가 사용해야 할 SQL 구문을 설정했으니 잘 작동하는지 테스트를 수행할 차례다.

#### TEST : EmpDaoTest.java

```java
package com.example.demo.dao;

import static org.junit.Assert.fail;

import java.util.List;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.example.demo.domain.Emp;

@RunWith(SpringRunner.class)
@SpringBootTest
public class EmpDaoTest {
//	@Autowired
	@Resource(name="empDao")
	private EmpDao dao;

	@Test
	public void testInsert() {
		fail("Not yet implemented");
	}

	@Test
	public void testUpdate() {
		Emp emp = new Emp();
		emp.setEname("Tom");
		emp.setJob("Programmer");
		emp.setSal(700);
		
		System.out.println("전: " + emp);
		
		int affected = dao.insert(emp);
		System.out.println("affected = " + affected);
		
		// 수정을 위해서는 키 값인 empno 값이 필요합니다.
		// 별다른 작업을 하지 않는다면 여전히 0 값이 상태이고 
		// 이러면 수정할 수 없습니다.
		
		System.out.println("후: " + emp);
		
		emp.setJob("Designer");
		emp.setSal(900);
		
		affected = dao.update(emp);
		System.out.println("affected = " + affected);
		
		Emp e = dao.findOne(emp.getEmpno());
		System.out.println(e);
		
		affected = dao.delete(emp.getEmpno());
		System.out.println("affected = " + affected);
	}

	@Test
	public void testDelete() {
		fail("Not yet implemented");
	}

	@Test
	public void testFindAll() {
		List<Emp> emps = dao.findAll();
		System.out.println(emps.size());
		
		for (Emp emp : emps) {
			System.out.println(emp);
		}
	}

	@Test
	public void testCount() {
		int count = dao.count();
		System.out.println(count);
	}

	@Test
	public void testFindOne() {
		int empno = 1;
		Emp emp = dao.findOne(empno);
		System.out.println(emp);
	}

}
```

## SqlSessionTemplate

`@Mapper` 애노테이션을 사용하지 않고 개발자가 직접 인터페이스 구현 클래스를 작성한 다음 마이바티스가 제공하는 SqlSessionTemplate을 사용해서 처리할 수도 있다. 과거에 만든 프로젝트에서 많이 사용하던 방식이다.

이를 확인하기 위해서 다음 코드를 작성하자.

#### EmpDaoImpl.java

```java
package com.example.demo.dao;

import java.util.List;

import org.apache.ibatis.session.SqlSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.example.demo.domain.Emp;

@Repository
public class EmpDaoImpl implements EmpDao {
	@Autowired
	private SqlSession sqlSession;

	@Override
	public int insert(Emp emp) {
		return session.insert(
				"com.example.demo.dao.EmpDao.insert", emp);
	}

	@Override
	public int update(Emp emp) {
		return session.update(
				"com.example.demo.dao.EmpDao.update", emp);
	}

	@Override
	public int delete(int empno) {
		return session.delete(
				"com.example.demo.dao.EmpDao.delete", empno);
	}

	@Override
	public List<Emp> findAll() {
		// "com.example.demo.dao.EmpDao.findAll" 문자열로 
		// SQL 문자열을 가지고 있는 매퍼파일을 찾는다.
		// 리턴자료형을 보고 판단하여 사용해야 할 메소드를 선택한다.
		return sqlSession.selectList("com.example.demo.dao.EmpDao.findAll");
	}

	@Override
	public int count() {
		return session.selectOne(
				"com.example.demo.dao.EmpDao.count");
	}

	@Override
	public Emp findOne(int empno) {
		return session.selectOne(
				"com.example.demo.dao.EmpDao.findOne", empno);
	}

}
```

앞서서 만든 findAll() 메소드가 잘 작동하는지 테스트 해 보자.

#### EmpDaoImplTest.java

```java
package com.example.demo.dao;

import static org.junit.Assert.*;

import java.util.List;

import javax.annotation.Resource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.example.demo.domain.Emp;

@RunWith(SpringRunner.class)
@SpringBootTest
public class EmpDaoImplTest {
//	@Autowired
	@Resource(name="empDaoImpl")
	private EmpDao dao;
	
	@Test
	public void testFindAll() {
		List<Emp> emps = dao.findAll();
		System.out.println(emps.size());
		
		for (Emp emp : emps) {
			System.out.println(emp);
		}
	}

}
```

***

# 두 번째 프로젝트

## 프로젝트 생성

앞에서 작성한 프로젝트를 선택한 후 그대로 복사하여 `spring-mybatis-2` 라는 이름으로 두 번째 마이바티스 프로젝트를 만든다. 코드를 살펴서 변화를 적용한다.

####  EmpDao.java

```java
package com.example.demo.dao;

import java.util.List;

import com.example.demo.domain.Emp;

public interface EmpDao {
	public int insert(Emp emp);
	public int update(Emp emp);
	public int delete(int empno);
	
	public List<Emp> findAll();
	public int count();
	public Emp findOne(int empno);
	
	/*
	 * 테이블의 특정 범위의 로우들만을 조회하고 싶다.
	 * 
	 * 전제: 정렬은 이미 되어 있다. 또는 조회 시 마다 order by 구문을 사용한다.
	 * 기본적으로 거의 대부분의 디비는 PK의 오름차순으로 정렬한 상태로 데이터를 유지한다.
	 */
	
	// start: empno의 시작 값, end: empno의 종료 값
	public List<Emp> findByStartEnd(int start, int end);
	
	// skip: 앞에서부터 얼마나 스킵할 것인지에 대한 값, limit: 구하는 로우의 개수
	public List<Emp> findBySkipLimit(int skip, int limit);
	
	// page: ceil(전체 로우의 개수/size) 후 구해지는 페이지 번호, size: 구하는 로우의 개수
	/*
	 * 전체 로우의 개수 = 21
	 * size = 10
	 * ceil(21/10) = 3
	 */
	public List<Emp> findByPageSize(int page, int size);
}
```

#### emp-mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
	"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.dao.EmpDao">

	<resultMap type="Emp" id="empRowMapper">
		<result property="empno" column="empno"/>
		<result property="ename" column="ename"/>
		<result property="job" column="job"/>
		<result property="sal" column="sal"/>
	</resultMap>

	<insert id="insert">
		insert into EMP (ename, job, sal) 
		values (#{ename}, #{job}, #{sal})
		
		<selectKey order="AFTER" keyProperty="empno" resultType="int" >
			select LAST_INSERT_ID() as empno
		</selectKey>
	</insert>
	
	<update id="update">
		update EMP set ename=#{ename}, job=#{job}, sal=#{sal} 
		where empno=#{empno}
	</update>
	
	<delete id="delete">
		delete from EMP where empno=#{empno}
	</delete>
	
	<select id="findAll" resultMap="empRowMapper">
		select * from emp order by empno asc
	</select>
	
	<select id="count" resultType="int">
		select count(*) from emp
	</select>

	<select id="findOne" resultType="Emp">
		select * from emp where empno=#{empno}
	</select>

	<!-- 
		파라미터 매핑 시 사용할 수 있는 예약어가 존재한다.
		select * from emp order by empno asc limit #{arg0}, #{arg1}
		select * from emp order by empno asc limit #{param1}, #{param2}
		
		인터페이스 추상 메소드 파라미터 부분에서 @Param으로 지정한 키명을 사용할 수 있다.
	 -->
	<select id="findBySkipLimit" resultMap="empRowMapper">
		select * from emp order by empno asc limit #{skip}, #{limit}
	</select>
	
	<select id="findByPageSize" resultMap="empRowMapper">
	<![CDATA[
		select * from emp order by empno asc limit #{skip}, #{limit}
	]]>
	</select>
	
	<!-- 
		select * from emp 
		where empno between #{start} and #{end}
		order by empno asc
		
		select * from emp 
		where empno >= #{start} and empno <= #{end}
		order by empno asc
		between 대신 >, < 기호 사용 시 에러가 발생한다.
		
		해결 방법 2가지
		
		1. 치환기호를 사용한다.
		select * from emp 
		where empno &gt;= #{start} and empno &lt;= #{end}
		order by empno asc
		
		2. CDATA Section으로 감싼다. 
		범위 안에 태그가 존재하지 않는다는 뜻이다.
	 -->
	<select id="findByStartEnd" resultMap="empRowMapper">
	<![CDATA[
		select * from emp 
		where empno >= #{start} and empno <= #{end}
		order by empno asc
	]]>
	</select>

</mapper>
```

#### EmpDaoImpl.java

```java
package com.example.demo.dao;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.ibatis.session.SqlSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.example.demo.domain.Emp;

@Repository
public class EmpDaoImpl implements EmpDao {
	// SqlSessionTemplate
	@Autowired
	private SqlSession session;
	
	@Override
	public int insert(Emp emp) {
		return session.insert(
				"com.example.demo.dao.EmpDao.insert", emp);
	}

	@Override
	public int update(Emp emp) {
		return session.update(
				"com.example.demo.dao.EmpDao.update", emp);
	}

	@Override
	public int delete(int empno) {
		return session.delete(
				"com.example.demo.dao.EmpDao.delete", empno);
	}

	@Override
	public List<Emp> findAll() {
		// "com.example.demo.dao.EmpDao.findAll" 문자열로 
		// SQL 문자열을 가지고 있는 매퍼파일을 찾는다.
		// 리턴자료형을 보고 판단하여 사용해야 할 메소드를 선택한다.
		return sqlSession.selectList("com.example.demo.dao.EmpDao.findAll");
	}

	@Override
	public int count() {
		return session.selectOne(
				"com.example.demo.dao.EmpDao.count");
	}

	@Override
	public Emp findOne(int empno) {
		return session.selectOne(
				"com.example.demo.dao.EmpDao.findOne", empno);
	}
	
	@Override
	public List<Emp> findByStartEnd(int start, int end) {
		Map<String, Integer> map = new HashMap<>();
		map.put("start", start); // 시작
		map.put("end", end); // 끝
	        
		return session.selectList(
			"com.example.demo.dao.EmpDao.findByStartEnd", map);
	}

	@Override
	public List<Emp> findBySkipLimit(int skip, int limit) {
		Map<String, Integer> map = new HashMap<>();
		map.put("skip", skip); // 스킵
		map.put("limit", limit); // 개수
	        
		return session.selectList(
			"com.example.demo.dao.EmpDao.findBySkipLimit", map);
	}

	@Override
	public List<Emp> findByPageSize(int page, int size) {
		int skip = 0;
		if (page > 0) {
			skip = (page - 1) * size;
		}
		
		Map<String, Integer> map = new HashMap<>();
		map.put("skip", skip); // 스킵
		map.put("limit", size); // 개수
        
		return session.selectList(
			"com.example.demo.dao.EmpDao.findByPageSize", map);
	}

}
```

#### TEST : EmpDaoImplTest.java

```java
package com.example.demo.dao;

import static org.junit.Assert.fail;

import java.util.List;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.example.demo.domain.Emp;

@RunWith(SpringRunner.class)
@SpringBootTest
public class EmpDaoImplTest {
	@Autowired
	private EmpDao dao;

	@Test
	public void testInsert() {
		fail("Not yet implemented");
	}

	@Test
	public void testUpdate() {
		fail("Not yet implemented");
	}

	@Test
	public void testDelete() {
		fail("Not yet implemented");
	}

	@Test
	public void testFindAll() {
		System.out.println(dao instanceof EmpDaoImpl);
		dao.findAll().forEach(System.out::println);
	}

	@Test
	public void testCount() {
		fail("Not yet implemented");
	}

	@Test
	public void testFindOne() {
		fail("Not yet implemented");
	}

	@Test
	public void testFindBySkipLimit() {
		int skip = 10;
		int limit = 10;
		List<Emp> emps = dao.findBySkipLimit(skip, limit);
		emps.forEach(System.out::println);
	}
	
	@Test
	public void testFindByPageSize() {
		int page = 2;
		int size = 10;
		List<Emp> emps = dao.findByPageSize(page, size);
		emps.forEach(System.out::println);
	}
	
	@Test
	public void testFindByStartEnd() {
		int start = 11;
		int end = 20;
		List<Emp> emps = dao.findByStartEnd(start, end);
		emps.forEach(System.out::println);
	}
}
```

***

# 레가시 프로젝트의 마이바티스 빈 설정

스프링 레가시 프로젝트에서는 개발자가 직접 마이바티스와 관련한 빈 설정을 해야 한다. 트랜잭션 처리는 스프링이 맡는다.

#### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<setting name="cacheEnabled" value="true"/>
		<setting name="mapUnderscoreToCamelCase" value="true" />
		<setting name="defaultFetchSize" value="100" />
		<setting name="defaultStatementTimeout" value="30" />
	</settings>

	<typeAliases>
		<package name="com.example.board.model" />
		<package name="com.example.user.model" />
	</typeAliases>
</configuration>
```

설정이 직관적이라서 코드를 보면 해당 설정이 무엇을 위한 것인지 파악할 수 있다.

#### root-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="
	http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context 
	http://www.springframework.org/schema/context/spring-context-4.3.xsd
	http://www.springframework.org/schema/tx 
	http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">

	<context:property-placeholder
		location="classpath:jdbc.properties" />

	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName"
			value="${jdbc.driverClassName}" />
		<property name="url" value="${jdbc.url}" />
		<property name="username" value="${jdbc.username}" />
		<property name="password" value="${jdbc.password}" />
	</bean>

	<bean id="SqlSessionFactory"
		class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="configLocation"
			value="classpath:/mybatis/mybatis-config.xml" />
		<property name="mapperLocations"
			value="classpath:/mybatis/mapper/**/*.xml" />
	</bean>

	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage"
			value="com.example.board.repository,com.example.user.repository" />
	</bean>

	<bean id="sqlSession"
		class="org.mybatis.spring.SqlSessionTemplate"
		destroy-method="clearCache">
		<constructor-arg name="sqlSessionFactory"
			ref="SqlSessionFactory" />
	</bean>

	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>

	<tx:annotation-driven
		transaction-manager="transactionManager" />
</beans>
```

***

# 정리

마이바티스는 SQL 쿼리를 분리할 수 있는 유용한 기술입니다. 클래스 안에 쿼리를 기술하는 경우, 이를 수정하려면 코드를 이해하는 개발자가 필요하겠지만 별도의 파일로 분리해 놓았다면 굳이 개발자가 아니더라도 수정할 수 있게 됩니다.

마이바티스는 DAO 인터페이스의 추상메소드의 파라미터, 리턴자료형, SQL 쿼리의 종류를 보고서 처리해야 하는 로직을 판단할 수 있으므로 개발자 대신 인터페이스 구현체를 만들어 주는 서비스를 제공합니다. 

따라서 마이바티스를 사용하면 개발자가 해야 하는 업무는 DAO 인터페이스 생성, 추상 메소드 작성, 추상 메소드가 사용하는 쿼리 지정 입니다. 개발자가 구현 클래스를 직접 만들지 않아도 되는 점에서 Spring JDBC 보다 편리합니다.
