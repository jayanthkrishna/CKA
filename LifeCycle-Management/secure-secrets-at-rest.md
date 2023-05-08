
First create a secret.

    kubectl create secret generic my-secret --from-literal=key1=supersecret

Install etcdctl

    apt-get install etcd-client

Below command helps us to ping etcd api to get the secret as it stored in etcd. Here we are getting secret1.

    ETCDCTL_API=3 etcdctl \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key  \
    get /registry/secrets/default/secret1 | hexdump -C

Configuration and determining whether encryption at rest is already enabled

The kube-apiserver process accepts an argument --encryption-provider-config that controls how API data is encrypted in etcd. The configuration is provided as an API named EncryptionConfiguration.

kube-apiserver runs as a process.so,

    ps -aux | grep kube-apiserver

It shows all the options available.

    ps -aux | grep kube-apiserver | grep "encryption-provider-config"

This shows if the encryption-provider-config option is available or not.

kubernetes has its control plane manifests in /etc/kubernetes/manifests

check the kube-apiserver manifest file.

Now create a EncryptionConfiguraion file. Then you can attach this file to encryption-provider-config option.

    apiVersion: apiserver.config.k8s.io/v1
    kind: EncryptionConfiguration
    resources:
    - resources:
        - secrets
        - configmaps
        - pandas.awesome.bears.example
        providers:
        - identity: {}
        - aesgcm:
            keys:
                - name: key1
                secret: c2VjcmV0IGlzIHNlY3VyZQ==
                - name: key2
                secret: dGhpcyBpcyBwYXNzd29yZA==
        - aescbc:
            keys:
                - name: key1
                secret: c2VjcmV0IGlzIHNlY3VyZQ==
                - name: key2
                secret: dGhpcyBpcyBwYXNzd29yZA==
        - secretbox:
            keys:
                - name: key1
                secret: YWJjZGVmZ2hpamtsbW5vcHFyc3R1dnd4eXoxMjM0NTY=
    - resources:
        - events
        providers:
        - identity: {} # do not encrypt events even though  *.* is specified below
    - resources:
        - '*.apps'
        providers:
        - aescbc:
            keys:
            - name: key2
                secret: c2VjcmV0IGlzIHNlY3VyZSwgb3IgaXMgaXQ/Cg==
    - resources:
        - '*.*'
        providers:
        - aescbc:
            keys:
            - name: key3
                secret: c2VjcmV0IGlzIHNlY3VyZSwgSSB0aGluaw==



Refer this link - https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

Below is a sample file.

    apiVersion: apiserver.config.k8s.io/v1
    kind: EncryptionConfiguration
    resources:
    - resources:
        - secrets
        - configmaps
        - pandas.awesome.bears.example
        providers:
        - aescbc:
            keys:
                - name: key1
                secret: <BASE 64 ENCODED SECRET>
        - identity: {}

Generate a 32-byte random key and base64 encode it. If you're on Linux or macOS, run the following command:

head -c 32 /dev/urandom | base64

Place that value in the secret field of the EncryptionConfiguration struct.

Set the --encryption-provider-config flag on the kube-apiserver to point to the location of the config file.

You will need to mount the new encryption config file to the kube-apiserver static pod. Here is an example on how to do that:

Save the new encryption config file to /etc/kubernetes/enc/enc.yaml on the control-plane node.
Edit the manifest for the kube-apiserver static pod: /etc/kubernetes/manifests/kube-apiserver.yaml similarly to this:

    name: kube-apiserver
    namespace: kube-system
    spec:
    containers:
    - command:
        - kube-apiserver
        ...
        - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # <-- add this line
        volumeMounts:
        ...
        - name: enc                           # <-- add this line
        mountPath: /etc/kubernetes/enc      # <-- add this line
        readonly: true                      # <-- add this line
        ...
    volumes:
    ...
    - name: enc                             # <-- add this line
        hostPath:                             # <-- add this line
        path: /etc/kubernetes/enc           # <-- add this line
        type: DirectoryOrCreate             # <-- add this line

Now wait thill the pods are available. It shows the kube-apiserver is working.

Now create a new secret and run the ETCD API command to check how the value is stored in ETCD. It will be encrypted based on the provider you have chosen. Only the new objects created after the above configuration are encrypted.


