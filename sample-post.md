+++
title = "Implementing Scaled Dot-Product Attention From Scratch"
date = 2026-07-12
draft = false
description = "A from-scratch walkthrough of the attention mechanism that powers Transformers — the math, a NumPy implementation, and a PyTorch sanity check."
summary = "The attention formula looks intimidating the first time you see it. Here's what each term actually does, plus a working implementation in under 30 lines of NumPy."
tags = ["Transformers", "Attention", "PyTorch", "Deep Learning"]
categories = ["Machine Learning"]
series = ["Transformers From Scratch"]

math = true
ShowToc = true
TocOpen = false
ShowCodeCopyButtons = true
ShowReadingTime = true
+++

Every Transformer — from the original 2017 architecture to the GenAI models behind
today's chat assistants — is built on one core operation: **scaled dot-product
attention**. It looks intimidating the first time you see it written out, but
the implementation is short, and once you've written it yourself the formula
stops being magic.

This post covers three things: what the formula means term by term, a plain
NumPy implementation you can run without a GPU, and a PyTorch check to confirm
the shapes and values line up with the built-in layer.

## The formula

Given query, key, and value matrices \\(Q\\), \\(K\\), and \\(V\\), attention is defined as:

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

Breaking that down term by term:

- \\(Q\\) (queries), \\(K\\) (keys), and \\(V\\) (values) are matrices of shape
  \\((\text{sequence length}, d_k)\\) — each row is one token's vector.
- \\(QK^\top\\) computes a similarity score between every query and every key via
  dot product. The result is a \\((\text{seq\_len}, \text{seq\_len})\\) matrix of
  raw scores.
- Dividing by \\(\sqrt{d_k}\\) keeps those scores from growing too large as
  \\(d_k\\) increases — without it, softmax gradients vanish for long vectors.
- \\(\text{softmax}\\) turns each row of scores into a probability distribution
  that sums to 1, so every token ends up with a weighted average over all
  values \\(V\\) based on relevance.

That's the entire mechanism. No recurrence, no convolution — just a similarity
lookup, a scaling term, and a weighted sum.

## Implementation in NumPy

Here's the whole thing with no dependencies beyond NumPy:

```python
import numpy as np

def softmax(x, axis=-1):
    # subtract the max for numerical stability before exponentiating
    x_shifted = x - np.max(x, axis=axis, keepdims=True)
    exp_x = np.exp(x_shifted)
    return exp_x / np.sum(exp_x, axis=axis, keepdims=True)

def scaled_dot_product_attention(Q, K, V):
    d_k = Q.shape[-1]
    scores = Q @ K.transpose(-2, -1)      # (seq_len, seq_len)
    scores = scores / np.sqrt(d_k)
    weights = softmax(scores, axis=-1)    # attention weights, rows sum to 1
    output = weights @ V                  # (seq_len, d_v)
    return output, weights
```

Try it on a toy example — 4 tokens, each represented by an 8-dimensional vector:

```python
np.random.seed(0)
seq_len, d_k = 4, 8

Q = np.random.randn(seq_len, d_k)
K = np.random.randn(seq_len, d_k)
V = np.random.randn(seq_len, d_k)

output, weights = scaled_dot_product_attention(Q, K, V)

print("Output shape:", output.shape)   # (4, 8)
print("Weights row sums:", weights.sum(axis=-1))  # should all be 1.0
```

Every row of `weights` sums to 1 by construction — that's the softmax doing
its job. Token 0's output is just a weighted blend of all four value vectors,
where the weights come from how strongly token 0's query matched each key.

## Sanity check against PyTorch

It's worth confirming this matches a real framework's implementation before
trusting it inside a bigger model:

```bash
pip install torch --break-system-packages
```

```python
import torch
import torch.nn.functional as F

Q_t, K_t, V_t = torch.from_numpy(Q), torch.from_numpy(K), torch.from_numpy(V)

torch_output = F.scaled_dot_product_attention(Q_t, K_t, V_t)

print(torch.allclose(
    torch_output,
    torch.from_numpy(output),
    atol=1e-6
))  # True
```

If that prints `True`, the from-scratch version is numerically equivalent to
PyTorch's fused, GPU-optimized implementation — the only difference is speed.

## Why the scaling term matters

A quick way to see why \\(\sqrt{d_k}\\) is necessary: as \\(d_k\\) grows, the dot
products \\(QK^\top\\) grow with it (each is a sum of \\(d_k\\) terms), which
pushes softmax into a region where gradients are nearly zero.

| \\(d_k\\) | Typical \|score\| without scaling | After dividing by \\(\sqrt{d_k}\\) |
|---|---|---|
| 8 | ~3–5 | ~1–2 |
| 64 | ~15–25 | ~2–3 |
| 512 | ~60–100 | ~3–4 |

> **Note:** this is the same reason Transformer papers describe \\(d_k\\) as
> part of the architecture, not just an implementation detail — it directly
> affects training stability.

## From single-head to multi-head

In practice, models don't run attention once — they run it in parallel across
multiple "heads," each learning to attend to different kinds of relationships
(syntax, coreference, position), then concatenate the results. The function
above is the core primitive that gets called once per head; multi-head
attention is a wrapper around it, which we'll build in the next post in this
series.

## Further reading

- Vaswani et al., *Attention Is All You Need* (2017) — the original paper that
  introduced this formula.
- PyTorch's [`scaled_dot_product_attention`](https://pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention.html)
  docs, for the production-grade, fused-kernel version of what we built above.
