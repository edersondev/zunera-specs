# Research: Visual Color Selector

- **Decision**: Retain existing semantic strings and expand accepted values. **Rationale**: All three APIs already store strings, and column lengths support all 16 names. **Alternative**: Migration to hex values would break old records and duplicate presentation concerns.
- **Decision**: Use existing IconPicker popover and ARIA pattern as the interaction reference. **Rationale**: It already handles dialog mounting, width measurement, focus restoration, and grid keyboard navigation. **Alternative**: Custom floating layer would add unnecessary behavior.
- **Decision**: Reuse six chart hues through palette aliases and define ten new palette tokens globally. **Rationale**: Old colors remain exact while new options stay theme-aware. **Alternative**: Raw colors in Vue components would violate the design foundation.
