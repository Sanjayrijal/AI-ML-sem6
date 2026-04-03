# Image Processing & PCA - Quick Reference & Code Examples

## **Quick Cheat Sheet**

### **PIL/Image Operations**
```python
from PIL import Image
import numpy as np

# Read image
image = Image.open('path.jpg')

# Convert to grayscale
gray_image = image.convert('L')

# Convert to NumPy
array = np.array(image)  # Shape: (H, W, 3) for RGB

# Convert back to PIL
image = Image.fromarray(array.astype(np.uint8))

# Rotate
rotated = image.rotate(-90)  # Clockwise

# Resize
resized = image.resize((200, 150))
```

---

### **NumPy Array Indexing**
```python
# Extract regions
top_left = array[:100, :100]              # First 100×100 pixels
bottom_right = array[-100:, -100:]        # Last 100×100 pixels
middle = array[150:250, 150:250]          # Middle 100×100 region
every_other = array[::2, ::2]             # Downsample by 2

# Separate channels
red = array[:, :, 0]
green = array[:, :, 1]
blue = array[:, :, 2]

# Extract row/column
row_100 = array[100, :]
col_50 = array[:, 50]

# Modify region
array[:100, :100] = 210           # Set to gray (all channels)
array[:100, :100] = [255, 0, 0]   # Set to red
```

---

### **Thresholding**
```python
# Simple threshold
binary = np.where(gray_array < 100, 0, 255)

# With NumPy
binary = (gray_array > 128).astype(int) * 255

# Otsu's threshold (auto-find optimal threshold)
from scipy import ndimage
threshold = ndimage.threshold_otsu(gray_array)
binary = gray_array > threshold
```

---

### **PCA from Scratch**
```python
import numpy as np

def pca_compress(image_array, variance_threshold=0.95):
    # 1. Center data
    mean = np.mean(image_array, axis=1, keepdims=True)
    centered = image_array - mean
    
    # 2. Covariance
    cov = np.cov(centered)
    
    # 3. Eigendecomposition
    evals, evecs = np.linalg.eigh(cov)
    
    # 4. Sort descending
    idx = np.argsort(evals)[::-1]
    evals = evals[idx]
    evecs = evecs[:, idx]
    
    # 5. Find k
    cum_var = np.cumsum(evals) / np.sum(evals)
    k = np.argmax(cum_var >= variance_threshold) + 1
    
    # 6. Project
    top_evecs = evecs[:, :k]
    projected = np.dot(top_evecs.T, centered)
    
    # 7. Reconstruct
    reconstructed = np.dot(top_evecs, projected) + mean
    reconstructed = np.clip(reconstructed, 0, 255)
    
    # 8. Error
    mse = np.mean((image_array - reconstructed) ** 2)
    
    return {
        'reconstructed': reconstructed,
        'k': k,
        'mse': mse,
        'variance': cum_var[k-1],
        'mean': mean
    }

# Usage
result = pca_compress(image_array, variance_threshold=0.95)
print(f"Used {result['k']} components")
print(f"MSE: {result['mse']:.2f}")
```

---

### **Color Space Conversions**
```python
from PIL import Image
import numpy as np

# RGB to Grayscale (manual)
gray = 0.299 * image_rgb[:,:,0] + 0.587 * image_rgb[:,:,1] + 0.114 * image_rgb[:,:,2]

# Grayscale to RGB
gray_image = Image.open('gray.jpg')
rgb_image = Image.merge("RGB", (gray_image, gray_image, gray_image))

# RGB to HSV
from matplotlib.colors import rgb_to_hsv
hsv = rgb_to_hsv(image_rgb / 255.0)

# Extract specific color
red_only = image_rgb.copy()
red_only[:, :, 1] = 0  # Remove green
red_only[:, :, 2] = 0  # Remove blue
```

---

### **Common Mistakes & Fixes**

| Mistake | Fix |
|---------|-----|
| `image[:100, :100] = 255` on RGB → Only sets red | Use `image[:100, :100] = [255, 255, 255]` |
| Shape mismatch in covariance | Ensure 2D before `np.cov()` |
| `fromarray()` fails | Convert to `uint8`: `array.astype(np.uint8)` |
| Rotation loses data | PIL auto-expands canvas for arbitrary angles |
| PCA eigenvectors misaligned | Sort BOTH eigenvalues and eigenvectors |
| Reconstruction values > 255 | Use `np.clip(array, 0, 255)` |
| `np.where()` output not binary | Output is 0 and 255, not 0 and 1 |
| Centering breaks image | Add `mean` back during reconstruction |

---

### **Interview Questions - Quick Answers**

**Q: Why is uint8 needed?**
A: Images are 8-bit, values 0-255. uint8 saves memory (1 byte vs 4-8 bytes for float).

**Q: What does eigh do differently?**
A: Works only on symmetric matrices (covariance is symmetric), faster and more stable than eig.

**Q: Why -90 for clockwise?**
A: Math convention: positive = counter-clockwise, negative = clockwise.

**Q: Can PCA reconstruct perfectly?**
A: Only if k = number of rows. Otherwise, loses information.

**Q: What if eigenvalues are equal?**
A: Bad for PCA. Means no preferred direction (random data, no patterns).

**Q: Storage gain formula?**
A: Original bytes / (stored bytes) = compression ratio. Usually 2-5× for images.

**Q: Why keepdims=True?**
A: Keeps dimensions for broadcasting. np.mean(axis=1) gives (H,), with keepdims→ (H,1).

**Q: What's the difference between downsample with PCA vs image resize?**
A: Resize: loses pixels directly. PCA: loses low-variance patterns intelligently.

---

### **Debugging Checklist**

- [ ] Image loaded correctly? Check shape and dtype
- [ ] Array centered? Verify mean is near 0
- [ ] Eigenvalues sorted? Should be descending
- [ ] Cumulative variance computed? Should end at 1.0
- [ ] k value reasonable? Usually 50-200 for typical images
- [ ] Reconstruction clipped? Values should be 0-255
- [ ] MSE makes sense? Compare k values

---

### **Performance Tips**

1. **For large images:** Resize first → PCA on smaller version
2. **Memory efficient:** Process by channels separately
3. **Faster eigendecomposition:** Use randomized SVD for huge matrices
4. **Batch processing:** PCA eigenvectors can be reused across similar images

```python
# Fast SVD approach for large data
from sklearn.decomposition import PCA
pca = PCA(n_components=50)
projected = pca.fit_transform(centered_data.T).T
```

---

### **Formula Sheet**

| Operation | Formula |
|-----------|---------|
| **Mean-center** | `X - mean(X)` |
| **Covariance** | `(1/n) × X × X^T` |
| **Eigenvalue problem** | `Σv = λv` where Σ is covariance |
| **Explained variance ratio** | `λ_i / Σ(all λ)` |
| **Cumulative variance** | `cumsum(variance_ratios)` |
| **Projection** | `W^T × X` where W contains top eigenvectors |
| **Reconstruction** | `W × (W^T × X) + μ` |
| **MSE** | `mean((X - X_reconstructed)²)` |
| **Compression ratio** | `(H × W) / (H × k + k × W + H)` |

---

### **Real-World Applications**

| Application | Why PCA Works |
|-------------|---------------|
| Face recognition | Faces have similar structure |
| Medical imaging | Repeated anatomical patterns |
| Image denoising | Noise is low-variance |
| Data compression | Removes redundancy |
| Feature extraction | Creates basis for classification |

