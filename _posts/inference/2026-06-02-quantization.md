---
layout: slide
title: "Inference: Quantization and Attention workflow"
date: 2026-06-02
category: inference
author:
 - Stephen Oni
excerpt: "A Brief introduction to Quantization and Attention workflow for Inference"
---

## Inference: Quantization and Attention workflow

A Brief introduction to Quantization and Attention workflow for Inference

---
<!-- use image representation -->
We have a data we want to transmit over a channel or we want to store it in a memory/storage, but we are limited interms of memory capactiy or transmission bandwidth , hence, we want to reduce the cost of storing or transmitting the data, but we want to do this without losing much information,

What can we do? yes we Quantize it.

---

### What can we do?

<!-- add list of option , like increase storage or network bandwidth -->
Quantize it... we can find a way to represent the large values in a smaller format, or better still, finding a way to represent the  distribution in a lower bit format.

<!-- Add an image showing somthing larger being represented with something smaller -->
<!-- Every information in a digital system is represented in bits, each data types has a specific number of bits allocated to it, for example, an integer might be represented using 32 bits, while a floating-point number might use 64 bits. -->

---

### What can happen?

1. Cost reduced
2. Faster computation
3. Less memory usage

However, we might lose some information in the process, and this can lead to a decrease in the quality of our data.

---

<!-- two-column -->

### What you can't measure , can't be controlled...

Shannon's Rate-Distortion theory provides a theoretical framework for understanding the trade-off between the amount of compression (or quantization) and the resulting distortion (or loss of information). It helps us to determine the optimal way to quantize data while minimizing the loss of information.

$$
R(D) = \min_{P_{\hat{X}\mid X}:\,\mathbb{E}[d(X,\hat{X})] \le D} I(X;\hat{X})
$$

Simplified:

$$
R(D) = \min I(X;\hat{X}) \quad \text{subject to} \quad \mathbb{E}[d(X,\hat{X})] \le D
$$

<!-- column -->
Where:
- $R(D)$ is the rate-distortion function, which represents the minimum number of bits required to encode the data with a distortion level of $D$.
- $I(X;\hat{X})$ is the mutual information between the original data $X$ and its quantized reconstruction $\hat{X}$, which measures how much information is preserved during the quantization process.
- $P_{\hat{X}\mid X}$ is the conditional distribution that defines how each original value $X$ is represented as a reconstruction $\hat{X}$.
- $\mathbb{E}[d(X,\hat{X})]$ is the expected distortion between the original data $X$ and its quantized reconstruction $\hat{X}$, which quantifies the loss of information due to quantization.
- $D$ is the distortion level, which represents the maximum acceptable loss of information.

---
<!-- two-column -->
### Simplified

R(D) measures how do we represent lage Data X with a smaller representation $\hat{X}$ with respect to a quality criteria d(X, $\hat{X}$) inline with an acceptable distortion level D.

<!-- column -->

![Quantization illustrated using suitcases](/assets/images/inference/quantization-suitcase.png)

---

# Distortion Function

<!-- two-column -->
The distortion function $d(X,\hat{X})$ is a measure of the difference between the original data $X$ and its quantized reconstruction $\hat{X}$. It quantifies the loss of information due to quantization. Common distortion functions  and problem space include:
- discrete data: Hamming distance - $d(X,\hat{X}) = \sum_{i=1}^{n} \mathbf{1}_{X_i \neq \hat{X}_i}$
- continuous data: Mean Squared Error (MSE) - $d(X,\hat{X}) = \frac{1}{n} \sum_{i=1}^{n} (X_i - \hat{X}_i)^2$
- perceptual data: Structural Similarity Index (SSIM) - $d(X,\hat{X}) = 1 - \text{SSIM}(X,\hat{X})$

<!-- column -->
The choice of distortion function depends on the specific application and the type of data being quantized. For example, in image compression, perceptual distortion functions like SSIM are often used to better capture human visual perception, while in audio compression, MSE might be more appropriate for measuring the fidelity of the reconstructed signal.

---

### Example: Guassian Source with MSE Distortion
<!-- two-column -->
$$
R(D) = \frac{1}{2}\log_2\left(\frac{\sigma^2}{D}\right),
\qquad 0 < D \le \sigma^2
$$

$$
D(R) = \sigma^2 2^{-2R}
$$

concrete:

Suppose we have a Gaussian source with variance $\sigma^2 = 1$ and we want to quantize it with a distortion level of $D = 0.1$. We can calculate the rate-distortion function as follows:

$$
R(D) = \frac{1}{2} \log_2 \left( \frac{1}{0.1} \right) = \frac{1}{2} \log_2(10) \approx 1.66 \text{ bits per sample}
$$

<!-- column -->
if we want to achieve a specific rate of 2 bits per sample, we can calculate the corresponding distortion level using the inverse function $D(R)$:

$$
D(R) = 1 \cdot 2^{-2 \cdot 2} = 1 \cdot 2^{-4} = 0.0625
$$

The higher the bit used for representation, the lower the distortion, and the closer it is to the original, hence no compression. On the other hand, the lower the bit used for representation, the higher the distortion, and the farther it is from the original, hence more compression.


---

<!-- two-column -->
### Quantization Design Checklist: Basic Heuristics

**1. Define the problem space**

- Is the data scalar, vector, or structured?
- Are the dimensions independent or correlated?

**2. Define the distortion function**

