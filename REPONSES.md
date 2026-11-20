# Réponses – TP 6 : Variables d'environnement, Inputs et Outputs

### Q1. Pourquoi la variable n'est-elle pas disponible dans la step qui l'a définie, mais seulement dans les suivantes ?
Chaque step d'un job s'exécute dans un sous-processus shell distinct et isolé. Lorsqu'une commande écrit dans le fichier pointé par `$GITHUB_ENV`, le runner GitHub Actions lit ce fichier **à la fin de l'exécution de la step courante** et injecte ces variables dans l'environnement des processus des steps suivantes. La variable n'existe donc pas encore dans l'environnement mémoire du processus en cours.

### Q2. Ajoutez un second job `verify-env` avec une step `Print APP_NAME` et tentez d'y afficher `$APP_NAME`. Que constatez-vous ? Quelle est donc la portée de `GITHUB_ENV` ?
**Constat :** L'affichage de `$APP_NAME` dans le job `verify-env` renvoie une chaîne vide (la variable n'est pas définie).
**Portée :** La portée de `GITHUB_ENV` est strictement limitée au **job courant** (et uniquement aux steps exécutées après sa définition). Chaque job s'exécute sur une machine virtuelle (runner) fraîche et isolée ; les variables d'environnement ne sont pas partagées entre différents jobs.

### Q3. Que se passe-t-il si vous écrivez `>` au lieu de `>>` dans `GITHUB_OUTPUT` alors que deux outputs sont définis dans la même step ?
L'opérateur `>` écrase (tronque) intégralement le contenu du fichier `$GITHUB_OUTPUT` avant d'écrire la nouvelle ligne. Par conséquent, le premier output est effacé et perdu : seul le second (dernier) output écrit sera conservé et accessible. Il faut impérativement utiliser `>>` pour ajouter (append) chaque output à la suite sans écraser les précédents.

### Q4. Pourquoi `GITHUB_ENV` ne suffit-il pas ici, alors que `GITHUB_OUTPUT` fonctionne entre jobs ?
`GITHUB_ENV` modifie uniquement les variables d'environnement locales de la VM du job en cours, qui est détruite à la fin du job. À l'inverse, `GITHUB_OUTPUT` permet d'exporter des valeurs vers le moteur d'orchestration de GitHub Actions. En les déclarant dans la section `outputs:` au niveau du job, GitHub Actions stocke ces valeurs côté serveur et les rend accessibles aux jobs dépendants via le contexte `needs.<job_id>.outputs.<output_name>`.

### Q5. Que se passe-t-il si vous retirez `needs: build` du job `deploy` ?
Deux choses se produisent :
1. **Exécution parallèle :** Les jobs `build` et `deploy` démarreront simultanément au lieu d'attendre la fin de `build`.
2. **Perte d'accès aux outputs :** Le contexte `needs.build.outputs.full_version` sera vide (ou invalide), car `deploy` n'a plus de dépendance déclarée envers `build`.
