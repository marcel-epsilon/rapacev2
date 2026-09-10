# 11 — DIVERGENCE ORACLE PERSISTANTE

**Fichiers sources regroupés :** 296DOP-40 — Divergence_Oracle_Persistante.md

---

## Source : 296DOP-40 — Divergence_Oracle_Persistante.md

# SYNTHÈSE EXÉCUTIVE

Stratégie retenue : **DOP-40 — Divergence Oracle Persistante** (achat du côté confirmé par le TWAP Chainlink 30 s quand il s'écarte de plus de 40,00$ du strike).
Capital final vs 5,00$ : **0,00$ (−99,97 %)** sous le résultat officiel de la session (▲ UP, PDF) ; **6,09$ (+21,83 %)** uniquement dans le monde décrit par les CSV — les deux jeux de pièces se contredisent sur l'issue.
Trades : **1** (0 gagnant / 1 perdant sous le résultat officiel ; 1 gagnant / 0 perdant sous les CSV).
Changement Polymarket le plus impactant : depuis début août 2026, les marchés crypto Up/Down ne se résolvent plus sur un tick unique mais sur un **TWAP Chainlink 30 s** (ouverture et clôture), consommable via RTDS lancé le 4 août 2026 (docs.polymarket.com/market-data/chainlink-twap).
VERDICT DE FIABILITÉ : **[NON FIABLE]**.
Étiquette : **[SESSION-SPÉCIFIQUE]**.
Raison en une phrase : le PDF annonce « Résultat : ▲ UP » avec strike 65 251$ alors que les quatre CSV décrivent une session intégralement baissière (spot 65 009,23 → 64 857,91$ ; oracle 64 962,89 → 64 816,34$ ; NO → 0,99$) — aucune stratégie ne peut être déclarée fiable sur des pièces mutuellement incompatibles.

---

# 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET (VÉRIFIÉ)

Voici l'état documenté en vigueur au 09/08/2026, chaque règle devenant contrainte dure pour la suite.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × 0,07 × p × (1−p) ; pic 1,75$ / 100 parts à p=0,50 ; arrondi 5 décimales, min 0,00001 USDC | docs.polymarket.com/trading/fees | consultée 09/08/2026 | Oui — frais étendus à tous les marchés crypto depuis le 06/03/2026
| P0-1 Frais maker | 0 ; rebate 20 % des frais taker crypto, payé quotidiennement | docs.polymarket.com/trading/fees + /programs/maker-rebates | consultée 09/08/2026 | Non
| P0-1 Gas / dépôt / retrait | 0$ pour le trader (règlement on-chain soumis par l'opérateur ; pas de frais de dépôt/retrait USDC) | docs.polymarket.com/trading/fees + /trading/orders | consultée 09/08/2026 | Non
| P0-2 Types d'ordres | Tous limit ; GTC, GTD, FOK, FAK (FOK/FAK = ordres market) ; post-only rejeté s'il croise | docs.polymarket.com/trading/orders | consultée 09/08/2026 | Non
| P0-2 Tick size | 0,01$ pour ces marchés (grille observée dans bbo.csv, 2 décimales) ; à vérifier par `getTickSize` avant de coter | docs.polymarket.com/trading/orders | consultée 09/08/2026 | Non
| P0-2 Taille minimale | DONNÉE NON VÉRIFIABLE (champ `min_order_size` par marché) — hypothèse conservatrice : non bloquante, trades.csv contient des trades de 0,0058$ | docs.polymarket.com/changelog/predictions.md | consultée 09/08/2026 | Non
| P0-2 Matching | CLOB V2 (live sur clob.polymarket.com), pipeline async depuis le 24/07/2026 : plus de `transactionHashes` inline, `tradeIDs` à poller ; heartbeat obligatoire sinon annulation des ordres sous 10–15 s | docs.polymarket.com/changelog/predictions.md + /trading/orders | 24/07/2026 | **Oui**
| P0-3 Résolution crypto 5 min | **TWAP Chainlink 30 s** pour la clôture ET le « Price to Beat » (plus de snapshot au dernier tick) ; flux via Chainlink Data Streams ou Polymarket RTDS (topic `crypto_prices_twap_thirty`) | docs.polymarket.com/market-data/chainlink-twap | RTDS lancé le 04/08/2026 | **Oui — c'est LE changement récent**
| P0-4 Rate limits IP | CLOB général 9 000 req/10 s ; `/book` 1 500/10 s ; POST /order 5 000/10 s burst, 120 000/10 min | docs.polymarket.com/api-reference/rate-limits | consultée 09/08/2026 | Oui (relevés)
| P0-4 Rate limits par signeur | Tier Standard : 40 ordres/s (burst 60), 80 annulations/s (burst 120) ; mode warning depuis le 24/07/2026, application stricte annoncée ≈ 2 semaines après (fenêtre de la session) | docs.polymarket.com/api-reference/trading-rate-limits | 24/07/2026 | **Oui**


**Ce qui a changé et ce que cela implique pour une session de 5 minutes :** la résolution ne dépend plus du dernier tick ≤ t_fin mais de la moyenne pondérée des 30 dernières secondes. Conséquence directe : (i) toute stratégie de « sniping » du dernier tick est morte ; (ii) le titre du Graphique 2 du PDF — « strike exact (dernier tick ≤ t0) » — applique la **méthodologie obsolète** ; (iii) les 30 dernières secondes deviennent partiellement prévisibles (un TWAP entamé se déplace de moins en moins), ce qui écrase l'incertitude finale plus tôt.

---

# 2. CADRE D'ANALYSE

J'analyse chaque pièce isolément, en n'extrayant que des faits chiffrés horodatés, avant toute idée de stratégie. Les temps sont en ms depuis l'ouverture (t_ms), les prix de contrats en $, BTC en $. Tout écart entre PDF et CSV est traité comme une anomalie de première classe, pas comme un détail.



3. PARTIE A — LES 6 GRAPHES DU PDF
A-G1 — CARNET YES/NO COMPLET, VUE 300 s
A1. Axe X : temps mm:ss.mmm sur [00:00 ; 05:00] ; axe Y gauche : prix contrat 0–1$ ; axe Y droit : probabilité 0–1. Titre : « Bitcoin Up or Down - August 7, 9:40AM-9:45AM ET | Résultat : ▲ UP », 2 267 événements BBO, 92 incohérents signalés. A2. Le marché a-t-il convergé progressivement vers l'issue, ou par ruptures ? A3. Lecture des annotations d'extrema imprimées et des six courbes bid/ask/mid YES et NO. A4. t=00:00.247 : 0,46$ (YES) et 0,55$ (NO) ; t=04:10.938 : 0,78$ et 0,23$ ; t=04:59.562 : 0,01$ et 0,99$ ; annotation « incohérents : 92 » ; annotation « ts fallback : 0/282 (0,0 %) ». A5. (a) Le marché ouvre quasi 50/50 (0,46/0,55). (b) Une série culmine à 0,78$ à 250,938 s puis s'effondre à 0,01$ à 299,562 s pendant que l'autre finit à 0,99$ : renversement complet dans les 49 dernières secondes. (c) 92 événements de carnet croisé sur 2 267 (4,1 %). A6. SPÉCIFIQUE À LA SESSION : un flip 0,78 → 0,01 en fin de fenêtre est un scénario de photo-finish ; rien dans la vue 300 s ne le rendait prévisible avant 04:10.

A-G2 — CARNET YES/NO, ZOOM 15 s [04:23 → 04:38]
A1. Mêmes axes, fenêtre 15 s, 317 étiquettes de valeurs lisibles par série. A2. À quelle vitesse le carnet absorbe-t-il un retournement du spot ? A3. Lecture séquentielle des étiquettes horodatées à la ms. A4. t=04:28.094 : 0,07$ / 0,94$ (extrême bas de la série faible) ; t=04:31.058 : 0,12$ ; t=04:31.345 : 0,62$ ; t=04:31.703 : 0,55$ ; t=04:33.000 : 0,50$ / 0,50$ ; t=04:37.512 : 0,27$ / 0,73$. A5. Le passage 0,07$ → 0,62$ s'exécute entre 04:31.058 et 04:31.345, soit 287 ms pour +0,55$ (55 mises à jour listées dans cette seule seconde). Puis retour à 0,27$ à 04:37.512 : le marché n'a pas conclu. A6. SPÉCIFIQUE : ce spike de 287 ms coïncide avec le retour du spot au-dessus du strike (A-G5 : spot 65 226,54 → 65 236,61$ entre 04:30 et 04:31). Durée de vie du signal exploitable : < 300 ms.

A-G3 — CARNET YES/NO, ZOOMS [03:19 → 03:34] ET [04:45 → 05:00]
A1. Mêmes axes, deux fenêtres de 15 s. A2. Les épisodes intermédiaires ont-ils la même signature que le final ? A4. Zoom 03:19 : t=03:20.563 : 0,46$ (18 mises à jour au même timestamp) ; t=03:26.519 : 0,34$ vs NO 0,66$ ; t=03:27.815 : 0,23$/0,77$. Zoom final : t=04:45.401 : 0,05$/0,95$ ; t=04:58.494 : 0,57$/0,43$ ; t=04:59.562 : 0,01$/0,99$. A5. (a) Rafales de 10–18 mises à jour au même timestamp ms (03:20.563, 04:33.214) : purges de niveaux en un seul événement de matching. (b) Dans les 75 dernières secondes : 0,05 → 0,57 → 0,01, soit deux renversements complets, le dernier 438 ms avant la clôture. A6. SPÉCIFIQUE : une fin à 0,01$ sur un marché résolu ▲ UP signifie que le carnet s'est trompé de côté à 99 % dans la dernière seconde — incohérence majeure interne au PDF, ou preuve que la série finissant à 0,99$ est le YES et que l'étiquetage des couleurs s'inverse entre annotations.

A-G4 — SPOT BINANCE vs ORACLE CHAINLINK, VUE 300 s (+ basis)
A1. Axe Y : prix BTC 65 150–65 325$ ; 29 628 ticks Binance, 282 ticks Chainlink ; strike affiché 65 251$ (« dernier tick ≤ t0 ») ; sous-graphe basis = spot − oracle, échelle 0–60$. A2. Le spot a-t-il fini au-dessus du strike, et avec quelle marge ? A4. t=00:00.299 : 65 299,70$ (spot) ; t=00:18.032 : 65 321,77$ (max) ; t=02:49.892 : 65 184,00$ (min) ; strike : 65 251$ ; oracle t=00:00 : 65 251,74$ ; oracle t=04:59 : 65 261,61$ ; spot t=04:59.683 : 65 308,35$. A5. (a) L'oracle finit à +9,87$ au-dessus du strike (65 261,61 − 65 251,74) : victoire UP à 0,015 % près. (b) Le strike est traversé au moins 6 fois (01:01, 01:37, 02:02, 03:05, 03:47, 04:31). (c) Le basis reste dans [0 ; 60$]. A6. SPÉCIFIQUE : marge finale de 9,87$ sur un actif qui bouge de 137,77$ dans la session (65 321,77 − 65 184,00) — issue quasi aléatoire. Et la méthodologie « dernier tick ≤ t0 » du titre est celle d'avant le changement P0-3.

A-G5 — SPOT vs ORACLE, ZOOMS 15 s [04:23], [03:19], [04:45]
A1. Mêmes axes, fenêtres 15 s. A2. Qui mène, spot ou oracle, aux moments des flips du carnet ? A4. t=04:28.546 : 65 271,50$ (Binance) ; t=04:30.000 : 65 226,54$ (oracle) ; t=04:31.000 : 65 236,61$ ; t=04:31.639 : 65 300,00$ (Binance) ; t=04:55.687 : 65 313,99$ ; t=04:59.683 : 65 308,35$. A5. Entre 04:30 et 04:32, l'oracle remonte de 65 226,54 à 65 248,84$ (+22,30$) pendant que Binance imprime 65 300,00$ à 04:31.639 : le spot Binance mène l'oracle d'au moins 1 s sur ce mouvement, et le flip du carnet (A-G2, 04:31.058–345) se produit pendant le mouvement Binance, pas après l'oracle. A6. SPÉCIFIQUE : le carnet a réagi plus vite que l'oracle 1 Hz — les teneurs de marché lisent Binance en direct ; avec 219 ms de latence Binance, vous arrivez systématiquement derrière eux.

A-G6 — FLUX DIRECTIONNEL NORMALISÉ (3 847 trades), VUE 300 s + ZOOMS
A1. Axe Y : volume $ par fenêtre 30 s, haussier positif / baissier négatif, échelle −8 000 à +4 000$. A2. Le flux agrégé a-t-il anticipé l'issue ? A4. t=00:01.705 : +23,00$ ; t=03:25.526 : +197,58$ (extremum haussier) ; t=04:59.864 : −1 148,42$ (extremum baissier, dernier point) ; zoom 03:19 : t=03:21.006 : −1 143,95$ ; t=03:21.376 : −543,73$ ; zoom final : t=04:47.699 : −209,93$ ; t=04:49.279 : −100,34$. A5. (a) Le flux est majoritairement baissier y compris dans la dernière minute (−1 148,42$ à 04:59.864). (b) Les plus gros prints unitaires (−1 143,95$) sont baissiers. (c) Le flux n'a jamais imprimé d'extremum haussier supérieur à +197,58$. A6. SPÉCIFIQUE : sur un marché résolu ▲ UP, le flux dominant a payé le mauvais côté jusqu'à la dernière seconde — suivre le flux agrégé aurait perdu ici.

4. PARTIE B — LES 4 CSV
B-F1 — bbo.csv
B1. Colonnes : t_ms, yes_bid, yes_ask, no_bid, no_ask. 1 357 lignes de données parsées, période 217 → 250 016 ms, cadence événementielle (médiane inter-mise-à-jour 24 ms). B2. Apport : granularité ms du carnet, absente des graphes ; permet de mesurer durées de vie des états croisés. B3. Calculs : spread = yes_ask − yes_bid ; croisement = yes_ask + no_ask < 1 ; mid = (bid+ask)/2 ; durée de vie = t(mise à jour suivante) − t(état). B4.

Métrique	Valeur	Colonnes source	Nb lignes utilisées
Lignes / période couverte	1 357 / 0,217–250,016 s	t_ms	1 357
Spread YES moyen / médian	0,0179$ / 0,02$	yes_bid, yes_ask	1 357
Spread max / min	0,07$ / −0,03$	yes_bid, yes_ask	1 357
Mid YES max	0,675$ à t=40 202	yes_bid, yes_ask	1 357
Mid YES min	0,01$ à t=250 016	yes_bid, yes_ask	1 357
États croisés (arb acheteur)	6, gap max 0,03$ à t=62 476	4 colonnes	1 357
Durée de vie des croisements	8–28 ms (5/6), 1 426 ms (1/6, gap 0,01$)	t_ms	6
Inter-mise-à-jour p25/p75/p90	8 / 144 / 697 ms ; max 8 203 ms	t_ms	1 161 gaps
B5. Le NO passe de 0,52$ (t=217) à 0,99$ (t=250 016) de manière quasi monotone après t=150 s ; à t=241 813, yes_bid=0,01 / no_ask=0,99 déjà. B6. ANOMALIES : (i) le fichier s'arrête à 250,016 s — les 50 dernières secondes (celles du flip du PDF) sont absentes ; (ii) 1 357 lignes vs 2 267 événements annoncés par le PDF ; (iii) 6 croisements vs 92 « incohérents » annoncés ; (iv) à t=250 016, YES=0,01$ alors que le PDF affiche 0,78$ à t=250 938 — irréconciliable.

B-F2 — trades.csv
B1. Colonnes : t_ms, usd, direction (+1 haussier / −1 baissier). 3 010 lignes, période 328 → 293 522 ms. B2. Apport : intensité et signe du flux à la trade près. B3. Sommes par direction, buckets 30 s, quantiles de taille. B4.

Métrique	Valeur	Colonnes source	Nb lignes utilisées
Volume total	44 977,01$	usd	3 010
Volume haussier / baissier	18 045,11$ / 26 931,90$	usd, direction	3 010
Flux net session	−8 886,79$	usd, direction	3 010
Nb trades +1 / −1	1 248 / 1 762	direction	3 010
Taille médiane / moyenne / max	4,07$ / 14,94$ / 1 201,56$	usd	3 010
Pire bucket 30 s	[120–150 s] : net −4 113,32$	usd, direction	bucket
Trades ≥ 200$	18 (max +1 201,56$ à t=239 075)	usd	18
Volume après 250 s	453,60$ dont 357,90$ haussier (42 trades)	usd, direction	42
B5. Le flux net est baissier dans 6 buckets sur 10 ; il bascule haussier après 250 s (357,90$ sur 453,60$), en cohérence interne avec un carnet déjà à NO=0,99 (plus rien à vendre côté baissier). B6. ANOMALIES : 3 010 trades vs 3 847 annoncés par le PDF (837 manquants) ; le PDF liste des dizaines de trades baissiers dans [04:45–05:00] (−115,20$ à 04:45.246, etc.) alors que le CSV n'en contient aucun baissier après 270 s — deuxième contradiction directe ; timestamps localement non monotones (t=4 854 avant t=4 839).

B-F3 — oracle.csv
B1. Colonnes : t_ms, price, ts_src. 285 lignes, période 0 → 298 000 ms, cadence 1 000 ms nominale (moyenne 1 049,3 ms), 0 fallback. B2. Apport : la série de résolution (Chainlink) tick par tick. B3. Strike proxy = premier tick (t=0) = 64 962,89$ ; TWAP30(t) = moyenne des ticks sur ]t−30 000 ; t]. B4.

