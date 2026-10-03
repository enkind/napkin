Napkin is a free agent plugin whose core is an MCP App: the agent calls one tool, a blank napkin appears in the conversation, the user sketches on it, and the drawing reaches the agent as an image in the user's next message.

It is one of a family of plugins built on the same recipe as Better Response (`../Engawa`): one Next.js deployment serving the website and the Streamable HTTP MCP route, a self-contained widget bundle, one installable directory under `plugins/`, and host marketplaces at the repository root. Keep the recipe's shape when changing any of those; each plugin in the family differs in what it does and in its design language, not in how it is consumed.

## Product context

`README.md` describes what Napkin is and how it is consumed; `DECISIONS.md` records the host evidence behind its architecture. Before product, architecture, or implementation work, read both and treat them as the current source of truth. When an assumption, term, or scope boundary is corrected, or when measured host behavior invalidates a recorded claim, update the relevant file in the same task. A decision that is reversed gets an amendment in `DECISIONS.md`, not a silent edit.

## Implementation guardrail

Keep the code minimal, direct, and straightforward.

- Implement only the path the current task requires.
- Do not add speculative features, abstractions, extensibility, fallbacks, or general-purpose infrastructure.
- Do not introduce extra helpers, files, or dependencies unless the current path genuinely requires them.
- Prefer simple inline code when logic is used once and remains readable.
- The napkin is a pen and a Send button. An eraser, undo, colors, or shapes are scope changes, not polish.

## Agent-facing text

The tool description in `app/mcp/route.ts`, the tool result text, and `plugins/napkin/skills/sketch/SKILL.md` are agent instruction rather than internal documentation. Describe what the napkin is and does, and when it helps; do not script the conversation around it.

## Website

The site lives in `app/` beside the MCP route. Its two design rules are stated at the top of `app/styles.css`, and every addition must keep to them. The chat mock's interior values are measured from a real client and do not follow the site's identity; do not restyle them to match the page.

The napkin in the site's chat mock is the real widget bundle, framed from `/demo` and hosted by `app/napkin-card.tsx`. If the widget starts using a new part of the host bridge, extend that host in the same change, as with `widget/preview.tsx`.

## Plugin package versioning

Versions are computed by semantic-release from Conventional Commits. Never edit a version by hand; CI writes it into every file listed in `plugin-release.json` and commits it back, and the MCP server, the widget, and the site's mock host read it from `package.json`. The pipeline lives in [`enkind/.github`](https://github.com/enkind/.github), shared by every Enkind plugin repository; change it there, not here.

Every commit subject follows Conventional Commits, and CI rejects any that does not. The type decides the next release:

- `fix:` is a patch: a fix, a packaging change, or any other change to installed plugin files, the MCP server, or the widget that preserves the public contract. Use it for such changes even when they are refactors, because a release is what gives hosts a new version to refresh their cached plugin.
- `feat:` is a minor: a backward-compatible addition to the public contract.
- `feat!:` or `fix!:`, or a `BREAKING CHANGE:` footer, is a major: an incompatible contract change.
- `docs:`, `ci:`, `chore:`, `test:`, and `refactor:` release nothing; use them only for changes that leave installed contents untouched.

Any change to the widget bundle also bumps `UI_VERSION` in `app/mcp/route.ts`, since hosts cache the resource by its URI. `widget/dist/` is a build artifact and must not be committed.

## Environments and releases

Work lands on `main`. Every push to `main` deploys to `https://napkin-neon-dev.vercel.app/mcp` and, when its commits warrant a release, publishes a `vX.Y.Z-dev.N` GitHub prerelease. `main`'s plugin manifests point at the dev server under the name "Napkin (Dev)".

Production at `https://napkin-neon.vercel.app/mcp` is the ChatGPT submission's server. It changes only through the manual Release run on `main` (`gh workflow run ci.yml --ref main`), which deploys and probes production before publishing the GitHub release and the `production` branch, where the manifests carry the production URL and name. Each release attaches `napkin-<version>.zip`, the archive to upload when the skill, manifest text, or icon changed. Never commit to `production`, never create version tags by hand, never run `vercel --prod` locally, and never release while a submission is in review if the release changes tools, schemas, instructions, or the widget.

## Verify

Run `pnpm lint`, `pnpm build`, and, against a running server, `pnpm test:client`. For site changes, check the page in a browser in both color schemes and draw on the napkin in the mock until the sketch lands in the thread.

## Local Codex CLI

For local plugin management on this Mac, use the signed CLI bundled with the app at `/Applications/ChatGPT.app/Contents/Resources/codex-cli/bin/codex`. Do not invoke `/usr/local/bin/codex`; macOS blocks that standalone installation and displays a malware warning.

## Commit ownership

The agent owns committing completed work. After each coherent user-requested task that changes the repository:

- Run the required validation and review the final diff.
- Stage only the files and hunks created for that task, preserving unrelated or pre-existing work.
- Create a clear, scoped Conventional Commits commit before reporting the task complete.
- Do not amend, rewrite, or push commits unless the user explicitly asks.

If validation fails or a clean task-only commit cannot be made safely, stop before committing and explain the blocker. A user request to leave changes uncommitted overrides this rule.
