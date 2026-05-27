# Ascii Art Converter
## Documentation

## 1. Overview

The ASCII Art Converter is a fully client-side web application that transforms raster images (JPEG, PNG, GIF, WEBP, BMP) into text-based ASCII representations. All processing occurs in the browser using the HTML5 Canvas API — no server, no uploads, no dependencies.

### Feature Summary

| **Feature** | **Options** | **Description** |
| ------------- | ------------- | ------------- |
| Input formats | JPG, PNG, GIF, WEBP, BMP | Any format the browser's Image API supports |
| Upload method | Click or Drag & Drop | File input dialog or drag-and-drop zone |
| Output width | 20 – 300 characters (changable) | Adjustable via slider; height auto-calculated |
| Character sets | 6 presets | Detailed, Simple, Blocks, Binary, Letters, Braille |
| Color modes | Dark, Light, Color | Green-on-black, black-on-white, or full RGB color |
| Contrast | 0.5 – 3.0 (changable) | Multiplier applied before luminance mapping |
| Brightness | 0.3 – 2.0 (changable) | Multiplier applied before |
| Export | Copy + Download | Clipboard copy or .txt file download |
| Server needed? | No | 100% client-side, Canvas API only |

## 2. Architecture

The application is structured as a single HTML file with three logical layers:
- Presentation layer —> HTML + CSS. A two-column grid layout with a control panel on the left and output panel on the right. Dark terminal aesthetic using CSS custom properties.
- Logic layer —> Vanilla JavaScript. No frameworks or build tools. Handles file I/O, canvas operations, character mapping, and DOM updates.
- Rendering layer —> HTML5 Canvas API. Images are drawn onto a hidden off-screen canvas at the target character resolution, then pixel data is read via getImageData().

### Data Flow
User selects image\
       &emsp;↓\
FileReader → Image object → preview <img>\
       &emsp;↓\
Canvas.drawImage(img, 0, 0, cols, rows)   ← scaled to char resolution\
       &emsp;↓\
ctx.getImageData() → Uint8ClampedArray (RGBA)\
       &emsp;↓\
Per-pixel: apply contrast + brightness → compute luminance\
       &emsp;↓\
Map luminance [0–255] → charset index → character\
       &emsp;↓\
Join chars into lines → render to < pre > element

## Core Algorithm
### Image Scaling

The target canvas size is computed from the user-selected column width and the image's natural aspect ratio:
~~~
const cols = parseInt(widthSlider.value);  // 20–300
const aspectRatio = image.height / image.width;
const rows = Math.round(cols * aspectRatio * 0.45);
//                                          ^^^^
//   0.45 corrects for character aspect ratio
//   (monospace chars are ~2× taller than wide)
~~~

*NOTE: The 0.45 correction factor prevents squashed-looking output. Adjust this value if using a different monospace font — taller fonts may need a smaller factor (e.g. 0.40).*

### Contrast & Brightness
Each channel is adjusted before luminance is computed:\
// For each channel value v ∈ [0, 255]:\
v = clamp(((v / 255 - 0.5) * contrast + 0.5) * brightness * 255, 0, 255);
 
// contrast: pivots around mid-gray (0.5)\
// brightness: scales the result\
// clamp: keeps values in valid [0, 255] range

### Luminance
Perceived brightness is computed using the standard Rec. 601 luma formula:\
const lum = 0.299 * r + 0.587 * g + 0.114 * b;\
//   ^^^^ human vision is most sensitive to green

### Character Mapping
Luminance maps to a character index. Character sets are ordered from darkest (index 0 = space) to brightest (index n = densest char):\
const idx = Math.floor((lum / 255) * (charset.length - 1));\
const char = charset[idx] || ' ';

## Character Sets

