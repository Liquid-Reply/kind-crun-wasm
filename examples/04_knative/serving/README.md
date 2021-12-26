# Scale to zero with Knative Serving

> ⚠️ WARNING: This example uses the experimental KinD node image

This example is just the default knative getting started guide with a modified KinD and a WASM container as `HelloWorld` example.

## Create KinD cluster
```bash
# Set up KinD cluster with WebAssembly support
kind create cluster --image ghcr.io/liquid-reply/kind-crun-wasm:experimental
```

## Install knative serving
To install serving we cant use the quickstart plugin since it does not work with custom KinD images but using the [install guide](https://knative.dev/docs/install/serving/install-serving-with-yaml/) with an existing KinD cluster works fine.

```bash
# Install CRDs
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.1.0/serving-crds.yaml
# Install serving
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.1.0/serving-core.yaml
# Installing kourier
kubectl apply -f https://github.com/knative/net-kourier/releases/download/knative-v1.1.0/kourier.yaml
# Patch ingress class
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'
# Patch kourier service
kubectl patch svc/kourier \
  --namespace kourier-system \
  --patch '{"spec":{"type":"NodePort"}}'
```

## Run wasm-echo service with knative serving
```bash
# Create wasm-echo service
kn service create wasm-echo \
    --image avengermojo/http_server:with-wasm-annotation \
    --port 1234 \
    --env run.oci.handler=wasm \
    --revision-name=wasm-echo
# Forward 
kubectl port-forward -n kourier-system svc/kourier 8080:80 &
curl -H "Host: wasm-echo.default.example.com" http://localhost:8080 -d data=payload

# Output:
#    echo: data=payload
```