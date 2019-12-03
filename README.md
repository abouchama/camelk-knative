# Prerequisties

```
export CAMELK=`pwd`
oc apply -f "$CAMELK/install/redhat-operators-csc.yaml"
```

```
oc -n openshift-marketplace get csc
```

```
export CAMELK=`pwd`
watch -n1 'oc get csv -n openshift-operators -ocustom-columns-file=$CAMELK/install/csv-columns.txt'
```


wait until server-less-operator will be installed
```
export CAMELK=`pwd`
watch -n1 'oc get csv -n knative-serving -ocustom-columns-file=$CAMELK/install/csv-columns.txt'
```

## Install Serverless
Subscribe to Serverless via OperatorHub

```
export CAMELK=`pwd`
oc apply -f "$CAMELK/install/knative-serving/subscription.yaml"
```

```
oc get csv -n openshift-operators
NAME                                        VERSION              PHASE
elasticsearch-operator.4.2.8-201911190952   4.2.8-201911190952   Succeeded
jaeger-operator.v1.13.1                     1.13.1               Succeeded
kiali-operator.v1.0.7                       1.0.7                Succeeded
serverless-operator.v1.2.0                  1.2.0                Succeeded
servicemeshoperator.v1.0.2                  1.0.2                Succeeded
```

```
oc adm new-project knative-serving && \
oc adm new-project knative-eventing && \
oc adm new-project camelkknativetutorial
```

```
oc get csv -n knative-serving

NAME                                        VERSION              PHASE
elasticsearch-operator.4.2.8-201911190952   4.2.8-201911190952   Succeeded
jaeger-operator.v1.13.1                     1.13.1               Succeeded
kiali-operator.v1.0.7                       1.0.7                Succeeded
serverless-operator.v1.2.0                  1.2.0                Succeeded
servicemeshoperator.v1.0.2                  1.0.2                Succeeded
```

```
export CAMELK=`pwd`
oc apply -f "$CAMELK/install/knative-serving/ks-cr.yaml"
```

```
watch -n1 'oc get pods -n knative-serving-ingress'
NAME                                    READY   STATUS    RESTARTS   AGE
cluster-local-gateway-c88b87557-kg8j6   1/1     Running   0          76s
istio-citadel-74b8cb775b-c8gwf          1/1     Running   0          2m33s
istio-galley-7669c98b7c-lh8rt           1/1     Running   0          2m8s
istio-ingressgateway-55655d8c79-hqvqf   1/1     Running   0          76s
istio-pilot-54c6f64549-sdw5p            1/1     Running   0          103s
```

```
watch -n1 'oc get pods -n knative-serving'
NAME                                READY   STATUS    RESTARTS   AGE
activator-dfb5b7b67-bnlf5           1/1     Running   0          9m13s
autoscaler-85bb4898c5-w2skb         1/1     Running   0          9m12s
autoscaler-hpa-865b6d49b7-nfgkc     1/1     Running   0          9m12s
controller-65c8dd48d6-mhfd5         1/1     Running   0          9m7s
networking-istio-7c9fb7dd4c-8rklc   1/1     Running   0          9m7s
webhook-95969d4fc-lx9hh             1/1     Running   0          9m6s
```

```
oc get smmr --all-namespaces
NAMESPACE                 NAME      MEMBERS
knative-serving-ingress   default   [knative-serving]

oc edit smmr -n knative-serving-ingress
servicemeshmemberroll.maistra.io/default edited

oc get smmr --all-namespaces
NAMESPACE                 NAME      MEMBERS
knative-serving-ingress   default   [knative-serving camelkknativetutorial]
```
##. Install CamelK
```
oc project camelkknativetutorial
kamel install
```

## Deploy CamelK CustomerAPI example:
```
git clone https://github.com/abouchama/CamelK-customerAPI

cd CamelK-customerAPI

kamel run customer-api.xml \
    --open-api customer-api.json \
    --name customers \
    --dependency camel-undertow \
    --dependency camel-rest \
    --logging-level org.apache.camel.k=DEBUG \
    --property camel.rest.port=8080 \
    -t affinity.pod-anti-affinity-labels="camel.apache.org/integration" \
    --env CAMEL_LOG_MSG=" ** Camelk ** This request is handled by this POD: {{env:HOSTNAME}}" \
    --env CAMEL_GET_SETBODY=" (Version 1) --> Enjoy the camelk Knative demo :-) | POD : {{env:HOSTNAME}} \n" \
    --env CAMEL_CREATE_SETBODY=" (Version 1) --> Enjoy the camelk Knative demo :-) | POD : {{env:HOSTNAME}} \n"
    
```