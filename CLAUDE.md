# Consignes pour Claude Code — espace de travail codif-ape

Ce dépôt est un **espace de travail**, pas un produit. Il regroupe les dépôts du projet de codification automatique de l'APE en sous-modules qui suivent `main`. L'architecture (qui produit quoi, où sont les données sur S3, qui déploie quoi) est décrite dans `README.md` : le lire avant toute tâche qui touche plusieurs dépôts.

## Sous-modules

- Mise à jour : `git submodule update --remote --recursive`. Les pointeurs du méta-dépôt sont avancés chaque nuit par le workflow `.github/workflows/maj-sous-modules.yml` : ne pas faire de commit qui ne fait que les avancer. `.gitmodules` déclare `ignore = all`, donc `git status` à la racine ne montre pas les changements faits dans les sous-modules : faire `git status` dans le sous-module.
- Chaque sous-module est un dépôt à part entière, avec sa CI, ses tags et ses conventions. Pour modifier un sous-module, s'y placer, faire `git switch main` (ou créer une branche, puisque `update --remote` laisse la HEAD détachée), puis commiter **dans le sous-module**. Le cas échéant, lire d'abord son `CLAUDE.md` ou son `README.md`.
- Ne jamais commiter de code applicatif à la racine de ce dépôt : elle ne contient que la documentation de l'espace de travail.

## Contrats entre dépôts

Les dépôts ne s'importent pas entre eux : ils communiquent par artefacts. Un changement qui casse l'un de ces contrats doit être répercuté de l'autre côté dans la même session, ou au minimum signalé.

1. **preprocess → train** : `s3://projet-ape/data/shared_constants.json` (dossier de données courant par révision, listes de variables, renommages) et les colonnes des `split/df_*.parquet`.
2. **train → api** : le modèle MLflow `pyfunc`. Son contrat d'entrée et de sortie est défini dans `codif-ape-train/src/api_wrapper/models/`, et `codif-ape-api/src/api/models/` doit rester aligné.
3. **api / modèle → cd** : tag d'image et `MLFLOW_MODEL_NAME` / `MLFLOW_MODEL_VERSION` dans `codif-ape-cd/<service>/deployment.yaml`. Argo CD synchronise automatiquement : **un commit sur `main` de codif-ape-cd est un déploiement**. Ne jamais en pousser un sans demande explicite.

## Prudence

- Les données Sirene et les buckets S3 (`projet-ape`) ne se modifient qu'à la demande. Pour explorer, ne faire que des lectures.
- Ne jamais écrire un secret (clés S3, tokens Label Studio, identifiants d'API) dans un fichier. Les manifestes ne référencent que des noms de secrets Kubernetes.
- La note publique (`codif-ape-docs`) s'adresse à un public externe : pas de chemins S3, pas d'URL de services internes, pas de feuille de route interne. Toute modification doit être faite dans les deux langues (`en/` et `fr/`).
