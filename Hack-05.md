## Hack-05 Kubenetes cluster operation and applications deployment
---

### Display parameter

```bash
echo $rg
echo $aksvnet
echo $akssubnet
echo $vnetaddress
echo $vnetsubnet
echo $akscluster
echo $acr
echo $acrserver
echo $middleserver
```
[How to initialize?](https://github.com/SmithMMTK/DevOpsHack/blob/master/Hack-01.md#prepare-environment-parameter)

---

### Kubenetest basic commands

Previous step : [Create AKS on VNET](https://github.com/SmithMMTK/DevOpsHack/blob/master/Hack-03.md#create-aks-on-vnet)

- Get AKS context
```bash
kubectl config get-contexts
#kubectl config use-context CONTEXT_NAME
```

- Get AKS Pods, Services and Deployments
```bash
kubectl get pods
kubectl get services
kubectl get deployments
```

- Push Docker image to ACR
```bash
docker images

docker tag multi-wfe $acrserver/multi-wfe:v1
docker push $acrserver/multi-wfe:v1

az acr repository list --name $acr --output table
az acr repository show-tags --name $acr --repository multi-wfe --output table

```

### Working with deployment file

Review deployment file [wfe.yaml](/sources/wfe/kubefiles/wfe-ake.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: multi-wfe
spec:
  replicas: 3
  selector:
    matchLabels:
      app: multi-wfe
  template:
    metadata:
      labels:
        app: multi-wfe
    spec:
      containers:
      - name: multi-wfe
        image: azsmi15acr.azurecr.io/multi-wfe:v1
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: middleserver
          value: "smi15middle.southeastasia.cloudapp.azure.com"
---
apiVersion: v1
kind: Service
metadata:
  name: multi-wfe
spec:
  type: LoadBalancer
  ports:
  - port: 8082
  selector:
    app: multi-wfe
```

Replace ACR container image to your acr server

```bash
echo $acrserver
```

*** Ignore middleserver URL, we will cover it later ***

Deploy application

```bash
kubectl apply -f wfeAKS.yaml
# kubectl delete all --selector app=multi-wfe
kubectl get pods
kubectl get services
```



---