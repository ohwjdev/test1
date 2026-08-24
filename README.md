# Let's Encode! — campaign template

This is the **template repository** every campaign is stamped from (via GitHub's
"create a repository from a template"). A copy of it becomes one crowd-encoding
campaign: it holds the score sources, the campaign configuration, five
machine-maintained tracking tables, and one generic caller workflow that runs
the campaign automation.

> You normally don't touch these files by hand. The instigation platform fills
> in the configuration at campaign start, and the volunteer client claims and
> submits work through pull requests. The files are kept human-readable so they
> review cleanly in pull-request diffs.

## Layout

```
config.example.yaml          # campaign config TEMPLATE (schema v3) — documented inline
config.yaml                  # written at instigation by the GUI (not in the template)
sources/
  img/                       # the source's committed page images (facsimile campaigns)
  <piece-id>/score.mei       # one MEI per piece, written at init (not in the template)
templates/
  score.template.mei         # barebones MEI: 1 measure, 1 note, header placeholders
tracking/                    # five tables keyed by (task_id, subtask_id) —
  task.csv                   #   task/subtask definitions (fragment, locator, gates, depends_on)
  state.csv                  #   live status + validation cells
  lock.csv                   #   active claims
  history.csv                #   append-only log of every action, incl. rejects
  comment.csv                #   review comments on tasks (threaded, resolvable)
                             # generated at init & maintained by the automation
.github/workflows/
  caller.yml                 # the ONE task-agnostic caller — forwards every event
                             # to the central automation named in config.yaml
```

## Lifecycle in one paragraph

The instigation GUI generates a campaign repo from this template, then commits —
in one go — a filled-in `config.yaml` (shaped like `config.example.yaml`), one
`sources/<piece-id>/score.mei` per piece stamped from
`templates/score.template.mei` (header filled from the config), and the five
tracking tables (tasks `encoding_required`, their validation subtasks
`pending`, empty lock, history and comment tables). Each facsimile piece opens
with a measure-correction pre-task; its per-page encoding tasks depend on it
via the `depends_on` column, so they unlock once the pre-task completes. From
there, volunteers claim and submit work as pull requests; on each one
`caller.yml` checks out the central automation repo named in `config.yaml` and
runs it, and that coordinator validates the contribution, mutates the tables,
and closes or merges the PR. An hourly cron run of the same workflow reaps
stale locks.

## File formats

`config.yaml` is YAML; the tracking tables are CSV (chosen for clean,
cell-level pull-request diffs and trivial machine parsing). The schema, the
column meanings, and the validation-cell encoding are specified in `DESIGN.md`
in the `instigation` repo (the single design + status document for the project).
