az postgres flexible-server create   --resource-group openenv-$GUID   --location eastus   --name microsweeper-$GUID   --admin-user myAdmin   --admin-password 'M1cr0Sw33p3r!'   --sku-name Standard_D2s_v3   --storage-size 64   --public-access all


psql \
  "host=microsweeper-$GUID.postgres.database.azure.com port=5432
  dbname=postgres
  user=myAdmin password=M1cr0Sw33p3r! sslmode=require" \
  -c "select now();"


cat << EOF | oc apply -f -
---
apiVersion: v1
kind: Secret
metadata:
  name: microsweeper-secret
  namespace: microsweeper-ex
type: Opaque
stringData:
  PG_URL: jdbc:postgresql://microsweeper-$GUID.postgres.database.azure.com:5432/postgres
  PG_USER: myAdmin
  PG_PASS: "M1cr0Sw33p3r!"
EOF
