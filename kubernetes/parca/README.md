# Parca

You probably want to open this up in the Devcontainer environment.

```bash
git clone https://github.com/parca-dev/parca.git
cd parca
make build
```

Then running `./bin/parca` works locally, but I don't understand what that means for kubernetes.
Let's try it.

```bash
GOOGLE_PROJECT=myproject
gcloud container clusters create test-cluster \
    --threads-per-core=1 \
    --placement-type=COMPACT \
    --num-nodes=2 \
    --no-enable-autorepair \
    --no-enable-autoupgrade \
    --region=us-central1-a \
    --project=${GOOGLE_PROJECT} \
    --machine-type=c2d-standard-8
```

Install parca

```
kubectl create namespace parca
kubectl apply -f https://github.com/parca-dev/parca/releases/download/v0.21.0/kubernetes-manifest.yaml
```

And the daemonset:

```bash
kubectl apply -f https://github.com/parca-dev/parca-agent/releases/download/v0.30.0/kubernetes-manifest.yaml
```

Install the flux operator:

```
kubectl apply -f https://raw.githubusercontent.com/flux-framework/flux-operator/main/examples/dist/flux-operator.yaml
```

Create the minicluster.

```bash
```
