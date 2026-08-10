# Prompts

Reusable prompts for working on this product. One folder per prompt, one page per version, prompt text kept verbatim so it can be copied and run as-is.

| Prompt | What it does | Versions | Current |
|---|---|---|---|
| [Flow Audit](flow-audit/README.md) | Reverse-engineers one user flow end-to-end — UI through API through persistence — ranks the pain by return on effort, and writes a change brief per side | 2 | [v2](flow-audit/v2.md) |

## How this section is organised

A prompt is a folder. A version is a page inside it. The folder's own page is the hub: what the prompt is for, which version to use, and what changed between them.

```
prompt/
├── README.md              this page — the catalogue
└── flow-audit/
    ├── README.md          hub: purpose + version table
    ├── v2.md              current
    └── v1.md              superseded
```

Versions are listed newest first, so the one you almost always want sits at the top.

## Adding a version

1. Add `vN.md` to the prompt's folder. Open with a one-line status, list what changed against the version before it, then the full prompt inside a `text` code fence.
2. Add a row to the version table on the prompt's hub page and move the **Current** marker.
3. Mark the previous version `Superseded` and point it at the new one.
4. Bump the version count in the table above, and update the **Current** link.
5. Register the page in `SUMMARY.md`, above the version it replaces.

## Adding a prompt

Create `prompt/<slug>/` with a `README.md` hub and `v1.md`, add a row to the table above, and register both in `SUMMARY.md`. Keep the slug descriptive of what the prompt does — the folder name is what people read in the sidebar.
