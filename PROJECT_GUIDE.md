# Cahier des charges – Indicateur TradingView modulable

**Ce fichier est l’unique source de référence** pour le cadre du projet, les besoins, les attentes, les décisions d’orientation et le détail des modules (**quoi** faire et **pourquoi**). Il fixe les objectifs, détaille les besoins précis et les méthodes générales. Il doit être complété ou ajusté au fil du projet et donne le **cadre et la direction**.

Le détail de l’exécution technique (phases, besoins par étape) est dans **PLAN_GUIDAGE.md**.

---

## Documentation du projet – Nature des fichiers et utilisation

La documentation du projet est organisée en **quatre fichiers**. Fournir cette sélection à un nouvel agent permet d’avoir tout le contexte et de travailler sur des parties spécifiques.

| Fichier | Nature | Contenu principal | Quand l’utiliser |
|--------|--------|-------------------|------------------|
| **PROJECT_GUIDE.md** (ce fichier) | **Cahier des charges** | Objectifs du projet, besoins précis, méthodes générales, sélection des timeframes, spécification des 4 modules (rôle, sous-outils, sorties, affichage), attentes par niveau (vue HTF / LTF), conventions de code. | Pour cadrer le projet, répondre à « quoi faire / pourquoi », ou travailler sur une partie dont les specs sont ici (ex. logique TF, Module 1, Module 2…). |
| **PLAN_GUIDAGE.md** | **Guide technique d’exécution** | Phases 0 à 7, cadre général et **besoins précis et spécifiques de chaque étape**, sous-tâches détaillées avec statut (à faire / fait), dépendances, rappels techniques (Pine v6, `request.security`, script unique). | Pour savoir **comment** mettre en place le projet étape par étape, quelle phase faire, quelles sous-tâches réaliser et où en est le projet. |
| **README.md** | **Présentation du projet (GitHub)** | Description du projet pour la page du dépôt : objectif, stack, liens vers la doc, utilisation rapide. | Pour découvrir le projet et orienter vers la doc (PROJECT_GUIDE, PLAN_GUIDAGE). |
| **ANNEXE.md** | **Informations complémentaires et contextuelles** | Contexte TradingView (types de scripts, édition, publication), référence au prompt source, sources et liens. Tout ce qui ne relève pas du cahier des charges ni du guide d’exécution. | En complément quand il faut des rappels sur TradingView, l’intention initiale du projet ou des références externes. |

**Fichier de code** : **`indicateur_mtf.pine`** — Seul script à modifier ; à copier dans l’éditeur TradingView pour tester.

### Ordre de lecture pour un nouvel agent

1. **PROJECT_GUIDE.md** (ce fichier) — Cadre, objectifs, besoins, modules.
2. **PLAN_GUIDAGE.md** — Process technique, phases, besoins par étape, statut.
3. **Guide de phase** (ex. GUIDE_PHASE_2.md) — Si un guide a été rédigé pour la phase à traiter (produit au moment de démarrer la phase), le lire avant de coder.
4. **ANNEXE.md** — Si besoin de contexte TradingView ou de références.

En fournissant **PROJECT_GUIDE.md** + **PLAN_GUIDAGE.md** (et éventuellement README.md, ANNEXE.md, le guide de phase en cours), un nouvel agent dispose de tout le contexte pour travailler sur des parties spécifiques du projet.

**Rôle de l’agent** : le maître d’œuvre est le développeur ; l’agent est un **assistant de codage**. Il ne code que sur demande ou accord explicite, ne prend pas d’initiatives sur les fonctionnalités ou les méthodes, et doit proposer des solutions techniques plus adaptées en les expliquant. Le détail du comportement attendu est dans **PLAN_GUIDAGE.md** (section « Comportement et rôle de l’agent »).

---

## Pour reprendre le travail (nouvel agent / nouvelle session)

