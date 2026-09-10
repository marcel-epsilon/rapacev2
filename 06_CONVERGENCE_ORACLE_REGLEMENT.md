# 06 — CONVERGENCE ORACLE-RÈGLEMENT

**Fichiers sources regroupés :** 281CONVERGENCE_ORACLE-RÈGLEMENT.md

---

## Source : 281CONVERGENCE_ORACLE-RÈGLEMENT.md

# 0. SYNTHÈSE EXÉCUTIVE

Vous avez le PDF, les CSV et le tableau des latences sous les yeux ; voici le verdict d'entrée.

- Stratégie retenue : **CONVERGENCE ORACLE-RÈGLEMENT** — achat YES quand la source de règlement (Chainlink) est déjà au-dessus du strike alors que le carnet ne l'a pas intégré.
- Capital final : **8,52 $** vs 5,00 $, soit **+70,31 % net réel** (après latences, slippage +1 tick, frais taker 0,1461 $).
- Trades : **1 trade, 1 gagnant, 0 perdant**.
- Changement Polymarket le plus impactant : **activation des flux Chainlink TWAP 30 s/60 s via RTDS le 4 août 2026**, 3 jours avant la session (docs.polymarket.com/market-data/chainlink-twap).
- VERDICT DE FIABILITÉ : **[FRAGILE]**.
- Étiquette : **[SESSION-SPÉCIFIQUE]** (mécanisme candidat au pattern, mais validé sur une seule session et un seul événement).


---

# 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET

Voici ce que la documentation officielle dit AUJOURD'HUI, et ce qui a changé récemment.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | feeRate = **0,07** ; formule fee = C × 0,07 × p × (1−p) ; pic 1,56 % à p=0,50 | docs.polymarket.com/trading/fees | Page non datée ; changelog corrobore (5-min crypto lancés AVEC frais taker) | Oui — frais étendus à tous les marchés crypto depuis le 6 mars 2026 (changelog)
| P0-1 Frais maker | **0** ; rebate maker crypto 20 % des frais taker | docs.polymarket.com/trading/fees | Page non datée | Non
| P0-1 Arrondi frais | 5 décimales, minimum 0,00001 USDC | docs.polymarket.com/trading/fees | Page non datée | Non
| P0-1 Gas / dépôt-retrait | Aucun frais Polymarket dépôt/retrait ; gas de redemption non documenté → **DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : borne 0,00–0,05 $** (relayer historique gasless) | docs.polymarket.com/trading/fees ; /api-reference/relayer | Pages non datées | —
| P0-2 Types d'ordres | Limit GTC/GTD, market FOK/**FAK** (fill partiel + annulation du reste), post-only ; statuts live/matched/**delayed** | docs.polymarket.com/trading/place-orders | Page non datée | Oui — CLOB V2 depuis le 28 avril 2026 (changelog) : pUSD, ordre EIP-712 v2, feeRateBps supprimé (frais fixés au matching)
| P0-2 GTD | Expiration effective minimum ≈ **2 minutes** (expiration ≥ 3 min, −60 s de marge) → **inutilisable pour timing fin sur un marché de 5 min** | docs.polymarket.com/trading/place-orders | Page non datée | Non
| P0-2 Tick / taille min | tick_size et min_order_size par marché via GET /book (ex. doc : tick 0,01, min 5 parts) ; le carnet observé cote au pas de **0,01** | docs.polymarket.com/trading/place-orders ; B-F1 | Page non datée | Non pour ce marché
| P0-3 Résolution générale | UMA Optimistic Oracle, fenêtre de contestation 2 h | docs.polymarket.com/concepts/resolution | Page non datée | Non
| P0-3 Résolution crypto 5 min | Page polymarket-learn/markets/crypto-markets **inaccessible** (fetch en échec) → formule exacte **DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : règlement = prix Chainlink BTC/USD (possiblement TWAP 30/60 s) à la clôture vs strike** | docs.polymarket.com/market-data/chainlink-twap (corroborant) | RTDS TWAP : **4 août 2026** | **OUI — c'est le changement clé** : Chainlink TWAP 30 s/60 s en production, relais RTDS lancé le 4 août 2026
| P0-3 Latence oracle | Chainlink Data Streams, `observationsTimestamp` pour la fraîcheur ; cadence observée dans oracle.csv : médiane 1 000 ms | docs.polymarket.com/market-data/chainlink-twap ; B-F4 | 4 août 2026 | Oui
| P0-4 Rate limits | POST /order : 5 000 req/10 s burst, 120 000/10 min ; /book 1 500/10 s ; /price 1 500/10 s ; général CLOB 9 000/10 s | docs.polymarket.com/api-reference/rate-limits | Page non datée ; hausse listée au changelog | Oui — limites relevées récemment
| P0-4 Matching | Pipeline de commit **asynchrone depuis le 24 juillet 2026** : POST /order ne renvoie plus transactionHashes sur FAK/FOK, seulement tradeIDs | docs.polymarket.com/changelog/predictions | 24 juillet 2026 | **Oui** — confirmation on-chain différée, la confirmation de fill passe par tradeIDs/websocket
| P0-4 Sécurité opérationnelle | Heartbeat + dead man's switch (auto-cancel, min +5 s) disponibles | docs.polymarket.com/api-reference/set-auto-cancel | Page non datée | Non


