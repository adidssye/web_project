# 어딧세이 · Odysay

여행지를 지도에서 발견하고, 직접 다녀온 장소와 여행 이야기를 공유하는 Flask 기반 웹 서비스입니다. 여행지 등록·찜·리뷰·커뮤니티부터 마이페이지, 1:1 문의, 관리자 신고 처리까지 제공합니다.

> 이 문서는 2026년 10월 14일 작업 폴더의 코드를 기준으로 작성했습니다. 새 설치에는 회원·여행지 데이터가 포함되지 않습니다.

## 주요 기능

| 구분 | 기능 |
| --- | --- |
| 지도·홈 | Leaflet 지도와 마커, 인기·최근 여행지, 여행지 상세 연결 |
| 회원 | 회원가입, 닉네임 중복 확인, 로그인·로그아웃, 프로필·비밀번호 변경, 회원 탈퇴 |
| 여행지 | 국가별 탐색, 장소·소개·추천 이유·사진·여행 태그 등록, 본인 여행지 수정·삭제 |
| 여행지 상세 | 찜, 별점 리뷰, 리뷰 추천, 여행톡·댓글·답글, 주변 등록 여행지 |
| 커뮤니티 | 여행 후기·여행 팁·자유 게시판, 검색·정렬, 좋아요·댓글 |
| 마이페이지 | 내 작성 글·여행지·찜 목록, 활동 현황, 관심 여행지·선호 스타일 |
| 문의·신고 | 본인 문의 작성·답변 확인, 콘텐츠 신고, 관리자 접수·처리 |
| 화면 설정 | 테마 전환, 한국어·영어 표시 및 저장된 콘텐츠 번역 활용 |

홈의 인기 여행지는 찜 수를 기준으로 표시합니다. 커뮤니티의 여행 후기는 등록된 여행지와 연결되며, 여행 팁·자유 게시판은 별도 게시글입니다.

## 기술 구성

- 서버: Python, Flask, Jinja2
- 데이터: SQLite, Flask-SQLAlchemy
- 폼·요청 검증: Flask-WTF, WTForms, CSRF 보호
- 화면: HTML, CSS, JavaScript, 일부 화면 Bootstrap
- 지도: Leaflet, OpenStreetMap
- 장소 검색: Gemini 명칭 보정 → 국내 TMAP 검색 → Nominatim 대체 검색
- 이미지 처리: Pillow

정확한 패키지 버전은 [requirements.txt](requirements.txt)를 참고하세요. 로컬 Python은 3.12.10이며 Dockerfile도 Python 3.12를 기준으로 합니다. 새 환경에서의 전체 의존성 설치는 별도 확인이 필요합니다.

## 로컬 실행

### 1. 저장소 받기

```bash
git clone https://github.com/adidssye/web_project.git
cd web_project
python -m venv .venv
```

사용하는 터미널에 맞게 가상환경을 활성화합니다.

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```bash
# Windows Git Bash
source .venv/Scripts/activate

# macOS / Linux
source .venv/bin/activate
```

이후 명령은 가상환경이 활성화된 상태에서 실행합니다.

```bash
python -m pip install -r requirements.txt
python setup_env.py
```

`setup_env.py`는 `.env`가 없을 때 무작위 `SECRET_KEY`를 생성합니다. 기존 `.env`는 덮어쓰지 않습니다.

### 2. 환경변수 설정

프로젝트 루트의 `.env`를 편집합니다. 아래 설명용 값을 실제 키로 사용하지 마세요.

```dotenv
SECRET_KEY=setup_env.py가_생성한_값을_유지
GEMINI_API_KEY=
TMAP_API_KEY=
ADMIN_USER_ID=
GEMINI_TRANSLATION_ENABLED=false
GEMINI_FREE_TIER_CONFIRMED=false
GEMINI_TRANSLATION_MODEL=gemini-3.6-flash
```

