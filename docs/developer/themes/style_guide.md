# Style Guide

This style guide is for use in writing themes for Omeka S. It borrows heavily from the _[Github style guide](http://primercss.io/)_. This style guide is a work in progress, so excuse our dust.

### General styles

* Use 4 spaces for indentation. Do not use tabs.
* Use spaces after `:` when writing property declarations.
* Put spaces before `{` in rule declarations.
* Use hex color codes `#000` unless using rgba.

Use the `/*` style of comments for headings you want to appear in the `.css` file. For Sass-only sections, such as the variables set in `_base.scss`, use the `//` commenting syntax. The `!` at the beginning of the header helps bookmark sections for theme writers using the Coda web editor.

### _base.scss

The `_base.scss` file should include any variables used across multiple `.scss` files. Otherwise, the variables appear at the top of the file in which they are used.

### _normalize.scss

We like to use Nicolas Gallagher's [`normalize.css`](http://necolas.github.io/normalize.css/) as our reset stylesheet. Include it as a `.scss` file and import it into `_base.scss`.
