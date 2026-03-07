# Python → TypeScript Porting Cheatsheet

Quick reference for translating Python MCP server code to TypeScript CLI code.

## Module Mapping

| Python | TypeScript |
|:-------|:-----------|
| `httpx.AsyncClient` | `fetch` (built-in Node 18+) |
| `aiofiles` | `fs/promises` |
| `tenacity` retry | `p-retry` or manual retry loop |
| `click` / `typer` | `commander` |
| `pydantic` models | TypeScript interfaces + runtime checks |
| `os.getenv()` | `process.env` |
| `pathlib.Path` | `path.resolve()` / `path.join()` |
| `mimetypes.guess_type()` | `mime-types` npm package |
| `json.dumps(indent=2)` | `JSON.stringify(data, null, 2)` |
| `asyncio.run()` | Top-level await (ESM) |
| `os.path.exists()` | `fs.existsSync()` or `fs.access()` |
| `os.path.getsize()` | `(await fs.stat(path)).size` |
| `os.makedirs(exist_ok=True)` | `fs.mkdir(path, { recursive: true })` |
| `base64.b64encode()` | `Buffer.from(data).toString('base64')` |

## Pattern Translations

### HTTP Request

```python
# Python (httpx)
async with httpx.AsyncClient(timeout=300) as client:
    response = await client.post(url, headers=headers, json=body)
    response.raise_for_status()
    return response.json()
```

```typescript
// TypeScript (fetch)
const res = await fetch(url, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', ...headers },
  body: JSON.stringify(body),
  signal: AbortSignal.timeout(300_000),
});
if (!res.ok) throw new Error(`${res.status} ${await res.text()}`);
return res.json();
```

### File Upload (multipart)

```python
# Python
files = {"document": open(file_path, "rb")}
data = {"model": "document-parse"}
response = await client.post(url, files=files, data=data)
```

```typescript
// TypeScript
const body = new FormData();
body.append('document', new Blob([await readFile(filePath)]));
body.append('model', 'document-parse');
const res = await fetch(url, { method: 'POST', body });
```

### Retry Logic

```python
# Python (tenacity)
@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=4, max=10))
async def call_api():
    ...
```

```typescript
// TypeScript (manual)
async function callApi(maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fetch(url);
    } catch (e) {
      if (i === maxRetries - 1) throw e;
      await new Promise(r => setTimeout(r, Math.min(4000 * 2 ** i, 10000)));
    }
  }
}
```

### Progress Reporting

```python
# Python MCP context
await ctx.report_progress(30, 100)
ctx.info("Processing document...")
```

```typescript
// TypeScript CLI (stderr)
process.stderr.write('Processing document...\n');
// Or use ora for spinners:
// import ora from 'ora';
// const spinner = ora('Processing...').start();
// spinner.succeed('Done');
```

### Environment Variables

```python
# Python
from dotenv import load_dotenv
load_dotenv()
api_key = os.getenv("API_KEY")
if not api_key:
    raise ValueError("API_KEY not set")
```

```typescript
// TypeScript
const apiKey = process.env.API_KEY;
if (!apiKey) {
  console.error('Set API_KEY env var');
  process.exit(1);
}
```
