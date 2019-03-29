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
# USECASE 1: Adding response header on replies to client

   The below yaml file will add the following headers with the http response sent from microservices to client when the requested url contains **/citrix-app/**
   
   * Add the client source port to the header
   * Add the server destination IP address
   * a random http header
   
   Similar way we can include a variety of information supported by the Citrix ADC on the response header
   
   * [add_response_headers.yaml](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/usecase1/add_response_headers.yaml)
  ```
  apiVersion: citrix.com/v1
kind: rewritepolicy
metadata:
  name: addresponseheaders
spec:
  rewrite-policies:
    - servicenames: 
        - frontend
      rewrite-policy:
        operation: insert_before_all
        target: http.res.full_header
        modify-expression: '"\r\nx-port: "+client.tcp.srcport+"\r\nx-ip:"+client.ip.dst+"\r\nx-new-dummy-header: Sending_a_gift"'
        multiple-occurence-modify: 'text("\r\n\r\n")'
        comment: 'Response header rewrite'
        direction: RESPONSE
        rewrite-criteria: 'http.req.url.contains("/citrix-app/")'
  ```
  ## Kubectl command to add the response headers
  ```
  kubectl create -f add_response_headers.yaml
  ```
  
  ## HTTP Header Response Output:
  ```
  Saravanas-MacBook-Pro:usecase1 saravanakumarr$ curl -vvv http://app.cic-citrix.org/citrix-app/
*   Trying 10.102.33.176...
* TCP_NODELAY set
* Connected to app.cic-citrix.org (10.102.33.176) port 80 (#0)
> GET /citrix-app/ HTTP/1.1
> Host: app.cic-citrix.org
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.8.1
< Date: Fri, 29 Mar 2019 11:14:04 GMT
< Content-Type: text/html
< Transfer-Encoding: chunked
< Connection: keep-alive
< X-Powered-By: PHP/5.5.9-1ubuntu4.14
< x-port: 22481 ==================> NEW RESPONSE HEADER
< x-ip:10.102.33.176 ==================> NEW RESPONSE HEADER
< x-new-dummy-header: Sending_a_gift ==================> NEW RESPONSE HEADER
< 
<html>
<head>
<title> Front End App - v1 </title>


TRIMMED
.......
  ```
  
# USECASE 3:

## a) Modifying the approot to another url


   
