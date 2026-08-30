---
layout: slide
title: "Devocle - RAG for DevOps Engineers"
date: 2026-03-06
category: rag
author:
 - Stephen Oni
 - Opeyemi Bamigbade
excerpt: "A case study in building a fine-tuned RAG system for DevOps log debugging, from dataset curation to Docker Desktop extension."
preview_image: /assets/images/insights/devocle-rag-devops-cover-v2.png
---

# Devocle

### RAG-Powered Debugging for DevOps Engineers


A case study showing how we built a fine-tuned Retrieval-Augmented Generation system to help DevOps engineers debug Nginx issues using StackOverflow data, Qwen2.5-1.5B-Instruct, QLoRA fine-tuning, vLLM serving on Modal, and a Docker Desktop extension interface.

---

## What Problem Are We Solving?

DevOps engineers spend a significant amount of time digging through logs, documentation, and StackOverflow threads to debug infrastructure issues.

**The question we asked:**

> *Can we build a system that takes a DevOps engineer's log error, understands it, and surfaces the most relevant solution - instantly?*

> *more or less how do we provide a template for building a RAG system that can be fine-tuned on domain-specific data, served efficiently, and integrated directly into engineers' workflows?*

---

## What Devocle Does

Devocle is a **Retrieval-Augmented Generation (RAG)** system purpose-built for DevOps engineers.

**Core goals:**
- Accept a natural-language question or a raw log error
- Retrieve the most relevant passages from a curated knowledge base
- Generate a precise, cited, actionable answer

**Where it lives:**
- As a REST API (FastAPI)
- As a **Docker Desktop Extension** - right where engineers already work
- and can also be extended to other observability platforms (Grafana, Kibana) or chat interfaces (Slack, Teams)

---

## Scope: Nginx + StackOverflow

Rather than solving every DevOps problem at once, we scoped tightly.

| Constraint | Choice |
|---|---|
| Domain | **Nginx** - one of the most common web servers / reverse proxies |
| Data source | **StackOverflow** Q&A threads tagged `nginx` |
| Goal | Debug Nginx log errors and config issues |

StackOverflow was ideal: real engineers asking real questions, with accepted answers and vote scores as a built-in quality signal.

---

# Section 1

## Dataset & Base Model

---

## Why StackOverflow Data?

StackOverflow nginx threads are a goldmine for this use case:

- Questions mirror exactly the kind of input a DevOps engineer types
- Accepted answers have been validated by the community
- `score` and `accepted` fields give a natural quality signal we preserve as retrieval metadata

```json
{
  "question_id": 24319662,
  "question_title": "From inside a Docker container, how do I connect to the localhost?",
  "question": "I have an Nginx instance running inside a Docker container...",
  "answer_id": 24326540,
  "answer": "If you are using Docker-for-mac or Docker-for-Windows 18.03+...",
  "accepted": true,
  "score": 4902,
  "question_score": 3561,
  "tags": ["docker", "nginx", "docker-container", "docker-network"],
  "url": "https://stackoverflow.com/questions/24319662"
}
```

Each record has a **question body**, an **answer body**, acceptance status, answer vote score, question score, tags, and a URL - all preserved as metadata during indexing.

---

## Why Alpaca-Style Format?

The StackOverflow JSONL is re-formatted into an **Alpaca-style instruction format** at training time:

```
### Instruction:
Why is nginx returning 502 Bad Gateway when proxying to my Node.js app?

### Response:
A 502 error means nginx received an invalid response from the upstream server.
Check that your Node.js app is actually running on the port nginx is proxying to...
```

**Why this format?**

LLMs learn behaviour from the shape of their training data. A raw "question\nanswer" pair teaches the model to complete text. An instruction-response pair teaches it to **follow a prompt** - to behave like an assistant, not just a text predictor.

Since our base model (`Qwen2.5-1.5B-Instruct`) was already pre-trained with this format, using it during fine-tuning keeps the model's existing instruction-following behaviour intact while injecting domain knowledge.

---

## Why Mix in Alpaca Data?

