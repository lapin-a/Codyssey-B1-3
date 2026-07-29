자유 주제: 학습 사이트 월간 총 반영시간 자동 알림

반복 업무 정의
매달 Codyssey 학습 플랫폼에 로그인해서 "총 반영시간"을 수동으로 확인하고 기록해야 하는 반복 작업을, 자동으로 조회하고 값이 바뀌었을 때만 메일로 알려주도록 자동화함.

도구 선정: Make
선정 이유:

로그인 → 세션 쿠키 추출(정규식) → 여러 API 순차 호출 → 조건 분기 → 이메일 발송까지, HTTP/Text parser/Router/Data store 등 다양한 내장 모듈을 하나의 시각적 캔버스에서 조합할 수 있어 복잡한 인증 흐름을 다루기에 적합했음
무료 플랜의 Operation 한도 내에서 매일 스케줄 실행 가능

워크플로우 흐름

| 단계 |	모듈 |	역할 |
| --- | --- | --- |
| 1	| Scheduler (Trigger)	매일 정해진 시각에 자동 실행 |
| 2	| HTTP – authenticate	아이디/비밀번호로 로그인, 세션 쿠키 발급 |
| 3	| Text parser	응답 헤더에서 JSESSIONID 값 추출 (정규식) |
| 4	| HTTP – menu/my	세션 유효성 확인용 호출 |
| 5	| HTTP – user/info/detail	로그인 계정의 mbrId 동적 조회 |
| 6	| HTTP – secom/detail	mbrId + 이번 연/월로 반영시간 원본 데이터 조회 |
| 7	| Set variable ×3	세션별 duration_seconds 합산 → 시/분 변환 |
| 8	| Data store – Get a record	지난 실행 때 저장해둔 총 반영시간과 비교 준비 |
| 9	| Router (조건 분기)	이번 값과 지난 값이 같은지/다른지에 따라 경로 분리 |
| 9-A	| (변경됨 경로) Gmail – Send an email	"업데이트" 메일 발송 → Data store 값 갱신 |
| 9-B	| (동일함 경로) Gmail – Send an email	"변동없음" 메일 발송 |

Trigger / Action / 조건 분기 요구사항 충족 근거

Trigger: Scheduler 1개
Action: HTTP 요청 5개 + 이메일 발송 2개(분기별) 등 다수
조건 분기: Router에서 totalSeconds vs lastTotalSeconds 비교로 2개 경로 분리, 실제로 두 경로 모두 실행 이력 확보(1차 실행=변경 경로, 2차 실행=동일 경로)
