# Image Stitching using SIFT and OpenCV

A Python application that combines multiple overlapping images into a seamless panoramic image using computer vision techniques (SIFT feature detection and homography transformation).

## 📋 What It Does

Takes a series of sequential photos with overlapping areas and automatically:
1. Detects unique points of interest in each image
2. Matches corresponding points between images
3. Calculates the transformation needed to align them
4. Blends them into one wide panoramic photo

**Real-world uses:** Panoramic photography, satellite map stitching, medical imaging, 3D scene reconstruction.

---

## 🔧 How It Works

### **Step 1: Load & Prepare Images**
```
Raw Images → Resize to 500×500 → Convert to Grayscale
```
- **Resize:** Speeds up processing while keeping quality
- **Grayscale:** SIFT works on single-channel images (faster computation)

---

### **Step 2: Find Distinctive Points (SIFT)**

**What is SIFT?** A feature detection algorithm that finds unique points in images that remain recognizable even if the image is rotated, scaled, or has different lighting.

**How it works:**
1. **Create scale pyramids** - Generate blurry versions at different zoom levels
2. **Find peaks & valleys** - Locate points that stand out at different scales
3. **Assign orientations** - Mark rotation direction of each point
4. **Create descriptors** - Build a 128-number "fingerprint" for each point

**Why SIFT is powerful:**
- ✅ Works at any zoom level (scale-invariant)
- ✅ Works with rotated images (rotation-invariant)
- ✅ Works with different lighting (illumination tolerant)
- ✅ Rarely gives false matches (distinctive)

---

### **Step 3: Match Points Between Images**

```
Image 1 Features ←→ Compare ←→ Image 2 Features
         ↓
    Best Matches Found
    (typically 50-500 matches)
```

Algorithm compares feature "fingerprints" to find which points in Image 1 correspond to which points in Image 2.

---

### **Step 4: Calculate Alignment (Homography)**

**What is Homography?** A mathematical relationship that tells us how to transform Image 2 to align with Image 1.

**Formula:**
```
[x']       [h11 h12 h13]   [x]
[y']   =   [h21 h22 h23] × [y]
[1]        [h31 h32 h33]   [1]
```

This 3×3 matrix encodes: rotation, scaling, skewing, and translation.

**How we find it (RANSAC algorithm):**
1. Pick 4 random matching points
2. Calculate homography from them
3. Count how many other points fit this transformation
4. Repeat 100+ times and keep the best result
5. **Advantage:** Ignores wrong matches (outliers)

---

### **Step 5: Warp & Blend**

```
Image 1 (Fixed) + Image 2 (Warped) → Blend Overlap → Final Panorama
```

- **Warp:** Transform Image 2 using the homography matrix
- **Blend:** Smooth the overlapping region so seams are invisible
- **Save:** Output as `stitchedOutput.jpg`

---

## 💻 Technical Details

### **Code Structure**

```python
# Step 1: Load & preprocess
images = [cv2.imread(path) for path in image_paths]
gray_images = [cv2.cvtColor(img, cv2.COLOR_BGR2GRAY) for img in images]

# Step 2: SIFT feature extraction
sift = cv2.SIFT_create()
features = [sift.detectAndCompute(img, None) for img in gray_images]

# Step 3 & 4: Matching & homography (handled internally)
stitcher = cv2.Stitcher.create()
error, result = stitcher.stitch(images)

# Step 5: Post-process & save
if error == cv2.Stitcher_OK:
    cv2.imwrite("stitchedOutput.jpg", result)
```

### **Key Libraries**

| Library | Purpose |
|---------|---------|
| **OpenCV** | Image processing, SIFT, stitcher |
| **NumPy** | Matrix operations |
| **Imutils** | Image resizing utilities |

---

## 📊 Complexity Analysis

| Operation | Time | Memory |
|-----------|------|--------|
| SIFT Detection | O(n log n) per image | O(k) keypoints |
| Feature Matching | O(k²) | O(k) |
| Homography (RANSAC) | O(iterations) | O(1) |
| Warping & Blending | O(width × height) | O(output size) |

**Overall:** Polynomial time, scales with image resolution

---

## 🚀 Requirements

```bash
pip install opencv-contrib-python numpy imutils
```

### **System Requirements**
- Python 3.6+
- 2GB RAM minimum (for high-res images)
- CPU with multi-core support (for faster processing)

---

## 📁 How to Use

1. **Prepare images:**
   - Place overlapping photos in a folder
   - Photos should have 30-50% overlap
   - Similar exposure/white balance helps

2. **Update path:**
   ```python
   image_paths = glob.glob(r'path/to/your/images/*.jpg')
   ```

3. **Run:**
   ```bash
   python imagestitch.py
   ```

4. **Get output:**
   - Check `stitchedOutput.jpg` in current directory

---

## ⚠️ Common Issues & Fixes

| Error | Cause | Solution |
|-------|-------|----------|
| `ERR_NEED_MORE_IMGS` | Too few matching points | Use clearer, sharper images |
| `ERR_HOMOGRAPHY_EST_FAIL` | Insufficient distinct features | Ensure images have texture/details |
| `ERR_CAMERA_PARAMS_ADJUST_FAIL` | Camera moved too much | Keep camera level, move sideways |
| Black borders in output | Perspective warping | Use `cv2.copyMakeBorder()` to clean up |

---

## 💡 Key Concepts Explained

**Why not just concatenate images?**
- ❌ Overlapping regions would be visible
- ❌ No smooth blending
- ❌ Different perspectives won't align

**Why feature matching instead of pixel-by-pixel?**
- ✅ Works with rotation/scaling
- ✅ Robust to lighting changes
- ✅ Handles partial overlaps
- ✅ Much faster computationally

**Why RANSAC for homography?**
- ✅ Automatically removes wrong matches
- ✅ Works with noisy data
- ✅ Only needs one good match set to work
- ✅ 95%+ accuracy in practice

---

## 🎯 What You Learn

✅ Scale-space image processing (Gaussian pyramids)
✅ Feature detection & descriptor extraction
✅ Geometric image transformations (homography)
✅ Robust estimation techniques (RANSAC)
✅ Image blending & compositing
✅ Computer vision workflow design

---

## 📚 References

- Lowe, D. G. (2004) - SIFT: Distinctive Image Features from Scale-Invariant Keypoints
- Fischler & Bolles (1981) - RANSAC: Random Sample Consensus
- OpenCV Documentation: https://docs.opencv.org/

---

## 🎨 Example Use Cases

| Use Case | Advantage |
|----------|-----------|
| **Panoramic Photos** | Create 180° views from multiple phones |
| **Satellite Maps** | Stitch aerial photos for large area coverage |
| **Medical Imaging** | Combine histology slide scans into mosaics |
| **Augmented Reality** | Generate background textures |
| **3D Reconstruction** | First step of photogrammetry pipeline |

---

**Author:** Deepali B  
**Created:** September 2024  
**Language:** Python 3.x  
**Status:** ✅ Fully Functional | 📖 Well Documented
