***

# 부록 C. MariaDB 설치방법

## 1. MariaDB 설치

#### MariaDB 소개

MariaDB는 오픈 소스의 관계형 데이터베이스 관리 시스템(RDBMS)이다. MySQL과 동일한 소스 코드를 기반으로 하며, GPL v2 라이선스를 따른다. 오라클 소유의 현재 불확실한 MySQL의 라이선스 상태에 반발하여 만들어졌으며, 배포자는 몬티 프로그램 AB(Monty Program AB)와 저작권을 공유해야 한다. 이것은 MySQL과 높은 호환성을 유지하기 위함이며, MySQL APIs와 명령에 정확히 매칭하여, 라이브러리 바이너리와 상응함을 제공하여 교체 가능성을 높이고자 함이다. 마리아 DB에는 새로운 저장 엔진인 아리아(Aria)뿐만 아니라, InnoDB를 교체할 수 있는 XtraDB 저장 엔진을 포함하고 있다. 이것은 트랜잭션과 비트랜잭션 엔진 그리고 미래에 나올 MySQL 판에 대응하고자 함일 것이다.

마리아 DB의 주요 개발자는 MySQL과 몬티 프로그램 AB를 설립한 마이클 몬티 와이드니어스(Michael Monty Widenius)이다. 그는 이전에 자신의 회사, MySQL AB를 썬마이크로시스템즈에 10억 달러에 판매를 한 적이 있으며, 마리아 DB는 그의 둘째 딸인 마리아의 이름을 딴 것이다.

#### 다운로드

사이트: `https//download.mariadb.org`

#### 설치

~.msi 파일을 다운받아 설치하면 인스톨 위저드를 지원하므로 편리하다.
~.zip 파일을 다운받아 설치하는 경우 온라인 문서를 참고하자.

#### 한글처리

만약 한글이 깨진다면 한글처리를 위해 데이터베이스가 설치된 폴더 밑으로 존재하는 my.ini 파일을 수정한다. 리눅스일 경우 설정파일은 my.cnf 파일이다.

```ini
[mysqld]
datadir=c:/패스/디비폴더
init_connect = SET collation_connection = utf8_general_ci
init_connect = SET NAMES utf8
character-set-server = utf8
collation-server = utf8_general_ci

[client]
default-character-set = utf8

[mysqldump]
default-character-set = utf8

[mysql]
default-character-set = utf8
```

## 2. HeidiSQL 설치

DB에 접속해서 데이터 등을 확인하기 위해서 사용하는 GUI 클라이언트 프로그램이다.

#### 다운로드

사이트: `https://www.heidisql.com/download.php`

다운로드 받은 파일을 클릭하여 디폴트 설정으로 설치한다. 특별히 변경해야 할 설정은 없
다. 설치 후, 마리아디비 설치 시 정한 패스워드를 사용하여 접속한다.

## 3. 테스트 용 테이블 생성

다음 쿼리는 MySQL, MariaDB에서 사용할 수 있다. 연습을 위한 테스트 용도의 테이블 및 데이터이다.

#### schema.sql

```sql
CREATE TABLE IF NOT EXISTS `dept` (
  `deptno` INT(11) NOT NULL,
  `dname` VARCHAR(14) NOT NULL,
  `loc` VARCHAR(13) NULL DEFAULT NULL,
  PRIMARY KEY (`deptno`)
) ENGINE=InnoDB;

CREATE TABLE IF NOT EXISTS `emp` (
  `empno` INT(11) NOT NULL,
  `comm` DOUBLE NULL DEFAULT NULL,
  `ename` VARCHAR(10) NOT NULL,
  `hiredate` DATE NULL DEFAULT NULL,
  `job` VARCHAR(9) NULL DEFAULT NULL,
  `sal` DOUBLE NULL DEFAULT NULL,
  `deptno` INT(11) NULL DEFAULT NULL,
  `mgr` INT(11) NULL DEFAULT NULL,
  PRIMARY KEY (`empno`),
  INDEX `FKfqt0j25nlvjwt7qt1t3x7v6qf` (`deptno`),
  INDEX `FKfehivfm7m674r8qrrnug1of2q` (`mgr`),
  CONSTRAINT `FKfehivfm7m674r8qrrnug1of2q` FOREIGN KEY (`mgr`)
  REFERENCES `emp` (`empno`),
  CONSTRAINT `FKfqt0j25nlvjwt7qt1t3x7v6qf` FOREIGN KEY (`deptno`)
  REFERENCES `dept` (`deptno`)
) ENGINE=InnoDB;

CREATE TABLE IF NOT EXISTS `salgrade` (
  `grade` INT(11) NOT NULL,
  `hisal` DOUBLE NOT NULL,
  `losal` DOUBLE NOT NULL,
  PRIMARY KEY (`grade`)
) ENGINE=InnoDB;
```

