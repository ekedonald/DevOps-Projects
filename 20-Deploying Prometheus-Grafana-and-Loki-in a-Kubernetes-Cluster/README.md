# Deploying Prometheus, Grafana and Loki in a Kubernetes Cluster

This projects shows the steps taken to install and configure monitoring tools. I will be deploying Prometheus for collecting metrics, Loki for logs and Grafana for visualizing the data.

### Prerequisites

* Create a K8 Cluster using [EKS](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.htmlhttps://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html) or [minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download).
* Install [Helm](https://helm.sh/docs/intro/install/).

_**Note**: I am using `minikube` to execute this project_.


## Step 1: Installing Prometheus

Create a namespace called `Prometheus`.

```sh
kubectl create ns prometheus
```

![create bs prometheus](./images/1%20create%20ns%20prometheus.png)

Add the Prometheus Helm chart repository.

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

![repo add prometheus-community](./images/1%20repo%20add%20prometheus.png)

Update the local Helm chart repository cache.

```sh
helm repo update
```

![repo update](./images/1%20helm%20repo%20update.png)

Install the Prometheus chart in the prometheus namespace.

```sh
helm install prometheus prometheus-community/prometheus -n prometheus
```

![install prometheus](./images/1%20helm%20install%20prometheus.png)

Verify that Prometheus is running.

```sh
kubectl get pods -n prometheus
```

![get pods](./images/1%20kubectl%20get%20pods%20-n%20prometheus.png)

Get the Prometheus service details.

```sh
kubectl get svc -n prometheus
```

![get svc prometheus](./images/1%20get%20svc%20prometheus.png)

Change the Service Type of the `Prometheus-Server` service from **ClusterIP** to **NodePort**.

```sh
kubectl edit svc/prometheus-server -n prometheus
```

![edit svc/prometheus-server](./images/1%20edit%20prom-server.png)

## Step 2: Access the Prometheus server from a Web Browser

The `Prometheus-Server` service now has a NodePort which permits access from the internet. Run the following command to get the value of NodePort:

```sh
kubectl get svc/prometheus-server -n prometheus
```

![get svc/prometheus-server](./images/2%20get%20svc:prom-server.png)

Get the IP address of the minikube cluster.

```sh
minikube ip
```

![minikube ip](./images/2%20minikube%20ip.png)

Go to your web browser and paste the following url:

```sh
http://<minikube_ip>:<prometheus-server-nodeport>
```

![prom url](./images/2%20http%20minikube-ip:prom-nodeport.png)

Verify if Prometheus can access the components of the Kubernetes Cluster by clicking on `Status` and `Targets`.

![status](./images/2%20promserver%201.png)

![targets](./images/2%20promserver%202.png)
_You can see that the components of the Kubernetes Cluster can be accessed by Prometheus_.

## Step 3: Installing Grafana Loki

Create a namespace called `loki`.

```sh
kubectl create ns loki
```

Add the Bitnami Helm chart repository.

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Update the local Helm chart repository cache.

```sh
helm repo update
```

Install the Grafana Loki chart in the loki namespace.

```sh
helm install loki bitnami/grafana-loki -n loki
```

Verify that Grafana Loki is running.

```sh
kubectl get pods -n loki
```

Get the Grafana Loki service details.

```sh
kubectl get svc -n loki
```

Change the Service Type of the `loki-grafana-loki-gateway` service from **ClusterIP** to **NodePort**.

```sh
kubectl edit svc/loki-grafana-loki-gateway -n loki
```

Run the following command to get the value of the NodePort:

```sh
kubectl get svc/loki-grafana-loki-gateway -n loki
```

## Step 4: Installing Grafana

Create a namespace called `grafana`.

```sh
kubectl create ns grafana
```

Add the Grafana Helm chart repository.

```sh
helm repo add grafana https://grafana.github.io/helm-charts
```

Update the local Helm chart repository cache.

```sh
helm repo update
```

Install the Grafana chart in the Grafana namespace.

```sh
helm install grafana grafana/grafana -n grafana
```

_**Note**: Copy and run the highlighted command shown above to get the default of the `password`. The username: `admin`, these will be used to login to *Grafana*._

Verify that Grafana is running.

```sh
kubectl get pods -n grafana
```

Get the Grafana service details.

```sh
kubectl get svc -n grafana
```

Change the Service Type of the `grafana` service from **ClusterIP** to **NodePort**.

```sh
kubectl edit svc/grafana -n grafana
```

## Step 5: Access the Grafana server from a Web Browser

The `grafana` server now has a NodePort which permits access from the internet. Run the following command to get the value of the NodePort:

```sh
kubectl get svc -n grafana
```

Get the IP address of the minikube cluster.

```sh
minikube ip
```

Go to your web browser and paste the following url:

```sh
http://<minikube_ip>:<grafana-nodeport>
```

_Paste the password generated from **Step 4** and the username stated._

## Step 6: Configuring Prometheus and Loki as Data Sources to Grafana

On the Grafana webpage, click on the `Open menu` tab then click on the `Connections` dropdown and `Add New Connections`.

Search for the `Prometheus` Data Source and click on it.

Click on `Add new data source`.

Paste the `http://<minikube_ip>:<prometheus-server-nodeport>` into the `Prometheus server URL` box.

Scroll to the bottom and click on `Save & test`.

_**Note**: You will see a prompt that shows you have **Successfully queried the Prometheus API**_.

Repeat the same procedure to add **Grafana Loki** as a Data source.

## Step 7: Visualizing The K8 Cluster on Grafana using Dashboards

With the Prometheus and Grafana Loki configured as Data Sources, we can visualize the cluster using metrics, traces and logs by [Importing Grafana Dashboards](https://grafana.com/grafana/dashboards).

On your Grafana Server homepage, click on `Open menu` and `Dashboards`.

Click on `Create Dashboard`.

Click on `Import dashboard`.

You can [Import Grafana Dashboards](https://grafana.com/grafana/dashboards) to visualize the Kubernetes Cluster.

Search for `K8s` and click on it.

Click on `Copy ID to clipboard`.

Go back to the Grafana server, paste the `GUID` and click on `Load`.

Click on the Prometheus tab, select `Prometheus` as the Data source and click on `Import`.

_You now have a visual representation of your K8 cluster_.

Repeat the steps above to import more dashboards.