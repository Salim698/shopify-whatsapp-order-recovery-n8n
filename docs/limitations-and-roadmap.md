# Limites connues et roadmap

Cette page liste les problèmes connus et les améliorations possibles. **Chaque ligne peut devenir une issue GitHub** : les labels suggérés sont indiqués. Les points marqués `good first issue` conviennent pour une première contribution.

## Corrigé dans cette version publique

- Clé Gemini passée dans l'URL : désormais dans le header `x-goog-api-key`
- Secrets, numéros et identifiants remplacés par des variables d'environnement
- Nœud `Alerte Admin Risque` : corps de requête invalide (`==`) et champ `number` au lieu de `chatId`

## 🐞 Bugs probables (à vérifier dans n8n)

| # | Sujet | Détail | Labels |
|---|---|---|---|
| 1 | Expressions préfixées par `==` | n8n traite le premier `=` comme marqueur d'expression ; le second devient du texte littéral, donc la valeur évaluée commence par `=` (ex. `=22670000000`). 12 nœuds concernés, voir tableau ci-dessous | `bug`, `good first issue` |
| 2 | Liste noire inopérante (flux 1) | `Check Blacklist` cherche `=={{ clean_phone }}`, donc `=226...`, qui ne correspond à aucune ligne ; de plus `Node IF Blacklist` teste `$json.length`, qui n'existe pas sur un item | `bug` |
| 3 | Liste noire absente (flux 2) | `Node IF Blacklist1` n'est précédé d'aucune recherche dans l'onglet `Blacklist` | `bug` |
| 4 | Test WhatsApp du flux 2 | `Has WhatsApp ?1` utilise `=$json.existsWhatsapp` (sans accolades) : le texte est littéral et non évalué | `bug`, `good first issue` |
| 5 | Durées des `Wait` | `Wait 24h`, `Wait 30min1`, `Wait6`, `Wait24h1` ont des paramètres vides (valeurs par défaut de n8n, soit 1 heure sauf vérification) alors que leur nom annonce 24 h ou 30 min ; `Wait 30min` est réglé sur `10` | `bug` |
| 6 | Relances après achat | Le flux 1 ne vérifie pas si le client a commandé entre deux relances | `bug`, `enhancement` |
| 7 | SMS Twilio incomplets | Pas d'authentification, `ACCOUNT_SID` et numéro `From` en placeholder, corps JSON alors que l'API attend du `form-urlencoded` | `bug`, `help wanted` |
| 8 | Fuseaux horaires incohérents | `Check Heure Envoi` raisonne en UTC, les deux autres en UTC+1 | `bug`, `good first issue` |

### Détail du bug 1

| Nœud | Paramètre |
|---|---|
| `Check Blacklist` | `lookupValue` |
| `Has WhatsApp ?` | `leftValue` |
| `Déjà Enregistré?` | `leftValue` |
| `Log Relance`, `Log Relance1`, `Log Relance2` | `timestamp` |
| `Peut envoyer ?`, `Peut envoyer ?1`, `Peut envoyer ?2` | `leftValue` |
| `Wait` | `dateTime` |
| `Wait1`, `Wait2` | `amount` |

Correction attendue : remplacer `=={{ ... }}` par `={{ ... }}`, puis tester chaque nœud.

## 🚀 Améliorations

| # | Idée | Labels |
|---|---|---|
| 9 | **Pays, devise et fuseau configurables** : l'indicatif `226`, la validation `226[5-7]...`, le texte `DH` et la liste des villes premium sont en dur | `enhancement`, `help wanted` |
| 10 | **Un seul module de scoring** partagé entre les flux 1 et 2 | `enhancement`, `refactor` |
| 11 | **Supprimer les nœuds `Restore ...`** (une vingtaine) au profit de références `$('Nœud')` ou du nœud `Set` | `refactor`, `help wanted` |
| 12 | **Adaptateur pour les webhooks entrants de Green API** (format natif différent de celui attendu) | `enhancement` |
| 13 | **Authentification des webhooks** (Header Auth ou token secret) | `security`, `good first issue` |
| 14 | **Épingler le modèle Gemini** (`gemini-flash-latest` change dans le temps) et externaliser les prompts | `enhancement`, `good first issue` |
| 15 | **Workflow d'erreur global** avec notification à l'administrateur | `enhancement` |
| 16 | **Arrêt des relances** dès qu'une commande est confirmée, payée ou annulée | `enhancement` |
| 17 | **Support de la WhatsApp Business Platform officielle** (Cloud API de Meta) comme alternative à Green API | `enhancement`, `help wanted` |
| 18 | **En-têtes de colonnes propres** dans le Google Sheet (supprimer les espaces) et mise à jour du mapping | `refactor`, `good first issue` |
| 19 | **Purge automatique** des données personnelles anciennes dans Google Sheets | `enhancement`, `security` |
| 20 | **Tableau de bord** (Looker Studio) : taux de confirmation, relances, paniers récupérés | `enhancement` |

## 📚 Documentation

| # | Idée | Labels |
|---|---|---|
| 21 | Traduire la documentation en anglais | `documentation`, `good first issue` |
| 22 | Ajouter des captures d'écran des messages WhatsApp reçus | `documentation`, `good first issue` |
| 23 | Guide pas à pas de configuration du compte de service Google | `documentation` |
| 24 | Guide de configuration des webhooks Shopify (ou Shopify Flow) | `documentation` |

## Limites techniques

- **Green API** pilote un compte WhatsApp non officiel : risque de blocage du numéro en cas d'envoi massif.
- **Webhooks non authentifiés** par défaut.
- **Google Sheets** comme base de données : adapté à un prototype, limité en volume et en concurrence.
- **Aucun test automatisé** : les essais se font à la main avec `examples/`.
- **Données personnelles** (numéros, adresses) stockées dans un classeur : à protéger et à purger.

## Comment transformer cette liste en issues

1. Sur GitHub : **Issues > New issue**
2. Titre = la ligne du tableau, description = le détail
3. Ajoutez les labels indiqués (`good first issue` et `help wanted` existent par défaut)
