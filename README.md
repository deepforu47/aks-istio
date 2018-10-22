# Blue/Green deployment testing on AKS using istio (Here I already have AKS setup along with istio installed)
## Get Namespaces
```bash
> kubectl get ns
NAME           STATUS    AGE
azure-system   Active    103d
default        Active    103d
istio-system   Active    1m
kube-public    Active    103d
kube-system    Active    103d
```

## Get Everything running in istio-system namespace

```bash
> kubectl get all -n istio-system
NAME                            AGE
deploy/grafana                  1m
deploy/istio-citadel            1m
deploy/istio-egressgateway      1m
deploy/istio-galley             1m
deploy/istio-ingressgateway     1m
deploy/istio-pilot              1m
deploy/istio-policy             1m
deploy/istio-sidecar-injector   1m
deploy/istio-telemetry          1m
deploy/istio-tracing            1m
deploy/prometheus               1m
deploy/servicegraph             1m

NAME                                   AGE
rs/grafana-56947478f7                  1m
rs/istio-citadel-8566d7c458            1m
rs/istio-egressgateway-7bcdcf57b9      1m
rs/istio-galley-677745f7d6             1m
rs/istio-ingressgateway-7f4c4c57f6     1m
rs/istio-pilot-cf944f77d               1m
rs/istio-policy-787bb9599c             1m
rs/istio-sidecar-injector-5f55468d6d   1m
rs/istio-telemetry-6df7c97dcd          1m
rs/istio-tracing-6bc87d5c59            1m
rs/prometheus-544bf8c86                1m
rs/servicegraph-6f9dc968f5             1m

NAME                       REFERENCE                         TARGETS           MINPODS   MAXPODS   REPLICAS   AGE
hpa/istio-egressgateway    Deployment/istio-egressgateway    <unknown> / 80%   1         5         1          58s
hpa/istio-ingressgateway   Deployment/istio-ingressgateway   <unknown> / 80%   1         5         1          58s
hpa/istio-pilot            Deployment/istio-pilot            <unknown> / 80%   1         5         1          57s
hpa/istio-policy           Deployment/istio-policy           <unknown> / 80%   1         5         1          58s
hpa/istio-telemetry        Deployment/istio-telemetry        <unknown> / 80%   1         5         1          57s

NAME                               DESIRED   SUCCESSFUL   AGE
jobs/istio-cleanup-secrets         1         0            1m
jobs/istio-grafana-post-install    1         0            1m
jobs/istio-security-post-install   1         0            1m

NAME                                         READY     STATUS              RESTARTS   AGE
po/grafana-56947478f7-x2tqh                  0/1       ContainerCreating   0          1m
po/istio-citadel-8566d7c458-c5jk5            0/1       ContainerCreating   0          1m
po/istio-cleanup-secrets-4hjbl               0/1       ContainerCreating   0          1m
po/istio-egressgateway-7bcdcf57b9-tcf8r      0/1       ContainerCreating   0          1m
po/istio-galley-677745f7d6-fkvh7             0/1       ContainerCreating   0          1m
po/istio-grafana-post-install-pzdq6          0/1       ContainerCreating   0          1m
po/istio-ingressgateway-7f4c4c57f6-vkmkf     0/1       ContainerCreating   0          1m
po/istio-pilot-cf944f77d-86h6c               0/2       ContainerCreating   0          1m
po/istio-policy-787bb9599c-zxltf             0/2       ContainerCreating   0          1m
po/istio-security-post-install-jg5rj         0/1       ContainerCreating   0          1m
po/istio-sidecar-injector-5f55468d6d-ghp5r   0/1       ContainerCreating   0          1m
po/istio-telemetry-6df7c97dcd-qflcs          0/2       ContainerCreating   0          1m
po/istio-tracing-6bc87d5c59-hsh46            1/1       Running             0          1m
po/prometheus-544bf8c86-nm2bs                1/1       Running             0          1m
po/servicegraph-6f9dc968f5-9dwhj             1/1       Running             0          1m

NAME                         TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                                                                                                   AGE
svc/grafana                  ClusterIP      10.0.1.190     <none>          3000/TCP                                                                                                                  1m
svc/istio-citadel            ClusterIP      10.0.12.95     <none>          8060/TCP,9093/TCP                                                                                                         1m
svc/istio-egressgateway      ClusterIP      10.0.117.150   <none>          80/TCP,443/TCP                                                                                                            1m
svc/istio-galley             ClusterIP      10.0.78.30     <none>          443/TCP,9093/TCP,9901/TCP                                                                                                 1m
svc/istio-ingressgateway     LoadBalancer   10.0.144.57    23.97.234.106   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15011:31777/TCP,8060:30336/TCP,853:31498/TCP,15030:31563/TCP,15031:32717/TCP   1m
svc/istio-pilot              ClusterIP      10.0.7.193     <none>          15010/TCP,15011/TCP,8080/TCP,9093/TCP                                                                                     1m
svc/istio-policy             ClusterIP      10.0.20.7      <none>          9091/TCP,15004/TCP,9093/TCP                                                                                               1m
svc/istio-sidecar-injector   ClusterIP      10.0.187.218   <none>          443/TCP                                                                                                                   1m
svc/istio-telemetry          ClusterIP      10.0.19.106    <none>          9091/TCP,15004/TCP,9093/TCP,42422/TCP                                                                                     1m
svc/jaeger-agent             ClusterIP      None           <none>          5775/UDP,6831/UDP,6832/UDP                                                                                                57s
svc/jaeger-collector         ClusterIP      10.0.195.232   <none>          14267/TCP,14268/TCP                                                                                                       57s
svc/jaeger-query             ClusterIP      10.0.178.252   <none>          16686/TCP                                                                                                                 57s
svc/prometheus               ClusterIP      10.0.16.120    <none>          9090/TCP                                                                                                                  1m
svc/servicegraph             ClusterIP      10.0.255.35    <none>          8088/TCP                                                                                                                  1m
svc/tracing                  ClusterIP      10.0.247.105   <none>          80/TCP                                                                                                                    56s
svc/zipkin                   ClusterIP      10.0.226.124   <none>          9411/TCP                                                                                                                  56s
```

