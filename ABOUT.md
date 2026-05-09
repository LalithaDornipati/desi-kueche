# Desi Küche

**Indian recipes from your German pantry.** A single-file static web app — no backend, no API key, no install.

Open `desi-kueche.html` in a browser. Pick a category, set a few options, hit generate. Two outputs side-by-side:

1. A **recipe** with checkable steps you tap as you cook
2. A **copy-paste prompt** for an LLM (Claude / ChatGPT / Gemini) for live follow-up help — substitutions, scaling, "I burnt the onions, can I save it?"

---

## What makes it different

- **German supermarket reality.** Recipes only call for spices Edeka and Rewe actually stock. Asafoetida, curry leaves, kasuri methi are mentioned only as optional "vom Indien-Laden" — the recipe always works without them.

- **Equipment translation.** "Medium-low heat" means nothing on an Induktion 1–6 stove. Every step says exactly what to do: *"auf Stufe 4 anbraten"*, *"Varoma, Stufe 1"*, or *"mittlere Flamme"*.

- **Pantry-aware.** Tell it which masala mixes you have on your shelf. The two toggles (Garam Masala, Curry Pulver) actually change how the recipe gets spiced — missing Garam Masala? You get a substitute (½ TL Zimt + 2 Nelken + 2 Kardamom frisch gemörsert) with an EMPFOHLEN badge. Other mixes you might own (Tandoori Masala, Chaat Masala, Chicken Curry Masala, Maggi Masala, Peri Peri…) aren't toggles — there's just a note: "add ½–1 TL and taste-test, like salt or chili".

- **Cultural bridges.** Every recipe anchors to something you already know:
  - Dal → *"Wie eine Linsensuppe, aber mit Tadka — heißem Würzöl wie Schmalz aufgießen"*
  - Khichdi → *"Wie ein indischer salziger Milchreis, lockerer als Risotto"*
  - Tandoori → *"Wie Joghurt-mariniertes Hähnchen vom Grill, denk an Souvlaki"*
  - Aloo Puff → *"Eine herzhafte Apfeltasche mit Kartoffelfüllung statt Apfel"*

- **Live updates.** Change any selection — equipment, servings, pantry, spice level — and the output rerenders instantly. No "Generate" needed.

- **Progressive flow.** The form unfolds one step at a time: category dropdown first, options appear after that, kit (equipment + scale + pantry) after that. The page scrolls smoothly to each new section as it appears. A small back-to-top button fades in once you scroll past the fold so changing categories is one tap.

---

## Categories

| Category | What it is |
|---|---|
| 🥟 **Puffs** | Indian-spiced filling in Blätterteig — like Samosa, but baked |
| 🍛 **Curry** | 9 bases × 7 proteins × 3 spice levels — onion-tomato, tomato paste, palak, cashew, kadhi-style yogurt, green herbs, garlicky, peppery, coconut |
| 🥣 **Dal** | 4 lentil varieties (Rote / Berg / Beluga / Mung), 4 styles |
| 🍚 **Khichdi** | One-pot rice-and-lentils with whatever vegetables you've got |
| 🍜 **Indo-Chinese Noodles** | Hakka-style with the Asia-Regal essentials |
| 🥦 **Sabzi / Tandoori** | Pan-fried, oven-roasted, or yogurt-marinated |
| 🌾 **Sides** | Chilla pancakes, basmati / jasmin / parboiled rice, wraps |

---

## The two outputs explained

When you click Generate, the page shows two cards:

**Recipe card** — categorized ingredients (legumes, vegetables, protein, dairy, whole spices, ground spices, misc), numbered steps with checkboxes, a cultural-bridge callout, and pairing suggestions. Save it as a favorite to come back to.

**Prompt card** — copy this into Claude / ChatGPT / Gemini if you want a chat partner while you cook. The prompt knows your pantry, equipment, scale, and the recipe — so when you ask *"can I skip the yogurt?"* halfway through, the answer fits *your* situation, not a generic one.

---

## Getting started

```
1. Open desi-kueche.html in a browser
2. Pick a category card
3. Fill in the option pills that appear
4. Set your stove and how many people you're feeding
5. Toggle the masala mixes you actually have on your shelf
6. Click "Rezept generieren"
```

That's it. State persists in localStorage; favorites survive browser restarts; everything works offline once the page loads.

For technical details (file structure, data model, how to add a new recipe), see `README.md`.