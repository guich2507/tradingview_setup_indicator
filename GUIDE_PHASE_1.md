# Guide de phase 1 – Paramètres globaux et logique TF

Ce document détaille **les besoins précis** et **comment réaliser** la Phase 1 du projet (paramètres globaux et logique de sélection des timeframes). Il est produit au moment de démarrer la phase et sert de référence pendant l’implémentation.

**Références :** PROJECT_GUIDE.md (section « Sélection des timeframes »), PLAN_GUIDAGE.md (Phase 1, suivi des tâches). **Script à modifier :** `indicateur_mtf.pine`.

---

## 1. Objectif de la phase

Implémenter dans le script Pine la **logique de sélection des timeframes** : selon la TF du graphique et le mode choisi par l’utilisateur, le script doit exposer de façon fiable les variables **tf_ltf**, **tf_htf** et **is_htf_only**, afin que les modules 2 à 6 puissent s’appuyer sur elles (notamment pour les appels `request.security()` et pour activer ou non les modules 3 et 4).

**Livrable principal :** variables TF exposées et utilisables pour les modules ; optionnel : affichage (label ou table) pour vérification.

---

## 2. Entrées et sorties

### 2.1 Entrées (données et paramètres utilisateur)

| Entrée       | Source                    | Description |
|-------------|----------------------------|-------------|
| **tf_graph** | Graphique (ou input dérivé) | Timeframe du graphique courant (ex. 1m, 5m, 15m, 1h, 4h, 1d). |
| **trading_mode** | Input utilisateur | Indique si l’utilisateur trade sur des LTF ou sur des HTF. Deux valeurs : **ltf_trading** (scalping ou day trading), **htf_trading** (swing, hautes timeframes). La logique du script utilise cette valeur quand tf_graph est dans les deux listes (15m ou 1h) pour décider de la vue. |

**Listes de timeframes (modifiables par l’utilisateur, valeurs par défaut) :**

- **ltf_array** : listes des TF « basses ». Par défaut [1m, 5m, 15m, 1h] → en Pine `["1", "5", "15", "60"]` (indices 0 à 3). L’utilisateur peut définir ses propres valeurs.
- **htf_array** : listes des TF « hautes ». Par défaut [15m, 1h, 4h, 1d] → en Pine `["15", "60", "240", "D"]` (indices 0 à 3). L’utilisateur peut définir ses propres valeurs.

**Contrainte d’ordre (à vérifier dans le code) :** dans chaque liste (ltf_array et htf_array), les valeurs doivent être **strictement croissantes** : de la TF la plus petite à la TF la plus grande. Exemple valide : 1, 5, 15, 60. Exemple invalide : 1, 15, 60, 5. Le code devra vérifier cette contrainte ; en cas de non-respect, appliquer une règle définie (ex. rejet et usage des valeurs par défaut, ou tri automatique, ou message à l’utilisateur).

### 2.2 Sorties exposées (variables pour le reste du script)

| Sortie          | Type    | Description |
|-----------------|---------|-------------|
| **tf_ltf**      | string  | Timeframe LTF d’étude. En vue LTF+HTF uniquement : égal à tf_graph. En vue HTF seule : non utilisé (modules 3/4 inactifs). |
| **tf_htf**      | string  | Timeframe HTF d’étude. Toujours défini dès qu’on a une vue (HTF seule ou LTF+HTF). En vue HTF seule = tf_graph ; en vue LTF = htf_array[x]. |
| **is_htf_only** | booléen | `true` = vue HTF seule (contexte HTF uniquement, pas de LTF). `false` = vue LTF+HTF (contexte HTF + confirmations LTF + setup). |
| **x**           | int     | (Optionnel mais utile en debug.) Indice calculé (0 à 3) pour la paire (ltf, htf). |

**Règles métier :**

- En **vue HTF seule** : `tf_htf` = tf_graph ; `tf_ltf` peut être laissé à une valeur par défaut (ex. `na` ou `""`) car non utilisé.
- En **vue LTF+HTF** : `tf_ltf` = tf_graph ; `tf_htf` = htf_array[x].

