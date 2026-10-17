# Shared conventions for the owner's browser extensions

These rules apply to every extension that imports this file (see `README.md` for the list). Each extension's own
`AGENTS.md` adds what is specific to it and **wins where the two differ**; it should say so explicitly when it
deviates. When a rule here changes, check it against every extension listed in `README.md`.

**Where a rule belongs:** if it would be true in any of the extensions, it goes here; if it names a file, setting,
site or API of one extension, it goes in that extension's `AGENTS.md`. When a project rule, pattern or tool turns
out to be general, **propose it to the owner; do not edit this kit unasked**, since a change here affects every
extension. Once they agree: update this kit (its own git repo, so its own commit), check the change against every
extension in `README.md`, remove the duplicate from the project's `AGENTS.md`, and say what changed in the final message.

## What these extensions are

- Chrome/Edge Manifest V3 extensions, built for the owner's own use and friends; others are welcome to use them.
  Published on the Chrome Web Store under the GitHub account `Atsukio`.
- Plain JS/CSS/HTML. No framework, no bundler, **no runtime dependencies**, no build step to run the extension:
  load the repo folder unpacked at `chrome://extensions`. npm is for dev tooling (lint, tests, assets, packaging) only.
- Settings apply live (`chrome.storage.onChanged`), without a page reload. A feature that can be switched off must
  undo itself cleanly when switched off.

## Layout for new extensions

New extensions use this layout. The existing ones keep theirs (each `AGENTS.md` describes its own); do not move
files in them without the owner asking, since manifest paths, build lists, tests and screenshot scripts all follow.

```text
manifest.json      at the root (Chrome requires it); folder names starting with "_" are reserved by Chrome
background/        service worker (a module) and its helpers
content/           scripts and CSS injected into sites (classic scripts, not modules: share code via globalThis)
ui/                extension pages: popup, side panel, options, unsupported page; their CSS, the generated
                   M3 tokens, changelog.js and the What's new card. One .js per page (no inline scripts allowed)
lib/               pure helpers shared by the worker, the pages and (as classic scripts) content scripts
fonts/             bundled Roboto and its licence
icons/             icon-16.png, icon-32.png, icon-48.png, icon-128.png
tools/             dev-only scripts: packaging, palette/assets, screenshots (never shipped)
tests/             automated tests (never shipped)
store/             LISTING.md, screenshots, promo images and the sources they are rendered from
dist/              built upload zips
CLAUDE.md          imports this file and AGENTS.md
AGENTS.md  PRIVACY.md  README.md  LICENSE  package.json (dev tooling only)  .github/workflows/
```

- The zip is built from an explicit list (`manifest.json`, `background`, `content`, `ui`, `lib`, `fonts`, `icons`),
  never "everything except", so dev files cannot leak into an upload.
- Leave out folders an extension does not need (an extension with no content scripts has no `content/`).
- No framework (WXT, Plasmo): they need a bundler and a build step. Reconsider only for Firefox support,
  TypeScript or a UI framework.

## Permissions and privacy

- Adding an install-time permission or host permission in an update disables the extension for existing users until
  they accept it. Avoid that: prefer `optional_host_permissions` requested when the user turns the feature on, and
  check a permission's install warning before adding it. If an update must add one, say so in the release notes.
- Anything that reads, stores or sends user data differently needs `PRIVACY.md` (and its effective date) and
  `LISTING.md` (permission justifications, data-usage answers) updated in the same change.
- API keys and tokens live only in `chrome.storage.local`: never in sync storage, exported settings, the repo, logs or
  screenshots.
- Data in repo files, tests, fixtures, demos and store screenshots is **synthetic**. Never commit, paste, read or repeat
  real user data, and never open `.env` files.
- Extension pages make no network requests they do not need: fonts and icons are bundled.

## Design: Material Design 3

- Our own UI follows Material Design 3: colour roles, type scale, shape, state layers, with **Roboto** bundled locally
  (licence file shipped alongside it). A remote font would leak IPs and contradict `PRIVACY.md`.
- Colours are real M3 roles generated from a seed colour by a script using `@material/material-color-utilities`
  pinned at **0.3.0** (0.4.0 ships broken ESM imports under Node 24; do not upgrade without checking). The generated
  tokens are never hand-edited: change the seed in the script and regenerate.
