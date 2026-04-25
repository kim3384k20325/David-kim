# 에이전트 빌드 지침 — YouTube 채널 성장 진단·전략 에이전트

생성일: 2026-04-25
대상: 본 에이전트를 새 저장소·새 환경에서 재구축하려는 운영자

본 문서는 **빌드 순서와 검증 기준**만 담는다. 각 단계 산출 파일의 본문은 본 저장소를 1차 참조한다.

---

## 1. 사전 요구사항

- Claude Code (또는 `.claude/{agents,skills}/<name>.md` 컨벤션 + YAML frontmatter 호환 환경)
- git 저장소 1개 (브랜치 명명 자유)
- 운영자 ≥ 1인, 진단 대상 채널 ≥ 1개
- 사용자가 YouTube Studio 분석 화면 캡처 권한 보유 (스크린샷 업로드용)

본 에이전트는 **외부 도구를 직접 호출하지 않는다** — Suno·Midjourney·CapCut 호출 / YouTube API / 업로드 자동화는 모두 사용자가 수동 실행. 에이전트는 텍스트 자산만 산출.

---

## 2. 최종 디렉토리 레이아웃

```
/<repo-root>/
├── CLAUDE.md                           # 메인 오케스트레이터 운영 지침
├── /docs/
│   ├── channel-profile-template.md     # 채널 프로파일 스키마
│   └── setup-guide.md                  # 본 문서
├── /channels/
│   ├── <channel-id-1>/profile.md
│   ├── <channel-id-2>/profile.md
│   └── ...
├── /.claude/
│   ├── /agents/
│   │   ├── competitor-researcher.md
│   │   ├── keyword-strategist.md
│   │   ├── performance-analyst.md
│   │   └── content-creator.md
│   └── /skills/
│       ├── youtube-research/SKILL.md
│       ├── keyword-mining/SKILL.md
│       ├── analytics-parser/SKILL.md
│       ├── script-drafter/SKILL.md
│       └── thumbnail-prompter/SKILL.md
└── /output/<channel-id>/               # 채널별 산출물 — Mode A·B 결과
    ├── /benchmarks/                    # YYYY-MM-DD_benchmark.md
    ├── /keywords/                      # YYYY-MM-DD_keywords.md
    ├── /diagnostics/
    │   ├── YYYY-MM-DD_<videoID>.md     # 진단 리포트
    │   └── /<videoID>/                 # 영상별 입력 자산
    │       ├── meta.json               # B1 영상 메타
    │       └── /screenshots/           # D+3/7/14_*.png
    ├── /strategy/                      # YYYY-MM-DD_next-upload.md
    └── /content/                       # YYYY-MM-DD_{script,thumbnail}.md
```

`<channel-id>` 는 폴더명과 일치하는 슬러그. 채널 간 파일 절대 섞지 않는다 (CLAUDE.md §6).

---

## 3. 빌드 순서 (8단계)

각 단계 산출 파일은 다음 단계의 입력이 되므로 순서를 지킨다.

### Step 1 — 프로파일 템플릿 + 첫 채널 프로파일

| 만들 파일 | 핵심 책임 |
|----------|----------|
| `/docs/channel-profile-template.md` | 10개 섹션 스키마 (식별·장르·타겟·정책·KPI·대본깊이·도구·운영·스냅샷·메모) + KPI 선택지 명시 |
| `/channels/<id>/profile.md` | 가장 활성된 채널 1개부터. 미상 필드는 `<...>` placeholder (lazy elicitation) |

**필수 채워야 하는 필드** (Step 0 라우팅용 5개 키): `channel-id`, `장르`, `KPI 우선순위 1~3`, `대본 깊이`, `제약/금기`
**검증**: 위 5개 키가 schema 검증 통과 → 다음 단계 가능

### Step 2 — CLAUDE.md 메인 오케스트레이터

| 만들 파일 | 핵심 책임 |
|----------|----------|
| `/CLAUDE.md` | 9개 섹션 — §1 정체성 / §2 Step 0 라우팅 + lazy elicitation / §3 스키마 참조 / §4 모드 라우팅 표 + B2 스크린샷 트리거 / §5 서브에이전트 호출 규약(파일 경로만) / §6 산출 저장 규칙 / §6.1 영상 작업 디렉토리 + meta.json + 윈도 추적 / §7 검토 게이트 / §8 에스컬레이션 / §9 구현 상태 |

