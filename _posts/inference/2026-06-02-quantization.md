---
layout: slide
title: "Inference: Quantization and Attention workflow"
date: 2026-06-02
category: inference
author:
 - Stephen Oni
excerpt: "A Brief introduction to Quantization and Attention workflow for Inference"
---

# Inference: Quantization and Attention workflow

A Brief introduction to Quantization and Attention workflow for Inference

---
![An overloaded truck representing limited transmission capacity](/assets/images/inference/overloaded-truck-bw.jpg)

*Photo: [strudelt](https://commons.wikimedia.org/wiki/File:Truck_in_India_-_overloaded.jpg), licensed under [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/).*

We have a data we want to transmit over a channel or we want to store it in a memory/storage, but we are limited interms of memory capactiy or transmission bandwidth , hence, we want to reduce the cost of storing or transmitting the data, but we want to do this without losing much information,

What can we do?

---

## What can we do?

1. Increase the capacity ; memory capacity or network bandwidth
2. Chunk the data into different size for easy movement or storing in multiple memory
3. `Quantize it`; Compact/Compress the data by reducing the size and not leaving anything behind

In this session, we'll take a look at the `3rd` option


---

## What can happen if we quantize?

1. Cost reduced
2. Faster computation
3. Less memory usage

However, we might lose some information in the process, and this can lead to a decrease in the quality of our data.

---

<!-- two-column -->

## What you can't measure , can't be controlled...

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
## Simplified

R(D) measures how do we represent lage Data X with a smaller representation $\hat{X}$ with respect to a quality criteria d(X, $\hat{X}$) inline with an acceptable distortion level D.

<!-- column -->

![Quantization illustrated using suitcases](/assets/images/inference/quantization-suitcase.png)

---

## Distortion Function

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
## Quantization Design Checklist: Basic Heuristics

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

![Problem type, distortion, and data distribution](/assets/images/inference/quantization-checklist-foundations.svg)

---

<!-- two-column -->
## Quantization Design Checklist: Choose a Structure

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

![Scalar, vector, and transform quantization](/assets/images/inference/quantization-checklist-structure.svg)

---

<!-- two-column -->
## Quantization Design Checklist: Optimize and Control Rate

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

![Iterative codebook optimization and rate control](/assets/images/inference/quantization-checklist-optimization.svg)

---

<!-- two-column -->
## Quantization Design Checklist: Evaluate

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

![Rate-distortion evaluation and finite-space approximation](/assets/images/inference/quantization-checklist-evaluation.svg)

---

<!-- two-column -->
## Simple Quantization

We want to convert `float32` LLM weights into smaller integer values like `int8` so the model uses less memory, while staying close enough to the original values.

**Running example:** quantize a tensor of LLM weights from `float32` to `int8`.

**1. Define the problem space**
- Object: a tensor of weights
- Goal: reduce memory
- Constraint: keep the quantized weights close enough

**2. Define the distortion function**
- Use a simple numeric error
- e.g. MSE between original and dequantized weights

<!-- column -->

**3. Characterize the distribution**
- Look at `min` and `max`
- Check whether weights are centered around zero
- Check whether the range is symmetric

**4. Choose the representation**
- Use `int8`
- Range: `[-128, 127]`
- If the float range is not centered around zero, use asymmetric quantization

---

<!-- two-column -->
## Simple Quantization

**5. Check for a closed-form solution**
- For min-max quantization, yes
- Compute `scale` and `zero_point` directly from `min` and `max`

**6. Choose an optimization method**
- Use simple min-max calibration
- No GPTQ or k-means here
- Just map the observed float range into the integer range

**7. Design the codebook**
- The reconstruction levels come from:
- `scale = (max - min) / (q_max - q_min)`
- `zero_point = round(q_min - min / scale)`

<!-- column -->

**8. Control the rate**
- Choose the bit-width
- `int8` gives 256 levels
- Fewer bits like `int4` save more memory, but increase error

**9. Measure the result**
- Quantize and dequantize
- Measure memory saved
- Measure reconstruction error
- Check whether model quality drops too much

---

<!-- two-column -->
## Simple Quantization: Asymmetric Min-Max Quantization

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
## Simple Quantization: Symmetric Linear Quantization

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

- `Model Quantization`: Reducing the precision of model parameters (weights and activations) to lower bit-widths (e.g., int8, float16) to reduce memory usage and increase inference speed.

- `KV Cache `Quantization`: Quantizing the key-value pairs stored in the attention mechanism of transformer models to reduce memory footprint and speed up attention computations.

---

## Model Quantization

Based on our implementation, we will focus on:

- GTPQ
- AWQ

---

## Model Quantization: GPTQ
GPTQ tries to find a quantized weight matrix `W_q` such that `W_q @ X ≈ W @ X` for calibration inputs `X`. It does this by minimizing the layer reconstruction error `||W @ X - W_q @ X||²` using an `OBQ-style` sequential approach: it quantizes one column of the weight matrix at a time, computes the quantization error, and updates the remaining unquantized columns using inverse-Hessian information to compensate for that error.

---

## Model Quantization: GPTQ

Psudocode

```
def GPTQ_quantize_layer(W, X, bits=4, block_size=128, damp_percent=0.01):
    out_features, in_features = W.shape
    H = 2 * X @ X.T
    damp = damp_percent * mean(diagonal(H))
    H = H + damp * identity(in_features)
    H_inv = inverse(H)
    H_inv = cholesky(H_inv).T
    Q = zeros_like(W)

    for block_start in range(0, in_features, block_size):
        block_end = min(block_start + block_size, in_features)
        E = zeros((out_features, block_end - block_start))

        for j in range(block_start, block_end):
            local_j = j - block_start
            w = W[:, j]
            q = quantize_to_grid(w, bits) // asymmetric quantization
            Q[:, j] = q
            err = (w - q) / H_inv[j, j]
            E[:, local_j] = err
            W[:, j:block_end] = W[:, j:block_end] - outer(
                err, H_inv[j, j:block_end]
            )

        W[:, block_end:] = W[:, block_end:] - (
            E @ H_inv[block_start:block_end, block_end:]
        )

    return Q
```

---

<!-- two-column -->
## Model Quantization: GPTQ

GPTQ tries to find a quantized wieght matri $$W_q$$ such that $$W_q X$$ is close to $$WX$$

The obejective is:

$$\min_{W_q} ||W X - W_q X||^2
$$

From this reconstruction loss, we get an approximate Hessian:
$$
H = 2 X X^T
$$

where X is the calibration activation matrix. The Hessian tells us how sensitive the error is to changes in the wieghts.

GPTQ uses the invers Hessian $$H^{-1}$$ because after quantizing one weight/column, it needs to know how to update the remaining unquantized weights to compensate for the quantization error.

<!-- column -->
It then uses a cholesky-based reformulation for numerical stability so it does not repeatedly perform unstable inverse-Hessian updates while quantizing each column.

For each column `j` inside a block:

```
q = quantize(W[:, j])
err = (W[:, j] - q) / H_inv[j, j]

```
GPTQ stores each column error in a temporary block error matrix E.

Then it updates the remaining columns inside the current block using this error.

After finishing the block, it applies one lazy update to all columns after the block using the stored errors E.

---


## Model Quantization: AWQ

AWQ (Activation-aware Weight Quantization), this also solving similar problem as GPTQ, Finding $$W_q$$ but unlike GPTQ, AWQ is based on the observation that weight quantization can easily damage the small fraction of salient weight channels that respond strongly to activations and are important for good output performance.

To solve this problem, AWQ scale the weight before Quantizing it

$$W_q = \text{Quantize}(W * s)$$

and then dequantize using the same scale factor

$$Y_q = \text{Dequantize}(W_q) @ (X/s)$$

How do we determin the scale factor `s`? 

$$s= S_x^\alpha$$.

Where $$S_x$$ is the per-channel activation scale, which is the maximum absolute value of the activation, and $$\alpha$$ is a hyperparameter that controls how much we want to scale the weights based on the activation scale.

AWQ searches $$\alpha$$ over $$[0, 1]$$ and chosses the one that minimizes:

$$||Q(W * s)(X/s) - WX||$$

---

## Model Quantization: AWQ
Psudocode

```
def AWQ_quantize_layer(W, X, bits=4, group_size=128, grid_size=20):
    Y_ref = W @ X
    act_mag = mean(abs(X), axis=1)
    best_error = infinity
    best_scale = None

    for alpha in linspace(0.0, 1.0, grid_size):
        channel_scale = normalize_scale(act_mag ** alpha)
        W_scaled = W * channel_scale[None, :]
        X_scaled = X / channel_scale[:, None]

        q_int, q_scale, q_zero = group_quantize_to_int(
            W_scaled, bits=bits, group_size=group_size
        )
        W_dequant = dequantize(q_int, q_scale, q_zero)
        Y_q = W_dequant @ X_scaled
        error = mse(Y_q, Y_ref)

        if error < best_error:
            best_error = error
            best_scale = channel_scale

    W_scaled = W * best_scale[None, :]
    q_int, q_scale, q_zero = group_quantize_to_int(
        W_scaled, bits=bits, group_size=group_size
    )
    W_packed = pack_low_bit_weights(q_int, bits)
    return W_packed, q_scale, q_zero, best_scale
```

---

## KV Cache Quantization
In transformer models, the attention mechanism relies on key-value (KV) pairs to compute attention scores and generate outputs. Quantizing the KV cache involves reducing the precision of these key-value pairs to save memory and speed up computations during inference.

We look into two methods for KV cache quantization:

- Turbo-Quant
- Saw int4

---

## KV Cache Quantization: Turbo-Quant

Turbo-Quant is a method for quantizing the KV cache. It does this in two stages:

- Minimize MSE error

- Reduce the unbiased inner product error

---

<!-- two-column -->
## KV Cache Quantization: Turbo-Quant

The MSE error is minimized by first rotating the input vector using a randomized Hadamard transform. This spreads the vector energy across its coordinates and makes each coordinate follow a known Beta-like distribution, which becomes close to a Gaussian distribution in high dimensions.

Since the coordinate distribution is now known, TurboQuant applies Lloyd-Max quantization via 1D k-means. It partitions the interval [-1, 1] into 2^b clusters and uses the resulting centroids as the optimal scalar codebook.

Each rotated coordinate is then replaced with the nearest codebook centroid. After dequantization, the inverse Hadamard transform is applied to reconstruct the original vector.

<!-- column -->
However, a vector can look well reconstructed but still give a biased attention dot product.

To fix this, TurboQuant checks what was lost after `TurboQuant_mse`. It takes the small leftover error, projects it with a random matrix `S`, and stores only the signs, `sign(S · r)`, as a cheap 1-bit description of the error direction.

During dequantization, those signs and the error size are used to rebuild a correction term, which is added back to the MSE reconstruction. This makes the final vector better preserve attention dot products.

---

<!-- two-column -->
## TurboQuant Algorithms

### Algorithm 1: `TurboQuant_mse`

```text
setup_mse(d, b):
    Pi = random_rotation_matrix(d, d)
    codebook = optimize_mse_centroids(
        count=2^b, range=[-1, 1]
    )
    return Pi, codebook

quant_mse(x, Pi, codebook):
    y = Pi @ x
    idx = empty_vector(length(x))

    for j in range(length(y)):
        idx[j] = nearest_centroid(y[j], codebook)

    return idx

dequant_mse(idx, Pi, codebook):
    y_hat = codebook[idx]
    x_hat = transpose(Pi) @ y_hat
    return x_hat
```

<!-- column -->

### Algorithm 2: `TurboQuant_prod`

```text
setup_prod(d, b):
    Pi, codebook = setup_mse(d, b - 1)
    S = random_normal_matrix(d, d)
    return Pi, codebook, S

quant_prod(x, Pi, codebook, S):
    idx = quant_mse(x, Pi, codebook)
    x_mse = dequant_mse(idx, Pi, codebook)
    residual = x - x_mse

    qjl = sign(S @ residual)
    gamma = l2_norm(residual)

    return idx, qjl, gamma

dequant_prod(idx, qjl, gamma,
             Pi, codebook, S):
    d = number_of_rows(S)
    x_mse = dequant_mse(idx, Pi, codebook)
    scale = sqrt(pi / 2) * gamma / d
    x_qjl = scale * transpose(S) @ qjl

    return x_mse + x_qjl
```

---

## KV Cache Quantization: SAW-INT4

SAW-INT4 is a system-aware method for quantizing the KV cache to INT4. Its goal is not only to reduce memory, but to do it in a way that still works inside real LLM serving systems.

It focuses on three things:

- Use per-token, per-head affine INT4 quantization.
- Reduce key outlier damage with block-diagonal Hadamard rotation.
- Keep the method compatible with paged KV cache and fused decode-attention kernels.

---
<!-- two-column -->
## KV Cache Quantization: SAW-INT4

Think of the KV cache as a tensor:


```text

K_cache[119, 2, :] = [0.02, 0.04, 0.03, 4.80, ...]
scale[119, 2]      = computed for this vector only
zero_point[119, 2] = computed for this vector only
INT4[119, 2, :]    = quantized along the D axis

```

However, naive INT4 loses accuracy when one channel has a large outlier:

```text
K = [0.02, 0.04, 0.03, 4.80]
                         ↑
                      outlier

```

The outlier stretches the quantization range, causing smaller values to lose precision.

<!-- column -->

To reduce this problem, SAW-INT4 applies a normalized block-diagonal Hadamard rotation to keys before INT4 quantization.

```text

Original key
K = [0.02, 0.04, 0.03, 4.80]

After block rotation
K_rot ≈ [1.22, -1.18, 1.20, -1.16]

Then quantize
INT4(K_rot) → 4-bit values + scale + zero-point

```
The rotation spreads the outlier across a small block, so INT4 no longer wastes its range on one large channel.



---

<!-- two-column -->
## SAW-INT4 Algorithm

### Setup and Cache Write

```text
setup_saw_int4(head_dim, hadamard_order=16, rotate_v=false):
    validate(hadamard_order is a power of two)
    validate(hadamard_order divides head_dim)

    allocate packed uint8 K/V paged caches
    allocate K/V scale-zero metadata
    store hadamard_order, rotate_v, and head_dim

store_kv(keys, values, slot_mapping):
    for each token and KV head:
        k = blocked_hadamard(keys[token, head])
        v = values[token, head]

        if rotate_v:
            v = blocked_hadamard(v)

        quantize_and_store(k, K_cache, K_metadata)
        quantize_and_store(v, V_cache, V_metadata)
```

<!-- column -->
### Quantization and Attention Use

```text
quantize_and_store(x, cache, metadata):
    scale = max(max(x) - min(x), 1e-8) / 15
    zero = -min(x) / scale

    q = clamp(round(x / scale + zero), 0, 15)
    packed_q = pack_two_int4_values_per_byte(q)

    store packed_q, scale, zero

decode_attention(query, paged_cache, rotate_v):
    # Backend applies matching Q rotation,
    # fused into the kernel when supported.
    output = packed_int4_attention(
        query,
        paged_cache,
        rotate_query=true
    )

    if rotate_v:
        output = blocked_hadamard(output)

    return output

prefill_attention(query, paged_cache, rotate_v):
    K, V = dequantize_active_pages(paged_cache)
    output = dense_attention(blocked_hadamard(query), K, V)

    if rotate_v:
        output = blocked_hadamard(output)

    return output
```
---
<!-- two-column -->
## Attention Mechanism: Why FlashInfer?

Our default FP16/BF16 attention backend uses FlashInfer to compute exact scaled dot-product attention:

```text
scores = Q @ K^T / sqrt(head_dim)
weights = softmax(mask(scores))
output = weights @ V
```

FlashInfer implements this using FlashAttention-style online softmax. It avoids materializing the full attention matrix in GPU memory and supports causal attention and grouped-query attention.

On NVIDIA Turing GPUs, our prefill wrappers explicitly select FlashInfer's FA2 backend. FlashInfer supports NVIDIA architectures from Turing through Hopper, while selecting architecture-appropriate kernels.

<!-- column -->

FlashInfer was created for attention during LLM inference serving, where workloads are more dynamic than training:

- Prefill and decode have very different query lengths.
- Requests have different and changing KV-cache lengths.
- KV caches may use paged, shared-prefix, or other non-contiguous layouts.
- Serving systems need flexible kernels that remain compatible with CUDA Graphs.

The paper addresses this with block-sparse KV-cache representations, customizable attention templates, and runtime load-balanced scheduling.

Source: [FlashInfer paper](https://arxiv.org/pdf/2501.01005)

---
<!-- two-column -->
## FlashInfer Setup in nano-vLLM

### Backend Setup

```text
create_flashinfer_backend(device, dtype):
    workspace = reusable_gpu_buffer(128 MiB)

    if device is Turing:
        backend = "fa2"
    else:
        backend = "auto"

    ragged_prefill = RaggedPrefillWrapper(
        workspace, layout="NHD", backend=backend
    )

    paged_prefill = PagedPrefillWrapper(
        workspace, layout="NHD", backend=backend
    )

    paged_decode = PagedDecodeWrapper(
        workspace, layout="NHD"
    )
```

The workspace and wrappers are cached and reused. The workspace is GPU scratch memory, not CUDA shared memory.

<!-- column -->
### Plan and Run

```text
prefill(q, k, v, sequence_metadata):
    wrapper = ragged_prefill
    wrapper.plan(
        cumulative_q_lengths,
        cumulative_kv_lengths,
        causal=true
    )
    return wrapper.run(q, k, v)

decode(q, paged_kv_cache, block_table, seq_lens):
    page_metadata = build_page_metadata(
        block_table, seq_lens
    )

    wrapper = paged_decode
    wrapper.plan(page_metadata, attention_shape)
    return wrapper.run(q, paged_kv_cache)
```

Decode wrappers can use fixed metadata buffers during CUDA Graph capture and update their plans before replay.

---
<!-- two-column -->
## Quantized Attention Paths

### SAW-INT4

```text
prefill(q, packed_cache):
    K_rot, V = dequantize_active_pages(packed_cache)
    q_rot = blocked_hadamard(q)
    output = FlashInfer.ragged_prefill(q_rot, K_rot, V)
    return undo_optional_value_rotation(output)

decode(q, packed_cache):
    output = custom_Triton_attention(
        q,
        packed_cache,
        apply_query_rotation=true,
        fuse_dequantization=true
    )
    return undo_optional_value_rotation(output)
```

SAW-INT4 always uses its cache-backed prefill path when real paged-cache blocks are available. It dequantizes active pages before dense FlashInfer attention, but decode reads packed INT4 directly.

<!-- column -->
### TurboQuant

```text
prefill(q, cache):
    if first_prompt_chunk:
        return FlashInfer.ragged_prefill(raw_q, raw_k, raw_v)
    if small_continuation:
        return custom_packed_attention(rotate(q), cache)

    K_rot, V = dequantize_cached_sequence(cache)
    return dense_masked_attention(rotate(q), K_rot, V)

decode(q, cache):
    if packed_decode_enabled:
        return custom_packed_attention(rotate(q), cache)
    else:
        K_rot, V = dequantize_into_reusable_workspace()
        return dense_scaled_dot_product_attention(
            rotate(q), K_rot, V
        )
```

TurboQuant uses raw FlashInfer attention for the first prompt chunk, packed kernels for decode and small continuation prefills, and PyTorch dense scaled dot-product attention after dequantization for larger prefix-cache prefills or the optional dense decode fallback.

---

## Resources

- SAW INT4 : https://arxiv.org/pdf/2604.19157
- GPTQ: https://arxiv.org/pdf/2210.17323
- awq: https://arxiv.org/pdf/2306.00978
- FlashInfer: https://arxiv.org/pdf/2501.01005
- Nano-vllm: https://github.com/Brassinai/nano-vllm