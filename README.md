# Generic Web API for LLM Integration

A flexible Node.js web API that allows you to create LLM-powered applications by simply editing a Markdown prompt file. No code changes required for different use cases.

## よりみち旅（AI日帰りプランナー）

このプロジェクトには、入力した行き先・時間・予算・気分に合わせて、AIが日帰りのおでかけプランを作るWebアプリを追加しています。

1. `.env.local` に `OPENAI_API_KEY` を設定して `npm start` を実行します。
2. ブラウザーで `http://localhost:8080` を開き、「よりみち旅」を選びます。
3. 条件を入力してプランを作成します。生成したプランはタイムラインで確認でき、JSONも展開して見られます。

`POST /api/trip` は `trip-prompt.md` の指示と入力条件を OpenAI API に送り、`{"itinerary":[...]}` 形式のJSONを求めます。サーバーで応答を `JSON.parse()` し、各スポットの項目を確認してから画面へ返します。プロンプトとUIはクイズアプリから独立しているため、既存の `quiz.html` も引き続き利用できます。

各スポットには `time`、`title`、`description`、`location`、`estimatedCost`、`category`、`tip` を含みます。費用や施設情報はAIによる目安です。

### Renderへデプロイ

1. このリポジトリをGitHubへpushし、Renderで **New → Blueprint** からリポジトリを接続します。
2. ルートの `render.yaml` を使ってデプロイします。Build Command は `npm ci`、Start Command は `npm start` です。
3. Renderの初回セットアップで `OPENAI_API_KEY` を入力します。`.env.local` はGitに含めず、RenderのEnvironmentにもキーを登録してください。

デプロイ完了後、Renderが発行する `onrender.com` URLを共有できます。Freeプランは一定時間アクセスがないと停止するため、最初のアクセス時に起動待ちが発生することがあります。

## Features

- 🎯 **Generic Design**: One API endpoint for any LLM application
- 📝 **Markdown Prompts**: Define your application logic in `prompt.md`
- 🔄 **Variable Substitution**: Automatic replacement of `${variable}` placeholders
- 🤖 **Multi-Provider**: Supports OpenAI and Google Gemini
- ⚡ **No Code Changes**: Switch between applications by editing `prompt.md`

## Quick Start

### 1. Installation

```bash
npm install
```

### 2. Environment Setup

Copy `.env.example` to `.env.local` and set your API key:

```bash
cp .env.example .env.local
```

Edit `.env.local`:
```env
# For OpenAI (default)
OPENAI_API_KEY=your_openai_api_key_here

# For Gemini (if switching)
GEMINI_API_KEY=your_gemini_api_key_here

PORT=8080
```

> `.env.local` holds your real keys and is git-ignored. Do not commit it.

### 3. Configure LLM Provider

Edit the `PROVIDER` constant near the top of `server.js`:

```javascript
// For OpenAI (default) -> gpt-6-luna
const PROVIDER = 'openai';

// For Gemini -> gemini-4-hydrogen
// const PROVIDER = 'gemini';
```

The model is selected automatically from the `MODELS` map based on `PROVIDER`.

### 4. Start Server

```bash
npm start
```

Visit `http://localhost:8080`

## How It Works

### Architecture

```
Client (quiz.html) → POST /api/ → server.js → LLM → Response
                                     ↓
                              prompt.md (template)
```

### Variable Substitution

The API automatically replaces variables in `prompt.md` with request data:

**prompt.md:**
```markdown
Create ${count} questions about ${topic}.
Format: JSON array
```

**Request:**
```json
{
  "count": 5,
  "topic": "JavaScript"
}
```

**Result:** Variables `${count}` and `${topic}` are replaced with actual values.

### API Endpoint

**POST** `/api/`

**Request Body:**
```json
{
  "title": "My Quiz",
  "count": 5,
  "any_variable": "value"
}
```

**Response:**
```json
{
  "title": "My Quiz",
  "data": [...]
}
```

## Example Applications

### 1. IT Certification Quiz (Included)

**Files:**
- `prompt.md` - Defines IT quiz generation logic
- `public/quiz.html` - Quiz interface

**Usage:** Generate IT certification practice questions

### 2. Translation App (Example)

**prompt.md:**
```markdown
# Translation Service

Translate the following text to ${target_language}:

"${text}"

Return only the translated text.
```

**Request:**
```json
{
  "text": "Hello world",
  "target_language": "Japanese"
}
```

### 3. Code Review App (Example)

**prompt.md:**
```markdown
# Code Review Assistant

Review this ${language} code and provide feedback:

```${language}
${code}
```

Provide:
1. Issues found
2. Suggestions for improvement
3. Best practices
```

**Request:**
```json
{
  "language": "Python",
  "code": "def hello():\n    print('world')"
}
```

## File Structure

```
generic-webapi/
├── server.js          # Generic API server (no changes needed)
├── prompt.md          # Application-specific prompt template
├── package.json       # Dependencies
├── .env.example       # Environment variables template
├── public/            # Static files
│   ├── quiz.html     # IT quiz application
│   ├── style.css     # Styles
│   └── quiz.css      # Quiz-specific styles
└── README.md         # This file
```

## Creating New Applications

1. **Edit `prompt.md`** - Define your application logic and variables
2. **Create client HTML** - Build your user interface in `public/`
3. **Send requests** - Use any variables you defined in `prompt.md`

No server code changes required!

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `GEMINI_API_KEY` | Google Gemini API key | Yes (if using Gemini) |
| `OPENAI_API_KEY` | OpenAI API key | Yes (if using OpenAI) |
| `PORT` | Server port | No (default: 8080) |

## LLM Provider Configuration

### Switch to OpenAI (default)

1. Edit `server.js`:
```javascript
const PROVIDER = 'openai';
```

2. Set OpenAI API key in `.env.local`:
```env
OPENAI_API_KEY=your_key_here
```

### Switch to Gemini

1. Edit `server.js`:
```javascript
const PROVIDER = 'gemini';
```

2. Set Gemini API key in `.env.local`:
```env
GEMINI_API_KEY=your_key_here
```

## Development

### Run with auto-restart:
```bash
npm run dev
```

### Supported Models

**OpenAI:**
- `gpt-6-luna` (default)

**Gemini:**
- `gemini-4-hydrogen` (called with `thinkingLevel: "LOW"`; thinking tokens count toward `maxOutputTokens`)

## License

MIT

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request
