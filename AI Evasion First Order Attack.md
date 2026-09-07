# AI Evasion - First-Order Attacks: Complete Notes

---

## Section 1: Introduction to First-Order Evasion Attacks

### What Are Adversarial Examples?

Machine learning models can reach 98–99% accuracy on test data, but they can still be fooled by tiny, carefully crafted changes to inputs. These modified inputs are called **adversarial examples**. For instance, a digit image clearly showing a "7" can be tweaked by less than 1% and the model will confidently predict "2" instead. The changes are so small that a human would not notice them, but the model is completely fooled. This gap between what humans see and what models predict is the core problem that adversarial attacks expose.

### What Are First-Order Attacks?

First-order attacks use **gradient information** to craft adversarial examples. During normal training, gradients tell us how to adjust model weights to reduce error. During an attack, we freeze the weights and instead compute gradients with respect to the input — asking: "if I change this pixel slightly, how much does the prediction change?" The gradient answers this for every pixel at once. The attacker then makes coordinated small changes across all pixels to push the input across a decision boundary, while keeping total modification as small as possible. These attacks work in **white-box** (full model access) and **black-box** (query-only) scenarios.

### Two Fundamental Approaches

**FGSM (Fast Gradient Sign Method)** is the direct approach: decide upfront how much you are willing to change the input (epsilon budget), compute the gradient, then move each pixel in the direction that increases loss. Only the sign (positive or negative direction) of each gradient component matters, not its size — so every pixel changes by exactly epsilon. This makes FGSM extremely fast: one gradient computation, one step, done. Despite its simplicity, it is effective enough to be the standard baseline for measuring adversarial robustness.

**DeepFool** asks a fundamentally different question: what is the **smallest** change that fools the model? Instead of choosing a budget upfront, DeepFool iteratively searches for the closest decision boundary. It approximates the boundary as a flat surface locally, takes the shortest step to that surface, then repeats with a fresh approximation from the new position. After a few iterations it reaches the true boundary with minimal perturbation. This makes DeepFool a precise measurement instrument for model robustness, not just an attack tool.

### Why This Matters

Adversarial examples reveal a clear gap: a model can correctly classify 99% of normal test images while catastrophically failing under adversarial examples. First-order attacks are also **transferable** — an adversarial example crafted against one model often fools other models with different architectures or training. This enables realistic black-box attacks where the adversary does not need full access to the target model. For defenders, understanding these attacks is essential for evaluating defenses like adversarial training, input preprocessing, and detection systems.

### Security Frameworks

Both OWASP and Google's SAIF (Secure AI Framework) recognize evasion attacks as major threats. OWASP's Machine Learning Security Top 10 lists input manipulation as **ML01:2023**, the highest-ranked risk for traditional ML systems. SAIF recommends defense in depth: adversarial training during development, robustness evaluation before deployment, and input filtering during operation. SAIF also recommends that security teams establish red teams to continuously test production models with adversarial examples, measuring how much perturbation is needed to achieve target attack success rates.

---

## Section 2: Understanding Norms

### What Is a Norm?

A **norm** is a mathematical ruler for measuring how much an adversarial attack has changed an input. Just like distance can be measured in miles or kilometers, different norms measure changes in completely different ways. Think of travelling in New York City: you could count intersections passed (one measure), walk along streets following the grid (another measure), fly in a straight line (another), or only care about the single longest stretch you travel (yet another). Each of these is an analogy for a different norm. Norms give us precise, consistent ways to compare perturbation sizes across attacks and defenses.

### The Three Rules Every Norm Must Follow

For something to qualify as a proper norm, it must follow three rules. First, **zero means zero**: the only thing with zero size is nothing — if the measurement says zero, nothing was changed. Second, **doubling means doubling**: if you make a change twice as big, the measurement must also be twice as big, keeping measurements predictable. Third, **shortcuts do not exist**: the direct path between two points is never longer than going the roundabout way — in math this is the triangle inequality, meaning one side of a triangle cannot be longer than the other two sides combined. These rules ensure our measurement system behaves consistently and makes logical sense.

### The p-Norm Family

The most common norms are called **p-norms**, where the number p determines which measuring tool we use. The general formula is: take each change, raise it to the power p, add them all up, then take the p-th root. When p=1, we get the **L1 norm** (like walking in Manhattan — all changes add up equally). When p=2, we get the **L2 norm** (like flying in a straight line — bigger changes count much more because of squaring). When p approaches infinity, we get the **L∞ norm** (only the single largest change matters). As p increases, the norm increasingly focuses on larger values, which affects how attacks spread their perturbation budget.

### The L0 Norm

The **L0 norm** simply counts how many pixels were changed, regardless of how much each pixel changed. A pixel changed by 0.001 counts exactly the same as a pixel changed by 255. It is the simplest norm conceptually — like being allowed to use a paintbrush on only 10 pixels, but being free to paint them any color. L0 constraints naturally lead to attacks that make large changes to very few pixels. Importantly, L0 is technically not a true norm because doubling the changes does not double the count — yet it is still widely used because it captures "how many things did we touch" rather than "how much did we change them."

### The L1, L2, and L∞ Norms

**L1** adds up the absolute values of all changes, like having a total budget to distribute however you like — you could make 100 small changes or 1 large one. L1 tends to produce sparse perturbations, concentrating changes on fewer pixels. **L2** is the classic Euclidean (straight-line) distance, squaring each change before summing, which heavily penalizes large individual changes. L2 perturbations naturally spread changes evenly across all pixels, looking like a thin fog over the image. **L∞** only cares about the biggest single change across all pixels — like a speed limit where every pixel can change up to X but none can change more. L∞ perturbations look uniform, with many pixels changed by similar amounts near the maximum.

### How Norms Relate and Why They Matter

Different norms are mathematically connected — limiting one gives some automatic limits on others. In practice, the choice of norm determines the shape of the allowed perturbation region (a diamond for L1, a sphere for L2, a box for L∞) and affects both attack effectiveness and imperceptibility. L0 is computationally the hardest (requires greedy or combinatorial search). L1 requires special techniques (soft-thresholding). L2 is smooth and easy to optimize. L∞ is efficient but has sparse gradients. Understanding these trade-offs is crucial for both building attacks and building defenses.

---

## Section 3: FGSM (Fast Gradient Sign Method)

### History and Core Idea

FGSM was introduced by Goodfellow et al. in 2014 in the paper "Explaining and Harnessing Adversarial Examples." It was a wake-up call for deep learning, showing that reliable-looking classifiers could be flipped by tweaks so tiny that humans barely notice. The core idea is to **turn the model's own learning signal against it**: instead of adjusting model weights to reduce error, we adjust the input to increase error. The gradient tells us how the loss reacts to each pixel; we keep only the direction (sign) of that reaction and make a tiny, coordinated step across all pixels.

### The FGSM Formula

The adversarial input is computed as:

```
x_adv = x + ε · sign(∇_x L(θ, x, y))
```

Here, `x` is the original input, `y` is the true label, `θ` are the model parameters (frozen during attack), `L` is the loss function, `ε` is the perturbation budget, and `sign(...)` extracts only the positive or negative direction of each gradient component. The result is that every pixel changes by exactly ε in the direction that most increases the model's loss. This keeps the maximum pixel change bounded by ε (an L∞ constraint) while making the attack as damaging as possible within that budget.

### Why FGSM Works: Local Linearity and High Dimensionality

FGSM works for two reasons. First, **local linearity**: near any input point, deep neural networks behave almost linearly — small input changes produce roughly proportional output changes. So one gradient-based step already pushes the loss in the steepest direction. Second, **high dimensionality**: images have many pixels, and tiny correctly aligned tweaks at each pixel add up to a large change in the model's output score. Even if each pixel changes by only a tiny amount (bounded by ε), the cumulative effect across hundreds or thousands of pixels can easily flip a classification decision.

### Targeted vs. Untargeted FGSM

**Untargeted FGSM** increases the loss for the true label — it just tries to cause any misclassification:

```
x_adv = x + ε · sign(∇_x L(θ, x, y))
```

**Targeted FGSM** tries to make the model predict a specific target class `y_t` by decreasing the loss toward that target — the sign is flipped:

```
x_adv_targeted = x - ε · sign(∇_x L(θ, x, y_t))
```

The only differences are: (1) which label is used in the gradient computation, and (2) whether we add or subtract the perturbation. Targeted attacks are generally harder and may need a slightly larger ε since they must steer toward a specific class rather than just away from the current one.

### Backpropagation to Inputs

During a standard training pass, gradients flow with respect to model parameters to update weights. During an FGSM attack, **parameters are frozen** and we compute gradients with respect to the input itself. In PyTorch, this means setting `requires_grad=True` on the input tensor, running a forward pass to compute the loss, calling `loss.backward()`, and reading `x.grad` as the sensitivity map. This input gradient `∇_x L` is then used directly in the FGSM formula to determine the attack direction.

---

## Section 4: FGSM Setup

### Library Installation

Before running any FGSM code, install the HTB AI Library which provides common utilities for the module.

```bash
# Install the AI Library (or update it)
pip install --upgrade git+https://github.com/PandaSt0rm/htb-ai-library
```

This library provides pre-built functions for setting up reproducible environments, loading MNIST data, defining model architectures, training, and evaluating models — so you can focus on the attack logic rather than boilerplate setup code.

### Environment Setup

Reproducibility is critical — without it, gradient computations can vary between runs even on identical inputs, making debugging and comparisons meaningless. The library provides `set_reproducibility` which locks all sources of randomness.

```python
import os, random, numpy as np, torch
from torch import nn, Tensor
import torch.nn.functional as F
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

from htb_ai_library import (
    set_reproducibility, SimpleCNN, get_mnist_loaders,
    mnist_denormalize, train_model, evaluate_accuracy
)

# Seed value 1337 is used throughout; change this and results will differ
set_reproducibility(1337)
```

`set_reproducibility` controls three randomness sources: Python's `PYTHONHASHSEED` (dictionary ordering), Python's `random` module and NumPy's generator (both seeded to 1337), and PyTorch's cuDNN backend (switched to deterministic mode, trading a small speed penalty for perfect reproducibility).

### Device Configuration

PyTorch operations need a target device (CPU or GPU). Checking availability ensures code runs everywhere from laptops to cloud GPU instances.

```python
# Configure computation device
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```

`torch.cuda.is_available()` returns `True` when NVIDIA drivers and CUDA toolkit are installed. GPU acceleration typically speeds training by 10–50x for convolutional networks, but the small MNIST model trains fast enough on CPU that device choice will not significantly affect the demonstration.

### Data Loading

The library's `get_mnist_loaders` handles downloading MNIST, converting images to tensors, applying normalization, and creating batched data loaders.

```python
# Prepare data loaders (normalized space)
train_loader, test_loader = get_mnist_loaders(batch_size=128, normalize=True)
```

`get_mnist_loaders` applies `transforms.ToTensor()` to convert PIL images to PyTorch tensors (rescaling pixels from 0–255 to 0–1), then applies MNIST-specific normalization (mean=0.1307, std=0.3081) when `normalize=True`. It downloads MNIST to `./data` if not already present, seeds the training loader's shuffle generator to 1337 for reproducibility, and sets `num_workers=0` to avoid multiprocessing issues.

