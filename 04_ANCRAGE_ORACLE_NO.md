# 04 — ANCRAGE ORACLE CÔTÉ NO

**Fichiers sources regroupés :** NO_ancré_à _l'oracle.md, 280_ANCRAGE-ORACLE_NO.md, 298NO-ORACLE-CONFIRMÉ.md

---

## Source : NO_ancré_à _l'oracle.md

# 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Voici mon verdict, avant toute démonstration.

- Stratégie retenue : **« NO ancré à l'oracle »** — achat du contrat NO quand la source de règlement (Chainlink) est durablement sous le strike alors que le marché price encore l'incertitude.
- Capital final : **9,62$** contre 5,00$ de départ, soit un rendement net de **+92,31%**.
- Nombre de trades : **1**, dont **1 gagnant, 0 perdant**.
- Critère de sélection : **bénéfice net en $ sur la session**, départage par nombre de pièces convergentes.
- Étiquette globale : **[SESSION-SPÉCIFIQUE]** — le mécanisme d'entrée est un pattern candidat, mais le gain final a tenu à moins de 0,30$ d'oracle dans les 3 dernières secondes.


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

# 1. CADRE D'ANALYSE CHOISI

J'adopte un cadre **structurel micro-marché** : je confronte la source de règlement (oracle Chainlink) au prix affiché du marché de prédiction, tick par tick. Ce cadre est adapté à ces données précises : 5 minutes, précision milliseconde, et surtout deux flux de prix distincts (spot Binance vs oracle) alors qu'un seul décide du règlement. La question centrale devient mesurable : le marché price-t-il la bonne source ?

# 2. PARTIE A — LES 6 GRAPHES

## A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET

**A1. IDENTITÉ.** Bid/ask/mid des contrats YES et NO plus probabilité consensus, 980 événements BBO, axe X en mm:ss.mmm de t=0,000s à t=300,000s, axe Y en $ de 0,0 à 1,0. Vue 300 s + 3 zooms 15 s (00:00–00:15, 03:25–03:40, 04:06–04:21).

**A2. QUESTION.** Que fait le prix des contrats quand le temps passe ? Où sont les ruptures de niveau et à quelle vitesse s'exécutent-elles ?

**A3. COMMENT.** Lecture des étiquettes d'extrema sur la vue 300 s, puis lecture séquentielle des étiquettes des zooms pour dater les ruptures à la milliseconde.

**A4. VALEURS EXTRAITES.**

- t=1,102s : YES bid = 0,41$
- t=1,101s : NO = 0,58$
- t=212,931s : YES = 0,52$ (chute depuis 0,69$ en 113 ms, zoom 03:25–03:40)
- t=263,775s : YES = 0,04$
- t=296,527s : YES = 0,98$
- t=298,640s : YES = 0,98$


**A5. OBSERVATIONS BRUTES.**

- Le marché ouvre proche de l'équilibre (YES 0,41$ / NO 0,58$ à t=1,102s) et y reste toute la première minute.
- Les ruptures sont quasi instantanées : YES passe de 0,68$ à 0,52$ entre t=212,818s et t=212,931s (113 ms).
- La dérive vers NO est graduelle de t=120s à t=260s : YES 0,42$ → 0,04$.


**A6. SPÉCIFIQUE À LA SESSION.** Le retournement final YES 0,04$ → 0,98$ entre t=263,775s et t=296,527s, alors que le résultat officiel est DOWN : le marché a terminé du mauvais côté à 0,98$. C'est un accident de fin de fenêtre, non reproductible comme règle.

## A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK

**A1. IDENTITÉ.** Deux séries de prix BTC en $ : Binance BTC/USDT (3109 ticks) et Chainlink BTC/USD (286 ticks), ligne de strike à 64 320$, sous-graphe basis = spot − oracle. Période t=0,000s à t=298,363s.

**A2. QUESTION.** Les deux sources de prix racontent-elles la même histoire par rapport au strike ?

**A3. COMMENT.** Comparaison de la position de chaque série par rapport à la ligne de strike, lecture des étiquettes d'extrema, lecture de l'échelle du sous-graphe basis.

**A4. VALEURS EXTRAITES.**

- t=0,172s : spot = 64 368,64$
- t=0,000s : oracle = 64 315,83$
- t=263,953s : spot = 64 356,00$ (minimum spot, étiqueté 04:23.953)
- t=264,000s : oracle = 64 301,10$ (minimum oracle)
- t=298,363s : spot = 64 374,43$
- Sous-graphe basis : échelle 0$ à 60$, jamais négative.


**A5. OBSERVATIONS BRUTES.**

- Le spot est au-dessus du strike 64 320$ sur toute la fenêtre visible ; l'oracle est en dessous sur la quasi-totalité.
- Le basis spot−oracle est massif et stable (bande visible ≈ 50$–60$).
- Les deux sources donnent des verdicts OPPOSÉS sur ce marché : spot ⇒ UP, oracle ⇒ DOWN.


**A6. SPÉCIFIQUE À LA SESSION.** L'oracle remonte à 64 320,29$ à t=297,000s, soit au niveau du strike affiché, dans les 3 dernières secondes. Le résultat DOWN implique que le strike exact est supérieur à 64 320,29$ ; sa valeur exacte : DONNÉE NON DISPONIBLE. Dénouement à quelques centimes, non reproductible.

## A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ

**A1. IDENTITÉ.** Volume 30 s glissant en $ des flux haussier (BUY YES + SELL NO) et baissier (SELL YES + BUY NO, en négatif), 1447 trades, t=0,000s à t=299,872s, axe Y −2000$ à +2000$.

**A2. QUESTION.** Quand et dans quel sens l'argent entre-t-il ? Les gros ordres précèdent-ils les mouvements de prix ?

**A3. COMMENT.** Lecture des extrema étiquetés sur la vue 300 s, puis des rafales dans les zooms.

**A4. VALEURS EXTRAITES.**

- t=2,120s : +16,10$
- t=13,203s : +297,11$ (plus gros flux haussier)
- t=93,579s : −271,27$ (plus gros flux baissier)
- t=215,867s : −175,54$ (zoom 03:25–03:40)
- t=246,198s : −200,66$ (zoom 04:06–04:21)
- t=299,753s : −18,75$


**A5. OBSERVATIONS BRUTES.**

- Les rafales baissières de t=254,047s à t=260,358s (zoom : −70,40$, −90,00$, −68,00$, −75,20$...) accompagnent la chute de YES vers 0,10$ (A-G1, A4).
- Aucun flux ne dépasse 300$ en valeur absolue : marché mince.


**A6. SPÉCIFIQUE À LA SESSION.** Le pic +297,11$ à t=13,203s est un trade isolé sans suite de tendance (YES reste ≈0,44$ deux minutes après, A-G1) : bruit.

## A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ.** Ratio haussier/(haussier+baissier) sur les 1447 trades, ligne d'équilibre à 0,5, t=0,000s à t=299,872s, axe Y de 0,0 à 1,0.

**A2. QUESTION.** L'agressivité d'un camp domine-t-elle durablement ?

**A3. COMMENT.** Lecture des extrema étiquetés et de la position vs la ligne 0,5 dans chaque zoom.

**A4. VALEURS EXTRAITES.**

- t=1,249s : 0,00
- t=13,989s : 0,80 (maximum)
- t=206,198s : 0,27 (zoom 04:06, étiquette 04:06.198)
- t=260,950s : 0,16 (minimum de zoom, étiquette 04:20.950)
- t=299,872s : 0,62


**A5. OBSERVATIONS BRUTES.**

- L'imbalance oscille autour de 0,5 sans jamais s'installer durablement au-delà de 0,8 ou sous 0,16.
- Dans la fenêtre 04:06–04:21, l'imbalance tombe de 0,32 (t=249,613s) à 0,16 (t=260,950s) pendant que YES chute de 0,24$ à 0,06$ (A-G1) : le flux suit le prix plus qu'il ne le précède.


**A6. SPÉCIFIQUE À LA SESSION.** La valeur 0,00 à t=1,249s reflète seulement le premier trade (fenêtre 30 s vide avant lui) : artefact de démarrage.

## A-G5 — GRAPHE 5 : SPREADS YES ET NO

**A1. IDENTITÉ.** Spread ask−bid des deux contrats, événementiel, 980 évts, t=0,000s à t=298,640s, axe Y −0,2$ à +0,5$.

**A2. QUESTION.** Combien coûte l'exécution, et quand devient-elle impossible ?

**A3. COMMENT.** Lecture des extrema et des paliers dans les zooms.

**A4. VALEURS EXTRAITES.**

- t=1,102s : 0,01$
- t=212,910s : 0,13$ (pic du zoom 03:25–03:40)
- t=253,891s : 0,13$ (pic du zoom 04:06–04:21)
- t=290,778s : 0,52$ (maximum de session)
- t=291,489s : −0,20$ (spread négatif = carnet croisé)
- t=298,640s : 0,01$


**A5. OBSERVATIONS BRUTES.**

- Régime normal : spread 0,01$–0,02$ sur l'essentiel de la session.
- Chaque choc de prix multiplie le spread par 6 à 13 (0,13$) pendant moins d'une seconde, puis retour à 0,01$–0,02$.
- En fin de session le carnet devient inexploitable : 0,52$ de spread à t=290,778s.


**A6. SPÉCIFIQUE À LA SESSION.** Le spread négatif −0,20$ à t=291,489s (carnet croisé) appartient au chaos final ; fenêtre de quelques millisecondes.

## A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ.** yes_mid − (1 − no_mid), 980 évts, 30 points suspects marqués, ligne de cohérence parfaite à 0, t=0,000s à t=298,640s, axe Y −0,16$ à 0,00$.

**A2. QUESTION.** Les deux carnets racontent-ils le même prix ? Existe-t-il des fenêtres d'incohérence exploitables ?

**A3. COMMENT.** Lecture de l'échelle et des marqueurs ; ce graphe ne porte aucune étiquette de valeur individuelle, je m'appuie sur ses métadonnées et je renvoie les valeurs exactes au CSV (B-F1, B4).

**A4. VALEURS EXTRAITES.**

- Échelle des écarts : bornée entre −0,16$ et 0,00$ (jamais positive)
- Nombre d'événements : 980
- Points incohérents marqués : 30
- ts fallback : 0/286 (0,0%)
- Ligne de référence : 0,00$
- Valeurs individuelles étiquetées : DONNÉE NON DISPONIBLE sur ce graphe (calculées en B-F1)


**A5. OBSERVATIONS BRUTES.**

- L'écart inter-carnets est nul ou négatif : quand les carnets divergent, la somme des prix implicites est INFÉRIEURE à 1$ — configuration d'achat des deux jambes.
- 30 incohérences sur 980 évts (3,1%) : rares mais récurrentes.


**A6. SPÉCIFIQUE À LA SESSION.** L'amplitude −0,16$ correspond au chaos de t=291,489s (cf. A-G5) ; en régime calme l'écart reste collé à 0.

Vous venez de voir ce que les graphes montrent ; les CSV vont maintenant chiffrer chaque intuition au tick près.

# 3. PARTIE B — LES FICHIERS CSV

## B-F1 — bbo.csv

**B1. IDENTITÉ.** Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask — toutes utilisées. 980 lignes de données, dont 977 complètes et 3 sans côté YES (dont la première, t=1,101s). Période t=1,101s → t=299,161s, échantillonnage événementiel (à chaque changement de BBO).

**B2. APPORT.** Les valeurs exactes de spread, d'écart inter-carnets et de croisements que le graphe 6 ne permet pas de lire.

**B3. COMMENT.** Moyenne/médiane/min/max des spreads (ask−bid) ; écart inter-carnets yes_mid−(1−no_mid) ; comptage des lignes où yes_ask+no_ask`<1 et yes_bid+no_bid>`1 ; consensus (yes_mid+(1−no_mid))/2.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Spread YES moyen | 0,0338$ | yes_ask, yes_bid | 977
| Spread YES médian | 0,02$ | yes_ask, yes_bid | 977
| Spread YES max | 0,52$ (t=290,778s) | yes_ask, yes_bid | 977
| Évts spread ≥ 0,05$ | 161 | yes_ask, yes_bid | 977
| Évts spread ≥ 0,10$ | 53 | yes_ask, yes_bid | 977
| Écart inter-carnets min | −0,155$ (t=291,489s) | les 4 colonnes prix | 977
| Évts yes_ask+no_ask < 1$ | 14 (edge max 0,20$ à t=291,489s) | yes_ask, no_ask | 977
| Évts yes_bid+no_bid > 1$ | 13 | yes_bid, no_bid | 977
| Consensus min / max | 0,035 (t=263,775s) / 0,985 (t=296,527s) | les 4 colonnes prix | 977


**B5. OBSERVATIONS BRUTES.**

- 14 fenêtres d'arbitrage à l'achat des deux jambes, la première dès t=14,354s (somme 0,99$, edge 0,01$).
- Hors chocs, le coût d'un aller-retour est de 0,02$ par contrat (spread médian).
- NO ask = 0,52$ encore à t=60,642s, soit une minute après l'ouverture.


**B6. SPÉCIFIQUE À LA SESSION.** 9 des 14 croisements se concentrent entre t=290,921s et t=294,117s (chaos final) ; les edges ≥ 0,04$ n'existent que là et à t=214,456s (0,11$). Verdict : signal réel mais fenêtres de millisecondes — exploitable seulement en petit.

## B-F2 — oracle.csv

**B1. IDENTITÉ.** Colonnes t_ms, price, ts_src — toutes utilisées. 286 lignes, t=0,000s → t=298,000s, pas nominal 1 000 ms.

**B2. APPORT.** La position exacte de la source de règlement par rapport au strike, tick par tick — invisible à cette précision sur le graphe 2.

