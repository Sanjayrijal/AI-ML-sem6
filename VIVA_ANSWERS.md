# Image Processing & PCA - Viva Answers

---

## **Exercise 1: Basic Image Operations - Answers**

### **1. What is PIL (Pillow) and why is it used?**
**Answer:** 
- PIL (Pillow) is the Python Imaging Library - a Python library for opening, manipulating, and saving images
- **Advantages:**
  - Simple API for image I/O (imread, imread)
  - Support for multiple formats (JPEG, PNG, BMP, GIF, etc.)
  - Image transformation operations (rotate, resize, crop, filter)
  - Lightweight compared to OpenCV for basic operations
  - Easy integration with NumPy arrays
- Better than basic file I/O because it automatically handles image formats and compression

### **2. RGB Color Model Explanation**
**Answer:**
- **R (Red):** Red channel - values 0-255 indicate red intensity
- **G (Green):** Green channel - values 0-255 indicate green intensity
- **B (Blue):** Blue channel - values 0-255 indicate blue intensity
- **Pure colors:**
  - Pure Red: [255, 0, 0]
  - Pure Green: [0, 255, 0]
  - Pure Blue: [0, 0, 255]
  - White: [255, 255, 255] (all channels maximum)
  - Black: [0, 0, 0] (all channels minimum)
  - Gray: [128, 128, 128] (all channels equal)

### **3. Why Convert Images to NumPy Arrays?**
**Answer:**
- **Benefits of NumPy arrays:**
  - Efficient indexing: `array[i, j, k]` vs PIL's slow getpixel()
  - Vectorized operations: Can apply operations to entire regions at once
  - Mathematical operations: Can use NumPy's functions (mean, std, dot product)
  - Integration with scientific libraries: Works with matplotlib, sklearn, scipy
  - Memory efficient: Contiguous memory layout
- **Difference:** PIL operations are slower for batch operations; NumPy is optimized for numerical computing

### **4. What is reshape in image processing?**
**Answer:**
- Reshape changes array dimensions without changing data
- `image.reshape(28, 28)`: Converts 784 flat values → 28×28 2D grid
- `image.reshape(-1)`: Flattens 2D → 1D array (useful for machine learning)
- `-1` means "infer this dimension"
- **Use case:** MNIST flattens images from 28×28=784 pixels for ML algorithms

---

## **Exercise 1: Code-Specific Answers**

### **5. Image Array Shape and Data Type**
**Answer:**
```python
image = Image.open(image_path)
image_array = np.array(image)
```
- **For color image:** Shape = (height, width, 3) → e.g., (800, 600, 3)
- **For 800×600 image:** Shape = (800, 600, 3) → 1.44 million pixels
- **Data type:** `uint8` (8-bit unsigned integer, values 0-255)
- Example with actual numbers:
  - Image dimensions: 800×600
  - Array shape: (800, 600, 3)
  - Total elements: 1,440,000
  - Memory: ~1.4 MB

### **6. Array Slicing Explanation**
**Answer:**
```python
top_left_corner = image_array[:100, :100]
```
- **`:100`** means "from 0 to 100 (exclusive)" - selects first 100 rows and columns
- **Shape of `top_left_corner`:** (100, 100, 3)
- **`image_array[50:150, 50:150]`** extracts:
  - Rows 50-150 (100 rows)
  - Columns 50-150 (100 columns)
  - Shape: (100, 100, 3) - center square
- **More examples:**
  - `image_array[::2, ::2, :]` → Every 2nd pixel (downsampling)
  - `image_array[:, :, 0]` → Only red channel

### **7. Color Channel Separation**
**Answer:**
```python
red_channel = image_array[:, :, 0]    # Red
green_channel = image_array[:, :, 1]  # Green
blue_channel = image_array[:, :, 2]   # Blue
```
- **Third index (0, 1, 2):** Selects RGB channels
  - 0 = Red channel
  - 1 = Green channel
  - 2 = Blue channel
- **Shape of each channel:** (height, width) → 2D array, not 3D
- **`:, :` for first two dimensions:** "Select all rows and all columns" (complete spatial dimensions)
- **Output:** Grayscale representations of each color

### **8. Colormap Explanation**
**Answer:**
```python
plt.imshow(red_channel, cmap='Reds')
```
- **`cmap='Reds'`:** Maps intensity values to red shades
  - Low values → black
  - High values → bright red
