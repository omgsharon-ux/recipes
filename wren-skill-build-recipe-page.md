# Wren Skill: Build Recipe Page

## What This Skill Does

Wren builds a new individual recipe page HTML file for the Recipes project. The page follows the established template, links into the index, and is ready to open directly in a browser.

---

## Step 0 — Pre-processing: Extract Embedded Images

Before building the page, check whether the source file's `<img>` tag uses a `data:image/...;base64,` URL instead of a file path.

**How to detect it:** Open the source file and look at the `<img src="...">` value. If it starts with `data:image/`, the image is embedded as Base64.

**If a Base64 image is found:**

1. **Extract the Base64 data** — read everything after the comma in the `src` attribute (i.e. strip `data:image/jpeg;base64,` and any leading whitespace). The remaining string is the raw Base64.

2. **Derive the image filename** — convert the recipe name to kebab-case (e.g. `Chicken Cacciatore` → `chicken-cacciatore.jpg`).

3. **Save the image** — decode the Base64 and write it as a binary JPG to:
   `Projects/Recipes/images/{kebab-case-recipe-name}.jpg`
   Use this Python snippet (the Base64 string spans multiple lines with whitespace — Python handles that cleanly):
   ```python
   import re, base64

   with open("/path/to/Wren/Recipe Name.html", "r") as f:
       content = f.read()

   match = re.search(r'src="data:image/jpeg;base64,([^"]+)"', content, re.DOTALL)
   if match:
       b64 = match.group(1).replace(" ", "").replace("\n", "").replace("\r", "")
       img_bytes = base64.b64decode(b64)
       with open("/path/to/images/recipe-name.jpg", "wb") as f:
           f.write(img_bytes)
   ```

4. **Open the saved JPG in Preview** to confirm it looks correct:
   ```bash
   open "/path/to/Projects/Recipes/images/recipe-name.jpg"
   ```

5. **Close the Preview window** once confirmed — don't leave images piling up:
   ```bash
   osascript -e 'tell application "Preview" to close (every window whose name contains "recipe-name")'
   ```
   Replace `recipe-name` with the image filename stem (e.g. `chicken-cacciatore`).

5. **Use the saved filename** as the `IMAGE_FILENAME` placeholder when building the new page (e.g. `chicken-cacciatore.jpg`).

**If no Base64 image is found:** the `src` already points to a file in `images/` — use that filename as-is and skip this step.

---

## Inputs Needed

Before starting, confirm you have:

- **Recipe name** (exact — will be used for the file name and page title)
- **Image filename** (saved to `Projects/Recipes/images/` — either extracted from Base64 above, or already present)
- **Ingredients list** (quantity + ingredient + any notes, one per line)
- **Instructions** (numbered steps)
- **Nutrition info** (serving size + all available nutrients)
- **Notes** (optional — omit the notes section entirely if none provided)

---

## File Naming

| Item | Convention |
|------|------------|
| Recipe page | `{Recipe Name}.html` — exact recipe name with spaces (e.g. `Turkey Taco Bowl.html`) |
| Image | `images/{kebab-case-recipe-name}.jpg` (e.g. `images/turkey-taco-bowl.jpg`) |
| Save location | `Projects/Recipes/` |

---

## Template

