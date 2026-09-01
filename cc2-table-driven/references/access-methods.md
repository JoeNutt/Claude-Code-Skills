# Table-Driven Access Methods — worked patterns

Examples are written in neutral pseudocode. Translate to the target language's idiom;
prefer its native constructs (sum types, exhaustive `match`, frozen maps, enums) over a
literal transcription.

---

## 1. Direct access

The input maps straight onto a key or index. No search, no comparison chain.

**Before**

```
function shippingCost(method):
    if method == "standard": return 4.99
    else if method == "express": return 12.99
    else if method == "overnight": return 24.99
    else if method == "pickup": return 0.00
    else: return 4.99
```

**After**

```
SHIPPING_COST = {
    "standard":  4.99,
    "express":  12.99,
    "overnight": 24.99,
    "pickup":    0.00,
}
DEFAULT_SHIPPING_COST = 4.99

function shippingCost(method):
    return SHIPPING_COST.get(method, DEFAULT_SHIPPING_COST)
```

Adding a method is now a one-line data edit with no control flow touched.

### Dispatching behavior, not just values

The same shape holds when what varies is an action. Keep each handler a named routine —
a table of long inline lambdas is less readable than the conditional it replaced.

```
EVENT_HANDLERS = {
    "created":   handleCreated,
    "updated":   handleUpdated,
    "deleted":   handleDeleted,
}

function dispatch(event):
    handler = EVENT_HANDLERS.get(event.type)
    if handler is null:
        raise UnknownEventType(event.type)   # explicit miss, never silent
    return handler(event)
```

### Miss-case rules

Decide deliberately which of these you want, and make it visible at the lookup site:

- **Default value** — a sensible fallback exists (`.get(key, DEFAULT)`).
- **Explicit error** — an unknown key is a bug or bad input; raise.
- **Null object** — a do-nothing handler that satisfies the interface.

Never let an unmatched key fall through to an undefined value.

---

## 2. Indexed access

Use when the key space is too large, too sparse, or too irregular for direct indexing —
a five-digit product code where only forty codes exist, and a 100,000-slot array would be
almost entirely empty.

Two structures: a small **index** mapping the sparse key to a dense position, and a
compact **data table** holding the actual rows.

```
# Sparse, irregular keys -> dense positions
PRODUCT_INDEX = {
    "10057": 0,
    "20399": 1,
    "84412": 2,
}

# Dense table, cheap to scan, store, and extend
PRODUCT_RULES = [
    { discount: 0.10, taxable: true,  category: "tools"    },
    { discount: 0.00, taxable: true,  category: "food"     },
    { discount: 0.25, taxable: false, category: "medical"  },
]

function rulesFor(productCode):
    position = PRODUCT_INDEX.get(productCode)
    if position is null:
        return DEFAULT_RULES
    return PRODUCT_RULES[position]
```

Why bother rather than one big map: the index stays small and cache-friendly, several
tables can share one index, and the row data can be reordered or bulk-loaded (from a file,
a database, a config) without touching the key mapping.

In a language with a good hash map and a modest key count, a single direct-access map is
simpler — use it. Reach for indexed access when multiple tables key off the same sparse
domain, or when the row data is loaded from outside the source file.

---

## 3. Stair-step access

Use when inputs fall into **ranges**: tax brackets, grading scales, shipping weight bands,
discount tiers, retry backoff levels.

**Before**

```
function grade(score):
    if score >= 90: return "A"
    else if score >= 80: return "B"
    else if score >= 70: return "C"
    else if score >= 60: return "D"
    else: return "F"
```

**After**

```
# Ordered ascending by upper bound. Order is load-bearing.
GRADE_BANDS = [
    { upperBound:  59.9, grade: "F" },
    { upperBound:  69.9, grade: "D" },
    { upperBound:  79.9, grade: "C" },
    { upperBound:  89.9, grade: "B" },
    { upperBound: 100.0, grade: "A" },
]

function grade(score):
    for band in GRADE_BANDS:
        if score <= band.upperBound:
            return band.grade
    return "A"   # scores above the last bound
```

### The traps

- **Order is load-bearing.** The list must be sorted by bound, and a reader must be told so.
  Add a comment, or sort it at load time and let the code enforce what the comment claims.
- **Endpoints.** Fix whether each bound is inclusive or exclusive, state it once, and hold
  to it. Off-by-one at a bracket boundary is the classic defect in this pattern, and it is
  invisible in testing unless you test the boundaries themselves.
- **Both ends.** Handle input below the first bound and above the last explicitly.
- **Floats.** For money or anything where accumulated rounding matters, use integer minor
  units (cents) or a decimal type for the bounds, never binary floats.
- For a long, hot table, binary-search the bounds instead of walking them — but only once a
  measurement says the walk matters.

### Test what the table promises

Boundary values are the whole risk surface. For each band, test the exact bound, one unit
either side of it, and the open ends below the first and above the last.
