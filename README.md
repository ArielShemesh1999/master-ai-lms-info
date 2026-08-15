# MasterAI LMS

> **A Hebrew, right-to-left learning platform for the MasterAI Spring 2026 cohort — 8 modules, 31 lessons, no database behind it.**

**Live:** [master-ai-lms.vercel.app](https://master-ai-lms.vercel.app)

Each lesson carries an overview with takeaways, a 12-18 beat transcript, a five-question quiz, a runnable code exercise and an interactive demo; the eight tool-facing lessons add a terminal - 155 quiz questions in all, sourced from Anthropic Academy courses and the platform docs, not synthetic filler.

## Running a whole course out of `localStorage`

No backend holds learner state. Progress, quiz scores, spaced-repetition boxes and the study plan live in versioned `localStorage` namespaces behind one export / import / clear-all path. Activity is logged in the **data layer** - inside `markLessonDone`, `saveQuizScore`, `Review.record` - not in UI handlers, so every present and future call site counts toward the streak and nothing double-counts. Review is a Leitner scheduler over the quiz bank on a 1/2/4/8/16-day ladder.

The one server-side piece is sign-in: a Vercel function comparing the password against a scrypt hash in an env var, constant-time, answering `503` if that var is missing - it fails closed, never open.

## Building a code runner and a chart kit inside a self-only CSP

The lesson terminal runs a real JS REPL, evaluating inside `public/js-sandbox.html` with `sandbox="allow-scripts"` and deliberately **without** `allow-same-origin` - opaque origin, so a pasted `localStorage.clear()` cannot reach the progress store. It must be a served file, not a `srcdoc` iframe: `srcdoc` inherits the parent CSP (`script-src 'self'`), which silently blocked the evaluator. As a real file it gets its own `default-src 'none'` header - and learner code has no network at all.

The same CSP forbids remote scripts, so charts are hand-rolled SVG - bar, line with crosshair, donut ring, heatmap, sparkline - sized by a `ResizeObserver`. Their six categorical hues were validated **as a set** against the app's cream surface: colour-vision-deficiency separation (worst adjacent pair 9.4 delta-E under protanopia) and a 3:1 contrast floor.

## Scoping the anti-bot probes to the platform they describe

Phones black-screened, and it was never CSS. A protection layer read *zero plugins* as proof of headless Chrome - mobile Chrome reports zero plugins too, so Android Chrome, WebView and in-app browsers got wiped; its devtools watcher hid the document when `|outerHeight - innerHeight| > 180`, which on mobile is browser chrome (iOS Safari idles at exactly 180) and which the keyboard blows past. Separately, `.side` went `position:fixed` while `.app` kept a two-column grid; fixed elements leave grid flow, so `.main` landed in the 0px track and the app rendered inside a zero-width box. Both were found by measuring a real 390px iPhone profile, not by reading stylesheets.

Desktop-only heuristics now sit behind a `pointer:coarse` guard - a false positive costs the entire app, far too expensive for a deterrent. Content is guarded at the edge: middleware screening scraper user-agents and CORS-proxy hosts.

## Verifying the answer keys adversarially, then the app live

Quiz keys are re-solved by a skeptic, not trusted from the author; the latest module's pass applied 23 corrections, including a question whose scenario could not produce its own answer. Each shipping pass then runs a build, a data-integrity scan (lesson and module counts, every quiz `correct` index in range) and a browser E2E harness - most recently 74 checks on desktop and a 390px profile, then 11 of 11 against production on a real iPhone, zero console errors.

## Building on Vite, React 18 and four dependencies

`Vite 5` · `React 18` · `Framer Motion` · `GSAP` · `localStorage` · Vercel Edge middleware · Vercel functions.

Four runtime dependencies: no chart library, no state library, no UI kit, no analytics.

Source private. Built by [@ArielShemesh1999](https://github.com/ArielShemesh1999).