Métrique	Valeur	Colonnes source	Nb lignes utilisées
Premier / dernier tick	64 962,89$ / 64 816,34$	price	2
Min / Max	64 807,68$ (t=255 000) / 64 987,63$ (t=52 000)	price	285
Ticks au-dessus du strike-proxy	0 / 285	price	285
Gaps > 1,5 s	14 (max 2 000 ms)	t_ms	284
TWAP30(120 000) − strike	−33,78$	price	30
TWAP30(150 000) − strike	−79,06$	price	30
TWAP30(270 000) − strike	−130,51$	price	29
TWAP30(298 000) − strike	−141,59$	price	28
B5. Sous la règle P0-3 (TWAP 30 s), la clôture CSV vaut 64 821,30$ (TWAP30 à 298 s) contre un « price to beat » proxy de 64 962,89$ : résolution DOWN, marge −141,59$ — sans la moindre ambiguïté. B6. ANOMALIES : 285 ticks vs 282 annoncés par le PDF ; surtout, la série PDF (65 251,74 → 65 261,61$, résolution UP à +9,87$) et la série CSV (64 962,89 → 64 816,34$, résolution DOWN à −141,59$) ne peuvent pas décrire le même marché : ni offset constant (écart 288,85$ à t=0, 445,89$ à t=298 s), ni même signe de tendance.