### Model Definition and Training

We use `SimpleCNN`: two convolutional layers extracting spatial features, followed by two fully connected layers for classification. After one epoch on MNIST, it achieves ~98% accuracy.

```python
# Initialize model
model = SimpleCNN().to(device)

# Train the model (1 epoch is enough for MNIST)
trained_model = train_model(model, train_loader, test_loader, epochs=1, device=device)

# Evaluate baseline accuracy
baseline_acc = evaluate_accuracy(trained_model, test_loader, device)
print(f"Baseline test accuracy: {baseline_acc:.2f}%")
# Expected output: Baseline test accuracy: 98.41%
```

`SimpleCNN` architecture: Conv1 (1→32 channels, 3×3, padding=1) → ReLU → Conv2 (32→64, 3×3) → two MaxPool ops reducing 28×28 to 7×7 → flatten to 3136 → FC(3136→128) → FC(128→10 logits). The `.to(device)` call moves all parameters to GPU or CPU. One epoch suffices for MNIST because the dataset is small (60,000 training images) and patterns are simple.

---

## Section 5: Normalization

### What Normalization Does

Normalization transforms input data to have **zero mean and unit variance** using the formula:

```
x_norm = (x - μ) / σ
```

For MNIST, `μ = 0.1307` (average pixel intensity across all training images in [0,1] range — low because most pixels are black background) and `σ = 0.3081` (how much pixels vary around that mean). A bright pixel with value 0.8 becomes `(0.8 - 0.1307) / 0.3081 ≈ 2.17`, and a black background pixel with value 0.0 becomes approximately `-0.42`. Visually, normalized and unnormalized images look identical to humans — the transformation only changes the numbers the network sees, not the visual appearance.

### Why Normalization Matters for Training

Without normalization, all pixels are positive (0–1 range), causing all gradients in the first layer to point in similar directions — making training slow and inefficient. With normalization, inputs have both positive and negative values (roughly -0.42 to 2.82), giving gradients positive and negative components that explore the parameter space more efficiently. The practical difference is huge: training a CNN on unnormalized MNIST might reach 85% accuracy after 20 epochs, while normalized MNIST reaches 98% accuracy in just **1 epoch**. Normalization also keeps gradient magnitudes stable across all layers, which is essential for deeper networks.

### Normalization and Attack Vulnerability

Faster training with normalization creates sharper, more confident decision boundaries. Sharp boundaries mean small input perturbations can push samples across them more easily — making the model simultaneously **more accurate on clean data and more vulnerable to adversarial examples**. An underfit model has fuzzy, uncertain boundaries; FGSM often fails against it because gradients point nowhere useful. A well-trained normalized model has crisp boundaries that gradient-based attacks can exploit effectively. This paradox — accuracy and vulnerability going hand in hand — drives much of modern adversarial robustness research.

### Impact on Epsilon Budgets

Normalization changes how we interpret ε budgets. When the model is trained on normalized inputs, gradients are computed in normalized space, so ε must also be specified in normalized units. The conversion between normalized-space and pixel-space epsilon is:

```
ε_pixel = σ · ε_norm
```

For MNIST with σ=0.3081: `ε_norm = 0.8` in normalized space corresponds to `ε_pixel = 0.8 × 0.3081 ≈ 0.25` in [0,1] pixel space, which equals roughly 64 intensity levels at 8-bit precision. Attack code typically works in normalized space (where the model operates), but visualization converts back to [0,1] pixel space (where humans perceive images) using `mnist_denormalize`.

### Valid Input Range After Normalization

When MNIST pixels are normalized, the valid input range changes from [0,1] to approximately [-0.424, 2.821]. These bounds come from applying the normalization formula to the original limits:

```
x_min = (0.0 - 0.1307) / 0.3081 ≈ -0.424
x_max = (1.0 - 0.1307) / 0.3081 ≈ 2.821
```

These constants (often called `MNIST_NORM_MIN` and `MNIST_NORM_MAX`) appear throughout attack implementations as clamping bounds. When we clamp adversarial images to these bounds, we ensure they remain valid MNIST inputs that can be converted back to [0,1] pixel space for visualization without clipping artifacts.

---

## Section 6: Core FGSM Implementation

### Computing Loss Without Side Effects

A clean helper function separates forward computation and loss calculation, keeping gradient flow scoped to the input tensor and making debugging simpler.

```python
def _forward_and_loss(model, x, y):
    if getattr(model, "training", False):
        raise RuntimeError("Expected model.eval() for attack computations")
    logits = model(x)
    loss = F.cross_entropy(logits, y)
    return logits, loss
```

For a batch of 128 MNIST images, `x` has shape [128, 1, 28, 28]. The model outputs `logits` of shape [128, 10] (one row per image, 10 class scores). Cross-entropy reduces these to a single scalar loss. The function raises an error if the model is in training mode, because batch normalization and dropout state updates would corrupt gradient computations during an attack.

### Gradient Computation

The attack requires gradients with respect to inputs rather than parameters. The implementation clones the input (to avoid modifying the original), enables gradient tracking, computes loss, runs backward, then returns the gradient.

```python
def _input_gradient(model, x, y):
    x_req = x.clone().detach().requires_grad_(True)
    _, loss = _forward_and_loss(model, x_req, y)
    model.zero_grad(set_to_none=True)
    loss.backward()
    return x_req.grad.detach()
```

`x.clone().detach()` creates a new tensor disconnected from computational history. `.requires_grad_(True)` enables gradient tracking. After `loss.backward()`, PyTorch stores `∇_x L` in `x_req.grad`. The final `.detach()` returns a standalone gradient tensor with the same shape as the input — [128, 1, 28, 28] input produces [128, 1, 28, 28] gradient.

### Core FGSM Attack Function

With gradients in hand, the actual attack is four lines: extract the sign, scale by ε, add to the image, and clamp to valid range.

```python
def fgsm_attack(model, images, labels, epsilon, targeted=False):
    MNIST_NORM_MIN = (0.0 - 0.1307) / 0.3081   # ≈ -0.424
    MNIST_NORM_MAX = (1.0 - 0.1307) / 0.3081   # ≈ 2.821

    if epsilon < 0:
        raise ValueError("epsilon must be non-negative")
    if not images.is_floating_point():
        raise ValueError("images must be floating point tensors")

    grad = _input_gradient(model, images, labels)
    step_dir = -1.0 if targeted else 1.0
    x_adv = images + step_dir * epsilon * grad.sign()
    x_adv = torch.clamp(x_adv, MNIST_NORM_MIN, MNIST_NORM_MAX)
    return x_adv.detach()
```

`grad.sign()` operates element-wise, returning +1, -1, or 0 for each gradient element. Multiplying by `epsilon` creates a perturbation where each pixel changes by exactly epsilon (or 0 if gradient was zero). `step_dir = 1.0` for untargeted (add perturbation, increase loss) and `-1.0` for targeted (subtract perturbation, decrease loss toward target). `torch.clamp` ensures all values stay in the valid normalized range.

### Testing the Attack

```python
images, labels = next(iter(test_loader))
images, labels = images.to(device), labels.to(device)

model.eval()
epsilon = 0.8  # in normalized space (≈0.25 in pixel space)
with torch.no_grad():
    clean_pred = model(images).argmax(dim=1)

x_adv = fgsm_attack(model, images, labels, epsilon)
with torch.no_grad():
    adv_pred = model(x_adv).argmax(dim=1)

originally_correct = (clean_pred == labels)
flipped = (adv_pred != labels) & originally_correct
success = flipped.sum().item() / max(int(originally_correct.sum().item()), 1)
print(f"FGSM flips (first batch): {success:.2%}")
# Expected output: FGSM flips (first batch): 71.09%
```

With ε=0.8 in normalized space (≈0.25 pixel space, ≈64 intensity levels), FGSM successfully flips 71.09% of originally correct predictions. The exact flip rate varies with the trained weights and the specific batch.

### Pixel-Space FGSM Variant

For workflows that start with raw [0,1] pixel-space images but need to attack a model expecting normalized inputs, a separate variant handles the conversion. The key step is dividing normalized-space gradients by σ to map them back to pixel space before applying the sign step.

```python
def fgsm_pixel_space(model, images, labels, epsilon, mean, std, targeted=False):
    mean_t, std_t = _norm_params(images, mean, std)
    x = images.clone().detach()
    x_norm = (x - mean_t) / std_t
    x_norm.requires_grad_(True)

    _, loss = _forward_and_loss(model, x_norm, labels)
    model.zero_grad(set_to_none=True)
    loss.backward()

    # Convert gradient from normalized space to image space by dividing by std
    grad_img = x_norm.grad / std_t
    step_dir = -1.0 if targeted else 1.0
    x_adv = torch.clamp(x + step_dir * epsilon * grad_img.sign(), 0.0, 1.0)
    return x_adv.detach()
```

Dividing by `std_t` undoes the scaling introduced by normalization, so `epsilon=8/255 ≈ 0.031` means exactly 8 intensity levels in the original image. If you already work with normalized data (as with `get_mnist_loaders(normalize=True)`), use `fgsm_attack` directly.

---

## Section 7: Evaluation Metrics

### What Metrics Do We Need?

Evaluating an adversarial attack requires looking beyond whether predictions flip. We need: **accuracy metrics** (how many samples the model classifies correctly before and after), **success rate** (what percentage of originally correct predictions the attack flips), **confidence metrics** (how certain the model is in its predictions before and after), and **perturbation norms** (how much the image actually changed, measured in both L2 distance and L∞ maximum change). Together these give a complete picture of attack impact.

### Building the Evaluation Function

```python
from typing import Dict

def evaluate_attack(model, clean_images, adversarial_images, true_labels) -> Dict[str, float]:
    model.eval()
    with torch.no_grad():
        clean_logits = model(clean_images)
        adv_logits = model(adversarial_images)

        clean_probs = F.softmax(clean_logits, dim=1)
        adv_probs = F.softmax(adv_logits, dim=1)

        clean_pred = clean_logits.argmax(dim=1)
        adv_pred = adv_logits.argmax(dim=1)
```

`torch.no_grad()` disables gradient tracking since we are only evaluating. `F.softmax` converts logits to probabilities (each row sums to 1). `argmax(dim=1)` extracts the highest-scoring class index for each sample. For 128 images with 10 classes, `clean_probs` has shape [128, 10] and `clean_pred` has shape [128].

### Computing Success Rate and Confidence

```python
        clean_correct = (clean_pred == true_labels)
        adv_correct = (adv_pred == true_labels)
        flipped = (~adv_correct) & clean_correct

        # True class confidence before and after attack
        conf_clean = clean_probs.gather(1, true_labels.view(-1, 1)).squeeze(1)
        conf_adv = adv_probs.gather(1, true_labels.view(-1, 1)).squeeze(1)
```

`flipped` uses logical operations: True where clean was correct AND adversarial is wrong. `gather(1, true_labels.view(-1, 1))` selects the probability for the true class from each row — for example if `true_labels=[3,7]`, it picks column 3 from row 0 and column 7 from row 1. `squeeze(1)` removes the extra dimension, yielding a 1D tensor of confidence values.

### Computing Perturbation Norms and Returning Metrics

