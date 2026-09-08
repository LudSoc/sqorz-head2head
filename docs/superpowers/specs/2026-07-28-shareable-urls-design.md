# URLs partageables — Design (h2h_stats)

Date : 2026-07-28

Voir spec complète : `club_stats/docs/superpowers/specs/2026-07-28-shareable-urls-design.md`

## Paramètres URL

`?a=<normKeyA>&b=<normKeyB>`

## Fonctions à ajouter

```js
function parseUrlState()          // lit ?a= et ?b=
function buildUrl(keyA, keyB)     // construit l'URL
function pushUrlState()           // pushState quand h2h lancé
function replaceUrlState()        // replaceState si un pilote effacé
```

## Restauration

1. Lire `?a=` et `?b=` après chargement de l'index
2. Les deux trouvés → sélectionner A + B, déclencher le h2h
3. Un seul → sélectionner ce pilote uniquement
4. Aucun → état vide