---

## 3. Contraintes techniques Pine Script v6

- **Représentation des timeframes :** en Pine, la timeframe est une **chaîne** (string). On obtient celle du graphique avec `timeframe.period` (ex. `"1"`, `"5"`, `"15"`, `"60"`, `"240"`, `"D"`). Les tableaux seront donc des **array de strings** (ou, si besoin, on compare tf_graph à des constantes string).
- **Tableaux :** Pine v6 permet `array.new_string()`, `array.get()`, `array.indexOf()`. Pour des listes fixes de 4 éléments, on peut soit initialiser un array en tête du script, soit utiliser des fonctions qui retournent la valeur à l’indice i (ex. helper qui retourne `"1"` si i=0, `"5"` si i=1, etc.) pour éviter les soucis d’initialisation.
- **Index dans un array :** `array.indexOf(id, value)` retourne l’index de la première occurrence, ou -1 si absent. À utiliser pour « tf_graph est-il dans ltf_array ? » et « tf_graph est-il dans htf_array ? ».
- **Pas de type « timeframe » dédié pour des listes :** tout reste en string ; les appels `request.security(symbol, tf_htf, ...)` acceptent une string.

---

## 4. Règles de calcul (rappel détaillé)

### 4.1 Déterminer la « catégorie » de tf_graph

- **idx_ltf** = index de tf_graph dans ltf_array (-1 si absent).
- **idx_htf** = index de tf_graph dans htf_array (-1 si absent).

Ensuite :

- Si **idx_ltf >= 0 et idx_htf < 0** → tf_graph **uniquement dans LTF** (ex. 1m, 5m).
- Si **idx_htf >= 0 et idx_ltf < 0** → tf_graph **uniquement dans HTF** (ex. 4h, 1d).
- Si **idx_ltf >= 0 et idx_htf >= 0** → tf_graph **dans les deux** (15m ou 1h) → trading_mode décide.

### 4.2 Calcul de x et des sorties

**Cas A – tf_graph uniquement dans LTF (ex. 1m, 5m)**  
- **x** = idx_ltf.  
- **tf_ltf** = tf_graph.  
- **tf_htf** = htf_array[x].  
- **is_htf_only** = false (vue LTF+HTF).

**Cas B – tf_graph uniquement dans HTF (ex. 4h, 1d)**  
- **x** = idx_htf.  
- **tf_htf** = tf_graph.  
- **tf_ltf** = non utilisé (laisser na ou "" selon convention).  
- **is_htf_only** = true (vue HTF seule).

**Cas C – tf_graph dans les deux (15m ou 1h)**  
- Si **trading_mode == "ltf_trading"** : traiter comme LTF → même logique que Cas A (x = idx_ltf, tf_ltf = tf_graph, tf_htf = htf_array[x], is_htf_only = false).  
- Si **trading_mode == "htf_trading"** : traiter comme HTF → même logique que Cas B (x = idx_htf, tf_htf = tf_graph, pas de tf_ltf utile, is_htf_only = true).

### 4.3 Cas non prévus (tf_graph ni dans LTF ni dans HTF)

Exemples : graphique en 3m, 2h, W, M. **Décision à trancher** : soit on considère que l’indicateur ne s’applique pas (désactiver les modules ou afficher un message), soit on applique une règle de repli (ex. considérer comme HTF si tf_graph >= 60, sinon LTF). Pour la Phase 1, proposer une règle simple et la documenter (ex. **repli : is_htf_only = true, tf_htf = tf_graph, tf_ltf = na**).

### 4.4 Ordre croissant des listes (ltf_array et htf_array)

Chaque liste doit être **strictement croissante** : de la valeur la plus petite à la valeur la plus grande (en termes de durée de la timeframe).  
- **Valide** : 1, 5, 15, 60 (ltf) ou 15, 60, 240, D (htf).  
- **Invalide** : 1, 15, 60, 5 ou 60, 15, 240, D.  

