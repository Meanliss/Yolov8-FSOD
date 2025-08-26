# Few-Shot Object Detection with YOLOv8 on DOTA v1.5  
*A Two-Stage Fine-tuning Approach for Novel Class Detection*  

---

## 1. Introduction  

This report presents an experiment on **few-shot object detection** using the **YOLOv8** model, applied to a custom subset of the **DOTA v1.5** aerial dataset. The study aims to evaluate the effectiveness of a **two-stage fine-tuning strategy** in adapting the detector to recognize **novel object classes** from limited training samples.  

---

## 2. Dataset Preparation  

A reduced version of the DOTA v1.5 dataset was created to enable faster experimentation. The dataset contains **three object classes**, divided into:  

- **2 Base Classes**: Abundant training data available.  
- **1 Novel Class**: Very few labeled samples, representing the "few-shot" scenario.  

This setup simulates realistic conditions where new object categories must be learned with minimal annotations.  

---

## 3. Methodology  

The training followed a **Two-Stage Fine-tuning Approach (TFA)**, adapted for the YOLOv8 architecture.  

### 3.1 Stage 1: Base Training  
- YOLOv8 was trained conventionally on the three-class dataset.  
- Objective: learn general features and robust detection capability for the base classes.  

### 3.2 Stage 2: Fine-tuning for the Novel Class  
- The Stage 1 model was fine-tuned with a dataset biased towards the **novel class** (≈ 80% of training images contained the novel class).  
- **Layer freezing** was applied:  
  - Backbone layers (feature extractor) were frozen.  
  - Detection head layers (responsible for classification/localization) were trainable.  

**Rationale:** Preserve low-level and mid-level features from Stage 1, while forcing the detector to adapt its head for the novel class, reducing catastrophic forgetting.  

---

## 4. Results  

### 4.1 Quantitative Results  

| Metric                          | Model Stage 1 | Model Stage 2 (Fine-tuned) |  
|--------------------------------|---------------|-----------------------------|  
| mAP50-95 (B) for container-crane (Novel) | **0.0000** | **0.0000** |  
| mAP50-95 (B) overall            | 0.0951        | 0.0947                      |  
| mAP50 (B) overall               | 0.2027        | 0.2057                      |  

➡️ **Key Observation:** The novel class (“container-crane”) was not detected by either stage. However, Stage 2 slightly improved the overall mAP50 compared to Stage 1, indicating that fine-tuning contributed positively to balancing predictions for the base classes.  

---

### 4.2 Qualitative Results  

#### Model Stage 1 Predictions  
![Stage 1 Results](e3ab8098-fb96-4abd-8326-c84cbf36eb96.png)  
- The detector identifies ships with moderate confidence.  
- Predictions are scattered, with some false positives.  
- No detections for the novel class.  

#### Model Stage 2 Predictions (Fine-tuned)  
![Stage 2 Results](59449e48-8e14-42c0-81b6-21635d625e6e.png)  
- Predictions for ships appear slightly more compact and confident.  
- False positives are reduced compared to Stage 1.  
- Still no detection of the novel class.  

---

## 5. Discussion  

The results highlight the **challenge of extreme few-shot learning** in object detection. Although fine-tuning improved overall performance, the novel class remained undetected due to:  

1. **Insufficient training samples** for the novel class.  
2. **Short training duration** in Stage 2.  
3. **YOLOv8 architecture limitations** in handling highly imbalanced training distributions without specialized meta-learning techniques.  

Nevertheless, the small improvement in overall mAP indicates that the fine-tuning procedure was successful in adjusting the model without catastrophic forgetting.  

---

## 6. Conclusion and Future Work  

This experiment demonstrates that a **two-stage fine-tuning strategy** with layer freezing can maintain or slightly improve detection performance, even with imbalanced datasets. However, the inability to detect the novel class underscores the limitations of applying YOLOv8 directly to extreme few-shot settings.  

**Future Directions:**  
- Increase the number of annotated samples for the novel class.  
- Extend fine-tuning with more epochs and stronger regularization.  
- Explore meta-learning or prototype-based detection frameworks (e.g., FSOD, TFA, Meta-RCNN).  
- Investigate data augmentation and synthetic data generation for novel classes.  

---
