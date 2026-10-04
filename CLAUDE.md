# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal learning repo (see README.md): the owner is a designer new to Git, GitHub and the command line. It holds small hand-written static web pages — no framework, no build step, no package.json, no tests, no linter.

- `index.html` — the home page ("Hello, Sam" bedtime note: night sky, moon, twinkling stars and floating hearts).
- `tea/index.html` — a separate page served at `/tea` ("Green tea, please").

Each page is fully self-contained: all CSS lives in an inline `<style>` block, any JS in an inline `<script>` at the end of `<body>`, and the only external resource is a Google Fonts stylesheet. Pages don't share styles or link to each other — add a new page as `<name>/index.html` so it's served at `/<name>`.

## Conventions used in the pages

- Colors are defined as CSS custom properties on `:root` and referenced via `var(--…)`.
- Fluid sizing with `clamp()`; layout is a centered CSS grid (`display: grid; place-items: center; min-height: 100vh`).
- Decorative elements created from JS get `aria-hidden="true"`.
- Every animated page includes a `@media (prefers-reduced-motion: reduce)` block that disables animations and shows the final state; JS-driven motion checks `matchMedia('(prefers-reduced-motion: reduce)')` too. Keep this when adding motion.

## Previewing and deploying

- Preview locally by opening the file in a browser (`open index.html`), or `npx serve .` to get the `/tea` route.
- Hosted on Vercel (project `hello-world`, account `fernandezcarolinna-7970`, Hobby plan; framework preset "Other", no `vercel.json`). Production: https://hello-world-delta-dun.vercel.app
- Changes land through GitHub PRs into `main` (remote `fernandezcarolina/hello-world`); PRs get a Vercel preview deployment and merging produces a production deployment. Manual deploy, if ever needed: `vercel --prod` from the repo root.
- The Vercel CLI is installed via Homebrew at `/opt/homebrew/bin/vercel`; non-login shells may need `/opt/homebrew/bin` added to `PATH`.

## Working with the owner

Explain steps in plain, non-technical language. Commands the owner must run themselves (browser logins, password prompts) should be quote-free and given one at a time.
