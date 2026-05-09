# Desi Küche

> **Indian recipes from your German pantry** — a single-file static web app, no backend, no API keys, no build step.

Open `desi-kueche.html` in any browser. That's it.

---

## What it does

Pick a category (Curry, Dal, Khichdi, Puffs, Noodles, Sabzi, Sides), set a few options (base, protein, spice level), tell it your stove and how many people you're feeding, and it gives you two outputs side by side:

1. **A recipe** with checkable steps you tap as you cook
2. **A copy-paste prompt** for an LLM (Claude / ChatGPT / Gemini) for live follow-up help while cooking — substitutions, scaling, "I burnt the onions, can I save it?"

The recipes are written to **work with what's actually in a German supermarket** (Edeka, Rewe, Kaufland) and to **adapt to what's on your spice shelf**.

---

## Design principles

These shape every recipe and every UI decision:

### 1. German-first ingredient naming, English in parentheses
Always `Rote Linsen (red lentils)`, never `toor dal`. A German user reads German; an English speaker still has the bridge.

### 2. Spice palette restricted to German supermarket reality
The recipes only call for spices commonly stocked in German supermarkets:
- **Whole**: Zimtstange, grüner Kardamom, Nelken, Sternanis, Lorbeerblatt, Fenchelsamen, Kreuzkümmelsamen, Senfsamen, schwarzer Pfeffer
- **Ground**: Kurkuma, Paprika edelsüß / scharf, Garam Masala, Korianderpulver, Kreuzkümmel gemahlen, Chilipulver

Asafoetida, curry leaves, kasuri methi, amchur are mentioned only as *optional, vom Indien-Laden* — the recipe always works without them.

### 3. Equipment-specific heat translation
Indian recipes say "medium-low" which means nothing on an Induktion 1–6 vs 1–10 stove. Every step says exactly what to do:
- Induktion 1–6 → "auf Stufe 4 anbraten"
- Induktion 1–10 → "auf Stufe 6 anbraten"
- Gas → "mittlere Flamme"
- Thermomix → "Varoma, Stufe 1"

Equipment options are also filtered by what makes culinary sense — no Thermomix for stir-fry, no stove pills for oven-only dishes.

### 4. Pantry-aware: recipes adapt to your spice shelf
Tell the app which masala mixes you have. Recipes adapt:
- **Have Garam Masala** → recipe uses it normally
- **Don't have it** → ingredient line shows an "EMPFOHLEN" badge with a substitute (`½ TL Zimt + 2 Nelken + 2 Kardamom frisch gemörsert`); step text says "die Garam-Masala-Mischung (oder den Ersatz) einrühren"
- **Have Tandoori Masala** → tandoori recipe surfaces a bonus suggestion ("you have it — use 1 TL extra")
- **Don't have it** → no nagging mention

### 5. Cultural bridges to familiar German dishes
Every recipe has a "Wenn du das kennst" callout that anchors the dish:
- Aloo Puff → *"Eine herzhafte Apfeltasche, gleiche Falt-Technik"*
- Dal → *"Wie eine Linsensuppe, aber mit Tadka — heißem Würzöl wie Schmalz aufgegossen"*
- Khichdi → *"Wie ein indischer salziger Milchreis, lockerer als Risotto"*
- Tandoori → *"Wie Joghurt-mariniertes Hähnchen vom Grill, denk an Souvlaki"*

### 6. Live updates
Every selection change re-renders the output instantly. No "Generate again" needed once it's visible.

### 7. Progressive disclosure with auto-scroll
The form is presented one decision at a time. Initially only the category dropdown shows. Pick a category → options appear and the page smooth-scrolls to them. Fill in the last required option → the kit section (equipment + scale + pantry) appears and the page scrolls to it. Click Generate → the page scrolls to the recipe + prompt cards. A small floating back-to-top button fades in once you scroll past the fold so jumping back to the dropdown is one tap. Re-clicking Generate always scrolls to the recipe — useful for changing options after viewing and coming back.

---

## File structure

It's all in one file: `desi-kueche.html`. The script section is split into 12 numbered sections. The numbers are stable; if you `Ctrl-F "5. UI RENDERING"` you'll land in the right place.

```
1. STATIC DATA               Categories, equipment, pantry items
2. CORE HELPERS              heat() qty() pantry checks
3. OPTIONS PER CATEGORY      Form schema (which fields, which choices)
4. APP STATE & PERSISTENCE   STATE object + localStorage round-trip
5. UI RENDERING              Form render functions + createPill helper
6. PANTRY-AWARE SPICE HELPERS gmIng() bonusMasala()
7. RECIPE BUILDERS           One function per category
8. INGREDIENT CATEGORIZATION Auto-grouping by keyword matching
9. PROMPT BUILDER            LLM hand-off prompt construction
10. OUTPUT RENDERING         Recipe card + prompt card
11. FAVORITES                Save/load/delete user favorites
12. INIT                     Event handlers, first render
```

