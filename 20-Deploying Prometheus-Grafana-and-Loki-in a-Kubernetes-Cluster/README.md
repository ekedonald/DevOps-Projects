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

![create ns loki](./images/3%20create%20ns%20loki.png)

Add the Bitnami Helm chart repository.

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
```

![repo add bitnami](./images/3%20repo%20add%20bitnami.png)

Update the local Helm chart repository cache.

```sh
helm repo update
```

![repo update](./images/3%20repo%20update.png)

Install the Grafana Loki chart in the loki namespace.

```sh
helm install loki bitnami/grafana-loki -n loki
```

![install loki](./images/3%20install%20loki.png)

Verify that Grafana Loki is running.

```sh
kubectl get pods -n loki
```

![get pods loki](./images/3%20get%20pods%20loki.png)

Get the Grafana Loki service details.

```sh
kubectl get svc -n loki
```

![get svc loki](./images/3%20get%20svc%20loki.png)

Change the Service Type of the `loki-grafana-loki-gateway` service from **ClusterIP** to **NodePort**.

```sh
kubectl edit svc/loki-grafana-loki-gateway -n loki
```

![edit svc/loki-grafana-loki-gateway](./images/3%20edit%20svc:loki.png)

Run the following command to get the value of the NodePort:

```sh
kubectl get svc/loki-grafana-loki-gateway -n loki
```

![get svc/loki](./images/3%20get%20svc:loki-gateway.png)

## Step 4: Installing Grafana

Create a namespace called `grafana`.

```sh
kubectl create ns grafana
```

![create ns grafana](./images/4%20create%20ns%20grafana.png)

Add the Grafana Helm chart repository.

```sh
helm repo add grafana https://grafana.github.io/helm-charts
```

![repo add grafana](./images/4%20repo%20add%20grafana.png)

Update the local Helm chart repository cache.

```sh
helm repo update
```

![repo update](./images/4%20repo%20update.png)

Install the Grafana chart in the Grafana namespace.

```sh
helm install grafana grafana/grafana -n grafana
```

![install grafana](./images/4%20install%20grafana.png)
_**Note**: Copy and run the highlighted command shown above to get the default of the `password`. The username: `admin`, these will be used to login to *Grafana*._

Verify that Grafana is running.

```sh
kubectl get pods -n grafana
```

![get pods grafana](./images/4%20get%20pods%20-n%20grafana.png)

Get the Grafana service details.

```sh
kubectl get svc -n grafana
```

![get svc grafana](./images/4%20get%20svc%20grafana.png)

Change the Service Type of the `grafana` service from **ClusterIP** to **NodePort**.

```sh
kubectl edit svc/grafana -n grafana
```

![edit svc/grafana](./images/4%20edit%20svc:grafan.png)

## Step 5: Access the Grafana server from a Web Browser

The `grafana` server now has a NodePort which permits access from the internet. Run the following command to get the value of the NodePort:

```sh
kubectl get svc -n grafana
```

![get svc grafana](./images/5%20get%20svc%20grafana.png)

Get the IP address of the minikube cluster.

```sh
minikube ip
```

![minikube ip](./images/2%20minikube%20ip.png)

Go to your web browser and paste the following url:

```sh
http://<minikube_ip>:<grafana-nodeport>
```

![grafana url](./images/5%20grafana%20url.png)
_Paste the password generated from **Step 4** and the username stated._

## Step 6: Configuring Prometheus and Loki as Data Sources to Grafana

On the Grafana webpage, click on the `Open menu` tab then click on the `Connections` dropdown and `Add New Connections`.

![open menu, connections, add new connections](./images/6%20menu,%20connections,%20add%20new.png)

Search for the `Prometheus` Data Source and click on it.

![prometheus](./images/6%20search%20prometheus.png)

Click on `Add new data source`.

Paste the `http://<minikube_ip>:<prometheus-server-nodeport>` into the `Prometheus server URL` box.

![paste url](./images/6%20paste%20url.png)

Scroll to the bottom and click on `Save & test`.

![save and test](./images/5%20save%20n%20test.png)

_**Note**: You will see a prompt that shows you have **Successfully queried the Prometheus API**_.

Repeat the same procedure to add **Grafana Loki** as a Data source.

## Step 7: Visualizing The K8 Cluster on Grafana using Dashboards

With the Prometheus and Grafana Loki configured as Data Sources, we can visualize the cluster using metrics, traces and logs by [Importing Grafana Dashboards](https://grafana.com/grafana/dashboards).

On your Grafana Server homepage, click on `Open menu` and `Dashboards`.

![open menu, dashboards](./images/7%20open%20menu,%20dashboards.png)

Click on `Create Dashboard`.

![create dashboards](./images/7%20create%20dashboards.png)

Click on `Import dashboard`.

![import dashboard](./images/7%20import%20dashboard.png)

You can [Import Grafana Dashboards](https://grafana.com/grafana/dashboards) to visualize the Kubernetes Cluster.

Search for `K8s` and click on it.

![search k8s](./images/7%20k8s.png)

Click on `Copy ID to clipboard`.

![copy id to clipboard](./images/7%20copy%20id%20to%20clipboard.png)

Go back to the Grafana server, paste the `GUID` and click on `Load`.

![guid and load](./images/7%20paste%20guid%20n%20load.png)

Click on the Prometheus tab, select `Prometheus` as the Data source and click on `Import`.

![select prometheus, import](./images/7%20prometheus%20and%20import.png)

You now have a visual representation of your K8 cluster.

![k8s cluster](./images/7%20k8s%20cluster.png)

Repeat the steps above to import more dashboards.

![dashboards 1](./images/7%20dashboard1.png)
![dashboards 2](./images/7%20dashboard2.png)
![dashboards 3](./images/7%20dashboard3.png)

## Step 8: Setting Up Alerts on Prometheus

Create a `prometheus.yaml` file in your directory and add an **alert rule**.

```sh
cat <<EOF | tee prometheus.yaml
serverFiles:
  alerting_rules.yml:
    groups:
    - name: example-group
      rules:
      - alert: HighErrorRate
        expr: sum(rate(http_server_errors_total{status="500"}[5m])) > 10
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: High error rate on {{ $labels.instance }}
          description: "The rate of HTTP 500 errors is above the threshold"
EOF
```
_In the configuration above, an alert named **HighErrorRate** is defined. It triggers when the **HTTP 500 errors exceed 10 errors per second over a 5 minute window**. The alert remains active for at least 1 minute_.

Upgrade the Prometheus Helm release with the `prometheus.yaml` configuration.

```sh
helm upgrade prometheus prometheus-community/prometheus -f prometheus.yaml -n prometheus
```

![helm upgrade](./images/8%20helm%20upgrade.png)

_**Note**: The NodePort of the Prometheus-Server changes after upgrading the Helm release_.

Go your web browser and access the Prometheus-Server with the new NodePort and click on Alert.

```sh
http://<minikube_ip>:<new-prometheus-server-nodeport>
```

![prometheus alert rule browser](./images/8%20prom%20alert.png)
_You will see that Prometheus is using the alert rule_.

## Step 9: Examining and Visualizing the `etcd` Pod Log Entries with Loki

Access the Grafana server using your browser, click on `Open menu` and `Explore`.

![open menu and explore](./images/9%20open%20menu%20n%20explore.png)

Click on `Builder`, select `Pod` and `etcd-minikube` as the **label filters** respectively then click on `Run query`.

![builder and pod](./images/9%20builder%20pod%20etcd.png)

_The query displays the **Log Volume** and **Logs** of the `etcd-minikube` pod within an hour_.
