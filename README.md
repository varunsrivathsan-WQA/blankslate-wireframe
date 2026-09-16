# Blankslate Wireframe

A Claude Code / Cowork plugin that builds lo-fi wireframes as matched **Figma + HTML/CSS** deliverables from a plain-English brief, using WQA's 66-primitive Blankslate design system (grayscale + one link blue, real Lucide icons, shadcn/ui-parity radii).

## What it does

Ask Claude for a wireframe ("wireframe a checkout flow", "mock up a SaaS pricing page") and this plugin's `blankslate-wireframe` skill will:

1. Confirm which Figma file to build into, and check that file's component library and the bundled CSS agree with each other before building anything.
2. Map every element in your brief to one of Blankslate's 66 documented primitives (never an invented shape or class).
3. Build a real Figma frame out of actual component instances — proper auto-layout, real variant properties, nothing hand-drawn.
4. Build a matching self-contained HTML file using the same primitives' CSS classes.
5. Report back both deliverables, flagging anything it wasn't sure how to map.

## Requirements

This skill builds *into* an existing Figma file that already has the Blankslate component library set up — all 66 primitives, one per page, plus the Icon component set. It does not create that library from scratch. If you don't have a Blankslate library file yet, you'll need one duplicated/shared before this plugin can build a Figma side (it can still produce the HTML/CSS side on its own).

## What's included

- `skills/blankslate-wireframe/SKILL.md` — the full build process: primitive vocabulary, token rules, icon handling, Figma structural-quality requirements (auto-layout, true variant component sets), and the hard rules that keep both outputs in sync.
- `skills/blankslate-wireframe/references/blankslate-tokens.css` — the canonical design tokens (colors, type scale, spacing, radius) that every primitive references.
- `skills/blankslate-wireframe/references/blankslate.css` — the full primitive stylesheet (one `.wf-*` class per primitive).

## Installing

Drag the `.plugin` file into Claude Code or Cowork, or add this repo as a marketplace source and install `blankslate-wireframe` from it.

## Updating

This plugin bundles its own copies of the two CSS files so it works standalone. If the canonical Blankslate token/primitive files change, refresh the copies under `skills/blankslate-wireframe/references/` and bump the version in `.claude-plugin/plugin.json` before re-publishing.
