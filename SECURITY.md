# Politique de sécurité

## Signaler une vulnérabilité

Si vous découvrez une faille ou un **secret exposé** (clé API, token, numéro, identifiant) dans ce repo :

1. **Ne l'écrivez pas dans une issue publique**
2. Contactez le mainteneur via son profil GitHub, ou utilisez **Security > Report a vulnerability** sur la page du repo si l'option est activée
3. Indiquez le fichier concerné et, si possible, une suggestion de correction

Une réponse est apportée dès que possible.

## Bonnes pratiques pour les utilisateurs

- Stockez les secrets dans des variables d'environnement n8n, jamais dans le workflow
- Protégez les webhooks (token secret, Header Auth) avant de les exposer sur Internet
- Utilisez un compte de service Google avec accès limité à la seule feuille nécessaire
- Limitez la durée de conservation des numéros de téléphone et des adresses dans Google Sheets (données personnelles)
- Régénérez immédiatement toute clé qui aurait été partagée ou commitée

## Données personnelles

Ce workflow traite des numéros de téléphone, prénoms et adresses de clients. Assurez-vous d'avoir une base légale (consentement, intérêt légitime) et d'informer vos clients conformément à la réglementation applicable.
