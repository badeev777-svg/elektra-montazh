# Repair Cost Calculator Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add an interactive electrical-work cost calculator section to the static `index.html` landing page, with an apartment (point-based) mode and a house/cottage (area-based) mode, that ends in a Telegram-link lead capture.

**Architecture:** Everything lives inside the existing single-file `index.html` (no build step, no server). New CSS rules go in the existing `<style>` block, new markup goes in a new `<section id="calculator">` between `#process` and `#pricing`, new JS goes in the existing `<script>` block near the other UI-behavior functions (`mob()`, reveal observer). Nav/mobile-menu/footer links follow the same pattern as the other four sections.

**Tech Stack:** Vanilla HTML/CSS/JS, no dependencies, no package manager (confirmed: no `package.json` in the repo).

**Spec:** `docs/superpowers/specs/2026-07-27-repair-cost-calculator-design.md`

---

## Testing approach

This is a static single-file site with no test runner, no `package.json`, no CI. "Testing" here means opening `index.html` directly in a browser (`file://` path works — no server needed) and manually verifying behavior, exactly as the spec's "Тестирование" section describes. Each implementation task ends with a concrete manual-verification step that states exactly what to click and what result to expect, so it's unambiguous rather than "test it works."

---

### Task 1: Add price data and calculator CSS

**Files:**
- Modify: `index.html` (style block — insert before the `/* ══════════════════ RESPONSIVE ══════════════════ */` comment, which is currently at line 491)

- [ ] **Step 1: Insert the calculator CSS block**

Open `index.html` and find this exact block (currently ends around line 489-491):

```css
    .proof-sep { width: 1px; height: 36px; background: rgba(255,255,255,0.09); }

    @keyframes fu { from{opacity:0;transform:translateY(22px)} to{opacity:1;transform:translateY(0)} }
```

Immediately after the `@keyframes fu` line above (and before `/* ══════════════════ RESPONSIVE ══════════════════ */`), insert:

```css

    /* ── CALCULATOR ── */
    .calc-box {
      background: var(--card); border: 1px solid rgba(255,255,255,0.05);
      border-radius: var(--r); padding: 40px; max-width: 640px;
    }
    .calc-toggle {
      display: flex; gap: 8px; margin-bottom: 32px;
      background: rgba(255,255,255,0.03); border-radius: 12px; padding: 4px;
    }
    .calc-toggle-btn {
      flex: 1; padding: 12px 16px; border-radius: 9px; border: none;
      background: transparent; color: var(--sub); font-weight: 700; font-size: 0.9rem;
      cursor: pointer; transition: background 0.25s, color 0.25s;
      font-family: inherit;
    }
    .calc-toggle-btn.active { background: linear-gradient(135deg, var(--accent), var(--a2)); color: #000; }
    .calc-panel { display: flex; flex-direction: column; gap: 18px; margin-bottom: 28px; }
    .calc-field { display: flex; flex-direction: column; gap: 8px; }
    .calc-field label { font-size: 0.85rem; color: var(--sub); font-weight: 600; }
    .calc-input {
      background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.1);
      border-radius: 10px; padding: 12px 14px; color: var(--text); font-size: 0.95rem;
      font-family: inherit; width: 100%;
    }
    .calc-input:focus { outline: none; border-color: rgba(251,191,36,.4); }
    .calc-btn { width: 100%; justify-content: center; }
    .calc-result { margin-top: 28px; padding-top: 28px; border-top: 1px solid rgba(255,255,255,0.06); }
    .calc-sum {
      font-size: 2.2rem; font-weight: 900; letter-spacing: -0.03em;
      background: linear-gradient(135deg, var(--accent), var(--a2));
      -webkit-background-clip: text; background-clip: text; -webkit-text-fill-color: transparent;
      margin-bottom: 8px;
    }
    .calc-note { font-size: 0.83rem; color: var(--muted); margin-bottom: 20px; }
    .calc-lead { display: flex; flex-direction: column; gap: 12px; }
```

