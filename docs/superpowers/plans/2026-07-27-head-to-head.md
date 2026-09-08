# Head-to-Head (H2H) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a single-file SPA (`h2h_stats/index.html`) that lets users compare two BMX pilots head-to-head based on shared event results from `pilots-index.json`.

**Architecture:** Static HTML file with inline CSS and JS, same pattern as `sqorz_stats` and `category_stats`. Loads `pilots-index.json` from GitHub raw URL once, stores in memory, all computation is local. No build step, no external dependencies.

**Tech Stack:** Vanilla HTML/CSS/JS, localStorage for theme preference only.

---

## File Structure

```
sqorz_tools/h2h_stats/
  index.html    ← SPA complète (HTML + CSS + JS inline)
```

---

## Task 1: HTML Scaffold + Full CSS

**Files:**
- Create: `h2h_stats/index.html`

- [ ] **Step 1: Create the file with HTML skeleton + theme init script**

```html
<!doctype html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Sqorz Stats — Face-à-face pilotes</title>
<script>
  (() => {
    const saved = localStorage.getItem('theme');
    const isDark = saved === 'dark' ||
      (saved !== 'light' && window.matchMedia('(prefers-color-scheme: dark)').matches);
    document.documentElement.classList.toggle('dark', isDark);
    if (saved) document.documentElement.dataset.themePref = saved;
  })();
</script>
<style>
/* CSS ici — voir step 2 */
</style>
</head>
<body>
<!-- topbar + main ici — voir step 3 -->
</body>
</html>
```

- [ ] **Step 2: Copier le bloc CSS complet depuis `sqorz_stats/index.html`**

Copier intégralement les blocs CSS suivants depuis `sqorz_stats/index.html` (lignes ~21–1109) :
- `:root { }` (variables light)
- `:root.dark { }` (variables dark)
- `*, html, body`
- `:focus`, `:focus-visible`
- `.topbar`, `.topbar-inner`, `.brand-row`, `.brand`, `.theme-btn`, `.topbar h1`, `.topbar p`
- `body.has-results .topbar` (topbar sticky compact)
- `main`, `.card`, `.filters`, `.grid`, `.field`
- `.control`, `.btn`, `.btn-ghost`, `.hint`, `.status`, `.progress`
- `.skeleton`, `@keyframes shimmer`, `.skeleton-card`, `.skeleton-line`
- `.section-title`, `.stats`, `.stat`, `.stat::before`, `.stat::after`, toutes les `.stat.XXX`
- `.empty-state` (à créer si absent — voir step 2b)
- Tout le responsive `@media`

Ajouter en plus les styles spécifiques H2H :

```css
/* ── H2H Specific ─────────────────────────────── */
.autocomplete-wrap { position:relative }
.autocomplete-dropdown {
  position:absolute; top:calc(100% + 4px); left:0; right:0; z-index:100;
  background:var(--card); border:1px solid var(--line); border-radius:10px;
  box-shadow:var(--shadow-xl, var(--shadow)); max-height:240px; overflow-y:auto;
  display:none;
}
.autocomplete-dropdown.open { display:block }
.ac-item {
  padding:10px 14px; cursor:pointer; display:flex; flex-direction:column; gap:2px;
  border-bottom:1px solid var(--line); transition:background .1s;
}
.ac-item:last-child { border-bottom:none }
.ac-item:hover, .ac-item.focused { background:var(--accent-soft) }
.ac-item .ac-name { font-weight:600; font-size:14px; color:var(--ink) }
.ac-item .ac-meta { font-size:11.5px; color:var(--muted) }

.pilot-selected-badge {
  display:flex; align-items:center; gap:10px;
  padding:10px 14px; border-radius:10px;
  background:color-mix(in srgb,var(--accent) 8%, var(--card));
  border:1.5px solid color-mix(in srgb,var(--accent) 30%, var(--line));
}
.pilot-selected-badge .psb-name { font-weight:700; color:var(--ink); flex:1; min-width:0 }
.pilot-selected-badge .psb-meta { font-size:12px; color:var(--muted) }
.pilot-selected-badge .psb-clear {
  background:none; border:none; cursor:pointer; color:var(--muted);
  padding:4px 6px; border-radius:6px; font-size:13px; flex-shrink:0;
  transition:color .12s, background .12s;
}
.pilot-selected-badge .psb-clear:hover { color:var(--bad); background:color-mix(in srgb,var(--bad) 10%,transparent) }

.suggestions-list {
  display:flex; flex-direction:column; gap:8px; margin-top:12px;
}
.suggestion-item {
  display:flex; align-items:center; gap:12px;
  padding:10px 14px; border-radius:10px; cursor:pointer;
  background:var(--card-soft); border:1px solid var(--line);
  transition:border-color .12s, background .12s;
}
.suggestion-item:hover { border-color:var(--accent); background:var(--accent-soft) }
.suggestion-item .si-name { font-weight:600; color:var(--ink); flex:1; min-width:0 }
.suggestion-item .si-meta { font-size:12px; color:var(--muted) }
.suggestion-item .si-score {
  font-size:12px; font-weight:600; color:var(--muted);
  white-space:nowrap; text-align:right;
}
.suggestion-item .si-badge {
  font-size:11px; padding:2px 8px; border-radius:99px; white-space:nowrap;
  background:color-mix(in srgb,var(--sky) 18%, transparent);
  color:color-mix(in srgb,var(--sky) 65%, var(--ink));
}

/* H2H Header VS */
.h2h-header {
  display:grid; grid-template-columns:1fr auto 1fr; gap:12px; align-items:center;
  padding:24px; margin-bottom:0;
}
.h2h-pilot {
  display:flex; flex-direction:column; gap:4px;
}
.h2h-pilot.right { text-align:right; align-items:flex-end }
.h2h-pilot .hp-name { font-size:18px; font-weight:800; color:var(--ink) }
.h2h-pilot .hp-club { font-size:12px; color:var(--muted) }
.h2h-pilot .hp-wins {
  font-size:28px; font-weight:900; color:var(--accent);
  line-height:1;
}
.h2h-pilot .hp-wins.leader { color:var(--good) }
.h2h-vs {
  display:flex; flex-direction:column; align-items:center; gap:4px;
}
.h2h-vs .vs-label {
  font-size:20px; font-weight:900; color:var(--muted);
  letter-spacing:1px;
}
.h2h-vs .vs-sub { font-size:11px; color:var(--muted); text-align:center }

.h2h-table { width:100%; border-collapse:collapse; font-size:13.5px }
.h2h-table th {
  padding:8px 10px; text-align:left; font-size:11px; font-weight:600;
  text-transform:uppercase; letter-spacing:.5px; color:var(--muted);
  border-bottom:2px solid var(--line);
}
.h2h-table td { padding:10px; border-bottom:1px solid var(--line); vertical-align:middle }
.h2h-table tr:last-child td { border-bottom:none }
.h2h-table tr.dnx-row td { color:var(--muted); font-style:italic }
.h2h-table .rank-a, .h2h-table .rank-b { font-weight:700; font-variant-numeric:tabular-nums }
.h2h-table .rank-a.winner { color:var(--good) }
.h2h-table .rank-b.winner { color:var(--good) }
.h2h-table .win-icon { text-align:center; font-size:16px }
.h2h-table .ev-name { font-weight:600; color:var(--ink) }
.h2h-table .ev-cat { font-size:12px; color:var(--muted) }
.h2h-table .ev-date { font-size:12px; color:var(--muted); white-space:nowrap }

.invert-btn {
  display:flex; align-items:center; gap:6px; margin:0 auto;
  background:none; border:1px solid var(--line); border-radius:99px;
  padding:6px 14px; font-size:13px; color:var(--ink); cursor:pointer;
  transition:border-color .12s, background .12s;
}
.invert-btn:hover { border-color:var(--accent); background:var(--accent-soft) }

.cat-filter-row {
  display:flex; gap:8px; flex-wrap:wrap; align-items:center;
  padding:12px 18px; border-top:1px solid var(--line);
}
.cat-pill {
  padding:4px 12px; border-radius:99px; font-size:12px; font-weight:600;
  cursor:pointer; border:1.5px solid var(--line);
  background:var(--card-soft); color:var(--muted);
  transition:border-color .12s, background .12s, color .12s;
}
.cat-pill:hover { border-color:var(--accent) }
.cat-pill.active {
  border-color:var(--accent);
  background:color-mix(in srgb,var(--accent) 14%,transparent);
  color:var(--ink);
}

.empty-state {
  text-align:center; padding:48px 24px; color:var(--muted);
}
.empty-state .es-icon { font-size:40px; margin-bottom:12px }
.empty-state .es-title { font-size:16px; font-weight:600; color:var(--ink); margin-bottom:8px }
.empty-state .es-text { font-size:13px; line-height:1.6 }

@media (max-width:640px) {
  .h2h-header { grid-template-columns:1fr; text-align:center }
  .h2h-pilot.right { text-align:left; align-items:flex-start }
  .h2h-vs { flex-direction:row; padding:8px 0 }
}
```

