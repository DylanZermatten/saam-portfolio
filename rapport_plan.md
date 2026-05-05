---
title: "Portfolio Allocation with a Carbon Objective"
subtitle: "Plan du rapport — SAAM | HEC Lausanne 2026"
author:
  - "Région PAC | Scope 1 CO~2~ | Travail de groupe (3 personnes)"
date: "Mai 2026"
geometry: "margin=2.5cm"
fontsize: 11pt
linestretch: 1.3
toc: true
toc-depth: 3
numbersections: true
header-includes:
  - \usepackage{booktabs}
  - \usepackage{longtable}
  - \usepackage{float}
  - \usepackage{array}
  - \renewcommand{\arraystretch}{1.3}
---

\newpage

# Vue d'ensemble et répartition du travail

## Résumé de l'attribution

| Personne | Sections | Pages cibles |
|----------|----------|-------------|
| **A** | Introduction, Données & Méthodologie | ~9 pages |
| **B** | Partie I : Portefeuille standard, Métriques carbone | ~11 pages |
| **C** | Partie II : Portefeuilles contraints, Discussion, Limites, Conclusion | ~10 pages |
| **Total** | | **~30 pages** |

## Figures et tableaux à inclure

| Réf. | Description | Section |
|------|-------------|---------|
| Fig. 1 | Rendements cumulatifs — MV vs VW (2014–2025) | 3.1 |
| Fig. 2 | Évolution WACI — tous portefeuilles (2013–2024) | 3.2 |
| Fig. 3 | Évolution CF — tous portefeuilles (2013–2024) | 3.2 |
| Fig. 4 | Rendements cumulatifs — MV vs MV(0.5) | 4.1 |
| Fig. 5 | Rendements cumulatifs — VW vs VW(0.5) | 4.2 |
| Fig. 6 | Comparaison finale — 5 portefeuilles | 4.3 |
| Fig. 7 | Évolution des poids — MV (top firmes) | 3.1 |
| Fig. 8 | Top 10 firmes par intensité carbone (CI) | 3.2 |
| Fig. 9 | Changement de composition MV → MV(0.5), Déc. 2016 | 3.2 |
| T1 | Statistiques descriptives — données brutes | 2.2 |
| T2 | Performance — Partie I (MV vs VW) | 3.1 |
| T3 | Métriques carbone — tous portefeuilles | 3.2 |
| T4 | Performance — Partie II (5 portefeuilles) | 4.3 |

\newpage

---

# PERSONNE A — Introduction & Données (~9 pages)

## Introduction (2 pages)

### Contexte et motivation

L'intégration des critères environnementaux dans la gestion d'actifs constitue l'un des défis majeurs de la finance contemporaine. Les émissions de CO~2~ représentent un risque systémique croissant : risques de transition (réglementations carbone, taxe carbone), risques physiques (événements climatiques extrêmes affectant la valeur des actifs), et risques de réputation. Dans ce contexte, les investisseurs institutionnels cherchent à réduire l'empreinte carbone de leurs portefeuilles sans sacrifier la performance financière.

Ce projet s'inscrit dans le cadre du cours **Sustainability Aware Asset Management (SAAM)** de HEC Lausanne. Il vise à implémenter et évaluer plusieurs stratégies d'allocation carbone-consciente sur la région Pacifique (PAC), en utilisant les données d'émissions Scope 1 de CO~2~.

**Questions centrales :**

1. Un portefeuille minimum-variance hors-échantillon surpasse-t-il le benchmark pondéré par la capitalisation en région PAC ?
2. Peut-on réduire l'empreinte carbone de 50 % sans coût de performance significatif ?
3. La trajectoire net-zéro (−10 %/an) est-elle compatible avec des rendements compétitifs ?

### Structure du rapport

Le rapport est organisé comme suit : la Section 2 décrit les données et la méthodologie de nettoyage ; la Section 3 présente les résultats de la Partie I (portefeuilles standard) ; la Section 4 présente la Partie II (portefeuilles contraints en carbone) ; la Section 5 discute les limitations et les pistes d'amélioration ; la Section 6 conclut.

**Divulgation LLM :** Ce rapport a été rédigé avec l'assistance de Claude Code (Anthropic), utilisé pour le débogage du code Python, l'audit de l'implémentation par rapport au descriptif du projet, et la structuration du rapport. Tous les choix méthodologiques et les interprétations restent sous la responsabilité des auteurs.

---

## Données & Méthodologie (7 pages)

### Description des données (2 pages)

**Source :** Fichier `Sustainability.xlsx` fourni par l'enseignant, contenant des données mensuelles et annuelles pour 2 545 firmes mondiales.

**Feuilles utilisées :**