| **Name** | **Characters (light→dark)** |
| ------------- | ------------- |
| Detailed | .,:;*?%S#@ |
| Simple | .-+#@ |
| Blocks | ░▒▓█ |
| Binary | 01 |
| Letters | .:abc...XYZ#@ |
| Braille | ⠁⠃⠇⠯⠷⠻⠿ |

### Adding a Custom Character Set

~~~
To add a new preset, extend the CHARSETS object in the script block:
const CHARSETS = {
  // ... existing sets ...
  myset: ' -=+*#%@'.split('').reverse(),
  //                        ^^^^^^^^
  //   .reverse() because charset[0] = lightest
};\
~~~

## Color Modes
Three rendering modes are supported:\
### Dark (default)\
White background is black; characters are rendered in CSS custom property --accent (green #00ff88). Uses textContent for maximum performance.
### Light\
Background is white; characters are black (#000). Also uses textContent. Ideal for printing.
### Color\
Each character is wrapped in a <span> with inline color: rgb(r,g,b) derived from the source pixel's adjusted channels. Uses innerHTML — slightly slower for large outputs but visually striking.

*PERFORMANCE: Color mode can be slow for outputs wider than 200 characters. Consider reducing width when using Color mode on low-powered devices.*

## JavaScript API Reference
All logic lives in a single <script> block. The key functions are:

### imageToASCII()
Reads the current image and all control values, draws the image to the off-screen canvas, processes pixel data, and returns the ASCII result object.
~~~
returns {\
  lines: string[],       // one string per row of output
  colorLines: Array<     // one entry per pixel
    { char: string, r: number, g: number, b: number }
  >[],\
  cols: number,          // output width in characters
  rows: number           // output height in characters
}
~~~

### displayResult(result)
Renders the result object to the #asciiText element. Switches between textContent (dark/light) and innerHTML (color) automatically based on currentMode.

### convert()
Thin wrapper around imageToASCII() + displayResult(). Disables the button and shows a pulse animation while processing runs on the next tick via setTimeout(..., 30).

## Color Modes
### CSS Custom Properties
All colors are defined as CSS custom properties on :root:
### loadFile(file)
Accepts a File object from either the file input or drag-and-drop handler. Creates an Image object via FileReader and stores it in currentImage.

| **Property** | **Default** | **Purpose** |
| --bg | #0a0a0a | Page background |
| --surface | #111 | Panel background |
| --accent | #00ff88 | Primary accent |
| --accent2 | #00ccff | Secondary accent |
| --text | #e0e0e0 | Body text color |
| --muted | #444 | Labels and secondary text |
| --glow | 0 0 12px #00ff8866 | Box-shadow glow effect |

### Adjusting Character Aspect Ratio
If you change the output font size (currently 6px in #asciiText), recalibrate the aspect ratio correction factor:
~~~
// In imageToASCII():
const rows = Math.round(cols * aspectRatio * 0.45);
//                                           ^^^^
//   Increase → taller output (for wider chars)
//   Decrease → shorter output (for narrower chars)
~~~

### Adding Live Preview
To convert automatically on every control change, attach the convert() function to input events:
~~~
[widthSlider, contrastSlider, brightnessSlider, charsetSelect]
  .forEach(el => el.addEventListener('input', () => {
    if (currentImage) convert();
  }));
~~~

## Known Limitations
- Very large images (>4000×4000 px) may cause the browser to allocate a large Canvas buffer. No explicit size cap is enforced; consider adding one if targeting mobile devices.
- Color mode renders one <span> per character. At 300 columns × ~100 rows = 30 000 DOM nodes, this can be slow on older browsers. A Canvas-based renderer would be faster for high-resolution color output.
- The aspect ratio correction (0.45) is calibrated for 6px Share Tech Mono. Embedding the font via @font-face or changing font-size will affect output proportions.
- The download button exports plain text (.txt). ANSI escape codes (for terminal color), HTML export, or PNG rendering are not currently implemented.
- GIF images are loaded as a static first frame only; animation is not supported. (sorry)

