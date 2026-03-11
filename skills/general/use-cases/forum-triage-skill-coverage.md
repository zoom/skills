---
title: "Improve Skill Coverage from Forum Threads"
---

# Improve Skill Coverage from Forum Threads

This repo includes a local triage/index tool to turn large sets of Developer Forum threads into an actionable backlog:

- mark threads as `addressed`, `unaddressed`, `product_missing`, `out_of_scope`, etc.
- cluster recurring question types
- export batches for labeling
- fold recurring answers into the right `skills/<product>/` references and `triggers`

## Where the data lives

- Forum threads: `~/outputvector/*.json`
- Triage DB: `skills/tools/thread-triage/thread_triage.sqlite`
- Reports: `skills/tools/thread-triage/reports/`

## Build the index (incremental)

```bash
cd ~/skills
python3 tools/thread-triage/thread_triage.py build --retry-failed --rebuild-changed
python3 tools/thread-triage/thread_triage.py files --failed
python3 tools/thread-triage/thread_triage.py stats
```

## Generate clusters (what customers keep asking)

```bash
cd ~/skills
python3 tools/thread-triage/thread_triage.py cluster --suggested-label unaddressed --limit 6000
python3 tools/thread-triage/thread_triage.py cluster --suggested-label product_missing
```

Pre-generated cluster reports (CSV + MD) are written to:

- `tools/thread-triage/reports/clusters_unaddressed_top6000.*`
- `tools/thread-triage/reports/clusters_unaddressed_gold.*`
- `tools/thread-triage/reports/clusters_product_missing.*`

## Batch export for labeling (200 per file)

```bash
cd ~/skills
rm -rf tools/thread-triage/batches
python3 tools/thread-triage/thread_triage.py export-batches --out-dir tools/thread-triage/batches --limit 6000 --batch-size 200
```

## Folding patterns back into skills

When a cluster is important, prefer **one good write-up per cluster** (covers many threads) instead of one-off notes.

Typical placement:

- Meeting SDK: `meeting-sdk/references/` (plus web/mobile sub-skill docs)
- Video SDK: `video-sdk/references/`
- REST API: `rest-api/troubleshooting/` and `rest-api/concepts/`
- Zoom Apps: `zoom-apps-sdk/troubleshooting/` and `zoom-apps-sdk/references/`

Then add/expand `triggers:` in the relevant `SKILL.md` so the right skill loads for that question type.