- *Static* : ISIN, noms de firmes, région d'appartenance
- *RI Monthly* : indices de rendement total (Total Return Index) mensuels, 1999–2025
- *Market Cap Monthly* : capitalisation boursière mensuelle
- *Scope1 Annual* : émissions CO~2~ Scope 1 annuelles (tCO~2~), 2002–2024
- *Revenue Annual* : revenus annuels (kUSD), 2002–2024
- *Market Cap Annual* : capitalisation boursière annuelle (kUSD), 2002–2024

**Filtrage régional :** Région PAC uniquement → **N = 786 firmes** dans l'ensemble initial.

**Fenêtre temporelle :**

- Données de marché : janvier 1999 – décembre 2025
- Données carbone : 2002–2024 (fiables à partir de ~2010)
- Analyse : décembre 2013 → décembre 2024 (rebalancement annuel), janvier 2014 → décembre 2025 (performance mensuelle, T = 144 mois)

**Tableau T1 — Statistiques descriptives (données post-nettoyage) :**

*(insérer ici : nombre de firmes par année d'investissement, distribution des rendements mensuels MV, statistiques de CI par année — min, médiane, max, 95e percentile)*

### Nettoyage des données (3 pages)

Le nettoyage suit strictement le descriptif du projet (§2), avec les étapes suivantes appliquées dans l'ordre :

**Étape C1 — Suppression des ISINs vides**
Toute firme sans ISIN valide est exclue de l'ensemble de données.

**Étape C2 — Délistings**
Les firmes dont le nom contient une date de délisting (format parsé depuis le champ *Name* de la feuille *Static*) reçoivent un rendement de −100 % au mois de clôture. Cela capture correctement la perte totale liée à la faillite ou à la fusion-absorption.

**Étape C3 — Données manquantes intermédiaires**
Les prix manquants entre deux observations valides sont imputés par *forward-fill* (`ffill`) sur l'axe temporel. Les NaN en début de série restent NaN jusqu'à la première donnée disponible.

**Étape C4 — Prix bas (RI < 0,50 USD)**
Tout RI inférieur à 0,50 USD est traité comme NaN. Ce seuil élimine les penny stocks illiquides dont les rendements sont biaisés.

**Étape C5 — Prix stales (> 50 % de rendements nuls)**
Sur la fenêtre d'estimation de 120 mois (τ = 120), toute firme dont plus de 50 % des rendements mensuels sont exactement zéro est exclue. Le dénominateur est τ = 120 (et non le nombre d'observations non-NaN), conformément au descriptif.

**Ensemble d'investissement (Investment Set)**
Pour chaque rebalancement annuel Y ∈ {2013, …, 2024}, une firme est incluse si elle satisfait simultanément :

- **Condition C1** : ISIN valide
- **Condition C2** : données de prix disponibles dans la fenêtre [Y−10+1, Y] (10 ans)
- **Condition C3** : ≥ 60 mois d'observations de rendements non-NaN *(déviation justifiée — voir §2.3)*
- **Condition C4** : stale ratio ≤ 50 % sur τ = 120 mois
- **Condition C5** : données Scope 1 et Revenus disponibles pour au moins une année dans la fenêtre

Le nombre de firmes éligibles varie entre **~120** (2013) et **~400** (2024).

**Données carbone annuelles**
Les émissions Scope 1 et revenus annuels manquants entre deux années disponibles sont imputés par *forward-fill*. Les firmes sans donnée carbone en début de série sont exclues jusqu'à la première année disponible.

**Retours simples**
$$R_{i,t} = \frac{RI_{i,t}}{RI_{i,t-1}} - 1$$

### Justification des déviations par rapport au descriptif (2 pages)

**Déviation 1 — Seuil minimum d'observations : 60 mois vs 36 mois**

Le descriptif indique "e.g., less than 3 years" (36 mois) comme seuil minimum. Nous avons retenu 60 mois (5 ans).

*Justification :* Avec τ = 120 mois de fenêtre d'estimation et une QP min-variance, une matrice de covariance estimée sur moins de 60 observations est extrêmement instable. Le seuil de 60 mois réduit le risque d'estimation, améliore le conditionnement numérique de la matrice Σ, et n'est pas en contradiction avec le descriptif qui utilise "e.g." (à titre d'exemple). L'impact est une réduction de l'ensemble d'investissement, particulièrement en 2013–2015 où les données PAC sont plus clairsemées.

**Déviation 2 — Estimateur de covariance : Ledoit-Wolf vs. covariance empirique**

Le descriptif prescrit l'estimateur empirique : $\hat{\Sigma}_Y = \frac{1}{\tau} \sum_{k=1}^{\tau} (R_{t-k} - \hat{\mu})(R_{t-k} - \hat{\mu})'$.

