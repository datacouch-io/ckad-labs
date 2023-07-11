The [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) is a Kubernetes API object that enables external access to services within a cluster by defining a set of rules. These rules are implemented and enforced by an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

In this Lab, we will guide you through the process of setting up a basic Ingress configuration that routes incoming requests to either the 'web' or 'web2' service based on the HTTP URI.

## Prerequisites
This Lab assumes that you have minikube installed to run a local Kubernetes cluster.

## Create a minikube cluster
If you haven't already set up a local cluster, use the command `minikube start` to create one.

## Enable the Ingress controller
* To enable the NGINX Ingress controller, run the following command:

  ```code
  minikube addons enable ingress
  ```

* Verify that the NGINX Ingress controller is running:

  ```code
  kubectl get pods -n ingress-nginx
  ```

> It may take a minute for the pods to start running. Once running, the output should show the Ingress controller pod(s).

  The output is similar to:

  ```code
  NAME                                        READY   STATUS      RESTARTS    AGE
  ingress-nginx-admission-create-g9g49        0/1     Completed   0          11m
  ingress-nginx-admission-patch-rqp78         0/1     Completed   1          11m
  ingress-nginx-controller-59b45fb494-26npt   1/1     Running     0          11m
  ```

## Deploy a hello, world app

* Create a Deployment using the following command:

  ```code
  kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
  ```
  The output should be:
  ```code
  deployment.apps/web created
  ```

* Expose the Deployment:

  ```code
  kubectl expose deployment web --type=NodePort --port=8080
  ```

The output should be:

  ```code
  service/web exposed
  ```

* Verify that the Service is created and available on a node port:

  ```code
  kubectl get service web
  ```
  The output should resemble the following:

  ```code
  NAME      TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
  web       NodePort   10.104.133.249   <none>        8080:31637/TCP   12m
  ```

* Visit the Service via NodePort:

  ```code
  minikube service web --url
  ```

  The output should resemble the following:

  ```none
  http://172.17.0.15:31637
  ```

  The resulting output should resemble the following:

  ```code
  Hello, world!
  Version: 1.0.0
  Hostname: web-55b8c6998d-8k564
  ```

  You can now access the sample application using the Minikube IP address and NodePort. The next step demonstrates accessing the application using the Ingress resource.

## Create an Ingress
The following manifest defines an Ingress that directs traffic to your Service using `hello-world.info`.


<details>
  <summary>View `example-ingress.yaml` from the following file: </summary>

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
```
</details><br/>

* Create the Ingress object by executing the following command:

  ```code
  kubectl apply -f https://k8s.io/examples/service/networking/example-ingress.yaml
  ```

  The output should be:

  ```
  ingress.networking.k8s.io/example-ingress created
  ```

  Verify that the IP address is set:

  ```code
  kubectl get ingress
  ```

  > This may take a few minutes.

  You should observe an IPv4 address in the ADDRESS column, similar to the following:

  ```
  NAME              CLASS    HOSTS              ADDRESS        PORTS   AGE
  example-ingress   <none>   hello-world.info   172.17.0.15    80      38s
  ```

* Verify that the Ingress controller is correctly directing traffic:

  ```code
  curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
  ```

  You should see the following response:
  ```
  Hello, world!
  Version: 1.0.0
  Hostname: web-55b8c6998d-8k564
  ```

You can also visit `hello-world.info` in your browser.

* **Optionally**

     Retrieve the external IP address reported by minikube:
     ```code
     minikube ip
     ```
     Add a line similar to the following to the bottom of the /etc/hosts file on your computer (you will need administrator access):

     ```none
     172.17.0.15 hello-world.info
     ```
     > Ensure that you change the IP address to match the output from minikube ip.

     After making this change, your web browser will send requests for `hello-world.info` URLs to Minikube.

## Create a second Deployment
* Create another Deployment using the following command:

  ```code
  kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
  ```

  The output should be:

  ```
  deployment.apps/web2 created
  ```
* Expose the second Deployment:

  ```code
  kubectl expose deployment web2 --port=8080 --type=NodePort
  ```

  The output should be:
  ```
  service/web2 exposed
  ```

## Modify the existing Ingress
Create/Edit the existing `example-ingress.yaml` manifest and add the following lines at the end:

  ```code

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 8080
          - path: /v2
            pathType: Prefix
            backend:
              service:
                 name: web2
                 port:
                   number: 8080

  ```

Apply the changes:

```code
kubectl apply -f example-ingress.yaml
```

You should see:
```
ingress.networking/example-ingress configured
```

## Test your Ingress
Access the first version of the Hello World app.

```code
curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
```
The output should resemble the following:

```
Hello, world!
Version: 1.0.0
Hostname: web-55b8c6998d-8k564
```

Access the second version of the Hello World app.

```code
curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info/v2
```

The output is similar to:

```
Hello, world!
Version: 2.0.0
Hostname: web2-75cd47646f-t8cjk
```

> Note: If you did the optional step to update `/etc/hosts`, you can also visit `hello-world.info` and `hello-world.info/v2` from your browser.

___

## Cleanup

```
kubectl delete all --all
```
