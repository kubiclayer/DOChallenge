# DOChallenge
DigitalOcean Kubernetes Challenge


# Create policies for mandatory labels for every deployment

## Policy
Based on the [Kyverno restrict_image_registries policy](https://kyverno.io/policies/best-practices/require_labels/require_labels/), a new "policies/require_labels.yaml" has been created to only allow the POD with mandatory labels "app.kubernetes.io/name":

## Testing Invalid Case
The policy has been tested with "manifests/invalid/invalid-no-label.yaml" manifest.

```
❯ kubectl apply -f manifests/invalid/invalid-no-label.yaml
Error from server: error when creating "manifests/invalid/invalid-no-label.yaml": admission webhook "validate.kyverno.svc-fail" denied the request:

resource Deployment/default/nginx-no-label was blocked due to the following policies

require-labels:
  check-for-labels: 'validation error: The label `app.kubernetes.io/name` is required.
    Rule check-for-labels failed at path /metadata/labels/'
```

# Create policies for image download only permitted from DOCR

## Policy
Based on the [Kyverno restrict_image_registries policy](https://kyverno.io/policies/best-practices/restrict_image_registries/restrict_image_registries/), a new "policies/restrict_image_registries.yaml" has been created to only allow the following DigitalOcean Container Registry:

```
registry.digitalocean.com/kubiclayer
```

Next, nginx:alpine container image has been pulled from Docker Hub and pushed to the DOCR to be used for this challenge
```
doctl registry login
docker pull nginx:alpine
docker tag nginx:alpine registry.digitalocean.com/kubiclayer/nginx:alpine
docker push registry.digitalocean.com/kubiclayer/nginx:alpine
```

Then, the policy has been tested with "manifests/invalid/invalid-with-public-image.yaml" manifest.

## Testing Invalid Case

```
❯ kubectl apply -f manifests/invalid/invalid-with-public-image.yaml
Error from server: error when creating "manifests/invalid/invalid-with-public-image.yaml": admission webhook "validate.kyverno.svc-fail" denied the request:

resource Deployment/default/nginx-public-image was blocked due to the following policies

restrict-image-registries:
  autogen-validate-registries: 'validation error: Unknown image registry. Rule autogen-validate-registries
    failed at path /spec/template/spec/containers/0/image/'
```

###