*Justification :* À partir du rebalancement de décembre 2017, le nombre de firmes éligibles N > 453 > τ = 120. Dans ce régime (N > τ), la matrice de covariance empirique est **rang déficient** : elle admet des valeurs propres nulles, ce qui la rend non-inversible et le problème QP dégénéré. Sans correction, la solution de minimisation de la tracking-error (Partie II) n'est pas unique car tout vecteur dans le noyau de Σ̂ est une solution optimale.

L'estimateur de Ledoit-Wolf (shrinkage vers la matrice identité) garantit une matrice définie positive, des valeurs propres strictement positives, et une solution QP unique et numériquement stable. Cet estimateur est standard en finance quantitative et son usage est documenté en cellule 57 du notebook.

*Impact :* Les poids du portefeuille MV sont légèrement moins concentrés qu'avec l'estimateur empirique brut. La performance globale reste comparable.

**Déviation 3 — Imputation des rendements manquants pour μ et Σ**

Le descriptif ne précise pas comment traiter les NaN dans le calcul de la moyenne et de la covariance sur la fenêtre τ.

*Justification :* Nous imputons les NaN par la moyenne de la firme sur la fenêtre disponible (et non par 0). Imputer par 0 biaiserait la moyenne vers le bas et surestimerait la variance pour les firmes avec peu d'observations. L'imputation par la moyenne individuelle est neutre en espérance et préserve la structure de covariance cross-sectionelle.

\newpage

---

# PERSONNE B — Partie I & Métriques carbone (~11 pages)

## Portefeuille standard — Partie I (6 pages)

### Cadre d'optimisation (1 page)

**Portefeuille minimum-variance hors-échantillon $P_{oos}^{(mv)}$**

À chaque rebalancement annuel Y ∈ {2013, …, 2024}, on résout :

$$\min_{\alpha} \; \alpha' \hat{\Sigma}_Y \alpha \quad \text{s.t.} \quad \mathbf{1}'\alpha = 1, \; \alpha \geq 0$$

où $\hat{\Sigma}_Y$ est la matrice de covariance Ledoit-Wolf estimée sur les 120 mois précédant Y, et $\alpha \geq 0$ impose le long-only (nécessaire pour l'interprétation de l'empreinte carbone).

Le problème est un **programme quadratique convexe (QP)**, résolu via le solveur OSQP (via CVXPY). La solution est unique lorsque Σ est définie positive.

**Benchmark pondéré par la capitalisation $P^{(vw)}$**

Les poids du benchmark au mois t sont :

$$w_{i,t}^{(vw)} = \frac{\text{Cap}_{i,t-1}}{\sum_j \text{Cap}_{j,t-1}}$$

La capitalisation du mois *précédent* (t−1) est utilisée, conformément au descriptif, pour éviter le look-ahead bias. Cela reflète un investisseur qui rebalance en début de mois sur la base des prix de clôture du mois précédent.

**Rebalancement annuel et dérive des poids (buy-and-hold)**

Après le rebalancement en décembre Y, les poids dérivent mensuellement selon :

$$\alpha_{i,t+k} = \alpha_{i,t+k-1} \times \frac{1 + R_{i,t+k}}{1 + R_{p,t+k}}$$

où $R_{p,t+k} = \sum_i \alpha_{i,t+k-1} R_{i,t+k}$ est le rendement du portefeuille au mois t+k. Les poids ne sont donc recalculés (via la QP) qu'une fois par an.

**Taux sans risque :** Rf = 0 (non spécifié dans le descriptif — choix conservateur et transparent).

### Résultats de performance — Partie I (2 pages)

**Tableau T2 — Statistiques de performance (janv. 2014 – déc. 2025, T = 144 mois)**

| Statistique | $P_{oos}^{(mv)}$ | $P^{(vw)}$ |
|-------------|-----------------|------------|
| Rendement annualisé | **9,81 %** | 7,74 % |
| Volatilité annualisée | **10,70 %** | 13,22 % |
| Ratio de Sharpe | **0,917** | 0,586 |
| Rendement cumulé | **187,4 %** | 120,7 % |
| Drawdown maximal | *(à calculer)* | *(à calculer)* |
| Skewness | *(à calculer)* | *(à calculer)* |

**Figure 1** — Rendements cumulatifs : MV vs VW (2014–2025). *(insérer fig1_cumulative_returns_part1.png)*

**Analyse :**

Le portefeuille MV surperforme le benchmark VW sur toute la période avec un ratio de Sharpe nettement supérieur (0,917 vs 0,586). Cette surperformance provient principalement d'une **volatilité significativement réduite** (−2,5 pp annualisés) plutôt que d'un rendement brut plus élevé.

