# Design Spec: ProofShot + Design Mathematics Library Integration

**Date:** 2026-03-24
**Status:** Implemented
**Author:** Claude (Prime Agent session)

## Summary

Expand the-library with two new skills and supporting trigger patterns:

1. **`proofshot`** — Visual verification of UI work via the ProofShot CLI (start/exec/stop/diff/pr workflow)
2. **`design-mathematics`** — Classical design proportions, information weighting formulas, and mathematical verification criteria for UI work

These integrate with the existing frontend-design skill suite so that design work flows naturally into mathematically-grounded verification.

## Deliverables

### 1. Skill: `proofshot`

**Source:** `/config/workspace/projects/proofshot/skills/claude/SKILL.md` (enhanced in-place)
**Catalog entry in library.yaml**

The existing ProofShot Claude skill covers start/exec/stop. Enhancement adds:
- Visual regression via `proofshot diff`
- PR proof posting via `proofshot pr`
- Integration cue: run verification after any `design-*` or `frontend-design` skill
- Decision logic: when to skip verification (non-visual changes)

### 2. Skill: `design-mathematics`

**Source:** New skill in the-library repo at `skills/design-mathematics/SKILL.md`
**Catalog entry in library.yaml**

A reference skill encoding classical design proportions and information design formulas. Teaches agents to apply mathematical principles when building or verifying UI. Contains:

#### Classical Proportions (Sections 1-10)

**1. Musical Typographic Scale**
Ratios derived from Western tonal intervals:

| Interval | Ratio | Origin | Use Case |
|----------|-------|--------|----------|
| Minor Second | 1.067 | 16/15 | Dense text, academic |
| Major Second | 1.125 | 9/8 | Web body copy |
| Minor Third | 1.200 | 6/5 | UI differentiation |
| Major Third | 1.250 | 5/4 | Headers, presentations |
| Perfect Fourth | 1.333 | 4/3 | Editorial, news |
| Augmented Fourth | 1.414 | sqrt(2) | Landing pages |
| Perfect Fifth | 1.500 | 3/2 | Banners, infographics |
| Golden Ratio | 1.618 | (1+sqrt(5))/2 | Luxury, portfolios |

**2. Le Corbusier's Modulor System**
Human-scale proportions: navel height (1.13m) subdivided by phi. Red series (navel-based) and Blue series (total height-based), each forming Fibonacci sequences. Design application: spacing and sizing that relates to human body proportions.

**3. Pythagorean Design Triangles**
Integer right triangles (3:4:5, 5:12:13, 8:15:17) for creating clean visual tension paths between UI elements. Place CTAs at Pythagorean offsets from primary content.

**4. Rule of Thirds vs Golden Sections**
Thirds grid (33%/67%) as the integer approximation of golden sections (38.2%/61.8%). Power positions at grid intersections. The rule of thirds trades irrational precision for practical simplicity.

**5. Fractal Self-Similarity**
Components at scale 1/phi of their parent: Page (1000px) -> Section (618px) -> Card (382px) -> Element (236px) -> Detail (146px). The same proportional relationships repeat at every nesting level.

**6. Color Wheel Geometry**
Angular relationships: Complementary (180deg), Triadic (120deg), Analogous (30deg), Split-complementary (150deg/210deg), Tetradic (90deg), Golden angle (137.508deg).

**7. Font-to-Icon Proportional Laws**
- sqrt(2) rule: icon = font x 1.414 (diagonal of font-height square)
- phi rule: icon = font x 1.618 (golden proportion)
- 2x rule: icon = font x 2 (Material Design default)

**8. Muller-Brockmann Grid Mathematics**
Column proportions at 1440px base with 24px gutters. Golden 2-column split: main 890px (61.8%), sidebar 550px (38.2%).

**9. Reading Pattern Geometry**
F-pattern (content-heavy): 100% first sweep, 60% second, left 20% vertical stem.
Z-pattern (hero pages): diagonal angles vary by aspect ratio (16:9 = 29.4deg, 4:3 = 36.9deg).

**10. Touch Target Mathematics**
Apple HIG (44px), Material Design (48dp), WCAG AA (24px), WCAG AAA (44px). Thumb zones map to rule of thirds: primary actions in bottom third.

#### Information Design Formulas (Sections 11a-11f)

**11a. Recency Decay**
`R(t) = e^(-lambda*t)` where lambda = ln(2) / half-life.
Content-type decay rates: news (1 day), blog (7 days), docs (30 days), reference (1 year).

