# 臺北市立復興高級中學 學分補考查詢系統

學生可透過學號查詢補考科目、日期、時間、地點。管理員透過 Google Sheets 上傳補考名單。

## 技術架構

| 層級 | 技術 |
|------|------|
| 前端 | React 19 + Vite |
| 後端 | FastAPI + SQLModel |
| 資料庫 | PostgreSQL 15 |
| 套件管理 | uv (Python) / npm (Node.js) |
| 容器化 | Docker Compose |
| 反向代理 | Nginx |

## 快速開始

```bash
# 啟動服務
docker compose up -d --build

# 查看服務狀態
docker compose ps

# 查看 Secret Token（若未手動設定）
docker compose logs backend | grep "ADMIN_SECRET_TOKEN"
```

服務位置：
- 前端：http://localhost
- 後端 API：http://localhost:8000

### 設定永久 Token（可選）

```bash
# 1. 產生 Token
python -c "import secrets; print(secrets.token_hex(32))"

# 2. 建立 .env 檔案
echo "ADMIN_SECRET_TOKEN=你產生的Token" > .env

# 3. 重啟服務
docker compose down && docker compose up -d --build
```

### 停止服務

```bash
docker compose down
```

## 功能說明

### 學生端
- 輸入學號查詢補考科目
- 顯示科目、日期、時間、地點
- 學生姓名自動遮罩保護隱私（王○明）
- 響應式設計，支援手機瀏覽
- 深色/淺色模式自動切換

### 管理端
- 透過 Google Apps Script 上傳 Excel 補考名單
- 使用 Secret Token 進行 API 驗證
- 全量覆蓋更新（每次上傳清除舊資料）

## API 文件

### 學生查詢

```
GET /api/exams/{student_id}
```

回應範例：
```json
{
  "student_id": "112001",
  "student_name": "王○明",
  "exams": [
    {
      "subject": "數學",
      "exam_date": "2月6日",
      "exam_time": "08:00-08:50",
      "location": "篤行樓209教室"
    }
  ]
}
```

### 管理員上傳

```
POST /admin/upload
Header: X-Admin-Token: <your_token>
Body: multipart/form-data (file: Excel檔案)
```

### 健康檢查

```
GET /health
```

## 安全機制

1. **無登入頁面**：消除暴力破解攻擊面
2. **Secret Token**：256 位元隨機金鑰
3. **Header 傳輸**：Token 透過 `X-Admin-Token` 傳送
4. **Timing Attack 防護**：使用 `secrets.compare_digest` 比對

## Google Apps Script 設定

1. 開啟你的 Google Sheets 補考名單
2. 點選「擴充功能」→「Apps Script」
3. 貼上 `google-apps-script.js` 的內容
4. 修改 `CONFIG` 設定：
   ```javascript
   const CONFIG = {
     API_URL: "http://你的伺服器IP/admin/upload",
     SECRET_TOKEN: "你的64字元Token"
   };
   ```
5. 儲存後重新整理 Google Sheets
6. 使用選單「📚 補考系統」→「🚀 上傳到資料庫」

## Excel 檔案格式

工作表名稱：**應到考名單 (班級座號序)**

| 欄位 | 必要 | 說明 |
|------|------|------|
| 學號 | ✓ | 學生學號 |
| 補考科目 | ✓ | 科目名稱 |
| 補考日期 | ✓ | 如「2月6日」 |
| 補考時間 | ✓ | 如「08:00-08:50」 |
| 補考教室 | ✓ | 如「篤行樓209教室」 |
| 姓名1 或 姓名 | | 學生姓名（會自動遮罩） |
| 班級 | | 班級名稱 |

## 專案結構

```
├── backend/
│   ├── main.py              # FastAPI 應用程式進入點
│   ├── models.py            # SQLModel 資料模型
│   ├── database.py          # 資料庫連線設定
│   ├── pyproject.toml       # Python 依賴（uv 格式）
│   ├── Dockerfile
│   ├── routers/
│   │   ├── api.py           # 學生查詢 API
│   │   └── admin.py         # 管理員上傳 API
│   ├── services/
│   │   └── parser.py        # Excel 解析服務
│   ├── utils/
│   │   ├── async_utils.py   # 非同步工具
│   │   ├── webpage.py       # 錯誤頁面渲染
│   │   └── upload_authenticate.py  # Token 驗證
│   └── templates/
│       └── error.jinja2     # 錯誤頁面模板
├── frontend/
│   ├── src/
│   │   ├── App.jsx          # React 主元件
│   │   └── index.css        # 樣式（含深色模式）
│   ├── nginx.conf           # Nginx 反向代理設定
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml       # 服務編排
└── google-apps-script.js    # Google Sheets 自動上傳腳本
```

