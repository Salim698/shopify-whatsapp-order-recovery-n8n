# Installation et configuration

## Prérequis

- Une instance **n8n** (auto-hébergée recommandée : les variables d'environnement y sont disponibles)
- Un compte **Green API** avec une instance WhatsApp connectée
- Une clé API **Google Gemini** (Google AI Studio)
- Un **compte de service Google** avec l'API Google Sheets activée
- Une boutique **Shopify** (ou toute source capable d'envoyer les payloads décrits dans [payloads-and-testing.md](payloads-and-testing.md))

## 1. Variables d'environnement

Copiez `.env.example` et renseignez les valeurs :

| Variable | Description |
|---|---|
| `GREEN_API_URL` | URL de votre instance Green API, ex. `https://1234.api.greenapi.com` |
| `GREEN_API_INSTANCE` | ID d'instance Green API |
| `GREEN_API_TOKEN` | Token d'API de l'instance |
| `GEMINI_API_KEY` | Clé API Gemini |
| `ADMIN_PHONE` | Numéro de l'administrateur (format international, sans `+`) |
| `PROMO_CODE` | Code promo affiché dans la relance finale |

Sur n8n auto-hébergé (Docker), injectez-les dans l'environnement du conteneur.

> Les versions récentes de n8n peuvent bloquer l'accès à `$env` dans les nœuds. En cas d'erreur "access to env vars denied", ajoutez `N8N_BLOCK_ENV_ACCESS_IN_NODE=false`. Sur n8n Cloud, `$env` n'est pas disponible : remplacez ces expressions par des credentials n8n.

## 2. Google Sheet et compte de service

1. Créez un classeur Google Sheets avec **3 onglets** : `Pending Orders`, `Relances Log`, `Blacklist`
2. Respectez exactement les en-têtes décrits dans [google-sheets-schema.md](google-sheets-schema.md) (attention aux **espaces** dans certains noms de colonnes)
3. Dans Google Cloud : créez un **compte de service**, activez l'API Google Sheets, générez une clé JSON
4. **Partagez le classeur** avec l'adresse email du compte de service (rôle Éditeur)
5. Dans n8n : créez une credential **Google Service Account API** avec cette clé
6. L'ID du classeur se trouve dans son URL : `https://docs.google.com/spreadsheets/d/<ID>/edit`

## 3. Import du workflow

1. n8n > **Workflows > Import from file** > `workflow.json`
2. Dans les **16 nœuds Google Sheets**, remplacez `YOUR_GOOGLE_SHEET_ID_HERE` par l'ID de votre classeur et sélectionnez votre credential Google
3. Vérifiez qu'aucun nœud n'affiche d'erreur de credential
4. Le workflow est importé **inactif** : ne l'activez qu'après les tests

> Astuce : n8n permet de modifier l'ID du classeur nœud par nœud. Pour aller plus vite, ouvrez un nœud, changez l'ID, puis copiez-collez ce nœud si besoin.

## 4. Brancher Shopify et WhatsApp

Les URL des webhooks (production) sont :

| Rôle | Chemin | Méthode |
|---|---|---|
| Panier abandonné | `/webhook/abandoned` | POST |
| Commande créée | `/webhook/order-created` | POST |
| Réponse WhatsApp du client | `/webhook/whatsapp-reply` | POST |
| Localisation WhatsApp | `/webhook/whatsapp-location` | POST |

Détails des payloads : [payloads-and-testing.md](payloads-and-testing.md).

- **Shopify** : configurez les webhooks (ou un outil comme Shopify Flow) pour appeler les 2 premiers chemins.
- **WhatsApp** : le format natif des notifications entrantes de Green API est différent du format attendu. Placez un petit workflow ou nœud d'adaptation devant les 2 derniers webhooks, ou modifiez les nœuds `Extract Order ID` et `Extract GPS Data`.

## 5. Tester puis activer

1. Utilisez les URL de test (`/webhook-test/...`) avec les fichiers de `examples/`
2. Vérifiez les lignes ajoutées dans Google Sheets et les messages WhatsApp reçus
3. Activez le workflow (interrupteur en haut à droite)

## Dépannage

| Symptôme | Cause probable |
|---|---|
| Aucun message WhatsApp | Variables Green API absentes ou instance non connectée |
| `access to env vars denied` | Voir `N8N_BLOCK_ENV_ACCESS_IN_NODE` ci-dessus |
| Erreur Google 403 | Classeur non partagé avec le compte de service |
| Colonnes vides dans le Sheet | En-têtes différents de ceux attendus (espaces !) |
| Erreur 400 Gemini | Clé invalide ou modèle indisponible |
| Numéro rejeté | Validation limitée aux numéros `226` (voir limites) |
