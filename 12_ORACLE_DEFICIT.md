# 12 — ORACLE DEFICIT

**Fichiers sources regroupés :** 289ORACLE-DEFICIT_HOLD.md, 287_ORACLE-DEFICIT_MOMENTUM.md

---

## Source : 289ORACLE-DEFICIT_HOLD.md

# SPEECH AU COMITÉ D'INVESTISSEMENT — SESSION POLYMARKET BTC 5MIN DU 7 AOÛT, 9:10–9:15 ET

---

## 0. SYNTHÈSE EXÉCUTIVE

Voici le verdict avant toute chose.

- Stratégie retenue : **ORACLE-DEFICIT HOLD** — achat du côté DOWN (jeton NO) dès que l'oracle Chainlink s'écarte durablement du strike, portage jusqu'à résolution.
- Capital final : **7,7340$** vs 5,00$, soit **+2,7340$ net (+54,68%)**, APRÈS latence de boucle 130 ms, slippage +1 tick et frais taker P0-1.
- Nombre de trades : **1** — 1 gagnant, 0 perdant.
- Changement Polymarket le plus impactant : la résolution des marchés crypto s'appuie désormais sur les prix **TWAP Chainlink relayés par Polymarket RTDS, lancés le 4 août 2026** — 3 jours avant cette session (docs.polymarket.com/market-data/chainlink-twap, consulté le 09/08/2026).
- VERDICT DE FIABILITÉ : **[FRAGILE]**
- Étiquette : **[SESSION-SPÉCIFIQUE]**


---

## 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET (VÉRIFIÉ LE 09/08/2026)

J'ai re-vérifié chaque règle dans la documentation officielle avant d'écrire la moindre règle de stratégie ; voici l'état en vigueur.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × **0,07** × p × (1−p), payé en pUSD, appliqué au match | docs.polymarket.com/trading/fees.md | consulté 09/08/2026 | Oui — étendu à tous les marchés crypto le 06/03/2026 (changelog)
| P0-1 Frais maker | **0** (jamais facturé) + rebate 20% des frais taker | docs.polymarket.com/trading/fees.md | consulté 09/08/2026 | Non
| P0-1 Gas | Aucun pour l'ordre : signature EIP-712 off-chain, settlement par l'opérateur | docs.polymarket.com/trading/place-orders.md | consulté 09/08/2026 | Non
| P0-1 Frais dépôt/retrait/résolution | 0$ côté Polymarket | docs.polymarket.com/trading/fees.md | consulté 09/08/2026 | Non
| P0-2 Tick size | **0,01$** (les prix hors grille sont rejetés) | docs.polymarket.com/market-data/market-details.md | consulté 09/08/2026 | Non pour crypto (0,0025 réservé Coupe du Monde)
| P0-2 Taille min. d'ordre | **5** — CONTRADICTION DOCUMENTAIRE : « minimum number of shares » (place-orders.md) vs « minimum USDC notional » (market-details.md) | les deux URL ci-dessus | consulté 09/08/2026 | DONNÉE NON VÉRIFIABLE sur l'unité — les deux lectures sont traitées en Partie D
| P0-2 Types d'ordres | Limit GTC/GTD (GTD ≥ 3 min de vie effective), FOK, FAK, post-only ; matching prix-temps sur CLOB V2 | docs.polymarket.com/trading/place-orders.md + changelog | CLOB V2 live 28/04/2026 | **Oui** — CLOB V2 + collatéral pUSD depuis le 28/04/2026
| P0-3 Résolution générale | UMA Optimistic Oracle, fenêtre de contestation 2h | docs.polymarket.com/concepts/resolution.md | consulté 09/08/2026 | Non
| P0-3 Résolution crypto 5min | Prix **TWAP Chainlink 30s/60s** calculés par Chainlink, relayés par Polymarket RTDS ; **lancement RTDS le 04/08/2026** ; Chainlink en soak testing jusqu'à cette date | docs.polymarket.com/market-data/chainlink-twap.md | dates du 04/08/2026 dans le doc | **OUI — c'est le changement récent.** La page polymarket-learn/markets/crypto-markets n'a pas répondu (3 tentatives) : le délai exact de résolution 5min = DONNÉE NON VÉRIFIABLE — hypothèse conservatrice appliquée : résolution automatique sur le flux Chainlink, capital immobilisé jusqu'au redeem, sans frais
| P0-4 Rate limits trading | POST /order : 5 000 req/10s (burst), 120 000 req/10min (sustained) | docs.polymarket.com/api-reference/rate-limits.md | consulté 09/08/2026 | Oui — relevés récemment (changelog)
| P0-4 Données temps réel | /book 1 500 req/10s ; /price 1 500 req/10s ; WSS Market Channel sans limite de tokens | docs.polymarket.com/api-reference/rate-limits.md | consulté 09/08/2026 | Non
| P0-4 Matching | Pipeline async depuis le 24/07 : POST /order ne renvoie plus transactionHashes, seulement tradeIDs | docs.polymarket.com/changelog/predictions.md | rollout 24/07, 04:00 UTC | Oui


**Ce qui a changé et ce que cela implique** : la résolution des marchés BTC 5 minutes est ancrée sur le prix Chainlink (TWAP 30s/60s via RTDS depuis le 04/08/2026), pas sur le spot Binance. Vos données le confirment empiriquement : le résultat est ▼ DOWN (PDF, titre) alors que le spot Binance n'est **jamais** passé sous le strike (0 tick sur 9 861 — B-F1). Toute stratégie qui lit Binance comme référence de résolution est invalide par construction. Une stratégie fondée sur un signal ponctuel de dernière seconde est également affaiblie : un TWAP 60s ne bascule pas sur un tick isolé.

---

## 2. CADRE D'ANALYSE

Je lis chaque pièce comme un flux horodaté en ms depuis t0 (ouverture de fenêtre), strike = dernier tick oracle ≤ t0 = 65 203,36$. Je cherche des écarts persistants entre la variable de résolution (oracle Chainlink) et le prix du marché Polymarket. Aucune règle de stratégie n'est formulée avant la fin des Parties A et B.

---

## 3. PARTIE A — LES 6 GRAPHES DU PDF

### A-G1 — CARNET YES/NO COMPLET, VUE 300 s

- **A1.** X : temps mm:ss.mmm sur [00:00.000 ; 05:00.000] ; Y gauche : prix $ [0 ; 1] ; Y droit : probabilité [0 ; 1]. 636 événements BBO, 32 marqués incohérents (critère du graphe), fallback ts 0/287 (0,0%).
- **A2.** Le marché a-t-il convergé tôt ou tard vers DOWN ?
- **A3.** Je lis les étiquettes d'extrema imprimées et la trajectoire des mids YES/NO.
- **A4.** t=00:00.215 : YES ask 0,46$ ; t=00:00.215 : NO ask 0,55$ ; t=04:36.936 : YES 0,01$ ; t=04:36.936 : NO 0,99$ ; consensus YES d'ouverture 0,455 (mid 0,45/0,46).
- **A5.** Le YES ouvre à 0,455 de probabilité et décroît de façon quasi monotone vers 0,01 : la convergence prend 277 s, elle n'est pas instantanée.
- **A6.** Spécifique : l'amplitude 0,455 → 0,01 dépend du résultat DOWN de cette fenêtre ; la vitesse de convergence n'est mesurée que sur cette session.


### A-G2 — CARNET YES/NO, ZOOMS 15 s (00:55–01:10, 03:41–03:56, 00:04–00:19)

- **A1.** Mêmes axes, fenêtres 15 s, précision ms.
- **A2.** Combien de temps une quote de meilleur niveau survit-elle ?
- **A3.** Je compte les paires d'étiquettes consécutives et leurs écarts temporels.
- **A4.** t=00:55.022 : YES 0,29$/NO 0,71$ ; t=01:00.946 : YES 0,25$ ; t=01:04.913 : NO 0,80$ ; t=03:48.742 : YES saute 0,15→0,18$ en 0 ms (deux étiquettes même ts) ; t=00:12.275 : YES 0,29$ contre 0,32$ 19 ms plus tôt.
- **A5.** Les niveaux évoluent par pas de 0,01–0,03$ avec des paliers de 0,5 à 3 s entre mises à jour ; le rebond 03:48.721→03:48.854 (0,14→0,23$ YES) montre qu'un choc de 9 ticks se déroule sur 133 ms.
- **A6.** Spécifique : le choc de 03:48 correspond au rebond spot vers 65 190$ (A-G4) — amplitude propre à cette session.


