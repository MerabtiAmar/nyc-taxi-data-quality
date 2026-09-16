# Qualité des données : évaluation et correction d'un jeu NYC Taxi

Projet du cours *Data Quality* du Master 2 DCI (Université Paris Cité, 2025–2026). On part d'un extrait de courses de taxi new-yorkaises, on y injecte des erreurs réalistes, puis on **mesure** la qualité et on la **répare** avec un pipeline reproductible.

## Démarche

1. **Scénarios métier** ([docs/scenarios_metier.pdf](docs/scenarios_metier.pdf)) : la qualité dépend de l'usage. Deux profils pondèrent différemment les attributs :
   - **chauffeur** (déclaration fiscale) : montant, dates et moyen de paiement sont critiques ;
   - **passager** (contestation d'une surfacturation) : distance et coordonnées GPS deviennent critiques.
2. **Profilage** (`profiling.ipynb`) : rapport automatique avec ydata-profiling.
3. **Évaluation** (`assessment.ipynb`) : score par attribut et par scénario sur quatre dimensions.
   - **Complétude** : taux de valeurs présentes, pondéré par la criticité de l'attribut.
   - **Exactitude** : formats valides, coordonnées à l'intérieur de New York (limites des arrondissements via geopandas/shapely), valeurs plausibles.
   - **Cohérence** : relations entre attributs, par exemple la distance déclarée comparée à la distance à vol d'oiseau (Haversine), ou la dépose après la prise en charge.
   - **Unicité** : détection des doublons.
4. **Amélioration** (`improvement.ipynb`) :
   - normalisation des formats de dates hétérogènes (`6 janvier 2023 à 9h08`, `19-01-2023 21h07`, `11-01-2023 20 heure 24 minutes`) ;
   - suppression des unités et devises parasites (`3.44 Miles`, `£ 5.84`, `12.83¥`) ;
   - suppression des doublons ;
   - imputation des distances et des tarifs manquants ou négatifs par régression linéaire (corrélation tarif/distance r ≈ 0,78) ;
   - correction des coordonnées GPS hors de la ville ;
   - correction des incohérences temporelles et imputation des attributs catégoriels.

La liste des erreurs injectées est décrite dans [`erreurs.txt`](erreurs.txt).

## Résultats (jeu corrompu, avant → après correction)

| Dimension | Chauffeur | Passager |
|---|---|---|
| Complétude | 0,69 → **0,93** | 0,70 → **0,94** |
| Exactitude | 0,63 → **0,91** | 0,64 → **0,93** |
| Cohérence | 0,86 → **1,00** | 0,83 → **1,00** |
| Unicité | 1,00 → 1,00 | 1,00 → 1,00 |

Le rapport complet est disponible ici : [docs/rapport_data_quality.pdf](docs/rapport_data_quality.pdf).

## Fichiers

| Fichier | Contenu |
|---|---|
| `Taxi.csv` | extrait de référence |
| `Taxi_modifié.csv` → `Taxi_corrigé.csv` | premier jeu d'erreurs et sa correction |
| `df_corrupted.csv` → `Taxi_corrupted_corrigé.csv` | jeu corrompu final et sa correction |
| `tl_2023_36061_areawater.*` | plans d'eau de Manhattan (TIGER/Line, US Census Bureau) |

## Exécution

```bash
pip install pandas numpy scikit-learn geopandas shapely ydata-profiling jupyter
jupyter notebook assessment.ipynb
```

Les limites des arrondissements de New York sont téléchargées depuis un service ArcGIS public : une connexion Internet est nécessaire.

## Licence

Code distribué sous [licence MIT](LICENSE).

## Auteurs

**Amar Merabti** — Master 2 DCI, Université Paris Cité.