1. **Lire en premier** : ce fichier (**PROJECT_GUIDE.md**), puis **PLAN_GUIDAGE.md** (phases, statut, prochaine étape). Si un guide de phase existe pour la phase à traiter, le lire avant de coder.
2. **Script principal** : **`indicateur_mtf.pine`** (Pine Script **v6**). Seul fichier de code à modifier ; à copier dans l’éditeur TradingView pour tester.
3. **Logique centrale** : **sélection des timeframes** (section « Sélection des timeframes » ci-dessous). **Ne pas la modifier** sans relire cette section et les règles de calcul de l’indice **x**.
4. **Prochaine étape** : voir **PLAN_GUIDAGE.md** (section suivi des tâches). Actuellement : **Phase 2 – Module 1 (Contexte HTF)**. Utiliser les variables `tf_htf`, `tf_ltf`, `is_htf_only` pour les appels MTF.

---

## Titre et objectif du projet

Indicateur TradingView **modulaire, hiérarchique et dynamique** pour l’analyse visuelle et la décision, destiné à identifier des **setups à haute probabilité**. L’indicateur **ne trade pas automatiquement** : il fournit une aide à la décision et à la compréhension du marché.

**Objectif final** : un script qui **prépare une analyse technique en temps réel** sur le graphique affiché. Selon le mode (Trading LTF / Trading HTF) et la TF du graphique, le script affiche soit le contexte et les zones clés HTF (vue HTF), soit en plus les confirmations LTF et une proposition d’entrée (vue LTF).

---

## Choix technique : Indicateur (pas stratégie, pas bibliothèque pour l’instant)

- **Indicateur retenu** : l’objectif est « analyse en temps réel + affichage de tout + proposition d’entrée », sans backtest ni ordres automatiques. L’indicateur permet tout cela (affichage, calculs, alertes via `alert()`). La stratégie ajouterait le Strategy Tester et des ordres simulés, inutiles pour ce besoin.
- **Migration vers une stratégie** : possible plus tard. La logique de calcul (tendance, BOS/CHoCH, OB, FVG, sweep, retest, setup valide) reste identique ; on remplace `indicator()` par `strategy()` et on ajoute `strategy.entry()` / `strategy.exit()` au point de proposition d’entrée. On peut aussi garder l’indicateur et créer un second script stratégie qui réutilise la même logique (éventuellement via une bibliothèque).
- **Bibliothèque** : pas pour l’instant. Toutes les fonctions sont dans l’indicateur. Une library sera envisagée si on ajoute une stratégie (ou un second indicateur) qui réutilise les mêmes calculs, pour éviter la duplication.

---

## Rôle exact de l’indicateur

**En une phrase** : l’indicateur prépare une **analyse technique en temps réel** sur le graphique affiché, en affichant **tous les éléments** qu’il sait produire (contexte HTF, zones clés, confirmations LTF) jusqu’à une **proposition d’entrée** lorsque toutes les conditions sont réunies.

**Ce qu’il fait** :
- S’exécute sur le **symbole** et la **timeframe** du graphique courant.
- Selon le **mode** (Trading LTF / Trading HTF) et la **TF du graphique**, décide si tu es en **vue HTF** ou **vue LTF** (règles ci-dessous : indice x, `is_htf_only`).
- **Vue HTF** : affiche uniquement les infos utiles pour analyser la HTF (contexte + zones clés).
- **Vue LTF** : affiche les infos HTF **et** les infos LTF utiles pour définir un point d’entrée (confirmations + proposition d’entrée si tout est aligné).
- **Pédagogie** : affiche **toutes** les infos détectées, même sans signal final.
- **Pas de trading automatique** : aucun ordre ; affichage et, si besoin, alerte sur « setup valide ».

**Ce qu’il ne fait pas** : pas de backtest (Strategy Tester), pas d’ordres (simulés ou réels). Il prépare l’analyse et propose une entrée ; la décision d’agir reste à l’utilisateur.