**À faire dans le code :** vérifier que ltf_array et htf_array (construits à partir des inputs utilisateur) respectent cet ordre. Si ce n’est pas le cas, appliquer une règle claire (ex. ignorer la config invalide et utiliser les valeurs par défaut, ou trier automatiquement les listes avant usage, ou afficher un avertissement). La règle choisie sera documentée ici ou dans le script.

---

## 5. Étapes d’implémentation détaillées

### Étape 1 – Obtenir tf_graph et ajouter l’input trading_mode

1. Dans la section « PARAMÈTRES ET INPUTS » de `indicateur_mtf.pine`, ajouter un **input** pour que l’utilisateur indique s’il trade sur des LTF ou des HTF :  
   - Nom du paramètre : **trading_mode**.  
   - Type : string ou enum.  
   - Options : **ltf_trading** (scalping ou day trading), **htf_trading** (swing, hautes timeframes).  
   - Descriptions dans le script : claires et centrées sur le type de trading (LTF vs HTF), sans interpréter le comportement de l’indicateur.  
   - Défaut recommandé : `ltf_trading`.
2. Définir **tf_graph** : utiliser `timeframe.period` (timeframe du graphique). Si le cahier des charges prévoit une « TF principale en input » en override, on peut ajouter plus tard un input optionnel ; pour cette phase, **tf_graph = timeframe.period** suffit.

**Vérification :** le script compile ; tf_graph et trading_mode sont disponibles.

---

### Étape 2 – Exposer ltf_array et htf_array (modifiables par l’utilisateur)

1. Rendre **ltf_array** et **htf_array** configurables via des inputs (ex. 4 inputs LTF + 4 inputs HTF, ou chaîne à parser), avec **valeurs par défaut** : LTF = ["1", "5", "15", "60"], HTF = ["15", "60", "240", "D"].
2. Construire les tableaux utilisés par la logique à partir de ces inputs.
3. **Vérifier que chaque liste est strictement croissante** (de la TF la plus petite à la plus grande). Exemple valide : 1, 5, 15, 60. Invalide : 1, 15, 60, 5. En cas de non-respect, appliquer la règle choisie (défaut, tri, ou avertissement — voir § 4.4).

**Vérification :** avec les défauts, array.get(ltf_array, 0) == "1", array.get(htf_array, 2) == "240". Après modification par l’utilisateur, les listes restent croissantes ou la règle de repli s’applique.

---

### Étape 3 – Calcul des index idx_ltf et idx_htf

1. Utiliser **array.indexOf(ltf_array, tf_graph)** → **idx_ltf** (ou -1).  
2. Utiliser **array.indexOf(htf_array, tf_graph)** → **idx_htf** (ou -1).

**Vérification :** sur un graphique 5m, idx_ltf = 1, idx_htf = -1. Sur 15m, idx_ltf = 2, idx_htf = 0.

---

### Étape 4 – Branchement selon la catégorie (LTF seul / HTF seul / les deux)

1. Déclarer les variables **x**, **tf_ltf**, **tf_htf**, **is_htf_only** (avec des valeurs par défaut si nécessaire pour que Pine soit satisfait sur tous les chemins).
2. Implémenter la logique en suivant les cas A, B, C de la section 4.2 :  
   - Si uniquement LTF → Cas A.  
   - Si uniquement HTF → Cas B.  
   - Si les deux → selon trading_mode, appliquer Cas A ou Cas B.
3. Gérer le cas « ni LTF ni HTF » selon la règle de repli choisie (section 4.3).

**Vérification :** pour 5m → is_htf_only = false, tf_ltf = "5", tf_htf = "60". Pour 4h → is_htf_only = true, tf_htf = "240". Pour 15m + ltf_trading → comme LTF ; pour 15m + htf_trading → comme HTF.

---

### Étape 5 – Exposer les variables pour les modules

