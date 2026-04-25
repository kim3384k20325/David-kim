---
name: keyword-mining
description: Use when generating, expanding, and labeling YouTube longtail keywords for a channel niche. Provides seed-source priorities, longtail expansion patterns, market separation rules, and a competition-strength heuristic that does NOT require an API. Output schema matches what keyword-strategist consumes.
---

# Keyword Mining

YouTube API 없이 검색·자동완성·벤치마크 패턴을 활용해 롱테일 키워드를 생성·라벨링한다.

## 1. 언제 사용하는가

- Mode A Step A3 — 신규 채널 키워드 랜드스케이프 작성
- 월간 갱신 시 키워드 세트 보강
- 다음 업로드 전략 수립 시 후보 제목 키워드 풀 도출

## 2. 시드 소스 우선순위

높은 순서대로:

1. **벤치마크 리포트의 빈출 단어** — `/output/<channel-id>/benchmarks/*.md` §2.1·§2.5 에서 추출. 가장 신뢰성 높은 시드 (이미 유사 채널이 사용 중).
2. **프로파일의 장르·세부 카테고리** — `/channels/<id>/profile.md` §2.
3. **YouTube 검색 자동완성** — 시드 단어를 YouTube 검색창에 넣었을 때의 추천. WebSearch 결과로 간접 관찰.
4. **상위 영상 제목 패턴** — `<seed> sleep music` 류 검색 결과 1페이지 제목들의 빈출 토큰.
5. **한국어/영어 등가 매핑** — 영문 시드 → 한국어 유사어, 역도 동일.

## 3. 롱테일 확장 패턴

각 시드를 다음 6개 슬롯과 조합해 3단어 이상 키워드를 생성한다.

| 슬롯 | 예 (영) | 예 (한) |
|------|--------|---------|
| 분량 | `1 hour`, `8 hours`, `10 hours` | `1시간`, `3시간`, `10시간` |
| 용도 | `for sleep`, `for studying`, `for meditation` | `수면`, `공부`, `명상` |
| 무드 | `dark`, `calm`, `mystical`, `peaceful` | `잔잔한`, `깊은`, `신비로운` |
| 악기 | `piano`, `harp`, `flute`, `strings` | `피아노`, `하프` |
| 환경 | `forest`, `rain`, `ocean`, `cabin`, `castle` | `숲`, `비`, `바다` |
| 시간대 | `night`, `morning`, `bedtime` | `밤에`, `잠잘때`, `새벽` |

조합 규칙:
- 최종 키워드는 **3단어 이상**이어야 한다 (롱테일 정의).
- 슬롯을 너무 많이 결합하면(4개+) 검색량이 0에 수렴 → 시드 + 슬롯 2~3개 권장.
- 한국어 키워드는 띄어쓰기 변종(예: `수면음악` vs `수면 음악`)을 둘 다 후보로 고려.

## 4. 시장 라벨

| 라벨 | 적용 |
|------|------|
| `ko` | 한국어 토큰 포함 |
| `en` | 영어 토큰만 |
| `multi` | 영문 고유명사가 한국어권에서도 그대로 검색됨 (예: "Celtic", "Lo-fi") |

## 5. 경쟁강도 휴리스틱 (API 없이)

다음 신호를 종합해 `상`/`중`/`하` 라벨을 부여한다. **한 키워드당 새 검색을 하지 말고**, 가능한 한 벤치마크 리포트와 기존 검색 결과를 재활용한다.

| 신호 | 라벨 |
|------|------|
| 키워드가 **1M+ 구독 채널**의 타이틀에 빈번히 등장 | `상` |
| 벤치마크(10K~1M) 채널이 사용 중 | `중` |
| 검색 시 상위 결과가 **<100K 채널** 또는 결과 자체가 희박 | `하` |
| YouTube 자동완성 1순위로 노출 | `상` 가산 |
| 한국어 검색 시 결과 페이지에 광고/대형 채널 다수 | `상` |

원칙:
- 신호가 모순될 때 **보수적**으로(높은 쪽) 라벨링.
- 라벨 근거를 1줄로 기재(`evidence` 필드).
- 신호 부족하면 `?` (미평가) — 추측 금지.

## 6. 인텐트 군집

키워드를 다음 군집 중 하나에 매핑한다 (필요 시 추가).

- `sleep` — 취침·수면 유도
- `study` — 공부·집중·작업 BGM
- `meditation` — 명상·마음챙김
- `mood` — 분위기·힐링·스트레스 해소
- `genre` — 장르 자체 (예: "celtic music", "lo-fi") — 인텐트 미지정·범용
- `niche` — 세부 환경/악기 매니아 검색

## 7. 출력 스키마 (호출자에게 반환)

```yaml
collected_at: YYYY-MM-DD
seeds:
  - <seed1>
  - <seed2>
keywords:
  - keyword: "celtic sleep music 1 hour"
    market: en
    competition: 중
    intent_cluster: sleep
    evidence: "Mind & Spirit Relaxation 채널 타이틀 패턴 일치 (벤치마크 §부록 A 채널 3)"
  - keyword: "켈틱 수면음악 1시간"
    market: ko
    competition: ?
    intent_cluster: sleep
    evidence: "한국어 시장 표본 미확보 — 갭"
sources_consulted:
  - <url 또는 "벤치마크 리포트 §2.1">
gaps:
  - <확보 못한 신호>
```

## 8. 품질 기준 (자기검증)

반환 전:

- [ ] 롱테일 키워드 ≥ 20개
- [ ] 각 키워드에 `market`, `competition`, `intent_cluster`, `evidence` 모두 채워짐 (competition 은 `?` 허용)
- [ ] 시장별 분포: `ko` ≥ 5, `en` ≥ 5 (multi 포함)
- [ ] 인텐트 군집 ≥ 3개 다양성
- [ ] `상` 라벨이 전체의 50% 미만 (전부 `상`이면 진입 어려움 → 재검토)
- [ ] 추측 데이터 없음 — 미평가는 `?`, 갭은 `gaps` 섹션

## 9. 금지

- 검색량·CPC·경쟁지수 같은 정량 수치를 적지 않는다 (확보 불가). 라벨 + 근거 1줄로 끝낸다.
- YouTube 검색 결과를 보지 않은 채로 키워드를 만들지 않는다 — 시드는 항상 관찰 데이터에서 파생.
- 저작권 보호 콘텐츠(곡명·아티스트명)를 키워드로 채택하지 않는다.
