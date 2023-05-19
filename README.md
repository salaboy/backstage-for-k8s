# [Backstage](https://backstage.io)

## Internal Developer Portal

In order to keep track of all of the new services, applications, documentation, resources, infrastructure, etc. that your teams will continue to create throughout the lifecycles of your intiatives - having an internal developer portal may be a very valuable addition to your platform

To simply check out backstage and see what it has to offer, run the following commands:

```sh
yarn install
yarn dev
```

### Building a Backstage image for Kind Cluster


First we'll build the backstage image locally and make it available for the cluster

```
yarn build:backend
yarn build-image --tag backstage:1.0.0
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

Enjoy!