- [ ] **Step 3: Écrire le HTML body — topbar + main**

```html
<body>
<div class="topbar">
  <div class="topbar-inner">
    <div class="brand-row">
      <div class="brand">
        <span class="logo" aria-hidden="true">
          <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="8" r="5"/>
            <circle cx="12" cy="8" r="1.6" fill="currentColor" stroke="none"/>
            <path d="M7 14 Q4 22 12 22 Q20 22 17 14"/>
          </svg>
        </span>
        Sqorz Stats
      </div>
      <button type="button" id="themeBtn" class="theme-btn" aria-label="Changer le thème" title="Changer le thème (auto / clair / sombre)"></button>
    </div>
    <h1>Face-à-face pilotes</h1>
    <p>Comparez deux pilotes BMX sur leurs confrontations directes — événements communs, victoires, classements.</p>
  </div>
</div>

<main>
  <!-- Carte chargement / statut -->
  <div class="card" id="statusCard" style="padding:18px">
    <div class="status loading" id="statusEl">Chargement des données…</div>
  </div>

  <!-- Sélection des pilotes -->
  <div class="card" id="selectionCard" hidden>
    <div style="padding:20px">
      <div style="display:grid; grid-template-columns:1fr auto 1fr; gap:16px; align-items:end" id="pilotGrid">
        <!-- Pilote A -->
        <div>
          <label class="field-label" style="font-size:11px;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.5px;display:block;margin-bottom:6px">Pilote A</label>
          <div class="autocomplete-wrap" id="wrapA">
            <input id="inputA" class="control" style="width:100%" type="text" placeholder="Nom du pilote…" autocomplete="off">
            <div class="autocomplete-dropdown" id="dropdownA"></div>
          </div>
          <div id="badgeA" hidden></div>
        </div>

        <!-- VS + Inverser -->
        <div style="display:flex;flex-direction:column;align-items:center;gap:6px;padding-bottom:2px">
          <span style="font-size:15px;font-weight:900;color:var(--muted);letter-spacing:1px">VS</span>
          <button type="button" class="invert-btn" id="invertBtn" disabled title="Inverser A et B">⇄</button>
        </div>

        <!-- Pilote B -->
        <div>
          <label class="field-label" style="font-size:11px;font-weight:600;color:var(--muted);text-transform:uppercase;letter-spacing:.5px;display:block;margin-bottom:6px">Pilote B</label>
          <div class="autocomplete-wrap" id="wrapB">
            <input id="inputB" class="control" style="width:100%" type="text" placeholder="Nom du pilote…" autocomplete="off" disabled>
            <div class="autocomplete-dropdown" id="dropdownB"></div>
          </div>
          <div id="badgeB" hidden></div>
        </div>
      </div>
    </div>

    <!-- Adversaires suggérés -->
    <div id="suggestionsSection" hidden style="padding:0 20px 20px">
      <div class="section-title" style="margin-top:4px">
        Adversaires fréquents
        <span class="meta" id="suggestionsSubtitle"></span>
      </div>
      <div class="suggestions-list" id="suggestionsList"></div>
    </div>
  </div>

  <!-- Résultats H2H -->
  <div id="h2hResults"></div>
</main>

<script>/* JS — voir tasks suivantes */</script>
</body>
```

