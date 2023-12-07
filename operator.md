# Webhooks and Operators

Picture and Idea


# Webhooks/[Admission]Controller

Let's get started with them

# Known  [AdmissionController](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

* LimitRanger
* RessourceQuota
* NamespaceLifecycle
* Priority
* MutatingAdmissionWebhook
* ValidatingAdmissionWebhook
* ImagePolicyWebhook
* AllwaysPullImages
* ....

~~~
PIC of ControllerPipeline
~~~

# 


* Security
* Governance
* ..



# Webhooks easy to extend

* [ImagePolicyWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook)
* MutatingWebhook
* * Changes the object 
* ValidatingAdmissionWebhook
* * Just returns yes/no
* * easy 💃


# Webhook Registration


* `validationWebhookConfiguration` tells K8s a kind of Workload should be sent to a specific Servcie
* This is the difference to an Operator. The Controller gets the traffic and can manipulate/stop the object before it hits etcd 
and even can be consumed bei an Operator


Behind the Service is a Webserver giving following answer: 
(`Content-Type: application/json`)

~~~
{ "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": false/true
    "status": {
    "code"    : 403
    "message" : "imho you suck"
} } }
~~~



# Everything prepared

(Controller/webhookregistration.yaml)

~~~
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "webhook-check-example"
webhooks:
- name: "webhook-check-example.linsenraum.de"
  failurePolicy: Fail
  rules:
  - apiGroups:   [""]
    apiVersions: ["v1"]
    operations:  ["DELETE"]
    resources:   ["pods"]
    scope:       "Namespaced"
  clientConfig:
    service:
      namespace: "webhook"
      name: "check"
    caBundle:
~~~








#  Let's register (nothing)


~~~
kubectl apply -f webhookregistration.yaml
kubectl create ns loeschen
kubectl label ns loeschen k8scamp=spielen
~~~

# failurePolicy


~~~
kubectl -n loeschen apply -f Pods/pod.yaml
kubectl -n loeschen delete -f Pods/pod.yaml
~~~

In webhookregistration.yaml =>  `failurePolicy: Ignore`

and
~~~
kubectl apply -f webhookregistration.yaml
~~~

Now let's delete the Pod again:

~~~
kubectl -n loeschen delete -f Pods/pod.yaml
~~~


# But let't deploy a servic

This is an  AdmissionReview:

~~~
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "request": {
    # Diese uid brauche wir so oder so :/
    "uid": "705ab4f5-6393-11e8-b7cc-42010a800002",
  ..
  "name": "my-deployment"
  "namespace": "my-namespace",
  ..
...
~~~

# This is what we need our "Service" to do

* Status: 200 HTTP
* Content-Type: application/json

~~~
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true/false
  }
}
~~~


# Webhook

~~~
kubectl apply -f webhook-deployment.yaml
~~~

Pod-dance again:

~~~
kubectl -n loeschen apply -f Pods/pod.yaml
kubectl -n loeschen delete -f Pods/pod.yaml
~~~



# MutatingAdmissionWebhook

* Info only
* We return a patch



~~~
{ "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true,
    "patchType": "JSONPatch",
    "patch": "base64-encoded patch"
 } }
~~~









# Operators

What's the difference?

Picture 



# Links

* [OperatorFramework](https://operatorframework.io/)
* [Kubebuilder](https://github.com/kubernetes-sigs/kubebuilder)
* [OperatorHub](https://operatorhub.io/)  <-- 🤷

# Agenda

* Create a CRD
* Use that CRD
* Ignore that CRD
* Impersonate an operator using kubectl
* Deploy a simple pseudo operator-script
* Explain the things

# Directory

~~~
./OperatorCRD
~~~


# Extend K8s with CRDs

~~~
kubectl api-resources | grep linsenraum
kubectl apply -f meineEigeneCRD.yaml
kubectl api-resources | grep linsenraum
kubectl explain --recursive erkan
~~~

~~~
kubectl apply -f erkan.yaml
kubectl get er
~~~

# 

* We ignore CRDs from now
* But that's *the* selling point

# Operator?

* Just some Code running *with* the Cluster
* Most of the time in the Cluster

Control Loop

* Maybe the most important part of an Operator [SHOULD]
* Here is the idea of converging K8s

~~~
                    +--- reconsile ----+
start  -> watch --> |  diff   -->  act |
            ^\      +---------------|--+
              \---------------------|
~~~

# Simple Example 

~~~
kubectl apply -f OperatorCRD/kubecl-operator-interactive.yaml  
kubectl exec -ti kubectl -- sh 
~~~

Explain also ServiceAccount etc.


# Also Simple Example 

Have a look into:

~~~
myoperator.yaml
~~~


# Verstanden?


~~~
kubectl apply -f myoperator.yaml
~~~

* Attention: script needs to be started manually
* Check for AdmissionReview




# Examples

Directory: ./Operators/Percona 

Check the README.md for installing the PerconaOperator
