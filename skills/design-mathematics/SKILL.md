---
name: design-mathematics
description: Classical design proportions, information weighting formulas, and mathematical
  verification criteria for UI work. Use when building layouts, choosing type scales,
  setting spacing, verifying visual hierarchy, or designing information-dense interfaces
  like dashboards. Integrates with proofshot for verification.
---

# Design Mathematics — Classical Proportions & Information Design

Mathematical foundations for UI design, drawn from architecture (Vitruvius, Le Corbusier), typography (Muller-Brockmann, Tschichold), music theory, information theory (Shannon), and human factors (Fitts). These are not aesthetic opinions — they are computable, verifiable properties.

## Iron Law

**Every design decision should trace to a named ratio, formula, or geometric relationship.** If you cannot name the mathematical principle behind a spacing value, a font size, a column width, or a color choice, it is arbitrary. Arbitrary choices compound into visual noise. Named proportions compound into harmony.

## 1. Musical Typographic Scale

Type sizes follow ratios derived from Western tonal intervals. Choose one ratio for a project and apply it consistently.

| Interval | Ratio | Origin | Best For |
|----------|-------|--------|----------|
| Minor Second | 1.067 | 16/15 | Dense text, legal, academic |
| Major Second | 1.125 | 9/8 | Web body copy, long-form |
| Minor Third | 1.200 | 6/5 | UI text levels, apps |
| Major Third | 1.250 | 5/4 | Marketing, presentations |
| Perfect Fourth | 1.333 | 4/3 | Editorial, news sites |
| Augmented Fourth | 1.414 | sqrt(2) | Landing pages, promos |
| Perfect Fifth | 1.500 | 3/2 | Banners, infographics |
| Golden Ratio | 1.618 | (1+sqrt(5))/2 | Luxury, portfolios |

**Choosing a ratio:**
- Mobile-first: start with Major Third (1.250), step up to Perfect Fourth on tablet, Augmented Fourth on desktop
- Information-dense dashboards: Minor Third (1.200) — enough differentiation without wasting vertical space
- Marketing: Perfect Fourth (1.333) or higher for dramatic hierarchy

**Computing a scale** from 16px base at Minor Third (1.200):
```
caption:  11px  (16 / 1.2^2)
small:    13px  (16 / 1.2)
body:     16px  (base)
lg:       19px  (16 × 1.2)
xl:       23px  (16 × 1.2^2)
2xl:      28px  (16 × 1.2^3)
3xl:      33px  (16 × 1.2^4)
```

## 2. Le Corbusier's Modulor

Human-scale proportions: navel height (1.13m) and arm-raised height (2.26m), each subdivided by phi. The Red series and Blue series form interlocking Fibonacci sequences grounded in the human body.

**Red Series** (cm): 113, 69.8, 43.2, 26.7, 16.5, 10.2, 6.3, 3.9
**Blue Series** (cm): 226, 139.7, 86.3, 53.4, 33.0, 20.4, 12.6, 7.8

**Design application:** Use Modulor values (converted to px or rem) for spacing in interfaces that need to feel physically grounded — dashboards viewed at arm's length, kiosk UIs, and signage.

## 3. Pythagorean Visual Paths

Integer right triangles create clean diagonal tension:

| Triple | Angle | Application |
|--------|-------|-------------|
| 3:4:5 | 53.1deg | Standard CTA offset from heading |
| 5:12:13 | 67.4deg | Steep card-to-action diagonal |
| 8:15:17 | 61.9deg | Wide layout cross-reference |

**How to use:** If your heading is at position (0, 0), place the primary CTA at (3n, 4n) for some unit n. The eye follows a path of length exactly 5n — no irrational visual tension.

## 4. Rule of Thirds & Golden Sections

```
Rule of thirds:   1/3 = 0.333  |  2/3 = 0.667
Golden section:   1/phi = 0.382  |  phi-1 = 0.618
```

The rule of thirds is the integer approximation of the golden section. Both place visual weight at off-center positions that feel balanced.

**Power positions** (where the eye naturally rests):
```
    1/3       1/3       1/3
 +--------+--------+--------+
 |        |   *    |        |  1/3    * = power positions
 +--------+--------+--------+
 |        |        |   *    |  1/3
 +--------+--------+--------+
 |        |        |        |  1/3
 +--------+--------+--------+
```

For dashboards: place the most critical metric at the upper-left power position. Place the primary action at the lower-right.

## 5. Fractal Self-Similarity

Components at scale 1/phi of their parent:

