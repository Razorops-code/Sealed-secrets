### install using Helm

### Add repository

```
helm repo add bitnami-labs https://bitnami-labs.github.io/sealed-secrets/

```

### Install chart in elk namespace 

### metion namespace otherwide it will be install in kube-system namespace

```
helm install elk-sealed-secret bitnami-labs/sealed-secrets --version 2.7.1 -n elk

```

### Check the installation

``` 
kubectl -n elk  get pods
```

### Check the logs of the sealed secret controller

```
kubectl -n elk logs sealed-secrets 
```

### From the logs we can see that it writes the encryption key its going to use as a kubernetes secret
Example log:

```
2022/11/27 21:38:20 New key written to kube-system/sealed-secrets-keymwzn9
```

### Encryption keys

```
kubectl -n elk get secrets
kubectl -n elk get secret sealed-secrets-keygxlvg -o yaml

```

### Download KubeSeal

The same way we downloaded the sealed secrets controller from the [GitHub releases](https://github.com/bitnami-labs/sealed-secrets/releases) page,
we'll want to download kubeseal from the assets section 
```

curl -L -o /tmp/kubeseal.tar.gz \
https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.19.2/kubeseal-0.19.2-linux-amd64.tar.gz
tar -xzf /tmp/kubeseal.tar.gz -C /tmp/
chmod +x /tmp/kubeseal
mv /tmp/kubeseal /usr/local/bin/
```
We can now run `kubeseal --help`

check the 'elasticsearch-master-credentials' in sercets
```
kubectl get secrets -n elk
```
copy that file in yaml format using 
```
kubectl get secret elasticsearch-master-credentials -o yaml -n elk > elasticsearch-master-credentials.yaml
```

### Sealing a basic Kubernetes Secret
 
```
   kubeseal \
      --controller-name=elk-sealed-secrets \
      --controller-namespace=elk \
      --format yaml > sealed-secret.yaml
```
### if you would rather not need to access to the cluster to genrate sealed secret you can run

```
   kubeseal \
      --controller-name=new-sealed-secrets \
      --controller-namespace=elk \
      --fetch-cert > cert.pem
```
to retrieve the public cert used for encryption and store it locally . You can run ' kubeseal --cert mycert.pem' insted to use the local cert e.g 
```
kubeseal < /home/ubuntu/ELK/elasticsearch-master-credentials.yaml --cert cert.pem -o yaml > /home/ubuntu/ELK/sealed-secret.yaml 
```

### apply the sealed secret 

``` 
kubectl apply -f sealed-secret.yaml
```
we can check the sealed secrets 
```
kubectl get sealedsecerets -n elk      ```

Both the SealedSecret and generated Secret must have the name and namespace.
