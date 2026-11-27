# Changelog

All notable changes to this project will be documented in this file.

## [2026-11-27]
### Fixed
- **Sumbreaker:** Added support for multiplier tiles (`×2`, `×4`, etc.) in DOM parsing and BFS state-space search.
- **Sumbreaker:** Stabilized synthetic pointer drags by dispatching `pointermove` and `pointerup` on the initial `pointerdown` target, adding a midpoint move, and increasing the post-drop delay to `350ms` for merge animations.

## [2026-11-23]
### Changed
- Formatted all bookmarklets with readable line breaks in `README.md`.

### Added
- Initial release of LinkedIn game bookmarklets: `Queen Solver`, `Sumbreaker`, `Sudokill`, `Patchwork` (Beta), and `Zipextract` (Beta).
