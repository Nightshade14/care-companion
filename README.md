# Care Companion: Comprehensive AI-Powered Medical Imaging and Patient Care Solution

## Value Proposition
The  Care Companion system will be seamlessly integrated into existing healthcare practices, providing doctors with enhanced diagnostic tools. In traditional healthcare systems, radiologists face challenges analyzing numerous X-rays and CT scans, leading to delays, burnout, and missed diagnoses. By incorporating AI-powered image analysis, doctors can quickly identify fractures from X-rays, and detect signs of pneumonia and tuberculosis in chest X-rays, improving diagnostic accuracy. This AI-powered system will save time for healthcare professionals, enabling them to focus on patient care. It will enhance workflow in hospitals, clinics, and radiology centers, allowing healthcare providers to deliver faster, more reliable results with Real-time predictions and continuous feedback. 

The system's success will be evaluated based on its ability to enhance diagnostic accuracy, reduce operational costs, and improve patient satisfaction. However, in medical AI, where human lives are at stake, performance metrics prioritize patient safety over raw accuracy. While precision and AUC-ROC are tracked, we focus on optimizing the model for high recall (sensitivity) to minimize false negatives—ensuring no critical fracture, pneumonia, or TB case goes undetected, even at the cost of more false positives. This aligns with clinical best practices, where follow-up tests for potential false alarms are preferable to missed diagnoses. By balancing recall with clinician trust, we reduce liability risks while enabling early interventions that cut long-term costs.

While the AI-powered detection system enhances diagnostic accuracy, it should not be solely relied upon for final decisions. The predictions made by the system are assistance tools, and healthcare professionals should always validate the results through their expertise and further testing. We also employ a warning system that provides the confidence for each prediction made while also alerting the radiologist  that the predictions made can be incorrect and needs a thorough review before finalizing the report. Additionally, we create a dashboard that allows the technicians to monitor the data drift for any potential flaws in the system. 

## Contributors
| Name                   | Responsible For | Link to their commits in this repo                                                       |
|------------------------|-----------------|------------------------------------------------------------------------------------------|
| Satyam Chatrola         | Model Serving and Monitoring |[Commits](https://github.com/Nightshade14/care-companion/commits/main/?author=Nightshade14) |
| Akash Peddaputha        | Data Pipeline |[Commits](https://github.com/Nightshade14/care-companion/commits/main/?author=akashp04)|
| Yash Patel              | Continuous X Pipeline |[Commits](https://github.com/Nightshade14/care-companion/commits/main/?author=YashPatel166)|
| Mamidala Sai Sandeep    | Model Training |[Commits](https://github.com/Nightshade14/care-companion/commits/main/?author=Sandeep2229)|

## System Design
![Image](https://github.com/Nightshade14/care-companion/blob/main/Images/systemdesign.png)

### Summary of Infrastructure Requirements
| **Requirement**      | **How many / When**                      | **Justification**                                                                                                                                     |
|----------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| `m1.medium`          | 1 instance, running 24/7                | Used for **backend tasks** such as **PostgreSQL database**, **logging**, and light processing that doesn't require heavy GPU acceleration. This is a lightweight node to handle non-GPU tasks. |
| `gpu_v100`           | 4-hour block, twice a week               | **High-performance model training** for deep learning tasks (e.g., CNN, Vision Transformers). The **V100** provides excellent compute power for training large models on complex datasets. |
| `gpu_a30`            | 4-hour block, 3-4 times a week for inference | **Low-latency inference** for real-time AI predictions (fracture detection, pneumonia, TB). The **A30** provides efficient inference for production with cost-effectiveness. It can handle large models and datasets. |
| **Floating IPs**     | 1-2 for public access to inference model | To provide access to the **AI-powered inference service** (e.g., API for predictions) via a **public-facing IP** for external users (doctors, hospitals, etc.). |
| **Persistent Storage**| 1TB SSD (for model, dataset, and logs)  | To store models, large datasets, and logs used in training and inference. The SSD ensures **fast read/write** speeds necessary for processing large medical images. |


### Detailed Design Plan

#### Model training
##### Strategy and Justification
###### Model-1 (Fracture Detection CNN):
`Model Choice`: We will start with pre-trained CNN backbones like DenseNet-121 (highest AUC on MURA dataset), with comparative testing of MobileNet (latency-optimized) and InceptionV3 (spatial feature strength) and fine-tune them on our fracture detection dataset. We will evaluate multiple architectures to identify the one that balances accuracy, latency, and model size.<br/>
`Training`: Fine-tune on MURA (40k X-rays) via transfer learning. Retrain weekly with 500–1k radiologist corrections using Ray Train on gpu_v100.

###### Model-2 (TB/Pneumonia CNN):
`Model Choice`: Similarly, for TB or pneumonia detection, we will fine-tune pre-trained models using the same approach but with a different dataset.<br/>
`Training`: Initialize with NIH ChestX-ray14 weights. 

###### Model-3 (LLM for Notes):
`Model Choice`: We will use a Llama-2-7B (open-source, locally deployed) fine-tuned with LoRA to generate patient-friendly notes. This model will require minimal fine-tuning, as it has already been trained on large amounts of textual data. <br/>
`Training`: Generate 5k synthetic reports (radiology notes ↔ patient summaries) for fine-tuning

`CNNs are highly suitable for image classification tasks because they are designed to automatically learn spatial hierarchies in data. This makes them especially effective for tasks like detecting fractures or identifying abnormalities in X-ray or CT images.` 
<br>
`Using an LLM like Llama allows us to generate human-readable summaries from complex medical terminology, making the information accessible to patients. Llama and other open-source LLMs come pretrained on vast amounts of textual data, reducing the amount of fine-tuning required for the specific task of converting technical notes to layman's terms.`

###### Workflow Integration:
`Feedback Loop`: Corrections from doctors update PostgreSQL → trigger retraining for CNNs. <br>
`Deployment`: CI/CD (GitHub Actions) promotes models to gpu_a30 (CNNs) and m1.medium (LLM).


#### Training Platforms

We will use MLflow for tracking our model training experiments on Chameleon Cloud. MLflow will serve as our centralized experiment tracking server, where we will log all experiment details, including hyperparameters, model architecture, metrics, and performance. 
 During training, we will log key details such as:
<ul> 
 <li>Hyperparameters (learning rate, batch size, model architecture)</li>
 <li>Performance metrics (accuracy, precision, recall, F1-score)</li>
 <li>Training and validation loss</li>
 <li>Model checkpoints (for retraining)</li>  
</ul>

To efficiently manage and scale our training jobs, we will use a Ray cluster
<ul><li>Parallelized training of models like DenseNet-121, InceptionV3, and MobileNet for fracture detection and TB/pneumonia detection. </li>
  <li>Hyperparameter tuning using Ray Tune, enabling us to experiment with different hyperparameter configurations to find the optimal settings for each model.</li>
</ul>

`Difficulty Points` Attempting from Unit-4 & Unit-5:
<ul> Using Ray Train </ul>




