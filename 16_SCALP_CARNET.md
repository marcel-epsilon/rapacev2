# 16 — SCALP DE CARNET CROISÉ

**Fichiers sources regroupés :** Scalp_de_carnet_croisé_avec_fusion _de_paires.md

---

## Source : Scalp_de_carnet_croisé_avec_fusion _de_paires.md

# 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Vous avez le PDF et les CSV sous les yeux ; voici le verdict d'entrée.

- Stratégie retenue : **Scalp de carnet croisé avec fusion de paires** — achat simultané YES+NO quand la somme des asks passe sous 1,00$.
- Capital final : **5,20$** contre 5,00$ de départ, soit **+0,20$ net, +4,00%**.
- Trades : **4**, dont **4 gagnants, 0 perdant**.
- Critère de sélection, fixé avant comparaison : **perte maximale possible (à minimiser), P&L net en départage**.
- Étiquette globale : mécanisme [PATTERN CANDIDAT], fréquence d'occurrence [SESSION-SPÉCIFIQUE].


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

# 1. CADRE D'ANALYSE CHOISI

J'adopte un cadre **microstructurel événementiel** : lecture tick par tick du carnet, des flux et des deux références de prix (spot et oracle), à la milliseconde. Il est adapté à ces données précises : 5 minutes seulement, 793 événements BBO horodatés à la ms, 1993 ticks spot, 278 ticks oracle, 1202 trades — aucune place pour une analyse fondamentale ou de tendance longue. Le seul gisement exploitable ici est la mécanique interne du marché : écarts entre carnets, retards entre sources de prix, anomalies de cotation.

# 2. PARTIE A — LES 6 GRAPHES DU PDF

## A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET

**A1. IDENTITÉ.** Vue 300 s + 3 zooms 15 s. Axe X : temps depuis l'ouverture (mm:ss.mmm), de 00:00.000 à 05:00.000. Axe Y : prix en $, de 0,0 à 1,0. Six séries : YES/NO bid, ask, mid (793 évts chacune), plus la probabilité consensus et 29 marqueurs "carnet croisé — suspects".

**A2. QUESTION.** Que fait le prix des deux jambes quand l'une bouge ? Le carnet reste-t-il cohérent (YES + NO = 1,00$) en permanence ?

**A3. COMMENT.** Lecture des étiquettes d'extrema sur la vue 300 s, puis des séquences tick par tick sur les 3 zooms ; repérage visuel des marqueurs creux "suspects".

**A4. VALEURS EXTRAITES.**

- t=1,069s : YES mid = 0,51$
- t=147,893s : YES mid = 0,21$ (minimum de session)
- t=298,788s : YES mid = 0,99$ (maximum, fin de session)
- t=164,706s (zoom 02:37–02:52, séquence 02:44.706) : YES passe de 0,28$ à 0,35$ en 0 ms affiché
- t=166,218s (02:46.218) : YES mid = 0,70$


**A5. OBSERVATIONS BRUTES.**

- Le carnet affiche 29 événements marqués "croisés/suspects" sur 793 (étiquette du titre du graphe).
- Le YES fait 0,21$ → 0,70$ entre t=164,391s et t=166,218s, soit +0,49$ en 1,827s (zoom 02:37–02:52).
- YES mid et 1 − NO mid se superposent visuellement sur toute la session.


**A6. SPÉCIFIQUE À LA SESSION.** Le trajet exact 0,51$ → 0,21$ → 0,99$ dépend du parcours du BTC de cette fenêtre ; non reproductible. La clôture à 0,99$ traduit le résultat UP de cette session-ci.

Vous venez de voir le prix du contrat ; voyons maintenant le sous-jacent qui le pilote.

## A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK

**A1. IDENTITÉ.** Vue 300 s + 3 zooms. Axe Y : prix BTC en $, de 64 290 à 64 360. Trois séries : Binance BTC/USDT (1993 ticks), Chainlink BTC/USD oracle (278 ticks), strike d'ouverture à 64 303$.

**A2. QUESTION.** Que fait l'oracle quand le spot bouge ? Lequel des deux détermine la position vis-à-vis du strike ?

**A3. COMMENT.** Lecture des étiquettes d'extrema du spot et de la série oracle seconde par seconde ; mesure de l'écart vertical constant entre les deux courbes ; repérage des traversées du strike par l'oracle seul.

**A4. VALEURS EXTRAITES.**

- t=1,357s : spot = 64 353,95$
- t=113,418s : spot = 64 348,16$ (minimum de session)
- t=227,288s : spot = 64 360,01$ (maximum de session)
- t=66,000s : oracle = 64 301,75$ (premier passage franc sous le strike)
- t=72,000s : oracle = 64 293,62$


**A5. OBSERVATIONS BRUTES.**

- Le spot ne descend jamais sous 64 348,16$ ; le strike est à 64 303$ : le spot reste au-dessus du strike 100% de la session.
- L'oracle, lui, traverse le strike : il cote 64 301,75$ à t=66,000s alors que le spot cote au-dessus de 64 348$ au même instant.
- L'écart vertical spot − oracle est visuellement constant, de l'ordre de 5 graduations sur 7 de l'axe — je le chiffre précisément en B-F1/B-F2 : 55,55$ de moyenne.


**A6. SPÉCIFIQUE À LA SESSION.** L'amplitude spot de 11,85$ (64 360,01 − 64 348,16) sur 5 minutes est une condition de volatilité propre à cette fenêtre. La valeur absolue du basis (55,55$) est propre à la paire USDT/USD de ce jour-là.

Le prix est piloté par l'oracle ; voyons maintenant qui achète et qui vend.

## A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ

**A1. IDENTITÉ.** Vue 300 s + 3 zooms. Axe Y : volume en $ par fenêtre 30 s, de −1000 à 3000 (baissier en négatif). Deux séries : flux haussier (BUY YES + SELL NO) et flux baissier (SELL YES + BUY NO), 1202 trades.

**A2. QUESTION.** Que fait le flux quand l'oracle traverse le strike ? Le flux anticipe-t-il ou suit-il le prix ?

**A3. COMMENT.** Lecture des étiquettes de trades individuels sur les zooms et des pics de la vue 300 s.

**A4. VALEURS EXTRAITES.**

- t=40,981s : trade baissier de −504,53$
- t=6,044s : trade haussier de 48,80$
- t=66,770s : trade baissier de −60,39$
- t=76,581s : trade haussier de 100,24$
- t=75,109s : trade haussier de 50,94$


**A5. OBSERVATIONS BRUTES.**

- Le flux haussier culmine visuellement en dernière minute, au-dessus de 3 000$ par fenêtre 30 s.
- Des trades haussiers de 50,94$ et 100,24$ apparaissent dès t=75–77s, pendant que l'oracle est sous le strike (A-G2, A4).


**A6. SPÉCIFIQUE À LA SESSION.** Le trade isolé de −504,53$ à t=40,981s est un ordre unique non répété ; bruit d'un acteur, pas un pattern.

