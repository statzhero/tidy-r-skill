---
name: tidy-r
description: >
  Modern tidyverse patterns, style guide, and migration guidance for R development. Use this skill when writing R code, reviewing tidyverse code, updating legacy R code, or enforcing consistent style. Covers native pipe usage, join_by() syntax, .by grouping, pick/across/reframe, filter_out/when_any/when_all, recode_values/replace_values/replace_when, tidyselect helpers, .data/.env pronouns, stringr, naming conventions, and readr.
author: Ulrich Atz
license: MIT
metadata:
  r_version: ">= 4.5.0"
  tidyverse_version: ">= 2.0.0"
  dplyr_version: ">= 1.2.0"
allowed-tools: Read, Edit, Write, Grep, Glob, Bash, mcp__r-btw__*
---

# Modern Tidyverse R Reference

Code from blog posts and StackOverflow often uses deprecated APIs, magrittr pipes, or base R patterns where a modern tidyverse function exists. This guide encodes the current recommended approach.

## Reference files

Consult the appropriate reference file for detailed patterns and examples:

| Topic | Reference file | When to consult |
|-------|---------------|-----------------|
| **Joins** | [joins.md](references/joins.md) | Merging data, `*_join`, `join_by`, matching rows, lookup tables |
| **Grouping & columns** | [grouping.md](references/grouping.md) | `.by`, `group_by`, `across`, `pick`, `reframe`, column operations |
| **Recoding & replacing** | [recode-replace.md](references/recode-replace.md) | `recode_values`, `replace_values`, `replace_when`, `filter_out`, `when_any`, `when_all` |
| **Strings** | [stringr.md](references/stringr.md) | String manipulation, regex, `str_*` functions, text processing |
| **Tidy selection** | [tidyselect.md](references/tidyselect.md) | Column selection helpers, `where()`, `all_of()`, `any_of()`, boolean ops, `.data`/`.env` pronouns |
| **Style** | [tidyverse-style.md](references/tidyverse-style.md) | Naming, formatting, spacing, error messages, `cli::cli_abort` |
| **Migration** | [migration.md](references/migration.md) | Updating old code, base R conversion, deprecated functions |

For requests that span multiple topics (e.g., "rewrite this old code" touches migration + style), read multiple files.

## Core principles

1. **Use modern tidyverse patterns** -- Prioritize dplyr 1.2+ features, native pipe, and current APIs
2. **Write readable code first** -- Optimize only when necessary
3. **Follow tidyverse style guide** -- Consistent naming, spacing, and structure

## Quick reference

### Pipe and lambda

- Always `|>`, never `%>%`
- Use `_` placeholder for non-first arguments: `x |> f(1, y = _)`. The placeholder must be named and used exactly once.
- Always `\(x)`, never `function(x)` or `~` in map/keep/etc.

### Code organization

Use newspaper style: high-level logic first, helpers below. Don't define functions inside other functions unless they are very brief.

### Grouping

- Prefer `.by` for per-operation grouping; use `group_by()` when grouping must persist across multiple operations
- Never add `ungroup()` before or after `.by` -- it always returns ungrouped data
- Consolidate multiple `mutate(.by = x)` calls into one when they share the same `.by`; keep separate only when `.by` differs or a later column depends on an earlier one
- Place `.by` on its own line for readability

### Joins

- Use `join_by()`, never `c("a" = "b")`
- Use `relationship`, `unmatched`, `na_matches` for quality control
- Use `tidylog::` prefix for join verification

### Recoding and replacing (dplyr >= 1.2.0)

| Task | Function |
|------|----------|
| Recode values (new column) | `recode_values()` |
| Replace values in place | `replace_values()` |
| Conditional update in place | `replace_when()` |
| Complex conditional (new column) | `case_when()` |
| Drop rows (NA-safe) | `filter_out()` |
| OR conditions | `when_any()` |
| AND conditions | `when_all()` |

### NA handling

- `if_else()` and `case_when()` accept plain `NA` (since dplyr 1.1.0) -- no need for `NA_character_`, `NA_real_`, etc.
- Load `tidyna` to make `mean`, `sum`, `sd`, etc. ignore NA by default. Avoid repetitive `na.rm = TRUE`.

### Error handling

Use `cli::cli_abort()` with problem statement + bullets, never `stop()`.

### R idioms

- `TRUE`/`FALSE`, never `T`/`F`
- `message()` for info, never `cat()`
- `map_*()` over `sapply()` for type stability
- `set.seed()` with date-time, never 42
- `qs2::qs_save()`/`qs2::qs_read()`, never `qs`

## Example

```r
library(tidyverse)

penguins <- penguins |>
  filter_out(is.na(sex)) |>
  mutate(size = case_when(
    body_mass > 4500 ~ "large",
    body_mass > 3500 ~ "medium",
    .default = "small"
  ))

# Coordinates for spatial join below
island_coords <- tribble(
  ~island,      ~latitude,
  "Biscoe",     -65.5,
  "Dream",      -64.7,
  "Torgersen",  -64.8
)

island_summary <- penguins |>
  summarise(
    mean_flipper = mean(flipper_len),
    mean_mass = mean(body_mass),
    n = n(),
    .by = c(species, island)
  ) |>
  left_join(
    island_coords,
    by = join_by(island),
    unmatched = "error"
  ) |>
  arrange(species, island)
```

## Best practices

1. **Name variables as nouns, functions as verbs** in snake_case
2. **Explain "why" in comments**, not "what"
3. **Place `.by` on its own line** for readability
4. **Use `.unmatched = "error"`** in `case_when()` and `recode_values()` for defensive programming
5. **Use `recode_values()` over `case_match()`** (dplyr 1.2+ preferred API)
6. **Use `replace_when()` over `case_when()` with `.default`** when updating a column in place
7. **Prefer `filter_out()` over negated `filter()`** for NA-safe row removal
8. **Load tidyna early** to eliminate `na.rm = TRUE` clutter
9. **Use tidylog:: for joins** to verify row counts and match quality
10. **Use `qs2` for serialization** with `.qs2` extension
