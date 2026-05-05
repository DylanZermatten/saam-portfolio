# Audit complet du notebook vs. descriptif du projet

---

## CE QUI EST CORRECT ✅

### Partie I — Données et nettoyage

| Point | PDF | Notebook | Statut |
|-------|-----|----------|--------|
| Suppressions (ISINs vides) | Supprimer de tous les tableaux | Cellule 8-10 ✓ | ✅ |
| Délistings | Prix → 0 = -100% au mois de clôture | Cellule 18-19 avec parsing du nom ✓ | ✅ |
| Valeurs manquantes intermédiaires | Forward-fill | `ffill(axis=1)` cellule 21 ✓ | ✅ |
| Prix bas (RI < 0.5) | Traiter comme NaN | Cellule 14 ✓ | ✅ |
| Prix stale (>50% retours = 0) | Exclure si proportion dépasse seuil | Condition C5, denominator = τ = 120 ✓ | ✅ |
| Retours simples | $R_{i,t} = P_{i,t}/P_{i,t-1} - 1$ | Cellule 16 ✓ | ✅ |

### Partie I — Construction et optimisation

| Point | PDF | Notebook | Statut |
|-------|-----|----------|--------|
| Investment set : données carbone obligatoires | Oui | `cond3` = scope1 + revenue non-NaN ✓ | ✅ |
| Fenêtre d'estimation | τ = 120 mois | `tau=120` ✓ | ✅ |
| QP min-variance | min α'Σα s.t. 1'α=1, α≥0 | `solve_min_variance` ✓ | ✅ |
| Weight drift (buy-and-hold) | α_{i,t+k} = α_{i,t+k-1} × (1+R_{i,t+k})/(1+R_{p,t+k}) | Cellule 32 ✓ | ✅ |
| Poids VW | w_{i,t} = Cap_{i,t} / ΣCap_{j,t} pour calculer R_{t+1} | `prev_cols[-1]` = cap du mois précédent = formule exacte du PDF ✓ | ✅ |
| Période de performance | Jan 2014 – Déc 2025, T = 144 mois | ✓ | ✅ |

### Partie II — Métriques carbone

| Point | PDF | Notebook | Statut |
|-------|-----|----------|--------|
| CI (unités) | E_{i,Y} / (Rev_{i,Y}/1000) [tCO₂/MUSD] | `CI = scope1 / (rev/1000)` ✓ | ✅ |
| WACI | Σ α_i · CI_{i,Y} | `compute_waci` ✓ | ✅ |
| CF portefeuille | Σ α_i · E_{i,Y}/Cap_{i,Y} | `compute_cf` ✓ | ✅ |
| CF VW | ΣE_{i,Y} / ΣCap_{i,Y} | `compute_cf_vw` ✓ | ✅ |
| P_oos^(mv)(0.5) | CF ≤ 0.5 × CF^(mv) | `solve_mv_carbon` ✓ | ✅ |
| P_oos^(vw)(0.5) | min TE s.t. CF ≤ 0.5 × CF^(vw) | `solve_te_carbon` ✓ | ✅ |
| Net-zero | (1−θ)^(Y−Y₀+1) × CF_{Y₀}^(vw) | `(1-NZ_THETA)**(Y-2013+1) * cf_base` — IDENTIQUE AU PDF ✓ | ✅ |

> **Note importante** : la formule net-zero avait été initialement suspectée d'avoir un exposant décalé. Après lecture du PDF §4.1, la formule exacte est $(1-\theta)^{Y-Y_0+1}$ avec $Y_0=2013$, ce qui donne $0.9^1$ pour Y=2013. Le notebook est parfaitement correct.

---

## DÉVIATIONS ACCEPTABLES ⚠️

### 1. Seuil minimum d'observations
- **PDF** : "e.g., less than 3 years" (= 36 mois)
- **Notebook** : 60 mois (5 ans), justifié pour la stabilité de la matrice de covariance
- **Impact** : réduit l'investment set, notamment en 2013. Acceptable car le PDF dit "e.g."

### 2. Estimateur de covariance — Ledoit-Wolf
- **PDF formule** : Σ_Y = (1/τ) Σ (R_{t-k} − μ̂)(R_{t-k} − μ̂)'
- **Notebook** : Ledoit-Wolf shrinkage (sklearn)
- **Justification** : À partir de Déc 2017, N > 453 > τ = 120, donc la covariance empirique est rang déficient. Sans LW, le problème TE était dégénéré (P_vw(0.5) ≡ P_vw(NZ) pour tous les poids dans ker(Σ̂)). La résolution est documentée en cellule 57.
- **Verdict** : justification solide, mais doit être clairement expliqué dans le rapport.

