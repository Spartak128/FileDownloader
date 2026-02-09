# Audit sécurité — connexions externes et envoi de données

## Périmètre
- `library/src/main/java`
- `demo/src/main/java`
- manifests Android

## Résultat synthétique
- **Aucune tentative d'exfiltration silencieuse** (analytics/tracking/upload automatique) n'a été trouvée dans le code source audité.
- Le projet **effectue des connexions réseau externes par conception** (c'est une librairie de téléchargement), en se connectant aux URL fournies par l'appelant / la démo.
- Les requêtes observées sont orientées téléchargement (lecture `InputStream`, en-têtes HTTP, `Range`, éventuellement `HEAD` pour un essai), sans corps de requête ni API explicite d'upload côté implémentation par défaut.

## Points vérifiés
1. **Création de connexions sortantes**
   - `FileDownloadUrlConnection` ouvre une connexion via `URL.openConnection(...)`, puis exécute `connect()`.
2. **Méthodes HTTP et en-têtes**
   - `ConnectionProfile` peut définir `HEAD` (mode d'essai) et ajoute l'en-tête `Range`.
   - `ConnectTask` ajoute des en-têtes utilisateur et un `User-Agent`.
3. **Lecture vs envoi de données**
   - L'interface `FileDownloadConnection` expose `getInputStream()` et ne force aucun flux de sortie (`getOutputStream`) dans l'implémentation par défaut.
4. **Permissions Android**
   - Le module démo déclare `INTERNET` (normal pour télécharger), mais pas de permission liée à un SDK analytics spécifique.
5. **URLs externes codées en dur**
   - Le module démo contient de nombreuses URLs HTTP de test dans `Constant.java`.

## Conclusion
- **Pas d'envoi de données non sollicité détecté** dans la librairie elle-même.
- **Connexions externes présentes et attendues** pour télécharger des fichiers.
- Si votre politique exige un consentement explicite utilisateur avant toute connexion, il faudra l'implémenter côté application appelante (avant `start()` des tâches), car la librairie exécute la connexion dès qu'une tâche de téléchargement est lancée.