Le flux brut est bruité ; le graphe suivant le normalise.

## A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ.** Vue 300 s + 3 zooms. Axe Y : imbalance = haussier / (haussier + baissier), de 0,0 à 1,0, ligne d'équilibre à 0,5. 1202 évts.

**A2. QUESTION.** Que fait l'imbalance autour des traversées de strike ? Reste-t-elle collée à 0,5 ou prend-elle parti ?

**A3. COMMENT.** Lecture des étiquettes d'extrema sur la vue 300 s et les zooms, comparaison à la ligne 0,5.

**A4. VALEURS EXTRAITES.**

- t=1,521s : imbalance = 0,00
- t=6,403s : imbalance = 0,98
- t=70,277s : imbalance = 0,12
- t=165,510s : imbalance = 0,88
- t=299,423s : imbalance = 0,90


**A5. OBSERVATIONS BRUTES.**

- L'imbalance tombe à 0,12 à t=70,277s, pendant la chute de l'oracle sous le strike (A-G2, A4 : oracle à 64 293,62$ à t=72,000s).
- Elle monte à 0,88 à t=165,510s, au moment exact de la remontée du YES de 0,21$ à 0,70$ (A-G1, A5).
- Elle finit à 0,90 à t=299,423s : le flux final est massivement aligné avec le résultat UP.


**A6. SPÉCIFIQUE À LA SESSION.** Les extrêmes 0,00 et 0,98 des 7 premières secondes portent sur un volume minuscule de début de fenêtre ; artefact de normalisation à faible dénominateur.

L'imbalance suit le prix ; regardons maintenant le coût de friction pour trader tout cela.

## A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS

**A1. IDENTITÉ.** Vue 300 s + 3 zooms. Axe Y : spread (ask − bid) en $, de 0,000 à 0,175. Deux séries, 793 évts chacune.

**A2. QUESTION.** Que fait le spread au repos et pendant les chocs ? Passe-t-il sous zéro ?

**A3. COMMENT.** Lecture des étiquettes d'extrema, en particulier la valeur négative — un spread négatif signifie bid > ask, donc carnet croisé exploitable.

**A4. VALEURS EXTRAITES.**

- t=1,069s : spread = 0,01$
- t=88,255s : spread = **−0,01$** (valeur négative, étiquetée sur les deux séries)
- t=164,706s : spread = 0,18$ (maximum de session)
- t=298,788s : spread = 0,00$
- t=5,624s : spread = 0,10$


**A5. OBSERVATIONS BRUTES.**

- Le spread devient négatif (−0,01$) à t=88,255s : le carnet est croisé, c'est un prix d'arbitrage affiché publiquement.
- Le spread explose à 0,18$ à t=164,706s, pendant le choc de prix identifié en A-G1 : trader en momentum paie 18 fois le spread médian.
- Hors chocs, le spread visuel reste dans la bande 0,01–0,03$.


**A6. SPÉCIFIQUE À LA SESSION.** Le pic exact à 0,18$ est lié au choc unique de t=164–166s ; sa valeur précise n'est pas reproductible, son existence pendant les chocs l'est.

Le spread négatif est l'anomalie centrale ; le dernier graphe vérifie la cohérence entre les deux carnets.

## A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ.** Vue 300 s + 3 zooms. Axe Y : écart = yes_mid − (1 − no_mid) en $, échelle ±1,0 × 10⁻¹⁶, ligne "cohérence parfaite" à 0. 793 évts, 29 marqueurs suspects.

**A2. QUESTION.** Les carnets YES et NO racontent-ils le même prix ? L'incohérence inter-carnets est-elle une source de profit distincte du carnet croisé ?

**A3. COMMENT.** Lecture de l'échelle de l'axe (facteur 1e−16) et des marqueurs suspects superposés à la ligne zéro.

**A4. VALEURS EXTRAITES.**

- t=1,069s à t=298,788s : écart = 0,00$ à la précision 10⁻¹⁶ sur toute la série visible
- t=88,255s : marqueur suspect présent, écart affiché 0,00$
- t=225,464s : marqueur suspect présent, écart affiché 0,00$
- t=231,015s : marqueur suspect présent, écart affiché 0,00$
- t=234,048s : marqueur suspect présent, écart affiché 0,00$


**A5. OBSERVATIONS BRUTES.**

- Les mids YES et NO sont parfaitement complémentaires (écart au niveau de l'erreur machine 10⁻¹⁶) : il n'existe AUCUN arbitrage entre mids.
- Les 4 marqueurs suspects que je peux dater tombent exactement sur les 4 instants de spread négatif de A-G5 : l'anomalie est dans les bid/ask croisés, jamais dans les mids.
- Le décompte "29 suspects" du titre n'est pas décomposable depuis ce graphe : la liste des 29 timestamps individuels est DONNÉE NON DISPONIBLE ; mon recomptage CSV en B-F3 en confirme 4.


**A6. SPÉCIFIQUE À LA SESSION.** Rien : ce graphe montre une propriété structurelle du marché (complémentarité des mids), pas un fait de session.

Les graphes posés, je passe aux fichiers pour chiffrer au tick près.

# 3. PARTIE B — LES FICHIERS CSV

## B-F1 — spot.csv

**B1. IDENTITÉ.** Colonnes : t_ms, price — toutes deux utilisées. 1993 lignes de données. Période : t=1,357s à t=298,846s. Échantillonnage événementiel, gap médian 0 ms (rafales), gap maximum 7 774 ms.

**B2. APPORT.** La granularité milliseconde du spot, que le graphe 2 ne donne qu'au trait ; permet de dater les chocs au tick près.

**B3. COMMENT.** Min/max/moyenne sur price ; détection de chocs = variation cumulée de |Δprice| ≥ 5$ dans une fenêtre glissante de 500 ms ; variation maximale sur 100 ms par balayage exhaustif des paires de ticks.

**B4. RÉSULTATS CHIFFRÉS.** Le tableau, tel quel :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Prix minimum | 64 348,16$ | price | 1993
| Prix maximum | 64 360,01$ | price | 1993
| Prix moyen | 64 354,39$ | price | 1993
| Amplitude totale | 11,85$ | price | 1993
| Chocs ≥5$/500ms (nombre) | 2 | t_ms, price | 1993
| Choc 1 : chute | −9,82$ entre t=69,288s et t=69,574s | t_ms, price | 1993
| Choc 2 : hausse | +7,99$ entre t=165,579s et t=165,897s | t_ms, price | 1993
| Variation max sur 100 ms | −9,82$ (t=69,506s → t=69,574s) | t_ms, price | 1993


**B5. OBSERVATIONS BRUTES.**

- Le spot ne produit que 2 chocs ≥5$/500ms en 300 s ; le reste du temps il oscille dans une bande inférieure à 3$.
- Le spot reste 100% de la session au-dessus du strike de 64 303,30$ (minimum 64 348,16$, écart minimal +44,86$).


**B6. SPÉCIFIQUE À LA SESSION.** Trou de données de 7 774 ms (gap maximum) ; verdict : bruit de flux, sans impact car aucun choc ne s'y produit. Les deux chocs eux-mêmes sont des événements de cette fenêtre précise.

## B-F2 — oracle.csv

**B1. IDENTITÉ.** Colonnes : t_ms, price, ts_src — toutes trois utilisées. 278 lignes. Période : t=0,000s à t=298,000s. Échantillonnage : gap médian 1 000 ms, gap maximum 2 000 ms, 21 gaps supérieurs à 1 000 ms.

**B2. APPORT.** Le prix qui règle le marché — la ligne 1 (t=0) donne le strike exact, invisible à cette précision sur les graphes : **64 303,29737414$**.

**B3. COMMENT.** Strike = price à t=0 ; delta = price − strike par ligne ; comptage des lignes sous le strike ; croisements = changements de signe de delta entre lignes consécutives ; basis = price_spot − price_oracle du dernier tick oracle antérieur (jointure asof sur les 1993 ticks spot) ; contrôle qualité sur ts_src.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Strike (t=0) | 64 303,29737414$ | price | 1
| Prix final (t=298,000s) | 64 306,42$, soit strike +3,13$ | price | 1
| Minimum de session | 64 289,34$ (strike −13,96$) à t=163,000s | t_ms, price | 278
| Ticks sous le strike | 102 sur 278 (36,7% du temps) | price | 278
| Croisements du strike | 6 (t=18s, 22s, 23s, 27s, 66s, 168s) | t_ms, price | 278
| Basis spot−oracle : moyenne | +55,55$ | price (×2 fichiers) | 1993
| Basis : écart-type / min / max | 3,19$ / +48,87$ / +63,74$ | price (×2 fichiers) | 1993
| Plus grand saut oracle 1 tick | +6,91$ à t=165,000s ; ts fallback : 0/278 | t_ms, price, ts_src | 278


**B5. OBSERVATIONS BRUTES.**

- Le marché se joue entièrement sur l'oracle : spot toujours UP (B-F1, B5), oracle sous le strike 36,7% du temps.
- Le basis de +55,55$ n'est pas exploitable en direction : son écart-type de 3,19$ est du même ordre que les mouvements d'oracle tick à tick.
- L'oracle bouge par sauts (jusqu'à +6,91$ en 1 tick) avec un pas d'échantillonnage de 1 s : toute stratégie indexée sur lui subit ce retard.


