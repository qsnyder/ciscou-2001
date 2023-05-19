# CISCOU-2001

This repository will serve as the artifactory for the CISCOU-2001 presentation at CLUS 2023 in Las Vegas.  The purpose of this reporistory will be to serve as a holding place for the demo code such that any user can instantiate their own EC2 instance with Terraform and explore a large-scale k8s cluster using KIND.

## Prerequisites
This lab assumes the use of an AWS account.  By default, the Terraform configurations will stand up an instance in `us-east-1` of a `c5a.8xlarge` instance, with 32vCPUs, 64GB of RAM, and a mapped drive of 75GB of GP2 storage.  This will come at a cost of around $1.50/hr USD.  This will also require an AWS account with a valid billing method on file.  If sufficient compute power is found outside of AWS, the rest of the files can be used, assuming Docker, KIND, and other prerequisites are installed prior to use.

An AWS SSH keypair, as well as an IAM user access and secret key will need to be generated using the AWS control panel.  The keypair will need to be saved to the local machine in an accessible location and saved using "400" permissions (only read by the user).

## Building the AWS EC2 Instance

Within the `terraform/terraform.tfvars` file, there are several pieces of information that will need to be filled in, including the AWS access and secret keys for the IAM user, as well as the name of the SSH keypair and its location on the local machine that will be running the Terraform configuration.  Once this information is filled in appropriately, the Terraform configuration can be run from the `terraform` folder.

```bash
terraform init
terraform plan -out tf.plan
terraform apply "tf.plan"
```

This will instantiate an EC2 instance within AWS, as well as print out the FQDN and the SSH command required to access the AWS instance.  Upon SSHing into the EC2 instance, the `demo` folder will be present in the home folder.  This folder contains the rest of the files required to build the K8s cluster

## Building the K8s Cluster Using KIND (Kubernetes in Docker)

To instantiate the cluster, change into the `~/demo/cluster/` folder.  This folder contains the `kind-config.yaml` file, which defines the characteristics of the K8s cluster, as well as a setup and teardown script to aid in managing the KIND cluster.  To start the cluster, perform the following command: `bash cluster-build.sh`.  The output will indicate that the node images are being pulled and the cluster is being built.

```
Creating cluster "demo" ...
 ✓ Ensuring node image (kindest/node:v1.23.10) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-demo"
You can now use your cluster with:

kubectl cluster-info --context kind-demo

Have a nice day! 👋
```

This indicates that the cluster has been created.  We'll need to apply the Calico (Tigera) overlay networking stack to bring the cluster to full function (the `kind-config.yaml` disabled the default Kindnet overlay).  To do this, move back a directory and apply the appropriate configurations.

```
cd ../
kubectl create -f tigera-operator.yaml
kubectl create -f custom-resources.yaml
```

After a minute or so, you can check the state of the pods to ensure that they are all in the appropriate state

```
ubuntu@ip-172-31-11-82:~/demo$ kubectl get pods -A
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
calico-apiserver     calico-apiserver-9d774f9bc-cmgjj             1/1     Running   0          15s
calico-apiserver     calico-apiserver-9d774f9bc-t9hwv             1/1     Running   0          15s
calico-system        calico-kube-controllers-7d6749878f-8pxzk     1/1     Running   0          60s
calico-system        calico-node-768cb                            1/1     Running   0          60s
calico-system        calico-node-czvbp                            1/1     Running   0          60s
calico-system        calico-node-tvfrk                            1/1     Running   0          60s
calico-system        calico-node-wfqmh                            1/1     Running   0          60s
calico-system        calico-typha-5cd58978f5-g6cdg                1/1     Running   0          60s
calico-system        calico-typha-5cd58978f5-vnqgl                1/1     Running   0          55s
calico-system        csi-node-driver-7gxgx                        2/2     Running   0          32s
calico-system        csi-node-driver-bq9fx                        2/2     Running   0          41s
calico-system        csi-node-driver-pm5gq                        2/2     Running   0          31s
calico-system        csi-node-driver-r6bc4                        2/2     Running   0          31s
kube-system          coredns-64897985d-xw6vq                      1/1     Running   0          4m31s
kube-system          coredns-64897985d-zxttm                      1/1     Running   0          4m31s
kube-system          etcd-demo-control-plane                      1/1     Running   0          4m47s
kube-system          kube-apiserver-demo-control-plane            1/1     Running   0          4m46s
kube-system          kube-controller-manager-demo-control-plane   1/1     Running   0          4m47s
kube-system          kube-proxy-45xv8                             1/1     Running   0          4m26s
kube-system          kube-proxy-fxn24                             1/1     Running   0          4m31s
kube-system          kube-proxy-gw7k2                             1/1     Running   0          4m26s
kube-system          kube-proxy-w4hhv                             1/1     Running   0          4m26s
kube-system          kube-scheduler-demo-control-plane            1/1     Running   0          4m46s
local-path-storage   local-path-provisioner-58dc9cd8d9-zr88b      1/1     Running   0          4m31s
tigera-operator      tigera-operator-6dcd98c8ff-dwphl             1/1     Running   0          69s
```