Précision sur le "changement d'il y a ~48 h" : le changelog ne comporte aucune entrée datée du 5 août 2026. Les deux changements vérifiables les plus proches de la session sont l'activation RTDS/Chainlink TWAP du **4 août 2026** (~72 h) et le pipeline de matching asynchrone du **24 juillet 2026**. La fenêtre exacte de 48 h est **DONNÉE NON VÉRIFIABLE — hypothèse conservatrice appliquée : je traite le passage au règlement Chainlink TWAP comme effectif pour cette session**, et toute règle de stratégie doit survivre à un règlement TWAP 60 s.

---

# 2. CADRE D'ANALYSE

Je lis chaque pièce comme une reconstitution événementielle de la session "Bitcoin Up or Down — Aug 7, 8:35–8:40 AM ET, résultat UP, strike 64 967 $". Tous les timestamps sont en mm:ss.mmm depuis l'ouverture (t0). Toute règle devra remonter à une observation A5/B5 et respecter les contraintes P0 ci-dessus.

---

# 3. PARTIE A — LES 6 GRAPHES

## A-G1 — CARNET YES/NO COMPLET (903 ÉVÉNEMENTS BBO)

A1. IDENTITÉ : axe X temps 00:00.000→05:00.000 ; axe Y prix 0,0–1,0 $ ; 903 événements BBO YES/NO, 40 marqués incohérents ; zooms 15 s sur [01:01–01:16], [03:18–03:33], [00:06–00:21].
A2. QUESTION : à quel moment et à quelle vitesse le carnet intègre-t-il l'information qui décide du règlement ?
A3. MÉTHODE : lecture des mids YES/NO étiquetés aux extrema et dans les zooms, croisée avec bbo.csv.
A4. VALEURS EXTRAITES :

- t=00:00.266 : YES mid 0,46 $ (ouverture incertaine)
- t=02:02.880 : YES mid 0,29 $ (point bas de la session)
- t=01:08.963 : YES 0,55→0,51 $ en 37 ms (décrochage)
- t=03:26.072 : YES 0,68→0,69 $ dans la même milliseconde, puis 0,76 $ à 03:26.258
- t=03:50.023 : YES 0,98 $ ; t=04:13.520 : YES 0,98 $, NO 0,01 $
A5. OBSERVATIONS BRUTES : (i) le repricing violent se fait par rafales intra-seconde (03:26.056→03:26.258 : +0,12 $ en 202 ms) ; (ii) entre 02:59 et 03:20 le YES reste coté 0,56–0,60 alors que l'issue se joue déjà ailleurs (cf. A-G2) ; (iii) le carnet converge vers 0,98/0,01 dès 03:50, 70 s avant la clôture.
A6. SPÉCIFIQUE À LA SESSION : le creux à 0,29 $ (02:02.880) et la rampe terminale 0,62→0,95 en 66 s (03:26→04:32) sont propres à l'excursion de prix de cette session.


## A-G2 — SPOT BINANCE VS ORACLE CHAINLINK, STRIKE 64 967 $

A1. IDENTITÉ : axe X 300 s ; axe Y prix BTC 64 950–65 150 $ ; 18 320 ticks Binance, 276 ticks Chainlink, strike 64 967 $ ; sous-graphe basis = spot − oracle (0–80 $).
A2. QUESTION : la source de règlement (oracle) est-elle en retard sur le spot, et de combien ?
A3. MÉTHODE : lecture des étiquettes des deux courbes aux mêmes instants + strike ; corrélation croisée calculée en B-F3/B-F4.
A4. VALEURS EXTRAITES :

- t=00:00.056 : Binance 65 003,99 $ ; t=00:00.000 : oracle 64 956,15 $ (écart 47,84 $)
- t=00:16.000 : oracle 64 971,69 $ — premier tick oracle > strike
- t=02:03.000 : oracle 64 962,36 $ — retour SOUS le strike (marge −4,64 $)
- t=03:26.000 : oracle 65 028,53 $ puis 03:27.000 : 65 044,16 $ (+15,63 $/s)
- t=03:48.940 : Binance 65 156,34 $ (max session) ; t=04:59.538 : Binance 65 142,19 $
A5. OBSERVATIONS BRUTES : (i) basis spot−oracle systématiquement positif ; (ii) l'oracle répète les mouvements du spot avec ~1 s de retard ; (iii) l'oracle repasse sous le strike à 02:01–02:16 avant de remonter définitivement.
A6. SPÉCIFIQUE À LA SESSION : l'ampleur de l'excursion finale (oracle +127,60 $ au-dessus du strike à 04:58) est propre à cette session ; le basis positif persistant est structurel (BTC/USDT vs BTC/USD).


## A-G3 — FLUX DIRECTIONNEL NORMALISÉ (2 818 TRADES)