B-F4 — spot.csv
B1. Colonnes : t_ms, price. 38 228 lignes, période 118 → 299 662 ms, 127,6 ticks/s. B2. Apport : le flux le plus rapide, celui que lisent les teneurs de marché. B3. Basis = spot(t) − oracle(t) échantillonné à chaque tick oracle ; détection de mouvements ≥ 8$/s. B4.

Métrique	Valeur	Colonnes source	Nb lignes utilisées
Premier / dernier tick	65 009,23$ / 64 857,91$	price	2
Min / Max	64 844,07$ (t=253 891) / 65 041,99$ (t=39 945)	price	38 228
Basis spot−oracle moyen	+45,56$	price ×2 fichiers	285
Basis min / max	+22,82$ / +61,82$	price ×2 fichiers	285
Densité	127,6 ticks/s	t_ms	38 228
Mouvements ≥ 8$ en ≤ 1 s (dédupliqués 2 s)	47	price	38 228
Suivis par le mid BBO dans les 2 s	32 (68 %)	+ bbo.csv	47
Délai médian 1ʳᵉ requote BBO après signal	234 ms	+ bbo.csv	47
B5. Le spot CSV termine à −151,32$ sous son ouverture : cohérent avec oracle.csv et bbo.csv (les trois CSV racontent la même session baissière), incohérent avec le PDF. B6. ANOMALIES : 38 228 ticks vs 29 628 annoncés par le PDF ; jusqu'à 90 ticks partageant le même t_ms ; le basis CSV (+22,82 à +61,82$) est compatible avec l'échelle 0–60$ du sous-graphe PDF, seul point de concordance PDF↔CSV.

