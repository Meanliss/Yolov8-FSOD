# Few-Shot Object Detection with YOLOv8 on DOTA v1.5
*An experiment on a two-stage training approach for few-shot learning.*

---

## 📝 Project Overview

This project demonstrates a few-shot learning approach for object detection, using the **YOLOv8** model on a custom version of the **DOTA v1.5** dataset. The goal is to test the model's ability to learn and recognize novel object classes from only a few examples.

Due to limited time and resources, a custom subset of the DOTA v1.5 dataset was created. This process involved:
* **Custom Subset Creation:** The original dataset was processed to create a smaller version containing only **3 object classes**.
* **Class Distribution:** These 3 classes are categorized as:
    * **2 Base Classes:** Common classes with abundant data.
    * **1 Novel Class:** A rare class with very limited data.

---

## ⚙️ Methodology & Technique

The methodology is inspired by the TFA (Two-stage Fine-tuning Approach), commonly seen in models like Faster R-CNN, and has been adapted for the YOLOv8 architecture.

### Stage 1: Base Training
The YOLOv8 model is trained conventionally on the entire 3-class dataset. This stage aims to equip the model with a foundational understanding of general features (edges, corners, textures) and the ability to reliably detect the Base Classes.

### Stage 2: Fine-tuning for the Novel Class
The "base model" from Stage 1 is then fine-tuned on a specially prepared dataset with an intentionally imbalanced ratio: **80%** of the images contain the Novel Class, while **20%** contain only the Base Classes.

A key technique used in this stage is **Layer Freezing**. Most of the network's layers (the backbone, responsible for feature extraction) are frozen, leaving only the final layers (the detection head, responsible for classification and localization) trainable.

> **The Logic Behind Layer Freezing:** This technique preserves all the feature knowledge the model acquired in Stage 1. By only updating the final layers, the model focuses its entire "learning capacity" on associating existing features with a new label (the Novel Class), rather than re-learning from scratch. This enhances training efficiency for the rare class and prevents "Catastrophic Forgetting" of the common classes.

---

## 📊 Results & Analysis

A performance comparison between the models from Stage 1 and Stage 2 reveals the following:

* **Regarding the Novel Class:** The model's ability to detect the rare class showed **no improvement**. It made almost no predictions for this class.
* **Regarding Overall Metrics (mAP):** Despite the issue with the novel class, the overall mAP (mean Average Precision) of the model **did increase** after Stage 2.

---

## 💡 Discussion & Future Work

The model's inability to detect the novel class can likely be attributed to an **insufficient number of samples and training epochs**.

However, the improvement in the overall mAP is a positive sign, suggesting that the fine-tuning approach is fundamentally sound and helped the model balance its predictions more effectively. This indicates the potential of the method: if scaled up with more data and longer training, the model's performance on the novel class would likely improve significantly.

**Future Development Directions:**
* Increase the number of images for the Novel Class in Stage 2.