- **`cmap='gray'`:** Maps to grayscale (black to white)
- **Why different maps?**
  - Better visualization for understanding channel contribution
  - Red channel displayed as red helps see red intensity
  - Easier human interpretation
  - Can use 'Blues', 'Greens', 'hot', 'cool', 'viridis', etc.

### **9. Pixel Modification**
**Answer:**
```python
image_array[:100, :100] = 210
```
- **Does it modify all channels?** YES - sets R, G, B all to 210
  - Equivalent to: RGB (210, 210, 210) = light gray
- **When all channels = 210:** Light gray color appears
- **`image_array[:100, :100] = [255, 0, 0]`:**
  - Sets R=255, G=0, B=0 → Pure RED in top-left corner
  - More sophisticated: Can use different values per channel

### **10. Image Reconstruction**
**Answer:**
```python
modified_image = Image.fromarray(image_array)
```
- **fromarray():** Converts NumPy array back to PIL Image
- **Required data type:** `uint8` (0-255 integer values)
- **Without uint8:** May cause errors or incorrect colors
- **If values exceed 255:** 
  - ValueError if dtype is uint8
  - Wrap-around if not clipped (256→0, 257→1)
- **Best practice:** Always use `np.clip(array, 0, 255).astype(uint8)`

---

## **Exercise 2: Grayscale & Transformations - Answers**

### **11. Grayscale vs RGB**
**Answer:**
- **Grayscale:** Single channel (intensity only) - values 0-255
  - Shape: (height, width)
  - Storage: 1/3 size of RGB
- **RGB:** Three channels (R, G, B)
  - Shape: (height, width, 3)
- **When to use grayscale:**
  - Edge detection
  - Object recognition (less information needed)
  - Faster processing
  - Reduced storage
  - When color isn't relevant

### **12. Image Thresholding**
**Answer:**
- **Purpose:** Convert grayscale → binary image (0 or 255 only)
- **Creates binary image:** All pixels either black or white (no gray)
- **Formula:** if pixel < threshold: 0, else: 255
- **Common thresholds:**
  - **128:** Middle value (standard)
  - **100:** Lower (more white areas)
  - **200:** Higher (more black areas)
- **Applications:**
  - Object detection
  - Document scanning
  - Medical imaging
  - Pattern recognition

### **13. Image Rotation**
**Answer:**
- **Why -90 for clockwise?** Mathematical convention:
  - Positive angles = counter-clockwise (standard math)
  - Negative angles = clockwise
  - -90° = rotate 90° clockwise
- **Affects pixel coordinates:** Original (x, y) → Rotated (y, max_x - x)
- **Rotation effects on size:** May increase canvas size if angle is arbitrary

### **14. Grayscale to RGB Conversion**
**Answer:**
```python
rgb_image = Image.merge("RGB", (image, image, image))
```
- **Why replicate values across channels:**
  - Preserves intensity information
  - (128, 128, 128) = gray
  - No color information added
  - Reversible: Can convert back to grayscale
- **Example:** If gray value = 100
  - RGB becomes (100, 100, 100)
  - Appears as dark gray
- **Could use different images:**
  - `Image.merge("RGB", (red_img, green_img, blue_img))` → Creates false-color image

---

## **Exercise 2: Code-Specific Answers**

### **15. Grayscale Conversion**
**Answer:**
```python
image = Image.open(image_path).convert('L')
```
- **`.convert('L')`:** Converts image to grayscale
  - 'L' = Luminosity (standard grayscale)
  - Uses formula: Gray = 0.299×R + 0.587×G + 0.114×B
  - Weights match human eye sensitivity
- **`.convert('RGB')`:** Converts to RGB (triples the data if grayscale)

### **16. Cropping Middle Section**
**Answer:**
```python
height, width = image_array.shape
start_row = (height - 150) // 2
end_row = start_row + 150
cropped_image = image_array[start_row:end_row, :]
```
- **`(height - 150) // 2`:** Centers the crop
  - Example: height=600 → (600-150)//2 = 225
  - Crops rows 225 to 375 (150 rows centered)
- **`//` vs `/`:** 
  - `//` = integer division (floor)
  - `/` = float division (would give 225.5, invalid for indexing)
- **Cropped shape:** (150, width) → e.g., (150, 600)
- **`:` for columns:** Keeps all columns (full width)

