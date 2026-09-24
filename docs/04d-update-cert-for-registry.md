# update root cert for registry

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
docker restart airgap

# untuk Nexus OSS, perlu restart service nginx saja
systemctl restart nginx
```

- step 5: Update NKP machine secret

```bash


# trigger rolling rebuild
export NS=tw3-ntnx-karbon-u01-7ftrf-rv8qg
export KCP=tw3-ntnx-karbon-u01-tcxmq
export MD=tw3-ntnx-karbon-u01-md-0-254db


## rolling rebuild control plane
kubectl patch kubeadmcontrolplane -n "$NS" "$KCP" \
  --type=merge \
  --patch='{"spec":{"rolloutAfter":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'"}}'

## rolling rebuild woker
kubectl patch machinedeployment -n "$NS" "$MD" \
  --type=merge \
  --patch='{"spec":{"rolloutAfter":"'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'"}}'

```