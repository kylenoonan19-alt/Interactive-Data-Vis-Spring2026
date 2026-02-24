---
title: "Intro to Observable Framework"
toc: true
---

# Intro to Observable Framework

---

### What is Observable Framework?
**Observable Framework** is an open-source **static-site generator** specifically designed for building high-performance data apps, dashboards, and reports. Unlike traditional notebooks, it allows you to develop locally on your own machine using your preferred text editor and version control systems like Git.

At its core, the Framework is three things in one:
*   **A local development server:** Used to preview your apps with instant "hot reloading" updates as you save changes.
*   **A static site generator:** Compiles Markdown, JavaScript, and other assets into a static site that can be hosted anywhere.
*   **A command-line interface (CLI):** Automates the process of building and deploying your projects.

### Project Structure
An Observable Framework project is a collection of files that work together to create your site. A typical project includes:
*   **`index.md`**: The home page of your application.
*   **Additional `.md` pages**: Other pages in your site, written in Markdown.
*   **Data loaders**: Scripts (written in Python, R, JavaScript, etc.) that fetch and transform data into snapshots.
*   **Static assets**: Files like `.csv`, `.json`, `.png`, or `.css`.
*   **`observablehq.config.js`**: An app configuration file.

### Enhanced Markdown
Observable Framework uses an extended version of **Markdown** as its primary authoring language. While standard Markdown is used for formatted text, Framework adds built-in tools for sophisticated layouts and interactivity:

*   **HTML**: You can write **HTML directly into Markdown** for precise control over your layout, such as adding custom containers or external stylesheets.
*   **Grids and Cards**: Built-in CSS classes like `.grid` and `.card` allow you to easily create responsive "bento box" layouts for dashboards.
*   **Notes**: Special classes (like `.note`, `.tip`, `.warning`) are available to insert labeled callouts into your prose.

### JavaScript and Reactivity
One of the most powerful features of Observable Framework is how it handles **JavaScript** within Markdown files. JavaScript can be included in two ways:
1.  **Fenced code blocks**: Use ` ```js ` blocks for top-level declarations like loading data or defining helper functions.
2.  **Inline expressions**: Use `${...}` to interpolate JavaScript values directly into your text or HTML.

**Reactivity** is the engine that drives these pages. The framework runs like a **spreadsheet**: when a variable is updated, any code that references that variable automatically re-runs. Because of this reactive nature, code runs independent of its order on the page, giving you complete flexibility in how you arrange your content.

### Interactive Inputs
To make data apps truly interactive, Framework provides **Inputs**—graphical UI elements like **dropdowns, sliders, radio buttons, and text boxes**. 

Inputs are typically displayed using the built-in `view` function. Because they are tied into the reactive system, choosing a value from an input (like filtering a table by selecting a category from a dropdown) will automatically trigger updates in every other part of the page that depends on that selection.

Next: [Intro to Observable Plot](./4_intro_to_observable_plot.md).
