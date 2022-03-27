kubectl create configmap licensingservice-config --from-file=licensingservice.yml
kubectl apply configmap licensingservice-config --from-file=licensingservice.yml
kubectl apply -f config1.yaml
kubectl delete -f config1.yaml

kubectl delete configmap licensingservice-config


SERVICE_ACCOUNT=sa1

# Get the ServiceAccount's token Secret's name
SECRET=$(kubectl get serviceaccount -n test sa1 -o json | jq -Mr '.secrets[].name | select(contains("token"))')
 
# Extract the Bearer token from the Secret and decode
TOKEN=$(kubectl get secret -n test ${SECRET} -o json | jq -Mr '.data.token' | base64 -d)
 
# Extract, decode and write the ca.crt to a temporary location
kubectl get secret -n test ${SECRET} -o json | jq -Mr '.data["ca.crt"]' | base64 -d > /tmp/ca.crt
 
# Get the API Server location
APISERVER=$(kubectl config view --minify | grep server | cut -f 2- -d ":" | tr -d " ")


curl --header "Authorization: Bearer $TOKEN" --insecure -s $APISERVER/api/v1/namespaces/test/pods/mytest-5494669fdf-9rqg4
 curl --header "Authorization: Bearer $TOKEN" --cacert /tmp/ca.crt -s $APISERVER/api/v1/namespaces/test/pods/test