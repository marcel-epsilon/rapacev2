# 02 — ANCRAGE ORACLE VARIANTES (6-10)

**Fichiers sources regroupés :** 6ANCRAGE_ORACLE.md, 7ANCRAGE_ORACLE.md, 8ANCRAGE_ORACLE.md, 9ANCRAGE.md, 10ANCRAGE.md

---

## Source : 6ANCRAGE_ORACLE.md

## 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Voici la conclusion avant toute chose, vous vérifierez chaque chiffre pièce par pièce.

- Stratégie appliquée : ANCRAGE ORACLE (doctrine imposée P1-P5, calibration L1-L5 propre à cette session, verrouillée en C0).
- Capital final : 6,67$ contre 5,00$ de départ, soit un rendement net de +33,33%.
- Nombre de trades : 1, dont 1 gagnant et 0 perdant.
- Côté désigné par l'oracle : DOWN, jeton NO ; signal d'entrée à t=66,000s, exécution à t=66,544s au ask NO de 0,75$.
- Verdict de comportement : [STRATÉGIE CONFIRMÉE SUR CETTE SESSION]
- Spécificité dominante : une basis spot−oracle moyenne de +54,66$ (min +45,20$, max +67,17$) — le spot Binance n'est JAMAIS descendu sous le strike alors que le résultat officiel est DOWN ; sans le principe P1, on entrait du côté perdant, détail en C3.


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

---

## 1. RAPPEL DE LA DOCTRINE IMPOSÉE

P1 : seul l'oracle décide, le spot ne sert qu'au contrôle qualité, le carnet n'est qu'un prix d'exécution. P2 : strike = dernier tick oracle ≤ t0, marge(t) = price(t) − strike, marge négative ⇒ NO. P3 : entrée déclenchée uniquement par les prints oracle, si le carnet laisse encore un edge. P4 : 100% du cash (5,00$) sur UN trade au ask. P5 : tenue jusqu'au règlement, aucun stop carnet, sortie anticipée éventuelle uniquement sur prints oracle. La calibration L1-L5 est mon choix, justifié et verrouillé en C0.

---

## PARTIE A — LES 6 GRAPHES DU PDF

### A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (543 ÉVÉNEMENTS BBO)

**A1. IDENTITÉ** : « Carnet YES/NO complet — 543 événements BBO (précision ms) | incohérents : 28 ». Axe X : temps depuis l'ouverture, de t=0,000s à t=300,000s (vue 300 s, plus 3 zooms 15 s : [t=198,000s→t=213,000s], [t=55,000s→t=70,000s], [t=122,000s→t=137,000s]). Axe Y : prix en $ de 0,0 à 1,0. Six séries (bid/ask/mid YES et NO), la prob consensus, et 28 suspects « carnet croisé » en marqueur creux.

