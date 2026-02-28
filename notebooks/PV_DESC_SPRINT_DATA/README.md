# PV_DESC_SPRINT_DATA – Notebooks d'analyse Gaia Star Saturation

**Auteur** : Sylvie Dagoret-Campagne  
**Date** : 2026-02-28  
**Contexte** : Sprint DESC PV – Analyse de la saturation des étoiles brillantes Gaia dans les données DP1 ComCam (ECDFS, bandes g et r).

---

## Objectif scientifique

Étude de l'offset photométrique entre les magnitudes LSST/ComCam et le catalogue de référence Gaia pour les étoiles brillantes (saturation, non-linéarité du détecteur). L'analyse croise les sources détectées dans `dp1.Source` avec le catalogue The Monster et les métadonnées observationnelles (`Visit`, `CcdVisit`) pour identifier les conditions d'observation corrélées aux offsets.

---

## Fichiers

### Notebooks RSP (exécutés sur la Rubin Science Platform)

| Notebook | Description |
|----------|-------------|
| `gaiaStarsSaturation_original.ipynb` | Version originale du sprint, accès via `lsst.rsp.get_tap_service` et `lsst.daf.butler` |
| `gaiaStarsSaturation_final.ipynb` | Version finale RSP avec corrections et résultats |
| `gaiaStarSaturation_workaround.ipynb` | Version RSP avec workaround (accès The Monster via Butler + correction RANSAC par visite) |

Ces notebooks **requièrent la RSP** et les packages LSST internes (`lsst.daf.butler`, `lsst.rsp`, `lsst.utils.plotting`, etc.).

---

### Notebook local (exécutable hors RSP)

| Notebook | Description |
|----------|-------------|
| `gaiaStarsSaturation_local_api.ipynb` | **Version locale** — accès via `pyvo` + token `RSP_TOKEN`, sans Butler ni packages LSST internes |

#### Ce que fait `gaiaStarsSaturation_local_api.ipynb`

1. Connexion au TAP via `pyvo` (token `RSP_TOKEN`)
2. **Requête principale** : JOIN ADQL `Source × Visit × CcdVisit` en une seule requête côté serveur
3. Chargement optionnel du CSV `table_for_visu_inspect.csv` (données Gaia pré-exportées depuis la RSP)
4. Préparation des données (coupure SNR, conversion flux → magnitude AB)
5. Correction photométrique RANSAC par visite (si colonnes Gaia disponibles)
6. Analyse de corrélation entre `delta_mag` et métadonnées observationnelles (`airmass`, `seeing`, `zeroPoint`, `skyBg`…)
7. Fonctions utilitaires : `query_star_lightcurve_joined()`, `binned_running_median_mad()`

#### Différences vs version RSP

| RSP | Local |
|-----|-------|
| `lsst.rsp.get_tap_service` | `pyvo` + token |
| `lsst.daf.butler` | Non disponible |
| `lsst.utils.plotting` | `matplotlib` standard |
| 3 requêtes séparées + `pd.merge` Python | **1 seul JOIN ADQL** |
| Visualisation Firefly (`afwDisplay`) | Non disponible |

---

## Données

### `table_for_visu_inspect.csv`

Table pré-exportée depuis la RSP contenant les sources `dp1.Source` (bande g, ECDFS) croisées avec Gaia via The Monster. Colonnes principales :

| Colonne | Description |
|---------|-------------|
| `sourceId` | Identifiant de source DP1 |
| `visit` | Identifiant de visite |
| `detector` | Identifiant de détecteur |
| `ra`, `dec` | Coordonnées (degrés) |
| `psfFlux`, `psfFluxErr` | Flux PSF et son erreur (nJy) |
| `band` | Filtre photométrique |
| `mag` | Magnitude AB |
| `gaia_g_mag`, `gaia_bp_mag`, `gaia_rp_mag` | Magnitudes Gaia |
| `gaia_color` | Couleur Gaia BP–RP |
| `delta_mag` | `mag – gaia_g_mag` |

---

## Pré-requis locaux

```bash
pip install pyvo astropy numpy pandas scipy scikit-learn matplotlib seaborn requests
export RSP_TOKEN="votre_token_ici"
```

Voir le [README parent](../README.md) pour les détails de connexion et les pièges de casse ADQL.
