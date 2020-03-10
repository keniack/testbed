# OpenFaas


How to run build and run customized version.

### Build customized openfaas-gateway 


```
 git clone https://github.com/openfaas/faas.git
 cd faas/gateway
 ./build.sh <version> <yourhubusername>
 ./build.sh 0.1 keniack
 ```
### Push Docker image

```
docker login --username=yourhubusername 
Enter your password

docker tag dfe8cfdbe783 keniack/custom-gateway:0.1
docker push keniack/custom-gateway
```

### Deploy customized imaged

```
kubectl apply -f opeenfaas/namespaces.yml
PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)	
kubectl -n openfaas create secret generic basic-auth --from-literal=basic-auth-user=admin --from-literal=basic-auth-password="$PASSWORD"	
kubectl apply -f opeenfaas/yaml_armhf
```