### 3. Imputation des retours manquants
- Le PDF ne précise pas comment traiter les NaN dans le calcul de μ et Σ
- Le notebook impute avec la moyenne propre à chaque firme (pas 0)
- Choix raisonnable, à mentionner dans le rapport.

---

## PROBLÈMES À SIGNALER 🚨

### Problème 1 — WACI_mv50 > WACI_mv (anomalie d'interprétation)

| Année | WACI_mv | WACI_mv50 | CF_mv | CF_mv50 |
|-------|---------|-----------|-------|---------|
| 2015 | 3147.9 | **3184.8** ↑ | 475.2 | 237.6 |
| 2016 | 4400.9 | **4487.1** ↑ | 371.3 | 185.6 |
| 2021 | 1739.9 | **1814.8** ↑ | 655.0 | 327.5 |

Le portefeuille contraint $P_{oos}^{(mv)}(0.5)$ a une WACI plus haute que l'unconstrained dans 3 années sur 12. Ce n'est pas un bug : la contrainte porte sur CF (E/Cap) et non sur WACI (E/Rev). Le solver peut trouver des firmes à bas E/Cap mais haut E/Rev (firmes à high price-to-sales) qui satisfont la contrainte CF tout en élevant la WACI. Cela doit être expliqué dans le rapport car c'est contre-intuitif.

### Problème 2 — Valeurs WACI_mv extrêmes et très volatiles

| Année | WACI_mv | WACI_vw | Ratio |
|-------|---------|---------|-------|
| 2013 | 1482 | 193 | 7.7× |
| 2016 | **4401** | 447 | **9.8×** |
| 2018 | **4144** | 397 | **10.4×** |
| 2023 | 261 | 206 | 1.3× |

Les valeurs 2016-2018 suggèrent que le portfolio MV concentre l'essentiel du poids sur 1-2 firmes extrêmement carboniques (type utilities électriques japonaises — voir Table T3 avec ELEC.POWER DEV. à CI=7599 tCO₂/MUSD). C'est mathématiquement cohérent (ces firmes ont une très faible volatilité) mais devrait être vérifié en inspectant les poids effectifs pour 2016 et 2018, et commenté dans le rapport comme une limitation de la stratégie MV non-contrainte.

### Problème 3 — WACI_vw double entre 2014 et 2016

| Année | WACI_vw |
|-------|---------|
| 2013 | 193 |
| 2014 | 177 |
| 2015 | **349** |
| 2016 | **447** |
| 2017 | 521 |

Le WACI du benchmark VW quasi-double entre 2014 et 2016. Cela peut refléter soit :
- L'élargissement de l'investment set (plus de firmes carboniques éligibles avec les données Scope 1 dès 2015-2016)
- Des changements de reporting dans la base Datastream

À investiguer : comparer les investment sets 2014 vs 2015 pour voir si de nouvelles firmes très carboniques y entrent.

### Problème 4 — Mineure : incohérence `compute_cf` vs `compute_cf_vw`

Pour les firmes avec Cap_i = NaN dans le set :
- `compute_cf(alpha_vw, ...)` → contribution = 0 (poids VW = 0 car `fillna(0)`)
- `compute_cf_vw(...)` → inclut E_i au numérateur mais pas Cap_i au dénominateur → surestime légèrement CF_vw

Impact pratique : probablement négligeable si les données de market cap annuel sont complètes pour les firmes ayant du Scope 1. À documenter.

### Problème 5 — Performance quasi-identique de P_vw(0.5) et P_vw(NZ)

| Portefeuille | Sharpe | Cum. Return |
|-------------|--------|-------------|
| P^(vw) | 0.586 | 120.66% |
| P_oos^(vw)(0.5) | 0.585 | 120.33% |
| P_oos^(vw)(NZ) | 0.583 | 120.04% |

Les trois sont quasi-indiscernables en termes financiers. C'est un résultat réel (le TE par rapport au VW reste faible même avec la contrainte carbone) mais doit être interprété : la contrainte carbone n'a presque pas coûté de performance dans cette période, ce qui est un argument de vente fort pour ces stratégies.

---

## RÉCAPITULATIF GLOBAL

| Catégorie | Nb. de points |
|-----------|--------------|
| ✅ Conforme au PDF | 17 |
| ⚠️ Déviation acceptable (justifiée) | 3 |
| 🚨 À documenter/investiguer | 5 |
| ❌ Erreur réelle | 0 |

Il n'y a pas d'erreur de code fondamentale. L'implémentation est solide et conforme au PDF sur tous les points critiques. Les points à traiter sont essentiellement d'ordre interprétatif (WACI vs CF, concentration MV) et doivent figurer dans la section "limitations et robustesse" du rapport.
