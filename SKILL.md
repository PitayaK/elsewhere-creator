---
name: elsewhere-creator
description: Elsewhere creator toolkit — register a new account (with invite code), publish articles as drafts, and manage content via Supabase.
user-invocable: true
metadata: {"openclaw":{"emoji":"✍️"}}
---

# Elsewhere Creator

创作者工具：注册账号、发布文章。

## Config

```
SUPABASE_URL=https://kkcktdhgzddmdjdjoenc.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtrY2t0ZGhnemRkbWRqZGpvZW5jIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM4MDgwNzIsImV4cCI6MjA4OTM4NDA3Mn0.7RaxKWdOEwM95KgDSEv3nUYlrO73drFuRaIgvjs-wZk
WEBSITE_URL=https://elsewhere.news
```

---

## Command: Register

Use when the user wants to register as a new Elsewhere creator.

### Step 1: Ask for invite code

Ask the user for their 6-digit invite code (uppercase letters and numbers).

### Step 2: Generate registration link

Call the API to validate the invite code and get a registration link. Use your own name (e.g. "Claude", "Cursor", "Kimi") as `agent_name`:

```bash
curl -s -X POST "https://elsewhere.news/api/register/init" \
  -H "Content-Type: application/json" \
  -d "{\"invite_code\": \"INVITE_CODE\", \"agent_name\": \"YOUR_NAME\"}" | python3 -m json.tool
```

Replace `INVITE_CODE` with the user's code, `YOUR_NAME` with your agent name.

If the response contains an error, tell the user (invite code may be invalid or already used).

### Step 3: Give the link to the human

On success, the API returns a `register_url`. Tell the user:

> 请在浏览器中打开以下链接完成注册：
>
> {register_url}
>
> 注册页面会要求你填写邮箱和密码，邮箱会收到一个 6 位验证码，输入后即完成注册。

### Step 4: Save credentials

Tell the user:

> 注册完成后，请告诉我你的邮箱和密码，我会保存到环境变量中，以后就可以直接帮你发布文章了。

When the user provides their email and password, save to `.env.local` (or equivalent config):

```bash
echo 'ELSEWHERE_EMAIL=user@example.com' >> .env.local
echo 'ELSEWHERE_PASSWORD=their_password' >> .env.local
```

### Step 5: Verify connection

```bash
curl -s -X POST "https://kkcktdhgzddmdjdjoenc.supabase.co/auth/v1/token?grant_type=password" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtrY2t0ZGhnemRkbWRqZGpvZW5jIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM4MDgwNzIsImV4cCI6MjA4OTM4NDA3Mn0.7RaxKWdOEwM95KgDSEv3nUYlrO73drFuRaIgvjs-wZk" \
  -H "Content-Type: application/json" \
  -d "{\"email\": \"USER_EMAIL\", \"password\": \"USER_PASSWORD\"}" | python3 -c "import sys,json; d=json.load(sys.stdin); print('Connected!' if 'access_token' in d else 'Error: ' + d.get('error_description', 'unknown'))"
```

Replace `USER_EMAIL` and `USER_PASSWORD` with the user's credentials.

---

## Command: Publish Article

Use when the user wants to publish an article as a draft. Requires `ELSEWHERE_EMAIL` and `ELSEWHERE_PASSWORD` (from registration).

### Step 1: Authenticate

```bash
ACCESS_TOKEN=$(curl -s -X POST "https://kkcktdhgzddmdjdjoenc.supabase.co/auth/v1/token?grant_type=password" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtrY2t0ZGhnemRkbWRqZGpvZW5jIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM4MDgwNzIsImV4cCI6MjA4OTM4NDA3Mn0.7RaxKWdOEwM95KgDSEv3nUYlrO73drFuRaIgvjs-wZk" \
  -H "Content-Type: application/json" \
  -d "{\"email\": \"$ELSEWHERE_EMAIL\", \"password\": \"$ELSEWHERE_PASSWORD\"}" | python3 -c "import sys,json; print(json.load(sys.stdin)['access_token'])")
```

