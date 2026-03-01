# Indicateur MTF modulable – TradingView

Indicateur **multi-timeframe (MTF)** pour TradingView : analyse visuelle et aide à la décision pour identifier des **setups à haute probabilité**, **sans trading automatique**. Développé en **Pine Script v6**.

---

## Description

L’indicateur prépare une **analyse technique en temps réel** sur le graphique affiché. Un paramètre **trading_mode** permet à l’utilisateur d’indiquer s’il trade sur des LTF (ltf_trading : scalping/day) ou des HTF (htf_trading : swing). Selon la timeframe et ce réglage, l’indicateur affiche :

- **Contexte HTF** : tendance, structure (HH/HL, LH/LL), BOS/CHoCH, zones de liquidité  
- **Zones clés HTF** : Order Blocks, FVG, Discount/Premium  
- **Confirmations LTF** (en vue LTF) : sweep, BOS/CHoCH LTF, retest des zones HTF  
- **Proposition d’entrée** (« Setup valide ») lorsque biais, zone et confirmation LTF sont alignés  

Toutes les infos détectées restent affichées même sans signal final, pour une approche **pédagogique** et une décision éclairée.

---

## Stack

- **Pine Script v6** (TradingView)  
- **MTF** : `request.security()` avec logique de sélection des timeframes (trading_mode : ltf_trading / htf_trading)  
- **Un seul script** : `indicateur_mtf.pine` (overlay sur le graphique)

---

## Documentation

| Fichier | Rôle |
|--------|------|
| **[PROJECT_GUIDE.md](PROJECT_GUIDE.md)** | Cahier des charges : objectifs, besoins, 4 modules, logique des timeframes, conventions. |
| **[PLAN_GUIDAGE.md](PLAN_GUIDAGE.md)** | Guide technique d’exécution : phases 0 à 7, besoins par étape, suivi des tâches. |
| **[ANNEXE.md](ANNEXE.md)** | Contexte TradingView, références, informations complémentaires. |

---

## Utilisation

1. Ouvrir le script **`indicateur_mtf.pine`** dans l’éditeur Pine de TradingView (ou copier-coller son contenu).  
2. Cliquer sur **« Ajouter au graphique »**.  
3. L’indicateur s’exécute sur le **symbole** et la **timeframe** du graphique. Dans les paramètres (engrenage), choisir **trading_mode** selon que vous tradez sur des LTF (ltf_trading) ou des HTF (htf_trading).  

*Il n’y a pas d’import de fichier .pine dans TradingView : le workflow est édition locale (IDE/Git) puis copier-coller dans l’éditeur pour tester.*

---

## État du projet

- **Phases 0 et 1** : terminées (structure du script, logique des timeframes).  
- **Prochaine étape** : Phase 2 – Module 1 (Contexte HTF : tendance, BOS/CHoCH, zones de liquidité).  

Le détail des phases et du suivi des tâches est dans **[PLAN_GUIDAGE.md](PLAN_GUIDAGE.md)**.