**B3. COMMENT.** Min/max/premier/dernier de price ; comptage price < 64 320 ; écarts temporels entre ticks consécutifs ; comptage ts_src ≠ payload.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Ticks sous le strike 64 320$ | 284 / 286 (99,3%) | price | 286
| Premier tick | 64 315,83$ (t=0,000s) | t_ms, price | 1
| Minimum | 64 301,10$ (t=264,000s) | t_ms, price | 286
| Maximum | 64 320,29$ (t=297,000s) | t_ms, price | 286
| Dernier tick | 64 320,29$ (t=298,000s) | t_ms, price | 1
| ts fallback | 0 / 286 | ts_src | 286
| Pas moyen | 1 045,6 ms | t_ms | 285
| Trous > 1,5 s | 12 (max 3 000 ms, t=283,000s→286,000s) | t_ms | 285


**B5. OBSERVATIONS BRUTES.**

- La source de règlement est sous le strike sur 284 ticks sur 286 ; les 2 seuls ticks ≥ 64 320$ sont t=297,000s et t=298,000s.
- De t=0s à t=296s, l'oracle ne s'approche jamais à moins de 3,69$ du strike (max intermédiaire 64 316,31$ hors fin de session : DONNÉE NON DISPONIBLE au-delà de ce qui précède ; le max global est en fin de session).


**B6. SPÉCIFIQUE À LA SESSION.** La remontée terminale +19,19$ en 8 s (64 301,10$ à t=264,000s → 64 320,29$ à t=297,000s vue sur les derniers ticks) : signal réel mais résolution du marché à moins de 0,30$ près — c'est le point de fragilité de toute la session.

## B-F3 — spot.csv

**B1. IDENTITÉ.** Colonnes t_ms, price — toutes utilisées. 3 109 lignes, t=0,172s → t=298,363s, flux tick par tick.

**B2. APPORT.** La mesure exacte du basis spot−oracle, que le graphe 2 ne donne qu'en bande.

**B3. COMMENT.** Min/max ; comptage price < 64 320 ; basis = spot − dernier tick oracle ≤ t (recherche par bissection), puis moyenne/médiane/min/max.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Ticks sous le strike | 0 / 3 109 | price | 3 109
| Minimum | 64 356,00$ (t=263,953s) | t_ms, price | 3 109
| Maximum | 64 374,43$ (t=298,363s) | t_ms, price | 3 109
| Premier / dernier | 64 368,64$ / 64 374,43$ | t_ms, price | 2
| Basis moyen | +53,18$ | price + oracle.price | 3 109
| Basis médian | +53,39$ | price + oracle.price | 3 109
| Basis min | +48,19$ (t=212,695s) | price + oracle.price | 3 109
| Basis max | +59,87$ (t=293,672s) | price + oracle.price | 3 109


**B5. OBSERVATIONS BRUTES.**

- Le basis ne devient jamais négatif ni même inférieur à +48,19$ : les deux flux ne convergent à aucun moment.
- Le spot, pris seul, aurait donné 100% de ticks côté UP — c'est la source qu'il fallait IGNORER pour trader ce marché.


**B6. SPÉCIFIQUE À LA SESSION.** L'amplitude du basis (48$–60$) est propre à ce couple de flux ce jour-là ; sa valeur absolue est session-spécifique, sa stabilité est le signal.

## B-F4 — trades.csv

**B1. IDENTITÉ.** Colonnes t_ms, usd, direction — toutes utilisées. 1 447 lignes, t=1,249s → t=299,872s.

**B2. APPORT.** Les volumes exacts par camp et par minute, que les graphes 3 et 4 n'affichent qu'en extrema.

**B3. COMMENT.** Sommes de usd par direction (globale, par minute de 60 000 ms) ; moyenne/médiane ; maximum.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Volume total | 15 410,04$ | usd | 1 447
| Trades haussiers | 627 pour 7 352,35$ | usd, direction | 627
| Trades baissiers | 820 pour 8 057,69$ | usd, direction | 820
| Part baissière du volume | 52,29% | usd, direction | 1 447
| Plus gros trade | 297,11$ haussier (t=13,203s) | t_ms, usd, direction | 1
| Trade moyen / médian | 10,65$ / 3,80$ | usd | 1 447
| Imbalance par minute (min 0→4) | 0,577 / 0,408 / 0,510 / 0,528 / 0,427 | t_ms, usd, direction | 1 447
| Dernière minute | 644 trades ; 2 668,34$ haussier vs 3 579,93$ baissier | t_ms, usd, direction | 644


**B5. OBSERVATIONS BRUTES.**

- Le trade médian fait 3,80$ : ce marché est dominé par de très petits ordres, un ordre de 5,00$ est dans la norme et n'impacte pas le carnet.
- Aucune minute ne dépasse 0,577 d'imbalance : le flux ne fournit jamais de signal directionnel franc.


**B6. SPÉCIFIQUE À LA SESSION.** 44,5% des trades (644/1447) se concentrent dans la dernière minute : structure d'expiration, attendue sur ce type de marché, mais les prix qui l'accompagnent sont propres au dénouement de cette session.

Vous connaissez maintenant chaque pièce ; voici les stratégies que ces faits engendrent, et celle que les chiffres retiennent.

# 4. PARTIE C

## C0 — STRATÉGIES CANDIDATES

Les 5 observations les plus fortes du dossier :

- **OBS-1** : l'oracle, source de règlement, est sous le strike sur 284/286 ticks (B-F2, B5).
- **OBS-2** : le basis spot−oracle reste entre +48,19$ et +59,87$ et le spot n'est jamais sous le strike (0/3 109) — les deux sources donnent des verdicts opposés (B-F3, B5).
- **OBS-3** : le marché price encore NO ask = 0,52$ à t=60,642s, une minute après l'ouverture, alors que l'oracle est 9$ sous le strike (B-F1, B5 ; A-G1, A5).
- **OBS-4** : 14 fenêtres où yes_ask+no_ask < 1$, première à t=14,354s avec un edge de 0,01$ (B-F1, B5).
- **OBS-5** : l'imbalance de flux ne dépasse jamais 0,577 par minute — le flux suit le prix, il ne le précède pas (B-F4, B5 ; A-G4, A5).


**Candidate A — « NO ancré à l'oracle »** (OBS-1, OBS-2, OBS-3). Logique : le règlement dépend de l'oracle, pas du spot ; tant que l'oracle est durablement sous le strike, tout prix NO proche de 0,50$ est une erreur de pricing. Règles : entrée au premier événement BBO avec t ≥ 60s, oracle < strike−5$ sur les 60 s précédentes, NO ask ≤ 0,60$ ; tenue jusqu'au règlement. Simulation sommaire : 1 trade, P&L net **+4,62$**.

**Candidate B — « Arbitrage carnets croisés »** (OBS-4). Logique : quand yes_ask+no_ask < 1$, acheter les deux jambes verrouille 1$ au règlement quel que soit le résultat. Règles : entrée au premier événement avec somme < 1$, tout le cash, tenue jusqu'au règlement. Simulation sommaire : 1 entrée (t=14,354s, somme 0,99$), P&L net **+0,05$**. Même en visant le meilleur edge de la session (0,20$ à t=291,489s), le plafond serait +1,25$.

**Candidate C — « Suivi de flux 30 s »** (OBS-5). Logique : suivre l'agression dominante ; flux net 30 s ≤ −200$ ⇒ acheter NO, flux net ≥ +100$ ⇒ solder. Simulation sommaire : 3 trades (entrées t=43,247s à 0,55$, t=81,086s à 0,61$, t=236,013s à 0,70$ ; sorties à 0,50$, 0,45$, 0,02$), P&L net **−4,90$**.

CRITÈRE DE SÉLECTION : bénéfice net en $ sur la session ; en cas d'écart inférieur à 0,50$, départage par le nombre de pièces sources convergentes. Critère fixé avant lecture du tableau.

| Stratégie | Observations sources | Nb trades | P&L net | Verdict
|-----|-----|-----|-----
| A — NO ancré à l'oracle | OBS-1, OBS-2, OBS-3 | 1 | +4,62$ | **RETENUE**
| B — Arbitrage carnets croisés | OBS-4 | 1 (2 jambes) | +0,05$ | Écartée
| C — Suivi de flux 30 s | OBS-5 | 3 | −4,90$ | Écartée


Décision : A domine avec +4,62$ contre +0,05$ (B) et −4,90$ (C) — écart supérieur à 0,50$, pas de départage nécessaire. B est saine mais son edge disponible au premier signal est de 0,01$ par paire (B-F1, B4) ; C détruit 98,08% du capital, cohérent avec OBS-5 : le flux ne précède rien.

## C1 — TABLE DE CONVERGENCE

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| R1 : l'oracle est la référence, pas le spot | A-G2 (verdicts opposés), B-F2 (284/286 sous strike), B-F3 (basis +48$ à +60$) | aucune | SOLIDE (3 sources)
| R2 : entrée si oracle < strike−5$ depuis 60 s ET NO ask ≤ 0,60$ | B-F2 (oracle à 64 311$ à t=60,000s), B-F1 (NO ask 0,52$ à t=60,642s), A-G1 (YES stable ≈0,48$) | aucune | SOLIDE (3 sources)
| R3 : tenue jusqu'au règlement, aucun stop | A-G5 (spread 0,52$ à t=290,778s : stop inexécutable), B-F1 (NO bid 0,01$ à t=296,527s : stop = ruine) | B-F2 (les 2 derniers ticks franchissent 64 320$ : un stop aurait « protégé » — à perte) | SOLIDE (2 sources pour, 1 nuance)
| R4 : taille = 100% du cash disponible | B-F4 (trade médian 3,80$ : 5,00$ passe sans impact) | aucune | [FRAGILE - 1 SOURCE]


Je le dis clairement : R4 ne tient qu'à une seule pièce, je la marque FRAGILE.

## C2 — PSEUDO-CODE EXÉCUTABLE

```plaintext
STRIKE = 64320.00        # A-G2, A4 (strike affiché)
CAPITAL = 5.00
etat = FLAT

POUR CHAQUE evenement_bbo(t, yes_bid, yes_ask, no_bid, no_ask):
    SI etat == FLAT ET t >= 60000 ms:                    # B-F2 B4 (pas oracle 1 s : 60 s = 60 ticks)
        fenetre = ticks_oracle dans [t-60000, t]
        SI tous(prix < STRIKE - 5.00 pour prix dans fenetre)   # B-F2 B4 (oracle 64 311$ a t=60s)
           ET no_ask <= 0.60:                            # B-F1 B5 (NO ask 0,52$ a t=60,642s)
            qty = CAPITAL / no_ask                       # [HYPOTHÈSE n°2] fills fractionnaires
            ACHETER NO qty au prix no_ask                # frais 0,00$ [HYPOTHÈSE n°1]
            cash = 0.00 ; etat = LONG_NO

SI etat == LONG_NO A t = 300000 ms:
    REGLEMENT: NO paie 1,00$ si resultat = DOWN          # PDF, titre : "Résultat : ▼ DOWN"
    cash = qty * 1.00                                    # [HYPOTHÈSE n°3] reglement a la valeur faciale

# AUCUN STOP: justification A-G5 A4 (spread 0,52$ a t=290,778s) et B-F1 B4 (NO bid 0,01$ a t=296,527s)
```

## C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

**ÉTAT DU PORTEFEUILLE.**

- t=0,000s : cash 5,00$ ; positions : aucune ; valeur totale 5,00$.
- t=60,642s : signal validé (oracle 64 311,31$ à t=60,000s, soit 8,69$ sous strike, sous strike−5$ sur les 60 ticks précédents (B-F2) ; NO ask 0,52$ (B-F1)). Achat 9,6154 contrats NO à 0,52$ = 5,00$. Cash 0,00$ ; position 9,6154 NO @ 0,52$ ; valeur au bid (0,50$) = 4,81$.
- t=263,775s : valeur de position au bid maximale : 9,6154 × 0,96$ = 9,23$. Cash 0,00$.
- t=296,461s : valeur de position au bid minimale : 9,6154 × 0,01$ = 0,10$. Cash 0,00$.
- t=300,000s : règlement DOWN, NO = 1,00$. Cash 9,62$ ; positions : aucune.


**RACE CONDITIONS.**
(a) Signal d'achat pendant un ordre en cours : RÈGLE : un verrou global par actif interdit tout second ordre tant que le premier n'est pas confirmé ou rejeté ; le second signal est rejeté, pas mis en file. Occurrences dans cette simulation : 0 (état FLAT interdit toute ré-entrée après t=60,642s).
(b) Signal de vente sur position non confirmée : RÈGLE : aucun ordre de vente n'est émis tant que le fill d'achat n'est pas confirmé ; le signal de vente est rejeté. Occurrences : 0 (la stratégie ne vend jamais avant règlement).
(c) Deux signaux simultanés sur le même actif : RÈGLE : les événements sont traités en file FIFO stricte par t_ms croissant, puis par ordre d'apparition dans le fichier à t_ms égal ; un seul signal peut armer le verrou. Occurrences : 0.
(d) Fill partiel : RÈGLE : en cas de fill partiel, la quantité non exécutée est annulée immédiatement (IOC) et la position est celle réellement remplie ; aucun re-envoi. Occurrences supposées : 0, car l'ordre de 5,00$ est supérieur de 1,20$ seulement au trade médian de 3,80$ (B-F4, B4) [HYPOTHÈSE n°2 : profondeur suffisante au ask affiché].

**JOURNAL DE TRADES.**

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=60,642s | 0,52$ | 9,6154 NO | t=300,000s (règlement) | 1,00$ | 0,00$ | +4,62$ | 9,62$


**RÉSULTATS.**

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 1 / +4,62$ | ligne 1
| Trades perdants / pertes cumulées | 0 / 0,00$ | —
| Frais totaux | 0,00$ (ligne 1 : 0,00$) | ligne 1
| BÉNÉFICE NET | +4,62$ | ligne 1
| Capital final vs initial | 9,62$ vs 5,00$ (+92,31%) | ligne 1
| Pire perte unitaire | aucune (0 trade perdant) | —
| Drawdown maximum (latent) | 9,13$, de t=263,775s (9,23$) à t=296,461s (0,10$) | ligne 1, valorisation au NO bid


