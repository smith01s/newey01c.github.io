---
title: Rendering and Saving Decision Trees in Jupyter Notebook
layout: post
---

This is a short memo for myself, really - but it may be useful to someone else.
I've found rendering decision trees in Jupyter Notebook using the functions
that they provide [in their documentation][skl-dt] annoying. Basically, they
use the `graphviz` library to render the file as an SVG, and then display it in
the notebook. Unfortunately, SVG is a vector format and for some reason,
Jupyter Notebook doesn't scale these images nicely - meaning that I kept
finding myself scrolling through *huge* decision tree figures just because they
were scaled badly.

<!-- more -->

I came up with a workaround; render the tree as a PNG file and display the
image in the notebook directly - and then if a nice SVG copy is needed for
later (for example, rendering on a blog post), render and save that separately.
Here's what I managed to come up with. For an example, if you want to save it
as an SVG file, simply call it like so: `show(the_tree, filename="out.svg")`.
Note that this transparently renders any other format supported by `graphviz`
too - PDF, PNG, etc. All you need to do is change the file extension.

```python
from IPython.display import Image
from sklearn.tree import export_graphviz
import graphviz

def show(decision_tree, class_names=None, feature_names=None, filename=None):
    dot_data = export_graphviz(
        decision_tree,
        class_names=class_names,
        feature_names=feature_names,
        out_file=None
    )
    g = graphviz.Source(dot_data)
    if fn:
        (fn, fmt) = filename.split(".")
        g.format = fmt
        g.render(filename=fn, cleanup=True)
    g.format = "png"
    return Image(data=g.pipe())
```

[skl-dt]: http://scikit-learn.org/stable/modules/tree.html#classification
