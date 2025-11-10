# workshop-aro-workaround


git clone https://github.com/redhat-jcpeixoto/workshop-aro-workaround/

cd workshop-aro-workaround

helm repo index .

python3 -m http.server 8080 &


AZR_STORAGE_ACCOUNT_NAME="storage${GUID}"

AZR_STORAGE_KEY=$(az storage account keys list --resource-group "openenv-${GUID}" -n "${AZR_STORAGE_ACCOUNT_NAME}" --query "[0].value" -o tsv)

helm upgrade -n custom-logging  aro-clf-blob localrepo/aro-clf-blob   --version 0.1.4   -n custom-logging   --install   --set azure.storageAccount="${AZR_STORAGE_ACCOUNT_NAME}"   --set azure.storageAccountKey="${AZR_STORAGE_KEY}"   --set azure.storageContainer="aro-logs"

while ! oc get grafana; do sleep 5; echo -n .; done 
while ! oc get ClusterLogForwarders; do sleep 5; echo -n .; done 
while ! oc get lokistack; do sleep 5; echo -n .; done 
while ! oc get resourcelocker; do sleep 5; echo -n .; done
while  ! oc get UIPlugins; do sleep 5; echo -n .; done

helm upgrade -n "custom-logging" aro-thanos-af --install localrepo/aro-thanos-af --version 0.7.1 --set "aro.storageAccount=${AZR_STORAGE_ACCOUNT_NAME}"  --set "aro.storageAccountKey=${AZR_STORAGE_KEY}"  --set "aro.storageContainer=aro-metrics"  --set "enableUserWorkloadMetrics=true"

oc -n custom-logging rollout status deploy aro-thanos-af-grafana-cr-deployment

oc -n custom-logging get route aro-thanos-af-grafana-cr-route -o jsonpath='{"https://"}{.spec.host}{"\n"}'

STORAGE_CLASS=$(oc get storageclass -o=jsonpath='{.items[?(@.metadata.annotations.storageclass\.kubernetes\.io/is-default-class=="true")].metadata.name}') echo ${STORAGE_CLASS}

helm upgrade -n custom-logging  aro-clf-blob localrepo/aro-clf-blob   --version 0.1.4   -n custom-logging   --install   --set azure.storageAccount="${AZR_STORAGE_ACCOUNT_NAME}"   --set azure.storageAccountKey="${AZR_STORAGE_KEY}"   --set azure.storageContainer="aro-logs"



 