### **17. Thresholding**
**Answer:**
```python
thresholded_image = np.where(image_array < 100, 0, 255)
```
- **`np.where(condition, true_val, false_val)`:**
  - If pixel < 100 → output 0 (black)
  - Else → output 255 (white)
  - Creates binary image
- **Output values:** Only 0 and 255, no intermediate values
- **Shape:** Same as input
- **With threshold=128:**
  - Pixels 0-127 → 0 (black)
  - Pixels 128-255 → 255 (white)
  - Different distribution depending on image

### **18. Rotation**
**Answer:**
```python
rotated_image = image.rotate(-90)
```
- **Negative angle:** Rotates clockwise
  - `.rotate(90)` → counter-clockwise
  - `.rotate(-90)` → clockwise (270° counter-clockwise)
- **`.rotate(45)`:** Diagonal rotation
  - May increase canvas (to avoid cropping)
  - Creates transparent corners by default
- **Operating on Image vs NumPy:**
  - Requires PIL Image object (not NumPy array)
  - NumPy approach: Use `np.rot90()` on arrays

### **19. Color Merging**
**Answer:**
```python
rgb_image = Image.merge("RGB", (image, image, image))
```
- **Three inputs:** Tuple of three 'L' mode (grayscale) images
- **If using different images:**
  - `Image.merge("RGB", (red, zero, zero))` → Show only red component
  - `Image.merge("RGB", (img1, img2, img3))` → False-color composite
  - Creates synthetic color image
- **Example - Temperature map:** Red=high temp, Blue=low temp

---

## **PCA Theory - Answers**

### **20. PCA and Image Compression**
**Answer:**
- **PCA:** Principal Component Analysis finds directions of maximum variance
- **Purpose:** Reduce dimensionality while retaining information
- **Image compression:**
  - Pixel correlations → Fewer components needed
  - Store: Eigenvectors + projected coefficients + mean
  - Reconstruction: Multiply back and add mean
- **Example:** 256×256 image
  - Original: 65,536 pixels
  - With k=50: Store 256×50 + 50 coefficients (much smaller if compressed)

### **21. Eigenvalues and Eigenvectors**
**Answer:**
- **Eigenvectors:** Directions in image space with most variance
  - Each eigenvector = one "face" or pattern
  - First eigenvector = direction of maximum variance
- **Eigenvalues:** Amount of variance in each eigenvector direction
  - Larger eigenvalue = more important component
  - Sum all eigenvalues = total variance
- **Why sort descending:**
  - First k components capture most information
  - Can discard low-variance components with minimal loss

### **22. Explained Variance**
**Answer:**
- **Explained variance:** % of total information captured by k components
- **Formula:** (sum of k largest eigenvalues) / (sum of all eigenvalues)
- **95% explained variance:** Retaining 95% of total variation
  - Lose only 5% of information
  - Much smaller storage needed
  - Good compression ratio
- **Trade-off:** 80% variance needs fewer k, but loses more detail

### **23. Why Center Data?**
**Answer:**
```python
mean = np.mean(image_array, axis=1, keepdims=True)
standardized_data = image_array - mean
```
- **Subtracting mean:**
  - Removes overall brightness offset
  - Centers data around origin
  - Covariance matrix measures deviations from mean
- **Effect on covariance:**
  - Without centering: Eigenvalues biased by mean
  - With centering: Eigenvectors represent true variations
- **Why important:** PCA assumes zero-mean distribution

### **24. Reconstruction with Fewer Components**
**Answer:**
- **Blurriness caused by:** Loss of high-frequency details
  - Low eigenvalue components = fine details
  - Discarding them removes sharp edges
  - Results in blurred image
- **MSE increases:** Reconstruction error grows
  - MSE = mean((original - reconstructed)²)
  - Fewer components → larger differences
  - MSE quadratically penalizes large errors
- **Trade-off:**
  - k=1: Very blurry, small storage ✓
  - k=100: Sharp, large storage ✗
  - k=30: Good balance

---

## **PCA: Code-Specific Answers**

### **25. Data Centering**
**Answer:**
```python
mean = np.mean(image_array, axis=1, keepdims=True)
standardized_data = image_array - mean
```
- **`axis=1`:** Average along columns (for each row)
  - Gives mean brightness per row
  - Shape of mean: (height, 1)
- **`keepdims=True`:** Keeps dimensions as (height, 1) not (height,)
  - Allows broadcasting in subtraction
  - Without it: Would need to reshape
