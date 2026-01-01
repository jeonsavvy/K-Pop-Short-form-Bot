## 📝 프로젝트 소개 (Executive Summary)

> **"매일 아침 K-Pop 숫폼 챌린지 소재를 발굴하여 Slack으로 기획안을 자동 발송합니다."**

**K-Pop Short-form Trend Hunter**은 **엔터사 직원 및 콘텐츠 기획자**를 위한 **자동화 트렌드 모니터링 시스템**입니다. **n8n 워크플로우와 Google Gemini API**를 활용하여 **숏폼 콘텐츠 소재 발굴을 위한 수동 작업**을 해결하고, 결과적으로 **매일 자동으로 바이럴 가능성 높은 K-Pop 챌린지를 선별하여 Slack으로 리포트를 전송**하는 업무 효율성을 제공합니다.

* **제작:** jeonsavvy@gmail.com

---

## ✨ 핵심 기능 (Key Features)

<table>
  <tr>
    <td align="center" width="50%">
      <h3>🔹 RSS 데이터 정제 로직</h3>
      <p>Reddit RSS의 XML 형태 데이터를 JavaScript 정규표현식 기반 파서로 처리하여 제목과 링크를 추출하고, CDATA 섹션을 제거하여 구조화된 JSON 객체로 변환하여 후속 LLM 노드에서 활용 가능하도록 처리합니다.</p>
    </td>
    <td align="center" width="50%">
      <h3>🔹 프롬프트 엔지니어링 최적화</h3>
      <p>AI의 평가 기준을 구체적으로 정의(독창성, 따라하기 쉬움, 바이럴 가능성)하고, Slack에 최적화된 마크다운 형식(이모지 포함)으로 출력 포맷을 강제하여 별도의 후처리 없이 즉시 읽을 수 있는 리포트를 생성합니다.</p>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <h3>🔹 자동화된 트렌드 모니터링</h3>
      <p>매일 지정된 시간(기본: 오전 9시)에 Reddit r/kpop 서브레딧에서 challenge 키워드 검색을 통해 최신 트렌드를 자동으로 수집하고, 바이럴 가능성 높은 소재를 AI가 선별합니다.</p>
    </td>
    <td align="center" width="50%">
      <h3>🔹 Slack 자동 리포트 전송</h3>
      <p>가독성 높은 리포트 형식(이모지, 마크다운)으로 Slack 채널에 자동 전송하며, 본론만 출력하여 효율성을 향상시킵니다. 에러 핸들링 및 재시도 로직을 통해 안정성을 확보합니다.</p>
    </td>
  </tr>
</table>

---

## 🏗 아키텍처 및 워크플로우 (Architecture)

### 🔄 데이터 흐름

1. **수집 (Input):** Schedule Trigger로 매일 지정된 시간(기본: 오전 9시)에 워크플로우가 자동 실행되며, HTTP Request 노드를 통해 Reddit RSS 피드(r/kpop 서브레딧, challenge 키워드 검색)에서 XML 형식 데이터를 수집합니다 (Retry: 3회, Timeout: 30초).

2. **처리 (Process):** JavaScript 노드에서 XML 데이터를 정규표현식으로 파싱하여 제목과 링크를 추출하고, CDATA 섹션을 제거하여 구조화된 JSON 객체로 변환합니다. 이후 Google Gemini API를 통해 Reddit 포스트 데이터를 분석하여 바이럴 가능성 높은 챌린지를 선별하고, 이모지와 마크다운 형식의 리포트를 생성합니다.

3. **결과 (Output):** 생성된 리포트를 지정된 Slack 채널로 자동 전송하여, 사용자는 매일 아침 업무에 바로 활용할 수 있는 트렌드 리포트를 받습니다.

<div align="center">
  <img src="./assets/workflow-diagram.png" alt="n8n Workflow Diagram" width="100%"/>
  <br><em>n8n 워크플로우 구조</em>
</div>

---

## 🛠 기술 스택 (Tech Stack)

