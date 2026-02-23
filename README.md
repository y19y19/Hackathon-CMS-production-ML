# Hackathon-CMS-production-ML



## set-up
- From any machine providing VOMS certificates, generate a GRID proxy (x509) file
- Copy the 509file (x509up\_uxxx) to your local machine (where kubectl is installed)
- Upload the proxy to the Kubernetes cluster as a secret:
```bash
kubectl create secret generic x509-proxy-<name> \
  --from-file=<509file> \
  --namespace <NAMESPACE>
```
- Change yaml files to specify your user details (username, namespace name, 509 proxy file, etc.)
- Deploy single triton server
```bash
kubectl apply -f triton.yaml
```
or
```bash
kubectl apply -f triton-nrp.yaml
```
- **Remember to clean up**, triton server uses GPU, when you are finished working, take down the deployment
```bash
Mac:SONIC ddiaz$ kubectl get deployments -n cms-ml-production
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
triton-server             0/1     1            0           34m
triton-server-ab          1/1     1            1           3d4h
triton-server-bhanda25    0/1     1            0           20m
triton-server-colberte    1/1     1            1           7m56s
triton-server-ddiaz-mlp   1/1     1            1           7m51s
Mac:SONIC ddiaz$ kubectl delete deployment triton-server-ddiaz-mlp -n cms-ml-production
deployment.apps "triton-server-ddiaz-mlp" deleted
```


## NRP links
- [nrp.ai](nrp.ai)
- [NRP Getting started](https://nrp.ai/documentation/userdocs/start/getting-started/)
- [NRP grafana main page](https://grafana.nrp-nautilus.io)
- [NRP grafana: namespace GPU](https://grafana.nrp-nautilus.io/d/dRG9q0Ymz/k8s-compute-resources-namespace-gpus)
- [NRP GPU resources](https://nrp.ai/documentation/userdocs/running/gpu-pods/)

## NRP details
- Specify GPU type (note some GPU use special resource label, e.g., nvidia.com/a40)
```yaml
spec:
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: nvidia.com/gpu.product
           operator: In
           values:
           - NVIDIA-GeForce-GTX-1080-Ti
```
- Specify node (may wish to develop on consistent node to avoid having to load the container image multiple times)
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - <node>
```
