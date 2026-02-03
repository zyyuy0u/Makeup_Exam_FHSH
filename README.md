# 臺北市立復興高級中學 學分補考查詢系統

## 🚀 快速開始

### 使用 Docker Compose (推薦)

```bash
# 1. 複製環境變數檔案
cp .env.example .env

# 2. 編輯 .env 設定 Secret Token
# 產生 token: python -c "import secrets; print(secrets.token_hex(32))"
vim .env

# 3. 啟動服務
docker compose up -d --build

# 4. 查看服務狀態
docker compose ps

# 5. 查看 Secret Token（若未手動設定）
docker compose logs backend | grep "ADMIN_SECRET_TOKEN"
```

服務將在以下位置啟動：
- 前端 (學生查詢): http://localhost
- 後端 API: http://localhost:8000

### 停止服務

```bash
docker compose down
```

## 📋 功能說明

### 學生端
- 輸入學號查詢補考科目
- 顯示科目、日期、時間、地點
- 響應式設計，支援手機瀏覽

### 管理端 (API)
- 使用 Secret Token 進行身份驗證
- 透過 Google Apps Script 上傳 Excel 補考名單
- 全量覆蓋更新 (每次上傳會清除舊資料)

## 🔐 安全機制

本系統移除傳統的網頁登入介面，改用 Secret Token 機制：

1. **無登入頁面**: 消除暴力破解攻擊面
2. **Secret Token**: 使用 256 位元 (32 bytes) 隨機金鑰
3. **Header 傳輸**: Token 透過 `X-Admin-Token` HTTP Header 傳送
4. **Timing Attack 防護**: 使用 `secrets.compare_digest` 進行比對

### Google Apps Script 呼叫範例

```javascript
function uploadExcel() {
  const url = 'https://your-domain.com/admin/upload';
  const token = 'your_secret_token_here';

  // 取得 Google Drive 中的 Excel 檔案
  const file = DriveApp.getFileById('your_file_id');
  const blob = file.getBlob();

  const options = {
    method: 'post',
    headers: {
      'X-Admin-Token': token
    },
    payload: {
      file: blob
    }
  };

  const response = UrlFetchApp.fetch(url, options);
  Logger.log(response.getContentText());
}
```

## 📁 專案結構

```
├── backend/           # FastAPI 後端
│   ├── main.py       # 主應用程式
│   ├── models.py     # SQLModel 資料模型
│   ├── database.py   # 資料庫設定
│   ├── routers/      # API 路由
│   ├── services/     # 服務層 (Excel 解析)
│   ├── utils/        # 工具函式 (async, webpage)
│   └── Dockerfile
├── frontend/          # React 前端
│   ├── src/
│   ├── nginx.conf
│   └── Dockerfile
└── docker-compose.yml
```

## 🔧 開發環境

### 後端開發

```bash
# 安裝 uv（如果尚未安裝）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 從專案根目錄執行
cd backend
uv pip install -r pyproject.toml

# 啟動開發伺服器（回到專案根目錄）
cd ..
uvicorn backend.main:app --reload
```

### 前端開發

```bash
cd frontend
npm install
npm run dev
```

## 📝 Excel 檔案格式

系統讀取工作表「**應到考名單 (班級座號序)**」，欄位需求如下：

**必要欄位：**
- 學號
- 補考科目
- 補考日期
- 補考時間
- 補考教室

**選填欄位：**
- 姓名1（或 姓名）
- 班級

## 🎨 配色方案

- 主色調: #00A99D (活潑藍綠)
- 強調色: #FF6F61 (珊瑚橘紅)
- 支援深色模式自動切換

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
