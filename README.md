# openwhisk-cloud-functions

How to set up openwhisk on MacOS + Docker + Kubernetes

```bash
foo@bar$ minikube start
foo@bar$ wget https://ftp.heanet.ie/mirrors/www.apache.org/dist/openwhisk/openwhisk-deploy-kube-1.0.0-sources.tar.gz
foo@bar$ tar -xzvf ./openwhisk-deploy-kube-1.0.0-sources.tar.gz
foo@bar$ cd openwhisk-deploy-kube-1.0.0/helm
foo@bar$ kubectl describe nodes | grep InternalIP
foo@bar$ vim ./openwhisk/values.yaml
  ingress:
    awsSSL: "false"
    apiHostName: "192.168.49.2" # update this
    apiHostPort: 31001
    apiHostProto: "https"
    type: NodePort
foo@bar$ helm package openwhisk
foo@bar$ helm install demo openwhisk-1.0.0.tgz 
```

or deploy with a cluster file

```bash
foo@bar$ helm repo add openwhisk https://openwhisk.apache.org/charts
foo@bar$ helm repo update
foo@bar$ foo@bar$ kubectl describe nodes | grep InternalIP
foo@bar$ mycluster.yaml # add your ip
foo@bar$ helm install demo openwhisk/openwhisk -n openwhisk --create-namespace -f mycluster.yaml
```

or alternatively with git

```bash
foo@bar$ git clone https://github.com/apache/openwhisk-deploy-kube.git
foo@bar$ cd openwhisk-deploy-kube
foo@bar$ mv ../mycluster.yaml .
foo@bar$ helm install demo ./helm/openwhisk -f mycluster.yaml
```

OR even easier

```bash
foo@bar$ helm repo add openwhisk https://openwhisk.apache.org/charts
foo@bar$ helm repo update
foo@bar$ API_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
foo@bar$ helm install demo openwhisk/openwhisk -f mycluster.yaml --set apiHostName=${API_IP}
```

Configure deployment and open networking

```bash
foo@bar$ kubectl label nodes --all openwhisk-role=invoker
foo@bar$ kubectl get all
foo@bar$ brew install wsk wskdeploy
foo@bar$ helm status demo
foo@bar$ minikube service demo-nginx --url
    http://127.0.0.1:51540
    http://127.0.0.1:51541
foo@bar$ wsk property set --apihost 127.0.0.1:51541
foo@bar$ wsk property set --auth 23bc46b1-71f6-4ed5-8c54-816aa4f8c502:123zO3xZCLrMN6v2BKK1dXYFpXlPkccOFqm12CdAsMgRU4VrNZ9lyGVCGuMDGIwP # default credential for guest account
```

Create your cloud function

```bash
foo@bar$ vim hello.py
```

with the contents

```python
def main(dict):
    if 'name' in dict:
        name = dict['name']
    else:
        name = "stranger"
    greeting = "Hello " + name + "!"
    print(greeting)
    return {"body":{"greeting": greeting}}
```

Deploy the cloud function

```bash
foo@bar$ wsk -i action create helloPy hello.py --web true --kind python:3
foo@bar$ wsk -i action list
foo@bar$ wsk -i -v action invoke helloPy --result --param name World
foo@bar$ wsk -i action get helloPy --url
foo@bar$ curl 'http://127.0.0.1:51540/api/v1/web/guest/default/helloPy' -H "Content-Type: application/json" -d '{"name":"World"}'
```

Cleanup

```bash
foo@bar$ wsk -i action delete helloPy
foo@bar$ helm delete demo
```