**B6. SPÉCIFIQUE À LA SESSION.** L'excursion à strike −13,96$ suivie d'un retour au-dessus en 5 ticks (t=163s à 168s) est l'événement singulier de la session ; verdict : signal pour cette session, non reproductible en amplitude.

## B-F3 — bbo.csv

**B1. IDENTITÉ.** Colonnes : t_ms, yes_bid, yes_ask, no_bid, no_ask — toutes utilisées. 793 lignes, dont 791 complètes et 2 incomplètes (t=1,069s et t=298,788s, jambes NO ou ask vides). Période : t=1,069s à t=298,788s. Événementiel, à la ms.

**B2. APPORT.** Les prix exécutables réels — les graphes montrent les mids, ce fichier donne les asks et bids auxquels on peut effectivement acheter et vendre.

**B3. COMMENT.** Sur les 791 lignes complètes : spread = ask − bid par jambe ; carnet croisé = yes_bid > yes_ask ou no_bid > no_ask ; coût de paire = yes_ask + no_ask ; écart inter-carnets = yes_mid − (1 − no_mid).

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Spread YES moyen | 0,0237$ | yes_bid, yes_ask | 791
| Spread YES médian / max | 0,02$ / 0,18$ (t=164,706s) | yes_bid, yes_ask | 791
| Lignes à spread négatif | 4 | yes_bid, yes_ask, no_bid, no_ask | 791
| Timestamps des 4 croisements | t=88,255s ; t=225,464s ; t=231,015s ; t=234,048s | t_ms | 4
| Coût de paire à ces 4 instants | yes_ask + no_ask = 0,99$ (les 4 fois) | yes_ask, no_ask | 4
| Écart inter-carnets non nul | 0 ligne sur 791 (tolérance 10⁻¹²) | les 4 colonnes de prix | 791
| YES mid min / max | 0,215$ (t=147,893s) / 0,99$ (t=298,788s) | yes_bid, yes_ask | 791
| Événements BBO par minute | 58, 159, 128, 188, 258 | t_ms | 793


**B5. OBSERVATIONS BRUTES.**

- À 4 instants datés, acheter YES au ask ET NO au ask coûte 0,99$ pour un panier qui vaut 1,00$ par construction : profit mécanique de 0,01$ par paire, sans vue directionnelle.
- Les mids sont parfaitement complémentaires sur 791 lignes sur 791 : aucune autre incohérence exploitable n'existe dans ce carnet.
- L'activité du carnet accélère de façon monotone hors minute 3 : 58 puis 258 évts/minute.


**B6. SPÉCIFIQUE À LA SESSION.** Le décompte "29 incohérents" du PDF n'est pas reproductible depuis ce fichier (je mesure 4 lignes croisées et 0 écart de mids) ; la définition exacte des 29 est DONNÉE NON DISPONIBLE. Verdict : je ne fonde rien sur les 25 non retrouvés. Les 2 lignes incomplètes sont les bornes d'ouverture/clôture ; bruit.

## B-F4 — trades.csv

**B1. IDENTITÉ.** Colonnes : t_ms, usd, direction — toutes utilisées. 1202 lignes. Gap médian entre trades : 116 ms, gap maximum 3 700 ms.

**B2. APPORT.** Le volume signé en dollars, que les graphes 3 et 4 n'affichent qu'agrégé par 30 s.

**B3. COMMENT.** Sommes et comptages par direction (+1 haussier, −1 baissier), globaux, par minute, et sur t≥240s ; imbalance = Σhaussier / Σtotal.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Volume total | 16 432,62$ | usd | 1202
| Volume haussier / baissier | 10 107,66$ (726 trades) / 6 324,96$ (476 trades) | usd, direction | 1202
| Imbalance globale | 0,6151 | usd, direction | 1202
| Taille moyenne / médiane | 13,67$ / 4,00$ | usd | 1202
| Plus gros trade | 826,56$, haussier, t=148,791s | t_ms, usd, direction | 1
| Trades ≥100$ | 26 trades, 6 201,33$ (37,7% du volume) | usd | 1202
| Imbalance par minute | 0,4327 ; 0,5289 ; 0,6073 ; 0,4670 ; 0,8633 | t_ms, usd, direction | 1202
| Dernière minute (t≥240s) | 375 trades, 3 947,92$ haussier vs 625,25$ baissier | t_ms, usd, direction | 375