La surperformance est cohérente avec la littérature sur l'effet "low-volatility" en Asie-Pacifique, documenté notamment par Blitz & Van Vliet (2007) et Frazzini & Pedersen (2014). Dans la région PAC, les grandes firmes à forte capitalisation (qui dominent le VW) sont souvent des conglomérats diversifiés avec une volatilité naturellement élevée, tandis que le MV identifie des firmes à faible variance idiosyncratique.

**Limitation :** La surperformance du MV peut être partiellement attribuée au **régime de taux bas** (2014–2021) qui favorise les actifs défensifs. Les performances post-2022 (remontée des taux) méritent une attention particulière.

### Analyse de la composition et évolution des poids (2 pages)

**Figure 7** — Évolution des poids du portefeuille MV (top firmes, 2013–2024). *(insérer fig7_weight_evolution.png)*

**Observations clés :**

- La composition du MV est **fortement concentrée** : les 5 premières firmes représentent souvent 40–60 % du portefeuille
- Les **utilities électriques japonaises** (ex. : Electric Power Development) apparaissent avec des poids importants en 2014–2018, car leur volatilité très faible les rend attractives pour le MV
- En 2017–2018, avec N > 453 > τ, l'estimateur Ledoit-Wolf atténue la concentration excessive qui se produirait avec l'estimateur empirique rang-déficient
- La composition se diversifie progressivement à partir de 2020, reflétant l'élargissement de l'ensemble d'investissement

**Rotation annuelle :** La rotation des poids est modérée (~20–30 % de turnover annuel en valeur absolue), ce qui est typique d'un MV long-only avec rebalancement annuel.

### Discussion — Biais d'estimation et stabilité (1 page)

Deux biais potentiels dans notre approche MV :

1. **Biais d'erreur d'estimation dans Σ̂ :** L'estimateur LW shrinks les valeurs propres extrêmes, ce qui peut sous-estimer la variance des firmes les plus volatiles et sur-estimer celle des moins volatiles. Cela introduit un biais favorable aux firmes à faible variance apparente.

2. **Absence de contrainte sur le turnover :** En l'absence de contrainte sur le turnover, le MV peut allouer des poids très élevés à un petit nombre de firmes (corner solutions). Dans les années à fort N, ce phénomène est atténué par LW, mais reste présent.

Ces biais justifient l'usage du MV comme stratégie de **référence conservatrice** plutôt que comme stratégie alpha.

---

## Métriques carbone — Bridge Partie I / Partie II (5 pages)

### Définitions et calcul des métriques (2 pages)

**Intensité Carbone (CI)**

$$CI_{i,Y} = \frac{E_{i,Y}}{Rev_{i,Y} / 1000} \quad [\text{tCO}_2 / \text{MUSD}]$$

où $E_{i,Y}$ sont les émissions Scope 1 en tonnes de CO~2~ et $Rev_{i,Y}$ les revenus en milliers USD. Le facteur 1/1000 convertit kUSD en MUSD.

**Weighted-Average Carbon Intensity (WACI)**

$$\text{WACI}_{p,Y} = \sum_i \alpha_{i,Y} \cdot CI_{i,Y} \quad [\text{tCO}_2 / \text{MUSD}]$$

Mesure l'exposition du portefeuille aux risques de transition carbone par unité de revenu. Recommandée par la TCFD pour le reporting.

**Carbon Footprint (CF)**

$$\text{CF}_{p,Y} = \sum_i \alpha_{i,Y} \cdot \frac{E_{i,Y}}{\text{Cap}_{i,Y}} \quad [\text{tCO}_2 / \text{kUSD Cap}]$$

Mesure les émissions attribuables au portefeuille par rapport à la valeur de marché investie. Utilisée comme contrainte dans la Partie II car elle reflète la "propriété" d'une fraction des émissions.

**CF du benchmark VW :**

$$\text{CF}_{vw,Y} = \frac{\sum_i E_{i,Y}}{\sum_i \text{Cap}_{i,Y}}$$

Formule agrégée (ratio de sommes, non somme de ratios), conformément au descriptif.

### Résultats carbone — Tableau T3 (1 page)

**Tableau T3 — Métriques carbone par année et par portefeuille**

| Année | WACI_mv | WACI_vw | CF_mv | CF_vw |
|-------|---------|---------|-------|-------|
| 2013 | 1 482 | 193 | — | — |
| 2014 | 1 588 | 177 | — | — |
| 2015 | 3 148 | 349 | 475 | — |
| 2016 | **4 401** | **447** | 371 | — |
| 2017 | 2 451 | 521 | — | — |
| 2018 | **4 144** | 397 | — | — |
| ... | ... | ... | ... | ... |
| 2024 | 261 | 206 | — | — |