```python
        l2 = (adversarial_images - clean_images).view(clean_images.size(0), -1).norm(p=2, dim=1)
        linf = (adversarial_images - clean_images).abs().amax()

        return {
            "clean_accuracy": clean_correct.float().mean().item(),
            "adversarial_accuracy": adv_correct.float().mean().item(),
            "attack_success_rate": (flipped.float().sum() / originally_correct.float().sum().clamp_min(1.0)).item(),
            "avg_clean_confidence": conf_clean.mean().item(),
            "avg_adv_confidence": conf_adv.mean().item(),
            "avg_confidence_drop": (conf_clean - conf_adv).mean().item(),
            "avg_l2_perturbation": l2.mean().item(),
            "max_linf_perturbation": linf.item(),
        }

# Example output for FGSM with epsilon=0.8:
# clean_accuracy: 0.9766
# adversarial_accuracy: 0.3203
# attack_success_rate: 0.6797
# avg_clean_confidence: 0.9824
# avg_adv_confidence: 0.3891
# avg_confidence_drop: 0.5933
# avg_l2_perturbation: 10.8451
# max_linf_perturbation: 0.8000
```

`.view(batch, -1)` flattens each image to 1D for per-sample L2 norm computation. `norm(p=2, dim=1)` computes L2 norm along dimension 1. `amax()` without specifying dimensions returns the single largest absolute perturbation across the entire batch, which should not exceed epsilon.

---

## Section 8: Visualization

### Visualization Setup

Consistent visual styling is applied using a dark HTB theme. A helper function `_style_axes` applies the theme to matplotlib axes objects.

```python
import matplotlib.pyplot as plt
import numpy as np
from htb_ai_library import (
    HTB_GREEN, NODE_BLACK, HACKER_GREY, WHITE,
    AZURE, NUGGET_YELLOW, MALWARE_RED, VIVID_PURPLE, AQUAMARINE
)

def _style_axes(ax):
    ax.set_facecolor(NODE_BLACK)
    ax.tick_params(colors=HACKER_GREY)
    for spine in ax.spines.values():
        spine.set_color(HACKER_GREY)
    ax.grid(True, color=HACKER_GREY, linestyle="--", alpha=0.25)
```

This sets the background color, tick colors, spine (border) colors, and adds a faint dashed grid with 25% transparency so it does not dominate the visualization.

### Main Visualization Function

The `visualize_attack` function displays three image panels (original, adversarial, perturbation) and one class probability bar chart side by side.

```python
def visualize_attack(model, image, label, make_adv, title, num_classes=10, targeted=False, target_class=None):
    model.eval()
    dev = next(model.parameters()).device
    image_dev = image.to(dev)
    label_dev = label.to(dev)

    # Compute clean predictions
    with torch.no_grad():
        clean_probs = F.softmax(model(image_dev.unsqueeze(0)), dim=1).squeeze(0)
        clean_pred = int(clean_probs.argmax().item())

    # Generate adversarial and compute predictions
    x_adv_dev = make_adv(model, image_dev.unsqueeze(0), label_dev.unsqueeze(0)).squeeze(0)
    with torch.no_grad():
        adv_probs = F.softmax(model(x_adv_dev.unsqueeze(0)), dim=1).squeeze(0)
        adv_pred = int(adv_probs.argmax().item())

    # Denormalize for visualization (convert from normalized space to [0,1] pixel space)
    image_vis = mnist_denormalize(image_dev.unsqueeze(0)).squeeze(0).detach().cpu()
    x_adv_vis = mnist_denormalize(x_adv_dev.unsqueeze(0)).squeeze(0).detach().cpu()
    perturbation_vis = x_adv_vis - image_vis
```

`unsqueeze(0)` adds a batch dimension since models expect batched inputs. `mnist_denormalize` converts from normalized space (values like -0.3 or 2.5) back to [0,1] pixel space for visualization. Without denormalization, images would be unrecognizable when displayed.

### Perturbation Scaling and Probability Bars

The perturbation panel uses a scaling trick to make tiny changes visible:

```python
    # Perturbation: scale for visibility
    # perturbation_vis * 10 amplifies the range; + 0.5 centers so no-change = gray
    pert_scaled = (perturbation_vis * 10 + 0.5).clamp(0, 1)
```

Raw perturbations are typically in the range [-0.05, +0.05] and would appear as uniform gray. Multiplying by 10 amplifies to [-0.5, +0.5], then adding 0.5 centers the range to [0,1]: neutral = medium gray (0.5), positive perturbation = lighter, negative = darker. `clamp(0,1)` keeps values in valid display range.

### FGSM-Specific Wrapper and Usage

```python
def visualize_fgsm_attack(model, image, label, epsilon, num_classes=10, targeted=False, target_class=None):
    def _make_adv(m, xb, yb):
        y_used = yb if not targeted else torch.full_like(yb, target_class)
        return fgsm_attack(m, xb, y_used, epsilon, targeted=targeted)

    mode = "Targeted" if targeted else "Untargeted"
    visualize_attack(model, image, label, _make_adv,
                    title=f"FGSM {mode}", num_classes=num_classes,
                    targeted=targeted, target_class=target_class)

# Usage: visualize a single test image
_ = visualize_fgsm_attack(model, images[0].detach().cpu(), labels[0].detach().cpu(), epsilon)
```

The wrapper creates a `_make_adv` function that calls `fgsm_attack` with the right label — true label for untargeted, or a full tensor of the target class for targeted attacks. The visualization shows how a small L∞-bounded change materially changes the model's predicted class probabilities.

---

## Section 9: Targeted FGSM

### Targeted vs. Untargeted: The Difference

Targeted FGSM changes the objective from reducing confidence in the true class to **increasing confidence in a specific target class**. The formula flips the sign of the gradient step and uses the target label in the loss:

```
x_adv_targeted = x - ε · sign(∇_x L(θ, x, y_target))
```

Only two things change from untargeted FGSM: (1) the gradient uses target label `y_t` instead of the true label, and (2) the update sign flips so the step reduces loss for `y_t` rather than increasing loss for `y`. The L∞ budget and clamping remain unchanged. Because targeted FGSM must steer toward a particular class (not just away from the current one), it often requires slightly larger ε.

### Targeted Attack Example: Forcing 1 → 7

The following code searches for a correctly classified digit "1" in the test set, then tries progressively larger epsilon values until the model predicts "7":

```python
eps_candidates = [0.5, 0.8, 1.0]
success_image, success_label, success_eps = None, None, None

model.eval()
candidate, candidate_label = None, None

# Step 1: Find a correctly classified digit 1
for xb, yb in test_loader:
    xb, yb = xb.to(device), yb.to(device)
    match_indices = (yb == 1).nonzero(as_tuple=True)[0]
    if len(match_indices) == 0:
        continue
    with torch.no_grad():
        preds = model(xb[match_indices]).argmax(dim=1)
        correct_mask = (preds == 1)
        if correct_mask.any():
            local_idx = correct_mask.nonzero(as_tuple=True)[0][0].item()
            idx = match_indices[local_idx].item()
            candidate = xb[idx]
            candidate_label = yb[idx]
            break
```

`(yb == 1)` creates a boolean tensor. `.nonzero(as_tuple=True)[0]` extracts indices of digit-1 samples. We verify those are correctly classified, then take the first success. The double indexing `match_indices[local_idx]` maps from the filtered subset back to the original batch position.

### Testing Epsilon Values and Visualizing Success

```python
# Step 2: Try each epsilon until attack succeeds
target_label = torch.tensor([7], device=device)

for eps_try in eps_candidates:
    x_adv = fgsm_attack(model, candidate.unsqueeze(0), target_label, epsilon=eps_try, targeted=True)
    with torch.no_grad():
        pred = model(x_adv).argmax(dim=1).item()
    print(f"epsilon={eps_try:.2f} -> predicted {pred}")

    if pred == 7:
        success_image = candidate
        success_label = candidate_label
        success_eps = eps_try
        break

# Step 3: Visualize the successful attack
if success_image is None:
    raise RuntimeError("Targeted FGSM did not achieve 1 -> 7 within tested epsilons.")

_ = visualize_fgsm_attack(model, success_image.detach().cpu(),
                          success_label.detach().cpu(), success_eps,
                          targeted=True, target_class=7)

# Expected output:
# epsilon=0.50 -> predicted 1
# epsilon=0.80 -> predicted 7
```

ε=0.5 fails (prediction stays 1), but ε=0.8 succeeds (prediction becomes 7). This shows targeted attacks often need more budget than untargeted. `.unsqueeze(0)` adds the batch dimension. The `break` terminates the loop early once we find the minimal working epsilon.

---

## Section 10: I-FGSM (Iterative FGSM)

### What Is I-FGSM?

The **Iterative Fast Gradient Sign Method** (I-FGSM), also called the Basic Iterative Method (BIM), was introduced by Kurakin et al. in 2016 as an extension of FGSM. Instead of one large step, the algorithm takes **several small, well-aimed steps**. Each step follows the gradient sign at the current position, then projects back onto the allowed L∞ budget around the original image. The projection ensures the total perturbation never exceeds ε even though we take multiple steps. The result is a stronger adversarial example that more reliably crosses decision boundaries with the same overall budget.

### Core Update Rule and Projection

Starting from x⁽⁰⁾ = x, the iterative update is:

```
x⁽ᵗ⁺¹⁾ = Π_B∞(x, ε) ( x⁽ᵗ⁾ + α · sign(∇ₓ L(θ, x⁽ᵗ⁾, y)) )
```

Where α is the step size (typically `α = ε/T` for T iterations) and `Π_B∞(x, ε)` is the projection back onto the L∞ ball of radius ε around the original x. The projection is simply per-pixel clipping: `clip(x' - x, -ε, ε)` measures how far we drifted from the original, clips any excess, then `clip(x', x_min, x_max)` ensures valid pixel values. Projection happens **relative to the original input**, not the previous iterate — so ε always means "ε away from the starting point."

### Why Iteration Helps

A single FGSM step approximates the curved loss surface with a flat plane. After stepping, that linear approximation is no longer accurate at the new position. I-FGSM recomputes the gradient at each new position `x⁽ᵗ⁾`, adjusting to the local geometry at each step. Over several steps, this better tracks the true curved loss surface and finds stronger perturbations within the same budget. For example: FGSM with ε=0.8 might move loss from 0.3 to 1.2. I-FGSM with 10 iterations and α=0.08 might evolve loss as: 0.3 → 0.45 → 0.62 → 0.81 → 0.98 → ... → 1.68, achieving **53% more loss increase with the same L∞ budget**.

### Prerequisites

Before implementing I-FGSM, ensure you have all the code from FGSM Setup and Core Implementation sections:
- A trained `SimpleCNN` model on MNIST with ~98% clean accuracy
- The `test_loader` and `device` configuration
- The `_input_gradient` and `fgsm_attack` functions

These are needed because I-FGSM builds directly on the FGSM infrastructure.

---

## Section 11: I-FGSM Implementation

### Core Iterative Algorithm

