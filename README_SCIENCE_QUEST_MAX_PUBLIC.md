# SCIENCE QUEST MAX PUBLIC — 실행 및 인터넷 공개 배포

## 포함 파일
- `science_quest_server.html`: 퀴즈 UI와 개발자 콘솔
- `server.js`: Node.js API 서버
- `package.json`: 실행 명령
- `render.yaml`: Render 배포 예시
- `Dockerfile`: 컨테이너 실행 예시
- `.gitignore`: 비밀값 및 데이터 파일 제외
- `.env.example`: 로컬 환경변수 예시(실제 비밀값을 넣지 않음)
- `SECURITY.md`: 공개 배포 전 보안 요구사항과 한계
- `DEPLOY_CHECKLIST.md`: 배포·운영 점검표
- `PRIVACY_POLICY_TEMPLATE.html`, `TERMS_TEMPLATE.html`: 운영자 검토가 필요한 한국어 정책 초안
- `server.test.js`: API 통합/보안 스모크 테스트
- `.github/workflows/test.yml`: GitHub에 push/PR할 때 자동 테스트

## 로컬 실행
필수: Node.js 18 이상. `.env.example`은 참고용이며 Node.js는 `.env` 파일을 자동으로 읽지 않습니다. 아래 명령처럼 환경변수를 직접 설정하세요.

Windows PowerShell:
```powershell
$env:ADMIN_KEY = [guid]::NewGuid().ToString() + [guid]::NewGuid().ToString()
npm start
```

macOS/Linux:
```bash
export ADMIN_KEY="$(node -e "console.log(require('crypto').randomBytes(32).toString('hex'))")"
npm start
```
브라우저에서 `http://localhost:3000` 접속. 상태 확인은 `http://localhost:3000/api/health`.

## 자동 테스트
Node.js 18 이상에서 프로젝트 폴더에 들어가 실행합니다.
```bash
npm test
node --check server.js
```
테스트는 임시 데이터 디렉터리를 사용하므로 실제 `data/db.json`을 수정하지 않습니다.

## 개발자 콘솔
1. 퀴즈 화면에서 개발자 모드를 연다.
2. 서버 시작 시 설정한 `ADMIN_KEY`를 입력하고 인증한다.
3. 문제를 추가하거나 목록에서 수정·삭제한다.
4. JSON 내보내기로 백업하거나 JSON 가져오기로 문제 묶음을 추가한다.

문제 JSON 한 항목 예시:
```json
{
  "q": "물의 화학식은?",
  "a": ["CO2", "H2O", "O2", "NaCl"],
  "c": 1,
  "e": "물은 수소 원자 두 개와 산소 원자 한 개로 이루어집니다.",
  "t": "화학",
  "d": "쉬움",
  "h": "원소 기호와 분자식을 구분해 보세요."
}
```
`c`는 0~3의 정답 선택지 인덱스입니다. JSON 가져오기는 기존 문제를 유지하면서 새 문제를 추가합니다.

## 인터넷 공개 배포 — Render 예시
1. 전체 폴더를 Git 저장소에 올린다.
2. Render에서 저장소를 연결해 Web Service를 만든다.
3. Start Command를 `npm start`로 지정한다.
4. `ADMIN_KEY` 환경 변수는 플랫폼에서 생성하거나 32바이트 이상 무작위 비밀값으로 설정한다. `NODE_ENV=production`도 설정해야 하며, 이때 유효한 `ADMIN_KEY`가 없으면 서버가 시작하지 않는다. 키를 저장소, HTML, 공개 문서에 넣지 않는다.
5. 영구 디스크를 `/opt/render/project/src/data`에 연결한다.
6. 배포 후 `https://서비스주소/api/health`를 확인한다.

호스팅 업체의 플랜별 영구 디스크 지원과 YAML 설정은 배포 시점의 공식 문서로 확인해야 합니다. `render.yaml`은 참고 설정입니다.

