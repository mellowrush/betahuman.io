# betahuman.io — Brand System Design
> Version 0.1 · Working title. Not finished. Not final. Still becoming.

---

## 1. Brand Concept

**betahuman.io** is a personal brand built around a single honest premise: an identity in progress. The name encodes the core tension — *beta* (iterative, unfinished, deliberate) + *human* (not a product, not a persona) + *.io* (builder's domain, technical credibility without the corporate weight).

This is not a portfolio. Not a resume. Not a thought leadership platform.
It is a digital home for someone navigating technology, complexity and becoming.

**The single positioning line:**
> Exploring systems, tools and the search for meaningful work.

**The manifesto fragment:**
> Not finished. Not final. Still becoming.

---

## 2. Audience

**Primary:** People who feel like they ended up somewhere they didn't fully choose — and are rebuilding deliberately. Builders, thinkers, career transitioners, makers who value depth over speed.

**Secondary:** Fellow frontend developers and creative technologists who care about craft and reject superficiality.

**Tone calibration:** Speak to someone who reads long-form, distrusts personal branding theater and would immediately notice if you were performing instead of thinking.

---

## 3. Visual Identity

### 3.1 Color Palette

| Name  | Hex       | Role                              |
|-------|-----------|-----------------------------------|
| Ink   | `#1a1814` | Primary background, dominant dark |
| Rust  | `#c4522a` | Accent, energy, signal            |
| Slate | `#3d4a5c` | Secondary tone, depth             |
| Gold  | `#b8962e` | Tertiary highlight, earned warmth |
| Paper | `#f5f2eb` | Light surfaces, negative space    |
| Bone  | `#e8e3d8` | UI borders, subtle dividers       |

**Color philosophy:** Warm-dark primary (Ink/Paper) avoids the cold sterility of pure black/white. Rust is used sparingly — it should feel deliberate, like a mark, not decoration. Gold signals earned insight, not performance.

### 3.2 Typography

| Role          | Family               | Weight | Notes                          |
|---------------|----------------------|--------|--------------------------------|
| Display/Logo  | DM Serif Display     | 400    | Italic for manifesto fragments |
| Headlines     | Syne                 | 700–800| Geometric authority, uncommon  |
| Body / UI     | Syne                 | 400    | Consistent with headlines      |
| Code / Labels | DM Mono              | 400–500| Anchors technical credibility  |

**Import:**
```css
@import url('https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Mono:wght@400;500&family=Syne:wght@400;600;700;800&display=swap');
```

**Typography rules:**
- Never use Inter, Roboto, or system fonts in brand contexts
- DM Serif Display italic = manifesto / emotional statements only
- DM Mono = metadata, labels, technical anchors
- Syne = everything else

### 3.3 Logo / Wordmark

```
betahuman.io
```

- `beta` in Paper or Ink
- `human` in Rust
- `.io` in DM Mono, reduced opacity (≈35%)
- Set in DM Serif Display at display sizes
- Set in Syne 700 at UI sizes

**Do not:** add icons, symbols, or graphic marks. The wordmark is the mark.

### 3.4 Grid & Layout

**Brand presentation grid:** 3-column asymmetric (1.6fr 1fr 1fr), 8px gap, 8px padding, bone background. Each panel is a self-contained statement — no panel should need another to make sense.

**Panel types:**
- Hero (dark ink, wordmark, tagline)
- Photography/person (slate, human presence)
- Manifesto (rust accent, italic serif)
- Interface mockup (paper, technical signal)
- Principles (gold-light, concrete values)
- Domain (slate-light, technical ownership)
- Palette (neutral, reference)
- Positioning (dark ink, closing statement)

### 3.5 Aesthetic Principles

- **Warmth over coldness:** Ink/Paper over pure black/white
- **Texture without decoration:** Bone/Paper contrast creates subtle material feel
- **Restraint:** One accent color per panel maximum
- **Typographic hierarchy:** Size + weight carry hierarchy, not color alone
- **Micro-labeling:** DM Mono at 8–9px for all metadata/labels — small but present

---

## 4. Voice & Tone

### What betahuman.io sounds like:

- First-principles reasoning, shown not declared
- Questions held open, not packaged as answers
- Specificity over generality ("I rebuilt this three times before understanding why" not "I'm passionate about quality")
- Honest about uncertainty, not performatively humble

### Register:
- Written. Considered. Slightly editorial.
- Not: casual, bro-ish, productivity-guru, thought-leader, startup-hype
- Closest references: Robin Sloan's writing andy Matuschak's notes, dense technical essays with moments of reflection

### Voice fragments (usable as copy):
- *"Notes from the journey toward a more deliberate life."*
- *"Still figuring it out. Documenting as I go."*
- *"The map is not the territory. But maps help."*
- *"A career is a working title."*

---

## 5. Content System

### Content types betahuman.io can hold:

| Type              | Format           | Frequency     |
|-------------------|------------------|---------------|
| Field notes       | Short written    | High          |
| Tool builds       | Case study       | Medium        |
| Systems thinking  | Long-form essay  | Low           |
| Reading traces    | Annotations      | Ongoing       |
| Project pages     | Single-page      | Per project   |

**The container test:** Would this content embarrass someone in five years if they became more certain? If yes, frame it as a note, not a declaration.

---

## 6. Technical Foundations

- **Stack:** React + JavaScript (existing expertise, no forced tool changes)
- **Hosting:** Static / edge-friendly (client-side generation where possible)
- **Performance philosophy:** Fast by default. No heavy frameworks for content.
- **Monetization:** Honest — if something is paid, label it clearly
- **Complexity budget:** Keep the site shippable by one person. Resist scope creep.

---

## 7. What betahuman.io is NOT

- Not a portfolio site (no "hire me" energy)
- Not a blog (not post-centric)
- Not a personal brand in the LinkedIn sense
- Not finished (the beta is real, not ironic)
- Not trying to be an authority (trying to be honest)

---

## 8. Evolution Path

This is version 0.1. The brand system should be treated like software:

- **v0.1:** Identity, domain, first presence
- **v0.2:** Content system live, first 10 field notes
- **v0.3:** First tool or project page under the brand
- **v1.0:** When it stops feeling like a working title

The versioning is part of the brand. Changelog thinking applied to a life.

---

*betahuman.io · Bucharest · v0.1*
*"Not finished. Not final. Still becoming."*
