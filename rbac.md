# RBAC

Role Based Access Control

# What is it all about

* Please remember it is about objects in etcd (database)
* We use `verbs` to describe the interaction with a specific object
* Objects like Pods, Services etc.

| verb | simplified description | 
| :--  | --- |
| get |  get an object by name |
| list | get a list of all objects of a type |
| patch | `kubectl patch ..` |
| update | am I allowed to change an object |
| watch |  `kubectl get pod -w` |
| delete | delete an object |
| create | guess what? |

# RBAC

* RoleBasedAccess
* AuthZ API-Server
* No ResourceQuotas/Limits/Security
* Think about etcd as a filesystem
* Think about RBAC as filepermissions
* Use something like Kyverno to decide based on the filecontent

# So

* Please change into the directory `./k8sworkshop/RBAC/1x1/`
* We are going to access the API-server using curl
* We also are going to get an idea about the fileystemlayou


# Fingerübungen

Execute in a Terminal/Pane:

~~~
watch kubectl get pods
~~~

Execute in another Terminal/Pane:

~~~
kubectl proxy
~~~

In a third Terminal/Pane:

~~~
# Verb: create
curl -X POST localhost:8001/api/v1/namespaces/default/pods -H 'Content-Type: application/json' -d @pod.json
# Verb: list
curl localhost:8001/api/v1/namespaces/default/pods
# Verb: get
curl localhost:8001/api/v1/namespaces/default/pods/rbacexample 
# Verb: delete
curl -X DELETE localhost:8001/api/v1/namespaces/default/pods/rbacexample 
~~~

# Namespaced Objects

We still need a way to Authorize/Restrict access to a specific `Namespace` (later)


Please enjoy the fileystemlayout :)


~~~
# kubectl get pods -A 
curl localhost:8001/api/v1/pods 
# kubectl get pods -n kube-system 
curl localhost:8001/api/v1/namespaces/kube-system/pods
# kubectl get  nodes # 
curl localhost:8001/api/v1/nodes 
# kubectl get sts -A
curl localhost:8001/apis/apps/v1/statefulsets 
~~~

# FYI

Btw. if you want to find out about the (non)-namespaced resources known to your cluster:

~~~
kubectl api-resources -o wide --namespaced=true 
kubectl api-resources -o wide --namespaced=false
~~~

# Roles and ClusterRoles

* Yes there are objects to store RBAC-Privileges
* Roles are limitted to a Namespaces
* Clusterroles are not 

* As a fact ClusterRoles are sufficient. 
* Later we are going to limit them on a Namespace

# CRD

Exerpt of `kubectl explain --recursive ClusterRoles`

~~~
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole  # or Role
metadata:
  name:
rules <[]PolicyRule>
  apiGroups   <[]string> #or nonResourceURLs     <[]string>
  resources   <[]string>
  verbs       <[]string> -required-
~~~

# ApiGroups? Resources? Verbs?

* We are doing an unpractical exercise
* At least something to learn
* Previeous we had `kubectl api-resources -o wide`


If you killed `kubectl proxy`, then start it again:

~~~
kubectl proxy
~~~

Check:

~~~
curl 127.0.0.1:8001/
~~~

# nonResourceURLs vs. apiGroups?

There are a bunch of nonResourceURLs 

* `/livez`
* `/readyz`
* `/healtz`

* We don't put any effort into nonResourceURLs 
* Feel free to check

~~~
kubectl get clusterrole system:public-info-viewer -o yaml
~~~

Ther you get the `PolicyRule`

~~~
rules:
- nonResourceURLs:
  - /healthz
  - /livez
  - /readyz
  - /version
  - /version/
  verbs:
  - get
~~~

# apiGroups

* `/api`  core resources
* `/apis` the other resources

# Let's check `/api`

* Looks like we've got only the `v1` apiGroup

~~~
curl http://localhost:8001/api
~~~

* Check the output of all the `resources` and there `verbs` in that `apiGroup` v1


~~~
curl http://localhost:8001/api/v1
~~~

# Gamification

~~~
curl 127.0.0.1:8001/api/v1 2>/dev/null | jq '.resources[] | select(.name |contains("pods"))| "\(.name) => \(.verbs)"
~~~

# `/apis`

* Check all the apiGroups and resources under `/apis`
* Hope you find the `verbs` also

# Filling the CRDs

Have a look at (you should understand it)

~~~
kubectl get clusterrole view -o yaml
kubectl get clusterrole admin -o yaml
kubectl get clusterrole system:coredns -o yaml
~~~

Or any ather

# Wildcard

* You can use '*' as wildcard.
* A wonderfull example:)

~~~
kubectl get clusterrole cluster-admin -o yaml 
~~~

# Reminder

