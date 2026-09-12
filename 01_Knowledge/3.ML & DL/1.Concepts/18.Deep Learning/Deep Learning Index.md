---
tags: [category/ml-dl, topic/deep-learning, index, moc]
---

# Deep Learning — Index

> **Position in vault**: `3.ML & DL/1.Concepts/18.Deep Learning/`
> **Purpose**: General deep learning — neural network fundamentals, sequence models, CNNs, attention/Transformers, and the practical mechanics of training deep networks. Built 2026-09 as this vault's first genuine DL coverage; previously a folder named "ML & DL" had 0% DL content.
> **Prerequisite**: [[Optimization Index]] (gradient descent), [[Loss and Cost Index]], [[Model Behavior Index]]

## Section Map

| Subfolder | Covers |
|---|---|
| [[Perceptron\|1. Neural Network Foundations]] | Perceptron, MLP, activation functions, computational graphs, forward/backpropagation, weight initialization |
| [[Sequence Models Index\|2. Sequence Models]] | RNN, LSTM, GRU, seq2seq, vanishing/exploding gradients — relocated here from NLP (general architecture, not NLP-specific) |
| [[CNN Fundamentals\|3. Convolutional Networks]] | Convolution, pooling, typical CNN architecture |
| [[Attention Mechanism\|4. Attention & Transformers]] | Self-attention, multi-head attention, the full Transformer architecture |
| [[Batch and Layer Normalization\|5. Training Deep Networks]] | BatchNorm/LayerNorm, dropout, optimizers (Adam etc.), learning rate scheduling |
| [[Transfer Learning and Fine-Tuning\|6. Applied Deep Learning]] | Transfer learning/fine-tuning, training loops/PyTorch, inference vs. training |

## Recommended Reading Order

Neural Network Foundations → (Sequence Models *or* CNNs, either order — both build on Foundations independently) → Attention & Transformers (assumes Sequence Models' seq2seq bottleneck as motivation) → Training Deep Networks → Applied Deep Learning

**Fastest path to "understand a modern LLM"**: Neural Network Foundations (just Perceptron → MLP → Activation Functions → Backpropagation) → Sequence Models' [[Sequence-to-Sequence Models]] (for the bottleneck problem) → Attention & Transformers in full → Applied Deep Learning's [[Transfer Learning and Fine-Tuning]] → Applied AI & LLM Systems (module 19)

## Why This Structure

Modules 1–9 of `3.ML & DL` build the classical foundation (linear models, gradient descent, bias-variance, regularization) that this module reuses directly rather than re-deriving — an MLP is trained with the exact same gradient descent covered in [[Optimization Index]], just with a more complex function to differentiate. Sections 1 and 5 here are genuinely new mechanics (backpropagation, normalization, modern optimizers); sections 2–4 are genuinely new *architectures*; section 6 is the practical, applied layer connecting theory to actually training and serving a model.

## Key Cross-Links

| DL Concept | Links to |
|---|---|
| Gradient descent for a neural network | [[Gradient Descent]] — same algorithm, different (much larger) parameter space |
| Overfitting in deep networks | [[Overfitting]], [[Regularization]] — dropout and weight decay are DL's versions of the same idea |
| Tokenization, embeddings feeding into sequence/attention models | [[NLP Index]] — canonical home, linked not duplicated |
| LLM-scale application of Transformers | Applied AI & LLM Systems (module 19) |

## Common Exam / Interview Questions

1. Why can't a single perceptron learn XOR, and what fixes that?
2. Walk through backpropagation's chain-rule computation for a two-layer network.
3. Why does ReLU largely avoid the vanishing-gradient problem that sigmoid/tanh suffer from?
4. What problem does attention solve that a plain seq2seq RNN doesn't?
5. Why does a Transformer need positional encoding?
6. BatchNorm vs. LayerNorm — what's the actual difference, and why do Transformers use LayerNorm?
7. What's the difference between feature-extraction and full fine-tuning, and when would you choose each?

## One-line Summary

> Neural network fundamentals (backprop, activations, initialization) are the shared substrate every architecture here builds on; sequence models and attention/Transformers are the two major architectural lineages, training-deep-networks covers the mechanics that make deep stacks trainable at all, and applied deep learning connects it to actually running a model in practice.
