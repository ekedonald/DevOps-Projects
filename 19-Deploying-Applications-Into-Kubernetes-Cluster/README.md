# Deploying Applications Into Kubernetes Cluster
In this project, applications will be deployed on a K8s cluster. Kuberbetes has a lot of moving parts, it operates with several layers of abstraction between your application and host machines where it runs.

Kuberentes has a lot of resources abd capablities that it is not realistic to learn it all at once. Hence the following concepts will be inttoduced:
* Pods
* ReplicaSets
* Deployments
* StatefulSets
* Services (ClusterIP, NodeIP, Loadbalancer)

## Getting Started
1. Spin up EC2 Instance on AWS (i.e. used to create the cluster).

2. On your AWS console, search and click on `IAM`.

![iam](./images/0%20IAM.png)

3. Click on the `Roles` tab and `Create role` button.

![roles create role](./images/0%20roles%20create%20role.png)

4. Select `AWS Service` and `EC2` as the use case.

![role1](./images/0%20role1.png)

5. Select `Adminstrator Access` permission policy.

![add permissions](./images/0%20add%20permissions.png)

6. Name the role `EKSCTL_Role` and click on `Create role`.

![name eksctl role](./images/0%20name%20eksctl%20role%20and%20create.png)

7. On your console home, search and click on EC2.

![ec2](./images/0%20ec2.png)

8. Click on the EC2 instance you spun up and click on `Actions`, `Security` and `Modify IAM role` respectively to attach the `EKSCTL_Role`.

![attach role to ec2 instance](./images/0%20attach%20role%20to%20ec2%20instance.png)

9. Select `EKSCTL_Role` and click on `Update IAM role`.

10. SSH into the EC2 Instance.

11. Install **awscli** using the code below:

```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

![install awscli](./images/0%20install%20awscli.png)

12. Verify the installation using:

```sh
aws --version
```

![install awscli](./images/0%20install%20awscli2.png)

13. Install **kubectl** using the code below:

```sh
wget https://storage.googleapis.com/kubernetes-release/release/v1.29.0/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

![install kubectl](./images/0%20install%20kubectl.png)

14. Install **eksctl** using the code below:

```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

![install eksctl](./images/0%20install%20eksctl.png)

15. Create a K8 cluster using eksctl.

```sh
eksctl create cluster \
  --name=my-cluster \
  --region=us-east-1 \
  --node-type=t3.medium \
  --nodes=2 \
  --zones=us-east-1b,us-east-1d
