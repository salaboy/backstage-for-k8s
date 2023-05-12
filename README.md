# [Backstage](https://backstage.io)

## Internal Developer Portal

In order to keep track of all of the new services, applications, documentation, resources, infrastructure, etc. that your teams will continue to create throughout the lifecycles of your intiatives - having an internal developer portal may be a very valuable addition to your platform.

To simply check out backstage and see what it has to offer, run the following commands:

```sh
yarn install
yarn dev
```

### Building a Backstage image for Kubernetes utilization


First we'll build the backstage image locally and make it available for the cluster

```
yarn --cwd backstage build:backend
yarn --cwd backstage build-image --tag backstage:1.0.0
kind load docker-image backstage:1.0.0 --name platform
```

Now let's deploy Backstage.io onto the kind cluster

```
kubectl create namespace backstage
kubectl create serviceaccount backstage -n backstage

kubectl apply -f backstage/kubernetes/rbac.yaml
kubectl apply -f backstage/kubernetes/backstage-secret.yaml
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
kubectl create configmap backstage-cm -n backstage --from-literal=ENDPOINT=$APISERVER

kubectl apply -f backstage/kubernetes/backstage-service.yaml
kubectl apply -f backstage/kubernetes/backstage.yaml
```

We'll port forward in order to get to the Backstage UI

```
kubectl port-forward --namespace=backstage svc/backstage 5434:80
```

Access http://localhost:5434/ on your browser and do some initial exploring.

Next we're going to integrate Backstage with resources on our Kubernetes cluster:

```
kubectl label pods --all -n dapr backstage.io/kubernetes-id=sample-app
kubectl label deployments --all -n dapr backstage.io/kubernetes-id=sample-app

kubectl label pods --all -n knative-serving backstage.io/kubernetes-id=demo-service
kubectl label deployments --all -n native-serving backstage.io/kubernetes-id=demo-service

kubectl label components --all --all-namespaces backstage.io/kubernetes-id=demo-service
kubectl label pods --all -n team-a-dev-env backstage.io/kubernetes-id=sample-app

kubectl patch deployment/dapr-dashboard \
  --namespace dapr-system \
  --type merge \
  --patch '{"spec": {"template": {"metadata": {"labels": {"backstage.io/kubernetes-id": "sample-app"}}}}}'
```

Look at your backstage components and be impressed!
