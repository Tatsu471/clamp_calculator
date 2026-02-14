# Clamp Calculator v3.1 🎨

> Advanced CSS `clamp()` calculation tool with SCSS mixin support and real-time previewer.

**🌟 Features:**
- Auto-scaling typography (Mobile 375px→768px, Desktop 768px→1440px)
- CSS `clamp()` + SCSS `@include txt()` mixin generation
- Real-time responsive preview with resizable container
- VW (Viewport Width) converter with context-aware logic
- Reverse calculator to analyze existing `clamp()` expressions
- Base font size customization (62.5% technique support)
- Modern glassmorphism UI with dark theme

## Quick Start

1. **Download & Open**: Simply open `clamp-calculator.html` in your browser
2. **No Installation Needed**: Vanilla HTML/CSS/JS (zero dependencies)
3. **No Data Collection**: All operations are performed locally

## Usage

### Mobile Card
- Set minimum (375px) and maximum (768px) font sizes
- Toggle "Auto-Scale" to auto-calculate based on 2.048× ratio
- Customize mixin options (weight, color, alignment)

### Desktop Card
- Set minimum (768px) and maximum (1440px) font sizes  
- Drag the resize handle to preview responsive behavior
- Width updates in real-time as you resize

### VW Converter
- Convert pixel values to viewport width (vw) units
- Automatically selects appropriate `toVw()` / `toVw2()` function

### Reverse Tool
- Paste existing `clamp()` code to analyze its parameters
- Understand min/max sizes and breakpoint widths at a glance

## ⚙️ Base Font Size Setting

Default is 16px. Adjust if your project uses:
- `62.5% technique`: Keep at 16px (root is 10px)
- Rem-based design: Match your project's root font-size

## 🔒 Security & Privacy

**Data Handling:**
- ✅ All calculations performed **locally in your browser**
- ✅ No network requests or external APIs
- ✅ No cookies or local storage usage
- ✅ Safe for sensitive project values

**Safe for:**
- Local development
- Intranet/team use
- Self-hosted deployment

**Not recommended for:**
- Public internet (unless HTTPS + CSP configured)
- Untrusted user input environments

## 📁 Project Structure

```
clamp_calculator/
├── clamp-calculator.html    # Main application (single file)
├── README.md                # This file
├── LICENSE                  # MIT License
└── SECURITY.md              # Security guidelines
```

## 🛠 Technical Stack

- **Language**: Vanilla HTML5 + CSS3 + JavaScript (ES6+)
- **Build Process**: None required
- **Browser Support**: All modern browsers (ES6+)
- **Bundle Size**: ~50KB (single HTML file)

## 📄 License

MIT License - See [LICENSE](./LICENSE) for details

## 📞 Support

For issues or suggestions:
1. Check [SECURITY.md](./SECURITY.md) for security-related questions
2. Review the in-app documentation ("使い方" card)
3. Examine HTML source code (well-commented)

## 🎯 Design Credit

- Glassmorphism UI pattern
- Responsive grid layout
- SCSS mixin framework integration (requires `_mixin.scss` + `_function.scss` in your project)

---

**Version**: 3.1 | **Updated**: 2026-02-15
