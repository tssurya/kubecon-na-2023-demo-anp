# kubecon-na-2023-demo-anp
Repo containing yamls for KubeCon NA 2023 ANP Demo

Demo Instructions

Roles:

* cluster-admin who has the highest priviledge and permissions across the entire cluster - Can create ANPs, NPs, BANPs. Whenever a tenant wants to join the cluster, the cluster-admin creates the namespace for that tenant and sets the right labels.
* namespace-admins a.k.a tenant owners who get the namespace from the cluster-admin - Cannot modify the namespace itself, but can create application pods in their respective namespaces. They can also create network policies inside their respective namespaces.

We have three namespaces (tenants):

* `dumbledore` namespace which has sensitive pods that are privy to the principal dumbledore's discretion
* `gryffindor` namespace which has tenant `gryffindor`'s pods that are privy to the captain harry-potter's discretion
* `slytherin` namespace which has tenant `slytherin`'s pods that are privy to the captain draco-malfoy's discretion

Create them using `kubectl create -f namespaces.yaml`

We have two Admin Network Policies in the cluster:

* `deny-ingress-from-houses` that strongly deny's all ingress traffic from both the houses to the dumbledore (principal's) namespace
* `allow-ingress-from-dumbledore` that strongly allow's all ingress traffic from dumbledore namespace to both the houses

With these two ANPs cluster-admin establishes that the principal can talk to both the houses, however the houses cannot talk to the principal

Create them using `kubectl create -f anps.yaml`

We have one network policy in the cluster:

* `allow-ingress-from-houses-to-dumbledore` created in the namespace `dumbledore` by our principal (tenant owner) that implicitly denies all ingress traffic and explicitly allows traffic from both the houses `gryffindor` and `slytherin`. This does the exact opposite of what the first ANP does.

With this NP, the tenant principal (dumbledore) establishes that both the houses can talk to him.

Create this using `kubectl create -f np.yaml`

We have one Baseline Admin Network Policy in the cluster (For the purpose of `Pass` action demo):

* `default` created by the cluster-admin to strongly deny all ingress traffic from both the houses to the dumbledore tenant. This does the opposite of what network policy does.

Create this using `kubectl create -f banp.yaml`
