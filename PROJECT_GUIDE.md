# Fichier de guidage – Indicateur TradingView modulable

## Pour reprendre le travail (nouvel agent / nouvelle session)

1. **Lire en premier** : ce fichier (**PROJECT_GUIDE.md**), puis **PLAN_ACTION.md** pour le statut des phases et la prochaine étape.
2. **Script principal** : **`indicateur_mtf.pine`** (Pine Script v5). C’est le seul fichier de code à modifier ; à copier dans l’éditeur TradingView pour tester.
3. **Logique centrale** : la **sélection des timeframes** (section ci-dessous « Sélection des timeframes »). Elle détermine quelles TF sont utilisées (1 LTF / 1 HTF) et si l’indicateur affiche seulement le contexte HTF ou contexte HTF + raffinement LTF. **Ne pas la modifier** sans relire cette section et les règles de calcul de l’indice **x**.
4. **Prochaine étape** : **Phase 2 – Module 1 (Contexte HTF)** dans PLAN_ACTION.md (données HTF via `request.security()`, tendance/structure, BOS/CHoCH, zones de liquidité). Utiliser les variables exposées `tf_htf`, `tf_ltf`, `is_htf_only` pour brancher les appels MTF.
5. **Autres fichiers utiles** : `tradingview_indicator_prompt.txt` (prompt source), `README.md` (vue d’ensemble et liens).

---

## Titre et objectif du projet

Indicateur TradingView **modulaire, hiérarchique et dynamique** pour l'analyse visuelle et la décision, destiné à identifier des **setups à haute probabilité**. L'indicateur **ne trade pas automatiquement** : il fournit une aide à la décision et à la compréhension du marché.

---

## Principes directeurs

- **Construction progressive** : les éléments sont construits dans l'ordre HTF → LTF → signal final. Chaque module alimente le suivant.
- **Pédagogie** : affichage de **toutes** les infos détectées (zones, structure, confirmations), même sans signal final, pour que le trader comprenne pourquoi un setup est valide ou non.
- **Modularité** : chaque module (Contexte HTF, Zones clés HTF, Confirmation LTF, Setup/entrée) est conçu pour être développé et testé de façon relativement indépendante.
- **Sous-outils indépendants** : au sein de chaque module, chaque élément d'info ou de détection est **codé indépendamment** (ex. Module 1 : tendance, BOS/CHoCH, rectangles de liquidité ; Module 2 : OB, FVG, Discount/Premium ; etc.). Cela permet de tester, ajuster et réutiliser chaque brique séparément avant de les assembler.

---

## Langage et documentation