We blend StackOverflow data with **2,000 samples from the Alpaca general instruction dataset** (`tatsu-lab/alpaca`).

| Data source | Role |
|---|---|
| StackOverflow nginx JSONL | Domain-specific knowledge |
| Alpaca (2 000 samples) | Retain general instruction-following ability |

**Why not train on domain data alone?**

Fine-tuning on a narrow corpus causes **catastrophic forgetting** - the model overwrites its general instruction-following ability with domain-specific patterns. After a few hundred steps on nginx-only data, it may refuse to answer anything outside that domain, or start producing malformed responses.

Mixing in general Alpaca examples acts as a regulariser, anchoring the model's conversational backbone while still shifting it toward DevOps expertise.

---

## The Base Model: Qwen2.5-1.5B-Instruct

We chose **Qwen/Qwen2.5-1.5B-Instruct** as our base model.

**Why Qwen2.5-1.5B?**

- 🪶 **Small enough** to fine-tune on a single GPU (Colab T4 / A10G)
- 🏎️ **Fast inference** - fits within tight latency budgets
- 📜 **Apache 2.0 license** - no token gate, no restrictions
- 💡 **Already instruction-tuned** - strong baseline for instruction-following out of the box
- 🧠 **1.5B parameters** - surprisingly capable for a focused domain like Nginx

```
Base model     : Qwen/Qwen2.5-1.5B-Instruct
Context window : 32 768 tokens (model) - current deployment constrains prompts to 2 048 tokens via vLLM to keep latency predictable
Quantisation   : 4-bit NF4 (QLoRA)
Adapter        : steveoni/qwen25-1.5b-qlora-adapter
```

---

# Section 2

## Fine-tuning: LoRA & QLoRA

---

## What is LoRA - and Why?

Full fine-tuning updates every weight in the model. For a 1.5B-parameter model in FP16 that's ~3 GB of weights, plus optimizer states and gradients - easily 10–15 GB of GPU memory. That rules out free Colab.

**LoRA (Low-Rank Adaptation)** solves this: freeze the original weights and inject small **trainable rank-decomposition matrices** alongside them.

```
Full fine-tune:   W' = W + ΔW         (ΔW has the same shape as W - expensive)

LoRA:             W' = W + A × B      (A and B are low-rank - cheap)
```

If `W` is `(d × d)`, `A` is `(d × r)` and `B` is `(r × d)` where `r << d`.

Trainable parameters drop from $d^2$ to $2 \cdot d \cdot r$ - orders of magnitude fewer.

Think of it as **writing notes in the margins** of a textbook instead of rewriting the whole thing.

---

## What is QLoRA - and Why?

**QLoRA (Quantized LoRA)** goes further: it compresses the frozen base model itself to **4-bit NF4**, so even the weights we're *not* training take far less VRAM.

```
┌─────────────────────────────────┐
│  Base Model (4-bit NF4)         │  ← stored at ~950 MB instead of ~3 GB
│  (frozen - never updated)       │
└─────────────────────────────────┘
          +
┌─────────────────────────────────┐
│  LoRA Adapters (FP16)           │  ← the only thing we actually train
│  (a few MB)                     │
└─────────────────────────────────┘
```

At compute time, weights are dequantized to FP16 on the fly. The math stays accurate; only the storage format is compressed.

> **Result:** fine-tuning a 1.5B model on a free Colab T4 (15 GB VRAM) becomes feasible.

---

## QLoRA Configuration - The Choices

```python
BNB_4BIT_QUANT_TYPE       = "nf4"      # NormalFloat4
BNB_4BIT_USE_DOUBLE_QUANT = True       # quantize the quantization constants too
BNB_COMPUTE_DTYPE         = "float16"  # dequantize to FP16 for compute
```

| Setting | Why |
|---|---|
| `nf4` over `fp4` | NF4 is information-theoretically optimal for normally-distributed weights (neural net weights are approximately normal) |
| Double quantization | Quantizes the quantization constants themselves - saves ~0.4 bits/parameter at no accuracy cost |
| `float16` compute | Compatible with all CUDA GPUs including T4 (sm_75) which does not support bfloat16 |