- [ ] **Step 4: Vérifier visuellement dans le navigateur**

Ouvrir `h2h_stats/index.html` dans Firefox/Chrome. Vérifier :
- Topbar bleu-acier avec dégradé, titre "Face-à-face pilotes"
- Bouton thème visible en haut à droite
- Message "Chargement des données…" avec spinner visible dans la `.card`

---

## Task 2: JS — Utilitaires + chargement de l'index

**Files:**
- Modify: `h2h_stats/index.html` (bloc `<script>` principal)

- [ ] **Step 1: Écrire le bloc `<script>` avec les utilitaires de base**

Remplacer le commentaire `/* JS — voir tasks suivantes */` par :

```javascript
(() => {
'use strict';

const INDEX_URL = 'https://raw.githubusercontent.com/LudSoc/sqorz-stats/main/pilots-index.json';

// ── Utilitaires ──────────────────────────────────
const norm = s => (s || '')
  .toString()
  .normalize('NFD').replace(/[̀-ͯ]/g, '')
  .toLowerCase()
  .replace(/[^a-z0-9 ]+/g, ' ')
  .trim()
  .replace(/\s+/g, ' ');

const escape = s => String(s ?? '').replace(/[&<>"']/g, c =>
  ({ '&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;' }[c])
);

function humanError(e) {
  if (!e) return 'Erreur inconnue.';
  if (e.name === 'AbortError') return 'Annulé.';
  const msg = e.message || String(e);
  if (/HTTP 5\d\d/.test(msg)) return 'Service momentanément indisponible.';
  if (/HTTP 4\d\d/.test(msg)) return 'Requête refusée.';
  if (/network|fetch failed|Failed to fetch|NetworkError/i.test(msg)) return 'Pas de réseau. Vérifie ta connexion.';
  return msg;
}

function specialLabel(r) {
  if (r == null || r < 100000) return null;
  if (r >= 103000) return 'DSQ';
  if (r >= 101000) return 'DNS';
  return 'DNF';
}

// ── État global ──────────────────────────────────
let pilotsIndex = null; // l'index expansé
let pilotA = null;      // { firstName, lastName, groupName, normKey }
let pilotB = null;      // idem

// ── DOM ──────────────────────────────────────────
const statusCard   = document.getElementById('statusCard');
const statusEl     = document.getElementById('statusEl');
const selectionCard = document.getElementById('selectionCard');
const inputA       = document.getElementById('inputA');
const inputB       = document.getElementById('inputB');
const dropdownA    = document.getElementById('dropdownA');
const dropdownB    = document.getElementById('dropdownB');
const badgeA       = document.getElementById('badgeA');
const badgeB       = document.getElementById('badgeB');
const invertBtn    = document.getElementById('invertBtn');
const suggestionsSection = document.getElementById('suggestionsSection');
const suggestionsSubtitle = document.getElementById('suggestionsSubtitle');
const suggestionsList = document.getElementById('suggestionsList');
const h2hResults   = document.getElementById('h2hResults');

// ── Expansion de l'index (format compact → complet) ─
function expandIndex(idx) {
  for (const ev of (idx.events || [])) {
    for (const cls of (ev.classes || [])) {
      for (const c of (cls.competitors || [])) {
        c.firstName = c.fn; c.lastName = c.ln; c.groupName = c.gn;
      }
    }
  }
  return idx;
}

// ── Chargement ───────────────────────────────────
async function loadIndex() {
  statusEl.className = 'status loading';
  statusEl.textContent = 'Chargement des données…';
  statusCard.hidden = false;
  selectionCard.hidden = true;
  try {
    const res = await fetch(INDEX_URL);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    pilotsIndex = expandIndex(await res.json());
    statusCard.hidden = true;
    selectionCard.hidden = false;
    inputA.focus();
  } catch (e) {
    statusEl.className = 'status error';
    statusEl.textContent = 'Impossible de charger les données : ' + humanError(e);
  }
}

loadIndex();
})();
```

- [ ] **Step 2: Vérifier dans le navigateur**

Ouvrir la page. Après ~1–3 secondes (selon la connexion) :
- La `.card` de statut disparaît
- La `.card` de sélection apparaît avec les deux champs de saisie et "VS"
- En cas d'erreur réseau : message d'erreur rouge visible

---

## Task 3: JS — Autocomplétion des pilotes

**Files:**
- Modify: `h2h_stats/index.html` (même bloc `<script>`, avant `loadIndex()`)

- [ ] **Step 1: Écrire la fonction de recherche dans l'index**

```javascript
// Cherche les pilotes correspondant à la requête.
// Retourne un tableau { firstName, lastName, groupName, normKey, eventCount }.
// Les doublons (même normKey) sont dédupliqués — on garde le plus récent.
function searchPilots(query, excludeNormKey = null) {
  if (!pilotsIndex || !norm(query)) return [];
  const tokens = norm(query).split(' ').filter(Boolean);
  const map = new Map(); // normKey → entry
  for (const ev of (pilotsIndex.events || [])) {
    for (const cls of (ev.classes || [])) {
      for (const c of (cls.competitors || [])) {
        const key = norm((c.firstName || '') + ' ' + (c.lastName || '')).trim();
        if (!key) continue;
        if (excludeNormKey && key === excludeNormKey) continue;
        if (!tokens.every(t => key.includes(t))) continue;
        if (!map.has(key)) {
          map.set(key, { firstName: c.firstName || '', lastName: c.lastName || '', groupName: c.groupName || '', normKey: key, eventCount: 0 });
        }
        map.get(key).eventCount++;
      }
    }
  }
  return [...map.values()]
    .sort((a, b) => b.eventCount - a.eventCount)
    .slice(0, 12);
}
```