```

![create cluster](./images/1%20create%20cluster.png)

16. Verify if the cluster has been setup correctly.

```sh
eksctl get cluster
```

![eksctl get cluster](./images/1%20eksctl%20get%20cluster.png)

```sh
kubectl get nodes
```

![kubectl get nodes](./images/1%20kubectl%20get%20nodes.png)


## Deploying Containerized Applications Into a Kubernetes Cluster
The following steps are taken to deploy containerized applications on a K8s cluster:

### Step 1: Deloying an Nginx Container into a Pod

1. Create a Pod yaml manifest.

```sh
sudo cat <<EOF | sudo tee ./nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP
EOF
```

2. Apply the manifest with the help of kubectl.

```sh
kubectl apply -f nginx-pod.yaml
```

![apply pod manifest](./images/2%20apply%20pod%20manifest.png)

3. Get an output of the pods running in the cluster.

```sh
kubectl get pods
```

![get pods](./images/2%20get%20pods.png)

4. To see other fields introduced by kubernetes after you have deployed the resource, simply run below command, and examine the output. You will see other fields that kubernetes updates from time to time to represent the state of the resource within the cluster. `-o `simply means the **output** format.

```sh
kubectl get pod nginx-pod -o yaml 
```

![get pod nginx-pod -o yaml](./images/2%20get%20pod%20nginx-pod.png)

```sh
kubectl describe pod nginx-pod
```

![describe pod](./images/2%20describe%20pod.png)

### Step 2: Accessing the Application from the Browser
The ultimate goal of any solution is to access it either through a web portal or some application (e.g. mobile app). We have a Pod with Nginx container, so we need to access it from the browser. But all you have is a running Pod that has its own IP address which cannot be accessed through the browser. To achieve this, we need another Kubernetes object called [Service](https://kubernetes.io/docs/concepts/services-networking/service/) to accept our request and pass it on to the Pod.

A service is an object that accepts requests on behalf of the Pods and forwards it to the Pod's IP address. If you run the command below, you will be able to see the Pod's IP address. But there is no way to reach it directly from the outside world.

```sh
kubectl get pod nginx-pod  -o wide 
```

![get pod -o wide](./images/3%20get%20pod%20-o%20wide.png)

1. We need an image that already has `curl` software installed. You can check it out [here](https://hub.docker.com/r/dareyregistry/curl).

2. Run kubectl to connect inside the container.

```sh
kubectl run curl --image=dareyregistry/curl -i --tty
```

![run curl](./images/3%20run%20curl.png)

3. Run curl and point to the IP address of the Nginx Pod (Use the IP address of your own Pod).

```sh
curl -v pod_id_address
```

![curl -v pod ip](./images/3%20curl%20-v%20pod_ip.png)

4. If the use case for your solution is required for internal use ONLY, without public Internet requirement. Then, this should be OK. But in most cases, it is NOT!

Assuming that your requirement is to access the Nginx Pod internally, using the Pod's IP address directly as above is not a reliable choice because Pods are ephemeral. They are not designed to run forever. When they die and another Pod is brought back up, the IP address will change and any application that is using the previous IP address directly will break.

To solve this problem, kubernetes uses **Service**. An object that abstracts the underlining IP addresses of Pods. A service can serve as a load balancer, and a reverse proxy which basically takes the request using a human readable DNS name, resolves to a Pod IP that is running and forwards the request to it. This way, you do not need to use an IP address. Rather, you can simply refer to the service name directly.

Let us create a service to access the **Nginx Pod**.

5. Create a Service yaml manifest file:

```sh
sudo cat <<EOF | sudo tee ./nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-pod 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF
```

![nginx-service yaml](./images/3%20nginx-service%20yaml.png)

6. Create an `nginx-service` resource by applying your manifest.

```sh
kubectl apply -f nginx-service.yaml
```

![apply nginx-service yaml](./images/3%20apply%20service%20manifest.png)

7. Check the created service.

```sh
kubectl get service
```

![get service](./images/3%20get%20service.png)

**Obervation**: The **TYPE** column in the output shows that there are [different service types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types).

* Cluster IP
* NodePort
* LoadBalancer
* Headless Service

Since we did not specify any type, it is obvious that the default type is **ClusterIP**.

8. Now that we have a service created, how can we access the app? Since there is no public IP address, we can leverage `kubectl's` **port-forward** functionality.

```sh
kubectl  port-forward svc/nginx-service 8089:80
```

**8089** is an arbitrary port number on your laptop or client PC, and we want to tunnel traffic through it to the port number of the `nginx-service` **80**.

Unfortunately, this will not work quite yet. Because there is no way the service will be able to select the actual Pod it is meant to route traffic to. If there are hundreds of Pods running, there must be a way to ensure that the service only forwards requests to the specific Pod it is intended for.

To make this work, you must reconfigure the Pod manifest and introduce **labels** to match the **selectors** key in the field section of the service manifest.

9. Update the Pod manifest with the below and apply the manifest:

```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-pod  
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP
```

![update pod manifest](./images/3%20update%20pod%20manifest.png)

Notice that under the metadata section, we have now introduced `labels` with a key field called `app` and its value `nginx-pod`. This matches exactly the `selector` key in the **service** manifest.