*(compléter avec les valeurs exactes du notebook pour toutes les années)*

**Figures 2 & 3** — Évolution WACI et CF (2013–2024). *(insérer fig2_waci_evolution.png et fig3_cf_evolution.png)*

### Anomalie de données — Power Assets Holdings (2 pages)

**Identification du problème**

L'analyse détaillée des métriques carbone révèle des valeurs WACI anormalement élevées pour le benchmark VW en 2015–2016 (doublement de 177 à 447), et des valeurs WACI_mv extrêmes en 2016 (4 401 vs 177 pour VW, ratio 9,8×) et 2018 (4 144 vs 397, ratio 10,4×).

**Cause racine identifiée : Power Assets Holdings (ISIN HK0006000050)**

Power Assets Holdings est une holding financière de Hong Kong spécialisée dans les actifs énergétiques internationaux. En 2014, elle a cédé sa filiale d'électricité (CLP Holdings reste une entité séparée), entraînant un **effondrement de 80 % de son chiffre d'affaires** :

| Année | Revenus (kUSD) | Scope 1 (tCO~2~) | CI (tCO~2~/MUSD) |
|-------|----------------|-----------------|-----------------|
| 2012 | 3 187 843 | 8 960 000 | 2 810 |
| 2013 | 1 316 528 | ~0 | ~0 |
| 2014 | 536 297 | NaN → ffill | — |
| 2015 | 318 756 | NaN → ffill | → 28 110 |
| 2016 | 167 919 | 8 960 000 | **53 359** |
| 2017 | 221 447 | NaN | — |

La restructuration corporative de 2014 réduit les revenus de 1 316 528 à 167 919 kUSD (−87 %), tandis que les émissions Scope 1 de 2012 (8 960 000 tCO~2~) sont *forward-fillées* par la procédure standard du notebook. Résultat : CI = 8 960 000 / (167 919 / 1000) = **53 359 tCO~2~/MUSD** en 2016, soit 20× la valeur de 2013.

**Impact sur les portefeuilles**

Cette firme contribue à elle seule à ~65 % du WACI_vw en 2016 et explique les pics de WACI_mv en 2016 et 2018 (le MV y alloue du poids en raison de sa faible volatilité de prix, indépendamment de ses émissions).

**Non-modification du code — Justification**

Le descriptif prescrit explicitement :
1. Le forward-fill des données carbone manquantes (§2.3 du descriptif)
2. L'utilisation de CI = E / (Rev / 1000) sans cap ni winsorisation

Modifier le code pour capper les valeurs de CI ou exclure les firmes avec des restructurations corporatives constituerait une **déviation non documentée** au descriptif. Par ailleurs, ce comportement de données est réel et académiquement intéressant : il illustre les limites des données ESG standardisées face aux événements corporatifs non anticipés.

**Mention dans le rapport :** Ce résultat sera présenté comme une **limitation de la méthodologie** dans la Section 5, avec une recommandation d'amélioration basée sur la winsorisation ou l'ajustement pour les événements corporatifs majeurs.

\newpage

---

# PERSONNE C — Partie II, Discussion & Conclusion (~10 pages)

## Portefeuilles contraints en carbone — Partie II (6 pages)

### Stratégie 1 : $P_{oos}^{(mv)}(0.5)$ — MV avec contrainte CF (2 pages)

**Formulation**

$$\min_{\alpha} \; \alpha' \hat{\Sigma}_Y \alpha \quad \text{s.t.} \quad \mathbf{1}'\alpha = 1, \; \alpha \geq 0, \; \text{CF}(\alpha) \leq 0.5 \times \text{CF}^{(mv)}$$

La contrainte carbone porte sur le **Carbon Footprint** (CF = Σ α_i E_i / Cap_i), pas sur la WACI. Ceci est conforme au descriptif §4.1 et permet d'interpréter la contrainte comme "réduire de moitié les émissions attribuables par dollar investi".

**Résultats de performance :**

| Statistique | $P_{oos}^{(mv)}$ | $P_{oos}^{(mv)}(0.5)$ | Δ |
|-------------|-----------------|----------------------|---|
| Rend. annualisé | 9,81 % | **9,91 %** | +0,10 pp |
| Volatilité | 10,70 % | 10,88 % | +0,18 pp |
| Sharpe | 0,917 | 0,911 | −0,006 |
| Rend. cumulé | 187,4 % | **190,0 %** | +2,6 pp |

**Figure 4** — Rendements cumulatifs : MV vs MV(0.5). *(insérer fig4_cumret_mv_vs_mv50.png)*

