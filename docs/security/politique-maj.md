# Politique de mise à jour des dépendances

Périmètre : dépendances npm du `package-lock.json` (66 directes + 51 de développement, 1457 entrées dans le lock) et images de base des `Dockerfile`. La détection est faite par le workflow `supply-chain.yml` (`npm audit`, `osv-scanner`, Trivy) ; cette politique dit **quand** et **qui** corrige.

## Cadence
- correctifs (patch) : regroupés chaque semaine, fusionnés si la CI (`ci`, `sast`, `supply-chain`) est verte.
- versions mineures : une fois par sprint (2 semaines), après relecture du changelog par un développeur.
- versions majeures : planifiées, avec un ticket dédié et des tests de non-régression sur les fonctions touchées ; délai cible 30 jours si une vulnérabilité high ou critical en dépend, sinon prochain trimestre.
- CVE critique activement exploitée (listée au catalogue CISA KEV ou exploit public) : mitigation sous 24 h (désactivation de la fonction, règle WAF, retrait de la route), correctif déployé sous 72 h, même s'il s'agit d'un majeur.
- images de base (`FROM`) : reconstruction au moins mensuelle, et sous 72 h en cas de CVE critical dans la couche système.

## Qui décide
| Type de montée | Décide | Valide | Trace |
|---|---|---|---|
| Patch | Développeur de permanence | CI verte (aucun nouveau finding high ou critical) | Pull request |
| Mineure | Développeur propriétaire du module | Relecture d'un second développeur | Pull request + changelog lu |
| Majeure | Référent technique | Responsable produit (impact fonctionnel) | Ticket + ADR si l'API change |
| CVE critique exploitée | Responsable sécurité | Référent technique, puis responsable produit a posteriori | Ticket d'incident + pull request |
| Risque accepté (pas de correctif) | Responsable produit, sur avis du responsable sécurité | Revue à chaque checkpoint et au plus tard sous 90 jours | Exclusion datée dans l'outil + justification écrite |

## Lien MCO-MCS
Le MCO (maintien en condition opérationnelle) couvre déjà la disponibilité du service : sauvegardes, supervision, montées de version de Node.js imposées par la fin de support.
Le MCS (maintien en condition de sécurité) ajoute la veille de vulnérabilités sur les dépendances et les images, même quand rien ne casse : la base Trivy de `gcr.io/distroless/nodejs24-debian13` est passée de debian 13.5 (image publiée v20.1.1) à 13.7 sans changer une ligne du `Dockerfile`.
Ce temps est budgété sur le run (et non sur le projet) : une demi-journée par sprint réservée aux montées de version et au triage des runs `supply-chain`.

## Cas traité aujourd'hui

**Chaîne** : `juice-shop` → `pdfkit 0.11.0` (directe, `"pdfkit": "^0.11.0"`) → `crypto-js 3.3.0` (transitive).

| Élément | Valeur relevée le 2026-10-05 |
|---|---|
| Paquet vulnérable | `crypto-js 3.3.0`, jamais choisi par nous |
| Identifiants | `GHSA-xwcq-pm8m-c4vf` / CVE-2023-46233 (PBKDF2 1 000 fois plus faible que prévu), `GHSA-rg76-677x-56q9` (entropie insuffisante) |
| Sévérité | critical (npm audit), CVSS 9.1 et 9.0 (osv-scanner) |
| Correctif disponible | `"fixAvailable": { "name": "pdfkit", "version": "0.20.2", "isSemVerMajor": true }` |
| Majeur ? | Oui : en 0.x, chaque mineure peut casser l'API, et on saute 9 versions (0.11 → 0.20) |

**Analyse d'exploitabilité.** Le seul appel est `routes/order.ts:42-43`, qui crée le PDF de confirmation de commande avec `new PDFDocument()` **sans option**. Dans `pdfkit 0.11.0`, `crypto-js` ne sert qu'à deux choses : un MD5 pour l'identifiant du document, et le chiffrement RC4/AES du PDF quand on fournit `userPassword` ou `ownerPassword`. pdfkit n'appelle jamais `PBKDF2` (CVE-2023-46233), et le générateur aléatoire faible (`WordArray.random`) ne sert qu'au chiffrement, que nous n'activons pas. Aucune donnée contrôlée par l'attaquant n'atteint le code vulnérable.

**Décision.** Ce n'est pas une CVE critique exploitable chez nous : elle ne relève donc pas de la règle des 72 h. On traite une **montée majeure planifiée** :
- **qui décide** : le référent technique, avec la validation du responsable produit, car le PDF de commande est une fonction visible par le client ;
- **critère** : le chemin vulnérable n'est pas atteignable aujourd'hui, mais un futur ajout d'options de chiffrement le rendrait atteignable. Un paquet non maintenu ne doit donc pas rester en place ;
- **délai** : montée vers `pdfkit 0.20.2` sous 30 jours, avec un test qui compare le PDF généré avant et après. En attendant, risque accepté et tracé jusqu'au 2026-11-04, avec une interdiction d'utiliser les options `userPassword` et `ownerPassword` avant la montée.

**Autre point relevé dans le même run, à traiter en priorité** : `jsonwebtoken ≤ 8.5.1` (critical, `GHSA-c7hr-j4mj-j2w6` et autres, correctif `9.0.3` majeur). Contrairement à `crypto-js`, cette dépendance est **directe** et se trouve sur le chemin d'authentification. Elle touche donc la menace **S1** du threat model et l'événement redouté **ER1** : c'est un candidat à la règle des 72 h.

**Écart avec les chiffres de référence du TP** (relevés le 2026-09-08) : `npm audit` remonte aujourd'hui 58 paquets (7 critical, 28 high, 20 moderate, 3 low) contre 48, et `osv-scanner` 101 vulnérabilités sur 40 paquets contre 79 sur 30. Le lock file n'a pas bougé : c'est la base d'avis qui a évolué. C'est exactement ce que la veille MCS doit absorber.
