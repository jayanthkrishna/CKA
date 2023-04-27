If no scheduler, you can schedule the pod on a particular node using nodeName field.

kubectl get pods --selector=bu=finance,bu=finance,tier=frontend

Taints and Tolerations 

Taints are for nodes. It helps nodes to have conditions for pods to be scheduled on them. Tolerations are for pods.

    kubectl taint nodes node-name key=value:taint-effect

    kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-   

    kubectl taint nodes node-name key=value:taint-effect-

this is to untaint a node. put a dash(-) at the end of the command to untaint a node.



taint-effect have 3 values - 
NoSchedule - cant be scheduled on the node.
PreferNoSchedule - System will try not to schedule.But not guarenteed
NoExecute - no new pods will be scheduled and also existing pods will be kicked out if they dont tolerate the taint. 

For tolerations, you add tolerations under spec. under tolerations, you add key value pairs along with the effects.

    tolerations:
        - key: "app"
          operator: "Equal"
          value:"blue"
          effect:"NoSchedule"

All the values should be in quotes.

Taints and tolerations doesnt tell a pod to go to particular node. They tell the nodes to only pods with certain tolerations.

Node affinity is used to restrict a pod to certain nodes.

Node Selector: To select a particular node for a pod.

    so, under spec section, add nodeSelector and the label by which the selection is done.

    nodeSelector:
        size: Large

"size: Large" is a label on node not a literal valueof how big is the node. IT matches the same way replicaset matches the pods using labels from selector section.

To label a node,

    kubectl label nodes node-name label-key:label-value

Node Affinity is a bit complex to write when you first look at it. You write affinity under the spec section in pod definition. for deployment, you write this affinity section under template.spec section.

    affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                    - matchExpressions:
                        - key: size
                          operator: In
                          values:
                            - Large
                            - Medium

    affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                    - matchExpressions:
                        - key: size
                          operator: Exists

Types of Node Affinity available:

Available:

requiredDuringSchedulingIgnoredDuringExecution

preferredDuringSchedulingIgnoredDuringExecution

planned:

requiredDuringSchedulingRequiredDuringExecution

Example for node affinity in a deployment:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: red
    spec:
        replicas: 2
        selector:
            matchLabels:
            run: nginx
        template:
            metadata:
            labels:
                run: nginx
            spec:
                containers:
                    - image: nginx
                    imagePullPolicy: Always
                    name: nginx
                affinity:
                    nodeAffinity:
                    requiredDuringSchedulingIgnoredDuringExecution:
                        nodeSelectorTerms:
                        - matchExpressions:
                        - key: node-role.kubernetes.io/control-plane
                            operator: Exists

Node Affinity vs Taints and Tolerations

Example:

Lets say you have a total of 5 nodes and 5 pods. We need 3 of the pods of color blue, red, green have to be scheduled to nodes of color blue, red, green respectively. The other 2 pods can get scheduled on other 2 nodes. And also the colored pods shoudnt schedule any other pods of diierent color. To achieve this, we use both Node affinity and taints and tolerations together.

First we taint thr nodes with respective colors so that no other pod will get scheduled on them expect for the same color pod. and you add tolerations to the pods so that they can enter the respective colored nodes. 

Now there is a chance that the pods of color may get scheduled on the no color nodes. To avoid this, we use node affinity on pods. We add nodeSelector and add the respective color label. Now the pods will go the desired nodes only and the nodes only accepts the same colored pods.


Resource Requests and limits:

Add resources under containers in pod spec.
    resources:
        requests:
            memory: "1Gi"
            cpu: 1
        limits:
            memory: "2Gi"
            cpu: 2

DaemonSet: on pod per node. Ex: kube-proxy or any monitoring application

    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
        name: monitor-daemon
    spec:
        selector:
            matchLabels:
                app: monitoring-agent
        template:
            metadata:
                labels:
                    app: monitoring-agent
            spec:
                containers:
                    - name: monitoring-agent
                      image: monitoring-agent
This is similar to replicaset

Static Pods

Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. Unlike Pods that are managed by the control plane (for example, a Deployment); instead, the kubelet watches each static Pod (and restarts it if it fails).

Static Pods are always bound to one Kubelet on a specific node.

The kubelet automatically tries to create a mirror Pod on the Kubernetes API server for each static Pod. This means that the Pods running on a node are visible on the API server, but cannot be controlled from there. The Pod names will be suffixed with the node hostname with a leading hyphen.

The spec of a static Pod cannot refer to other API objects (e.g., ServiceAccount, ConfigMap, Secret, etc).


the path for static manifest files is passed as option in --pod-manifest-path=/etc/kubernetes/manifests

--config=kubeconfig.yaml then in kubeconfig.yaml look for staticPodPath key to get the path for the manifests.

To check for the path for manifests in any node, ssh into the node
    ssh node1

then check for 