```python
def iterative_fgsm(model, images, labels, epsilon, num_iter, alpha=None, targeted=False, random_start=False):
    MNIST_NORM_MIN = (0.0 - 0.1307) / 0.3081
    MNIST_NORM_MAX = (1.0 - 0.1307) / 0.3081

    # Default step size divides budget across iterations
    if alpha is None:
        alpha = epsilon / max(num_iter, 1)

    # Optional random initialization within epsilon ball
    if random_start:
        torch.manual_seed(1337)
        delta = torch.empty_like(images).uniform_(-epsilon, epsilon)
        x_adv = torch.clamp(images + delta, MNIST_NORM_MIN, MNIST_NORM_MAX)
    else:
        x_adv = images.clone()

    for _ in range(num_iter):
        x_adv = x_adv.detach().requires_grad_(True)
        logits = model(x_adv)
        loss = F.cross_entropy(logits, labels)
        model.zero_grad(set_to_none=True)
        loss.backward()
        step_dir = -1.0 if targeted else 1.0
        x_adv = x_adv + step_dir * alpha * x_adv.grad.sign()
        # Project: clamp drift to [-ε, ε] relative to original, then clamp to valid range
        x_adv = torch.clamp(images + (x_adv - images).clamp(-epsilon, epsilon), MNIST_NORM_MIN, MNIST_NORM_MAX)

    return x_adv.detach()
```

The default `alpha = epsilon / max(num_iter, 1)` distributes the budget: if ε=0.8 and 10 iterations, then α=0.08 per step. Random start adds uniform noise in [-ε, ε] before iterating, helping escape local neighborhoods where gradients point nowhere useful. The two-step clamping at the end first enforces the L∞ constraint (drift ≤ ε from original) then enforces valid pixel range.

### Testing Iterative FGSM

```python
images, labels = next(iter(test_loader))
images, labels = images.to(device), labels.to(device)

epsilon = 0.8
num_iter = 10
alpha = epsilon / num_iter  # = 0.08

with torch.no_grad():
    clean_pred = model(images).argmax(dim=1)

x_adv_ifgsm = iterative_fgsm(
    model, images, labels,
    epsilon=epsilon, num_iter=num_iter, alpha=alpha,
    targeted=False, random_start=True
)

with torch.no_grad():
    adv_pred_ifgsm = model(x_adv_ifgsm).argmax(dim=1)

originally_correct = clean_pred == labels
flipped_ifgsm = (adv_pred_ifgsm != labels) & originally_correct
success_rate = flipped_ifgsm.float().sum() / originally_correct.float().sum().clamp_min(1.0)
print(f"I-FGSM flips (first batch): {success_rate.item():.2%}")
# Expected output: I-FGSM flips (first batch): 100.00%
```

At the same ε=0.8, I-FGSM achieves **100% flip rate** compared to FGSM's 71.09%. `random_start=True` helps by exploring different starting positions in the loss landscape.

### Measuring Full Attack Impact

```python
metrics_ifgsm = evaluate_attack(model, images, x_adv_ifgsm, labels)
for k, v in metrics_ifgsm.items():
    print(f"{k}: {v:.4f}")

# Expected output:
# clean_accuracy: 1.0000
# adversarial_accuracy: 0.0000
# attack_success_rate: 1.0000
# avg_clean_confidence: 0.9853
# avg_adv_confidence: 0.0115
# avg_confidence_drop: 0.9739
# avg_l2_perturbation: 14.0519
# max_linf_perturbation: 0.8000
```

Complete collapse in accuracy: all 128 samples (100%) flipped successfully. Average confidence in the true class dropped from 98.53% to just 1.15% — a 97.39 percentage point drop. `max_linf_perturbation: 0.8000` confirms the ε bound is respected. The higher `avg_l2_perturbation` (14.05 vs 10.85 for FGSM) reflects the budget being used more broadly across pixels through iterative refinement.

---

## Section 12: I-FGSM Analysis

### Visualization of I-FGSM

```python
def visualize_ifgsm(model, image, label, epsilon, num_iter, targeted=False, target_class=None):
    alpha = epsilon / max(num_iter, 1)

    def _make_adv(m, xb, yb):
        y_used = yb if not targeted else torch.full_like(yb, target_class)
        return iterative_fgsm(m, xb, y_used, epsilon, num_iter, alpha, targeted=targeted, random_start=True)

    mode = "Targeted" if targeted else "Untargeted"
    visualize_attack(model, image, label, _make_adv,
                    title=f"I-FGSM {mode}", targeted=targeted, target_class=target_class)

# Usage
_ = visualize_ifgsm(model, images[0].detach().cpu(), labels[0].detach().cpu(), epsilon, num_iter)
```

Iterative refinement often produces more structured perturbations than FGSM — focusing modifications on regions most sensitive to the model's decision rather than uniformly changing all pixels. The visualization reveals how repeated gradient recomputation leads to more targeted attacks.

### Step Size and Iteration Trade-offs

Choosing α and T balances speed and attack strength. With **large alpha and few iterations** (e.g., α=0.1, T=2), the method moves quickly but may overshoot optimal adversarials — loss might jump: 0.3 → 0.9 → 1.1. With **small alpha and many iterations** (e.g., α=0.01, T=20), the method refines gradually — loss evolves smoothly: 0.3 → 0.35 → 0.41 → ... → 1.45. The gradual path often finds stronger adversarials because it follows the curved loss surface more accurately. Random starts can increase success rate from 85% to 92% at the same budget by escaping local neighborhoods.

### Relation to PGD

**Projected Gradient Descent (PGD)**, formalized by Madry et al. (2017), is the general form of iterative attacks under constraints. With the sign step, α=ε/T, and optional random restarts, the I-FGSM implementation is essentially PGD for the L∞ threat model. The update rule is identical: `x⁽ᵗ⁺¹⁾ = Π_B∞(x⁽ᵗ⁾ + α · sign(∇L))`. BIM (Basic Iterative Method) is I-FGSM without random initialization. PGD adds multiple random restarts to explore different initialization points, keeping the strongest adversarial found. Random restarts further increase reliability while keeping the same budget.

### Comparison: FGSM vs. I-FGSM

```python
epsilon = 0.7
x_adv_fgsm = fgsm_attack(model, images, labels, epsilon)
x_adv_ifgsm = iterative_fgsm(model, images, labels, epsilon, num_iter=10, random_start=True)

with torch.no_grad():
    fgsm_pred = model(x_adv_fgsm).argmax(dim=1)
    ifgsm_pred = model(x_adv_ifgsm).argmax(dim=1)

orig_correct = clean_pred == labels
fgsm_success = ((fgsm_pred != labels) & orig_correct).float().sum() / orig_correct.float().sum().clamp_min(1.0)
ifgsm_success = ((ifgsm_pred != labels) & orig_correct).float().sum() / orig_correct.float().sum().clamp_min(1.0)

print(f"FGSM success rate: {fgsm_success:.1%}")
print(f"I-FGSM success rate: {ifgsm_success:.1%}")
print(f"Improvement: {(ifgsm_success - fgsm_success) / fgsm_success:.1%}")

# Expected output:
# FGSM success rate: 57.8%
# I-FGSM success rate: 95.3%
# Improvement: 64.9%
```

I-FGSM achieves 95.3% success where FGSM achieves 57.8% — a **64.9% relative improvement** at the same L∞ budget of ε=0.7. Both methods respect the identical per-pixel bound; the improvement comes entirely from better exploiting the loss landscape through iterative refinement.

---

## Section 13: FGSM Challenge

### Objective

Craft an adversarial example that fools an MNIST digit classifier using FGSM. You receive a baseline image the classifier correctly predicts. Add a small, carefully crafted perturbation that causes misclassification while staying within a strict L∞ perturbation budget: `‖x_adv − x‖∞ ≤ ε`. Both conditions must hold — predicted class differs from baseline label, AND maximum absolute pixel difference is at most epsilon. All API endpoints expect images in **[0,1] pixel space**, not normalized tensors.

### Step-by-Step: Setting Up and Checking the Challenge

```bash
# Step 1: Export your instance URL
export BASE_URL="http://instance_ip:port"

# Step 2: Check that the server is running and ready
curl -s "$BASE_URL/health"

# Step 3: Get the challenge — returns the image, true label, and epsilon
curl -s "$BASE_URL/challenge" | jq
# Example response:
# { "sample_index": 2, "label": 1, "epsilon": 0.25, "image_b64": "<base64 PNG>" }

# Step 4: Download the model weights for local gradient computation
curl -s -o fgsm_weights.pth "$BASE_URL/weights"
```

The `/health` endpoint returns `{"status": "ok", "epsilon": 0.25, "index": 2}`. The `/challenge` endpoint returns the baseline image as a base64-encoded PNG, the true label, and the epsilon constraint. The `/weights` endpoint downloads a PyTorch state dict compatible with the provided `SimpleClassifier` architecture.

### Step-by-Step: Loading the Model and Image

```python
import os, io, base64, numpy as np, requests, torch, torch.nn as nn
from PIL import Image

BASE_URL = os.getenv("BASE_URL", "http://127.0.0.1:8000")
MNIST_MEAN, MNIST_STD = 0.1307, 0.3081

# Step 1: Helper — convert base64 PNG to [0,1] numpy array
def x01_from_b64_png(b64):
    raw = base64.b64decode(b64)
    img = Image.open(io.BytesIO(raw)).convert("L")
    return np.clip(np.asarray(img, dtype=np.float32) / 255.0, 0.0, 1.0)

# Step 2: Helper — convert [0,1] array back to base64 PNG
def b64_png_from_x01(x2d):
    x255 = np.clip((x2d * 255.0).round(), 0, 255).astype(np.uint8)
    img = Image.fromarray(x255, mode="L")
    buf = io.BytesIO()
    img.save(buf, format="PNG", optimize=True)
    return base64.b64encode(buf.getvalue()).decode("ascii")

# Step 3: Fetch challenge details
ch = requests.get(f"{BASE_URL}/challenge", timeout=10).json()
x = x01_from_b64_png(ch["image_b64"])    # (28, 28) array in [0,1]
lab = int(ch["label"])                   # true label
eps = float(ch["epsilon"])               # L∞ constraint

# Step 4: Define the SimpleClassifier architecture (must match server)
class SimpleClassifier(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 32, 3, 1)
        self.conv2 = nn.Conv2d(32, 64, 3, 1)
        self.dropout1 = nn.Dropout(0.25)
        self.dropout2 = nn.Dropout(0.5)
        self.fc1 = nn.Linear(9216, 128)
        self.fc2 = nn.Linear(128, 10)

    def forward(self, x01):
        x = (x01 - MNIST_MEAN) / MNIST_STD   # internal normalization
        x = torch.relu(self.conv1(x))
        x = torch.relu(self.conv2(x))
        x = torch.max_pool2d(x, 2)
        x = self.dropout1(x)
        x = torch.flatten(x, 1)
        x = torch.relu(self.fc1(x))
        x = self.dropout2(x)
        x = self.fc2(x)
        return torch.log_softmax(x, dim=1)

# Step 5: Load downloaded weights
model = SimpleClassifier().eval()
state = torch.load("fgsm_weights.pth", map_location=torch.device("cpu"))
model.load_state_dict(state)

# Step 6: Verify local prediction matches server
x_tensor = torch.from_numpy(x[None, None, ...]).float()
logits = model(x_tensor)
local_pred = int(torch.argmax(logits, dim=1).item())
print(f"Local prediction: {local_pred}")
```

