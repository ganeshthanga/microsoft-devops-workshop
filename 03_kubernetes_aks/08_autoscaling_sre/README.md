# Autoscaling and Site Reliability Engineering

Now that you understand the basic principals of deploying an app to Kubernetes, we can discuss how to maintain uptime,  schedule workloads for redundancy, test for readiness to enable a proper rolling-update strategy, and scale horizontally when cpu thresholds are reached.

## Table of Contents
1. [Resources (Limits / Requests)](01_resources.md)
2. [NodeSelector/Affinity/Anti-Affinity](02_selector_affinity_antiaffinity.md)
3. [Liveness / Readiness Probes](03_readiness_liveness.md)
4. [Horizontal Pod Autoscaling](04_horizontal_pod_autoscaler.md)
5. [Cluster Scaling](05_cluster_autoscaler.md)