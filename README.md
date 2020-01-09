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

## Change the Istio Ingress to use the K8S Ingress
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

## Enable the namespace to use istio envoy
`kubectl label namespace app3 istio-injection=enabled`

## Comandos útiles
kubectl describe pod
kubectl describe managedcertificate
kubectl get ingress my-ingress -n istio-system


TODO: Gateway y Virtualservice crear en Namespace específico.
TODO: Crear DNS de manera automática.
