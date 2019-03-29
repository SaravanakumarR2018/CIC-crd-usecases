# CIC-crd-usecases
Rewrite policy CRD citrix-ingress-controller usecases

## Apply cert keys on kubernetes
* [citrix-ingress-gke.crt](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/cert_keys/citrix-ingress-gke.crt)
* [citrix-ingress-gke.key](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/cert_keys/citrix-ingress-gke.key)
```
kubectl  create secret tls apache --cert citrix-ingress-gke.crt --key citrix-ingress-gke.key
```
## Apply ingress and services on kubernetes

* [citrix_ingress_example.yaml](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/ingress_and_services/citrix_ingress_example.yaml)
* [web-frontend.yaml](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/ingress_and_services/web-frontend.yaml)

```
kubectl create -f citrix_ingress_example.yaml
kubectl create -f web-frontend.yaml
```
