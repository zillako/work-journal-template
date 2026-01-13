# MCP 서버 설정 가이드

Work Journal에서 사용하는 MCP(Model Context Protocol) 서버들을 설정하는 가이드입니다.

---

## 필수 MCP 서버

### 1. claude-mem (업무 기록 자동 수집)

**용도**: 오늘 작업한 내역을 claude-mem에서 자동으로 수집

**설치 방법**:
```bash
# PM2로 설치 (권장)
npm install -g pm2
npm install -g @anthonychu/claude-mem

# PM2로 실행
pm2 start claude-mem --name claude-mem
pm2 save
pm2 startup  # 시스템 재시작 시 자동 실행
```

**Claude Code 설정**:
`~/.claude/config.json`에 추가:
```json
{
  "mcpServers": {
    "claude-mem": {
      "command": "node",
      "args": ["/usr/local/lib/node_modules/@anthonychu/claude-mem/dist/index.js"],
      "env": {
        "CLAUDE_MEM_DB_PATH": "/Users/[username]/.claude/claude-mem.db"
      }
    }
  }
}
```

**확인**:
```bash
# PM2 상태 확인
pm2 status

# claude-mem이 실행 중이어야 함
# claude 실행 시 claude-mem MCP 서버 연결 확인
```

---

### 2. atlassian (JIRA 연동)

**용도**: JIRA 티켓 자동 조회 및 연동

**설치 방법**:
```bash
npm install -g @modelcontextprotocol/server-atlassian
```

**Claude Code 설정**:
`~/.claude/config.json`에 추가:
```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-atlassian"],
      "env": {
        "ATLASSIAN_API_TOKEN": "your-api-token",
        "ATLASSIAN_USER_EMAIL": "your-email@company.com",
        "ATLASSIAN_SITE_URL": "https://your-site.atlassian.net"
      }
    }
  }
}
```

**API Token 생성**:
1. https://id.atlassian.com/manage-profile/security/api-tokens 접속
2. "Create API token" 클릭
3. 토큰 이름 입력 (예: "claude-code-work-journal")
4. 생성된 토큰을 `ATLASSIAN_API_TOKEN`에 입력

---

## 선택 MCP 서버 (필요시)

### 3. google-docs (Google Docs 연동)

**용도**: Weekly Flash Report 등 Google Docs 문서 자동 생성

**설치 방법**:
```bash
npm install -g @modelcontextprotocol/server-google-docs
```

**Claude Code 설정**:
```json
{
  "mcpServers": {
    "google-docs": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-google-docs"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/credentials.json"
      }
    }
  }
}
```

**Google API 인증**:
1. Google Cloud Console에서 OAuth 2.0 클라이언트 ID 생성
2. credentials.json 다운로드
3. 경로를 `GOOGLE_APPLICATION_CREDENTIALS`에 설정

---

## 전체 설정 예시

`~/.claude/config.json` 전체 예시:

```json
{
  "mcpServers": {
    "claude-mem": {
      "command": "node",
      "args": ["/usr/local/lib/node_modules/@anthonychu/claude-mem/dist/index.js"],
      "env": {
        "CLAUDE_MEM_DB_PATH": "/Users/[username]/.claude/claude-mem.db"
      }
    },
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-atlassian"],
      "env": {
        "ATLASSIAN_API_TOKEN": "your-api-token",
        "ATLASSIAN_USER_EMAIL": "your-email@company.com",
        "ATLASSIAN_SITE_URL": "https://your-site.atlassian.net"
      }
    }
  }
}
```

---

## 설정 확인

```bash
# Claude Code 실행
claude

# MCP 서버 연결 확인
# 프롬프트에서 다음 명령 실행:
> /mcp list

# claude-mem, atlassian이 표시되어야 함
```

---

## 문제 해결

### claude-mem이 연결되지 않을 때

```bash
# PM2 상태 확인
pm2 status

# claude-mem 재시작
pm2 restart claude-mem

# 로그 확인
pm2 logs claude-mem
```

### JIRA 연동이 안 될 때

1. API Token이 올바른지 확인
2. Email 주소가 JIRA 계정과 일치하는지 확인
3. Site URL이 정확한지 확인 (https 포함)

### MCP 서버가 인식되지 않을 때

```bash
# Claude Code 재시작
# config.json 파일 경로 확인
cat ~/.claude/config.json

# JSON 문법 오류 확인
npx jsonlint ~/.claude/config.json
```

---

## 참고 링크

- [Claude MCP 공식 문서](https://modelcontextprotocol.io/)
- [claude-mem GitHub](https://github.com/anthonychu/claude-mem)
- [Atlassian API Token 관리](https://id.atlassian.com/manage-profile/security/api-tokens)