- [ ] **Step 2: Écrire le composant autocomplétion générique**

```javascript
// Crée la logique d'autocomplétion pour un champ donné.
// onSelect(pilot) est appelé quand un pilote est choisi.
// getExcludeKey() retourne la normKey à exclure (le pilote déjà sélectionné dans l'autre champ).
function setupAutocomplete(input, dropdown, onSelect, getExcludeKey) {
  let focusedIdx = -1;

  function updateDropdown(items) {
    focusedIdx = -1;
    if (!items.length) { dropdown.classList.remove('open'); dropdown.innerHTML = ''; return; }
    dropdown.innerHTML = items.map((p, i) =>
      `<div class="ac-item" data-idx="${i}" tabindex="-1">
        <span class="ac-name">${escape(p.firstName + ' ' + p.lastName)}</span>
        ${p.groupName ? `<span class="ac-meta">${escape(p.groupName)}</span>` : ''}
      </div>`
    ).join('');
    dropdown.classList.add('open');
    dropdown._items = items;
    dropdown.querySelectorAll('.ac-item').forEach((el, i) => {
      el.addEventListener('mousedown', e => { e.preventDefault(); onSelect(items[i]); closeDropdown(); });
    });
  }

  function closeDropdown() { dropdown.classList.remove('open'); dropdown.innerHTML = ''; focusedIdx = -1; }

  function moveFocus(delta) {
    const items = dropdown.querySelectorAll('.ac-item');
    if (!items.length) return;
    items[focusedIdx]?.classList.remove('focused');
    focusedIdx = (focusedIdx + delta + items.length) % items.length;
    items[focusedIdx]?.classList.add('focused');
    items[focusedIdx]?.scrollIntoView({ block:'nearest' });
  }

  input.addEventListener('input', () => {
    const results = searchPilots(input.value, getExcludeKey());
    updateDropdown(results);
  });

  input.addEventListener('keydown', e => {
    if (e.key === 'ArrowDown') { e.preventDefault(); moveFocus(1); }
    else if (e.key === 'ArrowUp') { e.preventDefault(); moveFocus(-1); }
    else if (e.key === 'Enter') {
      const focused = dropdown.querySelector('.ac-item.focused') || dropdown.querySelector('.ac-item');
      if (focused) {
        const idx = +focused.dataset.idx;
        onSelect(dropdown._items[idx]);
        closeDropdown();
        e.preventDefault();
      }
    }
    else if (e.key === 'Escape') closeDropdown();
  });

  document.addEventListener('click', e => {
    if (!input.contains(e.target) && !dropdown.contains(e.target)) closeDropdown();
  });
}
```

- [ ] **Step 3: Écrire la fonction d'affichage du badge pilote sélectionné**

```javascript
function renderBadge(pilot) {
  const fullName = escape(pilot.firstName + ' ' + pilot.lastName);
  const club = pilot.groupName ? `<span class="psb-meta">${escape(pilot.groupName)}</span>` : '';
  return `<div class="pilot-selected-badge">
    <div class="psb-name">${fullName}</div>
    ${club}
    <button type="button" class="psb-clear" aria-label="Effacer">✕</button>
  </div>`;
}
```

- [ ] **Step 4: Connecter les champs A et B à leur logique de sélection**

```javascript
function selectPilotA(pilot) {
  pilotA = pilot;
  inputA.value = '';
  inputA.hidden = true;
  badgeA.innerHTML = renderBadge(pilot);
  badgeA.hidden = false;
  badgeA.querySelector('.psb-clear').addEventListener('click', clearPilotA);
  inputB.disabled = false;
  inputB.focus();
  invertBtn.disabled = !(pilotA && pilotB);
  onPilotASelected();
}

function clearPilotA() {
  pilotA = null;
  inputA.value = '';
  inputA.hidden = false;
  badgeA.hidden = true;
  badgeA.innerHTML = '';
  inputB.disabled = true;
  inputB.value = '';
  invertBtn.disabled = true;
  clearPilotB();
  suggestionsSection.hidden = true;
  h2hResults.innerHTML = '';
  inputA.focus();
}

function selectPilotB(pilot) {
  pilotB = pilot;
  inputB.value = '';
  inputB.hidden = true;
  badgeB.innerHTML = renderBadge(pilot);
  badgeB.hidden = false;
  badgeB.querySelector('.psb-clear').addEventListener('click', clearPilotB);
  invertBtn.disabled = false;
  computeAndRenderH2H();
}

function clearPilotB() {
  pilotB = null;
  inputB.value = '';
  inputB.hidden = false;
  badgeB.hidden = true;
  badgeB.innerHTML = '';
  invertBtn.disabled = true;
  h2hResults.innerHTML = '';
  inputB.focus();
}

setupAutocomplete(inputA, dropdownA, selectPilotA, () => pilotB?.normKey ?? null);
setupAutocomplete(inputB, dropdownB, selectPilotB, () => pilotA?.normKey ?? null);

invertBtn.addEventListener('click', () => {
  const tmp = pilotA;
  clearPilotA();
  // Réinitialiser proprement puis réaffecter dans l'ordre B → A → B
  // On recrée les sélections dans le bon sens
  selectPilotA(pilotB ?? tmp);
  selectPilotB(tmp);
});
```

Note : `onPilotASelected()` et `computeAndRenderH2H()` sont définies dans les tâches suivantes. Les déclarer vides pour l'instant :

```javascript
function onPilotASelected() {}
function computeAndRenderH2H() {}
```

- [ ] **Step 5: Vérifier dans le navigateur**

- Taper "dupont" dans le champ A → dropdown avec résultats
- Cliquer sur un résultat → badge apparaît, champ A disparaît, champ B s'active
- Cliquer ✕ sur le badge → retour au champ A vide
- Taper dans champ B → le pilote A est exclu des suggestions

---

## Task 4: JS — Calcul et affichage des adversaires suggérés

**Files:**
- Modify: `h2h_stats/index.html`