- What does "error" mean for this task?
- Images: MSE or perceptual loss
- Bits: Hamming distance
- Embeddings: cosine distance

**3. Characterize the distribution**

- Is it Gaussian-like, heavy-tailed, multimodal, or learned?
- The distribution tells us where useful representations should be placed.


<!-- column -->

![Problem type, distortion, and data distribution](/assets/images/inference/quantization-checklist-foundations.png)

---

<!-- two-column -->
### Quantization Design Checklist: Choose a Structure

**4. Choose the representation**

- **Scalar quantization:** simple and fast; treats dimensions independently.
- **Vector quantization:** uses a codebook and captures correlations.
- **Transform + quantization:** decorrelates the data before quantizing it.

$$
\hat{x} \in \mathcal{C} = \{c_1,c_2,\ldots,c_K\}
$$

**5. Check for a closed-form solution**

- Gaussian source + MSE: a theoretical closed form is available.
- Most real-world problems: use numerical or iterative methods.

> If dimensions are correlated, consider a transform or vector quantization.

<!-- column -->

![Scalar, vector, and transform quantization](/assets/images/inference/quantization-checklist-structure.png)

---

<!-- two-column -->
### Quantization Design Checklist: Optimize and Control Rate

**6. Choose an optimization method**

- Scalar + MSE: Lloyd-Max
- Vector quantization: k-means or LBG
- Learned systems: gradient-based optimization

**7. Design the codebook**

- Choose the representative values and assignment regions.
- With a fixed-length code, $K = 2^R$ codewords are available.

**8. Control the rate**

$$
\theta^* = \underset{\theta}{\arg\min}\;
\mathbb{E}[d(X,\hat{X}_{\theta})] + \lambda R(\theta)
$$

- Decide between fixed bits and entropy coding.
- Use $\lambda$ to balance reconstruction quality against bitrate.

<!-- column -->

![Iterative codebook optimization and rate control](/assets/images/inference/quantization-checklist-optimization.png)

---

<!-- two-column -->
### Quantization Design Checklist: Evaluate

**9. Measure the result**

- What distortion was achieved?
- What bitrate was used?
- How far is the result from the theoretical rate-distortion bound?
- Does the chosen distortion metric reflect real task quality?

The entire design problem can be summarized as:

> Approximate a continuous space using a finite set of representations.

- **What counts as close?** The distortion function.
- **Where does the data live?** The distribution.
- **How is the space divided?** The quantizer and codebook.
- **How many points can we afford?** The available bits.

<!-- column -->

![Rate-distortion evaluation and finite-space approximation](/assets/images/inference/quantization-checklist-evaluation.png)

---

<!-- two-column -->
### Simple Quantization: Asymmetric Min-Max Quantization

Every data type are represented in bits, and each have a specific number of bits allocated to it.
e.g
Every data type is represented using a specific number of bits. For example:

int8 = 8 bits
float32 = 32 bits
- `float32` uses 32 bits per value.
- `int8` uses 8 bits per value and has a range of $-128$ to $127$.

**Basic steps**

```text
q_min = -128
q_max = 127
minimum = min(values)
maximum = max(values)

scale = (maximum - minimum) / (q_max - q_min) // can also be  expressed as (Max - Min) / (2^bits - 1)
zero_point = round( q_min - minimum / scale )

for each value:
    quantized = round(value / scale) + zero_point
    quantized = clip( quantized, q_min, q_max )
    recovered = scale * (quantized - zero_point)
```

<!-- column -->

**Example**

```text
values = [-1.0, 0.0, 1.0, 3.0]

minimum = -1.0
maximum = 3.0

scale = (3.0 - (-1.0)) / (127 - (-128))
      = 4 / 255
      = 0.015686

zero_point = round(-128 - (-1.0 / scale))
           = -64

quantized = [-128, -64, 0, 127]

recovered = [-1.0039,0.0, 1.0039, 2.9961]
```

The recovered values are approximate because quantization rounds floats to
integers.

---

<!-- two-column -->
### Simple Quantization: Symmetric Linear Quantization

Symmetric quantization represents an equal float range on both sides of zero.
It uses `-127` to `127`, leaving `-128` unused so that the range is symmetric.

**Basic steps**

```text
q_min = -127
q_max = 127

absolute_maximum = max(abs(values))

scale = absolute_maximum / q_max

for each value:
    quantized = round(value / scale)
    quantized = clip( quantized, q_min, q_max)
    recovered = quantized * scale
```

There is no zero point because float zero maps directly to integer zero.

<!-- column -->

**Example**

```text
values = [-1.0, 0.0, 1.0, 3.0]

absolute_maximum = 3.0

scale = 3.0 / 127
      = 0.023622

quantized = [-42, 0, 42, 127]

recovered = [-0.9921, 0.0, 0.9921, 3.0 ]
```

This method represents `-3.0` to `3.0`, although the original minimum is only
`-1.0`. Some negative integer values are therefore unused.

---

## Quantization in Inference

- Model Quantization: Reducing the precision of model parameters (weights and activations) to lower bit-widths (e.g., int8, float16) to reduce memory usage and increase inference speed.

- KV Cache Quantization: Quantizing the key-value pairs stored in the attention mechanism of transformer models to reduce memory footprint and speed up attention computations.


## Model Quantization

Based on our implementation, we will focus on:

- GTPQ
- AWQ

---

### Model Quantization: GPTQ