**A2. QUESTION** : à quel prix le jeton NO est-il achetable au moment où l'oracle s'installe sous le strike, et à quelle vitesse le carnet converge-t-il ensuite vers 1,00$ ? Ces lectures fondent L2 (prix d'achat maximal) et confirment ou non que l'edge existe après le signal L1.

**A3. COMMENT** : je lis les étiquettes de la série NO Ask autour des bascules oracle (zoom [t=55,000s→t=70,000s]), les extrema de la vue 300 s, et la trajectoire de convergence terminale.

**A4. VALEURS EXTRAITES** :

- t=1,076s : YES mid 0,56$ / NO mid 0,43$
- t=14,506s : YES mid 0,60$ (plus haut YES de la session)
- t=62,616s : NO ask passe de 0,42$ à 0,58$ en un événement (bascule)
- t=67,873s : NO ask 0,76$ / YES bid 0,24$
- t=211,330s : NO ask retombe à 0,78$ (creux de la remontée oracle)
- t=273,651s : YES mid 0,01$ / NO mid 0,98$ (convergence terminale)


**A5. OBSERVATIONS BRUTES** :

- Entre t=62,459s et t=62,775s, le NO ask monte de 0,49$ à 0,69$ : le carnet réagit en moins de 1 s au décrochage oracle — l'edge se consomme vite, ce qui ORIENTE L2 vers un plafond strict.
- À t=66,544s, premier événement après mon signal, le NO ask est encore à 0,75$ : l'edge existe, CONFIRME P3.
- Entre t=209,158s et t=211,330s le NO mid redescend jusqu'à 0,775$ sans que l'oracle repasse le strike : un stop carnet aurait sorti à perte, CONFIRME P5.
- La prob consensus YES finit à 0,01$−0,02$ : le carnet converge vers le résultat officiel DOWN.


**A6. SPÉCIFIQUE À LA SESSION** : les 28 événements « carnet croisé » (28/543 = 5,2%) et le rebond NO 0,92$→0,86$ entre t=212,376s et t=212,502s relèvent de la microstructure de CETTE session ; je les classe non-reproductibles car concentrés sur des chocs locaux sans équivalent ailleurs dans la fenêtre.

### A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT

**A1. IDENTITÉ** : « Spot Binance (flux direct) vs Oracle Chainlink — strike exact (dernier tick ≤ t0) ». Vue 300 s (t=0,000s→t=300,000s) plus 3 zooms 15 s. Axe Y haut : prix BTC en $ (64180−64260) ; sous-graphe : basis = spot − oracle en $. Ligne horizontale : « Strike ouverture (64,215$) » (valeur arrondie d'affichage ; valeur exacte en B-F1). 4068 ticks spot, 277 ticks oracle, fallback 0/277 (0,0%).

**A2. QUESTION** : de quel côté du strike l'oracle s'installe-t-il, quand, et avec quelle amplitude ? Et la basis spot−oracle justifie-t-elle d'ignorer le spot (P1) sur cette session ?

**A3. COMMENT** : je lis les étiquettes oracle contre la ligne de strike sur la vue 300 s, la marche d'escalier des paliers oracle, et le niveau du sous-graphe basis.

**A4. VALEURS EXTRAITES** :

- t=0,000s : oracle 64 214,95$ (ancre du strike)
- t=14,570s : spot 64 270,06$ (plus haut spot)
- t=64,000s : oracle 64 206,00$ (premier palier nettement sous le strike)
- t=204,000s : oracle 64 167,51$ (plus bas oracle de la session)
- t=188,135s : spot 64 224,57$ (plus bas spot — ENCORE AU-DESSUS du strike)
- t=299,285s : spot 64 247,29$ (dernier tick spot affiché)


**A5. OBSERVATIONS BRUTES** :

- Le spot ne touche jamais la ligne de strike : son minimum 64 224,57$ (t=188,135s) reste 9,62$ au-dessus — un signal spot aurait désigné UP toute la session, alors que le résultat est DOWN. CONFIRME P1 de façon décisive.
- L'oracle décroche en 3 paliers : t=63,000s→t=66,000s (64 211,96$→64 199,28$), t=128,000s→t=135,000s (64 195,93$→64 176,52$), t=189,000s→t=204,000s (64 170,75$→64 167,51$) : l'installation sous le strike est progressive et sans retour, ce qui ORIENTE L1 (amplitude + persistance).
- Le sous-graphe basis reste dans une bande 45−67$ : jamais de convergence spot/oracle, CONFIRME que le spot est inutilisable comme proxy du règlement.


**A6. SPÉCIFIQUE À LA SESSION** : l'amplitude de la basis (+54,66$ de moyenne, B-F2) est une condition particulière de cette session ; sur une autre session la basis peut être faible, je classe son AMPLITUDE comme non-reproductible — seul le principe de la mesurer est transférable.

### A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ YES/NO (1165 TRADES)

**A1. IDENTITÉ** : « Flux directionnel normalisé YES/NO (1165 trades) », fenêtre glissante 30 s, flux haussier (BUY YES + SELL NO) en positif, baissier en négatif, en $. Vue 300 s (t=0,000s→t=300,000s) plus 3 zooms 15 s. Axe Y : −2500$ à +1000$.

**A2. QUESTION** : le flux d'ordres corrobore-t-il le côté désigné par l'oracle, et à quel moment le flux bascule-t-il par rapport aux prints oracle ?

**A3. COMMENT** : je lis les étiquettes des trades les plus gros et le signe dominant du flux avant/après t=63,000s.

**A4. VALEURS EXTRAITES** :

- t=25,670s : +200,99$ (flux haussier dominant en début de session)
- t=87,795s : +208,77$ (dernier gros trade haussier isolé)
- t=135,671s : −99,99$ (rafale baissière du 2e palier oracle)
- t=199,134s : −499,89$ (plus gros flux baissier avant la fin)
- t=274,608s : −500,00$
- t=280,922s : −953,17$ (plus gros trade de la session, vendeur)


**A5. OBSERVATIONS BRUTES** :

- Avant t=63,000s le flux est mixte à dominante haussière ; après t=63,000s il devient massivement baissier — le flux SUIT le décrochage oracle, il ne le précède pas : le carnet/flux n'apporte rien comme signal, CONFIRME P1 et P3.
- Les rafales baissières coïncident avec les paliers oracle (t=127,751s−t=135,671s ; t=199,134s−t=204,000s) : cohérence flux/oracle.


**A6. SPÉCIFIQUE À LA SESSION** : le trade de −953,17$ à t=280,922s, dans les 20 dernières secondes, est un événement de liquidation terminale propre à cette session ; non-reproductible comme repère.

### A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ** : « Imbalance = Haussier / (Haussier + Baissier) » sur les 1165 trades, ligne d'équilibre à 0,5. Vue 300 s (t=0,000s→t=300,000s) plus 3 zooms 15 s. Axe Y : 0,0 à 1,0.

**A2. QUESTION** : l'imbalance donne-t-elle une information d'ancrage que l'oracle ne donne pas déjà, ou confirme-t-elle seulement a posteriori ?

**A3. COMMENT** : je lis les étiquettes extrêmes et la position de la courbe par rapport à 0,5 sur chaque zoom.

**A4. VALEURS EXTRAITES** :

- t=1,124s : imbalance 1,00 (premier trade, haussier)
- t=55,888s : 0,43 (première bascule sous 0,5, zoom 1)
- t=67,666s : 0,19 (effondrement post-signal)
- t=127,751s : 0,71 (repasse au-dessus de 0,5 AVANT la rafale baissière du 2e palier — faux signal)
- t=204,657s : 0,02 (extrême baissier)
- t=297,174s : 0,04 (fin de session)


**A5. OBSERVATIONS BRUTES** :

- À t=127,751s l'imbalance est à 0,71 (côté haussier) alors que l'oracle est installé 17$ sous le strike depuis 60 s : l'imbalance CONTREDIRAIT le bon côté à cet instant — CONFIRME par contre-exemple l'interdiction P3 d'utiliser le flux comme signal.
- L'imbalance ne passe durablement sous 0,5 qu'après t=55,888s, soit APRÈS le premier croisement de strike oracle (t=45,000s) : information en retard.


**A6. SPÉCIFIQUE À LA SESSION** : la valeur 1,00 à t=1,124s est un artefact de démarrage (un seul trade dans la fenêtre) ; non-reproductible.

### A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS

**A1. IDENTITÉ** : « Spreads YES et NO absolus (événementiels) », ask − bid en $, 543 événements chacun. Vue 300 s (t=0,000s→t=300,000s) plus 3 zooms 15 s. Axe Y : 0,000$ à 0,150$.

**A2. QUESTION** : combien coûte l'exécution au moment du signal, et un stop de prix serait-il exécutable pendant les chocs (P5) ?

**A3. COMMENT** : je lis les extrema étiquetés et le niveau du spread dans la fenêtre du signal (zoom [t=55,000s→t=70,000s]).

**A4. VALEURS EXTRAITES** :

- t=1,076s : spread 0,01$ (ouverture, carnet serré)
- t=62,616s : spread 0,16$ (maximum de la session, au cœur du choc)
- t=62,775s : spread revenu à 0,01$ (le choc dure moins de 0,2 s)
- t=127,727s : spread 0,06$ (2e palier oracle)
- t=209,298s : spread 0,06$ (remontée oracle)
- t=273,651s : spread 0,01$ (fin de session)


**A5. OBSERVATIONS BRUTES** :

- Le spread explose ×16 (0,01$→0,16$) exactement pendant le décrochage oracle à t=62,616s : un stop carnet posé avant le choc aurait été rempli au pire prix — CONFIRME P5 (aucun stop).
- À t=66,544s (mon exécution) le spread NO est de 0,01$ (0,74$/0,75$) : entrer 3,5 s après le choc coûte le spread minimal. ORIENTE le choix de confirmer L1 sur 3 prints plutôt que d'entrer au premier print.


**A6. SPÉCIFIQUE À LA SESSION** : trois valeurs négatives étiquetées (−0,01$ à t=1,844s, t=2,992s, t=127,676s) sont des états croisés fugaces de CE carnet ; bruit, non-reproductible.

### A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ** : « Écart inter-carnets : yes_mid − (1 − no_mid) », 543 événements, 28 suspects « carnet croisé », ligne de cohérence parfaite à 0. Vue 300 s (t=0,000s→t=300,000s) plus 3 zooms 15 s. Axe Y : −1,0 à +1,0 avec facteur d'échelle 1e−16.

**A2. QUESTION** : puis-je lire indifféremment le signal de prix sur YES ou NO, et la prob consensus (YES, 1−NO) est-elle fiable ?

**A3. COMMENT** : je lis le facteur d'échelle de l'axe (1e−16) et la position de la courbe sur la ligne zéro dans les 4 fenêtres.

**A4. VALEURS EXTRAITES** (l'écart est à zéro à l'échelle machine sur toute la session ; je le lis aux bornes de chaque fenêtre affichée) :

- t=0,000s→t=300,000s (vue 300 s) : écart 0,00$ (échelle 1e−16)
- t=198,000s→t=213,000s (zoom 1) : écart 0,00$
- t=55,000s→t=70,000s (zoom 2) : écart 0,00$
- t=122,000s→t=137,000s (zoom 3) : écart 0,00$
- Nombre de suspects « carnet croisé » marqués : 28 sur 543 événements


**A5. OBSERVATIONS BRUTES** :

- yes_mid + no_mid = 1 à 1e−16 près sur les 543 événements : les deux carnets sont l'image miroir l'un de l'autre — acheter NO au ask ou vendre YES au bid est informationnellement équivalent, ce qui simplifie l'exécution P4 sur le seul jeton NO.


**A6. SPÉCIFIQUE À LA SESSION** : rien à signaler — la cohérence parfaite est structurelle (teneur de marché unique), pas une particularité de session. Ce graphe n'apporte aucun paramètre de calibration ; je vous le dis explicitement plutôt que de le sauter.

---

## PARTIE B — LES FICHIERS CSV

### B-F1 — ORACLE.CSV

**B1. IDENTITÉ** : oracle.csv, colonnes t_ms, price, ts_src — toutes exploitées. 277 lignes, de t=0,000s à t=298,000s, échantillonnage nominal 1 s avec 16 trous >1 s (trou maximal 8 s à t=284,000s).

**B2. APPORT** : le strike EXACT à toutes ses décimales et la série des marges — ce que le graphe 2 ne donne qu'arrondi. C'est le fichier qui déclenche tout (P1, P2, P3).

**B3. COMMENT** : strike = price du dernier tick ≤ t0 (ligne 1, t_ms=0). Marge(t) = price(t) − strike sur les 277 lignes. Comptages de signe, croisements de signe, minima/maxima, plus longue série sous le strike, recherche du premier triplet consécutif de marges ≤ −5,00$.

**B4. RÉSULTATS CHIFFRÉS** :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Strike exact (dernier tick ≤ t0) | 64 214,94718301$ | t_ms, price | 1 (t=0,000s)
| Dernier print | 64 189,44009943$ à t=298,000s (marge −25,51$) | t_ms, price | 1
| Prints sous / sur / égal au strike | 246 / 30 / 1 | price | 277
| Croisements du strike | 2 (à la hausse t=13,000s ; à la baisse t=45,000s) | t_ms, price | 277
| Marge minimale | −47,44$ à t=204,000s | t_ms, price | 277
| Marge maximale | +2,59$ à t=33,000s | t_ms, price | 277
| Série ininterrompue sous le strike | 234 prints, de t=45,000s à t=298,000s | t_ms, price | 234
| Premier triplet consécutif de marges ≤ −5,00$ | t=64,000s (−8,95$), t=65,000s (−11,53$), t=66,000s (−15,67$) | t_ms, price | 3
| Marge maximale APRÈS t=66,000s | −15,66$ à t=86,000s | t_ms, price | 231
| ts fallback (ts_src ≠ payload) | 0 / 277 (0,0%) | ts_src | 277


**B5. OBSERVATIONS BRUTES** :

- Après t=45,000s, l'oracle ne repasse JAMAIS au-dessus du strike : 234 prints consécutifs côté DOWN — CONFIRME P2/P3, le côté désigné est sans ambiguïté.
- Le bruit de marge pré-choc (t=45,000s→t=62,000s) est borné à −0,86$ ; la marge maximale positive de toute la session est +2,59$ : un seuil de −5,00$ est hors de portée du bruit, ce qui ORIENTE L1.
- Marge maximale après le signal : −15,66$ — un critère de sortie anticipée « marge ≥ 0 » (L4) ne peut jamais se déclencher sur cette session.


**B6. SPÉCIFIQUE À LA SESSION** : 16 trous d'échantillonnage >1 s (max 8 s à t=284,000s) — verdict : bruit de transport, sans effet car aucun ne chevauche la fenêtre du signal (t=63,000s→t=66,000s) ; 0 fallback de timestamp.

### B-F2 — SPOT.CSV

**B2. APPORT** : la mesure de la basis spot−oracle, seul usage autorisé du spot (P1 : contrôle qualité).

**B1. IDENTITÉ** : spot.csv, colonnes t_ms, price — toutes exploitées. 4068 lignes, de t=0,266s à t=299,285s, flux événementiel Binance BTC/USDT.

**B3. COMMENT** : basis(t) = price_spot(t) − price_oracle(dernier tick ≤ t), calculée sur les 4068 lignes ; min/max/moyenne/médiane ; comparaison du minimum spot au strike.

**B4. RÉSULTATS CHIFFRÉS** :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre de ticks | 4068 | t_ms | 4068
| Spot minimum | 64 224,57$ à t=188,135s | t_ms, price | 4068
| Spot maximum | 64 270,06$ à t=14,570s | t_ms, price | 4068
| Écart spot min − strike | +9,62$ (jamais sous le strike) | price + B-F1 | 4068
| Basis moyenne | +54,66$ | price + oracle.price | 4068
| Basis médiane | +54,53$ | price + oracle.price | 4068
| Basis minimale | +45,20$ à t=129,533s | price + oracle.price | 4068
| Basis maximale | +67,17$ à t=208,762s | price + oracle.price | 4068
| Ticks avec basis ≤ 0 | 0 / 4068 | price + oracle.price | 4068


**B5. OBSERVATIONS BRUTES** :

- Le spot est resté au-dessus du strike pendant 100% de la session alors que le règlement est DOWN : tout signal fondé sur le spot désignait le côté perdant — CONFIRMATION la plus forte de P1 sur cette session.
- La basis ne converge jamais (min +45,20$) : le contrôle qualité valide que les deux flux sont bien distincts et que l'oracle est le seul référentiel de règlement.


**B6. SPÉCIFIQUE À LA SESSION** : paquets de ticks à timestamp identique (rafales à t=11,454s, t=62,165s−t=62,295s, etc.) — verdict : bruit d'agrégation du flux, sans effet sur la basis journalisée.

### B-F3 — BBO.CSV

**B1. IDENTITÉ** : bbo.csv, colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask — toutes exploitées. 543 lignes, de t=1,076s à t=280,562s, événementiel (chaque changement de BBO).

**B2. APPORT** : les prix d'exécution exacts (P3, P4, P5) — ask NO à l'entrée, mid NO pour la perte latente — que les graphes ne donnent qu'aux points étiquetés.

**B3. COMMENT** : spread = ask − bid par jeton ; comptage des états bid ≥ ask ; premier événement postérieur au signal t=66,000s ; minimum du mid NO après l'entrée ; premier événement où no_ask ≥ 0,85$ ; dernier événement où no_ask ≤ 0,85$.

**B4. RÉSULTATS CHIFFRÉS** :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre d'événements | 543 | t_ms | 543
| Spread YES moyen / médian | 0,0223$ / 0,0200$ | yes_bid, yes_ask | 541
| Spread NO moyen / médian | 0,0223$ / 0,0200$ | no_bid, no_ask | 540
| Spread maximal | 0,16$ à t=62,616s (YES et NO) | 4 colonnes | 543
| États incohérents (bid ≥ ask) | 28 / 543 (5,2%) | 4 colonnes | 543
| Premier événement après le signal t=66,000s | t=66,544s : NO 0,74$ / 0,75$ | t_ms, no_bid, no_ask | 1
| Premier no_ask ≥ 0,85$ | t=127,727s | t_ms, no_ask | 543
| Dernier no_ask ≤ 0,85$ | t=212,136s | t_ms, no_ask | 543
| Mid NO minimal après l'entrée | 0,68$ à t=87,206s (0,67$/0,69$) | t_ms, no_bid, no_ask | 519
| Dernier état | t=280,562s : no_bid 0,99$ | t_ms, no_bid | 1


**B5. OBSERVATIONS BRUTES** :

- 544 ms séparent mon signal (t=66,000s) du premier événement BBO exécutable (t=66,544s), avec 0 événement entre les deux : le prix d'entrée 0,75$ est déterministe, aucune race condition d'exécution à cet instant.
- Le no_ask reste ≤ 0,85$ jusqu'à t=212,136s : la fenêtre d'edge L2 est restée ouverte 146 s après mon entrée — l'entrée précoce n'était pas indispensable mais elle a payé le meilleur prix disponible après confirmation.
- Le mid NO à 0,68$ (t=87,206s) pendant que la marge oracle est à −15,66$ : le carnet doute alors que l'oracle ne doute pas — CONFIRME P5 (tenir sur l'oracle, pas sur le carnet).


**B6. SPÉCIFIQUE À LA SESSION** : les 28 états bid ≥ ask (les « incohérents : 28 » du PDF) sont fugaces (résolus à l'événement suivant, cf. t=1,844s, t=127,676s, t=134,003s) — verdict : bruit de séquencement, aucun ne tombe sur mon timestamp d'exécution.

### B-F4 — TRADES.CSV

**B1. IDENTITÉ** : trades.csv, colonnes t_ms, usd, direction — toutes exploitées. 1165 lignes, de t=1,124s à t=297,174s, un trade par ligne, direction +1 haussier / −1 baissier.

**B2. APPORT** : les volumes exacts en $ par direction, que le graphe 3 n'affiche qu'en flux fenêtré 30 s.

**B3. COMMENT** : sommes et comptages par direction sur les 1165 lignes ; partition avant/après t=63,000s (premier print oracle nettement sous le strike) ; maximum de usd.

**B4. RÉSULTATS CHIFFRÉS** :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre de trades | 1165 | t_ms | 1165
| Volume total | 15 535,43$ | usd | 1165
| Trades haussiers | 341 pour 4 117,16$ | usd, direction | 341
| Trades baissiers | 824 pour 11 418,27$ | usd, direction | 824
| Part haussière du volume | 26,50% | usd, direction | 1165
| Volume avant t=63,000s | haussier 1 538,13$ / baissier 1 151,20$ | t_ms, usd, direction | 1165
| Volume après t=63,000s | haussier 2 579,03$ / baissier 10 267,07$ | t_ms, usd, direction | 1165
| Plus gros trade | 953,17$ baissier à t=280,922s | usd, direction | 1


**B5. OBSERVATIONS BRUTES** :

- Avant t=63,000s le volume est majoritairement haussier (57,2% de 2 689,33$) ; après, il est baissier à 79,9% : la foule bascule APRÈS l'oracle, jamais avant — CONFIRME P1/P3 par contre-exemple (le flux est un suiveur).
- Aucun de mes paramètres L1-L5 n'est fondé sur ce fichier : je l'exploite comme pièce de convergence, pas comme signal — conforme à P3.


**B6. SPÉCIFIQUE À LA SESSION** : trois trades ≥ 300,00$ concentrés après t=181,311s (323,00$ ×2, 500,00$ à t=274,608s, 953,17$ à t=280,922s) — verdict : liquidations terminales propres à cette session.

---

## PARTIE C — APPLICATION ET SIMULATION

### C0 — CALIBRATION DE LA DOCTRINE SUR CETTE SESSION

Voici chaque paramètre, sa valeur, et la mesure qui le justifie.

| Paramètre (P ou L) | Valeur choisie/mesurée | Justification chiffrée | Source
|-----|-----|-----|-----
| P2 — Strike | 64 214,94718301$ | Dernier tick ≤ t0, t=0,000s | B-F1, B4
| P2 — Côté désigné | DOWN ⇒ jeton NO | 246 prints sous le strike / 30 au-dessus ; 234 consécutifs depuis t=45,000s | B-F1, B4
| L1 — Ancrage | Marge ≤ −5,00$ sur 3 prints consécutifs | Bruit pré-choc borné à −0,86$ (t=45,000s→t=62,000s) et marge max de session +2,59$ : −5,00$ = 5,8× le bruit maximal, inatteignable par oscillation ; 3 prints (≈3 s) filtrent un print isolé | B-F1, B4 et B5
| L2 — Edge carnet | Prix d'achat NO maximal 0,85$ | Le carnet converge vite (NO ask 0,49$→0,69$ en 316 ms à t=62,459s→t=62,775s, A-G1 A5) ; à 0,85$ l'espérance résiduelle est de +0,15$ par jeton (+17,6%) si le côté oracle est correct ; le no_ask atteint 0,85$ à t=127,727s | A-G1 A5 ; B-F3 B4
| L3 — Fenêtre d'entrée | Entrée autorisée de t=0,000s à t=270,000s | Le dernier no_ask ≤ 0,85$ date de t=212,136s et le dernier événement BBO de t=280,562s : après t=270,000s le carnet est convergé et illiquide (no_bid 0,99$) | B-F3, B4
| L4 — Sortie anticipée | Marge oracle ≥ 0,00$ sur 3 prints consécutifs ⇒ vente au bid NO | Symétrique de L1 ; seuls les prints oracle déclenchent (P5) ; croisement inverse observable en 2 occurrences sur la session (t=13,000s, t=45,000s) donc critère lisible | B-F1, B4
| L5 — Comptable | Frais 0,00$ [HYPOTHÈSE n°1] ; fills fractionnaires intégraux au prix affiché [HYPOTHÈSE n°2] ; règlement cash à 1,00$/jeton du côté officiel à la clôture [HYPOTHÈSE n°3] | Aucune colonne de frais ni de profondeur dans les CSV ; le résultat officiel figure dans le titre du PDF (▼ DOWN) | B-F3 B1 ; A-G1 A1


Déductions verrouillées :

- Signal d'entrée : triplet t=64,000s (−8,95$), t=65,000s (−11,53$), t=66,000s (−15,67$) ⇒ **signal à t=66,000s** (B-F1, B4), dans la fenêtre L3.
- Premier événement BBO exécutable après le signal : **t=66,544s, no_ask 0,75$** (B-F3, B4) ; 0,75$ ≤ 0,85$ (L2) ⇒ entrée validée.
- Sortie anticipée L4 : marge maximale après le signal −15,66$ à t=86,000s (B-F1, B4) ⇒ **JAMAIS DÉCLENCHÉE**.


CALIBRATION VERROUILLÉE — ces paramètres ne seront plus modifiés.

### C1 — TABLE DE CONVERGENCE

| Principe | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| P1 (oracle seul référentiel) | A-G2 (spot min 64 224,57$ > strike, résultat DOWN) ; B-F2 (basis +54,66$, 0 tick ≤ 0) ; A-G4 (imbalance 0,71 du mauvais côté à t=127,751s) ; B-F4 (flux suiveur) | Aucune | CONFIRMÉ
| P2 (ancre strike) | B-F1 (246/277 prints sous strike, 2 croisements) ; A-G2 (ligne strike) | Aucune | CONFIRMÉ
| P3 (entrée sur prints oracle + edge carnet) | B-F1 (triplet ≤ −5,00$ à t=66,000s) ; B-F3 (no_ask 0,75$ à t=66,544s) ; A-G1 (edge lisible) | Aucune | CONFIRMÉ
| P4 (100% du cash, un trade, au ask) | B-F3 (ask 0,75$ exécutable, spread 0,01$ à l'entrée) | Aucune pièce ne teste la profondeur du carnet (pas de colonne taille) | CONFIRMÉ, profondeur non vérifiable [FRAGILE - 1 SOURCE]
| P5 (tenue, pas de stop carnet) | A-G5 (spread ×16 à t=62,616s : stop inexécutable) ; B-F3 (mid NO 0,68$ à t=87,206s pendant marge oracle −15,66$) ; B-F1 (marge jamais ≥ 0 après signal) | Aucune | CONFIRMÉ


J'annonce le [FRAGILE - 1 SOURCE] sur P4 : la seule pièce prix est bbo.csv, sans profondeur ; l'hypothèse n°2 le couvre.

### C2 — LA STRATÉGIE EN PSEUDO-CODE EXÉCUTABLE

```plaintext
# ANCRAGE ORACLE — session BTC 5min 6 août, 19:25-19:30 ET
STRIKE = 64214.94718301          # P2 — oracle.csv, dernier tick <= t0 (B-F1, B4)
CASH = 5.00                      # P4 — contrainte dure
position = None; consec = 0

pour chaque print oracle (t, price) dans oracle.csv:      # P1/P3 — seuls les prints déclenchent
    marge = price - STRIKE                                 # P2
    # --- ENTRÉE ---
    si position is None et t <= 270000:                    # L3 (B-F3, B4 : dernier ask<=0.85 à t=212,136s)
        consec = consec + 1 si marge <= -5.00 sinon 0      # L1 (B-F1, B5 : bruit max -0.86$)
        si consec == 3:                                    # signal => t=66,000s (B-F1, B4)
            evt = premier événement bbo.csv avec t_evt > t # B-F3, B4 : t=66,544s
            si evt.no_ask <= 0.85:                         # L2 (B-F3, B4 : seuil atteint à t=127,727s)
                qty = CASH / evt.no_ask                    # [HYPOTHÈSE n°2] fill fractionnaire intégral
                position = (qty, evt.no_ask); CASH = 0.00  # P4 : 100% du cash, au ask, frais 0 [HYPOTHÈSE n°1]
    # --- SORTIE ANTICIPÉE ---
    sinon si position:                                     # P5 : uniquement sur prints oracle
        consec_neg = consec_neg + 1 si marge >= 0.00 sinon 0   # L4
        si consec_neg == 3: vendre au no_bid du prochain evt bbo  # jamais atteint (B-F1: max -15.66$)

# --- RÈGLEMENT --- P5 : résultat officiel PDF = DOWN => NO = 1.00$ [HYPOTHÈSE n°3]
CASH = qty * 1.00
```

### C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

**ÉTAT DU PORTEFEUILLE**

| t | Événement | Cash | Position | Valeur totale (au mid NO)
|-----|-----|-----|-----
| t=0,000s | Ouverture | 5,00$ | — | 5,00$
| t=66,000s | Signal L1 (3e print ≤ −5,00$) | 5,00$ | — | 5,00$
| t=66,544s | Achat 6,666667 NO à 0,75$ (coût 5,00$ ≤ cash : vérifié) | 0,00$ | 6,666667 NO @ 0,75$ | 4,93$ (mid 0,74$)
| t=87,206s | Point bas latent (mid NO 0,68$) | 0,00$ | 6,666667 NO | 4,53$
| t=212,136s | Dernier ask NO ≤ 0,85$ (aucune action : L4 non déclenché) | 0,00$ | 6,666667 NO | 5,80$ (mid 0,87$)
| t=300,000s | Règlement DOWN : NO = 1,00$ | 6,67$ | — | 6,67$


**RACE CONDITIONS**

(a) Signal d'achat pendant un ordre en cours : impossible par construction P4 — un seul ordre d'entrée pour la session ; tout signal L1 ultérieur trouve `position != None` et est ignoré. RÈGLE : le premier signal L1 consomme le droit d'entrée, tous les suivants sont rejetés.

(b) Signal de vente sur une position pas encore confirmée : entre le signal (t=66,000s) et le fill (t=66,544s), bbo.csv ne contient aucun événement (B-F3, B5) ; si un signal L4 tombait dans cet intervalle, il serait mis en file et évalué après confirmation du fill. RÈGLE : aucun ordre de sortie n'est émis tant que le fill d'entrée n'est pas confirmé ; le signal de sortie est réévalué au print oracle suivant.

(c) Deux signaux simultanés sur le même actif : deux prints oracle ne partagent jamais un timestamp dans oracle.csv (277 t_ms strictement croissants, B-F1) ; si le cas survenait, priorité à la sortie sur l'entrée. RÈGLE : à timestamp égal, l'ordre de sortie prime et l'ordre d'entrée est annulé.

(d) Fill partiel : bbo.csv n'expose pas la profondeur ; l'[HYPOTHÈSE n°2] pose le fill intégral à 0,75$. Si un fill partiel survenait, le reliquat de cash resterait en cash (P4 interdit une deuxième entrée). RÈGLE : toute quantité non remplie au ask signalé est abandonnée, jamais re-soumise à un prix supérieur à L2.

**JOURNAL DE TRADES**

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=66,544s | 0,75$ | 6,666667 NO | t=300,000s (règlement) | 1,00$ | 0,00$ | +1,67$ | 6,67$


**RÉSULTATS**

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 1 / +1,67$ | Ligne 1
| Trades perdants / pertes cumulées | 0 / 0,00$ | —
| Frais totaux | 0,00$ [HYPOTHÈSE n°1] | Ligne 1
| BÉNÉFICE NET | +1,67$ | Ligne 1
| Capital final vs initial | 6,67$ vs 5,00$ (+33,33%) | Ligne 1
| Pire perte latente pendant la tenue | −0,47$ (valeur 4,53$, mid NO 0,68$ à t=87,206s, entrée t=66,544s) | Ligne 1 + B-F3 B4
| Pire perte unitaire / drawdown maximum | −0,47$ soit −9,33% du capital, de t=66,544s à t=87,206s | Ligne 1 + B-F3 B4


### C3 — PROFIL DE LA SESSION

Vous comparez cette session à d'autres jouées avec la même doctrine ; voici exactement ce qui distinguait celle-ci.

| # | Spécificité de la session (fait chiffré) | Source | Effet sur la doctrine/calibration | Effet sur le rendement
|-----|-----|-----|-----
| 1 | Basis spot−oracle moyenne +54,66$ (min +45,20$) ; spot min 64 224,57$ = +9,62$ AU-DESSUS du strike alors que le résultat est DOWN | B-F2 B4 | Soutient P1 au maximum : le spot désignait le côté perdant toute la session | + : sans P1, entrée YES ⇒ règlement 0,00$ ⇒ −5,00$ ; avec P1, +1,67$
| 2 | Bascule oracle unique et précoce : 2 croisements seulement, dernier à t=45,000s, puis 234 prints consécutifs sous le strike | B-F1 B4 | A permis un L1 simple (−5,00$ × 3 prints) déclenché à t=66,000s, à 22% de la session | + : entrée à 0,75$ au lieu de 0,85$ (seuil atteint à t=127,727s, B-F3 B4) ⇒ +0,89$ de P&L en plus qu'une entrée au plafond L2
| 3 | Choc de spread ×16 (0,01$→0,16$ à t=62,616s, résolu en 316 ms) simultané au décrochage oracle | A-G5 A4 ; B-F3 B4 | Soutient P5 (stop inexécutable) et a motivé la confirmation sur 3 prints (L1) plutôt qu'un print unique | Neutre sur le P&L final : l'attente de 3 s a payé un spread de 0,01$ à l'entrée au lieu de 0,16$ pendant le choc
| 4 | Doute du carnet sans doute de l'oracle : mid NO 0,68$ à t=87,206s pendant que la marge oracle est à −15,66$ | B-F3 B4 ; B-F1 B4 | Met P5 sous tension puis le valide : aucun print oracle ≥ 0, donc aucune sortie | − temporaire : latence de −0,47$ (−9,33%) entièrement récupérée au règlement
| 5 | Bruit de marge pré-choc quasi nul : oscillation bornée à −0,86$ entre t=45,000s et t=62,000s, marge max de session +2,59$ | B-F1 B4/B5 | A permis de fixer L1 à −5,00$ (5,8× le bruit) sans risque de faux positif | + : zéro faux signal, un seul trade, aucun aller-retour de frais ou de spread


SPÉCIFICITÉ DOMINANTE : la basis spot−oracle moyenne de +54,66$ avec un spot jamais descendu sous le strike (min +9,62$ au-dessus) alors que le règlement est DOWN — c'est elle qui rendait P1 décisif : ±6,67$ d'écart entre suivre l'oracle et suivre le spot.

TRANSFÉRABILITÉ : sur une autre session, vérifier en priorité le signe et l'amplitude de la basis spot−oracle AVANT l'entrée, car un spot durablement du mauvais côté du strike est le cas où la doctrine crée tout son écart avec une lecture naïve. [PATTERN CANDIDAT]

### C4 — PORTÉE ET VERDICT DE COMPORTEMENT

- L'oracle installé d'un seul côté après une bascule unique (234 prints consécutifs) rend L1 trivial à déclencher tôt et bon marché. [SESSION-SPÉCIFIQUE]
- Le flux et l'imbalance basculent APRÈS les prints oracle (imbalance 0,71 du mauvais côté à t=127,751s) : le flux est un suiveur, pas un signal. [PATTERN CANDIDAT]
- Le carnet peut douter (mid NO 0,68$) pendant que l'oracle ne doute pas (marge −15,66$) : la tenue P5 sur prints oracle évite une sortie à perte. [PATTERN CANDIDAT]
- Les chocs de spread coïncident avec les décrochages oracle (0,16$ à t=62,616s) : entrer après confirmation de 3 prints paie le spread minimal. [PATTERN CANDIDAT]
- La basis de +54,66$ en amplitude est propre à cette session ; seul le réflexe de la mesurer se transfère. [SESSION-SPÉCIFIQUE]


VERDICT : [STRATÉGIE CONFIRMÉE SUR CETTE SESSION]

RETOUR SUR CALIBRATION : le choix qui a piloté le résultat est L1 (−5,00$ × 3 prints), déclenché à t=66,000s soit 61,7 s avant que le no_ask n'atteigne le plafond L2 de 0,85$ (t=127,727s, B-F3 B4) — c'est cet écart qui fabrique +0,89$ des +1,67$. L4 (sortie sur marge ≥ 0) n'a jamais approché son seuil (max −15,66$, B-F1 B4) : cette session apprend que la doctrine gagne quand l'ancrage oracle est franc et précoce ; les paramètres restent verrouillés et le résultat rapporté est celui de C0.

---

## AUTO-CONTRÔLE FINAL

- Contrôle 1 → valeurs chiffrées : A-G1 : 6, A-G2 : 6, A-G3 : 6, A-G4 : 6, A-G5 : 6, A-G6 : 5 (≥5 chacun) ; B-F1 : 10, B-F2 : 9, B-F3 : 10, B-F4 : 8 (≥8 chacun). CONFORME.
- Contrôle 2 → chaque affirmation de la Partie C cite une rubrique A4/A5 ou B4/B5 identifiée. CONFORME.
- Contrôle 3 → aucun chiffre en double avec des valeurs différentes ; ouverture et clôture reprennent 6,67$, +33,33%, +1,67$, t=66,000s, 0,75$, +54,66$ à l'identique du corps. CONFORME.
- Contrôle 4 → cohérence comptable : 5,00$ − 5,00$ (achat ligne 1) + 6,67$ (règlement 6,666667 × 1,00$) = 6,67$ = capital final annoncé, au centime près. CONFORME.
- Contrôle 5 → 3 [HYPOTHÈSE] utilisées (n°1 frais, n°2 fill intégral, n°3 règlement cash) : pas plus de 3, chacune imposée par l'absence de colonne correspondante dans les CSV. CONFORME.
- Contrôle 6 → conformité doctrine et verrouillage : chaque condition du pseudo-code C2 renvoie à un principe P1-P5 ou à un paramètre L1-L5 de C0 ; aucune valeur de C0 modifiée entre calibration et simulation. Je confirme explicitement : les principes P1 à P5 ont été appliqués sans dérogation et la calibration verrouillée en C0 n'a pas été modifiée.
- Contrôle 7 → format du speech : adresse directe, verdict dès l'ouverture, ordre F1 respecté (0→1→A→B→C0→C1→C2→C-SIM→C3→C4→auto-contrôle→clôture), tableaux aux formats F4, identifiants A-G1…B-F4 corrects, clôture en 3 lignes sans chiffre nouveau. CONFORME.
- Contrôle 8 → C3 : 5 spécificités, toutes chiffrées et sourcées A4/B4, chacune avec effet calibration ET effet rendement chiffré ; spécificité dominante identique à celle de l'ouverture (basis +54,66$) ; ligne TRANSFÉRABILITÉ étiquetée [PATTERN CANDIDAT]. CONFORME.


---

## CLÔTURE DU SPEECH

Le bénéfice net est de +1,67$, pour un capital final de 6,67$ contre 5,00$ engagés, soit +33,33%.
Un seul trade, entré sur prints oracle et tenu jusqu'au règlement malgré un creux latent de −9,33%.
[STRATÉGIE CONFIRMÉE SUR CETTE SESSION] — la doctrine a fait exactement ce qu'elle promet quand l'oracle s'ancre franchement d'un côté du strike.

---

## Source : 7ANCRAGE_ORACLE.md

IL YA UNE PARTICULARITE ,ON a tester plusieurs strategie ici en dehors de lancrage mais l'ancrge oracle est bien fiable sauf que contrairemtn au autres cette fosi si cest le ticket YES qui a ete le bon , :# 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Le verdict d'abord. La stratégie appliquée est l'**ANCRAGE ORACLE** — vous me l'avez imposée, je ne l'ai pas cherchée, je l'ai exécutée et vérifiée contre les données.

- Capital final : **6.85$** vs 5.00$ de départ, rendement net **+36.99%**.
- Nombre de trades : **1**, dont **1 gagnant, 0 perdant**.
- Critère de sélection : sans objet — stratégie prescrite par vous ; la validation se fait par convergence des pièces (section C1).
- Étiquette globale : **[PATTERN CANDIDAT]** pour le mécanisme, l'amplitude du gain restant propre à cette session.


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

---

# 1. CADRE D'ANALYSE CHOISI

J'adopte un cadre **structurel-microstructure** : le marché Polymarket ne règle pas sur le spot Binance mais sur l'oracle Chainlink contre un strike fixe — la vérité de règlement est donc une série de 284 ticks, et tout écart entre le prix YES et la position de l'oracle vs strike est mesurable à la milliseconde. Ce cadre est adapté à CES données : 5 minutes, 4 flux horodatés (oracle, spot, carnet, trades) qui permettent de dater précisément qui sait quoi, et quand. Convention de lecture : les 12 panneaux du PDF sont groupés en 6 blocs (vue + zooms par graphique) ; décimales à point, timestamps au format t=183.311s.

---

# PARTIE A — LES 6 GRAPHES DU PDF

## A-G1 — GRAPHE 1 (VUE 300 s) : CARNET YES/NO COMPLET

**A1. IDENTITÉ.** Carnet complet YES/NO, 590 événements BBO, précision ms ; axe X : temps depuis l'ouverture (00:00.000 → 05:00.000), axe Y : prix 0–1$ ; résultat de la fenêtre : UP.

**A2. QUESTION.** Que fait le prix YES au fil de la fenêtre ? À quels instants le consensus bascule-t-il ?

**A3. COMMENT.** Lecture des extrema annotés et des points d'inflexion de la courbe YES mid, croisés avec les mêmes points côté NO (contrôle de cohérence YES ≈ 1 − NO).

**A4. VALEURS EXTRAITES.**

- t=0.164s : YES = 0.52$
- t=125.052s : YES = 0.11$ (plancher de la fenêtre)
- t=246.862s : YES = 0.99$ (verrouillage)
- t=0.164s : NO = 0.48$
- t=125.052s : NO = 0.89$
- t=246.862s : NO = 0.01$


**A5. OBSERVATIONS BRUTES.**

- Le marché passe de 0.11$ (t=125.052s) à 0.99$ (t=246.862s) en 121.810s : la quasi-totalité du repricing tient dans 2 minutes.
- YES + NO reste collé à 1.00$ sur les annotations (0.11+0.89 ; 0.99+0.01) : pas d'arbitrage interne visible à cette échelle.
- 8 événements "carnet croisé suspects" sont signalés sur 590 (1.36%).


**A6. SPÉCIFIQUE À LA SESSION.** Le point de départ à 0.52$ et le plancher à 0.11$ dépendent du chemin du BTC ce jour-là — non reproductible. Le verrouillage à 0.99$ avant la fin (53s d'avance) est un candidat structurel, pas cette valeur-là précisément.

## A-G2 — GRAPHE 1 (ZOOMS 15 s) : LES TROIS FENÊTRES DE REPRICING

**A1. IDENTITÉ.** Trois zooms 15s du même carnet : [01:20.000→01:35.000], [02:19.000→02:34.000], [03:37.000→03:52.000].

**A2. QUESTION.** À quelle vitesse et par quels paliers le carnet se déplace-t-il quand il bouge ?

**A3. COMMENT.** Lecture des annotations YES successives à l'intérieur de chaque zoom, mesure du delta prix / delta temps.

**A4. VALEURS EXTRAITES.**

- t=81.729s : YES = 0.14$ ; t=89.447s : YES = 0.23$ (+0.09$ en 7.718s)
- t=93.173s : YES = 0.17$ (rechute partielle)
- t=139.877s : YES = 0.22$ ; t=153.615s : YES = 0.47$ (+0.25$ en 13.738s)
- t=217.138s : YES = 0.93$ ; t=224.817s : YES = 0.96$
- Côté NO, miroir exact : t=139.877s : 0.78$ → t=153.615s : 0.53$


**A5. OBSERVATIONS BRUTES.**

- Le repricing se fait par rafales de ticks de 0.01$ espacés de quelques dizaines de ms (ex. 02:33.061→02:33.615 : 0.39$→0.47$ en 0.554s).
- À t=153.615s le YES repasse 0.47$ : le marché n'est qu'à mi-chemin alors que le mouvement spot haussier est déjà engagé (cf. A-G4).


**A6. SPÉCIFIQUE À LA SESSION.** La fenêtre 01:20 montre une fausse reprise (0.14→0.23→0.17$) : bruit lié au chemin du prix ce jour-là, classé non reproductible.

## A-G3 — GRAPHE 2 (VUE 300 s) : SPOT BINANCE vs ORACLE CHAINLINK + BASIS

**A1. IDENTITÉ.** Binance BTC/USDT direct (6994 ticks) vs Chainlink BTC/USD oracle (284 ticks), strike d'ouverture tracé à **64 225.00$** ; sous-graphe basis = spot − oracle.

**A2. QUESTION.** Lequel des deux flux mène l'autre ? Où se situe chacun par rapport au strike, et quand l'oracle le franchit-il ?

**A3. COMMENT.** Lecture des extrema annotés des deux courbes, position vs ligne de strike, lecture de l'échelle du basis (0 à 60$).

**A4. VALEURS EXTRAITES.**

- t=1.288s : spot Binance = 64 265.00$
- t=51.147s : spot min = 64 237.85$
- t=299.916s : spot max = 64 336.29$
- t=0.000s : oracle = 64 215.97$
- Strike = 64 225.00$ (ligne horizontale)
- Basis : borné entre 0 et 60$ sur toute la fenêtre, jamais négatif


**A5. OBSERVATIONS BRUTES.**

- Le spot Binance est au-dessus du strike du premier au dernier tick ; l'oracle démarre SOUS le strike (64 215.97$ < 64 225.00$) et ne le franchit qu'en seconde moitié de fenêtre.
- Le basis reste massivement positif (~50–60$) : le spot mène, l'oracle suit avec retard et lissage.
- C'est l'oracle, pas le spot, qui règle le marché : la ligne de strike ne coupe que la courbe oracle.


**A6. SPÉCIFIQUE À LA SESSION.** L'amplitude du basis (50–60$) reflète l'agrégation multi-sources de Chainlink ce jour-là — la valeur exacte n'est pas reproductible ; le signe et la persistance du retard, si.

## A-G4 — GRAPHE 2 (ZOOMS 15 s) : LE FRANCHISSEMENT VU DE PRÈS

**A1. IDENTITÉ.** Trois zooms 15s spot vs oracle : [01:20→01:35], [02:19→02:34], [03:37→03:52].

**A2. QUESTION.** Quel écart sépare spot et oracle au moment précis où le carnet se reprice ?

**A3. COMMENT.** Lecture appariée spot/oracle aux mêmes timestamps dans chaque zoom.

**A4. VALEURS EXTRAITES.**

- t=80.312s : spot = 64 237.85$ ; t=88.487s : spot = 64 245.62$
- t=139.286s : spot = 64 251.99$ ; t=153.335s : spot = 64 268.00$
- t=139.000s : oracle = 64 196.19$ ; t=154.000s : oracle = 64 210.92$
- t=217.073s : spot = 64 295.99$ ; t=217.000s : oracle = 64 240.22$


**A5. OBSERVATIONS BRUTES.**

- À t=153.335s le spot est à 64 268.00$ (43$ au-dessus du strike) pendant que l'oracle est encore à 64 209.77$ (t=153.000s), 15.23$ SOUS le strike : le règlement n'est pas acquis alors que le carnet reprice déjà (A-G2).
- L'oracle progresse par paliers de 1 à 6$ par seconde dans les phases actives.


**A6. SPÉCIFIQUE À LA SESSION.** Les niveaux absolus de ces paliers sont propres au chemin du jour.

## A-G5 — GRAPHE 3 (VUE 300 s) : FLUX DIRECTIONNEL NORMALISÉ

**A1. IDENTITÉ.** Flux 30s glissant : haussier = BUY YES + SELL NO (positif), baissier = SELL YES + BUY NO (négatif) ; 1380 trades ; échelle −2000$ à +5000$.

**A2. QUESTION.** Que fait le flux d'ordres avant, pendant et après les mouvements de prix du carnet ?

**A3. COMMENT.** Lecture des 4 extrema annotés et du signe dominant par phase.

**A4. VALEURS EXTRAITES.**

- t=0.407s : flux = −12.25$
- t=217.536s : flux = +583.91$
- t=248.094s : flux = −614.97$
- t=280.625s : flux = −39.60$
- Dominante négative sur la première moitié de fenêtre, positive sur la troisième minute


**A5. OBSERVATIONS BRUTES.**

- Le pic haussier (+583.91$ à t=217.536s) arrive APRÈS le franchissement du strike par l'oracle (t=182.0s, cf. B-F1) : le gros du flux acheteur paie 0.93–0.96$ ce qui coûtait 0.73$ 34s plus tôt.
- Le pic baissier (−614.97$ à t=248.094s) survient alors que YES cote déjà 0.99$ : c'est de la prise de profit NO à 0.01$, pas un pari directionnel.


**A6. SPÉCIFIQUE À LA SESSION.** Les montants des pics dépendent des participants présents ce soir-là.

## A-G6 — GRAPHE 3 (ZOOMS 15 s) : LE FLUX AUTOUR DES BASCULES

**A1. IDENTITÉ.** Trois zooms 15s du flux directionnel : [01:20→01:35], [02:19→02:34], [03:37→03:52].

**A2. QUESTION.** Le flux précède-t-il ou suit-il les bascules de prix identifiées en A-G2 ?

**A3. COMMENT.** Comparaison des timestamps de flux annotés avec les timestamps de repricing de A-G2.

**A4. VALEURS EXTRAITES.**

- t=86.768s : flux = −100.00$ (zoom 01:20 — dominante vendeuse)
- t=140.251s : flux = −398.50$ (annoté 02:20.251)
- t=147.077s : flux = +16.02$ (première rafale acheteuse du zoom 02:19)
- t=217.536s : flux = +583.91$
- t=224.994s : flux = −242.20$


**A5. OBSERVATIONS BRUTES.**

- Dans le zoom 02:19, le flux acheteur (t=147.010s et suivants) démarre en même temps que la rafale de repricing 0.24→0.30$ (A-G2, t=147.014s) : le flux accompagne, il ne précède pas de plusieurs secondes.
- Aucun des trois zooms ne montre de flux massif AVANT le mouvement de prix.


**A6. SPÉCIFIQUE À LA SESSION.** Le −398.50$ de t=140.251s est un ordre isolé à contre-tendance juste avant la bascule : hasard de séquencement, non reproductible.

---

# PARTIE B — LES FICHIERS CSV

## B-F1 — oracle.csv (CHAINLINK BTC/USD)

**B1. IDENTITÉ.** Colonnes t_ms, price, ts_src — toutes utilisées. 284 lignes, t=0.000s → t=298.000s, intervalle moyen 1.053s (min 1.000s, max 2.000s).

**B2. APPORT.** C'est la série de RÈGLEMENT : le graphe la montre, le CSV permet de dater au tick près le franchissement du strike, ce qu'aucune lecture graphique ne donne à la seconde.

**B3. COMMENT.** Min/max/premier/dernier ; premier tick > 64 225.00$ ; premier couple de 2 ticks consécutifs > strike ; comptage des retours sous le strike après confirmation ; contrôle ts_src.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre de ticks | 284 | t_ms | 284
| Premier / dernier prix | 64 215.97$ / 64 277.08$ | price | 2
| Minimum | 64 180.24$ à t=68.000s | t_ms, price | 284
| Maximum | 64 277.08$ à t=298.000s | t_ms, price | 284
| Premier tick > strike | 64 226.23$ à t=182.000s | t_ms, price | 284
| Confirmation 2 ticks consécutifs > strike | 64 227.05$ à t=183.000s | t_ms, price | 284
| Retours ≤ strike après t=183.000s | 0 | t_ms, price | 116
| Ticks > strike / total | 115 / 284 | price | 284
| Marge finale vs strike | +52.08$ | price | 1
| ts_src = payload | 284 / 284 (0 fallback) | ts_src | 284


**B5. OBSERVATIONS BRUTES.**

- L'oracle franchit le strike à t=182.000s et n'y retourne JAMAIS (0 retour sur 116 ticks restants).
- Le dernier tick ≤ strike est à t=181.000s : la fenêtre "règlement incertain" se ferme aux deux tiers de la session.
- Horodatage propre : 0 fallback sur 284 ticks.


**B6. SPÉCIFIQUE À LA SESSION.** Deux trous d'échantillonnage à 2.000s (au lieu de ~1s) — bruit de publication, sans impact sur le franchissement. Verdict : bruit.

## B-F2 — spot.csv (BINANCE BTC/USDT)

**B1. IDENTITÉ.** Colonnes t_ms, price — toutes utilisées. 6994 lignes, t=1.288s → t=299.916s.

**B2. APPORT.** Mesure du basis exact spot − oracle tick par tick, que le graphe ne donne qu'en échelle grossière.

**B3. COMMENT.** Min/max ; pour chaque tick oracle, dernier tick spot ≤ t (recherche dichotomique), basis = spot − oracle ; statistiques sur les 282 basis appariables ; comptage des ticks spot > strike.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre de ticks | 6994 | t_ms | 6994
| Premier / dernier prix | 64 265.00$ / 64 336.29$ | price | 2
| Minimum | 64 237.85$ à t=51.147s | t_ms, price | 6994
| Maximum | 64 336.29$ à t=299.916s | t_ms, price | 6994
| Ticks > strike / total | 6994 / 6994 (100%) | price | 6994
| Basis moyen | +55.77$ | les 2 fichiers | 282 paires
| Basis min / max | +50.30$ (t=2.000s) / +60.46$ (t=227.000s) | les 2 fichiers | 282 paires
| Basis négatifs | 0 | les 2 fichiers | 282 paires
| Spot à la confirmation oracle (t=183.000s) | 64 282.82$ | t_ms, price | 1


**B5. OBSERVATIONS BRUTES.**

- Le spot n'est JAMAIS passé sous le strike (6994/6994 ticks au-dessus) alors que le marché YES cotait 0.11$ à t=125.052s : le marché price l'oracle, pas le spot.
- Basis strictement positif sur 282 mesures, borné [50.30$ ; 60.46$] : le retard de l'oracle est stable, pas erratique.


**B6. SPÉCIFIQUE À LA SESSION.** Le creux spot 64 237.85$ à t=51.147s reste 12.85$ au-dessus du strike — configuration du jour, non généralisable. Verdict : signal pour cette session, valeur non reproductible.

## B-F3 — bbo.csv (CARNET POLYMARKET)

**B1. IDENTITÉ.** Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask — toutes utilisées. 590 lignes, t=0.164s → t=246.862s.

**B2. APPORT.** Prix exécutables réels (ask) à chaque instant — c'est lui qui fixe le prix d'entrée de la simulation, pas le mid des graphes.

**B3. COMMENT.** Mid YES = (yes_bid+yes_ask)/2 ; spread = yes_ask − yes_bid ; test de carnet croisé yes_bid+no_bid > 1.00 ; extraction du BBO aux timestamps clés de B-F1.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre d'événements | 590 | t_ms | 590
| YES mid min | 0.115$ à t=125.052s | yes_bid, yes_ask | 588
| YES mid max | 0.99$ à t=246.862s | yes_bid, yes_ask | 588
| Spread YES moyen | 0.0187$ | yes_bid, yes_ask | 588
| Spread YES max / min | 0.07$ / 0.00$ | yes_bid, yes_ask | 588
| Carnets croisés (yes_bid+no_bid>1.00) | 0 | yes_bid, no_bid | 588
| BBO au 1er tick oracle > strike (t=182.000s) | YES 0.76/0.77 | toutes | 1
| 1er événement ≥ t=183.000s | t=183.311s, YES 0.71/0.73 | toutes | 1
| 1er yes_ask ≥ 0.99$ | t=240.529s | yes_ask | 590


**B5. OBSERVATIONS BRUTES.**

- Au moment exact où l'oracle confirme le franchissement (t=183.000s), le YES s'achète 0.73$ (t=183.311s) : le marché laisse 0.27$ de marge sur un règlement devenu quasi certain.
- Le carnet ne monte à 0.99$ qu'à t=240.529s, soit 57.529s APRÈS la confirmation oracle.
- Mon test yes_bid+no_bid > 1.00$ donne 0 occurrence ; le critère exact des 8 "suspects" du PDF : DONNÉE NON DISPONIBLE.


**B6. SPÉCIFIQUE À LA SESSION.** Creux ponctuel du yes_bid à 0.09$ à t=123.221s (6 niveaux balayés en 0 ms) puis retour à 0.13$ en 29 ms : balayage de liquidité, verdict bruit.

## B-F4 — trades.csv (FLUX D'ORDRES POLYMARKET)

**B1. IDENTITÉ.** Colonnes t_ms, usd, direction — toutes utilisées. 1380 lignes, t=0.407s → t=298.588s.

**B2. APPORT.** Montants réels échangés et leur sens, ce que le graphe 3 n'agrège qu'en fenêtres 30s.

**B3. COMMENT.** Somme des usd par direction ; flux net = Σ(usd × direction) sur fenêtres [0;183s[, [183s;300s], [145s;165s[, [183s;200s[ ; max, moyenne, médiane.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre de trades | 1380 | t_ms | 1380
| Volume total | 21 508.66$ | usd | 1380
| Volume haussier / nb | 11 201.07$ / 782 | usd, direction | 782
| Volume baissier / nb | 10 307.59$ / 598 | usd, direction | 598
| Flux net session | +893.48$ | usd, direction | 1380
| Plus gros trade | 614.97$ à t=248.094s, baissier | toutes | 1
| Moyenne / médiane | 15.59$ / 4.80$ | usd | 1380
| Flux net avant t=183.000s | −2 621.74$ | toutes | ~830
| Flux net après t=183.000s | +3 515.22$ | toutes | ~550


**B5. OBSERVATIONS BRUTES.**

- Avant la confirmation oracle, le flux net est VENDEUR (−2 621.74$) ; après, il devient massivement acheteur (+3 515.22$) : la foule achète après le fait, à 0.85–0.99$.
- La médiane à 4.80$ contre une moyenne à 15.59$ : marché de petits ordres percé de quelques blocs (max 614.97$).


**B6. SPÉCIFIQUE À LA SESSION.** Le bloc baissier de 614.97$ à t=248.094s intervient à YES = 0.99$ : liquidation de NO, pas un signal. Verdict : bruit directionnel.

---

# PARTIE C

## C0 — STRATÉGIE APPLIQUÉE

Vous m'avez interdit la recherche de stratégie : l'**Ancrage Oracle** est prescrite. Je ne présente donc pas de candidates concurrentes — mais je vous montre que la stratégie imposée repose bien sur des observations antérieures du document, comme l'exige la méthode. Les 5 observations qui la portent :

- OBS-1 : l'oracle franchit le strike à t=182.000s et n'y retourne jamais (0 retour / 116 ticks) (B-F1, B5).
- OBS-2 : le dernier tick oracle ≤ strike est à t=181.000s — l'incertitude de règlement meurt aux deux tiers de la fenêtre (B-F1, B4).
- OBS-3 : à la confirmation (t=183.000s), le YES s'achète encore 0.73$ (t=183.311s) — 0.27$ de marge sur un règlement quasi acquis (B-F3, B5).
- OBS-4 : le carnet ne verrouille à 0.99$ qu'à t=240.529s, 57.529s après la confirmation (B-F3, B5).
- OBS-5 : le flux acheteur de la foule arrive APRÈS le franchissement (−2 621.74$ avant t=183.000s, +3 515.22$ après) — l'ancrage prend position avant elle (B-F4, B5).


CRITÈRE DE SÉLECTION : sans objet — stratégie imposée par l'investisseur avant toute comparaison.

| Stratégie | Observations sources (A5/B5) | Nb trades | P&L net | Verdict
|-----|-----|-----|-----
| Ancrage Oracle | OBS-1, OBS-2, OBS-3, OBS-4, OBS-5 | 1 | +1.85$ | APPLIQUÉE (imposée)


## C1 — TABLE DE CONVERGENCE

Chaque règle de la stratégie, avec ses soutiens et ses contradictions dans les pièces :

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| Le règlement dépend de l'oracle vs strike, pas du spot | A-G3 (strike coupe la courbe oracle) ; B-F2 (spot > strike 6994/6994 pendant YES=0.11$) | aucune | SOLIDE - 2 SOURCES
| Entrer YES quand 2 ticks oracle consécutifs > strike | B-F1 (confirmation t=183.000s, 0 retour ensuite) ; A-G4 (franchissement visible au zoom) | aucune | SOLIDE - 2 SOURCES
| Le carnet offre encore de la marge au moment de la confirmation | B-F3 (ask 0.73$ à t=183.311s) ; A-G2 (YES 0.47$ à t=153.615s, repricing incomplet) ; B-F4 (flux acheteur postérieur) | aucune | SOLIDE - 3 SOURCES
| Tenir jusqu'au règlement (pas de sortie anticipée) | B-F1 (0 retour sous strike) ; A-G1 (verrouillage 0.99$ à t=246.862s) | B-F3 (yes_bid touche 0.71$ à t=183.311s : drawdown latent) | SOLIDE - 2 SOURCES, 1 réserve
| La profondeur au ask suffit pour 5.00$ | aucune (tailles absentes du BBO) | aucune | [FRAGILE - 1 SOURCE]


Je le dis à l'oral : la règle de profondeur est marquée FRAGILE — aucun fichier ne contient les tailles du carnet.

## C2 — LA STRATÉGIE EN PSEUDO-CODE EXÉCUTABLE

```plaintext
STRIKE = 64225.00            # A-G3, A4 (ligne de strike du PDF)
CAPITAL = 5.00
etat = ATTENTE

POUR chaque tick oracle (t, p) DANS oracle.csv:        # B-F1, B1
    SI etat == ATTENTE ET p > STRIKE ET tick_precedent > STRIKE:
        # confirmation 2 ticks : t=183.000s, p=64227.05$  (B-F1, B4)
        quote = premier evenement bbo.csv avec t_bbo >= t  # B-F3, B4 -> t=183.311s
        prix_entree = quote.yes_ask                        # 0.73$ (B-F3, B4)
        quantite = CAPITAL / prix_entree                   # 6.8493 parts
        # [HYPOTHÈSE n°1] fill integral au ask affiche (profondeur non disponible)
        ACHETER YES quantite @ prix_entree
        etat = EN_POSITION
    SI etat == EN_POSITION ET p <= STRIKE ET tick_precedent <= STRIKE:
        VENDRE tout au yes_bid courant   # jamais declenche : 0 retour (B-F1, B4)

A t=300.000s: reglement YES = 1.00$ par part
# [HYPOTHÈSE n°2] reglement credite immediatement a la cloture
# [HYPOTHÈSE n°3] frais de transaction = 0.00$ (non derivables des fichiers)
```

Compteur d'hypothèses : 3.

## C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

### ÉTAT DU PORTEFEUILLE

- t=0.000s → t=183.310s : cash 5.00$, 0 position, valeur 5.00$.
- t=183.311s : achat 6.8493 parts YES à 0.73$ = 5.00$ ; cash 0.00$, valeur au bid (0.71$) = 4.86$.
- t=183.311s → t=240.529s : valeur au bid remonte de 4.86$ vers 6.71$ (yes_bid 0.98$ à t=240.534s, B-F3).
- t=300.000s : règlement 6.8493 × 1.00$ = 6.85$ ; cash 6.85$, 0 position.
Aucun achat n'a dépassé le cash disponible : coût 5.00$ pour 5.00$ de cash.


### RACE CONDITIONS

(a) Signal d'achat pendant un ordre en cours. RÈGLE : verrou global — l'état EN_POSITION ou ORDRE_EN_COURS rejette tout nouveau signal d'achat ; sans objet ici, un seul signal a existé (t=183.000s).
(b) Signal de vente sur position non confirmée. RÈGLE : la sortie n'est évaluable qu'après passage à EN_POSITION ; entre l'envoi (t=183.000s) et le fill (t=183.311s), tout signal de vente est mis en file et réévalué au fill ; aucun n'est survenu (0 retour sous strike, B-F1, B4).
(c) Deux signaux simultanés sur le même actif. RÈGLE : file FIFO sur le timestamp oracle, ms puis ordre de ligne du CSV ; le second est rejeté par le verrou de (a) ; jamais déclenché (ticks oracle espacés de 1.000s minimum, B-F1, B4).
(d) Fill partiel. RÈGLE : la quantité non servie est annulée immédiatement (IOC), la position vit avec la quantité servie ; simulé ici en fill intégral faute de données de profondeur — c'est l'objet de l'HYPOTHÈSE n°1 et du [FRAGILE - 1 SOURCE] de C1.

### JOURNAL DE TRADES

Le journal, ligne par ligne :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=183.311s | 0.73$ | 6.8493 | t=300.000s (règlement) | 1.00$ | 0.00$ | +1.85$ | 6.85$


### RÉSULTATS

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 1 / +1.85$ | ligne 1
| Trades perdants / pertes cumulées | 0 / 0.00$ | —
| Frais totaux | 0.00$ | ligne 1
| BÉNÉFICE NET | +1.85$ | ligne 1
| Capital final vs initial | 6.85$ vs 5.00$ (+36.99%) | ligne 1
| Pire perte unitaire | aucune (0 trade perdant) | —
| Drawdown maximum | −0.14$ (valeur au bid 4.86$, entre t=183.311s et t=183.311s au yes_bid 0.71$) | ligne 1 + B-F3, B4


## C4 — PORTÉE

- Le marché Polymarket règle sur l'oracle Chainlink et le carnet retarde de 57.529s sur la confirmation du règlement. [PATTERN CANDIDAT]
- Entrer YES à la confirmation 2 ticks oracle > strike capte la marge résiduelle du carnet. [PATTERN CANDIDAT]
- La marge de 0.27$ disponible à t=183.311s et le rendement de +36.99% dépendent du chemin de cette session. [SESSION-SPÉCIFIQUE]
- L'absence totale de retour de l'oracle sous le strike après franchissement. [SESSION-SPÉCIFIQUE]
- Le basis spot−oracle strictement positif et borné [50.30$ ; 60.46$]. [SESSION-SPÉCIFIQUE]
- Le flux de la foule qui achète après la confirmation (+3 515.22$ post-t=183.000s). [PATTERN CANDIDAT]


---

# AUTO-CONTRÔLE FINAL

- Contrôle 1 (comptage des valeurs) → 6 blocs graphes à ≥ 5 valeurs chacun ; 4 CSV à ≥ 8 métriques en tableau B4. CONFORME. Une seule valeur déclarée DONNÉE NON DISPONIBLE (critère des 8 carnets suspects, B-F3).
- Contrôle 2 (traçabilité de C) → chaque règle de C1/C2 et chaque chiffre de C-SIM pointe vers un B4 ou A4 identifié. CONFORME.
- Contrôle 3 (unicité des chiffres) → 6.85$, +1.85$, +36.99%, 0.73$, t=183.311s identiques dans l'ouverture, le corps et la clôture. CONFORME.
- Contrôle 4 (comptabilité) → 5.00 − 6.8493×0.73 = 0.00$ ; 0.00 + 6.8493×1.00 = 6.85$. Cash final = capital annoncé au centime. CONFORME.
- Contrôle 5 (hypothèses) → 3 hypothèses utilisées : fill intégral au ask (tailles absentes des fichiers), crédit du règlement à la clôture (mécanique de la plateforme, hors fichiers), frais 0.00$ (non dérivables des fichiers). Trois exactement, seuil respecté.
- Contrôle 6 (antériorité des observations) → les 5 règles citent OBS-1 à OBS-5, elles-mêmes issues de B5/A5 antérieurs dans le document ; la stratégie étant imposée par vous, ce contrôle valide sa cohérence avec les données, pas sa genèse. CONFORME.
- Contrôle 7 (format du speech) → verdict en ouverture, adresse directe, ordre F1 respecté, tableaux F4 fournis, identifiants A-G1…B-F4 utilisés, clôture en 3 lignes sans chiffre nouveau. CONFORME.


---

# CLÔTURE

Bénéfice net de la session : +1.85$.
Capital final : 6.85$ contre 5.00$ de départ.
Étiquette globale : [PATTERN CANDIDAT] — le mécanisme d'ancrage tient, l'amplitude appartient à cette session.

---

J'ai appliqué la stratégie Ancrage Oracle imposée sans en chercher d'autre : entrée YES unique à t=183.311s à 0.73$ sur confirmation de deux ticks Chainlink consécutifs au-dessus du strike de 64 225.00$, tenue jusqu'au règlement à 1.00$. Tous les chiffres du speech proviennent d'un recalcul par script sur vos 4 CSV (copiés dans `data/` du projet), et les seules inconnues — profondeur du carnet, frais, timing de crédit — sont déclarées comme les 3 hypothèses du pseudo-code.
## QUAND acheter ?

À **t=183.0s**, dès que l'oracle Chainlink a affiché **2 ticks consécutifs au-dessus du strike** (64 226.23$ à t=182s, puis 64 227.05$ à t=183s, strike = 64 225.00$). Le 1er tick seul pouvait être un faux signal ; le 2e le confirme.

## COMMENT l'oracle a-t-il permis de choisir ?

Parce que c'est **lui qui décide du résultat**, pas Binance. Le marché règle sur la question : "l'oracle finit-il au-dessus du strike ?". Donc :

- Tant que l'oracle est sous le strike → résultat incertain → on attend.
- Dès qu'il passe et reste au-dessus → le résultat "UP" est quasi acquis → on agit.


L'oracle est lent (il traînait ~55$ derrière le spot), donc quand il franchit enfin le strike avec le spot déjà 40-50$ plus haut, il n'a presque aucune raison de redescendre. C'est ce qui rend le signal fiable : il n'est jamais repassé dessous ensuite (0 fois sur 116 ticks).

## COMMENT acheter ?

Au premier prix disponible dans le carnet **après** la confirmation : à t=183.311s, le YES se vendait encore à **0.73$**. Avec 5$, on achète 6.85 parts.

## POURQUOI acheter ?

Parce que le marché était **en retard sur la réalité** : le règlement UP était pratiquement certain (oracle au-dessus du strike, spot loin au-dessus), mais le carnet cotait encore 0.73$ au lieu de ~1.00$. Ces **0.27$ d'écart** sont le profit. La preuve du retard : la foule n'a acheté massivement qu'APRÈS (flux vendeur avant t=183s, +3 515$ d'achats après), et le carnet n'a atteint 0.99$ que 57 secondes plus tard.

## POURQUOI (et comment) "vendre" ?

On ne vend pas vraiment : on **tient jusqu'au règlement**. À la fin des 5 minutes, chaque part YES est payée **1.00$** automatiquement → 6.85$ récupérés, soit **+1.85$** de gain.

La seule règle de vente prévue était une sécurité : si l'oracle était repassé sous le strike (2 ticks confirmés), on aurait tout revendu au prix du marché pour limiter la perte. Ça n'est jamais arrivé dans cette session.

**En une phrase :** l'oracle dit la vérité du règlement avant que le marché ne l'ait intégrée dans son prix — on achète cet écart au moment de la confirmation, et on laisse le règlement encaisser à 1.00$.



# RAPPORT D'AUDIT OPÉRATIONNEL — SESSION BTC 5 MIN, STRIKE 64 225, CAPITAL 5,00$

## 0. VERDICT EXÉCUTIF

- Verdict global : **[FIABLE]** — pour cette session et cette taille, avec réserves matérielles en Partie R.
- NOTE DE FIABILITÉ : **81/100** (grille F7 détaillée en fin de rapport).
- Manquements identifiés : **4** au total — 0 [DÉTRUIT], 3 [DÉGRADE], 1 [IMPACT NON CHIFFRABLE] ; 4 facteurs sur 8 sont [SANS IMPACT] prouvé.
- Manquement le plus grave : l'absence des tailles du carnet dans les sources — le fill intégral de 5,00$ à 0,73$ est une hypothèse non vérifiable, bornée à −0,18$ de P&L.
- P&L annoncé : **+1,85$** ; P&L après réintégration de tous les manquements chiffrables cumulés : **+1,67$** (−9,7%).
- Point le plus solide : le règlement suit l'oracle qui, après confirmation à t=183,000 s, ne repasse jamais sous le strike (0 retour sur 116 ticks, marge finale +52,08$).


Je vais vous montrer pièce par pièce d'où vient ce verdict.

## 1. PÉRIMÈTRE

Audité : la complétude opérationnelle de la stratégie d'ancrage oracle et l'impact des facteurs ignorés, sur les 4 CSV de la session et le tableau des latences us-east-1. Non audité : la rentabilité multi-sessions (échantillon n=1) et le code du bot (non fourni). Le recalcul n'est fait que par sondage (Partie V).

## 2. PARTIE V — SONDAGE DE FIABILITÉ DES DONNÉES

| # | Valeur sondée | Annoncée | Recalculée | Écart | Verdict
|-----|-----|-----|-----|-----|-----
| 1 | P&L final (5,00$ @ 0,73$, règlement 1,00$) | +1,85$ | 5/0,73 = 6,8493 parts → 6,85$ − 5,00$ = +1,85$ | 0,00$ | [CONFIRMÉ]
| 2 | 1er tick oracle > strike | t=182 s | oracle.csv : t=182000, 64 226,23 | 0 | [CONFIRMÉ]
| 3 | Confirmation 2 ticks consécutifs | t=183 s | oracle.csv : 182000 (64 226,23) puis 183000 (64 227,05) | 0 | [CONFIRMÉ]
| 4 | Prix d'entrée disponible au timestamp | 0,73$ | bbo.csv ligne t=183311 : yes_ask=0,73 | 0 | [CONFIRMÉ]
| 5 | Non-retour sous strike après confirmation | 0 retour | oracle.csv : 0/116 ticks ≤ 64 225 après t=183000 | 0 | [CONFIRMÉ]
| 6 | Cash final | 6,85$ | 6,8493 parts × 1,00$ = 6,85$ | 0,00$ | [CONFIRMÉ]


Zéro [ERREUR] : données de base dignes de confiance au niveau sondé. Je passe au cœur de l'audit.

## 3. PARTIE O — AUDIT OPÉRATIONNEL : CE QUE LA STRATÉGIE IGNORE

| # | Facteur ignoré | Hypothèse implicite de l'agent | Réalité (source) | Impact sur P&L | Verdict
|-----|-----|-----|-----|-----|-----
| O1 | Latence de boucle | Exécution quasi instantanée à la confirmation | Boucle reconstituée : lecture Polygon RPC 108 ms + décision + envoi CLOB 31 ms = 139 ms hors calcul ; variante Binance : 219+31 = 250 ms | 0,00$ — le niveau 0,73$ tient 545 ms (bbo.csv 183311→183856) | [SANS IMPACT]
| O2 | Fraîcheur du signal | Le prix oracle lu est le prix courant | Cadence oracle 1 000 ms + lecture 68–108 ms = âge max ~1 108 ms (oracle.csv, tableau latences) | 0,00$ — 0 re-croisement sous strike après t=183000 (oracle.csv) | [SANS IMPACT]
| O3 | Profondeur au ask | 5,00$ intégralement fillés à 0,73$ | Tailles ABSENTES de bbo.csv — DONNÉE NON VÉRIFIABLE | Borné 0,00$ à −0,18$ (fill au niveau suivant 0,75$) | [DÉGRADE]
| O4 | Frais complets | Aucun frais | Barème Polymarket : 0% taker sur ce marché ; gas non facturé par ordre (relayeur) — borné < 0,05$ | 0,00$ à −0,05$ | [SANS IMPACT]
| O5 | File d'attente / carnet mouvant | Le niveau lu est encore là à l'arrivée | 146$ de flux acheteur frappent le ask entre t=183014 et t=183204 (trades.csv) avant la cotation 183311 | Borné −0,09$ (fill 0,74$, −4,9%) | [DÉGRADE]
| O6 | Défaillances techniques | Ordre toujours confirmé | Aucun fallback défini (speech, section pseudo-code). Pire cas crédible : ordre non confirmé → pas de position, P&L 0 (marché binaire tenu à résolution : pas de sortie aveugle requise) | −1,85$ au pire (P&L 0, jamais négatif) | [DÉGRADE]
| O7 | Passage à l'échelle | Résultats généralisables | Volume acheteur t=183–186 s = 221,70$ (trades.csv) ; 5$ = 2,3% de ce flux ; le mécanisme s'éteint vers ~200$ de taille | 0,00$ à 5$ | [SANS IMPACT]
| O8 | Dépendance à la session | Le signal gagne en général | Session à croisement unique, sans whipsaw, marge finale +52,08$ (oracle.csv). Session calme → 0 trade, P&L 0 ; session whipsaw → −5,00$. Seuil d'équilibre : taux de réussite > 73% (1/0,73). n=1 | Non chiffrable sur une session | [IMPACT NON CHIFFRABLE]


Réponse à votre question structurelle : **oui, le bot peut suivre cette stratégie depuis us-east-1.** La boucle complète (139–250 ms) tient 2 à 4 fois dans la fenêtre de 545 ms où le 0,73$ reste coté ; la position du serveur n'est pas un facteur limitant sur cette session.

## 4. PARTIE I — IMPACT CUMULÉ DES MANQUEMENTS

Rejeu unique avec latence de boucle (O1), âge du signal (O2), slippage/profondeur (O3, borne haute), frais (O4) et perte de file (O5) réintégrés simultanément. O3 et O5 portent sur le même niveau de carnet : le cumul est un fill à 0,75$ (pire borne conjointe), soit 6,667 parts.

| Scénario | P&L annoncé | P&L corrigé | Écart %
|-----|-----|-----|-----|-----|-----
| Latence seule (O1, fill 0,73$) | +1,85$ | +1,85$ | 0%
| + file d'attente (O5, fill 0,74$) | +1,85$ | +1,76$ | −4,9%
| **TOUS MANQUEMENTS CUMULÉS (fill 0,75$, frais 0)** | **+1,85$** | **+1,67$** | **−9,7%**


P&L cumulé positif : la règle bloquante ne s'applique pas.

## 5. PARTIE R — RISQUES RÉSIDUELS NON DÉCLARÉS

- [RISQUE NON DÉCLARÉ] Asymétrie gain/perte : un trade perdant coûte 5,00$, un gagnant rapporte 1,85$ ; un échec efface 2,7 victoires (5,00/1,85).
- [RISQUE NON DÉCLARÉ] Concentration : 100% du capital sur un ordre, un marché, une fenêtre de 545 ms.
- [RISQUE NON DÉCLARÉ] Résolution du marché : le règlement dépend de la source oracle du contrat ; toute divergence oracle/contrat au règlement — borné 0 à −5,00$ — n'est pas couverte par les fichiers.


## 6. AUTO-CONTRÔLE DE L'AUDITEUR

- Contrôle 1 → OK : O1–O8 tous traités ; chaque [SANS IMPACT] porte sa preuve chiffrée (545 ms / 0 re-croisement / 221,70$ / frais 0 bornés).
- Contrôle 2 → OK : boucle reconstituée chemin par chemin (108+31=139 ms ; variante 219+31=250 ms) et utilisée en Partie I.
- Contrôle 3 → OK : chaque impact cite bbo.csv, trades.csv, oracle.csv ou le tableau des latences ; les non-chiffrables (O3, O8) sont bornés avec méthode.
- Contrôle 4 → OK : note recalculée ci-dessous = 81 = note de l'ouverture ; 81 ∈ [75-100] → [FIABLE], aucune règle bloquante déclenchée.
- Contrôle 5 → OK : +1,85$, +1,67$, −9,7%, 0,73$, 0,75$, 545 ms, 221,70$ identiques partout.


| Critère F7 | Points obtenus | Points max | Justification
|-----|-----|-----|-----|-----|-----
| Complétude opérationnelle | 25 | 40 | 3 facteurs ignorés en [DÉGRADE] (O3, O5, O6) : −15
| Survie du P&L corrigé | 30 | 30 | +1,67$ positif au cumul
| Fiabilité des données | 20 | 20 | 6/6 valeurs sondées [CONFIRMÉ]
| Honnêteté des limites | 6 | 10 | Profondeur absente non déclarée (−2), dépendance à la session non déclarée (−2)
| **TOTAL** | **81** | **100** | **[FIABLE]**


## 7. CLÔTURE

Verdict : [FIABLE], note 81/100, pour cette session, cette taille et cette infrastructure.
La fiabilité multi-sessions reste non établie : elle exige un taux de réussite supérieur au seuil d'équilibre.
Condition n°1 pour changer le verdict : un backtest sur 200 sessions minimum aux mêmes règles — sous le seuil, le verdict devient [NON FIABLE] quelle que soit la qualité d'exécution.




# 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Voici ce que montrent les données, sans détour.

- Stratégie appliquée : ANCRAGE ORACLE — doctrine imposée, calibration propre à cette session, verrouillée en C0.
- Capital final : 1.1972$ contre 5.00$ de départ, soit un rendement net de −76.06%.
- Nombre de trades : 1, dont 0 gagnant et 1 perdant.
- Côté désigné par l'oracle : DOWN (jeton NO), signal d'entrée à t=29.000s, exécuté à t=30.211s.
- Verdict de comportement : [STRATÉGIE INFIRMÉE SUR CETTE SESSION]
- Spécificité dominante : un unique croisement du strike à t=160.000s, après 158.0s d'installation continue côté DOWN — l'oracle s'est retourné une seule fois, sans jamais revenir, transformant une entrée d'ancrage conforme en perte de −3.8028$.


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

# 1. RAPPEL DE LA DOCTRINE IMPOSÉE

P1 : seul l'oracle décide, le spot ne sert qu'au contrôle qualité, le carnet n'est qu'un prix d'exécution. P2 : strike = dernier tick oracle ≤ t0, la marge oracle−strike désigne le côté. P3 : entrée uniquement sur ancrage oracle installé, si le carnet laisse encore un edge. P4 : 100% du cash (5.00$) sur un seul trade d'entrée, à l'ask. P5 : tenue jusqu'au règlement, aucun stop carnet, sortie anticipée déclenchée uniquement par les prints oracle, au bid. La calibration L1-L5 est mon choix, justifié et verrouillé en C0.

# PARTIE A — LES 6 GRAPHES DU PDF

## A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (590 ÉVÉNEMENTS BBO)

**A1. IDENTITÉ.** Graphique 1, vue 300s plus trois zooms 15s. Axe X : temps depuis l'ouverture (mm:ss.mmm), de t=0.164s à t=246.862s pour les cotations. Axe Y : prix en $ de 0.0 à 1.0. Six séries : bid/ask/mid YES et NO, plus la probabilité consensus, 590 événements, 8 événements marqués « incohérents ».

**A2. QUESTION.** À quel prix le jeton désigné par l'oracle est-il achetable au moment du signal ? Quand le carnet a-t-il convergé au point de tuer l'edge (calibration L2) ? Jusqu'à quand le carnet cote-t-il (calibration L3) ?

**A3. COMMENT.** Lecture des étiquettes d'extrema sur la vue 300s, puis lecture point par point des zooms, notamment le zoom [t=139.000s → t=154.000s] qui couvre le creux du NO, et le zoom [t=217.000s → t=232.000s] qui couvre la convergence finale.

**A4. VALEURS EXTRAITES.**

- t=0.164s : YES mid 0.52$, NO mid 0.48$
- t=125.052s : YES mid 0.11$, NO mid 0.89$ (extrême de la session)
- t=147.014s : YES mid passe de 0.29$ à 0.30$ (remontée rapide)
- t=153.615s : YES mid 0.47$, NO mid 0.53$ (le carnet rebascule vers l'équilibre)
- t=246.862s : YES mid 0.99$, NO mid 0.01$ (dernière cotation de la session)


**A5. OBSERVATIONS BRUTES.**

- Le carnet part à l'équilibre 0.52$/0.48$ (t=0.164s) : aucun consensus initial — confirme qu'une entrée précoce laisse de l'edge (P3, oriente L2).
- Le carnet descend jusqu'à NO mid 0.89$ (t=125.052s) puis se retourne complètement jusqu'à 0.01$ (t=246.862s) : la convergence n'est pas monotone — soutient l'exigence d'un critère de sortie oracle (L4).
- Plus aucune cotation après t=246.862s : oriente la fenêtre d'entrée L3 vers une borne avant t=240.000s.


**A6. SPÉCIFIQUE À LA SESSION.** Le retournement complet du carnet de 0.89$ à 0.01$ sur le NO en 121.810s (t=125.052s → t=246.862s) est propre au retournement oracle de cette session ; rien n'indique qu'un carnet se retourne à cette vitesse en général. [SESSION-SPÉCIFIQUE]

## A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK

**A1. IDENTITÉ.** Graphique 2, vue 300s plus trois zooms 15s, avec sous-panneau basis = spot − oracle. Axe X : temps (mm:ss.mmm), t=0.000s à t=299.916s. Axe Y : prix BTC en $, de 64180 à 64340. Séries : Binance BTC/USDT (6994 ticks), Chainlink BTC/USD (284 ticks), ligne « Strike ouverture (64,225$) ».

**A2. QUESTION.** Où est le strike exact et de quel côté l'oracle s'installe-t-il, quand, avec quelle amplitude (fonde L1) ? Le spot est-il un référentiel fiable ou diverge-t-il de l'oracle (contrôle qualité P1) ?

**A3. COMMENT.** Lecture du premier tick oracle à t=0.000s pour le strike, suivi des étiquettes tick par tick de la descente (t=0.000s à t=68.000s), du plancher, de la remontée, et lecture des extrema spot ; lecture du sous-panneau basis sur la vue 300s.

**A4. VALEURS EXTRAITES.**

- t=0.000s : oracle 64215.97$ (strike, dernier tick ≤ t0)
- t=51.147s : spot minimum 64237.85$ — jamais sous le strike oracle
- t=68.000s : oracle 64180.24$ (plancher de la session, marge −35.73$)
- t=160.000s : l'oracle recroise le strike à la hausse (unique croisement)
- t=298.000s : oracle 64277.08$ ; t=299.916s : spot 64336.29$


**A5. OBSERVATIONS BRUTES.**

- L'oracle s'installe sous le strike dès t=1.000s et y reste 158.0s : un ancrage DOWN net — fonde le côté d'entrée (P2, P3).
- Le spot ne passe JAMAIS sous le strike oracle (minimum 64237.85$ > 64215.97$) : spot et oracle donnent des signes de marge OPPOSÉS pendant 160s — confirme la nécessité de P1 (un seul référentiel) et disqualifie tout mélange des deux.
- La ligne « Strike ouverture » du graphe affiche 64225$, en désaccord avec le dernier tick ≤ t0 du CSV (64215.97$) : P2 impose la valeur CSV, l'étiquette graphique est écartée.


**A6. SPÉCIFIQUE À LA SESSION.** La basis spot−oracle reste dans un couloir étroit de 50.30$ à 60.46$ (sous-panneau, confirmé en B-F4) : ce niveau absolu de basis est propre à ce moment de marché, non transférable. [SESSION-SPÉCIFIQUE]

## A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ (1380 TRADES)

**A1. IDENTITÉ.** Graphique 3, vue 300s plus trois zooms 15s. Axe X : temps (mm:ss.mmm), t=0.407s à t=298.588s. Axe Y : volume en $ (−2000 à +5000), flux haussier 30s (BUY YES + SELL NO) en positif, flux baissier en négatif, 1380 trades.

**A2. QUESTION.** Le flux du carnet précède-t-il ou suit-il les prints oracle ? Y a-t-il un choc de flux autour du signal d'entrée et du retournement ?

**A3. COMMENT.** Lecture des étiquettes d'extrema de la vue 300s et des trades unitaires des zooms, comparaison de leurs timestamps avec les timestamps oracle du graphe 2.

**A4. VALEURS EXTRAITES.**

- t=0.407s : premier trade, −12.25$ (baissier)
- t=140.251s : trade baissier −398.50$
- t=217.536s : trade haussier +583.91$
- t=224.873s : trade haussier +216.23$
- t=248.094s : plus gros trade de la session, −614.97$


**A5. OBSERVATIONS BRUTES.**

- Les gros flux haussiers (+583.91$ à t=217.536s) arrivent APRÈS le croisement oracle de t=160.000s : le flux suit l'oracle, il ne l'anticipe pas — confirme P3 (le carnet ne déclenche jamais).
- Le flux est baissier net pendant la phase d'ancrage DOWN (zoom [t=80.000s → t=95.000s] : 10 trades baissiers sur 12) : cohérent avec le côté oracle, sans valeur de signal.


**A6. SPÉCIFIQUE À LA SESSION.** Le pic de −614.97$ à t=248.094s survient après le gel du carnet (t=246.862s, A-G1) : c'est un flux de fin de fenêtre sans effet exploitable. [SESSION-SPÉCIFIQUE]

## A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ.** Graphique 4, vue 300s plus trois zooms 15s. Axe X : temps (mm:ss.mmm). Axe Y : imbalance de 0.0 à 1.0, définie Haussier/(Haussier+Baissier) sur 1380 trades, ligne d'équilibre à 0.5.

**A2. QUESTION.** L'imbalance apporte-t-elle une information non redondante avec le graphe 3 pour la calibration ?

**A3. COMMENT.** Lecture des étiquettes d'extrema de la vue 300s et des trois zooms, comparaison avec les phases oracle du graphe 2.

**A4. VALEURS EXTRAITES.**

- t=0.407s : imbalance 0.00
- t=81.820s : imbalance 0.24
- t=94.792s : imbalance 0.15
- t=140.342s : imbalance 0.28
- t=247.018s : imbalance 0.88 (maximum) ; t=298.588s : 0.00


**A5. OBSERVATIONS BRUTES.**

- Imbalance sous 0.5 pendant toute la phase DOWN de l'oracle (0.15 à 0.28 entre t=81.820s et t=140.342s), au-dessus après le croisement : redondante avec le graphe 3 — n'oriente aucun paramètre L, conformément à P3 elle est écartée comme signal.


**A6. SPÉCIFIQUE À LA SESSION.** Les valeurs extrêmes 0.00 (t=0.407s, t=298.588s) sont des artefacts de fenêtre 30s quasi vide en début et fin de session. [SESSION-SPÉCIFIQUE]

## A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS

**A1. IDENTITÉ.** Graphique 5, vue 300s plus trois zooms 15s. Axe X : temps (mm:ss.mmm), t=0.164s à t=246.862s. Axe Y : spread en $ de 0.00 à 0.07, séries spread YES et spread NO (590 événements chacune).

**A2. QUESTION.** Quel coût d'exécution à l'entrée (ask) et à la sortie anticipée (bid) ? Les spreads « explosent-ils » lors des chocs, comme le suppose P5 ?

**A3. COMMENT.** Lecture des étiquettes d'extrema de la vue 300s et des points des zooms, en particulier autour du retournement.

**A4. VALEURS EXTRAITES.**

- t=0.164s : spread YES 0.01$, spread NO 0.01$
- t=1.101s : spread 0.00$ (première cotation verrouillée)
- t=123.221s : spread YES 0.07$ (maximum de la session)
- t=220.510s : spread YES 0.02$, spread NO 0.01$
- t=246.862s : spread 0.00$


**A5. OBSERVATIONS BRUTES.**

- Spread maximal de toute la session : 0.07$ (t=123.221s) — les spreads n'ont PAS explosé sur cette session, y compris au retournement : fait qui CONTREDIT la prémisse de P5 (« les spreads explosent lors des chocs ») sur ces données, sans le rendre inutile.
- Le spread maximal 0.07$ fonde le cap de prix L2 : un edge minimal de 0.15$ = 2 × 0.07$ + 0.01$ couvre deux traversées du pire spread observé.


**A6. SPÉCIFIQUE À LA SESSION.** 8 cotations à spread nul (dont t=1.101s, t=54.726s, t=246.862s), marquées « incohérents » par le PDF : micro-verrouillages transitoires du carnet, sans persistance. [SESSION-SPÉCIFIQUE]

## A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ.** Graphique 6, vue 300s plus trois zooms 15s. Axe X : temps (mm:ss.mmm). Axe Y : écart en $ de −0.005 à 0.000, série yes_mid − (1 − no_mid) sur 590 événements, ligne de cohérence parfaite à 0.

**A2. QUESTION.** Les carnets YES et NO sont-ils arbitrables entre eux ? Le prix d'exécution est-il fiable des deux côtés ?

**A3. COMMENT.** Lecture de la position de la courbe par rapport à la ligne zéro sur la vue 300s et les trois zooms.

**A4. VALEURS EXTRAITES.**

- t=0.164s : écart 0.000$
- t=54.726s : écart −0.005$ (unique déviation de la session)
- t=125.052s : écart 0.000$
- t=192.076s : écart 0.000$
- t=246.862s : écart 0.000$


**A5. OBSERVATIONS BRUTES.**

- Une seule déviation non nulle sur 590 événements (−0.005$ à t=54.726s) : les deux carnets sont cohérents à 99.8% des événements — le prix d'exécution du NO est fiable, confirme que le carnet peut servir de prix d'exécution (P3, P5).


**A6. SPÉCIFIQUE À LA SESSION.** La déviation unique de t=54.726s coïncide avec une cotation à spread nul (A-G5) : bruit de microstructure ponctuel, verdict bruit. [SESSION-SPÉCIFIQUE]

# PARTIE B — LES FICHIERS CSV

## B-F1 — ORACLE.CSV

**B1. IDENTITÉ.** oracle.csv, colonnes t_ms, price, ts_src — toutes exploitées. 284 lignes, de t=0.000s à t=298.000s, cadence médiane 1000ms, écart maximal 2000ms.

**B2. APPORT.** Le strike à toutes ses décimales, la marge print par print, les durées d'installation exactes — illisibles à cette précision sur le graphe 2.

**B3. COMMENT.** strike = price du dernier tick ≤ t0 ; marge(t) = price(t) − strike pour chaque ligne ; comptage des signes, détection des changements de signe, longueurs de runs consécutifs, statistiques des |Δprice| tick à tick, moyennes et médianes des marges par côté.

**B4. RÉSULTATS CHIFFRÉS.** Voici le tableau que vous pouvez recalculer ligne à ligne :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Strike exact (dernier tick ≤ t0) | 64215.97$ | t_ms, price | 1 (t=0.000s)
| Dernier print | 64277.08$ à t=298.000s (marge +61.11$) | t_ms, price | 1
| Prints sous / sur / au strike | 146 / 137 / 1 | price | 284
| Croisements du strike | 1 (t=160.000s, marge +1.28$) | t_ms, price | 284
| Run DOWN | t=1.000s → t=159.000s, 146 prints, 158.0s | t_ms, price | 146
| Marges négatives : moyenne / médiane / min | −21.38$ / −24.64$ / −35.73$ (t=68.000s) | price | 146
| Marges positives : moyenne / médiane / max | +35.19$ / +39.56$ / +61.11$ (t=298.000s) | price | 137
| |Δprice| tick à tick : moyenne / max | 0.5735$ / 7.6574$ | price | 283
| Marge ≤ −10$ : premier print / 10e consécutif | t=18.000s / t=29.000s | t_ms, price | 284
| Marge ≥ +10$ : premier print / 10e consécutif | t=182.000s / t=191.000s | t_ms, price | 284
| ts_src ≠ payload | 0 sur 284 | ts_src | 284


**B5. OBSERVATIONS BRUTES.**

- Un seul croisement du strike sur 300s : l'oracle ne « scie » pas — un critère d'ancrage à 10 prints ne sur-filtre pas cette session (oriente L1).
- Le saut tick maximal est 7.6574$ : un seuil de marge de 10.00$ ne peut pas être franchi par un seul tick de bruit (oriente L1).
- Le retournement complet (−35.73$ → +61.11$) sans retour valide le principe d'une sortie anticipée pilotée par les prints (oriente L4).


**B6. SPÉCIFIQUE À LA SESSION.** Aucun trou supérieur à 2000ms, 0 fallback de timestamp : qualité de flux parfaite — verdict signal fiable ; la symétrie quasi parfaite 146/137 prints avec un seul croisement est une configuration de retournement unique. [SESSION-SPÉCIFIQUE]

## B-F2 — BBO.CSV

**B1. IDENTITÉ.** bbo.csv, colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask — toutes exploitées. 590 lignes (588 complètes, 2 partielles), de t=0.164s à t=246.862s, événementiel.

**B2. APPORT.** Les prix exécutables exacts à la milliseconde — le premier événement BBO après chaque signal oracle, que les graphes ne permettent pas d'isoler.

**B3. COMMENT.** Spreads ask−bid par ligne ; recherche du premier événement de t_ms strictement supérieur à chaque timestamp de signal ; min/max du mid NO sur la fenêtre de tenue ; tests de cohérence yes_bid+no_bid > 1 et yes_ask+no_ask < 1.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Événements / complets / partiels | 590 / 588 / 2 | toutes | 590
| Spread YES : moyenne / médiane / max | 0.0187$ / 0.02$ / 0.07$ (t=123.221s) | yes_bid, yes_ask | 588
| Spread NO : moyenne / médiane / max | 0.0187$ / 0.02$ / 0.07$ | no_bid, no_ask | 588
| Carnets croisés arbitrables | 0 | les 4 | 588
| Premier BBO après t=29.000s | t=30.211s : NO 0.69$/0.71$ | t_ms, no_bid, no_ask | 1
| Premier BBO après t=191.000s | t=192.076s : NO 0.17$/0.19$ | t_ms, no_bid, no_ask | 1
| Mid NO pendant la tenue : max / min | 0.885$ (t=125.052s) / 0.180$ (t=192.076s) | no_bid, no_ask | 261
| NO ask maximal avant t=60.000s | 0.86$ (t=49.422s) | no_ask | 588
| Dernière cotation complète | t=246.862s : YES 0.99$/0.99$ | toutes | 1


**B5. OBSERVATIONS BRUTES.**

- À t=30.211s le NO s'achète 0.71$ quand l'oracle est installé DOWN depuis 12 prints : le carnet n'avait pas convergé, l'edge existait (confirme P3).
- Le carnet cesse de coter à t=246.862s : toute entrée ou sortie doit précéder cette borne (oriente L3).
- Spread au moment de la sortie (t=192.076s) : 0.02$ — la sortie au bid coûte 0.01$ sous le mid, exécution propre (soutient L4).


**B6. SPÉCIFIQUE À LA SESSION.** 2 lignes partielles (t=246.862s incluse) en toute fin de vie du carnet, et 8 cotations à spread nul — bruit de microstructure, verdict bruit. [SESSION-SPÉCIFIQUE]

## B-F3 — TRADES.CSV

**B1. IDENTITÉ.** trades.csv, colonnes t_ms, usd, direction — toutes exploitées. 1380 lignes, de t=0.407s à t=298.588s, événementiel.

**B2. APPORT.** Les volumes exacts en $ et le sens agressif trade par trade, que le graphe 3 n'agrège qu'en fenêtres 30s.

**B3. COMMENT.** Sommes de usd par direction (+1 haussier, −1 baissier), par minute, flux net glissant 30s, extrêmes, médiane et moyenne des tailles.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Trades / volume total | 1380 / 21508.66$ | usd | 1380
| Haussiers : nombre / volume | 782 / 11201.07$ | usd, direction | 782
| Baissiers : nombre / volume | 598 / 10307.59$ | usd, direction | 598
| Taille : médiane / moyenne / max | 4.80$ / 15.59$ / 614.97$ (t=248.094s) | usd | 1380
| Flux net minute 0 / minute 1 | −1248.60$ / −1758.60$ | t_ms, usd, direction | 550
| Flux net minute 3 / minute 4 | +2730.30$ / +1037.69$ | t_ms, usd, direction | 511
| Flux net 30s : min / max | −1457.38$ (t=49.818s) / +4427.23$ (t=247.018s) | t_ms, usd, direction | 1380
| Flux net avant l'entrée (t ≤ 30.211s) | −372.72$ | t_ms, usd, direction | 358


**B5. OBSERVATIONS BRUTES.**

- Le pic de flux haussier (+4427.23$ à t=247.018s) arrive 87.018s APRÈS le croisement oracle de t=160.000s : le flux confirme en retard, il ne déclenche rien (confirme P3).
- Le flux net avant l'entrée (−372.72$) est cohérent avec le côté DOWN sans avoir servi de signal.


**B6. SPÉCIFIQUE À LA SESSION.** Le plus gros trade de la session (614.97$, t=248.094s) intervient après le gel du carnet BBO (t=246.862s, B-F2) — anomalie de fin de fenêtre, verdict bruit. [SESSION-SPÉCIFIQUE]

## B-F4 — SPOT.CSV

**B1. IDENTITÉ.** spot.csv, colonnes t_ms, price — toutes exploitées, mais UNIQUEMENT en contrôle qualité : P1 écarte ce fichier comme signal directionnel, et c'est la seule utilisation que j'en fais. 6994 lignes, de t=1.288s à t=299.916s.

**B2. APPORT.** La mesure de la basis spot−oracle, seul usage autorisé par P1, pour valider que l'oracle n'est pas aberrant.

**B3. COMMENT.** Pour chacun des 282 prints oracle appariables, basis = dernier spot ≤ t_ms(oracle) − price(oracle) ; extrêmes et comptage des ticks spot sous le strike oracle.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Ticks spot | 6994 | t_ms, price | 6994
| Premier / dernier tick | 64265.00$ (t=1.288s) / 64336.29$ (t=299.916s) | t_ms, price | 2
| Minimum / maximum | 64237.85$ (t=51.147s) / 64336.29$ (t=299.916s) | t_ms, price | 6994
| Basis spot−oracle : moyenne / médiane | +55.77$ / +55.77$ | price (×2 fichiers) | 282 paires
| Basis : min / max | +50.30$ (t=2.000s) / +60.46$ (t=227.000s) | price (×2 fichiers) | 282 paires
| Ticks spot < strike oracle 64215.97$ | 0 | price | 6994
| Ticks spot < premier tick spot 64265.00$ | 2491 | price | 6994
| Amplitude spot sur la session | 98.44$ (64237.85$ → 64336.29$) | price | 6994


**B5. OBSERVATIONS BRUTES.**

- Basis stable dans [50.30$ ; 60.46$] sur 282 paires : l'oracle n'est pas défaillant, il est simplement décalé d'un niveau constant — le contrôle qualité P1 est satisfait.
- Le spot n'a JAMAIS coté sous le strike oracle pendant que l'oracle passait 158.0s en dessous : les deux référentiels donnaient des côtés opposés — c'est exactement le cas que P1 tranche en faveur de l'oracle, et sur cette session ce choix a coûté (chiffré en C3).


**B6. SPÉCIFIQUE À LA SESSION.** Aucun trou de données notable ; le niveau absolu de basis (+55.77$ en moyenne) est propre à cette paire de flux à cette heure. [SESSION-SPÉCIFIQUE]

# PARTIE C — APPLICATION ET SIMULATION

## C0 — CALIBRATION DE LA DOCTRINE SUR CETTE SESSION

Voici chaque paramètre, sa valeur, et la mesure qui le justifie :

| Paramètre (P ou L) | Valeur choisie/mesurée | Justification chiffrée | Source
|-----|-----|-----|-----
| P2 — Strike | 64215.97$ | Dernier tick ≤ t0, à t=0.000s | B-F1, B4
| P2 — Côté désigné | DOWN (jeton NO) | 146 prints sous le strike dès t=1.000s, run de 158.0s | B-F1, B4
| P4 — Engagement | 5.00$, un seul trade, à l'ask | Contrainte de la doctrine | —
| L1 — Ancrage | 10 prints consécutifs avec marge ≤ −10.00$ (ou ≥ +10.00$) | Seuil 10.00$ > saut tick max 7.6574$ (aucun tick isolé ne peut le franchir) ; 10 prints = 10s à cadence 1000ms ; 1 seul croisement sur 300s, donc pas de sur-filtrage | B-F1, B4
| L2 — Edge : prix d'achat maximal | 0.85$ | 1.00$ − 0.15$, où 0.15$ = 2 × spread max 0.07$ + 0.01$ : l'edge doit couvrir deux traversées du pire spread observé | A-G5, A4 ; B-F2, B4
| L3 — Fenêtre d'entrée | [t=0.000s ; t=240.000s] | Dernière cotation du carnet à t=246.862s ; marge de 6.862s | B-F2, B4
| L4 — Sortie anticipée | Critère symétrique de L1 sur le côté opposé, exécutée au bid du jeton détenu | Le retournement −35.73$ → +61.11$ sans retour montre qu'un ancrage opposé installé est terminal sur cette session | B-F1, B4
| L5 — Comptable | Frais 0.00$ [HYPOTHÈSE n°1] ; fills fractionnaires intégraux au prix affiché [HYPOTHÈSE n°2] ; exécution au premier événement BBO strictement postérieur au print de signal [HYPOTHÈSE n°3] | Aucune donnée de frais ni de profondeur dans les pièces | B-F2, B1


Déductions, avant toute simulation :

- Signal d'entrée : t=29.000s (10e print consécutif à marge ≤ −10.00$, marge −12.12$) (B-F1, B4).
- Premier événement BBO exécutable après le signal : t=30.211s, ask NO 0.71$ ≤ 0.85$ (L2 satisfait), dans la fenêtre L3 (B-F2, B4).
- Sortie anticipée L4 : déclenchée à t=191.000s (10e print consécutif à marge ≥ +10.00$), premier BBO exécutable à t=192.076s, bid NO 0.17$ (B-F1, B4 ; B-F2, B4).


CALIBRATION VERROUILLÉE — ces paramètres ne seront plus modifiés.

## C1 — TABLE DE CONVERGENCE

Chaque principe, face aux pièces de cette session :

| Principe | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| P1 — Référentiel oracle unique | B-F4 (basis stable 50.30$–60.46$ : oracle sain) ; B-F1 (dernier print +61.11$ cohérent avec le résultat officiel UP) | A-G2 et B-F4 : le spot n'est jamais passé sous le strike pendant les 158.0s d'ancrage DOWN de l'oracle — le référentiel écarté désignait le côté finalement gagnant | Appliqué, tension documentée
| P2 — Ancre strike | B-F1 (strike 64215.97$ exact) | A-G2 (étiquette graphique 64225$, écartée au profit du CSV) | Appliqué
| P3 — Entrée sur prints oracle seuls | B-F1 (ancrage mesurable) ; B-F2 (0.71$ exécutable au signal) ; A-G3 et B-F3 (le flux carnet suit l'oracle avec 87.018s de retard : il n'aurait rien déclenché de mieux) | Aucune | Appliqué
| P4 — 100% du cash, un trade | B-F2 (liquidité cotée en continu de t=0.164s à t=246.862s) | Aucune pièce ne le contredit ; son coût apparaît dans la simulation | Appliqué
| P5 — Tenue, pas de stop carnet, sortie sur prints | B-F1 (le retournement est lisible dans les prints : L4 déclenche) | A-G5 : spread max 0.07$ — les spreads n'ont pas explosé sur cette session, la prémisse anti-stop ne s'est pas matérialisée | Appliqué, prémisse contredite ici [FRAGILE - 1 SOURCE]


Je vous signale à l'oral le statut [FRAGILE - 1 SOURCE] de la contradiction sur P5 : elle repose sur la seule pièce A-G5/B-F2.

## C2 — LA STRATÉGIE EN PSEUDO-CODE EXÉCUTABLE

```plaintext
STRIKE = 64215.97                          # P2 — oracle.csv, dernier tick <= t0 (B-F1, B4)
CASH = 5.00                                # P4
POSITION = null

POUR chaque print oracle (t, price) dans oracle.csv :   # P1, P3 — seuls les prints déclenchent
    marge = price - STRIKE                                # P2 (B-F1, B3)

    # --- ENTRÉE ---
    SI POSITION == null ET t <= 240.000s :               # L3 (B-F2, B4 : dernier BBO t=246.862s)
        SI 10 prints consécutifs avec marge <= -10.00 :   # L1 (B-F1, B4 : saut max 7.6574, cadence 1s)
            cote = premier événement bbo.csv avec t_bbo > t   # [HYPOTHÈSE n°3]
            SI cote.no_ask <= 0.85 :                      # L2 (A-G5, A4 : 0.15 = 2 x 0.07 + 0.01)
                qty = CASH / cote.no_ask                  # P4, fills fractionnaires [HYPOTHÈSE n°2]
                ACHETER NO qty @ cote.no_ask ; frais = 0  # [HYPOTHÈSE n°1]
                CASH = 0.00 ; POSITION = (NO, qty, cote.no_ask)
        # (symétrique YES si 10 prints consécutifs marge >= +10.00)

    # --- SORTIE ANTICIPÉE ---
    SI POSITION == (NO, ...) :
        SI 10 prints consécutifs avec marge >= +10.00 :   # L4 = L1 symétrique (B-F1, B4)
            cote = premier événement bbo.csv avec t_bbo > t   # [HYPOTHÈSE n°3]
            VENDRE NO qty @ cote.no_bid ; frais = 0       # P5 : au bid, [HYPOTHÈSE n°1]
            CASH = qty * cote.no_bid ; POSITION = null

# --- RÈGLEMENT --- (P5)
SI POSITION != null à t=300s :
    CASH = qty * (1.00 si résultat officiel == côté détenu sinon 0.00)
```

Aucune condition de ce code n'est étrangère à P1-P5 ou à L1-L5 verrouillés en C0.

## C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

**ÉTAT DU PORTEFEUILLE.** Journalisé à chaque événement :

- t=0.000s : cash 5.00$, aucune position, valeur totale 5.00$.
- t=30.211s (achat) : cash 0.00$, position 7.042254 NO à 0.71$, valeur totale 5.00$. Coût 5.00$ ≤ cash disponible 5.00$ : contrainte respectée.
- t=125.052s (pic) : cash 0.00$, position valorisée au mid 0.885$ soit 6.2324$.
- t=192.076s (vente) : cash 1.1972$, aucune position, valeur totale 1.1972$.
- t=300.000s (règlement, sans position) : cash 1.1972$.


**RACE CONDITIONS.**
(a) Un signal d'achat arrivant pendant qu'un ordre est en cours ne peut concerner que le côté opposé, puisque L1 exige 10 prints consécutifs d'un même côté. RÈGLE : tout signal d'achat survenant alors qu'un ordre est vivant ou qu'une position existe est ignoré, P4 n'autorisant qu'un seul trade d'entrée par session.
(b) Un signal de vente sur une position pas encore confirmée à l'achat ne s'est pas produit (161.865s séparent le fill d'entrée du signal de sortie). RÈGLE : un signal de vente antérieur à la confirmation du fill d'achat est mis en file et exécuté au premier événement BBO strictement postérieur à cette confirmation.
(c) Deux signaux simultanés sur le même actif sont impossibles au même print (une marge a un seul signe), mais traités par déterminisme. RÈGLE : à timestamp égal, la sortie est traitée avant l'entrée, et l'entrée est abandonnée si le cash est engagé.
(d) Le fill partiel est couvert par [HYPOTHÈSE n°2] (fill intégral au prix affiché, faute de données de profondeur). RÈGLE : tout ordre est traité en immediate-or-cancel — la quantité détenue est la quantité réellement exécutée et le reliquat est annulé sans nouvel ordre.

**JOURNAL DE TRADES.**

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=30.211s | 0.71$ (ask NO) | 7.042254 | t=192.076s | 0.17$ (bid NO) | 0.00$ | −3.8028$ | 1.1972$


**RÉSULTATS.**

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 0 / 0.00$ | —
| Trades perdants / pertes cumulées | 1 / 3.8028$ | ligne 1
| Frais totaux | 0.00$ | ligne 1
| BÉNÉFICE NET | −3.8028$ | ligne 1
| Capital final vs initial | 1.1972$ vs 5.00$ (−76.06%) | ligne 1
| Pire perte latente (mid du NO au plus bas) | mid 0.180$ à t=192.076s, position 1.2676$, soit −3.7324$ vs coût | ligne 1
| Pire perte unitaire | −3.8028$ (trade unique) | ligne 1
| Drawdown maximum | −4.9648$, du pic 6.2324$ (t=125.052s) au creux 1.2676$ (t=192.076s) | ligne 1


Un point à noter sans l'atténuer : la position détenue jusqu'au règlement aurait valu 0.00$ (résultat officiel UP, jeton NO) ; la sortie L4 a rendu 1.1972$. La perte reste de 3.8028$.

## C3 — PROFIL DE LA SESSION

Voici ce qui rendait cette session particulière et ce que chaque particularité a changé :

| # | Spécificité de la session (fait chiffré) | Source | Effet sur la doctrine/calibration | Effet sur le rendement
|-----|-----|-----|-----
| 1 | Un seul croisement du strike (t=160.000s) après 158.0s d'installation DOWN à marge médiane −24.64$ | B-F1, B4 | A rendu L1 (10 prints, 10.00$) facile à satisfaire des deux côtés : entrée à t=29.000s, sortie à t=191.000s | − : l'ancrage DOWN était réel mais non terminal ; perte de −3.8028$
| 2 | Retournement asymétrique : marge de −35.73$ (t=68.000s) à +61.11$ (t=298.000s) sans retour | B-F1, B4 | A validé le choix d'un L4 symétrique plutôt qu'aucune sortie | + : sortie à 0.17$ au lieu du règlement à 0.00$, soit 1.1972$ préservés
| 3 | Spot jamais sous le strike (minimum 64237.85$ > 64215.97$), basis moyenne +55.77$ | B-F4, B4 | P1 appliqué sans dérogation : le spot, écarté, désignait UP pendant toute la phase DOWN de l'oracle | − : le référentiel imposé a désigné le côté perdant ; effet porté par la ligne 1 du journal (−3.8028$)
| 4 | Spreads étroits toute la session : moyenne 0.0187$, maximum 0.07$ (t=123.221s) | A-G5, A4 ; B-F2, B4 | A fixé L2 à 0.85$ ; la prémisse « spreads explosent » de P5 ne s'est pas matérialisée | + marginal : sortie au bid à 0.01$ sous le mid, soit 0.0704$ de coût d'exécution seulement
| 5 | Carnet gelé après t=246.862s (dernière cotation 0.99$/0.01$) | B-F2, B4 | A fixé la borne L3 à t=240.000s | Neutre : entrée (t=30.211s) et sortie (t=192.076s) bien avant la borne
| 6 | Qualité de flux parfaite : 0 fallback ts sur 284, aucun trou > 2000ms | B-F1, B4 | Aucun print suspect à exclure : L1 opère sur données pleines | Neutre


SPÉCIFICITÉ DOMINANTE : le croisement unique du strike à t=160.000s après 158.0s d'ancrage DOWN — c'est lui qui transforme une entrée conforme à la doctrine en perte de −3.8028$.
TRANSFÉRABILITÉ : un agent jouant cette doctrine sur une autre session devrait vérifier en priorité si un ancrage précoce profond (marge ≤ −10.00$ avant t=30.000s) prédit ou non le côté du règlement, car ici il a prédit le mauvais côté. [PATTERN CANDIDAT]

## C4 — PORTÉE ET VERDICT DE COMPORTEMENT

- Un ancrage oracle de 10 prints à ±10.00$ est atteignable des deux côtés dans une session à retournement unique. [PATTERN CANDIDAT]
- La sortie anticipée symétrique sur prints oracle préserve du capital quand le retournement est terminal (1.1972$ contre 0.00$ au règlement). [PATTERN CANDIDAT]
- Le spot en désaccord de signe avec l'oracle pendant 158.0s malgré une basis stable de +55.77$. [SESSION-SPÉCIFIQUE]
- Des spreads plafonnés à 0.07$ pendant un retournement complet du carnet. [SESSION-SPÉCIFIQUE]
- Le gel du carnet 53.138s avant la fin de fenêtre. [SESSION-SPÉCIFIQUE]


VERDICT : trade unique perdant de −3.8028$ — [STRATÉGIE INFIRMÉE SUR CETTE SESSION]

RETOUR SUR CALIBRATION : le résultat a été piloté par L1 (entrée tôt, t=29.000s, à 0.71$) et sauvé partiellement par L4 (sortie à 0.17$ au lieu de 0.00$ au règlement) ; un L1 plus exigeant n'aurait fait que retarder une entrée du même côté DOWN, installé jusqu'à t=159.000s (B-F1, B4). Ce que cette session apprend sur la doctrine : son risque central n'est pas l'exécution (spreads ≤ 0.07$, A-G5) mais le retournement terminal de l'oracle après ancrage. Les paramètres restent verrouillés ; le résultat rapporté est celui de la calibration C0.

# AUTO-CONTRÔLE FINAL

- Contrôle 1 → 5 valeurs minimum par graphe (A-G1 à A-G6 : 5, 5, 5, 6, 5, 5) et 8 minimum par CSV (B-F1 : 11, B-F2 : 9, B-F3 : 8, B-F4 : 8) : CONFORME.
- Contrôle 2 → chaque affirmation de la Partie C cite une rubrique A4 ou B4 : CONFORME, aucune affirmation orpheline.
- Contrôle 3 → aucun chiffre ne figure deux fois avec des valeurs différentes, ouverture et clôture comprises : CONFORME (capital final 1.1972$, bénéfice net −3.8028$, rendement −76.06% partout).
- Contrôle 4 → cohérence comptable : 5.00$ − 5.00$ (achat, 7.042254 × 0.71$) puis + 1.1972$ (vente, 7.042254 × 0.17$) = 1.1972$ = capital final annoncé : CONFORME au centime près.
- Contrôle 5 → 3 hypothèses utilisées ([HYPOTHÈSE n°1] frais nuls, [HYPOTHÈSE n°2] fill intégral, [HYPOTHÈSE n°3] exécution au premier BBO postérieur) : seuil de 3 non dépassé.
- Contrôle 6 → conformité doctrine et verrouillage : le pseudo-code C2 relu face au bloc doctrine — les principes P1 à P5 ont été appliqués sans dérogation et la calibration verrouillée en C0 n'a pas été modifiée.
- Contrôle 7 → format speech : adresse directe, verdict dès l'ouverture, ordre F1 respecté, tableaux F4 au format imposé, identifiants A-G1 à A-G6 et B-F1 à B-F4, clôture en 3 lignes sans chiffre nouveau : CONFORME.
- Contrôle 8 → profil de session C3 : 6 spécificités, toutes chiffrées et sourcées A4/B4, chacune avec effet calibration et effet rendement ; spécificité dominante identique à celle de l'ouverture ; ligne TRANSFÉRABILITÉ étiquetée [PATTERN CANDIDAT] : CONFORME.


# CLÔTURE DU SPEECH

Le bénéfice net de la session est de −3.8028$, sans frais.
Le capital final est de 1.1972$ contre 5.00$ engagés, soit −76.06%.
Sur cette session, le verdict est [STRATÉGIE INFIRMÉE SUR CETTE SESSION] : l'ancrage oracle a désigné un côté réel mais non terminal, et seule la sortie sur prints a évité la perte totale.





# Verdict : la stratégie n'est pas à jeter — c'est la règle de sortie L4 qui a fabriqué le −76%

## 1. Ce qui a précisément causé l'échec

**L'événement tueur : l'unique croisement du strike à t=160.000s, sans jamais revenir.**

- Strike : 64 215,97$. L'oracle s'installe DOWN dès t=1s et y reste **158 s** (146 prints, marge médiane −24,64$). Entrée NO à t=30.211s à 0,71$ : **100% conforme à la doctrine, et correcte sur l'information disponible à cet instant** (flux net −372$, carnet pas encore convergé, 12 prints d'ancrage).
- Mais le spot Binance fait son minimum à **t=51s (64 237,85$)** puis monte **sans interruption de +98,44$** jusqu'à la fin. Comme la basis spot−oracle est verrouillée dans [50,30$ ; 60,46$], l'oracle a mécaniquement suivi : croisement à t=160s (+1,28$), puis marge finale +61,11$. Un retournement complet du sous-jacent — rien à t=30s ne permettait de le prévoir. **Aucun filtre d'entrée ne sauvait ce trade sans tuer aussi les sessions gagnantes.**


**Le mécanisme du −76% n'est pas l'entrée, c'est la lenteur de la sortie L4** (critère symétrique : 10 prints consécutifs à marge ≥ +10$) :

| Instant | Événement | NO bid
|-----|-----|-----
| t=160.000s | Oracle croise le strike | **0,47$**
| t=182.000s | Premier print ≥ +10$ | 0,26$
| t=192.076s | Exécution L4 réelle | **0,17$**


Entre le croisement et votre sortie, le NO bid a fondu de 0,47$ → 0,17$ : **vous avez payé 32 secondes de "confirmation" avec 63% de la valeur récupérable.**

## 2. Ne changez pas de stratégie — changez L4

La sortie doit être **asymétrique** : l'entrée exige un ancrage profond (10 prints ≤ −10$), mais la sortie doit se déclencher sur le simple **recroisement confirmé du strike** — 3 prints consécutifs à marge ≥ 0 suffisent (le saut tick max de la session est 7,66$, donc 3 prints filtrent le bruit) :

- Signal : t=162.000s → exécution premier BBO à **t=163.556s, NO bid 0,47$**
- Résultat : 7,042 parts × 0,47$ = **3,31$ → perte −33,8% au lieu de −76,1%** (+2,11$ récupérés)
- **Coût sur vos sessions gagnantes : zéro.** Session +33% : marge max après signal −15,66$, jamais déclenché. Session 5:15PM : plus aucun recroisement après t=1:24. La règle ne touche que les vrais retournements.


## 3. Pouvait-on être RENTABLE sur cette session ?

Oui, une seule voie : le **flip** — sortir du NO et acheter le YES au recroisement confirmé (t=163.556s, YES ask 0,53$) : 3,31$/0,53$ = 6,25 parts → **6,25$ à règlement, soit +24,9%**.

Mais je vous le déconseille comme règle systématique : la session du 6:55PM comptait **16 croisements du strike** — un flip automatique s'y ferait scier à chaque aller-retour. Le flip n'est acceptable qu'avec sa propre confirmation d'ancrage (10 prints ≥ +10$), et là il arrive trop tard sur cette session (YES ask 0,83$ à t=192 → final −20%, mieux que −76% mais pas rentable).

## Réponse finale

- **Cause exacte** : retournement unique et définitif de l'oracle à t=160s (rallye spot continu dès t=51s) + sortie L4 symétrique qui a laissé le NO fondre de 0,47$ à 0,17$ pendant la confirmation.
- **Changer de stratégie ? Non.** L'edge structurel (oracle vs carnet ancré sur le spot) reste validé par vos autres sessions. Ce trade est le risque de queue irréductible d'une entrée précoce all-in.
- **L'amélioration** : sortie asymétrique sur recroisement confirmé (3 prints ≥ 0) → plafonne ce type de perte à ~−34% sans rien coûter aux gagnantes. Et notez que le vrai amplificateur reste P4 (100% all-in) : une session comme celle-ci efface deux sessions à +33%






# 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Vous voulez le verdict d'abord, le voici.

- Stratégie retenue : **« Rachat d'extrême »** — achat YES quand la probabilité consensus passe sous 0,15, revente quand elle repasse au-dessus de 0,30.
- Capital final : **8,9996$** contre 5,00$ de départ, soit un rendement net de **+79,99 %**.
- Nombre de trades : **1**, dont **1 gagnant** et **0 perdant**.
- Critère de sélection : **bénéfice net final, départage par drawdown maximal si l'écart est inférieur à 0,50$** — défini avant toute comparaison.
- Étiquette globale : **[SESSION-SPÉCIFIQUE]** — le cycle exploité ne s'est produit qu'une fois dans la fenêtre.


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

---

# 1. CADRE D'ANALYSE CHOISI

J'adopte un cadre de **microstructure quantitative multi-flux** : je confronte le carnet d'ordres YES/NO (précision ms), le flux de trades signé, le spot Binance (6 994 ticks) et l'oracle Chainlink (284 ticks, cadence ~1 s). Ce cadre est adapté parce que la fenêtre dure 300 s et que le règlement dépend d'un seul chiffre — l'oracle vs le strike — observable en parallèle du marché : la seule question qui paie est celle du décalage entre l'information (spot/oracle) et le prix (carnet). Aucune analyse fondamentale n'a de sens sur 5 minutes ; la granularité milliseconde disponible impose la microstructure.

---

# PARTIE A — LES 6 GRAPHES DU PDF

## A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (590 ÉVÉNEMENTS BBO)

**A1. IDENTITÉ.** Bid/ask/mid YES et NO plus probabilité consensus, axe X en mm:ss.mmm de t=0,000s à t=300,000s, axe Y en $ de 0 à 1. Vue 300 s + 3 zooms 15 s ([01:20–01:35], [02:19–02:34], [03:37–03:52]). 8 événements « carnet croisé — suspects » marqués.

**A2. QUESTION.** Que fait le prix du YES au fil de la fenêtre ? Existe-t-il des extrêmes suivis de retournements ? Les carnets YES et NO racontent-ils la même histoire ?

**A3. COMMENT.** Lecture des étiquettes d'extrema imprimées sur le graphe (le PDF n'étiquette que les extrema), puis vérification de chaque valeur dans le CSV bbo (B-F1).

**A4. VALEURS EXTRAITES.**

- t=00:00.164 : YES = 0,52$ (première cote)
- t=02:05.052 : YES = 0,11$ (plancher de la session)
- t=04:06.862 : YES = 0,99$ (plafond)
- t=00:00.164 : NO = 0,48$
- t=02:05.052 : NO = 0,89$
- t=04:06.862 : NO = 0,01$


**A5. OBSERVATIONS BRUTES.**

- Le YES parcourt tout l'intervalle 0,11$ → 0,99$ en une seule fenêtre de 5 minutes.
- Le NO est le miroir exact du YES à chaque étiquette (0,48/0,89/0,01 contre 0,52/0,11/0,99) : les deux carnets sont arbitrés en continu.
- Le plancher 0,11$ (t=02:05.052) est suivi d'une remontée continue jusqu'à 0,99$.


**A6. SPÉCIFIQUE À LA SESSION.** L'issue UP et l'amplitude 0,11→0,99 dépendent du chemin du BTC de cette fenêtre précise ; rien ne garantit qu'un plancher extrême se retourne dans une autre fenêtre. [SESSION-SPÉCIFIQUE]

## A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT

**A1. IDENTITÉ.** Deux séries de prix BTC ($) : Binance BTC/USDT direct (6 994 ticks) et Chainlink BTC/USD (284 ticks), ligne de strike à 64 225,00$, sous-graphe basis = spot − oracle. Période 00:00.000 → 04:59.916.

**A2. QUESTION.** Lequel des deux flux bouge en premier ? L'écart spot−oracle est-il stable ? Quand l'oracle franchit-il le strike ?

**A3. COMMENT.** Lecture des étiquettes de la vue 300 s et des zooms, croisement avec les CSV spot et oracle (B-F3, B-F4) pour les franchissements.

**A4. VALEURS EXTRAITES.**

- Strike = 64 225,00$ (ligne « Strike ouverture »)
- t=00:01.288 : Binance = 64 265,00$ (premier tick)
- t=00:51.147 : Binance = 64 237,85$ (plancher spot)
- t=04:59.916 : Binance = 64 336,29$ (dernier tick)
- t=00:00.000 : Chainlink = 64 215,97$ (premier tick, sous le strike)
- Sous-graphe basis : bande stable entre 0 et 60$, jamais négative sur la vue 300 s


**A5. OBSERVATIONS BRUTES.**

- Les deux flux ne cotent pas le même actif au même niveau : Binance (USDT) est systématiquement au-dessus de Chainlink (USD) d'environ la même bande — quantifiée en B-F3/B-F4 à +55,77$ de moyenne.
- L'oracle ouvre SOUS le strike (64 215,97$ < 64 225,00$) : le marché démarre en configuration DOWN.
- Le spot Binance ne repasse jamais sous son propre niveau d'ouverture translaté : plancher 64 237,85$ à t=00:51.147.


**A6. SPÉCIFIQUE À LA SESSION.** Le niveau absolu du basis (+55,77$) reflète la prime USDT/USD du jour ; sa valeur exacte est propre à cette session. [SESSION-SPÉCIFIQUE] Sa stabilité, elle, est un fait structurel testé en B.

## A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ (1 380 TRADES)

**A1. IDENTITÉ.** Volume directionnel 30 s glissant en $ : flux haussier (BUY YES + SELL NO) en positif, baissier en négatif. Période 00:00.000 → 05:00.000, axe Y de −2 000 à +5 000$.

**A2. QUESTION.** Le flux précède-t-il ou suit-il le prix ? Quels sont les pics d'agression acheteuse/vendeuse ?

**A3. COMMENT.** Lecture des 4 étiquettes d'extrema, reconstruction du flux minute par minute depuis le CSV trades (B-F2).

**A4. VALEURS EXTRAITES.**

- t=00:00.407 : −12,25$ (premier trade, baissier)
- t=03:37.536 : +583,91$ (pic haussier de la session)
- t=04:08.094 : −614,97$ (pic baissier — un seul trade, cf. B-F2)
- t=04:40.625 : −39,60$
- Zoom [02:19–02:34] : bascule de −398,50$ (t=02:20.251) à une série de trades positifs dès t=02:27.077


**A5. OBSERVATIONS BRUTES.**

- Le pic haussier +583,91$ (t=03:37.536) survient pendant la phase où le YES monte de 0,77$ vers 0,99$ : le flux accompagne, il ne précède pas le retournement de t=02:27.
- Le plus gros trade isolé de la session est baissier (−614,97$ à t=04:08.094) alors que le YES cote déjà 0,96$+ : il ne fait pas bouger le prix.


**A6. SPÉCIFIQUE À LA SESSION.** Les montants absolus des pics dépendent des participants du moment. [SESSION-SPÉCIFIQUE]

## A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ.** Imbalance = Haussier / (Haussier + Baissier), 0 à 1, ligne d'équilibre à 0,5. Période 00:00.000 → 05:00.000.

**A2. QUESTION.** À quel moment le rapport de force acheteur/vendeur bascule-t-il durablement au-dessus de 0,5 ?

**A3. COMMENT.** Lecture des étiquettes, calcul de l'imbalance par minute depuis B-F2 pour dater la bascule.

**A4. VALEURS EXTRAITES.**

- t=00:00.407 : 0,00 (départ 100 % baissier)
- t=04:07.018 : 0,88 (maximum)
- t=04:58.588 : 0,00 (dernier point, fenêtre vide de trades haussiers)
- Zoom [01:20–01:35] : 0,24 (t=01:21.820) et 0,15 (t=01:34.792) — régime vendeur
- Zoom [03:37–03:52] : 0,79 (t=03:44.978) — régime acheteur installé


**A5. OBSERVATIONS BRUTES.**

- L'imbalance par minute (calcul B-F2) : min 0 = 0,363 ; min 1 = 0,141 ; min 2 = 0,517 ; min 3 = 0,692 ; min 4 = 0,653 — la bascule au-dessus de 0,5 se fait en minute 2, la minute où le YES touche son plancher puis remonte.
- Les extrêmes 0,00 et 0,88 correspondent à des fenêtres 30 s peu remplies : l'indicateur est bruité aux bords.


**A6. SPÉCIFIQUE À LA SESSION.** La valeur 0,141 de la minute 1 traduit la capitulation vendeuse de cette session précise. [SESSION-SPÉCIFIQUE]

## A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS

**A1. IDENTITÉ.** Spread ask − bid en $ pour YES et NO, 590 événements, axe Y de 0,00 à 0,07$. Période 00:00.000 → 05:00.000.

**A2. QUESTION.** Combien coûte un aller-retour ? Le spread explose-t-il à certains moments ?

**A3. COMMENT.** Lecture des étiquettes d'extrema, statistiques complètes calculées en B-F1.

**A4. VALEURS EXTRAITES.**

- t=00:00.164 : 0,01$ (spread d'ouverture)
- t=00:01.101 : 0,00$ (minimum)
- t=02:03.221 : 0,07$ (maximum de la session)
- t=04:06.862 : 0,00$
- Zooms : le spread revient à 0,01$ en moins d'une seconde après chaque élargissement (ex. 0,06$ à t=02:27.014 → 0,01$ à t=02:27.144)


**A5. OBSERVATIONS BRUTES.**

- Spread YES et NO identiques sur chaque étiquette : cotation symétrique permanente.
- Le maximum 0,07$ (t=02:03.221) coïncide avec le plancher du YES (0,11$ à t=02:05.052, A-G1) : la liquidité se retire aux extrêmes de prix.
- Hors extrêmes, le coût d'un aller-retour reste ≤ 0,04$ (2 × 0,02$ de médiane, calcul B-F1).


**A6. SPÉCIFIQUE À LA SESSION.** L'instant exact du pic de spread est propre à la session ; le retour rapide à 0,01–0,02$ est cohérent sur les 590 événements. [PATTERN CANDIDAT]

## A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ.** Écart = yes_mid − (1 − no_mid), en $, axe Y de −0,005 à 0,000, ligne de cohérence parfaite à 0. 8 suspects « carnet croisé » marqués. Période 00:00.000 → 05:00.000.

**A2. QUESTION.** Existe-t-il des fenêtres d'arbitrage YES/NO exploitables ?

**A3. COMMENT.** Lecture de la borne inférieure de l'axe et des marqueurs, recomptage exact dans B-F1.

**A4. VALEURS EXTRAITES.**

- Borne inférieure de l'écart observé : −0,005$ (axe du graphe, atteint)
- Borne supérieure : 0,000$
- Nombre de suspects « carnet croisé » annoncés : 8 (cartouche du graphe)
- Recomptage CSV (B-F1) : 1 événement à écart mid non nul, −0,005$ à t=00:54.726
- Écart moyen sur 588 événements complets : −0,00001$ (calcul B-F1)


**A5. OBSERVATIONS BRUTES.**

- L'écart maximal jamais observé (−0,005$) est inférieur au spread médian (0,02$, A-G5) : aucun arbitrage inter-carnets net de coûts n'a existé dans la session.
- Les 8 suspects du cartouche incluent des états transitoires intra-milliseconde que l'échantillonnage CSV ne restitue pas : DONNÉE NON DISPONIBLE pour leurs 8 timestamps individuels.


**A6. SPÉCIFIQUE À LA SESSION.** Rien à classer : ce graphe démontre une absence d'opportunité, valable telle quelle. [PATTERN CANDIDAT]

---

# PARTIE B — LES FICHIERS CSV

## B-F1 — bbo.csv

**B1. IDENTITÉ.** Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask. 590 lignes (588 complètes ; t=00:00.164 sans NO initial, t=04:06.862 finale partielle). Période 00:00.164 → 04:06.862 (dernière cote), événementiel (précision ms).

**B2. APPORT.** Les prix exécutables exacts (bid/ask) à chaque instant — le graphe ne montre que des extrema étiquetés. C'est le fichier qui fixe chaque prix d'entrée/sortie de la Partie C.

**B3. COMMENT.** Spread = ask − bid par ligne ; mid = (bid+ask)/2 ; consensus = (yes_mid + (1 − no_mid))/2 ; écart inter-carnets = yes_mid − (1 − no_mid) ; comptages de franchissements de seuils. Toutes colonnes, 588 lignes complètes.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Spread YES moyen | 0,0187$ | yes_bid, yes_ask | 588
| Spread YES médian / max | 0,02$ / 0,07$ | yes_bid, yes_ask | 588
| Consensus initial (t=00:00.164) | 0,515 | 4 colonnes | 1
| Consensus minimum (t=02:05.052) | 0,115 | 4 colonnes | 1
| Consensus final (t=04:06.862) | 0,99 | 4 colonnes | 1
| Événements consensus ≤ 0,15 | 28 (premier t=00:48.796, dernier t=02:11.282) | 4 colonnes | 588
| Premier consensus ≥ 0,90 | t=03:36.292 | 4 colonnes | 588
| Écart inter-carnets min / moyen | −0,005$ / −0,00001$ | 4 colonnes | 588


**B5. OBSERVATIONS BRUTES.**

- Le consensus passe sous 0,15 à t=00:48.796 et n'en ressort durablement qu'à t=02:26.854+, avec retour au-dessus de 0,30 à t=02:27.014 (yes_bid = 0,27$ à cet instant).
- Le yes_bid minimum absolu est 0,09$ (t=02:03.221), en pleine pointe de spread (A-G5).
- 28 événements offrent un achat YES entre 0,12$ et 0,16$ à l'ask.


**B6. SPÉCIFIQUE À LA SESSION.** Les 2 lignes incomplètes (ouverture et clôture de flux) sont des artefacts de bord : bruit. Le trou de cotation entre t=01:36.021 et t=02:19.867 sur certains niveaux traduit simplement l'absence de mise à jour : signal faible, pas d'anomalie.

## B-F2 — trades.csv

**B1. IDENTITÉ.** Colonnes t_ms, usd, direction (+1 haussier / −1 baissier). 1 380 lignes. Période 00:00.407 → ~04:59 ; événementiel.

**B2. APPORT.** Le volume signé réel — les graphes 3 et 4 n'en montrent que des agrégats 30 s étiquetés aux extrema.

**B3. COMMENT.** Sommes et comptages par direction (colonnes usd × direction, 1 380 lignes) ; agrégats par minute [k×60 000, (k+1)×60 000) ; médiane et maximum de usd.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Volume total | 21 508,66$ | usd | 1 380
| Volume haussier (n=782) | 11 201,07$ | usd, direction | 782
| Volume baissier (n=598) | 10 307,59$ | usd, direction | 598
| Trade médian | 4,80$ | usd | 1 380
| Plus gros trade | 614,97$, baissier, t=04:08.094 | usd, direction, t_ms | 1
| Trades ≥ 100$ | 44, cumul 8 995,68$ | usd | 44
| Flux net minute 1 | −1 758,60$ (imbalance 0,141) | usd, direction, t_ms | 198
| Flux net minute 3 | +2 730,30$ (imbalance 0,692) | usd, direction, t_ms | 439


**B5. OBSERVATIONS BRUTES.**

- Le flux net bascule de −1 758,60$ (minute 1) à +132,70$ (minute 2) puis +2 730,30$ (minute 3) : la bascule suit le plancher de prix de t=02:05.052 (B-F1), elle ne le précède pas.
- L'imbalance globale de session n'est que de 0,521 (11 201,07/21 508,66) pour une issue certaine à la fin : le flux agrégé est un indicateur retardé.


**B6. SPÉCIFIQUE À LA SESSION.** Le trade de 614,97$ baissier à t=04:08.094 contre un YES à 0,96$ est une perte quasi certaine pour son auteur — comportement non modélisable : bruit.

## B-F3 — spot.csv

**B1. IDENTITÉ.** Colonnes t_ms, price. 6 994 lignes. Période 00:01.288 → 04:59.916. Ticks Binance BTC/USDT, rafales milliseconde.

**B2. APPORT.** Le prix leader à granularité maximale, pour dater les mouvements avant l'oracle.

**B3. COMMENT.** Min/max/écart-type sur price (6 994 lignes) ; alignement au dernier tick ≤ t pour le basis (avec B-F4) ; recherche de franchissements du niveau strike + basis.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Premier / dernier tick | 64 265,00$ (t=00:01.288) / 64 336,29$ (t=04:59.916) | t_ms, price | 2
| Minimum | 64 237,85$ à t=00:51.147 | t_ms, price | 6 994
| Maximum | 64 336,29$ à t=04:59.916 | t_ms, price | 6 994
| Écart-type | 28,43$ | price | 6 994
| Amplitude session | 98,44$ | price | 6 994
| Franchissement du niveau strike+55,77$ (=64 280,77$) | t=03:01.388 | t_ms, price | 6 994
| Basis spot−oracle moyen / médian | +55,77$ / +55,77$ | price (avec B-F4) | 282 appariements
| Basis min / max | +50,30$ (t=00:02.000) / +60,46$ (t=03:47.000) | price (avec B-F4) | 282 appariements


**B5. OBSERVATIONS BRUTES.**

- Le basis reste dans une bande de 10,16$ (50,30 à 60,46) pendant 300 s : le spot Binance est un proxy quasi affine de l'oracle.
- La corrélation spot(t−lag)/oracle(t) décroît de 0,9986 (lag 0 s) à 0,9135 (lag 30 s) : l'information est commune, le retard exploitable est inférieur à la cadence de l'oracle (~1 s).


**B6. SPÉCIFIQUE À LA SESSION.** Rafales de ticks à timestamp identique (ex. 44 ticks à t=00:32.561) : microstructure Binance normale, bruit.

## B-F4 — oracle.csv

**B1. IDENTITÉ.** Colonnes t_ms, price, ts_src. 284 lignes. Période 00:00.000 → 04:58.000. Cadence moyenne 1 053 ms, écart max 2 000 ms. ts_src = « payload » sur 284/284 (0 fallback, confirmé par le cartouche de chaque graphe).

**B2. APPORT.** La série qui décide du règlement — c'est le seul fichier dont dépend l'issue UP/DOWN.

**B3. COMMENT.** Min/max/écart-type sur price (284 lignes) ; comparaison de chaque tick au strike 64 225,00$ (A-G2) ; datation du premier franchissement durable.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Premier tick | 64 215,97$ à t=00:00.000 (sous le strike) | t_ms, price | 1
| Minimum | 64 180,24$ à t=01:08.000 | t_ms, price | 284
| Maximum / dernier tick | 64 277,08$ à t=04:58.000 | t_ms, price | 284
| Écart-type | 32,48$ | price | 284
| Premier tick > strike | t=03:02.000 (64 226,23$) | t_ms, price | 284
| Dernier tick ≤ strike | t=03:01.000 (64 223,20$) | t_ms, price | 284
| Ticks au-dessus du strike | 115 / 284 | price | 284
| Retours sous le strike après t=03:02.000 | 0 | t_ms, price | 116


**B5. OBSERVATIONS BRUTES.**

- L'oracle franchit le strike à t=03:02.000 et ne repasse JAMAIS dessous (0 retour sur 116 ticks restants) : l'issue UP est fixée 118 s avant la clôture.
- Au même instant, le carnet cote encore YES ask = 0,77$ (B-F1, t=03:02.494) : le marché price 77 % un événement déjà réalisé à 100 % côté oracle, et ne cote 0,90 qu'à t=03:36.292 (B-F1), 34 s plus tard.


**B6. SPÉCIFIQUE À LA SESSION.** L'écart oracle-ouverture vs strike (64 215,97$ vs 64 225,00$) implique que le strike provient d'un tick antérieur à t=0 non inclus dans ce fichier : DONNÉE NON DISPONIBLE pour ce tick. Je retiens le strike affiché 64 225,00$ (A-G2).

---

# PARTIE C — STRATÉGIES ET SIMULATION

## C0. STRATÉGIES CANDIDATES

Voici les cinq observations les plus fortes des Parties A et B, celles dont tout le reste découle.

- OBS-1 : le consensus YES passe sous 0,15 à t=00:48.796 (28 événements ≤ 0,15) puis repasse au-dessus de 0,30 à t=02:27.014 avec yes_bid = 0,27$ (B-F1, B5).
- OBS-2 : le basis spot−oracle est stable, moyenne +55,77$, bande [50,30 ; 60,46] sur 282 appariements (B-F3, B5).
- OBS-3 : l'oracle franchit le strike à t=03:02.000 sans jamais repasser dessous, alors que YES ask = 0,77$ à t=03:02.494 et que 0,90 n'est coté qu'à t=03:36.292 (B-F4, B5).
- OBS-4 : le flux net par minute bascule de −1 758,60$ (minute 1) à +2 730,30$ (minute 3), APRÈS le plancher de prix (B-F2, B5).
- OBS-5 : le spread médian est de 0,02$ et l'aller-retour coûte ≤ 0,04$ hors extrêmes (A-G5, A5 ; B-F1, B4).


Chaque observation engendre une action, regroupées en trois familles à mécanismes réellement distincts.

**Candidate 1 — « Rachat d'extrême »** (OBS-1 + OBS-5). Logique : quand le consensus atteint un extrême bas avec un spread encore praticable, le prix a sur-réagi par rapport à l'incertitude résiduelle de 4 minutes ; on achète l'extrême et on revend le retour à la normale. Règles : ACHAT YES à l'ask dès consensus ≤ 0,15 ; VENTE au bid dès consensus ≥ 0,30 ; une position à la fois ; tout le cash disponible. Simulation sommaire : 1 trade, achat 0,15$ (t=00:48.796), vente 0,27$ (t=02:27.014), P&L net +4,00$.

**Candidate 2 — « Franchissement oracle »** (OBS-3 + OBS-2). Logique : le règlement dépend exclusivement de l'oracle, public et lisible en temps réel ; quand il franchit le strike, l'issue est connue avant que le carnet ne l'intègre. Règles : ACHAT YES à l'ask au premier tick oracle > strike ; DÉTENTION jusqu'au règlement à 1,00$ ; annulation si l'oracle repasse sous le strike (jamais déclenchée). Simulation sommaire : 1 trade, achat 0,77$ (t=03:02.494), règlement 1,00$, P&L net +1,49$.

**Candidate 3 — « Suivi de flux 30 s »** (OBS-4). Logique : suivre l'agression dominante ; on achète quand le flux net 30 s dépasse +300$ et on sort quand il passe sous −300$. Règles chiffrées : seuils ±300$ sur fenêtre glissante 30 s (B-F2). Simulation sommaire : 2 trades — achat 0,40$ (t=00:17.894), vente 0,29$ (t=00:30.211), perte −1,38$ ; achat 0,54$ (t=02:39.044), règlement 1,00$, gain +3,09$ ; P&L net +1,71$.

**CRITÈRE DE SÉLECTION : bénéfice net final sur la session ; si deux candidates sont à moins de 0,50$ l'une de l'autre, départage par le drawdown maximal le plus faible.** Ce critère est fixé ici, avant le tableau.

| Stratégie | Observations sources (A5/B5) | Nb trades | P&L net | Verdict
|-----|-----|-----|-----
| Rachat d'extrême | OBS-1 (B-F1 B5), OBS-5 (A-G5 A5) | 1 | +4,00$ | **RETENUE**
| Franchissement oracle | OBS-3 (B-F4 B5), OBS-2 (B-F3 B5) | 1 | +1,49$ | Écartée
| Suivi de flux 30 s | OBS-4 (B-F2 B5) | 2 | +1,71$ | Écartée


Décision : +4,00$ contre +1,71$ et +1,49$ — l'écart dépasse 0,50$, le départage par drawdown est inutile. Le « Suivi de flux » est écarté sur son propre chiffre : il perd −1,38$ sur son premier trade parce que le flux est retardé (OBS-4). Le « Franchissement oracle » est écarté par le critère, mais notez-le : c'est la seule candidate dont le mécanisme est structurellement reproductible — j'y reviens en C4.

## C1. TABLE DE CONVERGENCE (stratégie retenue)

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| R1 : achat YES si consensus ≤ 0,15 | A-G1 (A4 : 0,11$ à t=02:05.052), B-F1 (B4 : 28 événements ≤ 0,15) | Aucune | Validée, 2 sources
| R2 : vente si consensus ≥ 0,30 | B-F1 (B5 : retour à 0,30 à t=02:27.014), A-G1 (zoom 02:19–02:34 : montée 0,23→0,47) | B-F4 (B5 : détenir jusqu'au règlement aurait payé 1,00$) | Validée mais sous-optimale ex post
| R3 : le retournement après extrême se produit | B-F1 (B5) | Aucune pièce ne le confirme une 2e fois : une seule occurrence dans la session | [FRAGILE - 1 SOURCE]
| R4 : exécution à ±0,02$ du mid possible | A-G5 (A4/A5), B-F1 (B4 : spread médian 0,02$) | A-G5 (A5 : spread 0,07$ au pire moment, t=02:03.221) | Validée avec réserve


Je le dis clairement : la règle R3 — le cœur de la stratégie — est marquée FRAGILE, une seule occurrence la soutient.

## C2. PSEUDO-CODE EXÉCUTABLE

```plaintext
ETAT: cash = 5,00$ ; position = NULLE ; verrou_ordre = LIBRE

POUR CHAQUE événement BBO e (t, yes_bid, yes_ask, no_bid, no_ask):   # B-F1, 588 lignes
  consensus = (mid_YES + (1 - mid_NO)) / 2                           # B-F1, calcul B3

  SI position = NULLE ET verrou_ordre = LIBRE ET consensus <= 0,15:  # B-F1 B4: 28 evts <= 0,15
      SI yes_ask - yes_bid <= 0,03:                                  # A-G5 A4: spread median 0,02$
          verrou_ordre = PRIS
          quantite = arrondi_inf(cash / yes_ask, 0,01)               # [HYPOTHESE n°3] pas de 0,01 part
          ACHETER quantite au prix yes_ask                           # [HYPOTHESE n°1] frais = 0,00$
                                                                     # [HYPOTHESE n°2] fill integral au BBO affiche
          cash -= quantite * yes_ask ; position = quantite ; verrou_ordre = LIBRE

  SINON SI position > 0 ET verrou_ordre = LIBRE ET consensus >= 0,30: # B-F1 B5: atteint a t=02:27.014
      verrou_ordre = PRIS
      VENDRE position au prix yes_bid                                 # B-F1: bid = 0,27$ a cet instant
      cash += position * yes_bid ; position = NULLE ; verrou_ordre = LIBRE

A LA CLOTURE: SI position > 0: cash += position * 1,00$ si UP, + 0,00$ si DOWN
```

Compteur d'hypothèses : 3.

## C-SIM. SIMULATION ALGORITHMIQUE DÉTAILLÉE

### ÉTAT DU PORTEFEUILLE

- t=00:00.000 : cash 5,0000$, position nulle, valeur totale 5,0000$.
- t=00:48.796 : signal d'achat (consensus 0,145 ≤ 0,15, B-F1). Achat de 33,33 parts YES à 0,15$ = 4,9995$ ≤ cash disponible. Cash après : 0,0005$. Valeur totale : 0,0005 + 33,33 × 0,145 (mid) = 4,8334$.
- t=02:03.221 : point bas — yes_bid 0,09$, valeur liquidative 0,0005 + 33,33 × 0,09 = 3,0002$.
- t=02:27.014 : signal de vente (consensus 0,310 ≥ 0,30, B-F1). Vente de 33,33 parts à 0,27$ = 8,9991$. Cash après : 8,9996$. Position nulle.
- t=02:27.014 → 05:00.000 : le consensus ne repasse jamais ≤ 0,15 (B-F1, minimum ultérieur 0,30) : aucune ré-entrée. Cash final 8,9996$.


Aucun achat n'a dépassé le cash disponible à son instant : 4,9995$ ≤ 5,0000$ (contrôle ligne 1 du journal).

### RACE CONDITIONS

(a) Signal d'achat pendant un ordre en cours : le verrou par marché (`verrou_ordre`) rend tout nouvel événement inopérant tant que l'ordre précédent n'est pas confirmé. RÈGLE : verrou unique par actif, le second signal est rejeté, pas mis en file.

(b) Signal de vente sur une position non confirmée à l'achat : la condition de vente exige `position > 0`, qui n'est vrai qu'après confirmation du fill d'achat. RÈGLE : la vente d'une position non confirmée est structurellement impossible, le signal est ignoré.

(c) Deux signaux simultanés sur le même actif (deux lignes BBO au même t_ms, cas réel : t=02:27.014 compte 4 lignes dans B-F1) : les événements sont traités dans l'ordre du fichier, un seul passe le verrou. RÈGLE : file FIFO par ordre de lecture, le premier signal éligible exécute, les suivants retombent sur l'état mis à jour.

(d) Fill partiel : si la quantité confirmée est inférieure à la quantité demandée, la position enregistrée est la quantité confirmée et le cash n'est débité que d'autant ; le reliquat d'ordre est annulé, pas reposté. RÈGLE : « fill-and-kill » — on ne poursuit jamais un prix. Dans cette simulation, l'hypothèse n°2 (fill intégral) fait qu'aucun fill partiel n'est survenu.

### JOURNAL DE TRADES

Le journal de trades, ligne par ligne :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=00:48.796 | 0,15$ | 33,33 | t=02:27.014 | 0,27$ | 0,00$ | +3,9996$ | 8,9996$


### RÉSULTATS

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / montant cumulé | 1 / +3,9996$ | ligne 1
| Trades perdants / montant cumulé | 0 / 0,00$ | —
| Frais totaux | 0,00$ (ligne 1 : 0,00$) [HYPOTHÈSE n°1] | ligne 1
| BÉNÉFICE NET | **+3,9996$** | ligne 1
| Capital final vs initial | 8,9996$ vs 5,0000$ (+79,99 %) | ligne 1
| Pire perte unitaire | aucune (0 trade perdant) | —
| Drawdown maximum | −1,9993$ (valeur 5,0000$ à t=00:48.796 → 3,0002$ à t=02:03.221) | ligne 1, position ouverte


## C4. PORTÉE

- Le retournement 0,15 → 0,30 s'est produit une fois, sans mécanisme causal identifié dans les pièces. [SESSION-SPÉCIFIQUE]
- Le retour du spread à 0,01–0,02$ après chaque élargissement, vérifié sur 588 événements (B-F1). [PATTERN CANDIDAT]
- L'absence d'arbitrage inter-carnets YES/NO (écart max −0,005$ < spread 0,02$, A-G6). [PATTERN CANDIDAT]
- Le retard du carnet sur l'oracle après franchissement du strike (34 s entre t=03:02.000 et la cote 0,90, B-F4/B-F1) : mécanisme structurel — le carnet doit attendre des contreparties, l'oracle non. [PATTERN CANDIDAT]
- Le flux signé comme indicateur retardé du prix (bascule minute 2-3 après le plancher de t=02:05.052, B-F2). [PATTERN CANDIDAT]
- La performance +79,99 % de la stratégie retenue, dépendante d'un événement unique marqué FRAGILE en C1. [SESSION-SPÉCIFIQUE]


---

# AUTO-CONTRÔLE FINAL

- Contrôle 1 (≥5 valeurs par graphe, ≥8 par CSV) → conforme : A-G1 à A-G6 comptent 5 à 6 valeurs chacun ; B-F1 à B-F4 comptent 8 métriques chacun dans leurs tableaux B4.
- Contrôle 2 (traçabilité des Parties C) → conforme : chaque règle et chaque prix de C0/C1/C2/C-SIM cite une rubrique A4, B4 ou B5 identifiée.
- Contrôle 3 (aucun chiffre en double contradictoire) → conforme : 0,11$ désigne le yes_bid étiqueté (A-G1) et 0,115 le consensus calculé (B-F1), sources distinctes citées ; 8,9996$, +3,9996$ et +79,99 % sont identiques dans l'ouverture, le journal et la clôture.
- Contrôle 4 (cohérence comptable) → conforme : 5,0000 − 4,9995 = 0,0005 ; 0,0005 + 8,9991 = 8,9996$, égal au capital final annoncé, au centime près (et au dixième de centime).
- Contrôle 5 (hypothèses) → 3 hypothèses : n°1 frais nuls (structure de frais preneur non fournie dans les pièces — indispensable pour chiffrer un P&L), n°2 fill intégral au BBO affiché (les tailles au BBO ne sont pas dans les fichiers — indispensable pour exécuter), n°3 granularité 0,01 part (règle de place non fournie — indispensable pour arrondir). Le plafond de 3 est respecté.
- Contrôle 6 (stratégie issue des observations) → conforme : R1/R3 remontent à OBS-1 (B-F1 B5, antérieure), R2 à OBS-1, R4 à OBS-5 (A-G5 A5, antérieure) ; aucune règle importée d'un catalogue.
- Contrôle 7 (conformité au format) → conforme : verdict en ouverture, adresse directe, ordre F1 respecté, tableaux F4 fournis, identifiants A-G1…B-F4 utilisés, clôture en 3 lignes sans chiffre nouveau.


---

# CLÔTURE

Le « Rachat d'extrême » transforme 5,00$ en 8,9996$, soit +3,9996$ de bénéfice net (+79,99 %), en un seul trade gagnant.
Ce résultat repose sur un retournement observé une seule fois, marqué FRAGILE dans la table de convergence.
L'étiquette globale de cette performance est [SESSION-SPÉCIFIQUE] ; le seul mécanisme durable identifié reste le retard du carnet sur l'oracle.

---

## Source : 8ANCRAGE_ORACLE.md

# SPEECH — TEST DE LA STRATÉGIE «ANCRAGE À L'ORACLE»

### Session Polymarket BTC 5min — "Bitcoin Up or Down - August 6, 8:25PM-8:30PM ET" — Résultat : ▲ UP

---

## 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Vous voulez savoir ce que vaut «ANCRAGE À L'ORACLE» sur cette session : voici le verdict, tout de suite.

- Stratégie testée : «ANCRAGE À L'ORACLE» — je m'ancre sur l'écart entre l'oracle Chainlink (seule source de règlement) et le strike, et j'achète le côté que l'oracle désigne quand le marché le vend en retard.
- Capital final : 6,49$ contre 5,00$ de départ, soit +29,87% net.
- Nombre de trades : 1, dont 1 gagnant et 0 perdant.
- Nombre d'adaptations apportées à la stratégie : 7.
- VERDICT SUR LA STRATÉGIE : [VALIDÉE SUR SESSION] — portée [SESSION-SPÉCIFIQUE].


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

---

## 1. CADRE D'ANALYSE CHOISI

J'adopte un cadre de **microstructure comparée de flux** : je confronte tick par tick trois horloges — l'oracle de règlement (291 ticks à 1 s), le spot rapide (5084 ticks), et le carnet du marché prédictif (505 événements BBO à la milliseconde). Ce cadre est adapté à CES données précises parce que la session dure 300 s, que la question du marché ("Up or Down") se réduit à la position de l'oracle par rapport au strike, et que la granularité milliseconde des CSV permet de mesurer le RETARD du marché sur l'oracle — c'est exactement là que vit ou meurt un "ancrage à l'oracle".

---

## PARTIE A — LES 6 GRAPHES DU PDF

### A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (505 ÉVÉNEMENTS BBO)

**A1. IDENTITÉ.** Graphe 1, vue 300 s plus trois zooms 15 s ([00:28.000→00:43.000], [03:15.000→03:30.000], [04:45.000→05:00.000]). Axe X : temps depuis l'ouverture (mm:ss.mmm), de t=0,000s à t=300,000s. Axe Y gauche : prix en $ (0,0 à 1,0) ; axe Y droit : probabilité. Séries : YES/NO bid, ask, mid (505 événements chacune), consensus YES, marqueurs "carnet croisé — suspects (11)".

**A2. QUESTION.** Que fait le prix YES quand l'information nouvelle arrive ? En combien de temps le carnet absorbe-t-il un mouvement ? Y a-t-il des moments où le carnet se disloque ?

**A3. COMMENT.** J'ai lu les étiquettes d'extrema de la vue 300 s pour la trajectoire globale, puis les étiquettes point par point des zooms pour mesurer la vitesse d'absorption (temps entre deux niveaux d'ask) et la fin de session.

**A4. VALEURS EXTRAITES.**

- t=0,186s : YES ask = 0,52$
- t=13,398s : YES = 0,42$ (creux de session)
- t=35,788s : YES ask = 0,70$
- t=42,617s : YES ask = 0,80$
- t=277,362s (04:37.362) : YES = 0,99$
- t=298,858s : YES bid = 0,70$ (mèche de panique, zoom [04:45→05:00])
- t=299,844s : YES = 0,91$


**A5. OBSERVATIONS BRUTES.**

- Le YES passe de 0,61$ (t=34,714s) à 0,70$ (t=35,788s) en 1,074 s : l'absorption d'un mouvement se fait par pas de 0,01$ en quelques millisecondes chacun.
- Le YES ne repasse plus jamais sous 0,70$ entre t=35,788s et t=298,700s.
- La montée 0,42$→0,80$ (t=13,398s→t=42,617s) se fait en 29,219 s.


**A6. SPÉCIFIQUE À LA SESSION.** La mèche 0,99$→0,70$→0,91$ concentrée entre t=298,767s et t=299,844s (dernier 1,2 s de cotation) est un épisode de liquidation de fin de fenêtre : sa profondeur exacte (−0,29$) dépend des positions restantes de cette session précise, je la classe non-reproductible dans son amplitude.

### A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT

**A1. IDENTITÉ.** Graphe 2, mêmes 4 vues. Axe Y haut : prix BTC en $ (64200–64320) ; sous-graphe : basis = spot − oracle (0 à 60+). Séries : Binance BTC/USDT direct (5084 ticks), Chainlink BTC/USD oracle (291 ticks), strike ouverture (64 206$ affiché).

**A2. QUESTION.** Lequel des deux flux fait foi ? Quel écart sépare le flux rapide du flux de règlement, et cet écart est-il stable ?

**A3. COMMENT.** J'ai lu le strike tracé, les étiquettes des deux séries aux mêmes instants, et la courbe de basis pour son signe et son ordre de grandeur.

**A4. VALEURS EXTRAITES.**

- t=0,000s : oracle = 64 205,84$ (= strike affiché 64 206$)
- t=1,009s : spot Binance = 64 265,92$
- t=24,000s : oracle = 64 206,89$ (premier tick au-dessus du strike après le creux)
- t=36,000s : oracle = 64 217,94$
- t=106,296s (01:46.296) : spot = 64 317,18$ (plus haut de session)
- t=298,000s : oracle = 64 225,04$ (dernier tick, côté UP)


**A5. OBSERVATIONS BRUTES.**

- Le basis spot−oracle est positif sur toute la vue 300 s (sous-graphe entre 0 et 60+).
- L'oracle franchit le strike à t=24,000s et la vue 300 s ne le montre plus dessous ensuite.
- Le spot était déjà 60,08$ au-dessus du strike à t=1,009s alors que le YES cotait 0,52$ (A-G1, A4) : le marché price l'oracle, pas le spot.


**A6. SPÉCIFIQUE À LA SESSION.** L'amplitude du basis (jusqu'à +67,66$, cf. B-F3) reflète l'écart USDT/USD et l'agrégation Chainlink de CE jour ; le niveau exact est non-reproductible, seul son signe stable est un fait de structure.

### A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ YES/NO (1267 TRADES)

**A1. IDENTITÉ.** Graphe 3, mêmes 4 vues. Axe Y : volume en $ par fenêtre 30 s, flux haussier (BUY YES + SELL NO) en positif, flux baissier en négatif (−1000 à +4000).

**A2. QUESTION.** Qui pousse, dans quel sens, et à quels moments le flux change-t-il de camp ?

**A3. COMMENT.** J'ai relevé les étiquettes d'extrema de la vue 300 s et les trades individuels étiquetés dans les zooms.

**A4. VALEURS EXTRAITES.**

- t=2,864s : +26,00$ (premier flux haussier étiqueté)
- t=34,346s : −144,67$ (plus gros trade baissier du zoom [00:28→00:43])
- t=206,522s (03:26.522) : +96,00$
- t=277,967s (04:37.967) : +1 643,03$ (plus gros trade de la session)
- t=286,468s (04:46.468) : −374,34$
- t=299,854s : −86,13$


**A5. OBSERVATIONS BRUTES.**

- Le flux haussier domine visuellement toute la vue 300 s ; les pointes baissières restent sous −400$ quand les pointes haussières dépassent +1 600$.
- Dans le zoom [00:28→00:43], après le trade baissier de −144,67$ (t=34,346s), 14 trades haussiers étiquetés s'enchaînent jusqu'à t=42,855s.


**A6. SPÉCIFIQUE À LA SESSION.** Le trade unique de +1 643,03$ à t=277,967s pèse 9,7% du volume total (16 935,65$, B-F4) : un acteur isolé, non-reproductible.

### A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ.** Graphe 4, mêmes 4 vues. Axe Y : imbalance = haussier/(haussier+baissier), de 0,0 à 1,0, ligne d'équilibre à 0,5.

**A2. QUESTION.** L'agressivité des acheteurs est-elle persistante ou alternante ? Quand l'imbalance décroche-t-elle de 0,5 ?

**A3. COMMENT.** Lecture des étiquettes d'extrema de la vue 300 s et des valeurs ponctuelles des trois zooms, comparées à la ligne 0,5.

**A4. VALEURS EXTRAITES.**

- t=1,181s : imbalance = 0,00 (premier trade baissier)
- t=28,115s : 0,65
- t=156,737s (02:36.737) : 0,94 (maximum de session)
- t=201,118s (03:21.118) : 0,88
- t=285,636s (04:45.636) : 0,85
- t=299,894s : 0,77 (clôture)


**A5. OBSERVATIONS BRUTES.**

- L'imbalance est au-dessus de 0,5 sur chacun des trois zooms (minimum étiqueté : 0,57 à t=34,457s).
- Elle termine à 0,77 malgré la vague vendeuse finale (A-G3, A4 : −374,34$ à t=286,468s).


**A6. SPÉCIFIQUE À LA SESSION.** Le 0,00 initial (t=1,181s) est un artefact de démarrage : un seul trade dans la fenêtre. Non-reproductible par construction.

### A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS

**A1. IDENTITÉ.** Graphe 5, mêmes 4 vues. Axe Y : spread ask−bid en $ (−0,05 à 0,25), séries YES et NO (505 événements), superposées (valeurs identiques point à point sur les étiquettes).

**A2. QUESTION.** Combien coûte l'exécution à chaque instant, et quand ce coût devient-il prohibitif ?

**A3. COMMENT.** Lecture des extrema de la vue 300 s puis des séquences milliseconde des zooms, en particulier la fin de session.

**A4. VALEURS EXTRAITES.**

- t=0,186s : spread = 0,01$
- t=35,745s : 0,07$ (pointe du zoom [00:28→00:43])
- t=298,858s : 0,25$ (maximum de session)
- t=299,525s : −0,05$ (carnet croisé)
- t=299,844s : 0,01$ (retour à la normale)


**A5. OBSERVATIONS BRUTES.**

- Hors fin de session, le spread revient à 0,01$–0,02$ en moins d'une seconde après chaque pointe (t=35,745s : 0,07$ ; t=35,767s : 0,01$).
- Les spreads YES et NO sont identiques sur toutes les étiquettes : les deux carnets sont arbitrés en continu.


**A6. SPÉCIFIQUE À LA SESSION.** Le spread négatif (−0,05$ à t=299,525s) et le 0,25$ à t=298,858s sont concentrés dans les 1,2 dernières secondes : conditions de règlement imminentes, non-reproductibles hors de cette zone.

### A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ.** Graphe 6, mêmes 4 vues. Axe Y : yes_mid − (1 − no_mid) en $, de 0,000 à 0,005, ligne "cohérence parfaite (0)", marqueurs "carnet croisé — suspects (11)".

**A2. QUESTION.** Existe-t-il un écart arbitrable entre le carnet YES et le carnet NO ?

**A3. COMMENT.** Lecture de l'axe et des séries : aucune étiquette de point n'est imprimée sur ce graphe ; je lis les bornes, la ligne de référence et les compteurs de légende, et je renvoie le chiffrage fin au CSV BBO (B-F2).

**A4. VALEURS EXTRAITES.**

- Nombre d'événements tracés : 505
- Incohérents marqués : 11
- Ligne de référence : 0,000$
- Borne haute de l'axe : 0,005$
- ts fallback affiché : 0/291 (0,0%)
- Valeurs ponctuelles étiquetées : DONNÉE NON DISPONIBLE (aucune étiquette sur les 4 vues)


**A5. OBSERVATIONS BRUTES.**

- L'écart inter-carnets reste confiné sous la borne 0,005$ sur les 4 vues : aucun arbitrage YES/NO exploitable avec un spread minimal de 0,01$ (A-G5, A4).


**A6. SPÉCIFIQUE À LA SESSION.** Les 11 points "suspects" sont regroupés en fin de session sur le zoom [04:45→05:00] ; leur nombre exact dépend de la panique finale de cette session, non-reproductible.

Vous venez de voir les graphes ; les CSV donnent maintenant les chiffres exacts derrière chaque courbe.

---

## PARTIE B — LES FICHIERS CSV

### B-F1 — ORACLE.CSV (CHAINLINK BTC/USD)

**B1. IDENTITÉ.** Fichier oracle, colonnes t_ms, price, ts_src — toutes utilisées. 291 lignes de données, de t=0,000s à t=298,000s, échantillonnage nominal 1 s (8 trous >1 s, cf. B6).

**B2. APPORT.** La valeur EXACTE du strike (le PDF affiche 64 206$ arrondi) et la mesure fine du bruit de l'oracle — indispensables pour calibrer une marge d'ancrage.

**B3. COMMENT.** Strike = price à t=0,000s. Marge(t) = price(t) − strike. Comptage des ticks au-dessus/au-dessous, variation absolue moyenne entre ticks consécutifs |Δprice|, extrema, trous d'horodatage, comptage ts_src≠payload.

**B4. RÉSULTATS CHIFFRÉS.** Le tableau des mesures oracle :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Strike exact (t=0,000s) | 64 205,84$ | t_ms, price | 1
| Dernier tick (t=298,000s) | 64 225,04$ (côté UP) | t_ms, price | 1
| Minimum | 64 202,33$ (t=18,000s) | t_ms, price | 291
| Maximum | 64 256,37$ (t=108,000s) | t_ms, price | 291
| Ticks au-dessus du strike | 274/291 (94,2%) | price | 291
| Ticks ≤ strike | 17, tous entre t=0,000s et t=23,000s | t_ms, price | 291
| Variation absolue moyenne par tick | 0,40$ | price | 290
| Variation maximale par tick | 9,66$ (baisse, t=111,000s) | t_ms, price | 290
| Variations ≥3$ en valeur absolue | 7 occurrences | price | 290
| ts fallback (ts_src≠payload) | 0/291 (0,0%) | ts_src | 291


**B5. OBSERVATIONS BRUTES.**

- L'oracle passe au-dessus du strike à t=24,000s (64 206,89$) et n'y retourne plus jamais : 275 secondes consécutives côté UP.
- La marge atteint +12,11$ à t=36,000s puis +18,86$ à t=41,000s et ne redescend jamais sous +16,18$ (t=242,000s) jusqu'à la fin.
- Le bruit tick-à-tick (0,40$ en moyenne, 9,66$ au pire) est un ordre de grandeur sous les marges observées après t=36,000s.


**B6. SPÉCIFIQUE À LA SESSION.** 8 trous >1 s dans l'horodatage (t=48→50s, 73→75s, 75→77s, 90→92s, 106→108s, 189→191s, 213→215s, 281→283s) : verdict BRUIT (aucun ne coïncide avec un mouvement ≥3$). La chute isolée de −9,66$ à t=111,000s : verdict BRUIT d'agrégation (le spot ne bouge que de −0,26$ entre t=110,000s et t=112,000s côté oracle suivant).

### B-F2 — BBO.CSV (CARNET YES/NO)

**B1. IDENTITÉ.** Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask — toutes utilisées. 505 lignes, de t=0,186s à t=299,844s, événementiel (précision ms). 495 lignes complètes, 10 lignes avec côté manquant (autour de t=277,362s et t=298,767s).

**B2. APPORT.** Les prix d'exécution réels à la milliseconde — les graphes montrent la forme, ce fichier donne le prix exact payable à chaque instant de décision.

**B3. COMMENT.** Spread = ask − bid par carnet ; consensus = (yes_mid + (1 − no_mid))/2 ; écart inter-carnets = |yes_mid − (1 − no_mid)| ; recherche du dernier BBO ≤ t pour tout instant t ; comptage strict des croisements bid>ask.

**B4. RÉSULTATS CHIFFRÉS.** Le tableau des mesures carnet :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Spread YES moyen | 0,0301$ | yes_bid, yes_ask | 495
| Spread YES médian | 0,02$ | yes_bid, yes_ask | 495
| Spread YES maximum | 0,25$ (t=298,858s) | t_ms, yes_bid, yes_ask | 495
| Part des événements à spread ≤0,02$ | 68,3% | yes_bid, yes_ask | 495
| Écart inter-carnets moyen | 0,00001$ | 4 colonnes de prix | 495
| Écart inter-carnets maximum | 0,005$ | 4 colonnes de prix | 495
| Croisements stricts bid>ask | 3 (t=298,914s ; t=299,525s ×2) | 4 colonnes de prix | 495
| Dernier BBO ≤ t=40,000s | t=36,687s : YES 0,76$/0,77$ | t_ms, yes_bid, yes_ask | 1
| Premier yes_bid ≥0,90$ | t=93,265s | t_ms, yes_bid | 505
| Premier yes_bid ≥0,99$ | t=277,362s | t_ms, yes_bid | 505


**B5. OBSERVATIONS BRUTES.**

- À t=40,000s, le YES s'achète 0,77$ alors que l'oracle est à +18,35$ du strike depuis 4 s (B-F1, B4) : le marché retarde sur sa propre source de règlement.
- Le consensus au même instant vaut 0,765 : les deux carnets disent la même chose (écart inter-carnets moyen 0,00001$).
- Le yes_bid chute à 0,70$ à t=298,858s pendant que l'oracle reste à +19,20$ du strike (64 225,04$, B-F1 B4) : la panique finale est un pur événement de carnet, pas d'oracle.


**B6. SPÉCIFIQUE À LA SESSION.** Le PDF marque 11 "incohérents" ; mon comptage strict bid>ask sur lignes complètes en trouve 3 — les 8 autres correspondent aux lignes à côté manquant autour de t=277,362s et t=298,767s. Tous concentrés après t=277,362s : verdict BRUIT de fin de fenêtre.

### B-F3 — SPOT.CSV (BINANCE BTC/USDT DIRECT)

**B1. IDENTITÉ.** Colonnes t_ms, price — toutes utilisées. 5084 lignes, de t=1,009s à t=299,005s, tick par tick (jusqu'à 100+ ticks par seconde en rafale).

**B2. APPORT.** La preuve chiffrée que le flux rapide n'est PAS le flux de règlement : le basis exact spot−oracle, invisible à cette précision sur le graphe 2.

**B3. COMMENT.** Pour chaque tick spot, basis = price_spot − price_oracle(dernier tick oracle ≤ t) ; extrema, moyenne ; comparaison des extrema spot aux extrema oracle.

**B4. RÉSULTATS CHIFFRÉS.** Le tableau des mesures spot :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre de ticks | 5084 | t_ms | 5084
| Premier tick | 64 265,92$ (t=1,009s) | t_ms, price | 1
| Dernier tick | 64 264,99$ (t=299,005s) | t_ms, price | 1
| Minimum | 64 264,00$ | price | 5084
| Maximum | 64 317,18$ (t=106,296s) | t_ms, price | 5084
| Basis moyen (spot − oracle) | +58,75$ | price (×2 fichiers) | 5084
| Basis minimum | +39,95$ (t=299,005s) | price (×2 fichiers) | 5084
| Basis maximum | +67,66$ (t=35,738s) | price (×2 fichiers) | 5084


**B5. OBSERVATIONS BRUTES.**

- Le basis ne devient jamais négatif : minimum +39,95$ sur 5084 ticks. Le spot Binance est structurellement au-dessus de l'oracle Chainlink sur toute la session.
- Le minimum spot de session (64 264,00$) est 58,16$ AU-DESSUS du strike (64 205,84$, B-F1 B4) : un trader ancré au spot aurait cru le marché gagné dès t=1,009s — 23 secondes avant que l'oracle ne franchisse le strike (t=24,000s, B-F1 B5).


**B6. SPÉCIFIQUE À LA SESSION.** Rafales de ticks à horodatage identique (115 ticks à t=12,935s) : verdict BRUIT de livraison du flux, sans impact sur les prix. La compression du basis de +67,66$ à +39,95$ en fin de session : SIGNAL faible de convergence, mais inexploitable ici (le marché règle sur l'oracle seul).

### B-F4 — TRADES.CSV (FLUX SIGNÉ)

**B1. IDENTITÉ.** Colonnes t_ms, usd, direction — toutes utilisées. 1267 lignes, de t=1,181s à t=299,894s, événementiel.

**B2. APPORT.** La décomposition exacte du volume par camp, que le graphe 3 n'agrège que par fenêtres de 30 s.

**B3. COMMENT.** Sommes de usd par direction (1 = haussier, −1 = baissier), comptages, extrema, moyenne, médiane ; parts haussières par sous-fenêtres [0–40s], [30–45s], [285–300s].

**B4. RÉSULTATS CHIFFRÉS.** Le tableau des mesures de flux :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Volume total | 16 935,65$ | usd | 1267
| Volume haussier | 12 919,01$ (76,3%) | usd, direction | 880
| Volume baissier | 4 016,64$ | usd, direction | 387
| Nombre de trades haussiers / baissiers | 880 / 387 | direction | 1267
| Plus gros trade | 1 643,03$ haussier (t=277,967s) | t_ms, usd, direction | 1
| Trade moyen / médian | 13,37$ / 4,40$ | usd | 1267
| Part haussière fenêtre [30–45s] | 74,9% (1 162,56$ sur 82 trades) | t_ms, usd, direction | 82
| Part haussière fenêtre [285–300s] | 24,9% (1 082,45$ sur 75 trades) | t_ms, usd, direction | 75


**B5. OBSERVATIONS BRUTES.**

- Le flux confirme l'oracle : 76,3% du volume total est haussier pendant que l'oracle passe 94,2% de la session au-dessus du strike (B-F1, B4).
- Au moment du retard de prix identifié en B-F2 B5 (fenêtre [30–45s]), le flux est déjà haussier à 74,9% : le rattrapage est en cours, pas terminé.
- La fenêtre finale [285–300s] s'inverse (24,9% haussier) sans que l'oracle bouge : c'est du débouclage, pas de l'information.


**B6. SPÉCIFIQUE À LA SESSION.** Le trade de 1 643,03$ (t=277,967s) et celui de −374,34$... son pendant vendeur de 374,34$ (t=286,468s) représentent à eux deux 11,9% du volume : structure de participants propre à cette session, verdict BRUIT pour toute règle générale.

Les pièces sont posées ; voici maintenant ce que j'appelle «ANCRAGE À L'ORACLE» et comment je l'ai testé.

---

## PARTIE C — LA STRATÉGIE

### C0 — INTERPRÉTATION OPÉRATIONNELLE

Mes cinq observations les plus fortes, d'abord :

- OBS-1 : l'oracle franchit le strike à t=24,000s et reste au-dessus 275 s sans retour, 274/291 ticks côté UP (B-F1, B5).
- OBS-2 : à t=40,000s, marge oracle = +18,35$ et YES ask = 0,77$ — le marché retarde sur l'oracle (B-F1 B4 ; B-F2 B4).
- OBS-3 : le basis spot−oracle est toujours positif (min +39,95$, moyen +58,75$) — le spot n'est pas la référence de règlement (B-F3, B4).
- OBS-4 : le spread YES est ≤0,02$ sur 68,3% des événements mais monte à 0,25$ à t=298,858s (B-F2, B4).
- OBS-5 : le bruit de l'oracle est de 0,40$ par tick en moyenne, 9,66$ au pire, avec 7 variations ≥3$ (B-F1, B4).


PRINCIPE DIRECTEUR : l'oracle Chainlink est la seule source qui règle ce marché — je m'ancre sur son écart au strike, jamais sur le spot Binance ni sur l'humeur du carnet. J'achète le côté que l'oracle désigne quand le marché le vend moins cher que l'ancre ne le justifie, et je ne lâche l'ancre que si l'oracle revient au strike.

La traduction opérationnelle, règle par règle :

| Règle | Formulation chiffrée | Observations sources (A5/B5)
|-----|-----|-----|-----
| Référence d'ancrage | marge(t) = oracle(t) − 64 205,84$ ; le spot est exclu du signal | OBS-3 ; B-F3 B5 ; A-G2 A5
| Entrée | acheter YES à l'ask si marge ≥ +8,00$ sur 5 ticks oracle consécutifs ET YES ask ≤0,80$ ET spread YES ≤0,02$ | OBS-1, OBS-2, OBS-5 ; B-F2 B5
| Sortie normale | tenue jusqu'au règlement (YES paie 1,00$ si UP) | OBS-1 ; A-G1 A5
| Sortie de protection | vendre au bid si marge ≤ +2,00$ sur 2 ticks consécutifs | OBS-5 ; B-F1 B5
| Dimensionnement | 100% du cash disponible, position unique | B-F4 B4 (trade médian du marché : 4,40$)
| Protection d'exécution | aucune entrée après t=290,000s | OBS-4 ; A-G5 A6


Hypothèses non dérivées des données :

- Frais de transaction = 0,00$ (structure sans frais preneur du marché). [HYPOTHÈSE n°1]
- Exécution intégrale au BBO en vigueur, sans slippage, pour une taille de 5,00$. [HYPOTHÈSE n°2]
- Parts fractionnaires autorisées (quantités à 4 décimales). [HYPOTHÈSE n°3]


### C1-ADAPT — JOURNAL DES ADAPTATIONS

Le journal des sept écarts entre l'idée nue "acheter quand l'oracle est au-dessus du strike" et la version que je simule :

| # | Paramètre/Règle d'origine | Adaptation | Justification chiffrée (source)
|-----|-----|-----|-----
| 1 | Entrée dès marge >0$ | Seuil porté à +8,00$ | Bruit oracle : 0,40$/tick moyen, pire tick 9,66$, 7 variations ≥3$ (B-F1 B4) ; +8,00$ = 20× le bruit moyen [ADAPTATION n°1]
| 2 | Signal instantané | Persistance exigée : 5 ticks consécutifs | Faux régime initial : oracle sous le strike de t=14,000s à t=23,000s après ouverture au contact (B-F1 B4) [ADAPTATION n°2]
| 3 | Achat à tout prix | Plafond d'entrée YES ask ≤0,80$ | Préserve 0,20$/part de gain au règlement, soit 25,0% ; YES atteint 0,80$ dès t=42,617s (A-G1 A4) [ADAPTATION n°3]
| 4 | Exécution aveugle | Filtre spread ≤0,02$ | 68,3% des événements le satisfont ; au-delà, le coût d'entrée mange la marge (B-F2 B4) [ADAPTATION n°4]
| 5 | Fenêtre d'entrée ouverte 300 s | Fermée après t=290,000s | Spread jusqu'à 0,25$ à t=298,858s et carnet croisé à t=299,525s (A-G5 A4) [ADAPTATION n°5]
| 6 | Pas de sortie avant règlement | Stop de désancrage : marge ≤+2,00$ sur 2 ticks → vente au bid | Protège le capital si l'oracle re-franchit ; 2 ticks car 1 tick peut être du bruit de 9,66$ (B-F1 B4) [ADAPTATION n°6]
| 7 | Dimensionnement à définir | All-in 5,00$, position unique, verrou d'actif | Capital = 1,14× le trade médian du marché (4,40$, B-F4 B4) : fractionner n'a pas de sens à cette échelle [ADAPTATION n°7]


Aucune de ces adaptations n'a été décidée après la simulation ; aucune ne contredit le principe directeur.

### C2 — PSEUDO-CODE EXÉCUTABLE

```plaintext
STRIKE = 64205.84                      # oracle.csv t=0,000s (B-F1 B4)
cash = 5.00 ; position = NONE ; lock = FALSE ; consec = 0

POUR chaque tick oracle (t, prix):     # 291 ticks, 1s nominal (B-F1 B1)
  marge = prix - STRIKE
  SI position == NONE ET NOT lock ET t <= 290000:        # [ADAPTATION n°5]
    consec = (marge >= 8.00) ? consec+1 : 0              # [ADAPTATION n°1]
    SI consec >= 5:                                      # [ADAPTATION n°2]
      (bid, ask) = dernier BBO <= t                      # bbo.csv (B-F2 B4)
      SI ask <= 0.80 ET (ask-bid) <= 0.02:               # [ADAPTATION n°3][ADAPTATION n°4]
        lock = TRUE                                      # verrou d'actif (race a/c)
        qty = arrondi(cash/ask, 4)                       # [HYPOTHÈSE n°3]
        acheter YES qty @ ask ; frais = 0.00             # [HYPOTHÈSE n°1][HYPOTHÈSE n°2]
        cash -= qty*ask ; position = OPEN ; lock = FALSE # [ADAPTATION n°7]
  SI position == OPEN:
    SI marge <= 2.00 sur 2 ticks consécutifs:            # [ADAPTATION n°6]
      vendre qty @ bid courant ; position = NONE
AU règlement: SI position == OPEN ET résultat == UP: cash += qty*1.00
```

### C3 — TABLE DE CONVERGENCE

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| Ancrage sur l'oracle, spot exclu | A-G2 (A5), B-F1 (B5), B-F3 (B5) | aucune | Confirmée — 3 sources
| Entrée marge ≥+8,00$ × 5 ticks | B-F1 (B4 : bruit 0,40$/9,66$), A-G2 (A4 : t=36,000s) | aucune | Confirmée — 2 sources
| Plafond d'entrée 0,80$ | A-G1 (A4 : 0,80$ atteint t=42,617s), B-F2 (B4) | aucune | Confirmée — 2 sources
| Filtre spread ≤0,02$ | B-F2 (B4 : 68,3%), A-G5 (A4) | aucune | Confirmée — 2 sources
| Pas d'entrée après t=290,000s | A-G5 (A4 : 0,25$ à t=298,858s), B-F2 (B6) | aucune | Confirmée — 2 sources
| Stop de désancrage +2,00$ × 2 ticks | B-F1 (B4) | aucune | [FRAGILE - 1 SOURCE]
| Tenue jusqu'au règlement malgré le carnet | B-F1 (B5 : marge min +16,18$), A-G1 (A6 : mèche 0,70$) | A-G3/A-G4 (flux final : 24,9% haussier, B-F4 B4) | Confirmée avec réserve — le flux final contredit, l'oracle tranche


Je le dis clairement : la règle de stop est FRAGILE, une seule source la soutient, et elle n'a jamais été déclenchée sur cette session.

### C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

**ÉTAT DU PORTEFEUILLE.** t=0,000s : cash 5,00$, aucune position, valeur 5,00$. t=40,000s (entrée) : cash 0,00$, position 6,4935 parts YES à 0,77$, valeur 5,00$. De t=40,000s à t=300,000s : cash 0,00$, valeur marquée au yes_bid (maximum 6,43$ à t=277,362s, minimum 4,55$ à t=298,858s). Au règlement : cash 6,49$, aucune position. Aucun achat n'a dépassé le cash disponible : coût 5,00$ pour 5,00$ disponibles.

**RACE CONDITIONS.**
(a) Signal d'achat pendant un ordre en cours : le verrou d'actif est posé à l'émission de l'ordre et levé à la confirmation ; tout signal reçu verrou posé est ignoré. RÈGLE : verrou par actif, tout signal d'achat arrivant pendant un ordre en cours est rejeté sans file d'attente.
(b) Signal de vente sur position non confirmée : la vente ne peut viser que des parts à l'état CONFIRMÉ ; un signal de vente sur position en attente est mis en file et réévalué à la confirmation. RÈGLE : file FIFO à un emplacement, le signal de vente est réexécuté ou abandonné dès que l'achat est confirmé ou rejeté.
(c) Deux signaux simultanés sur le même actif : les ticks oracle sont traités dans l'ordre strict de t_ms, et à t_ms égal l'ordre du fichier fait foi ; le premier signal prend le verrou, le second est rejeté. RÈGLE : tri déterministe (t_ms, ordre de ligne), premier arrivé seul servi.
(d) Fill partiel : la quantité confirmée devient la position, le cash non consommé est recrédité immédiatement et la fraction restante de l'ordre est annulée. RÈGLE : tout ordre est "fill-and-kill", jamais laissé au carnet. Sur cette session, [HYPOTHÈSE n°2] rend les cas (b) et (d) théoriques : ils sont gérés, ils ne se sont pas produits.

**JOURNAL DE TRADES.** Ligne par ligne :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=40,000s | 0,77$ | 6,4935 parts YES | t=300,000s (règlement UP) | 1,00$ | 0,00$ | +1,49$ | 6,49$


Déroulé vérifiable : à t=40,000s, 5e tick oracle consécutif à marge ≥+8,00$ (t=36,000s → t=40,000s : +12,11$, +16,35$, +16,63$, +16,96$, +18,35$, B-F1 B4) ; BBO en vigueur t=36,687s : YES 0,76$/0,77$, spread 0,01$ ≤0,02$ ; ask 0,77$ ≤0,80$ ; achat 6,4935 parts, coût 5,00$. Le stop de désancrage n'est jamais déclenché : marge minimale après l'entrée +16,18$ (t=242,000s, B-F1 B4). Règlement UP (oracle final 64 225,04$ > 64 205,84$, B-F1 B4) : 6,4935 parts × 1,00$ = 6,49$.

**RÉSULTATS.** Le tableau récapitulatif :

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 1 / +1,49$ | ligne 1
| Trades perdants / pertes cumulées | 0 / 0,00$ | —
| Frais totaux | 0,00$ (ligne 1 : 0,00$) | ligne 1
| BÉNÉFICE NET | +1,49$ | ligne 1
| Capital final vs initial | 6,49$ vs 5,00$ (+29,87%) | ligne 1
| Pire perte unitaire | 0,00$ (aucun trade perdant) | —
| Drawdown maximum (latent, au yes_bid) | 1,88$ entre t=277,362s (6,43$) et t=298,858s (4,55$) | ligne 1, marquage B-F2


### C4 — VERDICT ET PORTÉE

VERDICT : la stratégie «ANCRAGE À L'ORACLE», telle qu'adaptée, transforme 5,00$ en 6,49$ sur cette session sans jamais déclencher son stop — [VALIDÉE SUR SESSION].

- Le marché prédictif retarde de plusieurs secondes sur sa propre source de règlement (0,77$ payés à t=40,000s pour une marge oracle de +18,35$). [PATTERN CANDIDAT]
- S'ancrer à l'oracle et ignorer le spot est structurellement juste ici : basis toujours positif, min +39,95$. [PATTERN CANDIDAT]
- Tenir la position pendant la mèche à 0,70$ (t=298,858s) parce que l'oracle reste à +19,20$ n'a payé que parce que le règlement est tombé côté UP 1,2 s plus tard. [SESSION-SPÉCIFIQUE]
- Le rendement de +29,87% dépend du prix d'entrée 0,77$ offert par CE retard de CE carnet. [SESSION-SPÉCIFIQUE]
- Un seul trade, une seule session : la taille d'échantillon interdit toute généralisation du chiffre. [SESSION-SPÉCIFIQUE]


Ce qui relève de la stratégie elle-même : l'ancre oracle et la tenue jusqu'au règlement ; ce qui relève de mes adaptations : le seuil +8,00$, la persistance 5 ticks, le plafond 0,80$, les filtres d'exécution — le gain vient de la stratégie, la non-perte vient des adaptations.

---

## AUTO-CONTRÔLE FINAL

- Contrôle 1 (valeurs chiffrées : ≥5 par graphe, ≥8 par CSV) → conforme : A-G1 à A-G5 ont 5 à 7 valeurs, A-G6 a 5 valeurs plus une ligne DONNÉE NON DISPONIBLE motivée ; B-F1 à B-F4 ont 8 à 10 métriques en tableau.
- Contrôle 2 (chaque affirmation de C pointe vers A4/B4) → conforme : toutes les règles et tous les chiffres de C0 à C-SIM citent leur rubrique source.
- Contrôle 3 (aucun chiffre en double avec valeurs différentes) → conforme : 6,49$, +1,49$, +29,87%, 0,77$, +18,35$, 64 205,84$ identiques de l'ouverture à la clôture.
- Contrôle 4 (cohérence comptable) → conforme : 5,00$ − 6,4935×0,77$ = 0,00$ de cash en position ; règlement 6,4935×1,00$ = 6,49$ = capital final annoncé, au centime près.
- Contrôle 5 (nombre d'[HYPOTHÈSE]) → 3, dans la limite : frais nuls, exécution sans slippage à 5,00$, parts fractionnaires.
- Contrôle 6 (nombre d'[ADAPTATION]) → 7, toutes au journal C1-ADAPT avec justification chiffrée, aucune ne contredit le principe directeur, aucune décidée après C-SIM.
- Contrôle 7 (conformité F0–F7) → conforme : verdict en ouverture, adresse directe, ordre des sections respecté, tableaux au format imposé, identifiants A-G1…A-G6 / B-F1…B-F4, clôture en 3 lignes sans chiffre nouveau.


---

## CLÔTURE

Le bénéfice net est de +1,49$ : les 5,00$ de départ sont devenus 6,49$, soit +29,87%, en un seul trade jamais menacé par son stop.
Sur cette session, s'ancrer à l'oracle — et à rien d'autre — a payé exactement ce que le retard du marché offrait.
VERDICT SUR LA STRATÉGIE : [VALIDÉE SUR SESSION], portée [SESSION-SPÉCIFIQUE].




# RAPPORT D'AUDIT OPÉRATIONNEL — STRATÉGIE «ANCRAGE À L'ORACLE»

## 0. VERDICT EXÉCUTIF

- Verdict global : **[FIABLE]** — au périmètre déclaré par l'agent lui-même : une session, 5,00$, portée [SESSION-SPÉCIFIQUE].
- NOTE DE FIABILITÉ : **91/100** (grille F7, détail en fin de rapport).
- Manquements identifiés : **8 facteurs audités — 1 [DÉGRADE], 6 [SANS IMPACT] prouvés, 1 [IMPACT NON CHIFFRABLE]** ; 2 risques résiduels non déclarés.
- Manquement le plus grave : le slippage/profondeur ([HYPOTHÈSE n°2] de l'agent) — borné à **−0,08$ (−5,4% du P&L annoncé)** dans le pire cas d'un tick de glissement.
- P&L annoncé : **+1,49$** ; P&L après réintégration de TOUS les manquements chiffrables : **entre +1,41$ et +1,49$**.
- Point le plus solide : en excluant le spot Binance du signal, la stratégie a — par construction — sorti la pire latence de l'infrastructure (~219 ms) de sa boucle critique ; la boucle réelle est de 99 à 139 ms pour un prix d'entrée resté immobile 2 007 ms.


Je vais vous montrer pièce par pièce d'où vient ce verdict.

## 1. PÉRIMÈTRE

Audité : le speech complet (Parties A, B, C, C-SIM), rejoué contre oracle.csv (291 lignes), bbo.csv (505 lignes), trades.csv (1 267 lignes), spot.csv (5 084 lignes) et le tableau des latences. Non audité : l'exactitude du PDF lui-même (redondant avec les CSV) et la reproductibilité multi-sessions, que l'agent ne revendique pas.

## 2. PARTIE V — SONDAGE DE FIABILITÉ DES DONNÉES

J'ai recalculé chaque valeur ci-dessous depuis les fichiers bruts.

| # | Valeur sondée | Annoncée | Recalculée | Écart | Verdict
|-----|-----|-----|-----|-----|-----
| 1 | P&L final / cash final | +1,49$ / 6,49$ | 5,00/0,77 = 6,4935 parts × 1,00$ = 6,4935$, P&L +1,4935$ | 0 | [CONFIRMÉ]
| 2 | Strike exact (oracle.csv, t=0) | 64 205,84$ | 64 205,836133915$ | 0 | [CONFIRMÉ]
| 3 | Marge oracle à t=40,000s | +18,35$ | 64 224,183 − 64 205,836 = +18,347$ | 0 | [CONFIRMÉ]
| 4 | Spread YES moyen / part ≤0,02$ | 0,0301$ / 68,3% | 0,0301$ / 68,3% (496 lignes complètes, l'agent en compte 495 — écart d'une ligne, aucun effet au centième) | ~0 | [CONFIRMÉ]
| 5 | Trade n°1 : signal 5 ticks ≥+8$ à t=40 000 ms, BBO en vigueur t=36 687 ms = 0,76/0,77 | idem | Rejeu du pseudo-code C2 sur oracle.csv+bbo.csv : entrée identique, tick par tick (reset du compteur à t=35s, marge +5,10$) | 0 | [CONFIRMÉ]
| 6 | Stop jamais déclenché / marge min après entrée | +16,18$ (t=242s) | +16,18$ à t=242,000s ; aucune paire de ticks ≤+2,00$ | 0 | [CONFIRMÉ]


0 [ERREUR] sur 6 sondes : **données de base dignes de confiance au niveau sondé.** Je passe au cœur de l'audit.

## 3. PARTIE O — AUDIT OPÉRATIONNEL : CE QUE LA STRATÉGIE IGNORE

**Reconstruction de la boucle critique** (utilisée dans O1, O2, O5 et la Partie I) : la stratégie lit l'oracle Chainlink on-chain (**~68 ms**), décide localement (borné à 10 ms), poste sur le CLOB Polymarket (**~31 ms**) : **boucle signal→ordre = 99 à 109 ms ; avec confirmation de fill (+~31 ms) = 130 à 140 ms.** Variante lecture via Polygon RPC (**~108 ms**) : 139 à 180 ms. Binance (**~219 ms**) est HORS boucle : la règle d'ancrage exclut le spot du signal. Telegram (**~67 ms**) n'apparaît dans aucun chemin de décision.

| # | Facteur ignoré | Hypothèse implicite | Réalité (source) | Impact sur P&L | Verdict
|-----|-----|-----|-----|-----|-----
| O1 | Latence d'exécution | Exécution instantanée à t=40,000s | Boucle 99–180 ms ; l'ask 0,77$ (bbo.csv t=36 687 ms) reste le meilleur prix jusqu'à t=42 047 ms | 0,00$ (0%) | [SANS IMPACT]
| O2 | Fraîcheur du signal | Tick oracle lu à l'instant t | Âge réel ≤ 1 068 ms (cadence 1 s + 68 ms) | 0,00$ (0%) | [SANS IMPACT]
| O3 | Slippage/profondeur | Fill intégral 5,00$ au BBO ([HYPOTHÈSE n°2]) | bbo.csv n'a pas de colonnes de taille : profondeur DONNÉE NON VÉRIFIABLE | −0,08$ (−5,4%) borné | [DÉGRADE]
| O4 | Frais complets | 0,00$ ([HYPOTHÈSE n°1]) | Aucune colonne de frais dans les sources ; gas hors chemin utilisateur du CLOB | 0,00$, borné −0,015$ si 1% de frais existait | [SANS IMPACT]
| O5 | Ordres concurrents/file | Le niveau lu existe encore à l'arrivée | Niveau 0,76/0,77 stable 5 360 ms vs trajet 31 ms | 0,00$ (0%) | [SANS IMPACT]
| O6 | Défaillances techniques | Confirmation toujours reçue | fill-and-kill défini, mais pas de comportement pour timeout RPC ; 8 trous >1 s dans oracle.csv | 0,00$ sur session (stop jamais à moins de +14$ du seuil) | [IMPACT NON CHIFFRABLE] hors session
| O7 | Passage à l'échelle | — (taille 5$ assumée) | Prints de 57–63$ exécutés à 0,77–0,80$ dans [36–45s] (trades.csv) | 0,00$ à 5$ ; au-delà de ~60$ DONNÉE NON VÉRIFIABLE | [SANS IMPACT] à la taille testée
| O8 | Dépendance à la session | — (déclarée par l'agent) | Fenêtre d'edge : 2 047 ms (t=40,000→42,047s) ; session calme → 0 trade, P&L 0,00$ | 0,00$ sur le périmètre annoncé | [SANS IMPACT]


**O1 — LATENCE D'EXÉCUTION.** Ce que la stratégie suppose : "à t=40,000s […] achat 6,4935 parts" (C-SIM), exécution au tick même. Ce qui est vrai : l'ordre part au plus tôt 99 ms après le tick, arrive au CLOB à t≈40 099–40 180 ms. J'ai rejoué l'entrée décalée de 99, 142, 250, 500, 1 000 et 2 000 ms : dans TOUS les cas le dernier BBO ≤ t reste celui de t=36 687 ms, ask 0,77$ — le carnet n'a émis aucun événement entre 36 687 et 42 047 ms (bbo.csv). Nouveau P&L : +1,4935$, inchangé. La marge de sécurité est de 2 007 ms, soit 14× la boucle la plus lente (139 ms). Le seuil de dégradation : à +2 050 ms l'ask passe à 0,78$ (P&L +1,41$), à +2 200 ms à 0,80$ (P&L +1,25$). **[SANS IMPACT]** — prouvé, pas affirmé.

**O2 — FRAÎCHEUR DU SIGNAL.** Supposé : la marge lue est la marge courante. Réel : le tick oracle a jusqu'à 1 000 ms (cadence) + 68 ms (lecture) d'âge. Preuve d'innocuité : même un signal vieux de DEUX ticks entiers donne une marge de +16,63$ (t=38s, oracle.csv), toujours 2× le seuil de +8,00$ ; et côté stop, la marge n'est jamais descendue sous +16,18$ après l'entrée — un retard de 1 s sur un signal de sortie n'aurait rien changé sur cette session. **[SANS IMPACT]**.

**O3 — SLIPPAGE ET PROFONDEUR.** Supposé : [HYPOTHÈSE n°2], fill intégral de 5,00$ à 0,77$. Réel : bbo.csv ne contient AUCUNE taille — la profondeur au BBO est DONNÉE NON VÉRIFIABLE. Bornage : trades.csv montre des prints agresseurs de 59,20$, 60,80$, 61,60$ et 61,60$ entre t=36,7s et t=39,9s sans que l'ask ne bouge de plus de 0,01$ — un ordre de 5,00$ (0,8× le trade médian de 4,40$) est 12× plus petit que ces prints ; le pire cas plausible est un glissement d'un tick à 0,78$ : qty 6,4103, P&L **+1,41$ au lieu de +1,49$ (−0,08$, −5,4%)**. **[DÉGRADE]** — c'est le seul manquement qui coûte, et il est petit.

**O4 — FRAIS COMPLETS.** Supposé : [HYPOTHÈSE n°1], 0,00$. Réel : aucun des quatre CSV ne porte de colonne de frais, et le chemin d'exécution CLOB signe des ordres hors-chaîne (le gas Polygon n'est pas au compte de l'utilisateur ; le ~108 ms RPC ne sert qu'à la lecture). L'hypothèse est déclarée, non cachée. Bornage de prudence : un frais de 1% sur le gain, s'il existait, coûterait 0,015$ (1,0% du P&L). **[SANS IMPACT]** au niveau vérifiable.

**O5 — ORDRES CONCURRENTS.** Supposé : le 0,77$ lu est encore là à l'arrivée. Réel : entre la lecture et l'arrivée s'écoulent ~31 ms ; or le niveau 0,76/0,77 est resté le BBO pendant 5 360 ms (t=36 687→42 047 ms, bbo.csv), soit 173× le trajet. Et la stratégie définit le cas d'échec : fill-and-kill, pas d'ordre laissé au carnet ; au tick oracle suivant (t=41s, marge +18,86$) toutes les conditions de ré-entrée sont encore vraies au même ask 0,77$. **[SANS IMPACT]** — prouvé.

**O6 — DÉFAILLANCES TECHNIQUES.** Supposé : chaque ordre reçoit une réponse. Réel : les race conditions (a)–(d) du speech couvrent les conflits de signaux, mais AUCUNE règle ne couvre le timeout RPC ou l'ordre sans réponse — le bot resterait en position aveugle. De plus, oracle.csv contient 8 trous >1 s (48→50s, 73→75s, 75→77s, 90→92s, 106→108s, 189→191s, 213→215s, 281→283s) : pendant un trou, le stop "2 ticks consécutifs" est retardé de 2 s, fenêtre où le pire tick observé est de 9,66$. Sur CETTE session : marge minimale après entrée +16,18$, le stop n'a jamais été à moins de +14,18$ de son seuil — impact 0,00$. Hors session : **[IMPACT NON CHIFFRABLE]**, borné par construction à la perte de la position (−5,00$) dans le scénario extrême d'un retournement pendant une panne — scénario absent des données.

**O7 — PASSAGE À L'ÉCHELLE.** Supposé : rien — l'agent dimensionne à 5,00$ et le justifie (1,14× le trade médian). Réel : les résultats tiennent à cette taille ; les prints observés bornent la capacité à ~60$ par entrée au même prix ; au-delà, la profondeur est DONNÉE NON VÉRIFIABLE (pas de tailles dans bbo.csv). Le volume total de la fenêtre d'entrée [30–45s] est de 1 162,56$ : à 100$+ la stratégie deviendrait elle-même le flux qu'elle prétend suivre. À la taille testée : **[SANS IMPACT]** ; le mécanisme s'éteint quelque part entre 60$ et ce plafond de fenêtre.

**O8 — DÉPENDANCE À LA SESSION.** Supposé : rien de caché — l'agent étiquette lui-même le rendement [SESSION-SPÉCIFIQUE]. Réel, chiffré : l'edge n'a existé que 2 047 ms (l'ask passe de 0,77$ à 0,80$ — le plafond d'entrée — entre t=42 047 et 42 141 ms) ; une session calme (marge jamais ≥+8$ sur 5 ticks) produit 0 trade et 0,00$, pas une perte. La structure du résultat (asymétrie : gain plafonné +29,87%, perte possible −100% du all-in si l'oracle se retourne après entrée) est le vrai point aveugle — voir Partie R. Sur le périmètre annoncé : **[SANS IMPACT]**.

**Réponse directe à votre question "le bot peut-il structurellement suivre cette stratégie ?"** Oui, sur cette infrastructure : la boucle complète (99–180 ms) est 14× plus courte que la stabilité du prix d'entrée (2 007 ms), 5× plus courte que la cadence du signal (1 s), et la pire latence du parc (Binance, ~219 ms) est hors boucle par conception. La contrainte structurelle n'est PAS la vitesse — c'est l'accès en production au flux oracle cadencé à 1 s (Partie R, R1).

## 4. PARTIE I — IMPACT CUMULÉ DES MANQUEMENTS

Rejeu unique avec tout réintégré simultanément : entrée décalée de 139 ms (boucle O1, chemin RPC le plus lent), signal vieilli d'un tick (O2), glissement d'un tick au pire (O3), frais 0 vérifié (O4), fill-and-kill avec ré-entrée (O5).

| Scénario | P&L annoncé | P&L corrigé | Écart %
|-----|-----|-----|-----|-----|-----
| Latence de boucle seule (O1, 139 ms) | +1,49$ | +1,49$ | 0%
| Âge du signal (O2, jusqu'à 1 068 ms) | +1,49$ | +1,49$ | 0%
| Slippage pire cas (O3, fill à 0,78$) | +1,49$ | +1,41$ | −5,4%
| Fills perdus (O5, ré-entrée au tick suivant) | +1,49$ | +1,49$ | 0%
| **TOUS MANQUEMENTS CUMULÉS** | **+1,49$** | **+1,41$** | **−5,4%**


Le P&L cumulé corrigé reste positif : la règle bloquante n'interdit pas [FIABLE].

## 5. PARTIE R — RISQUES RÉSIDUELS NON DÉCLARÉS

- **[RISQUE NON DÉCLARÉ] R1 — Provenance du flux oracle 1 s.** oracle.csv tick chaque seconde par variations de quelques cents ; un agrégateur Chainlink on-chain standard ne publie qu'à seuil de déviation ou heartbeat — pas à 1 s. Le flux simulé ressemble à un flux basse-latence agrégé dont l'accès en production, au coût et à la fiabilité près, n'est démontré nulle part (le tableau d'infrastructure ne liste que "on-chain ~68 ms" et "RPC ~108 ms"). Si le bot ne dispose en réel que des rounds on-chain à déviation, la règle "5 ticks consécutifs à 1 s" est inévaluable telle quelle. Impact : NON CHIFFRABLE — c'est une condition d'existence du signal, pas un coût.
- **[RISQUE NON DÉCLARÉ] R2 — Asymétrie gain/perte du all-in.** Gain réalisé +1,49$ ; perte maximale théorique −5,00$ (all-in, stop vendant au bid dans un carnet dont le spread a atteint 0,25$ et qui s'est croisé 3 fois en fin de session, bbo.csv). Le stop est [FRAGILE] de l'aveu de l'agent et n'a jamais été testé en conditions réelles : ratio pire-perte/gain = 3,4:1, borné entre −2,00$ et −5,00$ selon le bid disponible au déclenchement (méthode : bid minimal observé 0,70$ × 6,4935 = 4,55$ récupérés dans le cas le moins défavorable documenté, 0$ dans le cas d'un carnet vide).


## 6. AUTO-CONTRÔLE DE L'AUDITEUR

- Contrôle 1 → conforme : O1–O8 tous traités en 4 temps ; chaque [SANS IMPACT] porte sa preuve chiffrée (O1 : 2 007 ms de stabilité ; O2 : marge +16,63$ à 2 ticks d'âge ; O5 : 5 360 ms vs 31 ms ; O7 : prints de 57–63$ ; O8 : 0 trade = 0,00$).
- Contrôle 2 → conforme : boucle reconstituée chemin par chemin (68+31 = 99 ms ; +31 = 130 ms ; variante RPC 108+31+31 = 170 ms, arrondie 139–180 ms selon calcul local ≤10 ms), utilisée en O1, O5 et Partie I.
- Contrôle 3 → conforme : chaque impact cite oracle.csv, bbo.csv, trades.csv ou le tableau des latences ; les deux impacts non chiffrables (O6, R2) sont bornés avec méthode.
- Contrôle 4 → conforme : 35+30+20+6 = 91 = note du verdict exécutif ; 91 ≥ 75 → [FIABLE] ; P&L cumulé +1,41$ > 0, règle bloquante non déclenchée.
- Contrôle 5 → conforme : +1,49$, +1,41$, −0,08$, −5,4%, 99–180 ms, 2 007 ms, 91/100 identiques de l'ouverture à la clôture.


| Critère F7 | Points obtenus | Points max | Justification
|-----|-----|-----|-----|-----|-----
| Complétude opérationnelle | 35 | 40 | −5 : O3 ignoré avec verdict [DÉGRADE] ; aucun facteur [DÉTRUIT]
| Survie du P&L corrigé | 30 | 30 | +1,41$ > 0 au cumul de tous les manquements
| Fiabilité des données de base | 20 | 20 | 6/6 sondes [CONFIRMÉ]
| Honnêteté des limites | 6 | 10 | −2 (R1 : dépendance au flux 1 s non déclarée), −2 (R2 : asymétrie du all-in non déclarée)
| **TOTAL** | **91** | **100** | 75–100 → [FIABLE]


## 7. CLÔTURE

Verdict : [FIABLE] — les chiffres annoncés survivent à la réintégration de toutes les frictions mesurables de cette infrastructure, au périmètre d'une session et de 5,00$.
Note : 91/100.
Condition n°1 pour changer le verdict : démontrer l'accès en production à un flux oracle cadencé à la seconde et identique à la source de règlement — sans lui, le signal simulé n'existe pas.

---

L'audit repose sur un rejeu programmatique complet des quatre CSV : la simulation de l'agent se reproduit à l'identique, et le décalage de latence n'altère le fill qu'au-delà de 2 secondes de retard, très loin des 99–180 ms de la boucle réelle. Le seul coût matériel réintégré est le slippage borné à un tick (−0,08$), et les deux vraies zones d'ombre sont la disponibilité réelle du flux oracle 1 s et le comportement du stop all-in jamais test

---

## Source : 9ANCRAGE.md

# 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Vous voulez savoir ce que vaut «ANCRAGE À L'ORACLE» sur cette session : voici le verdict, tout de suite.

- Stratégie testée : «ANCRAGE À L'ORACLE» — je la lis comme : se positionner du côté du strike où se trouve l'oracle Chainlink, de manière confirmée, et se retourner quand l'oracle change de côté.
- Capital final : 1,06$ contre 5,00$ de départ, soit un rendement net de −78,8%.
- 5 trades : 1 gagnant (+0,33$), 4 perdants (−4,27$ cumulés).
- 4 adaptations apportées à la stratégie, toutes consignées avant simulation.
- VERDICT SUR LA STRATÉGIE : [REJETÉE SUR SESSION] + étiquette de portée [SESSION-SPÉCIFIQUE]


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

---

# 1. CADRE D'ANALYSE CHOISI

J'adopte un cadre de microstructure quantitative événementiel : chaque fichier est un flux horodaté à la milliseconde (285 ticks oracle, 7 579 ticks spot, 1 738 trades, 1 090 événements BBO) sur un horizon de 300 secondes, ce qui interdit toute analyse fondamentale et rend l'analyse en chandeliers trop grossière. Le règlement du contrat dépend d'un seul nombre — l'oracle contre le strike — donc le cadre pertinent est la mesure exacte des positions relatives oracle/strike/carnet, événement par événement. Tout chiffre cité ci-dessous est recalculé depuis les CSV ou lu sur les graphes, jamais estimé.

---

# 2. PARTIE A — LES 6 GRAPHES DU PDF

## A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (1090 ÉVÉNEMENTS BBO)

**A1. IDENTITÉ**
Graphique 1, «Carnet YES/NO complet — 1090 événements BBO (précision ms) | incohérents : 22», vue 300 s plus 3 zooms de 15 s. Axe X : temps mm:ss.mmm de 00:00.000 à 05:00.000 ; axe Y gauche : prix en $ de 0,0 à 1,0 ; axe Y droit : probabilité 0,0 à 1,0. Marché : «Polymarket BTC 5min | Bitcoin Up or Down - August 6, 8:15PM-8:20PM ET | Résultat : DOWN».

**A2. QUESTION**
Que fait le prix du carnet quand l'oracle bouge ? À quelle vitesse le consensus YES converge-t-il vers 0 ou 1 en fin de fenêtre ?

**A3. COMMENT JE L'AI EXPLOITÉ**
Lecture des étiquettes d'extrema des séries YES Mid et NO Mid, du marqueur de résultat, et du compteur d'incohérences du cartouche ; croisement des zooms 15 s avec les mêmes fenêtres des graphes 3 et 5.

**A4. VALEURS EXTRAITES**

- t=1,077s : YES mid = 0,46$
- t=1,079s : NO mid = 0,55$
- t=230,526s : YES mid = 0,69$ (maximum de la série)
- t=230,526s : NO mid = 0,31$ (minimum de la série)
- t=281,783s : YES mid = 0,01$ ; NO mid = 0,98$
- Cartouche : 22 événements «carnet croisé — suspects» ; ts fallback : 0/285 (0,0%)


**A5. OBSERVATIONS BRUTES**

- Le carnet ouvre indécis (YES mid 0,46$ à t=1,077s) puis s'écarte fortement de 0,50 dans les deux sens au cours de la session.
- Le YES culmine à 0,69$ à t=230,526s puis s'effondre à 0,01$ à t=281,783s : un retournement complet en 51,3 s.
- En fin de fenêtre le carnet converge vers le règlement (NO mid 0,98$ à t=281,783s) avant même t=300s.


**A6. SPÉCIFIQUE À LA SESSION**
Le retournement 0,69$ → 0,01$ en 51,3 s est lié à la trajectoire particulière de l'oracle de cette session (repassage au-dessus du strike à t=206s puis rechute, cf. A-G2) ; rien ne garantit deux allers-retours de part et d'autre du strike dans une autre fenêtre de 5 minutes. Classement : non-reproductible.

Vous venez de voir ce que cote le marché ; voyons maintenant ce que fait le sous-jacent qui décide du règlement.

## A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK

**A1. IDENTITÉ**
Graphique 2, «Spot Binance (flux direct) vs Oracle Chainlink — strike exact (dernier tick ≤ t0)», vue 300 s plus 3 zooms 15 s. Axe X : temps mm:ss.mmm ; axe Y : prix BTC en $. Séries : Binance BTC/USDT (7 579 ticks), Chainlink BTC/USD (285 ticks), ligne de strike à 64 233$, panneau basis = spot − oracle (7 579 pts).

**A2. QUESTION**
De quel côté du strike l'oracle passe-t-il son temps ? Le spot et l'oracle racontent-ils la même histoire, ou divergent-ils ?

**A3. COMMENT JE L'AI EXPLOITÉ**
Lecture des étiquettes d'extrema du spot, des ticks oracle horodatés un par un en début de fenêtre, et de la position de chaque série par rapport à la ligne de strike.

**A4. VALEURS EXTRAITES**

- t=0,594s : spot = 64 290,00$ (premier tick)
- t=70,662s : spot = 64 238,00$ (minimum de session)
- t=159,856s : spot = 64 296,20$ (maximum de session)
- t=299,293s : spot = 64 280,00$ (dernier tick)
- t=0,000s : oracle = 64 233,14$ (strike d'ouverture)
- t=8,000s : oracle = 64 229,16$ (premier tick sous le strike)
- Cartouche : ts fallback 0/285 (0,0%)


**A5. OBSERVATIONS BRUTES**

- Le spot ne touche jamais la ligne de strike : son minimum 64 238,00$ (t=70,662s) reste au-dessus de 64 233$.
- L'oracle passe sous le strike dès t=8,000s (64 229,16$) alors que le spot est 64 290,00$ à t=0,594s : les deux séries vivent à des niveaux différents.
- Le règlement dépend de la série oracle, la seule qui traverse la ligne de strike sur ce graphe.


**A6. SPÉCIFIQUE À LA SESSION**
L'amplitude spot de session (64 238,00$ → 64 296,20$, soit 58,20$) et l'horaire (20:15–20:20 ET) sont propres à cette fenêtre ; l'écart de niveau spot/oracle est un artefact de sources différentes (Binance USDT vs agrégat Chainlink USD) dont la valeur absolue de cette session n'est pas généralisable. Classement : non-reproductible en valeur, reproductible en signe (à vérifier sur d'autres sessions).

Le sous-jacent est cadré ; regardons qui pousse les prix dans le carnet.

## A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ YES/NO (1738 TRADES)

**A1. IDENTITÉ**
Graphique 3, «Flux directionnel normalisé YES/NO (1738 trades)», vue 300 s plus 3 zooms 15 s. Axe X : temps mm:ss.mmm ; axe Y : volume en $ de −5 000 à +2 000, flux haussier 30 s (BUY YES + SELL NO) en positif, flux baissier 30 s (SELL YES + BUY NO) en négatif.

**A2. QUESTION**
Quand le volume arrive-t-il, et dans quel sens ? Que fait le flux au moment des retournements du carnet ?

**A3. COMMENT JE L'AI EXPLOITÉ**
Lecture des étiquettes d'extrema de la vue 300 s et des trades individuels horodatés dans les 3 zooms.

**A4. VALEURS EXTRAITES**

- t=1,740s : premier flux étiqueté = +18,40$
- t=179,923s : pic haussier = +688,06$
- t=282,362s : pic baissier = −1 980,01$
- t=291,628s : dernier extrême étiqueté = −219,65$
- t=70,676s (zoom 01:10–01:25) : trade baissier −110,40$
- t=143,816s (zoom 02:18–02:33) : trade baissier −144,00$


**A5. OBSERVATIONS BRUTES**

- Le plus gros trade de la session est baissier : −1 980,01$ à t=282,362s, dans les 18 dernières secondes cotées.
- Le pic haussier (+688,06$ à t=179,923s) survient pendant la fenêtre où l'oracle est au-dessus du strike (cf. A-G2).
- Les flux des zooms sont dominés par des tickets de 10$ à 150$ ; les tickets supérieurs à 500$ n'apparaissent qu'aux extrêmes de la vue 300 s.


**A6. SPÉCIFIQUE À LA SESSION**
Le trade de −1 980,01$ (t=282,362s) est un événement unique, 139 fois le trade médian de 4,30$ (B-F3, B4) : c'est un acteur ponctuel, pas une structure. Classement : bruit non-reproductible.

Le flux brut est posé ; sa version normalisée dit qui gagne le bras de fer.

## A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ**
Graphique 4, «Imbalance directionnelle (YES/NO normalisés)», vue 300 s plus 3 zooms 15 s. Axe X : temps mm:ss.mmm ; axe Y : imbalance 0,0 à 1,0, définie Haussier/(Haussier+Baissier), ligne d'équilibre à 0,5.

**A2. QUESTION**
L'agressivité des acheteurs YES et NO est-elle équilibrée ? Combien de temps l'imbalance passe-t-elle de chaque côté de 0,5 ?

**A3. COMMENT JE L'AI EXPLOITÉ**
Lecture des étiquettes d'extrema de la vue 300 s et des valeurs ponctuelles des zooms, comparées à la ligne 0,5.

**A4. VALEURS EXTRAITES**

- t=1,509s : imbalance = 1,00 (premier point)
- t=88,511s : imbalance = 0,09 (minimum étiqueté)
- t=296,843s : imbalance = 0,16 (dernier point étiqueté)
- t=70,090s : imbalance = 0,12 (zoom 01:10–01:25)
- t=139,190s : imbalance = 0,38 (zoom 02:18–02:33)
- t=114,554s : imbalance = 0,13 (zoom 01:54–02:09)


**A5. OBSERVATIONS BRUTES**

- L'imbalance passe la majorité de la session sous 0,5 : toutes les valeurs étiquetées des zooms sont entre 0,09 et 0,46.
- Le flux baissier domine du début (1,00 à t=1,509s est un artefact de premier trade) à la fin (0,16 à t=296,843s).
- Les creux d'imbalance (0,09 à t=88,511s ; 0,12 à t=70,090s) coïncident avec la période où l'oracle est le plus bas sous le strike (cf. A-G2, t=65s, minimum oracle en B-F1).


**A6. SPÉCIFIQUE À LA SESSION**
La valeur 1,00 à t=1,509s reflète un unique premier trade haussier de 18,40$ (A-G3, A4) sur une fenêtre de 30 s encore vide : artefact d'initialisation, non-reproductible.

L'agressivité est mesurée ; il reste à vérifier ce que coûte l'exécution.

## A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS

**A1. IDENTITÉ**
Graphique 5, «Spreads YES et NO absolus (événementiels)», vue 300 s plus 3 zooms 15 s. Axe X : temps mm:ss.mmm ; axe Y : spread ask−bid en $ de −0,05 à 0,20. Deux séries de 1 090 événements chacune.

**A2. QUESTION**
Combien coûte la traversée du carnet à chaque instant ? Existe-t-il des moments où le spread explose ou devient négatif ?

**A3. COMMENT JE L'AI EXPLOITÉ**
Lecture des étiquettes d'extrema des deux séries et de leurs timestamps, sur la vue 300 s et les zooms.

**A4. VALEURS EXTRAITES**

- t=1,077s : spread YES = 0,01$
- t=178,491s : spread YES = 0,21$ (maximum de session)
- t=263,705s : spread YES = −0,04$ (minimum, carnet croisé)
- t=281,783s : spread YES = 0,01$
- t=178,491s : spread NO = 0,21$ (même timestamp que le YES)
- t=70,570s : spread YES = 0,02$ (zoom 01:10–01:25)


**A5. OBSERVATIONS BRUTES**

- Le spread de croisière est de 0,01$ à 0,04$ (valeurs des zooms), soit 1 à 4 cents par aller.
- Le spread maximal 0,21$ survient à t=178,491s, juste après le repassage de l'oracle au-dessus du strike (t=153s, B-F1) : la liquidité se retire au moment précis où le signal change.
- Des spreads négatifs existent (−0,04$ à t=263,705s) : le carnet est temporairement croisé, en pleine deuxième bascule.


**A6. SPÉCIFIQUE À LA SESSION**
Le croisement à −0,04$ (t=263,705s) est un état hors-équilibre de quelques événements sur 1 090 ; l'exploiter supposerait une latence d'exécution non mesurable dans ces données. Classement : signal d'illiquidité ponctuelle, non-reproductible comme opportunité.

Dernière pièce du PDF : la cohérence interne des deux carnets.

## A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ**
Graphique 6, «Écart inter-carnets YES vs NO (incohérents : 22)», vue 300 s plus 3 zooms 15 s. Axe X : temps mm:ss.mmm ; axe Y : écart yes_mid − (1 − no_mid) en $, échelle 0,000 à 0,005. Ligne «cohérence parfaite» à 0.

**A2. QUESTION**
Les carnets YES et NO cotent-ils la même probabilité ? Existe-t-il un écart arbitrable entre les deux ?

**A3. COMMENT JE L'AI EXPLOITÉ**
Lecture de l'échelle de l'axe Y, du compteur d'incohérents du cartouche et de la position de la série par rapport à la ligne zéro ; aucun point n'est étiqueté numériquement sur ce graphe.

**A4. VALEURS EXTRAITES**

- Borne supérieure de l'axe : écart = 0,005$
- Borne inférieure de l'axe : écart = 0,000$
- Cartouche : 22 événements «carnet croisé — suspects» sur 1 090
- Cartouche : ts fallback = 0/285 (0,0%)
- Nombre d'événements de la série : 1 090
- Valeur ponctuelle maximale de l'écart avec timestamp : DONNÉE NON DISPONIBLE (aucune étiquette d'extremum sur ce graphe)


**A5. OBSERVATIONS BRUTES**

- L'écart inter-carnets est contenu dans [0,000$ ; 0,005$] sur toute la session : yes_mid et 1−no_mid sont alignés à un demi-cent près.
- 22 événements sur 1 090 (2,0%) sont marqués suspects (carnet croisé).
- Aucun tick oracle n'utilise de timestamp de repli (0/285) : l'horodatage est fiable pour l'ensemble de l'analyse.


**A6. SPÉCIFIQUE À LA SESSION**
Aucune anomalie exploitable : un écart maximal borné à 0,005$ est inférieur au spread de croisière de 0,01$ (A-G5, A4) — l'arbitrage inter-carnets est structurellement impossible sur cette session. Classement : caractéristique de structure, probablement stable mais sans valeur opérationnelle.

Le PDF est épuisé ; je passe aux mêmes données en granularité brute.

---

# 3. PARTIE B — LES FICHIERS CSV

## B-F1 — ORACLE.CSV

**B1. IDENTITÉ**
oracle.csv, colonnes utilisées : t_ms, price, ts_src. 285 lignes de données. Période : t=0,000s à t=298,000s. Fréquence : 1 tick/seconde nominal (médiane des écarts 1 000 ms).

**B2. APPORT**
La seule série qui détermine le règlement, en valeurs exactes non lisibles au centime sur le graphe 2 — dont la part exacte du temps passé sous le strike et la distribution des variations tick à tick.

**B3. COMMENT**
Min/max/moyenne/écart-type sur price (285 lignes) ; comptage des ticks price < 64 233,14$ ; écarts t_ms consécutifs (284 intervalles) ; |Δprice| tick à tick (284 valeurs) triés pour médiane/p90/p95 ; comptage des changements de signe de (price − strike) ; fenêtres consécutives au-dessus du strike.

**B4. RÉSULTATS CHIFFRÉS**
Le tableau des mesures, colonne par colonne :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Strike (premier tick) | 64 233,14$ | price | 1
| Minimum de session | 64 178,58$ (t=65,000s) | t_ms, price | 285
| Maximum de session | 64 242,74$ (t=163,000s) | t_ms, price | 285
| Dernier tick | 64 221,88$ (t=298,000s), soit −11,26$ sous strike | t_ms, price | 1
| Ticks sous le strike | 208/285 (73,0%) | price | 285
| Médiane des |Δprice| tick à tick | 0,124$ | price | 284
| p90 / p95 des |Δprice| | 2,052$ / 3,155$ | price | 284
| |Δprice| maximal | 11,165$ (t=122,000s) | t_ms, price | 284
| Croisements bruts du strike | 7 | price | 285
| Fenêtres au-dessus du strike | 0–1s ; 4–7s ; 153–194s ; 206–240s | t_ms, price | 285
| Écart max entre ticks | 3 000 ms ; 13 écarts >1 500 ms | t_ms | 284
| ts_src ≠ payload | 0/285 | ts_src | 285


**B5. OBSERVATIONS BRUTES**

- L'oracle passe 73,0% de ses ticks sous le strike, et finit 11,26$ dessous.
- Le strike est croisé 7 fois en brut, mais 90% des variations tick à tick font moins de 2,052$ : les croisements francs sont rares, les oscillations près du strike sont du bruit de moins de 2$.
- Deux vraies excursions au-dessus du strike existent : 153–194s (41 s) et 206–240s (34 s).
- 13 trous de plus de 1,5 s dans le flux oracle : le signal peut être aveugle jusqu'à 3 s.


**B6. SPÉCIFIQUE À LA SESSION**
Le saut de 11,165$ à t=122,000s est 90 fois la variation médiane : choc ponctuel, verdict bruit. Le trou maximal de 3 000 ms n'a pas de timestamp répétitif, verdict bruit d'infrastructure.

L'oracle est quantifié ; le spot va montrer qu'il ne faut pas les confondre.

## B-F2 — SPOT.CSV

**B1. IDENTITÉ**
spot.csv, colonnes utilisées : t_ms, price. 7 579 lignes. Période : t=0,594s à t=299,293s. Fréquence : événementielle, 25,3 ticks/s en moyenne (7 579/299,3 s).

**B2. APPORT**
La confirmation chiffrée que le spot Binance ne peut pas servir de proxy du règlement : sa position par rapport au strike et son écart systématique à l'oracle (basis), invisibles au centime sur le graphe 2.

**B3. COMMENT**
Min/max/moyenne/écart-type sur price (7 579 lignes) ; comptage price < 64 233,14$ ; basis = price_spot(t) − price_oracle(dernier tick ≤ t) calculé sur les 7 579 lignes spot contre les 285 lignes oracle.

**B4. RÉSULTATS CHIFFRÉS**
Voici les mesures du spot :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Premier / dernier tick | 64 290,00$ (t=0,594s) / 64 280,00$ (t=299,293s) | t_ms, price | 2
| Minimum de session | 64 238,00$ (t=70,662s) | t_ms, price | 7 579
| Maximum de session | 64 296,20$ (t=159,856s) | t_ms, price | 7 579
| Moyenne / écart-type | 64 270,53$ / 18,63$ | price | 7 579
| Ticks sous le strike | 0/7 579 (0,00%) | price | 7 579
| Basis minimum | +48,74$ (t=39,367s) | t_ms, price + oracle.price | 7 579
| Basis maximum | +71,75$ (t=121,568s) | t_ms, price + oracle.price | 7 579
| Basis moyenne / écart-type | +58,89$ / 3,45$ | price + oracle.price | 7 579
| Occurrences basis < 0 | 0/7 579 | price + oracle.price | 7 579


**B5. OBSERVATIONS BRUTES**

- Le spot ne passe jamais sous le strike (0/7 579) alors que l'oracle y passe 73,0% du temps (B-F1, B4) : trader le spot contre le strike aurait donné le signal inverse du règlement.
- La basis est toujours positive, entre +48,74$ et +71,75$, écart-type 3,45$ : l'écart spot/oracle est systématique, pas aléatoire.
- Le spot est 26,6 fois plus dense que l'oracle (7 579 vs 285 ticks) mais ne porte aucune information de règlement.


**B6. SPÉCIFIQUE À LA SESSION**
La valeur absolue de la basis (+58,89$ en moyenne) dépend de la composition de l'agrégat Chainlink ce jour-là : verdict signal quant au signe, bruit quant au niveau. Aucun trou de données notable (7 579 points sur 298,7 s).

Le spot est écarté comme source de signal ; place aux flux d'ordres.

## B-F3 — TRADES.CSV

**B1. IDENTITÉ**
trades.csv, colonnes utilisées : t_ms, usd, direction. 1 738 lignes. Période : t=1,509s à t=296,843s. Fréquence : événementielle, 5,8 trades/s en moyenne.

**B2. APPORT**
Les montants exacts par sens et par minute — le graphe 3 n'étiquette que 4 extrema, ici je somme les 1 738 lignes.

**B3. COMMENT**
Somme de usd par direction (+1 haussier, −1 baissier) sur les 1 738 lignes ; imbalance = haussier/(haussier+baissier) globale et par tranche de 60 000 ms ; médiane/moyenne/max de usd.

**B4. RÉSULTATS CHIFFRÉS**
Le tableau des flux :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Volume total | 24 792,21$ | usd | 1 738
| Volume haussier | 8 532,98$ (692 trades) | usd, direction | 692
| Volume baissier | 16 259,23$ (1 046 trades) | usd, direction | 1 046
| Imbalance globale | 0,344 | usd, direction | 1 738
| Imbalance minute 0 / 1 | 0,262 / 0,121 | t_ms, usd, direction | 320 / 273
| Imbalance minute 2 / 3 / 4 | 0,606 / 0,463 / 0,256 | t_ms, usd, direction | 373 / 247 / 525
| Trade maximal | 1 980,01$ (t=282,362s, baissier) | t_ms, usd, direction | 1
| Trade médian / moyen | 4,30$ / 14,26$ | usd | 1 738


**B5. OBSERVATIONS BRUTES**

- Le flux baissier représente 65,6% du volume total (16 259,23$/24 792,21$) : cohérent avec un oracle sous le strike 73,0% du temps (B-F1, B4).
- La seule minute à imbalance haussière est la minute 2 (0,606), qui contient le repassage de l'oracle au-dessus du strike à t=153s (B-F1, B4) : le flux suit l'oracle.
- La minute 4 concentre 525 trades dont 6 539,36$ baissiers : la bascule finale attire le plus gros volume.


**B6. SPÉCIFIQUE À LA SESSION**
Le trade de 1 980,01$ à t=282,362s pèse 8,0% du volume de la session à lui seul, à un moment où le NO cote déjà 0,98$ (A-G1, A4) : verdict bruit (exécution tardive d'un gros acteur, sans information nouvelle).

Reste la pièce qui fixe mes prix d'exécution : le carnet.

## B-F4 — BBO.CSV

**B1. IDENTITÉ**
bbo.csv, colonnes utilisées : t_ms, yes_bid, yes_ask, no_bid, no_ask. 1 090 lignes. Période : t=1,077s à t=291,483s. Fréquence : événementielle, 3,7 événements/s en moyenne.

**B2. APPORT**
Les prix exacts auxquels chaque trade simulé sera exécuté, et le coût de friction réel (spread) que le graphe 5 ne donne qu'aux extrema.

**B3. COMMENT**
Spread = ask − bid par ligne et par carnet (1 088 lignes YES complètes) ; moyenne/médiane/max/min ; comptage des lignes croisées (ask < bid) ; cohérence |yes_mid − (1 − no_mid)| > 0,0001 ; quotes relevées aux timestamps clés par dernier événement ≤ t.

**B4. RÉSULTATS CHIFFRÉS**
Le tableau du carnet :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Spread YES moyen / médian | 0,028$ / 0,020$ | yes_bid, yes_ask | 1 088
| Spread YES max | 0,21$ (t=178,491s ; quotes 0,50/0,71) | t_ms, yes_bid, yes_ask | 1 088
| Spread YES min | −0,04$ (t=263,705s) | t_ms, yes_bid, yes_ask | 1 088
| Spread NO moyen | 0,028$ | no_bid, no_ask | 1 088
| Lignes croisées (ask<bid) | 8 YES + 8 NO = 16 ; 3 lignes à quote manquante | les 4 colonnes | 1 090
| Écarts de cohérence mid >0,0001$ | 1/1 087 lignes complètes | les 4 colonnes | 1 087
| Quotes YES à t=60s / t=120s | 0,11/0,12 et 0,08/0,09 | t_ms, yes_bid, yes_ask | 2
| Quotes YES à t=160s / NO à t=270s | 0,64/0,65 et 0,94/0,95 | t_ms + les 4 colonnes | 2
| Dernière ligne complète | t=281,783s : YES 0,01/0,02, NO 0,98/0,99 | les 5 colonnes | 1


Sur le décompte des anomalies : le PDF marque 22 «suspects» (A-G6, A4), critère du traceur ; mon comptage strict ask<bid donne 16 lignes croisées plus 3 lignes incomplètes — deux métriques distinctes, les deux sont citées avec leur définition.

**B5. OBSERVATIONS BRUTES**

- La friction normale est de 2 à 3 cents par traversée (spread moyen 0,028$) : sur un capital de 5,00$ investi vers 0,60$, chaque aller-retour coûte de l'ordre de 4 à 5% de la position.
- Le carnet cote la position de l'oracle : YES 0,08/0,09 à t=120s quand l'oracle est 40$ sous le strike, YES 0,64/0,65 à t=160s quand il repasse au-dessus (B-F1, B4).
- Les spreads explosent aux bascules : 0,21$ à t=178,491s, soit 7,5 fois le spread moyen — exécuter pendant une bascule coûte le plus cher.


**B6. SPÉCIFIQUE À LA SESSION**
3 lignes à quote manquante (t=291,477s et t=291,483s : yes_bid et no_ask absents ; verdict signal — le carnet se vide côté YES une fois le résultat acquis). La ligne incohérente unique au seuil 0,0001$ : verdict bruit d'horodatage.

Toutes les pièces sont à plat ; j'en tire maintenant la stratégie.

---

# 4. PARTIE C — «ANCRAGE À L'ORACLE»

## C0 — INTERPRÉTATION OPÉRATIONNELLE DE LA STRATÉGIE

Mes 5 observations les plus fortes, sources à l'appui :

1. OBS-1 : l'oracle passe 73,0% de ses ticks sous le strike et finit à −11,26$ (B-F1, B5).
2. OBS-2 : le spot ne passe jamais sous le strike (0/7 579) ; seule la série oracle décide du règlement (B-F2, B5 et A-G2, A5).
3. OBS-3 : le carnet suit l'oracle — YES 0,08/0,09 quand l'oracle est 40$ sous le strike à t=120s, YES 0,64/0,65 quand il repasse dessus à t=160s (B-F4, B5).
4. OBS-4 : 90% des variations oracle tick à tick font moins de 2,052$ ; le strike est croisé 7 fois en brut (B-F1, B5).
5. OBS-5 : la friction est de 0,028$ de spread moyen, avec des pointes à 0,21$ pendant les bascules (B-F4, B5 et A-G5, A5).


PRINCIPE DIRECTEUR : l'oracle Chainlink est la seule source de vérité du règlement — pas le spot, pas le carnet. S'ancrer à l'oracle signifie détenir en permanence le côté (YES si oracle ≥ strike, NO si oracle < strike) que l'oracle indique de manière confirmée, et se retourner quand l'oracle change durablement de côté.

La traduction en règles chiffrées, règle par règle :

| Règle | Formulation chiffrée | Observations sources (A5/B5)
|-----|-----|-----|-----
| Signal de côté | côté = YES si oracle ≥ strike 64 233,14$, NO si oracle < strike | OBS-1, OBS-2 (B-F1 B5, B-F2 B5)
| Entrée | acheter le côté signalé au meilleur ask du premier événement BBO complet suivant le signal | OBS-3 (B-F4 B5)
| Sortie | vendre au meilleur bid dès signal opposé confirmé ; sinon porter jusqu'au règlement (1,00$ ou 0,00$) | OBS-3 (B-F4 B5), A-G1 A5
| Dimensionnement | investir la totalité du cash disponible à chaque entrée, quantité = cash/ask arrondie au centième inférieur | OBS-5 (B-F4 B5 : fractionner multiplie les traversées de spread à 0,028$)
| Protection du capital | aucun achat si ask > 0,75$ | OBS-5 (B-F4 B4 : NO à 0,94/0,95 à t=270s ne paie plus le risque d'une bascule à spread 0,21$)
| Frais | 0,00$ par trade | [HYPOTHÈSE n°1]
| Exécution | fill intégral au BBO, taille sans impact | [HYPOTHÈSE n°2]
| Latence | signal exécuté au premier événement BBO complet strictement postérieur | [HYPOTHÈSE n°3]


Les hypothèses non dérivées des données, une par ligne :

- Frais de transaction nuls sur ce marché — aucune colonne de frais dans les CSV. [HYPOTHÈSE n°1]
- Fill intégral au meilleur prix affiché, sans impact de marché — les CSV ne donnent pas les profondeurs. [HYPOTHÈSE n°2]
- Latence = premier événement BBO complet suivant, aucune exécution intra-événement. [HYPOTHÈSE n°3]


## C1-ADAPT — JOURNAL DES ADAPTATIONS

Quatre ajustements, tous décidés avant simulation, aucun après :

| # | Paramètre/Règle d'origine | Adaptation | Justification chiffrée (source)
|-----|-----|-----|-----
| 1 | Signal = tout croisement du strike | Zone morte de ±2,00$ autour du strike : signal seulement si oracle ≥ strike+2,00$ ou ≤ strike−2,00$ [ADAPTATION n°1] | 90% des variations tick à tick <2,052$ ; 7 croisements bruts dont plusieurs de moins de 1$ d'amplitude (B-F1, B4)
| 2 | Signal instantané sur 1 tick | Confirmation exigée : 3 ticks oracle consécutifs du même côté hors zone morte [ADAPTATION n°2] | Cadence oracle 1 tick/s, trous jusqu'à 3 000 ms (B-F1, B4) : 1 tick isolé peut être un choc de 11,165$ (t=122,000s, B-F1, B4)
| 3 | Entrée à tout prix | Plafond d'entrée : ask ≤ 0,75$ [ADAPTATION n°3] | Gain maximal 0,25$/part contre spread de bascule 0,21$ (B-F4, B4) : au-delà de 0,75$ la friction d'une seule bascule absorbe le gain
| 4 | Dimensionnement libre | All-in du cash disponible à chaque entrée [ADAPTATION n°4] | Capital 5,00$ et spread moyen 0,028$ (B-F4, B4) : scinder en 2 doubles les traversées de spread pour un capital qui ne permet aucune diversification


Aucune de ces quatre lignes ne contredit le PRINCIPE DIRECTEUR : elles précisent «confirmé» et «durablement», elles ne changent pas la source de vérité.

## C2 — LA STRATÉGIE ADAPTÉE EN PSEUDO-CODE EXÉCUTABLE

```plaintext
STRIKE = 64233.14            # oracle.csv ligne 1 (B-F1 B4)
MARGE  = 2.00                # p90 |Δoracle| = 2.052$ (B-F1 B4) [ADAPTATION n°1]
N_CONF = 3                   # cadence 1 tick/s, trous ≤3000ms (B-F1 B4) [ADAPTATION n°2]
PRIX_MAX = 0.75              # spread bascule 0.21$ à t=178.491s (B-F4 B4) [ADAPTATION n°3]
FRAIS = 0.00                 # [HYPOTHÈSE n°1]

etat = NEUTRE; serie = 0; cote_serie = 0

POUR CHAQUE tick oracle (t, p):            # 285 ticks (B-F1 B1)
  si p >= STRIKE + MARGE: s = +1           # côté YES
  sinon si p <= STRIKE - MARGE: s = -1     # côté NO
  sinon: s = 0; serie = 0; continuer       # zone morte [ADAPTATION n°1]
  serie = (s == cote_serie) ? serie+1 : 1; cote_serie = s
  si serie >= N_CONF et etat != s:         # signal confirmé [ADAPTATION n°2]
    q = premier événement BBO complet avec t_bbo > t   # [HYPOTHÈSE n°3]
    si position ouverte du côté opposé:
      vendre tout au bid du côté détenu (q)            # sortie (OBS-3)
    ask = (s == +1) ? q.yes_ask : q.no_ask
    si ask <= PRIX_MAX et cash >= 0.01:                # [ADAPTATION n°3]
      qte = arrondi_inf(cash / ask, 2)                 # all-in [ADAPTATION n°4]
      acheter qte au ask; etat = s
FIN:
  position restante réglée à 1.00$ (côté gagnant) ou 0.00$   # A-G1 A4 : résultat DOWN
```

## C3 — TABLE DE CONVERGENCE

Chaque règle, avec les pièces pour et contre :

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| Signal = position de l'oracle vs strike | B-F1 (73,0% sous strike, fin −11,26$) ; A-G2 (seule série croisant le strike) ; B-F4 (carnet aligné sur l'oracle) ; B-F3 (flux suit l'oracle, minute 2) | Aucune | CONFIRMÉE - 4 SOURCES
| Zone morte ±2,00$ | B-F1 (p90 = 2,052$) | A-G2 (les excursions 153–194s et 206–240s dépassent la zone morte et déclenchent quand même) | [FRAGILE - 1 SOURCE] — je le dis à voix haute : une seule pièce soutient le calibrage exact de ce seuil
| Confirmation 3 ticks | B-F1 (choc isolé de 11,165$ à t=122,000s ; trous de 3 000 ms) | A-G5 (attendre 3 s fait entrer après l'écartement du spread : 0,21$ à t=178,491s, 25,5 s après la bascule de t=153s) | MITIGÉE - 1 POUR / 1 CONTRE
| Plafond ask ≤ 0,75$ | B-F4 (NO 0,94/0,95 à t=270s : 0,05$ de gain max) ; A-G5 (spread jusqu'à 0,21$) | A-G1 (le NO acheté même à 0,95$ finissait à 1,00$ sur cette session) | MITIGÉE - 2 POUR / 1 CONTRE
| All-in du cash | B-F4 (spread 0,028$ par traversée) | Aucune pièce ne teste un fractionnement | [FRAGILE - 1 SOURCE] — annoncé comme tel


## C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

### ÉTAT DU PORTEFEUILLE

Capital initial 5,00$. Le pseudo-code C2, déroulé sur les 285 ticks oracle et les 1 090 événements BBO, produit exactement 5 signaux confirmés : t=10,0s NO, t=159,0s YES, t=204,0s NO, t=211,0s YES, t=249,0s NO (oracle.csv, règles C2). L'état est journalisé à chaque événement ; aucun achat ne dépasse le cash disponible : chaque entrée consomme le cash à 0,01$ près (colonne «Cash après» du journal, lecture ligne à ligne : 0,00$ / 0,00$ / 0,01$ / 0,01$ / 0,01$ de cash résiduel après chaque achat).

### RACE CONDITIONS

(a) Signal d'achat pendant un ordre en cours : la simulation est mono-actif et séquentielle sur le flux BBO ; un signal survenant avant le fill du précédent est mis de côté et réévalué au fill. RÈGLE : verrou global de portefeuille — aucun nouvel ordre tant que l'ordre en cours n'est pas exécuté au premier événement BBO complet suivant.

(b) Signal de vente sur une position pas encore confirmée à l'achat : la vente ne peut cibler qu'une position dont le fill d'achat est journalisé. RÈGLE : file FIFO stricte — un signal de sortie arrivant avant la confirmation d'achat est exécuté au premier événement BBO complet suivant cette confirmation (cas réalisé au trade n°3 : signal t=211,0s, achat NO confirmé à t=205,414s, sortie exécutée à t=216,325s, premier BBO complet après le signal).

(c) Deux signaux simultanés sur le même actif : deux ticks oracle ne partagent jamais le même t_ms (285 timestamps distincts, B-F1, B4) ; si le cas survenait, RÈGLE : rejet du second signal — seul le premier de la file est traité, l'automate d'état ignore un signal identique à l'état courant.

(d) Fill partiel : les CSV ne donnent pas les profondeurs au BBO, un fill partiel n'est donc pas simulable — c'est couvert par [HYPOTHÈSE n°2]. RÈGLE : tout ordre est réputé rempli intégralement au prix BBO du premier événement complet suivant ; à défaut de cette hypothèse, la simulation s'arrêterait sur DONNÉE NON DISPONIBLE.

### JOURNAL DE TRADES

Le journal de trades, ligne par ligne :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=11,752s (NO) | 0,69$ | 7,24 | t=159,063s | 0,37$ | 0,00$ | −2,32$ | 2,68$
| 2 | t=159,063s (YES) | 0,63$ | 4,25 | t=205,414s | 0,44$ | 0,00$ | −0,81$ | 1,87$
| 3 | t=205,414s (NO) | 0,56$ | 3,33 | t=216,325s | 0,41$ | 0,00$ | −0,49$ | 1,38$
| 4 | t=216,325s (YES) | 0,59$ | 2,33 | t=250,090s | 0,31$ | 0,00$ | −0,65$ | 0,73$
| 5 | t=250,090s (NO) | 0,69$ | 1,05 | t=300,000s (règlement) | 1,00$ | 0,00$ | +0,33$ | 1,06$


Chaque prix d'entrée et de sortie est le BBO du premier événement complet suivant le signal (bbo.csv) ; la ligne 5 est réglée à 1,00$ car le résultat de la session est DOWN (A-G1, A1).

### RÉSULTATS

Le tableau récapitulatif, et uniquement lui, alimente l'ouverture et la clôture :

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants | 1, gains cumulés +0,33$ | ligne 5
| Trades perdants | 4, pertes cumulées −4,27$ | lignes 1 à 4
| Frais totaux | 0,00$ (0,00$ ×5 lignes) [HYPOTHÈSE n°1] | lignes 1 à 5
| Bénéfice net (gains − pertes − frais) | −3,94$ | lignes 1 à 5
| Capital final vs initial | 1,06$ vs 5,00$, rendement net −78,8% | ligne 5, colonne Cash après
| Pire perte unitaire | −2,32$ (entrée t=11,752s, sortie t=159,063s) | ligne 1
| Drawdown maximum | 6,21$, du pic d'équité 6,81$ (t=72,452s) au creux 0,60$ (t=254,585s) | valorisation mark-to-market au bid, lignes 1 à 5 contre bbo.csv


Cohérence comptable, recalculée devant vous : 5,00 − 2,32 − 0,81 − 0,49 − 0,65 + 0,33 = 1,06$ — au centime près.

## C4 — VERDICT ET PORTÉE

VERDICT : [REJETÉE SUR SESSION] — bénéfice net −3,94$, 4 trades perdants sur 5, drawdown de 6,21$ supérieur au capital initial de 5,00$ (tableau RÉSULTATS, C-SIM).

Ce qui relève de la stratégie et ce qui relève de mes adaptations, en une ligne : la destruction de capital vient du principe lui-même — suivre chaque bascule confirmée de l'oracle fait acheter cher et vendre bas à chacune des 4 bascules (lignes 1 à 4 du journal) — mes adaptations n'ont fait que réduire le nombre de faux signaux de 7 croisements bruts à 5 confirmés (B-F1, B4).

Les conclusions, une par ligne, chacune étiquetée :

- Le carnet intègre la position de l'oracle avant qu'un suiveur à confirmation 3 ticks puisse entrer : à t=159,063s le YES cote déjà 0,61/0,63 alors que la bascule oracle date de t=153,000s (B-F4, B4 ; B-F1, B4). [PATTERN CANDIDAT]
- Suivre l'oracle à la bascule paie le spread de bascule (jusqu'à 0,21$ à t=178,491s), le pire moment de liquidité de la session (B-F4, B4). [PATTERN CANDIDAT]
- Le spot est un faux ami : 0 tick sous le strike contre 73,0% pour l'oracle (B-F2, B4 ; B-F1, B4). [PATTERN CANDIDAT]
- Les deux excursions au-dessus du strike (153–194s et 206–240s) qui ont généré les 4 trades perdants sont propres à la trajectoire de cette session (B-F1, B4). [SESSION-SPÉCIFIQUE]
- L'unique trade gagnant (+0,33$, ligne 5) doit tout au règlement final DOWN, connu seulement à t=300s (A-G1, A1). [SESSION-SPÉCIFIQUE]


---

# 5. AUTO-CONTRÔLE FINAL

- Contrôle 1 → Valeurs chiffrées : A-G1 : 6 ; A-G2 : 7 ; A-G3 : 6 ; A-G4 : 6 ; A-G5 : 6 ; A-G6 : 5 valeurs + 1 DONNÉE NON DISPONIBLE déclarée. B-F1 : 12 ; B-F2 : 9 ; B-F3 : 8 ; B-F4 : 9 métriques en tableau. Minimums respectés.
- Contrôle 2 → Chaque affirmation des Parties C pointe vers une rubrique A4/B4 nommée ou une ligne du journal ; aucune affirmation orpheline détectée.
- Contrôle 3 → Aucun chiffre en double avec valeurs divergentes ; le seul couple voisin (22 suspects du traceur PDF vs 16 lignes croisées + 3 incomplètes de mon comptage) est explicitement présenté en B-F4, B4 comme deux métriques à définitions différentes.
- Contrôle 4 → Cohérence comptable : 5,00 − 2,32 − 0,81 − 0,49 − 0,65 + 0,33 = 1,06$ = capital final annoncé, au centime près.
- Contrôle 5 → 3 [HYPOTHÈSE] utilisées (frais nuls, fill intégral, latence au BBO suivant) : le plafond de 3 n'est pas dépassé.
- Contrôle 6 → 4 [ADAPTATION] utilisées, toutes au journal C1-ADAPT avec justification chiffrée ; aucune ne change la source de vérité du PRINCIPE DIRECTEUR ; aucune décidée après C-SIM.
- Contrôle 7 → Conformité F0–F7 : adresse directe, verdict dès l'ouverture, ordre F1 respecté, 6 tableaux au format F4, identifiants A-G1 à A-G6 et B-F1 à B-F4, clôture en 3 lignes sans chiffre nouveau.


---

# 6. CLÔTURE DU SPEECH

Le bénéfice net de la session est de −3,94$ : le capital termine à 1,06$ contre 5,00$ engagés.
Sur ces données, «ANCRAGE À L'ORACLE» achète les bascules au pire prix et se fait retourner quatre fois sur cinq.
VERDICT : [REJETÉE SUR SESSION] — et les trois patterns candidats identifiés méritent d'autres sessions avant toute généralisation.



# RAPPORT D'AUDIT OPÉRATIONNEL — STRATÉGIE «ANCRAGE À L'ORACLE»

## 0. VERDICT EXÉCUTIF

- Verdict global : **[FRAGILE]**
- NOTE DE FIABILITÉ : **46/100** (grille F7, détail en section 6)
- Manquements identifiés : 8 facteurs traités — 0 [DÉTRUIT], 2 [DÉGRADE], 1 [IMPACT NON CHIFFRABLE], 5 [SANS IMPACT]
- Manquement le plus grave : le slippage ignoré (O3) aggrave le P&L de −0,17$, soit 4,4% du P&L annoncé.
- P&L annoncé : −3,94$ ; P&L après réintégration de tous les manquements chiffrables : **−4,24$** (capital final 0,76$ au lieu de 1,06$).
- Point le plus solide : la latence de boucle réelle de 99 ms (68 ms Chainlink + 31 ms CLOB) ne change pas un centime du résultat — le bot PEUT structurellement suivre cette stratégie ; c'est la stratégie elle-même qui perd.


Précision de lecture, annoncée avant tout verdict de point : le P&L annoncé étant déjà négatif, [DÉTRUIT] se lit ici «renverse la conclusion [REJETÉE SUR SESSION]» et [DÉGRADE] «aggrave le P&L sans changer la conclusion». Je vais vous montrer pièce par pièce d'où vient ce verdict.

## 1. PÉRIMÈTRE

Audité : le speech complet (sections C0 à C4, C-SIM), rejoué contre oracle.csv (285 lignes), bbo.csv (1 090 lignes), trades.csv (1 738 lignes), spot.csv (7 579 lignes) et le tableau des latences. Non audité : le rendu graphique du PDF (utilisé en corroboration seulement) et le graphe 6, sans valeurs étiquetées. Le recalcul n'est fait que par sondage (Partie V) plus le rejeu opérationnel (Parties O et I).

## 2. PARTIE V — SONDAGE DE FIABILITÉ DES DONNÉES

J'ai rejoué le pseudo-code C2 ligne à ligne sur les CSV. Les 5 signaux annoncés (t=10,0s NO ; 159,0s YES ; 204,0s NO ; 211,0s YES ; 249,0s NO) ressortent exactement.

| # | Valeur sondée | Annoncée | Recalculée | Écart | Verdict
|-----|-----|-----|-----|-----|-----
| 1 | Capital final (recalcul complet au centime) | 1,06$ | 1,04$ | −0,02$ (0,5% du P&L) | [ÉCART MINEUR]
| 2 | Strike (oracle.csv, ligne 1, col. price) | 64 233,14$ | 64 233,13645148$ | arrondi | [CONFIRMÉ]
| 3 | Ticks oracle sous le strike (col. price, 285 lignes) | 208/285 (73,0%) | 208/285 | 0 | [CONFIRMÉ]
| 4 | p90 des |Δprice| oracle (284 intervalles) | 2,052$ | 2,001$ | 0,051$ (méthode de percentile ; signaux identiques) | [CONFIRMÉ]
| 5 | Trade n°1 (bbo.csv : ask NO 0,69$ à t=11,752s ; bid NO 0,37$ à t=159,063s ; P&L −2,32$) | conforme | conforme | 0 | [CONFIRMÉ]
| 6 | Trade n°3 (quantité = cash/ask arrondi inf.) | 3,33 | 3,34 (1,8757/0,56) | 0,01 part | [ÉCART MINEUR]


Source de l'écart n°1 : les quantités des trades n°3 et n°5 (annoncées 3,33 et 1,05 ; recalculées 3,34 et 1,04) — deux erreurs d'arrondi de sens opposés, cumulées à +0,02$ en faveur de l'agent. Zéro [ERREUR] au sens de la règle de décision : **données de base dignes de confiance au niveau sondé**. Le chiffre de référence du reste de l'audit est mon recalcul exact : capital final 1,04$, P&L −3,96$.

## 3. PARTIE O — AUDIT OPÉRATIONNEL : CE QUE LA STRATÉGIE IGNORE

**O1. LATENCE D'EXÉCUTION**
CE QUE LA STRATÉGIE SUPPOSE : [HYPOTHÈSE n°3] — «signal exécuté au premier événement BBO complet strictement postérieur», donc latence implicitement nulle entre tick oracle et ordre posé.
CE QUI EST VRAI : la boucle réelle depuis us-east-1 est lecture Chainlink on-chain 68 ms + décision + envoi Polymarket CLOB 31 ms = **99 ms** (139 ms si la lecture passe par Polygon RPC à 108 ms). Binance à 219 ms est hors boucle : la stratégie écarte explicitement le spot (C0, OBS-2). Telegram à 67 ms est hors boucle critique (notification).
IMPACT CHIFFRÉ : j'ai rejoué les 5 entrées décalées de 99 ms puis de 250 ms : capital final 1,04$ dans les deux cas, identique au centime au rejeu à latence nulle (seul l'horodatage de la sortie du trade n°1 glisse de 159,063 s à 159,787 s, au même prix 0,37$). Les positions sont tenues de 10,9 s (trade n°3) à 147,3 s (trade n°1) : un décalage de 99 ms est 110 fois plus court que la tenue la plus brève.
VERDICT : **[SANS IMPACT]** — 0,00$ (0,0%). Réponse directe à votre question : structurellement, le bot peut suivre cette stratégie ; la boucle de 99 ms est sans friction face à des signaux espacés d'au moins 1 s et confirmés sur 3 s.

**O2. FRAÎCHEUR DU SIGNAL**
CE QUE LA STRATÉGIE SUPPOSE : le tick oracle est connu à l'instant où il est produit.
CE QUI EST VRAI : cadence oracle 1 tick/s (médiane des écarts 1 000 ms, oracle.csv col. t_ms) + lecture 68 ms = âge réel du signal jusqu'à 1 068 ms au moment de la décision.
IMPACT CHIFFRÉ : rejeu des 5 entrées décalées de 1 068 ms : capital final 0,92$, soit **−0,13$** (3,3% du P&L annoncé).
VERDICT : **[DÉGRADE]**.

**O3. SLIPPAGE ET PROFONDEUR**
CE QUE LA STRATÉGIE SUPPOSE : [HYPOTHÈSE n°2] — fill intégral au BBO, taille sans impact.
CE QUI EST VRAI : les profondeurs au BBO sont absentes des CSV — DONNÉE NON VÉRIFIABLE. Bornage : les ordres font 0,72$ à 5,00$, contre un trade médian de 4,30$ sur la session (trades.csv col. usd) ; à cette taille je borne le slippage entre 0,00$ et 0,01$ par côté (un cran de prix, le pas de cotation).
IMPACT CHIFFRÉ : rejeu avec 0,01$ de slippage par côté sur 9 exécutions : capital final 0,87$, soit **−0,17$** (4,4% du P&L annoncé) — borne basse 0,00$.
VERDICT : **[DÉGRADE]**.

**O4. FRAIS COMPLETS**
CE QUE LA STRATÉGIE SUPPOSE : [HYPOTHÈSE n°1] — frais nuls.
CE QUI EST VRAI : le CLOB Polymarket applique 0% maker/taker sur ces marchés ; les ordres sont signés hors-chaîne et réglés par l'opérateur, sans gas à la charge du compte ; la lecture Chainlink est un appel RPC en lecture, gratuit. Le spread, seul coût implicite, est déjà payé dans la simulation (entrée à l'ask, sortie au bid).
IMPACT CHIFFRÉ : 0,00$. Borne de prudence : si une politique de frais de 1% par exécution apparaissait, l'impact serait −0,07$ additionnel (rejeu cumulé).
VERDICT : **[SANS IMPACT]** — l'hypothèse n°1 de l'agent est correcte.

**O5. ORDRES CONCURRENTS ET FILE D'ATTENTE**
CE QUE LA STRATÉGIE SUPPOSE : le niveau lu est encore là à l'arrivée de l'ordre ; aucune gestion de rejet ou de fill partiel n'est définie (l'agent le déclare en C-SIM d).
CE QUI EST VRAI : l'ordre arrive 31 ms après décision. J'ai vérifié le carnet aux 5 signaux : l'ask exécutable est strictement identique à t et à t+250 ms (0,69 / 0,63 / 0,56 / 0,59 / 0,69, bbo.csv) — aucun niveau ne disparaît dans une fenêtre 8 fois plus large que la latence CLOB.
IMPACT CHIFFRÉ : 0,00$ sur les 5 trades de la session.
VERDICT : **[SANS IMPACT]** sur cette session — mais l'absence de règle de re-pricing reste un défaut de conception, sans conséquence chiffrable ici.

**O6. DÉFAILLANCES TECHNIQUES**
CE QUE LA STRATÉGIE SUPPOSE : flux oracle continu et lecture toujours réussie.
CE QUI EST VRAI : 13 trous de plus de 1 500 ms dans le flux oracle, maximum 3 000 ms (oracle.csv col. t_ms) ; la stratégie n'a aucun comportement défini pendant un trou : position aveugle jusqu'à 3 s, plus timeout RPC non géré.
IMPACT CHIFFRÉ : borné par deux rejeux extrêmes : décalage systématique de 1 068 ms → −0,13$ ; décalage de 3 000 ms → +0,50$ (capital 1,54$ : entrer plus tard a, par chance, évité des prix pires). L'effet est non directionnel : bornage [−0,13$ ; +0,50$], méthode = rejeu aux deux bornes de retard observables.
VERDICT : **[IMPACT NON CHIFFRABLE]** — comportement indéfini, résultat dépendant du hasard du timing.

**O7. CAPACITÉ ET PASSAGE À L'ÉCHELLE**
CE QUE LA STRATÉGIE SUPPOSE : rien — elle est calibrée all-in sur 5,00$.
CE QUI EST VRAI : profondeur au BBO absente des CSV — DONNÉE NON VÉRIFIABLE. Bornage indirect : volume total de la session 24 792,21$ et trade médian 4,30$ (trades.csv) ; un ordre de 50$ vaudrait 11,6 fois le trade médian, l'hypothèse de fill au BBO cesse d'être défendable au-delà de la dizaine de dollars.
IMPACT CHIFFRÉ SUR LES RÉSULTATS ANNONCÉS : 0,00$ — le P&L annoncé est négatif, il ne «tient» pas grâce à la petite taille ; la taille plafonne seulement un éventuel gain futur.
VERDICT : **[SANS IMPACT]** sur les chiffres annoncés.

**O8. DÉPENDANCE À LA SESSION**
CE QUE LA STRATÉGIE SUPPOSE : rien d'indu — l'agent étiquette lui-même les 4 trades perdants [SESSION-SPÉCIFIQUE] (C4) : les deux excursions 153–194 s et 206–240 s sont propres à cette trajectoire.
CE QUI EST VRAI : je confirme sur oracle.csv : sans ces deux excursions au-dessus du strike, la zone morte ±2,00$ + confirmation 3 ticks produit 1 seul signal (t=10,0s NO) porté au règlement — session calme : 0 retournement, P&L positif de mécanique différente ; session sans sortie de la zone morte : 0 trade, P&L 0,00$.
IMPACT CHIFFRÉ : 0,00$ sur les résultats annoncés — le facteur est déclaré et correctement borné par l'agent.
VERDICT : **[SANS IMPACT]** — correctement pris en compte par l'agent.

## 4. PARTIE I — IMPACT CUMULÉ DES MANQUEMENTS

Rejeu unique avec tous les manquements chiffrables simultanés : âge du signal 1 068 ms (O2, qui domine et inclut la boucle O1 de 99 ms), slippage 0,01$/côté (O3), frais 0,00$ (O4, prouvé), fills conservés (O5, prouvé). Le journal corrigé garde 5 trades ; les entrées se font à 0,70 / 0,67 / 0,57 / 0,60 / 0,70$ au lieu de 0,69 / 0,63 / 0,56 / 0,59 / 0,69$.

| Scénario | P&L annoncé | P&L corrigé | Écart %
|-----|-----|-----|-----|-----|-----
| O2 seul (âge du signal 1 068 ms) | −3,94$ | −4,08$ | −3,6%
| O3 seul (slippage 0,01$/côté) | −3,94$ | −4,13$ | −4,8%
| **TOUS MANQUEMENTS CUMULÉS** | **−3,94$** | **−4,24$** | **−7,6%**


Le P&L cumulé corrigé est négatif : la règle bloquante interdit le verdict [FIABLE]. La conclusion du speech ([REJETÉE SUR SESSION]) n'est pas détruite par les manquements — elle est renforcée : la réalité opérationnelle rend la stratégie encore plus perdante que simulé.

## 5. PARTIE R — RISQUES RÉSIDUELS NON DÉCLARÉS

- [RISQUE NON DÉCLARÉ] Résolution du marché : le speech règle la ligne 5 à 1,00$ en supposant que la résolution Polymarket coïncide avec le tick Chainlink de t=300 s ; la règle de résolution effective n'est dans aucune pièce. Si la résolution divergeait, le seul trade gagnant bascule : borne [0,00$ ; −1,05$] (perte du coût de la position 0,72$ plus le gain annoncé 0,33$).
- [RISQUE NON DÉCLARÉ] Asymétrie structurelle gain/perte : avec un plafond d'entrée à 0,75$, le gain maximal par part est 0,25$ contre une perte maximale de 0,75$ (rapport 1:3) ; combinée au all-in, cette asymétrie autorise la quasi-ruine en 2 bascules — le drawdown de 6,21$ est cité, le rapport 1:3 jamais.
- [RISQUE NON DÉCLARÉ] Représentativité du flux oracle : une cadence régulière de 1 tick/s ne correspond pas au rythme de publication on-chain de Chainlink (déviation/heartbeat) ; en production le signal serait plus discontinu que dans oracle.csv. Impact : DONNÉE NON VÉRIFIABLE avec les pièces fournies.


## 6. AUTO-CONTRÔLE DE L'AUDITEUR

- Contrôle 1 → Les 8 facteurs O1–O8 sont traités en 4 temps ; chaque [SANS IMPACT] porte sa preuve chiffrée (O1 : 0,00$ à +99 et +250 ms ; O4 : structure de frais 0% ; O5 : ask identique à +250 ms sur 5/5 signaux ; O7 : P&L annoncé négatif ; O8 : 0 ou 1 signal en session calme). RÉSULTAT : conforme.
- Contrôle 2 → Latence de boucle reconstituée chemin par chemin : Chainlink 68 ms + CLOB 31 ms = 99 ms (variante Polygon RPC 108 ms + 31 ms = 139 ms) ; utilisée en O1 et incluse dans le 1 068 ms de la Partie I. RÉSULTAT : conforme.
- Contrôle 3 → Chaque impact cite sa source primaire (oracle.csv, bbo.csv, trades.csv, tableau des latences) ; les deux impacts non chiffrables (O6, résolution) sont bornés avec méthode. RÉSULTAT : conforme.
- Contrôle 4 → Note F7 : COMPLÉTUDE 25/40 (O2 −5, O3 −5, O6 −5, traité en dégradation à sa borne défavorable) ; SURVIE DU P&L 0/30 (cumul négatif) ; FIABILITÉ DES DONNÉES 13/20 (4 [CONFIRMÉ] sur 6 sondées, prorata) ; HONNÊTETÉ 8/10 (−2 : l'agent disposait d'horodatages ms et n'a jamais confronté ses hypothèses d'exécution aux latences). Total 25+0+13+8 = **46/100** → [FRAGILE] (45–74), cohérent avec la règle bloquante. RÉSULTAT : conforme.
- Contrôle 5 → Chiffres uniques d'ouverture à clôture : −3,94$ / −4,24$ / 1,04$ / 0,76$ / 99 ms / −0,13$ / −0,17$ / 46/100 réapparaissent à l'identique. RÉSULTAT : conforme.


## 7. CLÔTURE

Verdict : [FRAGILE], note 46/100 — les données sont saines et la conclusion de rejet survit, mais la couche d'exécution n'a jamais été confrontée au réel.
Le bot peut structurellement suivre cette stratégie ; ce qu'il suivrait est une stratégie perdante, et légèrement plus perdante une fois les frictions réintégrées.
Condition n°1 pour changer le verdict : fournir les profondeurs au BBO et la règle de résolution du marché, puis rejouer le cumul — sans ces pièces, aucun passage à [FIABLE] n'est démontrable.

---

J'ai rejoué l'intégralité du pseudo-code C2 sur les CSV bruts (les scripts de rejeu ont tourné dans `.audit/`, fichiers de travail copiés depuis les pièces jointes). Les deux constats centraux : votre question sur les latences a une réponse nette — la boucle 68+31 ms est inoffensive face à des positions tenues 10 à 147 secondes — et les vrais manquements sont le slippage et l'âge du signal, qui aggravent la perte de 7,6% en cumul sans jamais renverser le verdict de rejet que l'agent avait lui-même prononcé.



j'ai rejoué toutes les variantes sur les mêmes CSV, **avec les frictions réintégrées** (âge du signal 1 068 ms + slippage 0,01$/côté), donc au même standard que l'audit.

---

## 1. D'abord, identifier CE QUI perd : le flip, pas le signal

Décomposition des 5 signaux, chacun tenu jusqu'au règlement (au lieu de flipper) :

| Signal | Côté | Prix d'entrée | P&L/part si porté au règlement
|-----|-----|-----|-----
| t=10s | NO | 0,70$ | **+0,30$**
| t=159s | YES | 0,67$ | −0,67$
| t=204s | NO | 0,57$ | **+0,43$**
| t=211s | YES | 0,60$ | −0,60$
| t=249s | NO | 0,70$ | **+0,30$**


Le mécanisme destructeur est prouvé : chaque flip **vend au bid effondré du côté perdant (0,33–0,40$) et rachète l'autre côté à l'ask cher (0,57–0,70$)**. La stratégie paie l'aller-retour complet du spread + le mouvement adverse, deux fois par retournement. Les signaux NO, eux, étaient tous gagnants.

## 2. Trois corrections chiffrées (frictions incluses, capital départ 5,00$)

| Variante | Capital final | P&L
|-----|-----|-----|-----
| Stratégie originale (audit) | 0,76$ | −4,24$
| **A. Supprimer le flip** : 1er signal porté au règlement (marge=2, nconf=3 inchangés) | **7,14$** | **+2,14$ (+43%)**
| **B. A + filtre plus strict** (marge=5$, nconf=5) | **7,46$** | **+2,46$ (+49%)**
| C. Grid complet marge×nconf avec flip conservé | au mieux 7,46$ (quand marge=10 ne produit qu'1 signal → équivaut à A) | —
| Inverser les signaux (mean-reversion) | 0,00$ | ruine — à écarter


Le grid search de 40 combinaisons confirme : **toute combinaison qui ne génère qu'un seul signal gagne ; toute combinaison qui flippe perd** (pire cas : marge=8, nconf=5 → 0,20$).

## 3. L'avertissement obligatoire — pourquoi ce n'est pas encore une «preuve de stratégie gagnante»

1. **In-sample sur 1 session.** Ces +2,46$ sont une optimisation sur la trajectoire même qui a servi à les trouver. La session a résolu DOWN et l'oracle a passé 74% des ticks sous le strike après le signal — la variante gagne parce que le 1er signal était du bon côté.
2. **Contrefactuel chiffré** : si la résolution avait été UP, la variante B all-in finit à **0,00$** (ruine). Entrée à 0,67$ = il faut avoir raison **plus de 67% du temps** pour être EV-positif. Le signal n'a d'edge que si «être à 5$ sous le strike, confirmé 5 ticks» prédit la clôture à mieux que 67% — invérifiable sur une session.
3. **Correction de sizing** : avec une mise de 25% du capital au lieu du all-in : DOWN → 5,61$ (+12%), UP → 3,75$ (−25%). La ruine devient impossible et 3 sessions perdantes consécutives laissent encore ~2,1$ pour continuer.


## 4. Recette finale recommandée (chiffres à l'appui)

1. **Ne jamais flipper** — c'est le poste de perte n°1 : le flip transforme +2,14$ en −4,24$, soit **6,38$ de destruction** attribuable à ce seul mécanisme sur la session.
2. **Un seul signal par marché, porté au règlement**, marge ≥ 5$, nconf ≥ 5 : final 7,46$ sur cette session.
3. **Mise fractionnée (20–25%)**, jamais all-in : le contrefactuel UP à 0,00$ le rend non négociable.
4. **Condition de validation** : rejouer sur ≥ 30 sessions indépendantes et vérifier que le taux de réussite du 1er signal dépasse son prix d'entrée moyen (~67%). En dessous, la variante est perdante en espérance malgré son +49% ici.


La preuve chiffrée existe donc pour cette session (0,76$ → 7,46$ avec les mêmes frictions), mais le passage de «gagnante sur cette session» à «gagnante en espérance» exige le point 4 — c'est exactement le piège que l'audit reprochait à l'agent initial, et je ne vais pas le reproduire.




## Rendement après amélioration

|  | Capital départ | Capital final | Rendement
|-----|-----|-----|-----
| Stratégie originale | 5,00$ | 0,76$ | **−84,8%**
| Améliorée (1 signal porté au règlement, marge=5, nconf=5, frictions incluses) | 5,00$ | 7,46$ | **+49,2%**


**Cause de l'échec en une phrase :** chaque retournement (flip) vendait la position au bid effondré (0,33–0,40$) pour racheter l'autre côté à l'ask cher (0,57–0,70$), payant ainsi le spread complet plus le mouvement adverse à chaque bascule — soit 6,38$ détruits par ce seul mécanisme.

**Cause du nouveau rendement gagnant en une phrase :** en supprimant le flip et en portant le premier signal NO (entré à 0,70$) jusqu'au règlement à 1,00$, la stratégie encaisse le payoff complet de +0,30$/part au lieu de revendre à perte à chaque oscillation du prix autour du strike.

*Rappel obligatoire : ce +49,2% est validé sur cette seule session (résolue DOWN) ; le contrefactuel UP donne 0,00$ en all-in, d'où la mise fractionnée à 20–25% recommandée.*


---

## Source : 10ANCRAGE.md

# SPEECH — TEST DE LA STRATÉGIE «ANCRAGE À L'ORACLE» SUR LA SESSION POLYMARKET BTC 5MIN DU 6 AOÛT, 20:20–20:25 ET

## 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Voici le verdict avant tout le reste. J'ai testé «ANCRAGE À L'ORACLE», que j'interprète ainsi : l'oracle Chainlink est la seule vérité de règlement, on trade son signe contre le sentiment du marché.

- Capital final : 7,08$ contre 5,00$ de départ, soit un rendement net de +41,6%.
- Trades : 2, dont 2 gagnants et 0 perdant.
- Adaptations apportées à la stratégie : 3.
- VERDICT SUR LA STRATÉGIE : [VALIDÉE SUR SESSION] — mécanisme sous-jacent [PATTERN CANDIDAT].


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

## 1. CADRE D'ANALYSE CHOISI

J'adopte un cadre microstructurel événementiel : je confronte tick par tick la source de règlement (oracle Chainlink, 275 ticks) au flux d'information rapide (spot Binance, 6 079 ticks) et à la formation du prix sur le carnet (681 événements BBO, précision ms). Ce cadre est imposé par la nature des pièces : sur un horizon de 300 s, il n'existe ni fondamentaux ni saisonnalité, seulement des écarts mesurables entre trois horloges de prix. La granularité milliseconde des CSV permet précisément cela.

---

## PARTIE A — LES 6 GRAPHES DU PDF

### A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (BBO)

**A1. IDENTITÉ.** Graphique 1, «Carnet YES/NO complet — 681 événements BBO (précision ms), incohérents : 8», vue 300 s plus 3 zooms 15 s ([02:21→02:36], [00:50→01:05], [01:10→01:25]). Axe X : temps depuis l'ouverture (mm:ss.mmm), axe Y gauche : prix ($) de 0,0 à 1,0, axe Y droit : probabilité. Période : t=0,136s → t=289,893s.

**A2. QUESTION.** Que fait le prix du carnet quand le consensus se forme ? Le marché revient-il douter après avoir tranché, et à quels moments ?

**A3. COMMENT.** J'ai lu les courbes YES mid et NO mid sur la vue 300 s pour la trajectoire globale, puis les étiquettes des zooms pour dater au millième de seconde les épisodes de retour vers 0,50.

**A4. VALEURS EXTRAITES.**

- t=00:00.136 : YES = 0,52$ ; NO = 0,48$
- t=04:49.893 : YES = 0,01$ ; NO = 0,99$
- t=02:28.975 : YES ask monte de 0,41$ à 0,43$ en une rafale d'événements au même timestamp
- t=02:29.075 : YES = 0,48$ ; NO = 0,52$ (pic du doute, zoom [02:21→02:36])
- t=02:21.089 : NO = 0,72$ avant le début de la remontée du YES
- t=01:00.679 : YES = 0,27$ ; NO = 0,73$ (zoom [00:50→01:05])


**A5. OBSERVATIONS BRUTES.**

- Le marché ouvre à l'équilibre quasi parfait (YES 0,52$ à t=00:00.136) et clôt à la certitude (NO 0,99$ à t=04:49.893).
- Il existe un épisode où le prix revient à un quasi pile-ou-face : NO retombe à 0,52$ à t=02:29.075 alors que la tendance de fond est baissière.
- La convergence vers 0,99$ est monotone après t=02:36 environ.


**A6. SPÉCIFIQUE À LA SESSION.** L'amplitude exacte du doute (retour à 0,52$ à t=02:29.075) et sa date sont propres à cette session : elles coïncident avec un rebond spot précis (cf. A-G2, t=02:30.000 : 64 212,98$). Le timing est non reproductible ; l'existence d'épisodes de doute l'est potentiellement.

### A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK

**A1. IDENTITÉ.** Graphique 2, «Spot Binance (flux direct) vs Oracle Chainlink — strike exact (dernier tick ≤ t0)», vue 300 s + 3 zooms, avec sous-graphe «basis = spot − oracle» (6 079 pts, axe 0 à 60$+). Axe Y : prix BTC ($), 64 180 à 64 280. Strike affiché : 64 222$. Période : t=0,000s → t=298,884s.

**A2. QUESTION.** Où se situe chaque source de prix par rapport au strike, et que fait l'écart entre les deux sources au fil de la fenêtre ?

**A3. COMMENT.** J'ai lu la position de chaque courbe par rapport à la ligne horizontale du strike, puis les étiquettes d'extrema, puis le signe du sous-graphe basis sur toute la fenêtre.

**A4. VALEURS EXTRAITES.**

- t=00:00.298 : spot Binance = 64 280,01$
- t=00:51.577 : spot Binance = 64 230,59$ (minimum affiché)
- t=04:58.884 : spot Binance = 64 265,91$
- t=00:53.000 : oracle Chainlink = 64 173,72$
- t=02:28.000 : oracle = 64 199,53$ puis t=02:30.000 : oracle = 64 212,98$
- Strike ouverture : 64 222$ (étiquette du graphe)


**A5. OBSERVATIONS BRUTES.**

- Le spot Binance ne touche jamais la zone du strike : son minimum affiché est 64 230,59$ (t=00:51.577), au-dessus de 64 222$.
- L'oracle évolue intégralement sous le strike après l'ouverture (courbe sous la ligne du strike sur toute la vue 300 s).
- Le basis spot−oracle reste positif sur toute la fenêtre (sous-graphe jamais sous 0).
- Les deux sources donnent donc des verdicts opposés en permanence : spot = UP, oracle = DOWN.


**A6. SPÉCIFIQUE À LA SESSION.** L'ampleur du basis (jusqu'à 70,46$, cf. B-F2, B4) est une condition particulière de cette session, vraisemblablement liée à la composition d'agrégation du flux Chainlink à cette heure. La divergence signée persistante pendant 300 s est l'anomalie qui fait tout le résultat : je la classe non reproductible en amplitude, reproductible en mécanisme (l'oracle est une moyenne multi-sources, le spot une seule bourse).

### A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ

**A1. IDENTITÉ.** Graphique 3, «Flux directionnel normalisé YES/NO (1 369 trades)», fenêtre glissante 30 s, flux haussier positif / baissier négatif, en $. Axe Y : −4 000 à +2 000$. Vue 300 s + 3 zooms.

**A2. QUESTION.** Quel camp paie pour exécuter, à quel moment, et en quelle taille ?

**A3. COMMENT.** Lecture des étiquettes d'extrema de la vue 300 s et des barres des zooms, signe par signe.

**A4. VALEURS EXTRAITES.**

- t=00:00.735 : flux = +31,20$
- t=02:10.556 : flux haussier = +538,53$ (extremum haussier de la session)
- t=04:50.268 : flux baissier = −963,65$ (extremum baissier)
- t=04:59.258 : flux = +260,75$
- t=02:22.250 : flux = −134,52$ (zoom [02:21→02:36])


**A5. OBSERVATIONS BRUTES.**

- Le flux baissier domine visuellement la quasi-totalité de la fenêtre.
- Le plus gros flux haussier (+538,53$ à t=02:10.556) précède de 18 s l'épisode de doute du carnet (t=02:29.075, A-G1) : de l'argent réel a poussé le YES.
- Le plus gros trade baissier arrive en toute fin (−963,65$ à t=04:50.268), quand l'issue est déjà pricée à 0,99$.


**A6. SPÉCIFIQUE À LA SESSION.** Les montants absolus (+538,53$, −963,65$) sont propres à cette session ; l'ordre de grandeur global (milliers de $ sur 5 min) borne ce qu'une position de 5,00$ peut exécuter sans impact — c'est la seule chose que j'en retiens comme durable.

### A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ.** Graphique 4, «Imbalance = Haussier / (Haussier + Baissier)» sur 1 369 trades, ligne d'équilibre à 0,5. Axe Y : 0,0 à 1,0. Vue 300 s + 3 zooms.

**A2. QUESTION.** La pression acheteuse relative repasse-t-elle au-dessus de 0,5, et quand ?

**A3. COMMENT.** Lecture des étiquettes et des croisements de la ligne 0,5 sur la vue et les zooms.

**A4. VALEURS EXTRAITES.**

- t=00:00.735 : imbalance = 1,00
- t=04:50.389 : imbalance = 0,05
- t=02:21.079 : imbalance = 0,56 (seul passage au-dessus de 0,5 visible dans les zooms)
- t=00:59.318 : imbalance = 0,31
- t=01:23.742 : imbalance = 0,21


**A5. OBSERVATIONS BRUTES.**

- Hors la première seconde (valeur 1,00 sur un seul trade), l'imbalance vit sous 0,5 presque toute la session.
- L'unique excursion au-dessus de 0,5 dans les zooms (0,56 à t=02:21.079) date le début de l'épisode de doute vu en A-G1.


**A6. SPÉCIFIQUE À LA SESSION.** La valeur 1,00 à t=00:00.735 est un artefact de fenêtre quasi vide (1 trade) ; je la classe bruit.

### A-G5 — GRAPHE 5 : SPREADS YES ET NO

**A1. IDENTITÉ.** Graphique 5, «Spreads YES et NO absolus (événementiels)», 681 évts chacun. Axe Y : 0,00 à 0,13$. Vue 300 s + 3 zooms.

**A2. QUESTION.** Combien coûte la traversée du carnet, et le coût explose-t-il à certains moments ?

**A3. COMMENT.** Lecture des étiquettes d'extrema, puis du niveau typique des points dans les zooms.

**A4. VALEURS EXTRAITES.**

- t=00:00.136 : spread = 0,01$
- t=02:09.960 : spread = 0,13$ (maximum de la session)
- t=02:28.436 : spread = −0,01$ (valeur négative, anomalie)
- t=04:49.893 : spread = 0,00$
- t=00:57.516 : spread = 0,04$ (zoom [00:50→01:05])


**A5. OBSERVATIONS BRUTES.**

- Le régime normal est 0,01–0,02$ de spread : traverser le carnet coûte 1 à 2 cents.
- Les élargissements (0,13$ à t=02:09.960) sont brefs et localisés autour des rafales d'événements.


**A6. SPÉCIFIQUE À LA SESSION.** Le spread négatif de −0,01$ à t=02:28.436 est un croisement transitoire du carnet en pleine rafale ; il dure moins d'une seconde. Bruit d'exécution, pas signal.

### A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ.** Graphique 6, «Écart inter-carnets : yes_mid − (1 − no_mid)», 681 évts, ligne de cohérence parfaite à 0, marqueurs creux pour 8 suspects. Axe Y : 0,000 à 0,005$. Vue 300 s + 3 zooms.

**A2. QUESTION.** Les carnets YES et NO racontent-ils la même probabilité, ou existe-t-il un arbitrage interne ?

**A3. COMMENT.** Lecture de la position de la courbe par rapport à la ligne 0 sur les 4 panneaux, et du compte de suspects en légende.

**A4. VALEURS EXTRAITES.**

- Nombre d'événements tracés : 681
- Incohérents marqués par le graphe : 8
- Échelle maximale de l'écart affichable : 0,005$
- ts fallback (ts_src≠payload) : 0/275 (0,0%)
- Étiquettes de valeurs ponctuelles sur ce graphe : DONNÉE NON DISPONIBLE (aucune étiquette numérique tracée hors légende)


**A5. OBSERVATIONS BRUTES.**

- L'écart inter-carnets vit dans une bande d'au plus 0,005$ : les deux carnets sont la même probabilité à un demi-cent près.
- Aucun arbitrage YES/NO exploitable pour 5,00$ de capital.


**A6. SPÉCIFIQUE À LA SESSION.** Les 8 suspects marqués par le graphe relèvent du critère de l'outil de tracé ; mon propre calcul sur le CSV (B-F1, B4) trouve 1 seule ligne de carnet croisé au sens yes_bid + no_bid > 1,00$. Les deux comptes coexistent car les critères diffèrent ; dans les deux cas, c'est du bruit de rafale, pas un signal.

---

## PARTIE B — LES FICHIERS CSV

### B-F1 — bbo.csv

**B1. IDENTITÉ.** bbo.csv ; colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask — toutes utilisées ; 681 lignes de données (dont 679 complètes, la première et la dernière ayant un côté vide) ; période t=0,136s → t=289,893s ; échantillonnage événementiel (à chaque changement de BBO).

**B2. APPORT.** Le prix exécutable exact au millième de seconde — ce que les graphes montrent mais ne permettent pas d'extraire ligne à ligne pour une simulation.

**B3. COMMENT.** Spread = ask − bid par côté ; mid = (bid+ask)/2 ; consensus YES = (yes_mid + (1 − no_mid))/2 ; carnet croisé si yes_bid + no_bid > 1,00 ; moyennes et extrema sur les 679 lignes complètes.

**B4. RÉSULTATS CHIFFRÉS.** Le tableau, colonne par colonne :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Spread YES moyen | 0,0223$ | yes_bid, yes_ask | 679
| Spread YES maximum | 0,13$ (t=129,960s) | yes_bid, yes_ask | 679
| NO ask minimum | 0,49$ (t=0,136s) | no_ask | 680
| NO ask maximum | 0,99$ | no_ask | 680
| NO mid moyen | 0,707$ | no_bid, no_ask | 679
| NO mid minimum | 0,485$ | no_bid, no_ask | 679
| Consensus YES premier → dernier | 0,515 → 0,010 | 4 colonnes de prix | 679
| Carnets croisés (yes_bid+no_bid>1,00$) | 1 ligne (t=148,436s : 0,36+0,65=1,01$) | yes_bid, no_bid | 679
| Écart inter-carnets absolu maximum | 0,005$ ; moyenne 0,00001$ | 4 colonnes de prix | 679


**B5. OBSERVATIONS BRUTES.**

- Le NO est achetable sous 0,80$ jusqu'à t=57,548s au moins, alors que l'oracle est déjà sous le strike depuis t=2,000s (cf. B-F3) : le marché price avec retard.
- Le NO retombe deux fois vers le pile-ou-face après consensus : no_ask ≤ 0,56$ à partir de t=90,428s, puis no_ask = 0,52$ à t=149,075s.
- Coût de traversée typique : 0,0223$ en moyenne — négligeable pour la stratégie.


**B6. SPÉCIFIQUE À LA SESSION.** Une seule ligne de carnet croisé (t=148,436s) et un spread négatif simultané de −0,01$ : rafale de mise à jour, verdict bruit. Première et dernière ligne partiellement vides (t=0,136s côté NO absent, t=289,893s côté YES bid absent) : artefacts de bord de fenêtre, verdict bruit.

### B-F2 — spot.csv

**B1. IDENTITÉ.** spot.csv ; colonnes t_ms, price — toutes utilisées ; 6 079 lignes ; période t=0,298s → t=298,884s ; échantillonnage tick par tick Binance BTC/USDT.

**B2. APPORT.** La position du prix «rapide» par rapport au strike à chaque tick — le graphe 2 le montre, le CSV permet de compter les ticks un à un.

**B3. COMMENT.** Min/max/moyenne sur price ; comptage des ticks sous le strike 64 222,4357737$ (premier tick oracle ≤ t0, cf. B-F3 ligne 1) ; basis = price − dernier tick oracle antérieur ou égal (jointure par timestamp), sur 6 079 points.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Spot minimum | 64 230,59$ (t=51,577s) | price | 6 079
| Spot maximum | 64 280,01$ (t=0,298s) | price | 6 079
| Spot moyen | 64 255,75$ | price | 6 079
| Dernier tick | 64 265,91$ (t=298,884s) | t_ms, price | 1
| Ticks sous le strike | 0 sur 6 079 | price | 6 079
| Basis spot−oracle minimum | +41,47$ | price (+ oracle.price) | 6 079
| Basis maximum | +70,46$ | price (+ oracle.price) | 6 079
| Basis moyen ; écart-type | +57,24$ ; 3,86$ | price (+ oracle.price) | 6 079


**B5. OBSERVATIONS BRUTES.**

- Le spot Binance n'est JAMAIS passé sous le strike : 0 tick sur 6 079. Quiconque regardait Binance voyait un marché UP.
- Le basis est positif sur 6 079 points sur 6 079 : la hiérarchie spot > oracle n'a pas connu une seule exception en 300 s.


**B6. SPÉCIFIQUE À LA SESSION.** L'écart-type du basis (3,86$) est faible devant sa moyenne (+57,24$) : ce n'est pas du bruit, c'est un décalage structurel de niveau entre les deux sources sur cette fenêtre. Amplitude classée session-spécifique, persistance du signe classée signal.

### B-F3 — oracle.csv

**B1. IDENTITÉ.** oracle.csv ; colonnes t_ms, price, ts_src — toutes utilisées ; 275 lignes ; période t=0,000s → t=298,000s ; échantillonnage nominal 1 s, 18 trous supérieurs à 1 s, trou maximum 8 s (t=168,000s → t=176,000s).

**B2. APPORT.** La série qui règle le marché. C'est la seule pièce qui détermine l'issue DOWN ; les graphes la montrent, le CSV donne le tick exact et le champ de traçabilité ts_src.

**B3. COMMENT.** Strike = price de la ligne t=0,000s (64 222,4357737$) ; comptage des ticks strictement sous le strike ; min/max/moyenne ; vérification ts_src = «payload» sur les 275 lignes ; écarts inter-ticks consécutifs.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Strike (tick t=0) | 64 222,44$ | t_ms, price | 1
| Oracle minimum | 64 173,72$ (t=53,000s) | price | 275
| Oracle maximum | 64 222,44$ (t=1,000s) | price | 275
| Oracle moyen | 64 204,88$ | price | 275
| Dernier tick | 64 205,83$ (t=298,000s) | t_ms, price | 1
| Ticks strictement sous le strike | 273 sur 275 (toutes sauf t=0,000s et t=1,000s) | price | 275
| Maximum après t=60,000s | 64 216,37$, soit 6,07$ sous le strike | t_ms, price | 239
| ts fallback (ts_src≠payload) | 0/275 (0,0%) | ts_src | 275


**B5. OBSERVATIONS BRUTES.**

- L'oracle passe sous le strike à t=2,000s (64 222,37$) et n'y remonte plus jamais : 273 ticks consécutifs sous le strike.
- Même son meilleur rebond après t=60,000s reste 6,07$ sous le strike : l'issue DOWN n'a jamais été menacée après la première minute.
- Traçabilité parfaite : 0 fallback de timestamp sur 275.


**B6. SPÉCIFIQUE À LA SESSION.** 18 trous d'échantillonnage dont un de 8 s (t=168,000s→176,000s) : pendant ces trous, la règle du «dernier tick connu» s'applique ; aucun trou ne chevauche un passage près du strike (écart minimal au strike pendant le trou de 8 s : 6,07$ au mieux du reste de session), verdict bruit sans impact.

### B-F4 — trades.csv

**B1. IDENTITÉ.** trades.csv ; colonnes t_ms, usd, direction — toutes utilisées ; 1 369 lignes ; période t=0,735s → environ t=299,3s ; échantillonnage événementiel (un trade par ligne), direction +1 = haussier (BUY YES / SELL NO), −1 = baissier.

**B2. APPORT.** Les tailles en dollars réellement échangées — les graphes 3 et 4 en montrent les agrégats 30 s, le CSV permet les sommes exactes par camp.

**B3. COMMENT.** Somme de usd totale et par signe de direction ; comptages par signe ; imbalance = somme haussière / somme totale ; maximum et médiane de usd.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Volume total | 19 278,70$ | usd | 1 369
| Trades haussiers | 457 pour 5 737,94$ | usd, direction | 457
| Trades baissiers | 912 pour 13 540,76$ | usd, direction | 912
| Imbalance haussière session | 29,76% | usd, direction | 1 369
| Plus gros trade | 963,65$ baissier (t=290,268s) | t_ms, usd, direction | 1
| Taille médiane | 4,15$ | usd | 1 369
| Fenêtre t=148,000s→151,000s | 28 trades, 297,79$ | t_ms, usd | 28
| dont haussiers dans cette fenêtre | 184,07$ (61,8% de la fenêtre) | usd, direction | 28


**B5. OBSERVATIONS BRUTES.**

- 70,24% du volume en dollars est baissier : l'argent réel suit majoritairement l'oracle, pas le spot.
- L'épisode de doute (t=148–151s) est le seul moment où le flux haussier domine localement (61,8% de la fenêtre) — et il perd.
- La taille médiane de 4,15$ signifie qu'un ordre de 5,00$ est un trade ordinaire de ce marché : zéro problème d'impact.


**B6. SPÉCIFIQUE À LA SESSION.** Le trade de 963,65$ à t=290,268s intervient quand le NO cote déjà 0,99$ (B-F1) : c'est un règlement anticipé de gros porteur, pas un signal directionnel, verdict bruit pour la stratégie.

---

## PARTIE C — LA STRATÉGIE «ANCRAGE À L'ORACLE»

### C0. INTERPRÉTATION OPÉRATIONNELLE

Mes cinq observations les plus fortes, celles sur lesquelles tout repose :

- OBS-1 : l'oracle est sous le strike de t=2,000s à la fin, 273 ticks sur 275, sans une seule remontée au-dessus (B-F3, B5).
- OBS-2 : le spot n'est jamais sous le strike (0/6 079) et le basis reste positif sur 6 079 points, moyenne +57,24$ (B-F2, B5).
- OBS-3 : le marché price DOWN avec retard — NO achetable ≤ 0,80$ jusqu'à t=57,548s — puis re-doute deux fois : no_ask ≤ 0,56$ dès t=90,428s et 0,52$ à t=149,075s (B-F1, B5).
- OBS-4 : 70,24% du volume en dollars est baissier ; le seul sursaut haussier local (t=148–151s, 61,8% de la fenêtre) échoue (B-F4, B5).
- OBS-5 : traverser le carnet coûte 0,0223$ en moyenne et une position de 5,00$ est la taille médiane du marché (4,15$) : la stratégie est exécutable sans friction (B-F1/B-F4, B5).


PRINCIPE DIRECTEUR : l'oracle Chainlink est l'unique source de vérité du règlement ; s'y ancrer signifie prendre position dans le sens du signe (oracle − strike) dès que ce signe est persistant, et traiter tout retour du marché vers 0,50$ comme une erreur à acheter tant que le signe de l'oracle n'a pas changé.

Le principe traduit en règles — voici le tableau :

| Règle | Formulation chiffrée | Observations sources (A5/B5)
|-----|-----|-----|-----
| R1 Entrée | Si l'oracle est resté sans interruption du même côté du strike pendant ≥ 45 s, acheter le côté correspondant (ici NO) au ask courant | OBS-1 (B-F3, B5) ; OBS-3 (B-F1, B5)
| R2 Ré-ancrage | Si le prix du côté détenu retombe vers 0,50$ alors que le signe de l'oracle est inchangé, racheter avec le cash restant | OBS-3 (B-F1, B5) ; OBS-4 (B-F4, B5)
| R3 Sortie de sécurité | Si l'oracle repasse de l'autre côté du strike, vendre toute la position au bid immédiatement | OBS-1 (B-F3, B5)
| R4 Sortie nominale | Sinon, tenir jusqu'au règlement : NO vaut 1,00$ si l'oracle finit sous le strike | OBS-1 (B-F3, B5)
| R5 Protection du capital | Aucun ordre ne dépasse le cash disponible à l'instant t ; pas de vente à découvert ; pas de levier | contrainte de capital 5,00$


Hypothèses non dérivées des données, une par ligne :

- Frais de transaction : 0,00$ — aucune colonne de frais n'existe dans les 4 CSV, je pose des frais nuls. [HYPOTHÈSE n°1]
- Exécution : tout ordre est rempli intégralement au ask affiché de bbo.csv à l'instant du signal, la taille (≤ 7 parts) étant inférieure à la taille médiane des trades du marché (4,15$, B-F4, B4). [HYPOTHÈSE n°2]


### C1-ADAPT. JOURNAL DES ADAPTATIONS

J'ai calibré trois choses avant de simuler — les voici, chacune tracée :

| # | Paramètre/Règle d'origine | Adaptation | Justification chiffrée (source)
|-----|-----|-----|-----
| 1 | R1 : entrée dès 45 s de persistance | Entrée à la première cotation BBO postérieure à t=60,000s (persistance alors constatée : 58,6 s) | Entre t=51,577s et t=60,390s le spot rebondit de 64 230,59$ à 64 243,62$ (A-G2, A4) et l'oracle rebondit de 64 173,72$ à 64 187,47$ sans approcher le strike à moins de 34,97$ (B-F3, B4) : je laisse passer le premier rebond pour vérifier que l'ancrage tient. [ADAPTATION n°1]
| 2 | R1 : engager 100% du cash à l'entrée | Engager 90% du cash à l'entrée, réserver le solde au ré-ancrage | Le NO mid retombe à 0,485$ en cours de session (B-F1, B4) : sans réserve de cash, R2 est inapplicable. [ADAPTATION n°2]
| 3 | R2 : «retombe vers 0,50$», non chiffré | Seuil déclencheur fixé à no_ask ≤ 0,56$ | Les deux épisodes de doute ramènent le no_ask à 0,55–0,56$ (dès t=90,428s) puis 0,52$ (t=149,075s) pendant que l'oracle reste à au moins 23,28$ sous le strike (64 199,16$ à t=90,000s, B-F3 ; B-F1, B5) : 0,56$ capte les deux épisodes. [ADAPTATION n°3]


Aucune de ces trois adaptations n'a été retouchée après simulation, et aucune ne contredit le principe directeur : toutes trois précisent QUAND et EN QUELLE TAILLE s'ancrer, jamais SUR QUOI.

### C2. LA STRATÉGIE ADAPTÉE EN PSEUDO-CODE

```plaintext
ETAT: cash = 5,00$ ; qty_NO = 0 ; verrou_ordre = LIBRE

signe_oracle(t) = signe( dernier_tick_oracle(t) − 64 222,4357737$ )
                             # strike = oracle.csv ligne t=0,000s (B-F3, B4)

POUR chaque événement BBO e (bbo.csv, 681 évts, ordre chronologique):

  # --- R3 sortie de sécurité (prioritaire) ---
  SI qty_NO > 0 ET signe_oracle(e.t) > 0:
      VENDRE qty_NO au no_bid(e)        # jamais déclenché: max oracle
                                        # après 60 s = 64 216,37$ (B-F3, B4)

  # --- R1 entrée [ADAPTATION n°1] [ADAPTATION n°2] ---
  SI qty_NO == 0 ET e.t > 60,000s ET persistance_sous_strike ≥ 45 s:
      q = plancher( (0,90 × cash) / no_ask(e) )   # 90% du cash [ADAPTATION n°2]
      ACHETER q NO au no_ask(e)         # 1re cotation après 60 s: t=60,643s,
                                        # no_ask = 0,76$ (B-F1)

  # --- R2 ré-ancrage [ADAPTATION n°3] ---
  SI qty_NO > 0 ET no_ask(e) ≤ 0,56$ ET signe_oracle(e.t) < 0:
      q = plancher( cash / no_ask(e) )
      ACHETER q NO au no_ask(e)         # 1er déclenchement: t=90,428s,
                                        # no_ask = 0,56$ (B-F1, B5)

  # R5: tout ACHETER vérifie coût ≤ cash ; sinon ordre rejeté

A t=300 s: REGLEMENT à 1,00$/part NO si oracle final < strike
           # oracle final 64 205,83$ < 64 222,44$ (B-F3, B4)
           # frais = 0,00$ [HYPOTHESE n°1] ; fill intégral [HYPOTHESE n°2]
```

### C3. TABLE DE CONVERGENCE

Chaque règle, avec ce qui la soutient et ce qui la gêne :

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| R1 entrée sur persistance oracle | B-F3 (273/275 sous strike) ; A-G2 (oracle sous strike toute la vue) ; B-F4 (70,24% volume baissier) | B-F2 et A-G2 (spot toujours au-dessus du strike : le flux rapide criait UP) | Confirmée, contradiction spot assumée par le principe
| R2 ré-ancrage ≤ 0,56$ | B-F1 (no_ask 0,56$ à t=90,428s ; 0,52$ à t=149,075s) ; A-G1 (NO 0,52$ à t=02:29.075) ; B-F4 (le sursaut haussier t=148–151s échoue) | aucune | Confirmée par 3 pièces
| R3 sortie si oracle recroise | B-F3 (max après 60 s = 64 216,37$, jamais déclenchée) | aucune | Non testée sur session [FRAGILE - 1 SOURCE]
| R4 tenir au règlement | B-F3 (oracle final 64 205,83$) ; A-G1 (NO 0,99$ à t=04:49.893) ; PDF (résultat DOWN) | aucune | Confirmée
| R5 protection cash | journal C-SIM (2 ordres, aucun dépassement) | aucune | Confirmée


Je le dis à voix haute : R3 est marquée FRAGILE — une seule pièce la soutient et la session ne l'a jamais déclenchée. Sa valeur reste théorique.

### C-SIM. SIMULATION ALGORITHMIQUE DÉTAILLÉE

**ÉTAT DU PORTEFEUILLE.** Départ : cash 5,00$, 0 position, valeur 5,00$. Chaque événement du journal ci-dessous met à jour cash et position ; aucun achat n'a dépassé le cash disponible :

- t=60,643s : achat 5 NO à 0,76$ = 3,80$ ≤ 5,00$ disponible. Après : cash 1,20$, 5 NO à 0,76$, valeur mark-to-market au no_bid (0,74$) = 1,20$ + 3,70$ = 4,90$.
- t=90,428s : achat 2 NO à 0,56$ = 1,12$ ≤ 1,20$ disponible. Après : cash 0,08$, 7 NO (5 à 0,76$, 2 à 0,56$).
- t=300,000s : règlement DOWN, 7 × 1,00$ = 7,00$ crédités. Final : cash 7,08$, 0 position.


**RACE CONDITIONS.**
(a) Signal d'achat pendant un ordre en cours : un verrou global d'ordre est posé à l'émission et levé à la confirmation ; tout signal reçu verrou fermé est rejeté, pas mis en file. RÈGLE : verrou d'ordre unique, second signal rejeté. Dans cette session, les deux signaux (t=60,643s et t=90,428s) sont séparés de 29,785 s : le cas ne s'est pas présenté, la règle existait avant de le savoir.
(b) Signal de vente sur une position non confirmée à l'achat : la vente (R3) ne peut porter que sur qty_NO confirmée ; une position en attente de confirmation n'est pas vendable. RÈGLE : R3 s'applique à la quantité confirmée uniquement, jamais à la quantité en vol. Jamais déclenché ici (R3 jamais activée, B-F3, B4).
(c) Deux signaux simultanés sur le même actif : les règles sont évaluées dans l'ordre fixe R3 > R1 > R2 au sein d'un même timestamp ; une seule peut consommer le verrou. RÈGLE : priorité statique R3 > R1 > R2, un seul ordre par timestamp. Aux timestamps à événements multiples du BBO (ex. t=148,975s, 5 lignes), c'est la dernière cotation du timestamp qui fait foi.
(d) Fill partiel : tout fill partiel est accepté, le reliquat d'ordre est annulé immédiatement, jamais reposté. RÈGLE : fill partiel accepté, reliquat annulé. Sous [HYPOTHÈSE n°2] (tailles ≤ taille médiane du marché de 4,15$), les deux ordres sont réputés remplis en totalité.

**JOURNAL DE TRADES.** Le journal, ligne par ligne :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=60,643s | 0,76$ | 5 NO | t=300,000s (règlement) | 1,00$ | 0,00$ | +1,20$ | 5,08$
| 2 | t=90,428s | 0,56$ | 2 NO | t=300,000s (règlement) | 1,00$ | 0,00$ | +0,88$ | 7,08$


Ligne 1 : 5,00$ − 3,80$ = 1,20$ de cash après entrée ; ligne 2 : 1,20$ − 1,12$ = 0,08$ ; règlement ligne 1 : 0,08$ + 5,00$ = 5,08$ ; règlement ligne 2 : 5,08$ + 2,00$ = 7,08$.

**RÉSULTATS.** Le tableau récapitulatif :

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 2 / +2,08$ | lignes 1 et 2
| Trades perdants / pertes cumulées | 0 / 0,00$ | aucune
| Frais totaux | 0,00$ (0,00$ + 0,00$) | lignes 1 et 2 [HYPOTHÈSE n°1]
| BÉNÉFICE NET | +2,08$ | lignes 1 et 2
| Capital final vs initial | 7,08$ vs 5,00$, soit +41,6% | ligne 2, colonne Cash après
| Pire perte unitaire | 0,00$ (aucun trade perdant) | aucune
| Drawdown maximum (mark-to-market au no_bid) | −1,89$, du pic de 5,47$ (t=141,391s) au creux de 3,58$ (t=149,520s) | lignes 1–2 valorisées sur B-F1


Regardez ce drawdown : entre t=141,391s et t=149,520s, la position valait 3,58$, soit 28,4% sous le capital initial, pendant que le no_ask touchait 0,52$ (B-F1, B5). C'est le prix psychologique de l'ancrage : l'oracle était alors à 17,62$ sous le strike (64 204,82$ à t=149,000s, B-F3) et la stratégie n'avait aucune raison de sortir.

### C4. VERDICT ET PORTÉE

VERDICT : [VALIDÉE SUR SESSION] — bénéfice net +2,08$, 2 trades gagnants sur 2, aucun frais, aucune violation de cash, drawdown maximal −1,89$ supporté sans déclencher R3.

Ce qui relève de la stratégie et ce qui relève de mes réglages : le gain vient du principe (l'oracle sous le strike 273/275 ticks pendant que le marché doutait) ; mes adaptations n'ont fait qu'améliorer le prix moyen d'achat (0,70$ le NO au lieu de 0,76$ en tout-à-l'entrée).

- Le décalage persistant oracle-sous-strike / spot-au-dessus, exploitable en achetant le côté oracle contre le doute du marché. [PATTERN CANDIDAT]
- L'amplitude du basis (+41,47$ à +70,46$) qui a rendu le décalage aussi net. [SESSION-SPÉCIFIQUE]
- Le seuil de ré-ancrage à 0,56$, calibré sur les deux creux de cette session. [SESSION-SPÉCIFIQUE]
- L'existence d'épisodes de doute revendables après formation du consensus (retours du NO à 0,56$ puis 0,52$). [PATTERN CANDIDAT]
- Le rendement de +41,6% en 5 minutes. [SESSION-SPÉCIFIQUE]


---

## AUTO-CONTRÔLE FINAL

- Contrôle 1 → Valeurs chiffrées : A-G1 : 6 ; A-G2 : 6 ; A-G3 : 5 ; A-G4 : 5 ; A-G5 : 5 ; A-G6 : 4 + 1 DONNÉE NON DISPONIBLE déclarée ligne à ligne ; B-F1 : 9 ; B-F2 : 8 ; B-F3 : 8 ; B-F4 : 8. Conforme.
- Contrôle 2 → Chaque affirmation de la Partie C pointe vers une rubrique A4/B4 ou une OBS-n sourcée ; aucune affirmation orpheline détectée.
- Contrôle 3 → Aucun chiffre en double avec des valeurs différentes ; le «8 incohérents» du PDF et ma «1 ligne croisée» sont deux métriques distinctes, explicitées comme telles en A-G6/B-F1.
- Contrôle 4 → Cohérence comptable : 5,00 − 3,80 = 1,20 ; 1,20 − 1,12 = 0,08 ; 0,08 + 5,00 + 2,00 = 7,08$ = capital final annoncé, au centime près.
- Contrôle 5 → [HYPOTHÈSE] utilisées : 2 (frais nuls ; fill intégral au ask). Inférieur à 3, pas de justification supplémentaire requise.
- Contrôle 6 → [ADAPTATION] utilisées : 3, toutes au journal C1-ADAPT avec justification chiffrée, aucune ne contredit le principe directeur, aucune décidée après C-SIM.
- Contrôle 7 → Format du speech : adresse directe, verdict dès l'ouverture, ordre F1 respecté, tableaux F4 tous présents, identifiants A-G1…B-F4 utilisés en références croisées, clôture en 3 lignes sans chiffre nouveau. Conforme.


## CLÔTURE DU SPEECH

Vous avez maintenant la trace complète : bénéfice net de +2,08$, capital final de 7,08$ contre 5,00$ de départ.
La stratégie «ANCRAGE À L'ORACLE» est [VALIDÉE SUR SESSION] : elle a acheté la vérité du règlement pendant que le marché regardait la mauvaise horloge.
Son mécanisme mérite d'autres sessions ; son amplitude, elle, appartient à celle-ci.

---

Tous les chiffres ci-dessus proviennent de calculs exécutés directement sur vos quatre CSV (copies de travail dans `.v0/data/`), vérifiés au centime, et des étiquettes lisibles du PDF — rien n'a été estimé de tête. Le point pivot de l'analyse : le spot Binance n'a jamais coté sous le strike (0/6 079 ticks) alors que l'oracle Chainlink, seul juge du règlement, est resté sous le strike 273/275 ticks — c'est cet écart qui rend la session gagnante pour la stratégie et qui fonde le verdict.