- **Langage** : **Pine Script** (langage propriétaire TradingView pour indicateurs et stratégies).
- **Version ciblée** : **v5** en premier (stable, documentation complète). **v6** pourra être envisagée plus tard pour des appels dynamiques à `request.security()` (TF en série dans des boucles).
- **Références** :
  - [Pine Script Language Reference (v5)](https://www.tradingview.com/pine-script-reference/v5/)
  - [Pine Script Language Reference (v6)](https://www.tradingview.com/pine-script-reference/v6/)
  - Documentation Timeframes et `request.security()` pour l'analyse multi-timeframe (MTF).

---

## Sélection des timeframes (logique centrale du projet)

Cette logique détermine **quelles TF sont étudiées** et **quel type d'informations** l'indicateur renvoie selon le graphique affiché. C'est un **élément majeur** du projet.

### Données fixes

- **`ltf_array`** = `[1, 5, 15, 60]` (1m, 5m, 15m, 1h) — indices 0 à 3.
- **`htf_array`** = `[15, 60, 240, D]` (15m, 1h, 4h, 1d) — indices 0 à 3.
- **`tf_mode`** : `"focus_ltf"` ou `"focus_htf"` (choix utilisateur).
- **`tf_graph`** : la timeframe du graphique (ou TF principale choisie en input).

### Principe : un indice x, une paire (ou triplet) de TF

Les TF utilisées sont dérivées d'**un seul indice** `x` dans ces tableaux :

- **ltf** = `ltf_array[x]`
- **htf** = `htf_array[x]` (et en évolution : **htf2** = `htf_array[x+1]`)

L'objectif est de déterminer si le graphique doit être considéré comme **HTF** ou **LTF**, puis de calculer `x` en conséquence.

### Règles de calcul de l'indice x

1. **`tf_graph` présent uniquement dans `ltf_array`** (ex. 1m, 5m)  
   → Le graphique est LTF. **x** = index de `tf_graph` dans `ltf_array`.  
   → On étudie **ltf** = `tf_graph` et **htf** = `htf_array[x]`.

2. **`tf_graph` présent uniquement dans `htf_array`** (ex. 4h, 1d)  
   → Le graphique est HTF. **x** = index de `tf_graph` dans `htf_array`.  
   → On ne renvoie que le **contexte HTF** : **htf** = `tf_graph`. Aucune LTF d'étude (pas de raffinement entrée).

3. **`tf_graph` présent dans les deux tableaux** (15m ou 1h)  
   → **`tf_mode`** décide :
   - **focus_ltf** : on considère le graphique comme LTF → **x** = index de `tf_graph` dans **htf_array** → ltf = `ltf_array[x]`, htf = `htf_array[x]`.
   - **focus_htf** : on considère le graphique comme HTF → **x** = index de `tf_graph` dans **ltf_array** → on ne renvoie que le contexte HTF (htf = `tf_graph`), pas de LTF d'étude.

### Mode d'affichage : ce que renvoie l'indicateur

- **Vue HTF seule** (`is_htf_only` = true) : graphique considéré comme HTF.  
  L'indicateur renvoie **uniquement** le contexte HTF : tendance, structure, BOS/CHoCH HTF, zones de liquidité, OB/FVG HTF, etc. Pas de couche LTF (pas de sweep, retest, FVG pour l'entrée).

- **Vue LTF + HTF** (`is_htf_only` = false) : graphique considéré comme LTF.  
  L'indicateur renvoie le **contexte HTF** (tendance, zones, BOS/CHoCH…) **et** le **raffinement LTF** : OB, FVG, sweep, retest, etc., pour affiner vers un setup / point d'entrée.

### Implémentation actuelle et évolution

- **Actuellement** : une seule paire **1 LTF / 1 HTF** (ltf, htf) par graphique. Suffisant pour poser la logique et les modules.
- **Évolution prévue** : passer à **1 LTF / 2 HTF** (ltf, htf1, htf2 avec htf2 = `htf_array[x+1]`) lorsque les résultats sur une paire seront satisfaisants, pour enrichir le contexte HTF.

### Sorties exposées (variables pour le code)

- **`tf_graph`** : TF du graphique.
- **`is_htf_only`** : `true` = afficher seulement contexte HTF ; `false` = afficher contexte HTF + raffinement LTF.
- **`tf_ltf`** : TF LTF à étudier (ou vide/na si vue HTF seule).
- **`tf_htf`** : TF HTF à étudier (toujours renseignée).  
  (Plus tard : **`tf_htf2`** pour la deuxième HTF.)

---

## Les 4 modules (objectifs et sous-outils)

Les **méthodes** (algorithmes de détection) seront définies module par module au fur et à mesure. Chaque **sous-outil** ou **élément d'info** est codé **indépendamment**, puis intégré au module.

| Module | Objectif | Sous-outils / éléments à coder séparément | Sorties visuelles (rappel) |
|--------|----------|-------------------------------------------|----------------------------|
| **1 – Contexte HTF** | Structure dominante et biais, BOS/CHoCH HTF, zones de liquidité. | Tendance (HH/HL ou LH/LL) ; BOS/CHoCH HTF ; rectangles/zones de liquidité (Equal High/Low, anciens High/Low). | Labels tendance/biais, flèches BOS/CHoCH, rectangles/liens liquidité. |
| **2 – Zones clés HTF** | OB, FVG, zones Discount/Premium. | Détection OB ; détection FVG ; zones Discount ; zones Premium. | Rectangles colorés + labels ("OB H1", "FVG H1"), couleur selon biais. |
| **3 – Confirmation LTF** | Valider les zones HTF avec l'action de prix locale. | Sweep ; BOS/CHoCH LTF ; retest OB/FVG. Chaque élément retourne ses états/sorties indépendamment. | Flèches, labels ("Sweep détecté"), marqueurs de retest. |
| **4 – Setup / point d'entrée** | Combiner HTF + LTF → signal final. | Chaque condition (biais aligné, prix en zone, confirmation LTF) puis agrégation. | Label "Setup valide", flèche directionnelle, résumé des éléments validés. |

---

## Conventions de code (à compléter au fil du projet)

- **Nommage** : préfixe par module (ex. `ctxHTF_*` pour Contexte HTF, `zonesHTF_*` pour Zones clés HTF, etc.).
- **Organisation du script** : blocs par module, paramètres et inputs en tête.
- **Affichage** : `label.new()`, `plotshape()` / `plotarrow()`, `box.new()` ; couleurs et transparence selon type et force de zone.

---

## Évolution du projet

- **Phase 0** : définir le cadre et les étapes ; poser un premier script avec **uniquement la structure minimale nécessaire** (indicator, pas d'exécution automatique).
- **Déploiement étape par étape** : les méthodes seront précisées par module au moment du codage ; **chaque sous-outil est codé indépendamment** puis assemblé (voir PLAN_ACTION.md pour le détail des sous-étapes).
- **Sélection TF** : implémentation actuelle en 1 LTF / 1 HTF ; passage à 1 LTF / 2 HTF prévu dans un second temps.
- Ce fichier et le plan d'action (**PLAN_ACTION.md**) peuvent être **mis à jour à chaque phase**.

---

## Référence du prompt source

[tradingview_indicator_prompt.txt](tradingview_indicator_prompt.txt)