The `SimpleClassifier` does **internal normalization** in its `forward` method — it accepts [0,1] pixel-space inputs and normalizes them internally. This is why the challenge API uses pixel space, not normalized space.

### Step-by-Step: Running the Attack and Submitting

```bash
# Step 1: Test that submitting the clean image fails (expected)
curl -s -X POST "$BASE_URL/predict" \
  -H 'content-type: application/json' \
  -d '{"image_b64": "<clean_image_base64>"}' | jq
# Returns: {"pred": 1, "confidence": 0.99} -- should match label

# Step 2: Implement FGSM attack in pixel space using fgsm_pixel_space() function
# (apply the attack with epsilon = 0.25, mean=[0.1307], std=[0.3081])

# Step 3: Submit your adversarial image
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image_b64": "<adversarial_image_base64>"}' | jq
# Success response:
# {"ok": true, "pred": 8, "linf": 0.24, "flag": "HTB{...}"}
# Failure: HTTP 400 with error message like "L_inf too large: 0.26 > 0.25"
```

Use `fgsm_pixel_space()` (from Section 6) since the model expects [0,1] inputs but does internal normalization. Specify `epsilon=0.25` (the pixel-space budget), `mean=[0.1307]`, `std=[0.3081]`. Convert your adversarial tensor back to a base64 PNG using `b64_png_from_x01` and POST it to `/submit`.

---

## Section 14: DeepFool and the Quest for Minimality

### From Fixed Budgets to Minimal Perturbations

FGSM asks: "Given a hammer of size ε, where should I strike?" DeepFool asks: "What is the **smallest hammer** that will break this?" FGSM maximizes damage within a fixed L∞ budget ε chosen upfront. DeepFool minimizes the budget needed to achieve misclassification, discovering constraints rather than imposing them. The choice of ε in FGSM is a hyperparameter that conflates the attack method with the model's true robustness — two models might appear equally robust under one ε but show very different vulnerabilities when ε changes. DeepFool eliminates this arbitrary choice, making the resulting perturbation magnitude a direct measurement of each input's true robustness.

### Mathematical Foundations: Linear Classifiers

For a **binary linear classifier** `f(x) = wᵀx + b`, the minimal L2 perturbation to reach the decision boundary (where f(x)=0) is given by:

```
distance = |f(x₀)| / ‖w‖₂
```

The minimal perturbation is:

```
r* = -(f(x₀) / ‖w‖₂²) · w
```

