# CH01 프로젝트 환경구성

실습을 위한 환경구성만 했음.

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/fistkim
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        format_sql: true
        show_sql: true

logging:
  level:
    org.hibernate.sql: debug
    org.hibernate.type: trace
```

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.10'
    id 'io.spring.dependency-management' version '1.0.15.RELEASE'

    //querydsl 추가
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    //querydsl 추가
    implementation 'com.querydsl:querydsl-jpa'
    implementation 'com.querydsl:querydsl-apt:5.0.0'
    implementation 'com.mysql:mysql-connector-j:8.0.32'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}

//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

sourceSets {
    main.java.srcDir querydslDir
}

configurations {
    querydsl.extendsFrom compileClasspath
}

compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
//querydsl 추가 끝
```



강의에서는 위와 같이 잡았다. 아래는 내가 예전에 백기선님 강의 들을때 설정했던것 다시 첨부한다. 이제보니 똑같다.

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.6'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'com.ewerk.gradle.plugins.querydsl' version "1.0.10"
}

group = 'com.fistkim'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
    mavenCentral()
}

dependencies {

    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'mysql:mysql-connector-java:8.0.30'
    implementation 'com.querydsl:querydsl-jpa'
    implementation 'com.querydsl:querydsl-apt:5.0.0'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}

// querydsl 사용할 경로 지정합니다. 현재 지정한 부분은 .gitignore에 포함되므로 git에 올라가지 않습니다.
def querydslDir = "$buildDir/generated/'querydsl'"

// JPA 사용여부 및 사용 경로 설정
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

// build시 사용할 sourceSet 추가 설정
sourceSets {
    main.java.srcDir querydslDir
}

// querydsl 컴파일 시 사용할 옵션 설정
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}

// querydsl이 compileClassPath를 상속하도록 설정
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    querydsl.extendsFrom compileClasspath
}
```



## Spring 3.x 에서 설정(추가)

### mysql-library 모듈

```gradle
dependencies {
    api 'org.springframework.boot:spring-boot-starter-data-jpa'
    api 'javax.persistence:javax.persistence-api:2.2'
    api 'mysql:mysql-connector-java:8.0.33'
}
```

### 사용하는 모듈(ex. -domain)

```gradle
dependencies {

    implementation project(':common-web-support')
    implementation project(':auth-support')
    implementation project(':aws-support')
    implementation project(':mysql-library')
    implementation project(':shared')

    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor 'com.querydsl:querydsl-apt:5.0.0:jakarta'
    annotationProcessor 'jakarta.annotation:jakarta.annotation-api'
    annotationProcessor 'jakarta.persistence:jakarta.persistence-api'

    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

def querydslDir = "$buildDir/generated/querydsl"

sourceSets {
    main.java.srcDirs += [ querydslDir ]
}

tasks.withType(JavaCompile) {
    options.annotationProcessorGeneratedSourcesDirectory = file(querydslDir)
}

clean.doLast {
    file(querydslDir).deleteDir()
}

```

### 멀티모듈 최상위 build.gradle

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.0.6'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'com.ewerk.gradle.plugins.querydsl' version '1.0.10'
    id 'idea'
    id 'java-library'
}

subprojects {
    apply plugin: 'java'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'
    apply plugin: 'idea'
    apply plugin: 'java-library'

    dependencies {
        implementation 'org.projectlombok:lombok:1.18.22'
        annotationProcessor 'org.projectlombok:lombok'
        annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
    }

    sourceCompatibility = '17'

    test {
        useJUnitPlatform()
    }

    task prepareKotlinBuildScriptModel {

    }

    repositories {
        mavenCentral()
    }

}

repositories {
    mavenCentral()
}
```
