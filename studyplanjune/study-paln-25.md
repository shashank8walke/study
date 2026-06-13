# 25-Day Pro Prep Plan — SWE / AI / ML / DS

**Owner:** Sk (Shashank)
**Start:** ~mid-June 2026 | **Graduated:** May 9, 2026
**Goal:** Be interview-ready and *practically* strong across SWE, AI, ML, DS — not just theory. Apply daily.

---

## How this plan works

**Daily mandatory tracks (every single day):**
- **(1) System Design LLD** — design + actually code it
- **(2) System Design HLD**
- **(3) LeetCode** (you're ~60% through NeetCode)
- **(5) C++** (basic → advanced, memory, niches)
- **(6) Python** (advanced + idioms/coding standards)
- **(9) Applications** (apply + outreach + track)

**Rotating track (pick ONE per day, cycle through):**
- **(4) AI** / **(7) ML** / **(8) DS** — rotate A→B→C so each hits ~every 3rd day

**Coding standards & STAR** are folded into LLD/Python (4 specific weak spots you named: enums, naming, readability, STAR flow). I've flagged these explicitly on the days they appear.

**Time budget:**
- Days 1–10: **9 hrs/day**
- Days 11–25: **7 hrs/day**

**9-hr day allocation (suggested):** LLD 1.5 · HLD 1.0 · LeetCode 1.5 · C++ 1.5 · Python 1.0 · AI/ML/DS 1.5 · Apps 1.0
**7-hr day allocation:** LLD 1.25 · HLD 0.75 · LeetCode 1.25 · C++ 1.0 · Python 0.75 · AI/ML/DS 1.25 · Apps 0.75

**Practical rule:** every concept gets *code you write yourself first*, then Claude Code for cleanup/insight. You explicitly lack hands-on (wrote functions in big files, never owned design). So: LLD days = you build a runnable mini-system; C++ days = compile and run; Python days = lint + type-check your own code.

---

## Legend for columns
- **Day** — day number + theme
- **1 LLD / 2 HLD / 3 LC / 5 C++ / 6 Py / [4/7/8] AI·ML·DS / 9 Apps** — what to do, where, practical task
- **Resources** — primary sources
- **Done** — Yes/No
- **Sk's Notes** — my remarks / pitfalls / what to emphasize

---

## MASTER TABLE — Days 1–10 (9 hrs/day)

| Day | 1 · LLD | 2 · HLD | 3 · LeetCode | 5 · C++ | 6 · Python | Rotating (4 AI / 7 ML / 8 DS) | 9 · Applications | Resources | Done | Sk's Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| **D1** — Foundations + OOP | **OOP design principles** (SOLID). Code: design a `ParkingLot` skeleton — classes, enums for `VehicleType`, interfaces. Run it. | **HLD basics**: client-server, latency vs throughput, vertical vs horizontal scaling, load balancer role. | 5 problems: Arrays/Hashing review (you know these — speed run). | **Refresher**: compile/run, references vs pointers, `const` correctness, stack vs heap. Write 3 tiny programs. | **Naming + readability**: snake_case, PEP8, type hints, `enum.Enum`, dataclasses. Rewrite an old messy function of yours cleanly. | **(4) AI basics**: what a transformer is (attention intuition), tokens, embeddings. Watch + write 1-page summary. | Set up tracker sheet. Apply to **5** roles. Define your 3 target buckets (SWE/AI-ML/DS). | Grokking LLD (educative), SOLID (refactoring.guru), NeetCode, *Effective C++* It.1–4, PEP8/PEP484, 3Blue1Brown attention, jobright.ai | No | **This is your #2 PaloAlto weakness — naming/enums/readability. Make Python clean code a daily habit, not a D1 event.** |
| **D2** — LLD core + Apply engine | **Design patterns** (Strategy, Factory, Observer, Singleton). Code: implement Strategy + Factory in Python with enums. | **HLD**: caching (cache-aside, write-through), CDN, Redis use-cases. | 5: Two Pointers + Sliding Window. Write your own first, then ask me to find bugs + idiomatic cleanup. | **Smart pointers I**: `unique_ptr`, ownership, RAII. Code: a class managing a resource via RAII. | **Idioms**: comprehensions, generators, `with`, context managers. Build a context manager yourself. | **(7) ML basics**: bias-variance, train/val/test, overfitting, regularization (L1/L2). | Apply **5**. Start recruiter outreach: 3 LinkedIn connect-with-note (280–300 char). | Refactoring.guru patterns, Gaurav Sen HLD, *Effective Modern C++* It.18–22, Real Python context managers, ISLR Ch.2 | No | **STAR prep starts now: pick 6 stories from Veritas/NetApp. Write them in strict S-T-A-R bullets — your #4 weakness.** |
| **D3** — LLD practice + Concurrency | **Design: Elevator System** end-to-end. Code a working simulation (state machine, request queue). | **HLD**: databases — SQL vs NoSQL, sharding, replication, CAP theorem. | 5: Stack + Binary Search (you've done some — target weak spots). | **Smart pointers II**: `shared_ptr`, `weak_ptr`, ref counting, cycle problem. Code: demonstrate a cycle + fix with weak_ptr. | **Concurrency**: threading vs multiprocessing vs asyncio, GIL. Write an asyncio fetcher. | **(8) DS basics**: stats — distributions, mean/median/var, CLT, p-values, hypothesis testing. | Apply **5**. Email 3 recruiters via Apollo (only after applying). | NeetCode, *Effective Modern C++* It.19–21, Real Python asyncio, Khan/StatQuest stats, Grokking | No | **weak_ptr/shared_ptr cycle is a classic C++ interview Q — make sure you can draw it and code it.** |
| **D4** — LLD + AI depth | **Design: Rate Limiter (LLD)**. Code: token bucket + sliding window log, both runnable, with tests. | **HLD**: rate limiting at scale, API gateway, idempotency. | 5: Linked List + Trees (start). | **Memory management**: new/delete, memory leaks, valgrind concept, move semantics intro. | **Testing**: pytest, fixtures, mocking, parametrize. Write tests for your D3 asyncio code. | **(4) AI**: **MCP** (Model Context Protocol) — what it is, client/server, tools/resources. Build a tiny MCP server with Claude Code. | Apply **5**. Follow up on any responses. | *Effective Modern C++* move semantics, pytest docs, **modelcontextprotocol.io**, Anthropic MCP docs | No | **This directly fixes your Goldman gap (MCP, Claude Code vs Cursor/VSCode). Be able to explain MCP in 2 min + why agentic tooling matters.** |
| **D5** — LLD + ML | **Design: LRU/LFU Cache (LLD)**. Code both from scratch (HashMap + DLL). This is also a LC-hard. | **HLD**: message queues (Kafka/RabbitMQ), pub-sub, event-driven arch. | 5: Trees (BFS/DFS, traversals). | **Templates & STL**: vector, map, unordered_map internals, iterators. | **Decorators + functools**: write 2 decorators (timing, retry). | **(7) ML**: linear/logistic regression from scratch (numpy), gradient descent. Code it. | Apply **5**. Tailor 1 resume variant for top target. | LC LRU, STL ref (cppreference), Real Python decorators, ISLR Ch.3–4, Andrew Ng | No | **Coding cache from scratch shows real LLD skill — NetApp-style. Explain time complexity of every op.** |
| **D6** — LLD + DS | **Design: Notification Service (LLD)** — multi-channel (email/SMS/push) using Strategy + Observer. Code it. | **HLD**: design Twitter feed (fan-out on write vs read). | 5: Tries + Heaps. | **C++ niches**: RVO, copy elision, rule of 0/3/5, lvalue/rvalue. | **Type system**: typing module, Protocols, generics, mypy. Type-check a module. | **(8) DS**: pandas deep — groupby, merge, pivot, window funcs. Real dataset EDA. | Apply **5**. Audit which portals convert best for you. | cppreference, mypy docs, pandas docs, Kaggle dataset, Grokking | No | **rule of 3/5/0 + RVO are advanced C++ that trading/NetApp-type interviewers probe. Know them cold.** |
| **D7** — LLD + AI | **Design: Parking Lot (full)** — revisit D1, now complete with pricing, multiple floors, tests. | **HLD**: design URL shortener (hashing, DB, scale, analytics). | 5: Backtracking. | **Exception safety**, noexcept, smart use of `auto`, structured bindings. | **Performance**: profiling (cProfile), Big-O in practice, when C extensions help. | **(4) AI**: **Fine-tuning** — full FT vs LoRA/QLoRA, PEFT, when to use which. Run a tiny LoRA via Claude Code or HF. | Apply **5**. Send 3 more recruiter notes. | HF PEFT docs, LoRA paper (intuition), System Design Primer, cppreference | No | **You wanted depth on fine-tuning. Be able to compare LoRA vs full FT on cost/quality/use-case verbally.** |
| **D8** — LLD + ML | **Design: Vending Machine (state pattern)**. Code state machine with enums + transitions + tests. | **HLD**: design a chat app (WebSocket, presence, message delivery). | 5: 1-D Dynamic Programming. | **Move semantics deep**: perfect forwarding, `std::move`, `std::forward`. Code examples. | **Packaging**: modules, virtualenv, pyproject, clean project structure. | **(7) ML**: trees → random forest → **gradient boosting** (XGBoost/LightGBM). You used these at NetApp — go deep on how they work. | Apply **5**. Review tracker; identify stalled apps. | *Eff. Modern C++* forwarding, XGBoost docs, StatQuest GB, System Design Primer | No | **You have real LightGBM/XGBoost experience — make it an interview strength. Know split-finding, regularization params.** |
| **D9** — LLD + DS | **Design: Splitwise (LLD)** — balances, simplify debts. Code with clean models + tests. | **HLD**: design a news feed / Instagram (media storage, CDN, ranking). | 5: 2-D DP. | **Concurrency in C++**: std::thread, mutex, atomic, condition_variable. | **Data engineering basics**: SQL in Python, SQLAlchemy, batch processing. | **(8) DS**: SQL deep — window functions, CTEs, ranking, joins. Solve 15 SQL problems. | Apply **5**. Polish LinkedIn (keywords for all 3 buckets). | LC SQL / StrataScratch, cppreference threads, System Design Primer, Grokking | No | **DS/analytics interviews (Meta, McKinsey) are SQL-heavy. StrataScratch + DataLemur are your friends.** |
| **D10** — LLD + AI + Review | **Design: Tic-Tac-Toe / Chess board (LLD)** — clean abstractions, extensible. Code it. | **HLD**: design YouTube/Netflix (video pipeline, transcoding, CDN). **Review all HLD so far.** | 5: review weak categories from D1–9. | **C++ review**: smart ptrs, move, memory — solve 3 tricky snippets. | **Review**: clean-code checklist on all your week-1 code. | **(4) AI**: **Inference optimization** — quantization (INT8/INT4), pruning, distillation, KV-cache, batching. | Apply **5**. **Week-1 retro**: response rate, fix funnel. | Grokking review, System Design Primer, llama.cpp/quantization blogs, HF optimization | No | **Inference opt = your NetApp/edge angle (you did INT8 TorchAO + distillation in coursework). Tie it to your GNN project.** |

---

## MASTER TABLE — Days 11–25 (7 hrs/day)

| Day | 1 · LLD | 2 · HLD | 3 · LeetCode | 5 · C++ | 6 · Python | Rotating (4 AI / 7 ML / 8 DS) | 9 · Applications | Resources | Done | Sk's Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| **D11** — ML | **Design: Logging framework (LLD)**. Code with levels (enum), handlers, formatters. | **HLD**: design distributed cache (consistent hashing). | 4: Graphs (BFS/DFS, islands). You're strong here — keep sharp. | **CMake + build systems**, linking, headers vs source, ODR. | **CLI tools**: argparse/click, build a real CLI. (Ties to your CloudForge project.) | **(7) ML**: feature engineering, encoding, scaling, pipelines (sklearn Pipeline). Build one. | Apply **5**. Try a referral request (2 alumni). | sklearn pipelines, CMake tutorial, System Design Primer | No | **Referrals convert 5–10× cold apply. Use NCSU alumni on LinkedIn — short specific notes.** |
| **D12** — DS | **Design: Library Management (LLD)** — you built this in Spring; redo clean in Python. | **HLD**: design a payment system (idempotency, consistency, ledger). | 4: Advanced graphs (Dijkstra, Union-Find). | **Debugging C++**: gdb basics, sanitizers (ASan/UBSan). | **asyncio advanced**: tasks, gather, semaphores, real concurrent pipeline. | **(8) DS**: A/B testing — design, power, significance, common pitfalls. | Apply **5**. Follow up week-1 apps (Day-3 of sequence). | gdb/ASan docs, Trustworthy A/B Testing (Kohavi intuition), Grokking | No | **A/B testing is THE Meta/Google DS interview topic. Know p-hacking, peeking, novelty effects.** |
| **D13** — AI | **Design: Search autocomplete (LLD)** — Trie-backed, ranked. Code it. | **HLD**: design a recommendation system (candidate gen → ranking). | 4: Intervals + Greedy. | **C++ STL algorithms**: `<algorithm>`, lambdas, `std::function`, ranges (C++20). | **functools/itertools mastery**: lru_cache, partial, chain, groupby. | **(4) AI**: **Agentic workflows** — ReAct, tool use, planning, multi-agent. Build a small agent with Claude Code. | Apply **5**. Refresh resume bullets with metrics. | LangChain/agent docs, ReAct paper intuition, cppreference algorithms | No | **Agentic + MCP together = your modern-AI story for Goldman-type gaps. Build something demoable.** |
| **D14** — ML | **Design: File system (LLD)** — directories/files tree, ops. Code it. | **HLD**: design Google Drive/Dropbox (sync, chunking, dedup). | 4: Math & bit manipulation. | **Optimization in C++**: cache locality, branch prediction, `inline`, when to optimize. | **NumPy deep**: vectorization, broadcasting, strides, memory layout. | **(7) ML**: model evaluation — ROC/AUC, PR curve, confusion matrix, calibration, cross-val. | Apply **5**. Identify 10 dream-company targets, research each. | NumPy internals, StatQuest ROC, System Design Primer | No | **NetApp & systems roles love cache-locality talk. For ML, never confuse ROC-AUC with PR-AUC on imbalanced data.** |
| **D15** — DS | **Design: Snake & Ladder / board game (LLD)**. Code clean + extensible. | **HLD**: design a ride-sharing service (Uber) — matching, geo, surge. | 4: review + 2 hard mixed. | **C++ project**: build a small multi-file project with CMake + tests (Catch2/gtest). | **Pandas → Polars**, large data, chunking, dtypes optimization. | **(8) DS**: regression analysis, multicollinearity, feature importance, SHAP. | Apply **5**. **Mid-point retro** — adjust strategy. | gtest, Polars docs, SHAP docs, Grokking | No | **Multi-file C++ + tests = the hands-on you said you lacked. This day matters most for C++ confidence.** |
| **D16** — AI | **Design: Meeting scheduler (LLD)** — conflicts, rooms. Code with tests. | **HLD**: design WhatsApp (E2E msg, delivery, groups, scale). | 4: company-tagged set (pick a target company on LC). | **Modern C++ (C++17/20)**: optional, variant, concepts, ranges, coroutines intro. | **Pydantic + data validation**, building robust APIs (FastAPI intro). | **(4) AI**: **Local & small models** — llama.cpp, Ollama, GGUF, quant tradeoffs, when small models win. Run one locally. | Apply **5**. Send 3 recruiter DMs. | Ollama, llama.cpp, FastAPI/Pydantic docs, cppreference C++20 | No | **Run a 7B/3B model locally and benchmark tokens/sec at INT4 vs FP16 — concrete talking point on speed vs accuracy.** |
| **D17** — ML | **Design: ATM (LLD)** — state, transactions, card auth flow. Code it. | **HLD**: design a distributed job scheduler / cron. | 4: DP review (you tend to be weaker here — drill). | **C++ design patterns** (Singleton, Factory, Observer in C++ with smart ptrs). | **Design patterns in Python** (revisit D2 in Python idioms — duck typing changes things). | **(7) ML**: clustering (k-means, DBSCAN, hierarchical), dimensionality reduction (PCA, t-SNE, UMAP). | Apply **5**. Quality > quantity: 3 highly-tailored apps. | StatQuest clustering/PCA, cppreference, Grokking | No | **DP is the most common LC weak spot — 1 focused DP day now pays off. Patterns in 2 langs deepens understanding.** |
| **D18** — DS | **Design: Hotel booking (LLD)** — availability, overlapping reservations. Code it. | **HLD**: design a stock exchange / order matching engine (low-latency). | 4: hard mixed (graphs + DP). | **C++ for trading**: low-latency idioms, lock-free intro, memory pools, `std::atomic`. | **Profiling & optimization**: line_profiler, memory_profiler, Cython intro. | **(8) DS**: time series (trend, seasonality, ARIMA intuition, forecasting metrics). | Apply **5**. Research trading/quant firms if interested (Citadel, Jane St, HRT). | Lock-free intro talks, ARIMA/StatQuest, System Design Primer | No | **Order matching + low-latency C++ = HFT interview gold. Even if not applying there, it's elite C++ signal.** |
| **D19** — AI | **Design: Online code editor / collaborative doc (LLD)** — operational transform intro. | **HLD**: design a collaborative editor (Google Docs) — OT/CRDT. | 4: backtracking + recursion drill. | **C++ memory model**: happens-before, memory_order, when atomics aren't enough. | **Metaclasses & descriptors** (advanced Python internals). | **(4) AI**: **Transformers deep** — attention math, multi-head, positional encoding, encoder vs decoder, KV cache revisited. | Apply **5**. Follow-ups + thank-you notes to any interviewers. | Illustrated Transformer, Attention paper, cppreference memory model | No | **Be able to explain self-attention with the Q/K/V math, not just hand-wave. This separates you at AI companies.** |
| **D20** — ML | **Design: Distributed key-value store (LLD pieces)** — code a simple consistent-hash ring. | **HLD**: design a search engine (crawl, index, rank). | 4: review all weak categories. | **C++ review + mock**: solve a timed C++ problem, explain memory. | **Python mock**: timed clean-code problem, you narrate naming/readability choices. | **(7) ML**: **deep learning** — CNNs, RNNs/LSTMs, basic PyTorch training loop. Write a training loop. | Apply **5**. **Week-2 retro.** | PyTorch tutorials, System Design Primer, Grokking | No | **Narrating your code choices out loud fixes the PaloAlto gap. Practice talking while coding.** |
| **D21** — DS | **Design: Twitter (full LLD)** — tweet, follow, timeline models. Code it. | **HLD**: design Twitter (full) — tie LLD + HLD together. | 4: 2 mediums + 1 hard, timed. | **C++ mock interview** (use Claude Code as interviewer / self-record). | **Python**: build a mini end-to-end project (CLI + API + tests). | **(8) DS**: causal inference, experiment design, business metrics framing (DAU/MAU, retention). | Apply **5**. Mock behavioral with STAR (record yourself). | System Design Primer, StatQuest, Grokking | No | **Behavioral mock today — strict STAR. Record, watch, cut filler. Your #4 gap closes with reps.** |
| **D22** — AI | **Design: Distributed logging/metrics (LLD)** — aggregation, code a collector. | **HLD**: design a monitoring system (Prometheus-like, metrics, alerting). | 4: company-tagged hard set. | **C++ advanced**: template metaprogramming intro, SFINAE, concepts, CRTP. | **Python C-extensions / performance**: when & how to drop to C, ctypes/cffi. | **(4) AI**: **RAG** — embeddings, vector DBs, chunking, retrieval, eval. Build a RAG pipeline. | Apply **5**. Tailored apps to dream companies (from D14 list). | RAG tutorials, vector DB docs (Chroma/FAISS), cppreference templates | No | **RAG is the most-asked applied-AI build. Pair with MCP/agents for a complete modern-AI portfolio story.** |
| **D23** — ML | **Design: E-commerce cart & order (LLD)** — inventory, checkout. Code it. | **HLD**: design Amazon (catalog, cart, order, inventory, payment). | 4: review + speed (target sub-times). | **C++ full mock** — LLD in C++ with memory reasoning, timed. | **Python full mock** — design + clean code, timed, narrated. | **(7) ML**: ML system design — feature store, training/serving skew, model monitoring, drift. | Apply **5**. Interview-pipeline push: confirm any scheduled rounds. | ML System Design (Chip Huyen intuition), System Design Primer | No | **ML system design is its own interview at Meta/Google. Training-serving skew & drift are must-know terms.** |
| **D24** — DS | **Design: Polished portfolio LLD** — pick your best design, make it demo-clean. | **HLD**: 2 rapid-fire HLD designs in 30 min each (interview pace). | 4: mixed mock set, timed, no hints. | **C++ rapid review**: 5 conceptual Qs (smart ptrs, move, memory, virtual, vtable). | **Python rapid review**: idioms, GIL, async, decorators — explain each aloud. | **(8) DS**: full DS case — frame a business problem end-to-end (metric → analysis → recommendation). | Apply **5**. Polish all materials, prep references. | StatQuest, System Design Primer, Grokking | No | **vtable/virtual dispatch is a near-guaranteed C++ Q. DS case framing is the McKinsey/Meta closer.** |
| **D25** — Full Mock | **LLD mock**: pick a random design, 45 min, code + explain. | **HLD mock**: random system, 45 min, whiteboard-style. | 4: full timed mock (1 easy, 2 med, 1 hard). | **C++ Q&A** mock — rapid fire. | **Python Q&A** mock — rapid fire. | **(4 AI)**: explain MCP, agents, fine-tuning, inference opt, transformers — out loud, 2 min each. | Apply **5**. **Full retro + 30-day forward plan.** | All above | No | **Final day = simulate a real onsite loop. Whatever feels shaky, that's your next-cycle focus.** |

---

## Rotating-track coverage check (so nothing is missed)

**AI (4)** hits: D1, D4, D7, D10, D13, D16, D19, D22, D25 → basics, MCP, fine-tuning/LoRA, inference opt (quant/prune/distill), agentic, local/small models, transformers deep, RAG, final review. **Full coverage of your list.**

**ML (7)** hits: D2, D5, D8, D11, D14, D17, D20, D23 → bias-variance, regression from scratch, boosting, feature eng, evaluation, clustering/PCA, deep learning, ML system design. **Big-company ML loop covered.**

**DS (8)** hits: D3, D6, D9, D12, D15, D18, D21, D24 → stats, pandas, SQL, A/B testing, regression/SHAP, time series, causal inference, business case. **Meta/Google/McKinsey DS loop covered.**

---

## Standing weak-spot fixes (woven into the plan)

1. **Goldman — modern AI (MCP, Claude Code vs Cursor/VSCode):** D4 (MCP), D13 (agents), D16 (local models), D22 (RAG). Build demoable artifacts, not just notes.
2. **Palo Alto — coding standards (enums, naming, readability, snake_case):** baked into **every Python day** + explicit on D1, D17, D20. Narrate choices aloud.
3. **NetApp — system design LLD/HLD + explaining it:** LLD + HLD **every day**, with you *coding* the LLD. Explain-aloud on D20–D25.
4. **Behavioral — strict STAR:** start D2 (write 6 stories), mock D21 (recorded). Keep one STAR doc updated.

---

## Applications strategy (track 9) — beyond jobright.ai

**Portals & where to apply:**
- **Direct company career pages** (highest signal — apply here even if you found it elsewhere).
- **LinkedIn** — but apply on company site, then "Easy Apply" only as backup.
- **Wellfound (AngelList)** — startups, your Raleigh-startup focus (Cloneable/Pendo/Relay/Levitate).
- **Levels.fyi job board, Hiring.cafe, Simplify.jobs** — aggregators with good filters.
- **jobright.ai** — keep, but treat as discovery, not primary apply.
- **Otta / Trueup / builtin** — quality startup + tech listings.

**Daily cadence (5 apps/day = 125 over 25 days):**
- 3 tailored (resume + 1-line note), 2 quick-apply.
- Match resume variant to bucket (you already have SWE/AI-ML/DS variants).

**Outreach sequence (your staggered Day 1–3 strategy, formalized):**
- **Day 0:** Apply on company site.
- **Day 1:** LinkedIn connect w/ note to recruiter or hiring manager (280–300 char, mention you applied + 1 specific hook).
- **Day 2:** If no LinkedIn response, Apollo email (short, specific, role + 1 metric).
- **Day 3:** Light follow-up / move on.

**Referrals (highest ROI — add this):**
- NCSU alumni on LinkedIn at target companies. Short note: who you are, the exact role/req ID, why you fit in 1 line, ask if they'd refer.
- Aim for **2 referral asks/day** on dream companies.

**Tracking:** one sheet — Company · Role · Bucket · Date applied · Portal · Outreach stage · Status · Next action. Review every retro day (D10, D15, D20, D25).

---

## Daily ritual (repeat every day)
1. 10-min plan: read today's row, set the 3 LLD/Python/AI-ML-DS practical targets.
2. Code-first: write your own solution before asking me for cleanup.
3. End-of-day: mark Done column, jot 1 thing you struggled with (that's tomorrow's warm-up).
4. STAR doc: if you used any past project today, add/refine a story.

---

## 📋 PROMPT TO USE EACH DAY

> Copy this, replace `DAY X` with the day number, and paste the matching table row beneath it.

```
I'm working through my 25-day prep plan. Give me the detailed breakdown for DAY X.

Here is the row from my master README:
[PASTE DAY X ROW HERE — all columns]

For this day, give me:
1. For LLD: the exact design problem, the classes/enums/interfaces to define, and the full runnable code I should aim to write (or scaffold it so I fill gaps). Include where I'll likely go wrong.
2. For HLD: the design walkthrough, key components, tradeoffs, and 2-3 follow-up questions an interviewer would ask + how to answer.
3. For LeetCode: 5 specific problems (names + numbers) matching the day's category, ordered easy→hard, with the pattern each teaches.
4. For C++: the concept explained deeply + a hands-on coding task + the gotcha interviewers probe.
5. For Python: the concept + a clean-code task, applying naming/readability standards I'm weak on.
6. For the rotating AI/ML/DS topic: theory depth + a practical task (what to build with Claude Code) + interview Q&A.
7. For Applications: today's specific action items.
8. The STAR story or behavioral point to prep today (if any).

Keep it copy-paste ready and practical. Assume I'll actually code everything. Flag the single most important thing for today.
```

---

*Built for Shashank — adjust freely. The plan assumes consistency over perfection: a Yes in the Done column every day beats a perfect day skipped.*
