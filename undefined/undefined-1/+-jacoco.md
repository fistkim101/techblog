# (+) jacoco

개인적으로 단순히 테스트 커버리지 퍼센트가 높다고 해서 '테스트 코드가 잘 짜여져 있다'고 말할수는 없다고 생각한다. jacoco 가 실제로 많은 도움이 되는지는 잘 모르겠다. 개인적으로 실무에서도 하나의 프로젝트에 적용되어 있었는데, 특별히 일정 퍼센트 밑으로 빌드가 실패하게 한다던가 하는 식으로 사용하지도 않았었다.

일단 예전에 실습으로 세팅해둔 코드가 있어서 여기 첨부한다. [gradle document](https://docs.gradle.org/current/userguide/jacoco\_plugin.html) 에도 안내가 되어있다.

```gradle
plugins {
    id 'org.springframework.boot' version '2.4.5'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
    id 'jacoco'
}

group = 'me.fistkim101'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
    mavenCentral()
}

jacoco {
    toolVersion = "0.8.6"
}

jacocoTestReport {
    reports {
        xml.enabled false
        csv.enabled false
        html.destination layout.buildDirectory.dir('jacocoHtml').get().asFile
    }

    dependsOn test // tests are required to run before generating the report
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    implementation group: 'mysql', name: 'mysql-connector-java', version: '8.0.22'
    testImplementation group: 'org.testcontainers', name: 'mysql', version: '1.15.3'
    testImplementation group: 'org.testcontainers', name: 'junit-jupiter', version: '1.15.2'

}

test {
    useJUnitPlatform()
    finalizedBy jacocoTestReport
}
```

이렇게 해두면 test 라는 Task 가 jacocoTestReport에 연결된다. 즉 test를 실행하면 test -> jacocoTestReport 로 task의 의존성이 맺어져서 자동으로 흐름이 이어진다. 그래서 test를 할 때마다 build > jacocoHtml > index.html 이 새로 갱신되고, 여기서 코드 커버리지를 알 수 있다.

<figure><img src="http://localhost:4000/assets/images/spring/code-coverage.png" alt=""><figcaption></figcaption></figure>

원리는 jacoco가 바이트코드를 읽어서 코드 커버리지로 체크해야할 포인트를 세어 두고 테스트시 해당 포인트를 지나갔는지를 체크해서 이걸 백분율로 보여주는 것이다.

> 1. 프로젝트 빌드 시 JaCoCo 플러그인 또는 설정을 적용합니다. 이를 통해 컴파일된 클래스 파일에 JaCoCo의 에이전트(agent)를 적용할 수 있습니다.
> 2. JaCoCo 에이전트는 JVM에 연결되어 클래스 파일이 로드되는 시점에 바이트 코드를 조작합니다. 이 때, 추가적인 코드를 삽입하여 코드 커버리지 정보를 수집하는 데 사용됩니다.
> 3. 프로그램이 실행되는 동안 JaCoCo 에이전트는 코드 커버리지 정보를 수집하고, 이를 보고서로 출력하거나 다른 도구와 통합하여 분석할 수 있습니다.

<figure><img src="http://localhost:4000/assets/images/spring/code-coverage-jacoco.png" alt=""><figcaption></figcaption></figure>

이렇게 거쳐간 부분은 초록색, 거쳐가지 않은 부분은 빨간색으로 코드를 line by line 으로 보여준다. 노란색은 분기문에서 일부만 테스트가 되었을 때에 노란색 다이아몬드가 나온다.
