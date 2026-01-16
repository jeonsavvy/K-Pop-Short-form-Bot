# 🔧 설정 가이드

이 문서는 K-Pop Short-form Trend Hunter 워크플로우를 n8n에 설정하는 상세 가이드를 제공합니다.

## 📋 사전 준비사항

### 1. n8n 인스턴스 준비
- Self-hosted n8n 또는 n8n Cloud 계정이 필요합니다.
- n8n 설치 방법: [n8n 공식 문서](https://docs.n8n.io/hosting/)

### 2. API 키 준비
- **Google Gemini API Key**: [Google AI Studio](https://makersuite.google.com/app/apikey)에서 발급
- **Slack Webhook URL** 또는 **Slack Bot Token**: Slack App 생성 필요

## 🚀 단계별 설정

### Step 1: 워크플로우 Import

1. n8n 대시보드에 로그인
2. 좌측 상단의 **"Workflows"** 메뉴 클릭
3. **"Import from File"** 버튼 클릭
4. `kpop-trend-hunter-workflow.json` 파일 선택
5. 워크플로우가 성공적으로 import되었는지 확인

### Step 2: Google Gemini API 설정

1. 워크플로우에서 **"Google Gemini Chat Model"** 노드 클릭
2. **"Credentials"** 섹션에서 **"Create New Credential"** 클릭
3. Credential 타입으로 **"Google Gemini API"** 선택
4. Google AI Studio에서 발급받은 **API Key** 입력
5. Credential 이름을 **"Google Gemini(PaLM) Api account"**로 설정 (또는 원하는 이름)
6. **"Save"** 클릭

### Step 3: Slack 설정

#### 옵션 A: Slack Webhook 사용

1. [Slack API](https://api.slack.com/apps)에서 새 App 생성
2. **"Incoming Webhooks"** 기능 활성화
3. 워크스페이스에 추가하고 Webhook URL 복사
4. n8n에서 **"Send a message"** 노드의 **"Webhook"** 옵션 선택
5. Webhook URL 입력

#### 옵션 B: Slack Bot Token 사용

1. [Slack API](https://api.slack.com/apps)에서 새 App 생성
2. **"OAuth & Permissions"**에서 다음 스코프 추가:
   - `chat:write`
   - `channels:read`
   - `channels:join`
3. **"Install to Workspace"** 클릭하여 워크스페이스에 설치
4. **"Bot User OAuth Token"** 복사
5. n8n에서 **"Send a message"** 노드 클릭
6. **"Credentials"** 섹션에서 **"Create New Credential"** 클릭
7. Credential 타입으로 **"Slack API"** 선택
8. **"Access Token"**에 Bot Token 입력
9. Credential 이름을 **"Slack account"**로 설정
10. **"Save"** 클릭

### Step 4: Slack 채널 설정

1. **"Send a message"** 노드에서 **"Channel"** 필드 클릭
2. 원하는 Slack 채널 선택 (예: `#kpop-trends`)
3. 또는 채널명을 직접 입력 (예: `#reddit-test`)

### Step 5: 스케줄 설정

기본값은 매일 오전 9시(Asia/Tokyo)입니다. 변경하려면:

1. **"Schedule Trigger"** 노드 클릭
2. **"Trigger Times"** 섹션에서 원하는 시간 설정
3. 타임존은 워크플로우 설정에서 변경 가능

### Step 6: 에러 핸들링 설정 (기본 적용됨)

워크플로우 파일(`kpop-trend-hunter-workflow.json`)에 이미 **재시도(Retry)** 및 **타임아웃** 설정이 적용되어 있습니다. 필요시 조정하세요:

1. **HTTP Request** 노드 클릭
2. **"Options"** 섹션 확장
3. **"Retry"** 설정 확인:
   - **Max Retries**: `3` (기본값)
   - **Retry On Fail**: 활성화됨
4. **"Timeout"**: `30000` (30초)

> **Note**: Reddit 서버가 불안정할 경우 자동으로 3번까지 재시도합니다.

### Step 7: 워크플로우 활성화

1. 워크플로우 상단의 **"Active"** 토글을 켜기
2. 첫 실행을 위해 **"Execute Workflow"** 버튼으로 테스트 실행

## ✅ 테스트

### 수동 실행 테스트

1. 워크플로우 상단의 **"Execute Workflow"** 버튼 클릭
2. 각 노드의 실행 결과 확인:
   - **HTTP Request**: Reddit RSS 데이터 수집 확인
   - **Code**: XML 파싱 결과 확인
   - **Basic LLM Chain**: AI 분석 결과 확인
   - **Send a message**: Slack 메시지 전송 확인

### 예상 결과

Slack 채널에 다음과 같은 형식의 리포트가 도착해야 합니다:

```
🔥 **1. [챌린지 제목]**
🔗 원본: https://reddit.com/r/kpop/...
💡 **이유:** 바이럴 가능성 높은 이유
📹 **제안:** 챌린지 촬영 아이디어 또는 기획안

---
(다음 항목...)
```

## 🔍 문제 해결

### 문제: Reddit RSS 데이터를 가져오지 못함
- **해결**: 
  - Reddit URL이 올바른지 확인하고, 네트워크 연결 상태 확인
  - HTTP Request 노드의 실행 로그 확인 (에러 메시지 확인)
  - Retry 설정이 활성화되어 있는지 확인
  - Reddit 서버 상태 확인 (일시적 장애 가능성)

### 문제: Google Gemini API 오류
- **해결**: 
  - API Key가 유효한지 확인
  - API 할당량 확인 (일일/월별 한도 초과 여부)
  - Rate Limit 에러인 경우: n8n이 자동으로 재시도하지만, 빈번한 실행 시 간격 조정 필요
  - API 응답 형식 확인 (에러 메시지 확인)

### 문제: Slack 메시지가 전송되지 않음
- **해결**: 
  - Bot Token 또는 Webhook URL이 올바른지 확인
  - Slack App이 올바른 채널에 접근 권한이 있는지 확인
  - 채널명이 정확한지 확인 (앞에 `#` 포함)
  - Slack API Rate Limit 확인 (과도한 요청 시 제한 가능)

### 문제: LLM 응답이 비어있음
- **해결**: 
  - Reddit 데이터가 제대로 파싱되었는지 확인 (Code 노드 출력 확인)
  - Code 노드의 출력이 올바른 형식인지 확인
  - Gemini API 할당량 확인
  - 프롬프트 형식 확인 (특수문자 이스케이프 문제 가능성)

### 문제: 워크플로우가 반복적으로 실패함
- **해결**:
  - n8n 워크플로우 실행 로그에서 에러 원인 확인
  - 각 노드의 실행 결과를 단계별로 확인
  - 에러가 발생한 노드 이전 노드의 출력 확인
  - 네트워크 연결 상태 확인 (Self-hosted n8n인 경우)
  - n8n 서버 리소스 확인 (메모리, CPU 사용량)

## 📝 커스터마이징

### Reddit 검색 쿼리 변경

`HTTP Request` 노드의 URL을 수정:

```
https://www.reddit.com/r/kpop/search.rss?q=YOUR_KEYWORD&restrict_sr=on&sort=new&t=week
```

### 다른 서브레딧 사용

URL의 `/r/kpop` 부분을 원하는 서브레딧으로 변경:

```
https://www.reddit.com/r/YOUR_SUBREDDIT/search.rss?q=challenge&restrict_sr=on&sort=new&t=week
```

### 리포트 형식 변경

`Basic LLM Chain` 노드의 prompt를 수정하여 원하는 리포트 형식으로 변경할 수 있습니다.

## 🔒 보안 주의사항

- n8n의 credentials 관리 기능을 사용하여 안전하게 저장하세요
- 프로덕션 환경에서는 환경 변수를 사용하는 것을 권장합니다
