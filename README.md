<div align="center">

# 🍚 밥BTI

### 오늘 점심, 누구랑 뭐 먹지?

음식 취향으로 **밥메이트를 찾고**, 함께 먹을 **식당을 추천받는** 웹 서비스

[서비스 체험하기](http://kmu-agent-46-babbti-s3.s3-website-us-east-1.amazonaws.com)

</div>

![밥BTI 시작 화면](images/babbti-landing.png)

> 국민대학교 AWS 부트캠프 AI Workflow 팀 1 프로젝트

## 목차

- [1. 프로젝트 소개](#1-프로젝트-소개)
- [2. 주요 기능](#2-주요-기능)
- [3. 이용 흐름](#3-이용-흐름)
- [4. 구현 구조와 추천 방식](#4-구현-구조와-추천-방식)
- [5. 기술 스택](#5-기술-스택)
- [6. 실행 방법](#6-실행-방법)
- [7. 팀 역할](#7-팀-역할)

---

## 1. 프로젝트 소개

밥 약속을 잡을 때 함께 먹을 사람과 메뉴를 정하기 어려운 순간을 위한 서비스입니다. 선호 음식, 매운맛·짠맛·단맛, 향신료, 예산, 식사량을 입력하면 취향을 분석해 **밥BTI 유형**, **잘 맞는 밥메이트**, **그룹에 어울리는 식당**을 보여줍니다.

설치나 빌드 과정이 없는 단일 페이지로 만들었습니다. 기본 결과는 샘플 참가자 15명과 가상 식당 12곳을 사용해 바로 확인할 수 있고, Firebase Realtime Database에 연결되면 같은 룸 코드로 입장한 실제 참가자와 실시간 그룹 매칭도 할 수 있습니다.

## 2. 주요 기능

| 기능 | 내용 |
|---|---|
| 취향 입력 · 밥BTI | 음식 카테고리와 6가지 세부 취향을 입력받아 9가지 유형 중 하나를 보여줍니다. |
| 밥메이트 추천 | 7가지 취향 요소의 유사도를 계산해 잘 맞는 사람 3명과 매칭 이유를 보여줍니다. |
| 식당 추천 | 그룹의 평균 취향과 음식 카테고리 선호도를 바탕으로 큐레이션 식당 3곳을 추천합니다. |
| 내 주변 식당 | 위치 권한을 허용하면 반경 800m의 OpenStreetMap 식당 데이터를 검색하고 취향·거리로 순위를 매깁니다. 검색에 실패하면 기존 큐레이션 결과를 유지합니다. |
| 라이브 룸 | 같은 룸 코드의 참가자를 Firebase로 동기화하고 취향이 가까운 사람끼리 최대 4명씩 묶습니다. |
| 취향 통계 · 좋아요 | 참여자의 맛 선호와 인기 음식 분포를 시각화하고, 룸의 좋아요를 집계합니다. |
| AI 매칭 코멘트 | Anthropic API 키를 별도로 설정한 경우 Claude가 매칭 이유를 짧은 문장으로 설명합니다. 기본 설정에서는 비활성화됩니다. |

## 3. 이용 흐름

1. 닉네임과 비밀번호로 시작하고, 참여할 **룸 코드**를 입력합니다.
2. 좋아하는 음식과 맛·예산·식사량을 선택합니다.
3. 밥BTI 유형, 밥메이트 TOP 3, 추천 식당을 확인합니다.
4. 원하면 **내 주변 실제 식당 찾기**를 누르거나 같은 룸 코드로 참여한 사람들의 실시간 그룹을 확인합니다.

> 시작 화면의 닉네임·비밀번호는 데모용 저장 방식입니다. 비밀번호가 브라우저 저장소와 Firebase 데이터베이스에 평문으로 저장되므로 다른 서비스에서 쓰는 비밀번호를 입력하지 마세요.

## 4. 구현 구조와 추천 방식

```text
index.html                 화면 · 스타일 · 추천 로직 · 외부 서비스 연동
images/
└── babbti-landing.png     README 시작 화면
```

`index.html` 한 파일에 HTML, CSS, JavaScript가 들어 있습니다. 별도의 Node.js 서버나 패키지 설치가 필요하지 않습니다. Firebase SDK는 CDN에서 불러오며, 주변 식당 검색은 브라우저에서 Overpass API를 호출합니다.

| 단계 | 코드에서 사용하는 방식 |
|---|---|
| 밥BTI 유형 | 입력값에 우선순위 규칙을 적용해 9가지 유형 중 하나를 판정 |
| 밥메이트 | 음식 카테고리의 자카드 유사도와 매운맛·짠맛·단맛·향신료·예산·식사량의 근접도를 가중합 |
| 그룹 식당 | 그룹의 음식 선호 투표와 평균 맛·예산을 식당 속성과 비교 |
| 실시간 그룹 | 같은 룸 참가자 중 취향이 비슷한 사람부터 묶는 그리디 방식 |
| 주변 식당 | OpenStreetMap의 식당 데이터를 취향 적합도 45%·거리 45%·기본 점수 10%로 정렬 |

밥메이트 유사도 가중치는 **음식 카테고리 32% · 매운맛 18% · 짠맛 12% · 단맛 10% · 향신료 8% · 예산 12% · 식사량 8%**입니다. 첫 방문에도 추천이 가능하도록, 과거 이용 기록 대신 현재 입력한 취향을 사용합니다.

## 5. 기술 스택

| 구분 | 기술 |
|---|---|
| **Frontend** | ![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white) ![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white) ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) |
| **실시간 데이터** | ![Firebase](https://img.shields.io/badge/Firebase_Realtime_Database-FFCA28?style=flat-square&logo=firebase&logoColor=black) |
| **위치 · 식당 검색** | ![Geolocation API](https://img.shields.io/badge/Geolocation_API-4285F4?style=flat-square&logoColor=white) ![OpenStreetMap](https://img.shields.io/badge/OpenStreetMap-7EBC6F?style=flat-square&logo=openstreetmap&logoColor=white) ![Overpass API](https://img.shields.io/badge/Overpass_API-4C9F38?style=flat-square&logoColor=white) |
| **AI 코멘트 (선택)** | ![Anthropic Claude](https://img.shields.io/badge/Anthropic_Claude-191919?style=flat-square&logo=anthropic&logoColor=white) |
| **배포** | ![Amazon S3](https://img.shields.io/badge/Amazon_S3-569A31?style=flat-square&logo=amazons3&logoColor=white) |

## 6. 실행 방법

저장소를 내려받아 루트 디렉터리에서 정적 서버를 실행합니다.

```bash
git clone https://github.com/juneddoha/2026-kookmin-ai-workflow-team1.git
cd 2026-kookmin-ai-workflow-team1
python -m http.server 3000
```

브라우저에서 `http://localhost:3000`에 접속합니다. `index.html`을 직접 열어 화면을 볼 수도 있지만, 실시간 연결과 위치 기능을 확인할 때는 로컬 서버를 사용하세요.

- Firebase 설정은 현재 `index.html`의 `firebaseConfig`에 들어 있습니다. 실시간 룸 기능은 해당 데이터베이스의 연결 상태와 규칙에 따라 동작합니다.
- Claude 코멘트는 기본적으로 꺼져 있습니다. API 키를 공개 HTML 파일에 넣으면 방문자에게 노출되므로, 실제 운영에서는 서버를 통해 API를 호출해야 합니다.
- 위치 기능은 브라우저의 위치 권한과 **HTTPS 또는 `localhost`**가 필요합니다. 위 S3 정적 웹사이트 주소는 HTTP이므로 그 주소에서는 주변 식당 찾기가 동작하지 않을 수 있습니다.

## 7. 팀 역할

| 담당 | 역할 |
|---|---|
| [junseok0929](https://github.com/junseok0929) | 프론트엔드 UI · 취향 매칭 알고리즘 |
| [ChoHyeonChan](https://github.com/ChoHyeonChan) | Firebase 연동 · 실시간 동기화 |
| [juneddoha](https://github.com/juneddoha) | GPS 기반 주변 식당 추천 |