| 구분 | 기술 (Technology) | 선정 이유 (Reason) |
| :--- | :--- | :--- |
| **Workflow Automation** | n8n (Self-hosted) | 코드 작성 없이 워크플로우를 시각적으로 구성할 수 있으며, 다양한 외부 서비스(Reddit, Slack, Gemini)와의 연동이 용이합니다. |
| **AI / ML** | Google Gemini API | 무료 할당량이 제공되며, 한국어 처리 성능이 우수하고 n8n과의 통합이 간단합니다. |
| **Data Source** | Reddit RSS (r/kpop) | 실질적인 팬덤 반응이 즉각적으로 나타나는 커뮤니티로, 유효 데이터 확보율이 높습니다. RSS 피드를 통해 구조화된 데이터 수집이 가능합니다. |
| **Notification** | Slack API | 업무 환경에서 널리 사용되는 협업 도구로, Incoming Webhook 또는 Bot Token을 통한 간편한 메시지 전송이 가능합니다. |
| **Data Processing** | JavaScript | n8n의 Code 노드에서 실행되며, XML 파싱 및 데이터 정제 작업을 수행합니다. 정규표현식을 활용하여 구조화된 데이터로 변환합니다. |

---

## 🚀 시작 가이드 (Getting Started)

### 전제 조건 (Prerequisites)

