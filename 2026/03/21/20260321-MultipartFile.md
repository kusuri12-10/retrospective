# MultipartFile

MultipartFile은 스프링에서 HTTP 요청으로 들어온 파일 데이터를 다루기 위한 [InputStream](20260321-InputStream.md) 구현체이다. 

파일 이름, 크기, 컨텐츠 타입(MIME) 같은 정보를 담고 있으며, 파일을 저장하거나 읽을 수 있는 편리한 메서드들을 제공한다.

`getName()`: HTTP 요청 파라미터의 이름(name) 확인

`getOriginalFilename()`: 실제 파일의 이름 확인

`getSize()`: 파일 크기 확인

`getInputStream()`: 파일의 실제 데이터에 접근

## Multipart란?

- HTTP 프로토콜의 바디 부분에 데이터를 여러 부분으로 나눠서 보내는 방식

- 보통 파일을 전송할 때 사용

- Multipart data: 여러개의 파일 정보를 body 내에 여러개의 부분으로 나누어서 전송되는 것

## Multipart/form-data란?

1. `application/x-www-form-urlencoded` : 가장 기본적인 폼 데이터 형식

    - 데이터 형태: `key=value` 쌍, `&`로 구분
       
        - `name=kusuri&age=18` 같은 형태
    
        - 한글, 특수문자 등은 URL 인코딩된다. (%XX 형태)

    - 텍스트 위주의 짧은 데이터를 보낼 때 효율적

    - 바이너리 데이터(이미지, 파일 등)를 보내기에는 부적합
        - 이진 데이터를 텍스트로 변환(Base64 등)하면 크기가 약 33% 커지기 때문

2. `Multipart/form-data` : 파일 업로드를 위해 고안된 복합 데이터 전송 방식

    - 데이터 형태: 하나의 요청 바디 안에 여러 종류의 데이터를 구분자(Boundary)로 나누어 담는다. 
    
        - 텍스트는 텍스트대로, 파일은 바이너리 그대로 포함된다.

        - 각 파트(Part)마다 Content-Type과 Content-Disposition 헤더를 가질 수 있다.

    - 장점
        - 텍스트와 대용량 바이너리 파일을 원본 그대로 섞어서 보낼 수 있다. 
        - 인코딩 오버헤드가 없어 파일 전송에 최적화되어 있다.

    - 단점
        - 텍스트만 보낼 때는 구분자(Boundary) 문자열이 추가되므로 urlencoded 방식보다 데이터 양이 아주 약간 더 많아진다.

3. MultipartResolver

    - 사용자의 파일업로드 요청에 대한 처리를 하는 인터페이스

    - SpringBoot에서는 기본 구현체가 자동으로 빈으로 등록됨

    - MultipartFile 객체 사용 시 필요

### 요약

| 구분 | x-www-form-urlencoded | multipart/form-data |
| --- | --- | --- |
| 기본 용도 | 단순 텍스트 데이터 전송 | 파일 및 이진 데이터 전송 |
| 데이터 구조 | key1=value1&key2=value2 | Boundary로 나뉜 개별 파트 구조 |
| 파일 전송 | 비효율적 | 매우 적합 |
| Spring 처리 | @RequestParam, @ModelAttribute | @RequestPart, MultipartFile |

## MultipartFile vs java.io.File

1. 데이터 형태

    - java.io.File: 하드디스크 내 경로에 파일이 실재해야 객체 생성 가능 (물리적 경로)

    - MultipartFile: 네트워크 패킷(바이트 흐름) 형태 (가상 메모리)

2. 메타데이터 포함

    - 파일 메타데이터: Content-Type, Form Parameter Name, Original Filename 등

    - File 객체는 이런 정보를 담을 공간이 없지만, MultipartFile은 이를 모두 제공한다.

3. 서버 자원 보호
    - File 객체는 모든 파일을 하드디스크에 저장한 뒤에 로직을 시작할 수 있다.

    - MultipartFile은 설정에 따라 작은 파일은 메모리에서 바로 처리하고, 큰 파일만 임시 디렉토리에 저장한다.
    
    - 파일을 S3 같은 외부 스토리지로 전달하고 싶을 때(Streaming), File은 쓰기 과정이 필요하지만 MultipartFile은 inputStream을 통해 바로 전달이 가능하다.

### 요약

| 구분 | java.io.File | MultipartFile |
| --- | --- | --- |
| 대상 | 로컬 시스템에 존재하는 파일 | 네트워크를 통해 넘어오는 파일 |
| 생성 조건 | 물리적인 파일 경로가 반드시 필요함 | HTTP 요청(Multipart)이 있으면 자동 생성 |
| 효율성 | 항상 디스크 I/O(읽기/쓰기) 발생 | 메모리와 임시 저장소를 유연하게 활용 |
| 데이터 포함 | 파일 데이터만 가짐 | 파일 데이터 + 메타데이터(ContentType 등) |
| 주요 용도 | 서버 로컬 파일의 읽기/쓰기 작업 | 클라이언트의 업로드 파일 처리 |
| 의존성 | 자바 표준 라이브러리 | 스프링 프레임워크 전용 인터페이스 |

## 결론

MultipartFile은 스프링에서 파일 데이터를 다루기 위해 사용하는 InputStream 구현체이며, 대용량 이미지 파일 등을 다룰 때 효율적이다.

파일 데이터를 S3에 업로드 할 경우에 자주 사용한다.