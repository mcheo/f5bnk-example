# f5bnk



1. Create namespace
```
kubectl create ns tenant1
kubectl create ns tenant2
```

2. Deploy vLLM and NGINX apps into both namespace
```
kubectl apply -f tenant1-vllm-app.yaml -n tenant1
kubectl apply -f tenant2-vllm-app.yaml -n tenant2

kubectl apply -f nginx-app -n tenant1
kubectl apply -f nginx-app -n tenant2
```

3. Deploy Gateway resource for these apps
```
kubectl apply -f tenant1-vllm-gateway.yaml -n tenant1
kubectl apply -f tenant2-vllm-gateway.yaml -n tenant2

kubectl apply -f tenant1-nginx-gateway.yaml -n tenant1
kubectl apply -f tenant2-nginx-gateway.yaml -n tenant2
```

4. Test access to these application from outside cluster
```
# Access tenant1 vLLM, the model return should be "TheBloke/TinyLlama"
curl http://10.176.12.201/v1/models

# Access tenant1 NGINX, pay attention to the source IP address. It should be BNK's internal self IP (10.176.13.11)
curl http://10.176.12.200

# Access tenant2 vLLM, the model return should be "Qwen/Qwen2.5"
curl http://10.176.12.202/v1/models

# Access tenant2 NGINX, pay attention to the source IP address. It should be BNK's internal self IP (10.176.13.11)
curl http://10.176.12.203
```

5. Test firewall policy rule
```
kubectl apply -f tenant1-fwpolicy-ingress.yaml -n tenant1

# My test client's IP is 10.176.10.20

curl http://10.176.12.200 ---> this will be dropped and no result return
```


6. Test cross name space restriction
```
# We only allow ns f5-bnk (TMM) and tenant2 traffic to access pods in tenant2
kubectl apply -f east-west-isolation.yaml
```
In tenant1, ssh into netshoot container of NGINX pod, curl any service of tenant2. This will be blocked.

However, if we curl to Tenant2 vLLM or NGINX's Gateway, this traffic will go through. f5-bnk with appropriate firewall policy can be the control point for multi tenancy traffic


7. Test egress control
```
kubectl apply -f snat-tenant.yaml
kubectl apply -f egress.yaml
kubectl apply -f ipify-route.yaml

#In tenant2, ssh into netshoot container of NGINX pod, and access ipify.org api
curl -H "Host: api.ipify.org"  http://104.26.13.205
```