Regardez ce drawdown : la position est passée de 9,23$ à 0,10$ de valeur liquidative dans les 33 dernières secondes avant de payer 9,62$ au règlement. Le gain est réel ; le chemin est brutal.

## C4 — PORTÉE

- Confronter systématiquement le prix du marché à la source de règlement, pas au spot le plus visible. [PATTERN CANDIDAT]
- Un basis stable de +48$ à +60$ entre spot et oracle sur 5 minutes. [SESSION-SPÉCIFIQUE]
- NO à 0,52$ une minute après l'ouverture avec oracle 8,69$ sous strike : erreur de pricing exploitable. [PATTERN CANDIDAT]
- Le règlement DOWN acquis à moins de 0,30$ d'oracle près dans les 3 dernières secondes. [SESSION-SPÉCIFIQUE]
- Les fenêtres d'arbitrage carnets croisés existent mais offrent 0,01$ d'edge en régime calme. [PATTERN CANDIDAT]
- Le suivi de flux détruit le capital sur ce marché (imbalance jamais > 0,577). [SESSION-SPÉCIFIQUE]
- Le rendement de +92,31% lui-même, dépendant du dénouement à un cheveu. [SESSION-SPÉCIFIQUE]


# 5. AUTO-CONTRÔLE FINAL

- Contrôle 1 (volumétrie) → 6 valeurs ou plus par graphe (G6 : 5 métriques lisibles + 1 DONNÉE NON DISPONIBLE explicite), 8 ou plus par CSV dans chaque tableau B4. CONFORME.
- Contrôle 2 (traçabilité C→A4/B4) → chaque règle de C1/C2 cite sa rubrique ; les P&L de C0 sortent des simulations sur B-F1/B-F2/B-F4. CONFORME.
- Contrôle 3 (unicité des chiffres) → 4,62$ / 9,62$ / 92,31% / 0,52$ / t=60,642s identiques en ouverture, corps et clôture. CONFORME.
- Contrôle 4 (comptabilité) → 5,00$ − 5,00$ (achat) + 9,6154 × 1,00$ = 9,62$ au centime, égal au capital final annoncé. CONFORME.
- Contrôle 5 (hypothèses) → 3 [HYPOTHÈSE] : frais nuls, fills fractionnaires au ask, règlement NO à 1,00$. Limite de 3 respectée.
- Contrôle 6 (antériorité des observations) → R1←OBS-1/OBS-2, R2←OBS-3, R3←A-G5 A5/B-F1 B4, R4←B-F4 B5 ; toutes antérieures à la Partie C. CONFORME.
- Contrôle 7 (format speech) → verdict en ouverture, ordre F1 respecté, tableaux F4 présents, clôture sans chiffre nouveau. CONFORME.


# 6. CLÔTURE

Bénéfice net : +4,62$, capital final 9,62$ contre 5,00$ de départ, en un seul trade.
Le mécanisme — pricer contre la source de règlement — mérite d'être retesté ; le gain de cette session, lui, a tenu à trois secondes et quelques centimes d'oracle.
Étiquette globale : [SESSION-SPÉCIFIQUE].






RAPPORT D'AUDIT — STRATÉGIE « NO ANCRÉ À L'ORACLE » (SESSION BTC 5MIN, CAPITAL 5,00$)

---

**0. VERDICT EXÉCUTIF**

- Verdict global : **[FIABLE]**
- Note de fiabilité : **93/100** (grille F7)
- Manquements identifiés : 8 facteurs audités, dont **0 [DÉTRUIT]**, **2 [DÉGRADE]** (O3, O5), **6 [SANS IMPACT]**
- Manquement le plus grave : la profondeur au ask n'est pas vérifiable dans les sources (bbo.csv sans tailles) ; un fill partiel de 3,80$ à 0,54$ ramènerait le P&L à +3,24$, soit −29,9% du P&L annoncé.
- P&L annoncé : **+4,62$** ; P&L après réintégration de tous les manquements chiffrables : **+4,26$** (borne basse +3,24$).
- Point le plus solide : la fenêtre d'entrée à 0,52$ survit à toutes les latences réelles de l'infrastructure — le niveau reste affiché 164 ms après l'événement (bbo.csv, lignes t=60642 à t=60806), au-delà de la boucle complète oracle→CLOB de 99 à 139 ms.


Je vais vous montrer pièce par pièce d'où vient ce verdict.

---

**1. PÉRIMÈTRE**

Audité : la stratégie Candidate A du speech (section C0/C3), ses règles d'entrée (C2/R2), sa simulation (C3) et ses 3 hypothèses déclarées, confrontées à oracle.csv, spot.csv, bbo.csv, trades.csv et au tableau des latences us-east-1. Non audité : les candidates B et C (rejetées par l'agent lui-même), et le PDF au-delà des valeurs recoupées par les CSV.

---

**2. PARTIE V — SONDAGE DE FIABILITÉ DES DONNÉES**

| # | Valeur sondée | Annoncée | Recalculée (source) | Écart | Verdict
|-----|-----|-----|-----|-----|-----
| 1 | P&L final et cash final | +4,62$ / 9,62$ | 5,00/0,52 = 9,6154 contrats × 1,00$ = 9,6154$ ; P&L +4,6154$ (bbo.csv t=60642, no_ask=0,52) | 0,00$ | [CONFIRMÉ]
| 2 | Oracle à t=60,000s (règle R2) | 64 311,31$, marge 8,69$ | 64 311,3073$, marge 8,69$ (oracle.csv, t_ms=60000) | 0,00$ | [CONFIRMÉ]
| 3 | Fenêtre 60 s < strike−5$ avant entrée | validée | 56 ticks dans [0,642s ; 60,642s], max 64 313,85$ < 64 315$ (oracle.csv) | néant | [CONFIRMÉ]
| 4 | Oracle sous strike 284/286 ; spot 0/3 109 | 284/286 ; 0/3 109 | 284/286 ; 0/3 109 (oracle.csv, spot.csv, colonne price) | néant | [CONFIRMÉ]
| 5 | Trade n°1 : NO ask 0,52$ à t=60,642s | 0,52$ | 0,52$ (bbo.csv, ligne t_ms=60642, no_ask) | 0,00$ | [CONFIRMÉ]
| 6 | Basis borné +48,19$ / +59,87$ | +48,19$ à +59,87$ | min 48,19$, max 59,87$, moyenne 53,18$ (spot.csv − oracle.csv apparié) | 0,00$ | [CONFIRMÉ]


0 [ERREUR] sur 6 sondages : données de base dignes de confiance au niveau sondé. Je passe au cœur de l'audit.

---

**3. PARTIE O — AUDIT OPÉRATIONNEL : CE QUE LA STRATÉGIE IGNORE**

| # | Facteur ignoré | Hypothèse implicite de l'agent | Réalité (source) | Impact sur P&L | Verdict
|-----|-----|-----|-----|-----|-----
| O1 | Latence d'exécution | Exécution instantanée au no_ask de t=60,642s | Boucle oracle on-chain 68 ms (ou RPC 108 ms) + CLOB 31 ms = 99 à 139 ms ; le ask 0,52$ tient 164 ms (bbo.csv t=60642→60806) | 0,00$ à ces latences ; −0,18$ (−3,9%) seulement si la boucle dépasse 164 ms | [SANS IMPACT]
| O2 | Fraîcheur du signal | Oracle lu à l'instant t | Cadence oracle 1 tick/s + 68 ms de lecture = signal âgé jusqu'à 1,07 s ; plus grand saut 1-tick de la session : 4,00$ (oracle.csv, t=213s) contre une marge de 8,69$ | 0,00$ : même le pire saut ne peut pas invalider la condition strike−5$ | [SANS IMPACT]
| O3 | Slippage et profondeur | Profondeur suffisante pour 5,00$ au ask ([HYPOTHÈSE n°2]) | bbo.csv ne contient AUCUNE taille : profondeur DONNÉE NON VÉRIFIABLE ; trades exécutés autour de l'entrée : 0,60$ à 2,08$ (trades.csv, t=60348 à 60855), médiane session 3,80$ | Borné entre 0,00$ et −1,38$ (−29,9%) : fill partiel 3,80$ à 0,54$ → +3,24$ (méthode : plus gros fill plausible = trade médian, chase au ask suivant) | [DÉGRADE]
| O4 | Frais complets | Frais 0,00$ ([HYPOTHÈSE n°1]), pas de gas | Frais CLOB et gas de règlement : DONNÉE NON VÉRIFIABLE dans les sources ; hypothèse déclarée par l'agent | Borné entre 0,00$ et −0,10$ (−2,2%) (méthode : coût de gas Polygon d'une redemption relayée) | [SANS IMPACT]
| O5 | Ordres concurrents / file | Le niveau 0,52$ existe encore à l'arrivée de l'ordre (+31 ms) | Le niveau tient 164 ms puis monte : 0,53$ à t=60,806s, 0,54$ à t=60,858s, 0,55$ à t=61,598s (bbo.csv) ; l'agent a défini verrou + IOC (C3-d) mais pas de re-pricing | Si le niveau disparaît : re-fill à 0,53$ → +4,43$ (−3,9%) ou 0,54$ → +4,26$ (−7,7%) | [DÉGRADE]
| O6 | Défaillances techniques | Règles (a)-(d) définies ; Telegram hors boucle critique | Aucun ordre de sortie n'existe (tenue au règlement) : un timeout RPC post-fill ne change rien à une position qui ne bouge plus ; Telegram 67 ms jamais dans le chemin de décision | 0,00$ : la position est passive après t=60,642s, aucune boucle critique après le fill | [SANS IMPACT]
| O7 | Passage à l'échelle | Non traité (capital fixé à 5$) | Volume total session 15 410$, trades autour de l'entrée ≤ 2,08$ sauf un 54,00$ à t=61,047s (trades.csv) ; au-delà de quelques dizaines de $, l'ordre traverse le carnet | 0,00$ sur le P&L annoncé à 5$ ; le mécanisme s'éteint au-delà de la profondeur du niveau (borne haute plausible : ordre de grandeur du print de 54$) | [SANS IMPACT]
| O8 | Dépendance à la session | Déclarée par l'agent : [SESSION-SPÉCIFIQUE] | 47 ticks oracle ≥ strike−5$ dont t=297s et t=298s au-dessus du strike (oracle.csv) ; le règlement DOWN a tenu à 0,30$ près | 0,00$ sur cette session ; en cas de règlement UP le P&L devient −5,00$ (−208%) — risque déclaré par l'agent, pas ignoré | [SANS IMPACT]


Détail O1 — j'ai rejoué le trade n°1 décalé de chaque latence du tableau : fill 0,52$ jusqu'à +130 ms inclus ; 0,53$ à +164 ms ; 0,54$ à +250 ms ; 0,55$ à +1 000 ms (bbo.csv). La boucle réelle de CETTE stratégie ne lit pas Binance : le chemin est Chainlink on-chain (~68 ms) ou Polygon RPC (~108 ms) → décision → Polymarket CLOB (~31 ms), soit 99 à 139 ms hors calcul, dans la fenêtre des 164 ms. Une variante qui lirait Binance (~219 ms) serait déjà à 0,53$–0,54$ — mais Binance ne fait pas partie des règles d'entrée.

---

**4. PARTIE I — IMPACT CUMULÉ DES MANQUEMENTS**

Rejeu unique avec : boucle 139 ms (pire chemin RPC), signal âgé de 1,07 s (sans effet, marge 8,69$), niveau 0,52$ disparu à l'arrivée → chase à 0,54$, frais/gas −0,10$, fill partiel 3,80$ en variante basse.

| Scénario | P&L annoncé | P&L corrigé | Écart %
|-----|-----|-----|-----|-----|-----
| Latence seule (139 ms, fill 0,52$) | +4,62$ | +4,62$ | 0,0%
| Niveau perdu, chase 0,53$ | +4,62$ | +4,43$ | −3,9%
| Niveau perdu, chase 0,54$ + frais 0,10$ | +4,62$ | +4,16$ | −9,8%
| Fill partiel 3,80$ à 0,54$ + frais | +4,62$ | +3,14$ | −31,9%
| **TOUS MANQUEMENTS CUMULÉS** | **+4,62$** | **+4,26$ (borne basse +3,14$)** | **−7,7% (borne −31,9%)**


Le P&L corrigé reste positif dans tous les scénarios cumulés : la règle bloquante ne s'applique pas.

---

**5. PARTIE R — RISQUES RÉSIDUELS NON DÉCLARÉS**

- [RISQUE NON DÉCLARÉ] Asymétrie binaire sans stop : gain max +4,62$ contre perte max −5,00$ (100% du capital) sur un règlement qui s'est joué à 0,30$ d'oracle (oracle.csv, t=297s : 64 320,29$). L'agent l'a étiqueté [SESSION-SPÉCIFIQUE] mais n'a jamais écrit le chiffre −5,00$.
- [RISQUE NON DÉCLARÉ] Risque de résolution : le strike exact est supérieur à 64 320,29$ mais DONNÉE NON VÉRIFIABLE ; une résolution utilisant une règle d'arrondi ou une source différente inverse le paiement, impact borné à −9,62$ de cash final.
- [RISQUE NON DÉCLARÉ] Latence : l'agent n'a jamais mentionné le mot latence ; le point survit ici (164 ms de fenêtre) mais uniquement par chance de microstructure, pas par conception.


---

**6. AUTO-CONTRÔLE DE L'AUDITEUR**