**Analyse :** La contrainte carbone de 50 % **n'a pas de coût significatif** sur la performance financière (Sharpe quasi-identique). La légère baisse du Sharpe (0,917 → 0,911) est économiquement négligeable. Ce résultat, contre-intuitif au premier abord, s'explique par :

1. L'ensemble d'investissement PAC contient des firmes à faible volatilité ET faible empreinte carbone (notamment dans les secteurs technologie et santé japonais/australien)
2. Le MV sans contrainte concentre déjà certains poids sur des utilities à faible volatilité mais haute émission → la contrainte carbone redirige vers des firmes à faible émission ET faible volatilité

**Anomalie WACI — Explication**

Le paradoxe WACI_mv50 > WACI_mv en 2015, 2016 et 2021 est documenté et explicable :

- La contrainte porte sur CF (E/Cap), pas sur WACI (E/Rev)
- Le solver peut trouver des firmes à bas E/Cap mais haut E/Rev, i.e., des firmes à **ratio prix/ventes élevé** (price-to-sales élevé, typique des firmes technologiques)
- Résultat : CF diminue de 50 % comme requis, mais WACI augmente car les firmes sélectionnées ont de faibles revenus relativement à leur capitalisation

Ce résultat illustre la **divergence entre CF et WACI** comme métriques carbone et souligne l'importance du choix de la métrique pour définir les contraintes ESG.

**Figure 9** — Changement de composition MV → MV(0.5), Déc. 2016. *(insérer fig9_composition_change_mv50.png)*

La figure 9 montre que CLP Holdings perd environ 8 pp de poids (haute E/Cap ≈ 6 200 tCO~2~/kUSD Cap), tandis que NTT et Kintetsu gagnent du poids (faible E/Cap). Ces redistributions confirment que le solver optimise bien sur la contrainte CF.

### Stratégie 2 : $P_{oos}^{(vw)}(0.5)$ — Tracking Error avec contrainte CF (2 pages)

**Formulation**

$$\min_{\alpha} \; (\alpha - w^{(vw)})' \hat{\Sigma}_Y (\alpha - w^{(vw)}) \quad \text{s.t.} \quad \mathbf{1}'\alpha = 1, \; \alpha \geq 0, \; \text{CF}(\alpha) \leq 0.5 \times \text{CF}^{(vw)}$$

Cette stratégie minimise l'**erreur de suivi (tracking error)** par rapport au benchmark VW tout en réduisant le CF de 50 %. Elle cible les investisseurs indiciel cherchant à "verdir" leur exposition sans s'écarter significativement du benchmark.

**Résultats :**

| Statistique | $P^{(vw)}$ | $P_{oos}^{(vw)}(0.5)$ | Δ |
|-------------|-----------|----------------------|---|
| Rend. annualisé | 7,74 % | 7,73 % | −0,01 pp |
| Volatilité | 13,22 % | 13,23 % | +0,01 pp |
| Sharpe | 0,586 | 0,585 | −0,001 |
| Rend. cumulé | 120,7 % | 120,3 % | −0,4 pp |

**Figure 5** — Rendements cumulatifs : VW vs VW(0.5). *(insérer fig5_cumret_vw_vs_vw50.png)*

La quasi-indiscernabilité des performances confirme que la contrainte carbone de 50 % peut être atteinte à **coût financier quasi-nul** en région PAC sur cette période. La tracking error ex-post est infime.

**Note sur Ledoit-Wolf :** Sans l'estimateur LW, la matrice de covariance empirique rang-déficiente aurait rendu ce problème numériquement instable à partir de 2017 (N > τ). L'usage de LW est donc essentiel pour la validité de cette stratégie, pas seulement pour le MV.

### Stratégie 3 : $P_{oos}^{(vw)}(NZ)$ — Trajectoire Net-Zéro (2 pages)

**Formulation**

La contrainte net-zéro impose une réduction annuelle de 10 % de l'empreinte carbone par rapport à la valeur de base VW de 2013 :

$$\text{CF}(\alpha_Y) \leq (1-\theta)^{Y-Y_0+1} \times \text{CF}_{Y_0}^{(vw)} \quad \theta = 0.1, \; Y_0 = 2013$$