```
Page:      1000px
Section:    618px  (1000 / phi)
Card:       382px  (1000 / phi^2)
Element:    236px  (1000 / phi^3)
Detail:     146px  (1000 / phi^4)
```

**The test:** Does the same proportional relationship hold at every nesting level? If your page is split 62/38, are your cards split 62/38 internally? If not, the fractal is broken.

## 6. Color Wheel Geometry

| Harmony | Angle(s) | Feeling |
|---------|----------|---------|
| Complementary | 180deg | High contrast, energetic |
| Split-complementary | 150deg, 210deg | Contrast without tension |
| Triadic | 120deg, 240deg | Balanced, vibrant |
| Tetradic (square) | 90deg, 180deg, 270deg | Rich, needs care |
| Analogous | 30deg, 60deg | Calm, cohesive |
| Golden angle | 137.508deg | Nature's distribution (phyllotaxis) |

**Verification:** If you cannot name the geometric relationship between your colors on the wheel, the palette is noise.

## 7. Font-to-Icon Ratio

| Method | Formula | When to use |
|--------|---------|-------------|
| sqrt(2) rule | icon = font x 1.414 | Default — diagonal of font-height square |
| phi rule | icon = font x 1.618 | When icons need more breathing room |
| 2x rule | icon = font x 2.0 | Material Design, touch-heavy interfaces |

At 16px body text: icons should be 23px (sqrt2), 26px (phi), or 32px (2x). Never 16px — that makes icons appear smaller than text.

## 8. Grid Mathematics (Muller-Brockmann)

**Golden 2-column split** at 1440px: main 890px (61.8%), sidebar 550px (38.2%).

Standard grids with 24px gutters at 1440px base:

| Columns | Width each | Use case |
|---------|-----------|----------|
| 2 | 708px | Split layout |
| 3 | 464px | Card grid |
| 4 | 342px | Dashboard panels |
| 6 | 220px | Dense dashboard |
| 12 | 98px | Maximum flexibility |

## 9. Reading Pattern Geometry

**F-Pattern** (content-heavy pages, dashboards):
- First horizontal sweep: 100% width
- Second horizontal: ~60% width
- Vertical stem: left 20%
- Attention decays exponentially: ~1/e per viewport fold

**Z-Pattern** (hero/landing pages):
- Top-left to top-right (brand/nav)
- Diagonal to bottom-left (visual bridge)
- Bottom-left to bottom-right (CTA)

**Dashboard implication:** Place the most critical KPI in the top-left. The user's eye hits it first on every visit.

## 10. Touch Target Mathematics

| Standard | Size | Context |
|----------|------|---------|
| Apple HIG | 44 x 44 pt | iOS minimum |
| Material Design | 48 x 48 dp | Android minimum |
| WCAG 2.5.5 (AA) | 24 x 24 CSS px | Web minimum |
| WCAG 2.5.8 (AAA) | 44 x 44 CSS px | Web recommended |

**Thumb zone rule of thirds:** On mobile, the bottom third of the screen is easy reach. The top third is hard reach. Place primary actions in the bottom third.

---

## Information Design Formulas

For information-dense interfaces (dashboards, feeds, admin panels), mathematical weighting determines what to show, how prominently, and what to hide.

### 11a. Recency Decay

```
R(t) = e^(-lambda * t)
lambda = ln(2) / half_life_in_days
```

| Content Type | Half-life | lambda | Weight at 7d | Weight at 30d |
|-------------|-----------|--------|-------------|---------------|
| Breaking alert | 1 day | 0.693 | 0.01 | ~0 |
| Status update | 7 days | 0.099 | 0.50 | 0.05 |
| Documentation | 30 days | 0.023 | 0.85 | 0.50 |
| Reference | 365 days | 0.002 | 0.99 | 0.94 |

**Application:** Old alerts should fade, shrink, and eventually hide. Reference data stays visible indefinitely.

### 11b. Importance Scoring

```
I(p) = 1 + log2(p)    where p = priority (1-10)
```

Logarithmic scaling prevents priority-10 items from overwhelming everything else. A priority-10 item gets 4.3x the visual weight of priority-1, not 10x. This preserves a readable hierarchy.

### 11c. Volume Normalization

```
V(n) = 1 / sqrt(n)    where n = items in category
```

| Items | Weight | Layout |
|-------|--------|--------|
| 1 | 1.000 | Full prominence — hero card |
| 4 | 0.500 | Small group — medium cards |
| 9 | 0.333 | Grid — compact cards |
| 25 | 0.200 | Index — list items |
| 100 | 0.100 | Archive — table rows |

