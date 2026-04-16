## Devtools란?

Spring boot에서 제공하는 개발 편의를 위한 모듈

코드가 변경 시 자동으로 어플리케이션을 재시작하여 브라우저에도 업데이트를 해준다.

    주요 기능

    1. Property Defaults: 개발 설정 자동화
        - 캐시 설정을 자동으로 false로 바꿔줌 
        
    2. Automatic Restart: 자동 재시작
        - 바뀐 클래스 파일만 재로드(Restart Classloader)

    3. Live Reload: 라이브 리로드
        - 리소스(HTML, CSS, JS 등)가 변경되었을 때 자동으로 새로고침

    4. Global Settings: 전역 설정

    5. Remote Applications: 원격 애플리케이션 지원


### 의존성 추가

```gradle
dependencies {
    compileOnly("org.springframework.boot:spring-boot-devtools")
}
```

### 결론

전체 애플리케이션을 처음부터 다시 시작하는 것보다 훨씬 적은 시간이 소요되어 로직을 수정하고 결과를 확인하기까지의 불필요한 대기 시간을 획기적으로 줄여준다. 또한 개발 환경에 최적화된 설정을 자동으로 해주어 번거로운 설정을 일일이 할 필요가 없으므로 개발 시간을 획기적으로 줄이는 데 도움이 될 것 같다.