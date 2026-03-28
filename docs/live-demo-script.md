LIVE DEMO SCRIPT — BUILD A RAG SERVER IN 30 MINUTES
RAG-to-MCP Workshop · Facilitator Copy
Tools: Gemini CLI (terminal) + TRAE by ByteDance

═══════════════════════════════════════════════════════════════════════
PRE-DEMO SETUP (do this before Block 2 starts — not on screen)
═══════════════════════════════════════════════════════════════════════

Terminal 1 — your working terminal:
  cd RAG-to-MCP/uc-rag
  ls                          # confirm rag_server.py is the starter file
  cat rag_server.py           # confirm it has NotImplementedError stubs

Terminal 2 — Gemini CLI ready:
  gemini                      # start the CLI, confirm it responds

TRAE:
  Open RAG-to-MCP/uc-rag/rag_server.py in TRAE
  Open uc-rag/README.md in a split pane (participants can see the source)

Confirm:
  echo $GEMINI_API_KEY         # must print a key, not blank
  python3 -c "from sentence_transformers import SentenceTransformer; print('OK')"
  python3 -c "import chromadb; print('OK')"

Have ready (not visible to room):
  /home/[you]/recovery/chunk_documents.py    # pre-written fallback per stage
  /home/[you]/recovery/build_index.py
  /home/[you]/recovery/retrieve_and_answer.py

═══════════════════════════════════════════════════════════════════════
THE SCRIPT
═══════════════════════════════════════════════════════════════════════

────────────────────────────────────────────────────────────────────
MIN 0–2  OPENING — SET THE FRAME
────────────────────────────────────────────────────────────────────

[Show rag_server.py on screen — the starter file with NotImplementedError]

SAY:
  "This is what you have right now. Four functions.
   All of them raise NotImplementedError.
   In 30 minutes this file will run against our three policy documents,
   embed them into ChromaDB, retrieve the right chunks for any query,
   and answer with a citation.

   We are going to build it using the same workflow you just used in UC-0A.
   RICE first. agents.md and skills.md before any code.
   Then TRAE implements it function by function.
   Then we run it, find the failure, fix it.

   That is the CRAFT loop applied to RAG.
   Watch what I do — because in 30 minutes you are doing it yourself."

[Switch to Terminal 2 — Gemini CLI visible]

────────────────────────────────────────────────────────────────────
MIN 2–5  STEP 1 — GENERATE agents.md WITH GEMINI CLI
────────────────────────────────────────────────────────────────────

[In Gemini CLI terminal — type this live, participants watch]

SAY:
  "First step — RICE. I am going to paste the UC-RAG README into
   Gemini CLI and ask it to generate agents.md.
   I am not writing the RICE prompt myself.
   The README contains everything the AI needs."

TYPE in gemini CLI:
┌─────────────────────────────────────────────────────────────────┐
  Read the following UC README. Using the R.I.C.E framework,
  generate an agents.md YAML file with four fields:
  role, intent, context, enforcement.

  R.I.C.E:
  - Role: who the agent is and its operational boundary
  - Intent: what a correct output looks like — verifiable
  - Context: what the agent may use — state exclusions explicitly
  - Enforcement: every rule listed under
    "Enforcement Rules Your agents.md Must Include"

  Output only valid YAML. No explanation, no code fences.

  README:
  [paste full contents of uc-rag/README.md here]
└─────────────────────────────────────────────────────────────────┘

[Wait for output. Copy it.]

SAY:
  "Now I read it. I do not commit the first draft."

