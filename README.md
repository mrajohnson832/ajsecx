# ajsecx: Designing a Grounded, Local Pentest Assistant (Case Study)

A design case study for **ajsecx**, a local-first assistant I conceived and directed to guide
authorized web-application penetration testing through curated playbooks, tool orchestration,
and report drafting. This write-up covers the problem, the design decisions, and the
architecture. The source is kept private (see the note at the end); this document is about the
*system design and the reasoning behind it*, not the code.

- **My role:** product design and architecture, security design decisions, infrastructure and
  deployment. Implementation was AI-assisted (I directed an AI coding tool against my
  requirements and design).
- **Stack:** FastAPI (server-rendered), SQLite, a local LLM served over an OpenAI-compatible
  endpoint, file-based retrieval over a curated Markdown knowledge base.
- **Status:** private project; this is a conceptual case study only.

---

## The problem
Learning and running web-app testing means constantly jumping between a methodology reference,
a pile of CLI tools, and a blank page for the eventual report. I also saw a specific failure
mode when I tried using a general LLM for help: it would **confidently hallucinate
vulnerabilities** that did not exist, which is worse than no help at all, because chasing a
fake finding wastes time and erodes trust.

I wanted a single local tool that would (1) walk me through a sound, repeatable methodology,
(2) launch the supporting tools, and (3) draft the finding report, while being **structurally
incapable of inventing vulnerabilities**.

## Design principles (the decisions that matter)
These are the calls I made, and the reasoning behind them:

- **The LLM advises and drafts; it never "finds" bugs.** This is the core security-design
  decision. The model is scoped to explain methodology and rewrite my own capture notes into a
  report. It is never the thing that decides whether a vulnerability exists. That judgment stays
  with me and with deterministic playbooks, which removes the hallucinated-vulnerability failure
  mode by design rather than by hoping the model behaves.
- **Grounded, cited answers only.** The advisor answers strictly from a curated knowledge base
  and cites its sources, and says so when the answer is not covered, rather than drawing on the
  model's training data.
- **Curated, not generated, playbooks.** The decision trees are hand-authored and version-
  controlled, so guidance is deterministic and reviewable, not improvised per run.
- **Local and private.** It runs entirely on infrastructure I control, with a local model, so no
  engagement data leaves the environment.

## Architecture (conceptual)
```
  Curated Markdown knowledge base        Hand-authored playbooks (decision trees)
  (methodology, per-topic notes)                      |
                 |                                    v
                 v                          Guided, step-by-step run
        Retrieval (RAG): pull the            (captures notes + evidence)
        most relevant chunks at query time             |
                 |                                     v
                 v                            SQLite: runs, steps, findings
     Grounded prompt (sources + question)              |
                 |                                     v
                 v                            Report drafter (LLM rewrites
        Advisor (LLM answers from the                 MY captured notes into a
        provided sources, with citations)             HackerOne-shaped report)

        Tool orchestration: launch CLI tools (recon, cracking, etc.) as background jobs,
        parse their output into in-app findings.
```

## How the grounding works (RAG)
Retrieval-Augmented Generation means the model does not answer from its trained-in memory. At
query time the system **retrieves** the most relevant text from my curated knowledge base and
places it into the prompt, then the model **generates** an answer from that provided text and
cites it. It is an open-book approach: the model answers with the trusted material in front of
it instead of from memory, which is what keeps the advisor accurate and prevents it from
inventing specifics. ajsecx uses straightforward keyword/section retrieval rather than an
embedding model, which keeps it simple, fast, and fully local.

## Infrastructure and deployment
I provisioned a dedicated Ubuntu VM to host the application and the local model, and configured
the firewall rules to allow only the access the build and the app required. Keeping the model
and all data local was a deliberate privacy decision for a tool meant to touch engagement
material.

## How it was built
I acted as the architect and director: I defined the requirements, made the design and
security decisions above, and drove an AI coding tool to implement them iteratively, reviewing
and steering the output to match the intended behavior and look. I also built and secured the
infrastructure it runs on. I consider directing AI tooling effectively to be part of the skill
set, and I am transparent that the implementation was AI-assisted.

## What this project demonstrates
- **Security-design judgment:** recognizing the hallucinated-vulnerability failure mode and
  designing it out by scoping the LLM to advisory-only, with human/deterministic decisioning.
- **Systems thinking:** grounding via retrieval, deterministic playbooks, structured capture,
  and templated reporting composed into one coherent workflow.
- **AI orchestration:** translating a clear product and security vision into a working system
  with AI-assisted implementation.
- **Infrastructure and deployment:** self-hosting on infrastructure I provisioned and secured.

## A note on code visibility
The source is intentionally private. The tool wraps capable, already-public tooling, and while
its distinctive value is the curated methodology and the grounding design rather than any novel
attack capability, I prefer to keep the full build to myself for now. This case study is meant
to convey the design and the reasoning without publishing the implementation.
