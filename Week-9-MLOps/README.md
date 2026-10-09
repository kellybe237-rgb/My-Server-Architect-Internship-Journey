# Week 9 — MLOps: Model Tracking, API Deployment, Kubernetes, and Load Testing

## Overview

This week I built and deployed an Iris classification inference service and tested its behavior under sustained concurrent requests. The work covered experiment tracking, API development, Docker containerization, Kubernetes deployment, and performance measurement.

## Day 1 — Build an ML Inference API

- Built a machine-learning inference API with **FastAPI**.
- Used a trained Iris classification model saved as `model.pkl`.
- Implemented a `GET /health` health-check endpoint and a `POST /predict` prediction endpoint.
- Used JSON request data containing sepal and petal measurements.
- Verified the API through HTTP requests and FastAPI's interactive Swagger UI at `/docs`.

## Day 2 — MLflow Experiment Tracking

- Set up MLflow to track Iris classification experiments.
- Used a Random Forest classifier and the experiment name `Iris_Classification_Experiment`.
- Logged model-training parameters and evaluation metrics to compare experiment runs.
- Worked through model-artifact serialization configuration, including the trusted-type setting needed by the installed MLflow/skops stack.

**Key learning:** Experiment tracking makes model configurations and evaluation results easier to compare and reproduce. Model artifact logging must also be verified, not just parameter and metric logging.

## Day 3 — Containerize the Inference Service

- Created a Dockerfile based on `python:3.14-slim`.
- Installed application dependencies from `requirements.txt`.
- Packaged `app.py` and `model.pkl` into the image.
- Built the Docker image `primeone-ml-service:v1`.
- Ran the API in a container and checked its health and prediction endpoints.

**Key learning:** Containerization bundles the application and its dependencies into a repeatable runtime environment.

## Day 4 — Deploy to Kubernetes with Minikube

- Loaded `primeone-ml-service:v1` into Minikube.
- Created a Kubernetes manifest, `ml-deployment.yaml`, for the inference service.
- Deployed **2 replicas** using the local image with `imagePullPolicy: Never`.
- Configured readiness and liveness probes against `/health`.
- Exposed the application through the `ml-inference-svc` NodePort service.
- Verified that both pods were running and tested the deployed health and prediction endpoints.

**Key learning:** Kubernetes deployments support replicated workloads, service discovery, and health checks that help manage application availability.

## Day 5 — Stress Testing and Latency Profiling

I used ApacheBench (`ab`) to send concurrent JSON POST requests to the Kubernetes-hosted `/predict` endpoint for approximately 60 seconds.

### Benchmark configuration

- Duration: 60 seconds
- Concurrency: 20
- Socket timeout: 30 seconds
- Endpoint: `/predict`
- Tool: ApacheBench

### Results

| Metric | Result |
|---|---:|
| Test duration | 60.038 seconds |
| Completed requests | 20,069 |
| Failed requests | 0 |
| Throughput | 334.27 requests/second |
| Mean time per request | 59.831 ms |
| P50 latency | 60 ms |
| P95 latency | 71 ms |
| P99 latency | 94 ms |
| Longest observed request | 1,066 ms |

### Analysis

The service completed 20,069 requests with no failed requests during this run and sustained approximately 334 requests per second. The P50, P95, and P99 latency values were 60 ms, 71 ms, and 94 ms respectively. The longest request took 1,066 ms, showing that occasional latency spikes can occur even when typical response times are low.

These measurements describe this specific local Minikube environment and workload; they should not be treated as a production capacity guarantee. Further runs at different concurrency levels and resource monitoring would help characterize performance more completely.

### Benchmark command

```bash
ab -t 60 -c 20 -s 30 \
  -p payload.json \
  -T 'application/json' \
  http://127.0.0.1:55531/predict | tee day5-ab-results.txt
```

## Main Skills Practiced

- FastAPI endpoints and JSON request handling
- MLflow experiment tracking
- Docker image creation and container execution
- Kubernetes Deployments, replicas, Services, and health probes
- ApacheBench load testing and latency percentile interpretation
- Recording benchmark results and documenting engineering observations

## Local Project Artifacts

The local project includes the API and model files, Docker configuration, Kubernetes manifest, and Day 5 benchmark artifacts. The Day 5 results and report created locally are:

- `day5-ab-results.txt` — full ApacheBench output
- `day5-benchmark-report.md` — benchmark summary and analysis

## Week 9 Takeaway

I practiced taking an ML model from a local inference API through containerization and Kubernetes deployment, then measuring the deployed endpoint under concurrent load. I also learned to distinguish average throughput and latency from tail latency, and to keep experiment and performance evidence alongside the project documentation.
