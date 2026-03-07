# semantic_cache_playground

An interactive 3-page visual demo that teaches developers why **semantic caching can serve wrong answers** and how to fix it with a hybrid key strategy.

---

## What It Is

A single-file HTML playground that walks through the core risk of cosine-similarity-based caching: two queries can score **0.96+ similarity** while requiring completely different answers. One wrong digit. One wrong account ID. The cache doesn't know the difference.

The demo covers three pages:

1. **Choose a scenario** : pick from 5 real-world query pairs (safe and dangerous)
2. **The problem** : see the similarity score, drag a threshold slider, and watch the cache hit or miss in real time
3. **The solution** : toggle between Pure Semantic and Hybrid Key caching to see how entity extraction prevents collisions

---

## Scenarios Included

| Scenario | Risk | Why It's Interesting |
|---|---|---|
| Medication Dosage | 🔴 Lethal | `10mg` vs `100mg` : 0.96 similarity |
| Account Balance | 🟡 Wrong Data | `acct:4521` vs `acct:4522` : 0.98 similarity |
| Password Reset | 🟢 Safe | Same intent, different phrasing : ideal cache hit |
| Explain Gravity | 🟢 Safe | Sits right on the threshold boundary |
| Cancel Subscription | 🟢 Safe | Structure differs, meaning identical |

---

## The Core Concept

### Pure Semantic Caching
```
Redis key: {company:user}:[vector_hash]
```
Only the vector hash determines cache lookup. Two queries with high cosine similarity → same cache bucket → potentially wrong answer served.

### Hybrid Key (The Fix)
```
Redis key: {company:user}:[extracted_entity]:[vector_hash]
```
Critical entities (dosages, account IDs, drug names) are extracted from the prompt and embedded in the key itself. The cache now has **two gates**:

- **Gate 1** : Exact string match on extracted entity
- **Gate 2** : Vector similarity check (only runs if Gate 1 passes)

If Gate 1 fails, Gate 2 never runs. A `$0.003` LLM call replaces a free but wrong cached answer.

---

## How to Run

No build step. No dependencies. Just open the file:

```bash
open index.html
```

Or serve it locally:

```bash
npx serve .
# or
python -m http.server 8080
```

---

## File Structure

```
semantic_cache_playground/
└── index.html        # entire app — HTML, CSS, and JS in one file
└── README.md
```

---

## Key Interaction: The Threshold Slider

On page 2, a slider lets you set the cosine similarity threshold (0.80 – 0.99). This demonstrates the fundamental trade-off:

- **Too loose** → dangerous pairs become cache hits → wrong answers
- **Too strict** → safe pairs become cache misses → unnecessary LLM calls
- **The insight** → a single threshold can't solve both problems. Entity extraction is required.

---

## Tech Stack

- Vanilla HTML/CSS/JS : zero dependencies
- No framework, no build tool, no backend
- Fully self-contained in one file