5. PARTIE C — STRATÉGIES ET SIMULATION
C0. CANDIDATES
Mes cinq observations les plus fortes :

OBS-1 — La divergence TWAP30 oracle−strike est persistante et croissante : −33,78$ à t=120 s, −79,06$ à 150 s, −130,51$ à 270 s (B4-F3) ; l'oracle n'a jamais coté au-dessus du strike-proxy (0/285, B4-F3).
OBS-2 — Le NO monte de 0,52$ à 0,99$ sur 250 s (B5-F1) porté par un flux net −8 886,79$ (B4-F2) : la convergence carnet↔oracle est lente à l'échelle de la minute.
OBS-3 — 68 % des mouvements spot ≥ 8$/s sont suivis par le mid dans les 2 s, mais la première requote du carnet arrive en médiane 234 ms après le signal (B4-F4).
OBS-4 — Les états croisés du carnet vivent 8–28 ms (5/6) avec un gap ≤ 0,03$ (B4-F1).
OBS-5 — Le PDF (▲ UP, +9,87$) et les CSV (DOWN, −141,59$) sont mutuellement incompatibles (B6-F3, A6-G1) ; le flux dominant a payé le côté perdant du PDF (A5-G6).
CRITÈRE DE SÉLECTION : P&L net réel maximal (après latences du tableau, slippage +1 tick par règle, frais P0-1) sur les données CSV, sous réserve de conformité P0 totale et de faisabilité structurelle. Annoncé avant toute comparaison.

