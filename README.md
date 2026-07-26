# 면접의 정석 — AI 면접 시뮬레이터

기업 등급(소기업/중소기업/중견기업/대기업/글로벌기업)별로 다른 말투와 디자인의
AI 면접관이 10개 질문을 던지고, 답변 조합에 따라 합격/불합격을 판정하는 웹 게임입니다.

## 구조

```
interview-game/
├── index.html         # 프론트엔드 (정적 파일, GitHub Pages든 Vercel이든 어디에 올려도 됨)
├── api/
│   └── interview.js   # Vercel 서버리스 함수 — Anthropic API 키를 서버에서만 사용
└── package.json
```

`index.html`은 절대 API 키를 직접 들고 있지 않습니다. 대신 같은 도메인의
`/api/interview`로 요청을 보내고, 그 서버리스 함수가 환경변수로 저장된
API 키를 이용해 Anthropic에 대신 요청합니다. → **키가 브라우저에 노출되지 않습니다.**

이 구조는 market-rotation-desk에서 쓰신 Vercel(stock-opinion-api) 패턴과 동일합니다.

## 배포 순서 (GitHub + Vercel)

### 1. GitHub 레포 생성 & 푸시

```bash
cd interview-game
git init
git add .
git commit -m "init: 면접 게임"
gh repo create interview-game --public --source=. --push
```
(`gh` CLI가 없으면 GitHub 웹에서 새 레포 만든 뒤 `git remote add origin ...` 으로 연결)

### 2. Vercel에 Import

1. https://vercel.com → **Add New → Project**
2. 방금 만든 GitHub 레포 선택 → Import
3. Framework Preset: **Other** (정적 파일 + api 폴더 자동 인식됨)
4. **Environment Variables**에 아래 추가:
   - `ANTHROPIC_API_KEY` = 본인의 Anthropic API 키 (console.anthropic.com → API Keys에서 발급)
5. Deploy 클릭

배포가 끝나면 `https://your-project.vercel.app` 주소가 생기고, 그 주소를
아무나 접속해서 게임을 플레이할 수 있습니다. **GitHub Pages는 필요 없습니다** —
정적 파일과 서버리스 함수를 둘 다 Vercel이 처리해주기 때문입니다.

> GitHub는 코드 저장 + Vercel과의 연동(푸시하면 자동 재배포) 용도로만 쓰입니다.
> 실제 게임이 도는 곳은 Vercel입니다.

### 3. 확인

배포된 주소에 접속해 기업을 하나 선택하고 질문이 정상적으로 뜨는지 확인하세요.
만약 "서버에 ANTHROPIC_API_KEY 환경변수가 설정되어 있지 않습니다" 에러가 뜨면
Vercel 프로젝트 Settings → Environment Variables에서 키가 제대로 등록됐는지,
등록 후 **재배포(Redeploy)**를 했는지 확인하세요. (환경변수는 재배포해야 반영됩니다.)

## 비용 / 모델 변경

`api/interview.js`의 `model` 값을 바꾸면 됩니다.
- `claude-sonnet-5` (기본값, 품질 우선)
- `claude-haiku-4-5-20251001` (더 저렴, 응답 조금 단순해질 수 있음)

10개 질문 + 최종 평가까지 한 판에 API 호출이 총 11번 발생합니다.
방문자가 많아질 것 같으면 haiku로 바꾸는 걸 추천합니다.

## 로컬에서 테스트하기

Vercel CLI로 서버리스 함수까지 포함해서 로컬 테스트 가능합니다.

```bash
npm i -g vercel
vercel dev
```

`ANTHROPIC_API_KEY`는 `vercel env pull` 로 받아오거나, 프로젝트 루트에
`.env` 파일을 만들어 `ANTHROPIC_API_KEY=sk-ant-...` 로 넣어주세요.
(`.env`는 절대 GitHub에 커밋하지 마세요 — `.gitignore`에 이미 포함되어 있습니다.)
