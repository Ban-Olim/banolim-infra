# Banolim Infra

> 경계선 지능 특성을 가진 아동을 위한 학습 플랫폼 **Banolim**의 인프라 및 배포 관리 레포지토리입니다.

<br>

## 📚 목차

1. [프로젝트 소개](#-프로젝트-소개)
2. [기술 스택](#️-기술-스택)
3. [인프라 아키텍처 및 역할](#-인프라-아키텍처-및-역할)
4. [서비스 구성 및 디렉토리 구조](#️-서비스-구성-및-디렉토리-구조)
5. [CI/CD 및 배포 파이프라인](#️-cicd-및-배포-파이프라인)
6. [환경 변수 설정 가이드](#-환경-변수-설정-가이드)
7. [트러블 슈팅](#-트러블-슈팅)
8. [Team Members](#-team-members)

<br>

## 📌 프로젝트 소개

**Banolim**은 경계선 지능 특성을 가진 아동을 위한 학습 플랫폼입니다.  
본 레포지토리는 Banolim 서비스의 **인프라 및 컨테이너 오케스트레이션**을 담당합니다. 

Nginx 리버스 프록시 설정, Docker Compose를 통한 멀티 컨테이너 환경 관리, FastAPI 및 DB 접근을 위한 환경변수 제어, GitHub Actions 기반의 GitOps 배포 자동화의 중심점 역할을 수행합니다.

<br>

## 🛠️ 기술 스택

### Infra & DevOps

![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS_EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)

### Environment & Target Languages

![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)

<br>

## 🧩 인프라 아키텍처 및 역할

Banolim 인프라는 안정적인 서비스 서빙과 네트워크 격리, 배포 자동화를 목표로 구성되어 있습니다.

- **Nginx 리버스 프록시**: 외부 요청을 안전하게 받아 내부 컨테이너(FastAPI 등)로 라우팅하며, 외부 API 호출 시 발생할 수 있는 타임아웃(Timeout) 시간을 안정적으로 관리합니다.
- **Docker Compose 기반 컨테이너 관리**: 다중 애플리케이션 환경을 격리된 네트워크 환경에서 유기적으로 구동합니다.
- **환경 변수 자동화 및 보안**: FastAPI의 DB 접근 키 등 민감 정보를 환경 변수(.env) 기반으로 안전하게 주입하고 관리합니다.

<br>

## 🏗️ 서비스 구성 및 디렉토리 구조

Banolim 서비스는 기능별로 레포지토리를 분리하여 관리하며, 본 레포지토리는 배포 환경의 최상위 오케스트레이션을 수행합니다.

```text
banolim-infra
├─ .github/workflows     # GitHub Actions를 통한 CD 자동화 워크플로우
├─ nginx                 # Nginx 리버스 프록시 및 타임아웃 설정 파일
├─ .gitignore            # 환경 변수 및 설정 파일 보안 관리
├─ docker-compose.yml    # FastAPI, DB 및 인프라 컨테이너 통합 관리 스크립트
└─ README.md             # 인프라 가이드 문서
```

## ⚙️ CI/CD 및 배포 파이프라인
Spring Boot 백엔드 및 FastAPI 레포지토리의 배포 트리거(Repository Dispatch)를 전달받아 EC2 서버에 컨테이너를 자동으로 갱신하고 배포합니다.

```text
백엔드/FastAPI 배포 완료
        ↓
Infra 레포지토리 Dispatch 트리거
        ↓
GitHub Actions Workflows 실행
        ↓
서버 환경 변수(.env) 자동 생성 및 검증
        ↓
Docker Compose Up (컨테이너 갱신)
        ↓
Nginx 프록시 및 외부 API 타임아웃 적용 후 서비스 개시
```

## 🔑 환경 변수 설정 가이드
본 인프라 구조를 정상적으로 구동하기 위해서는 docker-compose.yml이 참조하는 .env 파일에 아래와 같은 환경 변수 키 (예시) 정의가 필요합니다. 

```text
FASTAPI_DB_URL=postgresql://username:password@host:port/database
FASTAPI_ENV=production
```

## 💫 트러블 슈팅

> 다중 배포 충돌로 인한 HTTP 502 Bad Gateway

### 1. 문제 상황 및 초기 오해
**< 문제 1 >** 
- 초기 오해: Spring Boot와 FastAPI 레포지토리에서 각각 컨테이너를 직접 배포하고, GitHub Secrets도 각 레포에 분산 주입해야 하는 줄 착각했습니다.
- 현상: 여러 레포에서 서버로 동시 배포가 수행되며 배포 타이밍 충돌 및 컨테이너 상태 꼬임이 발생했고, Nginx가 백엔드 응답을 받지 못해 HTTP 502 Bad Gateway가 지속 발생했습니다.

<br>**< 문제 2 >**
- Certbot 충돌: Docker Compose의 기존 Nginx entrypoint 설정과 Certbot의 인증서 발급 명령어가 충돌하여 SSL 인증서가 발급되지 않았습니다.
- 환경 변수 유실: GitHub Actions(appleboy/ssh-action) 작성 시, 전달할 환경 변수 목록(envs)을 잘못된 문법(줄바꿈 |)으로 작성하여 서버에 환경 변수가 전혀 주입되지 않았습니다.
<hr>

### 2. 원인 분석 및 해결
**< 문제 1 >**
- 원인: 인프라 레포(banolim-infra)가 있음에도 각 애플리케이션 레포에서 개별 배포 명령을 내려 배포 지휘권과 책임 분리가 모호했습니다.
- 해결: 구조를 전면 개편하여 Spring과 FastAPI 레포는 '이미지 빌드 및 푸시'만 담당하게 제한했습니다. 실제 서버 배포는 banolim-infra 레포로 단일화하고, 모든 환경 변수(Secrets)도 인프라 레포 한 곳으로 집중시켜 중앙 관리하도록 수정했습니다.

<br>**< 문제 2 >**
- 기존 주기를 우회하는 --entrypoint 옵션을 명시하여 SSL 인증서를 즉시 정상 발급받아 HTTPS 통신을 활성화했습니다.
- GitHub Actions의 envs 구문에서 줄바꿈 대신 쉼표(,)를 사용해 한 줄로 표기함으로써 수많은 API 키와 DB 접근 정보가 누락 없이 주입되도록 수정했습니다.
<hr>

### 3. 학습 및 회고
- 서비스 규모가 커질수록 개발(Dev)과 인프라(Ops)의 책임을 명확히 분리하는 CI/CD 아키텍처 설계의 중요성을 깨달았습니다. 배포 파이프라인 단일화를 통해 동시 배포 충돌을 근본적으로 차단했습니다.
- Docker 컨테이너의 라이프사이클을 명확히 이해하게 되었으며, CI/CD 스크립트의 미소한 문법 오류 하나가 인프라 전체 장애를 유발할 수 있음을 배워 공식 문서 기반의 철저한 문법 검증 습관을 지니게 되었습니다.


## 👥 Team Members
<table>
  <tr>
    <td align="center" width="220px">
      <a href="https://github.com/gimn70009">
        <img src="https://github.com/gimn70009.png" width="120px;" alt="gimn70009"/>
      </a>
      <br />
      <a href="https://github.com/gimn70009">
        <b>gimn70009</b>
      </a>
      <br />
      <sub>Backend Developer</sub>
      <br />
      <br />
      <span> AWS EC2 서버 구축 <br> Nginx 프록시 설정 <br> Docker Compose 환경 구축 <br> CI·CD 배포 자동화 AWS EC2 서버 구축 </span>
    </td>
    <td align="center" width="220px">
      <a href="https://github.com/youserlol">
        <img src="https://github.com/youserlol.png" width="120px;" alt="youserlol"/>
      </a>
      <br />
      <a href="https://github.com/youserlol">
        <b>youserlol</b>
      </a>
      <br />
      <sub>Backend Developer</sub>
      <br />
      <br />
      <span> AWS EC2 서버 구축 <br> Nginx 프록시 설정 <br> Docker Compose 환경 구축 <br> CI·CD 배포 자동화 <br> 환경 변수 관리</span>
    </td>
  </tr>
</table>
