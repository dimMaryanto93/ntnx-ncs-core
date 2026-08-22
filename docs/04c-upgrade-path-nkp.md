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
```