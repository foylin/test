# Markdown Preprocessor

## Introduction

The Markdown Preprocessor is a Python module designed to add extended features
on top of the excellent Markdown syntax defined by John Gruber.  These additions
are mainly focused on creating larger technical documents without needing to use
something as heavy and syntactically complex as Docbook.

For more detail, see https://github.com/jreese/markdown-pp .

## Use !INCLUDE

Usage one, included as it is:

    !INCLUDE "include/file"

* Include as verbatim:

This comes from an **included** markdown files.

This is useful for:
1. Contents will be reused.
2. The article stays one.

* Include in a code block:

```
This comes from an **included** markdown files.

This is useful for:
1. Contents will be reused.
2. The article stays one.
```

Usage two, included with shifted section level:

    !INCLUDE "include/file", <n>

The shifted level is relative to the section level in the included file.

For example, this is the content of the included file:
```md
# Section 1 in included file

This is section one

## Section 2 in Included file

This is section two

```

The current section level is 2. In order to transform the section level to level 2, the shifted number should be `2 - 1 = 1`

    !INCLUDE "share/mdpp-demo-include-section.md", 1

### I'm section 3 in host file

## Section 1 in included file

This is section one

### Section 2 in Included file

This is section two


## Other useful stuff
 - `!IncludeURLs`: include markdown from url
 - `!TOC`: add table of contents
 - `!REF`: insert a list of links like `[name]: <url> "Title"`
 - latex rendering
 - `!VIDEO "http://www.youtube.com/embed/7aEYoP5-duY"`: youtube embedding
