I'll distribute the AI-related contributions among your team of 5, focusing on the model development, training, and AI concepts (not frontend or server setup).

---

## 👥 **Team Contribution Distribution**

### **🎯 Project: Bone Fracture Detection using YOLOv8**

---

## **1. SAHIL- Dataset Preparation & Data Engineering**

### **Responsibilities:**

#### **A. Dataset Collection & Organization**
```python
# Tasks:
- Downloaded bone fracture dataset from Roboflow
- Organized dataset structure:
  ├── train/ (3,631 images)
  ├── valid/ (348 images)
  └── test/ (169 images)
- Verified image-label pairs
```

#### **B. Data Configuration**
```python
# Created data.yaml configuration file
dataset_yaml = """
names:
- elbow positive
- fingers positive
- forearm fracture
- humerus
- humerus fracture
- shoulder fracture
- wrist positive

nc: 7

train: path/to/train/images
val: path/to/valid/images
test: path/to/test/images
"""
```

#### **C. Data Preprocessing**
* Ensured all images are in correct format (JPG)
* Verified label files (YOLO format: class x_center y_center width height)
* Handled missing or corrupt images
* Created train/validation/test splits

#### **AI Concepts Applied:**
* ✅ **Data Representation** (Unit-I)
* ✅ **Problem Formulation** - Defining the dataset structure
* ✅ **Knowledge Acquisition** - Gathering training data

---

## **2. KEDAR- Model Training & Optimization**

### **Responsibilities:**

#### **A. Model Selection & Training**
```python
# Implemented in yolo_model.ipynb

# 1. Load pre-trained YOLOv8 model
from ultralytics import YOLO
model = YOLO('yolov8s.pt')  # Transfer learning

# 2. Configure training parameters
model.train(
    data="path/to/data.yaml",
    epochs=100,              # Training iterations
    batch=16,                # Batch size
    imgsz=416,               # Image size
    amp=True                 # Mixed precision training
)
```

#### **B. Hyperparameter Tuning**
* Experimented with different:
  - Learning rates (lr0=0.01)
  - Batch sizes (16)
  - Image sizes (416×416)
  - Epochs (100)
  - Optimizer (AdamW)

#### **C. Training Monitoring**
```python
# Monitored training metrics:
- Box loss (localization accuracy)
- Class loss (classification accuracy)
- DFL loss (distribution focal loss)
- mAP50, mAP50-95 (detection performance)
```

#### **AI Concepts Applied:**
* ✅ **Intelligent Agents** - Model as an agent learning from environment
* ✅ **Knowledge Representation** - Neural network weights
* ✅ **Transfer Learning** - Using pre-trained COCO weights
* ✅ **Supervised Learning** - Training with labeled data

---

## **3. NANDIKA - Model Evaluation & Performance Analysis**

### **Responsibilities:**

#### **A. Model Validation**
```python
# Implemented validation metrics

# 1. Run validation
metrics = model.val()

# 2. Extract performance metrics
print("Precision:", results.box.p[0])
print("Recall:", results.box.r[0])
print("F1 Score:", results.box.f1[0])
print("mAP50:", results.maps[0])
print("mAP50-95:", results.maps[1])
```

#### **B. Confusion Matrix Analysis**
```python
# Created confusion matrix to analyze model performance

from sklearn.metrics import confusion_matrix, accuracy_score, f1_score

# Generate predictions on validation set
true_labels = []
pred_labels = []

for img_path, label_path in zip(image_files, label_files):
    prediction = model.predict(source=img_path)
    pred_label = prediction[0].boxes.cls[0].item()
    
    with open(label_path, 'r') as f:
        true_label = int(f.readlines()[0].split()[0])
    
    true_labels.append(true_label)
    pred_labels.append(pred_label)

# Calculate metrics
conf_matrix = confusion_matrix(true_labels, pred_labels)
accuracy = accuracy_score(true_labels, pred_labels)
f1 = f1_score(true_labels, pred_labels, average="weighted")
f2 = fbeta_score(true_labels, pred_labels, beta=2, average="weighted")
```

#### **C. Visualization of Results**
```python
# Created performance visualizations

import matplotlib.pyplot as plt
import seaborn as sns

# Confusion matrix heatmap
plt.figure(figsize=(8, 6))
sns.heatmap(conf_matrix, annot=True, fmt='d', cmap='Blues')
plt.xlabel('Predicted Labels')
plt.ylabel('True Labels')
plt.title('YOLO Model - Confusion Matrix')
plt.savefig('confusion_matrices.png')
```

#### **D. Per-Class Performance Analysis**
* Analyzed which fracture types are detected best
* Identified confusion between similar classes (e.g., humerus vs humerus fracture)
* Documented model strengths and weaknesses

#### **AI Concepts Applied:**
* ✅ **Performance Measure** - Evaluating agent rationality
* ✅ **Confusion Matrix** - Understanding model knowledge
* ✅ **Accuracy, Precision, Recall, F1, F2 scores**
* ✅ **Statistical Analysis**
---

## **4. SHRINIVAS - Inference Pipeline & Detection Logic**

### **Responsibilities:**

#### **A. Model Loading & Inference**
```python
# Implemented in app.py

# 1. Load trained model
from ultralytics import YOLO
model = YOLO('models/model.pt')

# 2. Run inference on new images
def detect_fractures(image_path):
    image = Image.open(image_path)
    image_array = np.array(image)
    
    # Run YOLOv8 inference
    results = model(image_array)
    
    return results
```

