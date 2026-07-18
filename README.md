# Conversation → First Chat (Stimuler onboarding prototype)

A high-fidelity, self-contained prototype of Stimuler's **first-run onboarding**: a guided
conversation with Sarah that gets to know the user, teaches the interface, and lands them on
their **first impromptu spoken answer** — the moment that decides whether they come back.

**Live demo:** https://conversation-lead-to-first-chat.vercel.app

This is a design/interaction prototype (not production code) intended as a spec for engineering:
every animation, timing, and copy string is intentional. It runs as a single HTML file with
no build step.

## Run it

Just open `index.html` in a browser (Chrome recommended). No install, no server.

- **Sound** starts after the first click anywhere on the page (browser autoplay policy).
- Debug controls under the phone: **Replay**, **Pause**, **scene jumps 1–9**, **Speed** (0.5–2×).
- Deep-link a scene: `index.html?scene=5`.

## Files / assets

| File | What it is |
|---|---|
| `index.html` | The entire prototype — inline CSS + JS, no dependencies to build |
| `sarah.png` | Sarah greeting avatar (head/shoulders bust) |
| `sarah-thinking.png` | Sarah "reviewing her notes" avatar for the Assembly Loader (bg-removed, flipped) |
| `interactions/clap-and-collect.md` | Saved spec for the named **Clap & Collect** select micro-interaction |

**Fonts load from CDN** (not bundled): InterDisplay (`rsms.me/inter`), Geist + Noto Sans
Devanagari (Google Fonts). Requires network on first load; swap for self-hosted if needed.

## The flow (9 scenes)

1. **Greeting** — "Hi! I'm Sarah 👋" + "See these 3 stars up top? That's today's plan."
2. **Agenda tour** — a guided coach-mark walkthrough of the 3 stars (dim + circle + title/desc
   callout, EN + Hindi), setting the whole journey upfront.
3. **Translate coach** — teaches the translate button; opens a bottom translation sheet.
4. **Interests** — multi-select chip cloud (auto-advances after 3 picks, no CONTINUE button).
5. **Goal probe** — digs into their everyday-communication goal.
6. **Your day** — context for Sarah's examples.
7. **Gender** — quick single-select.
8. **Feeling check** — "How do you feel about speaking English?" → mood-matched reassurance
   right before the mic (the fear-killer).
9. **Impromptu** — the payoff. ⭐1 flies to the tray ("get to know you" done) → the
   **Assembly Loader** (Sarah thinks over her notes, personalising) becomes the first question →
   the user speaks their first real answer → ⭐2 flies to the tray.

The 3 header stars are a **progress contract**: Act 1 *Get to know you*, Act 2 *Your first
conversation*, Act 3 *Practice* (locked/teased). Stars are previewed in scene 2 and fill for
real at their milestones.

## Named interactions

- **Clap & Collect** — on option select, the emoji rises, performs (the clapboard "claps"),
  then flies into the progress bar which ripples/glows. Full spec in
  `interactions/clap-and-collect.md`.
- **Assembly Loader** — a generative loading bubble with an internal gradient glow that cycles
  Sarah-voice personalising lines echoing the user's answers, then resolves in place into the
  question. Sarah's "thinking" avatar sits above it and leaves as it resolves.

## Design intent (for implementation)

- Short bubbles (≤2 lines), simple B1 vocabulary, no idioms, warm + slow pacing.
- Bilingual coach tooltips on every new affordance.
- Every input states its purpose; every input visibly feeds the collector progress bar.
- Stuck-triggered hints (not preemptive); hint clears on submit before the loading state.
- Haptics: the screen-pulse on collect is a stand-in — wire real haptics on the collect/star beats.
