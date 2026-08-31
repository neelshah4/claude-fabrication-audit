# fabrication-audit

Verifies the specific, checkable numbers in a draft before it ships. Doses, thresholds, resistance percentages, mortality figures, effect sizes, guideline names and years, registry numbers, fees, prices.

This is the canonical copy. Two other plugins bundle it as a dependency.

---

## Contents

- [The problem](#the-problem)
- [How it classifies a claim](#how-it-classifies-a-claim)
- [What triggers it](#what-triggers-it)
- [Install](#install)
- [Worked examples](#worked-examples)
- [The failure it was built around](#the-failure-it-was-built-around)
- [Scope boundary](#scope-boundary)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Design notes](#design-notes)

---

## The problem

A language model states a retrievable value with identical confidence whether it retrieved the value or generated it. The output reads the same either way. There is no tell in the prose, no hedge, no formatting difference — which is the entire problem, and the reason this cannot be solved by reading carefully.

The claim shape is always the same: **"the X is Y"** or **"X serves Y"**, where Y is a specific value someone could look up. That pattern is what this skill walks.

---

## How it classifies a claim

Every specific in the draft lands in one of four buckets:

| Class | Handling |
|---|---|
| **Supplied by the user** | Accept. It is their number. Do not re-verify. |
| **Verified this session by a tool call** | Accept, note the source. |
| **Retrievable but unverified** | **Verify now, or flag.** This is the whole job. |
| **Not retrievable** | Flag as unverifiable and say why. |

Row three is where fabrications live. A number that *could* have been looked up and *was not* is indistinguishable from one that was, until someone checks.

---

## What triggers it

Nearly every substantive turn. It is deliberately aggressive:

- Numbers, dates, distances, drive times
- Addresses and jurisdictions — which office or agency covers which place
- Government fees and processing times
- Drug doses, prices, statistics
- Antimicrobial susceptibility and resistance rates
- Version numbers, current officeholders
- Sports and finance specifics — cap hits, ETF holdings, salaries

**Embedded sub-claims get the same treatment as direct answers.** A number buried in paragraph four of a long response is exactly as unverified as one given as the whole answer, and is likelier to slip through.

It does **not** trigger on greetings, pure code, pure mathematical derivation, or fictional creative writing.

---

## Install

```
/plugin marketplace add neelshah4/claude-plugins
/plugin install fabrication-audit@neel-plugins
```

Or directly:

```
/plugin install neelshah4/claude-fabrication-audit
```

---

## Worked examples

### Auditing a draft before sending

```
Audit every specific in this appeal letter before I send it
```

```
FABRICATION AUDIT — 14 specifics found

USER-SUPPLIED (5)          accepted, not re-checked
  patient age, LOS, two lab values, the denial date

VERIFIED THIS SESSION (6)
  Policy number format          ✓ matches the payer's published form
  Guideline name + year         ✓ retrieved, current edition
  Appeal window: 60 days        ✓ from the payer's own policy page
  ...

FLAGGED (3)
  "the average length of stay for this DRG is 4.2 days"
      → Retrievable, never retrieved. Plausible; unsourced.
        Verify or cut.

  "the Kansas City office handles Missouri appeals"
      → JURISDICTION CLAIM. Wrong for this address. Corrected.

  "processing typically takes 30 days"
      → Not retrievable as stated. No published figure supports
        "typically". Recommend removing the quantifier.
```

### As a silent pass

Normally you never invoke it. It runs after drafting and before the response is sent, fixing what it can and flagging what it cannot. You see nothing unless something failed.

### The pattern it is best at

```
The Denver field office serves Wyoming, and processing runs about
six weeks.
```

Two claims, both plausible, both specific, neither looked up. One is a jurisdiction claim that is either right or badly wrong with no middle ground. The other has a quantifier ("about") doing work that no source supports. Both get flagged.

---

## The failure it was built around

A jurisdiction claim — which government field office serves which region — stated confidently and wrong, with a fabricated processing time attached to it.

Both looked like facts. Both were fluent. Neither had been looked up. The person acting on that answer would have sent paperwork to the wrong office and waited on a timeline that did not exist.

The worked examples in `SKILL.md` come from that class of error: **geography and source mismatches**, where the value is correct for *somewhere* and wrong for *here*. That is the hardest fabrication to catch by reading, because nothing about it looks wrong.

---

## Scope boundary

**It does not handle academic identifiers.** PMIDs, DOIs, PMC, arXiv, NCT numbers go to [citation-verification](https://github.com/neelshah4/claude-citation-verification), which uses different lookups and different acceptance rules.

Run both. Each alone leaves half the claim surface unchecked:

| | fabrication-audit | citation-verification |
|---|---|---|
| Drug dose | ✓ | |
| Resistance rate | ✓ | |
| Guideline year | ✓ | |
| Jurisdiction, fee | ✓ | |
| PMID / DOI | | ✓ |
| Trial registry number | | ✓ |

The two compose deliberately and neither tries to cover the other's territory.

---

## Configuration

There is very little to configure, which is intentional. The skill is a procedure, not a rule table.

**Aggressiveness.** The shipped default triggers on nearly everything. If that is too much for your work, the trigger list at the top of `skills/fabrication-audit/SKILL.md` is the dial. The bias is deliberate: a superfluous check costs a lookup, a missed one costs your credibility.

**Web access is required.** Verification means retrieval. Without network access the skill degrades to flagging rather than checking, and says so.

---

## Troubleshooting

**It flagged a number I know is right.** It is telling you the number was never looked up in this session, not that it is wrong. Supply the source and it accepts it.

**It slows down every response.** It only verifies row-three claims. If it is checking constantly, your drafts are dense with unsourced specifics — which is the finding, not the bug.

**It re-checks numbers I gave it.** It should not. User-supplied values are class one and accepted. Report this if you see it.

**It missed something.** Most misses are values that do not match the "the X is Y" shape — a number embedded in a narrative clause, or a comparative claim without an explicit figure. Worth an issue with the sentence.

---

## Limitations

- **It verifies values, not arguments.** A correctly retrieved statistic used to support a claim it does not support will pass.
- **It cannot check what is not published.** Internal figures, institutional data, and unpublished results are flagged unverifiable, which is correct but not always useful.
- **Retrieval can be stale.** A guideline that changed last month may not be reflected in what the web returns.
- **It is not a fact-checker for contested claims.** It checks whether a specific value matches a retrievable source, not whether the source is right.

---

## Design notes

**Why provenance classes rather than a confidence score.** A score invites a threshold, and a threshold invites shipping everything above it. Classes force a binary: this was checked, or it was not.

**Why user-supplied values are never re-verified.** They are the user's claim, not the model's. Re-checking them is both wasteful and slightly insulting, and it dilutes attention from the class that actually matters.

**Why it runs after drafting rather than during.** Verifying sentences you are about to cut wastes lookups. Write, then audit what survived.

**Why it is aggressive by default.** The two failure modes are not symmetric. An unnecessary check costs a few seconds. A fabricated dose, jurisdiction, or fee that reaches a real reader costs considerably more.

---

## If you install more than one of these plugins

`icu-clinical-consult` and `citation-verification` each bundle a copy of this skill so they work standalone. Installing this repo alongside them gives you the canonical version, and if the copies drift, this one wins.

The marketplace repo ships `scripts/sync-fabrication-audit.sh`, which reports drift across all bundled copies and can repair it:

```bash
./scripts/sync-fabrication-audit.sh check   # report, change nothing, exit 1 on drift
./scripts/sync-fabrication-audit.sh apply   # copy canonical over the bundles
```

## Requirements

- Web access.
- Claude Code with the Agent tool for the `fabrication-auditor` subagent path; without it the audit runs inline.

## Version

`2026.8.29`, matching the skill's latest declared version date. Calendar versioning, because the skill is date-versioned. See [CHANGELOG.md](CHANGELOG.md).

## Contributing

Issues and pull requests welcome. The most useful contribution is a new claim shape it fails to catch — send the sentence.

## License

MIT. Author: Neel Shah, MD, MSc.
