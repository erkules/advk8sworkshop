# Authentication and Authorization (RBAC)

## Agenda

* Overview Authentication
* * Certs
* * Oauth2/JWT 
* Nittygritty of Authorization aka RoleBasedAccessControl (RBAC)

<!-- Wir sehen RBAC bei Webhooks und Controllern wieder) -->

# Authentication

* There will be nearly no exercises
* It is about to find out who you/we are
* * User
* * Group
* * ServiceAccount

# Authentication Methods

There are some ways to authenticate against Kubernetes

As a user you are most likely facing

* x509 Certificates
* OpenID Connect Tokens (or any JWT)

# x509 Certificats

* You access the k8s cluster using `~/.kube/config` (on your Jumpserver)
* There is a cert

Gonna get it

~~~
yq -r  '.users[0].user."client-certificate-data"'   ~/.kube/config | base64 -d >/tmp/auth.crt
openssl x509  -text -noout -in /tmp/auth.crt
~~~

Check for the Subject

* O  ==> Group
* CN ==> User

Let's draw a picture why that cert-stuff works

# OIDC/Oauth2

* Most likely you are going to authenticate against an oauth-Provider
* The provider maybe is just a proxy to your AD (but 🤷)


# Authorization

* So after being authenticated
* What are we allowed to do?

# RBAC

* Role Based Access Control
* Let's make it more complicated

# Plan

* using curl to traverse the API
* Have a look into (Cluster)Roles
* Check (Cluster)RoleBinding
* Do some simple stuff

# using curl to traverse the API

Pic about etcd

~~~
kubectl proxy
~~~

Then do ./RBAC/1x1/README.md

# interim conclusion

So it is about to access

* cluster-scoped objects
* namespaced objects
* non-resource endpoints exists, but we don't care

Where access means:

Check for the verbs:

~~~
curl localhost:8001/api/v1
~~~

verbs:

| | |
|---|---|---|
| get | list | listcollection |
| patch | update |  watch |
| delete | deletecollection | create | 
...


# Have a look into (Cluster)Roles

* admin
* cluster-admin
* view
* system:coredns

# Check (Cluster)RoleBinding

Basic idea bind (Cluster)Roles to identities/subjects

| subjects  | How to impersonate |
| --- | --- | 
| User            | kubectl get pods --as --as-group team1                    |
| Group           | kubectl get pods --as username                            |
| Serviceaccount  | kubectl get pods --as system:servcieaccount:testing:test  |




# Do some simple stuff

Please have a look before you deploy it 

~~~
kubectl apply -f RBAC/hallo-ns-sa-rb.yaml
~~~

Question:

But that was a ClusterRole. Is the User `hallo` now a super-user? 😱


# It's so complicated

* RBAC is no fun at all 
* Let's see some tips

~~~
kubectl auth whoami
~~~

# Check Permissions

~~~
kubectl auth can-i get pods haha
kubectl auth can-i --as aishe list pods
kubectl auth can-i list svc --as system:serviceaccount:kube-system:coredns
~~~

# Krew ./. rbac-view

* Nice WebUI to see (Cluster)Roles
* Not "working" on the jumphost
* Because it binds to localhost

~~~
kubectl krew install  rbac-view
kubectl rbac-view
~~~

# Krew ./. rbac-lookup


Easy way to find (Cluster)Roles (and the bindings)

~~~
kubectl krew install  rbac-lookup
kubectl rbac-lookup --kind serviceaccount kamaji-controller-manager -o wide
kubectl rbac-lookup --kind user  bob -o wide
~~~