**Vérification de la formule (note d'audit) :**

La formule $(1-\theta)^{Y-Y_0+1}$ donne pour Y = 2013 : $(0.9)^1 = 0.9$ × CF_base, soit une réduction immédiate de 10 % dès la première année. Pour Y = 2014 : $(0.9)^2 = 0.81$, etc. Le notebook implémente exactement cette formule, conforme au §4.1 du descriptif.

**Calendrier des plafonds CF (θ = 0.1, base 2013) :**

| Année Y | Exposant | Facteur | Plafond CF |
|---------|----------|---------|-----------|
| 2013 | 1 | 0,900 | 90 % × CF_base |
| 2014 | 2 | 0,810 | 81 % × CF_base |
| 2017 | 5 | 0,590 | 59 % × CF_base |
| 2020 | 8 | 0,430 | 43 % × CF_base |
| 2024 | 12 | 0,282 | 28 % × CF_base |

**Résultats :**

| Statistique | $P^{(vw)}$ | $P_{oos}^{(vw)}(NZ)$ |
|-------------|-----------|---------------------|
| Rend. annualisé | 7,74 % | 7,72 % |
| Volatilité | 13,22 % | 13,24 % |
| Sharpe | 0,586 | 0,583 |
| Rend. cumulé | 120,7 % | 120,0 % |

La trajectoire net-zéro est encore plus proche du benchmark VW que la stratégie VW(0.5). En 2013–2016, le plafond NZ est moins contraignant que le plafond 50 % car la base 2013 est relativement élevée. En 2020–2024, la contrainte devient plus serrée mais la structure de l'ensemble d'investissement PAC permet toujours de la satisfaire à faible coût de tracking error.

---

## Comparaison globale des 5 portefeuilles (1 page)

**Tableau T4 — Synthèse performance et carbone (2014–2025)**

| Portefeuille | Rend. Ann. | Vol. Ann. | Sharpe | Cum. | CF moy. | WACI moy. |
|-------------|-----------|---------|--------|------|---------|---------|
| $P_{oos}^{(mv)}$ | 9,81 % | 10,70 % | 0,917 | 187,4 % | — | — |
| $P^{(vw)}$ | 7,74 % | 13,22 % | 0,586 | 120,7 % | — | — |
| $P_{oos}^{(mv)}(0.5)$ | 9,91 % | 10,88 % | 0,911 | 190,0 % | −50 % | — |
| $P_{oos}^{(vw)}(0.5)$ | 7,73 % | 13,23 % | 0,585 | 120,3 % | −50 % | — |
| $P_{oos}^{(vw)}(NZ)$ | 7,72 % | 13,24 % | 0,583 | 120,0 % | ≤ 28 % en 2024 | — |

*(compléter CF moy. et WACI moy. avec les valeurs du notebook)*

**Figure 6** — Comparaison finale des 5 portefeuilles. *(insérer fig6_final_comparison.png)*

**Résultat clé :** Dans la région PAC sur 2014–2025, la réduction de l'empreinte carbone de 50 % est atteignable **sans sacrifice financier mesurable** pour toutes les stratégies. Ce résultat est fort et constitue le message principal du rapport.

---

## Discussion et limitations (2 pages)

### Résultats principaux

1. **Surperformance MV :** Le minimum-variance hors-échantillon surperforme le benchmark VW avec un Sharpe de 0,917 contre 0,586. La surperformance est robuste sur toute la période 2014–2025.

2. **Pas de coût financier de la contrainte carbone :** Les trois stratégies carbone-contraintes sont quasi-indiscernables du portefeuille non-contraint correspondant en termes financiers.

3. **CF vs WACI — métriques non-interchangeables :** La contrainte sur CF peut augmenter la WACI, ce qui illustre l'importance du choix de la métrique ESG pour l'investisseur.

4. **Région PAC favorable :** La présence de nombreuses firmes technologiques et de santé à faible volatilité ET faible émission en Asie-Pacifique facilite la construction de portefeuilles carbone-efficaces.

### Limitations

**L1 — Qualité des données Scope 1**

Les données d'émissions Scope 1 souffrent de lacunes importantes, en particulier avant 2015. Le forward-fill peut générer des CI artificiellement élevés (cas Power Assets Holdings). Les données ESG tiers ont des méthodes d'estimation hétérogènes selon les fournisseurs (Datastream, MSCI, CDP), ce qui rend la comparaison inter-études difficile.

**L2 — Anomalie Power Assets Holdings**

Comme détaillé en §3.2.3, la restructuration corporative de Power Assets Holdings en 2014 génère un CI de 53 359 tCO~2~/MUSD en 2016 (20× la valeur normale), biaisant fortement WACI_vw et WACI_mv pour ces années. Une meilleure pratique serait d'appliquer une **winsorisation** des CI au 99e percentile ou d'exclure les firmes ayant subi une restructuration majeure de plus de 50 % de revenus.

**L3 — Concentration du portefeuille MV**

Le MV est intrinsèquement concentré : 5 firmes peuvent représenter 50 % du portefeuille. Cette concentration crée un risque idiosyncratique élevé, non capturé par la volatilité ex-ante. Des contraintes de diversification (poids max par titre ≤ 5 % ou 10 %) seraient souhaitables en pratique mais ne sont pas requises par le descriptif.

**L4 — Estimateur Ledoit-Wolf et biais**

L'estimateur LW introduit un biais de shrinkage qui favorise les firmes à variance apparente modérée. Ce biais peut être atténué par des estimateurs plus sophistiqués (shrinkage non-linéaire de Ledoit-Wolf analytique, factor models, POET) mais cela dépasse le cadre de ce projet.

**L5 — Période d'évaluation**

La période 2014–2025 inclut des régimes de taux très différents (ZIRP 2014–2021, remontée des taux 2022–2023, normalisation 2024–2025). Les stratégies carbone-contraintes n'ont pas été testées sous différents régimes économiques. La robustesse des résultats dans un environnement inflationniste ou de récession reste à explorer.

---

## Conclusion (1 page)

Ce projet a implémenté et évalué cinq stratégies de portefeuille sur la région Pacifique, couvrant à la fois la performance financière et l'empreinte carbone.

**Principaux enseignements :**

1. Le portefeuille minimum-variance hors-échantillon **surperforme significativement le benchmark VW** (Sharpe 0,917 vs 0,586) grâce à une réduction de la volatilité de 2,5 pp annualisés, sans sacrifice de rendement brut.

2. La **contrainte carbone de 50 %** est atteignable sans coût financier mesurable dans la région PAC, que ce soit en optimisation MV (−0,006 de Sharpe) ou en minimisation de tracking error (−0,001 de Sharpe). Ce résultat est encourageant pour les investisseurs souhaitant intégrer des objectifs climatiques sans compromettre leur mandate de performance.

3. La **trajectoire net-zéro** (−10 %/an) reste réalisable sur la période observée, mais la contrainte se reserre fortement après 2020. L'évolution du tissu industriel PAC (transition énergétique, croissance du secteur technologique) sera déterminante pour la viabilité à long terme.

4. La **qualité des données ESG** représente le principal risque méthodologique. L'anomalie Power Assets Holdings illustre qu'une approche systématique aveugle peut produire des métriques carbone artificiellement élevées lors de restructurations corporatives.

**Pistes futures :**

- Intégrer les émissions Scope 2 et Scope 3 (données plus complètes post-2018)
- Tester des contraintes de diversification sur les poids
- Comparer avec d'autres régions (US, EUR) pour évaluer si le résultat "zéro coût" est spécifique au PAC
- Appliquer un ajustement pour les événements corporatifs dans le calcul de CI

\newpage

---

# Annexes

## Annexe A — Divulgation LLM

Ce projet académique a utilisé Claude Code (Anthropic, modèle claude-sonnet-4-6) comme assistant de programmation pour :

- Le débogage du code Python (résolution des erreurs CVXPY, gestion des NaN dans les matrices de covariance)
- L'audit de l'implémentation par rapport au descriptif du projet
- La structuration et la rédaction du plan de rapport

Tous les choix méthodologiques (seuils de filtrage, estimateur de covariance, formulation des QP) ont été validés manuellement par l'équipe. Les analyses et interprétations sont exclusivement le fait des auteurs. Le code Python dans le notebook n'a pas été généré automatiquement par l'IA — il a été écrit par les auteurs avec assistance pour le débogage.

## Annexe B — Informations techniques

**Environnement :**

- Python 3.11
- Bibliothèques principales : `pandas`, `numpy`, `cvxpy`, `scikit-learn`, `matplotlib`, `openpyxl`
- Solveur QP : OSQP (via CVXPY)
- Estimateur de covariance : `sklearn.covariance.LedoitWolf`

**Reproductibilité :**

Le notebook `saam_portfolio.ipynb` est entièrement reproductible. Placer `Sustainability.xlsx` dans le même dossier que le notebook et exécuter toutes les cellules séquentiellement. Aucun chemin absolu. Durée d'exécution estimée : ~3–5 minutes.

## Annexe C — Références

- Ledoit, O. & Wolf, M. (2004). "A well-conditioned estimator for large-dimensional covariance matrices." *Journal of Multivariate Analysis*, 88(2), 365–411.
- Blitz, D. & Van Vliet, P. (2007). "The volatility effect." *Journal of Portfolio Management*, 34(1), 102–113.
- Frazzini, A. & Pedersen, L. H. (2014). "Betting against beta." *Journal of Financial Economics*, 111(1), 1–23.
- TCFD (2017). "Recommendations of the Task Force on Climate-related Financial Disclosures."
- Roncalli, T. (2023). "Handbook on Financial Risk Management." Chapman & Hall.
