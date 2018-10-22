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
>  kubectl get all -n istio-system
NAME                                          READY   STATUS      RESTARTS   AGE
pod/grafana-56947478f7-x2tqh                  1/1     Running     0          6h
pod/istio-citadel-8566d7c458-c5jk5            1/1     Running     0          6h
pod/istio-cleanup-secrets-4hjbl               0/1     Completed   0          6h
pod/istio-egressgateway-7bcdcf57b9-tcf8r      1/1     Running     0          6h
pod/istio-galley-677745f7d6-fkvh7             1/1     Running     0          6h
pod/istio-grafana-post-install-pzdq6          0/1     Completed   0          6h
pod/istio-ingressgateway-7f4c4c57f6-vkmkf     1/1     Running     0          6h
pod/istio-pilot-cf944f77d-86h6c               2/2     Running     0          6h
pod/istio-policy-787bb9599c-zxltf             2/2     Running     0          6h
pod/istio-security-post-install-jg5rj         0/1     Completed   0          6h
pod/istio-sidecar-injector-5f55468d6d-ghp5r   1/1     Running     0          6h
pod/istio-telemetry-6df7c97dcd-qflcs          2/2     Running     0          6h
pod/istio-tracing-6bc87d5c59-hsh46            1/1     Running     0          6h
pod/prometheus-544bf8c86-nm2bs                1/1     Running     0          6h
pod/servicegraph-6f9dc968f5-9dwhj             1/1     Running     0          6h

NAME                             TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                                                                                                                   AGE
service/grafana                  ClusterIP      10.0.1.190     <none>          3000/TCP                                                                                                                  6h
service/istio-citadel            ClusterIP      10.0.12.95     <none>          8060/TCP,9093/TCP                                                                                                         6h
service/istio-egressgateway      ClusterIP      10.0.117.150   <none>          80/TCP,443/TCP                                                                                                            6h
service/istio-galley             ClusterIP      10.0.78.30     <none>          443/TCP,9093/TCP,9901/TCP                                                                                                 6h
service/istio-ingressgateway     LoadBalancer   10.0.144.57    23.97.234.106   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15011:31777/TCP,8060:30336/TCP,853:31498/TCP,15030:31563/TCP,15031:32717/TCP   6h
service/istio-pilot              ClusterIP      10.0.7.193     <none>          15010/TCP,15011/TCP,8080/TCP,9093/TCP                                                                                     6h
service/istio-policy             ClusterIP      10.0.20.7      <none>          9091/TCP,15004/TCP,9093/TCP                                                                                               6h
service/istio-sidecar-injector   ClusterIP      10.0.187.218   <none>          443/TCP                                                                                                                   6h
service/istio-telemetry          ClusterIP      10.0.19.106    <none>          9091/TCP,15004/TCP,9093/TCP,42422/TCP                                                                                     6h
service/jaeger-agent             ClusterIP      None           <none>          5775/UDP,6831/UDP,6832/UDP                                                                                                6h
service/jaeger-collector         ClusterIP      10.0.195.232   <none>          14267/TCP,14268/TCP                                                                                                       6h
service/jaeger-query             ClusterIP      10.0.178.252   <none>          16686/TCP                                                                                                                 6h
service/prometheus               ClusterIP      10.0.16.120    <none>          9090/TCP                                                                                                                  6h
service/servicegraph             ClusterIP      10.0.255.35    <none>          8088/TCP                                                                                                                  6h
service/tracing                  ClusterIP      10.0.247.105   <none>          80/TCP                                                                                                                    6h
service/zipkin                   ClusterIP      10.0.226.124   <none>          9411/TCP                                                                                                                  6h

NAME                                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana                  1         1         1            1           6h
deployment.apps/istio-citadel            1         1         1            1           6h
deployment.apps/istio-egressgateway      1         1         1            1           6h
deployment.apps/istio-galley             1         1         1            1           6h
deployment.apps/istio-ingressgateway     1         1         1            1           6h
deployment.apps/istio-pilot              1         1         1            1           6h
deployment.apps/istio-policy             1         1         1            1           6h
deployment.apps/istio-sidecar-injector   1         1         1            1           6h
deployment.apps/istio-telemetry          1         1         1            1           6h
deployment.apps/istio-tracing            1         1         1            1           6h
deployment.apps/prometheus               1         1         1            1           6h
deployment.apps/servicegraph             1         1         1            1           6h

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-56947478f7                  1         1         1       6h
replicaset.apps/istio-citadel-8566d7c458            1         1         1       6h
replicaset.apps/istio-egressgateway-7bcdcf57b9      1         1         1       6h
replicaset.apps/istio-galley-677745f7d6             1         1         1       6h
replicaset.apps/istio-ingressgateway-7f4c4c57f6     1         1         1       6h
replicaset.apps/istio-pilot-cf944f77d               1         1         1       6h
replicaset.apps/istio-policy-787bb9599c             1         1         1       6h
replicaset.apps/istio-sidecar-injector-5f55468d6d   1         1         1       6h
replicaset.apps/istio-telemetry-6df7c97dcd          1         1         1       6h
replicaset.apps/istio-tracing-6bc87d5c59            1         1         1       6h
replicaset.apps/prometheus-544bf8c86                1         1         1       6h
replicaset.apps/servicegraph-6f9dc968f5             1         1         1       6h

NAME                                                       REFERENCE                         TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-egressgateway    Deployment/istio-egressgateway    20%/80%   1         5         1          6h
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   20%/80%   1         5         1          6h
horizontalpodautoscaler.autoscaling/istio-pilot            Deployment/istio-pilot            0%/80%    1         5         1          6h
horizontalpodautoscaler.autoscaling/istio-policy           Deployment/istio-policy           45%/80%   1         5         1          6h
horizontalpodautoscaler.autoscaling/istio-telemetry        Deployment/istio-telemetry        70%/80%   1         5         1          6h

NAME                                    DESIRED   SUCCESSFUL   AGE
job.batch/istio-cleanup-secrets         1         1            6h
job.batch/istio-grafana-post-install    1         1            6h
job.batch/istio-security-post-install   1         1            6h
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