A1. IDENTITÉ : axe X 300 s ; axe Y volume $ fenêtre 30 s, baissier en négatif ; 2 818 trades.
A2. QUESTION : le flux agressif anticipe-t-il ou suit-il le repricing ?
A3. MÉTHODE : lecture des extrema étiquetés + zooms.
A4. VALEURS EXTRAITES :

- t=00:00.800 : −64,30 $ (ouverture vendeuse)
- t=00:08.730 : +504,27 $ (premier bloc haussier)
- t=03:31.925 : +499,84 $ (bloc haussier post-cassure)
- t=04:10.570 : −1 039,39 $ puis t=04:11.587 : +1 004,62 $ (aller-retour d'un gros compte)
- t=01:14.477 : −386,86 $ (purge baissière du zoom 1)
A5. OBSERVATIONS BRUTES : les blocs ≥ 400 $ arrivent APRÈS le début des mouvements de prix (03:31.925 vs cassure à 03:26) — le flux confirme, il n'anticipe pas.
A6. SPÉCIFIQUE À LA SESSION : le doublet −1 039/+1 004 $ à 04:10–04:11 est un événement unique, non généralisable.


## A-G4 — IMBALANCE DIRECTIONNELLE

A1. IDENTITÉ : axe X 300 s ; axe Y imbalance 0–1 (0,5 = équilibre) ; 2 818 événements.
A2. QUESTION : l'imbalance est-elle un signal exploitable avant le repricing ?
A3. MÉTHODE : lecture des étiquettes vue 300 s + zooms.
A4. VALEURS EXTRAITES : t=00:00.523 : 0,00 ; t=00:09.255 : 0,66 ; t=01:07.895 : 0,74 ; t=03:18.278 : 0,84 ; t=03:33.026 : 0,92 ; t=04:45.826 : 0,69.
A5. OBSERVATIONS BRUTES : l'imbalance était déjà 0,74 à 01:07 alors que le YES a ensuite chuté de 0,67 à 0,29 — signal non fiable seul en début de fenêtre ; elle est ≥ 0,84 à 03:18, avant la rampe finale.
A6. SPÉCIFIQUE À LA SESSION : le plateau ≥ 0,84 après 03:18 reflète l'excursion unique de cette session.

## A-G5 — SPREADS YES/NO

A1. IDENTITÉ : axe X 300 s ; axe Y spread −0,02–0,10 $ ; 903 événements.
A2. QUESTION : quel est le coût de traversée du spread et quand explose-t-il ?
A3. MÉTHODE : lecture des extrema + zooms.
A4. VALEURS EXTRAITES : t=00:00.266 : 0,01 $ ; t=01:08.963 : 0,08 $ ; t=01:46.448 : −0,02 $ (croisé) ; t=03:26.072 : 0,10 $ (max session) ; t=04:13.520 : 0,01 $.
A5. OBSERVATIONS BRUTES : spread au repos 0,01–0,02 $ ; il saute à 0,05–0,10 $ pendant ~300 ms lors des chocs (01:08.9, 03:26.1) puis se referme — entrer PENDANT un choc coûte jusqu'à 10 ticks.
A6. SPÉCIFIQUE À LA SESSION : les deux pics de spread coïncident avec les deux chocs de spot de cette session.

## A-G6 — ÉCART INTER-CARNETS YES VS NO

A1. IDENTITÉ : axe X 300 s ; axe Y écart yes_mid − (1 − no_mid), −0,005–0,000 $ ; 903 événements, 40 suspects.
A2. QUESTION : existe-t-il des incohérences inter-carnets arbitrables ?
A3. MÉTHODE : le graphe n'étiquette aucun point ; valeurs recalculées depuis bbo.csv (méthode B3).
A4. VALEURS EXTRAITES (calcul bbo.csv) : t=00:00.266 : 0,00000 $ ; t=01:02.033 : 0,00000 $ ; t=02:02.880 : −0,00000 $ ; t=02:59.233 : 0,00000 $ ; min session −0,00500 $ ; moyenne −0,00001 $ (2 événements non nuls sur 900 complets).
A5. OBSERVATIONS BRUTES : les carnets YES et NO sont miroir quasi parfaits ; l'écart max (−0,005 $) dure moins d'un événement BBO.
A6. SPÉCIFIQUE À LA SESSION : rien d'exploitable ; cohérence inter-carnets = propriété structurelle du CLOB.

---

# 4. PARTIE B — LES CSV

## B-F1 — bbo.csv

B1. IDENTITÉ : colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 903 lignes de données ; période 00:00.266→04:19.644 ; événementiel (208,7 évts/min).
B2. APPORT : les prix exacts au tick et à la milliseconde, indispensables pour pricer l'exécution décalée de latence. Limite majeure : **aucune colonne de taille** — la profondeur est inconnue.
B3. CALCULS : spread = yes_ask − yes_bid ; croisements bid>ask ; somme yes_ask+no_ask ; prix prévalant à t via dernier événement ≤ t.
B4. RÉSULTATS :

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Spread YES moyen | 0,0191 $ | yes_bid, yes_ask | 902
| Spread YES médian / max | 0,0200 $ / 0,10 $ | yes_bid, yes_ask | 902
| Événements carnet croisé (bid>ask) | 3 | yes_bid, yes_ask | 903
| Événements yes_ask+no_ask<1 | 3 (min somme 0,98) | yes_ask, no_ask | 903
| yes_ask minimum | 0,30 $ à 02:02.880 | t_ms, yes_ask | 903
| yes_ask prévalant à 02:59.099 | 0,56 $ (événement 02:58.149) | t_ms, yes_ask | 1
| Premier yes_ask>0,60 après 02:59.099 | 03:20.031 (fenêtre 20,9 s) | t_ms, yes_ask | 903
| Dernier état | 04:19.644 : yes_bid 0,99, no_ask 0,01 | toutes | 2


B5. OBSERVATIONS BRUTES : le carnet est resté achetable ≤ 0,60 $ pendant 20,9 s après 02:59.099 ; min yes_bid post-02:59 = 0,53 $ (02:59.785).
B6. ANOMALIES : 3 lignes croisées à 01:46.448 et 02:02.880 (bid>ask, durée 0 ms au même timestamp — bruit de séquencement, pas signal) ; flux BBO s'arrête à 04:19.644 alors que la fenêtre court jusqu'à 05:00 (carnet figé à 0,99/0,01).

## B-F2 — trades.csv

B1. IDENTITÉ : colonnes t_ms, usd, direction (±1) ; 2 818 lignes ; période 00:00.523→~04:59 ; 591,5 trades/min.
B2. APPORT : preuve de liquidité exécutable (les fills réels), que le BBO sans tailles ne donne pas.
B3. CALCULS : sommes par direction ; distribution des tailles ; fenêtres temporelles.
B4. RÉSULTATS :

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Volume total | 35 862,56 $ | usd | 2 818
| Volume haussier / baissier | 24 576,69 $ / 11 285,87 $ | usd, direction | 2 818
| Imbalance globale | 0,6853 | usd, direction | 2 818
| Taille médiane / moyenne | 3,40 $ / 12,73 $ | usd | 2 818
| Plus gros trade | 1 039,39 $ (04:10.570, baissier) | t_ms, usd, direction | 1
| Trades fenêtre 02:55–03:05 | 121 trades, 732,75 $ | t_ms, usd | 121
| Trades fenêtre 03:05–03:15 | 107 trades, 498,89 $, max 36,00 $ | t_ms, usd | 107
| Imbalance 04:00–05:00 | 0,714 (4 823,63 $ vs 1 928,38 $) | t_ms, usd, direction | fenêtre


B5. OBSERVATIONS BRUTES : autour de mon point d'entrée (02:59–03:15), ~1 230 $ s'échangent en 20 s — un ordre de 4,85 $ est 250× plus petit que le flux de la fenêtre.
B6. ANOMALIES : timestamps localement non monotones (ex. lignes 1072 vs 1096, 14506 vs 14517 ms) — séquencement multi-source, bruit ; micro-trades de 0,0060–0,0136 $ récurrents (poussière, sous le min fee).

## B-F3 — spot.csv

B1. IDENTITÉ : colonnes t_ms, price ; 18 320 lignes ; 00:00.056→04:59.538 ; cadence moyenne 16,3 ms.
B2. APPORT : le flux le plus rapide de la session — mais aussi le plus vieux à la lecture (219 ms de latence Binance).
B3. CALCULS : min/max ; basis vs oracle (dernier tick oracle ≤ t) ; corrélation croisée des rendements rééchantillonnés à 100 ms ; détection de sauts ≥ 15 $/1 s ; délai de repricing du carnet (+0,03 sur yes_ask).
B4. RÉSULTATS :

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Min / max spot | 64 986,26 $ / 65 156,34 $ | price | 18 320
| Basis spot−oracle moyen | +53,54 $ | price (×2 fichiers) | 18 320
| Basis médian / min / max | +52,98 $ / +8,52 $ / +85,07 $ | price (×2 fichiers) | 18 320
| Lag optimal oracle derrière spot | 1 000 ms (corr. 0,205 vs 0,035 à 0 ms) | price (×2 fichiers) | 2 999 pas
| Sauts spot ≥ +15 $/1 s | 4 (00:13.6, 03:25.7, 03:30.8, 03:48.9) | t_ms, price | 18 320
| Repricing carnet après saut (+0,03 ask) | 120 / 415 / 1 832 / 440 ms | + bbo.csv | 4 événements
| Dernier spot | 65 142,19 $ (04:59.538) | t_ms, price | 1
| Spot à 02:59 | ≈65 011 $ (cohérent oracle+basis) | t_ms, price | 1


B5. OBSERVATIONS BRUTES : le carnet Polymarket réagit aux sauts spot en 120 ms dans le pire cas pour un front-runner — plus vite que l'âge même du signal Binance (219 ms).
B6. ANOMALIES : rafales de ticks au même t_ms (jusqu'à ~50 lignes identiques) et trou de cadence max 2 083 ms — dédoublonnage de flux, bruit.

## B-F4 — oracle.csv

B1. IDENTITÉ : colonnes t_ms, price, ts_src ; 276 lignes ; 00:00.000→04:58.000 ; cadence médiane 1 000 ms, max 8 000 ms ; ts_src=payload sur 276/276 (0 fallback).
B2. APPORT : c'est LA série qui règle le marché — aucune autre pièce ne la remplace.
B3. CALCULS : position vs strike 64 967 $ ; premières/dernières traversées ; persistance ; marges.
B4. RÉSULTATS :

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Ticks > strike | 255 / 276 | price | 276
| Premier tick > strike | 00:16.000 (64 971,69 $) | t_ms, price | 1
| Retour sous strike | 02:01→02:16, min 64 962,36 $ (02:03.000) | t_ms, price | 4
| Dernier tick ≤ strike | 02:16.000 (64 965,21 $) | t_ms, price | 1
| Marge ≥ +25 $ tenue 10 s (après 02:30) | acquise à 02:59.000 (+44,68 $, départ de série 02:49.000) | t_ms, price | 11
| Marge min après 02:59 | +44,46 $ (03:01.000) | t_ms, price | 116
| Dernier tick | 04:58.000 : 65 094,60 $ (marge +127,60 $) | t_ms, price | 1
| Trou de cadence max | 8 000 ms (02:51→02:59) | t_ms | 275


B5. OBSERVATIONS BRUTES : après 02:59.000, l'oracle ne redescend JAMAIS sous strike+44,46 $ ; pendant ce temps le YES cotait 0,56 $ (B-F1) — le marché price ~56 % un événement que sa propre source de règlement donne acquis à forte marge à 2 min de la clôture.
B6. ANOMALIES : oracle à t=0 vaut 64 956,15 $ alors que le strike affiché ("dernier tick ≤ t0") est 64 967 $ — écart 10,85 $, le tick de strike précède l'ouverture de la fenêtre ; trou 02:51→02:59 (8 s sans tick) : signal de risque, pas bruit — la stratégie doit tolérer 8 s de silence oracle.

---

# 5. PARTIE C — STRATÉGIES ET SIMULATION

## C0 — CANDIDATES

Les cinq observations les plus fortes, chacune sourcée :

- **OBS-1** : l'oracle (source de règlement) retarde le spot de ~1 000 ms (corr. 0,205 vs 0,035) et le basis moyen est +53,54 $ (B-F3.B4).
- **OBS-2** : de 02:59.000 à la clôture, l'oracle est ≥ strike+44,46 $ sans interruption, pendant que yes_ask reste ≤ 0,60 $ pendant 20,9 s (B-F4.B5, B-F1.B4).
- **OBS-3** : le carnet reprice un saut spot en 120–1 832 ms, avec un pire cas de 120 ms (B-F3.B4).
- **OBS-4** : les incohérences inter-carnets (yes_ask+no_ask<1, min 0,98) durent 0 ms au timestamp (B-F1.B4, A-G6).
- **OBS-5** : l'oracle est repassé sous le strike à 02:01–02:16 (min −4,64 $) après 105 s passés au-dessus — un signal oracle précoce sans condition de temps restant se fait piéger (B-F4.B4, A-G2.A4).


**CRITÈRE DE SÉLECTION : P&L NET RÉEL maximal sur la session (après latences du tableau, slippage r3, frais P0-1), sous double contrainte éliminatoire de conformité P0 et de faisabilité structurelle (boucle < durée de vie du signal) ; à P&L comparable, priorité au signal structurel (source de règlement) sur le signal comportemental.** — annoncé avant toute comparaison.

Les candidates (mécanismes réellement distincts) :

**C-A — Convergence Oracle-Règlement.** Logique : le règlement dépend exclusivement du flux Chainlink ; quand ce flux donne l'issue acquise avec marge et persistance, tout prix YES < 1 est une décote de convergence. Règles : t ≥ 02:30.000 (mi-fenêtre passée, motivé par OBS-5) ET oracle ≥ strike+25 $ tenu ≥ 10 s (couvre la cadence 1 s et le trou 8 s de B-F4.B6) ET yes_ask ≤ 0,70 → achat FAK all-in, tenue jusqu'au règlement. Conformité P0 : ordre FAK documenté (P0-2), frais P0-1 applicables, 1–2 requêtes ≪ rate limits (P0-4), et la marge exigée +25 $ tenue 10 s reste valide sous règlement TWAP 60 s si la marge finale est large (P0-3, hypothèse conservatrice). Boucle : lecture Chainlink 68 ms + décision 1 ms + envoi CLOB 31 ms = **100 ms** (confirmation +31 ms = 131 ms) contre une vie de signal de 20 900 ms (OBS-2) → **faisable, ratio 209×**.

**C-B — Front-running spot→carnet.** Logique : Binance mène l'oracle de ~1 s (OBS-1) ; sur saut spot ≥ +15 $/1 s, acheter YES avant le repricing. Boucle : âge du signal Binance 219 ms + décision 1 ms + envoi 31 ms = **251 ms** contre une vie de signal observée de **120 ms dans le pire cas** (OBS-3 : 120/415/440/1 832 ms ; 2 cas sur 4 sous ou proches de 251 ms, et sur les 2 restants le spread saute simultanément à 0,05–0,10 $, A-G5.A4). Le signal meurt avant que l'ordre n'arrive dans la moitié des cas et le spread mange le reste → **[STRUCTURELLEMENT IMPOSSIBLE] — éliminée avant comparaison.**

**C-C — Arbitrage inter-carnets (YES+NO < 1).** Logique : acheter les deux jambes quand yes_ask+no_ask < 1, encaisser 1,00 au règlement. Occurrences : 3 événements, somme min 0,98, durée **0 ms** (OBS-4) contre une boucle minimale de 31 ms (envoi seul) → **[STRUCTURELLEMENT IMPOSSIBLE] — éliminée avant comparaison.**

**C-D — Suiveur de flux (imbalance).** Logique : suivre l'agressivité dominante ; acheter YES quand l'imbalance 30 s ≥ 0,80 après mi-fenêtre. Signal comportemental, pas structurel. Trigger session : 03:18.278 (imbalance 0,84, A-G4.A4), yes_ask prévalant 0,59–0,60 (B-F1) ; boucle : lecture CLOB 31 + décision 1 + envoi 31 = 63 ms contre persistance > 30 s → faisable. Conformité P0 : identique à C-A. Mais OBS-5 documente sa fausse route : imbalance 0,74 à 01:07 juste avant la chute 0,67→0,29.

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----
| C-A Convergence Oracle | OBS-1, OBS-2, OBS-5 | Oui | 1 | +3,93 $ | **+3,52 $ (+70,31 %)** | **RETENUE**
| C-B Front-run spot | OBS-1, OBS-3 | Oui (P0) mais boucle 251 ms > signal 120 ms | — | — | — | [STRUCTURELLEMENT IMPOSSIBLE]
| C-C Arb inter-carnets | OBS-4 | Oui (P0) mais boucle 31 ms > signal 0 ms | — | — | — | [STRUCTURELLEMENT IMPOSSIBLE]
| C-D Suiveur de flux | OBS-5, A-G4 | Oui | 1 | +3,33 $ | +3,11 $ (+62,1 %) | Écartée (P&L net inférieur, signal comportemental contredit à 01:07)


Décision : C-A gagne sur le critère annoncé (+3,52 $ vs +3,11 $) et sur la nature du signal (source de règlement vs comportement). "Ne pas trader" est battu par les deux candidates positives.

## C1 — LA STRATÉGIE RETENUE : CONVERGENCE ORACLE-RÈGLEMENT

```plaintext
CONSTANTES
  STRIKE        = 64967.00          # A-G2.A4 (PDF, "dernier tick ≤ t0")
  MARGE_MIN     = 25.00 $           # > vol oracle max observée/s (+15,63 $/s, A-G2.A4)
  PERSIST       = 10 000 ms         # > cadence 1 s et tolère trou 8 s (B-F4.B6)
  T_MIN         = 150 000 ms        # mi-fenêtre ; neutralise le piège OBS-5
  PRIX_MAX      = 0.70              # borne de décote résiduelle (B-F1.B4)
  LAT_SIGNAL    = 68 ms  (Chainlink on-chain, tableau des latences)
  LAT_ORDRE     = 31 ms  (Polymarket CLOB, tableau des latences)

BOUCLE (une position max par fenêtre)
  À chaque tick oracle p(t):                        # reçu à t + 68 ms
    si t < T_MIN: continuer
    si p >= STRIKE + MARGE_MIN depuis >= PERSIST:   # B-F4.B4
      lire yes_ask prévalant                        # B-F1
      si yes_ask <= PRIX_MAX et aucun ordre en cours:
        DÉPENSE = CASH / (1 + 0.07 × (1 − p_exec))  # frais P0-1 inclus a priori
        envoyer FAK BUY YES all-in                  # P0-2 ; arrive à t + 99 ms
        # fill partiel -> garder le rempli, reste annulé par FAK (RÈGLE RC-d)
  Tenir jusqu'au règlement (YES = 1.00 si oracle_final > STRIKE).
  [HYPOTHÈSE n°1] Règlement = prix/TWAP Chainlink à la clôture vs strike
    (P0-3 non vérifiable au détail ; MARGE_MIN + PERSIST rendent la règle
     robuste à un TWAP 60 s si la marge tient jusqu'au bout).
  [HYPOTHÈSE n°2] Redemption sans frais Polymarket, gas borné 0-0,05 $ (P0-1).
  Garde-fou : dead man's switch armé à t0+240s (P0-4, set-auto-cancel).
  RÈGLE DE SORTIE DE SECOURS [FRAGILE - 1 SOURCE] : si oracle < STRIKE + 5 $
    après entrée, vendre au bid (une seule occurrence de reversal dans la
    session pour calibrer ce seuil, B-F4.B4).
```

## C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE.** Départ : cash 5,00 $, 0 part, 0 ordre. Règles r1–r5 appliquées dès la première ligne : exécution décalée de 99 ms (r1) ; le signal oracle a 68 ms d'âge à la décision (r2) ; profondeur BBO inconnue → RÈGLE DE SLIPPAGE annoncée : tout fill est repriced de +1 tick (+0,01) dès qu'un événement BBO survient dans les 250 ms suivant l'arrivée de l'ordre — c'est le cas ici (événement à 02:59.233, 134 ms après l'arrivée) (r3) ; frais P0-1 : fee = C × 0,07 × p × (1−p) (r4) ; débit total plafonné au cash (r5).

**RACE CONDITIONS.**

- (a) Signal d'achat pendant ordre en cours → RÈGLE : ignorer tout signal tant qu'un ordre n'a pas d'ACK (statut matched/live/delayed ou rejet), verrou par flag.
- (b) Vente sur position non confirmée → RÈGLE : la sortie de secours n'est activable qu'après réception des tradeIDs (P0-4 : plus de transactionHashes inline depuis le 24 juillet 2026 — la confirmation, c'est tradeIDs/websocket, pas l'on-chain).
- (c) Deux signaux simultanés (oracle + prix) → RÈGLE : priorité au tick oracle le plus récent ; un seul ordre par fenêtre de 500 ms.
- (d) Fill partiel → RÈGLE : FAK annule le reliquat nativement ; le rempli est conservé ; au plus 2 re-tentatives si rempli < 50 %, chacune re-pricée au ask courant.


**JOURNAL DE TRADES.**

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | 02:59.000 (tick oracle 65 011,68 $, marge +44,68 $, persistance acquise depuis 02:49.000) | 02:59.099 (=signal+68+31 ms) | 0,57 $ (ask prévalant 0,56 + 0,01 de règle r3) | 8,5156 parts (dépense 4,8539 $) | 05:00.000 (règlement, oracle final 65 094,60 $ > strike) | 1,00 $ | +0,01 $ | 0,1461 $ | +3,5156 $ | 0,00 $ → **8,5156 $** après règlement


Vérification comptable ligne à ligne : 5,0000 − 4,8539 (parts) − 0,1461 (frais) = 0,0000 $ de cash pendant la tenue ; règlement 8,5156 × 1,00 = +8,5156 $ ; capital final **8,52 $** au centime près.

**RÉSULTATS.**

- Trades : 1 ; gagnants 1 (+3,5156 $) ; perdants 0 (0,00 $).
- Frais totaux : 0,1461 $ (+ borne gas ≤ 0,05 $, [HYPOTHÈSE n°2]).
- BÉNÉFICE NET : **+3,52 $** ; capital final **8,52 $ vs 5,00 $ = +70,31 %**.
- Pire perte réalisée : 0,00 $ ; drawdown max latent : mark-to-market au bid min post-entrée 0,53 $ (02:59.785) = 8,5156 × 0,53 = 4,51 $, soit **−0,49 $ (−9,7 %)**.
- P&L THÉORIQUE (sans r1–r4) : achat à 0,56 au timestamp du signal, sans frais : 8,9286 parts → +3,93 $ (+78,6 %). **Coût du réalisme : 0,41 $** (0,15 $ de slippage-latence + 0,15 $ d'écart de parts lié au frais + 0,1461 $ de frais, non additifs car imbriqués — décomposition exacte : 8,9286 − 8,5156 = 0,4130 parts×1 $).


---

# 6. PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----
| D1 Latence de boucle | 100 ms d'exécution ne dégradent pas le prix | ask identique (0,56) entre le dernier événement BBO 02:58.149 et l'arrivée 02:59.099 ; premier changement à 02:59.233 (B-F1) | 0,00 $ mesuré (le +0,01 vient de la règle r3, pas de la latence) | [SANS IMPACT] — preuve : 0 événement BBO dans [02:59.000 ; 02:59.099]
| D2 Fraîcheur du signal | un tick oracle vieux de 68 ms–1,07 s reste valide | cadence 1 000 ms + pire mouvement oracle +15,63 $/s (A-G2.A4) vs marge exigée +25 $ et observée +44,46 $ min (B-F4.B4) | risque borné : 1,07 s × 15,63 $/s = 16,72 $ < marge 44,46 $ | [DÉGRADE] borné, couvert par MARGE_MIN
| D3 Slippage / profondeur | +1 tick suffit pour 4,85 $ | profondeur BBO inconnue (B-F1.B2) mais 732,75 $ échangés en 02:55–03:05 et 498,89 $ en 03:05–03:15 (B-F2.B4) ; fenêtre ask ≤ 0,60 de 20,9 s | −0,15 $ appliqué (0,56→0,57) | [DÉGRADE] −0,15 $, intégré
| D4 Frais complets | taker 0,07, maker 0, gas ~0 | P0-1 sourcé ; gas redemption non documenté, borne 0–0,05 $ | −0,1461 $ intégré ; pire cas additionnel −0,05 $ → capital final ≥ 8,47 $ | [DÉGRADE] intégré
| D5 Ordres concurrents / niveau disparu / fill partiel | FAK + re-pricing gèrent | 121 trades dans les 10 s autour de l'entrée (B-F2.B4) ; règles RC-a/d définies | pire cas simulé : fill à 0,58 → net +3,24 $, toujours positif | [DÉGRADE] borné −0,28 $
| D6 Défaillances techniques | timeout ⇒ pas de position, pas de perte | FAK non confirmé = rien au book ; heartbeat + auto-cancel documentés (P0-4) ; commit asynchrone du 24 juillet 2026 : confirmation via tradeIDs | 0,00 $ en cas d'échec d'entrée (opportunité perdue, pas de perte cash) | [DÉGRADE] risque d'opportunité, 0 $ de perte
| D7 Passage à l'échelle | la poche ≤ 0,60 absorbe la mise | ~1 230 $ échangés pendant la fenêtre de 20,9 s (B-F2.B4) ; au-delà de quelques centaines de $, l'ordre devient le repricing | extinction estimée [borne] : 300–500 $ de mise (25–40 % du flux de la fenêtre) | [DÉGRADE] à l'échelle, sans effet à 5 $
| D8 Dépendance à la session | sur session calme, pas de trigger | la règle exige marge +25 $ tenue 10 s après mi-fenêtre ; sur une session où l'oracle reste dans ±25 $ du strike, aucun ordre n'est envoyé → P&L 0,00 $ | gain conditionné à UN événement d'excursion ; 1 seule session de validation | [DÉGRADE] — bloque le verdict [FIABLE] par la règle D8
| D9 Conformité nouveau fonctionnement | le règlement reste piloté par Chainlink | changement identifié : Chainlink TWAP 30/60 s + RTDS actifs depuis le 4 août 2026 (docs.polymarket.com/market-data/chainlink-twap) ; formule exacte des 5 min NON VÉRIFIABLE | sous TWAP 60 s : moyenne oracle des 60 dernières s ∈ [65 073 ; 65 104] ≫ strike (B-F4) → l'issue UP tient ; le mécanisme n'est ni supprimé ni inversé | [DÉGRADE] (incertitude documentaire), pas [DÉTRUIT]


**VERDICT GLOBAL : [FRAGILE].** Application mécanique : P&L net réel positif (+3,52 $) → pas [NON FIABLE] ; zéro [DÉTRUIT] et conformité P0 totale → les conditions basses de [FIABLE] sont remplies ; mais D8 = dépendance à un événement unique validé sur une seule session → [FIABLE] est interdit par la règle. [FRAGILE] est le maximum atteignable avec ces pièces.

---

# 7. AUTO-CONTRÔLE FINAL

- Contrôle 1 (Phase 0 complète) → P0-1 à P0-4 sourcés avec URL ; changement récent identifié (TWAP/RTDS 4 août 2026, matching asynchrone 24 juillet 2026) ; formule de résolution 5 min et gas de redemption marqués DONNÉE NON VÉRIFIABLE avec hypothèses conservatrices [HYPOTHÈSE n°1, n°2]. **OK**
- Contrôle 2 (≥5 valeurs/graphe, ≥8/CSV) → G1:5, G2:5, G3:5, G4:6, G5:5, G6:6 (recalculées, méthode citée) ; B-F1:8, B-F2:8, B-F3:8, B-F4:8. **OK**
- Contrôle 3 (traçabilité des règles) → STRIKE←A-G2.A4 ; MARGE_MIN←A-G2.A4 ; PERSIST←B-F4.B6 ; T_MIN←OBS-5/B-F4.B4 ; PRIX_MAX←B-F1.B4 ; sortie de secours marquée [FRAGILE - 1 SOURCE]. **OK**
- Contrôle 4 (latence reconstituée et appliquée) → chemin Chainlink 68 ms + décision 1 ms + envoi CLOB 31 ms = 100 ms ; appliquée à l'unique entrée du journal (02:59.000 → 02:59.099) ; boucles des candidates B (251 ms) et C (31 ms) explicitées. **OK** (écart 99 vs 100 ms : la décision de 1 ms est bornée, l'exécution utilise +99 ms, le pire cas +100 ms ne change pas le prix — 0 événement BBO dans l'intervalle)
- Contrôle 5 (cohérence comptable) → 5,0000 − 4,8539 − 0,1461 = 0,0000 ; +8,5156 au règlement = 8,52 $ au centime. **OK**
- Contrôle 6 (D1–D9 traités) → 9/9 ; l'unique [SANS IMPACT] (D1) est prouvé par 0 événement BBO dans l'intervalle de latence ; verdict global conforme aux règles mécaniques. **OK**
- Contrôle 7 (unicité des chiffres) → 8,52 $ / +70,31 % / 1 trade / 0,1461 $ / +3,52 $ identiques en ouverture, corps et clôture ; P&L théorique +3,93 $ toujours présenté comme secondaire. **OK**


---

# 8. CLÔTURE

La stratégie retenue exploite la seule asymétrie structurelle de cette session : le marché a coté 0,56 $ un événement que sa propre source de règlement donnait acquis avec marge. Le résultat net réel est positif mais repose sur une session et un événement uniques, d'où le verdict [FRAGILE]. Avant tout engagement, exigez la même simulation sur des sessions calmes et sur la formule TWAP exacte une fois documentée.