#### data.sql

```sql
INSERT INTO SALGRADE(grade, losal, hisal) VALUES (1, 700,1200);
INSERT INTO SALGRADE(grade, losal, hisal) VALUES (2,1201,1400);
INSERT INTO SALGRADE(grade, losal, hisal) VALUES (3,1401,2000);
INSERT INTO SALGRADE(grade, losal, hisal) VALUES (4,2001,3000);
INSERT INTO SALGRADE(grade, losal, hisal) VALUES (5,3001,9999);

INSERT IGNORE INTO DEPT(deptno, dname, loc) VALUES (10,'ACCOUNTING','NEW YORK');
INSERT IGNORE INTO DEPT(deptno, dname, loc) VALUES (20,'RESEARCH','DALLAS');
INSERT IGNORE INTO DEPT(deptno, dname, loc) VALUES (30,'SALES','CHICAGO');
INSERT IGNORE INTO DEPT(deptno, dname, loc) VALUES (40,'OPERATIONS','BOSTON');

INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7839,'KING','PRESIDENT', NULL,STR_TO_DATE('17-11-1981','%d-%m-
%Y'),5000,NULL,10);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7566,'JONES','MANAGER', 7839,STR_TO_DATE('2-4-1981', '%d-%m-
%Y'),2975,NULL,20);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7698,'BLAKE','MANAGER', 7839,STR_TO_DATE('1-5-1981', '%d-%m-
%Y'),2850,NULL,30);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7782,'CLARK','MANAGER', 7839,STR_TO_DATE('9-6-1981', '%d-%m-
%Y'),2450,NULL,10);

INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7902,'FORD','ANALYST', 7566,STR_TO_DATE('3-12-1981', '%d-%m-
%Y'),3000,NULL,20);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7788,'SCOTT','ANALYST', 7566,STR_TO_DATE('13-07-1987', '%d-%m-
%Y'),3000,NULL,20);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno) 
VALUES(7499,'ALLEN','SALESMAN', 7698,STR_TO_DATE('20-2-1981', '%d-%m-
%Y'),1600,300,30);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7521,'WARD','SALESMAN', 7698,STR_TO_DATE('22-2-1981', '%d-%m-
%Y'),1250,500,30);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7654,'MARTIN','SALESMAN',7698,STR_TO_DATE('28-9-1981', '%d-%m-
%Y'),1250,1400,30);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7844,'TURNER','SALESMAN',7698,STR_TO_DATE('8-9-1981', '%d-%m-
%Y'),1500,0,30);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7900,'JAMES','CLERK', 7698,STR_TO_DATE('3-12-1981', '%d-%m-
%Y'),950,NULL,30);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7934,'MILLER','CLERK', 7782,STR_TO_DATE('23-1-1982', '%d-%m-
%Y'),1300,NULL,10);

INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7369,'SMITH','CLERK', 7902,STR_TO_DATE('17-12-1980','%d-%m-
%Y'),800,NULL,20);
INSERT IGNORE INTO EMP(empno, ename, job, mgr, hiredate, sal, comm, deptno)
VALUES(7876,'ADAMS','CLERK', 7788,STR_TO_DATE('13-07-1987', '%d-%m-
%Y'),1100,NULL,20);
```
