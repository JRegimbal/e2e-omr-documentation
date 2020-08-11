---
layout: default
title: Symbol Classification Tutorial
parent: OMR Tutorial
nav_order: 2
---

After the model trained in [the previous phase](./document-analysis)
has been used to generate layers for a page of the manuscript, it is time to
classify the music symbols. While the text and staff lines layers are
straightforward, the music symbols layer contains various neumes, clefs,
and custodes that require additional effort to differentiate.
For this, the Interactive Classifier can be used.

# Steps Before the Interactive Classifier

The Interactive (and regular) Classifier require connected components, not
just masks extracted from the source image. These are created by the
"CC Analysis" job which takes a cleaned-up black-and-white PNG image.
Optionally, a step could be inserted between the CC Analysis job and
Interactive Classifier job where [Diagonal Neume Slicing](https://github.com/DDMAL/diagonal-neume-slicing) is run to split neumes into neume components more often, reducing the number of classes to be classified.

![Workflow until Interactive Classifier](/assets/workflow-to-ic.png)

Above is a workflow containing the jobs necessary to prepare the inputs for Interactive Classifier.

# Interactive Classification

The [Interactive Classifier](/overview/classification#interactive-classifier)
allows the glyphs from the music symbols layer to be classified as different
element types. Typically naming of these classes follows some sort of hierarchy.
For example, a C clef would be classified as `clef.c` and a punctum would be
classified as `neume.punctum`. However a custos would simply be `custos`.

The names of these classes should correspond to names in a CSV file for use in the MEI Encoding job that can be produced by the [MEI Mapping Tool](https://github.com/DDMAL/mei-mapping-tool).

In addition to the symbols that actually should be classified, there will
likely still be specks that should be ignored. Rather than attempting to
delete all of these, an alternate approach is to just classify them as
`skip` so they are detected and filtered out by the automatic classification phase(s).
Otherwise the classifier would attempt to classify them as something like a clef or neume, regardless of the confidence level.

There are two main ways of classifying neumes: a neume based approach and
a neume component based approach. The former will attempt to classify entire
neumes (e.g., `neume.clivis.3`) [as described on the Interactive Classifier wiki](https://github.com/DDMAL/Interactive-Classifier/wiki/Classifying-Square-Notes).
The latter approach uses the diagonal neume slicing job first to then only
classify neume components by the shape of the note head (e.g., `neume.inclinatum`) and handle neume grouping later.
This approach is typically better for square-notation documents as it is
possible to quickly get much more training data for the smaller set of
classes compared to the many possible neumes that can appear.

# Note on Using Biollante

The Biollante job is an optional part of this process.
It attempts to optimize the features used in the classifier based on
training data. Training data can be obtained from the Interactive Classifier
through an optional output port.

After Biollante is run (for details see [the overview page on it and links there](/overview/classification#biollante)), feature weights or selections are generated.
These can be used as inputs to the non-interactive Classifier job along with
training data and unclassified connected components.