---

## LoRA Hyperparameters - The Choices

```python
LORA_R       = 8     # rank
LORA_ALPHA   = 16    # scaling: effective contribution = alpha / r = 2×
LORA_DROPOUT = 0.05

LORA_TARGET_MODULES = [
    "q_proj", "k_proj", "v_proj", "o_proj",   # attention projections
    "gate_proj", "up_proj", "down_proj",       # MLP projections
]
```

**Why rank 8?** Rank controls the adapter's capacity. Too low (r=4) and it can't absorb new domain knowledge. Too high (r=32+) and it trains slowly with diminishing returns on a 1.5B model. Rank 8 is the pragmatic sweet spot.

**Why target both attention and MLP?** Domain knowledge (Nginx config syntax, error semantics) lives in both the attention layers *and* the feed-forward MLP layers. Adapting only attention is common but leaves capability on the table for instruction-following tasks.

---

## Training Setup - The Choices

```python
BATCH_SIZE    = 2
GRAD_ACC      = 8       # effective batch = 16
EPOCHS        = 1
LEARNING_RATE = 2e-4
MAX_LENGTH    = 512
EARLY_STOPPING_PATIENCE = 2
VAL_SPLIT     = 0.05    # 5% held out, capped at 500 examples
```

**Why gradient accumulation?** A batch size of 2 fits in VRAM, but produces noisy gradients. Accumulating over 8 steps gives an effective batch of 16 - stable training without OOM.

**Why only 1 epoch?** With curated instruction data, 1 epoch is often enough. More epochs on a small, narrow dataset risk overfitting - the model memorises answers rather than generalising.

**Result:** the adapter trains in ~30 minutes on a Colab T4 and ~10 minutes on an A10G.

---

# Section 3

## Serving: vLLM + Modal

---

## Why vLLM?

After fine-tuning we need efficient inference. We use **vLLM** - a high-throughput LLM serving engine.

**Why vLLM over plain HuggingFace `generate()`?**

- **PagedAttention** - manages the KV cache in virtual memory pages, eliminating memory fragmentation and dramatically increasing throughput under concurrent load
- **LoRA hot-swapping** - serve the base model and a named adapter from a single process; the adapter is selected per-request
- **OpenAI-compatible API** - the RAG server calls it identically to how it would call OpenAI's API, making the LLM backend swappable

```python
cmd = [
    "python", "-m", "vllm.entrypoints.openai.api_server",
    "--model",         CACHE_DIR,                      # base Qwen2.5-1.5B
    "--enable-lora",
    "--lora-modules",  f"finetunedqa={ADAPTER_DIR}",   # adapter loaded by name
    "--max-lora-rank", "8",
    "--dtype",         "float16",
    "--max-model-len", "2048",
]
```

---

## Deploying to Modal

We deploy the vLLM server to **Modal** - a serverless GPU cloud.

```
modal deploy modal_deploy.py
→ https://steveoni--qwen25-vllm-lora-vllmserver-serve.modal.run
```

**Why Modal?**

| Property | Value |
|---|---|
| GPU | A10G (24 GB VRAM) |
| Min containers | 0 - scales to zero when idle |
| Timeout | 600 s |
| Weights | Cached in Modal Volumes (no re-download on cold start) |

Serverless means we pay only when the endpoint is actively serving requests. For a demo / low-traffic system this is dramatically cheaper than a always-on VM.

---

## The Serving Pipeline

```
User query
    │
    ▼
FastAPI (RAG server)
    │
    ├──► ChromaDB  ──► top-k relevant passages
    │
    ▼
Prompt assembly (passages + question)
    │
    ▼
vLLM on Modal  (Qwen2.5-1.5B + LoRA adapter "finetunedqa")
    │
    ▼
Generated answer + citations
```

The RAG server and LLM server are fully decoupled - the RAG system calls the Modal endpoint over HTTP using the OpenAI client, making the LLM backend swappable with zero code changes.

