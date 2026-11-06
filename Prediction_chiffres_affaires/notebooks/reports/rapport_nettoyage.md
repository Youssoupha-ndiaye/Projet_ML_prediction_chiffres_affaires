# Tâche 1 – Rapport de nettoyage des données (Rossmann Store Sales)

Responsable : Ibrahima Diop – Groupe 4, P1 IA, Sonatel Academy

Fichier analysé : `rossmann_stores_sales.csv`. Résultat : `data_clean.csv` (803 252 lignes × 16 colonnes, 0 valeur manquante, 0 doublon). Le dictionnaire des données est dans `reports/dictionnaire_donnees.csv`. Le code est dans `nettoyage_rossmann.ipynb`.

## 1. Structure du fichier brut

| Élément | Valeur |
|---|---|
| Lignes × colonnes | 810 056 × 16 |
| Période (Date) | du 2013-01-01 au 2014-12-23 (722 dates distinctes) |
| Nombre de magasins (Store) | 1 115 (de 1 à 1 115) |
| Doublons exacts | 6 000 (identiques aux doublons sur Store + Date) |
| Date à la lecture | texte (non datée) |
| Colonnes sans valeur manquante | Store, Date, DayOfWeek, Promo, SchoolHoliday, StoreType, Assortment, Promo2 |

Valeurs manquantes brutes (cellules vides) : Sales 814 ; StateHoliday 8 099 ; CompetitionDistance 53 992 ; CompetitionOpenSinceMonth 118 405 ; CompetitionOpenSinceYear 118 405 ; Promo2SinceWeek 509 983 ; Promo2SinceYear 509 983 ; PromoInterval 509 983.

Le fichier ne contient pas de colonne `Open` : on ne peut pas distinguer directement un magasin fermé d'une vente nulle.

## 2. Problèmes trouvés, correction et justification (colonne par colonne)

Les nombres de lignes concernées sont donnés sur le fichier brut ; entre parenthèses, le nombre de lignes réellement traitées une fois les doublons supprimés (et les Sales manquantes retirées).

| Colonne / sujet | Problème trouvé | Lignes concernées | Correction | Pourquoi |
|---|---|---|---|---|
| Lignes dupliquées | Lignes strictement identiques | 6 000 | Suppression (810 056 → 804 056 lignes) | Un même couple Store + Date ne doit apparaître qu'une fois ; les doublons gonfleraient artificiellement certaines ventes |
| Date | Stockée en texte | 810 056 | Conversion en datetime (format AAAA-MM-JJ, 0 date illisible) | Permet le tri chronologique et les calculs de dates |
| DayOfWeek | Cohérence avec Date (lundi = 1 … dimanche = 7) | 0 incohérence | Aucune | Colonne déjà cohérente |
| StoreType | 20 écritures différentes (a, A, " a", "a "…) | 63 696 lignes modifiées après dédoublonnage | `strip()` + minuscules → 4 modalités : a, b, c, d | Une même catégorie ne doit avoir qu'une seule écriture |
| Assortment | 15 écritures différentes (espaces, majuscules) | 64 591 lignes modifiées après dédoublonnage | `strip()` + minuscules → 3 modalités : a, b, c | Idem |
| StateHoliday | Cellules vides | 8 099 (7 237) | Remplacées par la valeur majoritaire du même jour : 11 lignes → `c` (2013-12-25), les autres → `0` | Un jour férié est une information de calendrier : chaque date de ligne manquante possède des valeurs connues pour d'autres magasins |
| Sales – valeurs manquantes | Cible vide | 814 (804) | Suppression des lignes (804 056 → 803 252) | On n'invente pas la variable cible |
| Sales – valeurs négatives | Ventes < 0 (min −18 554) | 2 875 (2 860) | Passage en valeur absolue | Les valeurs absolues des négatives ont la même distribution que les positives (minimum 2 576 contre 2 574, médiane 6 660,5 contre 6 689) : il s'agit d'erreurs de signe |
| Sales – zéros | Ventes = 0 | 80 767 (80 190) | Conservées | Valeur possible, toutes situées un dimanche (DayOfWeek = 7) ; colonne Open absente, à signaler aux membres suivants |
| Sales – outliers | Au-dessus de Q3 + 1,5 × IQR = 13 259 (calcul sur les ventes > 0) | 6 358 lignes après nettoyage (max 21 387) | Conservés | Valeurs plausibles et non impossibles ; les supprimer retirerait de l'information |
| CompetitionDistance | Cellules vides ; valeur unique par magasin (au maximum 1 valeur distincte par Store) | 53 992 (52 767) | 14 594 lignes complétées avec la valeur connue du même magasin ; 38 173 lignes (53 magasins sans aucune valeur) complétées par la médiane des magasins : 37 583,38 | La distance est une caractéristique du magasin ; la médiane est robuste quand aucune valeur du magasin n'existe |
| CompetitionDistance – outliers | Contrôle IQR au niveau magasin | 0 | Aucune | Valeurs entre 27,60 et 74 973,32 |
| CompetitionOpenSinceMonth / Year | Cellules vides (mois et année toujours vides ensemble, 0 incohérence) | 118 405 chacune (117 412) ; 163 magasins | Remplacement par 0 (= date inconnue / sans objet) | L'absence concerne des magasins entiers : une date d'ouverture ne peut pas être déduite sans l'inventer |
| Promo2SinceWeek / Promo2SinceYear | Vides exactement quand Promo2 = 0 (0 incohérence avec Promo2 = 1) | 509 983 (505 696) | Remplacement par 0 (sans objet) | L'absence a un sens : le magasin ne participe pas à Promo2 |
| PromoInterval | Vide exactement quand Promo2 = 0 | 509 983 (505 696) | Remplacement par `Aucun` | Idem |

