# OpenLDAP Helm Chart

## Introduction
[Openldap](https://github.com/openldap/openldap.git) 
OpenLDAP Software is an open source implementation of the Lightweight Directory Access Protocol.


## Prerequisites 
* Kubernetes 1.7+
* PV support on the underlying infrastructure
* Tiller 2.7.2 or later

## PodSecurityPolicy Requirements
This chart requires a PodSecurityPolicy to be bound to the target namespace prior to installation. Choose predefined ibm-anyuid-psp PodSecurityPolicy.


## Resources Required
The chart deploys pods consuming minimum resources as specified in the resources configuration parameter (default: Memory: 200Mi, CPU: 100m)

## Note 
The original work for this helm chart is present @ [Helm Charts Charts]( https://github.com/helm/charts) Based on the [openldap]( https://github.com/helm/charts/tree/master/stable/openldap) chart

## Chart Details
This chart will do the following:

* Instantiate an instance of OpenLDAP server

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
$ helm install --name my-release stable/ibm-openldap
```

## Configuration

We use the docker images provided by https://github.com/osixia/docker-openldap. The docker image is highly configurable and well documented. Please consult to documentation for the docker image for more information.

The following table lists the configurable parameters of the ibm-openldap chart and their default values.

| Parameter                       | Description                                                   | Default            |
| ------------------------------- | ------------------------------------------------------------- | -------------------|
| `replicas`                      | How many replicas to schedule                                 | `1`                |
| `image.repository`              | Container image repository                                    | `siji/openldap-ppc64le`  |
| `image.tag`                     | Container image tag                                           | `2.4.42`           |
| `image.pullPolicy`              | Container pull policy                                         | `IfNotPresent`     |
| `containerName`                 | Name of the container in the Pod                              | `slapd`            |
| `service.type`                  | Service type                                                  | `ClusterIP`        |
| `service.ldapPort`              | External service port for LDAP                                | `389`              |
| `service.sslLdapPort`           | External service port for SSL+LDAP                            | `636`              |
| `env`                           | List of key value pairs as env variables to be sent to the docker image. See https://github.com/osixia/docker-openldap for available ones | `[see values.yaml]`  |
| `adminPassword`                 | Password for admin user. Unset to auto-generate the password  | None               |
| `configPassword`                | Password for config user. Unset to auto-generate the password | None               |
| `customLdifFiles`               | Custom ldif files to seed the LDAP server. List of filename -> data pairs | None   |
| `persistence.enabled`           | Whether to use PersistentVolumes or not                       | `false`            |
| `persistence.storageClass`      | Storage class for PersistentVolumes.                          | `<unset>`          |
| `persistence.accessMode`        | Access mode for PersistentVolumes                             | `ReadWriteOnce`    |
| `persistence.size`              | PersistentVolumeClaim storage size                            | `8Gi`              |
| `resources`                     | Container resource requests and limits in yaml                | `{}`               |
| `test.image.repository`         | Test container image requires bats framework                  | `dduportal/bats`   |
| `test.image.tag`                | Test container tag                                            | `0.4.0`            |


Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install --name my-release -f values.yaml stable/ibm-openldap
```

> **Tip**: You can use the default `values.yaml`


## Cleanup orphaned Persistent Volumes

Deleting the Deployment will not delete associated Persistent Volumes if persistence is enabled.

Do the following after deleting the chart release to clean up orphaned Persistent Volumes.

```bash
$ kubectl delete pvc -l app=${RELEASE-NAME}-ibm-openldap
```

## Testing

Helm tests are included and they confirm the first three cluster members have quorum.

```bash
helm test <RELEASE_NAME>
RUNNING: foolish-mouse-ibm-openldap-service-test-akmms
PASSED: foolish-mouse-ibm-openldap-service-test-akmms
```

It will confirm that we can do an ldapsearch with the default credentials

## Note (Cluster Image Security)
As container image security feature is enabled, create an image policy for a namespace with the following rule for the chart to be deployed in the `default` namespace:

```console
apiVersion: securityenforcement.admission.cloud.ibm.com/v1beta1
kind: ImagePolicy
metadata:
  name: helm-chart
  namespace: default
spec:
  repositories:
  - name: docker.io/siji/openldap-ppc64le:2.4.42
    policy:
      va:
        enabled: false
```

## Limitations

## NOTE
This chart has been validated on ppc64le.
