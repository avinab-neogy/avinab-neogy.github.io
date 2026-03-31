---
layout: default
title: "makemore"
permalink: /makemore/
---

## makemore: a deep learning series

an ongoing project to strip away the abstractions of modern deep learning frameworks and build a character-level language model from the absolute ground up in pytorch. 

the goal is to understand the mathematical primitives of sequence prediction, starting from a pure statistical baseline and scaling up to multi-layer perceptrons, batch normalization, and beyond.

[view the full source code on github](https://github.com/avinab-neogy/makemore)

### the chapters

- [**part 1: bigrams, counting, and neural nets**](/makemore/part1/)
  implemented a statistical counting baseline and a single-layer neural network, proving both approaches converge to a negative log-likelihood of ~2.45.

- **part 2: the multi-layer perceptron (mlp)**
  *in progress* — breaking the single-character context limit by building a bengio 2003-style mlp with learned character embeddings.

- **part 3: batch normalization & activations**
  *planned* — implementing batch norm from scratch to stabilize deep neural network training.

---