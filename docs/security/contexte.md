# Contexte de sécurité - fork Juice Shop

## 1. Contexte métier

Juice Shop est une boutique en ligne de démonstration destinée aux clients d'un commerçant.
Elle permet de créer un compte, consulter le catalogue, gérer un panier, passer commande et publier des avis.
Elle manipule des données d'identité et de contact, des historiques d'achat ainsi que des données de carte et de portefeuille.
Une fuite peut entraîner fraude, préjudice RGPD et perte de confiance ; une indisponibilité empêche les ventes et dégrade la réputation du commerçant.

## 2. Biens essentiels

| # | Bien essentiel | Pourquoi il a de la valeur métier | Biens supports qui le portent |
|---|---|---|---|
| BE1 | Comptes et données personnelles des clients | Ils permettent l'identification des clients et la relation commerciale ; leur confidentialité est une obligation réglementaire et conditionne la confiance. | Modèles Sequelize dans la base SQLite (`User`, adresses et réponses de sécurité), serveur Express, frontend Angular, jetons JWT et clé `encryptionkeys/jwt.pub`. |
| BE2 | Commandes, paniers et historique d'achat | Ils constituent les transactions commerciales, permettent la préparation des ventes et servent de preuve en cas de litige. | Modèles Sequelize/SQLite pour les paniers et leurs articles, collection MarsDB `orders` pour les commandes, routes Express, frontend Angular et factures PDF dans `ftp/`. |
| BE3 | Moyens et données de paiement | Ils sont nécessaires à l'encaissement ; leur divulgation ou leur altération expose les clients et le commerçant à la fraude. | Modèles Sequelize/SQLite `Card` et `Wallet`, routes Express de paiement et de commande, jetons JWT. |
| BE4 | Catalogue et avis produits | Ils soutiennent les ventes et l'image de marque ; leur altération peut tromper les clients, perturber les commandes et faire perdre leur confiance. | Modèle Sequelize/SQLite `Product`, collection MarsDB `posts` pour les avis, routes Express, frontend Angular et fichiers d'images téléversés. |

## 3. Sources de risque

Un cybercriminel peut chercher à voler des données personnelles ou de paiement pour les revendre, commettre des fraudes ou usurper des comptes.
Un concurrent malveillant, un client mécontent ou un acteur opportuniste peut viser l'altération du catalogue, des avis ou l'indisponibilité du service afin de nuire à la réputation et au chiffre d'affaires de la boutique.

## 4. Événements redoutés

| # | Événement redouté (fait + impact) | Bien essentiel touché | Gravité (1 à 4) | Justification de la gravité |
|---|---|---|---|---|
| ER1 | Divulgation des comptes et données personnelles des clients, entraînant usurpation d'identité, préjudice RGPD et perte durable de confiance. | BE1 | 4 | Données personnelles de l'ensemble des clients potentiellement exposées, avec obligation de notification et impact réputationnel majeur. |
| ER2 | Altération ou divulgation des données de paiement et des commandes, entraînant des transactions frauduleuses, des pertes financières et des litiges. | BE2, BE3 | 4 | L'atteinte combine une fraude directe pour les clients et le commerçant avec une dégradation forte de la crédibilité de la boutique. |
| ER3 | Indisponibilité de la boutique pendant 24 heures ou altération du catalogue et des avis, empêchant ou perturbant les ventes et dégradant son image. | BE2, BE4 | 3 | La perte de chiffre d'affaires et la dégradation de réputation sont importantes, mais l'impact reste réversible après remise en service et restauration des données. |

## 5. Suivi

- Run `ci` de référence : à renseigner avec l'URL du premier run vert dans l'onglet **Actions**.
