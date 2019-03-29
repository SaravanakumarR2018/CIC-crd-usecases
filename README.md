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
# USECASE 2:
## Usecase 2a: Replace hostname with another

<img width="766" alt="Screenshot 2019-03-29 at 7 01 18 PM" src="https://user-images.githubusercontent.com/43468858/55236089-20f2ed80-5255-11e9-9973-a00e34ba5e74.png">

# USECASE 3:

## Usecase 3a) Modifying the approot to another url

* [http_approot_request_modify.yaml](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/usecase3/http_approot_request_modify.yaml)

 ```
 apiVersion: citrix.com/v1
kind: rewritepolicy
metadata:
  name: httpapprootrequestmodify
spec:
  rewrite-policies:
    - servicenames: 
        - frontend
      rewrite-policy:
        operation: replace
        target: http.req.url
        modify-expression: '"/citrix-approot/"'
        comment: 'HTTP app root request modify'
        direction: REQUEST
        rewrite-criteria: http.req.url.eq("/")
```
### Kubectl command to apply rewrite policy
```
kubectl create -f http_approot_request_modify.yaml
```

The url **/** is modified to **/citrix-approot/**

### Simple curl request with default url **/**

```
curl -vvv http://app.cic-citrix.org/
```
The url is modified to /citrix-approot/
![Screenshot 2019-03-29 at 5 09 19 PM](https://user-images.githubusercontent.com/43468858/55230505-01a09400-5246-11e9-884e-61a6b70232ea.png)

## Usecase 3b) Substitute url strings

   The requested url can be partially modified as per convenience using RewritePolicy CRDs
   
   * [http_url_replace_string.yaml](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/usecase3/http_url_replace_string.yaml)
   ```
   apiVersion: citrix.com/v1
kind: rewritepolicy
metadata:
  name: httpurlreplacestring
spec:
  rewrite-policies:
    - servicenames: 
        - frontend
      rewrite-policy:
        operation: replace_all
        target: http.req.url
        modify-expression: '"/"'
        multiple-occurence-modify: 'regex(re~((^(\/something\/))|(^\/something$))~)'
        comment: 'HTTP url replace string'
        direction: REQUEST
        rewrite-criteria: http.req.is_valid
   ```
   ### kubectl command to apply the url modification RewritePolicy CRD
    ```
    kubectl create -f http_url_replace_string.yaml
     ```
     
   ### A simple curl request containing *something* string should be modified
   #### Example 1
    ```
    curl http://app.cic-citrix.org/something/simple/citrix
    ```
   ##### Result:
   ![Screenshot 2019-03-29 at 5 21 57 PM](https://user-images.githubusercontent.com/43468858/55230913-3cef9280-5247-11e9-8b42-645ba384a95d.png)
   #### Example 2
    ```
    curl http://app.cic-citrix.org/something (OR)
    curl http://app.cic-citrix.org/something/
    ```
    
   ##### Result
   ![Screenshot 2019-03-29 at 5 33 17 PM](https://user-images.githubusercontent.com/43468858/55231494-c5226780-5248-11e9-9241-626727d7f09f.png)
    

# Usecase 4

## Usecase 4a) Adding custom headers to the HTTP response packet

* [add_custom_headers.yaml](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/usecase4/add_custom_headers.yaml)
```
apiVersion: citrix.com/v1
kind: rewritepolicy
metadata:
  name: addcustomheaders
spec:
  rewrite-policies:
    - servicenames: 
        - frontend
      rewrite-policy:
        operation: insert_before_all
        target: http.res.full_header
        modify-expression: '"\r\nx-request-time:"+sys.time+"\r\nx-using-citrix-ingress-controller: true"'
        multiple-occurence-modify: 'text("\r\n\r\n")'
        comment: 'Adding custom headers'
        direction: RESPONSE
        rewrite-criteria: 'http.req.is_valid'
 ```

### Kubectl command to apply the RewritePolicy CRD
 ```
 kubectl create -f add_custom_headers.yaml
 ```

### A simple curl request should have desired custom headers within HTTP response packet
```
Saravanas-MacBook-Pro:~ saravanakumarr$ curl -vvv http://app.cic-citrix.org/
*   Trying 10.102.33.176...
* TCP_NODELAY set
* Connected to app.cic-citrix.org (10.102.33.176) port 80 (#0)
> GET / HTTP/1.1
> Host: app.cic-citrix.org
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.8.1
< Date: Fri, 29 Mar 2019 12:15:09 GMT
< Content-Type: text/html
< Transfer-Encoding: chunked
< Connection: keep-alive
< X-Powered-By: PHP/5.5.9-1ubuntu4.14
< x-request-time:Fri, 29 Mar 2019 13:27:40 GMT =============> NEW HEADER ADDED
< x-using-citrix-ingress-controller: true  ===============> NEW HEADER ADDED
< 
<html>
<head>
<title> Front End App - v1 </title>
<style>

TRIMMED
........
```

## Usecase 4b) Adding X-Forwarded-For header within HTTP

* [http_x_forwarded_for_insert.yaml](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/usecase4/http_x_forwarded_for_insert.yaml)
```
apiVersion: citrix.com/v1
kind: rewritepolicy
metadata:
  name: httpxforwardedforaddition
spec:
  rewrite-policies:
    - servicenames: 
        - frontend
      rewrite-policy:
        operation: insert_http_header
        target: X-Forwarded-For
        modify-expression: client.ip.src
        comment: 'HTTP Initial X-Forwarded-For header add'
        direction: REQUEST
        rewrite-criteria: 'HTTP.REQ.HEADER("X-Forwarded-For").EXISTS.NOT'

    - servicenames: 
        - frontend 
      rewrite-policy:
        operation: replace 
        target: HTTP.REQ.HEADER("X-Forwarded-For")
        modify-expression: 'HTTP.REQ.HEADER("X-Forwarded-For").APPEND(",").APPEND(CLIENT.IP.SRC)'
        comment: 'HTTP Append X-Forwarded-For IPs'
        direction: REQUEST
        rewrite-criteria: 'HTTP.REQ.HEADER("X-Forwarded-For").EXISTS'
```

### Kubectl command to execute the X-Forwarded-For Yaml file
```
kubectl create -f http_x_forwarded_for_insert.yaml
```

### Example 1: X-Forwarded-For header not present within the HTTP request packet
```
curl http://app.cic-citrix.org/
```

#### Result
![Screenshot 2019-03-29 at 5 56 11 PM](https://user-images.githubusercontent.com/43468858/55232784-8db5ba00-524c-11e9-9b7e-c864bad4765e.png)

### Example 2: X-Forwarded-For header present within HTTP request packet

```
 curl --header "X-Forwarded-For: 1.1.1.1" http://app.cic-citrix.org/
```
#### Result
![Screenshot 2019-03-29 at 5 58 49 PM](https://user-images.githubusercontent.com/43468858/55232781-8bebf680-524c-11e9-86e1-2209d519f135.png)

## Usecase 5: HTTP request redirected to HTTPS request using responder policy

* [http_to_https_redirect.yaml](https://github.com/SaravanakumarR2018/CIC-crd-usecases/blob/master/usecase5/http_to_https_redirect.yaml)

```
apiVersion: citrix.com/v1
kind: rewritepolicy
metadata:
  name: httptohttps
spec:
  responder-policies:
    - servicenames:
        - frontend
      responder-policy:
        redirect: 
          url: '"https://" +http.req.HOSTNAME.SERVER+":"+"443"+http.req.url'
        respond-criteria: 'http.req.is_valid'
        comment: 'http to https'
```

### Kubectl command to apply redirect HTTP to HTTPS request RewritePolicy CRD
```
kubectl create -f http_to_https_redirect.yaml
```

### The http request will be redirected to https request
```
Saravanas-MacBook-Pro:~ saravanakumarr$ curl -vvv  http://app.cic-citrix.org
* Rebuilt URL to: http://app.cic-citrix.org/
*   Trying 10.102.33.176...
* TCP_NODELAY set
* Connected to app.cic-citrix.org (10.102.33.176) port 80 (#0)
> GET / HTTP/1.1
> Host: app.cic-citrix.org
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 302 Found : Moved Temporarily
< Location: https://app.cic-citrix.org:443/   =======> Redirected to HTTPS
< Connection: close
< Cache-Control: no-cache
< Pragma: no-cache
< 
* Closing connection 0

```


```
Saravanas-MacBook-Pro:~ saravanakumarr$ curl -vvv  http://app.cic-citrix.org/simple
*   Trying 10.102.33.176...
* TCP_NODELAY set
* Connected to app.cic-citrix.org (10.102.33.176) port 80 (#0)
> GET /simple HTTP/1.1
> Host: app.cic-citrix.org
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 302 Found : Moved Temporarily
< Location: https://app.cic-citrix.org:443/simple     ========> Redirected to HTTPS
< Connection: close
< Cache-Control: no-cache
< Pragma: no-cache
< 
* Closing connection 0

```

