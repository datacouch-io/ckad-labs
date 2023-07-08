[Kubectl](https://kubectl.docs.kubernetes.io/references/kubectl/) is a powerful command-line tool used to interact with a Kubernetes cluster. It provides various commands to deploy applications and manage objects within the cluster.

## Interacting with Nodes

Two commonly used Kubectl commands are `get` and `describe`. These commands allow you to retrieve information about different objects. Let's focus on the nodes in your cluster:

```code
kubectl get nodes
```

> This command displays a table with basic information about the nodes in your cluster.

```code
kubectl  describe  nodes
```

> The `describe` command provides more detailed and readable information about the nodes.

## Getting Help

Kubectl offers built-in help functionality to assist you in navigating its commands and resources. You can use it to list all available commands or view details about a specific command:

```code
kubectl  --help  kubectl  get  --help
```

Additionally, you can learn about Kubernetes resources by asking Kubectl to provide explanations:

```code
kubectl explain node
```

## Querying and Formatting Output

As you continue working with Kubectl, you'll often need to query and format the output to suit your requirements. Familiarize yourself with some key features, such as querying.

Kubectl supports different output formats. Try displaying your node details in JSON format:

```code
kubectl get node <your-node> -o json
```

Refer to the help documentation to explore other available output formats.

One useful feature is [JSON Path](https://kubernetes.io/docs/reference/kubectl/jsonpath/), a query language that allows you to extract specific fields from the JSON output. For example:

```code
kubectl get node <your-node> -o jsonpath='{.status.capacity.cpu}'
```

> This command retrieves the number of CPU cores associated with the specified node.

What happens if you execute the same command without specifying a node name?

## Lab Exercise

In Kubernetes, every object can be assigned labels, which are key-value pairs used to provide additional information about the object. Use Kubectl to find the labels associated with your node. This will help confirm the CPU architecture and operating system being used.

> If you need assistance, refer to the [hints](hints.md) or check the [solution](solution.md).