- [ ] **Step 1: Écrire la fonction de calcul des adversaires fréquents**

```javascript
// Retourne les N adversaires les plus fréquents de `pilot` (même eventId + même perpetualClassCode).
// Pour chaque adversaire : normKey, name, groupName, commonEvents, winsA, winsB.
function computeTopOpponents(pilot, maxN = 10) {
  if (!pilotsIndex || !pilot) return [];
  const pilotKey = pilot.normKey;

  // Map normKey → { name, groupName, events: Set<eventId+cat>, winsA, winsB }
  const opponents = new Map();

  for (const ev of (pilotsIndex.events || [])) {
    for (const cls of (ev.classes || [])) {
      // Chercher le pilote A dans cette classe
      const pilotEntry = cls.competitors.find(c =>
        norm((c.firstName || '') + ' ' + (c.lastName || '')).trim() === pilotKey
      );
      if (!pilotEntry) continue;

      const pilotRank = typeof pilotEntry.rank === 'number' ? pilotEntry.rank : null;

      for (const c of cls.competitors) {
        const oppKey = norm((c.firstName || '') + ' ' + (c.lastName || '')).trim();
        if (!oppKey || oppKey === pilotKey) continue;

        const confrontationKey = ev.event.eventId + '|' + cls.perpetualClassCode;
        if (!opponents.has(oppKey)) {
          opponents.set(oppKey, {
            firstName: c.firstName || '',
            lastName: c.lastName || '',
            normKey: oppKey,
            groupName: c.groupName || '',
            eventKeys: new Set(),
            winsA: 0, winsB: 0,
          });
        }
        const opp = opponents.get(oppKey);
        if (opp.eventKeys.has(confrontationKey)) continue;
        opp.eventKeys.add(confrontationKey);

        // Compter les victoires seulement si les deux ont un rang valide
        const oppRank = typeof c.rank === 'number' ? c.rank : null;
        if (pilotRank != null && pilotRank < 100000 && oppRank != null && oppRank < 100000) {
          if (pilotRank < oppRank) opp.winsA++;
          else if (oppRank < pilotRank) opp.winsB++;
        }
      }
    }
  }

  return [...opponents.values()]
    .sort((a, b) => b.eventKeys.size - a.eventKeys.size)
    .slice(0, maxN)
    .map(o => ({ ...o, commonEvents: o.eventKeys.size }));
}
```

- [ ] **Step 2: Écrire la fonction de rendu des suggestions**

```javascript
function renderSuggestions(opponents) {
  if (!opponents.length) {
    suggestionsList.innerHTML = '<div class="empty-state" style="padding:16px 0"><div class="es-text">Aucun adversaire trouvé pour ce pilote.</div></div>';
    return;
  }
  suggestionsList.innerHTML = opponents.map(opp => {
    const name = escape(opp.firstName + ' ' + opp.lastName);
    const club = opp.groupName ? escape(opp.groupName) : '';
    const score = `${opp.winsA} – ${opp.winsB}`;
    return `<div class="suggestion-item" role="button" tabindex="0" data-norm-key="${escape(opp.normKey)}">
      <div style="flex:1;min-width:0">
        <div class="si-name">${name}</div>
        <div class="si-meta">${club}</div>
      </div>
      <div class="si-score">
        <div class="si-badge">${opp.commonEvents} événement(s)</div>
        <div style="margin-top:3px;font-size:11px;color:var(--muted)">${score}</div>
      </div>
    </div>`;
  }).join('');

  // Attacher les événements de clic
  suggestionsList.querySelectorAll('.suggestion-item').forEach((el, i) => {
    const handler = () => selectPilotB(opponents[i]);
    el.addEventListener('click', handler);
    el.addEventListener('keydown', e => { if (e.key === 'Enter' || e.key === ' ') handler(); });
  });
}
```

- [ ] **Step 3: Implémenter `onPilotASelected`**

Remplacer le stub vide par :

```javascript
function onPilotASelected() {
  const opponents = computeTopOpponents(pilotA);
  suggestionsSubtitle.textContent = `${opponents.length} adversaire(s) fréquent(s)`;
  suggestionsSection.hidden = false;
  renderSuggestions(opponents);
}
```

- [ ] **Step 4: Vérifier dans le navigateur**

- Sélectionner un pilote A → section "Adversaires fréquents" apparaît
- Liste de pilotes avec nombre d'événements communs et bilan préliminaire
- Cliquer sur une suggestion → badge B se remplit, section suggestions reste visible

---

## Task 5: JS — Calcul du bilan H2H

**Files:**
- Modify: `h2h_stats/index.html`

- [ ] **Step 1: Écrire la fonction de calcul des confrontations**

```javascript
// Retourne la liste des confrontations directes entre pilotA et pilotB.
// Une confrontation = même eventId + même perpetualClassCode + les deux ont un rank renseigné.
function computeConfrontations(pilotA, pilotB) {
  if (!pilotsIndex || !pilotA || !pilotB) return [];
  const keyA = pilotA.normKey;
  const keyB = pilotB.normKey;
  const result = [];

  for (const ev of (pilotsIndex.events || [])) {
    for (const cls of (ev.classes || [])) {
      const entryA = cls.competitors.find(c =>
        norm((c.firstName || '') + ' ' + (c.lastName || '')).trim() === keyA
      );
      const entryB = cls.competitors.find(c =>
        norm((c.firstName || '') + ' ' + (c.lastName || '')).trim() === keyB
      );
      if (!entryA || !entryB) continue;
      if (typeof entryA.rank !== 'number' || typeof entryB.rank !== 'number') continue;

      result.push({
        event: ev.event,
        account: ev.account,
        cls: { className: cls.className, perpetualClassCode: cls.perpetualClassCode },
        rankA: entryA.rank,
        rankB: entryB.rank,
      });
    }
  }

  // Tri par date décroissante
  return result.sort((a, b) => (b.event.eventDate || '').localeCompare(a.event.eventDate || ''));
}
```

- [ ] **Step 2: Écrire la fonction de calcul des stats agrégées**

