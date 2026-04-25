# 베이스라인 분석 입력 데이터 — Nature Trail (sleep-music)

생성일: 2026-04-25
출처: 사용자 제공 YouTube Studio 스크린샷 5종 (개요·콘텐츠·시청자층 + 인기 콘텐츠 + 도달범위 부분)
관측 윈도: 2026-03-28 ~ 2026-04-24 (지난 28일)
용도: performance-analyst Mode A4 baseline 진단 입력

---

## 1. 채널 종합 지표

```yaml
channel_id: sleep-music
channel_name: Nature Trail
channel_url: https://www.youtube.com/@NatureTrail-r3u
channel_uc_id: UCp_HcmGOTTFXWCDyOclHnfw
window: 28d (2026-03-28 ~ 2026-04-24)

aggregates:
  views: 115        # 콘텐츠 탭 28일 / 개요 탭 116과 ±1 수치 혼재
  watch_time_hours: 28.7
  subscribers_gained: 5
  total_subscribers: 5
  monthly_viewers: 59

impressions:
  total: 1500       # "1.5천"
  source_breakdown:
    youtube_suggested_share_pct: 91.7   # "YouTube에서 추천한 내 콘텐츠"
    other_share_pct: 8.3
  ctr_pct: 3.3
  views_from_impressions: 49
  avg_view_duration_from_impressions_sec: 369   # 6:09
  watch_hours_from_impressions: 5.03

avg_view_duration_channel_sec: 760   # 12:40 — 채널 종합 (직접 유입 포함)
avg_view_duration_impression_only_sec: 369   # 6:09

traffic_sources:
  - { source: "추천 동영상",       share_pct: 44.4 }
  - { source: "탐색 기능",         share_pct: 20.9 }
  - { source: "직접 입력 또는 알 수 없음", share_pct: 18.3 }
  - { source: "채널 페이지",       share_pct: 7.8 }
  - { source: "외부",              share_pct: 5.2 }
  - { source: "기타",              share_pct: 3.5 }
  - { source: "YouTube 검색",      share_pct: 0.0, note: "탭 비활성 — 검색 트래픽 사실상 0" }

search_terms: []   # 노출 데이터 없음 → 추정 불가

audience_composition:
  new_share_pct: 100.0
  returning_share_pct: "<0.1"
  fixed_share_pct: "<0.1"
  notes: "월간 시청자 59명 모두 신규 (반복 시청 루프 미형성)"

devices:
  - { type: "휴대전화", share_pct: 43.8 }
  - { type: "TV",       share_pct: 20.0 }
  - { type: "컴퓨터",   share_pct: 19.2 }
  - { type: "태블릿",   share_pct: 1.6 }
  - { type: "기타",     share_pct: 15.5 }

demographics:
  age: insufficient_data
  gender: insufficient_data

real_time:
  subscribers: 5
  views_48h: 10
```

---

## 2. 영상별 지표 (4건)

```yaml
videos:
  - video_id: celtic-beats-sleep-focus
    title: "Celtic beats sleep focus | 心が落ち着く癒やしの秘密 | 불안함이 사라지는 마법 같은 치유"
    uploaded_at: 2026-04-10
    length_seconds: 4738   # 1:18:58
    views_28d: 73          # 인기 콘텐츠 1위
    views_total: 73
    avg_view_duration_sec: 645   # 10:45
    retention_pct_of_length: 13.6
    impressions: null            # 영상별 노출 미확보
    ctr_pct: null
    retention_curve: null        # 영상별 곡선 미확보 (B2 정밀 진단 시 캡처 필요)

  - video_id: rain-water-drops-3h
    title: "Rain & Water Drops — Deep Sleep Music | Healing Sounds for Deep Sleep & Relaxation"
    uploaded_at: 2026-04-15
    length_seconds: 11074   # 3:04:34
    views_28d: 18
    views_total: 19         # 채널 페이지 (수치 ±1)
    avg_view_duration_sec: 183   # 3:03
    retention_pct_of_length: 1.7
    impressions: null
    ctr_pct: null
    retention_curve: null

  - video_id: korean-temple-bell-2h
    title: "Korean Temple Bell | Zen Meditation Music | 2 Hours"
    uploaded_at: 2026-04-23
    length_seconds: 8292   # 2:18:12
    views_28d: 14
    views_total: 16        # 채널 페이지 시점차
    avg_view_duration_sec: 1235   # 20:35 (인기 콘텐츠 표) / 11:14 (최신 콘텐츠 카드 — 처음 2일 12시간)
    retention_pct_of_length: 14.9
    impressions: null
    ctr_pct: 2.7   # 최신 콘텐츠 카드에서만 확보 — 채널 평균 3.3% 보다 낮음
    retention_curve: null
    notes: "최신 영상 (D+2). 노출 표본 작아 CTR 변동 가능"

  - video_id: mountain-forest-sounds-3h
    title: "Mountain Forest Sounds 3 Hours | Sleep & Meditation Music"
    uploaded_at: 2026-04-20
    length_seconds: 12025   # 3:20:25
    views_28d: 4
    views_total: 4
    avg_view_duration_sec: 161   # 2:41
    retention_pct_of_length: 1.4
    impressions: null
    ctr_pct: null
    retention_curve: null
```

---

## 3. 데이터 갭 (analytics-parser §6 검증)

5지표 중 4개 확보, 1개 갭:

| 지표 | 상태 |
|------|------|
| impressions | ✓ 채널 합계 1,500. 영상별 갭 |
| ctr | ✓ 채널 평균 3.3%. 영상별: Korean Temple만 2.7% 확보 |
| avg_view_duration_sec | ✓ 영상 4건 모두 확보 |
| retention_curve | ❌ 영상별 곡선 미확보 — B2 정밀 진단 시 추가 캡처 필요 |
| traffic_sources | ✓ 6개 카테고리 분포 + 검색 0% 확정 |

**임계 ≥3 통과** → A4 베이스라인 진단 가능.

추가 보조 신호 확보:
- 노출의 추천 알고리즘 의존도 (91.7%)
- 채널 평균 vs 노출 유입 평균 시청 시간 격차 (760s vs 369s)
- 시청자 구성 (100% 신규)
- 기기 분포 (모바일 우세 + TV 20%)

## 4. 수치 일관성 메모

- 조회수: 콘텐츠 탭 115 vs 개요 탭 116 (±1 수치 정상 — 캡처 시점 차이)
- Korean Temple 평균 시청 시간: 인기 콘텐츠 표 20:35 vs 최신 콘텐츠 카드 11:14 — 후자는 "처음 2일 12시간" 한정 윈도이므로 다름
- 영상별 조회수: 인기 콘텐츠 표(28일 윈도) vs 채널 페이지(전체 누적) — Rain 18 vs 19, Korean Temple 14 vs 16 차이
