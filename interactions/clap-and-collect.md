# Clap & Collect — named interaction (saved snapshot)

**What it is:** the option-select micro-interaction in the Stimuler onboarding prototype.
On tap: the chip punches → its emoji lifts off and swells to ~76px just above the chip
(screen stays live, no blur veil) → the emoji **performs** (squash-stretch "clap" wiggle
with a pop) → shrinks and arcs up into the progress bar → bar **ripples + glows + collect
chime + screen pulse** as it swallows the emoji, and progress bumps.

Say "use Clap & Collect" / "revert to Clap & Collect" to restore exactly this.

Timing: rise 340ms → perform 560ms → fly 560ms → intake. ~1.7s total per pick.

## JS (drop-in `collect()` — depends on helpers below)

```js
/* Emoji performance collect — on tap the pick's emoji rises off the chip,
   scales up and DOES ITS THING (the clap claps, the ball bounces), then
   shrinks and flies up into the progress bar: ripple + glow on intake. */
async function collect(emoji, fromEl, bump) {
  const sr = screen.getBoundingClientRect();
  const fr = fromEl.getBoundingClientRect();
  const x0 = fr.left + fr.width / 2 - sr.left;
  const y0 = fr.top + fr.height / 2 - sr.top;
  // stage spot: floats just above the tapped chip (clamped inside the screen)
  const stageX = Math.max(70, Math.min(sr.width - 70, x0));
  const stageY = Math.max(120, y0 - 86);

  const el = document.createElement('div');
  el.className = 'fly-emoji';
  el.textContent = emoji;
  el.style.left = x0 + 'px';
  el.style.top = y0 + 'px';
  el.style.fontSize = '22px';
  screen.appendChild(el);

  // 1) rise off the chip and swell
  el.style.transition = `left ${340/SPEED}ms cubic-bezier(.2,.8,.3,1.15), top ${340/SPEED}ms cubic-bezier(.2,.8,.3,1.15), font-size ${340/SPEED}ms cubic-bezier(.2,.8,.3,1.15)`;
  void el.offsetWidth;
  el.style.left = stageX + 'px';
  el.style.top = stageY + 'px';
  el.style.fontSize = '76px';
  await wait(360);

  // 2) the performance — clap claps: squeeze-and-snap wiggle
  sfxPop();
  el.animate([
    { transform: 'translate(-50%,-50%) rotate(0deg) scale(1)' },
    { transform: 'translate(-50%,-50%) rotate(-16deg) scale(1.18, 0.92)', offset: 0.18 },
    { transform: 'translate(-50%,-50%) rotate(10deg) scale(0.94, 1.1)', offset: 0.38 },
    { transform: 'translate(-50%,-50%) rotate(-10deg) scale(1.12, 0.95)', offset: 0.58 },
    { transform: 'translate(-50%,-50%) rotate(5deg) scale(0.98, 1.04)', offset: 0.78 },
    { transform: 'translate(-50%,-50%) rotate(0deg) scale(1)' }
  ], { duration: 560 / SPEED, easing: 'ease-in-out' });
  await wait(600);

  // 3) scale down and fly up into the bar
  const { x: x1, y: y1 } = intakePoint();
  el.style.transition = `left ${560/SPEED}ms cubic-bezier(.5,.05,.5,.95), top ${560/SPEED}ms cubic-bezier(.25,.6,.2,1), font-size ${560/SPEED}ms ease-in`;
  void el.offsetWidth;
  el.style.left = x1 + 'px';
  el.style.top = y1 + 'px';
  el.style.fontSize = '14px';
  await wait(570);
  el.remove();
  spawnRipples(x1, y1);
  barIntake(bump);
  await wait(220);
}
```

## Required helpers (already in index.html)

```js
function barIntake(bump) {
  PROGRESS = Math.min(100, PROGRESS + bump);
  setProgress(PROGRESS);
  progressFill.classList.remove('glowpulse'); void progressFill.offsetWidth;
  progressFill.classList.add('glowpulse');
  screen.classList.remove('pulse'); void screen.offsetWidth;   // haptic stand-in
  screen.classList.add('pulse');
  sfxCollect();
}
function intakePoint() {
  const sr = screen.getBoundingClientRect();
  const bar = document.querySelector('.progress-bar');
  const br = bar.getBoundingClientRect();
  const fillR = progressFill.getBoundingClientRect();
  const x = Math.max(fillR.left - sr.left + 14, Math.min(fillR.right - sr.left + 10, br.right - sr.left - 16));
  const y = br.top + br.height / 2 - sr.top;
  return { x, y };
}
function spawnRipples(x, y) {
  for (let i = 0; i < 2; i++) {
    const r = document.createElement('div');
    r.className = 'bar-ripple';
    r.style.left = x + 'px'; r.style.top = y + 'px';
    r.style.animationDelay = (i * 140) + 'ms';
    screen.appendChild(r);
    setTimeout(() => r.remove(), 900 + i * 140);
  }
}
```

Selection punch on the chip itself: `pick.classList.add('selected', 'punch')`.

## CSS

```css
.fly-emoji { position: absolute; font-size: 22px; z-index: 72; pointer-events: none; transform: translate(-50%,-50%); }
.progress-fill.glowpulse { animation: barGlow 620ms ease-out; }
@keyframes barGlow {
  0%   { filter: brightness(1.7) saturate(1.2); box-shadow: 0 0 28px rgba(124,92,255,1), inset 0 1px 0 rgba(255,255,255,0.35); }
  100% { filter: brightness(1); }
}
.screen.pulse { animation: scrPulse 280ms ease-out; }
@keyframes scrPulse { 0% { filter: brightness(1); } 30% { filter: brightness(1.1); } 100% { filter: brightness(1); } }
.bar-ripple {
  position: absolute; z-index: 71; width: 26px; height: 26px; margin: -13px 0 0 -13px;
  border-radius: 50%; border: 2px solid rgba(205,180,255,0.95);
  pointer-events: none;
  animation: barRipple 640ms cubic-bezier(0.2,0.7,0.3,1) forwards;
}
@keyframes barRipple { 0% { opacity: 0.95; transform: scale(0.4); } 100% { opacity: 0; transform: scale(2.8); } }
.quick-chip.punch, .topic-chip.selected.punch { animation: chipPunch 380ms cubic-bezier(0.34,1.56,0.64,1); }
@keyframes chipPunch { 0% { transform: scale(0.9); } 55% { transform: scale(1.1); } 100% { transform: scale(1.04); } }
```

## Sound (WebAudio, unlocks on first user gesture)

`sfxPop()` at the performance beat, `sfxCollect()` inside `barIntake()` — see the
Sound section in index.html (`tone()`, ascending sine/triangle pairs).