1. S’assurer que **tf_ltf**, **tf_htf**, **is_htf_only** (et éventuellement **x**) sont calculés au même « niveau » (même barre, pas dans un bloc conditionnel qui pourrait les laisser non définis sur certains chemins).
2. Les modules suivants utiliseront :  
   - **tf_htf** pour tous les `request.security(..., tf_htf, ...)` (Modules 1 et 2).  
   - **tf_ltf** pour les `request.security(..., tf_ltf, ...)` ou le prix LTF (Modules 3 et 4), uniquement quand **is_htf_only == false**.  
   - **is_htf_only** pour activer ou non les Modules 3 et 4.

Aucun tracé ni label n’est obligatoire à cette étape ; l’objectif est d’avoir des variables correctes.

---

### Étape 6 – (Optionnel) Affichage de vérification

1. Ajouter un **label** (ou une **table**) qui affiche, par exemple :  
   - TF du graphique : tf_graph.  
   - Vue : « HTF seule » ou « LTF+HTF ».  
   - tf_ltf (ou « — » si vue HTF seule).  
   - tf_htf.  
2. Position du label : par exemple en haut à gauche du graphique, ou en overlay discret.  
3. Prévoir éventuellement un **input** « Afficher infos TF » (true/false) pour désactiver l’affichage en production.

**Vérification :** en changeant la TF du graphique et trading_mode, les valeurs affichées correspondent aux règles du cahier des charges.

---

## 6. Critères de fin de phase

- [ ] Input **trading_mode** présent et opérationnel : l’utilisateur peut indiquer s’il trade sur des LTF (ltf_trading — scalping/day) ou des HTF (htf_trading — swing, hautes TF). Descriptions explicites dans le script, sans sur-interprétation.  
- [ ] **ltf_array** et **htf_array** exposés en paramètres modifiables par l’utilisateur, avec valeurs par défaut (1m, 5m, 15m, 1h) et (15m, 1h, 4h, 1d).
- [ ] **Vérification d’ordre croissant** : le code vérifie que chaque liste (ltf_array, htf_array) est strictement croissante (ex. 1, 5, 15, 60 ✓ ; 1, 15, 60, 5 ✗). Règle en cas d’invalidité documentée ou implémentée (ex. défaut, tri, ou avertissement).  
- [ ] **tf_graph** = timeframe du graphique (timeframe.period).  
- [ ] Logique d’indice **x** correcte pour les trois cas (LTF seul, HTF seul, les deux + trading_mode).  
- [ ] Variables **tf_ltf**, **tf_htf**, **is_htf_only** exposées et correctes sur tous les cas prévus.  
- [ ] Cas « tf_graph hors listes » géré par une règle de repli documentée.  
- [ ] (Optionnel) Affichage label/table pour vérification TF et vue.  
- [ ] Script compilable et testé sur au moins : 1m, 5m, 15m (ltf_trading et htf_trading), 1h (idem), 4h, 1d.

---

## 7. Évolutions exclues de cette phase

- **1 LTF / 2 HTF** (htf2 = htf_array[x+1]) : prévu pour plus tard, ne pas implémenter en Phase 1.  
- Input optionnel pour « TF principale » en override de timeframe.period : possible en évolution, pas requis pour cette phase.

---

## 8. Synthèse « quoi modifier dans le script »

| Zone du script              | Action |
|----------------------------|--------|
| En-tête / paramètres       | Ajouter input trading_mode (ltf_trading / htf_trading). |
| Après les paramètres       | Définir tf_graph = timeframe.period ; construire ltf_array, htf_array. |
| Nouveau bloc « Logique TF »| Calcul idx_ltf, idx_htf ; branchement Cas A/B/C ; assigner x, tf_ltf, tf_htf, is_htf_only. |
| Affichage (optionnel)      | Label ou table avec tf_graph, vue, tf_ltf, tf_htf. |

---

*Document : première mouture du guide Phase 1. À ajuster si des choix d’implémentation (ex. repli pour TF hors listes) sont précisés.*
