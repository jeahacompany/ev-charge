# ⚡ 전기차 충전 관리 앱

개인용(1인) 전기차 충전 기록·요금 계산 웹앱. 모델 3 / 모델 Y 두 대를 분리 관리한다.
2026-07-09 제작. 개발·운영 전 과정을 Claude(AI)가 수행했다.

## 접속 주소

- **앱**: https://jeahacompany.github.io/ev-charge/
- 폰에서는 브라우저 "홈 화면에 추가"로 앱처럼 사용

## 뭐로 만들었나 (구조)

서버 없는 단일 HTML 앱 + 구글 스프레드시트 백업. 전부 무료.

```
[index.html 파일 1개]  ←  GitHub Pages가 호스팅 (정적 웹)
        │
        │  저장/접속 때마다 자동
        ▼
[구글 Apps Script 웹앱]  ←  백업 API (doPost=저장, doGet=동기화 읽기)
        │
        ▼
[구글 스프레드시트 "전기차충전 자동백업"]
   ├─ 기록 탭: 사람이 읽는 전체 기록 (엑셀처럼)
   └─ 백업JSON 탭: 복원용 스냅샷 최근 300개 (JSON을 4.5만자씩 여러 칸에 분할 저장)
```

- 기록 데이터는 각 기기(폰/PC) 브라우저 localStorage에 저장되고,
  앱을 열 때마다 시트의 최신 스냅샷과 **자동 동기화**(id 기준 합집합 + 삭제 톰스톤)된다.
- 요금 계산: 한전 전기차 충전전력요금(자가소비용) 저압, 계절·시간대별 단가.
  토요일 최대→중간, 일요일·공휴일 전시간 경부하(2026~27 공휴일 내장).

## 키·계정은 어디에 있나

| 무엇 | 계정 | 비고 |
|---|---|---|
| GitHub 저장소 | github.com/jeahacompany/ev-charge | 계정 jeahacompany, 이 PC에 git 인증 저장됨 |
| Apps Script 프로젝트 | ho910830@gmail.com | script.google.com → "제목 없는 프로젝트" |
| 백업 스프레드시트 | ho910830@gmail.com 드라이브 | 파일명 "전기차충전 자동백업" |
| 백업 API 주소 | index.html 안의 `BACKUP_URL` 상수 | 시크릿 아님(쓰기/읽기 전용 엔드포인트) |

비밀번호·API키 같은 시크릿은 **없다**. (Apps Script가 본인 계정 권한으로 실행됨)

## 배포 방법 (수정 → 반영)

```
1. C:\Users\windows11\전기차충전\index.html 수정
2. git add index.html && git commit -m "설명" && git push
3. 1~2분 뒤 반영. 앱이 새 버전을 감지해 사용자에게 파란 업데이트 배너를 띄움
```
- 수정할 때마다 index.html 안의 `APP_VER` 을 올리고 `CHANGELOG` 배열에 한 줄 추가할 것.

### 롤백 (배포 잘못했을 때)

```
git log --oneline          ← 커밋 목록 확인
git revert HEAD --no-edit  ← 마지막 배포 취소
git push
```

## 문제 생기면 어디부터 보나 (순서대로)

1. **앱이 안 열림** → https://www.githubstatus.com (GitHub Pages 장애 여부)
2. **기록이 안 보임** → 앱 상단 차량 선택(모델 3/Y)과 월(◀▶) 확인. 기기 간 안 맞으면 앱을 껐다 켜서 동기화
3. **저장이 안 됨** → 설정 탭 맨 아래 "⚠ 최근 오류" 표시 확인. 백업 시트의 백업JSON에도 lastError로 남음
4. **백업이 안 됨** → 설정 탭 "자동 백업: ..." 상태 확인 → script.google.com(ho910830)에서 실행 기록 확인
5. **데이터가 날아감** → 아래 복구 절차

## 복구 절차 (데이터 사고)

- **기록을 실수로 지움**: 다른 기기가 아직 안 열렸다면 그 기기에서 CSV 내보내기.
  이미 동기화됐다면 → 시트 "백업JSON" 탭에서 지우기 전 시점 행(3번째 칸부터 끝까지 이어붙이면 JSON)을 복사
  → 텍스트 파일(.json)로 저장 → 앱 설정 "JSON 복원"으로 불러오기.
