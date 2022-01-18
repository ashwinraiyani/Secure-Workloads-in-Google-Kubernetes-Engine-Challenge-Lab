### Secure Workloads in Google Kubernetes Engine: Challenge Lab :-

----------------------------------------------------------------------------------------------------------------------------------------------



### Task - 0 : Download the necessary files :-

```yaml
gsutil -m cp gs://cloud-training/gsp335/* .
```

### Task - 1 : Setup Cluster :-

```yaml
gcloud container clusters create security-demo-cluster620  --machine-type n1-standard-4 --num-nodes 2 --zone us-central1-c --enable-network-policy
gcloud container clusters get-credentials security-demo-cluster620 --zone us-central1-c
```

### Task - 2 : Setup WordPress :-

```yaml
gcloud sql instances create wordpress-db-707 --region=us-central1
gcloud sql databases create wordpress --instance wordpress-db-707
gcloud sql users create wordpress --instance=wordpress-db-707 --host=% --password='P@ssword!'
```

```yaml
gcloud iam service-accounts create sa-wordpress-827 --display-name sa-wordpress-827
```

```yaml
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT    --role roles/cloudsql.client  --member serviceAccount:sa-wordpress-827@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

```yaml
gcloud iam service-accounts keys create key.json    --iam-account sa-wordpress-827@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

```

```yaml
kubectl create secret generic cloudsql-instance-credentials    --from-file key.json
kubectl create secret generic cloudsql-db-credentials \
  --from-literal username=wordpress \
  --from-literal password='P@ssword!'

```

```yaml
kubectl create -f volume.yaml

```
```yaml
Go to the overview page of your Cloud SQL instance, and copy the Connection name 
```

```yaml
Click on Open Editor --> Open wordpress.yaml
```


Open wordpress.yaml with your favorite editor, and replace INSTANCE_CONNECTION_NAME (in line 61) with the Connection name of your Cloud SQL instance.

```yaml
kubectl apply -f wordpress.yaml
```

### Task - 3 : Setup Ingress with TLS :-

```yaml
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install nginx-ingress stable/nginx-ingress
```

```yaml
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.16.0/cert-manager.yaml
```

```yaml
kubectl get svc
```

Execute again until external IP comes

Check the service nginx-ingress-controller as an external IP address before continuing to the next step.

in below code change email id and service account name 

```yaml
sed -i s/student-04-59b075fae8a5@qwiklabs.net/sa-wordpress-827@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com/g issuer.yaml
kubectl apply -f issuer.yaml
```

If you face any internal error, execute the command again.

```yaml
. add_ip.sh
```

```yaml
HOST_NAME=$(echo $USER | tr -d '_').labdns.xyz
sed -i s/HOST_NAME/${HOST_NAME}/g ingress.yaml
kubectl apply -f ingress.yaml
```

### Task - 4 : Set up Network Policy :-

```yaml
kubectl apply -f network-policy.yaml
```

```yaml
nano network-policy.yaml
```

Add the following code at the last :-

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
 name: allow-nginx-access-to-internet
spec:
 podSelector:
  matchLabels:
    app: nginx-ingress
 policyTypes:
 - Ingress
 ingress:
 - {}
```

### Task - 5 : Setup Binary Authorization :-


In the Cloud Console, navigate to ### Security > Binary Authorization.
Enable the Binary Authorization API.
On the Binary Authorization page, click on CONFIGURE POLICY.
Select Disallow all images for the Default rule.
Scroll down to Images exempt from this policy, click ADD IMAGE PATTERN
add the four values and change as :-

docker.io/library/wordpress:latest
us.gcr.io/k8s-artifacts-prod/ingress-nginx/*
gcr.io/cloudsql-docker/*
quay.io/jetstack/*

save policy

goto Kubernetes Engine > Clusters.
Click on the pencil icon for Binary authorization under the Security section

Check Enable Binary Authorization in the dialog.
Click SAVE CHANGES.

### wait till cluster get update (5 minute)


### Task - 6 : Setup Pod Security Policy :-

```yaml
kubectl apply -f psp-restrictive.yaml
kubectl apply -f psp-role.yaml
kubectl apply -f psp-use.yaml
kubectl apply -f psp-restrictive.yaml
```

### Congratulations!

Follow on Twitter @techiezilla , Instagram @techiezilla.
