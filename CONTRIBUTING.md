# Contribuer

Merci de votre intérêt pour ce projet. Toutes les contributions sont les bienvenues : correction de bugs, nouvelles fonctionnalités, documentation, traductions, idées.

## Pour commencer

1. Lisez le [README](README.md) et la [documentation](docs/)
2. Parcourez les [issues](../../issues). Les labels `good first issue` et `help wanted` signalent les sujets accessibles
3. La liste des problèmes connus est dans [docs/limitations-and-roadmap.md](docs/limitations-and-roadmap.md)
4. Pour un gros changement, ouvrez d'abord une issue pour en discuter

## Processus

1. **Fork** le repo, puis créez une branche : `git checkout -b fix/blacklist-lookup`
2. Faites vos modifications
3. Testez-les dans une instance n8n (voir [docs/payloads-and-testing.md](docs/payloads-and-testing.md))
4. Committez avec un message clair (`fix:`, `feat:`, `docs:`, `refactor:`)
5. Ouvrez une **Pull Request** en remplissant le modèle

## ⚠️ Export sécurisé d'un workflow n8n (obligatoire)

Un export n8n brut contient des secrets. **Avant chaque commit** qui touche `workflow.json` :

- [ ] Supprimer tous les blocs `"credentials"` (identifiants et noms de credentials)
- [ ] Supprimer tous les `"webhookId"`
- [ ] Vider `"meta"` (`instanceId`) et retirer `"id"` et `"versionId"` du workflow
- [ ] Remplacer toute URL contenant un token ou une clé (`?key=...`, `/token/...`) par une variable `{{ $env.NOM }}`
- [ ] Remplacer tout identifiant Google Sheet par `YOUR_GOOGLE_SHEET_ID_HERE`
- [ ] Remplacer tout numéro de téléphone, email ou nom réel par un placeholder ou `{{ $env.ADMIN_PHONE }}`
- [ ] Remplacer tout code promo réel par `{{ $env.PROMO_CODE }}`
- [ ] Mettre `"active": false`
- [ ] Rechercher dans le fichier : `AIza`, `token`, `@gmail`, `@c.us`, `api.greenapi.com/waInstance`, `docs.google.com`, `ngrok`
- [ ] Vérifier que les captures d'écran ne montrent ni numéro, ni IP, ni URL privée

Si vous avez commité un secret par erreur : **révoquez-le immédiatement** chez le fournisseur (supprimer le repo ne suffit pas, l'historique git le conserve), puis prévenez via [SECURITY.md](SECURITY.md).

## Conventions

- **Nœuds Code (JavaScript)** : commentaires courts en français ou en anglais, pas de valeur en dur (utiliser `$env`)
- **Noms de nœuds** : explicites, sans doublons (`Relance 1`, pas `Relance1`)
- **Expressions n8n** : un seul `=` au début, accolades `{{ }}` pour les variables
- **Documentation** : mettre à jour `docs/` si le comportement change
- **Pas de nouvelle dépendance externe** sans discussion préalable

## Idées de contributions

- Corriger les expressions `==` listées dans la documentation (la liste de blacklist ne fonctionne probablement pas)
- Rendre le pays, la devise et le fuseau horaire configurables
- Remplacer les nœuds Twilio incomplets par un envoi SMS fonctionnel
- Ajouter un arrêt des relances dès que la commande est confirmée ou payée
- Ajouter des tests et des exemples de payloads
- Traduire la documentation en anglais

## Code de conduite

En participant, vous acceptez le [Code de conduite](CODE_OF_CONDUCT.md).

## Licence

En contribuant, vous acceptez que votre contribution soit publiée sous la licence MIT du projet.
