---
name: tidy-r
description: >
  Modern tidyverse patterns, style guide, and migration guidance for R development. Use this skill when writing R code, reviewing tidyverse code, updating legacy R code to modern patterns, or enforcing consistent style. Covers native pipe usage, join_by() syntax, .by grouping, pick/across/reframe operations, filter_out/when_any/when_all, recode_values/replace_values/replace_when, tidy selection, stringr patterns, naming conventions, and migration from base R or older tidyverse APIs. Use the R (btw) MCP tools to resolve function documentation and library references automatically.
author: Ulrich Atz
license: CC-BY-4.0
metadata:
  r_version: ">=4.5.0"
  tidyverse_version: ">=2.0.0"
  dplyr_version: ">=1.2.0"
allowed-tools: Read, Edit, Write, Grep, Glob, Bash, mcp__r-btw__*
---

# Modern Tidyverse R Reference

Code from blog posts and StackOverflow often uses deprecated APIs, magrittr pipes, or base R patterns where a modern tidyverse function exists. This guide encodes the current recommended approach.

## When to use this skill

- Writing new R code with dplyr, tidyr, stringr, purrr, or other tidyverse packages
- Reviewing or refactoring existing R code for modern patterns
- Migrating from base R, magrittr pipes, or older tidyverse APIs
- Applying tidyverse style conventions (naming, spacing, error handling)
- Choosing between similar functions (e.g., `case_when` vs `recode_values`)
- Working with joins, grouping, recoding, or string manipulation in R

## When NOT to use this skill

- Writing data.table code (different paradigm)
- Pure base R projects that intentionally avoid tidyverse
- Shiny UI/server logic (use a Shiny-specific skill)
- Package development internals (NAMESPACE, DESCRIPTION, roxygen)
- ggplot2 visualization (use the socviz skill)
- Statistical modeling or Bayesian analysis

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
4. **Use R MCP tools** -- Automatically resolve function documentation and library references without being asked. If the `mcp__r-btw__*` tools are unavailable, fall back to running R help via Bash (see below)

### R documentation lookup fallback

When `mcp__r-btw__*` tools are available, use them to look up function signatures, help pages, and package docs. When they are not available (e.g., the r-btw MCP server is not configured), fall back to Bash:

```bash
# Help page for a function
Rscript --vanilla -e '?dplyr::recode_values' 2>/dev/null || Rscript --vanilla -e 'utils::help("recode_values", package = "dplyr")'

# Function signature / arguments
Rscript --vanilla -e 'args(dplyr::recode_values)'

# List exported functions in a package
Rscript --vanilla -e 'ls("package:dplyr")'

# Check if a package is installed
Rscript --vanilla -e 'requireNamespace("tidyna", quietly = TRUE)'
```

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

### Recoding and replacing (dplyr >=1.2.0)

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

Load `tidyna` to make `mean`, `sum`, `sd`, etc. ignore NA by default. Avoid repetitive `na.rm = TRUE`.

### Error handling

Use `cli::cli_abort()` with problem statement + bullets, never `stop()`.

### R idioms

- `TRUE`/`FALSE`, never `T`/`F`
- `message()` for info, never `cat()`
- `map_*()` over `sapply()` for type stability
- `set.seed()` with date-time, never 42
- `qs2::qs_save()`/`qs2::qs_read()`, never `qs`

## Anti-patterns

| Avoid | Use instead |
|-------|-------------|
| `%>%` | `|>` |
| `function(x)` or `~` | `\(x)` |
| `by = c("a" = "b")` | `by = join_by(a == b)` |
| `multiple = "error"` in joins | `relationship = "many-to-one"` (or `"one-to-one"`) |
| `sapply()` | `map_*()` (type-stable) |
| `group_by() \|> ... \|> ungroup()` | `.by` argument |
| `cat()` for messages | `message()` or `cli::cli_inform()` |
| `stop()` for errors | `cli::cli_abort()` |
| `distinct(id)` | `distinct(id, .keep_all = TRUE)` |
| `mean(x, na.rm = TRUE)` | `mean(x)` with tidyna loaded |
| `case_match(x, ...)` | `recode_values(x, ...)` |
| `recode(x, ...)` | `recode_values(x, ...)` or `replace_values(x, ...)` |
| `filter(x != val \| is.na(x))` | `filter_out(x == val)` |
| `coalesce(x, default)` | `replace_values(x, NA ~ default)` |
| `na_if(x, val)` | `replace_values(x, val ~ NA)` |
| `qs::qsave()` / `qs::qread()` | `qs2::qs_save()` / `qs2::qs_read()` |

## Example

```r
library(tidyverse)
library(tidyna)

# Read and clean data
sales <- read_csv("data/sales.csv") |>
  rename(
    region = Region,
    product = Product,
    revenue = Revenue,
    date = Date
  ) |>
  mutate(
    quarter = quarter(date),
    product = product |>
      replace_values(
        c("Widget A", "WidgetA") ~ "Widget A",
        c("Widget B", "WidgetB") ~ "Widget B"
      )
  ) |>
  filter_out(is.na(revenue))

# Enrich with lookup table
sales_enriched <- sales |>
  tidylog::left_join(
    regions,
    by = join_by(region == region_code),
    unmatched = "error"
  )

# Summarise by group
quarterly <- sales_enriched |>
  summarise(
    total_revenue = sum(revenue),
    avg_revenue = mean(revenue),
    n_transactions = n(),
    .by = c(region_name, quarter)
  ) |>
  mutate(
    performance = case_when(
      total_revenue > 100000 ~ "high",
      total_revenue > 50000  ~ "medium",
      .default = "low"
    )
  ) |>
  arrange(region_name, quarter)
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
