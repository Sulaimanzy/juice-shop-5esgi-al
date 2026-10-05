# Triage des findings SAST et secrets - séance 3

Les verdicts ci-dessous ont été établis après lecture des lignes signalées et de leur contexte. Les numéros S1 et I1 renvoient au tableau STRIDE de `docs/security/threat-model.md`.

| Fichier:ligne | Règle | CWE | Sévérité | Vrai ou faux positif | Action décidée |
|---|---|---|---|---|---|
| `routes/login.ts:34` | `express-sequelize-injection` | CWE-89 | Error | **Vrai positif à corriger** : `req.body.email` est concaténé dans une requête SQL. L'appel à `security.hash()` présent sur la même ligne est un second problème, mais il n'est pas la cause de ce finding. | Aucune exigence STRIDE ne couvre explicitement l'injection SQL : lacune constatée. Créer un ticket H pour remplacer la concaténation par une requête paramétrée en S5 ; le MD5 est déjà couvert séparément par I1. |
| `lib/insecurity.ts:150` | `juiceshop-hardcoded-private-key` | CWE-798 | Error | **Vrai positif à corriger** : la ligne 150 ne contient pas de littéral, mais elle utilise `privateKey`, dont la valeur privée RSA est codée en dur à la ligne 21. La propagation de constante rend le finding valable. | Rattaché à S1. Externaliser la clé vers un gestionnaire de secrets, permettre sa rotation et ajouter une détection de secrets dans la CI. Correctif planifié en S5. |
| `lib/insecurity.ts:41` | `juiceshop-weak-hash-md5` | CWE-327 | Error | **Vrai positif à corriger** : la fonction utilisée pour les mots de passe applique MD5, algorithme rapide et inadapté au stockage de mots de passe. | Rattaché à I1. Planifier une migration vers Argon2id ou bcrypt avec sel individuel et test SAST empêchant le retour de MD5. |
| `routes/login.ts:64` | `generic-api-key` (gitleaks) | CWE-798 | Error | **Vrai positif à ne pas traiter dans ce fork pédagogique** : la valeur en base64 est bien un mot de passe écrit en dur, mais elle appartient volontairement au code du challenge `oauthUserPasswordChallenge` et ne protège aucun service de production. | Risque accepté pour conserver le challenge. Aucune exigence STRIDE associée ; exclusion ciblée et datée par empreinte dans `.gitleaksignore`, sans désactiver la règle `generic-api-key`. |

## Décisions et suivi

- Les trois vrais positifs à corriger feront l'objet d'un ticket ou d'un finding d'audit en S5/S8.
- L'exclusion gitleaks est limitée au credential pédagogique de `routes/login.ts:64` et ne masque pas la clé RSA détectée dans `lib/insecurity.ts:21` ; le job reste donc rouge comme demandé.
- Date du triage : 2026-09-10.

## Complément séance 4 (2026-10-05)

| Fichier:ligne | Règle | CWE | Sévérité | Vrai ou faux positif | Action décidée |
|---|---|---|---|---|---|
| `Dockerfile.ci:6` (commit `2fd9556`) | `generic-api-key` (gitleaks, scan de l'historique complet) | CWE-798 | Error | **Vrai positif volontaire** : `ENV JWT_SECRET=...` grave le secret dans un layer de l'image, lisible par `docker history` et `trivy image`. Le Dockerfile est volontairement dégradé, c'est le support du TP4. | **Pas d'exclusion**, le job reste rouge comme le demande le TP4. Le secret disparaît avec `Dockerfile.hardened`, qui ne contient aucun `ENV` ni `ARG` sensible : le secret est injecté au démarrage par l'orchestrateur. Le secret est considéré comme compromis puisqu'il est dans l'historique public : il ne doit être réutilisé nulle part. |

- Sur le push du merge de la PR #2, le step `gitleaks-action` (contrôle du push) affiche « No leaks detected » : il ignore les commits de fusion et n'a donc rien scanné. Seul le step « historique complet » (`fetch-depth: 0`) a attrapé ce secret. Cela justifie de garder les deux steps.
