# Pre-Session Install Checklist
**Complete this the night before. Not during the session.**

Estimated time: 15–30 minutes depending on internet speed.
The sentence-transformers model download is ~80MB. Do not skip it.

---

## Step 1 — Verify Python

```bash
python3 --version
```

Must show 3.9 or higher. If not installed: https://python.org

---

## Step 2 — Install Python Packages

Run all three install commands. Do not skip any.

```bash
pip3 install sentence-transformers
pip3 install chromadb
pip3 install google-generativeai
```

**Verify each installed correctly:**

```bash
python3 -c "from sentence_transformers import SentenceTransformer; print('sentence-transformers OK')"
python3 -c "import chromadb; print('chromadb OK')"
python3 -c "import google.generativeai; print('google-generativeai OK')"
```

All three must print OK. If any fail, re-run the pip3 install for that package.

---

## Step 3 — Pre-Download the Embedding Model

This downloads ~80MB once. After this it works offline.

```bash
python3 -c "
from sentence_transformers import SentenceTransformer
print('Downloading all-MiniLM-L6-v2...')
m = SentenceTransformer('all-MiniLM-L6-v2')
test = m.encode(['test sentence'])
print(f'Model ready. Vector size: {test.shape[1]}')
"
```

Must print: `Model ready. Vector size: 384`

---

## Step 4 — Get a Gemini API Key (Free)

1. Go to https://aistudio.google.com/app/apikey
2. Sign in with your Google account
3. Click **Create API key**
4. Copy the key

**Set it as an environment variable:**

Mac / Linux:
```bash
export GEMINI_API_KEY="your-key-here"
```

Windows (Command Prompt):
```
set GEMINI_API_KEY=your-key-here
```

Windows (PowerShell):
```
$env:GEMINI_API_KEY="your-key-here"
```

**Test it:**
```bash
python3 -c "
import os, google.generativeai as genai
genai.configure(api_key=os.environ['GEMINI_API_KEY'])
m = genai.GenerativeModel('gemini-1.5-flash')
r = m.generate_content('Say: Gemini API working')
print(r.text)
"
```

Must print something containing "Gemini API working".

> **Alternative LLMs:** If you have a Claude or OpenAI API key, you can
> use those instead. See `uc-mcp/llm_adapter.py` for instructions.
> You still need `google-generativeai` installed even if switching LLMs.

---

## Step 5 — Install Git and GitHub Desktop

- Git: https://git-scm.com/downloads
- GitHub Desktop: https://desktop.github.com

**Verify Git:**
```bash
git --version
```

Sign into GitHub Desktop with your GitHub account before the session.

---

## Step 6 — Fork and Clone the Repo

1. Go to https://github.com/nasscomAI/RAG-to-MCP
2. Click **Fork** → Create fork
3. Open GitHub Desktop → File → Clone Repository
4. Clone your fork to a local folder

**Verify the clone:**
```bash
cd RAG-to-MCP
ls
```

Must show: `uc-0a  uc-rag  uc-mcp  data  docs  README.md`

---

## Step 7 — Build the Stub Index

This builds the RAG index from the policy documents so UC-MCP works
even if your rag_server.py is not finished.

```bash
cd uc-rag
python3 stub_rag.py --build-index
```

Must print: `Index built at ./stub_chroma_db`

**Test a query:**
```bash
python3 stub_rag.py --query "Who approves leave without pay?"
```

Must return an answer citing `policy_hr_leave.txt`.

---

## Step 8 — Final Verification

Run all checks in one go:

```bash
python3 -c "
import sys
checks = []

# Python version
v = sys.version_info
checks.append(('Python 3.9+', v >= (3,9), f'{v.major}.{v.minor}'))

# Packages
for pkg in ['sentence_transformers','chromadb','google.generativeai']:
    try:
        __import__(pkg)
        checks.append((pkg, True, 'installed'))
    except ImportError:
        checks.append((pkg, False, 'NOT INSTALLED'))

# Model
try:
    from sentence_transformers import SentenceTransformer
    m = SentenceTransformer('all-MiniLM-L6-v2')
    checks.append(('Embedding model', True, 'ready'))
except Exception as e:
    checks.append(('Embedding model', False, str(e)))

# ChromaDB stub index
import os, sys
idx_exists = os.path.exists('../uc-rag/stub_chroma_db') or os.path.exists('./stub_chroma_db')
checks.append(('Stub index built', idx_exists, 'found' if idx_exists else 'CRITICAL: RUN python3 uc-rag/stub_rag.py --build-index'))

# Stub actually works
if idx_exists:
    try:
        sys.path.insert(0, '../uc-rag' if os.path.exists('../uc-rag') else 'uc-rag')
        from stub_rag import query as stub_query
        result = stub_query('test query')
        checks.append(('Stub query works', True, 'answered'))
    except Exception as e:
        checks.append(('Stub query works', False, f'CRITICAL: {str(e)[:50]}'))
else:
    checks.append(('Stub query works', False, 'CRITICAL: build index first'))

print()
for name, ok, note in checks:
    status = '✅' if ok else '❌'
    print(f'{status}  {name}: {note}')

all_ok = all(ok for _, ok, _ in checks)
print()
print('READY FOR SESSION' if all_ok else 'FIX THE ❌ ITEMS ABOVE BEFORE THE SESSION')
"
```

All items must show ✅ before the session starts.

---

## If Something Fails

- `pip3 install` fails → try `pip3 install --upgrade pip` first, then retry
- Model download fails → check internet connection, retry
- Gemini key fails → check for extra spaces when copying the key
- ChromaDB import error on Windows → try `pip3 install chromadb --upgrade`
- Any other issue → email your tutor with the error message before the session