The key/value pairs can be anything you specify. These are not Kubernetes specific keywords. As long as it matches the selector, the service object will be able to route traffic to the Pod.

10. Apply the manifest with:

```sh
kubectl apply -f nginx-pod.yaml
```

![apply pod manifest update](./images/3%20apply%20pod%20manifest%20update.png)

11. Run kubectl port-forward command again.

```sh
kubectl  port-forward svc/nginx-service 8089:80
```

![port forward svc/nginx-service](./images/3%20port-forward%20service.png)

12. Then go to your vm and enter `curl localhost:8089`, you should now be able to see the nginx page.

![curl localhost:8089](./images/3%20curl%20localhost_8089.png)

Let us try to understand a bit more about how the service object is able to route traffic to the Pod.

13. If you run the below command:

```sh
kubectl get service nginx-service -o wide
```

![get service nginx-service -o wide](./images/3%20get%20service%20-o%20wide.png)

As you already know, the service's type is ClusterIP, and in the above output, it has the IP address of `10.100.240.135` - This IP works just like an internal loadbalancer. It accepts requests and forwards it to an IP address of any Pod that has the respective selector label. In this case, it is `app=nginx-pod`. 

If there is more than one Pod with that label, service will distribute the traffic to all theese pods in a [Round Robin](https://en.wikipedia.org/wiki/Round-robin_scheduling) fashion.

14. Now, let us have a look at what the Pod looks like:

```sh
kubectl get pod nginx-pod --show-labels
```

![get pod show labels](./images/3%20get%20pod%20show%20labels.png)

**Notice that the IP address of the Pod, is NOT the IP address of the server it is running on. Kubernetes, through the implementation of network plugins assigns virtual IP adrresses to each Pod.**

```sh
kubectl get pod nginx-pod -o wide
```

Therefore, Service with IP `10.100.240.135` takes request and forwards to Pod with IP `192.168.20.129`.

### Step 3: Expose a Service on a Server's Public IP Address & Static Port

Sometimes, it may be needed to directly access the application using the public IP of the server (when we speak of a K8s cluster we can replace 'server' with 'node') the Pod is running on. This is when the [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) service type comes in handy.

A **NodePort** service type exposes the service on a static port on the node's IP address. NodePorts are in the `30000-32767` range by default, which means a NodePort is unlikely to match a service’s intended port (for example, 80 may be exposed as 30080).

1. Update the nginx-service yaml to use a NodePort Service.

```sh
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-pod
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30080
```

![nginx-service](./images/4%20nginx-service%20nodeport.png)

What has changed is:

* Specified the type of service (Nodeport)
* Specified the NodePort number to use.

2. Allow the inbound traffic in your **EKS Cluster NodeGroup** Security Group to the NodePort range 30000-32767.

![update security group of nodegroup](./images/4%20update%20security%20group%20of%20nodegroup.png)

3. Get the public IP address of the node the Pod is running on, append the nodeport.

```sh
kubectl get nodes -o wide
```

![get nodes to show ip to connect to nodeport](./images/4%20get%20nodes%20to%20show%20ip%20to%20connect%20to%20nodeport%20.png)

4. Access the app through the browser.

```sh
node_external_ip:30080
```

![node-ip_30080](./images/4%20node_ip_30080.png)

**How Kubernetes ensures desired number of Pods is always running?**
When we define a Pod manifest and appy it - we create a Pod that is running until it's terminated for some reason (e.g., error, Node reboot or some other reason), but what if we want to declare that we always need at least 3 replicas of the same Pod running at all times? 

Then we must use an [ReplicaSet (RS)](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) object - it's purpose is to  maintain a stable set of Pod replicas running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

**Note**: In some older books or documents you might find the old version of a similar object - [ReplicationController (RC)](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/), it had similar purpose, but did not support [set-base label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#set-based-requirement) and it is now recommended to use ReplicaSets instead, since it is the next-generation RC.

5. Let us delete our nginx-pod Pod:

```sh
kubectl delete -f nginx-pod.yaml
```

![delete nginx-pod](./images/4%20delete%20pod.png)

6. Let us create a rs.yaml manifest for a ReplicaSet object:

```sh
cat <<EOF | tee rs.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    app: nginx-pod
  template:
    metadata:
      name: nginx-pod
      labels:
         app: nginx-pod
    spec:
      containers:
      - image: nginx:latest
        name: nginx-pod
        ports:
        - containerPort: 80
          protocol: TCP
EOF
```

7. Apply the rs.yaml manifest.

```sh
kubectl apply -f rs.yaml
```

![apply -f rs.yaml](./images/4%20apply%20replicaset%20manifest.png)

The manifest file of ReplicaSet consist of the following fields:

**apiVersion**: This field specifies the version of kubernetes Api to which the object belongs. ReplicaSet belongs to **apps/v1** apiVersion.

**kind**: This field specify the type of object for which the manifest belongs to. Here, it is **ReplicaSet**.

**metadata**: This field includes the metadata for the object. It mainly includes two fields: name and labels of the ReplicaSet.

**spec**: This field specifies the **label selector** to be used to select the Pods, number of replicas of the Pod to be run and the container or list of containers which the Pod will run. In the above example, we are running 3 replicas of nginx container.

8. Let us check what Pods have been created:

```sh
kubectl get pods
```

![kubectl get pods](./images/4%20get%20pods.png)

Here we see three ngix-pods with some random suffixes (e.g. `-2z4ts`) - it means that these Pods were created and named automatically by some other object (higher level of abstraction) such as ReplicaSet.

9. Delete one of the Pods:

```sh
kubectl delete po nginx-rs-2z4ts
```

![delete po nginx-pod-g44](./images/4%20delete%20pod%20nginx-rs.png)

```sh
kubectl get pods
```

![get pods 2](./images/4%20get%20pods%202.png)

You can see, that we still have all 3 Pods, but one has been recreated.

10. Explore the ReplicaSet created:

```sh
kubectl get rs -o wide
```

![get rs -o wide](./images/4%20get%20rs%20-o%20wide.png)

11. To display detailed information about any Kubernetes object, you can use 2 different commands:

```sh
kubectl describe rs nginx-rs
```

![describe rs](./images/4%20describe%20rs.png)

```sh
kubectl get rs nginx-rs -o yaml
```

![get rs nginx-rs -o yaml](./images/4%20get%20rs.png)


12. We can easily scale our ReplicaSet up by specifying the desired number of replicas in an imperative command, like this:

```sh
kubectl scale rs nginx-rs --replicas=5
```

![scale rs 5](./images/4%20scarle%20rs%205.png)

**Advanced label matching**: As Kubernetes matures as a technology, so does its features and improvements to k8s objects. ReplicationControllers do not meet certain complex business requirements when it comes to using selectors. Imagine if you need to select Pods with multiple lables that represents things like:

* **Application tier**: such as Frontend, or Backend
* **Environment**: such as Dev, SIT, QA, Preprod, or Prod

So far, we used a simple selector that just matches a key-value pair and check only 'equality':

```sh
  selector:
    app: nginx-pod
```

But in some cases, we want ReplicaSet to manage our existing containers that match certain criteria, we can use the same simple label matching or we can use some more complex conditions, such as:

```sh
 - in
 - not in
 - not equal
 - etc...
```

13. Create an rs-yaml manifest with the advanced labels.

```sh
cat <<EOF | tee rs.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      env: prod
    matchExpressions:
    - { key: tier, operator: In, values: [frontend] }
  template:
    metadata:
      name: nginx
      labels: 
        env: prod
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
          protocol: TCP
EOF
```

In the above spec file, under the selector, **matchLabels** and **matchExpression** are used to specify the key-value pair. 

The **matchLabel** works exactly the same way as the equality-based selector and the matchExpression is used to specify the set based selectors. This feature is the main differentiator between **ReplicaSet** and previously mentioned obsolete **ReplicationController**.

14. Get the replication set:

```sh
kubectl get rs nginx-rs -o wide
```

![get rs nginx-rs -o wide](./images/4%20get%20rs%20-o%20wide.png)

### Step 4: Using AWS Load Balancer to access your service in Kubernetes

We have previously accessed the Nginx service through **ClusterIP** and **NodePort**, but there is another service type - [Loadbalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer). This type of service does not only create a Service object in K8s, but also provisions a real external Load Balancer (e.g. [Elastic Load Balancer - ELB](https://aws.amazon.com/elasticloadbalancing/) in AWS)

1. To get the experience of this service type, update your service manifest and use the **LoadBalancer** type. Also, ensure that the selector references the Pods in the replicaset.

```sh
cat <<EOF | tee nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
    - protocol: TCP
      port: 80 # This is the port the Loadbalancer is listening at
      targetPort: 80 # This is the port the container is listening at
EOF
```

2. Apply the configuration.

```sh
kubectl apply -f nginx-service.yaml
```

![apply service loadbalancer](./images/5%20apply%20service%20loadbalancer.png)


3. Get the newly created service:

```sh
kubectl get service nginx-service
```

![get service](./images/5%20get%20service.png)

An ELB resource will be created in your AWS console.

A Kubernetes component in the control plane called **[Cloud-controller-manager](https://kubernetes.io/docs/concepts/architecture/cloud-controller)** is responsible for triggeriong this action. It connects to your specific cloud provider's (AWS) APIs and create resources such as Load balancers. It will ensure that the resource is appropriately tagged:

![elb on console](./images/5%20loadbalancer%20on%20EC2%20console.png)

4. Get the output of the entire `yaml` for the service. You will see some additional  information about this service in which you did not define them in the yaml manifest. Kubernetes did this for you.

```sh
kubectl get service nginx-service -o yaml
```

![get service -o yaml](./images/5%20get%20service%20-o%20yaml.png)

* A clusterIP key is updated in the manifest and assigned an IP address. Even though you have specified a `LoadBalancer` service type, internally it still requires a `ClusterIP` to route the external traffic through.

* In the ports section, `NodePort` is still used. This is because Kubernetes still needs to use a dedicated port on the worker node to route the traffic through. Ensure that port range `30000-32767` is opened in your inbound Security Group configuration.

* More information about the provisioned balancer is also published in the `.status.loadBalancer` field.

5. Copy and paste the load balancer's address to the browser, and you will access the Nginx service.

### Step 5: Do not Use Replication Controllers - Use Deployment Controllers Instead

A [Deployment](./images/https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is another layer above ReplicaSets and Pods, newer and more advanced level concept than ReplicaSets. It manages the deployment of ReplicaSets and allows for easy updating of a ReplicaSet as well as the ability to roll back to a previous version of deployment. It is declarative and can be used for rolling updates of micro-services, ensuring there is no downtime.

Officially, it is highly recommended to use **Deployments** to manage replica sets rather than using replica sets directly.

Let us see Deployment in action.

1. Delete the ReplicaSet.

```sh
kubectl delete rs nginx-rs
```

![delete rs](./images/6%20delete%20rs%20nginx-rs.png)

2. Create a deployment.yaml manifest.

```sh
cat <<EOF | tee deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
EOF
```

![deployemnt yaml](./images/6%20deployment_yaml.png)

3. Apply the configuration.

```sh
kubectl apply -f deployment.yaml
```

![apply deployment](./images/6%20apply%20deployment.png)

4. Get the Deployment.

```sh
kubectl get deployment
```

![get deployment](./images/6%20get%20deployment.png)

5. Get the ReplicaSet.

```sh
kubectl get rs
```

![get rs](./images/6%20get%20rs.png)

6. Get the Pods.

```sh
kubectl get pods
```

![get pods](./images/6%20get%20pods.png)


7. Scale the replicas in the Deployment to 15 Pods.

```sh
kubectl scale deployment nginx-deployment --replicas=15
```

![scale deployment 15](./images/6%20scale%20deployment%2015.png)

8. Exec into one of the Pod's container to run Linux commands.

```sh
kubectl exec -it nginx-deployment-fc79b9898-2bdsq -- bash
```

![kubectl exec -it nginx-deployment](./images/6%20kubectl%20exec%20-it%20nginx-deployment.png)

9. List the files and folders in the Nginx directory.

```sh
ls -ltr /etc/nginx/
```

![ls -ltr /etc/nginx](./images/6%20ls%20-ltr%20:etc:nginx.png)

10. Check the content of the default Nginx configuration file.

```sh
cat /etc/nginx/conf.d/default.conf
```

![cat /etc/nginx/conf.d/default.conf](./images/6%20cat%20:etc:nginx:conf.d:default.conf.png)

Now, as we have got acquainted with most common Kubernetes workloads to deploy applications.

Deployments are stateless by design. Hence, any data stored inside the Pod's container does not persist when the Pod dies.

If you were to update the content of the `index.html` file inside the container, and the Pod dies, that content will be lost since a new Pod will replace the dead one.

Let us try that:

11. Scale the Pods down to 1 replica.

```sh
kubectl scale deployment nginx-deployment --replicas=1
```

![scale deployment 1](./images/6%20scale%20deployment%201.png)

12. Exec into the running container.

```sh
kubectl exec -it nginx-deployment-fc79b9898-8mmx5 -- bash
```

13. Install vim so that you can edit the file.

```sh
apt-get update
apt-get install vim
```

![install vim](./images/6%20kubectl%20exec%20-it%20install%20vim.png)

14. Update the content of the file and add the code below `/usr/share/nginx/html/index.html`

```sh
<!DOCTYPE html>
<html>
<head>
<title>Welcome to EKEDONALD.IO!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to EKEDONALD.IO!</h1>
<p>I love experiencing Kubernetes</p>

<p>Learning by doing is absolutely the best strategy at 
<a href="https://darey.io/">www.darey.io</a>.<br/>
for skills acquisition
<a href="https://darey.io/">www.ekedonald.io</a>.</p>

<p><em>Thank you for learning from EKEDONALD.IO</em></p>
</body>
</html>
```

![update index](./images/6%20update%20:usr:share:nginx:html:index.png)


15. Go to your browser and paste the elb url.

![alb url browswer](./images/6%20alb_url%20browser.png)

16. Delete the only running Pod so that a new one is automatically recreated.

```sh
kubectl delete pod nginx-deployment-fc79b9898-8mmx5
```

![delete pod](./images/6%20get%20-%20delete%20pod%20nginx-deployment-fc675.png)

17. Refresh the web page and you will see that the content you saved in the container is no longer there. That is because Pods do not store data when they are being recreated, that is why they are called `ephemeral` or `stateless`.

![alb browser 2](./images/6%20alb%20browser%202.png)

Storage is a critical part of running containers, and Kubernetes offers some powerful primitives for managing it. 

**Dynamic volume provisioning**, a feature unique to Kubernetes, which allows storage volumes to be created on-demand. Without dynamic provisioning, DevOps engineers must manually make calls to the cloud or storage provider to create new storage volumes, and then create **PersistentVolume** objects to represent them in Kubernetes. 

The dynamic provisioning feature eliminates the need for DevOps to pre-provision storage. Instead, it automatically provisions storage when it is requested by users.

To make the data persist in case of a Pod's failure, you will need to configure the Pod to use Volumes:

18. Clean up the Deployment.

```sh
kubectl delete deployment nginx-deployment
```

![delete deployment](./images/6%20delete%20deployment.png)

