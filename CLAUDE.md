# Communication quiz bank: daily run playbook

This repo is a bank of paired true/false questions that help John **communicate better at work**, not just memorise terms. Every pair is generated from the cited articles in [john1hsu/communication-graph](https://github.com/john1hsu/communication-graph). A daily scheduled run handles up to **100 pairs**: it refreshes stale pairs first, then spends the rest of the budget on new ones, at most **3 per article**. Everything goes to `main` as **one combined commit** with one push, and only when every check passes. Read `SCHEMA.md` before writing.

## 1. Sync

```bash
git pull --rebase origin main
rm -rf .source && git clone --depth 1 https://github.com/john1hsu/communication-graph .source/communication-graph
python3 scripts/validate.py   # the repo's own checks, without the source; must print OK; fix the repo first if not
```

Don't pass `--source` here: source articles change between runs, and step 2 is what handles that.

## 2. Mark stale pairs

```bash
python3 scripts/refresh.py mark-stale
python3 scripts/validate.py --source .source/communication-graph   # must print OK now; fix the repo first if not
```

`mark-stale` marks active pairs stale when their source article changed since they were verified (a corrected date or attribution, a new meta-analysis, a changed evidence status, a reworded step or example), or when the article was removed or renamed. Always run it before validating against the source: a changed or missing article is an error on an active pair, but expected on a stale one that is waiting for step 4a.

## 3. Pick this run's work

The budget is 100 pairs per run (refreshed plus new). Repeat this step until the budget is used or `pick` returns `"none"`:

```bash
python3 scripts/generate.py pick --n 3
```

- `"mode": "refresh"` → do step 4a for every pair in `batch`.
- `"mode": "generate"` → do step 4b for `article`, writing `n` pairs.
- `"mode": "none"` → stop picking and go to step 5. If nothing was refreshed or added but step 2 marked pairs stale, commit that with the message `quiz: mark <n> pairs stale`; if nothing changed at all, stop without committing.

Don't commit between picks. Run `python3 scripts/validate.py --source .source/communication-graph` after roughly every 15 new pairs to catch problems early.

Stale pairs always come before new ones (dated figures first). New articles are picked fewest-pairs first, and among those from the area with the fewest pairs, so practice stays spread across listening, writing, presenting, feedback, conflict, negotiation, meetings, executive communication and the rest.

## 4a. Refresh a stale pair

Read the pair (`questions/*/<id>.yaml`) and the **current** source article in full. Decide:

| Outcome | Command |
| --- | --- |
| Both statements still hold (true one true, false one false) per the article | `python3 scripts/refresh.py verify <id>` |
| The fact changed (a failed replication, a new meta-analysis, a corrected date or attribution, a rewritten step) | Write one replacement pair to a JSON file (format in 4b), then `python3 scripts/refresh.py supersede <id> --input <file>` |
| The article is gone, or no longer supports the fact | `python3 scripts/refresh.py retire <id> --reason "<why>"` |

If the article now says something that makes the explanation wrong but both statements still hold, supersede rather than verify. Watch `evidence_status` in particular: a pair that presents advice or an effect as solid must be superseded if the article now marks it `contested` or `failed-replication`.

## 4b. Write new pairs

Read the whole article (`.source/communication-graph/<path>`), including its frontmatter. `pick` also hands you the article's `steps`, `example` (a workplace before/after rewrite with a "why"), `self_check` questions and `evidence_status`: these are the best raw material for skill pairs. Check `existing_pairs_for_article` and `related_pairs_in_other_articles` so you don't repeat a fact: the second list holds the true statements already written from neighbouring articles, and a fact any of them tests is taken, even if this article words it differently. Write `n` pairs to a JSON file outside the repo (the scratchpad):

```json
{"article": "articles/<domain>/<slug>.md",
 "pairs": [{"kind": "scenario", "true_statement": "<a short workplace exchange>, which is <the right technique, habit, barrier or verdict>.",
            "false_statement": "<the same exchange>, which is <a neighbouring one that is easy to confuse>.",
            "explanation": "...", "difficulty": 2, "concepts": ["cm:<article-id>", "cm:<other-node>"]},
           {"kind": "practice", "true_statement": "...", "false_statement": "...", "explanation": "...", "difficulty": 2}]}
```

### The goal: practise the skill, not just the vocabulary

John wants this bank to make him a better communicator rather than quiz him on facts about communication. So:

- **If `needs_skill_pair` is true, at least one of the article's pairs this run must be a `scenario` or `practice` pair.** Write both when the article supports them (techniques, skills, frameworks, barriers, anti-patterns, contexts, channels and genres nearly always do). `pick` sets `needs_skill_pair` to false for person, work, study and field articles, which may have only knowledge kinds.
- Then prefer the kinds in `kinds_not_yet_used` (listed skill kinds first), so a topic is learned from several sides.

| kind | App label | Tests | Example true statement |
| --- | --- | --- | --- |
| `scenario` | Spot it | Recognising the move in a realistic workplace exchange: which technique, habit, barrier or anti-pattern it is, or whether the response works | A lead replies to a worried engineer, "Sounds like you're uneasy about being blamed if the date slips," which is labeling an emotion. |
| `practice` | Say it better | What to say or do: the right step or its order, the better rewrite, the fix for a barrier or bad habit, the self-check to run, when a technique backfires | Opening a status email with the decision you need and its deadline, before any background, applies bottom line up front. |
| `definition` | What it is | What the idea is, and what it is not | Paraphrasing restates the speaker's meaning in your own words and checks it with them. |
| `mechanism` | Why it works | Why it helps or hurts: the psychology, the cause and effect | The curse of knowledge makes experts underestimate how much background a newcomer needs. |
| `evidence` | Evidence and myths | What research shows, replication status, popular claims that outrun the evidence | The "93% of meaning is nonverbal" figure comes from narrow lab studies of single words, not everyday conversation. |
| `origin` | Who and when | Who introduced it, where and when | Fisher and Ury set out principled negotiation in *Getting to Yes* (1981). |
| `figure` | Numbers and dates | A dated number (needs `as_of`) | A sample size, effect size or survey result, with the year written into the statement |

Rules for each pair:

- **Grounding:** every pair must be answerable from this article alone. Don't add facts from memory, even true ones. A `scenario` may describe a new exchange, but the verdict must follow from the article's own definition, steps, example or warning signs, and the explanation must say which feature of the exchange decides it.
- **Scenarios:** neutral workplace settings (one-on-ones, status meetings, client calls, email and chat threads, performance reviews, negotiations, presentations, cross-team escalations). Quote a line or two of what someone says or writes when that is the clearest way to show the move. No real, named private individuals, no current partisan figures or parties, and no real company accused of anything. One or two sentences of setup, then the verdict, within 360 characters.
- **Practice pairs:** the false twin should be a plausible but worse move (a step out of order, advice before understanding, background before the ask, a "you" accusation instead of an I-statement, the tactic the article warns against), not an absurd one. Persuasion and negotiation stay ethical: never present a manipulative tactic as the right move.
- **Plausible false twin:** keep the exchange identical and swap the label for a neighbour that is easy to confuse (paraphrasing vs summarizing, mirroring vs labeling, BATNA vs reservation price, SBI vs feedback sandwich, curse of knowledge vs illusion of transparency, high- vs low-context), flip the verdict when it hinges on a stated condition, or swap the person, year, number, cause or direction of an effect. Never make the false statement by just adding "not".
- **One fact per pair**, and a different fact or exchange for each pair in the run and from every pair already in the bank, whichever article it came from.
- **Statements:** declarative, 20–280 characters (360 for `scenario`), readable on a phone card. No exam phrasing ("Which of the following", "Choose two") and never "according to the article".
- **Evidence honesty:** never present `contested` or `failed-replication` advice as established, and say when `evidence_status` is only `practitioner`. Myths the article debunks (the "7-38-55" rule, power posing's hormone claims) make good `evidence` pairs.
- **Dates and figures:** a study's sample size, effect size or result is only a `figure` pair, with `as_of` set and that year written into the true statement. Never undated.
- **Explanation:** 1–3 sentences in your own words saying why the true statement is true and what's wrong with the false one. Make it teach: name the tell-tale feature, what to say instead, or why the neighbouring label doesn't fit. No quote over 15 words.
- **Difficulty:** 1 = foundations (core definitions, classic examples), 2 = practitioner (spotting it in realistic exchanges, key distinctions, what to say), 3 = advanced (evidence and myths, edge cases where a technique backfires, cultural or channel differences).
- **Concepts:** the article's own id plus any graph node ids the fact involves (from the article's edges).

Then add them:

```bash
python3 scripts/generate.py add --input <file>
```

## 5. Validate, build, commit, push

```bash
python3 scripts/validate.py --source .source/communication-graph
python3 scripts/build_index.py
git add -A
git commit -m "<message>"
git pull --rebase origin main && git push origin main
```

- Make **one** commit for the whole run, containing every refresh and new pair, and push it once.
- If validation fails on a new or replacement pair, delete that pair's file, rewrite it once, and validate again. If it still fails, drop just that pair (remove its new file, or undo that refresh) and carry on, so the final commit always validates. Report dropped pairs in the run output.
- If validation says a new pair states the same fact as an existing one, the rewrite must test a different fact or exchange from the article, not the same one reworded. If it flags two pairs that were both already in the bank, keep the older one and run `python3 scripts/refresh.py retire <newer id> --reason "duplicate of <older id>"`.
- Commit message subject: `quiz: refresh <r> pairs, +<n> pairs from <k> articles` (drop the part that is zero). In the body, list one line per pair (`cm-000123: verified` / `superseded by cm-000130` / `retired: <why>` / `added`).
- If the push is rejected, pull --rebase, re-run `build_index.py`, amend that same commit (don't add a second one) and push again (once).

## Don'ts

- Don't handle more than 100 pairs in one run, or write more than 3 pairs from one article in one run.
- Don't make more than one commit or push per run.
- Don't edit `index/` or `coverage/` by hand; `build_index.py` writes them.
- Don't delete question files or reuse an id; retire instead.
- Don't edit communication-graph from this run, and never commit `.source/`.
- Don't commit if validation fails.