| 변수 | 용도 |
| --- | --- |
| `SECRET_KEY` | 세션·CSRF 서명. 생성된 임의 값을 사용하며 관리자 기능에는 기본값 `dev`를 사용할 수 없습니다. |
| `GEMINI_API_KEY` | 등록 시 장소명 보정과 선택적 콘텐츠 번역에 사용합니다. |
| `TMAP_API_KEY` | 국내 장소의 좌표 검색에 사용합니다. |
| `ADMIN_USER_ID` | 관리자 회원의 숫자 DB ID. 로그인 아이디 문자열이 아닙니다. |
| `GEMINI_TRANSLATION_ENABLED` | 콘텐츠 번역 생성 활성화 여부. 기본 비활성입니다. |
| `GEMINI_FREE_TIER_CONFIRMED` | 운영자가 번역용 프로젝트의 과금·할당량 조건을 확인했다는 설정입니다. 무료 사용을 보장하는 설정은 아닙니다. |
| `GEMINI_TRANSLATION_MODEL` | 콘텐츠 번역 모델. 코드의 허용 목록과 실제 계정의 사용 가능 모델이 모두 맞아야 합니다. |

현재 `setup_env.py`에는 과거 설정인 `KAKAO_REST_API_KEY`, `GEMINI_MODEL`이 포함되어 있습니다. 실제 여행지 등록 경로는 `TMAP_API_KEY`를 사용하므로 직접 추가해야 합니다. 등록용 Gemini 모델은 현재 코드에 `gemini-3.6-flash`로 지정되어 있으며, `GEMINI_TRANSLATION_MODEL`과 별개입니다. 모델의 실제 사용 가능 여부는 검증하지 않았습니다.

API 키가 비어 있으면 해당 명칭 보정·TMAP 검색을 건너뛰고 가능한 대체 검색을 시도합니다. 지도 타일과 장소 검색에는 인터넷 연결이 필요합니다. 번역을 꺼도 Gemini 키가 설정되어 있으면 여행지 등록 시 명칭 보정 API는 호출될 수 있습니다.

### 3. 새 DB 생성 — 최초 설치만

현재 Git 저장소는 `migrations/`를 제외하므로, 새로 받은 저장소에서 `flask db upgrade`만 실행하는 방식으로는 초기화할 수 없습니다.

**기존 DB를 가져온 경우에는 이 단계 대신 아래 ‘기존 데이터 사용’을 따르세요.**

```bash
python -m flask --app odysay shell
```

열린 Python 셸에서:

```python
from odysay import db
db.create_all()
exit()
```

현재 모델의 전체 테이블을 생성합니다. `db.create_all()`은 기존 테이블의 컬럼을 갱신하는 명령이 아닙니다. DB는 프로젝트 루트의 `odysay.db`에 저장됩니다.

### 4. 서버 실행

```bash
python -m flask --app odysay run --debug
```

