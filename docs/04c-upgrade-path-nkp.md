## Upgrade Roadmap NKP

Untuk upgrade NKP Kommander

| from NKP Version  | Prism Central | AOS Version   | to NKP version  | 
| :---              | :---          | :---          | :----           |
| 2.13.2            | pc.2024.2     | 6.10          | 2.14.3          |
| 2.14.3            | pc.2024.3     | 7.x           | 2.15.3          |
| 2.15.3            | pc.2024.3     | 7.x           | 2.16.1          |
| 2.16.1            | pc.           |               | 2.16.1          |


## Update expired certificate for registry

- Step 1: Extract a CSR from the expired certificateRun this command to create a request file (cert.csr) using your old certificate and private key (`cert.key`):

```bash
sudo openssl x509 -x509toreq -in domain.crt -signkey domain.key -out domain.csr
```

- Step 2: Generate the new renewed certificateSign the new CSR with your private key and set the new life span in days (e.g., 365 days):

```bash
sudo openssl x509 -req -days 365 -in domain.csr -signkey domain.key -out domain_new.crt && \
sudo mv domain.crt domain_old.crt && \
sudo mv domain_new.crt domain.crt
```

- Step 3: Check new expiration certificate
```bash
openssl x509 -enddate -noout -in domain.crt
```

- Step 4: Recreate/recreate registry to update
```bash
# untuk airgap (openregistry) running on docker, perlu destroy dan buat ulang


# untuk Nexus OSS, perlu restart service nginx saja
systemctl restart nginx
```

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
nkp upgrade workspace ${WORKSPACE_NAME}
```