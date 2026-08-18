# Storage Options

Pick one storage provisioning option and run only its playbooks. Choose based on
your needs:

| Option | Type | Replication | Dependencies |
|--------|------|-------------|--------------|
| [Local Path Provisioner](#option-1-local-path-provisioner) | hostPath / local | No | None |
| [Longhorn](#option-2-longhorn) | distributed block storage | Yes | iSCSI + NFS on every node |

## Option 1: Local Path Provisioner

Simple and lightweight. Provisions `hostPath`-based volumes from a directory on
each node (`/opt/local-path-provisioner`). No extra dependencies, works on a
single node.

```bash
ansible-playbook -i hosts.ini storage/local-path/install-local-path-provisioner.yml --ask-become-pass
```

Creates the `local-path` StorageClass (provisioner `rancher.io/local-path`,
`WaitForFirstConsumer`).

> Note: volumes are tied to the node they are provisioned on and are NOT
> replicated. Data is lost if that node dies.

## Option 2: Longhorn

Distributed, replicated block storage managed by Kubernetes. Recommended for
multi-node clusters.

```bash
# 1. Install dependencies on all nodes
ansible-playbook -i hosts.ini storage/longhorn/install-longhorn-deps.yml --ask-become-pass

# 2. Install Longhorn
ansible-playbook -i hosts.ini storage/longhorn/install-longhorn.yml --ask-become-pass
```

Creates the `longhorn` StorageClass.

## Make Your Choice the Default StorageClass

```bash
kubectl patch storageclass local-path -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
# or
kubectl patch storageclass longhorn -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Remove the Old Longhorn Playbooks

Longhorn was moved out of the root playbooks (`04a-install-longhorn-deps.yml`,
part of `05-install-apps.yml`) into this folder. If you had a previous install
that references those, run the storage playbooks from here instead.