Use the HTML below as the exact base. Replace all `PLACEHOLDER` values with the real recipe data.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RECIPE_NAME</title>
    <link rel="stylesheet" href="styles.css">
    <style>
        /* ── Page chrome ─────────────────────────────── */
        main {
            padding-bottom: 40px;
        }

        footer {
            padding: 24px 0 60px;
            font-size: 22px;
            text-align: center;
            color: #888;
            border-top: 1px solid #e8e8e8;
            margin-top: 40px;
        }

        /* ── Back link ───────────────────────────────── */
        .back-link {
            display: inline-flex;
            align-items: center;
            gap: 6px;
            margin: 20px 0 16px;
            font-size: 22px;
            color: #6bb8d4;
            text-decoration: none;
            font-weight: 600;
            letter-spacing: 0.01em;
        }

        .back-link:hover {
            color: #4a9db8;
        }

        /* ── Hero image ──────────────────────────────── */
        .hero-wrap {
            position: relative;
            margin-bottom: 32px;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: 0 4px 20px rgba(0,0,0,0.12);
        }

        .recipe-image {
            width: 100%;
            display: block;
            margin-bottom: 0;
        }

        /* ── Section headings ────────────────────────── */
        .section-heading {
            display: flex;
            align-items: center;
            gap: 10px;
            font-size: 26px;
            font-weight: 800;
            text-transform: uppercase;
            letter-spacing: 0.05em;
            color: #222;
            margin: 36px 0 18px;
            padding-bottom: 8px;
            border-bottom: 3px solid #6bb8d4;
        }

        .section-heading .icon {
            font-size: 22px;
        }

        /* ── Checklist items ─────────────────────────── */
        .check-item {
            display: flex;
            align-items: flex-start;
            gap: 16px;
            margin-bottom: 20px;
            padding: 14px 16px;
            background: #f8fbfd;
            border-radius: 8px;
            border-left: 4px solid transparent;
            transition: border-color 0.2s, background 0.2s;
        }

        .check-item:has(input:checked) {
            border-left-color: #6bb8d4;
            background: #eef7fb;
        }

        .check-item input[type="checkbox"] {
            flex-shrink: 0;
            width: 28px;
            height: 28px;
            margin-top: 2px;
            accent-color: #6bb8d4;
            cursor: pointer;
        }

        .check-item label {
            font-size: 24px;
            line-height: 1.4;
            cursor: pointer;
            flex: 1;
        }

        input[type="checkbox"]:checked + label {
            text-decoration: line-through;
            color: #999;
        }

        /* Step numbers */
        .step-num {
            flex-shrink: 0;
            width: 36px;
            height: 36px;
            background: #6bb8d4;
            color: white;
            font-size: 18px;
            font-weight: 800;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-top: 0;
        }

        .check-item:has(input:checked) .step-num {
            background: #a8d8ea;
        }

        /* ── Notes section ───────────────────────────── */
        .notes {
            margin-top: 12px;
        }

        .notes-body {
            background: #fffbf0;
            border-left: 4px solid #f0c040;
            border-radius: 0 8px 8px 0;
            padding: 16px 20px;
            font-size: 24px;
            line-height: 1.5;
            color: #555;
        }

        /* ── FDA Nutrition Facts Label ───────────────── */
        .nutrition {
            margin-top: 12px;
        }

        .nf-label {
            font-family: Arial, Helvetica, sans-serif;
            border: 2px solid #000;
            padding: 4px 6px 4px;
            max-width: 320px;
            box-sizing: border-box;
        }

        .nf-title {
            font-size: 36px;
            font-weight: 900;
            line-height: 1;
            margin: 0 0 0;
            letter-spacing: -0.5px;
        }

        .nf-servings-count {
            font-size: 14px;
            margin: 2px 0 0;
        }

        .nf-serving-size {
            font-size: 14px;
            font-weight: 700;
            display: flex;
            justify-content: space-between;
            margin: 2px 0 3px;
        }

        /* Thick black bars */
        .nf-bar-thick {
            background: #000;
            height: 10px;
            margin: 4px -6px;
        }

        .nf-bar-medium {
            background: #000;
            height: 5px;
            margin: 4px -6px;
        }

        /* Thin gray rules */
        .nf-rule {
            border: none;
            border-top: 1px solid #888;
            margin: 2px 0;
        }

        /* Calories block */
        .nf-calories-block {
            display: flex;
            align-items: flex-end;
            justify-content: space-between;
        }

        .nf-calories-label {
            font-size: 13px;
            font-weight: 700;
            line-height: 1.1;
        }

        .nf-calories-sub {
            font-size: 10px;
            font-weight: 400;
        }

        .nf-calories-num {
            font-size: 52px;
            font-weight: 900;
            line-height: 1;
        }

        /* DV header line */
        .nf-dv-header {
            font-size: 11px;
            font-weight: 700;
            text-align: right;
            margin: 2px 0;
        }

        /* Nutrient rows */
        .nf-row {
            display: flex;
            justify-content: space-between;
            align-items: baseline;
            font-size: 13px;
            padding: 1px 0;
        }

        .nf-row .nf-name {
            flex: 1;
        }

        .nf-row .nf-dv {
            font-weight: 700;
            white-space: nowrap;
        }

        .nf-row.bold .nf-name {
            font-weight: 700;
        }

        .nf-row.indent .nf-name {
            padding-left: 16px;
            font-weight: 400;
        }

        .nf-row.double-indent .nf-name {
            padding-left: 28px;
            font-weight: 400;
        }

        /* Vitamins section (thick bar before it, thin rules within) */
        .nf-vitamins {
            border-top: 7px solid #000;
        }

        .nf-vit-row {
            display: flex;
            justify-content: space-between;
            font-size: 12px;
            padding: 1px 0;
            border-bottom: 1px solid #888;
        }

        .nf-vit-row:last-child {
            border-bottom: none;
        }

        /* Footnote */
        .nf-footnote {
            font-size: 9px;
            border-top: 1px solid #000;
            margin-top: 4px;
            padding-top: 3px;
            line-height: 1.3;
        }

        /* ── Reset button ────────────────────────────── */
        .reset-button {
            width: 100%;
            max-width: 100%;
            height: auto;
            padding: 24px 0;
            margin: 40px auto 0;
            font-size: 22px;
            font-weight: 800;
            color: white;
            background: linear-gradient(135deg, #6bb8d4 0%, #4a9db8 100%);
            border: none;
            border-radius: 10px;
            cursor: pointer;
            display: block;
            letter-spacing: 0.05em;
            text-transform: uppercase;
            box-shadow: 0 4px 12px rgba(107,184,212,0.4);
            transition: transform 0.1s, box-shadow 0.1s;
        }

        .reset-button:active {
            transform: translateY(1px);
            box-shadow: 0 2px 6px rgba(107,184,212,0.3);
        }
    </style>
</head>
<body>
    <header>
        <div class="header-banner">RECIPE_NAME</div>
    </header>

    <main>
        <a href="index.html" class="back-link">&#8592; Back to Recipes</a>

        <div class="hero-wrap">
            <img src="images/IMAGE_FILENAME" alt="RECIPE_NAME" class="recipe-image">
        </div>

        <!-- INGREDIENTS ─────────────────────────────── -->
        <div class="ingredients">
            <h2 class="section-heading"><span class="icon">🛒</span> Ingredients</h2>
            INGREDIENTS_LIST
        </div>

        <!-- INSTRUCTIONS ────────────────────────────── -->
        <div class="preparation">
            <h2 class="section-heading"><span class="icon">👩‍🍳</span> Instructions</h2>
            STEPS_LIST
        </div>

        <!-- NOTES (omit entire block if no notes) ───── -->
        NOTES_SECTION

        <!-- NUTRITION ───────────────────────────────── -->
        <div class="nutrition">
            <h2 class="section-heading"><span class="icon">📊</span> Nutrition</h2>
            NUTRITION_FACTS_LABEL
        </div>

        <button class="reset-button" onclick="resetCheckboxes()">&#10003; Reset All Checkboxes</button>
    </main>

    <footer>Powered by &#9749; &amp; &#127790;</footer>

    <script>
        function resetCheckboxes() {
            document.querySelectorAll('input[type="checkbox"]').forEach(cb => cb.checked = false);
        }
    </script>
</body>
</html>
```

---

## Ingredient Block Pattern

Each ingredient uses this pattern. Use sequential IDs: `ing1`, `ing2`, `ing3`, etc.

```html
<div class="check-item">
    <input type="checkbox" id="ing1">
    <label for="ing1">INGREDIENT TEXT</label>
</div>
```

---

## Step Block Pattern

Each instruction step uses this pattern. Use sequential IDs: `step1`, `step2`, `step3`, etc. Include a `<span class="step-num">` for the visible step number circle.

```html
<div class="check-item">
    <input type="checkbox" id="step1">
    <span class="step-num">1</span>
    <label for="step1">STEP TEXT</label>
</div>
```

---

## Notes Section

Only include the notes section if the recipe has actual notes. If there are no notes, omit this block entirely — do not leave a placeholder or empty section.

```html
<div class="notes">
    <h2 class="section-heading">Notes</h2>
    <p class="notes-body">NOTES TEXT</p>
</div>
```

Place the notes section between the Instructions div and the Nutrition div.

---

## Nutrition Facts Label

The nutrition section renders as an authentic FDA-style Nutrition Facts label — pure HTML/CSS, no images, no external fonts. It uses `font-family: Arial, Helvetica, sans-serif` throughout.

**Label anatomy (top to bottom):**
1. "Nutrition Facts" in bold large type
2. Servings per container line
3. Serving size line (bold, space-between layout)
4. Thick black bar (10px)
5. "Amount per serving" + large calorie number (flex row, space-between)
6. Medium black bar (5px)
7. "% Daily Value *" right-aligned header
8. Nutrient rows — bold main nutrients, indented sub-nutrients, thin gray rules between
9. Thick black bar (7px via border-top) before vitamins/minerals
10. Vitamin/mineral rows with thin rules between
11. Fine-print footnote

**HTML pattern for `NUTRITION_FACTS_LABEL`:**

```html
<div class="nf-label">
    <p class="nf-title">Nutrition Facts</p>
    <p class="nf-servings-count">X servings per container</p>
    <div class="nf-serving-size"><span>Serving size</span><span>Xoz (Xg)</span></div>
    <div class="nf-bar-thick"></div>
    <div class="nf-calories-block">
        <div>
            <p class="nf-calories-label">Amount per serving</p>
            <p class="nf-calories-label" style="font-size:20px;">Calories</p>
        </div>
        <span class="nf-calories-num">XXX</span>
    </div>
    <div class="nf-bar-medium"></div>
    <p class="nf-dv-header">% Daily Value *</p>
    <hr class="nf-rule">
    <div class="nf-row bold"><span class="nf-name">Total Fat Xg</span><span class="nf-dv">X%</span></div>
    <hr class="nf-rule">
    <div class="nf-row indent"><span class="nf-name">Saturated Fat Xg</span><span class="nf-dv">X%</span></div>
    <hr class="nf-rule">
    <div class="nf-row indent"><span class="nf-name"><i>Trans</i> Fat Xg</span><span class="nf-dv"></span></div>
    <hr class="nf-rule">
    <div class="nf-row bold"><span class="nf-name">Cholesterol Xmg</span><span class="nf-dv">X%</span></div>
    <hr class="nf-rule">
    <div class="nf-row bold"><span class="nf-name">Sodium Xmg</span><span class="nf-dv">X%</span></div>
    <hr class="nf-rule">
    <div class="nf-row bold"><span class="nf-name">Total Carbohydrate Xg</span><span class="nf-dv">X%</span></div>
    <hr class="nf-rule">
    <div class="nf-row indent"><span class="nf-name">Dietary Fiber Xg</span><span class="nf-dv">X%</span></div>
    <hr class="nf-rule">
    <div class="nf-row indent"><span class="nf-name">Total Sugars Xg</span><span class="nf-dv"></span></div>
    <hr class="nf-rule">
    <div class="nf-row bold"><span class="nf-name">Protein Xg</span><span class="nf-dv"></span></div>
    <div class="nf-vitamins">
        <div class="nf-vit-row"><span>Vitamin D Xmcg</span><span>X%</span></div>
        <div class="nf-vit-row"><span>Calcium Xmg</span><span>X%</span></div>
        <div class="nf-vit-row"><span>Iron Xmg</span><span>X%</span></div>
        <div class="nf-vit-row"><span>Potassium Xmg</span><span>X%</span></div>
    </div>
    <p class="nf-footnote">* The % Daily Value (DV) tells you how much a nutrient in a serving of food contributes to a daily diet. 2,000 calories a day is used for general nutrition advice.</p>
</div>
```

**Notes on filling in the label:**
- Replace every `X` with the actual value from the nutrition data
- Omit rows for nutrients not provided (e.g. if no Vitamin D data, remove that row)
- Add extra vitamin/mineral rows inside `.nf-vitamins` as needed
- If potassium is listed in the main nutrients (not vitamins section), include it as a `nf-vit-row` anyway — it belongs in the vitamins block per FDA format
- % Daily Value figures: if not provided, calculate or omit the `<span class="nf-dv">` content (leave empty span so the flex layout holds)

---

## After Building the Page

1. Update `Projects/Recipes/index.html` — add the new recipe to the `recipes` array in the JavaScript block. Keep entries in alphabetical order by name.

```javascript
{ name: "Recipe Name Here", file: "Recipe Name Here.html" }
```

2. Delete the source file from the Wren working folder (`Projects/Recipes/Wren/`) — once the rebuilt page is confirmed saved to the main `Projects/Recipes/` folder, delete the original from `Wren/`. This keeps the queue clean.

3. **Commit and push to GitHub** — from the `Projects/Recipes/` directory, stage the new/modified files and push:

```bash
cd "/Users/sharonoverschmidt/Library/Mobile Documents/com~apple~CloudDocs/Cowork Brain/Projects/Recipes"

# Stage the recipe page, image, and index — never stage .DS_Store or files in Wren/
git add "Recipe Name.html" "images/recipe-name.jpg" index.html

git commit -m "Add Recipe Name"

git push origin main
```

   - Replace `Recipe Name.html` and `images/recipe-name.jpg` with the actual filenames
   - If only rebuilding an existing recipe (no new image), omit the image from `git add`
   - Never stage `.DS_Store`, `Wren/`, `template.html`, or `wren-skill-build-recipe-page.md`

---

## Quality Check Before Delivering

- [ ] If image was Base64 embedded: JPG saved to `Projects/Recipes/images/` and source file previewed in browser
- [ ] File saved with the exact recipe name (spaces preserved, no kebab-case)
- [ ] Image path matches the actual filename in `images/` (file path, not Base64 data URL)
- [ ] All ingredient IDs are unique (`ing1`, `ing2` … not duplicated)
- [ ] All step IDs are unique (`step1`, `step2` … not duplicated)
- [ ] Step blocks include `<span class="step-num">N</span>` with the correct number
- [ ] Back link points to `index.html`
- [ ] New recipe added to `index.html` recipes array in alphabetical order
- [ ] Notes section present only if notes were provided; omitted otherwise
- [ ] Nutrition rendered as `.nf-label` FDA-style label, not a grid
- [ ] No `<br>` tags used for spacing — all spacing via CSS
- [ ] Page opens correctly as a plain HTML file (no server required)
- [ ] Source file deleted from `Projects/Recipes/Wren/`
- [ ] Changes committed and pushed to GitHub (`git push origin main` confirmed successful)
