# Nexus_GCPBucket_blob
This repo consists of steps that are required to configure GCP bucket as blob storage by using the `nexus-blobstore-google-cloud` plugin.


## ARCHITECTURE DIAGRAM

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
- For the creation of Bucket and Datastore follow the links [**BUCKET**](https://cloud.google.com/storage/docs/creating-buckets#console), [**DATASTORE**]().
- Create a service account with following IAM role:-
  - **roles/editor** (for broad permissions including cloud-platform access)
  - **roles/compute.viewer** (for read-only Compute Engine access)
  - **roles/storage.objectAdmin** (for Cloud Storage write access)
  - **roles/datastore.user** (for Datastore access)
  - **roles/logging.logWriter** (for Cloud Logging write access) [ this role is optinal]

![image](https://github.com/user-attachments/assets/24db1145-05cb-4355-8871-f45b1eff3bc3)

### Creating service-account.JSON file
- Goto the above GCP Console, search for `"service account"`, then click on the service account that we have just created above with the required roles.
- Click on `Managed Keys`
![image](https://github.com/user-attachments/assets/6f8e8737-6c1a-49af-953e-fa6e50887b29)

- Create a new key with `key type`= `json`.
- Download the key and rename as a `service-account.json` file.
### Deploy Nexus 3 with nexus-blobstore-google-cloud plugin

To deploy Sonatype/Nexus3 with nexus-blobstore-google-cloud plugin , we must above two file(`service-account.json` and `plugin.kar`) on our local directory.
- `service-account.json` file need to to be copy at `/run/secrets/` location of nexus container. Similiraly we also need to copy `plugin.kar` file at `/opt/sonatype/nexus/deploy` folder.

Pull the `Sonatype/Nexus3:3.68` image.
Run container with below command
```docker
docker run -d -p {PORT}:8081 --name {NAME} -v /PATH/TO/json_FILE:/run/secrets/service-account.json -v /APTH/OF/PLUGIN.KAR_FILE:/opt/sonatype/nexus/deploy sonatype/nexus3:3.68.0
```

**NOTE:-** Replace `{PORT}` and `{NAME}` with your sutiable value. Also replace `/PATH/TO/` with sutiable path.

### Adding GCP Bucket as blob storage
- Goto nexus setting, click on `blob storage`.
- Click on ` Add new storage`, select `Google Colud Storage` as option.

  ![image](https://github.com/user-attachments/assets/f06acc5f-37af-49e0-8ce7-10b9fd7ec45f)

- Now fill out the details, like bucket name, region and path of ``service-account.json`` file.

  ![image](https://github.com/user-attachments/assets/08eb5ccd-2e11-41fa-ab4c-5a5118b57660)

- Once Nexus3 authenticates, the following blob storage will be created.

  ![image](https://github.com/user-attachments/assets/76313470-f772-4776-a7e8-86f4a86ea4bc)


-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
#### Author: `Avinash Yadav`

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