```javascript
// Calcule les statistiques globales à partir d'une liste de confrontations.
function computeH2HStats(confrontations) {
  let winsA = 0, winsB = 0, ties = 0, validCount = 0;
  const categories = new Set();
  let firstDate = '', lastDate = '';

  for (const c of confrontations) {
    const dateStr = c.event.eventDate || '';
    if (dateStr) {
      if (!firstDate || dateStr < firstDate) firstDate = dateStr;
      if (!lastDate || dateStr > lastDate) lastDate = dateStr;
    }
    categories.add(c.cls.perpetualClassCode || c.cls.className || '?');

    const specA = specialLabel(c.rankA);
    const specB = specialLabel(c.rankB);
    if (specA || specB) continue; // résultat spécial : ne compte pas dans les victoires
    validCount++;
    if (c.rankA < c.rankB) winsA++;
    else if (c.rankB < c.rankA) winsB++;
    else ties++;
  }

  return { winsA, winsB, ties, validCount, total: confrontations.length, categories: [...categories], firstDate, lastDate };
}
```

- [ ] **Step 3: Vérifier la logique (test console)**

Dans la console du navigateur, après avoir sélectionné A et B :

```javascript
// Exemple de vérification manuelle dans la console
const c = computeConfrontations(pilotA, pilotB);
console.log('Confrontations:', c.length);
console.log('Stats:', computeH2HStats(c));
```

Vérifier que les confrontations retournées sont cohérentes (même catégorie, les deux pilotes présents).

---

## Task 6: JS — Rendu du bilan H2H

**Files:**
- Modify: `h2h_stats/index.html`

- [ ] **Step 1: Écrire la fonction de formatage de date**

```javascript
function fmtDate(iso) {
  if (!iso) return '—';
  const [y, m, d] = iso.slice(0, 10).split('-');
  return `${d}/${m}/${y}`;
}
```

- [ ] **Step 2: Écrire la fonction de rendu du header VS**

```javascript
function renderH2HHeader(stats) {
  const aLeads = stats.winsA > stats.winsB;
  const bLeads = stats.winsB > stats.winsA;
  const nameA = escape(pilotA.firstName + ' ' + pilotA.lastName);
  const nameB = escape(pilotB.firstName + ' ' + pilotB.lastName);
  const clubA = pilotA.groupName ? `<div class="hp-club">${escape(pilotA.groupName)}</div>` : '';
  const clubB = pilotB.groupName ? `<div class="hp-club">${escape(pilotB.groupName)}</div>` : '';
  const period = stats.firstDate
    ? `${fmtDate(stats.firstDate)} → ${fmtDate(stats.lastDate)}`
    : '—';

  return `<div class="h2h-header">
    <div class="h2h-pilot">
      <div class="hp-name"><a class="sqorz-link" href="../sqorz_stats/index.html?name=${encodeURIComponent(pilotA.firstName + ' ' + pilotA.lastName)}" target="_blank">${nameA}</a></div>
      ${clubA}
      <div class="hp-wins${aLeads?' leader':''}">${stats.winsA}</div>
      <div style="font-size:12px;color:var(--muted)">victoire(s)</div>
    </div>
    <div class="h2h-vs">
      <div class="vs-label">VS</div>
      <div class="vs-sub">${stats.total} confrontation(s)<br>${period}</div>
    </div>
    <div class="h2h-pilot right">
      <div class="hp-name"><a class="sqorz-link" href="../sqorz_stats/index.html?name=${encodeURIComponent(pilotB.firstName + ' ' + pilotB.lastName)}" target="_blank">${nameB}</a></div>
      ${clubB}
      <div class="hp-wins${bLeads?' leader':''}">${stats.winsB}</div>
      <div style="font-size:12px;color:var(--muted)">victoire(s)</div>
    </div>
  </div>`;
}
```

- [ ] **Step 3: Écrire la fonction de rendu des stats globales**

```javascript
function renderH2HStatsCards(stats) {
  const tiesHtml = stats.ties > 0
    ? `<div class="stat teal"><div class="k">Égalités</div><div class="v">${stats.ties}</div></div>` : '';
  const catsHtml = stats.categories.length > 1
    ? `<div class="stat blueberry"><div class="k">Catégories</div><div class="v">${stats.categories.length}</div><div class="sub">${stats.categories.slice(0,3).map(escape).join(', ')}${stats.categories.length>3?'…':''}</div></div>` : '';

  return `<div class="stats" style="margin:16px 18px">
    <div class="stat sky"><div class="k">Confrontations</div><div class="v">${stats.total}</div><div class="sub">${stats.validCount} avec classements valides</div></div>
    <div class="stat good"><div class="k">Victoires A</div><div class="v">${stats.winsA}</div></div>
    <div class="stat rose"><div class="k">Victoires B</div><div class="v">${stats.winsB}</div></div>
    ${tiesHtml}
    ${catsHtml}
  </div>`;
}
```

- [ ] **Step 4: Écrire la fonction de rendu du tableau des confrontations**

