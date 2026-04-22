# API Response Schema

The agent expects the Claude API to return a single JSON object matching the schema below. All fields are required unless marked optional.

---

## Top-Level Structure

```json
{
  "phase1": { ... },
  "phase2": { ... },
  "phase3": { ... },
  "phase4": { ... }
}
```

---

## Phase 1 — Market Landscape

```json
{
  "phase1": {
    "summary": "string — 2-3 sentence overview",
    "categories": [
      {
        "name":  "string — category label",
        "size":  "string — market size descriptor",
        "trend": "string — one of: Growing | Stable | Declining",
        "notes": "string — key insight"
      }
    ],
    "keyTensions": [
      "string — market tension description"
    ]
  }
}
```

---

## Phase 2 — Gap Identification

```json
{
  "phase2": {
    "gaps": [
      {
        "id":                   "string — e.g. gap-1",
        "title":                "string — gap label",
        "icon":                 "string — single emoji character",
        "unmetNeed":            "string",
        "techFeasibility":      "integer — 1 to 5",
        "demandSignal":         "string",
        "competitiveWhitespace":"string",
        "culturalResonance":    "string",
        "overallScore":         "integer — 1 to 10"
      }
    ]
  }
}
```

Exactly **5 gap objects** are expected.

---

## Phase 3 — Instrument Briefs

```json
{
  "phase3": {
    "briefs": [
      {
        "id":               "string — e.g. brief-1",
        "name":             "string — instrument name",
        "tagline":          "string — short evocative tagline",
        "targetPlayer":     "string",
        "coreInnovation":   "string",
        "soundCapabilities":"string",
        "materials":        "string",
        "priceTier":        "string — one of: Beginner | Prosumer | Professional | Bespoke",
        "goToMarket":       "string",
        "risks":            "string",
        "opportunityScore": "integer — 0 to 100"
      }
    ]
  }
}
```

Exactly **5 brief objects** are expected, one per gap.

---

## Phase 4 — Ranked Opportunities

```json
{
  "phase4": {
    "rankedOpportunities": [
      {
        "rank":             "integer — 1 to 5",
        "instrumentId":     "string — matches a brief.id",
        "name":             "string",
        "innovationImpact": "integer — 0 to 10",
        "marketSize":       "integer — 0 to 10",
        "buildFeasibility": "integer — 0 to 10",
        "culturalTiming":   "integer — 0 to 10",
        "compositeScore":   "integer — 0 to 100",
        "rationale":        "string — 2-3 sentences"
      }
    ],
    "conclusion": "string — 2-3 sentence synthesis"
  }
}
```

Exactly **5 ranked objects** are expected.

---

## Validation Rules (enforced in `src/index.html`)

| Field | Sanitiser | Constraints |
|-------|-----------|-------------|
| All strings | `esc()` | HTML-encoded before DOM insertion |
| `techFeasibility` | `safeInt(val, 1, 5)` | Clamped to 1–5 |
| `overallScore` | `safeInt(val, 0, 10)` | Clamped to 0–10 |
| `opportunityScore` | `safeInt(val, 0, 100)` | Clamped to 0–100 |
| `rank` | `safeInt(val, 1, 99)` | Clamped to 1–99 |
| `innovationImpact` etc. | `safeInt(val, 0, 10)` | Clamped to 0–10 |
| `compositeScore` | `safeInt(val, 0, 100)` | Clamped to 0–100 |
| `priceTier` (CSS colour) | `tierColour()` allow-list | Returns safe hex, ignores unknown values |