- [ ] **Step 2: Verify the file still opens without CSS syntax errors**

Open `index.html` in a browser (double-click it or `file:///.../index.html`). Expected: the page renders exactly as before (calculator has no HTML yet, so no visual change) and the browser dev tools console (F12) shows no CSS parse errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add calculator CSS and price data placeholder styles"
```

---

### Task 2: Add calculator section markup

**Files:**
- Modify: `index.html` (insert new `<section>` between `#process` and `#pricing`, currently between line 906 `</section>` and line 908 `<!-- ════ PRICING ════ -->`)

- [ ] **Step 1: Insert the calculator section HTML**

Find this exact boundary in `index.html`:

```html
  </div>
</section>

<!-- ════ PRICING ════ -->
```

(This is the closing of `#process` immediately followed by the pricing section comment.)

Replace it with:

```html
  </div>
</section>

<!-- ════ CALCULATOR ════ -->
<section class="sec" id="calculator">
  <div class="rv-el">
    <div class="eyebrow">🧮 Калькулятор</div>
    <h2 class="sec-h">Рассчитайте <em>стоимость</em></h2>
    <p class="sec-p">Предварительный расчёт займёт 30 секунд. Точную смету составим после бесплатного выезда.</p>
  </div>

  <div class="calc-box rv-el">
    <div class="calc-toggle">
      <button type="button" class="calc-toggle-btn active" data-mode="apartment" onclick="calcSetMode('apartment')">Квартира</button>
      <button type="button" class="calc-toggle-btn" data-mode="house" onclick="calcSetMode('house')">Дом, коттедж</button>
    </div>

    <div class="calc-panel" id="calc-apartment">
      <div class="calc-field">
        <label for="calc-outlet">Розетки и выключатели, шт.</label>
        <input type="number" id="calc-outlet" class="calc-input" min="0" value="0" oninput="calcOnInput()">
      </div>
      <div class="calc-field">
        <label for="calc-light">Точки освещения, шт.</label>
        <input type="number" id="calc-light" class="calc-input" min="0" value="0" oninput="calcOnInput()">
      </div>
      <div class="calc-field">
        <label for="calc-panel-select">Электрощит</label>
        <select id="calc-panel-select" class="calc-input" onchange="calcOnInput()">
          <option value="none">Не требуется</option>
          <option value="install">Сборка нового</option>
          <option value="replace">Замена автоматов</option>
        </select>
      </div>
      <div class="calc-field">
        <label for="calc-cable">Штробление и кабель, м</label>
        <input type="number" id="calc-cable" class="calc-input" min="0" value="0" oninput="calcOnInput()">
      </div>
    </div>

    <div class="calc-panel" id="calc-house" style="display:none">
      <div class="calc-field">
        <label for="calc-area">Площадь дома, м²</label>
        <input type="number" id="calc-area" class="calc-input" min="0" value="0" oninput="calcOnInput()">
      </div>
    </div>

    <button type="button" class="btn-cta calc-btn" onclick="calcCalculate()">Рассчитать</button>

    <div class="calc-result" id="calc-result" style="display:none">
      <div class="calc-sum">≈ <span id="calc-sum-val">0</span> ₽</div>
      <p class="calc-note">Это ориентировочная оценка. Точную смету составим после бесплатного выезда.</p>
      <div class="calc-lead">
        <input type="text" id="calc-name" class="calc-input" placeholder="Ваше имя">
        <input type="tel" id="calc-phone" class="calc-input" placeholder="Телефон">
        <button type="button" class="btn-cta calc-btn" onclick="calcSendTelegram()">Отправить в Telegram</button>
      </div>
    </div>
  </div>
</section>

<!-- ════ PRICING ════ -->
```

- [ ] **Step 2: Verify the section renders**

