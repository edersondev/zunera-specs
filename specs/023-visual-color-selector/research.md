# Research: Visual Color Selector

- **Decision**: Retain existing semantic strings and expand accepted values. **Rationale**: All three APIs already store strings, and column lengths support all 19 accepted names. **Alternative**: Migration to hex values would break old records and duplicate presentation concerns.
- **Decision**: Use existing IconPicker popover and ARIA pattern as the interaction reference. **Rationale**: It already handles dialog mounting, width measurement, focus restoration, and grid keyboard navigation. **Alternative**: Custom floating layer would add unnecessary behavior.
- **Decision**: Keep chart tokens intact; expose ten visually distinct palette tokens and map nine nonselectable aliases only in presentation. **Rationale**: Existing records retain their strings, and the expanded API accepts alternate names while the picker offers ten clear choices. **Alternative**: Raw colors in Vue components would violate the design foundation.
