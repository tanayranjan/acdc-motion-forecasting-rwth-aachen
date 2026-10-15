# Motion Forecasting for Autonomous Driving: Argoverse 2

![Argoverse 2](https://img.shields.io/badge/Dataset-Argoverse_2-blue)
![Framework](https://img.shields.io/badge/PyTorch-2.8.0-EE4C2C?logo=pytorch&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white)

Anticipating the future movements of dynamic traffic agents is a critical link between environment perception and safe path planning. Developed as part of the Automated and Connected Driving Challenges (ACDC) research project at RWTH Aachen University, this repository contains a comprehensive suite of trajectory prediction models trained and evaluated on the Argoverse 2 (AV2) Motion Forecasting dataset.

## Repository Contents
* **`report.ipynb`**: The primary executable notebook containing the complete data preprocessing, model architectures, training loops, and evaluation scripts for all vehicle and pedestrian models.
* **`social_lstm_human.pth`**: Pre-trained weights for the pedestrian-specific Map-Aware Social-LSTM model.
* **`Baseline_LSTM_preprocessing_output/lstm_best_weights_200k_tuned.pt`**: Pre-trained weights for the baseline vehicle LSTM.
* **`Map_aware_output/multimodal_social_map_aware_V4.pt`**: Pre-trained weights for the multimodal vehicle architecture.

## Architecture & Models

This project tackles the fundamentally different dynamics of vehicles and pedestrians through specialized architectural branches.

### Pedestrian Motion Forecasting
Pedestrians exhibit short-range, highly irregular movements that require a tight, localized spatial context. 
* **Map-Aware Social-LSTM**: A dual-branch deep learning architecture that explicitly decomposes social and scene information into dedicated encoders.
* **Context Fusion**: Combines grid-based social pooling (capturing human-human interactions) with hand-crafted HD map features over a 2-second history/horizon.
* **Performance**: Achieved an Average Displacement Error (ADE) of 0.2187 m, a Final Displacement Error (FDE) of 0.5107 m, and a 23.67% miss rate at a 0.8 m threshold.

### Vehicle Motion Forecasting
Vehicles cover larger distances and are strictly bounded by lane geometry. The agent-agnostic pipeline progressed through increasing levels of complexity:
1. **Kinematic Baselines**: Constant Velocity (CV) and Constant Acceleration (CA).
2. **Vanilla Seq2Seq LSTM**: A single-agent temporal motion pattern learner.
3. **Social MHA LSTM**: Integrates cross-attention social pooling to model vehicle-vehicle interactions.
4. **Map-Aware Multimodal Model**: Combines cross-attention pooling with a 1D-CNN encoder over the nearest 16 map lane centerlines to output six diverse future trajectories ($K=6$) using a winner-take-all loss.

## Results & Critical Analysis
While the architectures successfully fused social and map context, edge-case evaluation revealed critical limitations in standard motion forecasting methodologies:

* **The ADE/FDE Fallacy**: Absolute metric tracking (ADE/FDE) is misleading for pedestrians. Because human motion covers a small spatial radius, absolute error values inherently remain low, masking behavioral failures (e.g., predicting a stationary state for a moving agent due to missing history data).
* **Architectural Constraints**: Grid-based social pooling occasionally enforced overly rigid repulsion, failing to predict agents cutting across paths. Mean-pooling map context geographically blurred crosswalk boundaries, dampening their attractive signal.
* **Single-Mode Output Limitations**: The pedestrian model's single-mode deterministic output struggled to capture the inherent multimodality of human intent.

## Installation & Setup

1. Clone the repository and install dependencies:
   ```bash
   pip install numpy==1.22.3 pandas==2.0.0 pyarrow==21.0.0 torch==2.8.0 matplotlib==3.4.3 av2==0.3.6

2. Download the Argoverse 2 Motion Forecasting Dataset and place the `train/`, `val/`, and `test/` folders inside a `Datasets/` directory at the project root.
3. Run `report.ipynb` top-to-bottom.
   - Note: Training cells are set to DEMO_MODE = True by default (2 epochs on 100 scenarios) to prevent overwriting the provided pre-trained weights. Set to False to reproduce full-scale training.
  
## Authors
- Tanay Ranjan - tanay.ranjan@rwth-aachen.de
- Ishita Jain - ishita.jain@rwth-aachen.de
- Manish Rajasekar - manish.rajasekar@rwth-aachen.de
