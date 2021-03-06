```
./setup.sh

kubectl apply -f common.yaml
kubectl apply -f deployment1.yaml
```


```
HOST=$(kubectl get node istio-control-plane -o json | jq '.status.addresses[0].address' -r)
PORT=$(kubectl -n opa get service opa-httpbin1 -o json | jq -r '.spec.ports[0].nodePort' )
ENDPOINT="http://$HOST:$PORT"
```

```
# 403
curl -v $ENDPOINT/headers

# 200
curl -v $ENDPOINT/ip

TOKEN=$(python3 gen-jwt.py key.pem --expire 10 --path /)
curl -v --header "Authorization: Bearer $TOKEN" $ENDPOINT/headers 

TOKEN=$(python3 gen-jwt.py key.pem --expire 10 --path / --role SuperSayan)
curl -v --header "Authorization: Bearer $TOKEN" $ENDPOINT/headers 

# 30 s gracetime on token
TOKEN=$(python3 gen-jwt.py key.pem --expire 10 --path /)
while true
do
date; curl --header "Authorization: Bearer $TOKEN" $ENDPOINT -s -o /dev/null -w "%{http_code}\n"
sleep 1
done
```

```
CLAIM BASED ROUTING ... this should really use the ingress gateway :)


HOST=$(kubectl get node istio-control-plane -o json | jq '.status.addresses[0].address' -r)
PORT=$(kubectl -n opa get service lb-httpbin2 -o json | jq -r '.spec.ports[0].nodePort' )
ENDPOINT="http://$HOST:$PORT"

kubectl apply -f deployment2.yaml

#403
curl -v $ENDPOINT/

# Always hit v1
TOKEN=$(python3 gen-jwt.py key.pem --expire 600)
curl --header "Authorization: Bearer $TOKEN" $ENDPOINT/

# Always hit v2
TOKEN=$(python3 gen-jwt.py key.pem --expire 600 --role "version2")
curl --header "Authorization: Bearer $TOKEN" $ENDPOINT/



```
