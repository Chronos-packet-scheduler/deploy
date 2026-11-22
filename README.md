# deploy

Configuration to run the whole Chronos system from the images published to `ghcr.io/chronos-packet-scheduler`. Works on [Rancher Desktop](https://rancherdesktop.io) (container engine: `dockerd (moby)`, Kubernetes enabled).

# Universal (other than Rancher Desktop)

Create k3d cluster:

```
k3d cluster create CHRONOS --servers 4 --agents 6
```

## Registry access

If the packages are private, log in once. Requires personal access token (classic) with the `read:packages` scope

```sh
export GITHUB_TOKEN=ghp_...    # your token
echo "$GITHUB_TOKEN" | docker login ghcr.io -u <github-user> --password-stdin
```

## Docker Compose

```sh
docker compose pull
docker compose up -d                           # frontend on http://localhost:3000
docker compose --profile data-plane up -d      # also run the data plane (privileged, host network)
docker compose down
```

## Kubernetes

```sh
kubectl -n chronos create secret docker-registry ghcr \
  --docker-server=ghcr.io --docker-username=<github-user> --docker-password="$GITHUB_TOKEN"
kubectl apply -k k8s/
kubectl -n chronos get pods
kubectl -n chronos port-forward svc/intent-engine-frontend 3000:3000
```

To pick up newly published `latest` images:

```
kubectl -n chronos rollout restart deploy,ds
```