**검증**: 메인이 단독으로 Mode A·B 라우팅 결정 가능 (서브에이전트 부재 상태에서도 다음 단계 안내 가능)

### Step 3 — competitor-researcher + youtube-research

| 만들 파일 | 핵심 책임 |
|----------|----------|
| `/.claude/skills/youtube-research/SKILL.md` | 검색 쿼리 패턴 6종 / 채널 스키마 (subs·videos·top_videos) / 출처 우선순위 (YouTube > SocialBlade > vidIQ > SERP) / 자기검증 (≥3채널, ≥10K subs) |
| `/.claude/agents/competitor-researcher.md` | A1·A2 워크플로우 / 6차원 패턴 추출 (제목/썸네일/길이/시간대/설명/태그) / KPI 매핑 / 자기검증 6항목 |

**검증**: 첫 채널의 벤치마크 리포트 1건 산출. ≥3개 채널, 6개 차원 모두 다룸 (없으면 "공통 패턴 없음" 명시), KPI 인용 시사점 ≥3.

### Step 4 — keyword-strategist + keyword-mining

| 만들 파일 | 핵심 책임 |
|----------|----------|
| `/.claude/skills/keyword-mining/SKILL.md` | 6슬롯 확장 매트릭스 (분량·용도·무드·악기·환경·시간대) / 무-API 경쟁강도 휴리스틱 (1M+/10K~1M/<100K → 상/중/하) / 시장 라벨 (ko/en/multi) |
| `/.claude/agents/keyword-strategist.md` | 벤치마크 §2 빈출 토큰을 1차 시드로 재활용 / Top 10 추천에 KPI 정합 명시 |

**검증**: 키워드 ≥20, 시장별 ≥5씩, 인텐트 군집 ≥3, `상` 라벨 < 50%, 추측 데이터 0.

### Step 5 — performance-analyst + analytics-parser

| 만들 파일 | 핵심 책임 |
|----------|----------|
| `/.claude/skills/analytics-parser/SKILL.md` | 5지표 표준 스키마 (impressions/ctr/avg_view_duration/retention_curve/traffic_sources) / CSV 헤더 변종 매핑 (한·영) / 스크린샷 5점 샘플링 / ≥3 of 5 검증 룰 / 결측 보존 (imputation 금지) |
| `/.claude/agents/performance-analyst.md` | Mode A4 baseline + Mode B2·B3 per-video 양방향 / 벤치마크 6차원 적합도 / 병목 1개 + 근거 ≥2 / 가설 3개 (모두 인용 강제) |

**검증**: 파일 구조만 빌드되면 단계 통과 (실 진단은 사용자 데이터 도착 시).

### Step 6 — content-creator + script-drafter + thumbnail-prompter

| 만들 파일 | 핵심 책임 |
|----------|----------|
| `/.claude/skills/script-drafter/SKILL.md` | 3 모드 템플릿 (full-narration/minimal/playlist) / 후킹 공식 4종 / 설명란 구조 / 안티-표절 가드 (≥7어 연속 일치 금지, 곡명·아티스트명 차용 금지) |
| `/.claude/skills/thumbnail-prompter/SKILL.md` | Midjourney v6/v7 문법 (--ar 16:9 --v 7 --no people 등) / 장르 시각 코드 (sleep/playlist/narration) / A/B 변주 1축 격리 / 안티-파생 가드 (살아있는 아티스트·IP 직접 호출 금지) |
| `/.claude/agents/content-creator.md` | 승인된 strategy 파일 전제 / 두 스킬 위임 / 텍스트 자산만 산출 (이미지·오디오 생성 X) |

**검증**: 파일 구조만 빌드되면 단계 통과 (실 산출은 B4 승인 후 트리거).

### Step 7 — Mode B 사이클 dry-run

메인이 직접 다음 흐름을 1회 통과:
1. 메인이 B4 전략 합성 (벤치마크 §·키워드 § 명시 인용) → `/output/<id>/strategy/YYYY-MM-DD_next-upload.md`
2. 사용자 검토 게이트 (CLAUDE.md §7-1)
3. 사용자 승인 + 조합 지정
4. content-creator 디스패치 (B5) → `script.md` + `thumbnail.md`