```javascript
function renderH2HTable(confrontations) {
  if (!confrontations.length) return '';
  const nameA = escape(pilotA.firstName + ' ' + pilotA.lastName);
  const nameB = escape(pilotB.firstName + ' ' + pilotB.lastName);

  const rows = confrontations.map(c => {
    const specA = specialLabel(c.rankA);
    const specB = specialLabel(c.rankB);
    const isDnx = specA || specB;

    const rankACellContent = specA
      ? `<span class="special-result" style="font-size:.85em;font-weight:700">${specA}</span>`
      : (c.rankA === 1 ? '🥇' : c.rankA === 2 ? '🥈' : c.rankA === 3 ? '🥉' : `${c.rankA}e`);
    const rankBCellContent = specB
      ? `<span class="special-result" style="font-size:.85em;font-weight:700">${specB}</span>`
      : (c.rankB === 1 ? '🥇' : c.rankB === 2 ? '🥈' : c.rankB === 3 ? '🥉' : `${c.rankB}e`);

    const aWins = !isDnx && c.rankA < c.rankB;
    const bWins = !isDnx && c.rankB < c.rankA;
    const winIcon = isDnx ? '' : aWins ? '← 🏆' : bWins ? '🏆 →' : '=';

    const eventName = c.account?.accountCode
      ? `<a class="sqorz-link" href="https://sqorz.com/${c.account.accountCode}/events/${c.event.eventId}" target="_blank" rel="noopener">${escape(c.event.eventName || c.event.eventId)}</a>`
      : escape(c.event.eventName || c.event.eventId);

    return `<tr class="${isDnx ? 'dnx-row' : ''}">
      <td class="ev-date">${fmtDate(c.event.eventDate)}</td>
      <td class="ev-name">${eventName}</td>
      <td class="ev-cat">${escape(c.cls.className || c.cls.perpetualClassCode || '')}</td>
      <td class="rank-a${aWins?' winner':''}">${rankACellContent}</td>
      <td class="rank-b${bWins?' winner':''}">${rankBCellContent}</td>
      <td class="win-icon">${winIcon}</td>
    </tr>`;
  }).join('');

  return `<div style="overflow-x:auto;padding:0 18px 18px">
    <table class="h2h-table">
      <thead>
        <tr>
          <th>Date</th>
          <th>Événement</th>
          <th>Catégorie</th>
          <th>${nameA}</th>
          <th>${nameB}</th>
          <th></th>
        </tr>
      </thead>
      <tbody>${rows}</tbody>
    </table>
  </div>`;
}
```

- [ ] **Step 5: Écrire la fonction de rendu du filtre par catégorie**

```javascript
function renderCatFilter(categories, activeCat) {
  if (categories.length <= 1) return '';
  const pills = [''].concat(categories).map(cat => {
    const label = cat === '' ? 'Toutes' : escape(cat);
    const isActive = cat === activeCat;
    return `<button type="button" class="cat-pill${isActive?' active':''}" data-cat="${escape(cat)}">${label}</button>`;
  }).join('');
  return `<div class="cat-filter-row" id="catFilterRow">${pills}</div>`;
}
```

- [ ] **Step 6: Implémenter `computeAndRenderH2H`**

Remplacer le stub vide par :

```javascript
let allConfrontations = [];
let activeCat = '';

function computeAndRenderH2H() {
  allConfrontations = computeConfrontations(pilotA, pilotB);
  activeCat = '';
  renderH2H();
}

function renderH2H() {
  const filtered = activeCat
    ? allConfrontations.filter(c => (c.cls.perpetualClassCode || c.cls.className) === activeCat)
    : allConfrontations;

  if (!allConfrontations.length) {
    // Vérifier s'ils ont des events communs mais pas de catégorie commune
    h2hResults.innerHTML = `<div class="card"><div class="empty-state">
      <div class="es-icon">🏁</div>
      <div class="es-title">Aucune confrontation directe</div>
      <div class="es-text">Ces deux pilotes n'ont jamais été classés dans la même catégorie lors du même événement.</div>
    </div></div>`;
    return;
  }

  const stats = computeH2HStats(filtered);
  const allStats = computeH2HStats(allConfrontations);
  const catFilter = renderCatFilter(allStats.categories, activeCat);
  const header = renderH2HHeader(stats);
  const statsCards = renderH2HStatsCards(stats);
  const table = renderH2HTable(filtered);

  h2hResults.innerHTML = `<div class="card" style="padding:0;overflow:hidden">
    ${header}
    ${catFilter}
    ${statsCards}
    <div class="section-title" style="padding:0 18px;margin-bottom:0">
      Détail des confrontations
      <span class="meta">${filtered.length} résultat(s)</span>
    </div>
    ${table}
  </div>`;

  // Attacher le filtre catégorie
  document.getElementById('catFilterRow')?.querySelectorAll('.cat-pill').forEach(btn => {
    btn.addEventListener('click', () => {
      activeCat = btn.dataset.cat;
      renderH2H();
    });
  });
}
```

- [ ] **Step 7: Vérifier dans le navigateur**

- Sélectionner A et B qui ont des confrontations → bilan H2H complet affiché
- Header VS avec victoires en vert pour le leader
- Stats cards : confrontations, victoires A, victoires B
- Tableau avec colonnes Date / Événement / Catégorie / Rang A / Rang B / Vainqueur
- Cliquer sur un filtre catégorie → tableau et stats se recalculent
- Sélectionner A et B qui n'ont aucune confrontation → message explicatif

---

## Task 7: JS — Thème + fix du bouton Inverser

**Files:**
- Modify: `h2h_stats/index.html`

- [ ] **Step 1: Ajouter le script de gestion du thème (après le bloc principal)**

```javascript
// Gestion du bouton thème (identique à sqorz_stats)
(() => {
  const btn = document.getElementById('themeBtn');
  function getTheme() { return localStorage.getItem('theme') || 'auto'; }
  function applyTheme(pref) {
    const icons = { auto:'🌓', light:'☀️', dark:'🌙' };
    btn.textContent = icons[pref] || icons.auto;
    btn.title = `Thème : ${pref}`;
    const isDark = pref === 'dark' || (pref !== 'light' && window.matchMedia('(prefers-color-scheme: dark)').matches);
    document.documentElement.classList.toggle('dark', isDark);
    if (pref === 'auto') delete document.documentElement.dataset.themePref;
    else document.documentElement.dataset.themePref = pref;
  }
  btn.addEventListener('click', () => {
    const order = ['auto','light','dark'];
    const next = order[(order.indexOf(getTheme()) + 1) % 3];
    localStorage.setItem('theme', next);
    if (next === 'auto') localStorage.removeItem('theme');
    applyTheme(next);
  });
  applyTheme(getTheme());
})();
```

- [ ] **Step 2: Corriger le bouton Inverser**

Remplacer le listener actuel du bouton inverser par une version correcte :

