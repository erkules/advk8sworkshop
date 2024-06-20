# Network

* Overview
* BuildIn noderouting (but s/topologyKeys/internalTrafficPolicy/)
* NetworkPolicies

# Overview

* One Network for all Pods
* Nothing special about Namespaces (Namespaces are just directories to organize objects (remember))

# Topologyawarness of Services aka ServiceDiscovery

* topologyKeys are deprecated :/
* There is a (non) replacement
* Let's meet internalTrafficPolicy

Window 1:

~~~
watch kubectl get svc,pods -o wide
~~~

Window 2:

~~~
kubectl apply -f Deployments/deployment-local-svc.yaml
kubectl exec -ti jumper -- sh
> apk add curl
> watch curl -s  http://wahnsinn
~~~

Repeat the curl-command and make sure you always connect to a node-local Pod

# That's all

* There is [Topology Aware Routing](https://kubernetes.io/docs/concepts/services-networking/topology-aware-routing/)
* There was topologyKeys for Services but they are deprecated

# Networkpolicies

What you need to understand:

* Networkpolicies are  namespaced object (why is that important?)
* Working with labels (like Service Discovery etc.)

Draw it or it didn't happen


Questions:

* What does it means only to have namespaced Networkpolicies?

# What do the NetworkPolicies look like?


~~~
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: tored
  namespace: netz
spec:
  podSelector:
    matchLabels:
      farbe: rot
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          farbe: blau
    ports: 
    - port: 80
~~~

# An example/exercise

Deploy a Namespace (named netz) with three Deployments (with unique Labels) and their fitting services

~~~
kubectl apply -f NetworkPolicies/basics.yaml
kubectl -n netz get deploy,svc,pod -o wide --show-labels 
~~~


Please:

* `Exec` into every Pod pod install curl `apk add curl` and connect to every service
* Apply a NetworkPolicy 

~~~
kubectl -n netz apply -f NetworkPolicies/pod2pod.yaml
~~~

* Now only the Pods from the blau Deployment should be able to access the Pod from the rot Deployment
* It doesn't matter if we use the svc or the IP of any Pod (labeled farbe=rot)


# Benefits

* Think about shipping your Application with the right NetworkPolicies 
* Think about defining default Networkpolicies for your Namespaces (let's show them)

# Some Examples

Recap:

~~~
kind: NetworkPolicy
spec:
  podSelector:     <<-- Find the Pods in the Namepace
    hallo: welt    <<-- using this Label
  ... 
  policyTypes:      #     Could be both or only one:
  - Ingress        <<--   Define Incomming Traffic
  - Egress         <<--   Define Outgoing Traffic
  egress: []       <<--[] Configuration-Array
  ingress:         <<--[] Configuration-Array
  - from:
    - podSelector:
        matchLabels:
          app: www
~~~

# nameSpaceSelector 

Why and where do they make sense?

~~~
spec:
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name=netzname
~~~

Can I mix namespaceSelector and podSelector?



# Let's Discuss

* This is supposed to take some time!!!!
* Exercises could arise \o/


* Wildcards 
* Empty Arrays
* Egress



# Tools to debug Network

Some nice tools to "debug" Network using `kubectl`

# "ephemeral Container"

What are ephemeral containers?


~~~
kubectl debug -ti --image=tollesimage <PODName>
~~~

And yes not only good for network debugging 



# Links

* [Gifs to understand NetworkPolicies](https://github.com/ahmetb/kubernetes-network-policy-recipes/blob/master/01-deny-all-traffic-to-an-application.md)
* [quasiEditor](https://editor.cilium.io)
* [Auch quasiEditor](https://orca.tufin.io/netpol/)
* [Inspector-Gadget](https://www.inspektor-gadget.io/)
* [k8spacket](https://github.com/k8spacket/k8spacket)
# [np-viewer](https://github.com/runoncloud/kubectl-np-viewer)

# Kubectl Plugins

~~~
kubectl krew install np-viewer
~~~

* Simple overview

~~~
$ kubectl np-viewer -n netz -p rot-5bb7cb48cb-5sj5t

+----------------+---------+-----------+-----------+---------------------+---------------+----------+--------+
| NETWORK POLICY |  TYPE   | NAMESPACE |   PODS    | NAMESPACES SELECTOR | PODS SELECTOR | IP BLOCK | PORTS  |
+----------------+---------+-----------+-----------+---------------------+---------------+----------+--------+
|     tored      | Ingress |   netz    | farbe=rot |        netz         |  farbe=blau   |    *     | TCP:80 |
+----------------+---------+-----------+-----------+---------------------+---------------+----------+--------+
~~~

Nice start but only evaluates podSelector `:/`

~~~
$ kubectl np-viewer -n netz -p blau-54d9867779-82f4k

No network policy was found
~~~


# [Inspector-Gadget](https://www.inspektor-gadget.io/)

You should have the [krew plugin manager](https://krew.sigs.k8s.io/) installed

~~~
kubectl krew install gadget
kubectl gadget deploy
~~~



Trace syscalls:

~~~
kubectl gadget trace exec -n namespace -p podname 
~~~

Trace tcp-connections:

~~~
kubectl gadget trace tcp -n nameaspace -p podname -c containername
~~~

# Network visualisation


I.e. [k8spacket](https://github.com/k8spacket/k8spacket)


# Skipping

I.e. Calico and Cilium offer additional CRDs ...