Now that the overlay networking stack is in place, we can deploy the application.

## Deploying the Sock-Shop Application

Once the pods are all in running state as above, we can deploy the sock-shop application.  We do this by applying the `sock-shop.yaml` configuration to the cluster.

```
ubuntu@ip-172-31-11-82:~/demo$ kubectl apply -f sock-shop.yaml
namespace/sock-shop created
Warning: spec.template.spec.nodeSelector[beta.kubernetes.io/os]: deprecated since v1.14; use "kubernetes.io/os" instead
deployment.apps/carts created
service/carts created
deployment.apps/carts-db created
service/carts-db created
deployment.apps/catalogue created
service/catalogue created
deployment.apps/catalogue-db created
service/catalogue-db created
deployment.apps/front-end created
deployment.apps/orders created
service/orders created
deployment.apps/orders-db created
service/orders-db created
service/payment created
deployment.apps/payment created
deployment.apps/payment-db created
service/payment-db created
deployment.apps/queue-master created
service/queue-master created
deployment.apps/rabbitmq created
service/rabbitmq created
deployment.apps/session-db created
service/session-db created
deployment.apps/shipping created
service/shipping created
service/user created
deployment.apps/user created
service/user-db created
deployment.apps/user-db created
```

It may take a minute or so, but we can check the status of the pods using `kubectl` to ensure that all pods in the `sock-shop` namespace are running

```
ubuntu@ip-172-31-11-82:~/demo$ kubectl get pods -n sock-shop
NAME                            READY   STATUS    RESTARTS   AGE
carts-85f4d4b45-fsnzl           1/1     Running   0          96s
carts-db-59578c5464-l2qtr       1/1     Running   0          96s
catalogue-bbdcf5467-hl85f       1/1     Running   0          96s
catalogue-db-6bf8d6ff8f-fxjvg   1/1     Running   0          96s
front-end-7b4dfc5669-5nh5m      1/1     Running   0          96s
orders-7f57779c86-jhxf6         1/1     Running   0          96s
orders-db-848c4c6db4-wc4wb      1/1     Running   0          96s
payment-cdf9694b4-5b294         1/1     Running   0          96s
payment-db-55cf8bcffb-68mj7     1/1     Running   0          96s
queue-master-96b8bb948-5jhs5    1/1     Running   0          96s
rabbitmq-f5b858486-kvzxf        2/2     Running   0          96s
session-db-7b8844d8d5-8nfnx     1/1     Running   0          95s
shipping-797475685c-v6sw8       1/1     Running   0          95s
user-7c48486d5-64q8g            1/1     Running   0          95s
user-db-767cf48587-nbktt        1/1     Running   0          95s
```

The application is now running.  However, we have not exposed any of the front-end services outside of the cluster to be able to be accessed.  We will do this in two steps, first by adding a `NodePort` service to the name-space.  This will expose the web front-end external to the cluster.

If we look before we apply the configuration, we see that there is no external service mapping