브라우저에서 [http://127.0.0.1:5000](http://127.0.0.1:5000)에 접속합니다. 첫 화면은 지도이며, 회원가입 후 로그인해 회원 기능을 사용할 수 있습니다. 종료는 터미널에서 `Ctrl+C`를 누릅니다. `--debug`는 로컬 개발용입니다.

## 사용자 이용 순서

### 회원가입·로그인

1. 지도 화면에서 회원가입으로 이동합니다.
2. 아이디·이메일·비밀번호·닉네임·생년월일·성별을 입력합니다.
3. 닉네임 중복 확인 후 가입을 완료합니다. 가입 시 자동 로그인됩니다.
4. 지도에서 여행 둘러보기로 홈페이지에 진입합니다.

현재 홈페이지 미리보기는 비로그인 상태에서도 접근할 수 있으나 여행지 목록·상세, 커뮤니티 회원 화면, 등록, 마이페이지 등은 로그인이 필요합니다.

### 여행지 등록·탐색

1. 여행지 등록 화면에서 등록 가이드를 확인합니다.
2. 국가·지역·장소명, 카테고리, 소개와 추천 이유를 입력합니다.
3. 사진과 선택적인 맛집·주변 정보를 추가하고 여행 스타일 태그를 선택합니다.
4. 가이드라인에 동의하고 등록합니다.
5. 서버의 명칭 보정·좌표 검색과 저장이 끝나면 상세 화면으로 이동합니다.

위치 검색은 장소를 찾지 못하면 지역·국가 단위 위치를 사용할 수 있으므로 등록 후 지도 위치를 확인하세요. 여행지 상세에서 찜·리뷰·여행톡을 이용하며, 작성자는 자신의 여행지를 수정·삭제할 수 있습니다.

### 커뮤니티·마이페이지

- 커뮤니티에서 분류·검색·정렬로 글을 탐색하고, 여행 팁이나 자유글을 작성합니다.
- 게시글에 좋아요·댓글을 남기고 본인 작성 내용을 관리합니다.
- 마이페이지에서 작성 글, 등록 여행지, 찜한 여행지와 활동 현황을 확인합니다.
- 프로필에서 닉네임·대표 사진·비밀번호를 변경합니다. 비밀번호 변경 후에는 다시 로그인합니다.
- 회원 탈퇴는 계정 비활성화 방식입니다. 기존 회원정보와 게시물은 보관되며 로그인은 차단됩니다.

### 문의·신고

- 마이페이지의 문의 메뉴에서 1:1 문의를 작성하고 답변을 확인합니다. 일반 회원은 본인 문의만 조회합니다.
- 신고 버튼에서 사유를 입력하면 관리자 신고 목록에 접수됩니다.
- 동일 회원의 동일 대상 신고는 중복 생성되지 않습니다.
- 관리자에게 이메일·푸시를 발송하지는 않습니다.

## 관리자 사용법

### 관리자 지정

1. 일반 회원가입으로 관리자에게 사용할 계정을 만듭니다.
2. Flask 셸에서 해당 계정의 숫자 ID를 확인합니다.

```bash
python -m flask --app odysay shell
```

```python
from odysay.models import User
user = User.query.filter_by(username='관리자로_사용할_로그인아이디').first()
print(user.id if user else '해당 회원 없음')
exit()
```

3. `.env`에 `ADMIN_USER_ID=확인한숫자`를 설정하고 서버를 재시작합니다.
4. 설정을 검사합니다.

```bash
python -m flask --app odysay admin-check
```

### 콘텐츠·문의·신고 처리

| 주소 | 기능 |
| --- | --- |
| `/admin/` | 콘텐츠 조회·수정·게시 중단·복원 |
| `/admin/logs` | 관리자 처리 이력 |
| `/homepage/inquiries/` | 문의 목록·답변 |
| `/admin/reports` | 신고 사유·신고 당시 내용 확인, 처리 결과 기록 |

문의·신고 화면에서 탭으로 전환할 수 있습니다. 신고에서 **콘텐츠 관리**로 이동해 필요한 게시 중단 등을 수행한 뒤 신고 처리 결과와 사유를 저장합니다. **신고 종결 자체는 게시물을 숨기지 않습니다.**

게시 중단은 원문을 보존하고 공개 노출을 제한합니다. 부모 콘텐츠가 중단되면 하위 콘텐츠도 가려지며, 개별 중단한 자식은 부모 복원 후에도 계속 숨겨집니다. 지정 관리자 계정은 탈퇴가 제한됩니다.

자세한 설명: [관리자 안내](docs/ADMIN_GUIDE.md), [신고 처리 안내](docs/REPORT_GUIDE.md)

## 기존 데이터 사용

서버를 종료하고 `odysay.db`와 `odysay/static/uploads/`를 함께 백업하세요. DB만 옮기면 사진 파일은 복원되지 않습니다.

기존 DB를 프로젝트 루트에, 업로드 파일을 같은 상대 경로에 배치한 뒤:

```bash
python -m flask --app odysay admin-init-db
python update_database.py --check
```

`admin-init-db`는 기존 SQLite DB를 `instance/admin-backups/`에 백업하고 누락된 관리·신고 테이블을 추가합니다. 전체 앱 DB 초기화 명령은 아닙니다. `update_database.py`의 일반 생성 허용 목록에는 `content_reports`가 없으므로 관리 테이블 준비를 먼저 실행합니다.

점검 결과가 지원 가능한 변경이고 오류가 없을 때만 적용합니다.

```bash
python update_database.py --apply
python update_database.py --check
```

이 도구는 기존 프로젝트 DB의 허용된 테이블·컬럼만 추가하며, 알 수 없는 스키마 차이는 거절합니다. 검사 오류가 나오면 반복 적용하거나 임의로 `db stamp`를 실행하지 말고 해당 차이를 먼저 확인하세요.

## 영어 번역 설정

번역은 저장 시 생성한 영어 사본을 조회하는 구조입니다. 언어 전환이나 조회만으로 Gemini 번역을 새로 호출하지 않습니다. 기존 글의 번역은 별도 보충이 필요할 수 있습니다.

번역을 사용하려면 API 키, 실제 사용 가능한 허용 모델, 과금·할당량 조건을 확인한 뒤 `.env`의 번역 활성화·확인 값을 `true`로 설정하고 재시작합니다.

```bash
# 외부 번역 호출 없이 누락 번역 작업을 등록
python -m flask --app odysay translate-missing --enqueue-only --limit 20

# 설정 확인 후 실제 번역 생성: 외부 API 호출 발생
python -m flask --app odysay translate-missing --limit 20
```

키·모델·할당량 오류 시 번역이 제공되지 않거나 원문이 표시될 수 있습니다. 모델명과 허용 목록은 구현값이며 외부 서비스의 현재 지원·무료 조건을 보증하지 않습니다.

## 폴더 구성

```text
web_project/
├── README.md
├── requirements.txt
├── config.py                 # DB·환경변수 설정
├── setup_env.py              # 로컬 .env 생성
├── update_database.py        # 기존 DB 검사·허용된 변경 적용
├── Dockerfile
├── odysay/
│   ├── __init__.py           # Flask 앱 생성·기능 등록
│   ├── models.py             # DB 모델
│   ├── forms.py              # 폼 검증
│   ├── moderation.py         # 관리자 권한·콘텐츠 노출 정책
│   ├── admin_commands.py     # 관리 DB 준비·설정 검사
│   ├── views/                # 화면·API 처리
│   ├── services/             # 번역 등 서비스
│   ├── templates/            # HTML 화면
│   └── static/               # CSS·JavaScript·이미지·업로드
├── docs/                     # 관리자·신고 안내
└── tests/                    # 자동 검증
```

## 테스트

가상환경에서 Python 테스트를 실행합니다.

```bash
python -m unittest discover -s tests -q
```

Node.js가 설치되어 있다면 테마·언어 URL 처리 테스트도 실행할 수 있습니다.

```bash
node --test tests/theme.test.cjs tests/test_language_url.cjs
```

테스트에는 관리자 권한·콘텐츠 관리, 신고, DB 갱신, 번역, 지도 팝업, 테마 등이 포함되어 있습니다. 자동 테스트 결과만으로 실제 외부 API·지도 타일·브라우저 화면까지 검증되지는 않습니다.

## 문제 해결

| 증상 | 확인 방법 |
| --- | --- |
| `no such table` | 새 설치인지 기존 DB인지 구분해 DB 준비 절차를 수행합니다. |
| `no such column` | 서버 종료 후 `update_database.py --check` 결과를 확인합니다. |
| 관리자 화면 403 | `ADMIN_USER_ID`, 회원 활성 상태, `SECRET_KEY`를 확인하고 `admin-check`를 실행합니다. |
| 요청 만료·CSRF 오류 | 새로고침하고 다시 로그인합니다. 오래 열린 화면의 요청 토큰을 확인합니다. |
| 여행지 좌표가 부정확함 | 국가·지역·장소명을 구체적으로 입력하고 API 설정·대체 검색 결과를 확인합니다. |
| 장소 수정 후 예전 지도 위치 표시 | 현재 일반 여행지 수정 경로에는 좌표 재검색이 연결되어 있지 않습니다. |
| 사진이 표시되지 않음 | DB의 파일명과 `static/uploads/` 파일이 함께 존재하는지 확인합니다. |
| 신고가 목록에서 안 보임 | 관리자 계정과 신고 상태 필터를 확인합니다. 처리 완료 신고는 ‘전체’에서 확인합니다. |
| CSS 수정이 안 보임 | 브라우저에서 `Ctrl+F5`로 새로고침합니다. |

## 배포·저장소 관리 참고

- `.env`, DB, 사용자 업로드, 백업, 가상환경은 Git 추적 제외 대상입니다. 이미 추적된 파일은 `.gitignore`만으로 제외되지 않습니다.
- 현재 Dockerfile은 Flask 개발 서버로 실행합니다. 운영 배포용 WSGI 서버 설정은 별도로 필요합니다.
- SQLite와 업로드 사진은 영속 저장소에 보관해야 재배포 시 데이터 유실을 방지할 수 있습니다.
- 현재 `.dockerignore`에는 `odysay.db` 포함 예외가 있습니다. 이미지 빌드 시 로컬 DB가 포함될 수 있으므로 배포 데이터 구성을 확인하세요.
- API 키·실제 회원 데이터·사용자 사진을 공개 저장소나 공개 배포 이미지에 넣지 마세요.
- OpenStreetMap 출처 표시는 유지하세요.

이 README 작성 과정에서는 실제 외부 API 호출, 새 가상환경 설치, 운영 배포를 실행하지 않았습니다.
