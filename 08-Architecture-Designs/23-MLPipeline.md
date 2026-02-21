# ML Pipeline Architecture on AWS

## Problem Statement
Build an end-to-end ML pipeline from data ingestion to model serving at scale.

## Architecture Diagram

```mermaid
graph LR
  subgraph Training["Training Pipeline"]
    S3Data["S3\n(Training Data)"] --> SagemakerProc["SageMaker\nProcessing\n(Feature Eng)"]
    SagemakerProc --> SageTrain["SageMaker\nTraining Job\n(Multi-GPU)"]
    SageTrain --> ModelReg["SageMaker\nModel Registry"]
  end
  subgraph Serving["Inference Pipeline"]
    ModelReg --> SM_EP["SageMaker\nEndpoint\n(A/B Testing)"]
    ModelReg --> SM_Batch["SageMaker\nBatch Transform"]
    SM_EP --> Lambda["Lambda\n(inference wrapper)"]
    Lambda --> Cache["ElastiCache\n(prediction cache)"]
  end
  subgraph Monitoring["Model Monitoring"]
    SM_EP --> Monitor["SageMaker\nModel Monitor"]
    Monitor --> CW["CloudWatch\n(data drift alerts)"]
    CW -->|Drift detected| Retrain["Retrain Trigger\n(EventBridge)"]
    Retrain --> SageTrain
  end
```

## Pipeline Stages
1. **Feature Engineering**: SageMaker Processing Jobs (PySpark/scikit-learn containers)
2. **Training**: SageMaker Training with Spot instances (70% savings); distributed training on p3.16xlarge cluster
3. **Registry**: SageMaker Model Registry with approval workflow
4. **Deployment**: Blue/Green endpoint deployment; A/B testing with traffic splitting
5. **Monitoring**: data drift + model drift detection; auto-retrain on drift

## Cost Optimization
- **Spot training**: SageMaker Managed Spot up to 90% savings; checkpointing to S3
- **Inference**: Graviton2 (ml.c7g) for CPU inference; 30% cheaper
- **Batch Transform**: for non-real-time inference; much cheaper than endpoint
- **Multi-model endpoints**: pack multiple models into one endpoint

## Interview Talking Points
- "SageMaker Spot Training is one of the biggest cost levers in ML — 90% savings with checkpointing"
- "Model Registry with approval workflow = MLOps governance (who approved this model?)"
- "Data drift monitoring is often overlooked; model accuracy degrades without it"
- "A/B testing on SageMaker endpoints: traffic split between old and new model; compare metrics"
