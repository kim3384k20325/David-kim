---
name: youtube-research
description: Use when researching YouTube channels and videos for benchmark/competitor analysis or for evidence-gathering on a niche. Provides query patterns, per-channel data extraction schema, source priorities, and the structured output format expected downstream.
---

# YouTube Research

웹 검색·페이지 패칭으로 특정 장르의 유튜브 채널/영상 정보를 수집해, 다운스트림(competitor-researcher, keyword-strategist 등)이 바로 쓸 수 있는 구조화 데이터로 반환한다.

## 1. 언제 사용하는가

- 신규 채널 셋업 시 벤치마크 채널 발굴 (Mode A Step A1)
- 월간 벤치마크 갱신
- 키워드 리서치에서 실제 채널·영상 사례를 인용해야 할 때
- 진단 단계에서 동장르 영상 메트릭 비교가 필요할 때

## 2. 검색 쿼리 패턴

장르·언어를 변수로 두고 다음 쿼리들을 조합한다.

```
"<niche>" youtube channel <language>
best <niche> youtube channels <year>
top <niche> youtubers site:socialblade.com
"<niche>" 인기 유튜브 채널        # 한국어 시장
<niche> playlist site:youtube.com
"<seed-keyword>" site:youtube.com
```

장르별 시드 예:
- 수면음악: `sleep music`, `Celtic sleep music`, `ambient sleep`, `수면 음악`, `잠잘 때 듣는 음악`
- 음악 플레이리스트: `lofi playlist`, `study playlist`, `한국 발라드 플레이리스트`
- 나레이션형 역사: `한국사 미스터리`, `숨은 역사 이야기`, `history mystery youtube`

## 3. 우선 출처

1. **YouTube 직접** (`youtube.com/@<handle>`) — 가장 신뢰 가능. 채널 정보·최근 업로드·조회수·길이 확보.
2. **Social Blade** (`socialblade.com/youtube/c/<id>`) — 구독자 추정, 업로드 빈도, 성장 추이.
3. **NoxInfluencer** — 보조용. 이름·구독자 추정.
4. **검색엔진 결과 페이지** — 큐레이션 기사("best X channels 2025") 단서 수집.

## 4. 채널당 추출 데이터 (스키마)

각 채널에 대해 다음 필드를 채운다. 확보 불가 시 `null` (추측 금지).

```yaml
- channel_name: <표시 이름>
  url: https://www.youtube.com/@<handle>
  subscribers_estimate: <예: 50000 / "100K">
  language: <ko | en | multi>
  niche_fit: <high | medium | low>           # 검색 쿼리 대비 적합도
  posting_cadence: <예: "주 2회" | null>
  total_videos: <정수 | null>
  top_videos:
    - title: <제목 원문>
      url: https://www.youtube.com/watch?v=...
      views: <정수>
      length_seconds: <정수 | null>
      upload_date: <YYYY-MM-DD | null>
      thumbnail_url: <url | null>
  notes: <특이사항 1~2줄>
```

채널당 `top_videos` 는 **상위 조회수 3건**을 우선, 보조로 최근 3건을 포함한다.

## 5. 패턴 추출에 필요한 이차 데이터

`competitor-researcher` 가 패턴 추출에 사용하므로 다음을 함께 캡처한다.

- 제목 텍스트 (전체) — 공식·이모지·숫자 패턴 추출용
- 영상 길이 분포 — 짧음(<10분) / 중간(10–60분) / 김(>60분) 분류
- 업로드 시간대(가능 시) — 요일·시각
- 설명란 첫 100자 — 후킹 패턴
- 태그(보이는 것) / 해시태그
- 썸네일 URL — 시각 요소 분석용

## 6. 산출 형식

호출자에게 다음 구조로 반환한다.

```yaml
query_seed: <사용한 시드 키워드>
language_filter: <ko | en | multi>
sub_range: 10000–1000000
collected_at: YYYY-MM-DD
channels:
  - <위 §4 스키마>
  - ...
sources_consulted:
  - <url>
  - <url>
gaps:
  - <확보하지 못한 필드 / 채널 메모>
```

## 7. 품질 기준 (자기검증)

반환 전 다음을 확인한다.

- 채널 ≥ 3개, 각 채널 구독자 추정 ≥ 10,000
- 채널당 `top_videos` ≥ 3
- `niche_fit = high` 채널 ≥ 2
- 각 채널의 `url` 이 `https://www.youtube.com/` 로 시작
- 추측·환각 없음 (확보 못한 값은 `null`, `gaps` 에 기재)

기준 미달 시 검색 쿼리를 확장해 1~2회 재시도. 그래도 미달이면 호출자에게 `gaps`로 보고하고 결정을 위임.

## 8. 금지

- 데이터 추측·합성 금지. 보지 못한 수치는 절대 적지 않는다.
- 저작권 보호 콘텐츠(가사·대본 전문 등) 복사 금지.
- 채널·영상 URL은 실제로 본 페이지의 URL만 기록 (생성 금지).
