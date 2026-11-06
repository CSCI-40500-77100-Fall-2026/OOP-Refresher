# Object-Oriented Programming Refresher

Lecture slides for CSCI 40500/77100 (Software Engineering), CUNY Hunter College, Fall 2026. A short refresher on the object-oriented features the Observer Pattern assignment relies on: classes and objects, references and copies, instance and static members, inheritance and constructors, overloading and overriding, polymorphism and dynamic dispatch, and abstract classes and interfaces. Examples are in Python and C++, side by side.

## Viewing

Open `oop_refresher.html` in any web browser. The deck uses [Slidy](https://www.w3.org/Talks/Tools/Slidy2/): advance with the arrow keys (or space), or by clicking; press `c` or the "Contents" button for the table of contents.

## Building from Source

The slides are written in Pandoc Markdown (`oop_refresher.md`) and built with [Pandoc](https://pandoc.org). The class diagram is a Mermaid block rendered by [mermaid-filter](https://github.com/raghur/mermaid-filter), which must be installed (`npm install --global mermaid-filter`):

```bash
make                  # build oop_refresher.html
make self-contained   # single-file HTML with resources inlined (for distribution)
```

See the `Makefile` for the exact Pandoc invocation and the other targets.

Source files:

- `oop_refresher.md`—the slide content (edit this)
- `header.html`—CSS and JavaScript injected into the document `<head>`

The `make deploy` target is for the author's own web host and relies on an ssh alias (`compsci`) defined in `~/.ssh/config`; adopters can ignore it.

## Attribution

The outline follows a Java overview by Atanas Rountev (The Ohio State University). The slides, examples, and notes were written anew, in Python and C++. The Observer class diagram redraws the UML model of the pattern from Ian Sommerville's *Software Engineering*.