**B5. OBSERVATIONS BRUTES.**

- La médiane à 4,00$ contre une moyenne à 13,67$ : le flux est fait de très petits ordres percés par 26 gros ordres qui portent 37,7% du volume.
- L'imbalance ne devient franchement directionnelle (0,8633) qu'en dernière minute, quand le YES cote déjà 0,72/0,73 (B-F3) : le flux confirme, il n'anticipe pas.


**B6. SPÉCIFIQUE À LA SESSION.** Le trade de 826,56$ à t=148,791s arrive au plus bas du YES (0,215$ à t=147,893s, B-F3) et s'avère gagnant ; un seul cas, verdict : bruit d'un acteur informé ou chanceux, inexploitable.

Les pièces sont posées ; j'en tire maintenant les stratégies.

# 4. PARTIE C

## C0 — STRATÉGIES CANDIDATES

Mes cinq observations les plus fortes, sourcées :

- **OBS-1** : 4 instants datés où yes_ask + no_ask = 0,99$ < 1,00$ (B-F3, B4 ; confirmé par le spread −0,01$ en A-G5, A4).
- **OBS-2** : le marché se règle sur l'oracle, qui passe 36,7% du temps sous le strike avec 6 croisements, pendant que le spot reste 100% au-dessus (B-F2, B4 ; B-F1, B5).
- **OBS-3** : 2 chocs spot ≥5$/500ms seulement, datés t=69,288s et t=165,579s (B-F1, B4), pendant lesquels le spread monte à 0,18$ (A-G5, A4).
- **OBS-4** : à t=240s, l'oracle est à strike +3,98$ et n'en redescend plus (minimum +2,74$ à t=280s), le YES cote 0,73$ au ask (B-F2 ; B-F3).
- **OBS-5** : le flux ne devient directionnel qu'en dernière minute, imbalance 0,8633, après le prix (B-F4, B4).


Quatre candidates, mécanismes réellement distincts :

- **C-A — Scalp de carnet croisé** (OBS-1). Logique : quand le panier YES+NO coûte 0,99$ au ask, l'acheter et le fusionner à 1,00$ ; profit sans direction. Règles : entrée si yes_ask + no_ask ≤ 0,99$, taille = plancher(cash / coût de paire), sortie immédiate par fusion. Simulation sommaire : 4 trades, **+0,20$**.
- **C-B — Suiveur d'oracle contre strike** (OBS-2). Logique : acheter la jambe du côté de l'oracle dès que |oracle − strike| > 2$, sortir au recroisement. Règles : entrée YES si delta > +2$ (ask ≤ 0,95$), NO si delta < −2$ ; sortie au bid au changement de signe. Simulation sommaire : 2 trades, **−1,62$** — l'excursion à −13,96$ (B-F2, B6) fait acheter du NO à 0,65$ revendu 0,29$.
- **C-C — Momentum de choc spot** (OBS-3). Logique : le spot mène l'oracle de 1 s d'échantillonnage ; acheter la jambe du sens du choc, tenir 10 s. Règles : entrée au ask à la fin d'un choc ≥5$/500ms, sortie au bid 10 s après. Simulation sommaire : 2 trades, **+0,17$** (+0,35$ puis −0,18$ : le carnet avait déjà bougé AVANT le choc — à t=164,719s le yes_bid est à 0,37$, 0,86s avant le début du choc spot de t=165,579s ; B-F3 et B-F1).
- **C-D — Convergence tardive** (OBS-4, OBS-5). Logique : à t=240s, oracle à +3,98$ du strike et flux à 0,8633 ; acheter YES à 0,73$ et porter au règlement. Règles : entrée à t≥240s si oracle > strike et yes_ask ≤ 0,85$ ; aucune sortie avant règlement. Simulation sommaire : 1 trade, **+1,62$**.


**CRITÈRE DE SÉLECTION : perte maximale possible de la stratégie (à minimiser), P&L net de session en départage.** Sur 5,00$ de capital et une session unique, la survie du capital prime le rendement.

Le tableau comparatif :

| Stratégie | Observations sources (A5/B5) | Nb trades | P&L net | Verdict
|-----|-----|-----|-----
| C-A Scalp carnet croisé | OBS-1 (B-F3 B5 ; A-G5 A5) | 4 | +0,20$ | **RETENUE** — perte max possible 0,00$
| C-B Suiveur oracle/strike | OBS-2 (B-F2 B5) | 2 | −1,62$ | Rejetée — perte réalisée −2,52$ sur un trade
| C-C Momentum choc spot | OBS-3 (B-F1 B5 ; B-F3 B5) | 2 | +0,17$ | Rejetée — perte max possible = 100% de la position ; 1 trade sur 2 perdant
| C-D Convergence tardive | OBS-4, OBS-5 (B-F2, B-F4 B5) | 1 | +1,62$ | Rejetée — perte max possible −4,38$ (6 parts × 0,73$) si l'oracle recroise ; il est passé à +2,74$ du strike à t=280s


Décision par les chiffres du tableau : C-A est la seule candidate dont la perte maximale possible est 0,00$ — le panier acheté 0,99$ vaut 1,00$ par construction du contrat. C-D rapporte plus (+1,62$) mais expose 87,6% du capital (4,38$/5,00$) à un recroisement d'oracle qui est passé à 2,74$ de se produire (B-F2, B4). Le critère annoncé tranche pour C-A.

## C1 — TABLE DE CONVERGENCE

Chaque règle de la stratégie retenue, avec ses pièces :

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| Entrée : yes_ask + no_ask ≤ 0,99$ | B-F3 B4 (4 lignes croisées) ; A-G5 A4 (spread −0,01$ à t=88,255s) ; A-G6 A4 (4 marqueurs suspects datés) | Aucune | CONFIRMÉ — 3 sources
| Le panier YES+NO vaut 1,00$ (fusion) | A-G6 A5 (mids complémentaires à 10⁻¹⁶ sur 791/791 lignes) ; B-F3 B4 (0 écart inter-carnets) | Aucune | CONFIRMÉ — 2 sources
| Taille : plancher(cash/0,99), jamais plus que le cash | Contrainte de capital de la mission | Aucune | CONFIRMÉ
| Fenêtre d'exécution : l'anomalie tient sur une seule ligne BBO (durée intra-ligne 0 ms mesurée) | B-F3 B4 (timestamps uniques) | Aucune pièce ne donne la durée de vie réelle de la cote | [FRAGILE - 1 SOURCE]


Je marque la règle de fenêtre d'exécution FRAGILE, une seule source la soutient : le fichier BBO ne dit pas combien de temps la cote croisée est restée frappable.

## C2 — PSEUDO-CODE EXÉCUTABLE

