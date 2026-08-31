# fabrication-audit

Verifies the specific, checkable numbers in a draft before it ships. Doses, thresholds, resistance percentages, mortality figures, effect sizes, guideline names and years, registry numbers, fees.

This is the canonical copy. Three other plugins bundle it as a dependency.

## What it catches

Language models state retrievable values with the same confidence whether they retrieved them or generated them. The output reads identically either way, which is the whole problem.

This skill walks every "the X is Y" claim in a draft and sorts it by provenance:

| Class | Handling |
|---|---|
| Supplied by the user | Accept, do not re-verify |
| Verified this session by a tool call | Accept, note the source |
| Retrievable but unverified | **Verify now, or flag** |
| Not retrievable | Flag as unverifiable |

Anything in the third row that fails verification is surfaced, not softened.

## The failure it was built around

A jurisdiction claim — which government field office serves which region — stated confidently and wrong, with a fabricated processing time attached. Both looked like facts. Neither had been looked up. The worked examples in `SKILL.md` come from that class of error: geography and source mismatches, where the value is plausible for *somewhere* but wrong for *here*.

## Install

```
/plugin marketplace add neelshah4/claude-plugins
/plugin install fabrication-audit@neel-plugins
```

Or directly:

```
/plugin install neelshah4/claude-fabrication-audit
```

## Use

```
Audit every specific in this draft before I send it
```

It runs well as a final gate, after the prose is settled. Running it mid-draft wastes verification on sentences you will cut.

## Scope boundary

It does **not** handle academic identifiers — PMIDs, DOIs, PMC, arXiv, NCT. That is [citation-verification](https://github.com/neelshah4/claude-citation-verification), which uses different lookups and different acceptance rules. Run both. Each alone leaves half the claim surface unchecked.

## If you install more than one of these plugins

`grant-reviewer`, `icu-clinical-consult`, and `citation-verification` each bundle a copy of this skill so they work standalone. Installing this repo alongside them gives you the canonical version. If the copies drift, this one wins.

## Requirements

- Web access.
- Claude Code with the Agent tool for the `fabrication-auditor` subagent path. Without it, the audit runs inline.

## Version

1.0.0. See [CHANGELOG.md](CHANGELOG.md).

## License

MIT. Author: Neel Shah, MD, MSc.
