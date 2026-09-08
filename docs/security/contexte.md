# Contexte de sécurité - fork Juice Shop

## 1. Contexte métier

Juice Shop est une boutique en ligne de démonstration qui permet aux clients de créer un compte, consulter un catalogue, ajouter des produits au panier, passer commande et publier des avis. Elle manipule des données d'identité et de contact, l'historique des achats ainsi que des informations liées au paiement. Pour le commerçant, une fuite de données peut provoquer un préjudice RGPD, des fraudes et une perte de confiance. Une indisponibilité de la boutique empêche les ventes et dégrade sa réputation.

## 2. Biens essentiels

| # | Bien essentiel | Pourquoi il a de la valeur métier | Biens supports qui le portent |
|---|---|---|---|
| BE1 | Comptes et données personnelles des clients | Ils permettent l'identification des clients et la relation commerciale ; leur confidentialité est une obligation réglementaire et conditionne la confiance. | Base SQLite via Sequelize, serveur Express, formulaires Angular, jetons JWT. |
| BE2 | Commandes, paniers et historique d'achat | Ils constituent les transactions commerciales, permettent la préparation des ventes et servent de preuve en cas de litige. | Base SQLite via Sequelize, API Express, interface Angular, journaux applicatifs. |
| BE3 | Moyens et données de paiement | Ils sont nécessaires à l'encaissement ; leur divulgation ou leur altération expose les clients et le commerçant à la fraude. | Base SQLite via Sequelize, API de paiement et serveur Express, variables/configuration applicatives. |
| BE4 | Catalogue, avis et disponibilité de la boutique | Ils soutiennent la vente et l'image de marque : des informations altérées ou un service indisponible font perdre des ventes et la confiance des clients. | Base SQLite via Sequelize, serveur Express, frontend Angular, système de fichiers (`ftp/`, `uploads/`, `logs/`). |

## 3. Sources de risque

Un cybercriminel peut chercher à voler des données personnelles ou de paiement pour les revendre, commettre des fraudes ou usurper des comptes. Un concurrent malveillant, un client mécontent ou un acteur opportuniste peut aussi viser l'altération du catalogue, des avis ou l'indisponibilité du service afin de nuire à la réputation et au chiffre d'affaires de la boutique.

## 4. Événements redoutés

| # | Événement redouté (fait + impact) | Bien essentiel touché | Gravité (1 à 4) | Justification de la gravité |
|---|---|---|---|---|
| ER1 | Divulgation des comptes et données personnelles des clients, entraînant usurpation d'identité, préjudice RGPD et perte durable de confiance. | BE1 | 4 | Données personnelles de l'ensemble des clients potentiellement exposées, avec obligation de notification et impact réputationnel majeur. |
| ER2 | Altération ou divulgation des données de paiement et des commandes, entraînant des transactions frauduleuses, des pertes financières et des litiges. | BE2, BE3 | 4 | L'atteinte combine une fraude directe pour les clients et le commerçant avec une dégradation forte de la crédibilité de la boutique. |
| ER3 | Indisponibilité ou altération du catalogue et des avis pendant 24 heures, empêchant les ventes et dégradant l'image de la boutique. | BE4 | 3 | La perte de chiffre d'affaires et la dégradation de réputation sont importantes, mais l'impact reste réversible après remise en service et restauration des données. |

## 5. Suivi

- Run `ci` de référence : à renseigner avec l'URL du premier run vert dans l'onglet **Actions**.
