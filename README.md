# Cardano K8s Setup

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

## What it does

This setup runs Cardano Node and cardano-db-sync on Kubernetes using the preprod testnet.

### Folder Structure

The project is organized by network overlay (testnet/mainnet), each have their own kustomization files and configs.

### How socket sharing works

Cardano Node uses a Unix socket by default, but cardano-db-sync needs to connect over the network.  
To solve this, I added a `socat` sidecar to the node pod, which exposes the socket over TCP. Then i configure db-sync to connect to the node via K8s Service.

### Autoscaling strategy

Only the Cardano Node is autoscaled:

- It's deployed as a Deployment instead of StatefulSet, to support HPA
- I use it as stateless resource
- Each pod starts quickly using a Mithril snapshot
- Storage uses `emptyDir` or a shared volume for caching snapshots

Cardano DB Sync and Postgres are not autoscaled:

- DB Sync writes to the database and must run as a single instance
- Postgres just have only one writer, can scale read-only externally but i dont think what you want that :>

I think Cardano Node can scale by using solutions like Hydra - a fast and lightweight layer to connect with applications that support payments and similar use cases. That's all i researched when building this with no blockchain experience at the start .

### Mithril snapshot

When a Cardano Node pod starts, an init container downloads a snapshot from Mithril and saves it to a shared volume.  
New pods reuse that snapshot to skip syncing from genesis and start up faster.

### System in Action

![](<image/CleanShot 2025-06-07 at 13.21.42@2x.png>)
*Pods starting up successfully with Mithril snapshot initialization*

![](<image/CleanShot 2025-06-07 at 14.14.48@2x.png>)
*Fast node sync in progress using Mithril snapshot*

![](<image/CleanShot 2025-06-07 at 15.28.16@2x.png>)
*DB Sync successfully connected to node and syncing data*

![](<image/CleanShot 2025-06-07 at 22.40.59@2x.png>)
*All system components running healthy in Kubernetes*

![](<image/TablePlus 2025-06-07 22.41.22.png>)
*Blockchain data successfully synced to PostgreSQL*