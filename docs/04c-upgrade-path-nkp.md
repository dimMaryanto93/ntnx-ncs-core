## Upgrade Roadmap NKP

Untuk upgrade NKP Kommander

| from NKP Version  | Prism Central | AOS Version   | to NKP version  | 
| :---              | :---          | :---          | :----           |
| 2.13.2            | pc.2024.2     | 6.10          | 2.14.3          |
| 2.14.3            | pc.2024.3     | 7.x           | 2.15.3          |
| 2.15.3            | pc.2024.3     | 7.x           | 2.16.1          |
| 2.16.1            | pc.           |               | 2.16.1          |

## Download Upgrade NKP Airgap Bundle

- Step 1: Download NKP Airgap Bundle from Nutanix Support Portal 
- Step 2: Extract a tar file 

```bash
tar -zxf nkp-bundle_v2.16.1_linux_amd64.tar.gz && \
cd nkp-v2.16.1

## Upload binary container image into container registry
export MIRROR_REGISTRY_URL='https://airgap.nutanix.local:5000'
export MIRROR_REGISTRY_USERNAME='admin'
export MIRROR_REGISTRY_PASSWORD='nutanix/4u'
export MIRROR_REGISTRY_CACERT='/etc/docker/certs.d/airgap.nutanix.local:5000/registry.crt'

cli/nkp push bundle --bundle ./container-images/konvoy-image-bundle*.tar --to-registry=${MIRROR_REGISTRY_URL} --to-registry-username=${MIRROR_REGISTRY_USERNAME} --to-registry-password=${MIRROR_REGISTRY_PASSWORD} --to-registry-ca-cert-file=${MIRROR_REGISTRY_CACERT} && \ 
cli/nkp push bundle --bundle ./container-images/kommander-image-bundle*.tar --to-registry=${MIRROR_REGISTRY_URL} --to-registry-username=${MIRROR_REGISTRY_USERNAME} --to-registry-password=${MIRROR_REGISTRY_PASSWORD} --to-registry-ca-cert-file=${MIRROR_REGISTRY_CACERT}
```

- Step 3: Load image into docker image

```bash
docker load -i konvoy-bootstrap-image* && \
docker load -i nkp-image-builder-image*
```

- Step 4: Upgrade kommander

```bash
nkp-v2.16.1]$ cli/nkp upgrade kommander --kommander-applications-repository ./application-repositories/kommander-applications-v2.16.1.tar.gz --disable-appdeployments ai-navigator-app
```

- Step 5: Upgrade all workspaces in kommander cluster

```bash
nkp get workspaces

## put one workspace into this `workspace_name`
cli/nkp upgrade workspace ${WORKSPACE_NAME}
```

## Upgrade Managed NKP nodes

- Step 1: Get all cluster in kommander 

```bash
kubectl get cluster -A
```

Step 2: Upgrade kommander cluster image vm

```bash
export MANAGEMENT_CLUSTER_NAME='nkp-kommander-hpoc1030'
export VM_IMAGE_NAME='nkp-rocky-9.6-release-cis-1.33.5-20251108010758.qcow2'

cli/nkp upgrade cluster nutanix \
--cluster-name ${MANAGEMENT_CLUSTER_NAME} \
--vm-image ${VM_IMAGE_NAME}
```

Step 3: Upgrade managed cluster image vm

```bash
export WORKLOAD_CLUSTER_NAME='nkp-trial-devsecops'
export VM_IMAGE_NAME='nkp-rocky-9.6-release-cis-1.33.5-20251108010758.qcow2'
export WORKLOAD_CLUSTER_NAMESPACE='nkp-trial-devsecops-b5gsx'

cli/nkp upgrade cluster nutanix \
--cluster-name ${WORKLOAD_CLUSTER_NAME} \
--vm-image ${VM_IMAGE_NAME} -n ${WORKLOAD_CLUSTER_NAMESPACE}
```