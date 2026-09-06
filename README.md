# ecommerce-gitops

Dépôt de **configuration de déploiement** (source unique de vérité GitOps) — synchronisé automatiquement dans le cluster par ArgoCD.

> Ne contient **aucun code applicatif**. Le code source vit dans [`ecommerce-app`](../ecommerce-app), le provisioning du cluster dans [`ecommerce-infra`](../ecommerce-infra).

## Structure

```
ecommerce-gitops/
├── apps/                        # 1 chart Helm par microservice
│   ├── catalog-service/
│   │   ├── Chart.yaml
│   │   ├── values.yaml          # tag d'image mis à jour automatiquement par la CI
│   │   └── templates/
│   ├── cart-service/
│   ├── order-service/
│   ├── user-service/
│   ├── notification-service/
│   ├── frontend/
│   └── api-gateway/
├── argocd/                      # manifestes Application ArgoCD (1 par service)
│   └── catalog-service-app.yaml
└── platform/                    # briques transverses (phase 6/7)
    ├── prometheus-grafana/
    ├── loki/
    └── sealed-secrets/
```

## Principe

1. La CI de `ecommerce-app` build une image, la scanne (Trivy), la pousse sur le registry.
2. Elle met à jour automatiquement `apps/<service>/values.yaml` (champ `image.tag`) dans **ce** dépôt.
3. ArgoCD détecte le changement et synchronise le cluster (`selfHeal: true`, `prune: true`).

Aucune action manuelle n'est nécessaire entre le commit sur `ecommerce-app` et le déploiement effectif.

## Ordre de mise en place (cf. cahier des charges)

- **Phase 4** : `apps/*` déployés manuellement (`helm install` / `kubectl apply`) pour valider que ça tourne.
- **Phase 5** : installation d'ArgoCD + manifestes dans `argocd/` → bascule en synchronisation automatique.
- **Phase 6** : `platform/prometheus-grafana/` et `platform/loki/`.
- **Phase 7** : `platform/sealed-secrets/`.

## Seul un service est pour l'instant complet (`catalog-service`)

Il sert de **modèle** — une fois validé, dupliquer sa structure (`Chart.yaml`, `values.yaml`, `templates/`) pour les 6 autres services.
