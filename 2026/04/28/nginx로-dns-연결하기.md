## nginx로 DNS 연결 + HTTPS 설정하기

#### nginx를 사용하게 된 이유

EC2 인스턴스를 띄우면 기본적으로 두 가지 문제가 있다.
- 외부에서 접근할 때 IP 주소를 직접 써야 함 → 도메인 연결 필요
- 기본이 HTTP라 통신이 평문으로 오감 → HTTPS 필요

이걸 해결하는 방법이 여러 가지 있는데, nginx 리버스 프록시 + Certbot 조합이 가장 보편적이다. AWS ALB나 CloudFront로도 SSL 처리가 가능하지만, 단순한 개인 프로젝트나 단일 서버 구성에서는 nginx가 훨씬 가볍고 빠르게 붙일 수 있어서 nginx를 사용해서 리버스 프록시를 설정해보기로 했다.

#### 기술 선택 기준

| 상황 | 추천 기술 |
| --- | --- |
|개인 프로젝트 / 단일 EC2 | nginx + Certbot |
| 트래픽이 많고 가용성이 중요한 서비스 | AWS ALB + ACM |
| (자동 인증서 관리)정적 사이트 | CloudFront + S3 |
 
---

### 1. ec2 인스턴스 접속 및 nginx 설치

```shell
sudo apt update
sudo apt install nginx -y

# 시작 및 부팅 시 자동 실행
sudo systemctl start nginx
sudo systemctl enable nginx

# 상태 확인
sudo systemctl status nginx
```

### 2. Certbot으로 SSL 인증서 발급

Let's Encrypt 인증서를 무료로 발급받는다. --nginx 옵션을 주면 nginx 설정까지 자동으로 수정해준다.

```shell
# Certbot 설치
sudo apt install certbot python3-certbot-nginx -y

# 인증서 발급
sudo certbot --nginx -d your-domain.com

# 인증서 자동 갱신 테스트 (인증서 유효기간 90일, cron으로 자동 갱신됨)
sudo certbot renew --dry-run
```

### 3. Nginx 설정 확인 및 수정

Certbot이 자동으로 SSL 설정을 넣어주지만, Spring Boot / Node.js로 트래픽을 넘기려면 proxy_pass 블록을 직접 추가해야 한다.

```shell
# Certbot 적용 후 자동 생성된 설정 확인
sudo cat /etc/nginx/sites-available/default
```

Spring Boot / Node.js 백엔드로 리버스 프록시를 붙이려면 아래처럼 수정:
```shell
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    # HTTP → HTTPS 리다이렉트
    return 301 https://$host$request_uri;  
}

server {
    listen 443 ssl;
    server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    location / {
        # Spring Boot 포트
        proxy_pass http://localhost:8080;  
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
> X-Forwarded-For, X-Real-IP 헤더를 넘기는 이유: nginx가 프록시로 끼어들면 Spring Boot 입장에서 클라이언트 IP가 127.0.0.1로만 보인다. 실제 IP를 로깅하거나 IP 기반 처리를 할 때 이 헤더가 필요하다.

### 흐름 요약
```
클라이언트
└─> your-domain.com (DNS A 레코드 → EC2 Elastic IP)
    └─> nginx :80 → 301 redirect
        └─> nginx :443 (SSL 종료)
            └─> localhost:8080 (Spring Boot / Node.js)
```

### 트러블슈팅

#### 1. certbot: command not found
snap으로 설치하면 certbot 버전 문제를 해결하였다.

```shell
shellsudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

#### 2. 인증서 발급 실패 (connection timeout)

EC2 보안 그룹에서 80, 443 포트 인바운드 허용 여부 확인. 

Certbot이 HTTP-01 챌린지로 도메인 소유를 검증할 때 80포트가 열려 있어야 한다.

#### 3. nginx 설정 수정 후 반영이 안 될 때
```shell
# 설정 문법 오류 체크
sudo nginx -t          

# restart 대신 reload로 무중단 반영
sudo systemctl reload nginx
```

### 느낀 점

Certbot 자동 갱신은 systemd timer 또는 cron으로 처리되고 있어서 별도 작업은 필요 없었다.

다음엔 nginx upstream 블록으로 로드밸런싱 구성도 해볼 것이다.