```plaintext
ÉTAT : cash = 5,00 ; verrou = LIBRE

POUR CHAQUE ligne BBO (t, yes_bid, yes_ask, no_bid, no_ask) :   # B-F3, 791 lignes complètes
  SI ligne incomplète : IGNORER                                  # B-F3 B1 (2 lignes)
  cout_paire = yes_ask + no_ask                                  # B-F3 B4
  SI cout_paire <= 0,99 ET verrou == LIBRE :                     # B-F3 B4 : 4 occurrences à 0,99
      verrou = OCCUPÉ                                            # règle race condition (a)
      q = plancher(cash / cout_paire)                            # contrainte capital 5,00$
      ENVOYER ordre couple {ACHAT q YES @ yes_ask ; ACHAT q NO @ no_ask}
      # [HYPOTHÈSE n°3] profondeur >= q au BBO affiché (tailles absentes de B-F3)
      SI les DEUX jambes remplies :
          cash -= q * cout_paire
          FUSIONNER q paires -> cash += q * 1,00                 # A-G6 A5 : YES+NO = 1,00
          # [HYPOTHÈSE n°1] fusion de paire disponible immédiatement à 1,00$
          # [HYPOTHÈSE n°2] frais de transaction = 0,00$
      SINON : ANNULER jambe orpheline, REVENDRE au bid si remplie # règle race condition (d)
      verrou = LIBRE
AU RÈGLEMENT : aucune position ouverte attendue ; cash = capital final
```

## C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

### ÉTAT DU PORTEFEUILLE

