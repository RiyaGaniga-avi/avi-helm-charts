## Description of tunables of AKO

This document is intended to help the operator make the right choices while deploying AKO with the configurable settings.
The values.yaml in AKO affects a configmap that AKO's deployment reads to make adjustments as per user needs. Listed are detailed
explanation of various fields specified in the values.yaml. If the field is marked as "editable", it means that it can be edited without an AKO POD restart.

### configs.controllerVersion

This field is used to specify the Avi controller version. While AKO is backward compatible with most of the 18.2.x Avi controllers,
the tested and preferred controller version is `18.2.8` 

### configs.controllerIP

This field is usually not present in the `value.yaml` by default but can be provided with the `helm` install command to specify
the Avi Controller's IP address. If you are using a containerized deployment of the controller, pls use a fully qualified controler
IP address/FQDN. For example, if the controller is hosted on 8443, then controllerIP should: `x.x.x.x:8443`

### configs.shardVSSize

AKO uses a sharding logic for Layer 7 ingress objects. A sharded VS involves hosting multiple insecure or secure ingresses hosted by
one virtual IP or VIP. Having a shared virtual IP allows lesser IP usage since reserving IP addresses particularly in public clouds
incur greater cost. 


### configs.fullSyncFrequency

This field is used to set a frequency of consitency checks in AKO. Typically inconsistent states can arise if users make changes out
of band w.r.t AKO. For example, a pool is deleted by the user from the UI of the Avi Controller. The full sync frequency is used
to ensure that the models are re-conciled and the corresponding Avi objects are restored to the original state. 

### configs.ingressApi

If your kubernetes APIs don't support the newly introduced ingress `networking.k8s.io/v1beta1` Api version but instead only supports
the deprecated (1.16 onwards) `extensionv1` APIs - then this field can be used to tune AKO accordingly. This feature will be soon
deprecated in AKO as well as the customer based move towards newer kubernetes version 1.16 and beyond.

### config.defaultDomain

If you have multiple sub-domains configured in your Avi cloud, use this knob to specify the default sub-domain.
This is used to generate the FQDN for the Service of type loadbalancer. If unspecified, the behavior works on a sorting logic.
The first sorted sub-domain in chosen, so we recommend using this parameter if you want to be in control of your DNS resolution for service of type LoadBalancer.

### configs.defaultIngController

This field is related to the ingress class support in AKO specified via `kubernetes.io/ingress.class` annotation specified on an
ingress object.

- If AKO is set as the default ingress controller, then it will sync everything other than the ones on which the ingress class is specified and is not equals to “avi”.
- If Avi is not set as the default ingress controller then AKO will sync only those ingresses which have the ingress class set to “avi”.

If you do not use ingress classes, then keep this knob untouched and AKO will take care of syncing all your ingress objects to Avi.

### configs.l7ShardingScheme

AKO allows two types of sharding logic currently. These are `hostname` based or `namespace` based. The hostname based sharding uses
the hostname specified in the Ingress rules to determine the shard VS number while the `namespace` based sharding logic shards the
ingress object based on the namespace on which it is created.

### configs.cniPlugin

Use this flag only if you are using calico as a CNI and you are looking to a sync your static route configurations automatically.
Once enabled, this flag is used to read the `blockaffinity` CRD in calico to determine the POD CIDR to Node IP mappings. If you are
on an older version of calico where `blockaffinity` is not present, then leave this field as blank. AKO will then determine the static
routes based on the Kubernetes Nodes object as done with other CNIs.

### configs.cloudName

This field is used to specify the name of the IaaS cloud in Avi controller. For example, if you have the VCenter cloud named as "Demo"
then specify the `name` of the cloud name with this field. This helps AKO determine the IaaS cloud to create the service engines on.

### configs.clusterName

The `clusterName` field primarily identifies your running AKO instance. AKO internally uses this field to tag all the objects it creates on Avi Controller. All objects created by a particular AKO instance have a prefix of `<clusterName>--` in their names and also populates the `created_by` like so `ako-<clusterName>`.

Each AKO instance mapped to a given Avi cloud should have a unique `clusterName` parameter. This would maintain uniqueness of object naming across Kubernetes clusters.

### configs.subnetIP and configs.subnetPrefix and configs.networkName

AKO supports dual arm deployment where the Virtual IP network can be on a different subnet than the actual Port Groups on which the kubernetes nodes are deployed.

These fields are used to specify the Virtual IP network details on which the user wants to place the Avi virtual services on.


### configs.disableStaticRouteSync

This flag can be used in 2 scenarios:
 - If your POD CIDRs are routable either through an internal implementation or by default.
 - If you are working with multiple NICs on your kubernetes worker nodes and the default gateway is not from the same subnet as
   your VRF's PG network.
   
### configs.logLevel *(editable)*

This flag defines the logLevel for logging and can be set to one of `DEBUG`, `INFO`, `WARN`, `ERROR` (case sensitive).
The logLevel value specified here gets populated in the ConfigMap and can be edited at any time while AKO is running. AKO picks up the change in the param value and sets the logLevel at runtime, so AKO pod restart is not required.



### configs.deleteConfig [editable]

This flag is intended to be used for deletion of objects in AVI Controller. The default value is false. 
If the value is set to true while while booting up, AKO won't process any kubernetes object and stop regular operations. 

While AKO is running, this value can be edited to "true" in AKO configmap to delete all abjects created by AKO in AVI.
After that, if the value is set to "false", AKO would resume processing kubernetes objects and recreate all the objects in AVI. 


### avicredentials.username and avicredentials.password

The username/passwword of the Avi controller is specified with this flag. The username/password are base64 encoded by helm and a corresponding secret
object is used to maintain the same. Editing this field requires a restart (delete/re-create) of the AKO pod.

### image.repository

If you are using a private container registry and you'd like to override the default dockerhub settings, then this field can be edited
with the private registry name.