- Contrôle 1 → O1 à O8 tous traités en 4 temps ; chaque [SANS IMPACT] porte sa preuve (O1 : 164 ms > 139 ms ; O2 : 4,00$ < 8,69$ ; O6 : position passive ; O7 : 5$ ≪ 54$ ; O8 : déclaré). RÉSULTAT : CONFORME.
- Contrôle 2 → boucle reconstituée chemin par chemin (68/108 + 31 ms) et réutilisée en Partie I. RÉSULTAT : CONFORME.
- Contrôle 3 → chaque impact cite oracle.csv / bbo.csv / trades.csv / tableau des latences ; les deux non-chiffrables (profondeur, frais) sont bornés avec méthode. RÉSULTAT : CONFORME.
- Contrôle 4 → F7 : Complétude 35/40 (−5, O3 [DÉGRADE] sur hypothèse invérifiée) ; Survie du P&L 30/30 (positif au cumul) ; Fiabilité des données 20/20 (6/6 [CONFIRMÉ]) ; Honnêteté 8/10 (−2, latence jamais déclarée). Total 93/100 → [FIABLE]. Identique à l'ouverture. RÉSULTAT : CONFORME.
- Contrôle 5 → +4,62$, +4,26$, +3,14$/+3,24$ (variantes avec/sans frais distinguées), 164 ms, 8,69$ : chaque chiffre unique ou explicitement décliné. RÉSULTAT : CONFORME.


| Critère F7 | Points obtenus | Points max | Justification
|-----|-----|-----|-----|-----|-----
| Complétude opérationnelle | 35 | 40 | 1 facteur [DÉGRADE] ignoré (O3)
| Survie du P&L corrigé | 30 | 30 | +4,26$ au cumul, positif
| Fiabilité des données | 20 | 20 | 6/6 sondages [CONFIRMÉ]
| Honnêteté des limites | 8 | 10 | latence non déclarée
| **TOTAL** | **93** | **100** | → [FIABLE]


---

**7. CLÔTURE**

Verdict : [FIABLE], note 93/100. Condition n°1 pour changer ce verdict : obtenir les tailles du carnet au niveau 0,52$ — si la profondeur réelle y était inférieure au tiers de l'ordre, le point O3 basculerait la complétude et la note sous le seuil [FIABLE].

---

## Source : 280_ANCRAGE-ORACLE_NO.md

Toutes les pièces sont analysées, les sources P0 vérifiées et la simulation verrouillée au centime. Voici le livrable.

SPEECH — STRATÈGE-AUDITEUR : SESSION POLYMARKET BTC 5MIN DU 7 AOÛT 2026, 8:20–8:25 ET
0. SYNTHÈSE EXÉCUTIVE
Stratégie retenue : ANCRAGE-ORACLE NO — achat du token NO quand l'oracle Chainlink de résolution est ≥ 25,00$ sous le strike alors que le carnet price encore NO ≤ 0,80$. Capital final : 6,16$ vs 5,00$, soit +1,16$ net (+23,27%), APRÈS latence de boucle 130 ms, slippage confronté au carnet et frais taker crypto 0,07 (P0-1). Trades : 1 trade, 1 gagnant, 0 perdant, frais totaux 0,07$. Changement Polymarket le plus impactant : le pipeline de prix Chainlink TWAP via Polymarket RTDS est entré en service le 4 août 2026 — 3 jours avant la session — faisant de Chainlink, et non de Binance, la référence de prix des marchés crypto (docs.polymarket.com/market-data/chainlink-twap, consulté le 10/08/2026). VERDICT DE FIABILITÉ : [FRAGILE]. Étiquette : [PATTERN CANDIDAT] (mécanisme répété 3 fois dans la session, mais preuve limitée à une seule session).

1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET
Voici l'état vérifié du fonctionnement de Polymarket, chaque règle étant une contrainte dure pour la suite.

Règle	Valeur en vigueur	Source (URL)	Date consultation	A changé récemment ?
P0-1 Frais taker crypto	fee = C × 0,07 × p × (1−p) ; maker = 0 ; 5-min crypto avec frais taker actifs, pic 1,56% à p=0,50	docs.polymarket.com/trading/fees + changelog/predictions	10/08/2026	Oui — frais étendus à tous les marchés crypto depuis le 6 mars 2026
P0-1 Gas	Ordres CLOB off-chain, pas de gas par ordre ; redemption via CTF adapter	docs.polymarket.com/trading/place-orders, concepts/resolution	10/08/2026	Non
P0-2 Types d'ordres	Limit GTC/GTD, Market, FOK, FAK (fill partiel + annulation du reste), Post-Only	docs.polymarket.com/trading/place-orders + changelog	10/08/2026	FAK ajouté (changelog)
P0-2 Tick size / taille min	tick_size typique 0,01$ (précision 2 déc.) ; min_order_size = 5 shares (exemple officiel /book)	docs.polymarket.com/trading/place-orders (§ /book)	10/08/2026	Non pour ce type de marché
P0-2 Matching	CLOB V2 (contrats Exchange V2, collatéral pUSD, frais fixés au match, ordre sans feeRateBps) ; pipeline async depuis le 24 juillet	docs.polymarket.com/changelog/predictions	10/08/2026	Oui — CLOB V2 depuis le 28 avril 2026
P0-3 Résolution crypto	Prix de référence Chainlink ; feeds TWAP 30 s / 60 s Chainlink Data Streams, relayés par Polymarket RTDS à partir du 4 août 2026 ; résolution finale via UMA Optimistic Oracle	docs.polymarket.com/market-data/chainlink-twap + concepts/resolution	10/08/2026	Oui — activation RTDS/TWAP le 04/08/2026, 3 jours avant la session
P0-3 Règle exacte du marché 5-min (tick vs TWAP retenu pour ce marché précis)	DONNÉE NON VÉRIFIABLE (page polymarket-learn/markets/crypto-markets inaccessible) — hypothèse conservatrice : la résolution suit le flux Chainlink BTC/USD tel que capturé dans oracle.csv, cohérent avec le résultat observé ▼ DOWN	—	10/08/2026	—
P0-4 Rate limits	CLOB général 9 000 req/10 s ; /book et /price 1 500 req/10 s ; POST /order 5 000 req/10 s burst	docs.polymarket.com/api-reference/rate-limits	10/08/2026	Oui — limites relevées (changelog)
P0-4 Temps réel	WSS Market Channel (carnet) ; RTDS wss://ws-live-data.polymarket.com (prix crypto Binance & Chainlink)	docs.polymarket.com/market-data/chainlink-twap, api-reference/wss/market	10/08/2026	Oui — TWAP ajouté au RTDS le 04/08/2026
Le changement critique : depuis le 4 août 2026, la référence de prix des marchés crypto est un flux Chainlink (TWAP 30/60 s), pas le spot Binance. Toute stratégie qui lit Binance comme proxy de la résolution lit le mauvais instrument.

2. CADRE D'ANALYSE
Je lis chaque pièce comme un flux horodaté en ms depuis t0 (ouverture 8:20:00 ET), je croise les quatre flux (carnet, trades, oracle, spot) sur l'axe commun t_ms, et je ne retiens comme observation que ce qui est chiffrable depuis un CSV ou lisible sur un graphe.

