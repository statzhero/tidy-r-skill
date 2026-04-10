# tidy-r-skill

An LLM skill for modern tidyverse R using R 4.5+, tidyverse 2.0+, and dplyr 1.2+.

## What does it do?

When you ask an LLM to write R code, it draws on its training data, which mixes base R, old StackOverflow posts, deprecated tidyverse APIs, and current best practice. This skill gives the model a structured reference for modern tidyverse patterns so it produces clean, idiomatic code with less back-and-forth.

The skill covers: native pipe and lambda syntax, `join_by()` joins, `.by` grouping, the dplyr 1.2 recode/replace family, tidy selection, `stringr`, error handling with `cli`, and migration from base R or older tidyverse APIs.

## Installation

### Claude Code (one command)

Clone directly into your skills directory:

```bash
git clone https://github.com/statzhero/tidy-r-skill.git ~/.claude/skills/tidy-r
```

That's it. The skill is available immediately as `/tidy-r` in any Claude Code session.

If you prefer not to use the terminal, you can add skills from the Claude desktop app:

1. [Download this repository as a ZIP](https://github.com/statzhero/tidy-r-skill/archive/refs/heads/main.zip) from GitHub.
2. Open the Claude desktop app and switch to the **Code** tab.
3. Click **Customize** in the left sidebar, then select **Skills**.
4. Click the **+** button, choose **Upload a skill**, and select the ZIP file.

### Codex

Clone into your user skills directory (available across all projects):

```bash
git clone https://github.com/statzhero/tidy-r-skill.git ~/.agents/skills/tidy-r
```

If you prefer not to use the terminal, [download the ZIP](https://github.com/statzhero/tidy-r-skill/archive/refs/heads/main.zip), unzip it, and move the folder to `~/.agents/skills/tidy-r/` or `.agents/skills/tidy-r/` inside your project.

### Other LLMs

Paste the contents of `SKILL.md` into your system prompt or attach it as context. The reference files in `references/` can be appended when you need coverage of a specific topic.

## Test the skill

After installing, try this prompt in Claude Code:

```
/tidy-r Rewrite this using modern tidyverse:
penguins %>% group_by(species) %>% summarise(avg_bill = mean(bill_length_mm)) %>% ungroup()
```

The skill should guide the model toward `.by` grouping and native pipe.

## Acknowledgements

This skill stands on the shoulders of giants, but I built it over many iterations and can no longer trace any single influence. I regret not keeping better records. If you recognize your ideas here, please open an issue so I can credit you properly.

## References

| Topic | Source |
|-------|--------|
| Tidyverse style guide | https://style.tidyverse.org/ |
| dplyr | https://dplyr.tidyverse.org/ |
| stringr | https://stringr.tidyverse.org/ |
| tidyselect | https://tidyselect.r-lib.org/ |
| cli (error messages) | https://cli.r-lib.org/ |
| readr | https://readr.tidyverse.org/ |


## License

MIT • Ulrich Atz ([ulrichatz](https://bsky.app/profile/ulrichatz.org))

