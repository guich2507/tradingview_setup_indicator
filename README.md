# Indicateur MTF modulable – TradingView

Indicateur **multi-timeframe (MTF)** pour TradingView : analyse visuelle et aide à la décision (setups à haute probabilité), **sans trading automatique**. Pine Script v5.

---

## Reprendre le travail (nouvel agent / nouvelle session)

1. **Lire dans cet ordre :**
   - **[PROJECT_GUIDE.md](PROJECT_GUIDE.md)** — Contexte, objectifs, **logique de sélection des timeframes** (élément majeur), les 4 modules, conventions.
   - **[PLAN_ACTION.md](PLAN_ACTION.md)** — Phases 0 à 7 avec statut ([x] fait / [ ] à faire) et **prochaine étape**.

2. **Fichiers importants :**
   - **`indicateur_mtf.pine`** — Script principal (seul fichier de code). À copier dans l’éditeur Pine de TradingView pour exécution et tests.
   - **PROJECT_GUIDE.md** — Référence pour la logique TF (indice x, ltf_array/htf_array, is_htf_only, 1 LTF / 1 HTF).
   - **PLAN_ACTION.md** — Suivi des tâches et détail des sous-étapes par phase.
   - **tradingview_indicator_prompt.txt** — Prompt source du projet.

3. **État actuel :**
   - **Phase 0** et **Phase 1** terminées (setup projet + paramètres globaux et logique TF en 1 LTF / 1 HTF).
   - **Prochaine étape : Phase 2** — Module 1 (Contexte HTF) : `request.security()` sur `tf_htf`, tendance/structure, BOS/CHoCH, zones de liquidité.

4. **Ne pas modifier** la logique de sélection des TF sans relire la section « Sélection des timeframes » de PROJECT_GUIDE.md.
