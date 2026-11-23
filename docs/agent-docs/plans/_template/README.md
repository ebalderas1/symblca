# Plan template

Copy this folder to start a new multi-session plan:

```sh
cp -r docs/agent-docs/plans/_template docs/agent-docs/plans/<YYMMDD>-<slug>
rm docs/agent-docs/plans/<YYMMDD>-<slug>/README.md
```

- `PLAN.md`: the what and why (approach, affected code, contracts, risks), plus a **Handoff** section updated at the end of each session.
- `TASKS.md`: the checklist, grouped into waves, each task with Files / Do / Tests / Done when.

Session prompts:

- Start: "Read `docs/agent-docs/plans/<YYMMDD>-<slug>/` and continue from the Handoff's next step."
- End: "Update the Handoff and TASKS.md checkboxes in `<plan path>`, then commit."

See `260918-result-pagination/` for a completed example.