Journalisé à chaque événement : cash disponible, positions (quantité, prix d'entrée), valeur totale. Entre l'achat et la fusion, la valeur du panier est q × 1,00$ ; la valeur totale du portefeuille ne descend jamais sous 5,00$. Aucun achat ne dépasse le cash : à chaque entrée, q = plancher(cash / 0,99) = 5 paires, coût 4,95$ ≤ cash courant.

### RACE CONDITIONS

(a) Un signal d'achat arrive alors qu'un ordre est en cours : le verrou global est OCCUPÉ pendant tout le cycle achat-fusion. RÈGLE : tout signal reçu pendant que le verrou est OCCUPÉ est rejeté définitivement, sans mise en file.

(b) Un signal de vente sur une position pas encore confirmée à l'achat : la fusion n'est déclenchée qu'à réception des DEUX confirmations d'achat. RÈGLE : aucune instruction de sortie n'est émise avant confirmation complète des deux jambes.

(c) Deux signaux simultanés sur le même actif : les 4 événements croisés portent chacun un timestamp distinct (t=88,255s ; 225,464s ; 231,015s ; 234,048s — B-F3, B4). RÈGLE : à timestamp égal, une seule ligne BBO est traitée, ordre de lecture du fichier, les suivantes sont rejetées par le verrou.

(d) Fill partiel : une jambe remplie, l'autre non. RÈGLE : la jambe orpheline est annulée et la jambe remplie est revendue immédiatement au bid de la même ligne ; le bid étant supérieur à l'ask sur un carnet croisé (yes_bid 0,33$ > yes_ask 0,32$ à t=88,255s, B-F3), ce dénouement est lui-même non perdant sur les 4 événements observés.

### JOURNAL DE TRADES

Le journal, ligne par ligne :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=88,255s | 0,99$ (YES 0,32$ + NO 0,67$) | 5 paires | t=88,255s (fusion) | 1,00$ | 0,00$ | +0,05$ | 5,05$
| 2 | t=225,464s | 0,99$ (YES 0,68$ + NO 0,31$) | 5 paires | t=225,464s (fusion) | 1,00$ | 0,00$ | +0,05$ | 5,10$
| 3 | t=231,015s | 0,99$ (YES 0,68$ + NO 0,31$) | 5 paires | t=231,015s (fusion) | 1,00$ | 0,00$ | +0,05$ | 5,15$
| 4 | t=234,048s | 0,99$ (YES 0,67$ + NO 0,32$) | 5 paires | t=234,048s (fusion) | 1,00$ | 0,00$ | +0,05$ | 5,20$


### RÉSULTATS

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 4 / +0,20$ | lignes 1–4
| Trades perdants / pertes cumulées | 0 / 0,00$ | —
| Frais totaux | 0,00$ (0,00$ × 4) [HYPOTHÈSE n°2] | lignes 1–4
| Bénéfice net (gains − pertes − frais) | **+0,20$** | lignes 1–4
| Capital final vs initial | 5,20$ vs 5,00$, soit **+4,00%** | ligne 4
| Pire perte unitaire | 0,00$ | —
| Drawdown maximum | 0,00$ (valeur portefeuille ≥ 5,00$ de t=1,069s à t=298,788s) | lignes 1–4


## C4 — PORTÉE

- Un carnet croisé à 0,99$ sur un binaire est un profit mécanique de 0,01$/paire, indépendant de la direction. [PATTERN CANDIDAT]
- La fréquence de 4 occurrences en 300 s, et leur concentration entre t=225s et t=234s, tiennent à cette session. [SESSION-SPÉCIFIQUE]
- La complémentarité parfaite des mids YES/NO (écart 10⁻¹⁶) est structurelle au produit. [PATTERN CANDIDAT]
- Le règlement par l'oracle avec basis spot de +55,55$ rend le spot seul trompeur sur ce marché. [PATTERN CANDIDAT]
- L'excursion oracle à strike −13,96$ et le retour en 5 ticks sont un accident de cette fenêtre. [SESSION-SPÉCIFIQUE]
- L'imbalance de flux (0,8633 en dernière minute) suit le prix et n'a pas de valeur prédictive mesurée ici. [SESSION-SPÉCIFIQUE]


# 5. AUTO-CONTRÔLE FINAL

- Contrôle 1 (5 valeurs/graphe, 8/CSV) → conforme : 5 valeurs en A4 pour chacun des 6 graphes ; 8 lignes dans chaque tableau B4 des 4 CSV. Deux mentions DONNÉE NON DISPONIBLE explicites (décomposition des 29 suspects, A-G6 A5 et B-F3 B6).
- Contrôle 2 (traçabilité de C) → conforme : chaque règle de C1/C2 et chaque candidate de C0 cite ses rubriques A4/B4/B5 sources.
- Contrôle 3 (unicité des chiffres) → conforme : 5,20$ / +0,20$ / +4,00% / 4 trades identiques dans l'ouverture, C-SIM et la clôture.
- Contrôle 4 (cohérence comptable) → conforme : 5,00 + 0,05 + 0,05 + 0,05 + 0,05 = 5,20$, égal ligne par ligne à la colonne "Cash après".
- Contrôle 5 (hypothèses) → 3 hypothèses : n°1 fusion de paire instantanée à 1,00$, n°2 frais 0,00$, n°3 profondeur ≥5 paires au BBO. Limite de 3 respectée.
- Contrôle 6 (stratégie issue des observations) → conforme : la stratégie retenue découle d'OBS-1, elle-même issue de B-F3 B5 et A-G5 A5, antérieures à C0 dans ce speech.
- Contrôle 7 (format du speech) → conforme : verdict en ouverture, ordre F1 respecté, tableaux F4 présents, adresse directe, clôture ci-dessous sans chiffre nouveau.


# 6. CLÔTURE

Bénéfice net : +0,20$. Capital final : 5,20$ contre 5,00$ engagés, soit +4,00%. Mécanisme étiqueté [PATTERN CANDIDAT], fréquence d'occurrence [SESSION-SPÉCIFIQUE] — les chiffres sont dans le journal, vérifiables ligne par ligne.



# RAPPORT D'AUDIT OPÉRATIONNEL — STRATÉGIE « SCALP DE CARNET CROISÉ », SESSION BTC 5MIN DU 6 AOÛT

## 0. VERDICT EXÉCUTIF

- Verdict global : **[NON FIABLE]**
- NOTE DE FIABILITÉ : **26/100** (grille F7, détail en fin de rapport)
- Manquements identifiés : **8 facteurs traités — 3 [DÉTRUIT] (O1, O2, O5), 2 [DÉGRADE] (O4, O6), 1 [IMPACT NON CHIFFRABLE] (O3), 2 [SANS IMPACT] (O7, O8)**
- Manquement le plus grave : les 4 cotes croisées ont une durée de vie de **0 ms** dans bbo.csv (corrigées dans la même milliseconde), alors que la boucle d'exécution minimale coûte **62 ms** (31 ms réception + 31 ms envoi, tableau des latences) — aucun des 4 trades du journal n'est exécutable.
- P&L annoncé : **+0,20$** / P&L après réintégration de tous les manquements chiffrables : **0,00$** (0 exécution, écart −0,20$, −100%).
- Point le plus solide : la lecture des données est exacte — les 4 croisements à 0,99$ existent bien dans bbo.csv aux timestamps annoncés, et l'arithmétique du journal est juste au centime.


Je vais vous montrer pièce par pièce d'où vient ce verdict.

## 1. PÉRIMÈTRE

Audité : le speech complet (sections 0 à 6), les 4 CSV (bbo, oracle, spot, trades), le PDF, le tableau des latences AWS us-east-1. Non audité : les stratégies rejetées C-B, C-C, C-D (hors mandat, sauf mention en Partie R) et la provenance des fichiers eux-mêmes, que je prends comme fidèles au marché.

## 2. PARTIE V — SONDAGE DE FIABILITÉ DES DONNÉES

J'ai recalculé depuis les fichiers bruts, jamais depuis le speech.

| # | Valeur sondée | Annoncée | Recalculée | Écart | Verdict
|-----|-----|-----|-----|-----|-----
| 1 | P&L final et cash final (recalcul complet) | +0,20$ / 5,20$ | 4 trades × 5 paires × 0,01$ = +0,20$ / 5,20$ (bbo.csv, lignes croisées ; journal C-SIM) | 0 | [CONFIRMÉ]
| 2 | Seuil d'entrée : nb de lignes yes_ask+no_ask ≤ 0,99$ et leurs timestamps | 4 (t=88,255s ; 225,464s ; 231,015s ; 234,048s) | 4, mêmes timestamps, coût 0,99$ les 4 fois (bbo.csv, colonnes yes_ask/no_ask, 791 lignes complètes) | 0 | [CONFIRMÉ]
| 3 | Strike exact | 64 303,29737414$ | 64 303,29737414$ (oracle.csv, ligne 1, t_ms=0) | 0 | [CONFIRMÉ]
| 4 | Oracle sous le strike | 102/278 (36,7%) | 102/278 = 36,7% (oracle.csv, colonne price) | 0 | [CONFIRMÉ]
| 5 | Trade n°1 du journal : YES 0,32$ + NO 0,67$ à t=88,255s | 0,99$ | bbo.csv à t_ms=88255 : yes_ask=0,32, no_ask=0,67 | 0 | [CONFIRMÉ]
| 6 | Trade n°3 du journal : YES 0,68$ + NO 0,31$ à t=231,015s | 0,99$ | bbo.csv à t_ms=231015 : yes_ask=0,68, no_ask=0,31 | 0 | [CONFIRMÉ]


0 [ERREUR] sur 6 valeurs sondées : **données de base dignes de confiance au niveau sondé**. Le problème de cette stratégie n'est pas dans ses chiffres — il est dans ce qu'elle ignore.

## 3. PARTIE O — AUDIT OPÉRATIONNEL : CE QUE LA STRATÉGIE IGNORE

### O1 — LATENCE D'EXÉCUTION

**CE QUE LA STRATÉGIE SUPPOSE.** Journal C-SIM : timestamp d'entrée = timestamp de sortie pour les 4 trades (« t=88,255s → t=88,255s (fusion) »). Le pseudo-code C2 lit une ligne BBO et envoie l'ordre dans le même tick. Exécution instantanée, latence 0.

**CE QUI EST VRAI EN RÉALITÉ.** La boucle minimale depuis us-east-1 est : réception de l'événement BBO du CLOB Polymarket ~31 ms + décision (je la compte à 0) + envoi de l'ordre couple vers le CLOB ~31 ms = **62 ms** avant que l'ordre n'existe dans le carnet, hors confirmation. Or j'ai mesuré la durée de vie des 4 cotes croisées dans bbo.csv : chacune des 4 est suivie, **au même t_ms**, d'une ligne où le carnet est revenu à yes_ask+no_ask > 0,99$ (idx 134→135, 404→405, 484→485, 515→516). Durée de vie mesurable : **0 ms**, borne supérieure < 1 ms.

**IMPACT CHIFFRÉ.** J'ai rejoué le journal avec chaque ordre décalé de 62 ms : à t+62 ms, le coût de paire est 1,00$ ou 1,01$ pour les 4 événements (bbo.csv, lignes suivantes : 0,33+0,67=1,00 ; 0,69+0,31=1,00 ; 0,69+0,31=1,00 ; 0,68+0,32=1,00). Le signal d'entrée n'existe plus dans aucun des 4 cas. Trades exécutés : 0/4. P&L : **0,00$ au lieu de +0,20$ (−0,20$, −100%)**.

**VERDICT DU POINT : [DÉTRUIT].** La fenêtre d'opportunité est au moins 62 fois plus courte que la boucle d'exécution la plus rapide possible depuis cette infrastructure.

### O2 — FRAÎCHEUR DU SIGNAL

**CE QUE LA STRATÉGIE SUPPOSE.** Que la ligne BBO lue décrit le carnet à l'instant de la décision.

**CE QUI EST VRAI EN RÉALITÉ.** Le signal est le flux BBO du CLOB lui-même : il a déjà ~31 ms d'âge à l'arrivée (tableau des latences, ligne Polymarket CLOB). L'opportunité vit < 1 ms (bbo.csv, mesure O1). Le bot prend donc sa décision sur un carnet qui, dans 100% des 4 cas, a déjà été corrigé **avant même que le signal n'arrive**. Notez que la stratégie retenue n'utilise ni Binance (~219 ms) ni Chainlink (~68 ms) — ce point ne concerne que le flux CLOB, mais il suffit.

**IMPACT CHIFFRÉ.** Identique et confondu avec O1 dans la simulation cumulée : 0 signal encore valide à réception, 0/4 trades. Je ne le compte qu'une fois dans la Partie I pour ne pas doubler la pénalité de P&L.

**VERDICT DU POINT : [DÉTRUIT].** Le signal est mort avant d'être lu.

### O3 — SLIPPAGE ET PROFONDEUR

**CE QUE LA STRATÉGIE SUPPOSE.** Hypothèse n°3 du speech, déclarée : « profondeur ≥ q au BBO affiché (tailles absentes de B-F3) » — 5 paires, soit ~4,95$ notionnel.

**CE QUI EST VRAI EN RÉALITÉ.** bbo.csv ne contient aucune colonne de taille : la profondeur au ask aux 4 timestamps est **DONNÉE NON VÉRIFIABLE**. Pour 4,95$ sur un marché qui traite 16 432,62$ en 300 s (trades.csv), l'hypothèse est plausible mais indémontrable.

**IMPACT CHIFFRÉ.** Borné entre 0,00$ (profondeur suffisante) et −0,20$ (aucune profondeur au prix croisé, donc 0 fill) — méthode : les deux cas extrêmes possibles en l'absence de données de taille. L'agent a déclaré cette hypothèse ; il ne l'a pas ignorée.

**VERDICT DU POINT : [IMPACT NON CHIFFRABLE]** (borné ci-dessus, sans objet dès lors que O1 supprime déjà les fills).

### O4 — FRAIS COMPLETS

**CE QUE LA STRATÉGIE SUPPOSE.** Hypothèse n°2, déclarée : frais = 0,00$, y compris sur la fusion des paires.

**CE QUI EST VRAI EN RÉALITÉ.** La fusion YES+NO → 1,00$ est une opération on-chain sur Polygon (merge du Conditional Token Framework). Même si les frais de trading CLOB sont nuls, le gas de 4 transactions de merge n'est pas nul, et le tableau des latences confirme que l'infrastructure parle bien à Polygon (RPC ~108 ms). Coût de gas par merge : **DONNÉE NON VÉRIFIABLE** dans les sources ; je le borne entre 0,01$ et 0,05$ par transaction — méthode : fourchette des coûts de transaction Polygon pour un appel de contrat simple, appliquée aux 4 merges.

**IMPACT CHIFFRÉ.** Entre −0,04$ et −0,20$ sur un gain brut de +0,20$, soit **−20% à −100% du P&L annoncé**. Le mécanisme entier gagne 0,01$/paire : il est du même ordre de grandeur que son propre coût de règlement.

**VERDICT DU POINT : [DÉGRADE]** (et [DÉTRUIT] à la borne haute du gas).

### O5 — ORDRES CONCURRENTS ET FILE D'ATTENTE

**CE QUE LA STRATÉGIE SUPPOSE.** Règle (d) du speech : en cas de fill partiel, « la jambe orpheline est annulée et la jambe remplie est revendue immédiatement au bid de la même ligne », bid qui est supérieur à l'ask sur carnet croisé — dénouement supposé non perdant.

**CE QUI EST VRAI EN RÉALITÉ.** Deux faits contre cette règle. Premier fait : « le bid de la même ligne » n'existe plus à l'arrivée de l'ordre de revente (+31 ms minimum) ; à t=88,255s+1 ligne, le yes_bid est déjà 0,33 face à un ask 0,33 puis 0,34 — la revente se fait sur un carnet reconstruit, pas sur la ligne croisée. Deuxième fait : la structure même des 4 événements — bid et ask corrigés **dans la même milliseconde** — indique une publication séquentielle des deux côtés du carnet par le même teneur de marché (le bid bouge dans un message, l'ask dans le suivant). Il est probable que la cote croisée n'ait jamais existé comme état atomique frappable. Les premiers trades du fichier après chaque événement n'arrivent qu'à +75 à +97 ms (trades.csv : t=88 352, 225 554, 231 112, 234 123) : personne, pas même les acteurs les plus rapides de cette session, n'a frappé ces cotes.

