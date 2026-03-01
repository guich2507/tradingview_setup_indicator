# Fichier de guidage – Indicateur TradingView modulable

## Titre et objectif du projet

Indicateur TradingView **modulaire, hiérarchique et dynamique** pour l’analyse visuelle et la décision, destiné à identifier des **setups à haute probabilité**. L’indicateur **ne trade pas automatiquement** : il fournit une aide à la décision et à la compréhension du marché.

---

## Principes directeurs

- **Construction progressive** : les éléments sont construits dans l’ordre HTF → LTF → signal final. Chaque module alimente le suivant.
- **Pédagogie** : affichage de **toutes** les infos détectées (zones, structure, confirmations), même sans signal final, pour que le trader comprenne pourquoi un setup est valide ou non.
- **Modularité** : chaque module (Contexte HTF, Zones clés HTF, Confirmation LTF, Setup/entrée) est conçu pour être développé et testé de façon relativement indépendante.
- **Sous-outils indépendants** : au sein de chaque module, chaque élément d’info ou de détection est **codé indépendamment** (ex. Module 1 : tendance, BOS/CHoCH, rectangles de liquidité ; Module 2 : OB, FVG, Discount/Premium ; etc.). Cela permet de tester, ajuster et réutiliser chaque brique séparément avant de les assembler.

---

## Langage et documentation

- **Langage** : **Pine Script** (langage propriétaire TradingView pour indicateurs et stratégies).
- **Version ciblée** : **v5** en premier (stable, documentation complète). **v6** pourra être envisagée plus tard pour des appels dynamiques à `request.security()` (TF en série dans des boucles).
- **Références** :
  - [Pine Script Language Reference (v5)](https://www.tradingview.com/pine-script-reference/v5/)
  - [Pine Script Language Reference (v6)](https://www.tradingview.com/pine-script-reference/v6/)
  - Documentation Timeframes et `request.security()` pour l’analyse multi-timeframe (MTF).

---

## Paramètres globaux (rappel)

- **`tf_mode`** (string) : `"focus_ltf"` ou `"focus_htf"`.
  - `"focus_ltf"` → TF principales = LTF (scalp / day trading), TF communes/intermédiaires = HTF.
  - `"focus_htf"` → TF principales = HTF (swing / long terme), TF communes/intermédiaires = LTF.

- **Tableaux de timeframes** :
  - **LTF** = `[1m, 5m, 15m, 1h]`
  - **HTF** = `[15m, 1h, 4h, 1d]`

- **TF communes** : 15m et 1h — leur rôle (LTF ou HTF) dépend de `tf_mode`.

---

## Logique TF dynamique (rappel)

1. **TF principale** : choisie par l’utilisateur (ou déduite du graphique). Son **rôle** (LTF ou HTF) est déterminé par `tf_mode`.
2. **TF correspondantes** :
   - Pour une **LTF** choisie → les **2 HTF supérieures** dans le tableau HTF (n et n+1).
   - Pour une **HTF** choisie → les **LTF inférieures** pour les confirmations locales.
3. **Règles d’usage** :
   - **HTF** : structure, OB/FVG, zones de liquidité, BOS/CHoCH.
   - **LTF** : confirmations locales (sweep, BOS/CHoCH, retest).
4. **Construction** : HTF → LTF → signal final. Affichage obligatoire de toutes les infos détectées, même sans signal final.

---

## Les 4 modules (objectifs et sous-outils)

Les **méthodes** (algorithmes de détection) seront définies module par module au fur et à mesure. Chaque **sous-outil** ou **élément d’info** est codé **indépendamment**, puis intégré au module.

| Module | Objectif | Sous-outils / éléments à coder séparément | Sorties visuelles (rappel) |
|--------|----------|-------------------------------------------|----------------------------|
| **1 – Contexte HTF** | Structure dominante et biais, BOS/CHoCH HTF, zones de liquidité. | Tendance (HH/HL ou LH/LL) ; BOS/CHoCH HTF ; rectangles/zones de liquidité (Equal High/Low, anciens High/Low). | Labels tendance/biais, flèches BOS/CHoCH, rectangles/liens liquidité. |
| **2 – Zones clés HTF** | OB, FVG, zones Discount/Premium. | Détection OB ; détection FVG ; zones Discount ; zones Premium. | Rectangles colorés + labels ("OB H1", "FVG H1"), couleur selon biais. |
| **3 – Confirmation LTF** | Valider les zones HTF avec l’action de prix locale. | Sweep ; BOS/CHoCH LTF ; retest OB/FVG. Chaque élément retourne ses états/sorties indépendamment. | Flèches, labels ("Sweep détecté"), marqueurs de retest. |
| **4 – Setup / point d’entrée** | Combiner HTF + LTF → signal final. | Chaque condition (biais aligné, prix en zone, confirmation LTF) puis agrégation. | Label "Setup valide", flèche directionnelle, résumé des éléments validés. |

---

## Conventions de code (à compléter au fil du projet)

- **Nommage** : préfixe par module (ex. `ctxHTF_*` pour Contexte HTF, `zonesHTF_*` pour Zones clés HTF, etc.).
- **Organisation du script** : blocs par module, paramètres et inputs en tête.
- **Affichage** : `label.new()`, `plotshape()` / `plotarrow()`, `box.new()` ; couleurs et transparence selon type et force de zone.

---

## Évolution du projet

- **Phase 0** : définir le cadre et les étapes ; poser un premier script avec **uniquement la structure minimale nécessaire** (indicator, pas d’exécution automatique).
- **Déploiement étape par étape** : les méthodes seront précisées par module au moment du codage ; **chaque sous-outil est codé indépendamment** puis assemblé (voir PLAN_ACTION.md pour le détail des sous-étapes).
- Ce fichier et le plan d’action (**PLAN_ACTION.md**) peuvent être **mis à jour à chaque phase**.

---

## Référence du prompt source

[tradingview_indicator_prompt.txt](tradingview_indicator_prompt.txt)