**Dashboard application:** If a category has 100 items, each gets 10% of the visual weight of a single item. This naturally drives progressive disclosure — show counts and summaries, not 100 full cards.

### 11d. Combined Visual Weight

```
W(i) = alpha * I(p) + beta * R(t) + gamma * V(n)
where alpha + beta + gamma = 1
```

| Context | alpha (importance) | beta (recency) | gamma (volume) |
|---------|--------------------|----------------|----------------|
| Ops dashboard | 0.5 | 0.3 | 0.2 |
| News feed | 0.2 | 0.6 | 0.2 |
| Documentation | 0.4 | 0.1 | 0.5 |
| Portfolio | 0.3 | 0.3 | 0.4 |

**Weight-to-design mapping:**

| W(i) | Font | Position | Contrast | Spacing |
|------|------|----------|----------|---------|
| > 0.8 | h1/3xl | Above fold, power position | 100% | phi^3 padding |
| 0.6-0.8 | h2/2xl | Top third | 87% | phi^2 padding |
| 0.4-0.6 | h3/xl | Mid page | 74% | phi^1 padding |
| 0.2-0.4 | body+ | Lower third | 60% | base padding |
| < 0.2 | caption | **HIDE or collapse** | 45% | half base |

**The hiding rule:** Items with W(i) < 0.2 should be collapsed behind progressive disclosure (expand/collapse, "show more", drill-down). They exist but do not compete for attention. This is not removing information — it is respecting the viewer's cognitive budget.

### 11e. Information Entropy

```
H = -sum(p_i * log2(p_i))    (Shannon entropy)
```

Entropy tells you which layout pattern to use:

| Entropy | Meaning | Layout |
|---------|---------|--------|
| H < 2 | Few dominant items | Hero + supporting cards |
| H ~ 3 | Mixed importance | Card grid with sections |
| H > 4 | Many equal items | Table, uniform rows |

**Dashboard application:** Compute the entropy of your KPI importance distribution. If one metric dominates (low entropy), give it a hero treatment. If all metrics are roughly equal (high entropy), use a uniform grid.

### 11f. Fitts's Law

```
T = a + b * log2(1 + D/W)
```

Time to reach a target increases with distance (D) and decreases with target width (W).

**Design implication:** The most important interactive elements need:
- Larger W (bigger touch/click targets)
- Smaller D (closer to resting cursor/thumb)
- Both — not just one

## Verification Checklist

When reviewing or verifying UI (especially with proofshot), check these:

1. **Named ratio:** Can you name the typographic scale ratio in use?
2. **Pythagorean path:** Do primary elements form clean triangle relationships?
3. **Power positions:** Do CTAs sit at rule-of-thirds intersections?
4. **Reading pattern:** Does hierarchy follow F-pattern (dashboard) or Z-pattern (hero)?
5. **Fractal consistency:** Does the 62/38 split repeat across nesting levels?
6. **Color harmony:** Can you name the geometric relationship on the wheel?
7. **Font-icon ratio:** Are icons sized at text x sqrt(2) or text x phi?
8. **Information weight:** Are low-W items hidden or collapsed?
9. **Entropy match:** Does the layout pattern match the content entropy?
10. **Fitts compliance:** Are primary actions large and close to resting position?

## Integration with ProofShot

After building UI, run proofshot and apply this checklist during the `exec` phase:

```bash
proofshot start --run "npm run dev" --port 3000 --description "Verify design mathematics"
proofshot exec screenshot before-verification.png
# Apply checklist items 1-10 by inspecting the screenshot
proofshot exec screenshot after-verification.png
proofshot stop
```

If any checklist item fails, fix the design and re-verify before posting proof to the PR.

## Red Flags — You're Rationalizing

| Thought | Reality |
|---------|---------|
| "I'll just eyeball the spacing" | Eyeballing produces arbitrary values. Use a named scale. |
| "Golden ratio is overrated" | It appears in sunflowers, nautilus shells, and the Parthenon. Use it. |
| "The type scale doesn't need to be exact" | Rounding 25.89px to 26px is fine. Picking 28px "because it looks good" is not. |
| "This dashboard doesn't need information hiding" | If W(i) < 0.2 for any element, it needs progressive disclosure. |
| "Users want to see everything" | Shannon entropy says otherwise. High entropy = uniform grid = cognitive overload. |
| "These formulas are too academic" | Fitts's Law is used by every major design system. Apple, Google, Microsoft. |
| "I know the right color without checking the wheel" | Name the harmony. If you can't, it's noise. |
| "The icon size is close enough" | 16px icon next to 16px text looks smaller. Use font x sqrt(2). |