**IMPACT CHIFFRÉ.** Scénario fill partiel sous latence réelle : achat d'une jambe à l'ask croisé, revente forcée au bid du carnet corrigé — perte de spread de 0,01$ à 0,02$ par paire, soit **jusqu'à −0,10$ par événement raté** (5 paires × 0,02$), là où le speech garantit « perte maximale possible 0,00$ ». La promesse centrale — risque zéro — est fausse dès que l'atomicité des deux jambes n'est pas garantie, et rien dans l'infrastructure ne la garantit.

**VERDICT DU POINT : [DÉTRUIT]** — pas seulement le P&L : l'argument de sélection de C-A (« perte max possible 0,00$ »), qui a servi à écarter les trois autres candidates, ne tient pas.

### O6 — DÉFAILLANCES TECHNIQUES

**CE QUE LA STRATÉGIE SUPPOSE.** Un verrou global et 4 règles de race condition (a)-(d), mais aucun timeout, aucun comportement défini pour un ordre non confirmé, un RPC muet ou une fusion qui échoue.

**CE QUI EST VRAI EN RÉALITÉ.** Le cycle réel comporte trois systèmes (CLOB ~31 ms, Polygon RPC ~108 ms pour le merge, Telegram ~67 ms si utilisé pour notifier — hors boucle critique dans ce design, sans impact). Un merge non confirmé laisse le capital immobilisé en paires YES+NO jusqu'au règlement : sur ce marché la paire vaut 1,00$ au règlement par construction, donc le coût d'une défaillance est un blocage de capital, pas une perte — sauf si une seule jambe est en portefeuille (cas O5), où le bot reste en position directionnelle aveugle sans règle de gestion.

**IMPACT CHIFFRÉ.** Borné entre 0,00$ (paires complètes portées au règlement) et −4,95$ (pire cas : 5 jambes orphelines du mauvais côté portées au règlement sans règle de sortie) — méthode : valeur de règlement 0$ ou 1$ d'un binaire sur la position maximale d'un trade.

**VERDICT DU POINT : [DÉGRADE]** (borne basse nulle, mais le pire cas non traité expose 99% du capital).

### O7 — CAPACITÉ ET PASSAGE À L'ÉCHELLE

**CE QUE LA STRATÉGIE SUPPOSE.** Rien : le speech ne revendique aucune montée en taille et étiquette la fréquence [SESSION-SPÉCIFIQUE].

**CE QUI EST VRAI EN RÉALITÉ.** À 5$ de capital, 5 paires par événement, face à 16 432,62$ de volume de session, la taille est négligeable. Preuve chiffrée du [SANS IMPACT] : même en supposant la profondeur minimale d'un seul lot au BBO, la question d'échelle ne modifie aucun des 4 trades de 4,95$ ; et comme O1 ramène de toute façon les fills à zéro, l'échelle est sans objet aux deux bornes.

