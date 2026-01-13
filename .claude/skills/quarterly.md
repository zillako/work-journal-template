---
name: quarterly
description: 분기 성과 작성. 주간 일지를 분석하여 분기별 성과 기록을 작성할 때 사용
allowed-tools: Read, Grep, Glob, Bash
---

# 분기 성과 작성

분기별 성과 기록을 작성합니다.

## Triggers

다음과 같은 요청에 반응:
- "분기 성과 정리해줘"
- "Q1 성과 작성해줘"
- "이번 분기 회고 작성"
- "3개월 성과 정리"

## Architecture Pattern

**Orchestration Mode**: `Sub-Agent Spawning + Parallel Processing + Memory Synthesis`

대량의 주간 일지(12주)를 효율적으로 처리하기 위해 월별 sub-agent를 병렬 실행합니다.

```
Main Agent (Orchestrator)
  ├─ Sub-Agent 1: month1-analyzer  (병렬)
  ├─ Sub-Agent 2: month2-analyzer  (병렬)
  ├─ Sub-Agent 3: month3-analyzer  (병렬)
  └─ Integration + User Q&A + Document Generation
```

## Sub-Agent Definitions

### Sub-Agent: Month Analyzer (x3)
```yaml
type: general-purpose
task: |
  Analyze weekly journals for {month} (4-5 weeks):
  - Path: journal/YYYY/{MM}/week-*.md
  - Extract:
    ✅ Completed projects/features
    📊 Quantitative metrics (performance, impact)
    🔧 Technical skills used/learned
    🤝 Team contributions (reviews, mentoring)
    🚧 Challenges and solutions
  - Return: structured summary (max 800 tokens per month)

  Format:
  ## {월} 주요 성과
  ### 프로젝트
  - [Project A] Feature X (impact: +30% conversion)
  - [Project B] Refactoring (tech debt: -40%)

  ### 기술 성장
  - React 18 Concurrent Features
  - Vite 5 Performance Optimization

  ### 팀 기여
  - Code reviews: 45건
  - Mentoring: 2명 (주니어 온보딩)

  ### 주요 해결 과제
  - CORS issue → Nginx config fix
```

## Main Agent Orchestration Flow

### Step 1: Determine Quarter
```javascript
// Calculate current quarter
// Q1: 1-3월, Q2: 4-6월, Q3: 7-9월, Q4: 10-12월
const quarter = Math.floor((month - 1) / 3) + 1
const months = getQuarterMonths(quarter)
```

### Step 2: Spawn Month Analyzers (Parallel)
```javascript
// Spawn 3 sub-agents concurrently (one per month)
Task(subagent_type="general-purpose",
     prompt="Analyze month 1 weekly journals...",
     run_in_background=true)

Task(subagent_type="general-purpose",
     prompt="Analyze month 2 weekly journals...",
     run_in_background=true)

Task(subagent_type="general-purpose",
     prompt="Analyze month 3 weekly journals...",
     run_in_background=true)
```

### Step 3: Collect Monthly Summaries
```javascript
// Wait for all sub-agents to complete
// Total context: ~2400 tokens (vs 60-80K before)
month1_summary = TaskOutput(task_id="month1-analyzer")
month2_summary = TaskOutput(task_id="month2-analyzer")
month3_summary = TaskOutput(task_id="month3-analyzer")
```

### Step 4: User Q&A Session
월별 요약을 통합하여 사용자에게 제시 후 추가 정보 수집:

1. **핵심 성과 선정**
   - "이번 분기 가장 중요한 성과 3-5개를 꼽는다면?"
   - "정량적 지표가 있으면 함께 알려주세요"

2. **기술 성장**
   - "새로 배우거나 깊이 있게 다룬 기술이 있나요?"
   - "특히 어려웠던 기술적 도전은?"

3. **팀 기여**
   - "코드리뷰, 멘토링, 지식공유 등 팀 기여 활동은?"
   - "정량적 수치가 있다면? (리뷰 건수, 멘토링 인원 등)"

4. **회고 및 개선**
   - "이번 분기 아쉬웠던 점은?"
   - "다음 분기 개선하고 싶은 점은?"

### Step 5: Generate Document
템플릿(`templates/quarterly-achievement.md`) 형식으로 문서 생성:
- Path: `achievements/YYYY/QN.md`
- Format: STAR 기법 적용 (Situation, Task, Action, Result)
- 정량화: 모든 성과에 수치 포함
- 비즈니스 임팩트: 기술적 성과를 비즈니스 가치로 연결

## Performance Metrics

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Context Usage | 60-80K tokens | 10-15K tokens | 82% ↓ |
| Execution Time | Sequential | Parallel (3x) | 200% ↑ |
| Token Cost | Very High | Low | 80% ↓ |
| Analysis Depth | Shallow | Deep | Better |

## 성과 작성 가이드

- **STAR 기법 활용**: Situation, Task, Action, Result
- **정량화 필수**: 모든 성과에 수치 포함 권장
- **비즈니스 임팩트**: 기술적 성과를 비즈니스 가치로 연결
- **키워드 태그**: 주요 기술/역량 키워드 포함

## File Structure

```
achievements/YYYY/QN.md
```

## Usage Example

```
User: Q1 성과 작성해줘

→ Step 1: 3개 month-analyzer sub-agents 병렬 실행
  - month1-analyzer: 1월 weekly 분석 (4-5주)
  - month2-analyzer: 2월 weekly 분석 (4주)
  - month3-analyzer: 3월 weekly 분석 (4-5주)

→ Step 2: 월별 요약 통합 (~2400 tokens)

→ Step 3: 사용자 Q&A (핵심 성과, 기술, 팀 기여, 회고)

→ Step 4: achievements/2026/Q1.md 생성
```

## Related

- Template: `templates/quarterly-achievement.md`
- Pattern: Plan-Then-Execute + Sub-Agent Spawning + Parallel Processing (agentic-patterns.com)
