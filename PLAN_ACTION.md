# Plan d'action – Indicateur TradingView modulable

**Pour reprendre le travail :** lire [PROJECT_GUIDE.md](PROJECT_GUIDE.md) (contexte + logique TF), puis ce fichier. Script à modifier : **`indicateur_mtf.pine`**. **Prochaine étape : Phase 2** (Module 1 – Contexte HTF).

---

Ce plan est voué à **évoluer** : sous-étapes et méthodes pourront être détaillées ou réordonnées au fil du projet.

**Légende statut :** `[ ]` à faire · `[x]` fait · indiquer « en cours » en commentaire si besoin.

---

## Phase 0 – Setup projet (cadre + structure minimale)

- [x] Définir le cadre et les étapes (fichier de guidage, plan d'action)
- [x] Créer le fichier de guidage à la racine (PROJECT_GUIDE.md)
- [x] Décider de la version Pine (v5 recommandée en premier)
- [x] Poser un premier script Pine avec **uniquement la structure minimale nécessaire** : `indicator()`, paramètres vides ou minimaux, symbole = chart actuel, pas d'exécution automatique

---

## Phase 1 – Paramètres globaux et logique TF

**Référence détaillée :** [PROJECT_GUIDE.md](PROJECT_GUIDE.md) → section « Sélection des timeframes (logique centrale du projet) ».

- [x] Ajouter l'input `tf_mode` (options : focus_ltf / focus_htf)
- [x] Définir les tableaux `ltf_array` = [1, 5, 15, 60] et `htf_array` = [15, 60, 240, D]
- [x] Implémenter la logique d'indice **x** : selon que `tf_graph` est seulement dans LTF, seulement dans HTF, ou dans les deux (alors `tf_mode` décide) → calcul de **x**, puis **ltf** = ltf_array[x], **htf** = htf_array[x]
- [x] Exposer **is_htf_only** (vue HTF seule = contexte uniquement ; vue LTF = contexte HTF + raffinement LTF) et les variables **tf_ltf**, **tf_htf**
- [x] (Optionnel) Afficher en label ou en table la TF du graphique, la vue (HTF seul / LTF+HTF), et les TF utilisées (LTF, HTF)

**Note :** Implémentation actuelle en **1 LTF / 1 HTF**. Passage à **1 LTF / 2 HTF** (ajout de htf2 = htf_array[x+1]) prévu dans un second temps du projet.

---

## Phase 2 – Module 1 – Contexte HTF

Chaque sous-outil est **codé indépendamment**, puis intégré au module.

- [ ] **Données HTF** : utiliser `request.security()` pour récupérer les données nécessaires depuis les HTF
- [ ] **Sous-outil Tendance / structure** : détection HH/HL ou LH/LL, biais ; sorties réutilisables ; affichage labels (tendance/biais)
- [ ] **Sous-outil BOS/CHoCH HTF** : détection et sorties réutilisables ; affichage flèches BOS/CHoCH
- [ ] **Sous-outil Zones de liquidité** : Equal High/Low, anciens High/Low ; sorties réutilisables ; affichage rectangles ou liens
- [ ] Assembler les sorties du module (biais, BOS/CHoCH, zones liquidité) pour les modules suivants

---

## Phase 3 – Module 2 – Zones clés HTF

Chaque élément d'info est **codé indépendamment**, puis intégré au module.

- [ ] S'appuyer sur les données HTF (`request.security`) et, si besoin, sur le biais du Module 1
- [ ] **Sous-outil OB (Order Block)** : détection sur HTF ; stockage zones (coordonnées, type, force) ; affichage rectangles + labels (ex. "OB H1"), couleur selon biais
- [ ] **Sous-outil FVG** : détection sur HTF ; stockage zones ; affichage rectangles + labels (ex. "FVG H1"), couleur selon biais
- [ ] **Sous-outil Discount / Premium** : identification des zones ; stockage et affichage cohérents
- [ ] Exposer les zones (coordonnées, type, force) pour les modules 3 et 4

---

## Phase 4 – Module 3 – Confirmation LTF

Chaque élément est **codé indépendamment** et retourne ses états/sorties ; puis intégration au module.

- [ ] Utiliser les données LTF et les zones clés HTF (Module 2) pour les validations
- [ ] **Sous-outil Sweep** : détection ; sortie (booléen/état) ; affichage flèches, labels ("Sweep détecté")
- [ ] **Sous-outil BOS/CHoCH LTF** : détection ; sorties réutilisables ; affichage
- [ ] **Sous-outil Retest OB/FVG** : validation retest des zones HTF ; sortie (retest validé ou non) ; affichage marqueurs de retest
- [ ] Exposer les sorties (sweep détecté, retest validé, etc.) pour le Module 4

---

## Phase 5 – Module 4 – Setup / point d'entrée

Chaque condition est traitée **indépendamment**, puis agrégée au signal final.

- [ ] **Condition biais HTF** : récupérer le biais du Module 1, exposer "aligné" ou non
- [ ] **Condition prix dans zone HTF clé** : croiser prix actuel et zones du Module 2, exposer "en zone" ou non
- [ ] **Condition confirmation LTF** : utiliser les sorties du Module 3 (sweep, retest, BOS/CHoCH LTF), exposer "validée" ou non
- [ ] **Agrégation** : signal final uniquement si toutes les conditions sont réunies
- [ ] Affichage : label "Setup valide", flèche directionnelle, résumé des éléments validés ; conserver l'affichage des modules 1–3 si pas de signal

---

## Phase 6 – Affichage pédagogique et options

- [ ] Centraliser les options d'affichage (activer/désactiver par module, couleurs, transparence)
- [ ] S'assurer que toutes les infos détectées restent visibles même sans signal final
- [ ] Vérifier la lisibilité (limites de zones affichées, nettoyage des anciens dessins si nécessaire)

---

## Phase 7 – Revue et évolution

- [ ] Tester sur plusieurs instruments et TF (1h/5m en focus_ltf, 4h/1h en focus_htf, etc.)
- [ ] Ajuster le plan : détailler sous-étapes, ajouter phases de refactor (ex. passage à v6), documenter les méthodes (PROJECT_GUIDE.md ou METHODES.md)

---

Voir [PROJECT_GUIDE.md](PROJECT_GUIDE.md) pour le contexte, les objectifs du projet et **la logique de sélection des timeframes**.
