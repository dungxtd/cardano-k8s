# Cardano K8s Setup

Simple Kubernetes setup for running Cardano nodes and DB Sync.

## Quick Start

For testnet:
```bash
kubectl apply -k kustomize-cardano-k8s/overlays/simple-testnet/
```

## What's Included

- Cardano Node
- DB Sync
- PostgreSQL database
- Ready-to-use K8s configs

## Requirements

- Kubernetes cluster
- 16GB+ storage (testnet)
- 100GB+ storage (mainnet)
- 4GB+ RAM