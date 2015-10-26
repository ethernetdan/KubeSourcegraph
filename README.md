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

### Sanity check
You should now have a running, yet unconfigured, instance of Sourcegraph.

To verify this get the IP of the load balancer by describing the service. This IP will be labeled as *LoadBalancer Ingress*.
```bash
kubectl describe services/sourcegraph
```

In your browser visit the IP address found above. You should see the Sourcegraph UI with a **Wrong application URL** error message.

### Configuration
To remove the error message above and to customize your Sourcegraph instance some configuration will be required.

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
There are several [configuration options](https://src.sourcegraph.com/sourcegraph/.docs/config/) but the required one to remove the error message is changing AppURL.
Use the IP discovered above as Endpoint.

**Note:** If you are setting up a domain to host Sourcegraph, point the A record to the IP above and set Endpoint below to the domain.
```
AppURL = http://<Endpoint>
```

Once you are done editing the configuration, restart the Sourcegraph instance by deleting the Pod it's running on
```bash
kubectl delete pods/<Pod Name>
```

### Errata
Currently DNS does not seem to be working inside the container. This means that the instance is unable to call home to Sourcegraph.com.