- **기기를 잃어버림/바꿈**: 새 기기에서 앱 주소만 열면 시트에서 전체 기록이 자동으로 들어온다.
- **시트를 실수로 지움**: 드라이브 휴지통에서 30일 내 복원. 기기들에 데이터가 있으므로 다음 저장 때 시트가 자동 재생성된다.
- **Apps Script를 지움**: 기기 데이터는 안전. 새 스크립트를 만들어 배포하고 index.html의 BACKUP_URL만 교체하면 된다.
  (스크립트 코드는 이 저장소 히스토리와 아래 "백업 API 코드" 참고)

## 사용자(소유자)가 직접 해야 하는 것

1. **1년에 한 번** (연말): 다음 해 공휴일(설날·추석 등 음력) 목록 갱신을 Claude에게 요청
   — 현재 2026~2027년까지 내장, 그 이후는 고정 공휴일만 자동 인식
2. 전기요금 개정 시: 앱 설정 탭에서 계절별 단가 수정 (화면에서 바로 가능)
3. 그 외 수정·기능 추가: Claude에게 요청 (이 폴더가 git 저장소라 바로 배포 가능)

## 파일 안내

| 파일 | 용도 |
|---|---|
| `index.html` | 앱 전체 (화면+로직+데이터 계층, 한글 주석) |
| `serve.ps1` | 로컬 미리보기 서버 (PowerShell, 포트 8734) — 배포와 무관 |
| `artifact.html` | 폐기된 초기 배포본 — 사용 안 함 |
| `README.md` | 이 문서 |

## 백업 API 코드 (Apps Script, 참고용 사본)

script.google.com(ho910830) "제목 없는 프로젝트"에 배포돼 있는 코드와 동일:

```javascript
// doPost: 앱이 보내는 전체 데이터를 '백업JSON'(스냅샷)과 '기록'(읽기용)에 저장
// doGet: 최신 스냅샷을 돌려줌 (기기 간 동기화용)
// 시트 한 칸은 5만 자 제한 → JSON을 4.5만 자씩 여러 칸에 분할 저장
function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);
  try {
    var ss = getSS_();
    var d = JSON.parse(e.postData.contents);
    var raw = ss.getSheetByName('백업JSON') || ss.insertSheet('백업JSON');
    var json = JSON.stringify(d.db);
    var parts = [];
    for (var i = 0; i < json.length; i += 45000) parts.push(json.substring(i, i + 45000));
    raw.appendRow([new Date(), d.device || ''].concat(parts));
    if (raw.getLastRow() > 300) raw.deleteRows(1, raw.getLastRow() - 300);
    var rec = ss.getSheetByName('기록') || ss.insertSheet('기록');
    rec.clearContents();
    rec.appendRow(['차량','날짜','요일','시작','종료','충전량(kWh)','부하구간','총요금(원)','평균단가','KM','구분']);
    var rows = d.rows || [];
    if (rows.length) rec.getRange(2, 1, rows.length, rows[0].length).setValues(rows);
    var def = ss.getSheetByName('Sheet1') || ss.getSheetByName('시트1');
    if (def && ss.getSheets().length > 2) ss.deleteSheet(def);
    return ContentService.createTextOutput('ok');
  } finally {
    lock.releaseLock();
  }
}
function doGet(e) {
  var ss = getSS_();
  var raw = ss ? ss.getSheetByName('백업JSON') : null;
  var out = '{}';
  if (raw && raw.getLastRow() >= 1) {
    var r = raw.getLastRow();
    var lastCol = Math.max(3, raw.getLastColumn());
    var vals = raw.getRange(r, 3, 1, lastCol - 2).getValues()[0];
    var joined = vals.join('');
    if (joined) out = joined;
  }
  return ContentService.createTextOutput(out).setMimeType(ContentService.MimeType.JSON);
}
function getSS_() {
  var name = '전기차충전 자동백업';
  var files = DriveApp.getFilesByName(name);
  return files.hasNext() ? SpreadsheetApp.open(files.next()) : SpreadsheetApp.create(name);
}
```

## 비용

- GitHub Pages: 무료 / Apps Script·스프레드시트: 무료 (개인 사용량은 구글 무료 한도 내)
- 유지비 0원. 결제 정보 등록된 곳 없음.