* **n8n 인스턴스** (Self-hosted 또는 n8n Cloud 계정)
* **Google Gemini API Key** ([Google AI Studio](https://makersuite.google.com/app/apikey)에서 발급)
* **Slack Webhook URL** 또는 **Slack Bot Token** (Slack App 생성 필요)

### 설치 및 실행 (Installation)

1. **레포지토리 클론**
   ```bash
   git clone https://github.com/ieonsavvy/K-Pop-Short-form-Trend-Hunter.git
   cd K-Pop-Short-form-Trend-Hunter
   ```

2. **n8n 워크플로우 Import**
   * n8n 대시보드에 로그인
   * 좌측 상단의 **"Workflows"** 메뉴 클릭
   * **"Import from File"** 버튼 클릭
   * `kpop-trend-hunter-workflow.json` 파일 선택

3. **환경 변수 설정 (API Key 설정)**
   
   **Google Gemini API Key 설정:**
   * 워크플로우에서 **"Google Gemini Chat Model"** 노드 클릭
   * **"Credentials"** 섹션에서 **"Create New Credential"** 클릭
   * Credential 타입으로 **"Google Gemini API"** 선택
   * Google AI Studio에서 발급받은 **API Key** 입력
   * **"Save"** 클릭

   **Slack 설정 (옵션 A: Webhook 사용):**
   * [Slack API](https://api.slack.com/apps)에서 새 App 생성
   * **"Incoming Webhooks"** 기능 활성화
   * 워크스페이스에 추가하고 Webhook URL 복사
   * n8n에서 **"Send a message"** 노드의 **"Webhook"** 옵션 선택 및 URL 입력

   **Slack 설정 (옵션 B: Bot Token 사용):**
   * [Slack API](https://api.slack.com/apps)에서 새 App 생성
   * **"OAuth & Permissions"**에서 스코프 추가 (`chat:write`, `channels:read`, `channels:join`)
   * **"Install to Workspace"** 후 **"Bot User OAuth Token"** 복사
   * n8n에서 **"Send a message"** 노드의 **"Credentials"**에서 **"Slack API"** 선택 및 Token 입력

   자세한 설정은 [SETUP.md](./SETUP.md) 참고

4. **프로젝트 실행**
   * n8n 대시보드에서 워크플로우의 **"Active"** 토글을 켜기
   * 첫 실행을 위해 **"Execute Workflow"** 버튼으로 테스트 실행
   * 매일 지정된 시간(기본: 오전 9시)에 자동 실행됩니다

---

## 📂 폴더 구조 (Directory Structure)

```bash
├── assets/                           # 이미지 및 정적 파일
│   ├── workflow-diagram.png          # n8n 워크플로우 구조 다이어그램
│   └── slack-demo.png                # Slack 리포트 예시 이미지
├── kpop-trend-hunter-workflow.json   # n8n 워크플로우 설정 파일
├── README.md                         # 프로젝트 문서
└── SETUP.md                          # 상세 설정 가이드
```

---

## ⚡ 트러블 슈팅 (Troubleshooting)

| 문제 (Issue) | 원인 (Cause) | 해결 (Solution) |
| --- | --- | --- |
| **초기엔 'Google News'를 연동했으나, 단순 보도자료 등 부적합한 데이터가 대량 수집되어 유효 데이터 확보율이 낮았음** | 보도자료 등 노이즈 데이터가 대량 수집되어 실제 팬덤 반응 기반의 데이터 확보가 어려웠음 | 데이터 소스를 실질적인 팬덤 반응이 즉각적으로 나타나는 Reddit (r/kpop) 서브레딧으로 전환하고, 검색 쿼리에 challenge 키워드를 추가하여 타겟팅 정확도를 향상시킴. RSS 피드 URL 최적화를 통해 유효 데이터 확보율을 대폭 상승시킴 |
| **LLM이 JSON 코드 블록을 반환하거나 인사말을 붙여 출력 형식이 일관되지 않음** | 프롬프트에 역할 정의와 출력 형식 제약이 불명확하여 AI의 출력이 주관적이고 포맷이 제각각이었음 | 시스템 프롬프트에 Role과 Output Style을 명확히 정의하고, "인사말 없이 본론만 출력", "JSON 사용 금지, 마크다운으로만 작성" 등의 제약 조건을 명시함. 프롬프트 버전을 단계적으로 개선 (V1 → V2 → V3)하여 별도의 후처리 없이 Slack에서 즉시 읽을 수 있는 깔끔한 리포트 생성 |
| **외부 API(Reddit RSS, Gemini)의 간헐적 응답 실패로 워크플로우 중단 발생 가능성** | 네트워크 장애나 일시적인 API 응답 지연 시 워크플로우가 중단될 수 있음 | HTTP Request 노드에 Timeout 설정 추가 (30초) 및 n8n UI에서 Retry 옵션을 활성화하여 네트워크 장애 시 자동 재시도 (최대 3회). Gemini API는 n8n 내장 재시도 로직을 활용하여 일시적 장애 시 워크플로우가 자동 복구되도록 개선 |

---

## 📸 데모 (Demo)

<div align="center">
  <img src="./assets/slack-demo.png" alt="Slack 리포트 예시" width="100%"/>
  <br><em>Slack으로 전송되는 트렌드 리포트 예시</em>
</div>

---

## 🎯 프롬프트 엔지니어링 최적화

프롬프트는 단계적으로 개선하여 AI 출력의 신뢰성과 일관성을 확보했습니다:

### Version 1: 초기 프롬프트 (단순 요청)
```
위 데이터를 분석해서 틱톡에서 터질만한 챌린지를 엄선해줘.
```
**문제점:** 평가 기준이 불명확하여 AI의 주관적 판단에 의존

### Version 2: 평가 기준 명시
```
다음 기준으로 평가해줘:
- 독창성 (Originality)
- 따라하기 쉬움 (Ease of Replication)
- 바이럴 가능성 (Viral Potential)
```
**개선점:** 평가 기준이 명확해졌으나 출력 형식이 일관되지 않음

### Version 3: 출력 포맷 강제화 (현재)
```
# Output Style
🔥 **1. [노래/챌린지 제목]**
🔗 원본: (링크 주소만)
💡 **이유:** (왜 뜰 것 같은지 핵심만)
📹 **가이드:** (어떻게 찍어야 하는지 한 줄 요약)
```
**최종 개선점:** 구조화된 출력으로 파싱 및 활용 용이

### 최적화 원칙
- **Role Definition**: AI의 역할 명확히 정의
- **Context 제공**: 입력 데이터 형식과 의미 설명
- **Constraint 설정**: 출력 형식과 제약 사항 명시

---

## 🔒 보안 주의사항

⚠️ **중요:** `kpop-trend-hunter-workflow.json` 파일에는 민감한 정보(credentials ID, webhook ID 등)가 포함되어 있을 수 있습니다. 
- 프로덕션 환경에서는 credentials를 n8n의 credentials 관리 기능을 통해 안전하게 관리하세요.
- 환경 변수를 사용하는 것을 권장합니다.

---

## 📚 문서 (Documentation)

- [SETUP.md](./SETUP.md) - 상세 설정 가이드 및 문제 해결

---