CSS is split into single-line-commented sections at the top of the file: color tokens, header, category dropdown, options panel, output, cultural callout, ingredient sections, prompt card, back-to-top button, etc.

---

## Data model

### `STATE`
The single source of truth. Persisted to localStorage on every change.

```js
{
  category: 'curry',                    // or 'dal' | 'khichdi' | 'puff' | …
  options: { base: 'onion-tomato', protein: 'chicken', spice: 'medium' },
  equipment: 'induktion-6',             // 'induktion-10' | 'gas' | 'thermomix'
  servings: 4,                          // 1–12 (used for everything except puffs)
  puffSheets: 1,                        // 1–4 (puffs only)
  puffCuts: 6,                          // 6 or 8 (puffs only)
  pantry: ['garam-masala', 'curry-pulver'],   // which masala mixes user has
  favorites: [{ id, name, category, options, equipment, servings, … }]
}
```

### Recipe shape
Every recipe builder returns this shape:

```js
{
  title:        'Aloo Curry',
  sub:          'Zwiebel-Tomate-Basis · mittelscharf',  // category line
  time:         '35 Min',
  culturalNote: 'Eine indische Schmor-Soße — wie eine langsame Tomaten-Pasta…',
  ingredients: [
    ['2 EL Öl oder Ghee', 'oil or ghee'],                  // simple tuple
    ['1 grüne Chili gehackt', 'green chili', true],         // optional flag
    { de: '1 TL Garam Masala', en: 'garam masala',         // pantry-aware
      pantryStatus: 'empfohlen', substitute: '…' },
    …
  ],
  steps:    [{ text: 'Öl in einem Topf bei Stufe 4 erhitzen…' }, …],
  pairings: ['Basmati-Reis', 'Salzige Pancakes (Chilla)', …]
}
```

### Ingredient categorization
Ingredients aren't tagged in recipes. They're auto-classified at render time by the `categorizeIngredient()` function, which keyword-matches the German name into one of seven sections:
1. **Hülsenfrüchte & Reis** (turmeric)
2. **Gemüse & Aromaten** (cardamom)
3. **Eiweiß** (rose)
4. **Milchprodukte** (peacock)
5. **Rohgewürze** (terracotta)
6. **Pulvergewürze** (chili)
7. **Sonstiges** (soft brown)

This means you can add new recipes without touching classification code — it just works.

---

## How to extend

### Add a new spice/masala to the pantry toggles
In §1, append to `PANTRY.empfohlen` to add a new toggleable masala mix that should change recipe behavior (substitute logic via `gmIng()`-style helper). The pill appears automatically. For non-recipe-altering spice mixes the user might already own (Tandoori Masala, Chaat Masala, etc.), don't add them to `PANTRY` — they're covered by the "add ½–1 TL to taste" note in the pantry section.

### Add a new equipment type
In §1, add to `EQUIPMENT`. In §2 `heat()`, add a column to the heat translation map. In §5 `getValidEquipment()`, decide for which categories it's valid.

### Add a new option to an existing category
In §3 `OPTIONS`, add an item to the relevant `items` array. The pill appears automatically. Then handle the new value inside the recipe builder in §7.

### Add a new recipe category
1. **§1** add a new `CATEGORIES` entry (id, icon, accent color, name, description)
2. **§3** add an `OPTIONS` entry declaring its form fields
3. **§7** add a builder function in `RECIPES` returning the standard recipe shape
4. **§5** if needed, add equipment-validity rules in `getValidEquipment`

### Add a new ingredient classification rule
In §8, edit `categorizeIngredient()`. The function uses tiered keyword matching — earlier tiers win, so put more-specific rules first.

---

## What it doesn't do

- **No backend.** Everything is in one HTML file.
- **No API key.** The LLM hand-off is via copy-paste, not direct API calls.
- **No images.** Photos would 10x the file size and bring questions of licensing.
- **No video.** Same.
- **No login, no accounts.** localStorage is the only persistence.
- **No analytics.** No third-party scripts at all.

This is by design. It's a personal hobby project meant to be cloneable, self-hostable on GitHub Pages, and trivially understandable by anyone reading the source.

---

## Tech notes

- **Single static HTML file**, ~2700 lines including CSS and JS
- **No frameworks, no build step, no dependencies** — just vanilla HTML/CSS/JS
- **Fonts via Google Fonts CDN**: Instrument Serif (display italic), Manrope (body), JetBrains Mono (prompt code)
- **Mobile-responsive** — tested down to ~360px wide
- **localStorage** for state and favorites persistence
- **No service worker** — but the file works fully offline once loaded since fonts and styles are inline/self-contained

---

## License

Personal project. Do whatever you want with it.