Anomalie signalée, non corrigée : dans StateHoliday, les valeurs `a` et `b` apparaissent chacune sur 718 des 722 dates, et `c` uniquement le 2013-12-25. Ces valeurs ne peuvent pas être vérifiées dans le fichier, elles ont donc été conservées telles quelles.

## 3. Résultat du nettoyage

| Indicateur | Avant | Après |
|---|---|---|
| Lignes | 810 056 | 803 252 |
| Colonnes | 16 | 16 |
| Doublons | 6 000 | 0 |
| Valeurs manquantes (cellules vides) | 1 829 664 | 0 |
| Écritures StoreType / Assortment | 20 / 15 | 4 / 3 |

## 4. Impact sur la distribution de Sales (avant et après)

| Statistique | Avant (brut) | Après |
|---|---|---|
| Valeurs renseignées | 809 242 | 803 252 |
| Moyenne | 6 157,5 | 6 205,4 |
| Écart-type | 3 141,6 | 3 045,4 |
| Minimum | −18 554 | 0 |
| Q1 | 4 501 | 4 524 |
| Médiane | 6 321 | 6 334 |
| Q3 | 8 137 | 8 144 |
| Maximum | 21 387 | 21 387 |
| Valeurs négatives | 2 875 | 0 |
| Valeurs égales à 0 | 80 767 | 80 190 |

La correction des signes remonte légèrement la moyenne et réduit l'écart-type (la queue négative disparaît) ; la médiane et les quartiles bougent à peine.

Les histogrammes sont dans `reports/sales_avant_apres.png`, en échelle logarithmique : en échelle linéaire, les 2 875 valeurs négatives sont invisibles à côté du pic de 80 000 ventes à 0. Sur la figure, une seconde bosse entre −18 554 et −2 576 apparaît avant le nettoyage et disparaît après ; la distribution positive reste quasi inchangée.

## 5. Points transmis aux membres suivants

- Les ventes à 0 (80 190 lignes, tous les dimanches) sont conservées : la décision de les exclure ou non de la modélisation revient au membre concerné [À COMPLÉTER PAR LE MEMBRE 2].
- Le code 0 utilisé pour CompetitionOpenSince* et Promo2Since* signifie « sans objet / inconnu » et non une vraie date.