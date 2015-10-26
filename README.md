# Sourcegraph GKE Configuration

This configuration will setup a Sourcegraph instance on a Google Container Engine cluster.
## Setup
### Volumes
1. Create a Persistent Disk (PD) for data directory. Choose disk size based on repository storage requirements. Replace compute zone with correct zone.
```bash
gcloud compute disks create --size=300GB --zone=<Compute Zone> sourcegraph-data
```
2. Create PD for configuration directory.
```bash
gcloud compute disks create --size=30GB --zone=<Compute Zone> sourcegraph-config
```

### Service
Creates a load balancer with a public IP.
```bash
kubectl create -f service.yml
```

### Replication Controller
Replication controller will pull required images and mount volumes created above.
```bash
kubectl create -f rc.yml
```

**DONE!**

## Try it out
Finally, get the IP address assigned to the load balancer. This will be labeled as *LoadBalancer Ingress*.
```bash
kubectl describe services/sourcegraph
```

