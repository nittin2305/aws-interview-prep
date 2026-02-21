# Edge Computing Architecture

## Problem Statement
Process IoT data at the edge (factory floor) while syncing to AWS cloud for analytics.

## Architecture Diagram

```mermaid
graph LR
  subgraph Factory["Factory Floor (Edge)"]
    Devices["IoT Devices\n(sensors, cameras)"] --> GG["AWS Greengrass\n(Core Device)"]
    GG --> LocalLambda["Local Lambda\n(real-time processing)"]
    GG --> GGML["Greengrass ML\n(inference at edge)"]
    LocalLambda --> LocalSQ["Local SQS-like\n(Greengrass message broker)"]
  end
  subgraph Cloud["AWS Cloud"]
    GG -->|MQTT/HTTPS| IoTCore["AWS IoT Core\n(device management)"]
    IoTCore --> Rules["IoT Rules\n(routing)"]
    Rules --> KDS["Kinesis Data Streams"]
    Rules --> DDB["DynamoDB\n(device shadow)"]
    KDS --> Flink["Analytics Flink"]
    Flink --> QuickSight["QuickSight\n(factory dashboard)"]
  end
```

## Edge Processing with Greengrass
- Run Lambda functions locally on edge device
- ML inference at edge: no cloud round-trip for time-sensitive decisions
- Local message broker: continue processing during internet outage
- Sync to cloud: batch or real-time, depending on bandwidth

## Device Shadow (AWS IoT Core)
- Persistent state for each device (last known good state)
- Device shadow document: `desired` + `reported` state
- App updates `desired`; device updates `reported`; IoT Core syncs
- Use case: send command to offline device; executed when it reconnects

## Interview Talking Points
- "Edge computing is about latency and bandwidth: run ML at the edge, sync insights to cloud"
- "Device Shadow is the key IoT pattern: desired vs reported state handles offline devices"
- "Greengrass extends AWS Lambda to the edge — same code, same IAM, runs locally"