Get the author ID:

```bash
AUTHOR_ID=$(curl -s "https://kkcktdhgzddmdjdjoenc.supabase.co/rest/v1/authors?select=id&user_id=eq.$(curl -s "https://kkcktdhgzddmdjdjoenc.supabase.co/auth/v1/user" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtrY2t0ZGhnemRkbWRqZGpvZW5jIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM4MDgwNzIsImV4cCI6MjA4OTM4NDA3Mn0.7RaxKWdOEwM95KgDSEv3nUYlrO73drFuRaIgvjs-wZk" \
  -H "Authorization: Bearer $ACCESS_TOKEN" | python3 -c "import sys,json; print(json.load(sys.stdin)['id'])")" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtrY2t0ZGhnemRkbWRqZGpvZW5jIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM4MDgwNzIsImV4cCI6MjA4OTM4NDA3Mn0.7RaxKWdOEwM95KgDSEv3nUYlrO73drFuRaIgvjs-wZk" \
  -H "Authorization: Bearer $ACCESS_TOKEN" | python3 -c "import sys,json; print(json.load(sys.stdin)[0]['id'])")
```

### Step 2: Parse the article content

From the article content in the conversation, extract:
- **Title** (标题)
- **Excerpt** (摘要): 1-2 sentence summary. Generate one if not provided.
- **Body** (正文)

### Step 3: Convert body to Markdown

- Preserve headings, bold, italic, links, lists, code blocks, blockquotes, tables, images
- Remove source document artifacts (Feishu/Word metadata)
- Clean paragraph separation (double newline)

### Step 4: Ask for category (if not specified)

Fetch available categories:

```bash
curl -s "https://kkcktdhgzddmdjdjoenc.supabase.co/rest/v1/categories?select=id,title_zh,slug" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtrY2t0ZGhnemRkbWRqZGpvZW5jIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM4MDgwNzIsImV4cCI6MjA4OTM4NDA3Mn0.7RaxKWdOEwM95KgDSEv3nUYlrO73drFuRaIgvjs-wZk" \
  -H "Authorization: Bearer $ACCESS_TOKEN" | python3 -m json.tool
```

Ask the user to pick one, or suggest one based on content.

### Step 5: Generate a slug

URL-friendly, lowercase, hyphenated. Use English title if available, otherwise romanize Chinese.

### Step 6: Create draft

Write JSON payload to temp file:

```bash
cat > /tmp/article.json << 'JSONEOF'
{
  "title_zh": "中文标题",
  "slug": "the-slug",
  "excerpt_zh": "中文摘要",
  "body_zh": "Full article body in Markdown",
  "author_id": "AUTHOR_ID_VALUE",
  "category_id": "CATEGORY_ID_VALUE",
  "published_at": "2024-01-01T00:00:00Z",
  "is_draft": true
}
JSONEOF
```

Then send:

```bash
curl -s -X POST "https://kkcktdhgzddmdjdjoenc.supabase.co/rest/v1/articles" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtrY2t0ZGhnemRkbWRqZGpvZW5jIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzM4MDgwNzIsImV4cCI6MjA4OTM4NDA3Mn0.7RaxKWdOEwM95KgDSEv3nUYlrO73drFuRaIgvjs-wZk" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=representation" \
  -d @/tmp/article.json
```

Use current date/time for `published_at`. Replace `AUTHOR_ID_VALUE` and `CATEGORY_ID_VALUE` with actual values.

### Step 7: Confirm

Tell the user: article title, category, status (Draft).

---

## Important Notes

- Registration links expire in 24 hours; each invite code is single-use
- Always save articles as **draft** (`is_draft: true`), never publish directly
- Only fill Chinese fields (title_zh, excerpt_zh, body_zh). English translation is handled separately
- Always write JSON to temp file and use `curl -d @file` to avoid shell escaping issues
- After registration, human can log into GUI dashboard at `https://elsewhere.news/dashboard/login`
