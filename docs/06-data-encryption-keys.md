# Generating the Data Encryption Config and Key

## The Encryption Key:

Generate an encryption key:

```
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## Manually Create The Encryption Config File:

View you encruption key. Copy it for the next step:

```
echo $ENCRYPTION_KEY
```

Create the `encryption-config.yaml` encryption config file:

```
vim encryption-config.yaml
```

```
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY} #manually copy and paste the encryption key
      - identity: {}
```
