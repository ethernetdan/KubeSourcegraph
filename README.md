# Sourcegraph GKE Configuration

This configuration will setup a Sourcegraph instance on a Google Container Engine cluster.
## Setup
### Volumes
Create a Persistent Disk (PD) for data directory. Choose disk size based on repository storage requirements. Replace compute zone with correct zone.
```bash
gcloud compute disks create --size=300GB --zone=<Compute Zone> sourcegraph-data
```
Create PD for configuration directory.
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

### Sanity check
You should now have a running, yet unconfigured, instance of Sourcegraph.

To verify this get the IP of the load balancer by describing the service. This IP will be labeled as *LoadBalancer Ingress*.
```bash
kubectl describe services/sourcegraph
```

In your browser visit the IP address found above. You should see the Sourcegraph UI, *congratulations!*

### Configuration
First, get the name of the Pod that Sourcegraph is running on by running:
```bash
kubectl get pods
```

Choose the pod which is prefixed by the name of the Replication Controller (if nothing was changed this is sourcegraph)
For example: `sourcegraph-4qlyr`

Edit the configuration file by running the following command. Be sure to replace **<Pod Name>**.
```bash
kubectl exec <Pod Name> -i -t -- vi /etc/sourcegraph/config.ini 
```

Once you are done editing the configuration, restart the Sourcegraph instance by deleting the Pod it's running on
```bash
kubectl delete pods/<Pod Name>
```
