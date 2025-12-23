# Retro Hacker Theme

A retro/hacker aesthetic theme for the WiFi Pineapple Pager, inspired by classic 1980s terminal interfaces and hacker culture.

## Description

This theme brings a nostalgic CRT terminal aesthetic to your Pineapple Pager with:
- Classic terminal green (#00FF00) on black color scheme
- Monospace font throughout the interface
- CRT scanline effects for authenticity
- Glowing text and borders reminiscent of phosphor displays
- Sharp, angular design elements with no rounded corners
- High contrast for excellent readability in various lighting conditions

## Author

Created by Claude

## Version

1.0.0

## Tested Firmware

This theme was developed for WiFi Pineapple Pager firmware version 1.0.4

## Features

### Visual Elements
- **Terminal Green Aesthetic**: Primary color scheme uses the iconic `#00FF00` green
- **CRT Effects**: Optional scanline overlay and text glow effects
- **Monospace Typography**: All text uses monospace fonts for that authentic terminal feel
- **Zero Border Radius**: Sharp, angular design with no rounded corners
- **Glowing Elements**: Text shadows and box glows simulate CRT phosphor glow

### Component Styling
- **Buttons**: Transparent with green borders, fills on hover
- **Panels**: Dark background with glowing green borders
- **Tables**: Green headers with alternating row highlights
- **Inputs**: Dark fields with green focus states
- **Status Indicators**: Color-coded (green=online, red=error, amber=warning, cyan=scanning)
- **Progress Bars**: Glowing animated fills
- **Modals**: Prominent borders with dark overlay

### Special Effects
- Scanline animation overlay (optional)
- Text glow effects
- Pulsing progress indicators
- Smooth transitions
- Hover state animations

## Installation

### Method 1: Manual Installation
1. Clone this repository or download the theme files
2. Copy the `retro-hacker` directory to your Pineapple Pager themes folder
3. Access the Pager settings
4. Select "Retro Hacker" from the available themes
5. Apply and enjoy!

### Method 2: Direct Upload
1. Navigate to your Pager's web interface
2. Go to Settings > Themes
3. Upload the `theme.json` file
4. Activate the theme

## Directory Structure

```
retro-hacker/
├── theme.json          # Main theme configuration
├── README.md           # This file
└── assets/            # (Optional) Additional assets like custom icons
```

## Customization

You can customize this theme by editing the `theme.json` file:

- **Colors**: Modify the color values in the `colors` object to change the palette
- **Typography**: Adjust font sizes and spacing in the `typography` section
- **Effects**: Enable/disable CRT effects in the `effects` section
- **Component Styles**: Fine-tune individual component appearances in the `components` section

### Popular Customization Ideas

**Amber Terminal**: Change primary colors to `#FFB000` for a classic amber terminal look
**Green Phosphor Variants**: Try `#33FF33` for a brighter green or `#00AA00` for a darker variant
**Blue Terminal**: Use `#00FFFF` (cyan) for a blue phosphor aesthetic

## Compatibility

- Developed for firmware 1.0.4
- Should work with minor variations on other 1.x firmware versions
- Test thoroughly before deploying in production environments

## Known Limitations

- CRT scanline effects may impact performance on older hardware (can be disabled)
- Monospace fonts may make some text slightly longer than proportional fonts
- High contrast may be too bright in extremely dark environments

## Performance Notes

This theme is optimized for performance:
- Minimal use of complex shadows
- Efficient animation keyframes
- Optimized color calculations
- Lightweight asset requirements

## Accessibility

While this theme prioritizes aesthetics, it maintains good accessibility:
- High contrast ratios for text readability
- Clear visual hierarchy
- Distinct button states
- Color-coded status indicators with shapes for colorblind users (where applicable)

## Screenshots

*Coming soon - screenshots will be added after testing on actual hardware*

## Contributing

Found a bug or want to suggest an improvement? Feel free to submit issues or pull requests to the main Pineapple Pager themes repository.

## Credits

Inspired by:
- 1980s CRT terminals and monitors
- Classic hacker aesthetic from films like WarGames
- Vintage computing and BBS culture
- The original green phosphor displays

## License

This theme is provided as-is for educational and customization purposes. Subject to the Hak5 Software License Agreement: https://hak5.org/license

## Disclaimer

Use at your own risk. While this theme has been developed with care, it is provided without warranty. Test thoroughly in your environment before relying on it for critical operations.

---

**WiFi Pineapple Pager** by Hak5 - [Shop](https://hak5.org/products/wifi-pineapple-pager) | [Documentation](https://docs.hak5.org) | [Discord](https://hak5.org/discord)