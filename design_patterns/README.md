# Reveal.js Code Presentation Template

A clean, professional presentation template optimized for code examples. Each slide is a separate markdown file for easy editing and organization.

## Project Structure

```
revealjs-presentation/
├── index.html          # Main presentation file
├── theme.css           # Custom theme styling
├── README.md           # This file
└── slides/             # Individual slide files
    ├── 01-title.md
    ├── 02-agenda.md
    ├── 03-javascript-example.md
    ├── 04-line-highlighting.md
    ├── 05-side-by-side.md
    ├── 06-key-takeaway.md
    ├── 07a-multi-language-typescript.md
    ├── 07b-multi-language-rust.md
    ├── 07c-multi-language-go.md
    ├── 08-terminal-commands.md
    ├── 09-configuration.md
    ├── 10-markdown-features.md
    └── 11-closing.md
```

## Quick Start

1. Serve the files with a local server (required for external markdown):
   ```bash
   # Using Python
   python -m http.server 8000
   
   # Using Node.js
   npx serve
   
   # Using PHP
   php -S localhost:8000
   ```
2. Open `http://localhost:8000` in your browser
3. Use arrow keys or swipe to navigate

## Features

- **Modular slides** - Each slide in its own markdown file
- **Left-aligned code** - Professional, readable code blocks
- **Dark theme** - Easy on the eyes, great for code
- **Syntax highlighting** - Atom One Dark theme
- **Line highlighting** - Step through code with `[1|3-5|7]`
- **Two-column layouts** - Compare before/after code
- **Fragment animations** - Reveal content step by step

## Adding a New Slide

1. Create a new `.md` file in the `slides/` directory:
   ```markdown
   ## My New Slide

   Content goes here with **markdown** support.

   ```javascript
   const code = "highlighted";
   ```
   ```

2. Add a reference in `index.html`:
   ```html
   <section data-markdown="slides/12-my-new-slide.md"></section>
   ```

## Slide Syntax

### Basic Slide
```markdown
## Slide Title

Your content here with **bold** and *italic* text.
```

### Code with Line Highlighting
```markdown
## Code Example

```javascript [1|3-4|6]
function example() {
    // Line 1 highlighted first
    const a = 1;
    const b = 2;
    // Then lines 3-4, then line 6
    return a + b;
}
```
```

### Fragment Animations
```markdown
- First item <!-- .element: class="fragment fade-up" -->
- Second item <!-- .element: class="fragment fade-up" -->
```

### Two Column Layout
```markdown
<div class="columns">
<div>

### Left Column
Content here

</div>
<div>

### Right Column
Content here

</div>
</div>
```

### Highlight Box
```markdown
> **Note:** This creates a styled highlight box.
```

### Vertical Slides
In `index.html`, wrap multiple sections:
```html
<section>
    <section data-markdown="slides/07a-typescript.md"></section>
    <section data-markdown="slides/07b-rust.md"></section>
    <section data-markdown="slides/07c-go.md"></section>
</section>
```

### Title Slide (Centered)
Add `class="title-slide"` to the section in `index.html`:
```html
<section data-markdown="slides/01-title.md" class="title-slide"></section>
```

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `→` / `Space` | Next slide |
| `←` | Previous slide |
| `↓` | Next vertical slide |
| `↑` | Previous vertical slide |
| `F` | Fullscreen |
| `O` | Overview mode |
| `S` | Speaker notes |
| `B` | Blackout |
| `ESC` | Exit fullscreen/overview |

## Customization

Edit CSS variables in `theme.css`:

```css
:root {
    --bg-color: #1a1a2e;      /* Background */
    --text-color: #eaeaea;     /* Body text */
    --heading-color: #ffffff;  /* Headings */
    --accent-color: #4fc3f7;   /* Highlights */
    --code-bg: #0d1117;        /* Code blocks */
}
```

## Supported Languages

JavaScript, TypeScript, Python, Rust, Go, Java, C++, C#, Ruby, PHP, SQL, Bash, JSON, YAML, HTML, CSS, and many more.

## Important Notes

- **Local server required**: External markdown files need to be served via HTTP due to browser security restrictions. Opening `index.html` directly won't work.
- **File naming**: Use numbered prefixes (01-, 02-) to keep slides organized.
- **Vertical slides**: Group related content using nested sections in `index.html`.

## License

MIT - Use freely for your presentations!
