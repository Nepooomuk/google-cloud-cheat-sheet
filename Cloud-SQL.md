# Google Cloud SQL

## Create Cloud SQL Instance
```
gcloud beta sql instances create nepooomuk-cloud-sql --region=europe-west1 --tier=db-f1-micro\
 --authorized-networks=`curl -s ifconfig.co` --backup-start-time=01:00 --enable-bin-log \
 --activation-policy=ALWAYS --storage-type=HDD --storage-size=10GB
 ```

## List Cloud SQL Instances
 ```
 gcloud sql instances list
  ```

## Create Service Account
 ```
gcloud iam service-accounts create nepooomuk-application --display-name="nepooomuk application"
 ```

## List Service Accounts
 ```
gcloud iam service-accounts list
 ```

## Extend Service Account permissions
 ```
gcloud projects add-iam-policy-binding nepooomuk-kubernetes-cloud-sql \
 --member serviceAccount:nepooomuk-application@nepooomuk-kubernetes-cloud-sql.iam.gserviceaccount.com \
 --role roles/editor
  ```

 ## Create keyfile
  ```
 gcloud iam service-accounts keys create \
--iam-account nepooomuk-application@nepooomuk-kubernetes-cloud-sql.iam.gserviceaccount.com nepooomuk-credentials.json
 ```

## Register keyfile in Kubernetes
 ```
kubectl create secret generic cloudsql-oauth-credentials --from-file=credentials.json=jhipster-credentials.json
 ```

## Add cloudsql proxy to the deployment definition
```
- image: b.gcr.io/cloudsql-docker/gce-proxy:1.05
  name: cloudsql-proxy
  command: ["/cloud_sql_proxy", "--dir=/cloudsql",
            "-instances=jhipster-kubernetes-cloud-sql:europe-west1:jhipster-sqlcloud-db=tcp:3306",
            "-credential_file=/secrets/cloudsql/credentials.json"]
  volumeMounts:
    - name: cloudsql-oauth-credentials
      mountPath: /secrets/cloudsql
      readOnly: true
    - name: ssl-certs
      mountPath: /etc/ssl/certs
```

## Provide SSL certificates
```
volumes:
  - name: cloudsql-oauth-credentials
    secret:
      secretName: cloudsql-oauth-credentials
  - name: ssl-certs
    hostPath:
      path: /etc/ssl/certs
```