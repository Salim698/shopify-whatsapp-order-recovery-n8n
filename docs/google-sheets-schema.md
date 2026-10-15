# Structure du Google Sheet

Le workflow utilise **un classeur** avec **3 onglets**. Les noms d'onglets et d'en-têtes doivent être respectés à la lettre.

> ⚠️ **Piège des espaces** : plusieurs en-têtes contiennent un espace parasite (`order_id␠`, `prenom␠`, `produit␠`, `phone␠`, `␠produit`) car le workflow a été construit ainsi. Les fichiers de `data/` reprennent ces noms exacts. Si vous préférez des en-têtes propres, corrigez aussi le mapping dans les nœuds Google Sheets (bonne première contribution, voir [limitations](limitations-and-roadmap.md)).

Dans les tableaux ci-dessous, `␠` représente un espace.

## Onglet `Pending Orders`

Commandes en attente de confirmation. Clé de recherche : `order_id␠`.

| Colonne | Contenu | Écrit par |
|---|---|---|
| `order_id␠` | ID de la commande | Save Pending Orders |
| `phone` | Numéro nettoyé (format `226...`) | Save Pending Orders |
| `prenom␠` | Prénom du client | Save Pending Orders |
| `produit␠` | Premier produit de la commande | Save Pending Orders |
| `total` | Montant total | Save Pending Orders |
| `score` | PLATINUM, VIP, PREMIUM, NORMAL ou RISQUE | Save Pending Orders |
| `status` | `pending`, `confirmed` ou `cancelled` | Save, Confirm, Cancel |
| `timestamp` | Date ISO de création | Save Pending Orders |
| `lien` | Lien de paiement ou de finalisation | Save Pending Orders |
| `adresse` | Adresse complète géocodée | Save GPS to Sheet |
| `ville` | Ville | Save GPS to Sheet |
| `quartier` | Quartier | Save GPS to Sheet |
| `latitude` | Latitude GPS | Save GPS to Sheet |
| `longitude` | Longitude GPS | Save GPS to Sheet |
| `maps_link` | Lien Google Maps | Save GPS to Sheet |

## Onglet `Relances Log`

Historique de chaque relance envoyée.

| Colonne | Contenu |
|---|---|
| `phone␠` | Numéro du client |
| `prenom` | Prénom |
| `␠produit` | Produit concerné |
| `relance_num` | 1, 2 ou 3 |
| `order_id` | ID de la commande ou du panier |
| `type` | `abandon` ou `commande` |
| `timestamp` | Date de l'envoi |

## Onglet `Blacklist`

Numéros à ne jamais contacter.

| Colonne | Contenu |
|---|---|
| `phone` | Numéro au format `226XXXXXXXX` |

## Données personnelles

Ces onglets contiennent des numéros, prénoms et adresses de clients. Limitez l'accès au classeur, ne le rendez jamais public, et purgez-le régulièrement.