---

# Section 4

## RAG: Indexing, Retrieval & Generation

---

## Why RAG Instead of Pure Fine-Tuning?

Fine-tuning teaches the model *how* to answer in a domain. It does not guarantee factual accuracy on specific questions - parametric memory degrades, and the model can still hallucinate.

**RAG separates knowledge from behaviour:**

```
Without RAG:   User → LLM → Answer  (from training memory - may hallucinate)

With RAG:      User → Retriever → Relevant passages
                                        ↓
                              User + Passages → LLM → Grounded answer
```

Every answer is **traceable to a source document**. When the knowledge base changes (e.g. a new Nginx version), we re-index - no retraining required.

---

## The RAG Workflow

```
                    ┌──────────────────────────────────┐
                    │         INDEXING (offline)        │
                    │                                   │
  stackoverflow     │  JSONL → parse Q&A pairs          │
  _nginx.jsonl ───► │       → split into chunks         │
                    │       → embed (all-MiniLM-L6-v2)  │
                    │       → store in ChromaDB         │
                    └──────────────────────────────────┘

                    ┌──────────────────────────────────┐
                    │         QUERYING (online)         │
                    │                                   │
  User question ──► │  embed query                      │
                    │  → cosine similarity → ChromaDB   │
                    │  → top-3 chunks + metadata        │
                    │  → assemble prompt                │
                    │  → Qwen2.5 on Modal               │
                    │  → answer + citations             │
                    └──────────────────────────────────┘
```

---

## Chunking Strategy - Why It Matters

Each Q&A record is split into a **question passage** and an **answer passage**, then chunked with overlap:

```python
chunk_size    = 800    # characters per chunk
chunk_overlap = 150    # overlap between consecutive chunks
```

**Why overlap?**

```
Chunk 1: [...context A...][ ← overlap → ]
Chunk 2:              [ ← overlap → ][...context B...]
```

A key sentence that falls at a chunk boundary won't be lost - both adjacent chunks carry enough context for the retriever to score them correctly.

**Why 800 characters?**

Our vLLM deployment constrains Qwen2.5-1.5B to a 2,048-token context window (the model natively supports 32,768). With 3 retrieved chunks (~2,400 characters) plus the system prompt and question, we stay comfortably within the limit while maximising the information density per retrieval.

---

## Embeddings & Vector Search - Why These Choices

```python
huggingface_embed_model = "all-MiniLM-L6-v2"
```

**Why `all-MiniLM-L6-v2`?**
- Runs **locally** - no external API, no latency, no cost
- 384-dimensional vectors - small, fast, and ChromaDB-friendly
- Strong semantic similarity on technical QA (benchmarked on BEIR)
- **Same model at index time and query time** - critical: mismatched embeddings produce random retrieval results

**Why ChromaDB?**
- Simple local-persistence mode for development
- Drop-in upgrade to ChromaDB Cloud for production
- Native HuggingFace embedding support
- Default deployment points to ChromaDB Cloud today, with a single env toggle to run the exact same stack locally when desired

---

## The Prompt - Why Restrictive?

```
You are a helpful assistant specialised in Nginx, web servers, and DevOps.
Use ONLY the CONTEXT PASSAGES below to answer the USER QUESTION.
If the context does not contain enough information, say so clearly.

CONTEXT PASSAGES:
[ANSWER] Nginx 502 Bad Gateway when proxying to Node.js (score=4902)
url: https://stackoverflow.com/questions/24319662
<passage text>

USER QUESTION:
Why is nginx returning 502?

Answer:
```

The `Use ONLY the CONTEXT PASSAGES` constraint is deliberate. Without it, the fine-tuned model will blend retrieved context with its parametric memory - producing confident but untraceable answers. Hard grounding keeps every claim auditable.

---

## Structured JSON Response with Citations

The `/query/json` endpoint returns a typed Pydantic schema:

