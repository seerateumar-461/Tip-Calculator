## 1. How to run

No build step, no dependencies to install.

# Step 1: Open cmd and go to Repository path
cd OneDrive\Documents\GitHub\Tip-Calculator

# Step 2: Rum this command
python -m http.server 8080

# Step 3: Open the Chrome and paste
http://localhost:8080/tCalculator.html

The entire app is one file: `tCalculator.html`.


---

## 2. Stack & design choices

**Stack:** Vanilla HTML/CSS/JS, no framework, no bundler. The task is a single form with live arithmetic — there's no state management problem that React would solve that a few variables and `addEventListener` don't handle in fewer lines. A zero-dependency single file is also the most reliably runnable thing a reviewer can open.

**Interaction: Preset buttons sit above the custom input, sharing the same field group.**
Instead of making tip presets a separate section, they sit inside the tip field as a 3-column grid above the text input. Clicking a preset writes its value into the input and highlights the button. This means the custom input and the presets are always in sync, you can't have "15% preset active" while the input shows "18." If you type a value that matches a preset (10, 15, 20), the button highlights automatically. This avoids the split-brain confusion of separate UI states.

---

## 3. Responsive & accessibility

**360px phone vs 1440px laptop:**
On a 360px phone the two-column layout collapses to a single column via a `@media (max-width: 600px)` breakpoint. The preset buttons use `grid-template-columns: repeat(3, 1fr)` and scale to fill the available width at any screen size, so they never overflow. `inputmode="decimal"` on the bill and tip fields and `inputmode="numeric"` on the people field trigger the appropriate mobile keyboard for each. The result panel appears below the inputs on mobile — the live-updating values are visible without any scrolling on a typical phone after filling in the first field.

On a 1440px desktop the max-width is capped at 760px and centred, so it doesn't stretch into an unreadable wide layout.

**Accessibility handled:**
- All inputs have explicit `<label>` elements linked via `for`/`id`. The preset button group has `role="group"` with `aria-label`. Error message elements use `role="alert"` and `aria-live="polite"` so screen readers announce validation feedback without interrupting the user.
- Each preset button carries `aria-pressed="true/false"` so a screen reader announces toggle state correctly.
- Tab order is natural: bill → preset buttons → custom tip input → people → reset. Enter on each input moves focus to the next field rather than submitting anything.
- Focus rings are preserved (visible `outline` on `:focus-visible`) — not suppressed for mouse users.
- The result region has `aria-live="polite"` and `aria-atomic="true"` so updates are read as a unit.


---

## 4. AI usage

I used Claude to help draft this project.

**Where AI was used:**

1. **Initial HTML/CSS structure** — asked for a dark terminal-aesthetic single-file tip calculator. The AI produced a working skeleton with IBM Plex Mono, a two-column grid, and styled inputs.

2. **The bump animation on result values** — the AI gave me a CSS class that used `transform: scale(1.1)` triggered by toggling the class with a `setTimeout`. I changed it in two ways: (a) I reduced the scale to `1.06` because 1.1 felt jumpy on a number that updates on every keystroke — the user is actively typing and a large pop on every character is visually noisy; (b) I added `void el.offsetWidth` before re-adding the class to force a reflow, which makes the animation re-trigger correctly if the value changes twice in quick succession (the AI's version swallowed the second trigger).

3. **Validation logic** — asked for inline validation for all three fields. The AI generated `window.alert()` calls as a first pass. I replaced them entirely with the `error-msg` `<p>` elements with CSS `max-height` / `opacity` transition, so errors animate in and out smoothly rather than flickering on/off with a hard `display: block` toggle.

4. **Rounding policy** — asked for options. AI listed round-to-nearest, round-up, and remainder-distribution. I chose `Math.ceil` (round-up to nearest paisa) and wrote the justification myself; the AI's framing of "so the group never underpays" was the deciding argument.

---

## 5. Honest gap

The per-person result shows one rounded-up number and a note saying "rounded up." What it doesn't show is the remainder: if 3 people split Rs 101 and the per-person share is Rs 33.67, the app shows Rs 33.67 × 3 = Rs 101.01, which is one paisa over the grand total. A rigorous splitter would show a breakdown like "2 people pay Rs 33.67, 1 person pays Rs 33.66" — distributing the rounding remainder so the total is exact.

With another day I'd add a small expandable breakdown row in the results panel: "2 × Rs 33.67 + 1 × Rs 33.66" — only shown when the ceiling-rounded total diverges from the grand total. It's the kind of detail that turns a toy into a tool someone actually uses at a restaurant table.
