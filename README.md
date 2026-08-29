# ASCII Art Converter

A lightweight, client-side web app that converts images to ASCII art — no server, no dependencies, one HTML file.

## Quick Start

1. Download `ascii-converter.html`
2. Open it in any modern browser
3. Drop an image in and hit **Konvertieren**

## Features

- **Input:** JPG, PNG, GIF, WEBP, BMP via drag & drop or file dialog
- **6 Character sets:** Detailed, Simple, Blocks (`█▓▒░`), Binary (`01`), Letters, Braille
- **3 Color modes:** Dark (green on black), Light (black on white), Color (full RGB)
- **Adjustable:** width (20–300 chars), contrast, brightness
- **Export:** copy to clipboard or download as `.txt`

## Controls

| Control | Range | Description |
|---|---|---|
| Width | 20–300 | Output width in characters |
| Contrast | 0.5–3.0 | Pivot-around-midgray contrast boost |
| Brightness | 0.3–2.0 | Linear brightness multiplier |
| Character set | 6 presets | Density of characters used |
| Mode | Dark / Light / Color | Output color scheme |

## Browser Support

Chrome 66+, Firefox 63+, Safari 13.1+, Edge 79+

## License

MIT
