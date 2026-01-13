# Weekly Journal Skill

주간 업무 일지 작성을 자연어로 요청할 때 자동 활성화됩니다.

## Triggers

다음과 같은 요청에 반응:
- "주간 보고서 작성해줘"
- "이번 주 업무 일지 써줘"
- "주간 회고 정리"
- "금요일이니까 주간 리뷰 하자"
- "한 주 정리해줘"

## Architecture Pattern

**Orchestration Mode**: `Sub-Agent Spawning + Memory Synthesis`

Context 효율성을 위해 메인 에이전트는 조율만 하고, 데이터 수집은 전문화된 sub-agent에 위임합니다.

```
Main Agent (Orchestrator)
  ├─ Sub-Agent 1: daily-collector   (병렬)
  ├─ Sub-Agent 2: git-analyzer      (병렬)
  ├─ Sub-Agent 3: jira-analyzer     (병렬)
  └─ Integration & User Interaction
```

## Sub-Agent Definitions

### Sub-Agent 1: Daily Collector
```yaml
type: general-purpose
task: |
  Read daily journal files from this week:
  - Path: journal/YYYY/MM/YYYY-MM-DD.md (7 files)
  - Extract: completed tasks, blockers, key decisions, notes
  - Group by: project/category
  - Return: structured summary in markdown (max 500 tokens)

  Format:
  ### [Project Name]
  - Task 1
  - Task 2

  ### 이슈/블로커
  - Issue 1
```

### Sub-Agent 2: Git Analyzer
```yaml
type: general-purpose
task: |
  Analyze git commits from this week:
  - Run: git log --since="7 days ago" --oneline --all
  - Group by: project (extract from commit messages)
  - Count: commits per project
  - Return: summary table (max 300 tokens)

  Format:
  | 프로젝트 | 커밋 수 | 주요 작업 |
  |---------|---------|----------|
  | proj-a  | 15      | Feature X |
```

### Sub-Agent 3: Jira Analyzer
```yaml
type: general-purpose
task: |
  Fetch Jira tickets updated this week:
  - Use: mcp__atlassian__searchJiraIssuesUsingJql
  - JQL: updated >= -7d
  - Categorize: Done, In Progress, Blocked
  - Return: summary list (max 400 tokens)

  Format:
  ✅ Done: GPRD-1234, GPRD-5678
  🔄 In Progress: GPRD-9012
  🚧 Blocked: GPRD-3456 (reason)
```

## Main Agent Orchestration Flow

### Step 1: Spawn Sub-Agents (Parallel)
```javascript
// Spawn 3 sub-agents concurrently
Task(subagent_type="general-purpose",
     prompt="Daily Collector task...",
     run_in_background=true)

Task(subagent_type="general-purpose",
     prompt="Git Analyzer task...",
     run_in_background=true)

Task(subagent_type="general-purpose",
     prompt="Jira Analyzer task...",
     run_in_background=true)
```

### Step 2: Collect Summaries
```javascript
// Wait for all sub-agents to complete
// Total context: ~1200 tokens (vs 15-20K before)
daily_summary = TaskOutput(task_id="daily-collector")
git_summary = TaskOutput(task_id="git-analyzer")
jira_summary = TaskOutput(task_id="jira-analyzer")
```

### Step 3: User Interaction
- 3개 요약 통합하여 사용자에게 제시
- 누락 작업 추가 요청
- 임팩트/성과 수치 보완
- 다음 주 계획 질문

### Step 4: Generate Document
`journal/YYYY/MM/week-WW.md` 파일 생성

## Performance Metrics

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Context Usage | 15-20K tokens | 5-7K tokens | 65% ↓ |
| Execution Time | Sequential | Parallel (3x) | 200% ↑ |
| Token Cost | High | Low | 70% ↓ |

## File Structure

```
journal/YYYY/MM/week-WW.md
```

## Usage Examples

```
User: 주간 보고서 써줘
→ 3개 sub-agent 병렬 실행
→ 요약 통합 (1200 tokens)
→ 사용자 확인 후 week-01.md 생성

User: 이번 주 뭐 했는지 정리해줘
→ Sub-agent 요약 결과만 제시
→ 빠르고 효율적
```

## Related

- Command: `/wj:weekly`
- Template: `templates/weekly-journal.md`
- Pattern: Plan-Then-Execute + Sub-Agent Spawning (agentic-patterns.com)
