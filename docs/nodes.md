# Liste des nœuds

Le workflow contient **95 nœuds**, regroupés ici par flux (ordre de parcours depuis le webhook).


## Flux 1 - Relance de panier abandonné (32 nœuds)

| Nœud | Type |
|---|---|
| 🛒 Webhook Abandon | Webhook |
| 📱 Clean + Score Abandon | Code |
| ❌ Téléphone Invalide? | IF |
| 🚫 Check Blacklist | Google Sheets |
| Node IF Blacklist | IF |
| 📲 Check WhatsApp | HTTP Request |
| 🤔 Has WhatsApp ? | IF |
| 🔗 Merge Phone Data | Code |
| 📩 Send SMS | HTTP Request |
| 💬 Send WhatsApp | HTTP Request |
| Merge | Merge |
| ♻️ Restore Data Abandon | Code |
| ⏱ Wait 30min | Wait |
| ⏰ Check Heure Envoi | Code |
| Peut envoyer ? | IF |
| 🔁 Relance 1 | HTTP Request |
| Wait | Wait |
| 📝 Log Relance | Google Sheets |
| ♻️ Restore After Log | Code |
| ⏱ Wait 6h | Wait |
| ⏰ Check Heure Envoi1 | Code |
| Peut envoyer ?1 | IF |
| RELANCE 2 | HTTP Request |
| Wait1 | Wait |
| 📝 Log Relance1 | Google Sheets |
| ♻️ Restore After Log1 | Code |
| ⏱ Wait 24h | Wait |
| ⏰ Check Heure Envoi2 | Code |
| Peut envoyer ?2 | IF |
| RELANCE 3 (FINAL) | HTTP Request |
| Wait2 | Wait |
| 📝 Log Relance2 | Google Sheets |

## Flux 2 - Confirmation de commande et relances (41 nœuds)

| Nœud | Type |
|---|---|
| 📦 Webhook Commande | Webhook |
| 🧠 Clean + Score Commande | Code |
| ❌ Téléphone Invalide?1 | IF |
| 💾 Save Client Temp | Code |
| 🔍 Check Doublon | Google Sheets |
| ❓ Déjà Enregistré? | IF |
| ♻️ Restore Before Save | Code |
| Save Pending Orders | Google Sheets |
| Node IF Blacklist1 | IF |
| Check WhatsApp | HTTP Request |
| 🤔 Has WhatsApp ?1 | IF |
| 🔗 Merge Phone Data1 | Code |
| 📩 Send SMS1 | HTTP Request |
| 🎯 Switch Score VIP | Switch |
| Merge1 | Merge |
| 👑 Send WhatsApp VIP | HTTP Request |
| ⚠️ Alerte Admin Risque | HTTP Request |
| 👑 Send WhatsApp PLATINUM | HTTP Request |
| 👑 Send WhatsApp  PREMIUM | HTTP Request |
| 💬 Send WhatsApp1 | HTTP Request |
| ♻️ Restore Commande | Code |
| ⏱ Wait 30min1 | Wait |
| 🔎 Check Statut 1 | Google Sheets |
| ❓ Encore Pending? 2 | IF |
| Restore For Relance | Code |
| 🔁 Relance  | HTTP Request |
| 📝 Log Relance3 | Google Sheets |
| ♻️ Restore For Relance | Code |
| Wait6 | Wait |
| 🔎 Check Statut 4 | Google Sheets |
| ❓ Encore Pending? 5 | IF |
| ♻️ Restore For Relance2 | Code |
| RELANCE  | HTTP Request |
| 📝 Log Relance4 | Google Sheets |
| ♻️ Restore For Relance3 | Code |
| Wait24h1 | Wait |
| 🔎 Check Statut 5 | Google Sheets |
| ❓ Encore Pending? 6 | IF |
| ♻️ Restore For Relance4 | Code |
| RELANCE 3 (FINAL)1 | HTTP Request |
| 📝 Log Relance5 | Google Sheets |

## Flux 3 - Analyse IA des réponses clients (16 nœuds)

| Nœud | Type |
|---|---|
| Webhook Réponse | Webhook |
| 🧠 Extract Order ID | Code |
| Get Order from Sheet | Google Sheets |
| 📚 Build Conversation Context | Code |
| 🤖 Analyse Réponse | HTTP Request |
| 🧹 Parse Analyse | Code |
| 🔀 Type Réponse | Switch |
| ✅ Confirm Order in Sheet | Google Sheets |
| ❌ Cancel Order in Sheet | Google Sheets |
| 🤖 Gemini Réponse Question | HTTP Request |
| 🎉 Envoyer Confirmation Client | HTTP Request |
| 😔 Envoyer Message Annulation | HTTP Request |
| 📝 Parse Gemini Answer | Code |
| 📍 Demander Localisation | HTTP Request |
| 💬 Envoyer Réponse Question | HTTP Request |
| 📢 Notifier Admin (Confirmé) | HTTP Request |

## Flux 4 - Adresse de livraison par GPS (6 nœuds)

| Nœud | Type |
|---|---|
| 🌍 Webhook Localisation | Webhook |
| 📐 Extract GPS Data | Code |
| 🗺️ Reverse Geocoding | HTTP Request |
| 📝 Format Adresse | Code |
| 💾 Save GPS to Sheet | Google Sheets |
| 📢 Notifier Admin avec GPS | HTTP Request |
