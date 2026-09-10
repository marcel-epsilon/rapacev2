# 09 — SNIPER ORACLE

**Fichiers sources regroupés :** 295SNIPER-ORACLE_7sT-.md, 285SNIPE-ORACLE.md

---

## Source : 295SNIPER-ORACLE_7sT-.md

# SPEECH STRATÈGE-AUDITEUR — SESSION POLYMARKET BTC 5MIN DU 7 AOÛT, 9:40–9:45 ET

---

## 0. SYNTHÈSE EXÉCUTIVE

Je vous livre le verdict d'entrée, chaque chiffre est retraçable dans le corps.

- Stratégie retenue : **SNIPER-ORACLE 7sT-** (achat YES en fin de fenêtre quand l'oracle de résolution contredit le prix du carnet).
- Capital final : **23,14$** vs 5,00$, soit **+362,85%** net, APRÈS latences, slippage et frais taker 0,07 (P0-1).
- Trades : **1 trade, 1 gagnant, 0 perdant**, frais totaux 0,2576$.
- Changement Polymarket le plus impactant : le taux de frais taker des marchés crypto est passé de 0,06 (rapport de juillet 2026) à **0,07** (docs.polymarket.com/polymarket-learn/trading/fees, consultée le 09/08/2026).
- VERDICT DE FIABILITÉ : **[FRAGILE]**.
- Étiquette : **[SESSION-SPÉCIFIQUE]**.


---

## 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET

Voici l'état vérifié des règles en vigueur au 09/08/2026, chaque ligne sourcée.

| Règle | Valeur en vigueur | Source (URL) | Date de consultation | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × **0,07** × p × (1−p), en USDC, arrondi à 5 décimales, min facturé 0,00001 USDC | docs.polymarket.com/polymarket-learn/trading/fees | 09/08/2026 | **OUI** — coefficient rapporté à 0,06 en juillet 2026 (recherche web, source tierce), 0,07 dans la doc aujourd'hui
| P0-1 Frais maker | **0** — « Makers are never charged fees » | même page | 09/08/2026 | Non (rebate maker crypto : 20% des frais redistribués)
| P0-1 Gas / dépôt / retrait | 0$ (relayer, pas de frais Polymarket) | même page + docs.polymarket.com (Relayer `/submit`) | 09/08/2026 | Non
| P0-2 Types d'ordres | Limit (dont marketable limit) via `POST /order` ; annulations `DELETE /order`, `/cancel-all` | docs.polymarket.com/api-reference (Create Orders, Cancel) | 09/08/2026 | Non
| P0-2 Tick size / taille min | Par token via endpoint « Get tick size » ; valeur pour CE marché : **DONNÉE NON VÉRIFIABLE** — hypothèse conservatrice appliquée : tick 0,01$ (cohérent avec les 2 267 lignes BBO, toutes multiples de 0,01$), ordre min 1,00$ | docs.polymarket.com/api-reference/market-data/get-tick-size | 09/08/2026 | Non déterminable
| P0-3 Résolution générale | UMA Optimistic Oracle, bond ~750 pUSD, fenêtre de contestation 2 h | docs.polymarket.com/concepts/resolution | 09/08/2026 | Non
| P0-3 Résolution crypto 5 min | Page dédiée non récupérable → **DONNÉE NON VÉRIFIABLE** — hypothèse conservatrice appliquée : résolution automatique sur l'oracle Chainlink BTC/USD, strike = dernier tick ≤ t0, convention explicitement affichée dans le matériel fourni (PDF, Graphique 2 : « strike exact (dernier tick ≤ t0) », strike 65 251$, Résultat ▲ UP) | docs.polymarket.com/llms.txt (index) + PDF fourni | 09/08/2026 | **Probable** — c'est le point de changement le plus plausible ; traité comme risque bloquant en D9
| P0-4 Rate limits | CLOB `POST /order` : 5 000 req/10 s burst ; `/book`,`/price`,`/midpoint` : 1 500 req/10 s ; général CLOB 9 000 req/10 s ; throttling Cloudflare (délai, pas rejet) | docs.polymarket.com/api-reference/rate-limits | 09/08/2026 | Non


Contrainte dure retenue : 1 seul trade taker de ~5$ = **2 requêtes** (1 lecture + 1 ordre), très en-deçà de toutes les limites P0-4. Aucune stratégie ci-dessous ne viole une règle P0.

---

## 2. CADRE D'ANALYSE

Je lis d'abord chaque pièce isolément et j'en extrais des faits chiffrés horodatés ; je ne formule aucune règle avant la fin de la Partie B. Le fil conducteur de mesure : écart entre le référentiel de RÉSOLUTION (oracle Chainlink vs strike 65 251,74$) et le référentiel de PRIX du carnet. Toutes les latences viennent exclusivement du tableau fourni.

---

## 3. PARTIE A — LES 6 GRAPHES

### A-G1 — CARNET YES/NO COMPLET (BBO, vue 300 s + zooms)

- **A1.** X : temps mm:ss.mmm depuis l'ouverture (0 → 300 s) ; Y : prix en $ (0 → 1) ; 2 267 événements BBO ; 92 marqueurs « carnet croisé — suspects ».
- **A2.** Le prix du carnet converge-t-il vers l'issue réelle (UP) en fin de fenêtre ?
- **A3.** Lecture des annotations d'extrema et des séries YES/NO bid/ask/mid, croisée avec les zooms 15 s.
- **A4.** Valeurs extraites : t=00:00.247 : YES mid 0,46$ ; t=04:10.938 : YES mid 0,78$ ; t=04:31.324 : YES 0,52$ (rebond) ; t=04:58.494 : YES mid 0,57$ ; t=04:59.562 : YES 0,01$ / NO 0,99$.
- **A5.** Observations brutes : (i) le carnet finit à YES=0,01$ alors que le résultat affiché est ▲ UP — le consensus du carnet s'est trompé de 0,99$ sur l'issue ; (ii) au zoom 04:23–04:38, YES passe de 0,05$ (04:30.370) à 0,65$ (04:31.401) en 1 031 ms, soit un aller de +0,60$ ; (iii) 92 événements de carnet croisé signalés.
- **A6.** Spécifique à la session : l'aller-retour 0,05→0,65→0,01 en 29 s est un choc unique, non un motif répété (1 seule occurrence sur 300 s).


### A-G2 — SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT

- **A1.** X : temps ; Y : prix BTC (65 150–65 325$) ; 29 628 ticks Binance, 282 ticks Chainlink ; ligne strike 65 251$ ; sous-graphe basis = spot − oracle (0 → 60$+).
- **A2.** Les deux flux racontent-ils la même histoire par rapport au strike ?
- **A3.** Lecture des annotations spot et des ticks oracle 1 s, comparaison au strike.
- **A4.** Valeurs extraites : t=00:00.299 : spot 65 299,70$ ; t=00:18.032 : spot 65 321,77$ (max) ; t=02:49.892 : spot 65 184,00$ (min) ; t=04:59.683 : spot 65 308,35$ ; oracle t=04:59.000 : 65 261,61$ ; strike 65 251$ (affiché), 65 251,74$ (CSV).
- **A5.** Observations brutes : (i) le basis spot−oracle est massivement positif sur TOUTE la fenêtre (sous-graphe collé entre ~20 et ~60$) ; (ii) l'oracle finit AU-DESSUS du strike (+9,86$, B-F4) pendant que le carnet (A-G1) prix NO à 0,99$ ; (iii) le spot ouvre à 65 299,70$, soit 47,96$ au-dessus du strike oracle — deux référentiels d'ouverture incompatibles coexistent.
- **A6.** Spécifique : l'ampleur du basis (méd +46,62$, B-F4) est propre à cette capture ; je la traite comme propriété de session, pas comme constante universelle.


### A-G3 — FLUX DIRECTIONNEL NORMALISÉ (3 847 trades)

- **A1.** X : temps ; Y : volume $ sur fenêtre glissante 30 s, haussier positif / baissier négatif (−8 000 → +4 000$).
- **A2.** Qui domine le tape, et surtout : dans quel sens en toute fin de fenêtre ?
- **A3.** Lecture des extrema annotés et des zooms.
- **A4.** Valeurs extraites : t=00:01.705 : +23,00$ ; t=03:21.006 : trade baissier −1 143,95$ ; t=03:25.526 : +197,58$ ; t=04:58.475 : −931,00$ ; t=04:59.864 : −1 148,42$.
- **A5.** Observations brutes : (i) les deux plus gros prints de la session (−1 143,95$ et −1 148,42$) sont BAISSIERS, le second à 136 ms de la clôture ; (ii) le flux baissier domine visuellement dès 01:00 et s'intensifie dans les 30 dernières secondes.
- **A6.** Spécifique : la foule vend l'issue UP jusqu'à la dernière seconde d'une fenêtre qui résout UP — c'est l'anomalie centrale de cette session.


### A-G4 — IMBALANCE DIRECTIONNELLE

- **A1.** X : temps ; Y : Imbalance = Haussier/(Haussier+Baissier) ∈ [0 ; 1], ligne d'équilibre 0,5.
- **A2.** L'imbalance anticipe-t-elle les retournements du carnet ?
- **A3.** Lecture des annotations vue 300 s et zooms.
- **A4.** Valeurs extraites : t=00:00.426 : 1,00 ; t=03:03.362 : 0,11 (min) ; t=04:23.068 : 0,46 ; t=04:31.382 : 0,36 ; t=04:59.968 : 0,26.
- **A5.** Observations brutes : (i) l'imbalance reste sous 0,5 quasi continûment après la minute 1 ; (ii) à 04:31 (rebond YES de 0,05$ à 0,65$, A-G1), l'imbalance ne remonte qu'à 0,36 — le flux n'a PAS précédé le repricing, il l'a suivi.
- **A6.** Spécifique : imbalance finale 0,26 contre résolution UP — l'indicateur de flux est contrarien sur cette session, en 1 occurrence, donc non généralisable.


### A-G5 — SPREADS YES ET NO ABSOLUS

- **A1.** X : temps ; Y : spread ask−bid en $ (−0,1 → 0,6).
- **A2.** Quand la liquidité affichée se dégrade-t-elle ?
- **A3.** Lecture des extrema et des zooms 15 s.
- **A4.** Valeurs extraites : t=00:00.247 : 0,01$ ; t=03:20.592 : 0,14$ ; t=04:31.324 : 0,28$ ; t=04:57.983 : **0,57$** (max) ; t=04:58.933 : **−0,07$** (spread négatif = carnet croisé) ; t=04:59.562 : 0,00$.
- **A5.** Observations brutes : (i) spread médian de l'ordre de 0,01–0,02$ en régime calme, mais pics à 0,14$ / 0,28$ / 0,57$ à chaque choc ; (ii) un spread NÉGATIF de −0,07$ apparaît à 66 ms de la clôture.
- **A6.** Spécifique : la dislocation finale (0,57$ puis −0,07$) montre un carnet démantelé dans les 2 dernières secondes ; toute exécution après 04:58 est non fiable par construction.


### A-G6 — ÉCART INTER-CARNETS yes_mid − (1 − no_mid)

- **A1.** X : temps ; Y : écart en $ (−0,010 → +0,020) ; ligne de cohérence parfaite à 0.
- **A2.** Les carnets YES et NO offrent-ils un arbitrage exploitable ?
- **A3.** Lecture de l'enveloppe de l'écart et des 92 suspects (marqueurs creux).
- **A4.** Valeurs extraites : bornes d'axe −0,010$ / +0,020$ ; enveloppe de travail |écart| ≤ 0,02$ sur la vue 300 s ; 92 événements suspects ; 0/282 fallback de timestamp (0,0%) ; cohérence nominale 0.
- **A5.** Observations brutes : l'écart inter-carnets vit dans ±0,02$, soit 1 à 2 ticks — l'espérance brute d'un arbitrage YES/NO est bornée par 0,02$/paire AVANT frais.
- **A6.** Spécifique : les 92 suspects se concentrent aux mêmes horodatages que les chocs d'A-G5 ; ce sont des transitoires de mise à jour, pas des fenêtres persistantes (confirmé en B-F1 : durée 0 ms).


---

## 4. PARTIE B — LES FICHIERS CSV

### B-F1 — bbo.csv

- **B1.** Colonnes : t_ms, yes_bid, yes_ask, no_bid, no_ask ; 2 267 lignes ; période 247 → 299 562 ms ; fréquence événementielle (précision ms).
- **B2.** Apport : donne les prix exécutables exacts à chaque milliseconde, là où A-G1 ne donne que des extrema annotés.
- **B3.** Calculs : spread = ask−bid ; carnet croisé acheteur = yes_ask+no_ask < 1 ; croisé vendeur = yes_bid+no_bid > 1 ; durée d'un état = t de l'événement suivant − t courant ; sur les 2 265 lignes complètes.
- **B4.**


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Lignes totales / complètes | 2 267 / 2 265 | toutes | 2 267
| Spread YES médian | 0,0200$ | yes_ask−yes_bid | 2 265
| Spread YES moyen | 0,0422$ | idem | 2 265
| Spread YES max | 0,5700$ (t=297 950) | idem | 2 265
| Spread YES min | −0,0700$ (t=298 933) | idem | 2 265
| Événements yes_ask+no_ask<1 | 23 (somme min 0,9300 à t=298 933) | 4 colonnes | 2 265
| Événements yes_bid+no_bid>1 | 23 (somme max 1,0700 à t=298 933) | 4 colonnes | 2 265
| Durée des états croisés | méd 0 ms, max 0 ms, aucun >100 ms | t_ms | 46 états
| yes_ask à t=293 021 (en vigueur jusqu'à 293 294) | 0,20$ | yes_ask | 1
| Dernier BBO propre | t=299 539 : YES 0,04/0,05 | toutes | 1


- **B5.** Observations brutes : (i) de 292 754 à 299 095 ms, yes_ask oscille entre 0,04$ et 0,24$ (hors transitoires) ; (ii) les 46 états croisés ont TOUS une durée de 0 ms — corrections intra-milliseconde.
- **B6.** Anomalies : t=298 933, YES bid 0,17 / ask 0,10 (croisé, spread −0,07$) ; t=299 562, triple mise à jour finale aboutissant à YES −/0,01 : démantèlement du carnet, bruit et non signal.


### B-F2 — trades.csv

- **B1.** Colonnes : t_ms, usd, direction (+1 haussier / −1 baissier) ; 3 847 lignes ; période 426 → 299 864 ms.
- **B2.** Apport : volumes réellement échangés — le seul proxy de profondeur disponible (le BBO n'a pas de tailles).
- **B3.** Calculs : sommes par direction, par minute, par tranche 270–300 s ; médiane/moyenne des tailles.
- **B4.**


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Volume total | 51 715,57$ | usd | 3 847
| Volume haussier | 16 375,14$ (31,66%) | usd, direction | 3 847
| Volume baissier | 35 340,43$ | usd, direction | 3 847
| Part haussière min 1 / min 2 | 0,192 / 0,189 | usd, direction | 564 / 611
| Part haussière min 4 | 0,315 | usd, direction | 1 205
| Derniers 30 s | up 3 285,71$ vs down 9 296,41$ (643 trades) | usd, direction, t_ms | 643
| Plus gros trade | 1 148,42$, baissier, t=299 864 | usd, direction | 1
| Taille médiane / moyenne | 3,90$ / 13,44$ | usd | 3 847
| Couverture temporelle | 300/300 secondes avec ≥1 trade | t_ms | 3 847


- **B5.** Observations brutes : un ordre taker de 4,60$ est INFÉRIEUR à la taille médiane+écart observée dans la zone 290–296 s (prints réguliers de 10–200$) — l'exécution d'un lot de 23 shares y est réaliste.
- **B6.** Anomalies : deux prints de 498,22$ (t=68 810) et 498,97$ (t=100 910), et le print final 1 148,42$ à 136 ms de la clôture — comportement de liquidation, pas de découverte de prix.


### B-F3 — spot.csv (Binance direct)

- **B1.** Colonnes : t_ms, price ; 29 628 lignes ; période 299 → 299 683 ms ; fréquence tick (rafales par ms).
- **B2.** Apport : le référentiel que la foule regarde (flux le plus dense), à confronter à l'oracle.
- **B3.** Calculs : premier/dernier/min/max ; alignement au dernier tick oracle ≤ t pour le basis.
- **B4.**


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Premier tick | 65 299,70$ (t=299) | price | 1
| Dernier tick | 65 308,35$ (t=299 683) | price | 1
| Min / Max | 65 184,00$ / 65 321,77$ | price | 29 628
| Amplitude | 137,77$ (0,21% du spot) | price | 29 628
| Basis spot−oracle médian | +46,62$ | price ×2 fichiers | 29 628
| Basis moyen | +46,32$ | idem | 29 628
| Basis min / max | +16,89$ / +69,64$ | idem | 29 628
| Écart ouverture spot vs strike oracle | +47,96$ (65 299,70 − 65 251,74) | price | 2


- **B5.** Observations brutes : le basis ne change JAMAIS de signe (min +16,89$) : quiconque juge « Up/Down » sur Binance par rapport à l'open Binance obtient une réponse systématiquement décalée de ~46$ par rapport au couple oracle/strike qui résout le marché.
- **B6.** Anomalies : rafales de ticks identiques au même t_ms (jusqu'à 40+ répétitions, ex. t=1 304) — duplication de flux, sans impact sur les prix extrêmes.


### B-F4 — oracle.csv (Chainlink BTC/USD)

- **B1.** Colonnes : t_ms, price, ts_src ; 282 lignes ; période 0 → 299 000 ms ; cadence nominale 1 000 ms (méd des gaps 1 000 ms, max 3 000 ms, 17 secondes manquantes) ; ts_src = payload sur 282/282 (0 fallback).
- **B2.** Apport : le référentiel de RÉSOLUTION (hypothèse P0-3) et le strike exact.
- **B3.** Calculs : strike = price(t=0) ; marge = price − strike ; comptage des croisements de strike.
- **B4.**


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Strike (dernier tick ≤ t0) | 65 251,74$ (65 251,743647) | price | 1
| Dernier tick | 65 261,61$ (t=299 000) | price | 1
| Issue vs strike | **UP par +9,86$** | price | 2
| Croisements du strike | 9 (2 000 ; 7 000 ; 11 000 ; 32 000 ; 200 000 ; 204 000 ; 247 000 ; 263 000 ; 292 000 ms) | price | 282
| Dernier croisement (haussier) | t=292 000, marge +0,53$ | price | 1
| Marges t=293 000 → 299 000 | +9,81 ; +7,43 ; +10,96 ; +14,57 ; +13,39 ; +10,48 ; +9,86$ | price | 7
| Min / Max oracle | 65 145,37$ / 65 275,28$ | price | 282
| Fraîcheur pire cas d'un tick | 1 000 ms de cadence + 3 000 ms de gap max | t_ms | 281 gaps


- **B5.** Observations brutes : (i) l'oracle est au-dessus du strike SANS INTERRUPTION de t=292 000 à t=299 000, soit 7 000 ms terminales, avec une marge ≥ +7,43$ dès t=293 000 ; (ii) pendant ces mêmes 7 000 ms, le carnet (B-F1) vend YES entre 0,04$ et 0,24$ et la foule (B-F2) déverse 9 296,41$ à la baisse.
- **B6.** Anomalies : 17 secondes sans tick (gaps 2–3 s) — la règle d'entrée devra exiger DEUX ticks consécutifs pour ne pas décider sur un tick isolé potentiellement stale.


### B-F5 — Corrélation croisée spot → carnet (calcul dérivé B-F1×B-F3)

Corrélation entre la variation 1 s du spot et la variation 1 s SUIVANTE du YES mid, par pas de 200 ms (grille 100 ms, 2 977 points) : 0,153 à lag 0 ; 0,065 à 200 ms ; **maximum décalé 0,083 à 1 200 ms** ; ≤ 0 au-delà de 2 200 ms. Le carnet suit le spot avec ~1 s de retard mais avec un pouvoir prédictif marginal (r² < 0,7%).

---

## 5. PARTIE C — STRATÉGIES ET SIMULATION

### C0 — CANDIDATES

Mes cinq observations les plus fortes :

- **OBS-1** (B-F3/A-G2) : basis spot−oracle persistant, méd +46,62$, jamais négatif [+16,89 ; +69,64].
- **OBS-2** (B-F4) : oracle > strike sans interruption de 292 000 à 299 000 ms, marge ≥ +7,43$ dès 293 000.
- **OBS-3** (B-F1/A-G1) : pendant cette fenêtre, yes_ask ∈ [0,04 ; 0,24] — divergence prix/résolution de ~0,80$.
- **OBS-4** (B-F2/A-G3) : flux baissier 68,34% du volume total ; derniers 30 s : 9 296,41$ down vs 3 285,71$ up — la foule trade le référentiel Binance (open 65 299,70$), pas le strike oracle (65 251,74$).
- **OBS-5** (B-F1/A-G5/A-G6) : 46 états de carnet croisé, durée 0 ms chacun ; écart inter-carnets borné à ±0,02$.


**Candidate 1 — ARB-CROISÉ (arbitrage structurel YES+NO < 1$).** Acheter simultanément yes_ask et no_ask quand leur somme < 1, encaisser 1$ à résolution quel que soit le résultat. Règles : déclenchement si somme ≤ 0,98 (OBS-5). Conformité P0 : OK (2 ordres taker). **FILTRE DE FAISABILITÉ : latence de boucle = lecture book 31 ms + décision 10 ms [HYPOTHÈSE n°1 : décision locale 10 ms] + envoi 31 ms = 72 ms minimum ; durée de vie du signal = 0 ms (B-F4 : les 46 états croisés sont corrigés au sein du même horodatage). 72 ms > 0 ms → [STRUCTURELLEMENT IMPOSSIBLE], éliminée.** Gain brut théorique max de toute façon : 0,07$/paire (somme min 0,93, t=298 933).

**Candidate 2 — MOMENTUM-BINANCE (suivre le spot, devancer le carnet).** Acheter YES quand le spot monte, revendre 1–2 s après, en exploitant le retard ~1 200 ms du carnet (B-F5). Conformité P0 : OK. Faisabilité : boucle = âge signal Binance 219 ms + décision 10 ms + envoi 31 ms = 260 ms < durée du signal (~1 200 ms) → structurellement possible. Mais chiffrage : pouvoir prédictif r=0,083 sur un mid dont la variation 1 s typique est ~0,01–0,02$, soit une espérance ≤ 0,002$/share/trade ; frais aller-retour taker à p≈0,5 : 2 × 0,07 × 0,5 × 0,5 = **0,035$/share** (P0-1). Espérance nette ≈ −0,033$/share par aller-retour → P&L net réel négatif certain sur 3 847 trades de tape comparables. Éliminée par les chiffres.

**Candidate 3 — SNIPER-ORACLE T-7s (divergence résolution/carnet en fin de fenêtre).** Le marché résout sur l'oracle vs strike 65 251,74$ (P0-3, hypothèse conservatrice) ; la foule price le référentiel Binance (OBS-1, OBS-4). Quand, à moins de 10 s de la clôture, l'oracle est durablement au-dessus du strike (OBS-2) alors que yes_ask ≤ 0,25 (OBS-3), acheter YES et porter à résolution. Conformité P0 : 1 ordre taker, frais 0,07 intégrés, 2 req → OK. Faisabilité : boucle = lecture Chainlink on-chain 68 ms + décision 10 ms [HYPOTHÈSE n°1] + envoi CLOB 31 ms = **109 ms** (confirmation : +31 ms = 140 ms) ; durée de vie du signal = 6 000–7 000 ms (OBS-2). 109 ms << 6 000 ms → faisable.

**CRITÈRE DE SÉLECTION : P&L net réel maximal (après latences r1-r2, slippage r3, frais P0-1 r4), sous contrainte d'élimination préalable de toute candidate structurellement impossible ou à espérance nette négative.** Critère figé avant comparaison.

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----|-----
| ARB-CROISÉ | OBS-5 | Oui | 0 (infaisable) | +0,07$/paire théorique | néant | [STRUCTURELLEMENT IMPOSSIBLE]
| MOMENTUM-BINANCE | OBS-1, B-F5 | Oui | ~dizaines | ≤ +0,002$/share | ≈ −0,033$/share | ÉLIMINÉE (net négatif)
| SNIPER-ORACLE T-7s | OBS-1→4 | Oui | 1 | +18,40$ | **+18,14$** | **RETENUE**


### C1 — STRATÉGIE RETENUE : SNIPER-ORACLE T-7s

```plaintext
# Paramètres figés avant session
STRIKE        = premier tick oracle de la fenêtre (B-F4 : 65 251,74$)
T_ARM         = 285 000 ms      # armement à T-15s (OBS-2 : zone terminale)
T_STOP        = 297 500 ms      # plus d'entrée après T-2,5s (A-G5 : carnet démantelé, spread 0,57$ à 297 950)
MARGE_MIN     = 5,00 $          # marge oracle minimale (OBS-2 : marge réelle ≥ +7,43$) [FRAGILE - 1 SOURCE]
ASK_MAX       = 0,25 $          # prix d'achat plafond (OBS-3 : yes_ask ≤ 0,24 observé)
BUDGET        = 5,00 $          # contrainte dure

boucle (à chaque tick oracle reçu, âge = 68 ms de lecture + ≤1 000 ms de cadence) :
  si t >= T_ARM et t <= T_STOP
  et tick_oracle(n) > STRIKE et tick_oracle(n-1) > STRIKE      # 2 ticks consécutifs (B-F6 : gaps oracle)
  et tick_oracle(n) - STRIKE >= MARGE_MIN                      # OBS-2
  et yes_ask <= ASK_MAX                                         # OBS-3, lu via CLOB (31 ms)
  et aucun ordre en cours et aucune position :
      N = floor( (BUDGET) / (yes_ask*(1 + 0,07*(1-yes_ask))) )  # frais P0-1 provisionnés
      envoyer BUY YES, limite = min(yes_ask + 0,05 ; ASK_MAX), taille N   # r3 : re-pricing borné
  position tenue jusqu'à résolution (aucune sortie anticipée)   # le signal est la résolution elle-même
```

Chaque condition remonte à OBS-1→OBS-5 ; la valeur MARGE_MIN=5$ n'est étayée que par cette session : [FRAGILE - 1 SOURCE].

### C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE.** Départ : 5,00$ cash, 0 position. Un seul ordre autorisé, jamais au-delà du cash (r5).

**RACE CONDITIONS.** RÈGLE (a) : signal pendant ordre en cours → ignoré (flag `ordre_en_cours`). RÈGLE (b) : aucune vente possible sur position non confirmée — la stratégie ne vend jamais, la règle est trivialement satisfaite. RÈGLE (c) : deux signaux au même tick → seul le premier traité, l'autre écarté par le flag position. RÈGLE (d) : fill partiel → conserver le fill, annuler le reliquat après 500 ms, ne pas re-soumettre.

**DÉROULÉ.** Tick oracle t=292 000 : marge +0,53$ < 5,00$ → pas d'entrée. Tick t=293 000 : marge +9,81$ ≥ 5,00$ ET tick précédent (292 000) > strike → signal. Chronologie r1/r2 : émission on-chain 293 000 + lecture 68 ms = 293 068 ; décision +10 ms = 293 078 ; arrivée CLOB +31 ms = **exécution à t=293 109 ms**. BBO en vigueur à 293 109 (B-F1) : événement t=293 021, YES 0,19/0,20, stable jusqu'à 293 294 → fill à l'ask **0,20$** (r1 : le prix APRÈS le décalage ; il se trouve que l'ask sans latence, à 293 000, était 0,21$ — la latence est ici favorable de 0,01$). r3 : profondeur au niveau 0,20$ = DONNÉE NON DISPONIBLE (BBO sans tailles) ; règle énoncée : limite 0,25$, et plausibilité vérifiée par B-F2 (prints de 10–200$ dans la zone 290–296 s, taille médiane 3,90$) — un lot de 4,60$ est dans la norme du tape. r4 : frais = 23 × 0,07 × 0,20 × 0,80 = 0,2576$. r5 : 4,60 + 0,2576 = 4,8576 ≤ 5,00 ✓.

**JOURNAL DE TRADES.**

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----
| 1 | 293 000 ms | 293 109 ms | 0,20$ | 23 shares (4,60$) | résolution (300 000 ms) | 1,00$ | −0,01$/sh (favorable) | 0,2576$ | **+18,1424$** | 0,1424$ puis **23,1424$**


**RÉSULTATS.** 1 trade gagnant, 0 perdant. Frais totaux : 0,2576$. BÉNÉFICE NET : **+18,1424$**, arrondi +18,14$. Capital final : **23,14$** vs 5,00$ = **+362,85%**. Pire perte réalisée : 0,00$. Drawdown maximal LATENT (mark-to-market au yes_bid) : à t=297 969 le bid touche 0,01$ → valeur 23×0,01+0,1424 = 0,37$, soit **−92,55%** de drawdown intra-trade non réalisé — il faut tenir la position sans stop, c'est constitutif de la stratégie. P&L THÉORIQUE sans r1–r4 : fill à l'ask de l'instant signal (0,21$ à 293 000), sans frais : 23 shares, coût 4,83$, net +18,17$, capital 23,17$. Coût total du réalisme : **0,03$** (frais 0,2576$ presque compensés par un slippage de latence favorable de 0,23$).

---

## 6. PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----
| D1 Latence de boucle | 109 ms d'entrée absorbés | Signal vivant 6 000 ms (B-F4) ; fill 0,20$ vs 0,21$ sans latence (B-F1) | +0,23$ (favorable) — preuve chiffrée | [SANS IMPACT]
| D2 Fraîcheur du signal | tick oracle âgé ≤ 68 + 1 000 ms | Gaps oracle jusqu'à 3 000 ms (B-F4, 17 s manquantes) ; marge min pendant la fenêtre +7,43$ ne change pas de signe | borne : décision retardée jusqu'à t=296 000 → ask ≤ 0,09$ (B-F1), net encore > +18$ | [DÉGRADE] borné
| D3 Slippage / profondeur | fill intégral au meilleur ask | Tailles BBO NON DISPONIBLES ; borne pire cas : fill à la limite 0,25$ → N=18, frais 0,2363$, net +13,16$ | −4,98$ max | [DÉGRADE] borné
| D4 Frais + gas | fee 0,07, gas 0 | P0-1 (docs 09/08/2026) : 0,2576$ payés, gas 0$ | −0,2576$ = 5,15% du capital, net reste +362,85% | [SANS IMPACT] prouvé
| D5 Niveau disparu / fill partiel | limite 0,25$, règle (d) | Prints concurrents de 10–200$ dans la zone (B-F2) | couvert par la borne D3 | [DÉGRADE] borné
| D6 Défaillances techniques | ordre confirmé en 140 ms | Timeout possible ; RÈGLE : non confirmé avant t=295 000 → annulation, aucune re-soumission après T_STOP | scénario : 0 trade, capital 5,00$ (0 perte) | [DÉGRADE]
| D7 Passage à l'échelle | taille 4,60$ | Volume haussier total des 30 dernières secondes : 3 285,71$ (B-F2) ; au-delà de ~100$ l'ordre traverse le carnet sans données de profondeur | mécanisme éteint entre ~20× et ~700× le capital ; borne haute non résolvable (tailles NON DISPONIBLES) | [DÉGRADE]
| D8 Dépendance à la session | la divergence résolution/carnet se reproduit | 1 seule occurrence observée ; sur une session calme les conditions (marge ≥5$ ET ask ≤0,25 ET t≥285 s) ne co-existent pas → 0 trade, 5,00$ | gain conditionné à UN événement de divergence | [DÉGRADE] — dépendance forte
| D9 Conformité au nouveau fonctionnement | résolution = oracle Chainlink vs strike 65 251,74$ ; frais 0,07 | Frais : vérifiés (P0-1). Résolution 5 min : DONNÉE NON VÉRIFIABLE par page datée ; convention prise du matériel fourni (PDF : strike « dernier tick ≤ t0 », Résultat ▲ UP). Contre-scénario : si la résolution suivait le référentiel de la foule, le trade perdrait 4,86$ | de +18,14$ à −4,86$ selon la règle réelle | [DÉGRADE] — point critique


**VERDICT GLOBAL (règles mécaniques).** Aucun [DÉTRUIT] ; P&L net réel positif (+18,14$) ; aucune violation P0. Mais [FIABLE] exige D8 ≠ dépendance exclusive à un événement unique — or le gain provient d'UNE divergence résolution/carnet, et D9 repose sur une hypothèse de résolution non vérifiable par source datée. **VERDICT : [FRAGILE]. Étiquette : [SESSION-SPÉCIFIQUE].** La seule composante défendable comme récurrente est le mécanisme (surveiller l'oracle de résolution plutôt que le flux que la foule regarde), pas les seuils.

---

## 7. AUTO-CONTRÔLE FINAL

1. Phase 0 complète → P0-1, P0-2, P0-4 sourcés (docs.polymarket.com, consultés 09/08/2026) ; P0-3 crypto 5 min marqué DONNÉE NON VÉRIFIABLE avec hypothèse conservatrice explicite ; changement récent identifié (taux crypto 0,06→0,07). **OK.**
2. Valeurs extraites : ≥5 par graphe (A-G1 : 5 ; A-G2 : 6 ; A-G3 : 5 ; A-G4 : 5 ; A-G5 : 6 ; A-G6 : 5) ; ≥8 par CSV (B4 : 10, 9, 8, 8). **OK.**
3. Chaque règle de C1 cite OBS-1→OBS-5, elles-mêmes ancrées A5/B5 antérieurs ; MARGE_MIN marquée [FRAGILE - 1 SOURCE]. **OK.**
4. Boucle reconstituée chemin par chemin : 68 (Chainlink) + 10 [HYPOTHÈSE n°1] + 31 (CLOB) = 109 ms, appliquée à l'unique entrée du journal (293 000 → 293 109). **OK.**
5. Comptabilité : 5,00 − 4,60 − 0,2576 = 0,1424 ; 0,1424 + 23×1,00 = 23,1424 = capital final annoncé (23,14$), au centime près. **OK.**
6. D1–D9 traités ; les deux [SANS IMPACT] portent leur preuve (+0,23$ mesuré ; 0,2576$ chiffré) ; verdict global conforme aux règles mécaniques. **OK.**
7. Cohérence des chiffres ouverture/corps/clôture : 23,14$ ; +362,85% ; 1 trade ; 0,2576$ ; 0,07 ; [FRAGILE] ; [SESSION-SPÉCIFIQUE] — identiques partout. **OK.**


---

## 8. CLÔTURE

Le mécanisme gagnant de cette session tient en une phrase : le marché se résout sur un référentiel que la foule ne regardait pas. Je vous recommande de ne déployer cette stratégie qu'après vérification directe, marché par marché, de la règle de résolution en vigueur — c'est elle, et non la latence, qui décide du signe du résultat. En l'état, le verdict reste celui annoncé en ouverture.



**Postambule.** J'ai copié les 4 CSV dans `data/` pour les analyser par script (statistiques, fenêtres de carnet croisé, corrélations lead-lag, simulation du fill à la milliseconde), lu les 6 graphes du PDF, et vérifié la doc Polymarket en ligne (frais, rate limits, résolution UMA) au 09/08/2026. La divergence centrale — oracle +9,86$ au-dessus du strike pendant que le carnet vendait YES à 0,20$ — est mesurée directement dans `bbo.csv` et `oracle.csv`, et la simulation applique intégralement vos latences, les frais taker 0,07 et la contrainte de 5,00$.

---

## Source : 285SNIPE-ORACLE.md

# 0. SYNTHÈSE EXÉCUTIVE

- Stratégie retenue : **SNIPE-ORACLE** — achat taker déclenché par saut de tick Chainlink ≥ |4,00$|, sortie sur stop oracle ou résolution.
- Capital final : **18,18$** vs 5,00$, soit **+263,56%** net, APRÈS latences (boucle 130 ms), slippage (0,00$ mesuré) et frais réels (0,33$).
- Trades : **2** — 1 gagnant (+13,64$), 1 perdant (−0,46$).
- Changement Polymarket le plus impactant : depuis le **07/08/2026**, les marchés crypto 5 min ne se résolvent plus sur un snapshot dernier-tick mais sur un **TWAP Chainlink de 30 secondes** (docs.polymarket.com/polymarket-learn/markets/crypto-markets, consulté le 10/08/2026).
- VERDICT DE FIABILITÉ : **[FRAGILE]**.
- Étiquette : **[SESSION-SPÉCIFIQUE]**.


---

# 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET

Je commence par ce qui est vérifié aujourd'hui dans la documentation officielle, car c'est le socle de contraintes dures de tout ce qui suit.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × 0,07 × p × (1−p), en USDC, arrondi 5 décimales, min 0,00001 USDC | docs.polymarket.com/polymarket-learn/trading/fees | consulté 10/08/2026 | OUI (frais taker crypto généralisés, rebate maker 20%)
| P0-1 Frais maker | 0 ; rebate maker 20% des frais, redistribué quotidiennement | docs.polymarket.com/polymarket-learn/trading/fees | consulté 10/08/2026 | OUI
| P0-1 Dépôt/retrait/résolution | 0$ de frais Polymarket ; rédemption à résolution non facturée | docs.polymarket.com/polymarket-learn/trading/fees | consulté 10/08/2026 | Non
| P0-1 Gas | DONNÉE NON VÉRIFIABLE (page settlement non consultée) — hypothèse conservatrice : 0$ au trade (matching off-chain), tout écart borné par D6 | — | — | —
| P0-2 Tick size | 0,01$ (100% des 590 événements BBO du CSV sont au pas de 0,01$) | observation B-F2 + docs.polymarket.com/market-data/market-details | consulté 10/08/2026 | Non
| P0-2 Taille min d'ordre | DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : 1,00$ minimum, appliquée à la simulation | — | — | —
| P0-2 Types d'ordres | DONNÉE NON VÉRIFIABLE (page orders non consultée) — hypothèse conservatrice : limites marketable uniquement, simulées en FOK | — | — | —
| P0-3 Résolution 5 min | **TWAP 30 s** du flux Chainlink Data Streams (60 s pour 15 min/4 h), remplace le snapshot single-tick | docs.polymarket.com/polymarket-learn/markets/crypto-markets (fetch direct indisponible ; contenu corroboré par recherche datée) | 07/08/2026 | **OUI — c'est le changement des dernières 48 h**
| P0-4 Rate limits CLOB | POST /order : 5 000 req/10 s burst, 120 000/10 min ; /book 1 500 req/10 s ; général CLOB 9 000 req/10 s | docs.polymarket.com/api-reference/rate-limits | consulté 10/08/2026 | Non


Implication du changement P0-3 pour une fenêtre de 5 minutes : le résultat n'est plus décidé par le dernier tick à t=300 s mais par la moyenne des ticks de t=270 s à t=300 s. Toute stratégie de "snipe du dernier tick" est morte par construction ; toute stratégie directionnelle voit son risque de manipulation terminale réduit.

# 2. CADRE D'ANALYSE

Le PDF contient 3 familles de panneaux (carnet YES/NO, spot vs oracle, flux directionnel), chacune en vue 300 s + zooms 15 s : je définis A-G1/A-G3/A-G5 = les 3 vues 300 s et A-G2/A-G4/A-G6 = les 3 séries de zooms. Tous les timestamps sont en mm:ss.mmm depuis l'ouverture ; strike = 64 225,00$ (étiquette PDF, dernier tick oracle ≤ t0). Résultat de session : UP.

---

# PARTIE A — LES 6 GRAPHES

## A-G1 — CARNET YES/NO COMPLET, VUE 300 s

- A1. X : temps 00:00.000→05:00.000 ; Y gauche : prix 0–1$ ; Y droit : probabilité ; 590 événements BBO, 8 marqués incohérents.
- A2. Le carnet a-t-il pricé l'issue UP progressivement ou par ruptures ?
- A3. Lecture des extrema étiquetés YES mid et des paliers.
- A4. t=00:00.164 : YES 0,52$ ; t=02:05.052 : YES 0,11$ (point bas) ; t=02:05.052 : NO 0,89$ ; t=04:06.862 : YES 0,99$ ; t=04:06.862 : NO 0,01$.
- A5. Le YES passe de 0,11$ à 0,99$ en 121,810 s (02:05.052→04:06.862) : le repricing est par ruptures, pas continu. Fallback ts : 0/284 (0,0%) — horodatage fiable.
- A6. Spécifique : un aller 0,52→0,11→0,99 dans une même fenêtre exige un retournement spot de +71,29$ (B-F4) ; c'est un profil de session volatile.


## A-G2 — CARNET YES/NO, ZOOMS 15 s (01:20, 02:19, 03:37)

- A1. Mêmes axes, fenêtres [01:20.000→01:35.000], [02:19.000→02:34.000], [03:37.000→03:52.000].
- A2. À quelle vitesse le carnet reprice-t-il après un choc ?
- A3. Suivi tick à tick des étiquettes YES/NO mid.
- A4. t=02:26.854 : YES 0,23$ ; t=02:27.014 : YES 0,29$ ; t=02:33.615 : YES 0,47$ ; t=01:27.512 : YES 0,14$ → t=01:27.697 : 0,19$ ; t=03:44.325 : YES 0,95$ → t=03:44.817 : 0,96$.
- A5. **Le repricing 0,23$→0,47$ s'étale sur 6,761 s (02:26.854→02:33.615)** : un signal exogène plus rapide que le carnet dispose de plusieurs secondes de fenêtre. C'est l'observation fondatrice.
- A6. Spécifique : l'amplitude (+0,24$ en 6,8 s) dépend du choc spot de cette session.


## A-G3 — SPOT BINANCE vs ORACLE CHAINLINK, VUE 300 s

- A1. X : temps ; Y : prix BTC 64 180–64 340$ ; 6 994 ticks Binance, 284 ticks Chainlink ; strike 64 225$ tracé.
- A2. Lequel des deux flux mène, et lequel décide ?
- A3. Comparaison des niveaux et des extrema étiquetés.
- A4. Binance t=00:01.288 : 64 265,00$ ; Binance min t=00:51.147 : 64 237,85$ ; Binance t=04:59.916 : 64 336,29$ ; Oracle t=00:00.000 : 64 215,97$ ; Oracle t=04:58.000 : 64 277,08$.
- A5. Le spot Binance ne passe **jamais** sous le strike (min 64 237,85$ > 64 225$) alors que l'oracle est sous le strike pendant 182 s : les deux séries vivent à des niveaux différents, seul l'oracle décide la résolution.
- A6. Spécifique : l'écart de niveau (basis) est propre au couple BTC/USDT Binance vs BTC/USD Chainlink de cette période.


## A-G4 — SPOT vs ORACLE, ZOOMS 15 s + BASIS

- A1. Mêmes axes + sous-panneau basis = spot − oracle (0–60$+), 6 994 points.
- A2. Le basis est-il un signal d'avance ou un simple offset ?
- A3. Lecture du basis sur les 3 fenêtres.
- A4. t=02:19.286 : Binance 64 251,99$ vs oracle 64 196,19$ (basis 55,80$) ; t=02:33.335 : 64 268,00$ ; t=03:37.073 : 64 295,99$ vs oracle 64 240,22$ (basis 55,77$) ; t=01:28.487 : 64 245,62$ ; t=01:20.312 : 64 237,85$.
- A5. Le basis reste dans une bande étroite (B-F4 : 46,86$–64,09$, moyenne 56,25$) : c'est un offset structurel, pas une avance temporelle exploitable de plusieurs secondes.
- A6. Spécifique : la valeur 56,25$ est propre à la session ; seul le caractère "quasi constant" est un candidat pattern.


## A-G5 — FLUX DIRECTIONNEL NORMALISÉ, VUE 300 s

- A1. X : temps ; Y : volume $ signé (haussier +, baissier −), fenêtre 30 s, 1 380 trades.
- A2. Le flux agressif précède-t-il ou suit-il le repricing ?
- A3. Lecture des extrema étiquetés.
- A4. t=00:00.407 : −12,25$ ; t=03:37.536 : +583,91$ (pic haussier) ; t=04:08.094 : −614,97$ (pic baissier) ; t=04:40.625 : −39,60$ ; échelle −2 000$/+5 000$.
- A5. Le pic haussier (+583,91$ à 03:37.536) arrive alors que YES cote déjà 0,93–0,94$ (A-G2) : le gros flux **suit** le repricing, il ne le crée pas.
- A6. Spécifique : les montants absolus (583,91$ ; 614,97$) sont propres à la session.


## A-G6 — FLUX DIRECTIONNEL, ZOOMS 15 s

- A1. Mêmes axes, 3 fenêtres 15 s.
- A2. Quelle est la granularité des trades pendant les ruptures ?
- A3. Lecture trade par trade des étiquettes.
- A4. t=02:20.251 : −398,50$ ; t=02:27.077 : +16,02$ ; t=02:32.948 : −116,67$ ; t=03:44.873 : +216,23$ ; t=03:44.994 : −242,20$ ; t=01:26.768 : −100,00$.
- A5. Pendant la rupture 02:27→02:33, les trades haussiers individuels sont petits (16,02$ ; 24,80$ ; 13,08$) : le repricing se fait à petites tailles — un ordre de 5$ y est exécutable sans distordre le carnet.
- A6. Spécifique : la composition exacte du flux est propre à la session.


---

# PARTIE B — LES CSV

## B-F1 — oracle.csv (Chainlink)

- B1. Colonnes t_ms, price, ts_src ; 284 lignes ; 00:00.000→04:58.000 ; cadence moyenne 1 053 ms, écart max 2 000 ms.
- B2. Apport : la série de résolution elle-même, tick par tick — invisible à cette précision sur les graphes.
- B3. Deltas tick-à-tick, TWAP fenêtre 270–300 s, franchissements de strike.
- B4.


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Premier tick | 64 215,97$ (t=00:00.000) | t_ms, price | 1
| Dernier tick | 64 277,08$ (t=04:58.000) | t_ms, price | 1
| Delta session | +61,11$ | price | 284
| Cadence moyenne | 1 053 ms (max 2 000 ms) | t_ms | 283
| Delta tick min / max | −7,66$ (t=00:49.000) / +6,35$ (t=02:27.000) | price | 283
| Sauts ≥ |4$| | 6 (t=48 s, 49 s, 65 s négatifs ; 136 s, 147 s, 225 s positifs) | price | 283
| 1er tick > strike / dernier ≤ strike | t=03:02.000 (64 226,23$) / t=03:01.000 (64 223,20$) | t_ms, price | 2
| TWAP 270–300 s | 64 275,56$ (29 ticks, min 64 272,95$) | t_ms, price | 29
| ts_src ≠ payload | 0/284 (0,0%) | ts_src | 284


- B5. La résolution TWAP (P0-3) était acquise dès t=270 s : minimum de la fenêtre à strike +47,95$.
- B6. Anomalie : le strike PDF (64 225,00$) ne correspond pas au premier tick du CSV (64 215,97$) — le "dernier tick ≤ t0" est antérieur à la fenêtre et absent du fichier ; je retiens 64 225,00$ (PDF autoritaire). Bruit, pas signal.


## B-F2 — bbo.csv (carnet)

- B1. Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 590 lignes ; 00:00.164→04:06.862 ; événementiel (gap médian 11 ms, max 10 420 ms).
- B2. Apport : les quotes exécutables exactes à la milliseconde.
- B3. Spread YES, tests de carnet croisé (seuil 0,001$), quotes aux instants d'exécution simulés.
- B4.


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Spread YES moyen / médian / max | 0,0187$ / 0,02$ / 0,07$ | yes_bid, yes_ask | 590
| Carnets croisés (seuil 0,001$) | 0 sur 590 | 4 colonnes | 590
| Quote à t=00:47.905 (valide jusqu'à 00:48.392) | NO ask 0,84$ | no_ask | 1
| Quote à t=02:16.021 (valide jusqu'à 02:19.867) | YES ask 0,23$ / NO bid 0,77$ | yes_ask, no_bid | 1
| YES mid t=02:27.014 → t=02:33.615 | 0,30$ → 0,475$ (6,601 s) | yes_bid, yes_ask | 60
| YES min de session | bid 0,09$ (t=02:03.221) | yes_bid | 1
| Dernier événement | 0,99/0,99 à t=04:06.862 | 4 colonnes | 1
| Tick observé | 0,01$ sur 100% des lignes | 4 colonnes | 590


- B5. Zéro carnet croisé exploitable : l'arbitrage YES+NO<1$ n'a eu **aucune** occurrence au seuil du tick.
- B6. Les "8 incohérents" du PDF ne franchissent pas le seuil 0,001$ dans le CSV : artefacts d'affichage, pas de signal.


## B-F3 — trades.csv (flux)

- B1. Colonnes t_ms, usd, direction ; 1 380 lignes ; 00:00.407→fin de fenêtre.
- B2. Apport : la taille réellement exécutée au BBO — mon proxy de profondeur.
- B3. Agrégats par direction et par minute, distribution des tailles.
- B4.


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Volume total | 21 508,66$ | usd | 1 380
| Volume haussier / baissier | 11 201,07$ (782) / 10 307,59$ (598) | usd, direction | 1 380
| Taille médiane / moyenne | 4,80$ / 15,59$ | usd | 1 380
| P90 / max | 36,00$ / 614,97$ | usd | 1 380
| Trades ≥ 100$ | 44, cumulant 8 995,68$ | usd | 44
| Flux 01:00→02:00 | +343,81$ haussier vs −2 102,41$ baissier | usd, direction | tranche
| Flux 03:00→04:00 | +4 916,72$ vs −2 186,42$ | usd, direction | tranche
| Trades 02:15.5→02:17.5 | 16 trades, aucun > 31,97$ | t_ms, usd | 16


- B5. Un ordre de ~5$ est **sous la taille médiane** exécutée au BBO : hypothèse de fill complet au top-of-book raisonnable pour ce capital.
- B6. Anomalie : 6 timestamps localement non monotones (ex. t=00:07.988 avant t=00:07.984) — désordre d'ingestion de quelques ms, bruit.


## B-F4 — spot.csv (Binance)

- B1. Colonnes t_ms, price ; 6 994 lignes ; 00:01.288→04:59.916.
- B2. Apport : le flux "rapide" pour tester si Binance offre une avance sur l'oracle.
- B3. Basis = spot − dernier tick oracle ≤ t, extrema, franchissements.
- B4.


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Premier / dernier tick | 64 265,00$ / 64 336,29$ | t_ms, price | 2
| Min / max | 64 237,85$ (t=00:51.147) / 64 336,29$ | price | 6 994
| Delta session | +71,29$ | price | 6 994
| Basis moyen / médian | +56,25$ / +56,23$ | price + B-F1 | 6 994
| Basis min / max | +46,86$ / +64,09$ | price + B-F1 | 6 994
| Ticks spot ≤ strike | 0 sur 6 994 | price | 6 994
| Spot à t=02:16.000 | 64 251,99$ (oracle : 64 195,19$) | price | 1
| Spot à t=03:37.073 | 64 295,99$ | price | 1


- B5. Basis borné dans une bande de 17,23$ : Binance n'apporte pas d'avance directionnelle nette au-delà de la cadence oracle (1 053 ms) — et son signal a déjà 219 ms d'âge.
- B6. Rafales de ticks au même t_ms (ex. 30 lignes à t=00:02.252) : agrégation de trades au même horodatage, bruit.


---

# PARTIE C — STRATÉGIES ET SIMULATION

## C0. CANDIDATES

Mes cinq observations les plus fortes :

- **OBS-1** : le carnet reprice en 6,761 s après un choc (A-G2 : 0,23$→0,47$, 02:26.854→02:33.615).
- **OBS-2** : 6 sauts oracle ≥ |4$| dans la session, chacun suivi d'un repricing (B-F1 + B-F2).
- **OBS-3** : boucle oracle→CLOB = 68 + 31 + 31 = **130 ms**, soit 2,0% de la durée de vie du signal OBS-1 (tableau des latences).
- **OBS-4** : zéro carnet croisé sur 590 événements (B-F2 B5) — pas d'arb YES+NO.
- **OBS-5** : dès t=04:06.862, YES cote 0,99/0,99 avec TWAP acquis à strike +47,95$ minimum (B-F1 B5, B-F2 B4).


Candidates (familles distinctes) :

1. **SNIPE-ORACLE** (momentum taker) : sur saut Chainlink ≥ +4,00$, acheter YES à l'ask si yes_ask ≤ 0,90$ (symétrique NO sur ≤ −4,00$) ; stop si l'oracle retrace de 4,00$ ; sortie forcée à t=270 s si l'oracle est du mauvais côté du strike ; sinon résolution. Fondée sur OBS-1/OBS-2/OBS-3. Conforme P0 (taker fee P0-1 intégrée, tick P0-2, TWAP P0-3 indifférent au signal, 2 ordres ≪ P0-4). Boucle 130 ms vs signal 6 761 ms → **faisable**.
2. **CONVERGENCE TWAP TERMINALE** (quasi-arb réglementaire, née du changement P0-3) : à t ≥ 270 s, si l'écart oracle–strike rend le TWAP mathématiquement inatteignable pour l'autre camp, acheter le camp gagnant à ≤ 0,99$. Fondée sur OBS-5. Conforme P0. Boucle 130 ms vs fenêtre 30 000 ms → faisable.
3. **MARKET-MAKING 2 TICKS** (maker, frais 0 + rebate 20%) : quoter bid/ask des deux côtés. Conforme P0-1. Boucle 31 ms×2 faisable, MAIS le P&L exige les données de profondeur et de file d'attente : DONNÉE NON DISPONIBLE dans les pièces → non simulable, borne haute de capture ≤ 0,02$ × rotations d'un inventaire de 5$, avec risque de résolution non couvrable. **Éliminée pour invérifiabilité.**
4. **ARB CARNET CROISÉ** : éliminée d'office — 0 occurrence (OBS-4).


**CRITÈRE DE SÉLECTION : P&L net réel maximal sur la session (après latences 130 ms, slippage confronté aux quotes B-F2, frais P0-1), sous conformité P0 et faisabilité structurelle.**

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----|-----
| SNIPE-ORACLE | OBS-1/2/3 | Oui | 2 | +13,51$ | **+13,18$** | **RETENUE**
| CONVERGENCE TWAP | OBS-5 | Oui | 1 | +0,05$ | +0,05$ | Écartée (critère)
| MARKET-MAKING | OBS-4 (a contrario) | Oui (P0-1) | — | DONNÉE NON DISPONIBLE | non calculable | Éliminée
| ARB YES+NO | OBS-4 | Oui | 0 | 0,00$ | 0,00$ | Éliminée (0 occurrence)


## C1. STRATÉGIE RETENUE — SNIPE-ORACLE

```plaintext
CAPITAL = 5.00 ; POSITION = null ; LOOP = 0.068 + 0.031 + 0.031 = 0.130 s   # tableau latences (OBS-3)
À CHAQUE TICK ORACLE o[i] :                                                  # cadence 1 053 ms (B-F1)
  d = o[i].price − o[i−1].price
  SI POSITION == null ET CASH ≥ 1.00 :                                       # P0-2 [HYPOTHÈSE n°1 : ordre min 1$]
    SI d ≥ +4.00 ET yes_ask ≤ 0.90 → BUY YES, FOK, tout le cash             # seuil issu de OBS-2 (B-F1 B4)
    SI d ≤ −4.00 ET no_ask  ≤ 0.90 → BUY NO,  FOK, tout le cash
    prix exécuté = ask en vigueur à t_signal + 0.130 s                       # r1, quotes B-F2
    frais = C × 0.07 × p × (1−p)                                             # P0-1
  SI POSITION ≠ null :
    STOP  : YES si o < o_entrée − 4.00 ; NO si o > o_entrée + 4.00 → SELL au bid (taker)
    FORCÉ : à t = 270 s, SELL si oracle du mauvais côté du strike            # P0-3 (TWAP 30 s)
    SINON : tenir jusqu'à résolution (rédemption 1.00, sans frais, P0-1)
[HYPOTHÈSE n°2 : fill complet au BBO pour ≤ 5$ — justifiée par B-F3 B5 (médiane 4,80$)] [FRAGILE - 1 SOURCE]
Seuil 4.00$ : soutenu par B-F1 seulement → [FRAGILE - 1 SOURCE]
```

## C-SIM. SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE** : départ 5,00000$ cash, 0 position, aucun levier.

**RACE CONDITIONS** :

- (a) RÈGLE : signal reçu pendant un ordre en cours → ignoré (pas de file).
- (b) RÈGLE : aucune vente avant confirmation du fill d'entrée (+31 ms).
- (c) RÈGLE : signaux simultanés → la sortie prime ; l'entrée est réévaluée après confirmation de la sortie (appliquée à t=02:16.000).
- (d) RÈGLE : FOK — fill partiel impossible ; en cas de rejet, une seule re-tentative au tick suivant.


**JOURNAL DE TRADES** (r1–r5 appliqués ; slippage = écart entre quote au signal et quote à t+0,130 s, confronté aux quotes B-F2) :

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----
| 1 | 00:48.000 (Δ−4,86$) | 00:48.130 | NO 0,84$ | 5 | 02:16.130 (stop : oracle 64 195,19$ > 64 191,00+4) | NO bid 0,77$ | 0,00$ | 0,04704 + 0,06199 | **−0,45902$** | 4,54098$
| — | 00:49.000 (Δ−7,66$) | — | — | — | — | — | — | — | ignoré : règle (a) + cash 0,75296$ < 1$ | 0,75296$
| — | 01:05.000 (Δ−4,19$) | — | — | — | — | — | — | — | ignoré : position en cours | —
| 2 | 02:16.000 (Δ+4,37$) | 02:16.260 (règle c : après confirmation sortie #1) | YES 0,23$ | 18 | résolution (UP) | 1,00$ | 0,00$ | 0,22315 | **+13,63685$** | 0,17783$ puis 18,17783$
| — | 02:27.000 (Δ+6,35$) / 03:45.000 (Δ+6,32$) | — | — | — | — | — | — | — | ignorés : position en cours | —


Vérification slippage : quote NO ask 0,84$ postée à 00:47.905, valide jusqu'à 00:48.392 > 00:48.130 ✓ ; quote YES ask 0,23$ postée à 02:16.207, valide jusqu'à 02:19.867 > 02:16.260 ✓ (B-F2 B4). Aucun achat n'excède le cash (r5) : 4,24704 ≤ 5,00000 ; 4,36315 ≤ 4,54098 ✓.

**RÉSULTATS** :

- Trades : 2 — 1 gagnant (+13,64$), 1 perdant (−0,46$).
- Frais totaux : 0,04704 + 0,06199 + 0,22315 = **0,33218$ ≈ 0,33$**.
- BÉNÉFICE NET : **+13,17783$ ≈ +13,18$**. Capital final : **18,17783$ ≈ 18,18$** vs 5,00$ = **+263,56%**.
- Pire perte : −0,46$ (trade #1). Point bas mark-to-market : 3,95783$ (t=02:19.877, yes_bid 0,21$) ; drawdown max : **−23,19%** depuis le pic 5,15296$ (t=02:05.052, no_bid 0,88$).
- P&L THÉORIQUE (sans r1–r4, exécution au quote du signal, 20 parts au trade 2) : capital final 20,05$, soit +15,05$. **Coût du réalisme : 1,87$** (0,33$ de frais + 1,54$ d'effet de dimensionnement induit par les frais).


---

# PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----
| D1 Latence boucle | 130 ms n'altère pas le prix | Quotes inchangées sur [t_signal ; t_signal+0,130 s] aux 2 trades (B-F2 B4) | 0,00$ mesuré | [SANS IMPACT]
| D2 Fraîcheur signal | Tick oracle frais à la décision | Âge 68 ms = 6,5% de la cadence 1 053 ms (B-F1 B4) ; Binance (219 ms) non utilisé | 0,00$ mesuré | [SANS IMPACT]
| D3 Slippage/profondeur | Fill complet ≤ 5$ au BBO | Profondeur : DONNÉE NON DISPONIBLE ; médiane des fills 4,80$ (B-F3 B4) ; borne : +1 tick = −0,18$ sur T2 (18×0,01$) | borné −0,23$ | [DÉGRADE]
| D4 Frais complets | fee = C×0,07×p(1−p), gas 0 | P0-1 vérifié ; gas : hypothèse conservatrice bornée par D6 | −0,33$ (2,5% du P&L) | [DÉGRADE]
| D5 Niveau disparu / fill partiel | FOK + 1 re-tentative | Quotes stables ≥ 262 ms aux 2 entrées (B-F2) ; échec = trade non ouvert | borné : gain manqué, 0$ de perte capital | [DÉGRADE]
| D6 Défaillances techniques | Annulation après 3×31 ms sans ack | Non simulable ; pire cas = T2 non ouvert → −13,64$ de gain manqué, capital intact | borné | [DÉGRADE]
| D7 Échelle | Mécanisme viable à 5$ | 44 fills ≥ 100$ observés, max 614,97$ (B-F3 B4) ; profondeur au-delà : DONNÉE NON DISPONIBLE | extinction bornée entre ~100$ et 615$ par ordre | [DÉGRADE]
| D8 Dépendance session | Le seuil ±4$ trade toute session | Sur session calme (aucun Δ≥|4$|) : 0 trade, capital 5,00$ intact ; MAIS 100% du P&L positif vient d'UN trade porté par l'issue UP | +13,64$ sur 1 seul événement | [DÉGRADE]
| D9 Conformité nouveau Polymarket | Indifférente au mode de résolution | TWAP 270–300 s = 64 275,56$ vs dernier tick 64 277,08$ : écart 1,52$, tous deux > strike+50$ → même issue (B-F1 B4 ; P0-3) | 0,00$ sur cette session | [SANS IMPACT]


**VERDICT GLOBAL : [FRAGILE].** Règles mécaniques : P&L net réel positif (+13,18$), zéro [DÉTRUIT], conformité P0 totale — mais D8 révèle une dépendance du gain à un événement unique de session, ce qui interdit [FIABLE]. Étiquette : **[SESSION-SPÉCIFIQUE]** — le mécanisme (oracle plus rapide que le carnet de 6,761 s, boucle 130 ms) est un [PATTERN CANDIDAT] à re-tester, mais le P&L de cette session ne l'est pas.

# AUTO-CONTRÔLE FINAL

- Contrôle 1 → P0-1 à P0-4 sourcés ; changement récent identifié (TWAP 30 s, 07/08/2026) ; 3 points marqués DONNÉE NON VÉRIFIABLE avec hypothèses conservatrices.
- Contrôle 2 → 5 valeurs min par graphe (A-G1 à A-G6 : 5 à 6 chacun) ; 8+ valeurs par CSV (B-F1 : 9, B-F2 : 8, B-F3 : 8, B-F4 : 8).
- Contrôle 3 → chaque règle de C1 référence OBS-1/2/3, B-F1/B-F2/B-F3 ou P0 ; 2 règles marquées [FRAGILE - 1 SOURCE].
- Contrôle 4 → boucle reconstituée chemin par chemin (68+31+31=130 ms) et appliquée aux 2 entrées du journal (00:48.130 ; 02:16.260 avec règle c).
- Contrôle 5 → cash ligne à ligne : 5,00000 − 4,24704 = 0,75296 ; +3,78802 = 4,54098 ; −4,36315 = 0,17783 ; +18,00000 = 18,17783$ = capital final annoncé, au centime près.
- Contrôle 6 → D1–D9 traités ; les 3 [SANS IMPACT] portent chacun leur preuve chiffrée (0,00$ mesuré ; 6,5% ; 1,52$) ; verdict [FRAGILE] conforme aux règles mécaniques.
- Contrôle 7 → tous les chiffres de la synthèse (18,18$ ; +263,56% ; 2 trades 1/1 ; 0,33$ ; TWAP 30 s ; 130 ms) réapparaissent à l'identique dans le corps.


# CLÔTURE

La stratégie exploite un retard mécanique du carnet sur l'oracle de résolution, et elle survit aux frais, aux latences et au nouveau régime TWAP. Mais son gain repose sur un seul trade d'une session volatile : elle mérite une campagne de validation multi-sessions, pas un déploiement. Vous avez toutes les pièces pour reproduire chaque ligne du calcul.


