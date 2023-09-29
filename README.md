# kubecon-na-2023-demo-anp
Repo containing yamls for KubeCon NA 2023 ANP Demo

Demo Instructions

Roles:

* cluster-admin who has the highest priviledge and permissions across the entire cluster - Can create ANPs, NPs, BANPs. Whenever a tenant wants to join the cluster, the cluster-admin creates the namespace for that tenant and sets the right labels.
* namespace-admins a.k.a tenant owners who get the namespace from the cluster-admin - Cannot modify the namespace itself, but can create application pods in their respective namespaces. They can also create network policies inside their respective namespaces.

We have `dumbledore`, who is the principal and administrator of cluster `hogwarts`.

We have four namespaces (tenants):

* `professors` namespace which has tenant `professor`'s pods that are privy to the professor `minerva-mcgonagall`'s discretion
* `gryffindor` namespace which has tenant `gryffindor`'s pods that are privy to the captain harry-potter's discretion
* `slytherin` namespace which has tenant `slytherin`'s pods that are privy to the captain draco-malfoy's discretion
* `forbidden-forrest` namespace which has tenant `forbidden-forrest`'s pods that have dangerous creatures

Create them using `kubectl create -f namespaces.yaml`

Each namespace has daemon-sets that can be created using `kubectl create -f pods.yaml`

## DEMO PART 1a) Showcase Strong ALLOW ANP that is non-overridable
Steps:
1) We have one network policy `deny-ingress` in the namespace `gryffindor` that has denied ingress traffic from all other namespaces (SLIDE-8) already existing in the cluster.
2) PING from `minerva-mcgonagall-0` pod to `harry-potter-0` pod and show traffic NOT working
3) Create admin network policy `anp-strong-allow.yaml` (`allow-ingress-from-professors-to-houses`)
4) PING from `minerva-mcgonagall-0` pod to `harry-potter-0` pod and show traffic is working

## DEMO PART 1b) Showcase Strong DENY ANP that is non-overridable
Steps:
1) We have two network policies `allow-egress-to-forbidden-forrest` in the namespaces `slytherin` and `professors` that have allowed egress to forbidden-forrest (SLIDE-14) already existing in the cluster.
2) PING from `minerva-mcgonagall-0` pod to `forbidden-0` pod and show traffic working
3) Create admin network policy `anp-strong-deny.yaml` (`deny-egress-to-forbidden-forrest-from-everyone`) at priority 5
4) PING from `minerva-mcgonagall-0` pod to `forbidden-0` pod and show traffic is NOT working

## DEMO PART 2a) Showcase PASS ANP - Interaction with NetworkPolicy
Steps:
1) Take-off from where we left previously...
2) PING from `minerva-mcgonagall-0` pod to `forbidden-0` pod and show traffic is NOT working
3) Create admin network policy `anp-pass.yaml` (`pass-from-professors-to-forbidden-forrest`) at priority 3 (SLIDE-16)
4) PING from `minerva-mcgonagall-0` pod to `forbidden-0` pod and show traffic working

* kubectl get nodes -owide
* kubectl get pods -n ovn-kubernetes -owide
* kubectl get pods -A -owide -l purpose=demo
* kubectl describe networkpolicy -n professors
* oc get anp -owide
* cat anp-strong-deny.yaml
* kubectl get pods -A -owide -l purpose=demo
* kubectl exec -n professors minerva-mcgonagall-0 -c minerva-mcgonagall-client -- /agnhost connect $forbidden --timeout=1s --protocol=tcp && echo "CONNECTED" -> FAILURE
* cat anp-pass.yaml
* kubectl create -f anp-pass.yaml
* oc get anp -owide
* kubectl exec -n professors minerva-mcgonagall-0 -c minerva-mcgonagall-client -- /agnhost connect $forbidden --timeout=1s --protocol=tcp && echo "CONNECTED" -> SUCCESS

## DEMO PART 2b) Showcase PASS ANP - Interaction with BaselineAdminNetworkPolicy
Steps:
1) Create baseline admin network policy `default`. This denies all ingress traffic to `professors` and all `egress` traffic from `professors` (SLIDE-17)
2) PING from `minerva-mcgonagall-0` pod to `forbidden-0` pod and show traffic working
3) Delete network policy `allow-egress-to-forbidden-forrest` from namespace `professors` (SLIDE-18)
4) PING from `minerva-mcgonagall-0` pod to `forbidden-0` pod and show traffic is NOT working

* oc get anp -owide
* kubectl get networkpolicy -n professors
* cat banp.yaml
* kubectl create -f banp.yaml
* oc get banp -owide
* kubectl exec -n professors minerva-mcgonagall-0 -c minerva-mcgonagall-client -- /agnhost connect $forbidden --timeout=1s --protocol=tcp && echo "CONNECTED" -> SUCCESS
* kubectl delete networkpolicy -n professors allow-egress-to-forbidden-forrest
* kubectl exec -n professors minerva-mcgonagall-0 -c minerva-mcgonagall-client -- /agnhost connect $forbidden --timeout=1s --protocol=tcp && echo "CONNECTED" -> FAILURE