Candidates (familles distinctes) :

C-A — Chasse de latence Binance→CLOB (OBS-3) : acheter le côté du mouvement spot ≥ 8$/s en FAK avant la requote. Boucle : 219 ms (âge signal Binance) + 5 ms décision [HYPOTHÈSE n°1] + 31 ms envoi CLOB = 255 ms, + 31 ms confirmation. Durée de vie du signal : la requote adverse arrive en médiane à 234 ms (B4-F4), soit avant mon ordre. [STRUCTURELLEMENT IMPOSSIBLE] — éliminée.
C-B — DOP-40, Divergence Oracle Persistante (OBS-1, OBS-2) : si TWAP30(oracle) s'écarte de plus de 40,00$ du strike après t=60 s, acheter en FAK le côté confirmé (NO si dessous) tant que son ask ≤ 0,90$, et porter jusqu'à résolution. Boucle : 68 (âge oracle) + 5 (décision) + 31 (envoi) = 104 ms, + 31 confirmation = 135 ms. Durée de vie du signal : des dizaines de secondes (OBS-1) ≫ 135 ms. Conforme P0 (FAK autorisé P0-2 ; frais P0-1 intégrés ; TWAP = la règle P0-3 elle-même). Retenue pour comparaison.
C-C — Market-making post-only (OBS-2, spread médian 0,02$) : coter les deux côtés, 0 frais maker + rebate 20 % (P0-1). Structurellement faisable (boucle 62 ms). Mais borne de gain : ≤ 10 rotations × 0,02$ × 6 parts = +1,20$ brut maximum, contre 47 événements d'anti-sélection (OBS-3) coûtant en borne basse 0,05$ × 6 parts = 0,30$ chacun s'ils me frappent — avec 5,00$ de capital, un seul inventaire adverse porté à la résolution coûte jusqu'à −5,00$. Net borné négatif en espérance. Éliminée au critère.
**C-D — Arbitrage carnet croisé YES+NO < 1** (OBS-4) : boucle minimale 31 (âge BBO) + 5 + 31 = 67 ms > durée de vie 8–28 ms pour 5 occurrences sur 6 : [STRUCTURELLEMENT IMPOSSIBLE] sur celles-ci. L'unique occurrence longue (1 426 ms, t=175 069, gap 0,01$) est tuée par P0-1 : frais taker des deux jambes à p≈0,5 = 2 × 0,0175 = 0,035$/part > gap 0,01$/part. Éliminée.
Stratégie	Obs. sources	Conforme P0 ?	Nb trades	P&L brut	P&L net réel	Verdict
C-A Chasse de latence	OBS-3	Oui	—	—	—	[STRUCTURELLEMENT IMPOSSIBLE]
C-B DOP-40	OBS-1, OBS-2	Oui	1	+1,25$	+1,09$ (monde CSV)	RETENUE
C-C Market-making	OBS-2	Oui	≤ 20	≤ +1,20$	borné < 0 (anti-sélection)	Éliminée au critère
C-D Arb croisé	OBS-4	Non (frais > gap)	0–1	+0,06$	−0,15$	Éliminée
Décision : C-B DOP-40, seule candidate structurellement possible à P&L net réel positif.

