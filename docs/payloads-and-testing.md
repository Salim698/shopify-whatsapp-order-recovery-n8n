# Payloads attendus et tests

Tous les webhooks acceptent des requêtes `POST` en JSON. Remplacez `<ton-n8n>` par l'URL de votre instance. Utilisez `/webhook-test/` pendant les essais (après avoir cliqué sur **Listen for test event**) et `/webhook/` quand le workflow est activé.

Des fichiers prêts à l'emploi sont dans [`examples/`](../examples/).

## 1. Panier abandonné - `/webhook/abandoned`

Champs lus par le nœud `Clean + Score Abandon` :

| Champ | Usage |
|---|---|
| `id` | Identifiant du panier (généré aléatoirement s'il est absent) |
| `phone` ou `customer.phone` | Numéro du client |
| `customer.first_name` | Prénom (par défaut `client`) |
| `line_items[0].title` | Produit affiché (par défaut `produit`) |
| `line_items.length` | Nombre d'articles (scoring) |
| `total_price` ou `subtotal_price` | Montant (scoring) |
| `shipping_address.city` | Ville (bonus de scoring pour les grandes villes) |
| `abandoned_checkout_url` | Lien inséré dans les messages |

```bash
curl -X POST https://<ton-n8n>/webhook-test/abandoned \
  -H "Content-Type: application/json" \
  -d @examples/abandoned-cart.json
```

Attendu : message WhatsApp avec le produit et le lien, puis relances planifiées.

## 2. Commande créée - `/webhook/order-created`

Mêmes champs que ci-dessus, avec `id` = identifiant de commande.

```bash
curl -X POST https://<ton-n8n>/webhook-test/order-created \
  -H "Content-Type: application/json" \
  -d @examples/order-created.json
```

Attendu : ligne ajoutée dans `Pending Orders` (statut `pending`) et message de confirmation selon le score.

## 3. Réponse du client - `/webhook/whatsapp-reply`

| Champ | Usage |
|---|---|
| `phone` | Numéro de l'expéditeur |
| `message` | Texte reçu. **Doit contenir `ID: <order_id>`**, sinon le flux s'arrête |

```bash
curl -X POST https://<ton-n8n>/webhook-test/whatsapp-reply \
  -H "Content-Type: application/json" \
  -d @examples/whatsapp-reply.json
```

Attendu : statut de la commande mis à jour (`confirmed`, `cancelled`) ou réponse SAV pour une question.

## 4. Localisation - `/webhook/whatsapp-location`

| Champ | Usage |
|---|---|
| `phone` | Numéro de l'expéditeur |
| `location.latitude`, `location.longitude` | Coordonnées GPS |
| `location.address` ou `message` | Adresse saisie à la main (secours) |

```bash
curl -X POST https://<ton-n8n>/webhook-test/whatsapp-location \
  -H "Content-Type: application/json" \
  -d @examples/whatsapp-location.json
```

Attendu : ligne `Pending Orders` du client mise à jour (adresse, quartier, ville) et alerte à l'administrateur. La ligne du client doit déjà exister (mise à jour par `phone`).

## Format des numéros

Le workflow normalise les numéros avec l'indicatif **226** (Burkina Faso) puis valide le motif `226[5-7]XXXXXXX`. Exemples acceptés pour le même numéro : `+22670000000`, `0022670000000`, `22670000000`, `70000000`. Voir [limitations](limitations-and-roadmap.md) pour rendre le pays configurable.

## Scénarios de test conseillés

| Cas | Comment | Résultat attendu |
|---|---|---|
| Numéro invalide | `phone` : `123` | Le flux s'arrête |
| Numéro en liste noire | Ajouter le numéro dans `Blacklist` | Aucun message (voir limites : le contrôle est à vérifier) |
| Commande en double | Envoyer deux fois le même `id` | La seconde est ignorée |
| Panier < 100 | `total_price` : `50` | Score RISQUE, alerte administrateur |
| Réponse OUI | Message `OUI ID: 123456789` | Statut `confirmed` et demande de localisation |
| Question | Message `Livraison possible demain ? ID: 123456789` | Réponse SAV de Gemini |