## Get Services running in istio-system(things which are necessary)
```bash
>kubectl get svc  -n=istio-system -
o=custom-columns=NAME:.metadata.name,IP:.spec.clusterIP,External-IP:.status.loadBalancer
NAME                     IP             External-IP
grafana                  10.0.1.190     map[]
istio-citadel            10.0.12.95     map[]
istio-egressgateway      10.0.117.150   map[]
istio-galley             10.0.78.30     map[]
istio-ingressgateway     10.0.144.57    map[ingress:[map[ip:23.97.234.106]]]
istio-pilot              10.0.7.193     map[]
istio-policy             10.0.20.7      map[]
istio-sidecar-injector   10.0.187.218   map[]
istio-telemetry          10.0.19.106    map[]
jaeger-agent             None           map[]
jaeger-collector         10.0.195.232   map[]
jaeger-query             10.0.178.252   map[]
prometheus               10.0.16.120    map[]
servicegraph             10.0.255.35    map[]
tracing                  10.0.247.105   map[]
zipkin                   10.0.226.124   map[]
```
## Deploy application with both version. 
```bash
> kubectl apply -f .\myapp.yaml
service "myapp" created
deployment "myapp-v1" created
deployment "myapp-v2" created
```

## Check if deployment and services are up and running
```bash
> kubectl get pod -l app=myapp
NAME                        READY     STATUS    RESTARTS   AGE
myapp-v1-584dc655f8-wjqbp   1/1       Running   0          1m
myapp-v2-566d7c75d-n9tbn    1/1       Running   0          1m

> kubectl get svc -l app=myapp
NAME      TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
myapp     LoadBalancer   10.0.174.48   13.93.33.62   80:31928/TCP   1m
```

## This is simple nginx based application, so you can do testing using proxy as below to check that both applications are working as expceted
```bash
>kubectl port-forward $(kubectl --namespace sftp get pod --no-headers=true -l app=myapp -l version=v1 -o=custom-columns=NAME:.metadata.name) 8080:80
>kubectl port-forward $(kubectl --namespace sftp get pod --no-headers=true -l app=myapp -l version=v2 -o=custom-columns=NAME:.metadata.name) 8081:80
```

## Create application gateway, which forward the incoming traffic to actual istio ingress created duing istio installation. We have defined routing login within gateway.yaml file. This will create gateway, destinationrule and virtualservice as shown below:
```bash
> kubectl apply -f .\app-gateway.yaml
gateway "app-gateway" created
destinationrule "myapp" created
virtualservice "myapp" created
```

## Now you can do testing using below curl trick as below. Intially you will see all traffic routing to Version 1.
```bash
$ while : ;do export GREP_COLOR='1;33';curl -s  http://23.97.234.106/  |  grep --color=always "Version 1" ; export GREP_COLOR='1;36'; curl -s  http://23.97.234.106/  | grepon 2" ; sleep 1; doneio
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
```

## You can change the weight in VirtualService and reploy this using below command.
```bash
> istioctl.exe replace -f .\app-gateway.yaml
Updated config gateway/sftp/app-gateway to revision 10276905
Updated config destination-rule/sftp/myapp to revision 10276906
Updated config virtual-service/sftp/myapp to revision 10286413
```

## After changing weight to 50-50, below will be the result.
```bash
$ while : ;do export GREP_COLOR='1;33';curl -s  http://23.97.234.106/  |  grep --color=always "Version 1" ; export GREP_COLOR='1;36'; curl -s  http://23.97.234.106/  | grepon 2" ; sleep 1; done
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 1 of the app!</h1>
```

## After final testing route all traffic to Version 2 and verify the results.
```bash
$ while : ;do export GREP_COLOR='1;33';curl -s  http://23.97.234.106/  |  grep --color=always "Version 1" ; export GREP_COLOR='1;36'; curl -s  http://23.97.234.106/  | grepon 2" ; sleep 1; done
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
<h1>Welcome to Version 2 of the app!</h1>
```
