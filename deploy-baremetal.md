# Deploying a bare metal cluster

## Preqrequisites

Before deploying your cluster, you will need:

1. 2 routeable ip addresses:

   - One for the api
   - One for the default ingress service

2. A DNS entry mapping `api.<cluster>.<basedomain>` to your API address

3. A DNS entry mapping `*.apps.<cluster>.<basedomain>` to your ingress address

4. A Red Hat pull secret. You can download one from <https://console.redhat.com/openshift/downloads>.

## Deploying a cluster using the hcp cli

You can use the `hcp create cluster agent` command to deploy a bare metal cluster. The complete command line will look something like this:

```
CLUSTERNAME=mycluster
BASEDOMAIN=nerc.mghpcc.org

hcp create cluster agent \
  --name="$CLUSTERNAME" \
  --pull-secret=pull-secret.txt \
  --agent-namespace=hardware-inventory \
  --base-domain=int.massopen.cloud \
  --api-server-address="api.$CLUSTERNAME.$BASEDOMAIN" \
  --etcd-storage-class=lvms-vg1 \
  --ssh-key my-ssh-key.pub \
  --namespace clusters \
  --control-plane-availability-policy HighlyAvailable \
  --release-image=quay.io/openshift-release-dev/ocp-release:4.17.9-multi
```

## Deploying a cluster from manifests

Instead of deploying directly with the `hcp` command, you can instead generate YAML manifests and then modify them before creating your cluster. You can generate the manifest by add the `--render` and `--render-sensitive` options to the `hcp create cluster agent` command; this will cause `hcp` to emit the manifests on _stdout_ instead of directly submitting them to OpenShift. The resulting manifest will contain several resources:

- A Namespace resource for the `cluster` namespace. You can discard this (the `cluster` namespace already exists).
- A Secret containing your pull secret.
- A Secret containing your ssh public key.
- A Secret containing the etcd encryption key.
- A Role granting access to the agent resources in the `--agent-namespace`. If you are acquiring nodes from the inventory in the `hardware-inventory` namespace you can discard  this Role (the necessary role already exists).
- A HostedCluster resource (this represents your cluster)
- A NodePool resource (this managed bare metal hardware for the cluster)

