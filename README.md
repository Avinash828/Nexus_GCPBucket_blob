# Nexus_GCPBucket_blob
This repo consist of steps that are required to configure GCP bucket as a blob storage, by using "nexus-blobstore-google-cloud" plugin.


## ARCHITECTURE

![Untitled Diagram (1)](https://github.com/user-attachments/assets/b57f380e-c39f-4aa3-91b2-aafc038f1ac9)

## Pre-Request
- Sonatype/Nexus3
- GCP bucket and Datastore for metadata storage
- nexus-blobstore-google-cloud plugin (.kar file)
- GCP service-account credentials in json formate.

## Docker Deployment

### Download nexus-blobstore-google-cloud plugin
- Go to the [website](https://search.maven.org/artifact/org.sonatype.nexus.plugins/nexus-blobstore-google-cloud) and download the following `.kar` file from the `archive`.

![image](https://github.com/user-attachments/assets/9404f9e7-c8dd-473e-987f-ace59a092cef)

- Command to download the `.kar` file
```shell
wget https://repo1.maven.org/maven2/org/sonatype/nexus/plugins/nexus-blobstore-google-cloud/0.61.0/nexus-blobstore-google-cloud-0.61.0.kar
```
```bash
Connecting to repo1.maven.org (repo1.maven.org)|199.232.196.209|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11379775 (11M) [audio/midi]
Saving to: ‘nexus-blobstore-google-cloud-0.61.0.kar’

nexus-blobstore-google-cloud-0.61.0.kar            100%[===============================================================================================================>]  10.85M  1.83MB/s    in 9.7s    

2024-10-17 15:18:16 (1.12 MB/s) - ‘nexus-blobstore-google-cloud-0.61.0.kar’ saved [11379775/11379775]

```
### Create GCP service account and object's
This plugin uses the following Google Cloud Platform services:

- **Google Cloud Storage** - for storing the content blobs[Bucket]
- **Google Cloud Firestore in Datastore mode** - for storing blobstore metadata
- For the creation of Bucket and Datastore follow the links [**BUCKET**](https://cloud.google.com/storage/docs/creating-buckets#console), [**DATASTORE**]()

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