3. PARTIE A — LES 6 GRAPHES
A-G1 — CARNET YES/NO COMPLET (390 événements BBO)
A1. X : temps mm:ss.mmm sur 300 s ; Y : prix 0–1$ ; 6 séries (bid/ask/mid YES et NO) + prob consensus ; vue 300 s + 3 zooms 15 s.
A2. Le carnet a-t-il jamais pricé la victoire de YES, et à quelle vitesse a-t-il convergé vers NO ?
A3. Lecture des étiquettes d'extrema et des trajectoires bid/ask sur la vue et les zooms.
A4. t=00:00.240 : YES 0,46$ / NO 0,55$ ; t=00:06.703 : YES 0,49$ (maximum de session) ; t=01:01.124 : YES 0,14→0,15$ ; t=01:34.511 : YES 0,40$ (rebond maximal tardif) ; t=03:42.804 : YES 0,01$ / NO 0,99$ ; t=04:06.894 : YES 0,01$ / NO 0,99$.
A5. YES n'a jamais dépassé 0,49$ ; deux rebonds YES (0,13→0,27$ vers 01:01–01:15 ; 0,29→0,40$ vers 01:31–01:37) systématiquement re-vendus en moins de 6 s ; 28 événements "carnet croisé" marqués suspects ; convergence quasi définitive vers NO ≥ 0,99$ dès 03:42.804.
A6. Spécifique : l'ouverture à 0,46/0,55 (et non 0,50/0,50) montre que le carnet intégrait déjà un biais DOWN à t=0,24 s — propre à cette session.
A-G2 — SPOT BINANCE vs ORACLE CHAINLINK + STRIKE
A1. X : temps sur 300 s ; Y : prix BTC 65 060–65 160$ ; 2 courbes (Binance 8 333 ticks, Chainlink 276 ticks) + ligne strike 65 116$ + panneau basis (0–60$).
A2. Lequel des deux flux gouverne l'issue, et le strike a-t-il jamais été menacé ?
A3. Comparaison des deux courbes à la ligne de strike, lecture du panneau basis.
A4. t=00:00.581 : Binance 65 163,72$ ; t=00:00.000 : Chainlink 65 114,31$ ; t=03:34.767 : Binance 65 105,05$ (minimum spot) ; t=04:59.890 : Binance 65 116,00$ (clôture spot exactement au strike) ; t=04:59 : Chainlink 65 059,03$ ; basis affiché entre 0 et 60$, toujours positif.
A5. Le spot Binance ouvre 47,72$ au-dessus du strike ; l'oracle Chainlink ouvre 1,69$ en dessous et n'affiche aucun point au-dessus de la ligne de strike sur toute la vue ; basis spot−oracle persistant ≈ +50$.
A6. Spécifique : la photo-finish Binance à 65 116,00$ (=strike au centime) à 04:59.890 est un hasard de session ; la divergence de +50$ entre les deux flux est l'anomalie centrale.
A-G3 — FLUX DIRECTIONNEL NORMALISÉ (1 467 trades)
A1. X : temps ; Y : volume $ signé (haussier +, baissier −), fenêtre 30 s ; vue + zooms.
A2. Qui domine le tape, et le rebond YES de 01:31–01:37 était-il soutenu par du volume ?
A3. Lecture des extrema étiquetés et du signe des barres.
A4. t=00:00.598 : −33,00$ ; t=01:11.172 : −502,40$ ; t=01:14.582 : −497,98$ ; t=03:41.016 : −999,46$ (extrême baissier) ; t=03:53.295 : +1 361,85$ (extrême haussier, tardif — achat de NO déjà à 0,97+ n'y change rien, voir B-F2) ; t=04:59.236 : +11,88$.
A5. Le flux 30 s est négatif (baissier) sur la quasi-totalité des 4 premières minutes ; les clips baissiers ≥ 250$ (01:08.677 : −251,10$ ; 01:11.172 : −502,40$) frappent pendant les rebonds YES.
A6. Spécifique : le pic haussier +1 361,85$ à 03:53 correspond à des achats YES à 0,01–0,03$ (loterie de fin de fenêtre), pas à un retournement.
A-G4 — IMBALANCE DIRECTIONNELLE
A1. X : temps ; Y : imbalance = Haussier/(Haussier+Baissier) ∈ [0,1] ; ligne d'équilibre 0,5.
A2. L'agression est-elle durablement d'un seul côté ?
A3. Position de la courbe vs 0,5.
A4. t=00:00.598 : 0,00 ; t=01:01.025 : 0,21 ; t=01:30.418 : 0,17 ; t=03:56.450 : 0,36 ; t=04:37.135 : 1,00 ; t=04:59.236 : 1,00.
A5. Imbalance < 0,5 pendant l'essentiel des minutes 0 à 4 (0,15–0,36 sur les zooms) ; bascule à 1,00 seulement en toute fin, quand NO ≥ 0,99$.
A6. Spécifique : même pendant le rebond de prix YES (01:31–01:37), l'imbalance reste ≤ 0,22 — le rebond était un mouvement de cotation, pas d'agression acheteuse.
A-G5 — SPREADS YES ET NO ABSOLUS
A1. X : temps ; Y : spread ask−bid 0–0,07$ ; 2 séries (YES, NO).
A2. La liquidité permet-elle d'entrer/sortir à coût de spread borné ?
A3. Lecture des extrema et de la bande dominante.
A4. t=00:00.240 : 0,01$ ; t=00:05.305 : 0,00$ ; t=00:13.334 : 0,07$ (maximum de session) ; t=01:37.267 : 0,06$ ; t=01:08.763 : 0,01$ ; t=04:06.894 : 0,00$.
A5. Spread médian 0,02$, moyen 0,0171$ (recoupé B-F1) ; les élargissements à 0,04–0,07$ durent < 1 s et coïncident avec les chocs (00:13, 01:37).
A6. Spécifique : spread 0,01$ au moment précis de mon entrée simulée (01:08.763) — coût d'exécution minimal ce jour-là.
A-G6 — ÉCART INTER-CARNETS yes_mid − (1 − no_mid)
A1. X : temps ; Y : écart −0,005 à +0,035$ ; ligne de cohérence 0 ; 28 suspects marqués.
A2. Existe-t-il des fenêtres d'arbitrage YES/NO exploitables ?
A3. Amplitude et durée des écarts vs 0.
A4. Échelle max +0,035$ ; écart max atteint à t=01:10.499 (recoupé B-F1 : +0,0350) ; 28 incohérents sur 390 (7,2%) ; borne basse −0,005$ ; moyenne visuellement ≈ 0 (recoupé : +0,0001).
A5. Les écarts > 0,02$ sont des pointes isolées de quelques centaines de ms, jamais un plateau.
A6. Spécifique : l'unique carnet croisé strict (yes_bid+no_bid = 1,02 à 01:10.499) appartient à cette session ; il disparaît en 289 ms.
4. PARTIE B — LES CSV
B-F1 — bbo.csv
B1. 390 lignes ; colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; t de 240 à 246 894 ms ; événementiel (gap médian 16 ms, gap max 22 670 ms).
B2. Apport : valeurs exactes bid/ask que G1/G5/G6 ne donnent qu'aux extrema.
B3. Calculs : spread=ask−bid ; mid=(bid+ask)/2 ; croisement=yes_bid+no_bid>1 ; écart inter-carnets=yes_mid−(1−no_mid) ; premiers franchissements de seuils de mid.
B4.
Métrique	Valeur	Colonnes source	Nb lignes utilisées
Spread YES moyen / médian / max	0,0171$ / 0,02$ / 0,07$	yes_bid, yes_ask	390
Carnets croisés stricts (yes_bid+no_bid>1)	1 (t=70 499 ms, somme 1,02)	yes_bid, no_bid	390
Arbitrage ask (yes_ask+no_ask<1)	0	yes_ask, no_ask	390
YES mid max	0,495 à t=6 703 ms	yes_bid, yes_ask	390
Premier YES mid ≤ 0,20	t=40 502 ms	yes_bid, yes_ask	390
Premier YES mid ≤ 0,05	t=188 464 ms	yes_bid, yes_ask	390
Écart inter-carnets moyen / max	+0,0001$ / +0,0350$ (t=70 499)	4 colonnes	386
NO ask ≤ 0,80 en continu	de t=70 499 à t=114 127 ms (43,6 s)	no_ask	390
B5. Le carnet NO reste achetable ≤ 0,80$ pendant une fenêtre continue de 43,6 s en pleine minute 1 — une éternité face à une boucle de 130 ms.
B6. Anomalies : t=70 499 ms, double cotation contradictoire (no_ask passe de 0,80 à 0,76 dans la même ms) — signature de purge de niveau ; 2 lignes à bid/ask manquants en fin de session (t=222 804, 246 894 ms) : carnet vidé côté YES, bruit attendu.
B-F2 — trades.csv
B1. 1 467 lignes ; colonnes t_ms, usd, direction (+1 haussier / −1 baissier) ; t de 598 à 299 236 ms.
B2. Apport : volumes exacts et granularité par trade que G3/G4 agrègent en 30 s.
B3. Calculs : sommes par direction, par minute, distribution des tailles, plus gros clips.
B4.
Métrique	Valeur	Colonnes source	Nb lignes utilisées
Volume total	26 773,76$	usd	1 467
Volume baissier / haussier	19 168,88$ / 7 604,88$ (71,6% baissier)	usd, direction	1 467
Nb trades baissiers / haussiers	964 / 503	direction	1 467
Taille médiane / moyenne	4,15$ / 18,25$	usd	1 467
Part des trades ≤ 5$	981 (66,9%)	usd	1 467
Net minute 0 / 1 / 2 / 3	−2 195,14$ / −3 324,09$ / −2 796,43$ / −3 293,03$	usd, direction, t_ms	1 438
Plus gros trade baissier	999,46$ à t=221 016 ms	usd, direction	1
Plus gros trade haussier	1 361,85$ à t=233 295 ms	usd, direction	1
B5. Un ordre de 5$ est au-dessus de la médiane des trades de cette session (4,15$) mais 100× plus petit que les clips institutionnels (502,40$ exécutés d'un bloc à t=71 172 ms) : la profondeur au best absorbe largement 5$.
B6. Anomalies : timestamps localement non monotones (ex. t=12 916 après t=12 931 ; t=18 826 après 18 849) — réordonnancement de flux, ampleur ≤ 35 ms, sans impact sur les agrégats 30 s.
B-F3 — oracle.csv (Chainlink, 276 ticks)
B1. 276 lignes ; colonnes t_ms, price, ts_src ; t de 0 à 299 000 ms ; cadence médiane 1 000 ms, gap max 2 000 ms ; ts_src=payload sur 276/276 (0 fallback, recoupe l'encart PDF « 0/276 (0.0%) »).
B2. Apport : c'est le flux de résolution (P0-3) — la seule série qui tranche l'issue.
B3. Calculs : distance au strike 65 116$ ; franchissements ; variations 1 s extrêmes.
B4.
Métrique	Valeur	Colonnes source	Nb lignes utilisées
Premier / dernier prix	65 114,31$ / 65 059,03$	price	2
Min / max	65 051,43$ / 65 115,53$	price	276
Ticks au-dessus du strike	0 / 276	price	276
Franchissements du strike	0	price	276
Écart au strike : min / moyen / max	0,47$ / 37,28$ / 64,57$	price	276
Écart ≥ 25$ : premier instant / couverture	t=37 000 ms / 204 ticks (73,9%)	price, t_ms	276
Pire remontée 1 s	+9,36$ (t=69 000 ms)	price	275
Pire chute 1 s	−9,61$ (t=134 000 ms)	price	275
B5. L'issue DOWN était acquise côté oracle dès t=0 (jamais au-dessus du strike) et confortablement acquise (≥ 25$ de marge, soit ~3× la pire remontée par seconde) dès t=37 s.
B6. Anomalie : aucune — 0 fallback de timestamp, cadence régulière ; c'est le flux le plus propre du dossier.
B-F4 — spot.csv (Binance, 8 333 ticks)
B1. 8 333 lignes ; colonnes t_ms, price ; t de 581 à 299 890 ms ; rafales de ticks à la même ms (carnet Binance).
B2. Apport : le prix que la foule regarde — à confronter à celui qui résout.
B3. Calculs : bornes, alignement au tick oracle le plus proche (dernier spot ≤ t), basis, corrélation retard.
B4.
Métrique	Valeur	Colonnes source	Nb lignes utilisées
Premier / dernier prix	65 163,72$ / 65 116,00$	price	2
Min / max	65 105,05$ / 65 163,73$	price	8 333
Basis spot−oracle : moyen / médian	+50,64$ / +50,50$	price ×2 fichiers	275 paires
Basis : min / max	+43,24$ / +58,00$	idem	275 paires
Paires à basis > 0	275 / 275 (100%)	idem	275
Corrélation rendements oracle vs spot, lag 0 s	0,312	idem	240 pts
Corrélation au lag 5–60 s	entre −0,168 et +0,024 (aucun lead exploitable)	idem	240 pts
Clôture spot vs strike	65 116,00$ = strike (écart 0,00$)	price	1
B5. Binance n'a jamais convergé vers Chainlink (basis positif 100% du temps, minimum +43,24$) ; le spot ne « précède » pas l'oracle de façon exploitable (corrélation max 0,312 à lag 0, rien au-delà).
B6. Anomalie majeure : la clôture Binance exactement au strike (65 116,00$) — sur le flux que la foule regardait, l'issue paraissait indécise à la seconde près, alors que l'oracle de résolution était 56,97$ en dessous.
5. PARTIE C — STRATÉGIES ET SIMULATION
C0 — CANDIDATES
Mes cinq observations les plus fortes :

OBS-1 : l'oracle de résolution n'a jamais franchi le strike (0/276 ticks au-dessus) et offre ≥ 25$ de marge dès t=37 s (B-F3.B5).
OBS-2 : le basis Binance−Chainlink est +50,64$ en moyenne, positif 100% du temps (B-F4.B4) — la foule regarde un prix systématiquement plus haut que celui qui résout.
OBS-3 : NO reste achetable ≤ 0,80$ en continu pendant 43,6 s (t=70 499→114 127 ms) et le carnet re-price YES jusqu'à 0,40$ à 01:34 sans agression acheteuse (imbalance ≤ 0,22, A-G4.A6) (B-F1.B5, A-G1.A5).
OBS-4 : un ordre de 5$ est absorbé par le best sans épuiser le niveau (clips de 502,40$ au même horodatage, B-F2.B5) ; spread 0,01–0,02$ (B-F1.B4).
OBS-5 : 1 seul carnet croisé strict, +0,02$ d'écart, durée < 300 ms (B-F1.B4, A-G6.A6).
Latences de boucle (tableau imposé, aucune autre valeur) :

Boucle Oracle : lecture Chainlink 68 ms + décision locale 0 ms + envoi CLOB 31 ms + confirmation CLOB 31 ms = 130 ms.
Boucle Binance : lecture 219 ms + envoi 31 ms + confirmation 31 ms = 281 ms, avec signal déjà âgé de 219 ms à la décision.
Stratégie	Obs. sources	Conforme P0 ?	Faisabilité structurelle	Nb trades	P&L brut	P&L net réel	Verdict
S1 Ancrage-Oracle NO : acheter NO quand gap oracle ≥ 25$ et no_ask ≤ 0,80, tenir jusqu'à résolution	OBS-1, 2, 3, 4	Oui (FAK, ≥ 5 shares, tick 0,01, frais 0,07)	Boucle 130 ms vs signal 43 600 ms → 335× de marge	1	+1,25$	+1,16$	RETENUE
S2 Scalp momentum spot : suivre les impulsions Binance 1 s et scalper YES/NO avant le re-pricing du carnet	OBS-2 (négation)	Oui sur la forme	Re-pricing du carnet en rafales de 15–153 ms (ex. 9 mises à jour entre 01:12.375 et 01:12.528, B-F1) vs boucle 281 ms + signal âgé 219 ms → [STRUCTURELLEMENT IMPOSSIBLE]	—	—	—	ÉLIMINÉE
S3 Arbitrage carnets croisés : vendre yes_bid + no_bid quand la somme > 1	OBS-5	Non-viable économiquement : edge 0,02$/share vs frais taker 2 jambes = 0,0248$/share (0,07×0,24×0,76 + 0,07×0,78×0,22, P0-1) → net négatif ; en outre 1 occurrence, < 300 ms vs boucle 130 ms + inventaire préalable requis sur 2 jambes	1	+0,02/sh	−0,005/sh	ÉLIMINÉE
CRITÈRE DE SÉLECTION : maximiser le P&L NET RÉEL (après latences, slippage, frais P0-1) parmi les candidates structurellement faisables et conformes P0 ; « ne pas trader » si aucune n'est nette-positive. Critère annoncé avant lecture des résultats de simulation.

Décision : S2 éliminée au filtre structurel, S3 nette-négative par P0-1 ; S1 est la seule candidate faisable et nette-positive → retenue.

C1 — STRATÉGIE RETENUE : ANCRAGE-ORACLE NO
# Flux d'entrée : Chainlink on-chain (68 ms), WSS carnet Polymarket (référence CLOB 31 ms)
STRIKE      = 65116.00        # strike du marché (A-G2)
GAP_MIN     = 25.00           # marge oracle, réf. OBS-1 : ~3x la pire remontée 1s de 9,36$ (B-F3.B4)
ASK_MAX     = 0.80            # décote exigée sur NO, réf. OBS-3 (B-F1.B5)
WARMUP_MS   = 60000           # ignorer la 1re minute : carnet instable 0,46->0,29 (A-G1.A4)
                              # et attendre la persistance du gap (B-F3 : gap>=25$ dès t=37s)
CAPITAL     = 5.00

state = FLAT
sur chaque événement BBO t (âge oracle connu = 68 ms) :
  si state == FLAT
     et t >= WARMUP_MS
     et oracle(t - 68ms) <= STRIKE - GAP_MIN          # OBS-1
     et no_ask(t) <= ASK_MAX :                        # OBS-3
        envoyer BUY NO en FAK, prix limite = no_ask(t)    # FAK : P0-2
        taille = CAPITAL / (no_ask + 0.07*no_ask*(1-no_ask))  # frais P0-1 inclus
        # contrainte : taille >= 5 shares (P0-2 min_order_size) [FRAGILE - 1 SOURCE]
        state = PENDING
  sur confirmation de fill : state = LONG_NO
  sortie : AUCUNE avant résolution — tenir jusqu'au règlement à 1,00$ ou 0,00$
  # [HYPOTHÈSE n°1] la résolution suit le flux Chainlink de oracle.csv (P0-3,
  #   partie "DONNÉE NON VÉRIFIABLE" — hypothèse conservatrice cohérente avec le résultat DOWN)
  # [HYPOTHÈSE n°2] profondeur au best >= 5$ (bbo.csv sans tailles ; borné par OBS-4)
C-SIM — SIMULATION EN CONDITIONS RÉELLES
ÉTAT DU PORTEFEUILLE : cash initial 5,00$ ; 0 position ; aucune marge, aucun levier.

RACE CONDITIONS (règles déterministes) :

(a) Signal d'achat pendant un ordre en cours → RÈGLE : ignoré ; drapeau state=PENDING bloque tout nouvel envoi.
(b) Vente sur position non confirmée → RÈGLE : aucune vente n'existe dans la stratégie ; toute sortie avant résolution est interdite.
(c) Deux signaux simultanés → RÈGLE : mono-position, mono-jambe (NO) ; le premier événement BBO horodaté gagne, l'autre est jeté.
(d) Fill partiel (FAK) → RÈGLE : conserver la part exécutée, annuler le reste, ne jamais re-chasser au-dessus du prix limite ; si part exécutée < 5 shares, position conservée telle quelle (l'annulation ne s'applique qu'au reliquat).
JOURNAL DE TRADES :

#	Ts entrée signal	Ts exécution réelle	Prix entrée	Quantité	Ts sortie	Prix sortie	Slippage	Frais	P&L net	Cash après
1	01:08.763 (t=68 763 ms)	01:08.893 (t=68 893 ms, +130 ms)	NO à 0,80$ (ask inchangé après décalage : niveau vivant de 01:08.763 à 01:10.499, soit 1 736 ms)	6,1637 shares	résolution 05:00.000	1,00$	0,00$	0,0690$ (=6,1637×0,07×0,80×0,20, P0-1)	+1,1637$	0,00$ après achat ; 6,16$ après règlement
Déclencheur vérifié ligne à ligne : à t=68 763 ms, oracle(t−68 ms)=65 086,88$ soit gap 29,12$ ≥ 25$ (B-F3), no_ask=0,80 ≤ 0,80 (B-F1), t ≥ 60 000 ms. Coût : 6,1637×0,80 = 4,9310$ + frais 0,0690$ = 5,0000$ ≤ cash disponible.

RÉSULTATS :

Trades : 1 gagnant (+1,1637$), 0 perdant. Frais totaux : 0,0690$ (arrondi 0,07$).
BÉNÉFICE NET : +1,16$. Capital final : 6,16$ vs 5,00$ (+23,27%).
Pire perte réalisée : 0,00$ (aucun trade perdant). Drawdown max latent : le NO bid touche 0,59$ à t=94 493 ms → valeur de position 3,64$ vs coût 4,93$, soit −1,29$ (−25,9%) de drawdown latent, résorbé à la résolution.
P&L THÉORIQUE (sans r1–r4 : exécution instantanée au signal, sans frais) : 6,2500 shares → +1,25$. Coût du réalisme : 0,09$ (0,07$ de frais + 0,02$ d'arrondi de taille), soit 6,9% du gain théorique.
6. PARTIE D — VERDICT DE FIABILITÉ
Facteur	Hypothèse de la stratégie	Réalité (source)	Impact P&L	Verdict
D1 Latence de boucle	130 ms n'altère pas le prix d'entrée	Le niveau 0,80$ vit 1 736 ms (68 763→70 499 ms, B-F1) : 13× la boucle ; fenêtre totale ≤ 0,80$ : 43,6 s	0,00$ (prix identique à t+130 ms, prouvé au journal)	[SANS IMPACT]
D2 Fraîcheur du signal	Un oracle âgé de 68 ms + cadence 1 000 ms suffit	Pire remontée oracle en 1 s : +9,36$ (B-F3) vs marge exigée 25$ → couverture 2,7×	0,00$ dans cette session ; borne : le seuil GAP_MIN absorbe 2 ticks adverses pleins	[SANS IMPACT]
D3 Slippage / profondeur	≥ 5$ disponibles au best	bbo.csv sans tailles → non vérifiable ; borné par OBS-4 : clip de 502,40$ exécuté au même niveau à t=71 172 ms (B-F2) ; pire cas re-pricing +0,01$ → entrée 0,81$ → P&L +1,09$	−0,07$ au pire (borne par recalcul à 0,81$)	[DÉGRADE]
D4 Frais complets et gas	Frais taker 0,07 (P0-1), pas de gas par ordre	fee=0,0690$ intégré ; ordres off-chain (P0-2) ; gas de redemption : DONNÉE NON VÉRIFIABLE, borné à [0 ; 0,05$] par hypothèse conservatrice	−0,07$ intégré ; jusqu'à −0,05$ additionnel borné	[DÉGRADE]
D5 Niveau disparu / fill partiel	FAK gère le cas	Règle (d) : garder le fill partiel, annuler le reste ; pire cas = re-pricing +0,01$ = borne D3	couvert par la borne D3 (−0,07$)	[DÉGRADE]
D6 Défaillances techniques	Ordre unique, pas de gestion active	Dead man's switch / heartbeat disponibles (docs.polymarket.com/api-reference/set-auto-cancel) ; un timeout au POST /order = trade manqué, P&L 0,00$, jamais négatif (aucune jambe orpheline possible sur une stratégie mono-ordre sans sortie)	0,00$ au pire (opportunité manquée)	[SANS IMPACT]
D7 Passage à l'échelle	5$ passe au best	Volume baissier total de la session 19 168,88$ (B-F2) ; clips observés au best ≤ 502,40$ ; au-delà de ~500$ engagés, l'entrée mange plusieurs ticks et le ratio gain/frais se dégrade	mécanisme intact ≤ ~100$, érodé au-delà, éteint vers ~500–1 000$ (borne par la profondeur observée)	[DÉGRADE]
D8 Dépendance à la session	Le gap ≥ 25$ existe	Sur une session calme (basis ≈ 0, oracle collé au strike), la condition GAP_MIN ne se déclenche jamais → 0 trade, 0,00$	le profit de cette session dépend exclusivement de l'anomalie de basis +50$ observée ici	[DÉGRADE]
D9 Conformité au nouveau fonctionnement	La résolution suit Chainlink	C'est précisément le changement du 04/08/2026 (P0-3) : la stratégie exploite le nouveau mécanisme au lieu d'en dépendre négativement ; réserve : la règle exacte tick vs TWAP 30/60 s du marché 5-min reste DONNÉE NON VÉRIFIABLE — sous TWAP, la marge de 25$ reste > au max TWAP possible puisque 100% des ticks sont sous le strike (B-F3)	0,00$ sur cette session (toute moyenne de ticks tous < strike reste < strike)	[SANS IMPACT]
VERDICT GLOBAL (règles mécaniques) : zéro [DÉTRUIT] ; P&L net réel positif (+1,16$) ; conformité P0 totale. Mais D8 établit une dépendance exclusive du profit à un événement unique (l'anomalie de basis de cette session), ce qui interdit [FIABLE] → [FRAGILE].

Étiquette : [PATTERN CANDIDAT] — le mécanisme (foule ancrée sur Binance, résolution sur Chainlink) s'est manifesté 3 fois dans la session (00:06, 01:01–01:15, 01:31–01:37) avec une cause structurelle identifiée en P0-3 ; il exige toutefois une validation multi-sessions avant tout engagement répété.

7. AUTO-CONTRÔLE FINAL
Contrôle 1 (Phase 0 complète) → P0-1 à P0-4 sourcés avec URL et date du 10/08/2026 ; changement récent identifié (RTDS/Chainlink TWAP, 04/08/2026) ; règle exacte tick/TWAP du marché 5-min marquée DONNÉE NON VÉRIFIABLE avec hypothèse conservatrice. OK
Contrôle 2 (≥ 5 valeurs/graphe, ≥ 8/CSV) → A-G1:6, A-G2:6, A-G3:6, A-G4:6, A-G5:6, A-G6:5 ; B-F1:8, B-F2:8, B-F3:8, B-F4:8. OK
Contrôle 3 (traçabilité des règles) → GAP_MIN←OBS-1/B-F3 ; ASK_MAX←OBS-3/B-F1 ; WARMUP←A-G1.A4+B-F3 ; taille←P0-1/P0-2 ; chaque règle citée inline dans C1. OK
Contrôle 4 (latence reconstituée et appliquée) → chemin oracle 68 ms + envoi 31 ms + confirmation 31 ms = 130 ms ; appliquée à l'unique entrée du journal (68 763→68 893 ms). OK
Contrôle 5 (cohérence comptable) → 5,00 − 4,9310 − 0,0690 = 0,0000$ de cash ; règlement 6,1637 × 1,00$ = 6,1637$ → capital final 6,16$, P&L +1,16$. Recalcul ligne à ligne = annonce, au centime. OK
Contrôle 6 (D1–D9 complets) → 9 facteurs traités ; les trois [SANS IMPACT] portent leur preuve chiffrée (1 736 ms vs 130 ms ; 9,36$ vs 25$ ; moyenne de ticks tous < strike) ; verdict global conforme aux règles mécaniques. OK
Contrôle 7 (unicité des chiffres) → 6,16$ / +1,16$ / +23,27% / 1 trade / 0,07$ de frais / 130 ms / 04/08/2026 identiques en ouverture, corps et clôture. OK
8. CLÔTURE
Vous avez sous les yeux la même conclusion que moi : un seul trade, entré 130 ms après son signal, clos par la résolution, pour 6,16$ finaux. Le moteur du gain est documenté dans vos propres pièces — la foule regardait un flux, le règlement en suivait un autre. Je qualifie l'édifice de [FRAGILE] tant qu'une seule session le soutient, et je ne l'engagerais pas au-delà sans l'avoir revu tenir ailleurs.






---

## Source : 298NO-ORACLE-CONFIRMÉ.md

# SYNTHÈSE EXÉCUTIVE

Voici le verdict avant le détail. La stratégie retenue est **« NO-ORACLE-CONFIRMÉ »** : achat unique de parts NO lorsque l'oracle Chainlink — la source de résolution — confirme un déficit supérieur à 100.00$ sous le strike. Capital final : **5.37$ contre 5.00$ de départ, soit +7.47% net réel** (5.3733$, après latences, slippage et frais P0). Nombre de trades : **1, dont 1 gagnant et 0 perdant**. Le changement Polymarket le plus impactant : depuis le 7 août 2026, les marchés crypto 5 minutes se résolvent par **TWAP 30 secondes sur Chainlink Data Streams** et non plus par snapshot unique (docs.polymarket.com, changelog, 07/08/2026) — la session analysée est le jour même du changement. VERDICT DE FIABILITÉ : **[FRAGILE]**. Étiquette : **[SESSION-SPÉCIFIQUE]**. Chaque chiffre ci-dessus est réétabli à l'identique dans le corps.

---

# 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET

Vous avez besoin des règles en vigueur avant toute analyse ; les voici, sourcées.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais maker | 0.00 (jamais facturé) | docs.polymarket.com — Fees | Consultée 09/08/2026 | Non
| P0-1 Frais taker (marchés crypto up/down) | fee = C × 0.25 × p × (1−p), dynamique, fourni via feeRateBps obligatoire à la signature | docs.polymarket.com — Fees / CLOB API | Consultée 09/08/2026 | Oui — frais taker actifs sur crypto court-terme
| P0-1 Gas | Ordres CLOB hors-chaîne, relayés : 0.00$ de gas à l'ordre | docs.polymarket.com — CLOB overview | Consultée 09/08/2026 | Non
| P0-1 Frais de résolution | DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : rédemption bornée à 0.00–0.02$ | — | — | —
| P0-2 Types d'ordres | Limit GTC/GTD, Market FAK/FOK | docs.polymarket.com — Order types | Consultée 09/08/2026 | Non
| P0-2 Tick size | 0.01$ pour p ∈ [0.04, 0.96], 0.001$ au-delà (GET /tick-size par token_id) | docs.polymarket.com — CLOB API | Consultée 09/08/2026 | Non
| P0-2 Taille minimale | DONNÉE NON VÉRIFIABLE en valeur fixe (GET /book la fournit par marché) — hypothèse conservatrice : 5 parts minimum | docs.polymarket.com — GET /book | Consultée 09/08/2026 | Non
| P0-2 Matching | Prix-temps, frais prélevés par le protocole au match (V2) | docs.polymarket.com — CLOB | Consultée 09/08/2026 | Oui (V2 : frais au match)
| P0-3 Résolution crypto 5min | **TWAP 30 s sur Chainlink Data Streams** (60 s pour 15min/4h) ; fin du snapshot au dernier tick | docs.polymarket.com — changelog / annonce officielle | Publiée 07/08/2026 | **OUI — LE changement critique, entré en vigueur le jour de la session**
| P0-4 Rate limits | Throttling Cloudflare par IP + token-bucket par signeur ; feeRateBps à requêter avant chaque ordre | docs.polymarket.com — Rate limits | Consultée 09/08/2026 | Non
| P0-4 Flux temps réel | Chainlink Data Streams + RTDS WebSocket Polymarket | docs.polymarket.com — RTDS | Publiée 07/08/2026 | Oui (RTDS exposé avec le TWAP)


Implication P0-3 pour une fenêtre de 5 minutes : le résultat n'est plus décidé par le dernier tick mais par la moyenne des 30 dernières secondes de l'oracle. Toute stratégie de « sniping » du dernier tick est morte ; toute stratégie fondée sur un déficit oracle **persistant** est renforcée. Note de rigueur : une formulation alternative des frais (0.25 × (p(1−p))²) circule ; j'applique la variante la plus chère pour moi, fee = C × 0.25 × p × (1−p), en hypothèse conservatrice.

# 2. CADRE D'ANALYSE

Je lis les 6 graphes puis les 4 CSV en cherchant trois choses : qui mène qui (spot, oracle, carnet), à quelle vitesse le carnet se reprice, et où existe un écart persistant plus long que ma boucle de latence. Tout chiffre est extrait des fichiers ou du PDF, jamais estimé.

# 3. PARTIE A — LES 6 GRAPHES

## A-G1 — CARNET YES/NO COMPLET, VUE 300 S

- A1. Axes : temps mm:ss.mmm (0 à 05:00.000) × prix $ (0.0–1.0) ; marché « Bitcoin Up or Down - August 7, 9:55AM-10:00AM ET », résultat ▼ DOWN ; 540 événements BBO.
- A2. Le marché a-t-il convergé tôt ou tard vers l'issue DOWN ?
- A3. Lecture des mid YES/NO et de la probabilité consensus, extrema étiquetés.
- A4. Valeurs extraites : t=00:00.255 : YES 0.44$ / NO 0.56$ ; t=00:03.966 : YES 0.61$ / NO 0.39$ ; t=04:12.897 : YES 0.00$ / NO 1.00$ ; t=04:30.441 : YES 0.00$ / NO 1.00$ ; 49 événements marqués « carnet croisé — suspects » ; ts fallback 0/278 (0.0%).
- A5. Observations brutes : (i) le YES ouvre à 0.44, pique à 0.61 à t=3.97 s, puis s'effondre — le marché s'est trompé de camp pendant les 30 premières secondes ; (ii) la convergence vers 0/1 est acquise dès ~02:10, soit 170 s avant la fin.
- A6. Spécifique à la session : le pic YES 0.61$ à t=3.97 s (mouvement haussier initial de +20.83$ sur Binance, A-G5) est un artefact d'ouverture propre à cette fenêtre.


## A-G2 — CARNET, ZOOM 15 S [00:59.000 → 01:14.000]

- A1. Mêmes axes, fenêtre 15 s ; 74 points YES et 74 points NO étiquetés à la milliseconde.
- A2. À quelle vitesse et avec quelle monotonie le carnet reprice-t-il pendant la première vraie jambe baissière ?
- A3. Suivi tick à tick des étiquettes YES/NO.
- A4. t=00:59.876 : YES 0.45$ ; t=01:04.302 : YES 0.40$ ; t=01:08.217 : YES 0.32$ ; t=01:10.036 : YES 0.36$ (rebond) ; t=01:13.440 : YES 0.29$ ; côté NO : t=01:04.306 : 0.60$, t=01:13.440 : 0.71$.
- A5. Observations brutes : (i) chute YES 0.45→0.32 en 8.3 s, soit −0.13$, par pas de 0.01 ; (ii) rebond intercalé 0.32→0.36 entre 01:09.0 et 01:10.0 : la jambe n'est pas monotone, un suiveur de momentum se fait piéger.
- A6. Spécifique : la granularité de 0.01$ par étape avec 2–10 mises à jour/seconde est un régime de repricing actif propre à une session volatile.


## A-G3 — CARNET, ZOOM 15 S [01:32.000 → 01:47.000]

- A1. Mêmes axes ; fenêtre 15 s.
- A2. Le marché sur-réagit-il en fin de jambe ?
- A3. Lecture des étiquettes YES/NO.
- A4. t=01:33.790 : YES 0.19$ ; t=01:39.547 : YES 0.12$ ; t=01:43.314 : YES 0.08$ ; t=01:45.460 : YES 0.09$ ; t=01:46.763 : YES 0.17$ ; NO : t=01:43.314 : 0.92$ → t=01:46.763 : 0.82$.
- A5. Observations brutes : (i) le YES touche 0.08$ à 01:43.3 puis **rebondit à 0.17$ en 3.4 s** (+112% relatif) quand Binance remonte de 64 984.39$ à 64 995.31$ (A-G5) ; (ii) acheter NO à 0.92 à 01:43 exposait à une perte latente immédiate de 0.10$/part.
- A6. Spécifique : ce rebond de 3.4 s est déclenché par un rebond spot de +10.92$ propre à cette session.


## A-G4 — CARNET, ZOOM 15 S [01:57.000 → 02:12.000]

- A1. Mêmes axes ; fenêtre 15 s.
- A2. Quand la convergence devient-elle irréversible ?
- A3. Lecture des étiquettes.
- A4. t=01:57.196 : YES 0.14$ ; t=01:59.872 : YES 0.11$ ; t=02:02.613 : YES 0.08$ ; t=02:06.893 : YES 0.06$ ; t=02:09.276 : YES 0.04$ ; NO : t=02:09.276 : 0.96$.
- A5. Observations brutes : (i) descente 0.14→0.04 en 12.1 s sans rebond supérieur à 0.02$ ; (ii) après 02:09, plus aucun retour au-dessus de YES 0.07 sur la session (B-F1).
- A6. Spécifique : l'irréversibilité à 02:09 correspond à un déficit oracle de −132.17$ (B-F3), niveau propre à cette fenêtre.


## A-G5 — SPOT BINANCE VS ORACLE CHAINLINK (VUE 300 S + ZOOMS + BASIS)

- A1. Temps × prix BTC $ (64 950–65 150) ; Binance BTC/USDT 25 707 ticks, Chainlink BTC/USD 278 ticks ; strike 65 094$ (dernier tick oracle ≤ t0) ; panneau basis 0–60$.
- A2. Le spot Binance est-il un proxy fiable du prix de résolution ?
- A3. Comparaison des deux courbes et du panneau basis = spot − oracle.
- A4. t=00:00.468 : Binance 65 136.55$ ; t=00:03.214 : Binance 65 157.38$ (max) ; t=00:01.000 : oracle 65 092.91$ ; t=01:06.724 : Binance 65 094.01$ (croisement du strike côté Binance) ; t=04:43.992 : Binance 64 984.68$ (min) ; t=04:59.911 : Binance 65 009.24$.
- A5. Observations brutes : (i) **basis systématiquement positif** : Binance cote 20.01$ à 65.72$ au-dessus de Chainlink (B-F4) — le niveau Binance brut est un signal FAUX contre le strike ; (ii) au moment où Binance touche le strike (01:06.7), l'oracle est déjà à 65 055.11$, soit 38.89$ sous le strike.
- A6. Spécifique : l'ampleur du basis (moyenne +43.17$, B-F4) dépend de la paire USDT/USD du jour ; sa constance intra-session est en revanche exploitable comme règle de prudence.


## A-G6 — FLUX DIRECTIONNEL NORMALISÉ YES/NO (VUE 300 S + ZOOMS)

- A1. Temps × volume $ signé (haussier positif, baissier négatif), agrégé 30 s ; 1 761 trades.
- A2. Le flux agressif anticipe-t-il ou suit-il le prix ?
- A3. Lecture des extrema et des zooms 15 s.
- A4. t=00:00.515 : −99.25$ ; t=00:48.510 : +1 205.84$ (pic haussier, un seul trade, B-F2) ; t=02:09.684 : −1 196.28$ (pic baissier) ; t=04:49.294 : +32.79$ ; zoom [00:59–01:14] : t=01:04.993 : +174.15$ à contre-tendance.
- A5. Observations brutes : (i) le pic haussier de +1 205.84$ à 00:48.5 précède de 15 s le début de la jambe baissière — le gros flux agressif de cette session est un signal contrarien perdant ; (ii) le flux baissier culmine à 02:09.7, EN MÊME TEMPS que le prix touche 0.04 : le flux confirme, il n'anticipe pas.
- A6. Spécifique : la domination baissière (74.2% du volume, B-F2) est propre à une session DOWN tendancielle.


# 4. PARTIE B — LES CSV

## B-F1 — bbo.csv (CARNET)

- B1. Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 540 lignes ; t de 255 ms à 270 441 ms ; fréquence événementielle (médiane des mises à jour < 1 s en phase active).
- B2. Apport : les 4 prix exacts au tick près, là où le graphe ne donne que les extrema étiquetés.
- B3. Calculs : spread = yes_ask − yes_bid ; somme des asks = yes_ask + no_ask sur les 536 lignes où les deux existent ; prix prévalant à t = dernière ligne ≤ t.
- B4. Résultats :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Événements BBO | 540 | toutes | 540
| Spread YES moyen | 0.0153$ | yes_bid, yes_ask | 538
| Spread YES médian / max | 0.01$ / 0.06$ | yes_bid, yes_ask | 538
| Événements yes_ask+no_ask < 1 | 1 (t=19 526 ms, somme 0.99) | yes_ask, no_ask | 536
| Événements yes_bid+no_bid > 1 | 1 (t=19 526 ms, somme 1.01) | yes_bid, no_bid | 536
| NO ask prévalant à t=103 100 ms | 0.91$ (posé à t=102 564, inchangé jusqu'à t=103 314) | no_ask | 2
| NO bid minimal après t=103 100 | 0.80$ (t=108 728 ms) | no_bid | 1
| Dernier YES bid > 0.05 | t=182 403 ms (0.07$) | yes_bid | 1


- B5. Observations brutes : (i) la fenêtre d'ask NO stable 0.91$ dure 750 ms (102 564→103 314) — largement plus que toute boucle d'exécution ; (ii) le carnet devient unilatéral après t=231 549 ms (yes_bid vide).
- B6. Anomalies : le PDF annonce 49 événements « carnet croisé — suspects » ; le CSV n'en montre que 2 stricts (t=19 526 ms). Les 47 autres sont vraisemblablement des lignes unilatérales/NaN comptées comme incohérentes — bruit de collecte, pas signal exploitable.


## B-F2 — trades.csv (FLUX)

- B1. Colonnes t_ms, usd, direction (+1 haussier / −1 baissier) ; 1 761 lignes ; t de 515 ms à 295 454 ms.
- B2. Apport : volumes exacts par trade, invisibles dans l'agrégat 30 s du graphe.
- B3. Calculs : sommes par direction, par minute, flux net 30 s glissant, médiane.
- B4. Résultats :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Volume total | 29 601.21$ | usd | 1 761
| Volume baissier | 21 968.57$ (74.2%, n=1 136) | usd, direction | 1 136
| Volume haussier | 7 632.65$ (n=625) | usd, direction | 625
| Plus gros trade | 1 205.84$ haussier, t=48 510 ms | usd, direction | 1
| Trade médian | 4.40$ | usd | 1 761
| Flux net minute 0 / 1 / 2 / 3 / 4 | +1 563.49$ / −2 698.83$ / −6 691.37$ / −6 304.93$ / −204.28$ | usd, direction | 1 761
| Flux net 30 s finissant à t=103 000 | −1 530.27$ (n=294) | usd, direction | 294
| Volume taker baissier 90–130 s | 5 627.85$ (n=270) | usd, direction | 270


- B5. Observations brutes : (i) le flux net bascule durablement négatif dès la minute 1 et le reste — cohérent avec A-G6 ; (ii) 5 627.85$ de volume baissier exécuté entre 90 et 130 s prouve qu'une entrée NO de ~4.89$ y trouvait de la contrepartie.
- B6. Anomalies : horodatages non monotones ponctuels (t=6 290 avant t=6 288 ; t=41 977 avant t=41 944) — désordre d'ingestion de flux, sans impact sur les agrégats.


## B-F3 — oracle.csv (CHAINLINK, SOURCE DE RÉSOLUTION)

- B1. Colonnes t_ms, price, ts_src ; 278 lignes ; t de 1 000 à 298 000 ms ; cadence médiane 1 000 ms, gap max 7 000 ms (151 000→158 000 ms) ; ts fallback 0/278 (0.0%).
- B2. Apport : c'est LE prix qui résout le marché (P0-3) ; les graphes ne donnent que 5 étiquettes.
- B3. Calculs : déficit = price − 65 094.00 ; TWAP 30 s finale = moyenne des ticks t ∈ [270 000, 300 000].
- B4. Résultats :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Strike | 65 094.00$ | PDF (dernier tick ≤ t0) | —
| Dernier tick AU-DESSUS du strike | t=30 000 ms (65 094.78$) | t_ms, price | 1
| Oracle à t=103 000 ms | 64 984.84$ (déficit −109.16$) | t_ms, price | 1
| Oracle min / max | 64 945.51$ (t=286 000) / 65 109.06$ (t=5 000) | price | 2
| Plus forte chute 1 s | −13.42$ (t=100 000 ms) | price | 2
| Plus forte hausse 1 s | +11.04$ (t=107 000 ms) | price | 2
| **TWAP 30 s finale** | **64 956.31$, soit −137.69$ vs strike** | price | 29
| Ticks fallback | 0/278 (0.0%) | ts_src | 278


- B5. Observations brutes : (i) l'oracle passe sous le strike à t=31 000 ms et **n'y revient jamais** (269 s consécutives en déficit) ; (ii) la TWAP finale à −137.69$ rend l'issue DOWN insensible à tout tick isolé — exactement ce que le nouveau P0-3 récompense.
- B6. Anomalies : gap de 7 s (151→158 s) : 7 ticks manquants ; un signal fondé sur l'oracle doit tolérer une staleness de 7 s (règle intégrée en C1).


## B-F4 — spot.csv (BINANCE)

- B1. Colonnes t_ms, price ; 25 707 lignes ; t de 468 à 299 911 ms ; multiples ticks par ms (flux direct).
- B2. Apport : la granularité milliseconde que l'oracle n'a pas ; mesure du lead-lag et du basis.
- B3. Calculs : basis = spot − dernier oracle ≤ t sur 25 694 points ; corrélation des rendements 1 s oracle vs spot décalé.
- B4. Résultats :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Ticks | 25 707 | toutes | 25 707
| Premier / dernier | 65 136.55$ (t=468) / 65 009.24$ (t=299 911) | t_ms, price | 2
| Min / Max | 64 984.68$ (t=283 992) / 65 157.38$ (t=3 214) | price | 2
| Basis moyen / médian | +43.17$ / +43.07$ | les deux fichiers | 25 694
| Basis min / max | +20.01$ / +65.72$ | les deux fichiers | 25 694
| Corrélation rendements, spot en avance de 1 s | 0.630 (max) ; 0.621 à 0 s ; 0.245 à 2 s | les deux fichiers | 277 paires
| Plus forte chute spot 1 s | −15.65$ (t=66 s) | price | 2
| Plus forte hausse spot 1 s | +19.68$ (t=23 s) | price | 2


- B5. Observations brutes : (i) l'avance de Binance sur l'oracle est de **0 à 1 seconde seulement** — trop courte pour compenser 219 ms de latence de lecture plus le basis ; (ii) le spot ne passe JAMAIS sous 64 984.68$ alors que l'oracle descend à 64 945.51$ : trader le niveau Binance contre le strike inverse le signal.
- B6. Anomalies : rafales de ticks au même t_ms (12 ticks à t=748) et gap 151–158 s partagé avec l'oracle — collecte par salves ; sans impact sur les agrégats 1 s.


# 5. PARTIE C — STRATÉGIES ET SIMULATION

## C0 — CANDIDATES

Mes cinq observations les plus fortes, d'abord.

- OBS-1 : l'oracle Chainlink passe sous le strike à t=31 s et n'y revient jamais ; déficit −109.16$ à t=103 s, TWAP finale −137.69$ (B-F3 B5).
- OBS-2 : le NO ask reste à 0.91$ pendant une fenêtre stable de 750 ms à t≈103 s, et 5 627.85$ de volume baissier s'exécute entre 90 et 130 s (B-F1 B5, B-F2 B5).
- OBS-3 : le basis Binance−oracle est de +43.17$ en moyenne [+20.01, +65.72] : le spot brut ment sur la distance au strike (B-F4 B5).
- OBS-4 : le carnet croisé n'existe qu'une fois (t=19 526 ms, somme des asks 0.99), pour une vie de 95 ms (B-F1 B4).
- OBS-5 : les jambes ne sont pas monotones : rebonds YES 0.32→0.36 (A-G2 A5) et 0.08→0.17 en 3.4 s (A-G3 A5) ; le flux confirme sans anticiper (A-G6 A5).


Latences de boucle (uniquement le tableau fourni) :

| Candidate | Mécanisme | Boucle (lecture + envoi + confirmation) | Vie du signal | Filtre structurel
|-----|-----|-----|-----|-----
| S1 Arbitrage carnet croisé | Acheter YES ask + NO ask quand somme < 1 | 31 + 31 + 31 = 93 ms | 95 ms (OBS-4) | Marginal, puis éliminé par frais
| S2 Scalp de latence Binance→CLOB | Suivre les chutes spot avant repricing du carnet | 219 + 31 + 31 = 281 ms | 84 à 1 384 ms mesurés (réaction BBO après choc oracle) | **[STRUCTURELLEMENT IMPOSSIBLE]** sur les signaux < 281 ms ; le reste meurt aux frais
| S3 NO-oracle-confirmé | Acheter NO quand l'oracle confirme un déficit > 100$ | 68 + 31 = 99 ms (confirmation +31 → 130 ms) | > 60 s (OBS-1) | PASSE
| S4 Suivi de flux aller-retour | Entrer NO sur flux 30 s < −1 500$, sortir sur inversion du flux | 31 + 31 + 31 = 93 ms | ~30 s | PASSE structurellement


Conformité P0 et chiffrage :

- S1 : conforme P0-2, mais frais taker sur DEUX jambes à p≈0.45/0.55 : 2 × 0.25 × 0.45 × 0.55 ≈ 0.124$/paire contre un edge brut de 0.01$/paire (OBS-4). P&L net −0.114$/paire. Éliminée (P0-1).
- S2 : conforme P0 formellement, mais la réaction BBO mesurée après les 5 plus gros chocs oracle va de 84 ms à 1 384 ms (B-F1/B-F3) : les occurrences rapides sont sous la boucle de 281 ms, et sur les lentes l'edge capté (0.01–0.03$/part) est inférieur aux frais à p≈0.3–0.5 (0.053–0.063$/part). S'ajoute OBS-3 : le niveau Binance brut est faux de +43.17$. Éliminée.
- S3 : conforme P0-1/P0-2/P0-3 (elle EXPLOITE le TWAP) ; chiffrée en C-SIM.
- S4 : entrée au NO ask 0.89$ à t=100 s (flux 30 s −1 555.42$, B-F2), sortie déterministe sur inversion de flux à t≈106.6 s au NO bid 0.83$ (rebond OBS-5) : brut −0.06$/part, frais entrée 0.25×0.89×0.11=0.024$ + sortie 0.25×0.84×0.16=0.034$ → net **−0.118$/part**, soit −0.66$ sur 5.62 parts. Éliminée (P&L net négatif).


CRITÈRE DE SÉLECTION : P&L net réel de la session (après latences r1–r2, slippage r3, frais P0-1 r4), avec capital ≤ 5.00$, sous réserve de passer le filtre structurel. Annoncé ici, avant le tableau.

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----|-----
| S1 Arb croisé | OBS-4 | Oui | 1 | +0.01$/paire | −0.11$ | Éliminée
| S2 Scalp latence | OBS-3, OBS-5 | Oui | 0 exécutable | — | ≤ 0 | [STRUCTURELLEMENT IMPOSSIBLE]
| S3 NO-oracle-confirmé | OBS-1, OBS-2 | Oui | 1 | +0.49$ | **+0.37$** | **RETENUE**
| S4 Flux aller-retour | OBS-5 | Oui | 1 | −0.34$ | −0.66$ | Éliminée


## C1 — STRATÉGIE RETENUE : NO-ORACLE-CONFIRMÉ

```plaintext
PARAMÈTRES
  STRIKE        = 65094.00          # PDF A-G5 (dernier tick oracle <= t0)
  SEUIL_DEFICIT = -100.00 $         # OBS-1 : déficit -109.16$ à t=103s [FRAGILE - 1 SOURCE]
  T_MIN, T_MAX  = 90s, 240s         # OBS-2 : liquidité prouvée 90-130s ;
                                    # borne 240s : carnet unilatéral après 231.5s (B-F1 B5)
  PRIX_MAX_NO   = 0.93 $            # A4/B4 : ask 0.91 disponible ; marge 2 ticks
  STALENESS_MAX = 7s                # B-F3 B6 : gap oracle max observé [FRAGILE - 1 SOURCE]

BOUCLE (à chaque tick oracle, âge lecture 68 ms) :
  SI position_ouverte OU ordre_en_cours : NE RIEN FAIRE          # race (a)
  SI age_tick > STALENESS_MAX : NE PAS TRADER
  SI t IN [T_MIN, T_MAX] ET (oracle - STRIKE) < SEUIL_DEFICIT
     ET no_ask <= PRIX_MAX_NO :                                  # lecture CLOB 31 ms
       feeRateBps = GET /fee (P0-4, obligatoire)                 # P0-1
       ENVOYER ordre FOK acheter NO à no_ask,
         taille = floor_2dec( cash / (no_ask + 0.25*no_ask*(1-no_ask)) )  # P0-1, r5
       SI rejet FOK (niveau disparu) : annulé, re-évaluer au tick suivant  # race (d), r3
  TENIR JUSQU'À RÉSOLUTION (TWAP 30s, P0-3). AUCUNE VENTE.       # race (b) sans objet
  [HYPOTHÈSE n°1] rédemption gasless, coût borné 0.00-0.02$ (P0-1 non vérifiable)
  [HYPOTHÈSE n°2] taille minimale 5 parts (P0-2 non vérifiable) — 5.37 parts la satisfait
```

## C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE.** Cash initial 5.00$, aucune position. Après le trade : 0.0033$ de cash, 5.37 parts NO. À la résolution : 5.3733$.

**RACE CONDITIONS.** RÈGLE (a) : tout signal pendant un ordre en cours est ignoré (drapeau position/ordre unique). RÈGLE (b) : aucune vente n'existe dans la stratégie — position tenue jusqu'à résolution, le cas est structurellement clos. RÈGLE (c) : deux signaux simultanés — priorité unique à la source oracle ; le carnet n'est qu'une condition de prix, jamais un déclencheur. RÈGLE (d) : fill partiel impossible par construction — ordre FOK (P0-2) ; si rejet, trade annulé et réévaluation au tick oracle suivant.

**JOURNAL DE TRADES.** Réalisme appliqué : (r1) exécution décalée de la boucle 99 ms ; (r2) âge du signal oracle 68 ms inclus ; (r3) prix confronté au carnet réel post-décalage ; (r4) frais P0-1 ; (r5) cash respecté au centime.

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----
| 1 | 01:43.000 (oracle 64 984.84$, déficit −109.16$) | 01:43.099 (+99 ms) | NO ask 0.91$ (prévalant, posé à 01:42.564, stable jusqu'à 01:43.314) | 5.37 parts (coût 4.8867$) | 05:00.000 (résolution TWAP, DOWN) | 1.00$ | 0.00$ (ask inchangé sur la fenêtre de 750 ms couvrant l'exécution) | 0.1100$ | **+0.3733$** | 0.0033$ puis 5.3733$ après rédemption


Vérification r5 : 4.8867 (coût) + 0.1100 (frais) = 4.9967 ≤ 5.00. ✓

**RÉSULTATS.**

| Poste | Valeur
|-----|-----|-----|-----|-----
| Trades gagnants / perdants | 1 (+0.3733$) / 0 (0.00$)
| Frais totaux | 0.11$
| BÉNÉFICE NET | **+0.3733$**
| Capital final vs 5.00$ | **5.3733$ ≈ 5.37$, soit +7.47%**
| Pire perte latente | −0.7007$ à t=108 728 ms (NO bid 0.80$ : 5.37×0.80+0.0033 = 4.2993$)
| Drawdown max | −14.01%
| P&L THÉORIQUE (sans r1–r4) | +0.4945$ (+9.89%) : 5.00/0.91 = 5.4945 parts payées 1.00$
| Coût du réalisme | 0.1212$ (frais 0.1100$ + arrondi de taille 0.0112$ ; latence 0.0000$, prouvé ligne « Slippage »)


# 6. PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----
| D1 Latence de boucle | 99 ms n'altère pas le prix | Ask NO 0.91$ stable 750 ms autour de l'exécution (B-F1 B4) | 0.0000$ — prouvé | [SANS IMPACT]
| D2 Fraîcheur du signal | Tick oracle âgé ≤ 1 068 ms (cadence 1 000 + lecture 68) | Pire hausse oracle 1 s : +11.04$ (B-F3) → déficit résiduel pire cas −98.12$, sous le seuil → entrée reportée d'1 tick, prix identique 0.91$ | Borné 0.00$ à −0.01$ (méthode : re-jeu au tick suivant) | [DÉGRADE]
| D3 Slippage / profondeur | Top-of-book ≥ 5.37 parts (4.89$) | Tailles absentes du BBO = DONNÉE NON DISPONIBLE ; mais 5 627.85$ de volume baissier exécuté sur 90–130 s, trade médian 4.40$ (B-F2 B4) ; pire cas : repricing FOK à 0.92$ → net +0.3186$ | Borné −0.0547$ | [DÉGRADE]
| D4 Frais complets et gas | fee = C×0.25×p×(1−p) ; gas 0 à l'ordre | P0-1 ; frais payés 0.1100$ ; rédemption bornée 0.00–0.02$ ([HYPOTHÈSE n°1]) | −0.1100$ à −0.1300$ | [DÉGRADE]
| D5 Niveau disparu / fill partiel | FOK annule tout fill partiel | P0-2 (FOK vérifié) ; règle (d) ; re-jeu au tick suivant, ask 0.91$ toujours coté à 103 314 ms (B-F1) | Borné 0.00$ à −0.0547$ (un tick) | [DÉGRADE]
| D6 Défaillances techniques | Timeout = pas de position = pas de perte | Ordre unique, aucune jambe orpheline possible ; sans position le P&L est 0.0000$ par construction | 0.0000$ — prouvé (aucune exposition sans fill) | [SANS IMPACT]
| D7 Passage à l'échelle | Mécanisme viable à 5$ | Volume taker total de la session 29 601.21$ (B-F2) ; l'edge est de 0.09$/part : au-delà de ~10³$ de taille, l'impact prix consomme les 9 ticks d'edge (méthode : edge/tick_size = 9 niveaux de profondeur max) | Nul à 5$ ; mécanisme éteint vers 10³$ | [DÉGRADE] à l'échelle
| D8 Session calme | Sans déficit >100$, la stratégie ne trade pas | Seuil −100$ calibré sur UNE session (OBS-1) ; sur session calme : 0 trade, P&L 0.00$, aucune perte — mais aucune preuve de fréquence du régime tendanciel | 0.00$ sur session calme ; espérance long-terme NON CALCULABLE (1 session) | [DÉGRADE]
| D9 Conformité nouveau Polymarket | La résolution est la TWAP 30 s Chainlink | P0-3 (07/08/2026) ; la stratégie lit la source exacte de résolution et la TWAP finale −137.69$ couvre 1.38× le seuil d'entrée | Positif : le TWAP réduit le risque de bascule au dernier tick | [SANS IMPACT]


**VERDICT GLOBAL — application mécanique.** Aucun [DÉTRUIT]. P&L net réel positif (+0.3733$). Zéro violation P0. Mais le critère [FIABLE] exige que D8 ne soit pas une dépendance à un événement unique : ici, tout le P&L provient d'un seul trade sur une seule session tendancielle, et le seuil −100$ est [FRAGILE - 1 SOURCE]. Verdict : **[FRAGILE]**, étiquette **[SESSION-SPÉCIFIQUE]**.

# 7. AUTO-CONTRÔLE FINAL

- Contrôle 1 → Phase 0 : P0-1 à P0-4 sourcés (docs.polymarket.com, consultation 09/08/2026 ; changement TWAP daté 07/08/2026) ; taille minimale et frais de rédemption marqués DONNÉE NON VÉRIFIABLE avec hypothèses conservatrices. ✓
- Contrôle 2 → Valeurs : A-G1:6, A-G2:7, A-G3:6, A-G4:6, A-G5:6, A-G6:5 ; B-F1:8, B-F2:8, B-F3:8, B-F4:8. ✓
- Contrôle 3 → Traçabilité : SEUIL_DEFICIT←OBS-1(B-F3 B5) ; T_MIN/T_MAX←OBS-2(B-F2 B5)/B-F1 B5 ; PRIX_MAX_NO←B-F1 B4 ; STALENESS_MAX←B-F3 B6. ✓
- Contrôle 4 → Boucle : oracle 68 ms + envoi CLOB 31 ms = 99 ms (confirmation +31 = 130 ms), appliquée à l'unique entrée du journal (103 000 → 103 099 ms). ✓
- Contrôle 5 → Comptabilité : 5.00 − 4.8867 − 0.1100 = 0.0033 ; 0.0033 + 5.37×1.00 = 5.3733 = capital final annoncé, au centime près (au dix-millième). ✓
- Contrôle 6 → D1–D9 tous traités ; les trois [SANS IMPACT] portent leur preuve chiffrée (0.0000$ mesuré ou par construction) ; verdict [FRAGILE] conforme aux règles mécaniques. ✓
- Contrôle 7 → Unicité des chiffres : 5.37$/5.3733$, +7.47%, +0.3733$, 0.11$, 1 trade 1/0, −137.69$, TWAP 30 s, [FRAGILE], [SESSION-SPÉCIFIQUE] identiques entre ouverture et corps. ✓


# 8. CLÔTURE

Vous avez la stratégie, sa simulation au centime et ses limites. Le mécanisme gagne ici parce qu'il lit la source de résolution elle-même, pas un proxy. Je ne l'engagerais pas au-delà de ce capital sans une deuxième session pour confirmer le seuil.

---

Les quatre CSV ont été copiés dans `/vercel/share/v0-project/data/` pour permettre les calculs reproductibles (basis, lead-lag, TWAP, flux, journal de trade) exécutés en Python. Si vous voulez, je peux transformer cette analyse en dashboard interactif (courbes carnet/oracle/flux + journal de simulation) dans l'app.


