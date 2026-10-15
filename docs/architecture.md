# Architecture des 4 flux

Le workflow contient 95 nœuds répartis en **4 flux indépendants**, chacun déclenché par un webhook.

## Flux 1 - Relance de panier abandonné

Webhook : `POST /webhook/abandoned`

```mermaid
flowchart TD
  A["Webhook Abandon"] --> B["Clean + Score Abandon"]
  B --> C{"Téléphone valide ?"}
  C -->|non| X["Fin"]
  C -->|oui| D["Check Blacklist - Sheet"]
  D --> E{"Dans la liste noire ?"}
  E -->|oui| X
  E -->|non| F["Check WhatsApp - Green API"]
  F --> G{"A WhatsApp ?"}
  G -->|oui| H["Send WhatsApp"]
  G -->|non| I["Send SMS - Twilio"]
  H --> J["Merge"]
  I --> J
  J --> K["Wait"]
  K --> L{"Plage horaire OK ?"}
  L -->|oui| M["Relance 1 + Log"]
  L -->|non| W["Attente jusqu'à l'ouverture"]
  W --> M
  M --> N["Wait 6h"]
  N --> O["Relance 2 + Log"]
  O --> P["Wait 24h"]
  P --> Q["Relance 3 finale + Log"]
```

1. Nettoyage du numéro (normalisation `226`), validation, calcul du score
2. Contrôle de la liste noire, puis vérification que le numéro est sur WhatsApp
3. Premier message WhatsApp (ou SMS de secours), puis 3 relances espacées
4. Avant chaque relance : contrôle de la **plage horaire d'envoi**, sinon attente
5. Chaque relance est enregistrée dans `Relances Log`

> ⚠️ Ce flux ne vérifie pas si le client a finalement passé commande entre deux relances (voir [limitations](limitations-and-roadmap.md)).

## Flux 2 - Confirmation de commande et relances

Webhook : `POST /webhook/order-created`

```mermaid
flowchart TD
  A["Webhook Commande"] --> B["Clean + Score Commande"]
  B --> C{"Téléphone invalide ?"}
  C -->|oui| X["Fin"]
  C -->|non| D["Check Doublon - Sheet"]
  D --> E{"Déjà enregistrée ?"}
  E -->|oui| X
  E -->|non| F["Save Pending Orders"]
  F --> G{"Dans la liste noire ?"}
  G -->|oui| X
  G -->|non| H["Check WhatsApp"]
  H --> I{"A WhatsApp ?"}
  I -->|non| S["Send SMS - Twilio"]
  I -->|oui| J{"Score client"}
  J -->|PLATINUM| P1["WhatsApp PLATINUM"]
  J -->|VIP| P2["WhatsApp VIP"]
  J -->|PREMIUM| P3["WhatsApp PREMIUM"]
  J -->|RISQUE| P4["Alerte admin + message standard"]
  J -->|NORMAL| P5["WhatsApp standard"]
  P1 --> M["Merge"]
  P2 --> M
  P3 --> M
  P4 --> M
  P5 --> M
  S --> M
  M --> W1["Wait"]
  W1 --> R1{"Toujours pending ?"}
  R1 -->|oui| RL1["Relance confirmation + Log"]
  RL1 --> W2["Wait 6h"]
  W2 --> R2{"Toujours pending ?"}
  R2 -->|oui| RL2["Relance stock limité + Log"]
  RL2 --> W3["Wait 24h"]
  W3 --> R3{"Toujours pending ?"}
  R3 -->|oui| RL3["Relance finale avec code promo + Log"]
```

1. Nettoyage, validation, scoring, détection de doublon
2. Enregistrement dans `Pending Orders` avec le statut `pending`
3. Message de confirmation **adapté au niveau du client** (le message demande une réponse OUI/NON avec l'ID de commande)
4. Relances en 3 étapes tant que le statut dans le Sheet reste `pending`

> ⚠️ Le nœud `Node IF Blacklist1` n'est précédé d'aucune recherche dans l'onglet `Blacklist` : dans ce flux, la liste noire n'est donc pas réellement consultée (voir [limitations](limitations-and-roadmap.md)).

## Flux 3 - Analyse IA des réponses clients

Webhook : `POST /webhook/whatsapp-reply`

```mermaid
flowchart TD
  A["Webhook Réponse"] --> B["Extract Order ID"]
  B --> C["Get Order from Sheet"]
  C --> D["Build Conversation Context"]
  D --> E["Gemini - Analyse Réponse"]
  E --> F["Parse Analyse"]
  F --> G{"Type de réponse"}
  G -->|CONFIRM| H["Confirm Order in Sheet"]
  H --> H2["Confirmation au client"]
  H2 --> H3["Demander la localisation"]
  H3 --> H4["Notifier l'admin"]
  G -->|CANCEL| I["Cancel Order in Sheet"]
  I --> I2["Message d'annulation"]
  G -->|QUESTION| J["Gemini - Réponse SAV"]
  J --> J2["Parse Gemini Answer"]
  J2 --> J3["Envoyer la réponse"]
```

1. L'ID de commande est extrait du message (motif `ID: xxxx`) ; sans ID, le flux s'arrête
2. Gemini classe la réponse (français ou darija) en `CONFIRM`, `CANCEL` ou `QUESTION`
3. **CONFIRM** : statut `confirmed`, message au client, demande de localisation, alerte admin
4. **CANCEL** : statut `cancelled`, message d'annulation
5. **QUESTION** : Gemini répond comme agent SAV, brièvement, dans la langue du client

## Flux 4 - Adresse de livraison par GPS

Webhook : `POST /webhook/whatsapp-location` (répond avec le dernier nœud)

```mermaid
flowchart TD
  A["Webhook Localisation"] --> B["Extract GPS Data"]
  B --> C["Reverse Geocoding - Nominatim"]
  C --> D["Format Adresse"]
  D --> E["Save GPS to Sheet"]
  E --> F["Notifier l'admin avec GPS"]
```

1. Extraction de la latitude et de la longitude (ou de l'adresse saisie)
2. Géocodage inverse via OpenStreetMap Nominatim
3. Mise à jour de la ligne du client (clé : `phone`) avec adresse, quartier, ville, coordonnées et lien Google Maps
4. Notification WhatsApp à l'administrateur

## Données qui circulent entre les nœuds

Le workflow utilise de nombreux nœuds `Restore ...` (JavaScript) pour ré-injecter les données du client après les appels HTTP et Google Sheets, qui remplacent le contenu de `$json`. Cette technique est lourde : une refactorisation avec le nœud `Set` ou des références `$('Nom du nœud')` est une piste d'amélioration (voir [limitations](limitations-and-roadmap.md)).