**11b. Importance Scoring**
`I(p) = 1 + log2(p)` — logarithmic scale prevents high-priority items from dominating. Priority-10 is 4.3x priority-1 weight, not 10x.

**11c. Volume Normalization**
`V(n) = 1 / sqrt(n)` — more items in a category = each gets less visual space, but preserves some visibility (not linear inverse).

**11d. Combined Visual Weight**
`W(i) = alpha*I(p) + beta*R(t) + gamma*V(n)` where alpha + beta + gamma = 1.
Coefficient presets: News (0.2/0.6/0.2), Dashboard (0.5/0.2/0.3), Documentation (0.4/0.1/0.5), Portfolio (0.3/0.3/0.4).

**11e. Information Entropy**
`H = -sum(p_i * log2(p_i))` (Shannon entropy).
Low entropy (H<2) = hero layout. Medium (H~3) = card grid. High (H>4) = table/list.

**11f. Fitts's Law**
`T = a + b * log2(1 + D/W)` — important elements need larger W (touch target) and smaller D (distance from resting position).

### 3. Trigger Patterns (triggers.yaml)

**New intent: `visual-verification` (high confidence)**
Patterns: visual verification, verify ui, visual proof, video proof, screenshot proof, proofshot, visual regression, visual diff, baseline comparison, verify changes, proof to pr, does it look right, check the ui, test the ui.
Items: [skill:proofshot]

**New intent: `design-mathematics` (high confidence)**
Patterns: golden ratio, rule of thirds, typographic scale, type scale, Fibonacci, Modulor, Pythagorean, fractal layout, color harmony, font icon ratio, information weight, recency decay, visual weight, Shannon entropy, Fitts law, classical proportions.
Items: [skill:design-mathematics]

**Cross-wire existing intents:**
Add `skill:proofshot` and `skill:design-mathematics` to both `frontend-design` and `design-operations` items arrays, so both surface during any design work.

### 4. library.yaml Additions

Two new entries in the skills section:
- `proofshot` pointing to the proofshot repo's Claude skill
- `design-mathematics` pointing to the new skill in the-library repo

## Architecture

```
the-library/
  skills/
    design-mathematics/
      SKILL.md              <-- NEW: Classical proportions + info design formulas
    mcp2cli-first/
      SKILL.md
    prime-agent/
      SKILL.md
    wolfram/
      SKILL.md
  library.yaml              <-- MODIFIED: +2 skill entries
  triggers.yaml             <-- MODIFIED: +2 intents, cross-wired existing intents

proofshot/
  skills/claude/
    SKILL.md                <-- ENHANCED: +diff, +pr, +design suite integration cue
```

## Integration Flow

```
User modifies UI
  |
  v
frontend-design suite fires (existing)
  |
  v
prime-agent surfaces proofshot (cross-wired trigger)
  |
  v
Agent runs proofshot start/exec/stop
  |
  v
Agent applies design-mathematics checklist during exec:
  - Pythagorean visual paths
  - Rule of thirds power positions
  - Reading pattern alignment
  - Font-to-icon ratios
  - Color wheel harmony
  - Information weight formula
  |
  v
proofshot diff (visual regression)
  |
  v
proofshot pr (post proof to PR)
```

## Success Criteria

1. `proofshot` skill appears in library catalog and auto-installs when visual verification patterns trigger
2. `design-mathematics` skill appears in library catalog and auto-installs when mathematical design patterns trigger
3. Both intents fire correctly via prime-agent routing
4. Cross-wiring causes proofshot to surface alongside frontend-design suite skills
5. All mathematical formulas are computationally verified (done in brainstorming)

## Sources

- [Figma: Golden Ratio](https://www.figma.com/resource-library/golden-ratio/)
- [Modulor — Wikipedia](https://en.wikipedia.org/wiki/Modulor)
- [Typographic Scale Types — Cieden](https://cieden.com/book/sub-atomic/typography/different-type-scale-types)
- [Fractal Design Framework — Think Design](https://think.design/blog/fractal-design-framework/)
- [Muller-Brockmann: Grid Systems in Graphic Design](https://monoskop.org/images/a/a4/Mueller-Brockmann_Josef_Grid_Systems_in_Graphic_Design_Raster_Systeme_fuer_die_Visuele_Gestaltung_English_German_no_OCR.pdf)
- [Responsive Typography Scales — Design Shack](https://designshack.net/articles/typography/guide-to-responsive-typography-sizing-and-scales/)
