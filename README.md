# epf-app — base commune du Projet CI/CD

![CI](https://github.com/<organisation>/<depot>/actions/workflows/ci.yml/badge.svg)

Point de départ de toutes les équipes : une petite API Express, sa chaîne d'intégration
continue et sa publication d'image. Vous y ajouterez, séance après séance, le monitoring,
Terraform, Ansible et vos rapports.

> Remplacez `<organisation>/<depot>` dans le badge ci-dessus dès la création de votre dépôt.

## Ce que le dépôt contient déjà

| Élément | Où | Rôle |
|---|---|---|
| Application | `src/`, `test/` | API Express, 2 fichiers de tests (`node --test`), ESLint |
| Image | `Dockerfile` | Multi-étapes, utilisateur non root, `HEALTHCHECK` sur `/health` |
| CI | `.github/workflows/ci.yml` | À chaque push sur `main` et chaque PR : lint, tests, build, détection de secrets (gitleaks), dépendances (`npm audit`), analyse statique (Semgrep) |
| Release | `.github/workflows/release.yml` | Sur un tag `v*` : build de l'image, scan Trivy, test de démarrage, publication sur GHCR (`vX.Y.Z` et `stable`), release GitHub |
| Mises à jour | `.github/dependabot.yml` | PR hebdomadaires pour npm et les actions |

## Ce que vous ajouterez

```
monitoring/        séance 2 — Prometheus, Grafana, Alertmanager
infra/terraform/   séance 3 — infrastructure décrite en code
infra/ansible/     séance 4 — déploiement automatisé
docs/              vos cinq livrables (01 à 05) et docs/img/ pour les captures
```

Les kits fournis en séance supposent que l'application reste **à la racine** du dépôt.

## Lancer

```bash
npm ci
npm run dev        # http://localhost:3000
```

| Route | Description |
|---|---|
| `GET /` | Message de bienvenue |
| `GET /health` | Statut, version (`APP_VERSION`), environnement (`APP_ENV`), uptime |
| `GET /metrics` | Métriques Prometheus (`http_requests_total`, `http_request_duration_seconds`, métriques Node) |
| `GET /greet?name=Ada` | `{"message":"Bonjour, Ada"}` (`Visiteur` par défaut) |
| `GET /welcome?name=Ada` | `{"message":"Bienvenue, Ada"}` |
| `GET /stats` | Nombre de requêtes reçues depuis le démarrage |
| `GET /slug/:text` | Slugifie un texte |
| `POST /total` | `{ "items": [{ "price": 10, "qty": 2, "vat": 0.2 }] }` → total TTC |
| `GET /slow?ms=500` | Répond après `ms` millisecondes (pour tester la latence et les alertes) |
| `GET /boom` | Répond 500 (pour tester les alertes d'erreurs) |

## Vérifier avant de pousser

```bash
npm run lint     # ESLint
npm test         # node --test
npm run build    # produit dist/ (artefact)
```

## Docker

```bash
docker build -t epf-app:local --build-arg APP_VERSION=local .
docker run --rm -p 3000:3000 -e APP_ENV=local epf-app:local
```

## Configuration

Uniquement par variables d'environnement (voir `.env.example`) : `PORT`, `APP_ENV`, `APP_VERSION`.
Le fichier `.env` n'est jamais versionné.

## Comment livrer

**Proposer un changement** : une branche (`feat/…`, `fix/…`), une Pull Request relue par un
autre membre, CI verte, *Squash and merge*. Commits au format `type(portée): description`
(`feat`, `fix`, `ci`, `docs`, `chore`…).

**Publier une version** : PR `chore(release): X.Y.Z` qui met à jour `package.json`, puis

```bash
git switch main && git pull
git tag vX.Y.Z && git push origin vX.Y.Z
```

Le workflow *Release* ne publie l'image que si le scan et le test de démarrage passent.

**Si une version est cassée** : elle n'est pas publiée, `stable` reste sur la précédente.
On corrige par `git revert <sha>` en PR, puis une version correctif `vX.Y.(Z+1)`.
