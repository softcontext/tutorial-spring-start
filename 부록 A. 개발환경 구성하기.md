<style>
body, p, li {
   font-size: 15px;
}
</style>

***
  
# 부록 A. 개발환경 구성하기

## Java

1. Java 설치
    1. 검색 : `java 8 download`
    2. 오라클 자바사이트 접속 후 다운로드
    3. 디폴트 설정으로 설치
    4. 설치확인 : 콘솔 >> `java -version`
<br/>
2. 윈도우 전역패스 환경설정
    1. 제어판\시스템 및 보안\시스템 >> `고급 시스템 설정` 클릭
    2. 시스템 속성 창 >> `환경변수` 클릭
    3. 시스템 변수 >> `새로 만들기` 클릭
    4. 시스템 변수 편집 창에서 다음 변수 등록
    변수 이름 : `JAVA_HOME`
    변수 값 : `C:\Program Files\Java\jdk1.8.0_111`
    주의 : `변수 값`은 설치된 자바의 패스정보를 사용하세요.
    5. Path 선택 >> `편집` 클릭
    6. '새로 만들기' >> `%JAVA_HOME%\bin` 등록
    7. `%JAVA_HOME%\bin` 선택 >> `위로 이동` 클릭하여 맨 위로 이동
    8. 설정확인 : 콘솔 >> `java -version`

## IDE Tool

1. STS 설치
    1. 검색 : `sts download`
    2. 스프링 사이트 접속 후 OS에 맞게 다운로드
    3. 압축해제
    4. \sts-bundle\sts-3.8.3.RELEASE\STS.exe 클릭
<br/>
2. 플러그인 설치
    1. Help >> Eclipse Marketplace
    2. SQL Development Tools 설치
    3. MyBatipse 설치
