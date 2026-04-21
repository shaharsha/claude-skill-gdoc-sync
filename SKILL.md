---
name: gdoc-sync
description: Sync a Markdown file to an existing Google Doc — pushes the markdown via Drive upload, rewrites Google's broken `[text](#slug)` anchor imports into working internal heading links, resizes oversized inline images, and optionally applies right-to-left paragraph direction for Hebrew/Arabic. Use when the user asks to sync/push/deploy markdown to a Google Doc, update an existing Google Doc from a local .md file, or maintain a Hebrew/Arabic Doc whose body must render RTL.
allowed-tools: Read, Write, Bash, Edit, Glob, Grep
---

# gdoc-sync

Pushes a local Markdown file into an existing Google Doc via the Drive + Docs APIs, then patches two things Google's native import gets wrong: internal anchor links and oversized images. Optional third pass applies RTL to every body paragraph for Hebrew/Arabic docs.

The script is a single Python file (`scripts/sync-gdoc.py`) with no install step beyond the Google auth library. It authenticates via a service-account JSON (recommended) or gcloud ADC (fallback).

## Quick start

```bash
scripts/sync-gdoc.py <markdown-file> --doc-id <FILE_ID> --sa-key <service-account.json> [--rtl] [--no-links] [--max-image-width 300]
```

**End-to-end example** (Hebrew doc, service account auth):

```bash
scripts/sync-gdoc.py plan.md \
  --doc-id 1lSspmI7TXXxVPjX8mLRa8LZEPkhE2rNDYJ_fFDU-BEE \
  --sa-key ~/Downloads/sa-key.json \
  --rtl
```

Output:

```
[1/3] Pushed markdown: plan.md (84,468 bytes)
[2/3] Fixed 35 anchor links
[2.5/3] Resized 2 oversized images (max width 300.0pt)
[3/3] Applied RTL across doc (1..52026)
Done. https://docs.google.com/document/d/1lSspmI7.../edit
```

## What it does (three-step pipeline)

| Step | What | Why |
|---|---|---|
| **1. Push** | `PATCH` markdown to Drive via `/upload/drive/v3/files/{id}` with `Content-Type: text/markdown` | Google converts natively — paragraphs, headings, tables, code blocks, inline images all land correctly |
| **2. Fix anchor links** | Walks the Docs structure, builds a `section-number → headingId` map from heading paragraphs, rewrites every text run whose `link.url` starts with `#` to use `link.headingId` instead | Google's markdown import turns `[text](#slug)` into a *URL link* pointing at literal `#slug` (broken). This step makes cross-references clickable natively. |
| **2.5. Resize images** | Finds inline images wider than `--max-image-width`, deletes and re-inserts with explicit `objectSize` preserving aspect ratio | Google's import sets inline image dimensions to the source image's native size (often 1000+ pt wide — blows past the page margins) |
| **3. RTL (optional)** | `batchUpdate` with `updateParagraphStyle` setting `direction: RIGHT_TO_LEFT` across the full body range | Neither Google's import nor its UI sets paragraph direction for Hebrew/Arabic; without this, every paragraph renders LTR by default |

## When to use this skill

| Scenario | Use this | Why |
|---|---|---|
| Existing Google Doc, updated by editing a local `.md` | Yes | That's the whole purpose |
| Hebrew / Arabic / RTL doc authored in markdown | Yes, with `--rtl` | No other tool sets direction across the doc |
| Doc with cross-section links like `[see §5.3](#53-foo)` | Yes, with links auto-fixed | Google's native import leaves these broken |
| One-shot export of markdown to a brand-new doc | Yes, but create the empty Doc first and pass its ID | Script operates on `--doc-id`; doesn't create docs |

## When NOT to use

| Scenario | Use instead |
|---|---|
| You want a local-only markdown render | Any markdown previewer; no Google round-trip needed |
| Round-trip editing (edit in Google Doc, sync back to markdown) | This skill is one-way (md → Doc). Use Google Docs export for the reverse direction. |
| Preserving comments/suggestions across syncs | You can't — the markdown import is destructive to pending suggestions and orphans comments whose anchored text no longer matches. See [reference/gotchas.md](reference/gotchas.md). |
| Managing Doc permissions / sharing | Out of scope — use the Drive UI or `gh api drive/v3/files/.../permissions`. |

## Auth

Two supported paths. Service account is strongly recommended — gcloud user ADC hits Google's "this app is blocked" policy for sensitive scopes (Drive write).

| Path | Flag | Setup | When |
|---|---|---|---|
| **Service account (recommended)** | `--sa-key path/to/sa.json` | One-time: create SA in GCP, enable Drive + Docs APIs in that SA's project, share the target Doc with the SA's email as Editor | Default — most reliable |
| **gcloud ADC (fallback)** | *(no flag, uses gcloud)* | `gcloud auth application-default login` with Drive scope — may be blocked by Google for the default gcloud OAuth client | Only if SA setup is unavailable; expect friction |

Full setup instructions: [reference/auth-setup.md](reference/auth-setup.md).

## Gotchas

The import is destructive, the anchor-link rewrite is regex-based, and RTL has quirks. Read [reference/gotchas.md](reference/gotchas.md) **before** running against a doc you care about — especially the first time.

## Non-goals

- **Not one-way only by accident — by design.** Does not read back from the Doc. If you need two-way sync, use a different tool (e.g., a proper CMS).
- **Does not create new Docs.** Takes `--doc-id` of an existing Doc.
- **Does not manage sharing/permissions.**
- **Does not handle Google Workspace Shared Drives specially** — should work, but tested primarily on personal / domain-owned Docs.
- **Does not preserve comments or suggested edits** — the markdown PATCH overwrites the body content, which wipes pending suggestions and orphans comments whose anchored text changed.

## Script layout

```
gdoc-sync/
├── SKILL.md                   # this file
├── README.md                  # public-facing overview (GitHub landing page)
├── LICENSE                    # MIT
├── scripts/
│   ├── sync-gdoc.py           # the main script (Python, stdlib + google-auth optional)
│   └── README.md              # per-script invocation + flags
└── reference/
    ├── auth-setup.md          # service account setup, API enablement, Doc sharing
    └── gotchas.md             # known quirks, destructive behaviors, edge cases
```

## Dependencies

- **Python 3.9+**
- **Service account path:** `pip install google-auth` (for the `google.oauth2.service_account` module)
- **gcloud ADC path:** `gcloud` CLI available on `$PATH`
- No other deps — uses Python's `urllib` stdlib for HTTP.

## Related skills

None at this time. If you need to generate Doc content (beyond syncing existing markdown), [brand-system](../brand-system/SKILL.md) scaffolds long-form BRAND.md documents — pair it with this skill for rendering to Google Docs.
