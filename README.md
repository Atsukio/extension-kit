# extension-kit

Shared conventions for the owner's browser extensions, read by Claude Code (and anyone else) in each of them.

## Extensions using it

| Folder (sibling of this one) | Repo |
| --- | --- |
| `clean-subs-for-youtube` | `Atsukio/everything-for-youtube` |
| `bank-to-wallet-sync` | `Atsukio/bank-to-wallet-sync` |

## Hooking up an extension

The extension must sit next to this folder. Give it a `CLAUDE.md` containing:

```md
@../extension-kit/CONVENTIONS.md
@AGENTS.md
```

Claude Code reads only `CLAUDE.md` when both files exist, so the import of `AGENTS.md` is needed. The first time
Claude Code opens the project it asks to approve the import from outside the folder; approve it once. Also put
a plain-text pointer at the top of that `AGENTS.md`, for tools that don't follow imports. Then add the extension to
the table above.

## What goes where

- `CONVENTIONS.md`: rules true for every extension (layout for new ones, design, permissions, privacy, releases,
  commits, Windows).
- Each extension's `AGENTS.md`: its layout, settings, site/API notes, and its own choices (seed colour, type sizes,
  commit scopes, release tag format).
