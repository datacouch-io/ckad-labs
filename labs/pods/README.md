The basic unit of compute in Kubernetes is the Pod. Pods are responsible for running containers and ensuring their continuous operation.

To define a Pod, you need a minimal YAML configuration that includes metadata and the name of the container image to run.

### API specs

- [Pod](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#pod-v1-core)

<details> <summary>YAML overview</summary>

The simplest Pod configuration looks like this:

```code
apiVersion: v1
kind: Pod
metadata:
  name: whoami
spec:
  containers:
    - name: app
      image: sixeyed/whoami:21.04
```

Every Kubernetes resource requires the following four fields:

-   `apiVersion`: Specifies the version of the resource API to ensure compatibility.
-   `kind`: Specifies the type of the object.
-   `metadata`: Contains additional metadata for the object.
-   `name`: Specifies the name of the object.

The structure of the `spec` field varies depending on the object type. For Pods, the minimum configuration includes:

-   `containers`: A list of containers to run in the Pod.
-   `name`: The name of the container.
-   `image`: The Docker image to run.

> Note: Proper indentation is important in YAML, as object fields are nested using spaces.

</details><br/>

## Running a simple Pod

The `kubectl` command-line tool is used for managing Kubernetes objects. You can create any object by applying the corresponding YAML configuration using the `apply` command.

-   [whoami-pod.yaml](../blob/main/labs/pods/specs/whoami-pod.yaml) contains the Pod configuration for a simple web app.

To deploy the app using your local copy of the course repository, run the following command:

```code
kubectl apply -f labs/pods/specs/whoami-pod.yaml
```

> The output indicates that nothing has changed. Kubernetes operates based on the desired state deployment model.

You can use familiar commands to retrieve information about the Pod:

```code
kubectl get pods kubectl get po -o wide
```
> The second command uses the shorthand notation `po` and includes additional columns, such as the Pod's IP address.

What additional information do you see in the second command's output? How can you print all the Pod information in a more readable format?

## Working with Pods

In a production cluster, a Pod can run on any node. You manage the Pod using `kubectl`, so direct access to the server is not required.

📋 To print the container logs, use the following command:

<details> <summary>Not sure how?</summary>

```code
kubectl logs whoami
```

</details><br/>

To connect to the container inside the Pod, run the following command:

```code
# This  command  will fail:  
kubectl exec -it whoami -- sh
```

> The container image used in this example does not have a shell installed.

Let's try another application:

-   [sleep-pod.yaml](../blob/main/labs/pods/specs/sleep-pod.yaml) runs an application that does nothing.

📋 Deploy the new application using the `labs/pods/specs/sleep-pod.yaml` file and verify that it is running.

<details> <summary>Not sure how?</summary>

```code
kubectl apply -f labs/pods/specs/sleep-pod.yaml
kubectl get pods
```

</details><br/>

This Pod's container does have a shell and includes useful tools.

```code
kubectl exec -it sleep -- sh
```
Now that you're connected inside the container, you can explore the container environment:

```code
hostname whoami
```

You can also examine the Kubernetes network:

```code
nslookup kubernetes  
# This  command  will fail:  ping kubernetes
```

> The Kubernetes API server is available for Pod containers to use, but internal addresses do not support ping.

## Connecting from one Pod to another

Exit the shell session inside the sleep Pod:

```code
exit
```

📋 To print the IP address of the original whoami Pod, use the following command:

<details> <summary>Not sure how?</summary>

```code
kubectl get pods -o wide whoami
```

</details><br/>

> The IP address displayed is the internal IP address of the Pod. Any other Pod in the cluster can connect to it using this address.

Make a request to the HTTP server in the whoami Pod from the sleep Pod:

```code
kubectl exec sleep -- curl -s <whoami-pod-ip>
```

> The output will be the response from the whoami server, which includes the hostname and IP addresses.

## Lab

Pods provide an abstraction layer over containers. They monitor the container and automatically restart it if it exits, ensuring continuous operation of your application. This is the first layer of high availability provided by Kubernetes.


> Stuck? Try the [hints](hints.md) or check the [solution](solution.md).

----------

## Cleanup

Before proceeding, let's clean up by deleting all the Pods we created:

```code
kubectl delete pod sleep whoami sleep-lab
```