### A-G3 — SPOT BINANCE vs ORACLE CHAINLINK + BASIS, VUE 300 s

- **A1.** X : mm:ss.mmm [0 ; 300 s] ; Y : prix BTC $ [65 160 ; 65 260] ; sous-graphe basis $ [−4 000 nominal, échelle 0–40 utile] ; 9 861 ticks Binance, 287 ticks Chainlink, strike 65 203$ tracé.
- **A2.** Les deux sources racontent-elles le même marché ?
- **A3.** Je compare les deux courbes au strike et je lis le sous-graphe basis = spot − oracle.
- **A4.** t=00:00.731 : Binance 65 255,08$ ; t=00:00.000 : oracle 65 203,36$ (= strike) ; t=02:19.278 : Binance 65 208,01$ (minimum affiché) ; t=04:59.804 : Binance 65 223,48$ ; t=02:21.000 : oracle 65 156,19$ (creux, lu 65 156–65 157 sur la courbe).
- **A5.** La courbe Binance reste intégralement AU-DESSUS du strike ; la courbe oracle intégralement EN-DESSOUS après t0 ; le basis reste dans une bande étroite positive (~38–57$ au sous-graphe).
- **A6.** Spécifique : un basis moyen de +49$ (B-F1) n'est pas une constante de marché ; il est propre à cet instant de la microstructure BTC/USDT vs BTC/USD.


### A-G4 — SPOT vs ORACLE, ZOOMS 15 s

- **A1.** Mêmes axes, fenêtres 00:55–01:10, 03:41–03:56, 00:04–00:19.
- **A2.** L'oracle retarde-t-il ou diverge-t-il du spot ?
- **A3.** Je compare tick à tick les étiquettes des deux courbes dans chaque fenêtre.
- **A4.** t=00:55.217 : Binance 65 224,01$ vs oracle t=00:55.000 : 65 172,61$ (écart +51,40$) ; t=01:02.830 : Binance 65 210,09$ ; t=03:55.272 : Binance 65 239,49$ vs oracle t=03:55.000 : 65 189,72$ (+49,77$) ; t=00:04.299 : Binance 65 242,00$ vs oracle t=00:04.000 : 65 189,92$ (+52,08$).
- **A5.** L'écart est un décalage de NIVEAU stable, pas un retard : les deux courbes montent et descendent ensemble à la seconde près.
- **A6.** Spécifique : la stabilité du décalage (±9$ autour de +49$) est constatée sur 300 s seulement.


### A-G5 — FLUX DIRECTIONNEL 30 s, VUE 300 s

- **A1.** X : mm:ss.mmm ; Y : volume $ signé [−4 000 ; +2 000], baissier en négatif ; 1 679 trades.
- **A2.** Le flux dominant précède-t-il ou suit-il le prix ?
- **A3.** Je lis les extrema étiquetés du flux glissant 30 s.
- **A4.** t=00:00.238 : −91,72$ ; t=01:11.707 : −994,64$ (pic baissier) ; t=04:45.327 : +1 158,27$ (pic haussier) ; t=04:54.001 : +13,55$ ; plancher visuel proche de −4 000$ autour de 02:20–02:30.
- **A5.** Le flux est majoritairement baissier pendant toute la fenêtre, avec un pic haussier isolé en toute fin (04:45) quand le NO cote déjà 0,99$.
- **A6.** Spécifique : le pic haussier final de +1 158,27$ est un achat de YES à prix résiduel — comportement de loterie propre à la fin de cette session.


### A-G6 — FLUX DIRECTIONNEL, ZOOMS 15 s

- **A1.** Mêmes axes, fenêtres 00:55–01:10, 03:41–03:56, 00:04–00:19.
- **A2.** Le flux unitaire est-il assez granulaire pour être un signal exploitable ?
- **A3.** Je lis chaque impulsion étiquetée.
- **A4.** t=01:01.079 : −88,80$ ; t=01:01.183 : −91,20$ ; t=01:01.198 : +100,00$ ; t=03:48.015 : −598,54$ ; t=00:09.276 : −76,80$.
- **A5.** Des impulsions opposées de ±90–100$ se succèdent à 15 ms d'écart (01:01.183 → 01:01.198) : le flux instantané est bruité, seule l'agrégation 30 s est directionnelle.
- **A6.** Spécifique : la salve baissière de 03:48 (−598,54$) coïncide avec le rebond du YES de A-G2 — séquence propre à la session.


---

## 4. PARTIE B — LES 4 FICHIERS CSV

### B-F1 — spot.csv (Binance BTC/USDT direct)

- **B1.** Colonnes : t_ms, price ; 9 861 lignes ; période 00:00.731 → 04:59.804 ; fréquence irrégulière, salves de ticks jusqu'à >50 par seconde.
- **B2.** Apport : la granularité tick-par-tick que le graphe A-G3 agrège visuellement.
- **B3.** Calculs : min/max/dernier sur `price` ; comptage `price ≤ strike` ; basis = price − dernier tick oracle ≤ t_ms (jointure asof sur les 9 861 lignes).
- **B4.**


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Nombre de ticks | 9 861 | t_ms | 9 861
| Minimum | 65 208,01$ à 02:19.278 | price, t_ms | 9 861
| Maximum | 65 255,08$ à 00:00.731 | price, t_ms | 9 861
| Dernier tick | 65 223,48$ à 04:59.804 | price, t_ms | 1
| Ticks ≤ strike (65 203,36$) | **0 / 9 861 (0,0%)** | price | 9 861
| Basis moyen (spot − oracle) | **+48,76$** | price + oracle.price | 9 861
| Basis min | +37,89$ à 01:02.830 | idem | 9 861
| Basis max | +56,64$ à 01:44.626 | idem | 9 861


- **B5.** Le spot Binance n'a JAMAIS touché le strike : quiconque lisait Binance voyait un marché « UP » toute la session, alors que la résolution a donné DOWN.
- **B6.** Anomalies : salves de dizaines de ticks au même t_ms (ex. t=00:00.940–953) — agrégation de trades au même horodatage, bruit sans signal directionnel.


### B-F2 — oracle.csv (Chainlink BTC/USD)

- **B1.** Colonnes : t_ms, price, ts_src ; 287 lignes ; période 00:00.000 → 04:58.000 ; cadence ~1 tick/s.
- **B2.** Apport : la variable de résolution elle-même, avec le strike exact (premier tick).
- **B3.** Calculs : déficit = price − 65 203,3598$ par tick ; gaps inter-ticks ; comptages de persistance ; premier signal « 2 ticks consécutifs ≤ −10$ ».
- **B4.**


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Strike (tick t=00:00.000) | 65 203,36$ | price | 1
| ts_src = payload | 287/287 (fallback 0,0%) | ts_src | 287
| Gap médian / max entre ticks | 1 000 ms / 2 000 ms | t_ms | 286
| Max après t0 | 65 203,36$ à 00:01.000 (déficit −0,00$) | price | 286
| Minimum | 65 156,19$ à 02:21.000 (déficit −47,17$) | price | 287
| Dernier tick | 65 172,65$ à 04:58.000 (déficit −30,71$) | price | 1
| Ticks à déficit ≤ −10$ / ≤ −20$ / ≤ −30$ | **284 / 210 / 102** sur 287 | price | 287
| Premier signal 2 ticks consécutifs ≤ −10$ | **t=00:04.000** (−13,44$ à 00:03.000 et 00:04.000) | price, t_ms | 287


- **B5.** L'issue DOWN était portée par l'oracle dès 00:02.000 (−4,80$) et verrouillée statistiquement dès 00:04.000 : 284 des 287 ticks de la session sont restés ≤ −10$ sous le strike.
- **B6.** Anomalies : secondes manquantes (00:06, 00:27, 00:51...) — gaps de 2 000 ms, 13 occurrences, cohérents avec un heartbeat oracle, pas un signal.