C1. STRATÉGIE RETENUE — DOP-40
ENTRÉES DE CONFIG
  STRIKE      = price-to-beat TWAP30 publié à t0            # P0-3
                [HYPOTHÈSE n°2 : indisponible pré-t0 dans les données,
                 proxy = premier tick oracle = 64 962,89$ ;
                 écart proxy↔TWAP30(0..30s) mesuré = 12,12$ < SEUIL]
  SEUIL       = 40,00$        # > 2× l'écart proxy ci-dessus, et > tout
                              # aller-retour TWAP30 observé avant 120 s (max 13,66$, B4-F3)
  PRIX_MAX    = 0,90$         # au-delà, gain < frais+risque   # P0-1
  T_MIN       = 60 000 ms     # évite le bruit d'ouverture (TWAP30 dev ±13,66$, B4-F3)

BOUCLE (à chaque tick oracle, âge 68 ms)                      # tableau latences
  dev = TWAP30(oracle) − STRIKE                               # OBS-1
  si |dev| > SEUIL et t > T_MIN et position = ∅ et aucun ordre en cours :
      côté = NO si dev < 0 sinon YES
      ask  = meilleur ask(côté) au moment de l'arrivée (t_signal + 104 ms)
      si ask ≤ PRIX_MAX :
          envoyer FAK  achat(côté, prix = ask + 0,01,          # règle slippage +1 tick
                             budget = cash × (prix + 0,07·p·(1−p))⁻¹)  # r5, P0-1
  position portée jusqu'à résolution (pas de sortie anticipée)
      [FRAGILE - 1 SOURCE : la persistance de la divergence n'est
       observée que sur cette session]