**검증**: 산출물이 전략·벤치마크·키워드를 본문에서 명시적으로 인용 (단순 헤더 참조 X).

### Step 8 — 추가 채널 프로파일

다른 장르 채널의 `profile.md` 추가. 동일 파이프라인이 KPI 우선순위·대본 깊이 변경만으로 작동하는지 검증.

**검증**: 새 채널의 `<channel-id>` 만 다를 뿐 동일 명령으로 Mode A 진입 가능.

---

## 4. 빌드 후 운영 트리거

| 사용자 의도 신호 | 메인 동작 |
|----------------|----------|
| "<id> 채널 셋업" / 신규 channel-id 언급 | 세팅 위저드 → profile.md 생성 → Mode A 진입 |
| "벤치마크/키워드 갱신" | Mode A 월간 재실행 (기존 산출은 `_v2.md`) |
| "방금 업로드했어 (영상 정보)" | B1 — meta.json 5필드 수집 + D+3/7/14 윈도 자동 등록 |
| 도달했지만 미진단 윈도 존재 | 새 세션 첫 응답에서 1순위 알림 + 4종 스크린샷 요청 |
| "성과 분석" / 스크린샷 업로드 | B2 → analytics-parser → performance-analyst → 진단 리포트 |
| "다음 업로드 뭐 만들지" | B4 메인 합성 → 검토 게이트 §7-1 |
| (B4 승인 후) "진행" / "이걸로 가" | B5 — content-creator 디스패치 → 검토 게이트 §7-2 |

---

## 5. 확장 시나리오

### 5.1 새 채널 추가
1. `/channels/<new-id>/profile.md` (필수 5필드)
2. `/output/<new-id>/{benchmarks,keywords,diagnostics,strategy,content}/` 골격 생성
3. Mode A 1회 → 베이스라인 보유 → Mode B 사이클 진입

### 5.2 새 장르 도입 (현재 3장르 외)
- `script-drafter` SKILL.md §2 에 새 모드 추가 (예: `vlog`, `tutorial`)
- `thumbnail-prompter` SKILL.md §3 에 새 장르 시각 코드 추가
- 다른 파이프라인 변경 없음

### 5.3 새 KPI 도입
- 프로파일 템플릿 §5 선택지 갱신
- `performance-analyst` AGENT.md §2 Step 4 병목 신호 표 갱신

---

## 6. 자주 만나는 갭과 처리

| 갭 | 처리 |
|----|------|
| Social Blade / vidIQ 페이지 403 차단 | 검색 메타로 대체, "추정" 라벨, `gaps` 명시 |
| 한국 시장 동장르 채널 표본 부족 | 영문권 인접 장르로 확장, `?` 라벨 유지 |
| YouTube 자동완성 직접 관찰 불가 | 운영자가 검색창에서 보강 |
| 영상 표본 작아 자기이전 비교 불가 (초반 3~4편) | 벤치마크 패턴 적합도 중심 진단 |
| 스크린샷에서 5지표 중 <3 확보 | 사용자에게 부족 항목 명시하여 추가 요청 (CLAUDE.md §8) |
| 대본/썸네일이 특정 작품과 유사 | 즉시 멈추고 사용자에게 경고 (자동 우회 금지) |

---

## 7. 빌드 검증 체크리스트

빌드 완료 시 다음을 확인:

- [ ] `CLAUDE.md` 9개 섹션 모두 존재 (§6.1 포함)
- [ ] 4개 서브에이전트 — 모두 YAML frontmatter (`name`·`description`·`tools`·`model`)
- [ ] 5개 스킬 — `.claude/skills/<name>/SKILL.md` + YAML frontmatter (`name`·`description`)
- [ ] 채널 프로파일 ≥ 1개 (필수 5필드 채워짐, placeholder 무관)
- [ ] `/output/<id>/` 5개 서브폴더(`benchmarks`·`keywords`·`diagnostics`·`strategy`·`content`) 골격
- [ ] 첫 채널 Mode A 1회 통과 (벤치마크·키워드 리포트 1건씩 산출 + 자기검증 통과)
- [ ] B5 dry-run 통과 (전략→대본→썸네일 상호 인용 확인)
- [ ] CLAUDE.md §9 구현 상태 표가 실제 저장소 상태와 일치

위 8개 모두 통과하면 정규 운영 진입 가능.
