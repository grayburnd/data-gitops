## Data GitOps

This repository defines production data services for the voting platform. ArgoCD deploys the PostgreSQL Helm chart to `data-prod` and manages Redis replication dependencies through the repository's third-party application configuration.

## PostgreSQL

The PostgreSQL chart uses the Zalando PostgreSQL operator and currently declares:

| Setting | Production value |
|---------|------------------|
| PostgreSQL version | `15` |
| Instances | `2` |
| Volume | `20Gi` using `ebs-gp3` |
| Resource requests | `100m` CPU and `100Mi` memory |
| Resource limits | `500m` CPU and `500Mi` memory |
| Connection pooler | Disabled |

The database is scheduled to the `data` team capacity. The chart creates the `db_admin` user and the `postgres` database. The generated credentials are pushed to AWS Secrets Manager by an External Secrets `PushSecret` using the `${CLUSTER_SECRET_STORE_NAME}` ClusterSecretStore. Workloads consume the resulting credential through the platform secret integration.

## Redis Dependencies

Redis replication is maintained under `third-party-apps/redis-replication` and is bootstrapped through the data ApplicationSets. The voting frontend and worker use Redis Sentinel for the shared vote queue and failover-aware connections. Redis credentials are consumed by the application GitOps repositories rather than defined in those application charts.

## Validation and Reconciliation

Pull requests run [`.github/workflows/ci-lint.yml`](.github/workflows/ci-lint.yml). It runs Gitleaks, Helm lint and Kubeconform validation for the `apps` and `bootstrap` directories:

```bash
helm lint -f=prod-values.yml apps/<application>
helm template apps/<application> -f prod-values.yml | kubeconform -ignore-missing-schemas -strict
```

After a change is merged, ArgoCD watches the `main` branch and reconciles the `data-prod` application. See the [umbrella GitOps guide](https://github.com/YOUR_GITHUB_ORG/aws-eks-gitops-argocd-terraform/blob/main/GitOps/README.md), the [backend GitOps repository](https://github.com/YOUR_GITHUB_ORG/backend-gitops), the [platform GitOps repository](https://github.com/YOUR_GITHUB_ORG/platform-gitops) and the [frontend GitOps repository](https://github.com/YOUR_GITHUB_ORG/frontend-gitops) for the adjacent parts of the delivery path.