Open `index.html` in a browser and scroll to (or navigate to `#calculator` in the URL bar). Expected: a new card titled "Рассчитайте стоимость" appears between the "Как мы работаем" and "Ориентировочные цены" sections, showing a two-button toggle ("Квартира" active by default), four input fields for the apartment mode, a "Рассчитать" button, and no result box yet. Clicking "Рассчитать" does nothing yet (JS not added) — that's expected at this step.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add calculator section markup"
```

---

### Task 3: Wire up calculator into navigation

**Files:**
- Modify: `index.html` (desktop nav `<ul class="nav-mid">` at line 664-669, mobile menu `<div class="mob" id="mob">` at line 650-656)

- [ ] **Step 1: Add the nav-mid link**

Find:

```html
  <ul class="nav-mid">
    <li><a href="#services">Услуги</a></li>
    <li><a href="#process">Как работаем</a></li>
    <li><a href="#pricing">Цены</a></li>
    <li><a href="#reviews">Отзывы</a></li>
  </ul>
```

Replace with:

```html
  <ul class="nav-mid">
    <li><a href="#services">Услуги</a></li>
    <li><a href="#process">Как работаем</a></li>
    <li><a href="#calculator">Калькулятор</a></li>
    <li><a href="#pricing">Цены</a></li>
    <li><a href="#reviews">Отзывы</a></li>
  </ul>
```

- [ ] **Step 2: Add the mobile menu link**

Find:

```html
<div class="mob" id="mob">
  <a href="#services" onclick="mob(0)">Услуги</a>
  <a href="#process"  onclick="mob(0)">Как работаем</a>
  <a href="#pricing"  onclick="mob(0)">Цены</a>
  <a href="#reviews"  onclick="mob(0)">Отзывы</a>
  <a href="#contact"  onclick="mob(0)" style="color:var(--accent)">Связаться</a>
</div>
```

Replace with:

```html
<div class="mob" id="mob">
  <a href="#services" onclick="mob(0)">Услуги</a>
  <a href="#process"  onclick="mob(0)">Как работаем</a>
  <a href="#calculator" onclick="mob(0)">Калькулятор</a>
  <a href="#pricing"  onclick="mob(0)">Цены</a>
  <a href="#reviews"  onclick="mob(0)">Отзывы</a>
  <a href="#contact"  onclick="mob(0)" style="color:var(--accent)">Связаться</a>
</div>
```

- [ ] **Step 3: Verify navigation works**

Open `index.html` in a browser. Expected: desktop nav bar shows "Калькулятор" between "Как работаем" and "Цены"; clicking it scrolls to the calculator section. Resize the window below 768px width (or use browser dev tools device mode), open the burger menu — expected: mobile menu shows "Калькулятор" in the same position and clicking it scrolls to the section and closes the menu.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add calculator link to desktop and mobile navigation"
```

---

### Task 4: Implement calculator logic in JS

**Files:**
- Modify: `index.html` (script block — insert before the `// ── REVEAL ──` comment, currently at line 1100)

- [ ] **Step 1: Insert the calculator JS**

Find this exact block in the `<script>` section:

```javascript
document.getElementById('brg').addEventListener('click', () => mob(!mobOpen));

// ── REVEAL ──
```

Replace with:

