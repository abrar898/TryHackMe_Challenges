# AI Evasion – Sparsity Attacks: Complete Notes

---

## Section 1 – Introduction to Sparsity Evasion Attacks

Sparsity attacks are adversarial attacks that try to fool an AI model by changing as **few input features (pixels) as possible**. Instead of spreading tiny changes across every pixel like other attacks, sparsity attacks concentrate edits on a small number of high-impact pixels. The measure used to count changed pixels is called the **L0 pseudo-norm** — it simply counts how many coordinates differ between the original and the adversarial input.

These attacks are important because many real systems work with discrete inputs (like pixels that can only be black or white), where controlling the *number* of changes matters more than *how small* each change is. Sparse perturbations can also evade anomaly detectors that look for widespread noise, since most of the input stays completely unchanged.

**Two main sparsity attack methods covered in this module:**
- **ElasticNet (EAD):** Adds an L1 penalty to the optimization to naturally push many pixel changes toward zero.
- **JSMA (Jacobian-based Saliency Map Attack):** Uses a saliency map to pick the single best pixel (or pair) to change each iteration, enforcing an explicit pixel budget (L0 limit).

**Threat model:** Attackers work at inference time (when the model is making predictions). In a white-box setting they can compute gradients. In a black-box setting they estimate importance by querying the model. Inputs always stay in the valid `[0,1]` range after each update.

---

## Section 2 – ElasticNet (EAD)

ElasticNet Attacks to Deep Neural Networks (EAD) combine two types of regularization — **L1** and **L2** — into a single attack objective. This combination creates perturbations that are both *sparse* (changing few pixels) and *smooth* (making small, coordinated changes). The paper was published in 2018 by Chen, Zhang, Sharma, Yi, and Hsieh.

**Why combine L1 and L2?**
- **Pure L2 attacks** (like C&W) spread changes across all pixels — smooth but dense, every pixel changes a little.
- **Pure L1 attacks** encourage sparsity through a sharp corner at zero, but are hard to optimize because the gradient doesn't exist exactly at zero.
- **ElasticNet** resolves this: the L2 part provides smooth, stable gradients for optimization, while the L1 part forces many pixel changes to become exactly zero.

**The β parameter** controls the balance between sparsity and smoothness. A larger β = sparser perturbations. A smaller β = behaves more like pure L2.

**Single-norm limitations explained with a concrete example:**
- Case A: 100 pixels changed by 0.10 each → L1 = 10.0, squared L2 = 1.00
- Case B: 10 pixels changed by 0.316 each → L1 = 3.16, squared L2 ≈ 1.00
- Both have the same L2 distance, but the L1 term prefers Case B because fewer pixels are changed.

**Binary search for the trade-off constant c:**
- Large c → model is fooled easily but perturbations are large.
- Small c → perturbations are small but the attack might fail.
- Binary search automatically finds the **minimum c** that just barely achieves misclassification.

**How EAD differs from previous attacks:**
- FGSM: single step, fixed ε, no adaptation.
- DeepFool: finds minimum L2 perturbation geometrically, no sparsity.
- EAD: iterative optimization, mixed norms, binary search — more flexible and targeted.

---

## Section 3 – Environment Setup

Before running ElasticNet, the environment needs to be set up with the right libraries and a trained target model. ElasticNet needs significantly more computation than single-step attacks like FGSM because it runs hundreds of FISTA iterations inside multiple binary search steps.

**Step 1: Install the HTB AI Library**

```bash
pip install --upgrade git+https://github.com/PandaSt0rm/htb-ai-library
```

This library provides model architectures, MNIST data loaders, training loops, and visualization utilities so you can focus on attack mechanics.

**Step 2: Set up the Python environment**

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.utils.data import DataLoader
from torchvision import datasets, transforms
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.patches as patches
from pathlib import Path
import warnings
warnings.filterwarnings('ignore')

from htb_ai_library.utils import (
    set_reproducibility, save_model, load_model,
    HTB_GREEN, NODE_BLACK, HACKER_GREY, WHITE,
    AZURE, NUGGET_YELLOW, MALWARE_RED, VIVID_PURPLE, AQUAMARINE,
)
from htb_ai_library.data import get_mnist_loaders
from htb_ai_library.models import MNISTClassifierWithDropout
from htb_ai_library.training import train_model, evaluate_accuracy
from htb_ai_library.visualization import use_htb_style

use_htb_style()
set_reproducibility(1337)

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print(f"Using device: {device}")
```

`set_reproducibility(1337)` fixes random seeds so results are the same every run. `use_htb_style()` applies a consistent visual theme to all plots.

**Step 3: Load or train the target model**

```python
train_loader, test_loader = get_mnist_loaders(batch_size=128)

output_dir = Path("output")
output_dir.mkdir(exist_ok=True)

model_path = output_dir / "mnist_target.pth"
model = MNISTClassifierWithDropout(num_classes=10).to(device)

if model_path.exists():
    print(f"Loading existing model from {model_path}")
    model = load_model(model, model_path, device)
else:
    print("Training new model...")
    model = train_model(model, train_loader, test_loader, epochs=5, device=device)
    save_model(model, model_path)