## 운영 보안 및 현실적인 한계
- `SECURITY.md`와 `DEPLOY_CHECKLIST.md`를 배포 전에 확인하고, `/privacy.html` 및 `/terms.html`의 템플릿 대괄호를 실제 운영자·보유기간·문의처 정보로 교체한 뒤 법률 검토를 받으세요. 현재 페이지는 완성된 법률 문서가 아닙니다.
- 관리자 키가 32자 미만이거나 없으면 경고하며, `NODE_ENV=production`에서는 서버 시작을 거부합니다. 프록시 헤더는 기본적으로 신뢰하지 않으며, 신뢰 가능한 프록시 환경에서만 `TRUST_PROXY=1`로 활성화하세요.
- 서버는 정적 파일을 허용 목록으로만 제공하여 `data/db.json`과 서버 소스 파일의 직접 노출을 막습니다.
- 로그인 비밀번호는 salt + scrypt 해시로 저장합니다. 관리자 API는 `X-Admin-Key`를 검증하며, 기본 키로 관리자 접근을 허용하지 않습니다.
- 간단한 요청 제한은 인스턴스 메모리 기준입니다. 다중 인스턴스 공개 서비스에는 프록시/공유 저장소 기반 제한을 추가하세요.
- 현재 저장소는 JSON 파일이며, 세션도 메모리에 있습니다. 따라서 작은 단일 인스턴스 배포용입니다. 여러 인스턴스를 띄우면 세션과 JSON 저장소가 공유되지 않아 데이터/로그인 문제가 발생할 수 있습니다. 서비스 규모가 커지면 PostgreSQL 및 영속 세션 저장소로 옮겨야 합니다.
- 개발자 콘솔에서 등록한 서버 문제는 공개 퀴즈 API를 통해 화면의 문제 풀에 추가됩니다. 공개 문제 응답에는 정답 인덱스와 해설이 포함되지 않습니다. 서버 문제 퀴즈는 로그인 후 서버가 1시간 유효한 퀴즈 세션을 발급하며, 각 문제는 세션당 한 번만 답할 수 있습니다. 서버가 채점 결과를 반환하고 제출된 세션은 재사용할 수 없습니다.
- 공식 서버 점수는 서버가 저장된 정답과 제출된 선택지를 비교해 계산합니다. 이전의 클라이언트 점수 API(`/api/scores`)는 허위 점수 방지를 위해 비활성화(HTTP 410)했습니다. 공식 순위는 서버 문제만으로 구성된 한 판을 완료해야 등록됩니다. 로컬 내장 문제는 여전히 브라우저에서 채점되므로 공식 서버 순위와 구분됩니다.
- 실제 서비스 운영 전 HTTPS, 백업 복구 테스트, 모니터링, 개인정보 처리방침, 이용약관, 가입/로그인 abuse 방어, 장애 대응 절차를 마련하세요.
- “완벽한 보안”은 보장할 수 없습니다. 이 패키지는 배포 가능한 강화 버전이며, 규모가 큰 상용 서비스의 보안 감사나 운영 인프라를 대체하지 않습니다.

## 현재 검증 범위
`npm test`는 상태 확인, 정적 페이지, 비공개 파일 차단, 회원가입/로그인, 관리자 키, 문제 CRUD, 공개 응답의 정답 숨김, 서버 채점 및 클라이언트 점수 제출 차단, 잘못된 URL을 확인합니다. 이는 회귀 테스트이지 완전한 보안 감사나 대규모 부하 테스트가 아닙니다.

## 추가 공개 페이지
- `/privacy.html`: 개인정보 처리방침 초안(배포 전 운영자 정보 입력 및 검토 필요)
- `/terms.html`: 이용약관 초안(배포 전 운영 정책에 맞춘 검토 필요)

## API
- `GET /api/health`
- `GET /api/questions` (정답/해설 비공개)
- `GET /api/leaderboard`
- `POST /api/auth/register`, `POST /api/auth/login`
- `POST /api/quiz/start` (로그인 필수, 서버 퀴즈 세션 발급)
- `POST /api/quiz/answer` (로그인 필수, 세션 문제 1회 답안 제출 및 서버 채점)
- `POST /api/quiz/submit` (로그인 필수, 세션 답안으로 공식 점수 계산; 세션 재사용 불가)
- `POST /api/scores` (비활성화, HTTP 410)
- 관리자: `GET /api/admin/status`, `GET /api/admin/questions`, `POST /api/admin/questions`, `PUT/DELETE /api/admin/questions/:id`, `POST /api/admin/questions/import`, `GET /api/admin/export`