This says: move in the direction of w (which points "uphill" in the classifier's landscape), by an amount proportional to how confident the model currently is `(|f(x₀)|)` and inversely proportional to the gradient magnitude `(‖w‖₂²)`. Unlike FGSM's sign operation that equalizes all pixel changes, this **preserves relative magnitudes** — a pixel with weight 10 will change ten times as much as a pixel with weight 1, because it has ten times the influence on classification.

### Extending to Deep Networks via Iterative Linearization

Deep networks have curved, complex decision boundaries — not flat hyperplanes. DeepFool handles this through **iterative linearization**: at each position, compute a local linear approximation of the boundary, take the shortest step to that approximation, then repeat from the new position. Think of navigating a curved mountain in dense fog — at each spot, look at the immediate terrain, take a small step in the best local direction, reassess from the new position, and repeat. Each iteration provides a fresh linear approximation, gradually building a path that follows the boundary's curvature until the classification changes.

### The Overshoot Parameter

DeepFool includes an **overshoot parameter** (typically 0.02) that slightly overshoots the decision boundary: the actual step taken is `(1 + overshoot) × r_i`. This serves two purposes: the linearization is only locally accurate, so a small overshoot ensures we actually cross the true non-linear boundary rather than just touching the linear approximation. It also accelerates convergence by taking slightly larger steps. The 2% typical value keeps the perturbation increase negligible while improving reliability.

### Comparison: DeepFool vs. Iterative FGSM

Both methods iterate and compute gradients multiple times, but they differ fundamentally. **Iterative FGSM** takes uniform steps in the gradient sign direction — each iteration applies the same step size across all pixels, and the sign operation discards magnitude information entirely (gradient of 0.001 treated identically to gradient of 100). **DeepFool** preserves gradient magnitudes to compute geometrically minimal steps — each iteration identifies the closest decision boundary among all classes and takes exactly the step needed to reach that specific boundary. Features with large gradients change more, small gradients change minimally. This adaptive weighting emerges automatically from the math rather than being imposed through hyperparameter selection.

---

## Section 15: DeepFool Theory and Formulation

### Multi-Class Formulation

Real classifiers handle many classes simultaneously. From any input, multiple decision boundaries exist — each separating the current predicted class from a different alternative. DeepFool's answer: target whichever boundary is **closest** at each iteration. Rather than committing to a predetermined target class, DeepFool dynamically identifies the nearest boundary and takes the minimal step toward it.

For input `x` classified as class `k̂(x) = argmax_k f_k(x)`, DeepFool defines for each alternative class `k ≠ k̂(x)`:

```
w_k = ∇f_k(x_i) - ∇f_{k̂}(x_i)     # gradient difference
f'_k = f_k(x_i) - f_{k̂}(x_i)       # score gap (negative if losing)
```

The closest boundary is found by selecting the class with minimum distance ratio:

```
l = argmin_{k ≠ k̂} |f'_k| / ‖w_k‖₂
```

The minimal step toward that boundary is:

```
r_i = (|f'_l| / ‖w_l‖₂²) · w_l
```

Then `x_{i+1} = x_i + r_i` before re-linearizing for the next iteration.

### Robustness Evaluation with DeepFool

DeepFool provides a quantitative robustness measure `ρ_adv`: the average minimal perturbation needed to fool the classifier across a dataset:

```
ρ_adv = (1/|D|) · Σ_{x ∈ D} ‖r(x)‖₂ / ‖x‖₂
```

This computes the average **relative perturbation size**: for each image, the perturbation norm is divided by the image norm to get a percentage-like measure. A model with `ρ_adv = 0.02` requires perturbations that are 2% the size of the original input on average to be fooled. A model with `ρ_adv = 0.10` requires 10% perturbations — five times more robust. This enables fair comparison across different models and architectures.

### L∞ Variant and Extensions

The core DeepFool algorithm uses the L2 norm, but it extends to L∞ by changing the distance computation. For L∞ DeepFool, the denominator switches from `‖w_k‖₂` to `‖w_k‖₁`, and the perturbation direction uses element-wise sign rather than normalized gradient. DeepFool's minimal perturbation principle also inspired **universal adversarial perturbations** — single perturbations that fool a network on most inputs from a dataset. The algorithm iteratively applies DeepFool to different training examples and accumulates perturbations, demonstrating that deep neural networks have systematic vulnerabilities consistent across different inputs.

---

## Section 16: Building and Training the Target Model for DeepFool

### Environment Setup

The setup is identical to the FGSM Setup section. The key difference is importing `MNISTClassifierWithDropout` instead of `SimpleCNN`, plus caching utilities.

```python
from htb_ai_library import (
    set_reproducibility, MNISTClassifierWithDropout, get_mnist_loaders,
    train_model, evaluate_accuracy, save_model, load_model, analyze_model_confidence,
    HTB_GREEN, NODE_BLACK, HACKER_GREY, WHITE, AZURE, NUGGET_YELLOW, MALWARE_RED, VIVID_PURPLE, AQUAMARINE
)

set_reproducibility(1337)
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
```

### Why Dropout Matters for DeepFool

DeepFool requires a different architecture than FGSM. An overfitted model might show extremely confident but fragile predictions, with decision boundaries tightly wrapped around memorized data points. **Dropout** forces the network to learn redundant representations where multiple feature combinations indicate the same digit, creating more realistic boundaries that better reflect genuine uncertainty. The architecture: two conv blocks (1→32 and 32→64 channels with 25% dropout) → MaxPool to 7×7 → flatten to 3136 → FC(3136→128) with 50% dropout → FC(128→10). During DeepFool's gradient computation, `model.eval()` automatically disables dropout for stable gradients.

### File Path Configuration and Smart Caching

```python
import os
model_path = 'output/mnist_model.pth'
os.makedirs('output', exist_ok=True)
```

The `output/` directory separates generated artifacts from source code. The caching system checks for existing models, validates their accuracy, and only retrains when necessary — saving significant time during iterative development.

### Loading, Validating, and Training

```python
# Step 1: Try to load a cached model
if os.path.exists(model_path):
    model_data = load_model(model_path)
    model = model_data['model'].to(device)
    model.eval()
    _, test_loader = get_mnist_loaders(batch_size=100, normalize=True)
    accuracy = evaluate_accuracy(model, test_loader, device)
    print(f"Cached model accuracy: {accuracy:.2f}%")
    if accuracy < 90.0:
        print("Accuracy below threshold, retraining required")
        model = None
else:
    model = None

# Step 2: Train if no valid cached model
if model is None:
    print("Training new model...")
    train_loader, test_loader = get_mnist_loaders(batch_size=64, normalize=True)
    model = MNISTClassifierWithDropout().to(device)
    model = train_model(model, train_loader, test_loader, epochs=5, device=device)

    # Step 3: Save trained model with metadata
    accuracy = evaluate_accuracy(model, test_loader, device)
    save_model({
        'model': model,
        'architecture': 'MNISTClassifierWithDropout',
        'accuracy': accuracy,
        'training_config': {'epochs': 5, 'batch_size': 64, 'device': str(device)}
    }, model_path)

# Expected training output:
# Epoch 1/5 - Loss: 0.2134 - Accuracy: 93.54%
# Epoch 5/5 - Loss: 0.0421 - Accuracy: 98.76%
# Test Accuracy: 98.95%
```

Five epochs are needed (instead of 1 for FGSM) because dropout regularization slows convergence — each training iteration randomly disables different neurons, preventing the network from memorizing efficient feature paths. The `save_model` dict stores the model object, architecture name, accuracy, and training configuration for reproducibility. Models typically achieve 98.5–99.0% accuracy (9,850–9,900 correct out of 10,000 test images).

---

## Section 17: DeepFool Implementation

### Function Definition and Initialization

```python
from typing import Tuple

def deepfool(image, net, num_classes=10, overshoot=0.02, max_iter=50, device='cuda'):
    image = image.to(device)
    net = net.to(device)

    # Get initial class scores and sort by confidence (descending)
    f_image = net(image).data.cpu().numpy().flatten()
    I = f_image.argsort()[::-1]    # class indices sorted by score, highest first
    label = I[0]                    # original predicted class

    # Initialize working variables
    input_shape = image.shape
    pert_image = image.clone()
    r_tot = torch.zeros(input_shape).to(device)    # accumulated perturbation
    loop_i = 0
```

`argsort()[::-1]` sorts class indices by their scores in descending order. `label = I[0]` is the predicted class (highest score). `r_tot` accumulates all incremental perturbations across iterations, starting at zero. `pert_image` is the current (gradually modified) image starting as a copy of the original.

### Main Iteration Loop

```python
    while loop_i < max_iter:
        x = pert_image.clone().requires_grad_(True)
        fs = net(x)
        k_i = fs.data.cpu().numpy().flatten().argsort()[::-1][0]

        # Stop when prediction changes (attack succeeded)
        if k_i != label:
            break

        pert = float('inf')    # best distance so far (starts at infinity)
        w = None               # best gradient direction so far
```

Each iteration re-evaluates the classifier at the current perturbed image `x`. `requires_grad_(True)` enables gradient computation for the input. If the current prediction `k_i` differs from the original `label`, the attack succeeded — we break out. `pert = float('inf')` ensures any valid candidate boundary distance will be smaller.

### Finding the Closest Decision Boundary

```python
        # Search through all candidate classes
        for k in range(1, num_classes):
            if I[k] == label:
                continue    # skip the current predicted class

            # Gradient for candidate class k
            if x.grad is not None:
                x.grad.zero_()
            fs[0, I[k]].backward(retain_graph=True)
            grad_k = x.grad.data.clone()

            # Gradient for original class
            if x.grad is not None:
                x.grad.zero_()
            fs[0, label].backward(retain_graph=True)
            grad_label = x.grad.data.clone()

            # Combined direction and distance to this boundary
            w_k = grad_k - grad_label          # direction that moves from label toward class k
            f_k = (fs[0, I[k]] - fs[0, label]).data.cpu().numpy()    # score gap
            pert_k = abs(f_k) / (torch.norm(w_k.flatten()) + 1e-10)  # normalized distance

            # Track minimum distance across all candidate classes
            if pert_k < pert:
                pert = pert_k
                w = w_k
```

`retain_graph=True` preserves the computation graph for multiple backward passes (one per candidate class). `w_k = grad_k - grad_label` points in a direction that simultaneously increases candidate class score and decreases original class score. `pert_k = |f_k| / ‖w_k‖₂` estimates the distance to the linearized boundary: smaller score gap = closer boundary, larger gradient magnitude = boundary closer in that direction.

### Computing and Applying the Perturbation

```python
        # Minimal step toward the closest boundary
        r_i = (pert + 1e-4) * w / (torch.norm(w.flatten()) + 1e-10)
        r_tot = r_tot + r_i

        # Apply with overshoot to ensure we cross the true non-linear boundary
        pert_image = image + (1 + overshoot) * r_tot
        loop_i += 1

    return r_tot, loop_i, label, k_i, pert_image
```

`r_i` normalizes direction `w` to unit length and scales by distance `pert`. The `1e-4` prevents numerical issues when distance is very small. `r_tot += r_i` accumulates all perturbations across iterations. Multiplying by `(1 + overshoot)` (typically 1.02) ensures the algorithm actually crosses the true non-linear boundary, not just the linear approximation. Returns: total perturbation `r_tot`, number of iterations `loop_i`, original label, adversarial label `k_i`, and final perturbed image.

---

## Section 18: Demonstrating DeepFool

### Single-Image Attack Pipeline

```python
# Step 1: Load the trained model
model_path = 'output/mnist_model.pth'
if os.path.exists(model_path):
    model_data = load_model(model_path)
    model = model_data['model'].to(device)
    model.eval()
else:
    raise FileNotFoundError("Model not found. Run Section 16 first.")

# Step 2: Get a single test sample
_, test_loader = get_mnist_loaders(batch_size=1, normalize=True)
dataiter = iter(test_loader)
image, true_label = next(dataiter)
image = image.to(device)
print(f"True label: {true_label.item()}")

# Step 3: Get baseline prediction and confidence
with torch.no_grad():
    original_output = model(image)
    original_pred = original_output.argmax(dim=1).item()
    original_confidence = F.softmax(original_output, dim=1).max().item()
print(f"Original: class {original_pred} (confidence: {original_confidence:.3f})")

# Step 4: Execute DeepFool attack
r_total, iterations, orig_label, pert_label, pert_image = deepfool(
    image, model, num_classes=10, overshoot=0.02, max_iter=50, device=device
)
print(f"Attack: {orig_label} -> {pert_label} in {iterations} iterations")
```

`model.eval()` is essential: it disables dropout for stable gradients during DeepFool's iterative refinement. `batch_size=1` enables per-sample tracking of exact iteration counts. Well-separated MNIST classes typically converge in 1–3 iterations; ambiguous classes may need 4–6. The 2% overshoot compensates for boundary curvature the linear approximation cannot capture.

### Quantifying Attack Efficiency

```python
# Step 5: Compute perturbation metrics
perturbation_norm_l2 = torch.norm(r_total).item()
perturbation_norm_linf = torch.abs(r_total).max().item()
relative_perturbation = perturbation_norm_l2 / torch.norm(image).item()

with torch.no_grad():
    adv_output = model(pert_image)
    adv_confidence = F.softmax(adv_output, dim=1).max().item()

print(f"L2 norm: {perturbation_norm_l2:.4f}")
print(f"L∞ norm: {perturbation_norm_linf:.4f}")
print(f"Relative perturbation: {relative_perturbation:.2%}")
print(f"Original confidence: {original_confidence:.3f}")
print(f"Adversarial confidence: {adv_confidence:.3f}")
```

The L2/L∞ ratio reveals spatial concentration. If the perturbation were perfectly uniform across all 784 pixels, L∞ ≈ L2/28 ≈ 0.276. An observed L∞ of 1.58 is 6x larger than this uniform baseline, proving DeepFool concentrates modifications on strategically important pixels. `relative_perturbation = ‖r‖₂ / ‖x‖₂` normalizes by image magnitude for fair comparison across images of different brightness. The adversarial confidence settling near 0.4 confirms minimal boundary crossing with slight overshoot.

### Four-Panel Visualization

```python
# Step 6: Visualize the attack
original_img = mnist_denormalize(image.squeeze()).cpu().numpy()
adversarial_img = mnist_denormalize(pert_image.squeeze()).cpu().numpy()
perturbation = r_total.cpu().squeeze().numpy()

# Normalize perturbation to [0,1] for display (amplify minimal changes)
pert_display = perturbation - perturbation.min()
if pert_display.max() > 0:
    pert_display = pert_display / pert_display.max()

fig, axes = plt.subplots(1, 4, figsize=(15, 5))
fig.patch.set_facecolor(NODE_BLACK)

# Panel 1: Original image
axes[0].imshow(original_img, cmap='gray', vmin=0, vmax=1)
axes[0].set_title(f"Original\nClass: {original_pred}", color=HTB_GREEN, fontweight='bold')

# Panel 2: Amplified perturbation (inferno colormap: dark purple=low, yellow=high)
axes[1].imshow(pert_display, cmap='inferno')
axes[1].set_title("Perturbation\n(amplified)", color=NUGGET_YELLOW, fontweight='bold')

# Panel 3: Perturbation magnitude heatmap
im = axes[2].imshow(np.abs(perturbation), cmap='viridis')
axes[2].set_title(f"Magnitude\nL2: {perturbation_norm_l2:.4f}", color=AZURE, fontweight='bold')
plt.colorbar(im, ax=axes[2], fraction=0.046, pad=0.04)

# Panel 4: Adversarial result
axes[3].imshow(adversarial_img, cmap='gray', vmin=0, vmax=1)
axes[3].set_title(f"Adversarial\nClass: {pert_label}", color=HTB_GREEN, fontweight='bold')

plt.tight_layout()
plt.show()
```

The normalization `(pert - min) / (max - min)` rescales tiny perturbations to [0,1] for visibility. The inferno colormap maps 0 (minimal change) to dark purple and 1 (maximum change) to bright yellow. Panel 2 shows where DeepFool concentrates modifications — bright yellow clusters at class-discriminative features; dark purple in irrelevant background regions. This asymmetry confirms iterative gradient refinement successfully identified which pixels matter most.

---

## Section 19: Batch Attack Generation

### Systematic Sample Processing

Single-sample attacks provide proof of concept, but statistical patterns require population-level analysis. Processing multiple samples reveals: which digit classes are most vulnerable, how iteration counts vary across samples, and what the typical perturbation magnitude looks like. DeepFool processes each sample independently because one may converge in 2 iterations while another needs 6 — batching would waste computation.

### Batch Attack Pipeline

```python
num_examples = 20
_, test_loader = get_mnist_loaders(batch_size=1, normalize=True)
model.eval()

results = []
success_count = 0

# Process each sample independently
for idx, (data, target) in enumerate(test_loader):
    if idx >= num_examples:
        break

    data = data.to(device)

    # Execute DeepFool attack on this sample
    r, iterations, orig_label, adv_label, pert_image = deepfool(
        data, model, num_classes=10, overshoot=0.02, max_iter=50, device=device
    )

    success = (orig_label != adv_label)
    if success:
        success_count += 1

    # Store all metrics for this sample
    results.append({
        'original_image': data.cpu(),
        'perturbation': r.cpu(),
        'perturbed_image': pert_image.cpu(),
        'original_label': orig_label,
        'adversarial_label': adv_label,
        'iterations': iterations,
        'true_label': target.item(),
        'l2_norm': torch.norm(r.cpu()).item(),
        'success': success
    })

    print(f"  Example {idx+1}: True={target.item()}, Orig={orig_label}, "
          f"Adv={adv_label}, Iter={iterations}, L2={torch.norm(r.cpu()).item():.4f}")

print(f"\nAttack Success Rate: {success_count}/{num_examples} ({100*success_count/num_examples:.1f}%)")
print(f"Average L2 norm: {np.mean([r['l2_norm'] for r in results]):.4f}")
print(f"Average iterations: {np.mean([r['iterations'] for r in results]):.1f}")

# Expected output:
# Attack Success Rate: 20/20 (100.0%)
# Average L2 norm: 4.8661
# Average iterations: 3.0
```

Tensors are moved to CPU immediately with `.cpu()` to free GPU memory. `torch.norm(r.cpu()).item()` converts from tensor to Python float. 100% success rate confirms DeepFool's reliability on MNIST. Individual L2 norms range from ~0.60 (structurally similar digit pairs like 3→5) to ~7.72 (very different pairs like 7→2), a 13x range demonstrating how decision boundary geometry varies across the input space.

### Grid Visualization

```python
def visualize_attack_grid(results, save_dir='output'):
    num_examples = min(10, len(results))
    fig, axes = plt.subplots(4, 5, figsize=(15, 12))
    fig.patch.set_facecolor(NODE_BLACK)

    for idx in range(num_examples):
        row = idx // 5    # which pair of rows (0 or 1)
        col = idx % 5     # which column (0-4)

        # Top row of the pair: original image
        ax_orig = axes[row * 2, col]
        img = mnist_denormalize(results[idx]['original_image'].squeeze()).numpy()
        ax_orig.imshow(img, cmap='gray', vmin=0, vmax=1)
        ax_orig.set_title(f"Original: {results[idx]['original_label']}", color=HTB_GREEN, fontsize=10)
        ax_orig.axis('off')

        # Bottom row of the pair: adversarial image
        ax_adv = axes[row * 2 + 1, col]
        adv_img = mnist_denormalize(results[idx]['perturbed_image'].squeeze()).numpy()
        ax_adv.imshow(adv_img, cmap='gray', vmin=0, vmax=1)
        title_color = MALWARE_RED if results[idx]['success'] else HACKER_GREY
        ax_adv.set_title(f"Adversarial: {results[idx]['adversarial_label']}", color=title_color, fontsize=10)
        ax_adv.axis('off')

    plt.tight_layout()
    plt.savefig(f'{save_dir}/deepfool_examples.png', facecolor=NODE_BLACK, dpi=150, bbox_inches='tight')
    plt.close()

visualize_attack_grid(results, save_dir='output')
```

`row = idx // 5` and `col = idx % 5` use integer division and modulo for grid positioning. Each original image is at `axes[row*2, col]` and its adversarial counterpart directly below at `axes[row*2+1, col]`. Red titles mark successful flips; grey marks failures.

---

## Section 20: Perturbation Analysis and Metrics

### Spatial Perturbation Analysis

```python
def visualize_perturbation_analysis(results, save_dir='output'):
    fig, axes = plt.subplots(2, 3, figsize=(15, 10))
    fig.patch.set_facecolor(NODE_BLACK)

    successful_attacks = [r for r in results if r['success']][:3]

    for idx, result in enumerate(successful_attacks):
        pert = result['perturbation'].squeeze().numpy()
        vmax = np.abs(pert).max() or 1e-6

        # Top row: Raw perturbation heatmap (RdBu_r: red=positive, blue=negative, white=no change)
        ax_top = axes[0, idx]
        im_top = ax_top.imshow(pert, cmap='RdBu_r', vmin=-vmax, vmax=vmax)
        ax_top.set_title(f'Perturbation (L2={result["l2_norm"]:.3f})', color=HTB_GREEN, fontsize=10)
        plt.colorbar(im_top, ax=ax_top, fraction=0.046, pad=0.04)

        # Bottom row: Amplified difference (10x magnification)
        ax_bottom = axes[1, idx]
        orig_img = result['original_image'].squeeze().numpy()
        adv_img = result['perturbed_image'].squeeze().detach().numpy()
        diff_amplified = (adv_img - orig_img) * 10
        im_bottom = ax_bottom.imshow(diff_amplified, cmap='RdBu_r', vmin=-0.5, vmax=0.5)
        ax_bottom.set_title(f"{result['original_label']} -> {result['adversarial_label']} ({result['iterations']} iters)",
                           color=NUGGET_YELLOW, fontsize=10)
        plt.colorbar(im_bottom, ax=ax_bottom, fraction=0.046, pad=0.04)

    plt.tight_layout()
    plt.savefig(f'{save_dir}/deepfool_perturbations.png', facecolor=NODE_BLACK, dpi=150, bbox_inches='tight')
    plt.close()

visualize_perturbation_analysis(results, save_dir='output')
```

RdBu_r colormap: red = positive perturbation (brightening pixels), blue = negative (darkening), white = no change. The symmetric range `vmin=-vmax, vmax=vmax` centers white at zero. The 10x amplification `(adv_img - orig_img) * 10` makes tiny changes of 0.01–0.05 visible. Heatmaps show concentrated modifications at digit boundaries and discriminative features, with background remaining white (unchanged), confirming DeepFool's targeted strategy.

### Statistical Distribution Analysis

```python
fig, axes = plt.subplots(1, 3, figsize=(15, 5))
fig.patch.set_facecolor(NODE_BLACK)

# Panel 1: L2 Norm Distribution histogram
l2_norms = [r['l2_norm'] for r in results]
axes[0].hist(l2_norms, bins=15, color=HTB_GREEN, alpha=0.7, edgecolor=HACKER_GREY)
axes[0].set_title('Perturbation Magnitude Distribution', color=HTB_GREEN)

# Panel 2: Iteration Count Distribution
iterations_list = [r['iterations'] for r in results]
axes[1].hist(iterations_list, bins=range(1, max(iterations_list)+2), color=AZURE, alpha=0.7, edgecolor=HACKER_GREY)
axes[1].set_title('Iterations Required', color=HTB_GREEN)

# Panel 3: Per-Class Success Rates
class_success = {}
for r in results:
    orig = r['original_label']
    if orig not in class_success:
        class_success[orig] = {'total': 0, 'success': 0}
    class_success[orig]['total'] += 1
    if r['success']:
        class_success[orig]['success'] += 1

classes = sorted(class_success.keys())
success_rates = [class_success[c]['success'] / class_success[c]['total'] * 100 for c in classes]
axes[2].bar(classes, success_rates, color=NUGGET_YELLOW, alpha=0.7, edgecolor=HACKER_GREY)
axes[2].set_title('Attack Success by Class', color=HTB_GREEN)

plt.tight_layout()
plt.savefig('output/deepfool_metrics.png', facecolor=NODE_BLACK, dpi=150, bbox_inches='tight')
plt.close()
```

The L2 histogram shows most perturbations clustering around 4–5, with minimum ~0.60 (easy transitions like 3→5) and maximum ~7.72 (hard transitions like 7→2). The iteration histogram uses `range(1, max+2)` to give each count its own bar. The per-class bar chart uses `class_success[c]['success'] / class_success[c]['total'] * 100` with a two-counter dictionary.

### Aggregate Attack Summary

```python
def print_summary_statistics(results):
    successful_attacks = [r for r in results if r['success']]
    avg_l2 = np.mean([r['l2_norm'] for r in successful_attacks])
    avg_iterations = np.mean([r['iterations'] for r in successful_attacks])

    print(f"Success Rate: {len(successful_attacks)}/{len(results)} ({100*len(successful_attacks)/len(results):.1f}%)")
    print(f"Average L2 Norm: {avg_l2:.4f}")
    print(f"Average Iterations: {avg_iterations:.1f}")

    # Find most common class transitions
    transitions = {}
    for r in successful_attacks:
        key = f"{r['original_label']}->{r['adversarial_label']}"
        transitions[key] = transitions.get(key, 0) + 1

    print("Most Common Misclassifications:")
    for trans, count in sorted(transitions.items(), key=lambda x: x[1], reverse=True)[:5]:
        print(f"  {trans}: {count} times")

print_summary_statistics(results)

# Expected output:
# Success Rate: 20/20 (100.0%)
# Average L2 Norm: 4.8661
# Average Iterations: 3.0
# Most Common Misclassifications:
#   9->4: 3 times
#   7->2: 1 times
#   2->6: 1 times
```

`transitions.get(key, 0) + 1` safely increments counts (defaulting to 0 for new keys). `sorted(..., key=lambda x: x[1], reverse=True)[:5]` ranks by frequency descending and takes the top 5. The most common transition 9→4 (3 times) makes geometric sense: both digits have similar upper loops and vertical strokes, so their decision boundary is closer than between very different shapes.

---

## Section 21: DeepFool Challenge

### Objective

Craft a **targeted** adversarial example using a DeepFool-style iterative attack. Unlike untargeted attacks, you must make the classifier predict a **specific target class**, while minimizing perturbation under an L2 distance constraint: `‖x_adv − x‖₂ ≤ threshold`. The L2 norm is measured in [0,1] **pixel space** (not normalized space). All three conditions must hold: predicted class equals the target, L2 distance is at most the threshold, and all pixel values remain in [0,1].

### Step-by-Step: Setting Up and Fetching the Challenge

```bash
# Step 1: Export your instance URL
export BASE_URL="http://instance_ip:port"

# Step 2: Check server status and configuration
curl -s "$BASE_URL/health" | jq
# Returns: {"status": "ok", "l2_threshold": 0.75, "index": 95, "target": 6}

# Step 3: Fetch the challenge image and parameters
curl -s "$BASE_URL/challenge" | jq
# Returns: {"sample_index": 95, "label": 4, "target": 6, "l2_threshold": 0.75, "image_b64": "..."}

# Step 4: Download model weights for local gradient computation
curl -s -o deepfool_weights.pth "$BASE_URL/weights"
```

The `/health` endpoint shows the l2_threshold, sample index, and target class. The `/challenge` endpoint gives the baseline image (base64 PNG), its true label, the specific target class you must achieve, and the L2 threshold constraint.

### Step-by-Step: Loading the Model and Challenge

```python
import os, io, base64, numpy as np, requests, torch, torch.nn as nn
from PIL import Image

BASE_URL = os.getenv("BASE_URL", "http://127.0.0.1:8000")
MNIST_MEAN, MNIST_STD = 0.1307, 0.3081

# Helper functions (same as FGSM challenge)
def x01_from_b64_png(b64):
    raw = base64.b64decode(b64)
    img = Image.open(io.BytesIO(raw)).convert("L")
    return np.clip(np.asarray(img, dtype=np.float32) / 255.0, 0.0, 1.0)

def b64_png_from_x01(x2d):
    x255 = np.clip((x2d * 255.0).round(), 0, 255).astype(np.uint8)
    img = Image.fromarray(x255, mode="L")
    buf = io.BytesIO()
    img.save(buf, format="PNG", optimize=True)
    return base64.b64encode(buf.getvalue()).decode("ascii")

def l2(a, b):    # Compute L2 distance (Euclidean distance) between two arrays
    return float(np.linalg.norm((a - b).ravel(), ord=2))

# Load challenge details
ch = requests.get(f"{BASE_URL}/challenge", timeout=10).json()
x = x01_from_b64_png(ch["image_b64"])    # (28, 28) array in [0,1]
lab = int(ch["label"])
tgt = int(ch["target"])                  # required prediction class
thr = float(ch["l2_threshold"])          # maximum L2 distance allowed

# Load model weights (same SimpleClassifier architecture as FGSM challenge)
model = SimpleClassifier().eval()
state = torch.load("deepfool_weights.pth", map_location=torch.device("cpu"))
model.load_state_dict(state)

x_tensor = torch.from_numpy(x[None, None, ...]).float()
logits = model(x_tensor)
local_pred = int(torch.argmax(logits, dim=1).item())
print(f"Local prediction: {local_pred}, Target: {tgt}, L2 threshold: {thr}")
```

### Step-by-Step: Running Targeted DeepFool and Submitting

```bash
# Step 1: Verify validation works by submitting the clean image (expect failure)
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image_b64": "<clean_image_b64>"}' | jq
# Expected: HTTP 400 "Wrong target: predicted 4, need 6"

# Step 2: Implement targeted DeepFool iteration
# (modify the deepfool function to target class `tgt` specifically)
# Key change: instead of targeting the closest boundary, repeatedly push toward the target class

# Step 3: Convert your adversarial numpy array to base64 PNG
# x_adv_b64 = b64_png_from_x01(adversarial_image_array)

# Step 4: Verify L2 distance is within threshold
# dist = l2(x, adversarial_image_array)
# print(f"L2 distance: {dist:.4f}, threshold: {thr}")  -- must be <= thr

# Step 5: Submit the adversarial image
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image_b64": "<adversarial_image_b64>"}' | jq
# Success: {"ok": true, "pred": 6, "target": 6, "l2": 0.68, "flag": "HTB{...}"}
# Failure: HTTP 400 with specific error ("L2 too large" or "Wrong target")
```

For targeted DeepFool, focus the search specifically on the target class at each iteration rather than finding the globally closest boundary. The constraint here is L2 (not L∞), so use the L2 variant of DeepFool where the perturbation direction is the normalized gradient (not the sign). Verify your L2 distance with `l2(x, adversarial_image_array) ≤ thr` before submitting.

---

## Section 22: Skills Assessment 1 — I-FGSM on CIFAR-10

### Objective

Craft a targeted adversarial example using **I-FGSM on CIFAR-10** color images. Transform a **dog image** (class 5) into one the classifier predicts as a **cat** (class 3), while staying within `‖x_adv − x‖∞ ≤ ε = 8/255 ≈ 0.0314` in pixel space. All conditions: predicted class = cat (3), max absolute pixel difference ≤ ε, all pixel values in [0,1] for a 32×32×3 RGB image.

### Step-by-Step: Setup and Challenge Fetch

```bash
# Step 1: Export instance URL and check health
export BASE_URL="http://instance_ip:port"
curl -s "$BASE_URL/health"

# Step 2: Fetch the challenge parameters
curl -s "$BASE_URL/challenge" | jq
# Returns: {"original_class": 5, "original_class_name": "dog",
#           "target_class": 3, "target_class_name": "cat",
#           "epsilon": 0.03137..., "normalization": {"mean": [...], "std": [...]}, ...}

# Step 3: Download model weights (~6.3MB CIFAR-10 model)
curl -s "$BASE_URL/model/weights" -o cifar10_model_best.pth
```

### Step-by-Step: Loading Model and Image

```python
import os, io, base64, requests
import torch, torch.nn.functional as F
import torchvision.transforms as transforms
from PIL import Image
import numpy as np

BASE_URL = os.getenv("BASE_URL", "http://127.0.0.1:8000")

# Helper: convert base64 PNG to (3, 32, 32) tensor in [0,1]
def tensor_from_b64_png(b64):
    raw = base64.b64decode(b64)
    img = Image.open(io.BytesIO(raw))
    return transforms.ToTensor()(img)    # returns (3, 32, 32) in [0,1]

# Helper: convert tensor back to base64 PNG
def b64_png_from_tensor(tensor):
    img_array = (tensor.permute(1, 2, 0).numpy() * 255).astype(np.uint8)
    img = Image.fromarray(img_array)
    buf = io.BytesIO()
    img.save(buf, format="PNG")
    return base64.b64encode(buf.getvalue()).decode("ascii")

# Load CIFAR-10 model (save architecture below as model.py first)
from model import load_model, NORMALIZATION_MEAN, NORMALIZATION_STD

device = "cuda" if torch.cuda.is_available() else "cpu"
model = load_model("cifar10_model_best.pth", device=device)

# Fetch challenge
ch = requests.get(f"{BASE_URL}/challenge", timeout=10).json()
x = tensor_from_b64_png(ch["image"])        # (3, 32, 32) tensor in [0,1]
orig_class = int(ch["original_class"])      # 5 (dog)
target_class = int(ch["target_class"])      # 3 (cat)
epsilon = float(ch["epsilon"])              # 8/255 ≈ 0.0314
mean = torch.tensor(ch["normalization"]["mean"]).view(3, 1, 1)
std = torch.tensor(ch["normalization"]["std"]).view(3, 1, 1)

# Verify clean prediction
x_norm = (x - mean) / std
with torch.no_grad():
    pred = model(x_norm.unsqueeze(0).to(device)).argmax(dim=1).item()
print(f"Original: class {orig_class}, Target: class {target_class}, Clean pred: {pred}")
```

### CIFAR-10 Model Architecture (save as model.py)

```python
import torch, torch.nn as nn

class CIFAR10CNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.conv1 = nn.Conv2d(3, 32, kernel_size=3, padding=1)
        self.bn1 = nn.BatchNorm2d(32)
        self.relu1 = nn.ReLU()
        self.pool1 = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.bn2 = nn.BatchNorm2d(64)
        self.relu2 = nn.ReLU()
        self.pool2 = nn.MaxPool2d(2, 2)
        self.fc1 = nn.Linear(64 * 8 * 8, 128)
        self.relu3 = nn.ReLU()
        self.dropout = nn.Dropout(0.5)
        self.fc2 = nn.Linear(128, num_classes)

    def forward(self, x):
        x = self.pool1(self.relu1(self.bn1(self.conv1(x))))
        x = self.pool2(self.relu2(self.bn2(self.conv2(x))))
        x = x.view(x.size(0), -1)    # flatten: 64*8*8 = 4096
        x = self.dropout(self.relu3(self.fc1(x)))
        return self.fc2(x)

def load_model(model_path, device='cuda'):
    model = CIFAR10CNN(num_classes=10)
    checkpoint = torch.load(model_path, map_location=device)
    if isinstance(checkpoint, dict) and 'model_state_dict' in checkpoint:
        model.load_state_dict(checkpoint['model_state_dict'])
    else:
        model.load_state_dict(checkpoint)
    return model.to(device).eval()

NORMALIZATION_MEAN = [0.4914, 0.4822, 0.4465]
NORMALIZATION_STD = [0.247, 0.2435, 0.2616]
CIFAR10_CLASSES = ['airplane', 'automobile', 'bird', 'cat', 'deer', 'dog', 'frog', 'horse', 'ship', 'truck']
```

### Step-by-Step: Running the Attack and Submitting

```bash
# Step 1: Implement targeted I-FGSM in pixel space
# Use fgsm_pixel_space or iterative_fgsm adapted for CIFAR-10 (3-channel, 32x32)
# Key parameters: epsilon=8/255, target_class=3, mean=[0.4914,0.4822,0.4465], std=[0.247,0.2435,0.2616]
# Use targeted=True and run ~40-100 iterations for better success

# Step 2: Verify L∞ constraint
# linf_dist = float(torch.abs(x_adv - x).max())
# print(f"L∞ distance: {linf_dist:.6f}, epsilon: {epsilon:.6f}")  -- must be <= epsilon

# Step 3: Verify model predicts target class locally
# with torch.no_grad():
#     pred = model(x_adv_norm.unsqueeze(0).to(device)).argmax(dim=1).item()
# print(f"Adversarial prediction: {pred} (need {target_class})")

# Step 4: Test submitting clean image (expect failure)
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image": "<clean_base64_png>"}' | jq
# Expected: {"success": false, "Target not achieved"}

# Step 5: Submit the adversarial image
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image": "<adversarial_base64_png>"}' | jq
# Success: {"success": true, "validation": {"linf_satisfied": true, "target_achieved": true}, "flag": "HTB{...}"}
```

Use `iterative_fgsm` adapted for CIFAR-10 (3-channel images, pixel-space epsilon). The model does NOT do internal normalization (unlike the MNIST `SimpleClassifier`), so normalize manually before the forward pass. The hint suggests up to 100 iterations — use more iterations with small alpha for better targeted success. The `/predict` endpoint helps validate intermediate candidates without consuming the submit attempt.

---

## Section 23: Skills Assessment 2 — DeepFool on CIFAR-10

### Objective

Implement the **DeepFool algorithm** to find a minimal L2 perturbation that causes misclassification on CIFAR-10. This is an **untargeted attack** — any misclassification of a horse image succeeds. The constraint is `‖x_adv_norm − x_norm‖₂ ≤ 3.5` where L2 is measured in **normalized space** (not pixel space). All conditions: predicted class differs from "horse" (7), L2 distance in normalized space ≤ 3.5, and all pixel values remain in [0,1].

### Step-by-Step: Setup and Challenge Fetch

```bash
# Step 1: Export instance URL and check health
export BASE_URL="http://instance_ip:port"
curl -s "$BASE_URL/health"

# Step 2: Fetch the challenge
curl -s "$BASE_URL/challenge" | jq
# Returns: {"original_class": 7, "original_class_name": "horse",
#           "l2_threshold": 3.5, "normalization": {...}, ...}

# Step 3: Download model weights (same CIFAR-10 CNN as Assessment 1)
curl -s "$BASE_URL/model/weights" -o cifar10_model_best.pth
```

### Step-by-Step: Loading and Verifying

```python
# (Use same helper functions and model.py as Assessment 1)

BASE_URL = os.getenv("BASE_URL", "http://127.0.0.1:8001")  # Note: different port

# Fetch challenge
ch = requests.get(f"{BASE_URL}/challenge", timeout=10).json()
x = tensor_from_b64_png(ch["image"])          # (3, 32, 32) in [0,1]
orig_class = int(ch["original_class"])        # 7 (horse)
l2_threshold = float(ch["l2_threshold"])      # 3.5
mean = ch["normalization"]["mean"]
std = ch["normalization"]["std"]
mean_t = torch.tensor(mean).view(3, 1, 1)
std_t = torch.tensor(std).view(3, 1, 1)

# Helper: compute L2 distance in normalized space
def l2_normalized_space(a, b, mean, std):
    mean_t = torch.tensor(mean).view(3, 1, 1)
    std_t = torch.tensor(std).view(3, 1, 1)
    a_norm = (a - mean_t) / std_t
    b_norm = (b - mean_t) / std_t
    return float(torch.norm(a_norm - b_norm))

# Verify clean prediction
x_norm = (x - mean_t) / std_t
with torch.no_grad():
    pred = model(x_norm.unsqueeze(0).to(device)).argmax(dim=1).item()
print(f"Original: class {orig_class}, Clean pred: {pred}, L2 threshold: {l2_threshold}")
```

### Step-by-Step: Running DeepFool and Submitting

```bash
# Step 1: Apply the deepfool() function to the CIFAR-10 image
# Key differences from MNIST DeepFool:
#   - Image shape: (1, 3, 32, 32) instead of (1, 1, 28, 28)
#   - num_classes=10 (same)
#   - The model does NOT do internal normalization -- normalize manually
#   - L2 constraint is in normalized space

# Step 2: After DeepFool, check constraint in normalized space
# dist_norm = l2_normalized_space(x, x_adv_pixel, mean, std)
# print(f"L2 (normalized space): {dist_norm:.4f}, threshold: {l2_threshold}")

# Step 3: Check that the adversarial image is misclassified
# with torch.no_grad():
#     x_adv_norm = (x_adv_pixel - mean_t) / std_t
#     pred_adv = model(x_adv_norm.unsqueeze(0).to(device)).argmax(dim=1).item()
# print(f"Adversarial prediction: {pred_adv} (must not be {orig_class})")

# Step 4: Verify by submitting clean image (expect failure)
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image": "<clean_base64_png>"}' | jq
# Expected: {"success": false, "Misclassification not achieved"}

# Step 5: Submit the adversarial image
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image": "<adversarial_base64_png>"}' | jq
# Success:
# {"success": true, "validation": {
#   "l2_norm": 0.9632, "l2_threshold": 3.5, "l2_satisfied": true,
#   "original_class": "horse", "adversarial_class": "truck", "misclassification": true},
#   "flag": "HTB{...}"}
```

The threshold of 3.5 in normalized space is quite generous — typical DeepFool perturbations on CIFAR-10 are well under this. The key adaptation from MNIST DeepFool is handling 3-channel (RGB) images and applying normalization externally before each forward pass (since `CIFAR10CNN.forward(x)` expects already-normalized inputs, unlike `SimpleClassifier` which normalizes internally). Use `max_iter=50` and `overshoot=0.02` as hinted. The `/predict` endpoint helps validate your pipeline before submitting.

---

*End of Notes — AI Evasion: First-Order Attacks (HTB Academy)*
