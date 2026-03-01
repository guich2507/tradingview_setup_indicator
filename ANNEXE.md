# Annexe – Informations complémentaires et contextuelles

Ce fichier regroupe les **informations annexes** qui ne relèvent pas du cahier des charges (PROJECT_GUIDE.md) ni du guide technique d’exécution (PLAN_GUIDAGE.md) : contexte TradingView, référence au prompt source, sources et liens. Il sert de base d’informations complémentaires et contextuelles.

---

## 1. Référence du prompt source

Le projet s’appuie sur un prompt initial qui a défini l’intention et les orientations. La formulation originale est conservée dans le fichier **`tradingview_indicator_prompt.txt`** à la racine du dépôt. Ce fichier permet de retrouver l’intention initiale ou des formulations du besoin en cas de clarification.

---

## 2. TradingView : création et utilisation d’indicateurs personnalisés

Résumé basé sur la documentation officielle Pine Script et le support TradingView (voir sources en fin de section). **Projet MTF : Pine Script v6.**

### 2.1 Indicateur, Stratégie, Bibliothèque : à quoi ça sert, comment ça s’utilise, pourquoi

TradingView distingue **trois types de scripts** selon la **déclaration** en tête du fichier : `indicator()`, `strategy()` ou `library()`. Le type **déverrouille ou verrouille** des fonctionnalités et **définit le mode d’utilisation** (ajout au graphique, backtest, réutilisation de code).

#### Indicateur (`indicator()`)

- **Rôle** : calculer des grandeurs (tendance, zones, niveaux, signaux) et les **afficher sur le graphique** (courbes, flèches, labels, rectangles, etc.). Pas d’ordres, pas de simulation de portefeuille.
- **Utilisation** : script dans l’éditeur Pine → « Ajouter au graphique ». S’exécute sur le symbole et la timeframe du graphique. Paramètres via l’engrenage, alertes possibles.
- **Contexte MTF** : adapté à notre objectif (aide à la décision visuelle, pas de backtest ni d’ordres automatiques).

#### Stratégie (`strategy()`)

- **Rôle** : affichage + **logique de positions** (entrées, sorties, SL/TP). Exécutée par le **Strategy Tester** (P&L, drawdown, etc.) et peut alimenter alertes/ordres réels.
- **Contexte MTF** : non retenu pour l’instant ; utile si on voulait backtester ou automatiser plus tard.

#### Bibliothèque (`library()`)

- **Rôle** : script non ajouté au graphique ; contient des **fonctions/types réutilisables** (`export`). Publiée puis importée dans d’autres scripts.
- **Contexte MTF** : optionnel ; envisageable si on ajoute une stratégie ou un second indicateur partageant la même logique.

#### Synthèse projet MTF

| Besoin | Indicateur | Stratégie | Bibliothèque |
|--------|------------|-----------|--------------|
| Afficher zones, structure, signaux | Oui | Oui (en plus) | Non |
| Backtest (Strategy Tester) | Non | Oui | Non |
| Ordres / alertes | Alertes possibles | Oui | Non |
| Réutiliser du code | — | — | Oui |
| **Notre objectif** | **Adapté** | Inadapté | Optionnel plus tard |

#### Intégrés et scripts communautaires

- **Scripts intégrés** : indicateurs TradingView (RSI, MACD, etc.) ; beaucoup en Pine, consultables via « Code source » puis « Enregistrer sous… ».
- **Scripts communautaires** : publiés par les utilisateurs ; visibilité (open / protégé / invite-only) selon l’auteur et l’abonnement.

### 2.2 Édition du script

- **Pas d’import de fichier .pine** dans l’éditeur Pine (pas de bouton « Ouvrir depuis le disque »).
- **Workflow** : édition dans l’éditeur Pine ou **copier-coller** du contenu d’un fichier local (ex. `indicateur_mtf.pine`).
- **Sauvegarde** : dans le compte TradingView (« Mes scripts »). En local : versioning Git, copier-coller pour synchroniser avec TradingView.

### 2.3 Création et utilisation d’un indicateur personnalisé

- **Nouveau** : Éditeur Pine → Ouvrir → Nouvel indicateur vide.
- **Existant** : Ouvrir → Mes scripts, ou coller le code depuis un fichier local.
- **Structure minimale** : `//@version=6` puis `indicator("Titre", shorttitle="Court", overlay=true)` ; limites utiles : `max_labels_count=500`, `max_boxes_count=500`, etc.
- **Tester** : « Ajouter au graphique » ; paramètres via l’engrenage.
- **Publier** (optionnel) : visibilité publique/privée ; type de code open / protégé / sur invitation selon l’abonnement.

### 2.4 Récap

- Indicateur = affichage et aide à la décision, pas d’ordres ni de backtest → adapté au projet MTF.
- Pas d’import de fichier : édition dans l’éditeur ; fichier local = copier-coller.
- Workflow : écrire/coller → Ajouter au graphique → paramètres → Enregistrer ; publier si besoin.

---

## 3. Sources

- [Pine Script Language Reference (v6)](https://www.tradingview.com/pine-script-reference/v6/)
- [Pine Script Documentation – Script structure](https://www.tradingview.com/pine-script-docs/language/script-structure/)
- [Pine Script Documentation – Libraries](https://www.tradingview.com/pine-script-docs/concepts/libraries/)
- [Pine Script Documentation – Publishing scripts](https://www.tradingview.com/pine-script-docs/writing/publishing/)
- [Pine Script Release notes](https://www.tradingview.com/pine-script-docs/release-notes/) (v6, dynamic requests, etc.)
- [Support : Voir le code source d’un indicateur intégré](https://fr.tradingview.com/support/solutions/43000481659)
- [Support : Créer un nouveau script](https://www.tradingview.com/support/solutions/43000711497)
- [Règles de publication des scripts](https://www.tradingview.com/support/solutions/43000590599)
- Stack Overflow : [Upload Pine Script from local file](https://stackoverflow.com/questions/74443194/) (pas d’upload possible)
