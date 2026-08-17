# Brainstorming-CIS

A standalone sandbox repository for **brainstorming, experiments, and test scaffolding**.

## Purpose

This repo is an independent companion to my main local project — separate codebase, but related in spirit. It exists as a low-stakes space to:

- **Brainstorm** — capture ideas, notes, and reference material (e.g. study notes, design sketches, question banks).
- **Experiment** — try out approaches, libraries, and patterns without touching the main project.
- **Build tests** — develop and run test scaffolding in isolation before promoting anything to the main repo.

Nothing here is production code. Treat it as a scratch space where breaking things is fine.

## How I work in this repo

I drive this repo through **remote sessions from the Claude Code mobile app** (Claude Code on the web). Each session starts from a fresh clone of the repo in an ephemeral container, so anything worth keeping must be committed and pushed.

## Structure

The repo is intentionally lightweight and will grow as needed. Current contents:

| Path | Description |
|------|-------------|
| `README.md` | This file. |
| `devops-interview-questions.md` | Study notes: 5 foundational DevOps interview questions with deep-dive answers. |
| `hauntology-liminal-horror.md` | Study notes: hauntology (Derrida → Fisher) and liminal horror — the Backrooms & the Farlands as expressions of our "cancelled future." |
| `hauntology-liminal-horror.html` | Self-contained, styled web version of the hauntology notes (readable in any browser; light/dark aware). |
| `hauntology-essay.md` | Companion essay: hauntology as one continuous argument (Derrida → Fisher → internet myths → media lineage), with an exhaustive cited bibliography. |
| `hauntology-essay.html` | Self-contained, styled web version of the essay, with clickable citation references. |
| `who-cancels-the-future.md` | Deep-dive companion: the *power-side* account — how the imagination is made to police itself (Gramsci → Marcuse → Foucault → Han → Fisher), with an exhaustive cited bibliography. |
| `who-cancels-the-future.html` | Self-contained, styled web version of the deep-dive, with clickable citation references. |

A test stack (language, test runner, linting) hasn't been chosen yet — it'll be added once the direction of the experiments is clearer.

## Conventions

- Brainstorming and reference material → Markdown documents.
- Experiments and test code → added under their own directories once a stack is picked.
- Keep commits small and descriptive; the git history doubles as a log of what was tried.