#### **B. Detection Post-Processing**
```python
# Extract and process detection results

detections = []
for result in results:
    boxes = result.boxes.cpu().numpy()
    
    for i, box in enumerate(boxes):
        # Extract bounding box coordinates
        x1, y1, x2, y2 = box.xyxy[0]
        
        # Extract confidence score
        confidence = box.conf[0]
        
        # Extract class prediction
        class_id = int(box.cls[0])
        class_name = result.names[class_id]
        
        # Store detection
        detections.append({
            "id": i,
            "class": class_name,
            "confidence": float(confidence),
            "box": {
                "x1": float(x1),
                "y1": float(y1),
                "x2": float(x2),
                "y2": float(y2)
            }
        })
```

#### **C. Confidence Thresholding**
```python
# Filter detections based on confidence
MIN_CONFIDENCE = 0.25

filtered_detections = [
    d for d in detections 
    if d["confidence"] >= MIN_CONFIDENCE
]
```

#### **D. Result Visualization**
```python
# Draw bounding boxes on image
results_plotted = results[0].plot()
cv2.imwrite(output_path, results_plotted)
```

#### **AI Concepts Applied:**
* ✅ **Intelligent Agent Actions** - Model making predictions
* ✅ **Probabilistic Reasoning** - Confidence scores
* ✅ **Decision Making** - Thresholding
* ✅ **State Space** - Input image → Output detections
---

## **5. ATHARVA - Explainability & Visualization (Grad-CAM)**

### **Responsibilities:**

#### **A. Explanation Image Generation**
```python
# Implemented in app.py

def generate_explanation_image(image_array, detection_boxes):
    """
    Highlight detected fracture regions
    """
    explanation_img = image_array.copy()
    
    for box in detection_boxes:
        x1, y1, x2, y2 = [int(coord) for coord in box]
        
        # Draw green rectangle around detection
        cv2.rectangle(explanation_img, (x1, y1), (x2, y2), (0, 255, 0), 2)
        
        # Add red highlight overlay
        highlight = np.zeros_like(explanation_img, dtype=np.uint8)
        pad = 10
        cv2.rectangle(
            highlight, 
            (max(0, x1-pad), max(0, y1-pad)), 
            (min(explanation_img.shape[1], x2+pad), 
             min(explanation_img.shape[0], y2+pad)), 
            (0, 0, 255), 
            -1
        )
        
        # Blend highlight with image
        explanation_img = cv2.addWeighted(
            explanation_img, 1, highlight, 0.3, 0
        )
    
    return explanation_img
```

#### **B. Grad-CAM Heatmap Generation**
```python
def generate_gradcam(image, boxes, result):
    """
    Generate attention heatmap for detected fractures
    """
    # 1. Prepare image
    vis_img = image.copy()
    if len(vis_img.shape) == 2:
        vis_img = cv2.cvtColor(vis_img, cv2.COLOR_GRAY2RGB)
    
    # 2. Create empty heatmap
    height, width = vis_img.shape[:2]
    heatmap = np.zeros((height, width), dtype=np.float32)
    
    # 3. Generate Gaussian heatmap for each detection
    for box in boxes:
        x1, y1, x2, y2 = [int(coord) for coord in box]
        center_x, center_y = (x1 + x2) // 2, (y1 + y2) // 2
        box_width, box_height = x2 - x1, y2 - y1
        
        # Create coordinate grids
        y, x = np.ogrid[:height, :width]
        
        # Calculate standard deviations
        sigma_x = max(box_width / 6, 10)
        sigma_y = max(box_height / 6, 10)
        
        # Gaussian distribution formula
        gaussian = np.exp(-(
            ((x - center_x) ** 2) / (2 * sigma_x ** 2) + 
            ((y - center_y) ** 2) / (2 * sigma_y ** 2)
        ))
        
        # Accumulate heatmaps
        heatmap = np.maximum(heatmap, gaussian)
    
    # 4. Apply JET colormap (red=hot, blue=cool)
    heatmap = np.uint8(255 * heatmap)
    heatmap_colored = cv2.applyColorMap(heatmap, cv2.COLORMAP_JET)
    
    # 5. Blend heatmap with original image
    gradcam_visualization = cv2.addWeighted(
        vis_img, 0.6,           # 60% original image
        heatmap_colored, 0.4,   # 40% heatmap
        0
    )
    
    # 6. Draw bounding boxes
    for box in boxes:
        x1, y1, x2, y2 = [int(coord) for coord in box]
        cv2.rectangle(
            gradcam_visualization, 
            (x1, y1), (x2, y2), 
            (0, 255, 0), 2
        )
    
    return gradcam_visualization
```

#### **C. Mathematical Implementation**
* Implemented 2D Gaussian distribution
* Applied colormap transformations
* Image blending and overlay techniques

#### **D. Visualization Pipeline**
```python
# Generate all three visualizations:
# 1. Result image (bounding boxes)
# 2. Explanation image (highlighted regions)
# 3. Grad-CAM heatmap (attention visualization)

if len(detection_boxes) > 0:
    # Explanation
    explanation_img = generate_explanation_image(image_array, detection_boxes)
    cv2.imwrite(f"results/explanations/{detection_id}_explanation.jpg", explanation_img)
    
    # Grad-CAM
    gradcam_img = generate_gradcam(image_array, detection_boxes, results[0])
    cv2.imwrite(f"results/gradcam/{detection_id}_gradcam.jpg", gradcam_img)
```

#### **AI Concepts Applied:**
* ✅ **Explainable AI** - Making model decisions interpretable
* ✅ **Spatial Knowledge Representation** - Heatmaps
* ✅ **Visualization Techniques** - Gaussian distributions
* ✅ **Attention Mechanisms** - Showing regions of interest