**VERDICT DU POINT : [SANS IMPACT]** (à 5$ ; toute montée en taille rouvrirait O3, non vérifiable).

### O8 — DÉPENDANCE À LA SESSION

**CE QUE LA STRATÉGIE SUPPOSE.** Le speech le déclare lui-même : 4 occurrences en 300 s, concentrées entre t=225s et t=234s, étiquetées [SESSION-SPÉCIFIQUE] ; le mécanisme est [PATTERN CANDIDAT].

**CE QUI EST VRAI EN RÉALITÉ.** Confirmé dans bbo.csv : 3 des 4 événements tombent dans une fenêtre de 9 s pendant le re-pricing violent post-choc (t=225,464s à 234,048s), le 4e à t=88,255s. Session calme = 0 signal = 0 trade = 0,00$ de P&L, ce que la structure de la stratégie assume sans perte.

**IMPACT CHIFFRÉ.** Sur le P&L annoncé de cette session : 0,00$ — l'agent n'a rien extrapolé. Preuve du [SANS IMPACT] : aucune règle de la stratégie ne dépend d'une fréquence future, et le speech ne projette aucun rendement hors session.

**VERDICT DU POINT : [SANS IMPACT]** — c'est le facteur le mieux traité par l'agent.

## 4. PARTIE I — IMPACT CUMULÉ DES MANQUEMENTS

Simulation unique avec tout réintégré : boucle de 62 ms (O1/O2), fills perdus si le niveau a disparu à l'arrivée (O5), gas de merge sur les fills restants (O4), profondeur à sa borne défavorable (O3).

Résultat mécanique : les 4 signaux ont une durée de vie < 1 ms ; à t+62 ms le coût de paire est revenu à ≥ 1,00$ dans les 4 cas (bbo.csv, lignes 135, 405, 485, 516) ; **0 trade exécuté, donc 0 gas payé, 0 slippage subi**.

| Scénario | P&L annoncé | P&L corrigé | Écart %
|-----|-----|-----|-----|-----|-----
| O1+O2 seuls (latence de boucle 62 ms) | +0,20$ | 0,00$ | −100%
| O4 seul (gas de merge, si les fills existaient) | +0,20$ | entre 0,00$ et +0,16$ | −20% à −100%
| O5 seul (fill partiel sous latence, pire cas par événement) | +0,20$ | jusqu'à −0,40$ | −300%
| **TOUS MANQUEMENTS CUMULÉS** | **+0,20$** | **0,00$** | **−100%**


Le P&L cumulé corrigé est 0,00$ — non négatif, la règle bloquante stricte ne se déclenche pas, mais le mécanisme central (frapper une cote croisée) cesse de fonctionner : la stratégie ne perd pas d'argent, elle **ne trade pas**. Et son plancher réel n'est pas 0,00$ : tout fill partiel sous latence produit une perte (ligne O5), ce qui invalide le critère même qui a fait retenir C-A.

## 5. PARTIE R — RISQUES RÉSIDUELS NON DÉCLARÉS

- [RISQUE NON DÉCLARÉ] Concurrence de priorité : toute autre machine colocalisée plus près du CLOB (< 31 ms) passe devant systématiquement ; la stratégie est une course dont le speech ne mentionne jamais les autres coureurs. Impact : borné à la totalité des opportunités (−0,20$, 100% du gain espéré).
- [RISQUE NON DÉCLARÉ] Nature de l'anomalie : la correction bid/ask dans la même milliseconde sur les 4 événements (bbo.csv) est compatible avec un artefact de publication séquentielle du flux, auquel cas le profit n'a jamais existé pour personne. Non chiffrable au-delà : DONNÉE NON VÉRIFIABLE sans les numéros de séquence du flux.
- [RISQUE NON DÉCLARÉ] Risque de résolution/contrepartie : le pire cas d'une jambe orpheline (O6) suppose que le marché se règle normalement ; un litige de résolution oracle bloque 100% du capital engagé (5,00$). Probabilité non chiffrable sur ces sources.


## 6. AUTO-CONTRÔLE DE L'AUDITEUR

- Contrôle 1 → conforme : O1 à O8 tous traités en 4 temps ; les 2 [SANS IMPACT] (O7, O8) portent leur preuve chiffrée (taille 4,95$ vs volume 16 432,62$ ; 0$ d'extrapolation hors session).
- Contrôle 2 → conforme : boucle reconstituée chemin par chemin (CLOB 31 ms réception + CLOB 31 ms envoi = 62 ms ; Polygon RPC 108 ms pour le merge, hors boucle d'entrée) et utilisée telle quelle en Partie I.
- Contrôle 3 → conforme : chaque impact cite bbo.csv/oracle.csv/spot.csv/trades.csv ou une ligne du tableau des latences ; les 3 impacts non chiffrables (O3, gas unitaire, séquence de flux) sont bornés avec méthode.
- Contrôle 4 → conforme : F7 recalculée ci-dessous = 26/100, identique à l'ouverture ; 26 ∈ [0-44] → [NON FIABLE], cohérent avec la Partie I.
- Contrôle 5 → conforme : +0,20$, 0,00$, 62 ms, 4 événements, 26/100 identiques de l'ouverture à la clôture.


| Critère F7 | Points obtenus | Points max | Justification
|-----|-----|-----|-----|-----|-----
| Complétude opérationnelle | 0 | 40 | O1 [DÉTRUIT] −10, O2 [DÉTRUIT] −10, O5 [DÉTRUIT] −10, O4 [DÉGRADE] −5, O6 [DÉGRADE] −5
| Survie du P&L corrigé | 0 | 30 | P&L cumulé = 0,00$, non positif ; hors pire manquement (O1), O5 annule encore les fills
| Fiabilité des données de base | 20 | 20 | 6/6 valeurs sondées [CONFIRMÉ]
| Honnêteté des limites | 6 | 10 | −2 latence jamais mentionnée (connaissable), −2 concurrence/priorité de file non déclarée ; hypothèses n°1-3 et règle FRAGILE déclarées, à son crédit
| **TOTAL** | **26** | **100** | 26 → [NON FIABLE]


## 7. CLÔTURE

Verdict : [NON FIABLE], note 26/100 — l'analyse des données est honnête et exacte, mais la stratégie est structurellement inexécutable par ce bot sur cette infrastructure. Condition n°1 pour changer le verdict : démontrer, numéros de séquence du flux à l'appui, que les cotes croisées ont existé comme états frappables pendant une durée supérieure à la boucle d'exécution.

---

Pour répondre directement à votre question initiale : **non, le bot ne peut pas suivre cette stratégie** — la fenêtre d'opportunité (< 1 ms) est au moins 62 fois plus courte que sa boucle minimale (62 ms), et même le serveur idéalement placé en us-east-1 n'y change rien


