# Résultats en Direct — Ligue Centre-Val de Loire

Application web de gestion et d'affichage en direct des résultats de compétitions de tir sportif, construite autour de **Firebase Realtime Database** (données en temps réel) et **Firebase Authentication** (protection des espaces d'administration et de saisie).

Le site public d'affichage se met à jour automatiquement à chaque saisie, sans rechargement de page : idéal pour un écran (TV, vidéoprojecteur, tablette) posé sur le lieu de la compétition.

## Origine du projet

Ce projet est basé sur le travail de la **Ligue de Lorraine de Tir** :
👉 [liguelrtir-lr/resultats-tir](https://github.com/liguelrtir-lr/resultats-tir) — *tous les résultats en ligne*

Il a été repris et adapté pour la Ligue Centre-Val de Loire, avec un ensemble d'améliorations fonctionnelles et techniques détaillées ci-dessous.

## Améliorations apportées par rapport au projet d'origine

- **Affichage détaillé des séries** — chaque série de tir est désormais affichée individuellement (et non plus seulement le score total), avec un nombre de séries calculé automatiquement selon l'épreuve (École de Tir, ISSF, TAR, Armes Anciennes...).
- **Import CSV depuis ISISWEB** — import direct des engagements et des résultats à partir d'un export CSV ISISWEB (séparateur `;`), avec reconnaissance automatique des colonnes (`Nom`, `Prenom`, `Association Nom`, `Catégorie E`, `Epreuve`, `Place`, `Série 1` à `Série N`, `Nombre mouches`...) et fusion avec les tireurs déjà engagés.
- **Mise à jour des noms des épreuves et des catégories** — les codes bruts issus d'ISISWEB (ex : `S3`, `MG`, `CF`...) sont automatiquement normalisés vers des libellés lisibles (`Senior 3`, `Minime Garçon`, `Cadet Fille`...), et les intitulés d'épreuves ont été harmonisés (École de Tir, ISSF 10 m, ISSF 25/50 m, TAR, MLAIC).
- **Gestion des tireurs DNS, DSQ, DNF et HM** — un statut par tireur (Présent / DNS / DSQ / DNF / HM) permet de les exclure proprement du classement (renvoyés en bas de liste, score neutralisé) sans les supprimer de la compétition (DNS et DSQ) ou bien de ne leur attribuer aucune place (DNF et HM) sans les supprimer de la compétition.
- **Affichage responsive multi-supports** — remise à plat complète du CSS (grille de résultats, en-têtes collants, tableaux) pour un rendu propre aussi bien sur écran TV/vidéoprojecteur que sur tablette et téléphone (tableaux à défilement horizontal, polices et espacements adaptatifs, recalcul dynamique en JavaScript des zones "collantes" pour éviter tout chevauchement de texte selon la taille d'écran).
- **Curseur de vitesse de défilement** — ajout d'un réglage à la volée (slider) dans la fenêtre d'aide de la page d'affichage, en complément du réglage classique par paramètre d'URL (`&speed=`) ou par configuration Firebase.
- **Affiche QR Code imprimable (PDF/A4)** — depuis le tableau de bord admin, génération à l'impression d'une affiche A4 portrait prête à afficher sur le lieu de compétition (logo, nom de la compétition, QR code en grand format, URL de secours).
- **Nom de la compétition visible sur la page de saisie** — affichage en temps réel du nom de la compétition en cours de saisie, en plus de son code technique.
- **Tri de classement affiné** — en cas d'égalité de score, départage automatique par nombre de mouches puis par comparaison des séries (de la dernière à la première).

## Structure du projet

| Fichier | Rôle |
|---|---|
| `index_admin.html` | Tableau de bord administrateur : création/suppression de compétitions, génération et impression des QR codes, accès aux pages de saisie et d'affichage. Accès protégé par Firebase Authentication. |
| `saisie_multicomp.html` | Interface de saisie des résultats : ajout manuel de tireurs, import CSV (ISISWEB), saisie des séries/mouches/statut. Accès protégé par Firebase Authentication. |
| `affichage_multicomp.html` | Page publique d'affichage des résultats en direct, avec classement automatique, défilement automatique et QR code d'accès. Ne nécessite aucune connexion. |
| `logo.png` | Logo du club/de la ligue, utilisé sur l'ensemble des pages (à fournir). |

## Fonctionnement général

Les données sont stockées dans Firebase Realtime Database, sous la structure suivante :

```
clubs/
  └─ {CLUB_KEY}/
       └─ {CODE_COMPETITION}/
            ├─ config/        → { name, type, scrollSpeed }
            └─ shooters/       → liste des tireurs
                 └─ {id}/      → { lastName, firstName, club, discipline,
                                    category, status, score, series, mouches }
```

Chaque page écoute les changements en direct via `db.ref(...).on('value', ...)`, ce qui permet une mise à jour instantanée de l'affichage public dès qu'une saisie est enregistrée.

## Configuration

Chaque page contient, en tête de son script, un bloc `CONFIG` à personnaliser pour un nouveau club/une nouvelle ligue :

```js
const CONFIG = {
  clubName: "LIGUE CENTRE-VAL DE LOIRE",   // Nom affiché
  clubKey:  "LIGUE_CENTRE_VDL",            // Dossier Firebase (sans espaces ni accents)
  githubBase: "https://.../resultats-tir-centre", // Base d'URL pour les QR codes (index_admin.html)
  firebase: { /* clés du projet Firebase */ }
};
```

## Disciplines et épreuves gérées

- **ISSF 10 m** et **ISSF 25/50 m**
- **EDT** — École de Tir
- **TAR** — Tir aux Armes Réglementaires
- **MLAIC** — Armes Anciennes

Le nombre de séries attendu par épreuve (3, 4, 6, 8, 10 selon les cas) est calculé automatiquement à partir du code de l'épreuve, aussi bien à l'affichage qu'à l'import CSV.

## Technologies utilisées

- HTML / CSS / JavaScript (vanilla, aucun framework)
- [Firebase Realtime Database](https://firebase.google.com/docs/database) — stockage et synchronisation en temps réel
- [Firebase Authentication](https://firebase.google.com/docs/auth) — protection des espaces admin et saisie
- [QRCode.js](https://github.com/davidshimjs/qrcodejs) — génération des QR codes d'accès
- Polices [Rajdhani](https://fonts.google.com/specimen/Rajdhani) et [Barlow](https://fonts.google.com/specimen/Barlow) (Google Fonts)
- Hébergement statique compatible GitHub Pages

## Déploiement

1. Créer un projet Firebase (Realtime Database + Authentication par email/mot de passe activés).
2. Renseigner le bloc `CONFIG` (clés Firebase, `clubKey`, `githubBase`) dans les trois fichiers HTML.
3. Déposer `logo.png` à la racine du projet.
4. Héberger l'ensemble sur GitHub Pages (ou tout hébergement statique équivalent).
5. Créer un compte administrateur dans Firebase Authentication pour accéder à `index_admin.html` et `saisie_multicomp.html`.

## Licence

Projet dérivé de [liguelrtir-lr/resultats-tir](https://github.com/liguelrtir-lr/resultats-tir). Se référer à la licence du dépôt d'origine pour les conditions de réutilisation.