- **Shape of mean:** (height, 1) e.g., (256, 1)
- **Why subtract:** Removes illumination variations
  - Different lighting shouldn't affect pattern recognition

### **26. Covariance Matrix**
**Answer:**
```python
cov_matrix = np.cov(standardized_data)
```
- **Shape of covariance matrix:** (height, height) e.g., (256, 256)
  - Doesn't change with image width
  - Measures correlation between rows
- **Each element:** Represents covariance between two rows
  - High value = rows are similar
  - Low value = rows are different
- **Interpretation:** 
  - Diagonal = variance of each row
  - Off-diagonal = correlation between rows

### **27. Eigenvalue Computation**
**Answer:**
```python
eigenvalues, eigenvectors = np.linalg.eigh(cov_matrix)
sorted_indices = np.argsort(eigenvalues)[::-1]
```
- **`eigh()` vs `eig()`:**
  - `eigh()` = for Hermitian/symmetric matrices (faster, more stable)
  - `eig()` = general matrices
  - Covariance is symmetric → use `eigh()`
- **`[::-1]`:** Reverses order (descending)
  - `argsort()` gives ascending order
  - `[::-1]` flips to descending
- **Why sort both:**
  - Eigenvalues need to be in same order as eigenvectors
  - If we sort eigenvalues, must reorder eigenvectors accordingly
  - Ensures matching pairs

### **28. Explained Variance Computation**
**Answer:**
```python
explained_variance_ratio = eigenvalues / np.sum(eigenvalues)
cumulative_variance = np.cumsum(explained_variance_ratio)
```
- **Division by sum:** Normalizes to [0, 1]
  - Each ratio = contribution of one eigenvalue
  - Sum of all ratios = 1.0 (100%)
- **`np.cumsum()`:** Cumulative sum
  - First element: 0.3 (30% from first component)
  - Second element: 0.55 (55% total from first two)
  - Last element: Always 1.0 (100%)
- **Values in cumulative_variance:** Monotonically increasing from 0 to 1

### **29. Finding k Components**
**Answer:**
```python
k = np.argmax(cumulative_variance >= threshold) + 1
```
- **`cumulative_variance >= threshold`:** Boolean array
  - True where variance ≥ threshold, False elsewhere
- **`argmax()`:** Index of first True value
  - First component where cumulative ≥ threshold
- **`+ 1`:** Convert from 0-indexed to count
  - Index 29 → need 30 components (indices 0-29)
- **Example:**
  - threshold = 0.95
  - cumulative = [0.1, 0.2, ..., 0.94, 0.951, ...]
  - argmax = 49 (first value ≥ 0.95)
  - k = 50

### **30. Projection Step**
**Answer:**
```python
top_eigenvectors = eigenvectors[:, :k]
projected = np.dot(top_eigenvectors.T, standardized_data)
```
- **`eigenvectors[:, :k]`:** Select first k eigenvectors
  - Shape: (height, k) e.g., (256, 50)
  - Contains top k "basis" vectors
- **`.T` (transpose):** (256, 50) → (50, 256)
- **Shape before projection:** (256, width) e.g., (256, 256)
- **After projection:** (50, 256)
  - Reduced from 256 dimensions to 50
  - Represents image in "compressed" subspace
- **Intuition:** Project high-dim image onto low-dim subspace

### **31. Reconstruction**
**Answer:**
```python
reconstructed = np.dot(top_eigenvectors, projected) + mean
reconstructed = np.clip(reconstructed, 0, 255)
```
- **`top_eigenvectors × projected`:**
  - (256, 50) · (50, 256) = (256, 256)
  - Maps from compressed back to original space
- **`+ mean`:** Adds back the removed average
  - Returns to original scale
  - Without it: Everything centered at 0
- **`np.clip()`:** Fixes overflow/underflow
  - If reconstruction > 255 → set to 255
  - If reconstruction < 0 → set to 0
  - Ensures valid pixel range
- **Without clipping:** Invalid pixel values cause errors

### **32. MSE Calculation**
**Answer:**
```python
mse = np.mean((image_array - reconstructed) ** 2)
```
- **MSE:** Measures average reconstruction error
- **Squaring:** Penalizes large errors more than small ones
  - Error of 10 = cost 100
  - Error of 1 = cost 1
