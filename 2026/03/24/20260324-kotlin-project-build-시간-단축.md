# 코틀린 프로젝트 빌드 시간 줄여보기

```
BUILD SUCCESSFUL in 1m 48s
6 actionable tasks: 4 executed, 2 up-to-date
오후 7:22:09: 실행이 완료되었습니다 ':classes'.
```

6개 중에 고작 4개 실행하는데 1분 48초나 걸렸다.
줄여보기로 했다.

```properties
# gradle 실행 엔진 최적화

# Gradle 데몬 프로세스를 백그라운드에 상주시켜 빌드할 때마다 JVM을 띄울 필요가 없다.
org.gradle.daemon=true

# 프로젝트 내의 여러 작업(Task)을 가능한 경우 병렬로 처리(멀티 코어 CPU)
org.gradle.parallel=true

# 변경되지 않은 코드는 다시 컴파일하지 않고 캐시에서 가져와 재사용
org.gradle.caching=true

# -Xmx4g: 최대 힙 메모리를 4GB로 설정
# -XX:MaxMetaspaceSize=512m: 클래스 메타데이터 저장 공간 제한
# -XX:+HeapDumpOnOutOfMemoryError: 메모리 부족 발생 시 분석을 위한 덤프 파일 생성
# -Dfile.encoding=UTF-8: 파일 인코딩을 UTF-8로 고정하여 한글 깨짐 방지
org.gradle.jvmargs=-Xmx4g -XX:MaxMetaspaceSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

# 증분 컴파일(Incremental Compilation)을 활성화하여 수정된 파일과 그에 연관된 파일들만 다시 컴파일
kotlin.incremental=true

# 코틀린 공식 코딩 스타일 가이드 적용
kotlin.code.style=official

# 프로젝트 내에서 코틀린 컴파일 태스크들을 병렬로 실행하도록 허용
kotlin.parallel.tasks.in.project=true
```

## 결과

```
BUILD SUCCESSFUL in 8s
6 actionable tasks: 1 executed, 5 up-to-date
오후 7:26:11: 실행이 완료되었습니다 ':classes'.
```
크게 시간을 줄일 수 있었다 👍