# Rubin LSST DP1 – Notebooks d'accès local via API TAP

**Auteur** : Sylvie Dagoret-Campagne  
**Date** : 2026-02-28  
**Contexte** : Accès aux données DP1 (Data Preview 1) de Rubin/LSST depuis un environnement local, sans passer par la Rubin Science Platform (RSP), en utilisant l'API TAP avec `pyvo`.

---

## Pré-requis

```bash
pip install pyvo astropy numpy pandas scipy scikit-learn matplotlib seaborn requests
```

Le token d'accès RSP doit être stocké dans la variable d'environnement `RSP_TOKEN` :

```bash
export RSP_TOKEN="votre_token_ici"   # à ajouter dans ~/.zshrc ou ~/.bashrc
```

Le token se crée sur [rsp.lsst.io/guides/auth/creating-user-tokens.html](https://rsp.lsst.io/guides/auth/creating-user-tokens.html) avec le scope `read:image`.

---

## Notebooks

### `00_TestAccessRSPAPI.ipynb` — Test de connexion TAP

Notebook minimal de validation de l'accès au service TAP de Rubin DP1.

- Connexion authentifiée via `pyvo` + token `RSP_TOKEN`
- Liste des tables disponibles dans le schéma `dp1.*`
- Exemple de requête ADQL simple sur `dp1.Visit` (région ECDFS, bandes g et r)
- Soumission et récupération d'un job asynchrone

**À exécuter en premier** pour vérifier que l'authentification et la connexion fonctionnent.

---

### `00b_ExploreDP1Schema.ipynb` — Schéma des tables DP1 et JOINs ADQL

Notebook de référence pour explorer la structure des tables DP1 et valider les JOINs ADQL corrects.

#### Contenu

1. Récupération du schéma de chaque table via `tap_schema.columns`
2. Tables couvertes : `Source`, `Visit`, `CcdVisit`, `ForcedSource`, `DiaSource`, `Object`, `DiaObject`
3. Récapitulatif des colonnes de JOIN disponibles par table
4. Tests de validation des JOINs (avec résultats documentés)
5. Requête principale finale : **Source × Visit × CcdVisit**

#### JOINs validés

| JOIN | Clé ADQL | Statut |
|------|----------|--------|
| `Source` × `Visit` | `src.Visit = v.visit` | ✅ OK |
| `Source` × `CcdVisit` | `src.Visit = cv.VisitId` | ✅ OK |
| `Source` × `Visit` × `CcdVisit` | les deux ci-dessus | ✅ OK |
| `Object` × `ForcedSource` × `CcdVisit` | `obj.objectId = fs.objectId` puis `fs.Visit = cv.VisitId` | ✅ OK |
| `DiaSource` × `Visit` × `CcdVisit` | `dia.visit = cv.VisitId` | ⚠️ à valider |

#### Pièges de casse (ADQL est sensible à la casse)

| Table | Colonne visite | Casse exacte |
|-------|---------------|--------------|
| `dp1.Source` | `Visit` | V majuscule |
| `dp1.ForcedSource` | `Visit` | V majuscule |
| `dp1.Visit` | `visit` | v minuscule |
| `dp1.CcdVisit` | `VisitId` | camelCase |

#### Point important

`dp1.Source` **n'a pas de colonne `detector`** — le JOIN avec `CcdVisit` est donc 1-to-many (plusieurs CCDs par visite). Pour un JOIN 1-to-1 avec les métadonnées CCD, utiliser `dp1.ForcedSource` qui possède `detector`.

---

## Sous-dossier `PV_DESC_SPRINT_DATA/`

Notebooks d'analyse en cours de développement pour le sprint DESC PV.  
Voir le [README](PV_DESC_SPRINT_DATA/README.md) dédié.

---

## Références

- Documentation DP1 : [dp1.lsst.io](https://dp1.lsst.io)
- Tutoriaux TAP : [dp1.lsst.io/tutorials/api/api-102-1.html](https://dp1.lsst.io/tutorials/api/api-102-1.html)
- Tutoriaux JOINs ADQL : [dp1.lsst.io/tutorials/portal/103/portal-103-4.html](https://dp1.lsst.io/tutorials/portal/103/portal-103-4.html)
- Schéma SDM DP1 : [sdm-schemas.lsst.io/dp1.html](https://sdm-schemas.lsst.io/dp1.html)
