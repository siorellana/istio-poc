# POC Istio con SSL

## Links de apoyo:
https://www.padok.fr/en/blog/https-istio-kubernetes

https://cloud.google.com/solutions/integrating-https-load-balancing-with-istio-and-cloud-run-for-anthos-deployed-on-gke

## Consideraciones:
You must install the gcloud command line tool.

# Create a cluster with the following configuration adding the HttpLoadBalancing and Istio Add-on.

### Create cluster public
```
gcloud beta container clusters create ssl-poc-2 \
    --addons HttpLoadBalancing,Istio \
    --enable-ip-alias \
    --enable-stackdriver-kubernetes \
    --machine-type n1-standard-1 \
    --zone us-central1-f
```

### Create cluster private
```
CLUSTER=istio-ssl-poc
ZONE=us-central1-f

gcloud beta container clusters create $CLUSTER \
    --addons HttpLoadBalancing,Istio \
    --enable-ip-alias \
    --enable-private-nodes \
    --enable-private-endpoint \
    --enable-stackdriver-kubernetes \
    --machine-type n1-standard-1 \
    --enable-master-authorized-networks \
    --master-ipv4-cidr "172.16.0.0/28" \
    --master-authorized-networks 10.0.0.0/8 \
    --zone $ZONE
```

## Cambiar tipo de ingress de Istio a Nodeport para utilizar el de K8S
```
kubectl -n istio-system patch svc istio-ingressgateway --type=json -p="$(cat istio-ingressgateway-patch.json)" --dry-run=true -o yaml | kubectl apply -f -
```

## Ejecutar los manifiestos en el siguiente orden
```
kubectl apply -f 03-namespace.yaml
kubectl apply -f 00-my-gateway.yaml
kubectl apply -f 01-istio-virtualsvc.yaml
kubectl apply -f 08-certificate.yaml
kubectl apply -f 02-my-ingress.yaml
kubectl apply -f 04-deployment-app.yaml 
kubectl apply -f 05-service.yaml
```

APP2
```
kubectl apply -f 09-namespace-app2.yaml
kubectl apply -f 11-deployment-app2.yaml
kubectl apply -f 10-service-app2.yaml
```

APP3 - Con Manifiestos en el mismo espacio de nombres
```
kubectl apply -f 12-namespace-app3.yaml
kubectl apply -f 13-service-app3.yaml
kubectl apply -f 14-deployment-app3.yaml
kubectl apply -f 15-my-gateway-app3.yaml
kubectl apply -f 16-my-ingress-app3.yaml
kubectl apply -f 17-istio-virtualsvc-app3.yaml
kubectl apply -f 18-certificate-app3.yaml
```

## Habilitar istio en namesoace
`kubectl label namespace app3 istio-injection=enabled`

## Comandos útiles
```
kubectl describe pod
kubectl describe managedcertificate
kubectl get ingress my-ingress -n istio-system
```

TODO: Gateway y Virtualservice crear en Namespace específico.

TODO: Crear DNS de manera automática.