* We are still ignoring Roles
* Roles got the same syntax as ClusterRoles anyway
* Besides `metadata.namespace`

# AuthN

* Who's that ...
* We are not talking about authentication 
* But we need personas

Authentication in K8s

* Certs  (CN=Username/O=Gruppe/O=Gruppe)
* JWT-Tokens (Oauth/Webhooks)

Wanna have a look at `ServiceAccounts`?

~~~
/var/run/secrets/kubernetes.io/serviceaccount/token
~~~

# Personas

In Kubernetes 'you' can identify as

* User
* Group
* ServiceAccounts

<!-- -->

* Technically they don't differ (regarding RBAC)
* We will see them again later

# [Cluster]RoleBinding

* We got ClusterRoles
* We got Personas

All we do is to bind a ClusterRole to a Persona

Please check:

~~~
./k8sworkshop/RBAC/clusterrolebinding-admin.yaml
./k8sworkshop/RBAC/clusterroleServiceaccount.yaml
~~~


# Cluster-RoleBinding

* Rolebinding sets Privileges of a Persona in a single Namespace
* A ClusterRolebinding needs a ClusterRole to bind
* A RoleBinding can bind a Role or a ClusterRole?!

* WTF?! How do I Rolebind a ClusterRole?!
* * Easy: You got the privileges of the ClusterRole in that Namespace only
* What ist about the non-namespaced Resources?
* * The cluster-resources don't fint into a Namespace and are dropped.

This is the reason you are fine with ClusterRoles.

~~~
kubectel get roles -A
kubectel get clusterroles
~~~


Mehr dazu in Authentifizierung

-->

# Impersonation

* As we don't want to create other Users
* Let's use impersonation of kubectl
* Also a nice way to debug user-stuff


(`--as-group` braucht immer einen User (`--as`))

~~~
kubectl get pods                    # Using my Persona
kubectl get pods --as erkan         # Impersonate User Erkan
kubectl get pods --as default --as-group system:masters  # Group system:masters
kubectl get pods --as system:serviceaccount:default:default  # ServiceAccount default in Namespace default
~~~

FYI:

~~~
system:serviceaccount:default:default 
            \            \      \-> name
             \            \-> Namespace
              \ -> It is a ServiceAccount

~~~


# `can-i`

* Dry-Run of Impersonation 

~~~
kubectl auth can-i create  pods --as system:serviceaccount:default:default
kubectl auth can-i create  pods --as erkan
kubectl auth can-i create  pods --as-group sysadmins
~~~


# So some Examples

~~~
kubectl apply  -f RBAC/clusterrolebinding-admin.yaml
kubectl  --as erkan get nodes
kubectl  --as allman get nodes
~~~

* What is working?
* Why it is working?

# NamespaceAdmin

* Wir machen den nicht existenten User `kunde` zum Admin des Namespaces `kundens`

~~~
kubectl create ns kundens
kubectl apply -n kundens -f RBAC/rolebinding-namespaceadmin.yaml
kubectl -n default --as kunde get pods
kubectl -n kundens --as kunde get pods
~~~

# (Cluster)Rolle erstellen

Please have a look before you deploy it

~~~
kubectl apply -f RBAC/role-pods-list.yaml
~~~


~~~
kubectl apply -f RBAC/hallo-ns-sa-rb.yaml
~~~

Question:

But that was a ClusterRole. Is the User `hallo` now a super-user? 😱




# Some Plugins

~~~
kubectl krew install access-matrix
kubectl access-matrix --verb get --as erkan -n default
kubectl access-matrix  --sa kube-system:default -n default
~~~

<!--
# Passende kubectl plugins

Welche Rollen hat er denn?

~~~
kubectl krew install rbac-lookup
kubecl rbac-lookup erkan
kubecl rbac-lookup kube-proxy
~~~
-->

# Passende kubectl plugins

~~~
kubectl krew install rbac-tool
kubectl rbac-tool lookup coredns
kubectl rbac-tool who-can delete nodes
kubectl rbac-tool viz --outformat dot && cat rbac.dot | dot -Tpng > rbac.png  && open rbac.png
kubectl rbac-tool policy-rules -e '^system:auth'
~~~

# Extra

How to assimilate new Rules (i.e. from an Operator)?

~~~
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.authorization.k8s.io/aggregate-to-admin: "true"
~~~

* ClusterRoles using a matching Label are being importet




# TODO

Recourcename

# Swagger

* Another way to access the API

Swagger.json:

~~~
kubectl get --raw /openapi/v2 >swagger.json
~~~

(Online)Editor or:

~~~
docker container run --rm -p 8080:8080\
            -e SWAGGER_JSON=/swagger.json \
            -v $(pwd)/swagger.json:/swagger.json \
             swaggerapi/swagger-ui
~~~


# Fin
