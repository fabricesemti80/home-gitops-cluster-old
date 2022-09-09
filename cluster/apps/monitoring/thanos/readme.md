# Notes

## Prepare bucket - subject of Minio present

`mc mb minio/thanos`

## Create config file

```yaml
---
# yamllint disable
apiVersion: v1
kind: Secret
metadata:
  name: thanos-objstore
  namespace: monitoring
stringData:
  objstore.yml: |
    type: S3
    config:
      bucket: "thanos"
      endpoint: ""
      region: ""
      access_key: ""
      secret_key: ""
```

[ref](https://thanos.io/tip/thanos/storage.md/)