```javascript
document.getElementById('brg').addEventListener('click', () => mob(!mobOpen));

// ── CALCULATOR ──
const PRICES = {
  apartment: {
    outlet: { labor: 0, material: 0 },
    light:  { labor: 0, material: 0 },
    panel:  { none: 0, install: 0, replace: 0 },
    cable:  { labor: 0, material: 0 },
  },
  house: { perSqm: 0 },
};

let calcMode = 'apartment';
let calcResultShown = false;
let calcLastTotal = 0;

function calcSetMode(mode) {
  calcMode = mode;
  document.querySelectorAll('.calc-toggle-btn').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.mode === mode);
  });
  document.getElementById('calc-apartment').style.display = mode === 'apartment' ? '' : 'none';
  document.getElementById('calc-house').style.display = mode === 'house' ? '' : 'none';
  document.getElementById('calc-result').style.display = 'none';
  calcResultShown = false;
}

function calcApartmentTotal() {
  const outlet = Number(document.getElementById('calc-outlet').value) || 0;
  const light = Number(document.getElementById('calc-light').value) || 0;
  const panel = document.getElementById('calc-panel-select').value;
  const cable = Number(document.getElementById('calc-cable').value) || 0;
  const p = PRICES.apartment;
  let total = 0;
  total += outlet * (p.outlet.labor + p.outlet.material);
  total += light * (p.light.labor + p.light.material);
  total += p.panel[panel];
  total += cable * (p.cable.labor + p.cable.material);
  return total;
}

function calcHouseTotal() {
  const area = Number(document.getElementById('calc-area').value) || 0;
  return area * PRICES.house.perSqm;
}

function calcCalculate() {
  calcLastTotal = calcMode === 'apartment' ? calcApartmentTotal() : calcHouseTotal();
  document.getElementById('calc-sum-val').textContent = calcLastTotal.toLocaleString('ru-RU');
  document.getElementById('calc-result').style.display = '';
  calcResultShown = true;
}

function calcOnInput() {
  if (calcResultShown) calcCalculate();
}

function calcApartmentDetails() {
  const outlet = document.getElementById('calc-outlet').value || 0;
  const light = document.getElementById('calc-light').value || 0;
  const panelSelect = document.getElementById('calc-panel-select');
  const panelText = panelSelect.options[panelSelect.selectedIndex].text;
  const cable = document.getElementById('calc-cable').value || 0;
  return `Квартира. Розетки/выключатели: ${outlet} шт, точки света: ${light} шт, электрощит: ${panelText}, штробление/кабель: ${cable} м.`;
}

function calcHouseDetails() {
  const area = document.getElementById('calc-area').value || 0;
  return `Дом/коттедж. Площадь: ${area} м².`;
}

function calcSendTelegram() {
  const name = document.getElementById('calc-name').value.trim();
  const phone = document.getElementById('calc-phone').value.trim();
  const details = calcMode === 'apartment' ? calcApartmentDetails() : calcHouseDetails();
  const sum = calcLastTotal.toLocaleString('ru-RU');
  const text = `Заявка с калькулятора.\n${details}\nОриентировочная сумма: ${sum} ₽\nИмя: ${name || '-'}\nТелефон: ${phone || '-'}`;
  const url = `https://t.me/CowboyAleksandr?text=${encodeURIComponent(text)}`;
  window.open(url, '_blank', 'noopener');
}

// ── REVEAL ──
```

- [ ] **Step 2: Verify apartment mode calculation**

Open `index.html` in a browser, open dev tools console (F12) to watch for errors. In the calculator section (apartment mode, default): set "Розетки и выключатели" to `10`, leave the rest at `0`, click "Рассчитать". Expected: a result box appears showing "≈ 0 ₽" (correct, since `PRICES.apartment.outlet` is `{labor: 0, material: 0}` — a placeholder until real prices are supplied), no console errors, and the lead form (name/phone/"Отправить в Telegram") is visible below the sum.

- [ ] **Step 3: Verify house mode calculation and mode switching**

Click the "Дом, коттедж" toggle button. Expected: the apartment fields hide, a single "Площадь дома, м²" field shows, and the result box from the previous step disappears (hidden again). Enter `120` in the area field and click "Рассчитать". Expected: result box reappears showing "≈ 0 ₽" (placeholder price), no console errors.

- [ ] **Step 4: Verify live recalculation**

With the result box visible (from Step 3), change the area value to `200`. Expected: the displayed sum updates immediately without needing to click "Рассчитать" again (via `calcOnInput`/`oninput`).

- [ ] **Step 5: Verify Telegram link generation**

Fill "Ваше имя" with `Тест Тестов` and "Телефон" with `+79991234567`. Click "Отправить в Telegram". Expected: a new browser tab/window opens to a `t.me/CowboyAleksandr?text=...` URL; inspect the URL — it should be percent-encoded and, when decoded, contain the mode details ("Дом/коттедж. Площадь: 200 м²."), the sum, the name, and the phone number entered.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Implement calculator mode switching, pricing logic, and Telegram lead link"
```

