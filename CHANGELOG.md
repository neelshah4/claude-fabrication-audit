# Changelog

All notable changes to this plugin are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versioning is calendar-based (`YYYY.M.D`), matching the skill's own dated version
string rather than imposing a semantic version it does not have.

## [2026.9.24] - 2026-09-24

### Added
- Optional web fallback: when WebFetch or WebSearch is blocked (4xx/5xx, paywall, bot wall, unfollowable redirect, or content missing the target), retry once with the Parallel Search MCP if it is installed. Parallel excerpts are partial, so a value missing from one is unverified. No PHI, unpublished text, or credentials in a query; never used to pass a login or CAPTCHA gate.
- `fabrication-auditor` agent granted the two Parallel tools, with the same fallback section.

## [2026.9.8] - 2026-09-08

### Changed
- SKILL.md body trimmed from 3,257 to 1,744 words: version notes, the origin-incidents narrative, the anti-patterns worked-example table, and the case log moved to a new `skills/fabrication-audit/references/history.md`, with a one-line pointer left in SKILL.md. No trigger phrase, Step 1-5 procedure, scope boundary, or output contract changed.
- The feedback-loop section has its body rewritten as a generic issue-driven prompt with no internal script or path names.
- Every internal user-machine path reference removed from SKILL.md and references.

## [2026.8.29] - 2026-08-31

### Changed
- Version realigned to match the skill's own declared version (2026-08-29). The initial
  publication used a placeholder 1.0.0.
- README expanded substantially: worked examples with sample output, configuration,
  troubleshooting, limitations, and design rationale.
- Added the provenance classification table, worked audit output, the scope-boundary comparison against citation-verification, and the maintainer sync instructions.


## [1.0.0] - 2026-08-31

### Added
- Initial public release as a Claude Code plugin.
- MIT license, plugin manifest, and installable marketplace entry.
- Declared the canonical copy; `icu-clinical-consult` and `citation-verification`
  bundle this skill as a dependency.

### Changed
- Worked examples in the jurisdiction section use neutral placeholder locations.
2026-09-03: Sync from canonical: Versions entry for the 2026-08-29 feedback-loop block reworded; no procedure change.