- Use tokens (CSS custom properties), never raw hex values, in the pages.
- Compact switches (40x24) instead of the M3 default 52x32; the owner found the default too big.
- Check every UI change in light and dark (`prefers-color-scheme`).
- Each extension records its seed, scheme variant, token prefix and type sizes in its own `AGENTS.md`; these are the
  owner's per-project choices, do not "harmonise" them without asking.

## Shared features

Features that exist in more than one extension. Keep their behaviour the same; a new extension that wants one copies
the approach from an extension that already has it.

- **What's new card** (both extensions). Chrome updates extensions silently, so: on `runtime.onInstalled` with reason
  `update` and a different `previousVersion` (an unpacked reload reports the same version), store `whatsNewFrom`
  in `chrome.storage.local` (keep an older value already stored, so two unseen updates show both) and set a "NEW"
  toolbar badge; re-set the badge on `runtime.onStartup` while `whatsNewFrom` is stored (badge text does not survive a
  restart). The extension's UI shows a card listing every changelog entry newer than `whatsNewFrom` and not newer than
  the running version, built with `textContent`. "Got it" removes the key and the badge. No tab is opened, no
  permission is needed, nothing happens on a fresh install. Preview while developing: set `whatsNewFrom` to an older
  version from the service worker console. The changelog is plain language about what users will notice, newest first.

## Releasing to the Web Store

1. The store rejects a package whose version is not higher than the last upload: bump `version` in `manifest.json`
   (and in `package.json` if that extension keeps it in step).
2. If users should hear about the release, add a plain-language changelog entry for that version in the same commit.
3. Lint/test, then build the zip with the extension's packaging script. The zip must use forward-slash paths: never
   build it with Git for Windows' GNU `tar` (it can silently write a tar archive) or PowerShell 5.1 `Compress-Archive`
   (it writes backslashes).
4. Update `LISTING.md` and `PRIVACY.md` if permissions or behaviour changed; re-render store screenshots after visible
   UI changes.
5. The owner uploads the zip from `dist/`.

## Commit messages

All commits follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/):

```text
<type>(<optional scope>)<optional !>: <imperative summary, lower case, no full stop>

<optional body: what changed and why, wrapped at 72 columns>

<optional footers: BREAKING CHANGE: ..., Co-Authored-By: ...>
```

- **Types:** `feat` (user-visible feature), `fix` (bug fix), `docs`, `refactor`, `test`, `ci`, `build`, `chore`,
  `perf`, `style`, `revert`. Scopes are listed in each extension's `AGENTS.md`.
- **Breaking changes:** put `!` after the type/scope and add a `BREAKING CHANGE:` footer that explains how to migrate
  (for example, settings that are reset or a permission that is newly required).
- **Versioning:** the commits since the last release decide the next version. Any breaking change means a major bump,
  otherwise any `feat` means minor, otherwise any `fix` means patch. `docs`, `ci`, `chore` and similar commits on their
  own don't need a release. The release commit is `chore(release): <version>`, in the format the extension's
  `AGENTS.md` gives.
- Commits made by an AI agent end with its `Co-Authored-By` trailer given by the session as the last footer.
- Commits before 2026-10-14 predate this rule. Don't rewrite history to fix them.

## Working with the owner

- Commit, push and archive only when the owner explicitly asks. Offer when a coherent set of changes is ready.
- They iterate on look and feel (colour, size, density) from screenshots: show a render before and after.
- When they report from a real site, their screenshots may contain real data: do not repeat or store it.
- Keep each extension's `AGENTS.md` (and this file) up to date in the same commit as the change that affects it.
- Match the surrounding style. Keep comments to what is non-obvious.

## Environment gotchas (Windows)

- Node must be on PATH. If a terminal started before install can't find `node`, add `C:\Program Files\nodejs` to
  PATH for that session or restart VS Code fully.
- Windows PowerShell 5.1 `Get-Content`/`Set-Content` re-encode UTF-8 (it mangled a (c) sign once). Edit text files
  with the editor tools or `sed`, not PowerShell read/write.
- Git warns "LF will be replaced by CRLF". It is harmless; do not "fix" line endings. When patching with a script,
  keep the file's existing line-ending style.
- In `sed`, do not use `#` as the delimiter when the replacement contains `#` (Markdown headings). For regex-heavy
  edits, write a script file: heredoc escaping mangles backslashes.
- The repos live under OneDrive; that is fine for git, but keep `node_modules` and `dist/` git-ignored.
