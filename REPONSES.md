# Réponses – TP 7 : Actions tierces et actions personnalisées

### Q1. Pourquoi `actions/checkout` n'a-t-il pas besoin de `with` alors que `setup-node` en a besoin ?
Tous les inputs de `actions/checkout` possèdent des valeurs par défaut adaptées au contexte courant (comme `repository: ${{ github.repository }}` et `ref: ${{ github.sha }}`), ce qui permet de cloner le dépôt actuel sans configuration supplémentaire.
À l'inverse, `actions/setup-node` a besoin de connaître la version cible exacte de Node.js exigée par le projet (`node-version: 20`) afin d'installer l'environnement compatible avec l'application.

### Q2. Pourquoi le job `verify` ne retrouve-t-il pas `report.txt` sans l'artefact, alors que les deux jobs partagent le même dépôt ?
Chaque job s'exécute sur une **machine virtuelle (runner) complètement distincte, neuve et isolée**. Le système de fichiers créé durant le job `build` (y compris `report.txt`) est détruit à la fin du job `build`. Comme `report.txt` n'est pas commité dans Git, le runner du job `verify` ne peut pas le voir lors du `checkout` : il faut utiliser `upload-artifact` et `download-artifact` pour transférer le fichier via le stockage intermédiaire de GitHub Actions.

### Q6. Pourquoi `actions/checkout` doit-il s'exécuter avant l'appel d'une action locale ?
Une action locale (`./.github/actions/build-info`) est stockée dans les fichiers de votre dépôt Git. Au démarrage du job, le répertoire de travail (`$GITHUB_WORKSPACE`) du runner est vide. Il faut donc exécuter `actions/checkout` en premier pour télécharger les fichiers du dépôt sur le runner afin que le fichier `action.yml` de l'action locale soit présent sur le disque au moment où GitHub Actions l'appelle.

### Q7. Pourquoi `shell: bash` est-il obligatoire sur les steps `run` d'une action composite ?
Dans un workflow standard, GitHub Actions déduit un shell par défaut selon l'OS du runner (`bash` sur Linux/macOS, `pwsh` sur Windows). Dans une **action composite** (`using: composite`), GitHub Actions impose de déclarer explicitement `shell:` sur chaque step `run` afin de garantir la portabilité de l'action et d'éviter toute ambiguïté d'exécution quel que soit le système d'exploitation du runner appelant.

---

### Note – Comparaison des temps d'exécution (Partie 4 : Action Composite vs Action Docker)
* **Action composite (`build-info`) :** Exécution quasi instantanée (~0 à 1 seconde), car les commandes s'exécutent directement dans le shell du runner déjà démarré.
* **Action Docker (`docker-hello`) :** Temps d'exécution nettement plus long (~5 à 15 secondes), car le runner doit télécharger l'image de base (`alpine:3.20`) et construire l'image Docker (`docker build`) avant de démarrer le conteneur.