- **Lower MSE:** Better reconstruction quality
- **Typical values:**
  - k=10: MSE ~ 1000 (high error)
  - k=50: MSE ~ 100 (medium error)
  - k=200: MSE ~ 1 (low error)

---

## **Critical Analysis - Answers**

### **33. k=10 vs k=100 Comparison**
**Answer:**
| Metric | k=10 | k=100 |
|--------|------|-------|
| MSE | ~1000 (High, bad) | ~50 (Low, good) |
| Explained Variance | ~70% | ~98% |
| Storage | Small ✓ | Larger ✗ |
| Quality | Blurry ✗ | Sharp ✓ |
| Compression Ratio | High ✓ | Low ✗ |

**Best choice depends on application:**
- Storage-critical → k=10
- Quality-critical → k=100

### **34. Maximum Components Limit**
**Answer:**
- **Can't use k > image_width:** Only (width) independent components exist
  - Eigenvectors have dimension = height
  - Maximum meaningful k = min(height, width)
- **At k = width:** Perfect reconstruction (100% variance)
- **Practical limit:** k < height-1 (covariance matrix rank)
- **Example:** 256×256 image
  - Can have at most 256 meaningful components
  - Extra dimensions are zero/noise

### **35. PCA on Color Images**
**Answer:**
**Two approaches:**
1. **Separate per channel:**
   - Apply PCA to R, G, B independently
   - Pros: Natural, preserves color
   - Cons: Misses cross-channel correlations
   - Code: separate_pca(R), separate_pca(G), separate_pca(B)

2. **Flatten all channels:**
   - Reshape (H, W, 3) → (H×3, W)
   - Apply single PCA
   - Pros: Captures all correlations
   - Cons: Mixes color and spatial information
   - Better compression typically

**Better approach:** Per-channel (method 1)
- Keeps color information natural
- RGB not strongly correlated
- Easier interpretation

### **36. Minimum k for 90% Variance**
**Answer:**
```python
threshold = 0.90
k = np.argmax(cumulative_variance >= threshold) + 1
```
**Pseudocode:**
```
1. Compute eigenvalues (sorted descending)
2. Compute cumulative_variance = cumsum(eigenvalues) / sum(eigenvalues)
3. Find first index where cumulative_variance >= 0.90
4. k = index + 1
5. Return k
```
**Example output:** k=120 (needs 120 components for 90% variance)

### **37. When PCA Works Well**
**Answer:**
**Good scenarios:**
- Natural images (high correlation between pixels)
- Mammograms, medical images (patterns repeat)
- Faces (similar structure across images)

**Poor scenarios:**
- Random noise (no structure, all eigenvalues equal)
- Text (sharp edges, needs high-frequency components)
- Discrete data (not naturally continuous)

**Image properties helping PCA:**
- Spatial coherence (nearby pixels similar)
- Low-rank structure (few underlying patterns)
- Repetitive patterns

### **38. Storage Calculation**
**Answer:**
**Original 256×256 image:**
- Storage: 256 × 256 = 65,536 values
- Size: 65.5 KB (uint8) or 256 KB (float32)

**PCA with k=50:**
- Eigenvectors: 256 × 50 = 12,800 values
- Projected: 50 × 256 = 12,800 values
- Mean: 256 values
- **Total:** 25,856 values
- **Without compression:** ~101 KB (float32)
- **Compression ratio:** 65,536 / 25,856 = **2.53×**
- **Space saved:** ~60% reduction

**With additional compression (zip/gzip):** Can achieve 5-10× total compression

---

## **Summary Table**

| Concept | Key Formula | Output Shape |
|---------|------------|--------------|
| Read Image | `np.array(Image.open(...))` | (H, W, 3) |
| RGB Channel | `image[:, :, 0]` | (H, W) |
| Crop/Slice | `image[r1:r2, c1:c2]` | (r2-r1, c2-c1, 3) |
| Threshold | `np.where(img < T, 0, 255)` | (H, W) binary |
| Mean-center | `x - np.mean(x, axis=1, keepdims=True)` | Same shape |
| Covariance | `np.cov(data)` | (H, H) |
| Eigenvalues | `np.linalg.eigh(cov)` | (H,) sorted |
| Explained Var | `evals / sum(evals)` | (H,) summing to 1 |
| Projection | `eigs[:,:k].T @ data` | (k, W) |
| Reconstruct | `eigs[:,:k] @ proj + mean` | (H, W) |
| MSE | `mean((original - recon)²)` | Scalar |

