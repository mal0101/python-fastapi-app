# Notes de sécurité – TP 7 (Actions tierces)

### Relevé sur le dépôt `actions/setup-node`
* **Badge créateur vérifié :** Oui (l'organisation `actions` dispose du badge bleu *Verified creator* sur GitHub).
* **Nombre d'étoiles :** ~4,5k étoiles.
* **Dossier contenant le code source :** `src/` (le code JavaScript compilé exécuté par le runner se trouve dans `dist/`).

---

### Q3. Pourquoi faut-il épingler une version (`@v4`) plutôt que `@main` ?
1. **Stabilité et reproductibilité :** La branche `@main` reçoit des modifications en continu. Un commit poussé sur `main` peut introduire un changement cassant (breaking change) ou un bug qui fera échouer vos pipelines sans aucune modification de votre dépôt. Un tag de version (`@v4`) garantit un comportement figé et prévisible.
2. **Sécurité (Supply Chain Attack) :** Si le dépôt tiers ou le compte d'un mainteneur est compromis et qu'un code malveillant est poussé sur `@main`, votre workflow exécutera automatiquement ce code dès la prochaine exécution (avec accès aux variables d'environnement et au `GITHUB_TOKEN`).

### Q4. Le badge « créateur vérifié » garantit-il qu'une action est sûre ? Justifiez.
**Non.** Le badge « créateur vérifié » (*Verified creator*) atteste uniquement que GitHub a vérifié l'identité de l'organisation qui publie l'action (appartenance du nom de domaine, etc.). Il ne garantit **pas** que GitHub a audité le code source de l'action. Une organisation vérifiée peut publier une faille de sécurité involontaire, utiliser une dépendance tierce compromise ou subir un vol d'identifiants de publication.

### Q5. Citez deux vérifications à faire avant d'utiliser, en production, une action d'un fournisseur inconnu.
1. **Auditer le code source et les scripts exécutés (`src/`, `dist/`, `Dockerfile`) :** Vérifier que l'action n'envoie pas de données sensibles (secrets, tokens, variables d'environnement) vers un serveur externe et ne contient pas de code obfusqué.
2. **Épingler l'action sur un hash de commit SHA-1 complet (`uses: owner/repo@<commit-sha>`) :** Une fois le code d'un commit précis audité, référencer son hash immuable empêche l'auteur de modifier silencieusement le contenu derrière un tag mobile (comme `@v1`).
