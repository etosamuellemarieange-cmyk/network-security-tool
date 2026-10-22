# Réponses au TP 2 — CI et sécurité

## Question 1
Les tests vérifient la validation du numéro de port : 443 est dans la plage autorisée, et 70000 est refusé. Ils ne tentent pas de connexion au port 443 ; aucun service réseau n’a donc besoin d’être ouvert.

## Question 2
Un événement `push` se déclenche quand des commits sont envoyés sur une branche. Un événement `pull_request` se déclenche quand une Pull Request vers `main` est créée ou mise à jour, avant la fusion.

## Question 3
Si pytest échoue, il renvoie un code d’erreur. GitHub Actions marque alors l’étape et le job en échec ; une règle de protection peut empêcher la fusion tant que le contrôle échoue.

## Question 4
Bloquer l’intégration quand les tests échouent évite d’ajouter à la branche principale une modification qui risque de casser le projet.

## Question 5
pytest exécute des tests pour vérifier le comportement du programme. Bandit analyse le code sans l’exécuter afin de repérer certaines pratiques potentiellement risquées.

## Vérification de compilation
`python -m compileall -q network_check.py` vérifie que Python peut compiler le fichier, notamment qu’il ne contient pas d’erreur de syntaxe. Cette vérification ne valide pas le comportement du programme ni sa sécurité ; elle complète donc pytest et Bandit.
