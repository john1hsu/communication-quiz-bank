# Question schema

Each true/false **pair** is one YAML file: `questions/<area>/<id>.yaml`. The folder names match the article domains in [communication-graph](https://github.com/john1hsu/communication-graph) (listening, writing, negotiation, …). The quiz serves one side of a pair at random and never both sides in one session.

```yaml
id: cm-000002                     # stable, never reused
area: listening                   # = source article's domain = folder
kind: scenario                    # scenario | practice | definition | mechanism | evidence | origin | figure
topic: Active listening           # the source article's title, shown after answering
topic_type: Skill                 # the article's class: Technique, Skill, Barrier, AntiPattern, Framework, Person, ...
true_statement: An engineer says the release date is unrealistic. Her lead replies by restating the two untested integrations she named and her worry about blame, then checks he has it right. That is active listening.
false_statement: An engineer says the release date is unrealistic. Her lead replies by restating the two untested integrations she named and her worry about blame, then checks he has it right. That is giving advice.
explanation: The lead reflects both the content and the feeling and checks his understanding before responding, the core of active listening. Advice would jump to fixes; here he offers none.
as_of: '2024'                     # only on figure pairs: the year the number is for
source:
  repo: john1hsu/communication-graph
  path: articles/listening/active-listening.md
  commit: <source repo HEAD when written or last verified>
  blob: <git blob of the article at that point>
concepts: [cm:active-listening, cm:paraphrasing]   # graph node ids; the source article's id comes first
related: []                       # optional: other pair ids
supersedes: cm-000041             # only on a pair that replaced an outdated one
difficulty: 2                     # 1 foundations | 2 practitioner | 3 advanced
created: '2026-11-16'
status: active                    # active | stale | retired
stale_reason: ...                 # only while stale
retired_reason: ...               # optional, on retired pairs
```

`schema/question.schema.json` has the same rules as a JSON Schema for editors.

## Kinds

Each pair says what it teaches. The first two practise the skill itself; the rest build the knowledge behind it. Every article that describes a move you can make or meet (techniques, skills, frameworks, principles, barriers, anti-patterns, situations, channels and formats) gets at least one skill pair.

| kind | App label | Tests |
| --- | --- | --- |
| scenario | Spot it | A short, realistic workplace exchange, and what it shows: which technique, habit, barrier or anti-pattern, or whether the response works |
| practice | Say it better | What to say or do: the right step or its order, the better rewrite, the fix for a barrier or bad habit, the self-check to run, or when a technique backfires |
| definition | What it is | What the idea is, and what it is not |
| mechanism | Why it works | The psychology, cause and effect: why it helps or why it misleads |
| evidence | Evidence and myths | What research shows, replication status, and popular claims that outrun their evidence |
| origin | Who and when | Who introduced it, where, and when |
| figure | Numbers and dates | A dated number (`as_of` required, and dated in the statement) |

## Lifecycle

Communication techniques don't ship in releases, so a pair changes only when its source article does (for example, when a new meta-analysis changes the evidence status of a piece of advice).

| Status | Meaning | In the quiz? |
| --- | --- | --- |
| active | Matches the latest version of its source article | Yes |
| stale | The source article changed (or disappeared) since the pair was verified | Only with `--include-stale` |
| retired | The fact changed (a newer pair `supersedes` it), the article is gone, or the pair repeated a fact another pair already tests | No |

## Checks (`scripts/validate.py`)

| Check | Fails when |
| --- | --- |
| Schema | A required field is missing, malformed or unknown |
| Unique ID | An ID repeats, or the file name or folder doesn't match |
| Kind | `kind` isn't one of the seven kinds |
| Dated figures | A `figure` pair has no `as_of`, `as_of` names no year, or the true statement doesn't mention it |
| Duplicate | A statement closely matches one in another non-retired pair |
| Same fact | Two non-retired pairs, from any articles, have true statements that state one fact in different words: most of their words match, or they share their numbers and most of the shorter statement's words |
| Pair quality | The two statements are identical or differ only by negation |
| Explanation | Shorter than 60 characters |
| Statement form | A statement is a question, under 20 characters, or over 280 (360 for `scenario`) |
| Source | Repo, path, commit or blob is malformed; with `--source`, the article's domain differs, a concept isn't an edge of the article, or an active pair's article is missing or changed (run `refresh.py mark-stale` first; stale pairs are exempt because they are waiting to be refreshed or retired) |
| Supersedes | Points to an ID that is missing or not retired |
| Exam-like wording | "Which of the following", "Choose two", "according to the article" and similar |
