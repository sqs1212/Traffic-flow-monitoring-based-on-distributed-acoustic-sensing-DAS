# DAS-Based Classified Traffic Monitoring

Code and dataset for the paper:

> **“DAS-Based Classified Traffic Monitoring: Adaptive Preprocessing and a Lightweight Dual-Model Framework”**

This repository implements a **DAS-based (Distributed Acoustic Sensing)** traffic monitoring system that jointly performs:

- **Vehicle trajectory detection** on DAS waterfall plots  
- **Vehicle type classification** on time–frequency DAS signals  

via an **adaptive preprocessing pipeline (DAS-PCSE)** and **two lightweight neural networks** designed for edge / real-time deployment.

> ⚠️ **Code release statement**  
> The **complete training and inference code** will be made public **within 7 working days after the paper is formally accepted**.  
> Before that, this repo may only contain partial scripts / documentation.

---

## 1. Method Overview

Our framework decomposes DAS-based Classified Traffic Volume (CTV) monitoring into two subtasks:

1. **Adaptive signal preprocessing – DAS-PCSE**

   - **DAS-PCSE**: *Phase-Consistent Superlet Enhancement with Envelope Demodulation*  
   - Based on Enhanced Superlet Transform (EST), with:
     - Band-pass focusing in the **10–80 Hz** band  
     - Phase-consistent Superlet enhancement  
     - Envelope demodulation for robust amplitude representation  
   - Converts **low-SNR raw DAS traces** into **more discriminative, normalized inputs** for both trajectory detection and vehicle classification.

2. **Dual-model lightweight architecture**

   - **DAS Trajectory Net**
     - Input: preprocessed DAS **waterfall images**  
     - Architecture: encoder–decoder backbone + **Adaptive Feature Enhancement (AFE)** module  
     - Task: per-pixel **vehicle trajectory detection / segmentation** on optical fiber.
     - Achieves around **69% IoU** with **13.41M parameters** on the self-built dataset.
   
   - **DAS Vehicle Classifier Net**
     - Input: local **time–frequency EST-envelope signals**  
     - Core modules: **Dual Stream Signal (DSS)** block + **Light Conv** for efficient spatiotemporal fusion  
     - Task: **vehicle type classification** (e.g., passenger cars vs. heavy trucks, etc.)  
     - Achieves **93.31% classification accuracy** with only **0.37M parameters**.

3. **Real-time performance**

   - Total per-sample inference latency (two models combined) is **below 50 ms** on our test platform, showing feasibility for **real-time, wide-area DAS-based CTV monitoring**.

---

## 2. Dataset Description

This repository will provide **two self-built DAS datasets** derived from real road segments.  
**Note:** The data are **not raw DAS backscatter**; they are **signals after Enhanced Superlet Transform and envelope extraction**.

### 2.1 Trajectory Detection Dataset

- **Total samples:** `2,646`  
- **Train / Val split:**  
  - Train: `2,117` samples  
  - Val: `529` samples  
  - Random split with ratio approximately `8 : 2`
- **Input size:**  
  - Each sample is a **1024 × 1024** image (preprocessed DAS waterfall plot).
- **Annotations:**
  - Trajectory ground-truth masks are obtained using **video-assisted labeling**,  
    ensuring **consistent and reliable segmentation supervision**.
- **Usage:**
  - Used to train **DAS Trajectory Net** for fiber-trajectory segmentation.

### 2.2 Vehicle Classification Dataset

- Due to the geographic location of the monitored road segment, the number of **large trucks** is relatively small.  
  To alleviate **class imbalance**, we apply **Mixup**-based data augmentation **within each class**.

- **After augmentation:**
  - Train: `12,111` samples  
  - Val: `2,332` samples  
- **Input size:**
  - Each sample is a **4096 × 25** tensor.
- **Signal type:**
  - These are **not original time-series**;  
  - They are **EST-based time–frequency representations**, where **envelopes** are extracted after Enhanced Superlet Transform, then used as input to the vehicle classification network.
- **Usage:**
  - Used to train **DAS Vehicle Classifier Net** for vehicle type recognition.

---