C-SIM. SIMULATION EN CONDITIONS RÉELLES
ÉTAT DU PORTEFEUILLE. Cash initial 5,0000$, aucune position, aucun ordre.

RACE CONDITIONS.

(a) RÈGLE : tout signal d'achat survenant pendant qu'un ordre est en cours (statut ≠ CONFIRMED/FAILED) est ignoré, pas mis en file.
(b) RÈGLE : aucune vente tant que le trade d'entrée n'est pas CONFIRMED (statuts P0-2 : MATCHED→MINED→CONFIRMED).
(c) RÈGLE : deux signaux simultanés (YES et NO impossibles ensemble par construction ; deux ticks oracle le même cycle) → seul le plus récent est traité.
(d) RÈGLE : fill partiel FAK → la quantité exécutée est conservée, le reliquat est annulé par le FAK lui-même, aucune re-chasse.
JOURNAL DE TRADES (latence appliquée : signal oracle âgé de 68 ms + 5 ms décision + 31 ms envoi = exécution à t_signal + 104 ms ; confirmation à +135 ms) :

#	Ts entrée signal	Ts exécution réelle	Prix entrée	Quantité	Ts sortie	Prix sortie	Slippage	Frais	P&L net	Cash après
1	124 000 ms (TWAP30=64 922,01$, dev=−40,88$)	124 104 ms (quote en vigueur : no_ask 0,80$, inchangée depuis 122 336 ms)	0,81$ (NO)	6,09 parts	résolution (300 000 ms)	1,00$ si DOWN / 0,00$ si UP	+0,01$ (1 tick)	0,0656$	+1,0915$ (CSV) / −4,9985$ (résultat officiel UP)	0,0015$ après entrée
Vérification r5 : débit total = 6,09 × 0,81 + 0,0656 = 4,9985$ ≤ 5,0000$. Aucun autre signal traité (position ouverte, règle (a)).

RÉSULTATS.

