# Daily Work Log Skill

일일 업무 기록을 자연어로 요청할 때 자동 활성화됩니다.

## Triggers

다음과 같은 요청에 반응:
- "오늘 한 일 기록해줘"
- "오늘 작업 내용 저장"
- "일일 메모 작성"
- "데일리 로그 추가"
- "오늘 뭐 했는지 적어줘"
- "오늘 작업 정리해줘" (자동 수집 모드)

## Architecture Pattern

**Orchestration Mode**: `Direct Execution (현재) + Optional Sub-Agent (미래)`

일일 기록은 이미 효율적이므로 현재 direct execution 유지. 필요 시 sub-agent 선택 가능.

```
Current: Main Agent → claude-mem 직접 검색 → 파일 작성
Optional: Main Agent → Sub-Agent (claude-mem collector) → 파일 작성
```

**Performance**: Context usage ~2-3K tokens (이미 최적화됨)

## Behavior

### Mode 1: 수동 입력
사용자가 직접 내용을 말하면 그대로 기록

### Mode 2: 자동 수집 (claude-mem 연동)
"오늘 작업 정리해줘" 등으로 요청 시:

1. **claude-mem에서 오늘 observations 수집**
   ```
   mcp__plugin_claude-mem_claude-mem-search__search
   - dateStart: 오늘 날짜
   - dateEnd: 오늘 날짜
   - limit: 50
   ```

2. **프로젝트별 그룹핑**
   - observations의 project 필드로 분류
   - 각 프로젝트별 작업 내용 정리

3. **일자별 파일에 기록** (`journal/YYYY/MM/YYYY-MM-DD.md`)
   ```markdown
   # 2026-01-03 (금)

   ## 오늘 한 일

   ### [project-name]
   - TASK-1234 상품 상세 성능 개선
   - 이미지 레이지로딩 구현

   ### [work-journal]
   - daily skill claude-mem 연동 추가

   ---

   ## 메모
   - claude-mem에서 자동 수집됨
   ```

## Auto-Collection Process

```
┌─────────────────────────────────────┐
│ 1. claude-mem search (오늘 날짜)    │
│    - type: feature, bugfix, change  │
│    - 모든 프로젝트 대상             │
└──────────────┬──────────────────────┘
               ▼
┌─────────────────────────────────────┐
│ 2. 프로젝트별 그룹핑                │
│    - project 필드로 분류            │
│    - title + subtitle 추출          │
└──────────────┬──────────────────────┘
               ▼
┌─────────────────────────────────────┐
│ 3. YYYY-MM-DD.md 업데이트           │
│    - 일자별 파일에 append/create    │
│    - 프로젝트별 서브섹션            │
└─────────────────────────────────────┘
```

## File Structure

```
journal/YYYY/MM/YYYY-MM-DD.md

예시:
- journal/2026/01/2026-01-02.md
- journal/2026/01/2026-01-03.md
```

> **Note**: 일자별로 파일을 분리하여 관리합니다. 월별로 하나의 파일에 기록하면 파일이 너무 커지므로, 일자별 파일로 분리하여 검색/수정이 용이하도록 합니다.

## Usage Examples

```
# 수동 입력
User: 오늘 API 리팩토링 완료했어
→ 입력 내용 그대로 기록

# 자동 수집
User: 오늘 작업 정리해줘
→ claude-mem에서 오늘 observations 수집
→ 프로젝트별로 그룹핑해서 기록
```

## Related

- Command: `/wj:daily`
- Template: `templates/daily.md`
- Hook: `.claude/hooks/session-end.sh`
