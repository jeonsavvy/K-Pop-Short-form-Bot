# K-Pop Short-form Bot

케이팝숏폼봇은 Reddit의 K-Pop 챌린지 글을 수집해서, Gemini로 요약/아이디어를 만들고 Slack으로 보내는 n8n 워크플로우입니다.

---

## 한눈에 보기

- **트리거**: 매일 09:00 스케줄 실행
- **수집**: Reddit RSS 다중 소스 (`r/kpop`, `r/kpopthoughts`, `r/kpophelp`)
- **가공**: JavaScript 코드 노드에서 XML 파싱/정제
- **분석**: Google Gemini Chat Model + Basic LLM Chain
- **전송**: Slack 채널 메시지

---

## 워크플로우 구조

```mermaid
graph TD
    A["Schedule Trigger"] --> B["Build Source Items"]
    B --> C["HTTP Request: Reddit RSS"]
    C --> D["Code in JavaScript: Parse + Dedupe"]
    D --> E{"Has Valid Posts?"}
    E -->|"Yes"| F["Basic LLM Chain"]
    G["Google Gemini Chat Model"] --> F
    F --> H["Send a message: Slack"]
    E -->|"No"| I["Skip Send"]
```

---

## 현재 구현 기준 기능

- Reddit RSS 요청을 **다중 소스 아이템**으로 분리해 순회
- User-Agent를 후보 리스트에서 **로테이션**해서 사용
- HTTP Request 노드에 **타임아웃(30s) + 재시도(최대 3회)** 설정
- `<entry>`/`<title>`/`<link>`를 정규식으로 파싱 후 **링크 기준 중복 제거**
- 유효 포스트가 없으면 IF 분기에서 Slack 전송을 **스킵**
- Gemini 출력은 JSON이 아니라 **마크다운 리포트 텍스트**
- Slack 노드는 기본 채널 `#reddit-test`로 설정

---

## 빠른 시작

### 1) 워크플로우 불러오기

1. n8n에서 **Import from File**
2. `kpop-short-form-bot-workflow.json` 선택

### 2) Credential 연결

- `Google Gemini Chat Model` 노드: Gemini API Credential 연결
- `Send a message` 노드: Slack API Credential 연결

### 3) 실행

- 먼저 **Execute Workflow**로 테스트
- 정상 확인 후 **Active** ON

---

## 설정 포인트

- 수집 소스/키워드 변경: `Build Source Items` 노드의 `targets` 배열 수정
- User-Agent 후보 변경: `Build Source Items` 노드의 `userAgents` 배열 수정
- 전송 채널 변경: `Send a message` 노드 `channelId` 변경
- 실행 시간/주기 변경: `Schedule Trigger` 노드 수정

---

## 주의사항 (현재 상태)

- 워크플로우 JSON의 기본 상태는 `"active": false` 입니다.
- Reddit RSS XML 구조가 바뀌면 파싱 정규식 수정이 필요할 수 있습니다.
- `Has Valid Posts?` 분기에서 `No` 경로는 알림을 보내지 않도록 설계되어 있습니다.

---

## 파일 구조

```bash
├── kpop-short-form-bot-workflow.json
├── SETUP.md
├── assets/
└── README.md
```