### B-F3 — bbo.csv (carnet Polymarket YES/NO)

- **B1.** Colonnes : t_ms, yes_bid, yes_ask, no_bid, no_ask ; 636 lignes ; période 00:00.215 → 04:36.936 ; événementiel (mise à jour au changement).
- **B2.** Apport : les prix exécutables exacts à chaque instant, absents du PDF en continu.
- **B3.** Calculs : spread YES = yes_ask − yes_bid sur les lignes complètes ; carnets croisés stricts (bid > ask) ; quote prévalant à un t donné (dernier événement ≤ t).
- **B4.**


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Événements BBO | 636 | t_ms | 636
| Première quote | YES 0,45/0,46 à 00:00.215 | yes_bid, yes_ask | 1
| Dernière quote | YES ask 0,01 / NO bid 0,99 à 04:36.936 | yes_ask, no_bid | 1
| Spread YES moyen | 0,0161$ | yes_bid, yes_ask | 632
| Spread YES max | 0,07$ | idem | 632
| Carnets croisés stricts (bid>ask) | **3** (00:12.275, 03:48.742, 04:03.981) — les « 32 incohérents » du PDF utilisent un critère plus large propre au graphe | 4 colonnes | 636
| Quote prévalant à 00:04.099 | YES 0,38/0,39 — **NO 0,61/0,62** (événement 00:03.341) | 4 colonnes | 1
| Tenue de cette quote | 00:03.341 → 00:05.751, soit **2 410 ms** | t_ms | 2


- **B5.** À 00:04.099, le NO s'achète à 0,62$ alors que l'oracle affiche −13,44$ de déficit depuis 2 ticks : le carnet price une probabilité DOWN de 62% quand la variable de résolution est déjà franchement du côté DOWN.
- **B6.** Anomalies : ligne 00:00.215 avec no_ask=0 et champs vides (initialisation) ; croisement 00:12.275 (yes_bid 0,31 > yes_ask 0,30) tenu 19 ms — bruit de séquencement, inexploitable après frais (voir C0).


### B-F4 — trades.csv (flux d'exécutions)

- **B1.** Colonnes : t_ms, usd, direction (+1 haussier / −1 baissier) ; 1 679 lignes ; période 00:00.238 → ~04:59 ; événementiel.
- **B2.** Apport : la profondeur RÉELLEMENT consommée — seule pièce qui borne le slippage, le carnet ne donnant pas les tailles.
- **B3.** Calculs : sommes par direction ; distribution des tailles (tri croissant, médiane, p90) ; trades ≥ 200$.
- **B4.**


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Nombre de trades | 1 679 | t_ms | 1 679
| Volume total | 28 030,62$ | usd | 1 679
| Volume baissier | 20 931,36$ (**74,7%**), 1 028 trades | usd, direction | 1 679
| Volume haussier | 7 099,26$, 651 trades | usd, direction | 1 679
| Flux net | −13 832,10$ | usd, direction | 1 679
| Taille médiane / p90 | 4,45$ / 38,40$ | usd | 1 679
| Plus gros trade | 1 158,27$ haussier à 04:45.327 | usd, t_ms | 1
| Trades ≥ 200$ | 10, dont 9 baissiers (max baissier 994,64$ à 01:11.707) | usd, direction | 10


- **B5.** Le marché a exécuté 74,7% de volume baissier mais le prix n'a convergé que progressivement : la liquidité YES a absorbé 20 931$ de ventes sur 300 s — il y avait de la contrepartie pour un acheteur de NO.
- **B6.** Anomalies : trades de poussière (0,0022$ à 02:12.029, 0,0032$ à 01:54.103) et horodatages localement non monotones (ex. 00:04.263 après 00:04.305) — artefacts de séquencement, sans impact sur les agrégats.


---

## 5. PARTIE C — STRATÉGIES ET SIMULATION

### C0 — CANDIDATES

Mes cinq observations les plus fortes, toutes tracées ci-dessus :

- **OBS-1** : l'oracle est ≤ −10$ sous le strike sur 284/287 ticks, dès 00:04.000 (B5-F2).
- **OBS-2** : le spot Binance n'est jamais passé sous le strike (0/9 861), basis moyen +48,76$ (B5-F1, A5-G3).
- **OBS-3** : le NO s'achetait 0,62$ à 00:04.099 malgré OBS-1 (B5-F3).
- **OBS-4** : 74,7% du volume est baissier, le prix converge en 277 s, pas en 5 s (B5-F4, A5-G1).
- **OBS-5** : la quote prévalant au signal a tenu 2 410 ms, contre une boucle d'exécution de 130 ms (B5-F3).


**Latences de boucle** (uniquement le tableau fourni) :

- Boucle oracle : lecture Chainlink on-chain 68 ms + décision 0 + envoi CLOB 31 ms + confirmation 31 ms = **130 ms**.
- Boucle carnet (arbitrage/MM) : lecture CLOB 31 + envoi 31 + confirmation 31 = **93 ms**.
- Boucle Binance : lecture 219 ms + envoi 31 + confirmation 31 = **281 ms**, signal déjà âgé de ~219 ms à la décision.


**Candidates** (familles distinctes par mécanisme) :