Monde CSV (résolution DOWN, TWAP30 clôture 64 821,30$ < 64 962,89$) : 1 trade, 1 gagnant / 0 perdant ; frais totaux 0,0656$ ; bénéfice net +1,0915$ ; capital final 6,0915$ (+21,83 %) ; pire perte réalisée 0 ; drawdown max latent −1,8622$ (−37,24 %) à t=205 978 ms (mid NO retombé à 0,515$).
Résultat officiel du PDF (▲ UP) : le même trade paie 0,00$ ; capital final 0,0015$ (−99,97 %).
P&L THÉORIQUE sans r1–r4 (entrée à 0,80$, sans frais ni slippage) : capital final 6,2500$ (+25,00 %). Coût du réalisme : 0,1585$, soit 12,7 % du gain brut.
6. PARTIE D — VERDICT DE FIABILITÉ
Facteur	Hypothèse de la stratégie	Réalité (source)	Impact P&L	Verdict
D1 Latence de boucle	135 ms n'altère pas le prix	Quote inchangée 122 336→124 104 ms, soit 1 768 ms > 135 ms (B-F1)	0,00$ sur ce trade	[SANS IMPACT] (prouvé)
D2 Fraîcheur du signal	Un tick oracle âgé ≤ 1 068 ms (cadence 1 s + 68 ms) reste valide	Plus grand saut oracle en 1 s : 18,65$ (t=251→252 s, B-F3) < marge 40,88$ à l'entrée	0,00$	[SANS IMPACT] (prouvé)
D3 Slippage / profondeur	+1 tick suffit	bbo.csv ne contient aucune taille ; profondeur DONNÉE NON DISPONIBLE ; règle +1 tick appliquée	−0,0761$ vs théorique (part du coût du réalisme)	[DÉGRADE]
D4 Frais complets + gas	fee = 6,09×0,07×0,81×0,19	0,0656$ (P0-1) ; gas 0$ (P0-1)	−0,0656$ (1,31 % du capital)	[DÉGRADE]
D5 Niveau disparu / fill partiel	FAK absorbe	Quote stable 1 768 ms autour de l'exécution (B-F1) ; règle (d) définie	0,00$ sur ce trade	[SANS IMPACT] (prouvé)
D6 Défaillances techniques	Monitoring continu	bbo.csv s'arrête à 250,016 s : cécité de 50 s (B6-F1) ; heartbeat obligatoire sous 10–15 s (P0-2) sinon annulation — sans effet sur une position déjà remplie, mais aucune sortie d'urgence possible	non chiffrable, borné par la perte max déjà comptée (−4,9985$)	[DÉGRADE]
D7 Passage à l'échelle	Réplicable en taille	Volume total 44 977$/300 s, trade médian 4,07$, 18 trades ≥ 200$ (B4-F2) ; une entrée > 10² $ consommerait plusieurs niveaux d'un carnet dont la profondeur est inconnue	extinction estimée dès ~100$ d'ordre (borne : 2,2 % du volume par bucket 30 s)	[DÉGRADE]
D8 Dépendance à la session	La divergence prédit l'issue	Session calme :	dev	< 40$ → 0 trade, P&L 0,00$. MAIS sur la session telle que documentée par le PDF (▲ UP, oracle finissant à +9,87$ du strike après être passé dessous), la même règle achète NO et rend 0,00$ : −4,9985$	−4,9985$	[DÉTRUIT]
D9 Conformité nouveau fonctionnement	Résolution TWAP 30 s	La stratégie lit précisément le TWAP30 (P0-3) : conforme ; le strike-proxy diffère du price-to-beat TWAP de 12,12$ mesurés (B4-F3) < seuil 40$	risque de faux signal borné à 12,12$/40,00$ de la marge	[DÉGRADE]
VERDICT GLOBAL (règles mécaniques). D8 est [DÉTRUIT] et non corrigé en C-SIM → au mieux [FRAGILE]. De plus, le seul résultat officiel documenté de la session (▲ UP, PDF) rend le P&L net réel du journal négatif (−4,9985$) → règle « P&L net réel négatif → [NON FIABLE] ». Le verdict est donc [NON FIABLE], et l'étiquette [SESSION-SPÉCIFIQUE] : le gain de +21,83 % n'existe que dans le monde des CSV, lequel contredit le PDF sur 5 points quantifiés (comptes d'événements 1 357/2 267, 3 010/3 847, 285/282, 38 228/29 628 ; niveaux de prix ; sens de la résolution). Tant que la chaîne de capture des données n'est pas fiabilisée, aucun capital — même 5,00$ — ne doit être engagé.

7. AUTO-CONTRÔLE FINAL
Contrôle 1 (Phase 0 sourcée) → P0-1 à P0-4 sourcés avec URL ; changement récent identifié : résolution TWAP Chainlink 30 s, RTDS 04/08/2026 ; taille minimale d'ordre marquée DONNÉE NON VÉRIFIABLE avec hypothèse conservatrice. ✔
Contrôle 2 (≥ 5 valeurs par graphe, ≥ 8 par CSV) → A-G1 : 5 ; A-G2 : 6 ; A-G3 : 6 ; A-G4 : 7 ; A-G5 : 6 ; A-G6 : 7 ; B4-F1 à B4-F4 : 8 chacun. ✔
Contrôle 3 (traçabilité des règles) → SEUIL ← B4-F3/OBS-1 ; PRIX_MAX ← P0-1 ; T_MIN ← B4-F3 ; côté ← OBS-1 ; portage ← OBS-2 marqué [FRAGILE - 1 SOURCE]. ✔
Contrôle 4 (latence reconstituée) → 68 ms (oracle) + 5 ms (décision, [HYPOTHÈSE n°1]) + 31 ms (envoi CLOB) = 104 ms appliqués à l'unique entrée du journal (124 000 → 124 104 ms) ; confirmation +31 ms = 135 ms. ✔
Contrôle 5 (comptabilité) → 5,0000 − 4,9329 (coût) − 0,0656 (frais) = 0,0015$ ; +6,09 (payout CSV) = 6,0915$ ; sous UP : 0,0015$. Au centime près. ✔
Contrôle 6 (D1–D9) → 9 facteurs traités ; les trois [SANS IMPACT] portent chacun leur preuve chiffrée (1 768 ms > 135 ms ; 18,65$ < 40,88$ ; quote stable 1 768 ms) ; verdict global conforme aux règles mécaniques (D8 [DÉTRUIT] + P&L officiel négatif → [NON FIABLE]). ✔
Contrôle 7 (cohérence des chiffres) → 6,0915$ / +21,83 % / 0,0015$ / −99,97 % / 0,0656$ / 1 trade / TWAP 30 s identiques dans l'ouverture, le corps et les tableaux ; aucun chiffre nouveau en clôture. ✔
8. CLÔTURE
Vous avez sous les yeux les mêmes pièces que moi : elles ne racontent pas la même session, et la stratégie la mieux classée ne survit pas au résultat officiel imprimé sur le PDF. Ma recommandation est de ne pas engager le capital et de fiabiliser d'abord la chaîne de capture, en particulier le flux de carnet et la série oracle. Je reste à disposition pour rejouer l'audit sur un jeu de données intègre.