accuracy = evaluate_accuracy(model, test_loader, device)
print(f"Test accuracy: {accuracy:.2f}%")
```

The checkpoint caching pattern checks if `mnist_target.pth` already exists. If yes, load saved weights instantly. If no, train for 5 epochs and save. This avoids wasting time re-training every run. The model achieves about **98.89% accuracy** after 5 epochs.

**Memory note:** A batch of 128 MNIST images (28×28 pixels, float32) takes about 392 KB just for input data. Adding gradients and activations brings total per-batch memory to roughly 50–80 MB.

---

## Section 4 – Proximal Operators and FISTA Framework

ElasticNet's mixed-norm objective (L1 + L2) cannot be solved with simple gradient descent because the **L1 norm has a sharp corner at zero** — the gradient doesn't exist there. This section introduces two mathematical tools that solve this problem: **Proximal Operators** and **FISTA** (Fast Iterative Shrinkage-Thresholding Algorithm).

**Why is L1 hard to optimize?**
The absolute value function `|x|` has no derivative at exactly `x = 0`. Naive gradient descent on `‖δ‖₁` makes values get smaller and smaller but never reach exactly zero. You get fake sparsity (many near-zero values) instead of true sparsity (exact zeros).

**Proximal operators** solve this by generalizing the idea of projection. Instead of asking "what's the closest point in a constraint set?", they ask "what point minimizes distance-from-here PLUS a penalty function?":

```
prox_{λh}(z) = argmin_x { (1/2)‖x - z‖₂² + λh(x) }
```

The first term keeps the answer close to `z`. The second term applies the penalty. For the L1 norm, the proximal operator has a beautiful closed-form solution called **soft thresholding**.

**Soft Thresholding (the L1 proximal operator):**

```
S_λ(z)_i = { z_i - λ   if z_i > λ
            { 0          if |z_i| ≤ λ
            { z_i + λ   if z_i < -λ
```

Three regions, three actions:
- Values with magnitude **below λ** → zeroed out completely (creates true sparsity)
- Positive values **above λ** → reduced by λ (shrunk toward zero from above)
- Negative values **below -λ** → increased by λ (shrunk toward zero from below)

Example with λ = 0.1 and z = [0.12, 0.08, -0.25]:
- 0.12 > 0.1 → becomes 0.02 (shrunk)
- 0.08 ≤ 0.1 → becomes 0.00 (zeroed)
- -0.25 < -0.1 → becomes -0.15 (shrunk)

**FISTA (Fast Iterative Shrinkage-Thresholding Algorithm):**
FISTA splits the problem into smooth parts (handled with gradient descent) and non-smooth parts (handled with proximal operators). It also uses **Nesterov momentum** to converge faster — O(1/k²) instead of O(1/k).

The FISTA update sequence:
1. Compute gradient at the look-ahead (momentum) point `y⁽ᵏ⁾`
2. Take a gradient step
3. Apply soft thresholding (proximal operator for L1)
4. Update momentum: `y⁽ᵏ⁺¹⁾ = x⁽ᵏ⁺¹⁾ + (tₖ-1)/(tₖ₊₁) * (x⁽ᵏ⁺¹⁾ - x⁽ᵏ⁾)`

In practice, the momentum coefficient is simplified to `k/(k+3)`, which is more stable.

**For ElasticNet specifically:**
- Smooth part `f`: adversarial loss (C&W margin) + L2 distance
- Non-smooth part `h`: L1 penalty (β * ‖x' - x‖₁)
- Soft thresholding is applied to the **perturbation**, not the image itself (we want sparse perturbations, not sparse images)

---

## Section 5 – Distance Metrics

ElasticNet needs three distance measurements to balance sparsity against smoothness during optimization. All three are computed per-image in a batch so each example can be handled independently.

**L1 distance** = total magnitude of all pixel changes added together
- If 100 pixels each change by 0.15 → L1 = 15.0
- If 50 pixels each change by 0.30 → L1 = 15.0 (same L1, fewer pixels)
- L1 doesn't care how many pixels changed, only total magnitude

**Squared L2 distance** = sum of squared pixel changes (Euclidean distance, squared)
- 100 pixels at 0.15 each → L2 = 100 × 0.15² = 2.25
- 50 pixels at 0.30 each → L2 = 50 × 0.30² = 4.50
- L2 penalizes concentrated large changes more heavily than distributed small ones

**Elastic-net distance** = L2² + β × L1 (combines both)
- When β = 0 → pure L2
- As β increases → more weight on L1 sparsity

```python
def compute_distances(adv_images, original_images, beta):
    l1_dist = torch.sum(torch.abs(adv_images - original_images), dim=(1, 2, 3))
    l2_dist = torch.sum((adv_images - original_images) ** 2, dim=(1, 2, 3))
    elastic_dist = l2_dist + beta * l1_dist
    return l1_dist, l2_dist, elastic_dist
```

`dim=(1, 2, 3)` collapses channel, height, and width while keeping the batch dimension. This gives one scalar per image, which is needed because binary search tunes constants individually for each example.

**Why keep per-example distances?**
Easy examples near the decision boundary need small trade-off constants (c = 0.01). Hard examples deep in the correct class need larger constants (c = 10). Without per-example metrics, you'd be forced to use one constant for everything, which over-perturbs easy cases or under-perturbs hard ones.

---

## Section 6 – Adversarial Loss

Distance metrics measure perturbation size, but they don't actually push the model toward misclassification. The **adversarial loss** provides the gradient signal that drives the attack to fool the classifier.

**The Carlini & Wagner (C&W) margin-based loss:**
```
f(x', y) = max( Z_y(x') - max_{j≠y} Z_j(x') + κ, 0 )
```

- `Z_y(x')` = logit (raw score) for the true class y
- `max_{j≠y} Z_j(x')` = highest logit among all other classes
- `κ` (confidence) = how much margin we require

**Walk-through example:**
- True class is digit 7, logit Z₇ = 2.8
- Best competitor is class 4, logit Z₄ = 1.2
- Margin = 2.8 - 1.2 + 0 = 1.6 → loss = 1.6 (attack not yet successful)
- Gradient pushes to reduce Z₇ and increase Z₄

Once the attack succeeds (e.g., Z₇ = 1.0, Z₂ = 1.5):
- Margin = 1.0 - 1.5 = -0.5 → max(-0.5, 0) = 0 (loss = 0, attack succeeded)
- Optimization shifts entirely to reducing distortion

```python
def compute_adversarial_loss(logits, labels_onehot, confidence, targeted=False):
    real = torch.sum(labels_onehot * logits, dim=1)
    other = torch.max((1 - labels_onehot) * logits - labels_onehot * 10000, dim=1)[0]
    if targeted:
        loss = torch.clamp(other - real + confidence, min=0)
    else:
        loss = torch.clamp(real - other + confidence, min=0)
    return loss
```

**How does one-hot encoding extract the true class logit?**
`labels_onehot * logits` zeros out all classes except the true one. For digit 7 with true label [0,0,0,0,0,0,0,1,0,0], only position 7 survives → `real = logit[7]`. The `-10000` trick on the true class position ensures it can never be selected as the "max other".

**Complete loss combining all components:**
```python
total_loss = const * adversarial_loss + l2_dist
```

The L1 term is **not** included here because FISTA's proximal operator handles it separately. Only the smooth parts (adversarial loss + L2) go through backpropagation.

---

## Section 7 – Implementing FISTA Components

This section translates the FISTA math into working code. Two key building blocks are implemented: **Nesterov momentum** for acceleration and **soft thresholding** for sparsity.

**Nesterov Momentum:**
```python
def compute_fista_momentum(iteration):
    return iteration / (iteration + 3.0)
```

Momentum grows from cautious to aggressive as iterations increase:
- Iteration 1 → momentum = 0.25 (conservative, exploring)
- Iteration 10 → momentum = 0.77 (building speed)
- Iteration 100 → momentum = 0.97 (strong acceleration)

The `+3.0` keeps the result below 1.0 (approaching 1 as k→∞), preventing momentum from overwhelming the gradient. This adaptive schedule is better than a fixed momentum coefficient because early iterations know nothing about the landscape and need small steps.

**Soft Thresholding (Shrinkage-Thresholding):**
```python
def apply_shrinkage_thresholding(y, original_images, threshold, clip_min=0.0, clip_max=1.0):
    diff = y - original_images
    shrink_positive = torch.clamp(y - threshold, min=clip_min, max=clip_max)
    shrink_negative = torch.clamp(y + threshold, min=clip_min, max=clip_max)

    cond_positive = (diff > threshold).float()
    cond_zero = (torch.abs(diff) <= threshold).float()
    cond_negative = (diff < -threshold).float()

    result = (cond_positive * shrink_positive
              + cond_zero * original_images
              + cond_negative * shrink_negative)
    return result
```

**To verify soft thresholding behavior (run this code):**
```python
perturbation_pattern = torch.tensor([
    [-0.15, -0.08, -0.02,  0.03,  0.12],
    [-0.10, -0.05,  0.00,  0.06,  0.18],
    [-0.05,  0.00,  0.05,  0.10,  0.20],
    [ 0.00,  0.05,  0.08,  0.15,  0.25],
    [ 0.05,  0.08,  0.12,  0.20,  0.30]
], device=device)

original_values = torch.ones_like(perturbation_pattern) * 0.5
perturbed_values = original_values + perturbation_pattern

beta_test = 0.1
thresholded = apply_shrinkage_thresholding(
    perturbed_values, original_values, beta_test, clip_min=0.0, clip_max=1.0
)
resulting_perturbation = thresholded - original_values
print(resulting_perturbation.cpu().numpy())
```

**Expected result:** Before thresholding: 22 of 25 elements non-zero (88% dense). After applying β=0.1: only 9 survive (64% sparse). Values below 0.10 in magnitude get zeroed; others get shrunk by exactly 0.10.

---

## Section 8 – Loss Gradients and Optimization

To optimize adversarial images with FISTA, we need gradients with respect to **input pixels**, not model weights. We freeze the network and enable gradients only on adversarial images. This tells `loss.backward()` to compute how each pixel affects the total loss.

**Adversarial gradients flow through the entire network** via the chain rule — through fully connected layers, ReLU activations (piecewise linear, gradient blocked when inactive), and convolutions (spreading gradients spatially across receptive fields). FISTA treats this as a black box: give an input, receive a loss value and gradient.

**Gradient evolution during a typical attack:**
- Early iterations (1–100): Large adversarial gradients (5–20 per pixel) dominate small L2 gradients (0.1–0.5). Attack is bold and aggressive.
- Middle iterations (100–500): Adversarial loss starts to saturate, L2 pulls back excessive perturbations. Both contribute 1–5 per pixel.
- Late iterations (500–1000): Adversarial loss fully saturated at 0. Only L2 minimization remains (0.1–1.0 per pixel). Distortion is being cleaned up.

**Effect of c on individual pixel gradients (local balance example):**
- Pixel perturbation δ = 0.15 → L2 gradient = 2 × 0.15 = 0.30 (pulling toward original)
- Adversarial gradient at that pixel = -2.5 (pushing toward misclassification)
- With c = 0.1: net gradient = 0.30 - 0.25 = +0.05 (slight pull toward original)
- With c = 1.0: net gradient = 0.30 - 2.50 = -2.20 (strong push toward misclassification)

**Convergence:** FISTA's O(1/k²) guarantee applies when f has Lipschitz-continuous gradients and h is convex. Neural networks violate convexity, but FISTA still works well empirically, finding good local minima. The implementation uses a fixed maximum iteration count (e.g., 1000) with periodic success checks to avoid wasted computation.

---

## Section 9 – Complete FISTA Iteration and Binary Search

This section combines all FISTA components into a single iteration function, then wraps it in binary search for adaptive trade-off constant tuning.

**One complete FISTA step:**
```python
def fista_step(adv_images, y_momentum, original_images, labels_onehot,
               const, model, beta, learning_rate, confidence, iteration,
               targeted=False, clip_min=0.0, clip_max=1.0):
    # Break old graph, start fresh
    y_momentum = y_momentum.detach().requires_grad_(True)

    # Compute loss at look-ahead point (not current point)
    total_loss, adversarial_loss, distances = compute_total_loss(
        y_momentum, original_images, labels_onehot, const, model, beta, confidence, targeted)

    total_loss_summed = total_loss.sum()
    total_loss_summed.backward()
    grad = y_momentum.grad

    # Gradient step
    y_new = y_momentum - learning_rate * grad

    # Apply proximal operator (soft thresholding for L1)
    adv_new = apply_shrinkage_thresholding(y_new, original_images, learning_rate * beta, clip_min, clip_max)

    # Momentum extrapolation
    momentum_coef = compute_fista_momentum(iteration)
    y_new_momentum = adv_new + momentum_coef * (adv_new - adv_images)

    return adv_new, y_new_momentum, total_loss_summed.item(), distances
```

**Why `detach()` then `requires_grad_(True)`?** `detach()` breaks the existing computational graph so gradients don't accumulate incorrectly across iterations. `requires_grad_(True)` starts a fresh graph for this iteration only.

**Why evaluate loss at `y_momentum` not `adv_images`?** This look-ahead evaluation is what makes FISTA faster than standard proximal gradient. Computing gradients at the extrapolated point better exploits momentum in consistent directions.

**Checking attack success:**
```python
def check_attack_success(adv_images, labels, model, targeted=False):
    with torch.no_grad():
        outputs = model(adv_images)
        predictions = outputs.argmax(dim=1)
        if targeted:
            success = predictions.eq(labels)
        else:
            success = predictions.ne(labels)
    return success
```

**Updating binary search bounds:**
```python
def update_binary_search_bounds(lower_bound, upper_bound, const, success_mask):
    for i in range(len(success_mask)):
        if success_mask[i]:
            # Success: try smaller c (tighten upper bound)
            upper_bound[i] = min(upper_bound[i], const[i])
            if upper_bound[i] < 1e10:
                const[i] = (lower_bound[i] + upper_bound[i]) / 2
        else:
            # Failure: need larger c (raise lower bound)
            lower_bound[i] = max(lower_bound[i], const[i])
            if upper_bound[i] < 1e10:
                const[i] = (lower_bound[i] + upper_bound[i]) / 2
            else:
                const[i] *= 10  # Exponential growth until first success
    return lower_bound, upper_bound, const
```

Binary search starts with `lower = 0`, `upper = 1e10`, `c = 0.001`. When a success occurs, exponential growth phase ends and bisection takes over, halving the search interval each step. Each binary search step roughly halves the interval, so 5 steps narrow by 2⁵ = 32×.

---

## Section 10 – Attack Execution and Performance

All components are now combined into the complete ElasticNet attack execution: FISTA optimization nested inside binary search.

**Attack hyperparameters:**
```python
config = {
    "beta": 0.01,              # L1 vs L2 trade-off (higher = sparser)
    "confidence": 0,           # Margin for misclassification
    "learning_rate": 0.01,     # FISTA step size
    "max_iterations": 1000,    # FISTA iterations per binary search step
    "binary_search_steps": 5,  # Number of binary search iterations
    "initial_const": 0.001,    # Starting trade-off constant
    "clip_min": 0.0,
    "clip_max": 1.0,
}
```

**Selecting correctly classified samples:**
```python
model.eval()
num_samples = 20

for data, targets in test_loader:
    data, targets = data.to(device), targets.to(device)
    outputs = model(data)
    predictions = outputs.argmax(dim=1)

    correct_mask = predictions.eq(targets)
    attack_data = data[correct_mask][:num_samples]
    attack_targets = targets[correct_mask][:num_samples]

    if len(attack_data) >= num_samples:
        break
```

**Initializing binary search state:**
```python
original_images = attack_data.clone()
labels_onehot = torch.zeros(batch_size, 10).to(device)
labels_onehot.scatter_(1, attack_targets.unsqueeze(1), 1)

lower_bound = torch.zeros(batch_size).to(device)
upper_bound = torch.ones(batch_size).to(device) * 1e10
const = torch.ones(batch_size).to(device) * config["initial_const"]

best_adv = original_images.clone()
best_l2 = torch.ones(batch_size).to(device) * 1e10
```

**The nested binary search + FISTA loop:**
```python
for binary_step in range(config["binary_search_steps"]):
    # Fresh start for each binary search step
    adv_images = original_images.clone().detach()
    y_momentum = adv_images.clone()

    # Inner FISTA optimization
    for iteration in range(config["max_iterations"]):
        adv_images, y_momentum, loss, distances = fista_step(
            adv_images, y_momentum, original_images, labels_onehot,
            const, model, config["beta"], config["learning_rate"],
            config["confidence"], iteration)

    # Check success and update best
    success_mask = check_attack_success(adv_images, attack_targets, model)
    l1_dist, l2_dist, elastic_dist = compute_distances(adv_images, original_images, config["beta"])

    for i in range(batch_size):
        if success_mask[i] and l2_dist[i] < best_l2[i]:
            best_adv[i] = adv_images[i]
            best_l2[i] = l2_dist[i]

    lower_bound, upper_bound, const = update_binary_search_bounds(
        lower_bound, upper_bound, const, success_mask)
```

**Why reinitialize from original images each binary search step?** Perturbations optimized for c=0.001 would contaminate results when testing c=0.01. Fresh start ensures each constant gets a fair evaluation.

**Performance profile:** For 20 MNIST examples, 5 binary search steps, and 1000 FISTA iterations: roughly 100,000 per-image model evaluations. On a modern GPU: 2–3 minutes. On CPU: 30–60 minutes.

---

## Section 11 – Attack Visualizations

After running the ElasticNet attack, visualizations help understand what the adversarial examples actually look like and how distortions are distributed.

**Visualizing the attack transformation (original → perturbation → adversarial):**
```python
fig, axes = plt.subplots(3, 10, figsize=(20, 6))

for i in range(10):
    # Row 1: Original image
    orig_img = original_images[i].detach().cpu().squeeze()
    axes[0, i].imshow(orig_img, cmap="gray", vmin=0, vmax=1)
    axes[0, i].set_title(f"True: {attack_targets[i].item()}")

    # Row 2: Perturbation (amplified 10x for visibility)
    pert = (best_adv[i] - original_images[i]).detach().cpu().squeeze()
    axes[1, i].imshow(pert * 10, cmap="seismic", vmin=-1, vmax=1)
    axes[1, i].set_title("Perturbation")

    # Row 3: Adversarial image
    adv_img = best_adv[i].detach().cpu().squeeze()
    axes[2, i].imshow(adv_img, cmap="gray", vmin=0, vmax=1)
    axes[2, i].set_title(f"Pred: {adv_predictions[i].item()}", color=MALWARE_RED)

plt.savefig(output_dir / "ead_attack_process.png", dpi=150)
```

Perturbations are amplified 10× because actual changes are imperceptible. The **seismic colormap** shows blue for negative changes and red for positive changes, with white for unchanged pixels. Adversarial image titles are colored red to emphasize misclassification.

**Distortion distribution histograms (what you will see):**
- Squared L2 values cluster below 5 (mean ≈ 4.18) with a long right tail reaching ~21 for harder examples.
- L1 magnitudes center near 20 (mean ≈ 20.20).
- L∞ distribution centers around ~0.45, meaning at least some pixels change by up to 0.9.
- Elastic-Net histogram follows squared L2 closely (mean 4.38 vs 4.18) because β=0.01 weights L2 much more than L1.

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# L1 histogram
axes[0, 0].hist(l1_values, bins=15, color=AZURE, alpha=0.7)
axes[0, 0].axvline(l1_values.mean(), color=MALWARE_RED, linestyle="--")

# Save figure
plt.savefig(output_dir / "ead_distortion_analysis.png", dpi=150)
```

---

## Section 12 – Sparsity Analysis

Sparsity analysis reveals *how many* pixels changed and *where* those changes are located. Scalar statistics like "52.6% sparsity" miss the spatial story that heatmaps reveal.

**Mixed-norm relationship analysis:**
```python
perturbations = best_adv - original_images
nonzero_mask = torch.abs(perturbations) > 1e-6
sparsity_per_example = (1 - nonzero_mask.float().sum(dim=(1, 2, 3)) / 784) * 100
```

**Estimating pixel count from L1 and L2:**
If perturbations have similar magnitude `a` across `k` pixels: L1 = k×a, L2² = k×a², so k ≈ L1² / L2². For example, L1=60 and L2²=9 gives k ≈ 3600/9 = 400 pixels modified, leaving 384 unchanged (~49% sparsity).

**Sparsity heatmap for individual examples:**
```python
fig = plt.figure(figsize=(16, 8))

for i in range(10):
    pert = (best_adv[i] - original_images[i]).detach().cpu().squeeze()
    ax.imshow(torch.abs(pert), cmap="hot", vmin=0, vmax=pert.abs().max())
```

The **hot colormap** shows zero change as black, progressing through red/orange to white for maximum change. Bright regions = large pixel modifications.

**Key findings from sparsity analysis:**
- Perturbations concentrate heavily along **digit boundaries and stroke edges**, not random pixels.
- Edge pixels carry maximum discriminative information for CNNs (early layers extract edge features).
- Modifying edges efficiently shifts the model's internal representation.
- Average sparsity around 52.6%, ranging from ~35% for hard examples to ~75% for easy ones.
- Harder examples require denser modifications to cross their decision boundaries.

**Sparsity bar chart across examples:**
```python
ax_stats.bar(range(len(sparsity_values)), sparsity_values, color=AQUAMARINE, alpha=0.7)
ax_stats.axhline(sparsity_values.mean(), color=MALWARE_RED, linestyle="--")
plt.savefig(output_dir / "ead_sparsity_analysis.png", dpi=150)
```

---

## Section 13 – ElasticNet Attack Challenge (CTF)

This section describes the hands-on challenge where you must craft an ElasticNet adversarial example that fools an MNIST classifier while satisfying three distance constraints simultaneously.

**Challenge setup:**
```bash
export BASE_URL="http://instance_ip:port"
curl -s "$BASE_URL/health"    # Check if server is ready
curl -s "$BASE_URL/challenge" # Get challenge parameters
```

**Challenge parameters returned:**
- `sample_index`: Which MNIST test image to attack
- `label`: Ground-truth class
- `beta`: Elastic-net trade-off parameter
- `elastic_max`, `l2_max`, `l1_max`: Maximum allowed distortion bounds
- `image_b64`: Base64-encoded 28×28 grayscale PNG in [0,1] range

**Constraints that must ALL be satisfied:**
1. Elastic-net: `‖x_adv - x‖₂ + β‖x_adv - x‖₁ ≤ elastic_max`
2. L2: `‖x_adv - x‖₂ ≤ l2_max`
3. L1: `‖x_adv - x‖₁ ≤ l1_max`
4. Predicted class must differ from the baseline label
5. All pixel values must stay in [0,1]

**Useful API endpoints:**

```bash
# Get model weights
curl -s -o elasticnet_weights.pth "$BASE_URL/weights"

# Test any image (returns prediction without flag)
curl -s -X POST "$BASE_URL/predict" \
  -H 'content-type: application/json' \
  -d '{"image_b64": "<base64 PNG>"}'

# Submit final adversarial example for flag
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image_b64": "<base64 PNG>"}'
```

**Model architecture (SimpleClassifier):**
```python
class SimpleClassifier(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 32, 3, 1)
        self.conv2 = nn.Conv2d(32, 64, 3, 1)
        self.dropout1 = nn.Dropout(0.25)
        self.dropout2 = nn.Dropout(0.5)
        self.fc1 = nn.Linear(9216, 128)
        self.fc2 = nn.Linear(128, num_classes)

    def forward(self, x):
        x = torch.relu(self.conv1(x))
        x = torch.relu(self.conv2(x))
        x = torch.max_pool2d(x, 2)
        x = self.dropout1(x)
        x = torch.flatten(x, 1)
        x = torch.relu(self.fc1(x))
        x = self.dropout2(x)
        return self.fc2(x)
```

**Image conversion helpers:**
```python
def x01_from_b64_png(b64):
    raw = base64.b64decode(b64)
    img = Image.open(io.BytesIO(raw)).convert("L")
    x = np.asarray(img, dtype=np.float32) / 255.0
    return np.clip(x, 0.0, 1.0)

def b64_png_from_x01(x2d):
    x255 = np.clip((x2d * 255.0).round(), 0, 255).astype(np.uint8)
    img = Image.fromarray(x255, mode="L")
    buf = io.BytesIO()
    img.save(buf, format="PNG", optimize=True)
    return base64.b64encode(buf.getvalue()).decode("ascii")
```

---

## Section 14 – JSMA Fundamentals

The **Jacobian-based Saliency Map Attack (JSMA)** takes a completely different philosophy from ElasticNet. Instead of minimizing perturbation *magnitude*, it minimizes the *number of changed pixels* (the L0 norm). It identifies which specific pixels most strongly influence a model's decision and modifies only those.

JSMA was introduced by Papernot et al. in 2016 ("The Limitations of Deep Learning in Adversarial Settings"). On MNIST, it can achieve misclassification by changing as few as **20–40 pixels out of 784**.

**Key philosophical difference from other attacks:**
- FGSM: constrains L∞ (max per-pixel change), modifies ALL 784 pixels by at most ε = 0.03
- DeepFool: minimizes L2, modifies most pixels with small varying changes
- ElasticNet: balances sparsity and magnitude, modifies 100–200 pixels with controlled changes
- JSMA: minimizes L0 (count), allows individual pixels to jump from 0.0 to 1.0, but modifies very few

**JSMA perturbations look different:** They appear as visible dots or strokes rather than invisible uniform noise. A defense looking for widespread coordinated changes might miss these isolated modifications.

**Computational cost scales with output classes:**
- MNIST (10 classes): 10 backward passes per iteration (manageable)
- ImageNet (1000 classes): 1000 backward passes per iteration (prohibitively expensive)

**Why use logits (not probabilities) for gradient computation?**
Computing gradients with respect to logits (pre-softmax) keeps the target sensitivity α and competitor sum β as **independent signals**. Post-softmax probabilities must sum to 1, which mathematically forces β = -α — this collapses saliency into a squared term and eliminates the competitor suppression entirely, breaking JSMA's intended behavior.

**The JSMA algorithm (overview):**
1. Compute the Jacobian matrix (one backward pass per class)
2. Build a saliency map ranking pixels by importance
3. Select and modify the highest-scoring pixel (or pair)
4. Update the search space (remove saturated pixels)
5. Repeat until misclassification or budget exhausted

**Two governing parameters:**
- `θ` (step size): how much each pixel changes per iteration
- `γ` (feature budget): maximum fraction of pixels that can be modified (e.g., γ=0.15 → max 117 of 784 pixels)

---

## Section 15 – Jacobian and Gradients

To score pixels by attack effectiveness, JSMA needs sensitivity information for every class-pixel pair — the full Jacobian matrix of shape (num_classes × num_pixels).

**Setup:**
```python
import torch, torch.nn as nn, torch.nn.functional as F
import numpy as np
from htb_ai_library.core import set_reproducibility
from htb_ai_library.data import get_mnist_loaders
from htb_ai_library.models import SimpleLeNet
from htb_ai_library.training import train_model
from htb_ai_library.utils import save_model, load_model
from htb_ai_library.visualization import use_htb_style

use_htb_style()
set_reproducibility(1337)
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
```

**Train or load the JSMA target model:**
```python
train_loader, test_loader = get_mnist_loaders(batch_size=128)
model_path = output_dir / 'mnist_target.pth'
model = SimpleLeNet().to(device)

if model_path.exists():
    model = load_model(model, model_path, device)
else:
    model = train_model(model, train_loader, test_loader, epochs=5, device=device)
    save_model(model, model_path)

model.eval()
```

**Computing gradient for a single class:**
```python
def compute_class_gradient(x, model, class_idx, wrt='logits'):
    x_grad = x.detach().requires_grad_(True)
    logits = model(x_grad)
    scalar = logits[0, class_idx]  # Must be scalar for backward()
    scalar.backward()
    grad = x_grad.grad.detach().cpu().numpy().flatten().copy()
    return grad
```

`flatten()` converts the (1,1,28,28) gradient tensor to a length-784 vector where index 352 = channel 0, row 12, column 16. `.copy()` creates an independent NumPy array so PyTorch's memory isn't corrupted during the next backward pass.

**Building the complete Jacobian (10 backward passes for MNIST):**
```python
def compute_jacobian_matrix(x, model, num_classes=10, wrt='logits'):
    if x.shape[0] != 1:
        raise ValueError("compute_jacobian_matrix expects batch size 1")
    jacobian = []
    for class_idx in range(num_classes):
        grad = compute_class_gradient(x, model, class_idx, wrt)
        jacobian.append(grad)
    return np.asarray(jacobian)
# Shape: (10, 784) for MNIST — row i holds ∂F_i/∂x_j for all pixels j
```

**Extracting target and competitor gradients:**
```python
def extract_target_gradient(jacobian, target_class):
    return jacobian[target_class].copy()  # Copy prevents accidental modification

def extract_other_gradients(jacobian, target_class):
    target_grad = jacobian[target_class]
    total_grad = jacobian.sum(axis=0)
    other_grad = total_grad - target_grad  # Fast: total - target = all others
    return other_grad
```

**Applying the search space mask (zeroing out unavailable pixels):**
```python
def apply_search_mask(gradient, search_space):
    return gradient * search_space  # True=1.0 keeps pixel, False=0.0 removes it
```

Boolean multiplication preserves array shape — unavailable pixels get gradient 0.0, ensuring they can never be selected.

---

## Section 16 – Saliency and Search Space

Saliency scoring ranks pixels by how effectively they can push misclassification toward the target class, using gradient sign constraints and magnitude products.

**For the INCREASE direction** (we want to raise that pixel's value):
- Need: target gradient > 0 (raising this pixel boosts target) AND competitor gradient < 0 (raising this pixel hurts competitors)
- Score: `α_j × |β_j|` when both conditions hold, otherwise 0

```python
def score_increase_saliency(target_grad, other_grad):
    increase_mask = (target_grad > 0) & (other_grad < 0)
    scores = target_grad * np.abs(other_grad) * increase_mask
    return scores
```

**For the DECREASE direction** (we want to lower that pixel's value):
- Need: target gradient < 0 (lowering this pixel boosts target) AND competitor gradient > 0 (lowering this pixel hurts competitors)
- Score: `|α_j| × β_j` when both conditions hold, otherwise 0

```python
def score_decrease_saliency(target_grad, other_grad):
    decrease_mask = (target_grad < 0) & (other_grad > 0)
    scores = np.abs(target_grad) * other_grad * decrease_mask
    return scores
```

**Tournament selection — pick best pixel and direction:**
```python
def select_best_direction(inc_scores, dec_scores):
    max_inc_idx = int(np.argmax(inc_scores))
    max_dec_idx = int(np.argmax(dec_scores))
    max_inc_score = float(inc_scores[max_inc_idx])
    max_dec_score = float(dec_scores[max_dec_idx])

    if max_inc_score > max_dec_score:
        return max_inc_idx, max_inc_score, True   # (pixel_idx, score, increase=True)
    else:
        return max_dec_idx, max_dec_score, False  # (pixel_idx, score, increase=False)
```

If all scores are zero → `argmax` returns index 0 with score 0.0 → calling code terminates the attack.

**Initializing the search space:**
```python
def initialize_search_space(shape):
    num_features = int(np.prod(shape[1:]))  # (C,H,W) → 784 for MNIST
    return np.ones(num_features, dtype=bool)
```

Boolean arrays consume 1 byte per element vs 4–8 bytes for integers, and make intent explicit (this is a binary mask).

**Removing saturated pixels:**
```python
def remove_saturated_pixels(search_space, x, clip_min=0.0, clip_max=1.0, epsilon=1e-6):
    x_flat = x.detach().cpu().numpy().flatten()
    saturated_min = (x_flat <= clip_min + epsilon)
    saturated_max = (x_flat >= clip_max - epsilon)
    saturated = saturated_min | saturated_max
    updated_mask = search_space & ~saturated  # Keep pixels that are available AND not saturated
    return updated_mask
```

Epsilon tolerance (1e-6) handles floating-point rounding: a pixel at exactly 0.0 might be stored as ±0.0000001.

---

## Section 17 – Single-Pixel Attack Fundamentals

This section builds the utility functions needed to actually modify pixels, check success, and monitor progress during the JSMA attack loop.

**Applying a single pixel perturbation:**
```python
def apply_single_pixel_perturbation(x, pixel_idx, theta, increase, clip_min=0.0, clip_max=1.0):
    original_shape = x.shape
    x_flat = x.view(-1).clone()  # Flatten AND clone (clone prevents memory aliasing)
    perturbation = theta if increase else -theta
    x_flat[pixel_idx] = torch.clamp(x_flat[pixel_idx] + perturbation, clip_min, clip_max)
    return x_flat.view(original_shape)
```

`.view(-1)` reshapes (1,1,28,28) to a 784-element vector in C-H-W row-major order. Flat index 352 maps to row 12, column 16 via: `0×28×28 + 12×28 + 16 = 352`. `torch.clamp()` prevents values from going below 0.0 or above 1.0.

**Checking if target misclassification was achieved:**
```python
def check_target_reached(x, target_class, model):
    with torch.no_grad():
        logits = model(x)
        prediction = int(logits.argmax(dim=1).item())
    return prediction == target_class
```

`torch.no_grad()` disables gradient tracking since we're only evaluating — saves memory and computation.

**Computing target class confidence (probability):**
```python
def compute_confidence(x, target_class, model):
    with torch.no_grad():
        logits = model(x)
        probs = F.softmax(logits, dim=1)
        confidence = float(probs[0, target_class].item())
    return confidence
```

Track this value during the attack: starts near 0 for an unlikely class, rises toward 1 as perturbations accumulate, crosses the decision boundary around 0.5 when misclassification occurs.

**Demonstrating one iteration (run step by step):**
```python
# Select a correctly classified sample
for x_batch, y_batch in test_loader:
    x_batch, y_batch = x_batch.to(device), y_batch.to(device)
    with torch.no_grad():
        preds = model(x_batch).argmax(dim=1)
    for i in range(x_batch.size(0)):
        if preds[i].item() == y_batch[i].item():
            x = x_batch[i:i+1]
            original_class = int(y_batch[i].item())
            target_class = (original_class + 5) % 10
            break
    break

# Initialize
x_adv = x.clone().detach()
theta = 0.25
search_space = np.ones(784, dtype=bool)

# One iteration
jacobian = compute_jacobian_matrix(x_adv, model, num_classes=10, wrt='logits')
alpha = extract_target_gradient(jacobian, target_class)
beta = extract_other_gradients(jacobian, target_class)
alpha_masked = apply_search_mask(alpha, search_space)
beta_masked = apply_search_mask(beta, search_space)

inc_scores = score_increase_saliency(alpha_masked, beta_masked)
dec_scores = score_decrease_saliency(alpha_masked, beta_masked)
pixel_idx, saliency, increase = select_best_direction(inc_scores, dec_scores)

x_adv = apply_single_pixel_perturbation(x_adv, pixel_idx, theta, increase, 0.0, 1.0)
search_space = remove_saturated_pixels(search_space, x_adv, 0.0, 1.0)
success = check_target_reached(x_adv, target_class, model)
```

---

## Section 18 – Single-Pixel Attack Loop

Full JSMA attack using single-pixel selection, demonstrating the complete loop structure with termination checks, progress tracking, and result evaluation.

**Attack configuration:**
```python
x_adv = x.clone().detach()
search_space = initialize_search_space(x.shape)

config = {
    'theta': 0.25,    # Step size per modification
    'gamma': 0.15,    # Feature budget fraction
    'max_iter': 100,  # Maximum iterations
    'wrt': 'logits',  # Gradient computation target
    'clip_min': 0.0,
    'clip_max': 1.0
}

num_features = int(np.prod(x.shape[1:]))
max_pixels = int(config['gamma'] * num_features)  # 0.15 × 784 = 117 pixels max
```

**Main attack loop:**
```python
stats = {'iterations': [], 'pixels_modified': [], 'target_confidence': [], 'saliency_scores': []}
pixels_modified = 0

for iteration in range(config['max_iter']):
    if check_target_reached(x_adv, target_class, model):
        print(f"Target reached at iteration {iteration}!")
        break
    if pixels_modified >= max_pixels:
        print("Budget exhausted")
        break

    jacobian = compute_jacobian_matrix(x_adv, model, num_classes=10, wrt=config['wrt'])
    alpha = extract_target_gradient(jacobian, target_class)
    beta = extract_other_gradients(jacobian, target_class)
    alpha_masked = apply_search_mask(alpha, search_space)
    beta_masked = apply_search_mask(beta, search_space)

    inc_scores = score_increase_saliency(alpha_masked, beta_masked)
    dec_scores = score_decrease_saliency(alpha_masked, beta_masked)
    pixel_idx, saliency, increase = select_best_direction(inc_scores, dec_scores)

    if saliency <= 0:
        print("No valid pixels remaining")
        break

    x_adv = apply_single_pixel_perturbation(x_adv, pixel_idx, config['theta'], increase,
                                             config['clip_min'], config['clip_max'])

    search_space[pixel_idx] = False  # Prevent re-selection
    search_space = remove_saturated_pixels(search_space, x_adv, 0.0, 1.0)
    pixels_modified += 1

    confidence = compute_confidence(x_adv, target_class, model)
    stats['iterations'].append(iteration)
    stats['pixels_modified'].append(pixels_modified)
    stats['target_confidence'].append(confidence)
    stats['saliency_scores'].append(saliency)
```

**Typical result (digit 7 → target 2, theta=0.25):** Attack FAILED after 100 iterations, modifying 100 of 117 allowed pixels. Target confidence reached only 0.0002 (0.02%). Saliency scores declined steadily from 0.641 to 0.031 — diminishing returns as high-value pixels were exhausted. Small theta requires many iterations to build enough change to cross the decision boundary.

---

## Section 19 – Single-Pixel Batch Evaluation

Evaluating across 10 samples reveals whether the failure on one sample is typical or exceptional.

**Collecting 10 correctly classified samples:**
```python
original_images, original_labels, target_labels = [], [], []

for x_batch, y_batch in test_loader:
    if len(original_images) >= 10:
        break
    x_batch, y_batch = x_batch.to(device), y_batch.to(device)
    with torch.no_grad():
        preds = model(x_batch).argmax(dim=1)
    for i in range(x_batch.size(0)):
        if preds[i].item() == y_batch[i].item():
            original_images.append(x_batch[i:i+1])
            original_labels.append(int(y_batch[i].item()))
            target_labels.append((int(y_batch[i].item()) + 5) % 10)  # Offset by 5
```

**Batch results (10 samples, theta=0.25):**
- Success rate: **7/10 (70%)**
- Pixels modified: Mean 73.6 ± 24.2, Median 76.5, Range [33, 100]
- Sparsity: Mean 73.6 / 784 = **9.39% of image changed**
- 3 failures all exhausted the 100-iteration limit

**Perturbation analysis for one sample:**
```python
x_orig = original_images[0].cpu().numpy()
x_adv_np = results['adversarial'][0].cpu().numpy()
perturbation = x_adv_np - x_orig

l0_norm = np.count_nonzero(perturbation)          # How many pixels changed
l1_norm = np.sum(np.abs(perturbation))            # Total change magnitude
l2_norm = np.linalg.norm(perturbation)            # Euclidean distance
linf_norm = np.max(np.abs(perturbation))          # Max single-pixel change
```

For sample 1: L0=51 (6.5% of image), L1=19.44, L2=3.54, L∞=0.9961 (one pixel nearly saturated).

---

## Section 20 – Single-Pixel Configuration

Parameter choices critically affect attack behavior. Testing θ values from 0.10 to 1.00 reveals a key insight:

**Step size experiment results (sample 1, digit 7→2):**
- θ = 0.10 → FAILED (100 iterations, 100 pixels)
- θ = 0.25 → FAILED (100 iterations, 100 pixels)
- θ = 0.50 → FAILED (100 iterations, 100 pixels)
- θ = 1.00 → **SUCCESS** (64 iterations, 63 pixels)

With θ=1.0, each selected pixel **saturates immediately** (jumps from 0.0 to 1.0 or vice versa), building enough perturbation magnitude to cross the decision boundary. Smaller values modify pixels incrementally and often can't accumulate sufficient change within the budget.

**Feature budget (gamma) effects:**
```
γ = 0.10 → max 78 pixels (10% of image)
γ = 0.15 → max 117 pixels (15% of image)  ← default
γ = 0.20 → max 157 pixels (20% of image)
γ = 0.30 → max 235 pixels (30% of image)
```

Most successful attacks (theta=0.25) used 33–92 pixels out of 117 allowed, consuming about 63% of the budget. Increasing gamma beyond 0.15 rarely improves success rates — successful attacks already complete well below the current limit.

**Key takeaway:** θ=1.0 is recommended for MNIST when success matters more than subtlety. The attack creates visible artifacts (pixels jumping fully black or white) but achieves misclassification with fewer total modified pixels.

---

## Section 21 – Pairwise Saliency

Pairwise JSMA improves on single-pixel selection by modifying **two pixels simultaneously**, exploiting feature interactions in nonlinear models. For a pair (p, q):
- Combined target gradient: `α_pq = α_p + α_q`
- Combined competitor gradient: `β_pq = β_p + β_q`
- Pair score (increase): `S_t⁺[p,q] = α_pq × |β_pq|` when `α_pq > 0` and `β_pq < 0`

**Why pairs capture interactions:** Two pixels might individually have weak gradients that pass sign constraints, but together their combined gradients create a stronger, more effective perturbation. The summation `α_pq = α_p + α_q` can satisfy sign constraints even when neither individual pixel would.

**Candidate pruning to control O(n²) cost:**
```python
def prune_candidates(alpha, beta, search_space, top_k):
    alpha_masked = alpha * search_space
    beta_masked = beta * search_space
    valid = np.where(search_space)[0]

    if valid.size < 2 or top_k is None or valid.size <= top_k:
        return valid

    prelim_scores = np.abs(alpha_masked[valid]) * np.abs(beta_masked[valid])
    idx = np.argsort(-prelim_scores)[:top_k]
    return valid[idx]
```

Without pruning: 784 pixels → (784 choose 2) = 306,936 pairs evaluated. With top_k=128: only 8,128 pairs evaluated (97% reduction).

**Pair scoring loop:**
```python
def evaluate_pairs(alpha, beta, valid, direction):
    best_p, best_q, best_score = -1, -1, 0.0
    for i in range(valid.size):
        p = valid[i]
        for j in range(i + 1, valid.size):  # j starts at i+1 to avoid duplicates
            q = valid[j]
            a_pq = alpha[p] + alpha[q]
            b_pq = beta[p] + beta[q]
            if direction == 'increase':
                if a_pq <= 0 or b_pq >= 0: continue
                score = a_pq * abs(b_pq)
            else:
                if a_pq >= 0 or b_pq <= 0: continue
                score = abs(a_pq) * b_pq
            if score > best_score:
                best_score, best_p, best_q = float(score), int(p), int(q)
    return best_p, best_q, best_score
```

**Applying pair perturbation (both pixels get same signed step):**
```python
def apply_pair_perturbation(x, p, q, theta, increase, clip_min=0.0, clip_max=1.0):
    original_shape = x.shape
    x_flat = x.view(-1).clone()
    step = theta if increase else -theta
    x_flat[p] = torch.clamp(x_flat[p] + step, clip_min, clip_max)
    x_flat[q] = torch.clamp(x_flat[q] + step, clip_min, clip_max)
    return x_flat.view(original_shape)
```

Both pixels must move in the same direction to preserve the mathematical assumptions behind the pair score formula.

---

## Section 22 – Pairwise Attack Loop

The complete pairwise JSMA attack loop follows the same structure as single-pixel, but selects and modifies two pixels per iteration.

**Configuration:**
```python
x_adv = x.clone().detach()
search_space = initialize_search_space(x.shape)

config = {
    'theta': 1.0,      # Aggressive: saturate pixels immediately
    'gamma': 0.15,     # Same budget as single-pixel
    'max_iter': 90,
    'wrt': 'logits',
    'clip_min': 0.0,
    'clip_max': 1.0,
    'top_k': 128       # NEW: candidate pruning for efficiency
}

max_pixels = int(config['gamma'] * 784)  # 117 pixels max
# With pairs: max ⌊117/2⌋ = 58 iterations before budget exhausted
```

**Pairwise iteration loop:**
```python
for iteration in range(config['max_iter']):
    if check_target_reached(x_adv, target_class, model): break
    if pixels_modified >= max_pixels: break

    jacobian = compute_jacobian_matrix(x_adv, model, 10, config['wrt'])
    alpha = extract_target_gradient(jacobian, target_class)
    beta = extract_other_gradients(jacobian, target_class)

    # Score BOTH directions and pick winner
    p_inc, q_inc, score_inc = compute_pairwise_saliency(alpha, beta, search_space, 'increase', config['top_k'])
    p_dec, q_dec, score_dec = compute_pairwise_saliency(alpha, beta, search_space, 'decrease', config['top_k'])

    if max(score_inc, score_dec) <= 0.0: break

    if score_inc >= score_dec:
        p, q, score, increase = p_inc, q_inc, score_inc, True
    else:
        p, q, score, increase = p_dec, q_dec, score_dec, False

    x_adv = apply_pair_perturbation(x_adv, p, q, config['theta'], increase, 0.0, 1.0)

    # Remove BOTH pixels from search space
    search_space[p] = False
    search_space[q] = False
    search_space = remove_saturated_pixels(search_space, x_adv, 0.0, 1.0)
    pixels_modified += 2  # Two pixels per iteration!
```

**Typical progress (digit 7→2, theta=1.0):**
Target confidence grows from 0.0000 at iteration 0, reaches 0.2927 at iteration 27, then 0.7262 at iteration 33, and the target is reached at **iteration 34** with only 68 pixels modified (8.7% of image). Compare this to single-pixel with theta=0.25 which FAILED after 100 iterations.

---

## Section 23 – Pairwise Batch Evaluation

Running pairwise attacks on the same 10 samples used for single-pixel evaluation enables direct performance comparison.

**Pairwise batch loop (same structure as single-pixel, but uses pair functions):**
```python
results_pairs = {'adversarial': [], 'success': [], 'pixels_modified': [], 'iterations': []}

for idx in range(len(original_images)):
    x = original_images[idx]
    x_adv = x.clone().detach()
    search_space = initialize_search_space(x.shape)
    pixels_mod = 0

    for iteration in range(config['max_iter']):
        if check_target_reached(x_adv, target_labels[idx], model): break
        if pixels_mod >= int(config['gamma'] * 784): break

        jacobian = compute_jacobian_matrix(x_adv, model, 10, config['wrt'])
        alpha = extract_target_gradient(jacobian, target_labels[idx])
        beta = extract_other_gradients(jacobian, target_labels[idx])

        p_inc, q_inc, score_inc = compute_pairwise_saliency(alpha, beta, search_space, 'increase', config['top_k'])
        p_dec, q_dec, score_dec = compute_pairwise_saliency(alpha, beta, search_space, 'decrease', config['top_k'])

        if max(score_inc, score_dec) <= 0.0: break
        if score_inc >= score_dec: p, q, score, increase = p_inc, q_inc, score_inc, True
        else: p, q, score, increase = p_dec, q_dec, score_dec, False

        x_adv = apply_pair_perturbation(x_adv, p, q, config['theta'], increase, 0.0, 1.0)
        search_space[p] = False
        search_space[q] = False
        search_space = remove_saturated_pixels(search_space, x_adv, 0.0, 1.0)
        pixels_mod += 2
```

**Pairwise results (10 samples, theta=1.0):**
- **9/10 succeeded** (90% success rate)
- Pixels used: 12, 118, 38, 12, 28, 32, 34, 32, 52, 14
- Iterations: 35, 60, 20, 7, 15, 17, 18, 17, 27, 8
- Only sample 2 (digit 2→7) failed, exhausting 118 pixels

**Direct comparison table:**

| Sample | Single-Pixel Iters | Single-Pixel Pixels | Pairwise Iters | Pairwise Pixels |
|--------|-------------------|---------------------|----------------|-----------------|
| 1      | 100               | 100                 | 35             | 68              |
| 2      | 100               | 100                 | 60             | 118             |
| 3      | 68                | 67                  | 20             | 38              |
| 4      | 34                | 33                  | 7              | 12              |
| 5      | 59                | 58                  | 15             | 28              |
| Mean   | 74.3              | 73.6                | 22.4           | 42.8            |

**Iteration reduction: 69.9%** — Pairwise requires 69.9% fewer iterations on average. Despite modifying 2 pixels per iteration, total pixel usage drops from 73.6 to 42.8 on average (42% fewer pixels).

---

## Section 24 – Pairwise Analysis

This section analyzes why pairwise selection outperforms single-pixel and the computational trade-offs involved.

**Computational cost comparison:**
- Single-pixel saliency: < 0.01 ms (vectorized numpy, negligible)
- Pairwise saliency (top_k=128): ~2.13 ms (evaluating 8,128 pairs)
- Jacobian computation: ~15 ms (10 backward passes, dominates)
- Per-iteration overhead: 14% more for pairwise (17.1 ms vs 15.0 ms)

**End-to-end cost analysis (the key insight):**
The 14% per-iteration overhead is completely offset by:
- 69.9% fewer total iterations (22.4 vs 74.3)
- Higher success rate (90% vs 70%)
- Fewer total pixels modified (42.8 vs 73.6)

**top_k pruning trade-offs:**
- `top_k=128` (default): ~3ms, 8,128 pairs — best balance
- `top_k=64` (fast): ~1ms, 2,016 pairs — slightly lower quality
- `top_k=None` (full): ~10-20ms, 300k+ pairs — rarely justified

**Key understanding:** The attack succeeds through cumulative sequential modifications across many iterations. Testing individual pixel pairs in isolation may show zero effect because success requires the combined context of all prior modifications. This is why the synergy analysis shows 0.000 for individual pairs — the real synergy is temporal, not spatial.

---

## Section 25 – Attack Visualizations (JSMA)

Three main visualizations reveal JSMA's behavior: attack transformation, convergence progress, and saliency maps.

**Attack process (4-panel: original → adversarial → perturbation magnitude → binary mask):**
```python
def visualize_attack_process(original, adversarial, original_class, target_class,
                             predicted_class, output_dir):
    orig = np.squeeze(original.cpu().numpy())
    adv = np.squeeze(adversarial.cpu().numpy())
    pert = adv - orig
    pert_mag = np.abs(pert)

    fig, axes = plt.subplots(1, 4, figsize=(14, 3.8))

    axes[0].imshow(orig, cmap='gray')
    axes[0].set_title(f'Original\nTrue: {original_class}')

    axes[1].imshow(adv, cmap='gray')
    axes[1].set_title(f'Adversarial\nPred: {predicted_class}')

    pm_norm = pert_mag / (pert_mag.max() + 1e-8)
    axes[2].imshow(pm_norm, cmap='hot')  # hot colormap: black=zero, yellow/white=large
    axes[2].set_title(f'Perturbation |Δ|\nL0={int(mod_map.sum())}')

    axes[3].imshow((pert_mag > 0), cmap='hot')  # Binary mask
    axes[3].set_title(f'Modified Pixels\n{int(mod_map.sum())}/784 ({int(mod_map.sum())/784*100:.2f}%)')

    plt.savefig(output_dir / 'jsma_attack_process.png', dpi=175)
```

**Convergence tracking:**
```python
def visualize_attack_progress(stats, output_dir):
    fig, ax = plt.subplots(figsize=(7.5, 3.6))
    ax2 = ax.twinx()

    ax.plot(stats['iterations'], stats['target_confidence'], color=HTB_GREEN, label='Target Confidence')
    ax2.plot(stats['iterations'], stats['pixels_modified'], color=NUGGET_YELLOW, label='Pixels Modified')

    ax.set_xlabel('Iteration')
    ax.set_ylabel('Target Confidence')
    ax2.set_ylabel('Pixels Modified')
    plt.savefig(output_dir / 'jsma_progress.png', dpi=175)
```

**Saliency map (heatmap showing which pixels JSMA considers most important):**
```python
def visualize_saliency_map(alpha, beta, search_space, original_shape, output_dir):
    inc_scores = score_increase_saliency(alpha, beta)
    dec_scores = score_decrease_saliency(alpha, beta)
    combined_scores = np.maximum(inc_scores, dec_scores) * search_space

    h, w = original_shape[-2:]
    saliency_map = combined_scores.reshape(h, w)

    fig, ax = plt.subplots(figsize=(6, 6))
    im = ax.imshow(saliency_map, cmap='hot', interpolation='nearest')
    plt.colorbar(im, ax=ax, label='Saliency Score')
    plt.savefig(output_dir / 'jsma_saliency_map.png', dpi=175)
```

---

## Section 26 – Aggregate Analysis

**L0 distribution across all samples (combined single-pixel + pairwise):**
```python
def visualize_l0_distribution(l0_values, output_dir):
    fig, ax = plt.subplots(figsize=(8, 4.5))
    ax.hist(l0_values, bins=30, color=HTB_GREEN, alpha=0.7)
    ax.axvline(np.mean(l0_values), color=NUGGET_YELLOW, linestyle='--', label=f'Mean: {np.mean(l0_values):.1f}')
    ax.axvline(np.median(l0_values), color=AZURE, linestyle='--', label=f'Median: {np.median(l0_values):.1f}')
    ax.set_xlabel('L0 Norm (Pixels Modified)')
    ax.set_ylabel('Count')
    plt.savefig(output_dir / 'jsma_l0_distribution.png', dpi=175)

all_l0_values = results['pixels_modified'] + results_pairs['pixels_modified']
visualize_l0_distribution(all_l0_values, output_dir)
# Expected output: Mean L0 ≈ 58.2, Median ≈ 55.0, Range [12, 118]
```

**Side-by-side comparison grid (all 10 samples, 5 rows):**
- Row 1: Original digits with true labels
- Row 2: Single-pixel perturbation heatmaps (hot colormap, L0 annotation)
- Row 3: Single-pixel adversarial results (green=success, red=failure)
- Row 4: Pairwise perturbation heatmaps
- Row 5: Pairwise adversarial results

```python
visualize_attack_comparison(
    original_images,
    results['adversarial'],
    results_pairs['adversarial'],
    original_labels,
    predicted_single,
    predicted_pairs,
    target_labels,
    output_dir
)
```

**Visual interpretation:** Single-pixel rows show 10 red predictions (all failed with theta=0.25). Pairwise rows show 9 green, 1 red (90% success). Sample 4 shows remarkable sparsity — only 12 pixels modified (1.5% of image). The contrast is stark and immediately visible.

---

## Section 27 – JSMA Attack Challenge (CTF)

Targeted JSMA challenge: modify at most `l0_budget` pixels to force the classifier to predict a specific `target_class`.

**Challenge setup:**
```bash
export BASE_URL="http://instance_ip:port"
curl -s "$BASE_URL/health"
curl -s "$BASE_URL/challenge" | jq
# Returns: sample_index, target_class, l0_budget, original_label, max_l2, image_b64
```

**Constraint:** `‖x_adv - x‖₀ ≤ budget` (count of pixels with |difference| > 1e-6)

**Model architecture (MNISTClassifier — LeNet-5 style):**
```python
class MNISTClassifier(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 6, kernel_size=5, stride=1, padding=0)
        self.conv2 = nn.Conv2d(6, 16, kernel_size=5, stride=1, padding=0)
        self.pool = nn.AvgPool2d(kernel_size=2, stride=2)
        self.fc1 = nn.Linear(16*4*4, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)
        self.act = nn.Tanh()

    def forward(self, x):
        x = self.act(self.conv1(x)); x = self.pool(x)
        x = self.act(self.conv2(x)); x = self.pool(x)
        x = torch.flatten(x, 1)
        x = self.act(self.fc1(x)); x = self.act(self.fc2(x))
        return F.log_softmax(self.fc3(x), dim=1)
```

**Key difference from EAD challenge:** This model uses `log_softmax` output and `Tanh` activations, not ReLU. Gradients behave differently — compute them with `wrt='logits'` before log_softmax.

**API for JSMA challenge:**
```bash
# Download model weights
curl -s -o jsma_weights.pth "$BASE_URL/weights"

# Test prediction
curl -s -X POST "$BASE_URL/predict" \
  -H 'content-type: application/json' \
  -d '{"image_b64": "<base64 PNG>"}'

# Submit for flag
curl -s -X POST "$BASE_URL/submit" \
  -H 'content-type: application/json' \
  -d '{"image_b64": "<base64 PNG>"}'
```

---

## Section 28 – Skills Assessment (CIFAR-10 / ResNet-18)

Final challenge: apply either EAD or JSMA to a **ResNet-18 classifier on CIFAR-10** (32×32 RGB images, 10 classes). Must achieve targeted misclassification with minimum L2 perturbation of 1.5 (anti-cheat measure).

**Setup:**
```bash
export BASE_URL="http://instance_ip:port"
curl -s "$BASE_URL/health"
curl -s "$BASE_URL/challenge" | jq  # Returns items array with sample_id, label, target, method
curl -s "$BASE_URL/model" | jq      # Returns arch, weights_sha256, normalize params
curl -sL "$BASE_URL/model/weights" -o cifar10_model.pth
```

**ResNet-18 (CIFAR-10 adapted) architecture:**
```python
class BasicBlock(nn.Module):
    expansion = 1
    def __init__(self, in_planes, planes, stride=1):
        super().__init__()
        self.conv1 = nn.Conv2d(in_planes, planes, 3, stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(planes)
        self.conv2 = nn.Conv2d(planes, planes, 3, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(planes)
        self.shortcut = nn.Sequential()
        if stride != 1 or in_planes != planes:
            self.shortcut = nn.Sequential(
                nn.Conv2d(in_planes, planes, 1, stride=stride, bias=False),
                nn.BatchNorm2d(planes))

    def forward(self, x):
        out = torch.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        out += self.shortcut(x)
        return torch.relu(out)

class ResNetCIFAR(nn.Module):
    def __init__(self, num_blocks=(2,2,2,2), num_classes=10):
        super().__init__()
        self.in_planes = 64
        self.conv1 = nn.Conv2d(3, 64, 3, 1, 1, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.layer1 = self._make_layer(64,  num_blocks[0], 1)
        self.layer2 = self._make_layer(128, num_blocks[1], 2)
        self.layer3 = self._make_layer(256, num_blocks[2], 2)
        self.layer4 = self._make_layer(512, num_blocks[3], 2)
        self.avgpool = nn.AdaptiveAvgPool2d(1)
        self.fc = nn.Linear(512, num_classes)

    def forward(self, x):
        out = torch.relu(self.bn1(self.conv1(x)))
        out = self.layer1(out); out = self.layer2(out)
        out = self.layer3(out); out = self.layer4(out)
        out = self.avgpool(out)
        return self.fc(torch.flatten(out, 1))
```

**CIFAR-10 normalization:**
```python
def cifar_normalize(x):
    mean = torch.tensor((0.4914, 0.4822, 0.4465))[None, :, None, None]
    std  = torch.tensor((0.2470, 0.2435, 0.2616))[None, :, None, None]
    return (x - mean.to(x.device)) / std.to(x.device)
```

**Image encoding helpers (RGB, 32×32):**
```python
def b64_from_x01(x4d):     # x4d shape: (1, 3, 32, 32)
    x = np.transpose(x4d[0], (1,2,0))  # → (32, 32, 3)
    x255 = np.clip((x*255).round(), 0, 255).astype(np.uint8)
    img = Image.fromarray(x255, mode="RGB")
    buf = io.BytesIO(); img.save(buf, format="PNG")
    return base64.b64encode(buf.getvalue()).decode("ascii")

def x01_from_b64(b64):     # Returns (1, 3, 32, 32) float32 in [0,1]
    raw = base64.b64decode(b64)
    img = Image.open(io.BytesIO(raw)).convert("RGB")
    x = np.asarray(img, dtype=np.float32) / 255.0
    return np.transpose(x, (2,0,1))[None, ...].astype(np.float32)
```

**Submission format:**
```bash
curl -s -X POST "$BASE_URL/submit_images" \
  -H 'content-type: application/json' \
  -d '{"items": [{"sample_id": 0, "method": "ead", "image_b64": "..."}]}'
```

**Anti-cheat:** Minimum L2 perturbation threshold of 1.5 — can't submit the clean image. Must contain real adversarial perturbations. Both EAD and JSMA methods are accepted (`method: "ead"` or `method: "jsma"`).

---

## Quick Reference Summary

| Attack | Norm | Key Feature | Best Use |
|--------|------|-------------|----------|
| FGSM | L∞ | Single step, all pixels | Speed, adversarial training data |
| DeepFool | L2 | Geometric, minimal distortion | Robustness measurement |
| ElasticNet (EAD) | L1+L2 | Sparse + smooth, binary search | Evading detection, feature analysis |
| JSMA (single-pixel) | L0 | Explicit pixel budget, θ=0.25 | Demonstration, subtle changes |
| JSMA (pairwise) | L0 | Pair synergy, θ=1.0 | Better success rate, efficiency |

**Critical parameters quick reference:**
- EAD: `beta=0.01` (sparsity), `learning_rate=0.01`, `binary_search_steps=5`, `max_iterations=1000`
- JSMA: `theta=1.0` (aggressive), `gamma=0.15` (15% pixel budget), `top_k=128` (pair pruning)
- FISTA momentum: `k/(k+3)` — grows from 0.25 at iteration 1 to 0.97 at iteration 100
- Soft threshold: `S_λ(z) = sign(z) × max(|z| - λ, 0)`
