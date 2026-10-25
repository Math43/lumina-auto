# Lumina — a glass theme for Home Assistant

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://github.com/hacs/integration)

Lumina is a dark glass theme for Home Assistant dashboards built with [Bubble Card](https://github.com/Clooos/Bubble-Card). Cards are frosted glass on a soft, colourful background. Cards can glow in their device's colour, and actions that would do nothing right now are greyed out.

<!-- Screenshot: add docs/screenshot.png (no personal details) and link it here -->

## What you get

- **Glass cards**: every Bubble Card button, slider and dropdown gets the frosted look, with no per-card setup.
- **Glass panels**: group a room's cards in a glass panel, and devices inside it in smaller inner panels.
- **Glow**: a card glows in a colour you choose, e.g. a light in its own colour or a switch green or red.
- **Greyed out**: dims a card or a single button when its action would do nothing, e.g. volume while the TV is off.
- **Section titles and dividers**: quiet uppercase titles with a fading line, and a centred divider between rows.
- **A colour palette** for glows and accents.
- **Glass dropdown menus**.

## Requirements

| | Tested with |
|---|---|
| Home Assistant | 2026.10 |
| [Bubble Card](https://github.com/Clooos/Bubble-Card) | 3.4.2 |
| [card-mod](https://github.com/thomasloven/lovelace-card-mod) | 4.2.1 |
| [stack-in-card](https://github.com/custom-cards/stack-in-card) (for glass panels) | 0.2.0 |

All three cards are available in HACS.

## Installation

### HACS (recommended)

1. Open HACS → ⋮ → **Custom repositories**.
2. Add `https://github.com/kjanko/lumina-homeassistant-theme` with category **Theme**.
3. Search for "Lumina" and download it.

### Manual

Copy `themes/lumina.yaml` into the `themes` folder in your Home Assistant config directory.

### Turn it on

1. Make sure Home Assistant loads themes from that folder. In `configuration.yaml`:
   ```yaml
   frontend:
     themes: !include_dir_merge_named themes
   ```
2. Restart Home Assistant, or call the `frontend.reload_themes` action.
3. Open your profile, set **Theme** to **Lumina** and **Mode** to **Dark**. Lumina is a dark design.

## Building blocks

The theme styles every Bubble Card on its own. The pieces below need a line or two on the card. [`examples/dashboard.yaml`](examples/dashboard.yaml) uses all of them.

### Glass panels

Put a room's cards in a `stack-in-card` with the `glass` class. For a device inside a room, nest another `stack-in-card` with `glass-inner`.

```yaml
type: custom:stack-in-card
card_mod:
  class: glass
cards:
  - type: custom:bubble-card
    card_type: separator
    name: Living Room
    icon: mdi:sofa-outline
  - type: custom:stack-in-card
    card_mod:
      class: glass-inner
    cards:
      - ...
```

### Divider

A Bubble separator with no name, with the `divider` class:

```yaml
type: custom:bubble-card
card_type: separator
card_mod:
  class: divider
```

### Glow

A card opts in from its Bubble `styles` by adding the `lumina-glow` class and setting two variables:

- `--glow`: the colour, as `r,g,b` (e.g. `255,180,50`) or a palette variable (e.g. `var(--lumina-green)`).
- `--glow-on`: the strength. `0` is off and looks like a plain card, `1` is the standard glow, and above `1` is stronger.

Optional: `--glow-dim` sets a second colour for the gradient, and `--glow-icon` sets an icon colour for glows too dark to use as one.

A light that glows in its own colour while it's on:

```yaml
styles: |
  ${(() => {
    card.classList.add('lumina-glow');
    const [r, g, b] = hass.states[entity]?.attributes?.rgb_color || [255, 180, 50];
    const on = state === 'on';
    return `.bubble-button-card-container {
      --glow: ${on ? `${r},${g},${b}` : '255,255,255'};
      --glow-on: ${on ? 1 : 0};
    }`;
  })()}
```

A switch that is green when on and red when off:

```yaml
styles: |
  ${(() => { card.classList.add('lumina-glow'); return ''; })()}
  .bubble-button-card-container {
    --glow: ${state === 'on' ? 'var(--lumina-green)' : 'var(--lumina-red)'};
    --glow-on: ${state === 'on' ? 0.65 : 0.4};
  }
```

### Greyed out

Add `lumina-disabled` to a card, or to one of its buttons, while its action would do nothing. It is dimmed, grey and can't be tapped.

```yaml
styles: |
  ${(() => {
    const off = ['off', 'unavailable', 'unknown'].includes(hass.states['media_player.tv']?.state);
    card.classList.toggle('lumina-disabled', off); // the whole card
    card.querySelector('.bubble-sub-button-1')?.classList.toggle('lumina-disabled', off); // or one button
    return '';
  })()}
```

### Palette

Each colour is an `r,g,b` triplet, so it works as a `--glow` value and in `rgba(var(--lumina-green), 0.3)`.

| Variable | Colour |
|---|---|
| `--lumina-amber` | 255,180,50 |
| `--lumina-green` | 80,200,140 |
| `--lumina-teal` | 45,212,191 |
| `--lumina-cyan` | 56,189,248 |
| `--lumina-blue` | 100,180,255 |
| `--lumina-indigo` | 99,102,241 |
| `--lumina-red` | 220,80,80 |
| `--lumina-magenta` | 226,0,116 |

## Customising

All sizes and colours are variables at the top of `themes/lumina.yaml`: radii, card and icon backgrounds, border, shadows, text colours, blur, the glass panel look, the separator line, the divider and the background. Change them there, or make a copy of the theme under a new name.

The background is `lovelace-background`. A dashboard view with its own `background` replaces it.

## Good to know

- **Lumina styles Bubble Card.** Other cards (weather, media players, entities) keep Home Assistant's dark look. To match them, give them their own `card_mod` style using the variables, e.g. `background: var(--lumina-glass-background)`.
- **Card styles and the theme.** The glow, greyed-out and glass rules use `!important` so they win over Bubble's inline colours. A card's own rule that should beat them needs a more specific selector, e.g. `ha-card .bubble-separator .bubble-line` instead of `.bubble-line`.
- **Cards that appear later.** Cards created after the page loads, such as inside a `conditional` card, may not pick up theme styles until the page is reloaded. To show and hide rows, keep the cards on the page and hide them from their own styles instead:
  ```yaml
  styles: |
    ${hass.states['input_boolean.show_more']?.state === 'on' ? '' : ':host { display: none !important; }'}
  ```

## License

[MIT](LICENSE)
