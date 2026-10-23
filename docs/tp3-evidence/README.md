# TP3 — Audit Docker avec Trivy

## Périmètre réalisé

Le dépôt `network-security-tool` contient l’outil Python du TP précédent : il vérifie si un port local est ouvert. Il ne contient pas l’API, le frontal NGINX ni la base de données décrits dans le sujet. Le TP a donc été appliqué à cet outil avec un seul service Docker.

La commande `docker compose run --rm network-check` a fonctionné et a affiché `Closed` pour le port testé. Aucun port n’est publié sur la machine hôte et le conteneur utilise `network_mode: none`.

## Résultats Trivy

- **Avant durcissement — `python:3.12-slim`** : 44 vulnérabilités HIGH et aucune CRITICAL ont été détectées dans l’image Debian. La capture montre notamment `CVE-2026-76642`, `CVE-2026-78408` et `CVE-2026-78409`.
- **Après durcissement — `python:3.12-alpine`** : aucune vulnérabilité ni aucun secret n’a été détecté dans l’image analysée.
- **Analyse Dockerfile** : Trivy a signalé une recommandation LOW concernant `HEALTHCHECK` (DS-0026). Elle n’a pas été ajoutée car le programme est une commande interactive qui s’arrête après son exécution, et non un service permanent à surveiller.

Captures : `trivy-before-alpine.png`, `trivy-after-alpine.png`, `trivy-dockerfile-check.png` et `github-checks-success.png`.

## Mesures appliquées

- Image de base remplacée par `python:3.12-alpine`.
- Exécution sous un utilisateur non-root (`10001:10001`).
- Système de fichiers en lecture seule.
- Capacités Linux supprimées (`cap_drop: ALL`).
- Élévation de privilèges désactivée (`no-new-privileges:true`).
- Réseau du conteneur désactivé (`network_mode: none`).
- Workflow GitHub Actions qui construit et scanne l’image lors des push et des pull requests vers `main`.
- Le scan CI échoue si Trivy détecte une vulnérabilité HIGH ou CRITICAL. Les quatre contrôles de la pull request ont réussi.

## Réponses aux questions

### Q1. Quelle différence entre scanner le code source et scanner une image Docker ?

Le scan du code source recherche des erreurs ou des pratiques risquées dans les fichiers du programme. Le scan de l’image Docker examine ce qui est réellement inclus dans l’image, notamment le système Linux et les paquets installés.

### Q2. Quelle différence entre un port exposé et un port publié sur l’hôte ?

Un port exposé indique qu’un programme dans le conteneur peut l’utiliser. Un port publié dans Compose, par exemple avec `ports:`, rend ce port accessible depuis la machine hôte. Ici, aucun port n’est publié.

### Q3. Pourquoi isoler une base de données sur le réseau backend ?

Cela limite l’accès à la base aux seuls services qui en ont besoin et évite de la rendre directement accessible depuis la machine hôte ou l’extérieur.

### Q4. Quel risque présente un conteneur exécuté en root ?

Si le programme ou le conteneur est compromis, l’attaquant dispose de privilèges élevés dans le conteneur. Un utilisateur non-root limite les actions possibles.

### Q5. Que signifient HIGH, CRITICAL, CVE et version corrigée ?

HIGH et CRITICAL indiquent la gravité estimée d’une vulnérabilité ; CRITICAL est la catégorie la plus grave. CVE est son identifiant public. La version corrigée est celle où le défaut a été réparé, si elle est disponible.

### Q6. Que fait `--exit-code 1` et pourquoi l’utiliser en CI ?

Cette option fait terminer Trivy avec le code `1` lorsqu’il trouve une vulnérabilité correspondant aux critères choisis. GitHub Actions interprète ce code comme un échec du pipeline.

### Q7. Pourquoi une image sans CVE détectée n’est-elle pas nécessairement sûre ?

Le scan repère les problèmes connus et couverts par ses bases de données. Il ne prouve pas l’absence de défauts inconnus, de problèmes dans le code, de mauvaise configuration ou de risques à l’exécution.

### Q8. Quels risques peuvent subsister après la mise à jour des dépendances ?

Une mise à jour peut introduire une vulnérabilité, casser la compatibilité ou provoquer une régression. Il faut donc relancer les tests et les scans après les mises à jour.
