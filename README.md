# SAAM — Portfolio Allocation with a Carbon Objective

**Sustainability Aware Asset Management** — HEC Lausanne, 2026  
Pacific Region (PAC) | Scope 1 CO₂ Emissions

---

## Quick Start (ordinateur neuf)

Suivre les étapes dans l'ordre. Durée totale estimée : **10–15 minutes**.

### Étape 1 — Installer Python

Télécharger Python **3.11** (ou 3.12) depuis [python.org/downloads](https://www.python.org/downloads/)

> **Windows** : cocher **"Add Python to PATH"** avant de cliquer sur Install.  
> **Mac** : l'installeur .pkg s'occupe de tout.

Vérifier l'installation en ouvrant un terminal et en tapant :
```
python --version
```
→ doit afficher `Python 3.11.x` ou `3.12.x`

---

### Étape 2 — Installer VS Code

Télécharger depuis [code.visualstudio.com](https://code.visualstudio.com/) et installer.

---

### Étape 3 — Cloner le projet

Dans un terminal :
```bash
git clone https://github.com/DylanZermatten/saam-portfolio.git
cd saam-portfolio
```

Ou télécharger le ZIP depuis GitHub → Code → Download ZIP, puis décompresser.

---

### Étape 4 — Ouvrir dans VS Code

```bash
code .
```

VS Code va proposer d'installer les extensions recommandées (**Python** et **Jupyter**) — cliquer **Install All**.

Si la notification n'apparaît pas : aller dans Extensions (icône carrés à gauche), chercher et installer :
- `Python` (Microsoft)
- `Jupyter` (Microsoft)

---

### Étape 5 — Créer un environnement virtuel

Dans le terminal **intégré de VS Code** (Menu → Terminal → New Terminal) :

**Mac / Linux :**
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

**Windows :**
```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

L'installation prend 1–3 minutes. Le terminal doit afficher `(.venv)` devant le prompt.

---

### Étape 6 — Sélectionner le kernel dans VS Code

1. Ouvrir `saam_portfolio.ipynb` (double-clic dans l'explorateur de fichiers VS Code)
2. En haut à droite du notebook, cliquer sur le nom du kernel (souvent `Python 3.x.x`)
3. Choisir **Python Environments…** → sélectionner `.venv`

---

### Étape 7 — Lancer le notebook

Cliquer sur **Run All** (double flèche en haut du notebook).

Durée d'exécution : **3–5 minutes** (optimisation QP sur 11 années).

> Le notebook est entièrement auto-contenu et reproductible de haut en bas.  
> `Sustainability.xlsx` est inclus dans le repo — aucun fichier externe à ajouter.

---

## Résultats

| Portefeuille | Rend. Ann. | Vol. Ann. | Sharpe | Rend. Cumulé |
|---|---|---|---|---|
| $P_{oos}^{(mv)}$ Min-Variance | ~9.8% | ~10.7% | ~0.9 | — |
| $P^{(vw)}$ Value-Weighted | ~7.7% | ~13.2% | ~0.6 | — |
| $P_{oos}^{(mv)}(0.5)$ CF−50% | ~9.9% | ~10.9% | ~0.9 | — |
| $P_{oos}^{(vw)}(0.5)$ TE+CF−50% | ~7.7% | ~13.2% | ~0.6 | — |
| $P_{oos}^{(vw)}(NZ)$ Net-Zero | ~7.7% | ~13.2% | ~0.6 | — |

Période : Jan 2014 – Déc 2024 (T = 132 mois) | Rebalancements : Déc 2013 → Déc 2023

---

## Structure du repo

```
saam-portfolio/
├── saam_portfolio.ipynb        # Notebook principal (tout reproduire top-to-bottom)
├── Sustainability.xlsx         # Données (inclus)
├── requirements.txt            # Dépendances Python
├── rapport_plan.md             # Plan du rapport (Markdown)
├── rapport_plan.pdf            # Plan du rapport (PDF)
├── SAAM_PartI_Results.xlsx     # Soumission préliminaire (12 avril)
├── fig1_cumulative_returns_part1.png
├── fig2_waci_evolution.png
├── fig3_cf_evolution.png
├── fig4_cumret_mv_vs_mv50.png
├── fig5_cumret_vw_vs_vw50.png
├── fig6_final_comparison.png
├── fig7_weight_evolution.png
├── fig8_top10_carbon_intensity.png
└── fig9_composition_change_mv50.png
```

---

## Méthodologie

| Choix | Valeur |
|---|---|
| Région | PAC (Pacifique) |
| Émissions | Scope 1 uniquement |
| Estimateur de covariance | Ledoit-Wolf (résout la singularité quand N > τ = 120, à partir de 2017) |
| Fenêtre d'estimation | 120 mois (10 ans) |
| Seuil d'observations min. | ≥ 60 mois non-NaN |
| Taux sans risque | Rf = 0 (aucune donnée dans le fichier source) |
| Poids VW | Décalés d'un mois (t−1) pour éviter le look-ahead bias |
| Rebalancement | Annuel, Décembre |
| Délistings | Rendement −100% injecté au mois de clôture |

---

## Dépendances

```
pandas, numpy, cvxpy, matplotlib, scikit-learn, openpyxl, ipykernel
```

Python ≥ 3.11 recommandé. Voir `requirements.txt`.
