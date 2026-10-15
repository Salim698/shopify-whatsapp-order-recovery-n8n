# Scoring clients et messages

## Niveaux de score

Le score est calculé par les nœuds `Clean + Score Abandon` et `Clean + Score Commande`. Les montants sont dans la devise de la boutique (le texte des messages affiche `DH`).

### Flux 1 (panier abandonné)

| Score | Condition |
|---|---|
| PLATINUM | total ≥ 1000, **ou** total ≥ 500 et ville premium |
| VIP | total ≥ 500, **ou** total ≥ 300 et au moins 3 articles |
| PREMIUM | total ≥ 200 |
| NORMAL | total ≥ 100 |
| RISQUE | total < 100 |

Villes premium : Casablanca, Rabat, Marrakech, Agadir, Tanger.

### Flux 2 (commande)

| Score | Condition |
|---|---|
| PLATINUM | total ≥ 1000 |
| VIP | total ≥ 500 |
| PREMIUM | total ≥ 200 |
| NORMAL | total ≥ 100 |
| RISQUE | total < 100 |

La logique n'est pas identique entre les deux flux (bonus ville et nombre d'articles seulement pour le panier abandonné). Un seul module partagé serait une amélioration.

## Messages de confirmation de commande

| Score | Message |
|---|---|
| PLATINUM | Livraison express 24 h et cadeau offert |
| VIP | Livraison prioritaire (client privilégié) |
| PREMIUM | Livraison prioritaire incluse |
| NORMAL | Message de confirmation standard |
| RISQUE | Alerte à l'administrateur (panier < 100), puis message standard |

Tous demandent une réponse **1 (oui) ou 2 (non)** avec l'ID de commande.

## Séquence de relances

### Panier abandonné

| Étape | Message | Délai |
|---|---|---|
| Premier contact | Rappel du produit et du lien | Immédiat |
| Relance 1 | Le panier est toujours disponible | Après le premier `Wait` |
| Relance 2 | Stock limité sur le produit | Après `Wait 6h` |
| Relance 3 | Dernière chance, `-10 %` avec le code `PROMO_CODE` | Après `Wait 24h` |

### Commande en attente

| Étape | Message | Condition |
|---|---|---|
| Relance 1 | Rappel : répondre OUI ou NON avec l'ID | Statut encore `pending` |
| Relance 2 | Stock limité | Statut encore `pending` |
| Relance 3 | Dernière chance avec le code promo | Statut encore `pending` |

> Les durées réelles des nœuds `Wait` sont à vérifier dans n8n : plusieurs nœuds ont des paramètres vides (valeurs par défaut de n8n) alors que leur nom annonce 30 min ou 24 h. Voir [limitations](limitations-and-roadmap.md).

## Plage horaire d'envoi

Trois nœuds `Check Heure Envoi` empêchent les envois de nuit :

| Nœud | Fuseau | Plage autorisée |
|---|---|---|
| `Check Heure Envoi` | UTC | 8 h - 21 h |
| `Check Heure Envoi1` et `2` | UTC+1 (Maroc) | 9 h - 22 h |

Hors plage, le workflow attend l'ouverture de la plage, puis envoie.

## Analyse IA des réponses

Le prompt de classification demande à Gemini de répondre par un seul mot : `CONFIRM`, `CANCEL` ou `QUESTION`, pour un message en **français ou en darija**. Toute réponse non reconnue est traitée comme `QUESTION`.

Les questions reçoivent une réponse courte d'un agent SAV, dans la langue du client. Température 0,7 et 300 tokens maximum.