**Ce qu’il doit nous indiquer (en général)** :
- Quel niveau est actif (vue HTF seule ou vue LTF+HTF ; idéalement : label ou table avec TF graph, vue, tf_htf, tf_ltf).
- Contexte HTF : tendance/structure, biais, BOS/CHoCH HTF, zones de liquidité.
- Zones clés HTF : OB, FVG, Discount/Premium (type, force, couleur selon biais).
- Confirmations LTF (en vue LTF uniquement) : sweep, BOS/CHoCH LTF, retest des zones HTF.
- Proposition d’entrée (en vue LTF uniquement) : label « Setup valide », flèche, résumé quand biais + prix en zone HTF clé + confirmation LTF sont validés.
- Même sans signal final : toutes les infos des modules actifs restent affichées.

---

## Principes directeurs

- **Construction progressive** : HTF → LTF → signal final. Chaque module alimente le suivant.
- **Pédagogie** : affichage de **toutes** les infos détectées, même sans signal final.
- **Modularité** : chaque module est développé et testé de façon relativement indépendante.
- **Sous-outils indépendants** : au sein de chaque module, chaque élément (tendance, BOS/CHoCH, OB, FVG, sweep, retest, etc.) est **codé indépendamment** puis assemblé.

---

## Langage et documentation

- **Langage** : **Pine Script** (TradingView). **Version ciblée** : **v6** (dynamic requests, booléens stricts, etc.).
- **Références** : [Pine Script v6](https://www.tradingview.com/pine-script-reference/v6/), [Release notes v6](https://www.tradingview.com/pine-script-docs/release-notes/), doc Timeframes et `request.security()` pour le MTF. Voir aussi **ANNEXE.md** pour le contexte TradingView (types de scripts, édition, publication).

---

## Sélection des timeframes (logique centrale du projet)

Cette logique détermine **quelles TF sont étudiées** et **quel type d’informations** l’indicateur renvoie selon le graphique affiché.

### Données fixes

- **`ltf_array`** = `[1, 5, 15, 60]` (1m, 5m, 15m, 1h) — indices 0 à 3.
- **`htf_array`** = `[15, 60, 240, D]` (15m, 1h, 4h, 1d) — indices 0 à 3.
- **`tf_mode`** : `"focus_ltf"` ou `"focus_htf"` (choix utilisateur).
- **`tf_graph`** : la timeframe du graphique (ou TF principale en input).

### Principe : un indice x, une paire (ou triplet) de TF

- **ltf** = `ltf_array[x]`, **htf** = `htf_array[x]` (évolution : **htf2** = `htf_array[x+1]`).
- L’objectif est de déterminer si le graphique est considéré comme **HTF** ou **LTF**, puis de calculer `x`.

### Règles de calcul de l’indice x

1. **`tf_graph` uniquement dans `ltf_array`** (ex. 1m, 5m) → graphique LTF. **x** = index de `tf_graph` dans `ltf_array`. **ltf** = `tf_graph`, **htf** = `htf_array[x]`.
2. **`tf_graph` uniquement dans `htf_array`** (ex. 4h, 1d) → graphique HTF. **x** = index dans `htf_array`. **htf** = `tf_graph`. Pas de LTF d’étude.
3. **`tf_graph` dans les deux** (15m ou 1h) → **`tf_mode`** décide : **focus_ltf** → on traite comme LTF (ltf, htf selon x dans htf_array) ; **focus_htf** → on traite comme HTF (htf = tf_graph, pas de LTF).

### Mode d’affichage et sorties exposées

- **Vue HTF seule** (`is_htf_only` = true) : contexte HTF uniquement (Modules 1 + 2). Pas de sweep, retest, proposition d’entrée.
- **Vue LTF + HTF** (`is_htf_only` = false) : contexte HTF + confirmations LTF + proposition d’entrée (Modules 1 + 2 + 3 + 4).

**Variables pour le code** : `tf_graph`, `is_htf_only`, `tf_ltf`, `tf_htf` (plus tard `tf_htf2`).

---

## Attentes par niveau (vue HTF vs vue LTF)

### Vue HTF seule (`is_htf_only` = true)

**Quand** : graphique considéré comme HTF (ex. 4h, 1d ; ou 15m/1h en mode focus_htf). **TF** : `tf_htf` = TF du graphique ; pas de `tf_ltf`.

**Attendu** :
- **Contexte HTF** : tendance (HH/HL ou LH/LL), biais, BOS/CHoCH sur la HTF, zones de liquidité (Equal High/Low, anciens High/Low). Données via `request.security` avec `tf_htf`.
- **Zones clés HTF** : OB, FVG, Discount/Premium. Rectangles + labels (ex. « OB H1 », « FVG H1 »), couleur selon biais.
- **Pas de couche LTF** : modules 3 et 4 inactifs (pas de sweep, retest, proposition d’entrée).

**Objectif** : tout ce qui sert à **analyser la HTF** ; pas encore de point d’entrée.

### Vue LTF + HTF (`is_htf_only` = false)

**Quand** : graphique considéré comme LTF (ex. 5m, 15m ; ou 15m/1h en mode focus_ltf). **TF** : `tf_htf` pour contexte/zones ; `tf_ltf` = TF du graphique pour confirmations et setup.

**Attendu** :
- **Contexte HTF** et **Zones clés HTF** : identiques à la vue HTF, affichés sur le graphique LTF.
- **Confirmations LTF** : sweep, BOS/CHoCH LTF, retest OB/FVG. Flèches, labels (« Sweep détecté »), marqueurs de retest.
- **Proposition d’entrée** : si biais aligné + prix en zone HTF clé + confirmation LTF validée → label « Setup valide », flèche, résumé. Sinon, pas de signal mais toutes les infos des modules 1–3 restent visibles.

**Objectif** : **définir un point d’entrée potentiel** ; signal clair quand tout est aligné.

### Synthèse par niveau

| Niveau        | Modules actifs | Ce que l’indicateur doit indiquer |
|---------------|----------------|-----------------------------------|
| **Vue HTF seule** | 1 + 2          | Contexte HTF + Zones clés HTF. Rien LTF ni proposition d’entrée. |
| **Vue LTF+HTF**   | 1 + 2 + 3 + 4  | Contexte HTF + Zones HTF + Confirmations LTF + proposition d’entrée (« Setup valide ») si conditions réunies. |

---

## Les 4 modules : objectifs, sous-outils, sorties, affichage

Les **méthodes** (algorithmes) sont précisées module par module au codage. Chaque **sous-outil** est codé **indépendamment**, puis intégré au module.

### Module 1 – Contexte HTF

**Rôle** : structure dominante et contexte sur la HTF (tendance, biais, BOS/CHoCH, zones de liquidité).

**Données** : OHLC (et si besoin volume) de la HTF via `request.security(symbol, tf_htf, ...)`.

**Sous-outils** :

| Sous-outil            | Produit / sorties | Affichage attendu |
|-----------------------|-------------------|-------------------|
| Tendance / structure  | HH/HL ou LH/LL, **biais** (bullish/bearish). Réutilisable modules 2 et 4. | Labels tendance/biais (« Bullish » / « Bearish », HH/HL, LH/LL). |
| BOS/CHoCH HTF         | Détection BOS/CHoCH sur HTF (bar/time, direction). | Flèches BOS/CHoCH. |
| Zones de liquidité    | Equal High/Low, anciens High/Low. Coordonnées ou zones réutilisables. | Rectangles ou liens (boxes/lignes). |

**Sorties du module** : biais, derniers BOS/CHoCH, zones de liquidité. **Actif** : dès qu’une HTF est définie (vue HTF et vue LTF).

---

### Module 2 – Zones clés HTF

**Rôle** : zones de réaction sur la HTF (Order Blocks, FVG) et zones Discount/Premium.

**Données** : données HTF (`request.security` avec `tf_htf`) ; **biais** du Module 1 (couleur, sens).

**Sous-outils** :

| Sous-outil     | Produit / sorties | Affichage attendu |
|----------------|-------------------|-------------------|
| OB (Order Block) | Détection OB HTF. Coordonnées, type (bullish/bearish), force. | Rectangles + labels (ex. « OB H1 »), couleur selon biais. |
| FVG            | Détection FVG HTF. Coordonnées, type, force. | Rectangles + labels (ex. « FVG H1 »), couleur selon biais. |
| Discount / Premium | Zones Discount (sous équilibre) et Premium (au-dessus). | Rectangles ou délimitations, labels si pertinent. |

**Sorties du module** : zones (coordonnées, type, force) pour Modules 3 et 4 (retest, « prix en zone »). **Actif** : dès qu’une HTF est définie (vue HTF et vue LTF).

---

### Module 3 – Confirmation LTF

**Rôle** : valider les zones HTF avec l’action de prix locale (LTF) : sweep, BOS/CHoCH LTF, retest OB/FVG.

**Données** : données LTF (graphique = LTF ou `request.security` avec `tf_ltf`) ; **zones clés HTF** du Module 2 (pour retest).

**Sous-outils** :

| Sous-outil      | Produit / sorties | Affichage attendu |
|-----------------|-------------------|-------------------|
| Sweep          | Détection sweep (niveau dépassé puis retour). Booléen/état, direction. | Flèches ou marqueurs + label « Sweep détecté ». |
| BOS/CHoCH LTF  | Détection BOS/CHoCH sur LTF. Sorties réutilisables. | Flèches ou formes BOS/CHoCH LTF. |
| Retest OB/FVG  | Prix LTF a retesté une zone HTF sans l’invalider. Retest validé ou non (par zone si besoin). | Marqueurs de retest (labels, formes) sur les zones. |

**Sorties du module** : états (sweep détecté, retest validé, BOS/CHoCH LTF) pour Module 4. **Actif** : **uniquement en vue LTF+HTF** (`is_htf_only` = false).

---

### Module 4 – Setup / point d’entrée

**Rôle** : agréger les conditions des modules 1 à 3 et produire une **proposition d’entrée** (signal final) uniquement si tout est aligné.

**Données** : biais (Module 1), zones et « prix en zone » (Module 2), confirmations LTF (Module 3).

**Conditions** (chacune traitée indépendamment, puis agrégées) :

| Condition              | Rôle | Sortie |
|------------------------|------|--------|
| Biais HTF aligné       | Biais Module 1 = direction du setup (ex. long → haussier). | Booléen : aligné ou non. |
| Prix dans zone HTF clé| Prix actuel (LTF) dans une zone OB ou FVG (ou Discount/Premium pertinente) du Module 2. | Booléen : en zone ou non. |
| Confirmation LTF validée | Sweep et/ou retest OB/FVG et/ou BOS/CHoCH LTF selon la règle. | Booléen : validée ou non. |

**Agrégation** : signal final **uniquement si** les trois conditions sont vraies (ou selon la règle exacte à fixer).

**Affichage** : si signal final → label « Setup valide » (ou « Long » / « Short »), flèche directionnelle, résumé des éléments validés. Si pas de signal → pas de « Setup valide », mais **conserver** tout l’affichage des modules 1–3 (pédagogie).

**Actif** : **uniquement en vue LTF+HTF**. En vue HTF seule, Module 4 inactif.

---

## Conventions de code (à compléter au fil du projet)

- **Nommage** : préfixe par module (`ctxHTF_*`, `zonesHTF_*`, `confLTF_*`, `setup_*`).
- **Organisation** : blocs par module, paramètres et inputs en tête.
- **Affichage** : `label.new()`, `plotshape()` / `plotarrow()`, `box.new()` ; couleurs et transparence selon type et force de zone.

---

## Évolution du projet

- **Déploiement** : méthodes précisées module par module au codage ; chaque sous-outil codé indépendamment puis assemblé. Process et suivi dans **PLAN_GUIDAGE.md**. Un **guide de phase** (ex. GUIDE_PHASE_2.md) peut être rédigé au moment de démarrer une phase pour détailler objectifs, entrées, méthodes et critères de fin.
- **Sélection TF** : actuellement 1 LTF / 1 HTF ; évolution 1 LTF / 2 HTF prévue.
- Ce cahier des charges et **PLAN_GUIDAGE.md** sont mis à jour à chaque phase. Les informations complémentaires (TradingView, prompt source, sources) restent dans **ANNEXE.md**.