## 資料庫結構

**資料表：makeup_exams**

| 欄位 | 類型 | 來源 |
|------|------|------|
| id | Integer (PK) | 自動產生 |
| student_id | String(20) | Excel：學號 |
| student_name | String(50) | Excel：姓名1 或 姓名 |
| class_name | String(20) | Excel：班級 |
| subject | String(50) | Excel：補考科目 |
| exam_date | String(20) | Excel：補考日期 |
| exam_time | String(50) | Excel：補考時間 |
| location | String(50) | Excel：補考教室 |
| created_at | DateTime | 自動產生 |

## 開發環境

### 後端

```bash
# 安裝 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 安裝依賴
cd backend
uv pip install -r pyproject.toml

# 啟動開發伺服器
cd ..
uvicorn backend.main:app --reload
```

### 前端

```bash
cd frontend
npm install
npm run dev
```

## 配色方案

- 主色調：#00A99D（活潑藍綠）
- 強調色：#FF6F61（珊瑚橘紅）
- 支援深色模式自動切換

## 授權

本專案為臺北市立復興高級中學內部使用。

## LLM Exposure

### AI Skill

#### Clean Code

```md
---
name: clean-code
description: Comprehensive code quality analysis and improvement guidance based on Clean Code principles. Use this skill when users request code review, refactoring suggestions, code quality assessment, or ask to improve code readability and maintainability. Triggers include requests like "review this code", "make this cleaner", "refactor this", "is this good code", "improve code quality", or when analyzing code structure, naming conventions, and design patterns.
---

# Clean Code

This skill provides systematic guidance for writing and reviewing clean, maintainable code based on industry best practices and Clean Code principles.

## Core Philosophy

> Anyone can write code that a computer can understand, but few programmers know how to write code that a human can understand.
> -- Martin Fowler

**Key principles:**
- True cost of software = its maintenance
- We READ 10x more than we WRITE - optimize for readability
- Boy Scout Rule: Always check in cleaner code than you found
- The day you stop refactoring is the day it becomes legacy

## Code Review Checklist

When reviewing code, evaluate these aspects in order:

### 1. Naming Conventions

**Variables and Functions:**
- Avoid abbreviations unless universally recognized (URL, VAT, API)
- Use meaningful, self-explanatory names
- Functions are verbs: `searchProduct()`, `sendTransaction()`, not `product()` or `transaction()`
- Booleans answer yes/no: `isGoldClient()`, `areHostsValid()`
- Classes are nouns: `Customer`, `OrderDetails`, `OrderFacade`
- Avoid meaningless suffixes: prefer `Order` over `OrderInfo` or `OrderData`

**Consistency:**
- Stick to one convention: `find()`, `fetch()`, or `get()` - not a mix
- Use consistent terminology: don't alternate between `buyer`, `client`, `customer` for the same entity
- Variable names within functions should be consistent: `removeBtn` and `addBtn`, not `removeBtn` and `addButton`

**Continuous Renaming:**
- Rename as you learn the domain
- No perfect names exist - iterate
- Use IDE refactoring tools

### 2. Function Design

**Size and Responsibility:**
- Functions should be small: 5-10 lines ideally, never more than screen height
- Single Responsibility Principle (SRP): do ONE thing, do it well, do it ONLY
- Break down functions that take too long to understand

**Parameters:**
- Limit function arguments (ideally ≤3)
- Use destructuring or object parameters for multiple arguments
- Provide default values for optional parameters

**Return Values:**
- Avoid returning null - use Optional types or throw exceptions
- Use early returns to reduce nesting
- Return expected values last, do validation checks first

**Side Effects:**
- Function name should describe ALL its effects
- Avoid unintended consequences
- Don't add hidden functionality not reflected in the name

### 3. Code Structure

**Formatting:**
- Line length: maximum 120 characters
- Function length: 5-10 lines, less than one screen
- Class length: 100-200 lines, never exceed 500

**Indentation and Nesting:**
- More indentation = less readability
- Prefer early returns over nested conditionals
- Check negatives first and return early
- Multiple `if` statements are better than nested ones

**Variable Declaration:**
- Declare variables close to their usage
- Avoid declaring all variables at the top

**Function Organization:**
- Function callers and callees should be close
- Related functions should be grouped

### 4. Comments

**When NOT to Comment:**
- Comments often indicate failure to write clear code
- Self-explanatory code is better than commented code
- Comments inevitably fall out of sync

**When to Comment:**
- Specific algorithms requiring explanation
- Bug workarounds (with reference to bug tracker)
- Warning of consequences
- Explaining calls to strange APIs
- TODO markers (with context)
- Public API documentation for libraries/frameworks

**Comment Quality:**
- Only comment business logic complexity
- Code describes HOW, comments explain WHY
- Clarify regex or complex expressions with examples
- Never leave commented-out code (use version control)
- No journal comments (use `git log`)
- No positional markers or decorative headers

### 5. Code Smells to Detect

**Duplication:**
- DRY principle (Don't Repeat Yourself)
- Extract repeated logic into functions, methods, or services
- Repeating code is a primary code smell

**Complexity:**
- Long functions or methods
- Large classes with too many responsibilities
- Switch statements that could be polymorphism
- Combinatorial explosions (many methods doing similar things)

**Design Issues:**
- Lazy classes (too little responsibility - merge into others)
- Feature envy (method using another class more than its own)
- Solution sprawl (solution scattered across multiple classes)
- Alternative classes with different interfaces
- Oddball solutions (inconsistent problem-solving approaches)

### 6. Testing

**Test Requirements:**
- Always test manually first
- Write automated tests: E2E or unit tests based on requirements
- Minimum automated test coverage for critical paths

**Testing Philosophy:**
- You're not done when code works - refactor and clean up
- Continuous refactoring approach
- Build design skills through reflection

### 7. Error Handling

**Best Practices:**
- Fail fast: errors should surface quickly
- Use global exception handlers
- Minimize `catch` blocks in business logic
- Throw meaningful exceptions with context

### 8. Language-Specific Patterns

**JavaScript/TypeScript:**
- Use destructuring for cleaner code
- Employ optional chaining: `person?.address?.street`
- Prefer `async/await` over callbacks (or vice versa - but be consistent)
- Use default parameters
- Destructure function parameters for clarity

**General Patterns:**
- Use language idioms and conventions
- Follow framework/library style guides
- Configure linters and formatters
- Automate style enforcement

## SOLID Principles

When evaluating design patterns, reference SOLID:

- **S**: Single Responsibility Principle - One reason to change
- **O**: Open/Closed Principle - Open for extension, closed for modification
- **L**: Liskov Substitution Principle - Derived classes must be substitutable
- **I**: Interface Segregation Principle - Don't force unused method dependencies
- **D**: Dependency Inversion Principle - Depend on abstractions, not concretions

## Code Review Process

**When Reviewing:**
1. Understand the context and constraints
2. Focus on what humans should review (not what can be automated)
3. Provide constructive, specific feedback
4. Be nice - the goal is to ship code, not prove cleverness
5. Respond in a timely fashion

**Submitting for Review:**
- Keep changes small and focused
- Annotate your thinking process in commit messages
- Include screenshots/GIFs for UI changes
- One issue per pull request

**Anti-Patterns to Avoid:**
- Nitpicking whitespace (automate it)
- Design changes at final review stage
- Inconsistent feedback
- Ghosting large PRs after nitpicking small ones
- Endless ping-pong reviews

## Refactoring Guidelines

**When to Refactor:**
- As soon as code starts working
- When changing any part of codebase (Boy Scout Rule)
- When names no longer reflect current understanding
- When functions become too complex

**How to Refactor:**
- Use IDE shortcuts for fast restructuring
- Refactor in small, safe steps
- Keep tests green
- Commit frequently
- Remove zombie code (dead/commented code)

## Output Format

When reviewing code, provide:

1. **Overall Assessment**: Brief summary of code quality
2. **Critical Issues**: Must-fix problems affecting correctness or security
3. **High-Priority Improvements**: Significant maintainability or readability issues
4. **Suggestions**: Nice-to-have improvements
5. **Positive Observations**: What's done well

For each issue, provide:
- Location (file, line, or function name)
- Explanation of the problem
- Specific refactoring suggestion with code example
- Rationale (why this improves the code)

**Important:** Balance thoroughness with pragmatism. Not every issue needs fixing immediately. Prioritize based on impact and effort.

## Performance Note

> Premature optimization is the root of all evil.
> -- Donald Knuth

- Write clean code first, optimize only when necessary
- Smaller methods often run faster due to JIT compiler optimizations
- Measure before optimizing
- Readability usually trumps micro-optimizations

## Remember

- Clean code is about empathy for future maintainers (including yourself)
- Code is read far more often than written
- Perfect is the enemy of good - iterate toward better code
- Automate what can be automated (formatting, style checks)
- Focus human review on logic, design, and clarity

```


### LLM Exposure :
`https://docs.google.com/document/d/1IKRIbrFQyxFAaFBiGm6GzmOVSJw6juq8UHtxVQjOmBw/edit?usp=sharing`