---

### Task 5: Responsive styles for mobile

**Files:**
- Modify: `index.html` (mobile media query block, inside `@media (max-width: 768px)`, currently ending around line 626)

- [ ] **Step 1: Add calculator mobile overrides**

Find this exact block inside the `@media (max-width: 768px)` section:

```css
      /* Process */
      .steps { max-width: 100%; }
      .step-r { padding: 8px 0 28px; }
      .step-r h4 { font-size: 0.95rem; }
      .step-r p { font-size: 0.83rem; }

      /* Pricing */
```

Replace with:

```css
      /* Process */
      .steps { max-width: 100%; }
      .step-r { padding: 8px 0 28px; }
      .step-r h4 { font-size: 0.95rem; }
      .step-r p { font-size: 0.83rem; }

      /* Calculator */
      .calc-box { padding: 24px 20px; max-width: 100%; }
      .calc-sum { font-size: 1.8rem; }

      /* Pricing */
```

- [ ] **Step 2: Verify mobile layout**

Open `index.html` in a browser, switch dev tools to device mode at a width of ~375px, scroll to the calculator section. Expected: the calculator card fills the available width with reduced padding, the toggle buttons and inputs remain full-width and usable (no horizontal overflow/scroll), and the sum text is legibly sized (not overflowing its container).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add mobile responsive styles for calculator section"
```

---

### Task 6: Full manual regression pass

**Files:** none (verification only)

- [ ] **Step 1: Full click-through on desktop viewport**

Open `index.html` in a browser at full desktop width. Walk through: nav link → calculator scrolls into view → apartment mode fill all four fields with nonzero values → "Рассчитать" shows a sum and lead form → switch to house mode → fields reset visibility correctly (apartment fields hidden, result hidden) → fill area → "Рассчитать" → fill name/phone → "Отправить в Telegram" opens a correctly formed URL. Expected: no console errors at any step, no visual glitches, no layout shift breaking other sections (scroll past `#calculator` to confirm `#pricing` still renders correctly right after it).

- [ ] **Step 2: Full click-through on mobile viewport**

Repeat Step 1 at a ~375px viewport width via dev tools device mode, including opening the calculator via the mobile burger menu link. Expected: identical functional behavior, no horizontal scrollbars, all inputs remain tappable/usable size.

- [ ] **Step 3: Confirm existing sections are untouched**

Diff-review: run `git diff master~6 -- index.html` (or the equivalent range covering all six commits from this plan) and confirm every changed hunk is inside the new calculator CSS, the new `<section id="calculator">`, the two nav link insertions, or the new JS block — no accidental edits to `#pricing`, `#hero`, or other existing sections.

```bash
git diff HEAD~6 -- index.html | head -300
```

Expected: only additive hunks in the areas described above.

- [ ] **Step 4: Final commit (if any fixups were needed)**

If Steps 1-3 surfaced no issues, there is nothing to commit here — this task is verification-only. If a fixup was needed, commit it with a message describing exactly what was wrong and fixed.

---

## Follow-up (not part of this plan)

Per the spec's "Вне рамок" section: the `labor` values in `PRICES.apartment.*` and `PRICES.house.perSqm` are placeholder `0`s. Once the user supplies actual labor prices, and material prices are sourced from the (renamed) Leroy Merlin equivalent, someone must edit the `PRICES` object in `index.html` directly — no code changes needed, just filling in numbers.
