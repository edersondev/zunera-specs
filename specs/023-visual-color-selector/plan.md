# Implementation Plan: Visual Color Selector

**Branch**: `023-visual-color-selector` | **Date**: 2026-10-07 | **Spec**: [spec.md](spec.md)

**Branch Coordination**: `023-visual-color-selector` is active in specs, backend, and frontend.

## Summary

Centralize the 16 semantic color values. Extend backend validation first, then replace the three text selects with one keyboard-accessible swatch palette based on the existing IconPicker popover pattern. Keep six old values and colors unchanged.

## Technical Context

**Language/Version**: PHP 8.3 / Laravel 13; JavaScript / Vue 3  
**Primary Dependencies**: Existing Element Plus, vue-i18n, Tailwind CSS  
**Storage**: Existing color string columns (24 or 32 characters); no migration  
**Testing**: PHPUnit, Vitest, Playwright, Pint, ESLint/Oxlint  
**Target Platform**: Authenticated responsive web app, pt-BR and en, dark and light themes  
**Project Type**: Full-stack feature  
**Performance Goals**: Immediate palette response  
**Constraints**: No new packages or unrelated form changes  
**Scale/Scope**: Three form fields and corresponding color presentations

## Delivery Scope and Order

1. **Backend** — A shared allowed-value definition for the three existing Form Requests and visual option services; preserve middleware, API Resources, and defaults. Add feature tests before frontend work.
2. **Frontend** — Shared `ColorPicker` receives `modelValue`, field label/name/test ID, and emits `update:modelValue`. Shared palette utility maps values to CSS tokens and translations. Integrate in three forms and existing color displays, then test.

## Constitution Check

- [x] Existing authenticated routes, Form Requests, services, and API Resources remain the API boundary.
- [x] Backend contract and tests precede frontend changes.
- [x] Matching branch exists in all three repositories.
- [x] Services retain defaults; no DTO or repository change is needed for the single color field.
- [x] Vue Composition API and Element Plus popover preserve project conventions; no new shared state or API service is needed.
- [x] Feature, unit, component, and isolated Playwright coverage is planned.
- [x] No secrets, uploads, migrations, or packages are introduced.

## Design

- Backend: One shared 16-value palette supplies `colors()` in all three visual option services. Existing six values and default behavior remain; Form Requests continue validating against `colors()`.
- Frontend: Keep `--chart-*` values for the original six. Add `--palette-*` tokens, with those six referencing the existing chart tokens and ten new curated hues in global theme CSS. One utility owns ordered values, labels, and safe token lookup; existing display helpers delegate to it.
- UI: `ColorPicker` uses `ElPopover` like `IconPicker`, full-width trigger matching `ElSelect`, a responsive 4/3-column grid with 44px targets, selected ring/check, localized tooltip, and listbox semantics. Arrow/Home/End navigation, Enter/Space selection, Escape/outside close, and focus return are required.
- i18n: Shared color labels in pt-BR and en; `rose` remains “Rosa”, `pink` is “Rosa-claro”. Update design component catalog and palette documentation.

## Project Structure

`specs/023-visual-color-selector/` holds this spec, plan, research, model, contract, quickstart, checklist, and tasks. Backend extends visual option services and tests. Frontend adds shared picker/palette, theme/i18n changes, form integration, and tests.