1. **ORACLE-DEFICIT HOLD** — mécanisme : divergence variable-de-résolution vs prix du marché. Acheter le côté oracle (NO ici) quand 2 ticks consécutifs affichent un déficit ≤ −10$ et que le prix du côté oracle est ≤ 0,70$ ; porter jusqu'à résolution. Règles issues d'OBS-1, OBS-3, OBS-5. Conformité P0 : ordre limit marketable, tick 0,01 ✓, frais taker 0,07 intégrés ✓. Boucle 130 ms vs durée de vie du signal 296 000 ms (OBS-1) → **faisable**.
2. **ARB CARNET CROISÉ** — mécanisme : incohérence interne du carnet (acheter YES ask + NO ask quand la somme `< 1). 3 occurrences strictes (B4-F3), edge brut 0,01$/paire. Frais P0-1 sur les deux jambes à 00:12.275 : 0,07×0,30×0,70 + 0,07×0,69×0,31 = 0,0297$/paire >` 0,01$ brut → **P&L net négatif par construction, éliminée (règle P0-1)**. Faisabilité secondaire : la fenêtre de 19 ms est en outre inférieure à la boucle de 93 ms → doublement morte.
3. **SUIVI DE FLUX 30 s** — mécanisme : momentum du flux directionnel (OBS-4). Acheter NO quand le flux baissier 30 s dépasse −900$ (pic −994,64$ à 01:11.707, A4-G5). Boucle 93 ms vs signal 30 000 ms → faisable. Exécution NO ask 0,80$ à 01:11.8 (+1 tick → 0,81$) : 6,09 parts, coût 4,9329$, frais 0,0656$, payout 6,09$ → **net +1,0915$**. Conforme P0.
4. **MARKET MAKING MAKER** (frais 0, rebate 20%) — mécanisme : capture du spread moyen 0,0161$. Boucle faisable, mais les tailles au carnet et la file d'attente sont absentes des données : probabilité de fill = DONNÉE NON DISPONIBLE, et l'inventaire YES pris pendant un flux baissier à 74,7% converge vers 0$ → non simulable honnêtement, **écartée pour insimulabilité** (pas pour infaisabilité).


**CRITÈRE DE SÉLECTION : maximiser le P&L net réel simulé sur la session (après boucle de latence, slippage +1 tick, frais P0-1), sous conformité P0 stricte et faisabilité structurelle.** Ce critère est posé avant lecture du tableau.

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----|-----
| ORACLE-DEFICIT HOLD | OBS-1,3,5 | Oui | 1 | +3,0628$ | **+2,7340$** | **RETENUE**
| ARB CARNET CROISÉ | B6-F3 | Non (frais > edge) | 3 max | +0,01$/paire | négatif | ÉLIMINÉE (P0-1)
| SUIVI DE FLUX 30 s | OBS-4 | Oui | 1 | +1,3125$ | +1,0915$ | Dominée
| MARKET MAKING | OBS-5 | Oui | n.d. | n.d. | DONNÉE NON DISPONIBLE | ÉCARTÉE


Décision : ORACLE-DEFICIT HOLD, +2,7340$ contre +1,0915$ pour la seule autre candidate simulable.

### C1 — STRATÉGIE RETENUE

```plaintext
# ORACLE-DEFICIT HOLD — pseudo-code exécutable
STRIKE      = premier tick oracle <= t0                      # B4-F2 : 65 203,36$
SEUIL       = -10,00$                                        # OBS-1 : 284/287 ticks <= -10$
CONFIRM     = 2 ticks oracle consécutifs                     # B4-F2 : signal à 00:04.000
PRIX_MAX    = 0,70$                                          # OBS-3 [FRAGILE - 1 SOURCE]
SLIP        = +1 tick (0,01$)                                # règle r3, profondeur non observable
CASH        = 5,00$

à chaque tick oracle o(t):                                   # lecture âgée de 68 ms (tableau latences)
  deficit = o.price - STRIKE
  si deficit <= SEUIL sur CONFIRM ticks et aucune position:
    quote = BBO prévalant à (t + 68ms + 31ms)                # B4-F3
    si quote.no_ask <= PRIX_MAX:
      p = quote.no_ask + SLIP
      n = floor(100 * CASH / (p + 0,07*p*(1-p))) / 100       # frais P0-1, n >= 5 parts requis [HYPOTHÈSE n°1 :
                                                             # min_order_size en parts (place-orders.md) ;
                                                             # lecture "notionnel USDC" traitée en D9]
      BUY NO limit p, n, marketable                          # confirmation +31 ms
      HOLD jusqu'à résolution                                # P0-3 : résolution oracle Chainlink/TWAP
  symétrique pour deficit >= +10$ (BUY YES)                  # [HYPOTHÈSE n°2 : symétrie non testée ici]
```

### C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE** : cash initial 5,00$, aucune position, aucun ordre.

**RACE CONDITIONS** :

- (a) RÈGLE : tout signal survenant entre l'envoi et la confirmation d'un ordre est ignoré (position max = 1).
- (b) RÈGLE : aucune vente tant que le fill n'est pas confirmé — sans objet ici, la stratégie ne vend jamais (portage à résolution).
- (c) RÈGLE : deux signaux au même tick → priorité au timestamp oracle le plus ancien ; un seul ordre.
- (d) RÈGLE : fill partiel → conserver la part exécutée, annuler le reliquat, ne jamais repricer au-delà de +1 tick.


**JOURNAL DE TRADES** (r1 : boucle 130 ms appliquée ; r2 : lecture oracle âgée de 68 ms ; r3 : +1 tick ; r4 : frais P0-1 ; r5 : 4,9960$ engagés ≤ 5,00$ disponibles) :

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----
| 1 | 00:04.000 (oracle −13,44$) | 00:04.099 (arrivée CLOB), confirmé 00:04.130 | 0,63$ (ask 0,62$ + 1 tick) | 7,73 NO | 05:00.000 (résolution DOWN) | 1,00$ | 0,0773$ | 0,1261$ | **+2,7340$** | 0,0040$ puis **7,7340$**


Contrôle d'exécution : la quote 0,61/0,62 prévalant à 00:04.099 est née à 00:03.341 et a vécu jusqu'à 00:05.751 (2 410 ms) — la boucle de 130 ms n'a rencontré aucun re-pricing ; le +1 tick est une marge, pas une nécessité constatée. Comptabilité : 5,0000 − (7,73×0,63 = 4,8699) − 0,1261 = 0,0040$ ; à résolution +7,7300$ → **7,7340$**.

**RÉSULTATS** :

- Trades : 1 gagnant (+2,7340$), 0 perdant. Frais totaux : 0,1261$. Slippage total : 0,0773$.
- BÉNÉFICE NET : **+2,7340$** ; capital final **7,7340$** vs 5,00$ (**+54,68%**).
- Pire perte réalisée : 0,00$. Drawdown max (mark-to-bid, NO bid 0,61$ à l'entrée, jamais inférieur ensuite — B4-F3) : −0,2807$ latent (−5,61%).
- P&L THÉORIQUE sans r1–r4 : 8,06 parts à 0,62$ → capital final 8,0628$ (**+3,0628$**, +61,26%). **Coût du réalisme : 0,3288$**, soit 10,7% du profit théorique.


---

## 6. PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----
| D1 Latence de boucle | 130 ms n'altère pas le prix | Quote stable 2 410 ms autour du signal (B4-F3) ; prix identique à 00:03.969 et 00:04.099 | 0,00$ — prouvé | [SANS IMPACT]
| D2 Fraîcheur du signal | Lecture oracle âgée de 68 ms reste valide | Cadence oracle 1 000 ms (B4-F2) : staleness 6,8% d'un tick ; marge du signal 3,44$ sous le seuil | 0,00$ — prouvé | [SANS IMPACT]
| D3 Slippage / profondeur | Fill à ask+1 tick | Tailles au carnet absentes des données ; notre ordre (4,87$) ≈ médiane des trades (4,45$, B4-F4) | −0,0773$ intégré ; borne si fill à 0,64$ : −0,0745$ de plus | [DÉGRADE]
| D4 Frais complets | Taker 0,07, maker 0, gas 0 | P0-1 sourcé ; ordres signés off-chain sans gas (place-orders.md) | −0,1261$ intégré | [DÉGRADE]
| D5 Niveau disparu / fill partiel | Règles r3 et (d) | Quote tenue 2 410 ms ; si fill 50% (3,86 parts) : net +1,3652$, toujours positif | borné, jamais négatif | [DÉGRADE]
| D6 Défaillances techniques | Timeout RPC 108 ms → retard d'1 tick oracle max | Signal valide sur 284 ticks / 296 s (B4-F2) : un retard de 1–2 s change le prix d'entrée de ≤ 0,01$ (BBO 00:05.751 : 0,63$) | ≤ −0,08$ borné | [DÉGRADE]
| D7 Passage à l'échelle | Taille ≈ médiane du flux | p90 des trades = 38,40$ (B4-F4) : au-delà de ~40$ par entrée, l'ordre excède 90% des exécutions constatées, profondeur au-delà = DONNÉE NON DISPONIBLE | mécanisme borné à ~40$/fenêtre | [DÉGRADE]
| D8 Dépendance à la session | N'entre que si le déficit existe | Session calme (déficit > −10$) : la condition CONFIRM n'est jamais vraie → 0 trade, P&L 0,00$ | 0,00$ — prouvé par construction de la règle | [SANS IMPACT]
| D9 Conformité nouveau fonctionnement | Résolution = oracle Chainlink/TWAP | La stratégie parie SUR ce mécanisme ; TWAP 60s au close : ticks 03:58→04:58 tous ≤ −28$ (B-F2) → DOWN inchangé. MAIS ambiguïté P0-2 : si min_order_size = 5$ de notionnel (market-details.md), notre ordre de 4,87$ est rejeté → P&L 0,00$ (pas de perte) | 0,00$ ou −2,7340$ de manque à gagner | [DÉGRADE]


**VERDICT GLOBAL (règles mécaniques)** : P&L net réel positif (+2,7340$) ✓ ; zéro [DÉTRUIT] ✓ ; aucune violation P0 sous la lecture « parts » de place-orders.md, mais la contradiction documentaire P0-2 (D9) et trois facteurs bornés sur données absentes (D3, D5, D7) interdisent [FIABLE]. **Verdict : [FRAGILE]**. Étiquette : **[SESSION-SPÉCIFIQUE]** — le mécanisme (mispricing vs oracle) est structurellement plausible, mais son amplitude (−47,17$ de déficit, NO à 0,62$) n'est démontrée que sur cette unique session.

---

## 7. AUTO-CONTRÔLE FINAL

- Contrôle 1 (Phase 0 complète) → P0-1 à P0-4 sourcés avec URL ; changement récent identifié (TWAP Chainlink/RTDS, 04/08/2026) ; deux DONNÉE NON VÉRIFIABLE déclarées (délai de résolution 5min ; unité du min_order_size) avec hypothèses conservatrices. ✓
- Contrôle 2 (5 valeurs/graphe, 8/CSV) → A-G1 à A-G6 : 5 valeurs chacun ; B-F1 à B-F4 : 8 valeurs chacun en tableau F4. ✓
- Contrôle 3 (traçabilité des règles) → SEUIL/CONFIRM ← OBS-1 (B5-F2) ; PRIX_MAX ← OBS-3 (B5-F3, marqué [FRAGILE - 1 SOURCE]) ; SLIP ← B6-F3/B4-F4 ; frais ← P0-1. ✓
- Contrôle 4 (latence reconstituée) → 68 ms (Chainlink) + 31 ms (envoi CLOB) + 31 ms (confirmation) = 130 ms, appliquée à l'unique entrée du journal (00:04.000 → 00:04.099 → 00:04.130). ✓
- Contrôle 5 (cohérence comptable) → 5,0000 − 4,8699 − 0,1261 = 0,0040 ; 0,0040 + 7,7300 = 7,7340$ = capital final annoncé, au centime (au dixième de centime) près. ✓
- Contrôle 6 (D1–D9) → 9 facteurs traités ; les trois [SANS IMPACT] portent leur preuve chiffrée (2 410 ms vs 130 ms ; 6,8% de staleness avec marge 3,44$ ; 0 trade par construction en D8) ; verdict [FRAGILE] conforme aux règles mécaniques. ✓
- Contrôle 7 (unicité des chiffres) → 7,7340$ / +2,7340$ / +54,68% / 1 trade / 0,1261$ / [FRAGILE] / [SESSION-SPÉCIFIQUE] identiques en ouverture, corps et clôture. ✓


---

## 8. CLÔTURE

Vous avez sous les yeux la même asymétrie que moi : un marché qui se résout sur Chainlink pendant qu'une partie du flux regarde Binance. La stratégie l'exploite proprement sur cette session, mais une session ne fait pas une preuve — le verdict reste [FRAGILE] tant que la réplication et la levée de l'ambiguïté sur la taille minimale d'ordre ne sont pas faites.

---

## Source : 287_ORACLE-DEFICIT_MOMENTUM.md

# SPEECH — AUDIT COMPLET DE STRATÉGIE, SESSION BTC 5 MIN DU 7 AOÛT, 9:00–9:05 ET

## 0. SYNTHÈSE EXÉCUTIVE

Je vous livre le verdict d'entrée, chaque chiffre est repris à l'identique dans le corps.

- Stratégie retenue : **ORACLE-DEFICIT MOMENTUM** — achat NO sur déficit oracle persistant, portage jusqu'à résolution.
- Capital final : **6,5515 $** contre 5,00 $ de départ, soit **+31,03 % net réel** (après latences, slippage +1 tick, frais taker 0,0860 $).
- Nombre de trades : **1** — 1 gagnant, 0 perdant.
- Changement Polymarket le plus impactant : la résolution des marchés crypto 5 min est passée d'un prix instantané à un **TWAP Chainlink de 30 secondes** (docs.polymarket.com/market-data/chainlink-twap, lancement RTDS 4 août 2026) — sur cette session, il transforme un déficit final de −0,86 $ (quasi-retournement) en −17,66 $ (DOWN sécurisé).
- VERDICT DE FIABILITÉ : **[FRAGILE]**.
- Étiquette : **[SESSION-SPÉCIFIQUE]**.


---

## 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET (VÉRIFIÉ LE 09/08/2026)

Je n'ai retenu aucune connaissance interne : chaque règle ci-dessous vient d'une source consultée aujourd'hui.

| Règle | Valeur en vigueur | Source (URL) | Date consultation | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × 0,07 × p × (1−p), en USDC, arrondi 5 décimales, min 0,00001 | docs.polymarket.com/polymarket-learn/trading/fees | 09/08/2026 | Non signalé
| P0-1 Frais maker | 0 (rebate 20 % catégorie crypto) | idem | 09/08/2026 | Non
| P0-1 Dépôt/retrait/gas | 0 frais Polymarket ; ordres CLOB relayés (gasless) ; gas de redemption : DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : borné [0 ; 0,02 $], traité à 0 car redemption relayée | idem | 09/08/2026 | Non
| P0-2 Tick size | 0,01 $ sur ce marché — vérifié empiriquement : 787/787 cotations de bbo.csv à 2 décimales | bbo.csv + docs.polymarket.com/api-reference/rate-limits (endpoint "Market tick size") | 09/08/2026 | Non
| P0-2 Taille min. d'ordre | DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : 1,00 $ minimum ; mes ordres font ~5 $ donc conformes dans tous les cas | — | — | —
| P0-2 Priorité de matching | DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : price-time, aucune stratégie ne repose sur la priorité de file | — | — | —
| P0-3 Résolution 5 min | **TWAP Chainlink 30 s** (60 s pour 15 min/4 h), via Chainlink Data Streams ; RTDS lancé le 4 août 2026 ; feeds en soak test jusqu'au 4 août | docs.polymarket.com/market-data/chainlink-twap | 09/08/2026 | **OUI — c'est LE changement**
| P0-4 Rate limits IP | CLOB général 9 000 req/10 s ; /book 1 500 req/10 s ; POST /order burst 5 000 req/10 s | docs.polymarket.com/api-reference/rate-limits | 09/08/2026 | Non
| P0-4 Limits par signer | Token bucket : tier Standard 40 ordres/s, burst 60 ; mode warning depuis le 24 juillet 2026, enforcement annoncé ~2 semaines après | docs.polymarket.com/api-reference/trading-rate-limits | 09/08/2026 | **OUI (second changement)**


**Ce qui a changé et ce que cela implique pour une session de 5 minutes** : l'ancien mécanisme résolvait sur un prix instantané ; le nouveau résout sur la moyenne pondérée des 30 dernières secondes du flux Chainlink. Sur cette session, dernier tick oracle = 65 243,00 $ contre strike 65 243,87 $ (écart −0,86 $, oracle.csv ligne 283) : une résolution au dernier tick se jouait à moins d'un dollar. Le TWAP des 30 dernières secondes vaut 65 226,20 $ (moyenne des 29 ticks t ≥ 270 000 ms), soit −17,66 $ sous le strike : le nouveau mécanisme rend le DOWN 20 fois plus robuste. Toute stratégie de "sniping du dernier tick" est morte ; toute stratégie de projection de fenêtre 30 s est née. La définition exacte du strike d'ouverture sous le nouveau régime (tick vs TWAP) est DONNÉE NON VÉRIFIABLE — hypothèse conservatrice appliquée : strike = dernier tick ≤ t0, cohérent avec le PDF (65 244 $).

## 2. CADRE D'ANALYSE

Je lis chaque pièce sans stratégie préconçue, j'extrais des faits chiffrés reproductibles, puis je ne formule des règles qu'à partir de ces faits. Toute latence vient exclusivement du tableau fourni (CLOB 31 ms, Chainlink 68 ms, Polygon RPC 108 ms, Binance 219 ms, Telegram 67 ms). Le critère final porte sur le P&L net réel, jamais brut.

---

## 3. PARTIE A — LES 6 GRAPHES

### A-G1 — CARNET YES/NO COMPLET, VUE 300 s

- A1. Axes : temps mm:ss.mmm (0→300 s) ; prix $ (0→1) ; 787 événements BBO, 35 incohérents marqués ; résultat ▼ DOWN.
- A2. Question : à quelle vitesse le carnet a-t-il pricé le DOWN ?
- A3. Lecture des mids YES/NO et de la probabilité consensus aux extrema étiquetés.
- A4. Valeurs : t=00:00.248 → YES 0,47 $ ; t=00:03.526 → YES 0,58 $ (pic haussier initial) ; t=04:52.900 → YES 0,01 $ ; t=04:59.407 → YES 0,02 $ ; prob consensus NO : 0,53 → 0,41 → 0,99 → 0,98 aux mêmes timestamps.
- A5. Observation brute : le carnet part à 47/53, monte à 58 % YES à 3,5 s, puis glisse en ~2 minutes vers NO ≥ 0,95 — la conviction DOWN se construit progressivement, pas en un saut.
- A6. Spécifique session : l'ouverture à 0,47 (et non 0,50) et le pic YES 0,58 sont propres à ce flux d'ordres initial.


### A-G2 — CARNET, ZOOMS 15 s (00:08–00:23 ; 02:33–02:48 ; 04:04–04:19)

- A1. Mêmes axes, fenêtres 15 s.
- A2. Question : combien de temps un niveau BBO survit-il (durée de vie du signal) ?
- A3. Comptage des ticks étiquetés successifs.
- A4. Valeurs : t=00:09.141→00:09.458, YES bid 0,51→0,58 (+0,07 en 317 ms, 12 mises à jour) ; t=00:16.270→00:16.363, YES 0,58→0,49 (−0,09 en 93 ms) ; t=02:40.770→02:42.920, YES ask 0,02→0,07 (micro-doute) ; t=04:15.253, NO bid 0,94 (creux local) ; t=04:16.495, retour NO 0,98.
- A5. Observation brute : les rafales font 5 à 15 mises à jour en <500 ms ; entre rafales, le BBO reste stable plusieurs secondes (ex. 0,74 NO ask stable de 00:59.571 à 01:03.310, bbo.csv lignes 191–194).
- A6. Spécifique session : le whipsaw 00:09–00:17 (aller-retour de 9 ticks en 8 s) reflète ce flux précis.


### A-G3 — SPOT BINANCE vs ORACLE CHAINLINK, VUE 300 s

- A1. Axes : temps ; prix BTC $ (65 140–65 300) ; 16 512 ticks Binance, 283 ticks Chainlink ; strike 65 244 $ ; fallback ts 0/283 (0,0 %).
- A2. Question : les deux flux racontent-ils le même prix ?
- A3. Lecture des extrema étiquetés des deux courbes.
- A4. Valeurs : t=00:00.042 → spot 65 292,31 $ ; t=00:09.827 → spot 65 300,00 $ (max) ; t=02:27.025 → spot 65 189,31 $ (min) ; t=04:59.914 → spot 65 291,13 $ ; oracle t=02:34.000 → 65 137,07 $ (min).
- A5. Observation brute : le spot Binance évolue ~50 $ AU-DESSUS de l'oracle sur toute la session — les deux courbes sont parallèles, jamais confondues.
- A6. Spécifique session : min spot 65 189,31 $ à 02:27.025 ; l'amplitude totale spot (110,69 $) est propre à cette fenêtre.


### A-G4 — SPOT vs ORACLE, ZOOMS 15 s + BASIS

- A1. Mêmes axes + sous-graphe basis = spot − oracle (0→60 $).
- A2. Question : le basis est-il stable (exploitable comme simple offset) ?
- A3. Lecture du sous-graphe basis dans les 3 fenêtres.
- A4. Valeurs : t=02:33.393 → spot 65 190,00 $ vs oracle 65 139,58 $ (basis +50,42 $) ; t=00:16.047 → spot 65 282,21 $ ; t=04:13.084 → spot 65 243,99 $ vs oracle 65 191,46 $ (basis +52,53 $) ; t=04:17.347 → spot 65 254,00 $ ; t=02:46.390 → spot 65 225,96 $.
- A5. Observation brute : le basis oscille dans une bande étroite autour de +50 $ sans jamais s'inverser — l'oracle n'est pas un Binance retardé, c'est un composite décalé.
- A6. Spécifique session : la valeur absolue du basis (+50 $) est propre à ce jour ; seule sa stabilité est un candidat de pattern.


### A-G5 — FLUX DIRECTIONNEL NORMALISÉ, VUE 300 s

- A1. Axes : temps ; volume $ fenêtre 30 s (−4 000→+3 000), baissier en négatif ; 1 833 trades.
- A2. Question : le flux agrégé anticipe-t-il ou suit-il l'oracle ?
- A3. Lecture des extrema du flux 30 s.
- A4. Valeurs : t=00:00.355 → −20,00 $ ; t=00:07.347 → −1 391,42 $ (premier bloc baissier massif) ; t=04:23.992 → +992,39 $ (contre-flux haussier tardif) ; t=04:59.557 → −251,94 $ ; le flux 30 s reste sous zéro sur l'essentiel de [00:30–04:00].
- A5. Observation brute : un trade de −1 391,42 $ tombe à 7,3 s alors que le carnet cote encore YES ~0,50 — un acteur a frappé fort avant que le carnet ne bouge.
- A6. Spécifique session : l'asymétrie massive (voir B-F4 : 70,2 % du volume en DOWN) appartient à cette session baissière.


### A-G6 — FLUX DIRECTIONNEL, ZOOMS 15 s

- A1. Mêmes axes, fenêtres 15 s.
- A2. Question : que fait le flux quand le prix est déjà extrême (NO ≥ 0,95) ?
- A3. Lecture des barres individuelles.
- A4. Valeurs : t=02:41.172 → −790,71 $ ; t=02:41.164 → −300,70 $ ; t=02:36.553 → +194,00 $ ; t=04:15.232 → −99,84 $ ; t=00:16.989 → −63,60 $.
- A5. Observation brute : même à NO = 0,97–0,98, des blocs de 300–790 $ continuent d'acheter DOWN — il reste des acheteurs de quasi-certitude à 2–3 % de rendement.
- A6. Spécifique session : les montants exacts de ces blocs sont non reproductibles.


---

## 4. PARTIE B — LES CSV

### B-F1 — oracle.csv (Chainlink BTC/USD)

- B1. Colonnes : t_ms, price, ts_src ; 283 lignes ; 0→298 000 ms ; cadence médiane 1 000 ms.
- B2. Apport : la seule série résolutive — c'est elle, pas Binance, qui décide UP/DOWN.
- B3. Calculs : strike = price(t=0) ; comptage ticks < strike ; moyenne des ticks t ≥ 270 000 (TWAP 30 s) ; gaps successifs.
- B4. Résultats :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Strike (t=0) | 65 243,87 $ | t_ms, price | 1
| Dernier tick (t=298 000) | 65 243,00 $ | t_ms, price | 1
| Écart final dernier tick | −0,86 $ | price | 2
| TWAP 30 s finales | 65 226,20 $ (−17,66 $) | t_ms, price | 29
| Ticks sous le strike | 270/283 (95,4 %) | price | 283
| Dernier tick ≥ strike | t=35 000 (65 244,29 $) | t_ms, price | 283
| Minimum | 65 137,07 $ à t=154 000 | t_ms, price | 283
| Gap max | 8 000 ms (t=27 000→35 000) | t_ms | 282
| ts fallback | 0/283 (0,0 %) | ts_src | 283


- B5. Observation brute : après t=00:35.000, l'oracle ne repasse plus JAMAIS au-dessus du strike — le DOWN est acquis côté oracle à 12 % de la session.
- B6. Anomalies : le gap de 8 s (t=27–35 s) est le seul trou majeur ; il tombe précisément avant le basculement définitif — bruit de flux probable, mais un lecteur on-chain y est aveugle 8 s.


### B-F2 — spot.csv (Binance BTC/USDT)

- B1. Colonnes : t_ms, price ; 16 512 lignes ; 42→299 914 ms ; rafales de ticks intra-milliseconde.
- B2. Apport : la granularité fine que l'oracle n'a pas ; permet de mesurer le basis.
- B3. Calculs : min/max ; basis = spot(t) − oracle(t) échantillonné chaque seconde (298 points).
- B4. Résultats :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Ouverture | 65 292,31 $ (t=42) | t_ms, price | 1
| Clôture | 65 291,13 $ (t=299 914) | t_ms, price | 1
| Delta session spot | −1,18 $ | price | 2
| Max | 65 300,00 $ à t=9 827 | t_ms, price | 16 512
| Min | 65 189,31 $ à t=147 025 | t_ms, price | 16 512
| Basis moyen | +50,13 $ | 2 fichiers | 298 pts
| Basis min / max | +30,43 $ / +59,08 $ | 2 fichiers | 298 pts
| Basis médian | +50,32 $ | 2 fichiers | 298 pts


- B5. Observation brute : jugé sur son propre référentiel, le spot Binance finit quasiment flat (−1,18 $) — c'est l'oracle, pas Binance, qui fait un DOWN net ; parier depuis Binance seul était un pari sur un pile-ou-face de 1 $.
- B6. Anomalie : rafales de dizaines de ticks au même t_ms (ex. 108 lignes à t=1 015) — agrégation d'un burst du carnet Binance ; bruit, pas signal.


### B-F3 — bbo.csv (carnet Polymarket)

- B1. Colonnes : t_ms, yes_bid, yes_ask, no_bid, no_ask ; 787 lignes ; 248→~297 000 ms ; événementiel.
- B2. Apport : les prix réellement exécutables, avec les timestamps exacts — c'est ici que se mesure le slippage.
- B3. Calculs : spreads YES ; livres croisés stricts (ya+na<0,999 ou yb+nb>1,001) ; trajectoire NO ask ; gaps.
- B4. Résultats :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| NO ask à t=40 000 | 0,59 $ | t_ms, no_ask | 1
| NO ask à t=60 000 | 0,73 $ | t_ms, no_ask | 1
| NO ask à t=80 000 | 0,85 $ | t_ms, no_ask | 1
| NO ask à t=100 000 | 0,93 $ | t_ms, no_ask | 1
| Spread YES médian / moyen / max | 0,02 / 0,021 / 0,10 | yes_bid, yes_ask | 787
| Livres croisés stricts | 5 (dont t=232 764 : ya 0,04 + na 0,95 = 0,99) | 4 colonnes | 787
| Durée de vie du croisement t=232 764 | 10 ms (correction à t=232 774) | t_ms | 2
| Gap BBO max | 14 434 ms (t=122 111→136 545) | t_ms | 786
| NO ask stable 0,74 | de t=59 571 à t=63 310 (3 739 ms) | t_ms, no_ask | 4


- B5. Observation brute : quand l'oracle est déjà à −28 $ sous le strike (t=45 000), le carnet cote encore NO ask 0,59–0,60 — le carnet retarde sur l'information résolutive de plusieurs dizaines de secondes.
- B6. Anomalies : les 35 "incohérents" du PDF descendent à 5 sous ma définition stricte à ±0,001 ; tous durent ≤10 ms (ex. t=255 638, t=267 138, t=292 900) — micro-états transitoires, pas des fenêtres d'arbitrage.


### B-F4 — trades.csv (flux exécuté)
### B-F4 — trades.csv (flux exécuté)

- B1. Colonnes : t_ms, usd, direction ; 1 833 lignes ; 250→~299 600 ms.
- B2. Apport : volumes réels par sens — la profondeur implicite que bbo.csv ne donne pas.
- B3. Calculs : volumes agrégés par sens, par tranche de 60 s ; distribution des tailles.
- B4. Résultats :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----
| Volume total | 35 376,73 $ | usd | 1 833
| Volume DOWN | 24 837,59 $ (70,2 %) | usd, direction | 1 127
| Volume UP | 10 539,14 $ | usd, direction | 706
| Flux net | −14 298,45 $ | usd, direction | 1 833
| Plus gros trade | −1 391,42 $ à t=7 347 | t_ms, usd | 1
| Taille médiane | 4,70 $ | usd | 1 833
| Flux net [0–60 s] | −3 037,10 $ | t_ms, usd, direction | 430
| Flux net [60–120 s] | −4 111,57 $ | t_ms, usd, direction | 421
| Flux net [240–300 s] | −1 573,80 $ | t_ms, usd, direction | 498


- B5. Observation brute : la taille médiane est 4,70 $ — un ordre de 5,00 $ est un trade MÉDIAN sur ce marché, pas une goutte négligeable mais pas un moteur de prix.
- B6. Anomalies : quelques timestamps localement non monotones (ex. ligne 41 : t=3 751 après t=3 766) — horodatage de confirmation vs matching ; sans impact sur des agrégats 30 s.


---

## 5. PARTIE C — STRATÉGIES ET SIMULATION

### C0 — CANDIDATES

Mes cinq observations les plus fortes :

- **OBS-1** : l'oracle est sous le strike 95,4 % du temps et définitivement après t=35 000 ms (B5-F1).
- **OBS-2** : le carnet retarde — NO ask 0,59 à t=40 000 quand le déficit oracle est déjà −28 $ (B5-F3).
- **OBS-3** : la résolution est désormais un TWAP 30 s Chainlink ; le déficit TWAP final vaut −17,66 $ contre −0,86 $ au dernier tick (P0-3, B4-F1).
- **OBS-4** : les livres croisés durent ≤10 ms (B6-F3).
- **OBS-5** : le flux est unidirectionnel, net −14 298,45 $, et le mid YES dérive de 0,50 à 0,01 (B4-F4, A5-G1).


**FILTRE DE FAISABILITÉ STRUCTURELLE** (latences du tableau uniquement) :

| Candidate | Boucle complète | Durée de vie du signal | Verdict structurel
|-----|-----|-----|-----
| S1 Oracle-Deficit Momentum | lecture Chainlink 68 + envoi CLOB 31 + confirmation 31 = **130 ms** (+ âge intrinsèque ≤1 000 ms, cadence oracle) | NO ask 0,74 stable 3 739 ms (B4-F3) | FAISABLE (3 739 ≫ 1 130)
| S2 Market-making maker | lecture BBO 31 + envoi 31 = 62 ms | plusieurs secondes | Faisable en latence, mais fills non modélisables (aucune profondeur dans bbo.csv) et OBS-5 garantit l'anti-sélection : le mid dérive de −0,49, toute quote YES-bid résiduelle est ramassée. Règle r3 (donnée manquante = hypothèse conservatrice) → **ÉLIMINÉE**
| S3 Arbitrage livres croisés | détection BBO 31 + envoi 31 = 62 ms | ≤10 ms (OBS-4) | **[STRUCTURELLEMENT IMPOSSIBLE]** — boucle 6× plus lente que le signal
| S4 TWAP Endgame Lock | 130 ms (même chemin que S1) | dernier creux NO ask 0,88 dure ~2 s (t=270 064→~272 000) | FAISABLE


**CRITÈRE DE SÉLECTION : P&L net réel maximal (après latences, slippage, frais P0-1), à conformité P0 totale — annoncé avant tout calcul comparatif.**

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----
| S1 Oracle-Deficit Momentum | OBS-1, OBS-2, OBS-3 | Oui | 1 | +1,7550 $ | **+1,5515 $** | **RETENUE**
| S2 Market-making | OBS-5 (contre elle) | Oui | n/a | non calculable | non calculable | Éliminée (r3)
| S3 Arb livres croisés | OBS-4 | Oui | 0 | +0,01 $/paire théorique | 0 (inexécutable) | Éliminée (filtre)
| S4 TWAP Endgame Lock | OBS-3 | Oui | 1 | +0,6742 $ | +0,5745 $ | Battue par S1


Décision : S1 gagne sur le critère (+1,5515 $ contre +0,5745 $ pour S4, seule autre candidate exécutable).

### C1 — STRATÉGIE RETENUE : ORACLE-DEFICIT MOMENTUM

```plaintext
# Capital initial : 5,00 $ (contrainte dure)
# Flux d'entrée : Chainlink BTC/USD on-chain (latence 68 ms, cadence ~1 s — B1-F1)
# Marché : BTC Up/Down 5 min, tick 0,01 (P0-2), taker fee 0,07 (P0-1)

strike = premier_tick_oracle(t <= t0)                # 65 243,87 $ (B4-F1) [HYPOTHÈSE n°1 : strike = dernier tick <= t0, non re-vérifiable sous régime TWAP]
MARGIN = 15 $ ; PERSIST = 20 s                       # calibrés sur OBS-1/OBS-2 [FRAGILE - 1 SOURCE : une seule session]
SI oracle < strike - MARGIN sans interruption pendant PERSIST :
    t_sig = maintenant                               # t_sig = 61 000 ms sur cette session
    ACHETER NO au marché, taille = tout le cash      # exécuté à t_sig + 130 ms (boucle C0)
    prix exécuté = no_ask(t_sig + 130 ms) + 1 tick   # règle slippage r3, énoncée à l'avance
PORTER jusqu'à résolution (TWAP 30 s, P0-3) ; redemption à 1,00 $/part
AUCUNE sortie anticipée ; AUCUN ré-achat (1 seul trade par session)
```

Chaque condition remonte au corps : MARGIN/PERSIST ← OBS-1 (bascule définitive à t=35 s) et OBS-2 (le carnet retarde) ; portage ← OBS-3 (le TWAP fiabilise l'issue) ; taille ← B5-F4 (5 $ = trade médian).

### C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE** : cash initial 5,0000 $, position 0 part, aucun ordre en cours.

**RACE CONDITIONS** :

- (a) RÈGLE : un seul ordre vivant à la fois — tout signal reçu pendant qu'un ordre est en cours est ignoré, jamais mis en file.
- (b) RÈGLE : aucune vente/action sur une position non confirmée — la confirmation CLOB (31 ms) est bloquante.
- (c) RÈGLE : deux signaux simultanés — priorité absolue au signal oracle (source résolutive), l'autre est jeté.
- (d) RÈGLE : fill partiel — conserver le rempli, DELETE /order immédiat sur le reste (31 ms), re-cotation interdite au-delà de +1 tick du prix initial.


**JOURNAL DE TRADES** (r1 : décalage 130 ms appliqué ; r2 : l'âge oracle ≤1 s est inclus dans la définition du signal ; r3 : profondeur absente de bbo.csv → +1 tick systématique ; r4 : frais P0-1 ; r5 : cash vérifié) :

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----
| 1 | 00:61.000 | 00:61.130 | 0,75 $ (ask 0,74 + 1 tick) | 6,55 NO | résolution (t>300 s) | 1,00 $ | 0,01 $/part | 0,0860 $ | +1,5515 $ | 6,5515 $


Vérification comptable ligne à ligne : débit = 6,55 × 0,75 = 4,9125 $ ; frais = 6,55 × 0,07 × 0,75 × 0,25 = 0,0860 $ ; total 4,9985 $ ≤ 5,00 $ (r5 respectée) ; cash résiduel 0,0015 $ ; redemption 6,55 × 1,00 = 6,5500 $ ; capital final 6,5500 + 0,0015 = **6,5515 $**.

**RÉSULTATS** :

- Trades : 1 gagnant (+1,5515 $), 0 perdant.
- Frais totaux : 0,0860 $. Gas : 0 (ordres relayés, P0-1).
- BÉNÉFICE NET : +1,5515 $ ; capital final 6,5515 $ vs 5,00 $ (**+31,03 %**).
- Pire perte réalisée : 0 $. Drawdown max (latent, mark-to-market au NO bid 0,73 juste après exécution) : 6,55 × 0,73 + 0,0015 = 4,7830 $, soit −0,2170 $ (−4,34 %).
- P&L THÉORIQUE en regard (sans r1–r4) : exécution instantanée à 0,74, sans frais → 6,75 parts, capital final 6,7550 $ (+35,10 %). **Coût du réalisme : 0,2035 $**, dont 0,0655 $ de slippage et 0,0860 $ de frais.


---

## 6. PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----
| D1 Latence de boucle | 130 ms n'altère pas le prix | NO ask 0,74 stable 3 739 ms autour de t_sig (B4-F3) | 0,00 $ — prix identique à t=61 000 et t=61 130 | [SANS IMPACT] (preuve : même cotation)
| D2 Fraîcheur du signal | tick oracle "récent" | cadence 1 000 ms + gap observé jusqu'à 8 000 ms (B4-F1) | signal potentiellement vieux de 8 s ; sur cette session, décalage d'entrée borné à [0 ; 8 s] soit ask ∈ [0,74 ; 0,77], P&L net ∈ [+1,37 ; +1,55 $] | [DÉGRADE]
| D3 Slippage/profondeur | +1 tick suffit | profondeur ABSENTE de bbo.csv ; trade médian 4,70 $ (B5-F4) fait de notre ordre un ordre normal | déjà facturé −0,0655 $ ; si 2 ticks : −0,1310 $, P&L reste +1,49 $ | [DÉGRADE]
| D4 Frais complets | fee = C × 0,07 × p(1−p), gas 0 | confirmé P0-1 (source datée) | −0,0860 $, déjà inclus | [SANS IMPACT] au-delà du montant facturé (preuve : formule exacte appliquée)
| D5 Niveau disparu/fill partiel | règle (d) borne le risque | rafales BBO <500 ms observées (A5-G2) | pire cas : fill 50 % → P&L +0,78 $ au lieu de +1,55 $ | [DÉGRADE]
| D6 Défaillances techniques | lecture Chainlink directe (68 ms), pas de RPC critique | Polygon RPC 108 ms non utilisé dans la boucle ; ts fallback 0/283 (B4-F1) | un timeout à t_sig décale l'entrée d'un cycle (1 s) : ask 0,74 inchangé (B4-F3) → 0,00 $ | [SANS IMPACT] (preuve : cotation stable sur le cycle suivant)
| D7 Passage à l'échelle | taille 5 $ | volume total 35 376,73 $, plus gros trade 1 391,42 $ (B4-F4) ; profondeur BBO non disponible | borne par la méthode des trades observés : au-delà de ~100–500 $ par entrée, consommation de plusieurs niveaux probable, rendement décroissant ; à 5 $ : nul | [SANS IMPACT] à 5 $ (preuve : 5 $ = taille médiane 4,70 $), [DÉGRADE] au-delà
| D8 Dépendance session | le déficit persistant prédit l'issue | démontré sur UNE session ; le spot a fini à −1,18 $ de son propre open (B4-F2) — une session en aller-retour complet ferait perdre jusqu'à 4,9985 $ (mise totale) ; sur session calme (déficit <15 $), zéro trade, zéro perte | perte max possible −4,9985 $ sur session en V | [DÉGRADE]
| D9 Conformité nouveau fonctionnement | résolution TWAP 30 s | P0-3 : c'est le mécanisme actuel ; la stratégie en BÉNÉFICIE (déficit TWAP −17,66 $ vs −0,86 $ au dernier tick, B4-F1) | +0 $ ici, robustesse accrue | [SANS IMPACT] (preuve : l'issue DOWN est confirmée sous les deux régimes, avec marge 20× sous TWAP)


**VERDICT GLOBAL (règles mécaniques)** : P&L net réel positif (+1,5515 $) ; zéro [DÉTRUIT] ; conformité P0 totale ; mais quatre [DÉGRADE] dont D8 — l'edge n'est démontré que sur une session, et une session en V coûterait la mise entière. Les règles n'imposent pas [NON FIABLE] ; l'exigence [FIABLE] échoue sur D8. Verdict : **[FRAGILE]**, étiquette **[SESSION-SPÉCIFIQUE]**.

---

## 7. AUTO-CONTRÔLE FINAL

1. Phase 0 complète → P0-1 à P0-4 sourcés (3 URL docs.polymarket.com, consultées 09/08/2026) ; changement récent identifié (TWAP 30 s) ; 3 données non vérifiables marquées avec hypothèses conservatrices.
2. Valeurs minimales → 5+ par graphe (A-G1 à A-G6 : 5, 5, 5, 5, 5, 5), 8+ par CSV (B4 : 9, 8, 9, 9).
3. Traçabilité des règles → MARGIN/PERSIST ← OBS-1/OBS-2 ; portage ← OBS-3 ; taille ← B5-F4 ; slippage ← B6-F3/r3.
4. Boucle de latence reconstituée → Chainlink 68 ms + envoi CLOB 31 ms + confirmation 31 ms = 130 ms, appliquée à l'unique entrée du journal (61 000 → 61 130 ms) ; âge intrinsèque oracle ≤1 000 ms traité en D2.
5. Cohérence comptable → 5,0000 − 4,9125 − 0,0860 = 0,0015 ; 0,0015 + 6,5500 = 6,5515 $ = capital final annoncé, au centime (au dixième de centime) près.
6. D1–D9 traités → 9/9 ; chaque [SANS IMPACT] porte sa preuve chiffrée ; verdict global conforme aux règles mécaniques.
7. Unicité des chiffres → 6,5515 $ / +31,03 % / 1 trade / 0,0860 $ / −17,66 $ identiques en ouverture, corps et clôture.


## 8. CLÔTURE

La stratégie retenue gagne sur cette session, et je vous ai montré exactement ce que le réalisme lui coûte.
Ce qu'elle ne prouve pas, c'est sa reproductibilité : une seule session baissière ne fait pas un edge.
Le verdict [FRAGILE] est la seule conclusion honnête que les pièces autorisent.