```json
{
  "answer": "A 502 means nginx can't reach the upstream. Check the upstream is running...",
  "citations": [
    {
      "question_id": "24319662",
      "question_title": "From inside a Docker container, how do I connect to localhost?",
      "url": "https://stackoverflow.com/questions/24319662",
      "passage_type": "answer",
      "quote": "Use --network=host in your docker run command..."
    }
  ],
  "confidence": 0.91,
  "recommended_next_actions": [
    "Check upstream server status",
    "Review nginx error_log for upstream connect() errors"
  ]
}
```

`confidence` is a top-level score on the whole answer. Citations carry a `quote` - an exact excerpt that lets the engineer verify the source in one click.

---

# Section 5

## Docker Desktop Extension

---

## Why a Docker Desktop Extension?

DevOps engineers already live in Docker Desktop. Rather than asking them to switch to a browser tab or CLI tool, we brought Devocle **directly into their workflow**.

The extension registers a **dashboard tab** - a single-page HTML/CSS/JS UI, zero additional installs required:

```json
{
  "schema": "0.3.0",
  "version": "1.0.0",
  "title": "Devocle",
  "vendor": "Brassin",
  "description": "Devocle - ask any Nginx / DevOps question. Answers sourced from StackOverflow and generated by Qwen2.5 fine-tuned on Modal.",
  "ui": {
    "dashboard-tab": {
      "title": "Devocle",
      "root": "/ui",
      "src": "index.html"
    }
  }
}
```

---

## How It All Fits Together

```
┌──────────────────────────────────────────────────────────────┐
│  Docker Desktop                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Devocle Extension Tab                                 │  │
│  │  "Why is nginx returning 502 on /api?"                 │  │
│  └────────────────────┬───────────────────────────────────┘  │
└───────────────────────┼──────────────────────────────────────┘
                        │ HTTP POST /query/json
                        ▼
            ┌───────────────────────┐
            │  FastAPI RAG Server   │
            │  (devocle-rag)        │
            └─────────┬─────────────┘
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
  ┌───────────────┐    ┌──────────────────────┐
  │   ChromaDB    │    │  Qwen2.5-1.5B + LoRA  │
  │  (SO nginx    │    │  via vLLM on Modal    │
  │   vectors)    │    │  (OpenAI-compat API)  │
  └───────────────┘    └──────────────────────┘
```

---

# Summary

---

## What We Built

| Component | Technology |
|---|---|
| Dataset | StackOverflow nginx JSONL (+ 2 000 Alpaca samples) |
| Base model | Qwen2.5-1.5B-Instruct (Apache 2.0) |
| Fine-tuning | QLoRA - 4-bit NF4, rank 8, bitsandbytes |
| Adapter | PEFT LoRA → HuggingFace Hub (`steveoni/qwen25-1.5b-qlora-adapter`) |
| Serving | vLLM with LoRA hot-swap, OpenAI-compatible API |
| Cloud deploy | Modal (A10G, serverless, scales to zero) |
| Embeddings | `all-MiniLM-L6-v2` (local, no API key) |
| Vector DB | ChromaDB (local dev → ChromaDB Cloud) |
| Chunking | RecursiveCharacterTextSplitter - 800 chars / 150 overlap |
| API | FastAPI - `/query` (text) + `/query/json` (structured + citations) |
| Interface | Docker Desktop Extension |

---

## Key Design Decisions

- **Small model (1.5B)** → fine-tunable on free Colab T4; deployable cheaply on Modal A10G
- **QLoRA** → full fine-tune quality at a fraction of the VRAM cost
- **Alpaca mix** → prevents catastrophic forgetting of instruction-following ability
- **RAG over pure fine-tuning** → answers are grounded in source documents, cited, and updatable without retraining
- **Restrictive prompt** → forces the model to cite context rather than hallucinate from memory
- **OpenAI-compatible API** → the LLM backend is swappable with zero code changes in the RAG layer
- **Docker Extension** → zero friction for DevOps engineers already in Docker Desktop

---

# Thank You!

Access the code here [Devocle](https://github.com/Brassin/Devocle)
