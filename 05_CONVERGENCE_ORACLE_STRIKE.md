# 05 — CONVERGENCE ORACLE-STRIKE

**Fichiers sources regroupés :** 283CONVERGENCE_ORACLE-STRIKE.md, 292CONVERGENCE_ORACLE-STRIKE.md, 290CONVERGENCE_ORACLE-STRIKE_EN_FIN_DE_FENÊTRE.md

---

## Source : 283CONVERGENCE_ORACLE-STRIKE.md

# 0. SYNTHÈSE EXÉCUTIVE

Voici le verdict avant le détail, chaque chiffre réapparaît à l'identique dans le corps.

- Stratégie retenue : **CONVERGENCE ORACLE-STRIKE** (achat YES unique quand l'oracle Chainlink s'installe ≥ strike + 20,00$ pendant 10 s alors que le YES cote ≤ 0,70$, tenue jusqu'à résolution).
- Capital final : **7,27$** vs 5,00$, soit **+45,40%** net, après latences, slippage borné et frais taker réels de 0,11$.
- Trades : **1 trade, 1 gagnant / 0 perdant**.
- Changement Polymarket le plus impactant : depuis le **07/08/2026**, les marchés crypto 5 min se règlent sur un **TWAP Chainlink de 30 secondes** et non plus sur un instantané de prix (docs.polymarket.com/market-data/chainlink-twap, page datée du 04/08/2026 pour le lancement RTDS, consultée le 10/08/2026).
- VERDICT DE FIABILITÉ : **[FRAGILE]** — P&L net réel positif et zéro [DÉTRUIT], mais profondeur du carnet DONNÉE NON DISPONIBLE et calibration sur une seule session.
- Étiquette : **[PATTERN CANDIDAT]** — le mécanisme (retard du prix sur la certitude oracle) est généralisable, sa magnitude ne l'est pas encore.


---

# 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET

Voici l'état vérifié du fonctionnement au 10/08/2026 ; les lignes de ce tableau sont des contraintes dures pour toute la suite.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × 0,07 × p × (1−p) ; maker 0 ; rebate maker 20% des frais | docs.polymarket.com/polymarket-learn/trading/fees | consultée 10/08/2026 | Non (le coefficient sports a changé le 10/07/2026, pas crypto)
| P0-1 Dépôt/retrait | 0$ de frais Polymarket ; gas de rédemption non chiffré dans la doc | même URL | consultée 10/08/2026 | Non
| P0-2 Tick size | 0,01$ typique, lu par marché via `GET /book` | docs.polymarket.com/trading/place-orders | consultée 10/08/2026 | Non (World Cup passé à 0,0025 le 02/07/2026, hors périmètre)
| P0-2 Taille min. d'ordre | 5 actions (`min_order_size`) | même URL | consultée 10/08/2026 | Non
| P0-2 Types d'ordres | Limit GTC/GTD, market FAK/FOK ; statuts live/matched/delayed ; **GTD : expiration ≥ 3 min dans le futur** | même URL | consultée 10/08/2026 | Oui : depuis le 24/07/2026, `POST /order` ne renvoie plus `transactionHashes` (docs.polymarket.com/changelog/predictions, 17/07/2026)
| P0-3 Résolution crypto 5 min | **TWAP Chainlink 30 s** (60 s pour 15 min/4 h), via Chainlink Data Streams ou Polymarket RTDS (lancé 04/08/2026) ; effectif 07/08/2026 | docs.polymarket.com/market-data/chainlink-twap + recherche presse du 07/08/2026 | 04/08/2026 (doc) | **OUI — c'est le changement critique**
| P0-3 Résolution générale (non-crypto) | Oracle optimiste UMA, bond 750$, challenge 2 h | docs.polymarket.com/concepts/resolution | consultée 10/08/2026 | Non
| P0-4 Rate limits | CLOB général 9 000 req/10 s ; `/book` et `/price` 1 500 req/10 s ; `POST /order` 5 000 req/10 s (burst), 120 000/10 min (sustained) | docs.polymarket.com/api-reference/rate-limits | consultée 10/08/2026 | Oui, relevés le 01/06/2026 (changelog)


Implication directe du changement P0-3 : le pipeline de données de la session affiche un strike « dernier tick ≤ t0 » — l'ancien monde. Le règlement du 07/08/2026 lisse la clôture sur 30 s ; toute stratégie qui vise le dernier tick exact est invalide, toute stratégie à marge large sur le niveau oracle survit. La définition exacte du strike d'ouverture sous le nouveau régime : DONNÉE NON VÉRIFIABLE — hypothèse conservatrice appliquée : je n'exploite que des marges ≥ 20,00$ au-dessus du strike affiché, ce qui rend le résultat insensible à un lissage de 30 s (démonstration chiffrée en D9).

# 2. CADRE D'ANALYSE

Je lis chaque pièce pour en extraire des faits horodatés, sans stratégie préconçue. Les timestamps sont en mm:ss.mmm depuis l'ouverture de la fenêtre (t0). Le marché analysé : « Bitcoin Up or Down — August 7, 8:45AM-8:50AM ET », résultat ▲ UP, strike exact 65 109,38$.

---

# 3. PARTIE A — LES 6 GRAPHES

## A-G1 — CARNET YES/NO COMPLET, VUE 300 s

A1. Axes : temps (mm:ss.mmm) × prix ($, 0–1) ; 519 événements BBO YES/NO, 18 points « carnet croisé » marqués ; période 00:00.000 → 05:00.000.
A2. Question : le prix du marché suit-il ou retarde-t-il l'information disponible ?
A3. Lecture des étiquettes d'extrema et de la trajectoire des mids YES/NO.
A4. Valeurs extraites : t=00:00.255 : YES mid 0,45$ ; t=00:00.255 : NO mid 0,56$ ; t=04:06.893 : YES 1,00$ ; t=04:06.893 : NO 0,00$ ; t=04:40.003 : YES 1,00$ ; ts fallback : 0/280 (0,0%).
A5. Observation brute : le YES part de 0,45$ et met plus de 4 minutes à atteindre 1,00$ alors que le résultat final est UP — montée progressive, pas de saut unique.
A6. Spécifique à la session : l'amplitude 0,45$ → 1,00$ en 247 s est propre à une session directionnelle ; justification : elle exige que l'oracle ne recroise jamais le strike (vérifié en B-F1 : 279/279 ticks au-dessus).

## A-G2 — CARNET, ZOOM OUVERTURE [00:00.000 → 00:15.000]

A1. Mêmes axes, fenêtre 15 s à l'ouverture.
A2. Question : à quelle vitesse le carnet digère-t-il l'information d'ouverture ?
A3. Lecture séquentielle des étiquettes YES ask.
A4. t=00:00.255 : YES 0,45$ ; t=00:02.248 : 0,50$ ; t=00:06.139 : 0,56$ ; t=00:08.252 : 0,59$ ; t=00:12.923 : 0,65$ ; en miroir NO : t=00:00.255 : 0,56$ → t=00:12.923 : 0,35$.
A5. Observation brute : +0,20$ sur le YES en 12,7 s — le repricing est rapide mais incomplet : à 00:15.000 le YES cote encore 0,64$/0,65$ alors que l'information haussière est déjà stable (cf. B-F1, oracle à +33,23$ du strike à t=15 s).
A6. Spécifique : la valeur de départ 0,45$ (légèrement sous 0,50$) est propre à cette session ; le retard de repricing, lui, est un candidat reproductible.

## A-G3 — CARNET, ZOOM RÉSOLUTION [04:04.000 → 04:19.000]

A1. Mêmes axes, fenêtre 15 s en fin de vie du marché.
A2. Question : reste-t-il de la valeur à capter quand l'issue est acquise ?
A3. Lecture des étiquettes YES/NO en fin de fenêtre.
A4. t=04:05.631 : YES 1,00$ ; t=04:06.823 : 0,99$ ; t=04:11.647 : 0,99$ ; t=04:13.040 : 1,00$ ; t=04:14.535 : 1,00$ ; NO oscille 0,00$–0,01$ sur les mêmes timestamps.
A5. Observation brute : le YES oscille entre 0,99$ et 1,00$ pendant 15 s — la marge résiduelle brute est ≤ 0,01$/action.
A6. Spécifique : oui, cette compression à 0,99–1,00 dès 04:05 suppose une session au verdict évident ; chiffré : l'oracle est alors à +98,01$ du strike (65 207,39$ à 04:05.000, B-F1).

## A-G4 — SPOT BINANCE vs ORACLE CHAINLINK + BASIS, VUE 300 s

A1. Axes : temps × prix BTC ($) ; 12 017 ticks Binance, 280 ticks Chainlink, strike 65 110$ affiché (exact 65 109,38$) ; sous-graphe basis = spot − oracle (0–60$).
A2. Question : lequel des deux flux porte l'information de résolution ?
A3. Comparaison des niveaux et de l'écart persistant.
A4. t=00:00.104 : spot 65 160,27$ ; t=00:00.000 : oracle 65 109,38$ ; t=03:59.309 : spot 65 254,97$ ; t=04:59.399 : spot 65 247,25$ ; t=04:57.000 : oracle 65 191,22$.
A5. Observation brute : l'écart spot−oracle est massif et persistant (bande 0–60$ du sous-graphe, jamais négative) : le spot Binance ne donne PAS le niveau de l'oracle, seulement sa direction. Seul l'oracle tranche la résolution.
A6. Spécifique : le niveau moyen du basis (+50,06$, B-F2) est propre à la paire de flux de cette session ; sa persistance de signe est un candidat structurel (agrégation multi-venues de Chainlink vs BTC/USDT Binance seul).

## A-G5 — SPOT vs ORACLE, ZOOM OUVERTURE [00:00.000 → 00:15.000]

A1. Mêmes axes, 15 premières secondes.
A2. Question : l'issue était-elle lisible dès l'ouverture ?
A3. Lecture des premiers ticks des deux flux contre le strike.
A4. t=00:00.000 : oracle 65 109,38$ (= strike) ; t=00:00.104 : spot 65 160,27$ (strike +50,89$) ; t=00:01.000 : oracle 65 114,31$ ; t=00:09.000 : oracle 65 130,55$ ; t=00:13.461 : spot 65 190,00$ ; t=00:15.000 : oracle 65 142,61$.
A5. Observation brute : dès t=00:01.000 l'oracle est au-dessus du strike et n'y reviendra jamais (B-F1) ; le marché cote pourtant YES 0,45$–0,65$ pendant ces 15 s (A-G2).
A6. Spécifique : l'ouverture exactement sur le tick bas de la session (65 109,38$ = minimum absolu des 280 ticks) est un hasard de session, chiffré en B-F1.

## A-G6 — FLUX DIRECTIONNEL NORMALISÉ 30 s, VUE 300 s

A1. Axes : temps × volume 30 s glissant ($, baissier en négatif) ; 1 378 trades.
A2. Question : le flux agressif précède-t-il ou suit-il le prix ?
A3. Lecture des extrema étiquetés contre la chronologie du prix (A-G1).
A4. t=00:00.433 : +22,50$ ; t=00:10.446 : −808,11$ ; t=03:49.577 : +2 806,87$ ; t=04:45.156 : −149,85$ ; ts fallback 0/280 (0,0%).
A5. Observation brute : le plus gros pic haussier (+2 806,87$) arrive à 03:49.577, quand le YES cote déjà ≥ 0,99$ (A-G3 par continuité, B-F3 : premier ask ≥ 0,99$ à 03:00.407) — le flux confirme, il ne prédit pas.
A6. Spécifique : le déséquilibre global +18 658,95$ (B-F4) est propre à une session UP tendancielle.

---

# 4. PARTIE B — LES CSV

## B-F1 — oracle.csv (Chainlink BTC/USD)

B1. Colonnes t_ms, price, ts_src ; 280 lignes ; 00:00.000 → 04:57.000 ; cadence médiane 1 000 ms, trou max 3 000 ms.
B2. Apport : les valeurs exactes du seul flux qui détermine la résolution, invisibles à cette précision sur les graphes.
B3. Calculs : min/max sur `price` ; comptage `price > 65 109,38` pour t > 0 ; écarts consécutifs ; moyenne des 30 lignes t ≥ 04:27.000 (proxy TWAP 30 s de clôture).

B4.

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Strike (tick t=0) | 65 109,38$ | t_ms, price | 1
| Ticks > strike après t0 | 279/279 (100,0%) | t_ms, price | 279
| Minimum post-t0 | 65 114,31$ (t=00:01.000) | price | 279
| Minimum après 00:15.000 | 65 141,42$ (strike +32,04$) | t_ms, price | 266
| Maximum | 65 207,86$ (t=04:01.000) | price | 280
| Dernier tick | 65 191,22$ (t=04:57.000) | t_ms, price | 1
| Moyenne 30 derniers ticks (t ≥ 04:27) | 65 184,73$ (strike +75,35$) | t_ms, price | 30
| Saut max entre 2 ticks / fallback ts | 9,60$ (t=00:12.000) / 0/280 | price / ts_src | 279 / 280


B5. Observations brutes : l'oracle ne repasse JAMAIS sous le strike après t0 ; il ne repasse jamais sous strike +32,04$ après 00:15.000 ; sa vitesse max est 9,60$ par tick de 1 s.
B6. Anomalies : trous de cadence à 00:10–00:11 et 00:12 (trou 3 000 ms), sans fallback de timestamp — bruit d'échantillonnage, pas signal.

## B-F2 — spot.csv (Binance BTC/USDT)

B1. Colonnes t_ms, price ; 12 017 lignes ; 00:00.104 → 04:59.399 ; rafales de ticks à la ms.
B2. Apport : mesure exacte du basis contre l'oracle, non lisible sur A-G4.
B3. Calculs : pour chaque tick spot, basis = price − dernier tick oracle ≤ t ; stats sur 12 017 valeurs.

B4.

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Premier tick | 65 160,27$ (t=00:00.104) | t_ms, price | 1
| Dernier tick | 65 247,25$ (t=04:59.399) | t_ms, price | 1
| Min / Max | 65 160,27$ / 65 254,97$ | price | 12 017
| Basis moyen | +50,06$ | les 2 fichiers | 12 017
| Basis médian | +50,43$ | les 2 fichiers | 12 017
| Basis min / max | +31,86$ / +60,62$ | les 2 fichiers | 12 017
| Ticks spot > strike | 100,00% | price | 12 017
| Écart premier tick vs strike | +50,89$ | price | 1


B5. Observations brutes : le basis ne change jamais de signe ; utiliser le spot Binance comme proxy du NIVEAU oracle induirait une erreur systématique de +50,06$ en moyenne — disqualifiant pour toute stratégie de seuil sur le strike via Binance.
B6. Anomalies : rafales de dizaines de ticks au même t_ms (ex. 43 lignes à t=00:00.457) — granularité du flux, bruit.

## B-F3 — bbo.csv (carnet YES/NO)

B1. Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 519 lignes ; 00:00.255 → 04:40.003 ; événementiel. Aucune colonne de taille : la profondeur est DONNÉE NON DISPONIBLE.
B2. Apport : prix exécutables exacts à chaque instant, et détection des états croisés.
B3. Calculs : spread = yes_ask − yes_bid ; croisement acheteur = yes_ask + no_ask < 1 ; consensus = (mid YES + (1 − mid NO))/2.

B4.

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Spread YES moyen / médian | 0,0144$ / 0,01$ | yes_bid, yes_ask | 517
| Spread max / min | 0,06$ / −0,03$ | yes_bid, yes_ask | 517
| Événements yes_ask + no_ask < 1 | 7 (meilleure somme 0,970 à 00:02.097) | yes_ask, no_ask | 519
| Durée de l'état croisé de 00:02.097 | 151 ms (événement suivant 00:02.248) | t_ms | 2
| Durée de l'état croisé de 00:37.184 | 18 ms | t_ms | 2
| Premier consensus ≥ 0,90 | t=02:06.323 | 4 colonnes | 519
| Premier yes_ask ≥ 0,99$ | t=03:00.407 | yes_ask | 519
| BBO prévalant à 00:19.104 | YES 0,65$/0,66$ (événement de 00:18.482 ; 0 mise à jour avant 00:20.477) | toutes | 2


B5. Observations brutes : à 00:19.104 on peut acheter du YES à 0,66$ alors que l'oracle est ≥ strike +20$ depuis 10 s ; le consensus ne franchit 0,90 qu'à 02:06.323, soit 107 s plus tard.
B6. Anomalies : les 18 points « carnet croisé » du PDF se décomposent en 7 croisements acheteurs (ya+na`<1) et 7 inversions yes_bid >` yes_ask (ex. 0,49/0,46 à 00:02.097), micro-états de 18 à 151 ms — signal théorique, tué par les frais (calcul en C0).

## B-F4 — trades.csv (flux agressif)

B1. Colonnes t_ms, usd, direction ; 1 378 lignes ; 00:00.433 → 04:49.608.
B2. Apport : volumes réels en dollars, indisponibles ailleurs — seule borne disponible sur la profondeur.
B3. Calculs : sommes par direction ; fenêtres temporelles ; extrema.

B4.

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Volume total | 30 272,59$ | usd | 1 378
| Flux haussier | 963 trades, 24 465,77$ | usd, direction | 963
| Flux baissier | 415 trades, 5 806,82$ | usd, direction | 415
| Flux net | +18 658,95$ | usd, direction | 1 378
| Plus gros trade | 2 806,87$ (t=03:49.577, haussier) | t_ms, usd | 1
| Trade médian | 4,70$ | usd | 1 378
| Volume 00:15–00:25 | 66 trades, 771,43$ | t_ms, usd | 66
| Volume 0–30 s | 245 trades, 4 173,45$ (dont 2 274,61$ haussier) | t_ms, usd, direction | 245


B5. Observations brutes : autour de mon point d'entrée (00:15–00:25), 771,43$ s'échangent en 10 s — un ordre de 4,62$ représente 0,6% de ce flux et se situe au niveau du trade médian (4,70$).
B6. Anomalies : trades poussière (0,0014$ à 02:19.281) et timestamps localement non monotones (00:00.762 après 00:00.764) — bruit de collecte.

---

# 5. PARTIE C — STRATÉGIES ET SIMULATION

## C0 — CANDIDATES

Mes cinq observations les plus fortes :

- **OBS-1** (B-F1) : l'oracle est au-dessus du strike sur 279/279 ticks post-t0 et jamais sous strike +32,04$ après 00:15.000.
- **OBS-2** (B-F2) : basis spot−oracle moyen +50,06$, jamais négatif — Binance donne la direction, jamais le niveau.
- **OBS-3** (B-F3/A-G2) : YES achetable 0,66$ à 00:19.104 alors que l'oracle est ≥ strike +20$ depuis 10 s ; consensus ≥ 0,90 seulement à 02:06.323.
- **OBS-4** (B-F3/B6) : 7 croisements acheteurs, meilleure somme 0,970, durées 18–151 ms.
- **OBS-5** (B-F4/A-G6) : flux net +18 658,95$ dont le pic (+2 806,87$) arrive quand le YES cote déjà ≥ 0,99$ — le flux suit.


FILTRE DE FAISABILITÉ STRUCTURELLE (latences du tableau, chemin par chemin) :

- S1 Convergence oracle : lecture Chainlink 68 ms + envoi CLOB 31 ms + confirmation 31 ms = **130 ms** vs signal vivant ≥ 280 s (OBS-1) → possible.
- S2 Arbitrage croisé : lecture book CLOB 31 + envoi 31 + confirmation 31 = **93 ms** vs états de 151 ms (00:02.097) et 18 ms (00:37.184) → possible sur 1 état sur 2, [STRUCTURELLEMENT LIMITE].
- S3 Momentum Binance : 219 + 31 + 31 = **281 ms** vs vagues de flux de 2–15 s → possible.
- S4 Market making : boucle requote 31 + 31 = **62 ms** → possible.


**CRITÈRE DE SÉLECTION : P&L net réel maximal sur la session (après latences, slippage borné, frais P0-1), sous contrainte de perte maximale ≤ cash disponible et de conformité P0 totale.** Annoncé avant toute comparaison.

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----|-----
| S1 Convergence oracle-strike (achat YES ≤ 0,70$, marge oracle ≥ +20$/10 s, tenue à résolution) | OBS-1, OBS-3 | Oui (taker, ≥ 5 actions, tick 0,01) | 1 | +2,38$ | **+2,27$** | **RETENUE**
| S2 Arbitrage carnet croisé (achat YES+NO si somme < 1) | OBS-4 | Oui | 1–2 | +0,03$/paire | −0,00488$/paire (frais 0,03488$ > écart 0,030$ à 00:02.097 ; −0,01589$/paire à 00:37.184) | Éliminée : frais P0-1 > écart
| S3 Momentum flux Binance (suivre le flux 30 s) | OBS-5, OBS-2 | Oui | 4–8 | ≈ +0,08$/action/vague | négatif : coût aller-retour 0,0336$/action (frais à p=0,60) + spread médian 0,01$ ≥ mouvement médian capté après 281 ms | Éliminée : coûts > signal, et OBS-5 montre un flux retardataire
| S4 Market making 0 frais + rebate 20% | P0-1, OBS-5 | Oui | >20 | spread 0,01$/fill | non simulable (profondeur DONNÉE NON DISPONIBLE) ; sélection adverse démontrée : trend 0,45$→1,00$, tout inventaire NO acquis vaut 0,00$ à la fin | Éliminée : risque non borné vs 5,00$


Décision : S1 est la seule candidate à P&L net réel positif et mesurable. Les quatre familles reposent sur des mécanismes distincts (information exogène, arbitrage interne, momentum, fourniture de liquidité).

## C1 — STRATÉGIE RETENUE : CONVERGENCE ORACLE-STRIKE

```plaintext
ENTRÉES : strike S du marché ; flux oracle Chainlink direct (68 ms) ; BBO CLOB (31 ms)
PARAMÈTRES :
  MARGE = 20,00 $        # OBS-1 : min post-00:15 = strike+32,04$ [FRAGILE - 1 SOURCE]
  CONFIRM = 10 s          # OBS-1 : couvre le trou de cadence max (3 s, B-F1/B6)
  PRIX_MAX = 0,70 $       # OBS-3 : YES 0,66$ disponible sous ce seuil [FRAGILE - 1 SOURCE]
  T_LIMITE = 240 s        # P0-3 : laisse > 30 s de TWAP + boucle 130 ms hors zone de clôture

RÈGLE UNIQUE (une exécution max par fenêtre de 5 min) :
  SI dernier_tick_oracle ≥ S + MARGE pendant CONFIRM consécutives   # B-F1
  ET yes_ask ≤ PRIX_MAX                                             # B-F3
  ET t < T_LIMITE
  ALORS acheter YES taker : C = plancher(cash / (yes_ask + 0,07×yes_ask×(1−yes_ask)))  # P0-1
        avec C ≥ 5 (P0-2), ordre FAK (P0-2)
  TENIR jusqu'à résolution TWAP 30 s (P0-3). Aucune sortie anticipée
  (évite un second frais taker P0-1 et le risque de spread A-G3).
  STOP INFORMATIONNEL : si dernier_tick_oracle < S pendant 10 s, vendre au bid
  [HYPOTHÈSE n°1 : jamais déclenché dans les données — OBS-1, 0 tick sous le strike].
```

## C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE** : cash initial 5,00$, aucune position, aucun ordre.

**RACE CONDITIONS** :

- (a) Signal d'achat pendant ordre en cours → RÈGLE : ignoré, 1 ordre vivant max.
- (b) Vente sur position non confirmée → RÈGLE : aucune vente tant que le fill n'est pas confirmé (confirmation à 00:19.135, aucune vente dans la stratégie).
- (c) Deux signaux simultanés → RÈGLE : priorité au stop informationnel, sinon premier timestamp oracle.
- (d) Fill partiel → RÈGLE : FAK, le reliquat est abandonné, pas de re-soumission sous 2 s.


Latence appliquée à chaque entrée (r1, r2) : le tick oracle de 00:19.000 est lu à 00:19.068 (68 ms d'âge, r2) ; ordre au CLOB à 00:19.099 (+31 ms) ; confirmation à 00:19.130 (+31 ms). Prix d'exécution = BBO existant à 00:19.099 : yes_ask 0,66$, inchangé de 00:18.482 à 00:20.477 (0 événement BBO entre les deux, B-F3) → slippage temporel nul, prouvé. Profondeur (r3) : sans tailles dans le BBO, je borne par B-F4 : 771,43$ échangés sur 00:15–00:25 pour un ordre de 4,62$ (0,6% du flux, égal au trade médian 4,70$) ; RÈGLE annoncée : si le niveau 0,66$ a disparu, re-pricing à 0,67$ (coût 4,80$, toujours ≤ cash) ; scénario retenu : fill à 0,66$. Frais (r4) : P0-1. Cash (r5) : 4,73$ ≤ 5,00$.

**JOURNAL DE TRADES** :

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----
| 1 | 00:19.000 | 00:19.099 (confirmé 00:19.130) | 0,66$ | 7 | 05:00.000 (résolution TWAP) | 1,00$ | 0,00$ (borne −0,07$ si re-pricing 0,67$) | 0,11$ (7×0,07×0,66×0,34 = 0,10996) | **+2,27$** | **7,27$**


Vérification comptable : 5,00 − (7×0,66) − 0,11 = 0,27$ de cash pendant la position ; à résolution UP (TWAP 30 s proxy 65 184,73$ > 65 109,38$, B-F1) : 0,27 + 7×1,00 = **7,27$**.

**RÉSULTATS** :

- Trades : 1 gagnant (+2,27$), 0 perdant. Frais totaux : 0,11$.
- BÉNÉFICE NET : **+2,27$** ; capital final **7,27$** vs 5,00$ = **+45,40%**.
- Pire perte réalisée : 0,00$. Drawdown max : −0,25$ (−5,00%) — yes_bid minimum post-entrée 0,64$ à 00:20.481 (B-F3), valorisation 0,27 + 7×0,64 = 4,75$.
- P&L THÉORIQUE (sans r1–r4) : 5,00/0,66 = 7,57 actions, P&L 7,57×0,34 = **+2,57$**. Coût du réalisme : 0,30$ (0,11$ de frais + 0,19$ de capacité perdue).


---

# 6. PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----
| D1 Latence de boucle | 130 ms sans effet prix | 0 événement BBO entre 00:18.482 et 00:20.477 (B-F3) : prix identique à ±1 995 ms autour de l'exécution | 0,00$ | [SANS IMPACT] (prouvé)
| D2 Fraîcheur du signal | tick oracle âgé de 68 ms utilisable | saut oracle max 9,60$/tick de 1 s (B-F1) → erreur max pendant 68 ms bornée à 0,65$ (9,60×0,068), négligeable devant la marge de 35,37$ à l'entrée | 0,00$ | [SANS IMPACT] (borné)
| D3 Slippage/profondeur | fill 4,62$ au ask | tailles absentes du BBO (DONNÉE NON DISPONIBLE) ; borne B-F4 : ordre = trade médian (4,70$), 0,6% du flux 00:15–00:25 ; re-pricing à 0,67$ coûterait 0,07$ | 0,00$ à −0,07$ | [DÉGRADE]
| D4 Frais complets | 0,11$ taker, 0$ maker, 0$ dépôt/retrait | P0-1 vérifié ; gas de rédemption non documenté = DONNÉE NON VÉRIFIABLE, couvert par la réserve cash de 0,27$ | −0,11$ (intégré) | [DÉGRADE]
| D5 Niveau disparu / fill partiel | FAK avec re-pricing +1 tick | règle (d) ; pire cas 0,67$ → P&L +2,20$ au lieu de +2,27$ | −0,07$ max | [DÉGRADE]
| D6 Défaillances techniques | timeout RPC possible | Polygon RPC 108 ms et rate limits P0-4 (1 500 req/10 s sur /book) très au-dessus du besoin (1 req/s) ; 1 seul ordre, statut vérifiable via GET /order | retard d'entrée de quelques ticks, marge OBS-1 intacte | [DÉGRADE]
| D7 Passage à l'échelle | taille 4,62$ | flux total au ask sur 00:15–00:25 = 771,43$ (B-F4) ; au-delà de ~35$ par ordre (plus gros trade voisin, 00:19.054) le fill au meilleur niveau n'est plus démontrable | s'éteint entre 35$ et 771$ de taille | [DÉGRADE]
| D8 Dépendance à la session | seuils MARGE/PRIX_MAX généralisables | calibrés sur 1 session ; sur session calme (oracle < strike+20$), la stratégie ne trade pas → P&L 0,00$, perte 0,00$ | 0,00$ sur session calme ; espérance hors échantillon non démontrée | [DÉGRADE]
| D9 Conformité nouveau fonctionnement | résolution TWAP 30 s sans effet sur une position tenue à terme | P0-3 : TWAP 30 s effectif 07/08/2026 ; proxy TWAP de clôture 65 184,73$ = strike +75,35$ (B-F1) : même verdict UP que le dernier tick (65 191,22$) ; il faudrait une chute > 75,35$ dans les 30 dernières secondes (vs saut max observé 9,60$/s) pour inverser | 0,00$ | [SANS IMPACT] (prouvé)


**VERDICT GLOBAL : [FRAGILE].** Règles mécaniques respectées : zéro [DÉTRUIT], P&L net réel positif (+2,27$), conformité P0 totale — [NON FIABLE] est exclu. Je refuse [FIABLE] : D3 repose sur une borne, pas une preuve (profondeur DONNÉE NON DISPONIBLE), et D8 laisse la magnitude de l'edge validée sur une seule session. Étiquette : **[PATTERN CANDIDAT]** — le mécanisme (le carnet met 107 s à refléter une certitude oracle lisible en 130 ms) est testable sur toute fenêtre de 5 min ; ses seuils, eux, restent [FRAGILE - 1 SOURCE].

---

# 7. AUTO-CONTRÔLE FINAL

- Contrôle 1 → Phase 0 complète : P0-1 à P0-4 sourcés (4 URL docs.polymarket.com, consultées le 10/08/2026) ; changement récent identifié (TWAP 30 s, effectif 07/08/2026) ; strike nouveau régime et gas de rédemption marqués DONNÉE NON VÉRIFIABLE avec hypothèses conservatrices.
- Contrôle 2 → A-G1 à A-G6 : 5 à 6 valeurs horodatées chacun ; B-F1 à B-F4 : 8 valeurs chacun dans les tableaux F4. OK.
- Contrôle 3 → chaque règle de C1 cite OBS-1/OBS-3 (A5/B5) ou une règle P0 ; 2 marquages [FRAGILE - 1 SOURCE], 1 [HYPOTHÈSE n°1]. OK.
- Contrôle 4 → boucle reconstituée chemin par chemin (68+31+31 = 130 ms) et appliquée à l'unique entrée du journal (00:19.000 → 00:19.099). OK.
- Contrôle 5 → comptabilité : 5,00 − 4,62 − 0,11 = 0,27 ; 0,27 + 7,00 = 7,27$ = capital final annoncé, au centime près. OK.
- Contrôle 6 → D1–D9 traités ; les 3 [SANS IMPACT] portent leur preuve chiffrée (0 événement BBO, borne 0,65$, marge TWAP +75,35$) ; verdict global conforme aux règles mécaniques. OK.
- Contrôle 7 → chiffres d'ouverture (7,27$ ; +45,40% ; 1 trade ; 0,11$ ; TWAP 30 s) identiques dans le corps ; aucun chiffre dupliqué avec valeur divergente. OK.


# 8. CLÔTURE

Vous avez le journal, les sources et les bornes : la stratégie gagne sur cette session en conditions réelles, mais je la déclare fragile tant que la profondeur du carnet et une deuxième session ne sont pas au dossier. Le changement de résolution est intégré, pas subi. La prochaine étape utile est la collecte, pas la mise.

---

## Source : 292CONVERGENCE_ORACLE-STRIKE.md

# 0. SYNTHÈSE EXÉCUTIVE

Vous avez le PDF, les CSV et le tableau des latences sous les yeux ; voici le verdict d'entrée.

- Stratégie retenue : **« CONVERGENCE ORACLE-STRIKE »** — entrée côté oracle Chainlink après 3 ticks consécutifs à ≥10,00$ du strike, tenue jusqu'à résolution.
- Capital final : **5,65786$** vs 5,00$, soit **+0,65786$ net (+13,16%)**, APRÈS latence de boucle (131 ms), slippage (0,00$ constaté) et frais taker crypto (0,04214$).
- Trades : **1 trade, 1 gagnant / 0 perdant**.
- Changement Polymarket le plus impactant : la résolution des marchés crypto 5 min est passée d'un snapshot unique à un **TWAP Chainlink 30 secondes** (infrastructure TWAP 30 s/60 s Chainlink Data Streams, relais RTDS lancé le 4 août 2026 — docs.polymarket.com/market-data/chainlink-twap).
- VERDICT DE FIABILITÉ : **[FRAGILE]**.
- Étiquette : **[SESSION-SPÉCIFIQUE]**.


Chaque chiffre ci-dessus réapparaît à l'identique dans le corps.

---

# 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET (VÉRIFIÉ)

Je n'ai retenu aucune connaissance interne : tout ce qui suit sort de la documentation officielle consultée aujourd'hui.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × 0,07 × p × (1−p) ; maker 0 ; rebate maker 20% des frais taker ; arrondi à 5 décimales ; aucun frais de dépôt/retrait/settlement | docs.polymarket.com/trading/fees | consultée 09/08/2026 | Non (frais 5 min actifs depuis leur lancement, extension à tous les marchés crypto le 06/03/2026 — changelog)
| P0-2 CLOB : types d'ordres | GTC, GTD, FOK, FAK (partiel possible), post-only ; batch max 15 ordres ; CLOB V2 depuis le 28/04/2026 (pUSD, frais fixés au matching, plus de feeRateBps dans l'ordre) | docs.polymarket.com/trading/place-orders ; /changelog/predictions | consultée 09/08/2026 | Oui (V2 le 28/04/2026 ; pipeline async 24/07/2026 : plus de transactionHashes inline, tradeIDs à la place)
| P0-2 Tick size | 0,01$ observé sur ce marché (tous les prix de bbo.csv sont sur grille 0,01) ; endpoint officiel /tick-size disponible | docs.polymarket.com/api-reference/market-data/get-tick-size + B-F1 | consultée 09/08/2026 | Non pour crypto (le tick 0,0025 ne concerne que certains marchés World Cup)
| P0-2 Taille minimale d'ordre | DONNÉE NON VÉRIFIABLE dans une source datée — hypothèse conservatrice appliquée : ordres en parts entières, minimum 5 parts | — | — | —
| P0-3 Résolution crypto 5 min | **TWAP Chainlink 30 secondes** (fenêtre de lookback, calculée et signée par Chainlink, feeds « BTC/USD — TWAP: 30s ») ; relais Polymarket RTDS lancé le 04/08/2026 ; remplace le snapshot à timestamp unique | docs.polymarket.com/market-data/chainlink-twap (+ recherche web du 09/08/2026 : bascule des marchés 5/15 min sur TWAP pour contrer la manipulation court-terme) | 04/08/2026 | **OUI — c'est LE changement (~72 h avant la session)**
| P0-4 Rate limits | CLOB général 9 000 req/10 s ; /book 1 500 req/10 s ; POST /order 5 000 req/10 s burst, 120 000/10 min soutenu ; WSS Market Channel temps réel disponible | docs.polymarket.com/api-reference/rate-limits | consultée 09/08/2026 | Oui (relèvement des limites, changelog)


Conséquence dure de P0-3 pour une session de 5 minutes : le résultat n'est plus le prix à t=300 s, c'est la **moyenne pondérée temps des 30 dernières secondes de l'oracle**. Un spike de dernière seconde ne peut plus retourner un marché ; la fenêtre [04:30 → 05:00] devient la zone de vérité. Aucune stratégie ci-dessous ne viole une règle P0.

---

# 2. CADRE D'ANALYSE

Je lis d'abord les 6 graphes du PDF (identité, valeurs, observations), puis les 4 CSV avec calculs exacts, et seulement ensuite je formule des stratégies. Le référentiel de résolution est l'oracle Chainlink (P0-3), pas le spot Binance. Tous les timestamps sont en mm:ss.mmm depuis l'ouverture de la fenêtre (session « Bitcoin Up or Down — August 7, 9:25–9:30 AM ET », résultat ▼ DOWN).

---

# 3. PARTIE A — LES 6 GRAPHES

## A-G1 — CARNET YES/NO COMPLET (708 ÉVÉNEMENTS BBO)

- A1. IDENTITÉ : bid/ask/mid YES et NO ($, axe 0–1) vs temps (0→300 s) + 3 zooms 15 s ([00:00–00:15], [02:29–02:44], [02:53–03:08]) ; 708 événements, 56 incohérents marqués.
- A2. QUESTION : à quelle vitesse et dans quel sens le carnet a-t-il pricé l'issue DOWN ?
- A3. MÉTHODE : lecture des étiquettes extrêmes et des zooms, croisée avec B-F1.
- A4. VALEURS : t=00:00.149 : YES bid 0,46$ ; t=00:00.360 : YES bid 0,40$ (rafale de requotes au même ms) ; t=00:03.449 : YES 0,15$ ; t=03:46.375 : YES 0,01$ / NO 0,99$ ; t=04:39.424 : YES 0,01$ ; NO à t=00:00.149 : 0,54$.
- A5. OBSERVATIONS BRUTES : (i) effondrement YES 0,46→0,15 en 3,300 s à l'ouverture ; (ii) rebond YES jusqu'à ~0,46 vers 01:34 (zoom implicite, confirmé B-F1 : mid 0,465 à 01:34.479) ; (iii) rechute terminale par paliers de 0,01$.
- A6. SPÉCIFIQUE À LA SESSION : l'amplitude 0,46→0,01 en 5 min est propre à ce marché résolu DOWN sans ambiguïté ; le rebond médian à 0,46 (01:34) montre que l'issue n'était PAS acquise à mi-session.


## A-G2 — SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT

- A1. IDENTITÉ : prix BTC ($, 65 080–65 200) vs temps ; Binance 17 184 ticks, Chainlink 281 ticks ; ligne strike affichée « 65,133$ » ; sous-graphe basis = spot − oracle (0–60$) ; 3 zooms.
- A2. QUESTION : lequel des deux flux fait foi et avec quel écart ?
- A3. MÉTHODE : lecture des annotations, croisée B-F3/B-F4.
- A4. VALEURS : spot t=00:00.035 : 65 178,70$ ; spot max t=01:35.233 : 65 196,00$ ; oracle t=03:37.863 : 65 132,83$ (passage sous strike) ; oracle min t=03:46.421 : 65 114,67$ (annotation zoom) ; oracle t=04:59.804 : 65 163,16$ (dernier point spot) ; oracle t=00:00.000 : 65 134,46$.
- A5. OBSERVATIONS BRUTES : le basis spot−oracle est visuellement positif sur TOUTE la fenêtre (bande 30–60$) ; les deux courbes sont co-mobiles mais jamais confondues.
- A6. SPÉCIFIQUE À LA SESSION : le strike affiché 65 133$ (arrondi du graphe) et l'oracle t=0 à 65 134,46$ (B-F3) diffèrent de 1,46$ — écart documenté en B6-F3 ; je retiens comme strike opérationnel 65 134,46$, la valeur exacte lisible dans les données.


## A-G3 — FLUX DIRECTIONNEL NORMALISÉ (2 122 TRADES)

- A1. IDENTITÉ : volume 30 s glissant haussier (+) / baissier (−), $, −6 000 à +2 000 ; 2 122 trades ; 3 zooms.
- A2. QUESTION : le flux agressif a-t-il précédé ou suivi le prix ?
- A3. MÉTHODE : extrema annotés + croisement B-F2.
- A4. VALEURS : t=00:00.439 : +20,40$ (premier trade) ; t=02:37.010 : −1 282,32$ ; t=03:49.820 : +1 195,55$ ; t=04:28.639 : −1 356,99$ ; t=04:53.852 : +191,93$.
- A5. OBSERVATIONS BRUTES : trois vagues baissières massives (ouverture, 02:36–02:38, 04:28) ; le plus gros trade haussier (+1 195,55$ à 03:49.820) intervient APRÈS que NO cote déjà ≥0,95$ — achat de quasi-certitude, pas un pari contrarien.
- A6. SPÉCIFIQUE À LA SESSION : dominance baissière quasi permanente, cohérente avec l'issue DOWN ; non généralisable.


## A-G4 — IMBALANCE DIRECTIONNELLE

- A1. IDENTITÉ : imbalance = haussier/(haussier+baissier) ∈ [0,1], ligne d'équilibre 0,5 ; 2 122 évts ; 3 zooms.
- A2. QUESTION : l'imbalance donne-t-elle un signal avancé exploitable ?
- A3. MÉTHODE : extrema annotés.
- A4. VALEURS : t=00:00.439 : 1,00 ; t=00:02.278 : 0,05 ; t=02:39.881 : 0,09 ; t=03:06.298 : 0,13 ; t=04:53.852 : 0,36.
- A5. OBSERVATIONS BRUTES : bascule 1,00→0,05 en 1,839 s à l'ouverture — le flux agressif a réagi en moins de 2 s ; l'imbalance reste <0,5 quasi toute la session.
- A6. SPÉCIFIQUE À LA SESSION : la vitesse de bascule (<2 s) borne la durée de vie de tout signal de flux à l'ouverture.


## A-G5 — SPREADS YES ET NO ABSOLUS

- A1. IDENTITÉ : ask−bid ($, −0,10 à +0,10) pour YES et NO ; 708 évts ; 3 zooms.
- A2. QUESTION : quel coût de traversée du spread, et quand explose-t-il ?
- A3. MÉTHODE : extrema + séries des zooms.
- A4. VALEURS : t=00:00.149 : 0,01$ ; t=00:00.360 : −0,12$ (carnet croisé) ; t=01:13.356 : 0,10$ (max) ; t=02:41.935 : 0,08$ ; t=04:39.424 : 0,00$.
- A5. OBSERVATIONS BRUTES : régime dominant 0,01–0,02$ ; élargissements brefs (≤2 s) lors des chocs (02:41–02:42) ; spreads YES et NO identiques point à point (carnets miroirs).
- A6. SPÉCIFIQUE À LA SESSION : le spread négatif −0,12$ à 00:00.360 est un artefact de rafale de requotes au même milliseconde, pas une opportunité (voir C0, candidate éliminée).


## A-G6 — ÉCART INTER-CARNETS YES VS NO

- A1. IDENTITÉ : yes_mid − (1 − no_mid) ($, −0,004 à +0,010), ligne 0 ; 708 évts ; 56 suspects « carnet croisé » ; 3 zooms.
- A2. QUESTION : existe-t-il une incohérence YES/NO arbitrable ?
- A3. MÉTHODE : le graphe ne porte aucune étiquette numérique point par point — valeurs point-à-point NON DISPONIBLES sur le graphe ; je lis les bornes d'axe et les compteurs, et je chiffre en B-F1.
- A4. VALEURS (lisibles) : borne basse −0,004$ ; borne haute +0,010$ ; référence 0 ; 708 évts ; 56 suspects. Chiffrage exact en B4-F1 : écart moyen +0,000022$, min −0,00500$, max +0,01000$.
- A5. OBSERVATIONS BRUTES : l'écart vit collé à 0 ; les excursions sont ponctuelles et de l'ordre du demi-tick.
- A6. SPÉCIFIQUE À LA SESSION : l'incohérence maximale (0,01$) est égale à UN tick (P0-2) — par construction inexploitable après frais.


---

# 4. PARTIE B — LES CSV




## B-F1 — bbo.csv

- B1. IDENTITÉ : colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 708 lignes ; t=149,0 → 279 424,0 ms ; événementiel (2,53 évts/s en moyenne).
- B2. APPORT : le prix exécutable exact à tout instant — c'est la base de la simulation.
- B3. CALCULS : spread = yes_ask − yes_bid ; mid ; écart inter-carnets = yes_mid − (1 − no_mid) ; croisement strict = yes_ask + no_ask < 1 ; sur les 676 lignes complètes.


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Événements BBO | 708 | toutes | 708
| Lignes complètes (4 côtés) | 676 | toutes | 708
| Spread YES moyen / médian | 0,0183$ / 0,0200$ | yes_bid, yes_ask | 676
| Part des spreads ≤0,02$ | 78,8% (533/676) | yes_bid, yes_ask | 676
| yes_mid min / max | 0,010$ (03:46.375) / 0,465$ (01:34.479) | yes_bid, yes_ask, t_ms | 676
| Écart inter-carnets moyen / min / max | +0,000022$ / −0,00500$ / +0,01000$ | 4 côtés | 676
| Croisements stricts (yes_ask+no_ask<1) | 14 (premier à 00:00.360) | yes_ask, no_ask | 676
| NO ask à 00:03.449 / no_bid min après 00:04.100 | 0,86$ / 0,53$ (01:32.973) | no_ask, no_bid, t_ms | 676


- B5. OBSERVATIONS BRUTES : OBS-2 (le carnet met 3,300 s à passer NO ask de 0,54$ à 0,85$) ; requote complet en 211 ms à l'ouverture (00:00.149 → 00:00.360) ; OBS-5 (14 croisements stricts, tous dans des rafales au même ms).
- B6. ANOMALIES : les 56 « suspects » du PDF vs mes 14 croisements stricts — le PDF marque aussi les quasi-croisements ; rafales de 10+ lignes au timestamp identique (00:00.360) = purge de carnet, bruit et non signal.


## B-F2 — trades.csv

- B1. IDENTITÉ : t_ms, usd, direction (+1 haussier / −1 baissier) ; 2 122 lignes ; t=439,0 → 293 852,0 ms.
- B2. APPORT : le flux agressif signé, invisible dans le BBO.
- B3. CALCULS : sommes par direction, par minute, extrema, imbalance = baissier/total.


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Volume total | 38 797,31$ | usd | 2 122
| Volume baissier / haussier | 28 706,48$ (1 142) / 10 090,83$ (980) | usd, direction | 2 122
| Imbalance baissière | 74,0% | usd, direction | 2 122
| Trade max | 1 356,99$ baissier à 04:28.639 | usd, direction, t_ms | 2 122
| Trade médian / moyen | 4,54$ / 18,28$ | usd | 2 122
| Flux net 0–3 s | −1 506,91$ (76 trades) | usd, direction, t_ms | 76
| Volume minute 1 / minute 4 | 7 862,96$ (net −3 723,32$) / 12 967,68$ (net −6 901,74$) | usd, direction, t_ms | 518 / 580
| Volume 0–15 s | 3 946,85$ (238 trades) | usd, t_ms | 238


- B5. OBSERVATIONS BRUTES : OBS-3 (74,0% baissier) ; les 5 minutes sont TOUTES nettes vendeuses (−3 723,32$ / −942,51$ / −4 851,08$ / −6 901,74$ / −2 197,01$) ; doublet institutionnel −1 282,32$ + −1 075,62$ à 02:37.010/02:37.013.
- B6. ANOMALIES : timestamps localement non monotones (ex. lignes 810,0 puis 812,0 après 815,0) = arrivée réseau désordonnée, ~ms, sans impact sur les agrégats.


## B-F3 — oracle.csv

- B1. IDENTITÉ : t_ms, price, ts_src ; 281 lignes ; t=0 → 298 000 ms ; cadence médiane 1 000 ms (max 3 000 ms) ; ts_src=payload sur 281/281 (0 fallback).
- B2. APPORT : le référentiel de RÉSOLUTION (P0-3) — aucune autre pièce ne fait foi.
- B3. CALCULS : écart au strike opérationnel 65 134,46$ (valeur t=0) ; runs au-dessus/en-dessous ; moyenne des 26 ticks de la fenêtre TWAP finale [04:30 → 04:58].


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Strike opérationnel (tick t=0) | 65 134,46$ | price, t_ms | 1
| Oracle min / max | 65 067,83$ (03:47) / 65 149,41$ (01:36) | price, t_ms | 281
| Ticks sous le strike | 198/281 (70,5%) | price | 281
| Plus fort saut de tick | −27,69$ à 00:02.000 | price, t_ms | 280
| Run continu AU-DESSUS du strike | 66 ticks, 01:28.000 → 02:37.000 | price, t_ms | 281
| Dernier passage au-dessus | 02:54.000 (fin du run 02:43–02:54) | price, t_ms | 281
| Moyenne fenêtre TWAP [04:30→04:58] | 65 110,92$ (min 65 109,63$, max 65 113,20$, 26 ticks) | price, t_ms | 26
| Écart TWAP final − strike | −23,54$ | price | 27


- B5. OBSERVATIONS BRUTES : OBS-1 (oracle à 65 105,85$ dès 00:02.000, −28,61$ sous strike, plus fort saut de la session) ; l'oracle repasse AU-DESSUS du strike pendant 66 ticks consécutifs (01:28→02:37, max +14,95$) — l'issue a réellement failli basculer ; la fenêtre TWAP finale est stable (amplitude 3,57$).
- B6. ANOMALIES : strike affiché PDF 65 133$ vs tick t=0 à 65 134,46$ (écart 1,46$ — arrondi d'affichage probable, le « dernier tick ≤ t0 » du titre n'étant pas inclus dans le CSV) ; 19 secondes manquantes (cadence max 3 000 ms).


## B-F4 — spot.csv

- B1. IDENTITÉ : t_ms, price ; 17 184 lignes ; t=35,0 → 299 804,0 ms ; flux direct Binance BTC/USDT.
- B2. APPORT : la seule série haute fréquence — teste si le spot est un prédicteur exploitable de l'oracle.
- B3. CALCULS : basis = spot − dernier tick oracle ≤ t (interpolation en escalier) ; moyennes rééchantillonnées 1 s ; plus forts mouvements 1 s.


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Ticks | 17 184 | toutes | 17 184
| Spot premier / dernier | 65 178,70$ / 65 163,16$ | price, t_ms | 2
| Spot min / max | 65 114,67$ (03:46.421) / 65 196,00$ (01:35.233) | price, t_ms | 17 184
| Basis moyen / médian | +45,84$ / +47,46$ | price (+ B-F3) | 17 184
| Basis min / max | +8,62$ / +59,94$ | price (+ B-F3) | 17 184
| Part du basis > 0 | 100,0% | price (+ B-F3) | 17 184
| Plus fort mouvement 1 s | −23,85$ à 00:01.000 | price, t_ms | 17 184
| Moyenne spot fenêtre [04:30→04:59] | 65 159,97$ | price, t_ms | 30 pts 1 s


- B5. OBSERVATIONS BRUTES : OBS-4 — le basis est positif sur 100,0% des ticks (moyenne +45,84$) : BTC/USDT Binance et BTC/USD Chainlink sont deux actifs différents ; le NIVEAU spot ne prédit pas la résolution, seuls ses RETOURS co-varient avec l'oracle. Preuve : moyenne spot de la fenêtre finale 65 159,97$ > strike (signal UP), alors que l'oracle y est à 65 110,92$ < strike (signal DOWN, réel).
- B6. ANOMALIES : rafales de 60+ ticks au même t_ms (ex. t=1 090,0) = niveaux de carnet dépilés en un ms ; sans impact après rééchantillonnage.


---

# 5. PARTIE C — STRATÉGIES ET SIMULATION

## C0 — CANDIDATES

Mes cinq observations les plus fortes, chacune déjà sourcée :

- OBS-1 : oracle −28,61$ sous strike dès 00:02.000, plus fort saut de tick (B5-F3).
- OBS-2 : le carnet met 3,300 s à repricer NO de 0,54$ à 0,85$ (B5-F1, A5-G1).
- OBS-3 : flux 74,0% baissier, 5 minutes sur 5 nettes vendeuses (B5-F2).
- OBS-4 : basis spot−oracle +45,84$ moyen, positif à 100,0% — le spot n'est pas le référentiel (B5-F4).
- OBS-5 : 14 croisements stricts inter-carnets, vie ≤ quelques ms, amplitude max 1 tick (B5-F1, A5-G6).


**Candidates (4 familles distinctes de mécanisme) :**

1. **« Convergence Oracle-Strike »** (OBS-1, OBS-2) — le référentiel de résolution (oracle) diverge du strike PENDANT que le carnet reprice lentement ; entrer du côté de l'oracle après confirmation (3 ticks consécutifs ≥10,00$ d'écart), tenir jusqu'à résolution. Conforme P0 (ordre FAK taker, frais 0,07 intégrés, résolution TWAP compatible avec la tenue). **Boucle** : Chainlink 68 ms + décision 1 ms + envoi CLOB 31 ms = 100 ms jusqu'au matching, +31 ms de confirmation = **131 ms** ; vie du signal ≥ cadence oracle 1 000 ms et le régime persiste des dizaines de s (OBS-2) → **FAISABLE**.
2. **« TWAP Lock-in »** (OBS-1, P0-3) — n'entrer que dans la fenêtre TWAP finale (t ≥ 04:30) si l'écart oracle-strike accumulé dépasse 20,00$ : l'issue est alors mathématiquement quasi verrouillée par le TWAP 30 s, on encaisse le résidu de prix. Conforme P0. Boucle 131 ms vs fenêtre de 30 000 ms → **FAISABLE**.
3. **« Spot-Lead Sniping »** (OBS-4 inversée) — utiliser le flux Binance pour devancer le repricing du CLOB. Boucle : signal Binance déjà âgé de 219 ms + décision 1 ms + envoi 31 ms = 251 ms au matching. Or le carnet requote en 211 ms à l'ouverture (00:00.149 → 00:00.360, B5-F1) : la boucle (251 ms) est PLUS LENTE que la vie du signal (≤211 ms) → **[STRUCTURELLEMENT IMPOSSIBLE]** — éliminée avant tout P&L.
4. **« Arbitrage inter-carnets »** (OBS-5) — acheter YES ask + NO ask quand la somme <1. 14 occurrences, toutes dans des rafales au même milliseconde, amplitude max 0,01$ = 1 tick ; deux jambes à 31 ms chacune contre un signal de vie ≤ quelques ms, et frais taker sur les deux jambes (≈0,7% par jambe à p=0,5) > gain max 1,0% → **[STRUCTURELLEMENT IMPOSSIBLE]** — éliminée.


**CRITÈRE DE SÉLECTION : P&L net réel de la session (après latences, slippage, frais P0-1) divisé par (1 + drawdown latent maximal en $), simulé aux règles r1–r5. Annoncé ici, avant comparaison.**

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----
| 1. Convergence Oracle-Strike | OBS-1, OBS-2 | Oui | 1 | +0,70000$ | **+0,65786$** (DD latent 1,69214$ → ratio 0,24437) | **RETENUE**
| 2. TWAP Lock-in | OBS-1, P0-3 | Oui | 1 | +0,05000$ | +0,04653$ (DD latent 0,10347$ → ratio 0,04217) | Écartée (ratio inférieur)
| 3. Spot-Lead Sniping | OBS-4 | Oui | — | — | — | [STRUCTURELLEMENT IMPOSSIBLE]
| 4. Arbitrage inter-carnets | OBS-5 | Oui | — | — | — | [STRUCTURELLEMENT IMPOSSIBLE]


Décision : la candidate 1 domine la candidate 2 sur le critère annoncé (0,24437 vs 0,04217). « Ne pas trader » est battu par deux candidates positives.

## C1 — STRATÉGIE RETENUE : « CONVERGENCE ORACLE-STRIKE »

```plaintext
ENTRÉES (référence des règles) :
  strike     = dernier tick oracle ≤ t0            # 65 134,46$ (B4-F3)
  SEUIL      = 10,00$                               # calibré sur saut max −27,69$ (B4-F3) [HYPOTHÈSE n°1 : 10$ sépare signal/bruit — FRAGILE - 1 SOURCE]
  N_CONF     = 3 ticks oracle consécutifs           # anti-bruit, cadence 1 000 ms (B4-F3)
  PRIX_MAX   = 0,90$                                # ne pas payer la certitude (fee curve P0-1)
  TAILLE     = min 5 parts entières                 # hypothèse conservatrice P0-2

BOUCLE (toutes latences du tableau, rien d'autre) :
  à chaque tick oracle (âge 68 ms à la lecture) :
    ecart = price - strike
    si |ecart| >= SEUIL sur N_CONF ticks consécutifs
       et position == vide et cash >= coût(5 parts) :
         side = NO si ecart < 0 sinon YES            # OBS-1
         si ask(side) <= PRIX_MAX :                  # lu sur WSS Market Channel (P0-4)
             envoyer FAK 5 parts @ ask               # +31 ms CLOB
  tenir jusqu'à résolution (TWAP 30 s, P0-3)         # pas de stop [HYPOTHÈSE n°2 :
     # un stop coûte 2 traversées de spread + 2 frais taker (~0,13$ sur 5 parts,
     # soit 2,6% du capital) — inacceptable à 5,00$ de capital ; le risque devient binaire]
  redeem à 1,00$/part si gagné, 0 sinon              # aucun frais de settlement (P0-1)
```

Chaque condition remonte à A4/B4/P0 comme annoté. [HYPOTHÈSE n°2] est le point que la Partie D jugera.

## C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE** : cash initial 5,00000$, aucune position, aucun ordre.

**RACE CONDITIONS** :

- (a) RÈGLE : tout signal reçu pendant qu'un ordre est en vol (fenêtre 131 ms) est ignoré — pas de file.
- (b) RÈGLE : aucune vente tant que le fill n'est pas confirmé (tradeIDs, P0-2) ; la stratégie ne vend d'ailleurs jamais avant résolution.
- (c) RÈGLE : deux signaux simultanés (YES et NO) = impossible par construction (un seul écart signé) ; signal opposé pendant une position ouverte → ignoré (position unique).
- (d) RÈGLE : fill partiel FAK → conserver le partiel, ne jamais courir après le solde.


**Latence appliquée à chaque entrée (r1, r2)** : tick oracle horodaté T → lecture T+68 ms → décision T+69 ms → matching CLOB T+100 ms → confirmation T+131 ms. Le prix d'exécution est le BBO en vigueur à T+100 ms dans bbo.csv.

**Déroulé** : ticks oracle 00:02.000 (65 105,85$, écart −28,61$), 00:03.000 (65 099,43$, −35,03$), 00:04.000 (65 097,89$, −36,57$) → 3 ticks confirmés à 00:04.000 ; matching à **00:04.100**. BBO en vigueur (dernière ligne ≤ 4 100 ms, B-F1) : NO 0,85 / **0,86**. NO ask 0,86 ≤ 0,90 → achat FAK 5 parts @ 0,86$. Frais P0-1 : 0,07 × 5 × 0,86 × 0,14 = **0,04214$**. Slippage vs prix au moment du signal (0,86$ à 00:03.449) : 0,00$ ; vs prix 2 ticks plus tôt (0,85$) : 0,01$/part — coût de la confirmation. (r3 : le niveau 0,86$ était posé depuis 651 ms, non disparu ; r5 : débit 4,34214$ ≤ cash 5,00000$.)

Signal opposé notable : 01:30.000–01:32.000, oracle à +10,94$/+10,95$/+11,62$ au-dessus du strike = 3 ticks confirmés côté YES à 01:32.000 — **ignoré par la règle (c)** (position ouverte, cash restant 0,65786$ < coût de 5 parts). J'y reviens en D8.

Résolution : TWAP final 65 110,92$ < strike 65 134,46$ (écart −23,54$, B4-F3) → **DOWN**, NO redeem à 1,00$/part, sans frais (P0-1).

**JOURNAL DE TRADES** :

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | 00:04.000 | 00:04.100 | 0,86$ (NO) | 5 | 05:00.000 (résolution) | 1,00$ (redeem) | 0,00$ | 0,04214$ | +0,65786$ | 0,65786$ → **5,65786$**


**RÉSULTATS** :

- Trades : 1 gagnant (+0,65786$), 0 perdant. Frais totaux : 0,04214$.
- BÉNÉFICE NET : **+0,65786$**. Capital final : **5,65786$** vs 5,00$ (**+13,16%**).
- Pire perte latente : à 01:32.973, no_bid = 0,53$ (B4-F1) → position marquée 2,65$ vs débit 4,34214$, soit **−1,69214$** latent ; equity plancher 3,30786$ ; **drawdown max −33,84%**. Aucune perte réalisée.
- P&L THÉORIQUE (sans r1–r4, mêmes 5 parts au même instant, sans frais) : 5 × (1,00 − 0,86) = **+0,70000$**. Coût du réalisme : 0,04214$ (100% imputable aux frais ; la latence n'a pas déplacé le prix sur cette entrée).


---

# 6. PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----
| D1 Latence de boucle | 131 ms négligeable vs signal | Prix identique à T+100 ms (0,86$ posé depuis 651 ms, B-F1) ; mais la confirmation 3 ticks coûte 0,01$/part vs entrée à 2 ticks | −0,05$ (5 × 0,01$) | [DÉGRADE]
| D2 Fraîcheur du signal | Oracle lu à 68 ms utilisable | 68 ms = 6,8% de la cadence oracle (1 000 ms, B4-F3) ; aucun tick n'a été manqué | 0,00$ (prouvé : même tick décisionnel) | [SANS IMPACT]
| D3 Slippage / profondeur | 5 parts absorbées au BBO | Ordre de 4,30$ = 0,05% du volume minute 1 (7 862,96$, B4-F2) ; trade médian de la session 4,54$ | 0,00$ constaté | [SANS IMPACT]
| D4 Frais complets | 0,07 × C × p × (1−p), pas de gas | Conforme P0-1 ; 0,04214$ = 6,0% du gain brut ; redeem gratuit | −0,04214$ | [DÉGRADE]
| D5 Niveau disparu / fill partiel | FAK, conserver le partiel | 0 re-pricing subi cette session ; le niveau a tenu 651 ms (B-F1) | 0,00$ constaté (risque réel non nul sur d'autres sessions) | [SANS IMPACT] (session)
| D6 Défaillances techniques | Ordre unique, échec = pas de trade | Dead man's switch et heartbeat disponibles (docs.polymarket.com/api-reference/set-auto-cancel) ; pire cas = entrée manquée | Borné à 0,00$ (P&L nul, jamais négatif : pas de jambe orpheline) | [SANS IMPACT]
| D7 Passage à l'échelle | Réplicable à taille supérieure | bbo.csv ne porte pas les tailles ; borne par le flux : au-delà de ~1% du volume minute 1 (≈78,63$), l'exécution au BBO n'est plus garantie — DONNÉE NON DISPONIBLE au-delà | Mécanisme borné à ~80$/entrée (méthode : 1% du volume B4-F2) | [DÉGRADE]
| D8 Dépendance à la session | Session calme = pas de trade (P&L 0,00$) | VRAI, mais dans CETTE session le signal symétrique YES s'est déclenché à 01:32.000 (3 ticks ≥ +10,00$, max +14,95$, B4-F3) et n'a été bloqué que par le cash ; exécuté avec du capital frais, YES @ 0,43$ (yes_ask à 01:32.925, B-F1) aurait perdu 100% de la jambe | Perte évitée par contrainte, pas par la règle : −2,17$ potentiels (5 × 0,43$ + frais 0,02107$) | **[DÉGRADE]** (majeur)
| D9 Conformité au nouveau fonctionnement | Tenue jusqu'à résolution compatible TWAP 30 s | P0-3 vérifié : TWAP final 65 110,92$ vs strike 65 134,46$, écart −23,54$ contre une amplitude oracle max de 3,57$ dans la fenêtre (B4-F3) — aucun scénario de bascule de dernière seconde | 0,00$ (prouvé : 23,54$ > 3,57$) | [SANS IMPACT]


**VERDICT GLOBAL** (règles mécaniques) : zéro [DÉTRUIT] ; P&L net réel positif (+0,65786$) ; conformité P0 totale. Mais D8 révèle une dépendance à un événement unique : le SEUIL de 10,00$ a produit un signal perdant dans la même session, neutralisé par la contrainte de cash et non par la logique. La condition « D8 ≠ dépendance exclusive à un événement unique » exigée pour [FIABLE] n'est pas remplie → **[FRAGILE]**, étiquette **[SESSION-SPÉCIFIQUE]** (le mécanisme oracle-vs-strike mérite un re-test multi-sessions avant toute promotion en pattern).

---

# 7. AUTO-CONTRÔLE FINAL

- Contrôle 1 (Phase 0 complète, sourcée, changement identifié) → OK : P0-1 à P0-4 avec URL ; changement TWAP 30 s daté du 04/08/2026 ; taille minimale d'ordre marquée NON VÉRIFIABLE avec hypothèse conservatrice.
- Contrôle 2 (≥5 valeurs/graphe, ≥8/CSV) → OK : 5–6 valeurs par bloc A-G1…A-G6 (G6 : bornes+compteurs, chiffrage renvoyé à B4-F1) ; 8 lignes dans chaque tableau B4.
- Contrôle 3 (chaque règle remonte à une observation antérieure) → OK : strike (B4-F3), seuil (B4-F3), confirmation (cadence B4-F3), prix max (P0-1), côté (OBS-1), tenue (P0-3).
- Contrôle 4 (latence reconstituée et appliquée) → OK : 68+1+31 = 100 ms au matching, +31 ms confirmation = 131 ms ; appliquée à l'unique entrée du journal (00:04.000 → 00:04.100).
- Contrôle 5 (cohérence comptable) → OK : 5,00000 − 4,30000 − 0,04214 = 0,65786 ; + 5,00000 de redeem = 5,65786$, au centime près.
- Contrôle 6 (D1–D9 traités, [SANS IMPACT] prouvés) → OK : D2 (6,8% de cadence), D3 (0,05% du volume), D6 (borne 0,00$), D9 (23,54$ > 3,57$) ; verdict global conforme aux règles mécaniques.
- Contrôle 7 (aucun chiffre en double contradictoire) → OK : strike PDF 65 133$ explicitement distingué du strike opérationnel 65 134,46$ (B6-F3) ; 5,65786$, +13,16%, 0,04214$, +0,65786$, [FRAGILE] identiques en ouverture et dans le corps.


---

# 8. CLÔTURE

La stratégie gagne sur cette session, mais elle gagne en partie parce qu'elle n'avait plus de cash au moment où sa propre règle se trompait. Vous financez le re-test multi-sessions, pas le déploiement. Le mécanisme à surveiller reste l'écart oracle-strike sous le nouveau régime TWAP.

---

## Source : 290CONVERGENCE_ORACLE-STRIKE_EN_FIN_DE_FENÊTRE.md

# SPEECH AU COMITÉ D'INVESTISSEMENT — SESSION POLYMARKET BTC 5MIN DU 07/08/2026, 9:15–9:20 AM ET

---

## 0. SYNTHÈSE EXÉCUTIVE

Voici mon verdict avant toute démonstration, chaque chiffre est repris à l'identique dans le corps.

- Stratégie retenue : **CONVERGENCE ORACLE-STRIKE EN FIN DE FENÊTRE** (achat YES sur marge oracle confirmée, portage jusqu'à résolution).
- Capital final : **6,09$** vs 5,00$ de départ, soit **+21,8 % net** après latences, slippage et frais réels.
- Trades : **1 trade, 1 gagnant / 0 perdant**.
- Changement Polymarket le plus impactant : depuis le **07/08/2026**, les marchés crypto 5 minutes se résolvent sur un **TWAP Chainlink de 30 secondes** et non plus sur un prix instantané (docs.polymarket.com, annonce du 07/08/2026) — la mèche de dernière seconde ne peut plus inverser un résultat.
- VERDICT DE FIABILITÉ : **[FRAGILE]** — P&L net réel positif, zéro facteur [DÉTRUIT], mais profondeur de carnet non vérifiable et une seule session de preuve.
- Étiquette : **[PATTERN CANDIDAT]** — le mécanisme est structurel, la preuve est mono-session.


---

## 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET (VÉRIFIÉ LE 09/08/2026)

Vous ne pouvez pas juger une stratégie sur des règles périmées, alors voici l'état du terrain tel que la documentation officielle le décrit aujourd'hui.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × 0,07 × p × (1−p) ; maker : 0 ; rebate maker 20 % ; arrondi à 5 décimales, min 0,00001 USDC | docs.polymarket.com/polymarket-learn/trading/fees | consulté 09/08/2026 | Non (barème en vigueur)
| P0-1 Gas / dépôt / retrait | 0$ de frais Polymarket ; ordres signés hors-chaîne, pas de gas payé par le trader au fill | docs.polymarket.com/polymarket-learn/trading/fees | consulté 09/08/2026 | Non
| P0-2 Types d'ordres | Limit uniquement : GTC, GTD, FOK, FAK ; matching prix-temps ; taker delay possible sur certains marchés | docs.polymarket.com (CLOB API) | consulté 09/08/2026 | Non
| P0-2 Tick size / taille min | Par marché, exposés via `GET /book` du CLOB. Valeur exacte pour CE marché : DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : tick 0,01$ (cohérent avec les 822 BBO observés, tous à 2 décimales), taille min ≤ 5,00$ | docs.polymarket.com (CLOB API) | consulté 09/08/2026 | Non
| P0-3 Résolution crypto 5 min | **TWAP Chainlink 30 s** (60 s pour les 15 min et 4 h), flux Chainlink Data Streams / RTDS WebSocket `wss://ws-live-data.polymarket.com`, topic `prices.crypto.chainlink.twap`, prix E18 | docs.polymarket.com + annonce officielle | **effectif 07/08/2026** | **OUI — remplace le snapshot instantané, il y a 48 h**
| P0-4 Rate limits CLOB | `POST /order` : 5 000 req/10 s burst, 120 000 req/10 min ; `/book` 1 500 req/10 s ; `/price` 1 500 req/10 s ; général CLOB 9 000 req/10 s | docs.polymarket.com/api-reference/rate-limits | consulté 09/08/2026 | Non


Le changement critique est P0-3 : cette session du 07/08/2026 est précisément la première journée du régime TWAP. Toute stratégie qui parie sur une manipulation ou une mèche du dernier tick est morte par construction ; toute stratégie qui parie sur la **persistance** d'un écart oracle-strike est mécaniquement renforcée, car le TWAP fige progressivement le résultat sur les 30 dernières secondes.

## 2. CADRE D'ANALYSE

Je lis les 6 graphes comme des hypothèses visuelles, je les vérifie au tick près dans les 4 CSV, et seules les observations chiffrées survivantes ont le droit de fonder une règle de trading. Le référentiel temporel est t=0 à l'ouverture de la fenêtre (9:15:00 ET), le référentiel de résolution est l'oracle Chainlink (P0-3), pas Binance. Résultat officiel de la session : ▲ UP.

---

## PARTIE A — LES 6 GRAPHES

### A-G1 — CARNET YES/NO COMPLET (822 ÉVÉNEMENTS BBO, VUE 300 s + ZOOMS 15 s)

**A1. IDENTITÉ.** Axe X : temps mm:ss.mmm depuis l'ouverture, 00:00 → 05:00 ; axe Y : prix en $ de 0,0 à 1,0 ; 6 séries (bid/ask/mid YES et NO), 822 événements, marqueurs creux pour 42 points "carnet croisé — suspects".

**A2. QUESTION.** À quel moment et à quel prix le carnet a-t-il offert du YES sous sa probabilité réelle de gain ?

**A3. MÉTHODE.** Lecture des étiquettes d'extrema et des trois zooms 15 s, croisée ensuite avec bbo.csv.

**A4. VALEURS EXTRAITES.**

- t=00:00.197 : YES mid 0,46$ / NO mid 0,54$ (première cotation complète)
- t=01:54.265 : YES mid 0,30$ (plancher de la session) / NO mid 0,69$
- t=02:16.929 : YES ask 0,46$ → 02:17.085 : 0,48$ (repricing en 156 ms)
- t=04:00.050 : YES bid/ask 0,79/0,80$ (zoom 04:00–04:15)
- t=04:11.465 : YES ask 0,97$ ; t=04:20.286 : YES mid 0,99$ / NO mid 0,01$ (convergence finale)


**A5. OBSERVATIONS BRUTES.** (1) Le carnet reprice les chocs en 150–700 ms (02:16.409 → 02:17.085 : ask YES 0,38→0,48$). (2) À 04:00.050 le carnet vend encore du YES à 0,80$ alors que la résolution interviendra 60 s plus tard. (3) Après 04:20.286 le côté ask YES disparaît : toute entrée tardive au-delà de 0,99$ est impossible.

**A6. SPÉCIFIQUE À LA SESSION.** Le creux à 0,30$ (01:54.265) et la remontée terminale à 0,99$ (04:20.286) dessinent un marché qui a inversé son consensus en 146 s — amplitude 0,69$, propre à cette session.

### A-G2 — SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT (VUE 300 s + ZOOMS)

**A1. IDENTITÉ.** Axe X : temps mm:ss.mmm ; axe Y : prix BTC en $ (65 160–65 260) ; 2 séries : Binance BTC/USDT direct (7 010 ticks), Chainlink BTC/USD oracle (278 ticks) ; ligne horizontale strike 65 173$ ; sous-graphe basis = spot − oracle (0 à 60$).

**A2. QUESTION.** Lequel des deux flux détermine le résultat, et avec quel décalage l'autre l'anticipe-t-il ?

**A3. MÉTHODE.** Lecture des étiquettes d'extrema et du sous-graphe basis, vérifiée dans spot.csv et oracle.csv.

**A4. VALEURS EXTRAITES.**

- t=00:00.000 : oracle 65 173,13$ = strike (dernier tick ≤ t0)
- t=00:00.066 : Binance 65 223,47$ — basis d'ouverture +50,34$
- t=01:54.415 : Binance 65 214,53$ (plancher Binance de la session)
- t=02:17.000 : oracle 65 174,94$ — premier tick au-dessus du strike après la phase basse
- t=04:45.189 : Binance 65 271,18$ (sommet) ; t=04:59.064 : Binance 65 270,00$


**A5. OBSERVATIONS BRUTES.** (1) Le basis est un plateau stable (bande 40–53$ sur le sous-graphe), donc Binance donne la **direction** de l'oracle, jamais son **niveau**. (2) Le saut Binance de 02:16.752 (65 222,83$) précède le saut oracle de 02:17.000 (65 174,94$) de 248 ms sur cette transition. (3) L'oracle est cadencé à 1 000 ms nominal — un signal Binance vieux de 219 ms reste plus frais que le prochain print oracle.

**A6. SPÉCIFIQUE À LA SESSION.** Le strike 65 173,13$ tombe dans le premier décile de la plage oracle de la session (65 163,63–65 224,78$) : la session était structurellement biaisée UP après 02:17.

### A-G3 — FLUX DIRECTIONNEL NORMALISÉ YES/NO (1 618 TRADES, FENÊTRE 30 s)

**A1. IDENTITÉ.** Axe X : temps ; axe Y : volume $ (−2 000 à +6 000), flux haussier 30 s (BUY YES + SELL NO) en positif, baissier en négatif ; 1 618 trades.

**A2. QUESTION.** Le gros argent a-t-il précédé ou suivi le basculement de l'oracle ?

**A4. VALEURS EXTRAITES.**

- t=00:00.319 : −100,01$ (premier bloc baissier)
- t=00:23.900 : −738,65$ (plus gros trade baissier de la session)
- t=02:18.715 : −557,76$ (contre-attaque baissière APRÈS le franchissement du strike)
- t=04:09.384 : +496,64$ ; t=04:20.867 : **+893,84$** (plus gros trade de la session, haussier)
- t=04:50.758 : −77,34$ (dernier bloc significatif)


**A5. OBSERVATIONS BRUTES.** (1) Les plus gros tickets haussiers (496,64$ ; 893,84$) arrivent à 04:09–04:21, soit APRÈS que la marge oracle a dépassé +16$ — la foule paie la certitude, elle ne l'anticipe pas. (2) À 02:18.715, un vendeur de 557,76$ se trompe de référentiel 1,7 s après le franchissement du strike par l'oracle.

**A6. SPÉCIFIQUE À LA SESSION.** Le pic haussier terminal à +6 000$/30 s est propre à ce dénouement UP sans suspense.

### A-G4 — IMBALANCE DIRECTIONNELLE (VUE 300 s + ZOOMS)

**A1. IDENTITÉ.** Axe Y : imbalance = haussier/(haussier+baissier) sur 0–1, ligne d'équilibre 0,5 ; 1 618 trades.

**A2. QUESTION.** L'imbalance a-t-elle de l'avance sur le prix du carnet ?

**A4. VALEURS EXTRAITES.**

- t=00:00.319 : 0,00 (démarrage 100 % baissier)
- t=02:09.176 : 0,32 ; t=02:12.706 : 0,29 (plancher du zoom 02:09–02:24)
- t=02:18.489 : 0,40 (rebond après le franchissement oracle)
- t=04:00.101 : 0,56 ; t=04:00.866 : 0,55
- t=04:14.002 : 0,82 ; t=04:21.844 : 0,84 (sommet de la vue 300 s)


**A5. OBSERVATIONS BRUTES.** À 04:00, l'imbalance n'est qu'à 0,56 alors que la marge oracle est déjà +16,58$ (B-F2) : le flux agrégé est en retard de 7 à 14 s sur l'information oracle (0,82 seulement à 04:14.002).

**A6. SPÉCIFIQUE À LA SESSION.** Le 0,00 d'ouverture (00:00.319) reflète un unique trade baissier initial, artefact de normalisation sur échantillon vide.

### A-G5 — SPREADS YES ET NO ABSOLUS (VUE 300 s + ZOOMS)

**A1. IDENTITÉ.** Axe Y : spread ask−bid en $ (0,00–0,12) ; 2 séries (YES, NO), 822 événements.

**A2. QUESTION.** Combien coûte la traversée du carnet, et quand explose-t-elle ?

**A4. VALEURS EXTRAITES.**

- t=00:00.197 : 0,01$ (ouverture)
- t=02:30.341 : **−0,01$** (spread négatif = carnet croisé)
- t=02:33.522 : **0,12$** (maximum de la session, pendant la reprise violente)
- t=04:00.050 : 0,00$ puis 0,01$ (spread minimal au moment de mon entrée)
- t=04:07.297 : 0,08$ (élargissement transitoire pendant le rally final) ; t=04:11.466 : 0,01$


**A5. OBSERVATIONS BRUTES.** (1) Le spread médian tient à 0,02$ (confirmé B-F1 : moyenne 0,0224$) : traverser coûte 1 tick en régime calme. (2) Les explosions de spread (0,12$ à 02:33.522) durent moins de 200 ms (0,12$ à 02:33.522 → 0,04$ à 02:33.611).

**A6. SPÉCIFIQUE À LA SESSION.** Un seul épisode ≥0,10$ en 300 s ; une session plus nerveuse en produirait davantage et dégraderait mon hypothèse de slippage.

### A-G6 — ÉCART INTER-CARNETS YES VS NO (INCOHÉRENTS : 42)

**A1. IDENTITÉ.** Axe Y : yes_mid − (1 − no_mid) en $, bande ±0,004 ; ligne de cohérence parfaite à 0 ; 42 points suspects en marqueur creux.

**A2. QUESTION.** Les 42 incohérences carnet croisé sont-elles une source de profit sans risque ?

**A4. VALEURS EXTRAITES.**

- Amplitude de l'écart : bornée par l'axe à ±0,004$ sur les 4 vues (aucune étiquette hors bande)
- 42 événements marqués suspects sur 822 (5,1 %)
- Vérification CSV (B-F1) : seulement **2** croisements stricts ask YES + ask NO < 1 : t=02:30.341 (0,42+0,57=0,99$) et t=03:41.925 (0,69+0,30=0,99$)
- Edge brut par croisement : 0,01$ par paire
- Fenêtre de vie du premier croisement : 16 ms (150 341 → 150 357 ms)


**A5. OBSERVATIONS BRUTES.** L'écart inter-carnets vit dans ±0,004$ : les deux carnets sont synchronisés par les arbitragistes en place ; les 42 "suspects" sont majoritairement des états transitoires d'un côté du carnet pendant la mise à jour de l'autre.

**A6. SPÉCIFIQUE À LA SESSION.** 2 croisements exploitables en 300 s, 0,01$ chacun : c'est le plafond de cette micro-inefficience ici.

---

## PARTIE B — LES 4 CSV

### B-F1 — bbo.csv (CARNET)

**B1. IDENTITÉ.** Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 822 lignes ; période 197 ms → 260 286 ms ; fréquence événementielle (médiane irrégulière, rafales <10 ms).

**B2. APPORT.** Le prix exécutable exact à toute milliseconde — ce que le graphe G1 ne donne que par extrema.

**B3. CALCULS.** Spread = yes_ask − yes_bid ligne à ligne ; croisement strict si yes_ask + no_ask < 1 ; état du carnet à t = dernière ligne ≤ t.

**B4. RÉSULTATS.**

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Événements BBO | 822 | toutes | 822
| Croisements stricts (ask+ask<1) | 2 (t=150 341 ms ; t=221 925 ms) | yes_ask, no_ask | 822
| Edge brut par croisement | 0,01$/paire | yes_ask, no_ask | 2
| Spread YES moyen | 0,0224$ | yes_bid, yes_ask | 820
| Spread YES médian / max | 0,02$ / 0,12$ (t=153 522 ms) | yes_bid, yes_ask | 820
| YES ask à t=240 050 ms | 0,80$ (bid 0,79$) | t_ms, yes_ask | 1
| YES ask à t=251 466 ms | 0,98$ | t_ms, yes_ask | 1
| Dernier BBO deux-côtés | t=260 286 ms (bid 0,99$, ask absent ensuite) | toutes | 1


**B5. OBSERVATIONS BRUTES.** (1) À 240 050 ms, acheter YES coûte 0,80$ et le niveau tient jusqu'à 241 724 ms (1 674 ms de fenêtre exécutable). (2) Entre 245 707 et 245 939 ms, le bid YES recule de 0,81 à 0,74$ puis récupère 0,80$ à 246 949 ms : dernière secousse avant convergence.

**B6. ANOMALIES.** Les 2 croisements (150 341 ; 221 925 ms) durent respectivement 16 ms et moins d'une ligne d'événement — bruit de synchronisation, pas signal. Première ligne (197 ms) sans côté YES : carnet en cours d'initialisation.

### B-F2 — oracle.csv (CHAINLINK BTC/USD)

**B1. IDENTITÉ.** Colonnes t_ms, price, ts_src ; 278 lignes ; 0 → 298 000 ms ; cadence nominale 1 000 ms ; ts_src=payload sur 278/278 (0 fallback).

**B2. APPORT.** Le référentiel de résolution lui-même (P0-3) — ni les graphes ni Binance ne font foi.

**B3. CALCULS.** Strike = price(t=0) ; marge(t) = price(t) − strike ; max drawdown 90 s = max sur i de [price(tᵢ) − min price sur ]tᵢ ; tᵢ+90 000]] ; TWAP final approximé par la moyenne des ticks ≥ 270 000 ms.

**B4. RÉSULTATS.**

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Strike (dernier tick ≤ t0) | 65 173,13$ | t_ms, price | 1
| Min / max session | 65 163,63$ (t=35 000) / 65 224,78$ (t=290 000) | price | 278
| Dernier tick sous le strike | t=136 000 ms (116 ticks sous strike au total) | t_ms, price | 278
| Marge minimale après t=137 000 ms | **+1,34$** (65 174,47$) | price | 163
| Marge à t=240 000 ms | **+16,58$** (65 189,70$) | t_ms, price | 1
| Baisse max sur toute fenêtre 90 s | **11,88$** | t_ms, price | 278
| Moyenne des ticks ≥ 270 000 ms (proxy TWAP 30 s) | 65 222,33$, marge **+49,20$** | t_ms, price | 28
| Trou de cadence max | 7 000 ms (après t=60 000) ; 16 trous >1 500 ms | t_ms | 277


**B5. OBSERVATIONS BRUTES.** (1) Franchissement définitif du strike à t=137 000 ms : 163 ticks consécutifs au-dessus, zéro retour. (2) La condition de victoire "marge > baisse max observable en 90 s" devient vraie à t=161 000 ms (+16,01$) et le reste jusqu'à la fin.

**B6. ANOMALIES.** Trou de 7 000 ms entre 60 000 et 67 000 ms et 16 trous >1 500 ms : l'oracle n'est pas un métronome ; toute stratégie doit tolérer 7 s sans mise à jour. Bruit, pas signal.

### B-F3 — spot.csv (BINANCE BTC/USDT DIRECT)

**B1. IDENTITÉ.** Colonnes t_ms, price ; 7 010 lignes ; 66 → 299 064 ms ; fréquence tick-par-tick avec rafales de timestamps identiques.

**B2. APPORT.** Le flux le plus rapide de la session — 25 fois plus dense que l'oracle — pour mesurer qui anticipe qui.

**B3. CALCULS.** basis(tᵢ) = dernier spot ≤ tᵢ − oracle(tᵢ), calculé aux 278 timestamps oracle.

**B4. RÉSULTATS.**

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Ticks | 7 010 | toutes | 7 010
| Premier / dernier | 65 223,47$ (t=66) / 65 270,00$ (t=299 064) | t_ms, price | 2
| Min / max session | 65 214,53$ (t=114 415) / 65 271,18$ (t=285 189) | price | 7 010
| Basis moyen (spot − oracle) | **+47,51$** | les 2 fichiers | 278
| Basis min / max | +40,28$ / +52,84$ | les 2 fichiers | 278
| Écart-type du basis | 2,21$ | les 2 fichiers | 278
| Avance de Binance sur la transition 02:17 | 248 ms (spot 65 222,83$ à t=136 752 vs oracle 65 174,94$ à t=137 000) | t_ms, price | 2
| Plage totale Binance | 56,65$ | price | 7 010


**B5. OBSERVATIONS BRUTES.** Le basis σ=2,21$ sur une moyenne de 47,51$ : l'écart USDT/USD + composition d'agrégation est un offset quasi constant. Binance ne prédit jamais le NIVEAU de résolution, uniquement sa dérivée.

**B6. ANOMALIES.** Rafales de dizaines de ticks au même t_ms (66 ms : 25 lignes) : granularité du flux agrégé, à dédupliquer avant tout calcul de fréquence. Bruit.

### B-F4 — trades.csv (FLUX D'ORDRES POLYMARKET)

**B1. IDENTITÉ.** Colonnes t_ms, usd, direction (+1 haussier / −1 baissier) ; 1 618 lignes ; 319 → ≈299 000 ms.

**B2. APPORT.** Qui paie, combien, quand — la matière première des graphes G3 et G4.

**B3. CALCULS.** Sommes par direction, par fenêtre de 60 s ; extrêmes ; quantiles.

**B4. RÉSULTATS.**

| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Trades | 1 618 (965 haussiers / 653 baissiers) | direction | 1 618
| Volume total | 26 085,68$ | usd | 1 618
| Volume haussier / baissier | 14 104,94$ / 11 980,74$ (net +2 124,21$) | usd, direction | 1 618
| Fenêtre 0–60 s | haussier 2 063,86$ / baissier 3 012,36$ | usd, direction | 345
| Fenêtre 240–300 s | haussier **4 826,18$** / baissier 1 560,94$ | usd, direction | 274
| Plus gros trade | +893,84$ (t=260 867, haussier) | usd | 1
| Plus gros baissier | −738,65$ (t=23 900) | usd | 1
| Médiane / p90 des tailles | 4,20$ / 39,25$ | usd | 1 618


**B5. OBSERVATIONS BRUTES.** (1) 75 % du flux haussier net de la session se concentre dans les 60 dernières secondes — l'argent suit l'oracle avec retard. (2) La médiane à 4,20$ signifie que mon ordre de 4,93$ est un trade médian : invisible, non impactant.

**B6. ANOMALIES.** Trois blocs baissiers >400$ (23 900 ; 138 715 ; 194 840 ms) exécutés alors que l'oracle était déjà au-dessus du strike pour les deux derniers — erreurs de référentiel ou couvertures, non reproductibles comme signal.

---

## PARTIE C — STRATÉGIES ET SIMULATION

### C0. CANDIDATES

Mes cinq observations les plus fortes, chacune déjà démontrée plus haut :

- **OBS-1** (B-F2/B5) : l'oracle franchit le strike à t=137 000 ms et n'y retourne jamais (marge min ultérieure +1,34$).
- **OBS-2** (B-F2/B4) : la pire baisse de l'oracle sur 90 s glissants est 11,88$ ; à t=240 000 ms la marge est +16,58$, supérieure à ce pire cas.
- **OBS-3** (B-F1/B5) : à t=240 050 ms le carnet vend du YES à 0,80$, et le niveau reste exécutable 1 674 ms.
- **OBS-4** (B-F3/B5) : le basis Binance−oracle est un offset stable (+47,51$, σ 2,21$) — Binance donne la direction, l'oracle seul donne le résultat.
- **OBS-5** (B-F4/B5 et A-G4/A5) : le flux haussier converge APRÈS 04:07 (imbalance 0,56 à 04:00.101 → 0,82 à 04:14.002) — la fenêtre 04:00–04:07 est mal pricée.


**Latences de boucle (chemins reconstitués depuis le tableau, uniques valeurs autorisées) :**

- Boucle A (oracle → CLOB) : lecture Chainlink on-chain 68 ms + décision 1 ms + envoi CLOB 31 ms = **100 ms**, confirmation +31 ms = **131 ms**.
- Boucle B (Binance → CLOB) : âge signal Binance 219 ms + décision 1 ms + envoi 31 ms = **251 ms**, confirmation +31 ms = **282 ms**.
- Boucle C (lecture carnet → 2 ordres) : lecture CLOB 31 ms + décision 1 ms + envoi 31 ms + confirmation 31 ms = **94 ms** minimum.


**CRITÈRE DE SÉLECTION : maximiser le P&L NET RÉEL simulé sur la session (après latences, slippage r3, frais P0-1), sous trois contraintes éliminatoires : conformité P0 totale, boucle plus rapide que la durée de vie du signal, et perte maximale bornée par une grandeur observée (OBS-2).** Ce critère est posé avant toute comparaison.

**Candidate 1 — CONVERGENCE ORACLE-STRIKE EN FIN DE FENÊTRE.** Logique : quand la marge oracle dépasse le pire reflux observable sur l'horizon restant et que le carnet price encore ≥15 % de probabilité inverse, j'achète YES et je porte jusqu'à la résolution TWAP. Règles : entrée si t ≥ 240 000 ms ET marge ≥ 15$ (OBS-2) ET yes_ask ≤ 0,85$ (OBS-3) ; aucune sortie avant résolution (OBS-1, P0-3). Signal vivant 1 674 ms minimum vs boucle 131 ms → **faisable**. Conforme P0 : ordre limite GTC taker (P0-2), frais P0-1 appliqués, résolution TWAP en ma faveur (P0-3).

**Candidate 2 — LATENCE BINANCE → CARNET.** Logique : utiliser le flux Binance (dense, directionnel, OBS-4) pour acheter le carnet avant qu'il ne reprice un choc spot. Test structurel : sur la transition de 02:16, le carnet commence à repricer à t=136 409 ms, soit **343 ms AVANT** le print Binance de 136 752 ms tel que je le recevrais (âge 219 ms → lecture effective 136 971 ms) ; mon ordre arriverait à 137 003 ms, quand l'ask YES cote déjà 0,46–0,49$ (B-F1 : 0,46$ à 136 931). La durée de vie du signal (≤343 ms, parfois négative) est inférieure à ma boucle de 251 ms. **[STRUCTURELLEMENT IMPOSSIBLE]** — éliminée avant comparaison.

**Candidate 3 — ARBITRAGE CARNET CROISÉ.** Logique : acheter YES et NO quand ask+ask <1 (A-G6) et encaisser 1$ garanti à résolution. Faisabilité : 2 occurrences, fenêtre de 16 ms vs boucle de 94 ms pour deux jambes → **[STRUCTURELLEMENT IMPOSSIBLE]**. Et même à latence nulle : frais taker des deux jambes = 0,07×[0,42×0,58 + 0,57×0,43] = 0,0342$/paire > edge 0,0100$/paire → P&L net négatif par construction (P0-1). Doublement éliminée.

**Candidate 4 — MARKET MAKING MAKER (FRAIS 0).** Logique : coter les deux côtés au spread médian de 0,02$ (A-G5), frais maker nuls et rebate 20 % (P0-1). Faisable en latence, conforme P0, mais bbo.csv ne contient **aucune colonne de taille** : la probabilité de fill, la position dans la file prix-temps et le risque d'inventaire en fin de fenêtre sont DONNÉE NON DISPONIBLE. Non simulable honnêtement — éliminée faute de données, pas faute de mérite.

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----|-----
| 1. Convergence oracle-strike | OBS-1,2,3,5 | Oui | 1 | +1,25$ | **+1,09$** | **RETENUE**
| 2. Latence Binance→carnet | OBS-4 | Oui | 0 | — | — | [STRUCTURELLEMENT IMPOSSIBLE]
| 3. Arbitrage carnet croisé | A-G6/B-F1 | Oui | 0 | +0,01$/paire | négatif (frais > edge) | [STRUCTURELLEMENT IMPOSSIBLE]
| 4. Market making maker | A-G5 | Oui | — | DONNÉE NON DISPONIBLE | DONNÉE NON DISPONIBLE | Éliminée (non simulable)


Décision : la Candidate 1 est la seule survivante du filtre structurel avec un P&L net réel simulable et positif — elle est retenue par application directe du critère.

### C1. LA STRATÉGIE RETENUE — PSEUDO-CODE EXÉCUTABLE

```plaintext
# CONVERGENCE ORACLE-STRIKE EN FIN DE FENÊTRE — capital 5,00$
CONSTANTES:
  STRIKE      = premier tick oracle de la fenêtre        # B-F2/B4 : 65 173,13$
  MARGE_MIN   = 15,00$        # > pire baisse 90 s observée 11,88$ (OBS-2/B-F2)
                              # [FRAGILE - 1 SOURCE] : borne issue d'une seule session
  T_ENTREE    = 240 000 ms    # ≤ 60 s de la clôture, horizon ≤ fenêtre de l'OBS-2
  ASK_MAX     = 0,85$         # plafond de re-pricing (OBS-3 + règle r3)
  BOUCLE      = 131 ms        # 68 (Chainlink) + 1 (décision) + 31 (envoi) + 31 (confirmation)

BOUCLE PRINCIPALE (par tick oracle reçu, âge 68 ms):
  si position_ouverte OU t < T_ENTREE: continuer
  marge = prix_oracle - STRIKE
  si marge >= MARGE_MIN:                          # OBS-2
    ask = yes_ask lu au moment d'arrivée de l'ordre (t_signal + 100 ms)   # B-F1
    prix_limite = min(ask + 0,01, ASK_MAX)        # règle r3 : re-pricing 1 tick max
    C = plancher_2dec( CASH / (prix_limite + 0,07*prix_limite*(1-prix_limite)) )  # P0-1, r5
    envoyer ORDRE LIMITE GTC BUY YES, taille C, prix prix_limite          # P0-2
    si non rempli en 2 000 ms: annuler, réévaluer au tick suivant         # r3
  # aucune vente avant résolution : portage jusqu'au TWAP 30 s            # OBS-1, P0-3
  # [HYPOTHÈSE n°1] taille min d'ordre du marché ≤ 4,93$ (P0-2 non vérifiable)
  # [HYPOTHÈSE n°2] profondeur au meilleur ask ≥ 6,09 parts, sinon fill partiel (règle d)
```

### C-SIM. SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE.** Départ : 5,00$ USDC, 0 part, aucun ordre en vol. Après trade : 0,0015$ cash + 6,09 parts YES. Après résolution : 6,0915$.

**RACE CONDITIONS.**

- (a) Signal d'achat pendant un ordre en cours → RÈGLE : tout nouveau signal est ignoré tant qu'un ordre est non confirmé (flag position_ouverte posé à l'envoi, pas au fill).
- (b) Vente sur position non confirmée → RÈGLE : aucune vente n'existe dans cette stratégie ; le portage jusqu'à résolution supprime la classe entière de ce risque.
- (c) Deux signaux simultanés → RÈGLE : les ticks oracle sont sérialisés par timestamp on-chain ; à égalité, seul le premier traité déclenche, les suivants voient position_ouverte=vrai.
- (d) Fill partiel → RÈGLE : un fill partiel est conservé (le GTC reste au carnet jusqu'à t+2 000 ms puis est annulé) ; la position partielle est portée à résolution comme la position pleine.


**JOURNAL DE TRADES.**

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----
| 1 | 04:00.000 (tick oracle 65 189,70$, marge +16,58$) | 04:00.100 (confirmé 04:00.131) | 0,81$ | 6,09 parts | résolution 05:00 (TWAP) | 1,00$ | +0,01$ (ask 0,80$ → 0,81$, règle r3) | 0,0656$ | **+1,0915$** | 0,0015$ → **6,0915$**


Détail comptable au centime : coût parts 6,09 × 0,81 = 4,9329$ ; frais P0-1 = 6,09 × 0,07 × 0,81 × 0,19 = 0,0656$ ; débit total 4,9985$ ≤ 5,00$ (r5 respecté) ; cash restant 0,0015$ ; règlement 6,09 × 1,00 = 6,0900$ ; capital final 6,0915$.

Vérification r1–r4 : (r1) l'ordre part du tick 04:00.000, vieux de 68 ms à la lecture, et atteint le CLOB à 04:00.100 ; le BBO en vigueur à cet instant est celui de 04:00.050 (0,79/0,80$, B-F1), valable jusqu'à 04:01.724 — le fill à 0,81$ est donc conservateur d'un tick entier. (r2) L'âge du signal est celui de l'oracle (68 ms), pas de Binance : c'est précisément pour cela que la stratégie lit le référentiel de résolution. (r3) Sans données de profondeur, slippage forfaitaire +0,01$ appliqué. (r4) Frais taker crypto P0-1, formule exacte.

**RÉSULTATS.**

| Grandeur | Valeur
|-----|-----|-----|-----|-----
| Trades gagnants / perdants | 1 / 0 (+1,0915$ / 0,00$)
| Frais totaux | 0,0656$
| BÉNÉFICE NET | **+1,09$**
| Capital final vs 5,00$ | **6,09$ (+21,8 %)**
| Pire perte réalisée | 0,00$
| Drawdown max (latent, mark-to-market) | −0,49$ (−9,8 %) : bid YES 0,74$ à 04:05.927, valeur 4,51$
| P&L THÉORIQUE (sans r1–r4) | +1,25$ (+25,0 %) : 6,25 parts à 0,80$
| Coût du réalisme | 0,16$ (12,8 % du P&L théorique)


---

## PARTIE D — VERDICT DE FIABILITÉ

Je passe ma propre stratégie au crible, facteur par facteur.

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----
| D1 Latence de boucle | 131 ms < vie du signal | Niveau 0,80$ exécutable 1 674 ms (B-F1) ; ratio 12,8× | 0,00$ (fill au même niveau, preuve : BBO inchangé de 240 050 à 241 724 ms) | [SANS IMPACT]
| D2 Fraîcheur du signal | Tick oracle vieux de 68 ms utilisable | Cadence oracle 1 000 ms, trous jusqu'à 7 000 ms (B-F2/B6) ; 68 ms = 6,8 % de la cadence | 0,00$ (le tick suivant, 04:01.000 à 65 189,72$, confirme la marge à 0,02$ près) | [SANS IMPACT]
| D3 Slippage & profondeur | +0,01$ forfaitaire, profondeur ≥ 6,09 parts [HYPOTHÈSE n°2] | Aucune donnée de taille dans bbo.csv — DONNÉE NON DISPONIBLE ; borne : au plafond r3 de 0,85$, le P&L net resterait +0,71$ | borné à −0,38$ vs simulation | [DÉGRADE]
| D4 Frais complets | fee = C×0,07×p×(1−p), zéro gas | P0-1 : 0,0656$ payés, 1,3 % du capital ; aucun frais de résolution ni retrait | −0,0656$, déjà intégré | [SANS IMPACT]
| D5 Ordres concurrents / niveau disparu / fill partiel | Règle r3 (re-price ≤0,85$) et règle (d) | Flux adverse à 04:00 : 274 trades en 240–300 s, médiane 4,20$ (B-F4) ; mon ordre est de taille médiane | borné par D3 ; fill partiel = position réduite, même signe de P&L | [DÉGRADE]
| D6 Défaillances techniques | 1 seul ordre, GTC, timeout 2 000 ms | Fenêtre d'entrée exploitable 240 000→251 466 ms (ask ≤0,98$, B-F1) : 11,5 s pour réessayer, soit 87 boucles complètes de 131 ms | manqué = P&L 0,00$, jamais négatif | [SANS IMPACT]
| D7 Passage à l'échelle | Taille médiane du carnet | Volume total au ask dans la fenêtre : DONNÉE NON DISPONIBLE (pas de tailles) ; borne haute observable : 4 826,18$ de flux haussier en 240–300 s (B-F4) — au-delà de quelques centaines de $, je DEVIENS le repricing | s'éteint entre 10² et 10³ $ (borne par B-F4) | [DÉGRADE]
| D8 Dépendance à la session | Conditions d'entrée objectives (marge ≥15$, t ≥240 s, ask ≤0,85$) | Sur une session calme (marge <15$ ou carnet déjà >0,85$), la stratégie ne trade pas : P&L 0,00$, capital intact | 0,00$ sur session calme — prouvé par la règle d'entrée elle-même | [SANS IMPACT]
| D9 Conformité au nouveau Polymarket | La résolution TWAP 30 s (P0-3) est un allié | TWAP final +49,20$ au-dessus du strike (B-F2) ; sous l'ancien régime snapshot, un tick unique adverse restait possible ; sous TWAP il faudrait un effondrement soutenu de 49$ en 30 s, 4,1× la pire baisse 90 s observée | 0,00$, le changement augmente la marge de sécurité | [SANS IMPACT]


**VERDICT GLOBAL : [FRAGILE].** Application mécanique : zéro [DÉTRUIT] et P&L net réel positif, donc [NON FIABLE] est exclu ; mais trois [DÉGRADE] (D3, D5, D7) reposent sur une profondeur de carnet non mesurable dans les pièces fournies, et la borne de risque centrale (11,88$, OBS-2) provient d'une session unique. Les conditions nécessaires de [FIABLE] sont numériquement atteintes sur cette session ; je refuse de le prononcer sur un échantillon de 1, ce qui ne contredit aucune règle mécanique — elles définissent des conditions nécessaires, pas suffisantes. Étiquette : **[PATTERN CANDIDAT]** — le mécanisme (retard du carnet sur le référentiel de résolution en fin de fenêtre, renforcé par le TWAP) est structurel et testable sur les prochaines sessions.

---

## AUTO-CONTRÔLE FINAL

- Contrôle 1 (Phase 0 complète) → P0-1 à P0-4 sourcés avec URL et dates ; changement récent identifié : TWAP 30 s effectif 07/08/2026 ; tick size/taille min marqués DONNÉE NON VÉRIFIABLE avec hypothèse conservatrice. **Conforme.**
- Contrôle 2 (≥5 valeurs par graphe, ≥8 par CSV) → A-G1:5, A-G2:5, A-G3:5, A-G4:6, A-G5:5, A-G6:5 ; B-F1:8, B-F2:8, B-F3:8, B-F4:8. **Conforme.**
- Contrôle 3 (traçabilité des règles) → MARGE_MIN←OBS-2(B-F2), T_ENTREE←OBS-2, ASK_MAX←OBS-3(B-F1), portage←OBS-1(B-F2)+P0-3 ; fragilités marquées inline. **Conforme.**
- Contrôle 4 (latence reconstituée et appliquée) → 68+1+31+31=131 ms, chemin par chemin en C0 ; appliquée à l'unique entrée du journal (signal 04:00.000 → exécution 04:00.100 → confirmation 04:00.131). **Conforme.**
- Contrôle 5 (cohérence comptable) → 5,0000 − 4,9329 − 0,0656 = 0,0015$ ; 0,0015 + 6,0900 = 6,0915$ = capital final annoncé (6,09$ arrondi). **Conforme au centime.**
- Contrôle 6 (D1–D9 traités) → 9/9 ; chaque [SANS IMPACT] porte sa preuve chiffrée (D1 : 1 674 ms vs 131 ms ; D2 : 0,02$ d'écart au tick suivant ; D4 : 0,0656$ intégré ; D6 : 87 boucles disponibles ; D8 : P&L 0,00$ par construction ; D9 : 4,1× la pire baisse) ; verdict global compatible avec les règles mécaniques. **Conforme.**
- Contrôle 7 (unicité des chiffres) → 6,09$ / +21,8 % / 1 trade / 0,0656$ / +1,09$ / 11,88$ / +16,58$ / +49,20$ identiques en ouverture, corps et clôture. **Conforme.**


## CLÔTURE

Vous avez sous les yeux la même chose que moi : un carnet qui vendait la certitude avec 60 secondes de retard sur son propre juge de paix. J'ai chiffré ce retard, je l'ai traversé avec les latences réelles, et j'en suis ressorti avec un gain net — sur une session. La suite raisonnable n'est pas d'augmenter la mise, c'est de rejouer ce protocole sur les prochaines fenêtres.

---

J'ai livré l'audit complet : Phase 0 sourcée (le changement clé est le passage au TWAP Chainlink 30 s le 07/08/2026, qui renforce la stratégie retenue), analyse des 6 graphes et 4 CSV avec valeurs recalculées au tick près, 4 candidates dont 2 éliminées structurellement, et une simulation à 131 ms de boucle aboutissant à 6,09$ (+21,8 %) net de frais. Le verdict est [FRAGILE] uniquement parce que la profondeur du carnet est absente des données et que la preuve est mono-session — les fichiers de travail sont dans `analysis/` si vous voulez que je pousse l'analyse (autres seuils d'entrée, sensibilité au slippage, ou backtest du même protocole sur d'autres sessions).


