# DevOps with Kubernetes 2026: configuration of the todo project

The Kubernetes configuration of the [todo project](https://github.com/priku/k8s-course/tree/main/project), kept apart from its [code](https://github.com/priku/k8s-course) (exercise 4.10). [Argo CD](https://argo-cd.readthedocs.io/) applies this repository to the cluster, so what is on `main` here is what runs.

| Change | How it gets to the cluster |
|---|---|
| New version of the code | The workflows of the code repository build the images and commit their tags to `overlays/` here |
| Anything else about the running system | A commit to this repository |

## Layout

```
apps/<app>/            what is the same in every cluster, one directory per application
base/                  all the applications and NATS
components/gateway/    Gateway API routing in AKS
components/db-backup/  the daily database backup, production only
overlays/production/   AKS, namespace project; every tag of the code repository
overlays/staging/      AKS, namespace staging; every commit to main of the code repository
overlays/k3d/          a local cluster
argocd/                the Argo CD Applications
```

Staging differs from production: its broadcaster only logs the messages and its database is not backed up.

A change in `apps/`, `base/` or `components/` reaches both environments at once. To try one in staging first, put it in `overlays/staging/` and move it to the base afterwards.

## Set up

The Applications and the database password are applied once by hand. The password is encrypted with [SOPS](https://github.com/getsops/sops) and stays outside Argo CD, since Argo CD cannot decrypt it here.

```bash
kubectl create namespace staging
sops --decrypt apps/todo-backend/secrets/postgres-secret.enc.yaml | kubectl apply -f -
sops --decrypt apps/todo-backend/secrets/postgres-secret.enc.yaml | sed 's/namespace: project/namespace: staging/' | kubectl apply -f -
kubectl apply -f argocd/
kubectl get application -n argocd
```

A local cluster has no Argo CD: `kubectl apply -k overlays/k3d`.
