## Nest CLU로 파일 생성하기

### 기본 커맨드

```shell
nest generate [factor] [name]
```

#### 한번에 생성하기
: 서비스, 컨트롤러, 모듈, DTO

```shell
nest generate resource [name]
nest g res [name]  # Alias
```

#### 옵션
- `--no-spec` : 테스트 코드 미생성
- `--flat` : 루트에 생성
- `--dry-run` : 파일 생성 테스트

### 요소 별 생성 커맨드

| 구성 요소     | 전체 커맨드                 | 약어 (Alias) | 설명                          |
| :----------- | :------------------------ | :----------- | :--------------------------- |
| Controller   | nest generate controller  | nest g co    | 라우트 처리 및 요청/응답 제어      |
| Service      | nest generate service     | nest g s     | 비즈니스 로직 및 데이터 처리       |
| Module       | nest generate module      | nest g mo    | 관련 컴포넌트들을 묶는 단위        |
| Middleware   | nest generate middleware  | nest g mi    | 요청 처리 전/후 로직 수행         |
| Pipe         | nest generate pipe        | nest g pi    | 데이터 변형 및 유효성 검사         |
| Guard        | nest generate guard       | nest g gu    | 인증 및 권한 부여                |
| Interceptor  | nest generate interceptor | nest g itc   | 요청/응답 가로채기 및 로직 추가     |
| Decorator    | nest generate decorator   | nest g d     | 커스텀 데코레이터 생성            |
| Resource     | nest generate resource    | nest g res   | CRUD 세트(전체) 한 번에 생성      |
