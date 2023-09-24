# Docker Hub + AWS Elastic Beanstalk + Github Actions을 이용하여 CI/CD 구축하기

## desscription

- 도커를 활용하면 가상환경을 구축하여 일정한 환경에서 개발을 진행할 수 있어서 개발 환경에 대한 의존성을 줄일 수 있습니다.
- CI/CD

## flow

- git repo의 main branch에 push를 하면 github actions이 브랜치의 변경을 감지하고 작성한 workflow 파일을 실행합니다.
- main branch 변경 감지
- docker login
- docker build & push
  - 새로운 버전의 이미지가 push 되어 올라갑니다.
- elastic beans talk가 Docker hub의 이미지를 받아와 앱을 재배포 하도록 명령합니다.

## docker hub

- docker hub에 연결하기 위해서 next.js 프로젝트를 dockerize 합니다.

```bash
  # Dockerfile

  # build stage
  FROM node:18.16.1-alpine
  #directory 지정
  WORKDIR /usr/src/app
  COPY package.json ./
  COPY yarn.lock ./
  #패키지 다운로드
  RUN yarn install
  # 필요한 모든 파일을 복사
  COPY . .
  # 빌드
  RUN yarn build
  # 컨테이너 포트 3000 설정
  EXPOSE 3000
  # 어플리케이션 실행
  CMD ["yarn", "start"]
  FROM node:14.17.6-alpine3.14
```

- docker ignore 파일을 생성하여 불필요한 파일을 제외합니다.

```bash
  # .dockerignore
  Dockerfile
  .dockerignore
  node_modules
  npm-debug.log
  README.md
  .next
  .git
```

- pakage.json에 docker build를 위한 명령어를 추가합니다.
- github repo에 General > Secret Docker와 Aws Key들을 등록합니다.

![githubsecret.png](https://taglog-image-uploader.s3.amazonaws.com/githubsecret.png)

```json
  # package.json
  "scripts": {
    ...
    "docker:build": "docker build -t [docker hub username] .",
    "docker:run": "docker run -p 3000:3000 [docker hub username]"
  }
```

```json
  # package.json
  "scripts": {
    ...
    "docker:build": "docker build -t [docker hub username] .",
    "docker:run": "docker run -p 3000:3000 [docker hub username]"
  }
```

## error

- https://web.archive.org/web/20200714173126/http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo.iam.managed-policies.html
- 권한 문제 발생
  ![error.png](https://taglog-image-uploader.s3.amazonaws.com/error.png)

accesskey
액세스 키 모범 사례 및 대안정보
보안 개선을 위해 액세스 키와 같은 장기 자격 증명을 사용하지 마세요. 다음과 같은 사용 사례와 대안을 고려하세요.

사용 사례
Command Line Interface(CLI)
AWS CLI를 사용하여 AWS 계정에 액세스할 수 있도록 이 액세스 키를 사용할 것입니다.
로 만들어보자

## https://gist.github.com/RobertoSchneiders/c9ee659cc5a565642fd9

# docker 컴테이너 용량 최적화

기존 Dockerfile

```
# build stage
FROM node:18.16.1-alpine

#directory 지정
WORKDIR /usr/src/app

COPY package.json ./
COPY yarn.lock ./

#패키지 다운로드
RUN yarn install

# 필요한 모든 파일을 복사
COPY . .

# 빌드
RUN yarn build

# 컨테이너 포트 3000 설정
EXPOSE 3000

# 어플리케이션 실행
CMD ["yarn", "start"]
```

변경 후 Dockerfile

```
  # build stage
  FROM node:18.16.1-alpine AS base

  #directory 지정
  WORKDIR /usr/src/app

  COPY package.json yarn.lock* ./

  FROM base AS deps
  #패키지 다운로드
  RUN yarn install


  FROM deps AS builder
  COPY . .
  RUN yarn build

  FROM base AS runner
  COPY --from=builder /usr/src/app/.next ./.next
  COPY --from=builder /usr/src/app/public ./public

  # 컨테이너 포트 3000 설정
  EXPOSE 3000

  # 어플리케이션 실행
  CMD ["yarn", "start"]
```

### 멀티 스테이지 빌드(Multi-stage Build)

Docker는 이미지를 빌드할 때 여러 단계를 거쳐서 최종 이미지를 생성하는 기능을 제공합니다. 각 스테이지에서는 필요한 작업을 수행하고, 최종 스테이지에서는 필요한 파일만을 최종 이미지에 포함시킵니다.
이 방법을 사용하면 불필요한 파일이나 의존성을 제거하여 최종 이미지의 크기를 줄일 수 있습니다.

### 멀티 스테이지 빌드의 적용

- 원래의 Dockerfile은 모든 작업을 단일 스테이지에서 수행하였습니다. 최적화한 버전은 여러 단계(스테이지)로 나누어 작업을 수행하는 멀티 스테이지 빌드를 사용합니다.
  각 스테이지는 특정 작업(의존성 설치, 빌드, 실행)을 수행합니다.

### 이미지 크기 최적화

- 멀티 스테이지 빌드를 사용하면 빌드 도중 생성된 임시 파일이나 불필요한 의존성을 최종 이미지에서 제외할 수 있습니다. 이로 인해 최종 이미지의 크기가 줄어듭니다.

### 정리 및 구조화

- 각 스테이지는 명확한 이름(예: deps, builder, runner)을 가지므로 Dockerfile의 각 부분이 어떤 목적으로 사용되는지 명확해집니다.

## 참고 자료

> 배포 자동화
> https://velog.io/@jadenkim5179/Docker-Hub-Elastic-Beanstalk-Github-Action%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-Node-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-CICD-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0

> ### 초보자를 위한 Next.js 애플리케이션 Dockerize 방법
>
> https://dev.to/rashidalikalwar/how-to-dockerize-a-nextjs-application-for-beginners-38k0

> ### 도커 용량 줄이기
>
> https://velog.io/@jadenkim5179/Next.js-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-docker-%EB%B0%B0%ED%8F%AC-%EC%9D%B4%EB%AF%B8%EC%A7%80-%ED%81%AC%EA%B8%B0-%EC%A4%84%EC%9D%B4%EA%B8%B0

> ### 도커 access code 발급
>
> https://teichae.tistory.com/entry/Docker-Hub%EC%97%90%EC%84%9C%EC%9D%98-Token-%EB%B0%9C%EA%B8%89-%EB%B0%A9%EB%B2%95
