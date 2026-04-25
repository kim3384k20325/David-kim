---
name: analytics-parser
description: Use when ingesting YouTube Studio analytics — CSV exports or dashboard screenshots — for a video or channel. Defines expected CSV column variants, screenshot OCR prompts, the 5-metric core schema, validation rules (≥3 of 5 metrics required), and a normalized output format that performance-analyst consumes.
---

# Analytics Parser

YouTube Studio 데이터(CSV / 스크린샷)를 표준화된 지표 객체로 변환한다. 결측·표기 변종을 견디고, 추측 금지 원칙을 강제한다.

## 1. 언제 사용하는가

- Mode A Step A4 — 기존 영상 베이스라인 진단
- Mode B Step B2 — 신규 업로드 후 D+3 / D+7 / D+14 데이터 수집

## 2. 입력 형태

| 형태 | 우선순위 | 비고 |
|------|---------|------|
| YouTube Studio CSV 내보내기 | 1 (가장 정확) | "분석 → 콘텐츠/시청자/도달범위 → 내보내기" |
| 대시보드 스크린샷 (PNG/JPG) | 2 (CSV 불가 시) | 멀티모달 비전으로 직접 독해 |
| 사용자 자유 텍스트 보고 | 3 (최후) | 사용자에게 스크린샷·CSV로 보강 요청 |

## 3. 핵심 5지표 (스키마 표준)

| 키 | 의미 | YouTube Studio 표기 변종 |
|----|------|------------------------|
| `impressions` | 노출 수 | `Impressions`, `노출수` |
| `ctr` | 노출 클릭률 (0–1 또는 %) | `Impressions click-through rate`, `노출 클릭률 (%)` |
| `avg_view_duration_sec` | 평균 시청 시간(초) | `Average view duration`, `평균 시청 지속 시간` (mm:ss → 초로 변환) |
| `retention_curve` | 시청 지속률 곡선 (시간 % 또는 timestamp:percent 시계열) | `Audience retention`, `시청 지속률` |
| `traffic_sources` | 트래픽 소스 분포 (소스명 → 시청 시간 비율) | `Traffic source types`, `트래픽 소스 유형` |

**보조 지표** (있으면 함께 캡처):
- `search_terms` — 유입 검색 키워드 리스트 (소스 = "YouTube 검색")
- `views`, `watch_time_hours`, `subscribers_gained`, `likes`, `comments`
- `audience_geo`, `audience_age` (선택)

## 4. CSV 파싱 가이드

1. **인코딩**: UTF-8 with BOM 또는 cp949 (한국어 export). BOM 자동 제거.
2. **헤더 정규화**: 공백 제거 + 소문자 + 한국어→영문 매핑.
   - `노출수` → `impressions`
   - `평균 시청 지속 시간` → `avg_view_duration` (mm:ss → seconds)
   - `노출 클릭률 (%)` → `ctr` (값 / 100)
3. **시간 파싱**: `mm:ss` 또는 `hh:mm:ss` → 초.
4. **백분율 파싱**: `12.3%` → 0.123. 0–1 스케일로 통일.
5. **결측 셀**: 빈 문자열·`-`·`N/A` → `null`. 0과 명확히 구분.

YouTube Studio CSV는 자주 바뀌므로 컬럼명 매핑이 실패하면 **사용자에게 헤더 확인을 요청**한다 (자동 추측 금지).

## 5. 스크린샷 OCR 가이드 (멀티모달 비전 직접 독해)

스크린샷을 받으면 다음 절차로 추출한다.

1. 이미지 영역별 우선순위:
   - **상단 카드**: 노출·CTR·평균시청시간·시청 시간 합계
   - **유지율 곡선**: 가로축 = 영상 시간, 세로축 = % — 곡선의 0%·25%·50%·75%·100% 지점 % 추정
   - **트래픽 소스 위젯**: 소스명·% 또는 시간
   - **검색 키워드 리스트**: 상위 10개 키워드
2. 추출 시 자기검증:
   - 숫자가 흐릿하면 `null` + `notes`에 "스크린샷 해상도 낮음" 기록
   - 곡선의 정확한 좌표값을 적지 않는다 — 5점 샘플(0/25/50/75/100%)만 기록
3. 한 스크린샷에 여러 위젯이 보이면 위젯별로 객체 분리.

## 6. 검증 규칙

수집된 5지표 중 **3개 이상**이 채워지면 통과. 미달이면 호출자에게 부족 항목을 명시하고 보강 요청.

| 통과 조건 | 동작 |
|----------|------|
| 5지표 중 3개 이상 | 정상 반환 |
| 2개 이하 | 호출자에게 추가 스크린샷/CSV 요청 (어떤 지표가 부족한지 명시) |
| `retention_curve`만 부재 | 통과 (다른 4지표로 진행 가능) — 단 `gaps`에 명시 |
| `impressions` 또는 `ctr` 둘 다 부재 | 노출-도입부 진단 불가 → 사용자에게 보강 요청 |

## 7. 출력 스키마

호출자에게 다음 객체로 반환.

```yaml
video_id: <YouTube videoID 또는 사용자가 부여한 ID>
title: <제목>
published_at: <YYYY-MM-DD HH:MM>
length_seconds: <int>
collected_at: <YYYY-MM-DD>
window: <D+3 | D+7 | D+14 | "all-time" | custom>
metrics:
  impressions: <int | null>
  ctr: <float 0–1 | null>
  avg_view_duration_sec: <int | null>
  retention_curve:
    - { t_pct: 0,   r_pct: 100 }
    - { t_pct: 25,  r_pct: <int|null> }
    - { t_pct: 50,  r_pct: <int|null> }
    - { t_pct: 75,  r_pct: <int|null> }
    - { t_pct: 100, r_pct: <int|null> }
  traffic_sources:
    - { source: "YouTube search", share_pct: <float> }
    - ...
  search_terms: ["...", "..."]   # 있으면
  views: <int | null>
  watch_time_hours: <float | null>
  subscribers_gained: <int | null>
sources:
  - { kind: csv|screenshot, path: <absolute path>, notes: "..." }
gaps:
  - <어떤 지표를 못 가져왔는지 / 이유>
notes:
  - <스크린샷 해상도 등 품질 메모>
```

## 8. 금지

- 결측치를 0이나 평균치로 채우지 않는다 — `null` 유지.
- 한 스크린샷에서 안 보이는 지표를 다른 지표로 추정하지 않는다 (예: 시청시간 합계 / 조회수로 평균시청시간 역산은 허용하되, **명시적으로 `derived: true` 표시**).
- 동영상 ID·제목·업로드 시각을 추측하지 않는다 — 사용자 입력 또는 메타 파일에서만.