```javascript
invertBtn.addEventListener('click', () => {
  if (!pilotA || !pilotB) return;
  const tmpA = pilotA;
  const tmpB = pilotB;
  // Réinitialiser sans déclencher de side-effects
  pilotA = null; pilotB = null;
  inputA.value = ''; inputA.hidden = false; badgeA.hidden = true; badgeA.innerHTML = '';
  inputB.value = ''; inputB.hidden = false; badgeB.hidden = true; badgeB.innerHTML = '';
  invertBtn.disabled = true;
  h2hResults.innerHTML = '';
  suggestionsSection.hidden = true;
  // Resélectionner dans l'ordre inversé
  selectPilotA(tmpB);
  selectPilotB(tmpA);
});
```

- [ ] **Step 3: Vérifier dans le navigateur**

- Thème : clic sur le bouton → cycle auto → clair → sombre, persistance au rechargement
- Inverser : avec A et B sélectionnés, cliquer Inverser → A et B échangés, H2H recalculé (résultat symétrique : wins A ↔ wins B)

---

## Task 8: Cas limites + polish final

**Files:**
- Modify: `h2h_stats/index.html`

- [ ] **Step 1: Gérer le cas "events communs mais pas de catégorie commune"**

Dans `computeAndRenderH2H`, avant `renderH2H()`, détecter ce cas :

```javascript
function computeAndRenderH2H() {
  allConfrontations = computeConfrontations(pilotA, pilotB);
  activeCat = '';

  // Vérifier s'ils ont des events en commun (même eventId, quelle que soit la catégorie)
  if (!allConfrontations.length) {
    const eventsA = new Set();
    const keyA = pilotA.normKey, keyB = pilotB.normKey;
    for (const ev of (pilotsIndex.events || [])) {
      let hasA = false, hasB = false;
      for (const cls of ev.classes) {
        for (const c of cls.competitors) {
          const k = norm((c.firstName||'') + ' ' + (c.lastName||'')).trim();
          if (k === keyA) hasA = true;
          if (k === keyB) hasB = true;
        }
      }
      if (hasA && hasB) { eventsA.add(ev.event.eventId); }
    }
    if (eventsA.size > 0) {
      h2hResults.innerHTML = `<div class="card"><div class="empty-state">
        <div class="es-icon">📋</div>
        <div class="es-title">Pas de confrontation dans la même catégorie</div>
        <div class="es-text">Ces pilotes ont participé à ${eventsA.size} événement(s) commun(s), mais jamais dans la même catégorie.</div>
      </div></div>`;
      return;
    }
  }

  renderH2H();
}
```

- [ ] **Step 2: Vérifier la prévention A = B**

La fonction `setupAutocomplete` reçoit `getExcludeKey` — le pilote A est déjà exclu des suggestions de B. Vérifier que si l'utilisateur tape exactement le même nom dans B, ce pilote n'apparaît pas dans le dropdown.

Tester : sélectionner "Jean Dupont" comme pilote A, taper "jean dupont" dans B → "Jean Dupont" absent du dropdown.

- [ ] **Step 3: Vérifier la sélection automatique quand un seul résultat correspond**

Ce comportement n'est **pas** dans la spec comme automatique — on laisse l'utilisateur choisir. Vérifier simplement que le dropdown s'affiche correctement avec un seul item.

- [ ] **Step 4: Tester les résultats spéciaux (DNF/DNS/DSQ)**

Chercher dans le navigateur des pilotes ayant des DNF. Vérifier :
- La ligne est grisée (classe `dnx-row`)
- Le rang est remplacé par "DNF"/"DNS"/"DSQ"
- Ces confrontations ne comptent pas dans les victoires/défaites

- [ ] **Step 5: Vérifier le responsive mobile**

Réduire la fenêtre à 375px. Vérifier :
- Les deux champs A et B s'empilent verticalement
- Le header H2H VS s'adapte
- Le tableau est scrollable horizontalement

- [ ] **Step 6: Commit final**

```bash
git -C /home/ludovic.socie/Documents/sqorz_tools add h2h_stats/index.html
git -C /home/ludovic.socie/Documents/sqorz_tools commit -m "feat: add head-to-head pilot comparison tool (h2h_stats)"
```

---

## Self-Review

### Spec coverage

| Exigence spec | Tâche |
|---|---|
| SPA statique, pattern sqorz_stats | Task 1 |
| Même thème/style que sqorz_stats | Task 1 (CSS copié), Task 7 (bouton thème) |
| Chargement pilots-index.json | Task 2 |
| Autocomplétion Pilote A | Task 3 |
| Adversaires suggérés après sélection A | Task 4 |
| Autocomplétion Pilote B | Task 3 |
| Bouton Inverser | Task 3 + Task 7 |
| Bilan H2H : header VS | Task 6 |
| Bilan H2H : stats globales (confrontations, victoires, égalités, catégories, période) | Task 5+6 |
| Tableau des confrontations (date, événement, catégorie, rang A, rang B, vainqueur) | Task 6 |
| Filtre par catégorie | Task 6 |
| Résultats spéciaux grisés avec statut | Task 6 |
| Lien pilote → sqorz_stats | Task 6 |
| Lien événement → Sqorz si accountCode disponible | Task 6 |
| Cas : aucune confrontation | Task 6 + Task 8 |
| Cas : même events mais pas même catégorie | Task 8 |
| Cas : A = B (prévention) | Task 3 (excludeKey) + Task 8 |
| Cas : disambiguation par club | Task 3 (ac-meta affiche groupName) |
| États : chargement, erreur réseau | Task 2 |

### Aucun placeholder détecté — chaque step contient le code complet.

### Cohérence des types
- `pilotA.normKey` → utilisé dans `computeTopOpponents`, `computeConfrontations`, `searchPilots(excludeNormKey)` ✓
- `computeConfrontations` retourne `{ event, account, cls, rankA, rankB }` — consommé dans `computeH2HStats` et `renderH2HTable` ✓
- `computeH2HStats` retourne `{ winsA, winsB, ties, validCount, total, categories, firstDate, lastDate }` — consommé dans `renderH2HHeader`, `renderH2HStatsCards` ✓
