# 실습 6 – Docker 전체 워크플로우

---

## 실습 목표

Spring Boot + Thymeleaf + Gradle 프로젝트를 대상으로,
“로컬 애플리케이션 → Docker 이미지 → 컨테이너 실행 → 웹 확인 -> 상태/로그 확인 -> 컨테이너 접속 -> 정리”까지의
전체 흐름을 직접 경험합니다.

이 실습을 통해 다음을 스스로 수행할 수 있어야 합니다.

* Gradle 기반 Spring Boot 애플리케이션을 JAR로 빌드한다
* 빌드 산출물을 기준으로 Dockerfile을 직접 작성한다
* Docker 이미지 생성 및 컨테이너 실행 과정을 이해한다
* 컨테이너로 실행된 웹 애플리케이션을 브라우저에서 확인한다

---

## 사전 준비

아래 환경이 모두 준비되어 있어야 실습을 진행할 수 있습니다.

* Docker Desktop 설치 및 실행
    * 터미널에서 `docker version` 명령이 정상 동작해야 함
* JDK 17 (프로젝트에서 사용하는 Java 버전 기준)
* Gradle Wrapper 포함 (`./gradlew` 사용)
* Git 클라이언트

---

## 프로젝트 클론 및 로컬 빌드

### 1. 작업 디렉터리 이동

```bash
cd ~/workspace   # 예시
```

---

### 2. Git 저장소 클론

```bash
git clone <이 저장소의 Git URL>
cd <클론된-프로젝트-폴더>
```

---

### 3. Gradle로 JAR 빌드

```bash
./gradlew clean bootJar
```

---

### 4. 빌드 결과 확인

```bash
ls build/libs
```

* `build/libs` 디렉터리 아래에 `.jar` 파일이 생성되어 있어야 합니다.

### 중요
- Docker 실습 전에 반드시 로컬에서 Gradle 빌드가 성공하는지 먼저 확인하세요.
- 로컬 빌드가 실패하면 Docker 단계에서도 정상 동작하지 않습니다.

---

## Dockerfile 직접 작성

프로젝트 루트 디렉터리
(= `build.gradle` 파일이 위치한 곳)에
`Dockerfile` 파일을 직접 작성합니다.

### Dockerfile 작성 요구사항 (Gradle 기준)

아래 조건을 모두 만족하도록 구성하세요.

* 베이스 이미지
    * JDK 17 계열의 경량 이미지 사용
    * 예: `eclipse-temurin:17-jdk-alpine`
* 작업 디렉터리
    * `/app`
* 빌드된 JAR 파일 복사
    * 호스트 경로: `build/libs/app.jar`
    * 컨테이너 경로: `/app/app.jar`
* 애플리케이션 포트 문서화
    * Spring Boot 기본 포트: `8080`
* 컨테이너 시작 시 실행 명령
    * `java -jar app.jar`

### 힌트
> Gradle 프로젝트이므로 `COPY build/libs/app.jar app.jar` 형태로 경로를 맞추면 됩니다.
### 도전
> 멀티스테이지 빌드 방식으로 Dockerfile을 개선해 보세요.

---

## Docker 이미지 빌드

`Dockerfile` 작성이 완료되면,
프로젝트 루트 디렉터리에서 아래 명령을 실행합니다.

```bash
docker build -t myapp:1.0 .
```

### 이미지 생성 확인

```bash
docker images myapp
```

* `myapp:1.0` 이미지가 목록에 보여야 합니다.

---

## 컨테이너 실행 및 웹 페이지 확인

### 1. 컨테이너 실행

```bash
docker run -d \
  --name myapp \
  -p 8080:8080 \
  myapp:1.0
```

---

### 2. 컨테이너 상태 확인

```bash
docker ps
```

* `myapp` 컨테이너가 `Up` 상태여야 합니다.

---

### 3. 브라우저 접속 확인

* 주소: [http://localhost:8080](http://localhost:8080)
* Spring Boot + Thymeleaf 화면이 정상적으로 표시되는지 확인합니다.

---

### 4. 컨테이너 로그 확인

```bash
docker logs -f myapp
```

---

## 제출용 스크린샷 가이드

아래 4가지 항목을 모두 캡처하여 제출하세요.

1. Dockerfile 전체 내용
    * 에디터 또는 IDE 화면
2. Docker 이미지 빌드 결과
    * `docker build -t myapp:1.0 .` 성공 로그
      또는
    * `docker images myapp` 실행 결과
      또는
    * `docker ps` 실행 결과
      또는
    * `docker logs myapp` 실행 결과
3. 웹 애플리케이션 실행 화면
    * 브라우저 전체 화면 캡처
    * 주소창에 `http://localhost:8080`이 보이도록

### 📁 제출 예시 파일명
* `Dockerfile.png`
* `build_ps_log.png`
* `app_browser.png`

---

## 실습 후 정리 (컨테이너 / 이미지 삭제)

실습이 끝난 후, 필요하다면 아래 명령으로 정리합니다.

### 컨테이너 중지 및 삭제

```bash
docker stop myapp
docker rm myapp
```

### 이미지 삭제 (선택)

```bash
docker rmi myapp:1.0
```

---

## 실습 체크리스트

* [ ] Git 저장소를 클론하고 Gradle `bootJar` 빌드에 성공했다
* [ ] Gradle 프로젝트 기준으로 `Dockerfile`을 직접 작성했다
* [ ] `docker build`로 Docker 이미지를 생성했다
* [ ] `docker run -p 8080:8080`으로 컨테이너를 실행했다
* [ ] 브라우저에서 `http://localhost:8080` 접속을 확인했다
* [ ] 요구된 스크린샷 3개를 모두 준비했다

---