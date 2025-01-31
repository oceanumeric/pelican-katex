# LaTeX Pre-rendering for Pelican

`pelican-katex` integrates LaTeX rendering directly into the pelican generation
process and eliminates the delay in displaying math you usually experience on
the web. It does so by hooking itself into docutils' reStructuredText parser as
well as the markdown package and processing the formulas with
[KaTeX](https://github.com/KaTeX/KaTeX). The generated HTML pages only contain
the finished HTML/MathML output. Therefore, you do not need to ship the KaTeX
javascript implementation with your website anymore and improve the
accessibility as well as the load time of your internet presence.

For a demo visit this [blog
post](https://martenlienen.com/blog/gaussian-processes-are-bayesian-linear-regression/).
Notice how all the formulas are just there. There is no loading and the website
does not even serve the javascript part of KaTeX.

Note, that you still need to include the KaTeX stylesheets with your website, for
example

```html
<link rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/katex/dist/katex.min.css"
      crossorigin="anonymous">
```

## Installation

First of all, you need to install `nodejs` so that `pelican-katex` can run
KaTeX. Then run `pip install pelican-katex` and add `"pelican_katex"` to the
`PlUGINS` setting in your configuration file. Finally, remove the `katex.js`
`<script>` tag from your template and enjoy a lighter website and instant
formulas.

## Syntax

```
reStructuredText
~~~~~~~~~~~~~~~~

In rst you write inline math with the usual math role (:math:`f(x)`) or
block math with

.. math::

    \int \textrm{math block}.

# markdown

In markdown you get inline math in between $ signs, like $f(x) = \sqrt{x}$.
Note, that $ only creates a math environment if it is preceded by whitespace
or at the beginning of a block and followed by some non-whitespace character.
This is necessary so that you can still write about the 5$ in your pocket.
Block math is triggered with

$$\int \textrm{math block}.$$

Math blocks can have linebreaks but no empty lines.
```

## Configuration

The plugin offers several configuration options that you can set in your
`pelicanconf.py`.

```python
# nodejs binary path or command to run KaTeX with.
# KATEX_NODEJS_BINARY = "node"

# Path to the katex file to use. This project comes with version `0.10` of
# katex but if you want to use a different one you can overwrite the path
# here. To use a katex npm installation, set this to `"katex"`.
# KATEX_PATH = "/path/to/katex.js"

# By default, this plugin will redefine reStructuredText's `math` role and
# directive. However, if you prefer to have leave the docutil's defaults
# alone, you can use this to define a `katex` role for example.
# KATEX_DIRECTIVE = "katex"

# How long to wait for the initial startup of the rendering server. You can
# increasing it but if startup takes longer than one second, something is
# probably seriously broken.
# KATEX_STARTUP_TIMEOUT = 1.0

# Time budget in seconds per call to the rendering engine. 1 second should
# be plenty since most renderings take less than 50ms.
# KATEX_RENDER_TIMEOUT = 1.0

# Define a preamble of LaTeX commands that will be prepended to any rendered
# LaTeX code.
# KATEX_PREAMBLE = None

# Here you can pass a dictionary of default options that you want to run
# KaTeX with. All possible options are listed on KaTeX's options page,
# https://katex.org/docs/options.html.
# KATEX = {
#     # Abort the build instead of coloring broken math in red
#     "throwOnError": True
# }
```

## Preamble

The `KATEX_PREAMBLE` option allows you to share definitions between all of your
math blocks across all files. It takes a string of any LaTeX commands you would
like, for example

```python
KATEX_PREAMBLE = r"""
\def\ceil#1{\lceil #1 \rceil}
\def\floor#1{\lfloor #1 \rfloor}
"""
```

If you have a large preamble, it might be nice to extract it into a `.tex` file.
Note, that pelican will not be aware of changes made to that file in autoreload
mode and you will have to restart pelican manually.

```python
from pathlib import Path
KATEX_PREAMBLE = Path("preamble.tex").read_text()
```

You can also add more definitions per file to the preamble with preamble-blocks
that do not produce any output.

```
reStructuredText
~~~~~~~~~~~~~~~~

.. math::
   :preamble:

   \def\pelican{\textrm{pelican}^2}

This definition will be available in subsequent blocks

.. math::

   \sqrt{\pelican}

or inline :math:`\pelican = 1`.

# markdown

In markdown it is not as easy to define properties of blocks, so we chose to
start a preamble block with an @ such as

$$@
\def\pelican{\textrm{pelican}^2}
$$

which works just the same in blocks

$$\sqrt{\pelican}$$

or inline $\pelican = 1$.
```
