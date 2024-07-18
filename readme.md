# Prometheus Setup

## ARCHITECTURE

![Screen-Shot-2016-02-28-at-21 53 32](https://github.com/Avinash828/Avinash-interview/assets/78551424/86f584ce-e049-48b8-9a94-445ef0e79a7f)

### Pre-Request
- Running Kubernetes CLuster.
- Helm3

### Deploying MySql application

- Create Mysql deployment with service as a NodePort. Below is the `mysql-deployment.yml` file.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:8.0
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
          name: mysql
---

apiVersion: v1
kind: Service
metadata:
   name: mysql
spec:
   type: NodePort
   selector:
      app: mysql
   ports:
      - port: 3306
        targetPort: 3306
        nodePort: 30007
```
- Apply the deployment.
```bash
kubectl apply -f mysql-deployment.yaml
```
- Create Screte using below `mysql-secret.yaml` file.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: kubernetes.io/basic-auth
stringData:
  password: test1234
```
Apply the above secret.
```bash
kubectl apply -f mysql-secret.yaml
```

### Deploying Prometheus Operator
```bash
helm install prometheus prometheus-community/kube-prometheus-stack
```
### Creating Prometheus Server service as a NodePort
We have to create the Prometheus server service as a NodePort service for exposing it to Grafana API's.
Use below command for editing Prometheus service
```bash
kubectl edit svc/prometheus-kube-prometheus-prometheus
```
```bash
kubectl get svc -A -o wide
```
![image](https://github.com/user-attachments/assets/fc33ccbf-56a4-49df-ac39-6832e307a0b4)


### Deploying Mysql-exporter

- Use below file for scaping metrics from deployed Mysql Server.
```yaml
mysql:
  db: ""
  host: "192.168.49.2"
  pass: "test1234"
  port: 30007
  protocol: ""
  user: "root"

serviceMonitor:
  # enabled should be set to true to enable prometheus-operator discovery of this service
  enabled: true
  additionalLabels:
    release: grafana
    release: prometheus
    app: mysql
MySql Database
```
``**Note:-** Replace host,pass` and `port` according to your requirement.``

- Install Mysql-exporter using below helm command

```bash
helm upgrade --install mysql-exporter prometheus-community/prometheus-mysql-exporter -f applyfile.yml
```
``**Note:-** Replace `applyfile.yml according to your requirement.``


### Create Grafana Dashboard

![Screenshot from 2024-07-09 18-58-55](https://github.com/Avinash828/Avinash-interview/assets/78551424/44fa15e8-15c4-44f6-986b-9776d49f1adb)

