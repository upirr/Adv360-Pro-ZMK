# Comma Key Push-to-Talk Dictation Design

**Date:** 2026-02-16
**Status:** Approved

## Overview

Transform the comma key on the base layer to support push-to-talk dictation while maintaining normal comma typing functionality.

## Behavior

- **Quick tap** (<250ms): Types comma character
- **Hold** (≥250ms): Sends Win+comma to start dictation
- **Release**: Sends Win+comma again to stop dictation
- **Idle guard**: 100ms pause required before comma when typing quickly to prevent accidental activation

## Architecture

### Components

1. **Macro: dictation_toggle**
   - Sends Win+comma on activation
   - Waits for key release
   - Sends Win+comma again on release

2. **Hold-tap behavior: comma_dictate**
   - Tap binding: regular comma keypress
   - Hold binding: dictation_toggle macro
   - Timing: 250ms tapping term
   - Flavor: tap-preferred
   - Idle guard: 100ms require-prior-idle-ms

3. **Keymap integration**
   - Replace `&kp COMMA` on base layer with `&comma_dictate 0 COMMA`

## Implementation

### Macro Definition (macros.dtsi)

```c
dictation_toggle: dictation_toggle {
  compatible = "zmk,behavior-macro";
  #binding-cells = <0>;
  bindings =
    <&macro_tap &kp LG(COMMA)>,
    <&macro_pause_for_release>,
    <&macro_tap &kp LG(COMMA)>;
  label = "Dictation Toggle";
};
```

### Behavior Definition (adv360.keymap)

```c
comma_dictate: comma_dictate {
  compatible = "zmk,behavior-hold-tap";
  label = "COMMA_DICTATE";
  #binding-cells = <2>;
  tapping-term-ms = <250>;
  quick_tap_ms = <175>;
  flavor = "tap-preferred";
  bindings = <&dictation_toggle>, <&kp>;
  require-prior-idle-ms = <100>;
};
```

### Keymap Update

Replace line 55 in adv360.keymap:
- Before: `&kp COMMA`
- After: `&comma_dictate 0 COMMA`

## Design Rationale

- **250ms timing**: Matches existing homerow_layer behavior for consistency
- **Idle guard**: Prevents accidental activation during fast typing (like "hello, world")
- **Tap-preferred flavor**: Prioritizes normal comma typing for common use case
- **Macro approach**: Uses ZMK's built-in macro_pause_for_release for clean press/release handling

## Testing

1. Quick tap comma → should type ","
2. Hold comma 250ms+ → should trigger dictation start
3. Release comma → should trigger dictation stop
4. Type quickly with commas → should not accidentally activate dictation
