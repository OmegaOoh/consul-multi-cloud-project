## Reference
1. Demo project accompanying a [Consul crash course video](https://www.youtube.com/watch?v=s3I1kKKfjtQ) on YouTube
2. GCP Demo project (boutique) [GCP Demo](https://github.com/GoogleCloudPlatform/microservices-demo)

## How to run

### Prerequisite 
Following application/command line tool must be installed: 

1. AWS CLI
2. HELM 
3. Google Cloud CLI 
4. kubectl

### AWS-based service

1. Navigate to `terraform` directory: this is a home of AWS terraform files
2. Setup variable by creating `terraform.tfvars`. Following is example I used for mine.
```tfvars
aws_access_key_id     = "your-aws-access-key-id"
aws_secret_access_key = "your-aws-access-secret-key"
aws_region            = "ap-southeast-7"
```
3. Create infrastructure with terraform
```sh
cd terraform

# initialise project & download providers
terraform init

# exeucute with preview
terraform apply -var-file terraform.tfvars
```
3. configure AWS connection
```sh
# Configure connection using key id and secret key (same as in tfvars file)
aws configure

# configure kubectl context (region can be changed according to the region set in previous step)
aws eks update-kubeconfig --region ap-southeast-7 --name myapp-eks-cluster
```
3. After infrastructure is created, run application using `config.yaml` in `kubernetes` directory
```sh
cd ..
cd kubernetes

kubectl apply -f config.yaml
```
4. Wait for every pods to run, then you'll be able to access it via your web browser
```sh
# Check for pods status  (It should be all running except paymentservice)
kubectl get pods

# Check for service endpoint
kubectl get services
```
Example of get services, external IP is an IP of the loadbalancer that can be used to access the site
```
❯ kubectl get services
NAME                          TYPE           CLUSTER-IP       EXTERNAL-IP                                                                    PORT(S)                                                                            AGE
adservice                     ClusterIP      172.20.164.147   <none>                                                                         9555/TCP                                                                           24m
cartservice                   ClusterIP      172.20.129.36    <none>                                                                         7070/TCP                                                                           24m
checkoutservice               ClusterIP      172.20.13.40     <none>                                                                         5050/TCP                                                                           24m
currencyservice               ClusterIP      172.20.181.105   <none>                                                                         7000/TCP                                                                           24m
emailservice                  ClusterIP      172.20.197.75    <none>                                                                         5000/TCP                                                                           24m
frontend-external             LoadBalancer   172.20.109.23    aa7b6f9e18e2e460baca6afcbb1489e7-2111907473.ap-southeast-7.elb.amazonaws.com   80:30977/TCP                                                                       24m
kubernetes                    ClusterIP      172.20.0.1       <none>                                                                         443/TCP                                                                            40m
paymentservice                ClusterIP      172.20.239.48    <none>                                                                         50051/TCP                                                                          24m
productcatalogservice         ClusterIP      172.20.204.111   <none>                                                                         3550/TCP                                                                           24m
recommendationservice         ClusterIP      172.20.31.102    <none>                                                                         8080/TCP                                                                           24m
redis-cart                    ClusterIP      172.20.198.93    <none>                                                                         6379/TCP                                                                           24m
shippingservice               ClusterIP      172.20.139.210   <none>                                                                         50051/TCP                                                                          24m
```

5. Create gp3 volume
```sh
# ensure that you still on kubernetes directory
kubectl apply -f gp3-default.yaml
```
6. Register consul in your helm repositories
```sh
helm repo add hashicorp https://helm.releases.hashicorp.com
```

7. Install consul with following command
```sh
helm install eks hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=eks
```

you may change its config as following
```sh
helm install <DEPLOYMENT-PREFIX> hashicorp/consul --version 1.0.0 --values consul-values.yaml --set global.datacenter=<DATACENTER-NAME>
```
After this, all services should be running. You can check using `kubectl get pods`

8. After all pods are running, check and connect to consul UI, you can check via `kubectl get services`

Examples:
```
❯ kubectl get services
NAME                          TYPE           CLUSTER-IP       EXTERNAL-IP                                                                    PORT(S)                                                                            AGE
adservice                     ClusterIP      172.20.164.147   <none>                                                                         9555/TCP                                                                           24m
cartservice                   ClusterIP      172.20.129.36    <none>                                                                         7070/TCP                                                                           24m
checkoutservice               ClusterIP      172.20.13.40     <none>                                                                         5050/TCP                                                                           24m
currencyservice               ClusterIP      172.20.181.105   <none>                                                                         7000/TCP                                                                           24m
eks-consul-connect-injector   ClusterIP      172.20.133.32    <none>                                                                         443/TCP                                                                            15m
eks-consul-dns                ClusterIP      172.20.30.170    <none>                                                                         53/TCP,53/UDP                                                                      15m
eks-consul-mesh-gateway       LoadBalancer   172.20.51.160    a79f3a94306b4413c8c27baffa5b44c2-482509621.ap-southeast-7.elb.amazonaws.com    443:30789/TCP                                                                      15m
eks-consul-server             ClusterIP      None             <none>                                                                         8501/TCP,8502/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP,8600/UDP   15m
eks-consul-ui                 LoadBalancer   172.20.8.158     a9c45f14bfb7145b59705850d65c05c7-1534254856.ap-southeast-7.elb.amazonaws.com   443:31699/TCP                                                                      15m
emailservice                  ClusterIP      172.20.197.75    <none>                                                                         5000/TCP                                                                           24m
frontend-external             LoadBalancer   172.20.109.23    aa7b6f9e18e2e460baca6afcbb1489e7-2111907473.ap-southeast-7.elb.amazonaws.com   80:30977/TCP                                                                       24m
kubernetes                    ClusterIP      172.20.0.1       <none>                                                                         443/TCP                                                                            40m
paymentservice                ClusterIP      172.20.239.48    <none>                                                                         50051/TCP                                                                          24m
productcatalogservice         ClusterIP      172.20.204.111   <none>                                                                         3550/TCP                                                                           24m
recommendationservice         ClusterIP      172.20.31.102    <none>                                                                         8080/TCP                                                                           24m
redis-cart                    ClusterIP      172.20.198.93    <none>                                                                         6379/TCP                                                                           24m
```

The IP of your UI will be at `<DEPLOYMENT-PREFIX>-consul-ui`'s External IP (https protocol is required).