[Read enforcement section aloud. Point out what's missing.]

SAY:
  "The similarity threshold is 0.6 — that is a specific number.
   Does the AI have it? Let me check."

[If threshold is missing or vague — edit agents.md live in TRAE]

TYPE the fix into agents.md directly:
  - "Retrieve only chunks scoring above similarity threshold 0.6.
     Chunks below 0.6 are excluded — never passed to the LLM."

SAY:
  "That is the refinement step. The AI gave me the structure.
   I gave it the specificity. Now we have a real Enforcement rule —
   not a wish."

[Save agents.md]

────────────────────────────────────────────────────────────────────
MIN 5–8  STEP 2 — GENERATE skills.md WITH GEMINI CLI
────────────────────────────────────────────────────────────────────

SAY:
  "Same process for skills.md."

TYPE in gemini CLI:
┌─────────────────────────────────────────────────────────────────┐
  Read the following UC README. Generate a skills.md YAML file
  defining two skills: chunk_documents and retrieve_and_answer.

  Each skill needs: name, description, input, output, error_handling.
  error_handling must address the failure modes in the README.

  Output only valid YAML. No explanation, no code fences.

  README:
  [paste full contents of uc-rag/README.md here]
└─────────────────────────────────────────────────────────────────┘

[Copy output to skills.md in TRAE. Read error_handling for retrieve_and_answer.]

SAY:
  "The error_handling for retrieve_and_answer must say what happens
   when no chunk passes the threshold. Does it?"

[If it says 'return error' generically — edit it live]

TYPE the fix:
  error_handling: "If no retrieved chunk scores above 0.6,
    return the refusal template exactly:
    'This question is not covered in the retrieved policy documents.
    Retrieved chunks: [sources]. Please contact the relevant department.'"

SAY:
  "The refusal template wording is an Enforcement rule.
   'Return an error' is not the same as the exact wording.
   The wording is what participants will test against."

[Save skills.md]

────────────────────────────────────────────────────────────────────
MIN 8–13  STEP 3 — IMPLEMENT chunk_documents IN TRAE
────────────────────────────────────────────────────────────────────

[Switch to TRAE — rag_server.py open]

SAY:
  "Now TRAE implements the code. I give it agents.md, skills.md,
   and the README. It builds one function at a time.
   I do not ask for the whole file at once — that is how you lose
   control of what the AI is doing."

[In TRAE chat/composer — type:]
┌─────────────────────────────────────────────────────────────────┐
  Implement the chunk_documents function in rag_server.py.

  Requirements from agents.md and skills.md:
  - Load all .txt files from the docs_dir argument
  - Split each document into chunks respecting sentence boundaries
  - Maximum 400 tokens per chunk — never split mid-sentence
  - Return list of dicts: {doc_name, chunk_index, text, id}

  Use sentence splitting on ". " and ".\n" boundaries.
  Do not use NLTK — use regex only.
  Do not implement any other functions yet.
└─────────────────────────────────────────────────────────────────┘

[TRAE generates chunk_documents. Review it briefly on screen.]

SAY:
  "Let me run it immediately — not after the whole file is done."

[In Terminal 1:]

  python3 -c "
  from rag_server import chunk_documents
  chunks = chunk_documents('../data/policy-documents')
  print(f'{len(chunks)} chunks from {len(set(c[\"doc_name\"] for c in chunks))} documents')
  for c in chunks[:3]:
      print(f'  [{c[\"doc_name\"]}, chunk {c[\"chunk_index\"]}]: {c[\"text\"][:80]}...')
  "

[Expected output: ~60-80 chunks from 3 documents]

SAY:
  "Three documents. Sentence-aware chunks. This is the Ingest and
   Chunk stages working. The embedding model has not been called yet —
   we are still building the pipeline."

⚠️  RECOVERY — if chunk_documents fails:
  SAY: "TRAE gave us something that needs one more CRAFT cycle.
        Let me fix the specific error."
  Show the error to the room. Fix it in TRAE. Re-run.
  If still failing after 60 seconds: paste recovery/chunk_documents.py
  and say: "This is what a working version looks like —
            in your own session you would iterate with TRAE until it runs."

────────────────────────────────────────────────────────────────────
MIN 13–18  STEP 4 — IMPLEMENT build_index IN TRAE
────────────────────────────────────────────────────────────────────

SAY:
  "Next — embed and store. This is where sentence-transformers
   and ChromaDB come in. I am going to ask TRAE to implement
   build_index using the chunks we just produced."

[In TRAE:]
┌─────────────────────────────────────────────────────────────────┐
  Implement the build_index function in rag_server.py.

  Requirements:
  - Call chunk_documents() to get all chunks
  - Load SentenceTransformer('all-MiniLM-L6-v2')
  - Embed all chunk texts using model.encode()
  - Create a ChromaDB PersistentClient at db_path
  - Delete and recreate the collection named 'policy_docs'
  - Add all chunks with: ids, documents (text), metadatas
    (doc_name, chunk_index), embeddings
  - Print progress: number of chunks indexed and db_path

  Do not implement any other functions.
└─────────────────────────────────────────────────────────────────┘

[TRAE generates build_index. In Terminal 1:]

  python3 rag_server.py --build-index

[Show the progress bar. This takes 15–30 seconds. Talk during it.]

SAY WHILE INDEX BUILDS:
  "This is the embedding step. The model is converting every
   sentence chunk into a list of 384 numbers — a vector.
   Two chunks that talk about similar things will have similar
   vectors. That similarity is how retrieval works.

   The vectors are being stored in ChromaDB on disk right now.
   After this we never embed the documents again — only queries."

[When complete:]
  Expected: "Index built. 67 chunks indexed at ./chroma_db"

SAY:
  "Ingest, chunk, embed, store — four stages done.
   The retrieval and generation stages are next."

⚠️  RECOVERY — if build_index fails:
  Most common issue: ChromaDB collection already exists.
  Fix: add _client.delete_collection() before create_collection().
  Show TRAE the error, ask it to fix that specific line.
  If still failing: python3 stub_rag.py --build-index
  SAY: "The stub index works — let me use that while we fix the main build."

────────────────────────────────────────────────────────────────────
MIN 18–24  STEP 5 — IMPLEMENT retrieve_and_answer IN TRAE
────────────────────────────────────────────────────────────────────

SAY:
  "This is the most important function — and the one where all
   three failure modes live. I am going to be specific with TRAE
   about the enforcement rules."

[In TRAE:]
┌─────────────────────────────────────────────────────────────────┐
  Implement the retrieve_and_answer function in rag_server.py.

  Requirements from agents.md:
  - Load SentenceTransformer('all-MiniLM-L6-v2') — reuse if already loaded
  - Load ChromaDB PersistentClient from db_path, get collection 'policy_docs'
  - Embed the query string
  - Query collection for top 3 results, include documents/metadatas/distances
  - Convert L2 distances to similarity: similarity = 1 - (distance / 2)
  - Filter: only keep chunks where similarity >= 0.6
  - If no chunks pass threshold: return the refusal template exactly:
    "This question is not covered in the retrieved policy documents.
     Retrieved chunks: [list sources]. Please contact the relevant department."
  - Build prompt: "Answer using ONLY these retrieved chunks.
    Do not use any information outside the provided context.
    Cite source document and chunk index for every claim."
    Include the chunk texts with their source labels.
  - Import call_llm from llm_adapter (in ../uc-mcp/llm_adapter.py)
  - Call call_llm(prompt) to get the answer
  - Return dict: {answer, cited_chunks: [{doc_name, chunk_index, score}], refused}

  Do not implement any other functions.
└─────────────────────────────────────────────────────────────────┘

[TRAE generates retrieve_and_answer.]

SAY:
  "Before I run the RICE version — I am going to run naive mode first.
   This is the Control step in CRAFT.
   You always see the failure before you claim the fix."

[In Terminal 1:]
  python3 rag_server.py --naive \
    --query "Can I use my personal phone to access work files from home?"

[Show the naive output — cross-document blend]

SAY:
  "There it is. Both policies blended. IT and HR merged into
   one answer that neither document supports.
   Now watch what happens with retrieval."

[In Terminal 1:]
  python3 rag_server.py \
    --query "Can I use my personal phone to access work files from home?"

[Show the RAG output — single source, cited]

SAY:
  "One source. One document. One chunk. Cited.
   The retrieval pipeline returned only the IT policy chunk
   because that is the only chunk that scored above 0.6.
   The HR policy chunks were below threshold — excluded."

[Run two more reference queries:]
  python3 rag_server.py --query "Who approves leave without pay?"
  python3 rag_server.py --query "What is the flexible working culture?"

SAY on the refusal:
  "That last one returns the refusal template.
   'Flexible working culture' is not in any of the three documents.
   No chunk scored above 0.6. The system refused — correctly.
   That refusal template is word-for-word what we put in
   the Enforcement section of agents.md."

⚠️  RECOVERY — if retrieve_and_answer fails:
  Most common: llm_adapter import path wrong.
  Fix in TRAE: change import to use sys.path.insert for ../uc-mcp.
  If Gemini API fails: remove the llm_call — show retrieved chunks only.
  SAY: "The retrieval is working — the generation step needs one fix.
        The chunks are correct. That is the important part."

────────────────────────────────────────────────────────────────────
MIN 24–28  STEP 6 — CRAFT LOOP: FIX ONE FAILURE LIVE
────────────────────────────────────────────────────────────────────

SAY:
  "The RAG pipeline is running. Now I am going to deliberately
   trigger the chunk boundary failure so you can see it —
   and fix it live."

[Show the current chunking — run this:]
  python3 -c "
  from rag_server import chunk_documents
  chunks = chunk_documents('../data/policy-documents')
  # Find clause 5.2
  for c in chunks:
      if 'Department Head' in c['text'] or 'HR Director' in c['text']:
          print(f'[{c[\"doc_name\"]}, chunk {c[\"chunk_index\"]}]:')
          print(c['text'])
          print('---')
  "

SAY:
  "Clause 5.2 says: LWP requires approval from the Department Head
   AND the HR Director. Are both conditions in the same chunk?
   Let me check."

[If they are split — show the room. If they are together — use this as a teaching moment:]

SAY:
  "In this run they happen to be together — but with fixed-size
   chunking that is luck, not engineering. Change the max_tokens
   to 150 and run again — you will see the split."

[Or if already split:]
SAY:
  "The second condition is in the next chunk. If retrieval returns
   chunk 11 but not chunk 12 — the answer will say
   'Department Head' and drop the HR Director requirement.
   That is the condition drop failure.

   Here is the fix in TRAE:"

[In TRAE:]
┌─────────────────────────────────────────────────────────────────┐
  Update chunk_documents in rag_server.py.

  Current issue: multi-condition clauses like clause 5.2
  ("LWP requires approval from the Department Head and the HR Director")
  may be split across chunks.

  Fix: after splitting on sentence boundaries, if a chunk ends with
  a conjunction (and, or, but) or an incomplete condition,
  merge it with the next sentence before starting a new chunk.

  Keep max_tokens at 400. Do not change any other logic.
└─────────────────────────────────────────────────────────────────┘

[Rebuild index with the fix:]
  python3 rag_server.py --build-index
  python3 rag_server.py --query "Who approves leave without pay?"

SAY:
  "Both conditions now in the answer. Department Head AND HR Director.
   That is the Fix step in CRAFT.
   One targeted change. Re-run. Verified."

────────────────────────────────────────────────────────────────────
MIN 28–30  STEP 7 — COMMIT AND HAND OFF
────────────────────────────────────────────────────────────────────

[In Terminal 1:]
  git add uc-rag/agents.md uc-rag/skills.md uc-rag/rag_server.py
  git commit -m "UC-RAG Live demo: sentence-aware chunking + 0.6 threshold + grounding enforcement"

SAY:
  "Commit. The formula: UC-ID, what was built or fixed,
   what the enforcement rules were.

   Here is what just happened in 30 minutes:

   [Point to agents.md] RICE — Enforcement rules defined first.
   [Point to skills.md] Skills — error_handling names the refusal template.
   [Point to rag_server.py] Code — TRAE implemented each function.
   [Point to terminal] CRAFT — naive run showed the failure,
   RAG run showed the fix, one more fix for chunk boundary.
   [Point to git] Track — committed with a meaningful message.

   That file — rag_server.py — is now your reference implementation.
   Your job is to build the same file from the starter,
   using the same workflow, in the next 60 minutes.
   You have stub_rag.py if your implementation gets stuck.
   You have agents.md and skills.md already generated —
   but you should regenerate them yourself and compare.

   Questions before you start?"

[Take 2–3 questions maximum. Then start Block 2 timer.]

═══════════════════════════════════════════════════════════════════
TIMING CHECKPOINTS
═══════════════════════════════════════════════════════════════════

Min 5  — agents.md saved with refined enforcement rule
Min 8  — skills.md saved with specific refusal template in error_handling
Min 13 — chunk_documents running, chunks printed to terminal
Min 18 — build_index complete, ChromaDB populated
Min 24 — retrieve_and_answer running, naive vs RAG contrast shown
Min 28 — one CRAFT fix applied and verified
Min 30 — committed, questions taken

If behind at min 18 (build_index not done):
  Skip the chunk boundary fix (Step 6). Go straight to retrieve_and_answer.
  The naive vs RAG contrast is the most important moment — do not skip it.

If behind at min 24 (retrieve_and_answer not done):
  Switch to stub_rag.py for the naive vs RAG contrast:
    python3 stub_rag.py --naive --query "personal phone work files"
    python3 stub_rag.py --query "personal phone work files"
  SAY: "The stub shows you what a working pipeline produces.
        Your rag_server.py will produce the same output
        once retrieve_and_answer is implemented."

═══════════════════════════════════════════════════════════════════
RECOVERY REFERENCE — PASTE THESE IF TRAE OUTPUT FAILS
═══════════════════════════════════════════════════════════════════

CHUNK_DOCUMENTS — working reference:
──────────────────────────────────────
import os, re

def chunk_documents(docs_dir: str, max_tokens: int = 400) -> list:
    results = []
    for fname in sorted(os.listdir(docs_dir)):
        if not fname.endswith('.txt'):
            continue
        text = open(os.path.join(docs_dir, fname), encoding='utf-8').read()
        sentences = re.split(r'(?<=[.!?])\s+', text.strip())
        chunks, current, count = [], [], 0
        for sentence in sentences:
            words = len(sentence.split())
            if count + words > max_tokens and current:
                chunk_text = ' '.join(current)
                results.append({
                    'doc_name': fname,
                    'chunk_index': len([r for r in results if r['doc_name']==fname]),
                    'text': chunk_text,
                    'id': f'{fname}::chunk_{len([r for r in results if r["doc_name"]==fname])}'
                })
                current, count = [sentence], words
            else:
                current.append(sentence)
                count += words
        if current:
            chunk_text = ' '.join(current)
            results.append({
                'doc_name': fname,
                'chunk_index': len([r for r in results if r['doc_name']==fname]),
                'text': chunk_text,
                'id': f'{fname}::chunk_{len([r for r in results if r["doc_name"]==fname])}'
            })
    return results

BUILD_INDEX — working reference:
──────────────────────────────────────
import chromadb
from sentence_transformers import SentenceTransformer

def build_index(docs_dir: str = '../data/policy-documents',
                db_path: str = './chroma_db'):
    model = SentenceTransformer('all-MiniLM-L6-v2')
    chunks = chunk_documents(docs_dir)
    client = chromadb.PersistentClient(path=db_path)
    try:
        client.delete_collection('policy_docs')
    except:
        pass
    collection = client.create_collection('policy_docs')
    ids   = [c['id']   for c in chunks]
    texts = [c['text'] for c in chunks]
    metas = [{'doc_name': c['doc_name'], 'chunk_index': c['chunk_index']}
             for c in chunks]
    embeddings = model.encode(texts, show_progress_bar=True).tolist()
    collection.add(ids=ids, documents=texts,
                   metadatas=metas, embeddings=embeddings)
    print(f'Index built. {len(chunks)} chunks at {db_path}')

RETRIEVE_AND_ANSWER — working reference:
──────────────────────────────────────
import sys
sys.path.insert(0, '../uc-mcp')

def retrieve_and_answer(query: str, db_path: str = './chroma_db',
                        top_k: int = 3, threshold: float = 0.6) -> dict:
    from llm_adapter import call_llm
    model = SentenceTransformer('all-MiniLM-L6-v2')
    client = chromadb.PersistentClient(path=db_path)
    collection = client.get_collection('policy_docs')
    q_emb = model.encode([query]).tolist()
    results = collection.query(query_embeddings=q_emb, n_results=top_k,
                               include=['documents','metadatas','distances'])
    docs  = results['documents'][0]
    metas = results['metadatas'][0]
    dists = results['distances'][0]
    passing = [(d,m,dist) for d,m,dist in zip(docs,metas,dists)
               if (1 - dist/2) >= threshold]
    if not passing:
        sources = ', '.join(f"{m['doc_name']}::chunk_{m['chunk_index']}"
                            for _,m,_ in zip(docs,metas,dists))
        return {'answer': f'This question is not covered in the retrieved '
                          f'policy documents. Retrieved chunks: {sources}. '
                          f'Please contact the relevant department.',
                'cited_chunks': [], 'refused': True}
    context = '\n\n'.join(
        f"[{m['doc_name']}, chunk {m['chunk_index']}]:\n{d}"
        for d,m,_ in passing)
    prompt = (f'Answer using ONLY the context below. Do not add any '
              f'information outside the provided chunks. Cite the source '
              f'document and chunk index for every claim.\n\n'
              f'Context:\n{context}\n\nQuestion: {query}\n\nAnswer:')
    answer = call_llm(prompt)
    cited = [{'doc_name': m['doc_name'], 'chunk_index': m['chunk_index'],
               'score': round(1 - dist/2, 3)}
             for _,m,dist in passing]
    return {'answer': answer, 'cited_chunks': cited, 'refused': False}

═══════════════════════════════════════════════════════════════════
KEY LINES TO SAY — MEMORISE THESE
═══════════════════════════════════════════════════════════════════

On starting with RICE before code:
  "agents.md before rag_server.py. Always.
   The enforcement rules determine what the code must do.
   The code implements them. Never the other way around."

On the naive vs RAG contrast:
  "Same question. Same documents. Same LLM.
   The only difference is the retrieval pipeline.
   One answer blends two documents. The other cites one chunk.
   That is what you are building."

On the refusal:
  "The system refused. That is not a failure.
   That is the enforcement working exactly as written."

On the chunk boundary fix:
  "One targeted change. Re-run. Verified.
   That is the F in CRAFT. Not a rewrite. Not a guess.
   One change, one test, one commit."

On handing off to participants:
  "I built it in 30 minutes with TRAE and Gemini CLI.
   You have 60 minutes.
   You have agents.md and skills.md to generate.
   You have stub_rag.py if you get stuck.
   The workflow is identical to what you just watched."
