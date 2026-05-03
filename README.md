# teichaku

**定着** — knowledge retention CLI for your markdown vault.

Spaced repetition (SM-2) for the notes you actually care about, written in [Almide](https://github.com/almide/almide). Cards live as JSON next to your vault so review history rides along in git.

## Why

You're building a knowledge base. Writing the notes is half the job; getting the content to *stick* is the other half. teichaku is the second half — a CLI that asks you "do you actually remember this?" on the schedule SM-2 says you should.

It's a CLI re-implementation of the spaced-repetition core in [MemRE](https://github.com/O6lvl4/memre), without the Go runtime, GUI, or LLM dependency. Just markdown vault + flat JSON + a terminal.

## Install

Requires [Almide](https://github.com/almide/almide).

```bash
git clone https://github.com/O6lvl4/teichaku
cd teichaku
almide run src/main.almd help
```

A binary install (`almide build`) will land once `[[bin]]` lands in `almide.toml` upstream.

## Use

### Configure the vault

teichaku writes cards under `<vault>/.srs/<note-slug>/<card-id>.json`. Point it at your vault:

```bash
export TEICHAKU_VAULT=$HOME/workspace/github.com/O6lvl4/O6lvl4-knowledge/vault
```

If `TEICHAKU_VAULT` is unset, `./vault` is used.

### Add a card

For each note you want to retain, capture one or more Q/A pairs:

```bash
almide run src/main.almd add agentic-coding
# Q: Almide の MSR とは何の指標か
# A: <multi-line answer, end with a blank line>
```

The card is saved at `<vault>/.srs/agentic-coding/agentic-coding-001.json` with initial `interval=0`, `ef=2.5`, `due=today`.

### Review what's due

```bash
almide run src/main.almd review
```

For every card with `due ≤ today`, teichaku shows the question, waits for `Enter`, reveals the answer, and asks for a quality rating:

| Quality | Meaning |
|---|---|
| 0 | total blackout |
| 1 | wrong, but the answer felt familiar when shown |
| 2 | wrong, with a clear "I should have known that" |
| 3 | correct, with serious effort |
| 4 | correct, after some hesitation |
| 5 | perfect recall |

The schedule updates via SM-2: if `q ≥ 3`, `interval` grows (1 → 6 → `interval × ef`); if `q < 3`, it resets to 1 and `ef` adjusts down (floor 1.3).

### Inspect the state

```bash
almide run src/main.almd stats               # everything
almide run src/main.almd stats agentic-coding   # one note
```

Reports total cards, cards due today, average easiness factor, and a per-note breakdown.

## Storage format

```
<vault>/.srs/agentic-coding/agentic-coding-001.json
```

```json
{
  "id": "agentic-coding-001",
  "note": "agentic-coding",
  "question": "Almide の MSR とは何の指標か",
  "answer": "Modification Survival Rate ...",
  "ef": 2.5,
  "interval": 0,
  "reps": 0,
  "due": "2026-05-04",
  "created": "2026-05-04",
  "history": []
}
```

JSON is intentionally simple so you can hand-edit, grep, and diff cards in git.

## Layout

```
teichaku/
├── almide.toml      package = teichaku
└── src/
    ├── main.almd       entry, sub-command dispatch
    ├── srs.almd        pure SM-2 math (apply, initial)
    ├── card.almd       Card type + JSON serialize/deserialize
    ├── store.almd      <vault>/.srs/ I/O
    ├── cmd_add.almd    `add` — interactive Q/A capture
    ├── cmd_review.almd `review` — due-card walk + rating loop
    └── cmd_stats.almd  `stats` — totals, due, avg EF, by-note breakdown
```

## Roadmap

- `teichaku init` — scaffold `<vault>/.srs/` with a `.gitignore` carve-out
- `teichaku sync` — write `retention`, `last_reviewed`, `card_count` to vault frontmatter
- `teichaku export` — Anki `.apkg` / CSV
- LLM-assisted card extraction (probably via [almai](https://github.com/almide/almai)) — keep manual capture as the primary path

## License

MIT