```
ubuntu@ip-172-31-11-82:~/demo$ kubectl get services -n sock-shop
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
carts          ClusterIP   10.96.228.214   <none>        80/TCP              5m38s
carts-db       ClusterIP   10.96.112.90    <none>        27017/TCP           5m38s
catalogue      ClusterIP   10.96.189.186   <none>        80/TCP              5m38s
catalogue-db   ClusterIP   10.96.64.181    <none>        27017/TCP           5m38s
orders         ClusterIP   10.96.115.153   <none>        80/TCP              5m38s
orders-db      ClusterIP   10.96.246.212   <none>        27017/TCP           5m38s
payment        ClusterIP   10.96.121.51    <none>        80/TCP              5m38s
payment-db     ClusterIP   10.96.110.194   <none>        27017/TCP           5m38s
queue-master   ClusterIP   10.96.204.143   <none>        80/TCP              5m38s
rabbitmq       ClusterIP   10.96.163.117   <none>        5672/TCP,9090/TCP   5m38s
session-db     ClusterIP   10.96.82.59     <none>        6379/TCP            5m38s
shipping       ClusterIP   10.96.41.51     <none>        80/TCP              5m38s
user           ClusterIP   10.96.46.230    <none>        80/TCP              5m38s
user-db        ClusterIP   10.96.254.113   <none>        27017/TCP           5m38s
```

However, once we apply the `sock-shop-nodeport.yaml` configuration and check the services again, we'll see the external-to-internal mapping.

```
ubuntu@ip-172-31-11-82:~/demo$ kubectl apply -f sock-shop-nodeport.yaml
service/front-end created
ubuntu@ip-172-31-11-82:~/demo$ kubectl get services -n sock-shop
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
carts          ClusterIP   10.96.228.214   <none>        80/TCP              6m54s
carts-db       ClusterIP   10.96.112.90    <none>        27017/TCP           6m54s
catalogue      ClusterIP   10.96.189.186   <none>        80/TCP              6m54s
catalogue-db   ClusterIP   10.96.64.181    <none>        27017/TCP           6m54s
front-end      NodePort    10.96.127.34    <none>        80:30001/TCP        6s
orders         ClusterIP   10.96.115.153   <none>        80/TCP              6m54s
orders-db      ClusterIP   10.96.246.212   <none>        27017/TCP           6m54s
payment        ClusterIP   10.96.121.51    <none>        80/TCP              6m54s
payment-db     ClusterIP   10.96.110.194   <none>        27017/TCP           6m54s
queue-master   ClusterIP   10.96.204.143   <none>        80/TCP              6m54s
rabbitmq       ClusterIP   10.96.163.117   <none>        5672/TCP,9090/TCP   6m54s
session-db     ClusterIP   10.96.82.59     <none>        6379/TCP            6m54s
shipping       ClusterIP   10.96.41.51     <none>        80/TCP              6m54s
user           ClusterIP   10.96.46.230    <none>        80/TCP              6m54s
user-db        ClusterIP   10.96.254.113   <none>        27017/TCP           6m54s
```

Now, the only thing remaining is to expose `tcp/30001` outside of the set of Docker containers, since we're running KIND (note, this would not be needed if running on a full K8s installation).  We do this by using the `caddy` reverse-proxy server (which was installed as part of the EC2 build).

```
bash caddy-command.sh
```

You can now access the Sock Shop application at the FQDN used to SSH to the instance on port 8080.  You should see socks in your browser!

## Stopping the EC2 Instance

When complete with the exploration, please exit the SSH session and perform a `terraform destroy -auto-approve` to completely remove the EC2 instance (and ensure that you aren't billed for more time than the platform is in use).

```
terraform destroy -auto-approve
...
local_file.ip-address: Destroying... [id=ae2dcdb159c8dae8814f36498f9d82cfaa4d966f]
local_file.ip-address: Destruction complete after 0s
aws_instance.kind_demo: Destroying... [id=i-0f2a231d9bd50a279]
aws_instance.kind_demo: Still destroying... [id=i-0f2a231d9bd50a279, 10s elapsed]
aws_instance.kind_demo: Still destroying... [id=i-0f2a231d9bd50a279, 20s elapsed]
aws_instance.kind_demo: Still destroying... [id=i-0f2a231d9bd50a279, 30s elapsed]
aws_instance.kind_demo: Still destroying... [id=i-0f2a231d9bd50a279, 40s elapsed]
aws_instance.kind_demo: Destruction complete after 41s
aws_security_group.allow_k8s: Destroying... [id=sg-04790f6fd4c07503d]
aws_security_group.allow_k8s: Destruction complete after 1s
aws_default_vpc.default: Destroying... [id=vpc-a54183c3]
aws_default_vpc.default: Destruction complete after 0s

Destroy complete! Resources: 4 destroyed.
```
