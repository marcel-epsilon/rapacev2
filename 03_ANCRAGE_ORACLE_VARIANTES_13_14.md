# 03 — ANCRAGE ORACLE VARIANTES (13-14)

**Fichiers sources regroupés :** 13ancrage.md, 14ancrage.md

---

## Source : 13ancrage.md

Voici le speech complet, conforme au format imposé.

---

# 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Vous voulez savoir ce que vaut «ANCRAGE À L'ORACLE» sur cette session de 5 minutes : voici le verdict, tout de suite.

- Stratégie testée : «ANCRAGE À L'ORACLE» — je la lis ainsi : l'oracle Chainlink est la seule vérité du règlement, on achète le côté que l'oracle désigne dès que sa marge sur le strike est durablement établie.
- Capital final : 6,67$ contre 5,00$ de départ, soit un rendement net de +33,33%.
- Trades : 2 trades, 2 gagnants, 0 perdant.
- Adaptations apportées à la stratégie : 3, toutes consignées avant simulation.
- VERDICT SUR LA STRATÉGIE : [VALIDÉE SUR SESSION] — étiquette de portée [PATTERN CANDIDAT] pour la décote persistante du marché face à l'oracle, le reste [SESSION-SPÉCIFIQUE].


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

# 1. CADRE D'ANALYSE CHOISI

J'ai choisi un cadre statistique de microstructure événementielle : chaque fichier est un flux horodaté à la milliseconde (288 ticks oracle, 3 611 ticks spot, 680 événements de carnet, 1 218 trades) sur un horizon de 300 secondes. À cette granularité et sur cet horizon, ni le fondamental ni le chartisme n'ont de prise ; ce qui est mesurable, c'est l'écart entre ce que dit l'oracle et ce que cote le carnet, événement par événement. Tout ce qui suit est donc du comptage, des écarts au strike et des états de carnet à timestamps exacts.

---

# PARTIE A — LES 6 GRAPHES DU PDF

## A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (680 ÉVÉNEMENTS BBO)

**A1. IDENTITÉ**
Graphique 1, «Carnet YES/NO complet — 680 événements BBO (précision ms) | incohérents : 22». Axe X : temps de t=0,0s à t=300,0s ; axe Y : prix des contrats YES et NO en $, bornés 0,00-1,00$. Vue 300 s plus trois zooms 15 s (t=285,0-300,0s ; t=74,0-89,0s ; t=113,0-128,0s).

**A2. QUESTION**
Que fait le prix YES quand l'oracle s'éloigne du strike ? À quelle vitesse le carnet converge-t-il vers 1,00$ ou 0,00$ en fin de session ?

**A3. COMMENT je l'ai exploité**
Lecture des niveaux bid/ask YES aux timestamps clés : ouverture, plateaux, creux intermédiaire, rampe finale ; croisement des courbes YES et NO avec la ligne 0,50$.

**A4. VALEURS EXTRAITES**

- t=0,2s : YES coté 0,49/0,50$
- t=27,4s : YES coté 0,74/0,75$
- t=101,0s (lecture au dernier événement t=99,0s) : YES coté 0,79/0,80$
- t=186,1s : YES coté 0,56/0,62$ — creux de la session après l'ouverture
- t=272,7s : ask YES atteint 0,95$ pour la première fois
- t=296,1s : bid YES à 0,99$, dernier événement du carnet


**A5. OBSERVATIONS BRUTES**

- Le carnet part à 0,49/0,50$ et ne repasse jamais sous 0,50$ au bid après t=27,4s (min post-entrée : 0,56$ à t=186,1s).
- La revalorisation 0,50$ → 0,75$ se fait dans les 28 premières secondes, en même temps que la marge oracle passe +10$.
- Le carnet ne dépasse 0,95$ à l'ask qu'à t=272,7s, soit 27,3s avant la fin, alors que l'oracle est au-dessus du strike depuis t=9,0s.


**A6. SPÉCIFIQUE À LA SESSION**
Le creux à 0,56$ (t=186,1s) survient alors que la marge oracle reste positive (+6$ à +8$ sur cette plage, B-F1) : c'est un stress de carnet propre à cette session, non reproductible comme règle. Le compteur «incohérents : 22» du titre est une donnée du PDF que je ne retrouve pas à l'identique dans le CSV (4 croisements mesurés, B-F3) — je le classe bruit de comptage d'affichage.

## A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT

**A1. IDENTITÉ**
Graphique 2, «Spot Binance (flux direct) vs Oracle Chainlink — strike exact (dernier tick ≤ t0)». Axe X : t=0,0s à t=300,0s ; axe Y : prix BTC en $. Ligne horizontale du strike à 64 250,16$. Mêmes trois zooms 15 s.

**A2. QUESTION**
L'oracle et le spot racontent-ils la même histoire ? Le prix repasse-t-il sous le strike, et quand ?

**A3. COMMENT je l'ai exploité**
Lecture des deux courbes aux extrêmes et aux traversées de la ligne de strike ; mesure visuelle de l'écart vertical spot-oracle.

**A4. VALEURS EXTRAITES**

- t=0,0s : oracle = strike = 64 250,16$
- t=8,0s : oracle au minimum 64 249,41$, seul passage net sous le strike
- t=8,4s : spot au minimum 64 304,64$, soit 54,48$ AU-DESSUS du strike
- t=101,0s : oracle au maximum 64 271,98$ (+21,82$ de marge)
- t=82,0s : spot au maximum 64 330,00$
- t=298,0s : oracle final 64 265,66$ (+15,50$ de marge)


**A5. OBSERVATIONS BRUTES**

- Le spot ne touche jamais le strike : écart minimal +54,48$ (t=8,4s).
- L'oracle ne passe sous le strike que sur 2 ticks (t=2,0s et t=8,0s), dans les 8 premières secondes.
- L'écart spot-oracle est un plateau stable : de +52,90$ à +64,55$ sur toute la session.


**A6. SPÉCIFIQUE À LA SESSION**
Le biais constant spot-oracle de +57,51$ en moyenne (B-F2) est un décalage de sources propre à ce flux et à ce jour ; sa valeur absolue n'est pas généralisable, seule sa stabilité intra-session l'est éventuellement.

## A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ YES/NO (1 218 TRADES)

**A1. IDENTITÉ**
Graphique 3, «Flux directionnel normalisé YES/NO (1218 trades)». Axe X : t=0,0s à t=300,0s ; axe Y : volume en $. Vue 300 s et trois zooms 15 s.

**A2. QUESTION**
Où le volume se concentre-t-il, et de quel côté quand il se concentre ?

**A3. COMMENT je l'ai exploité**
Sommation visuelle des barres par fenêtres, repérage de la plus grosse barre isolée, comparaison des zooms entre eux.

**A4. VALEURS EXTRAITES**

- t=0,0-60,0s : 3 249,18$ de volume
- t=74,0-89,0s (zoom 2) : 83 trades, 1 004,05$
- t=113,0-128,0s (zoom 3) : 64 trades, 448,45$
- t=285,0-300,0s (zoom 1) : 106 trades, 3 912,73$
- t=297,1s : plus gros trade de la session, 2 011,16$ côté YES


**A5. OBSERVATIONS BRUTES**

- Le volume total 14 968,71$ se concentre aux deux extrémités : le premier 60 s et le dernier 60 s portent 8 865,92$ à eux deux (B-F4).
- La dernière fenêtre de 15 s pèse 3 912,73$, soit plus que tout le zoom milieu de session multiplié par huit.


**A6. SPÉCIFIQUE À LA SESSION**
Le trade unique de 2 011,16$ à t=297,1s représente 13,4% du volume total : événement isolé, classé bruit — aucune règle ne peut s'appuyer sur un trade unique. [FRAGILE - 1 SOURCE]

## A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE (YES/NO NORMALISÉS)

**A1. IDENTITÉ**
Graphique 4, «Imbalance directionnelle (YES/NO normalisés)». Axe X : t=0,0s à t=300,0s ; axe Y : ratio d'imbalance acheteur YES. Mêmes zooms.

**A2. QUESTION**
Le flux confirme-t-il ou conteste-t-il ce que dit l'oracle, minute par minute ?

**A3. COMMENT je l'ai exploité**
Lecture du ratio par tranches de 60 s et sur la fenêtre finale de 30 s.

**A4. VALEURS EXTRAITES**

- t=0,0-60,0s : imbalance YES 68,5%
- t=60,0-120,0s : 69,0%
- t=120,0-180,0s : 41,6% — seule minute vendeuse
- t=180,0-240,0s : 64,8%
- t=240,0-300,0s : 87,8%
- t=270,0-300,0s : 89,3%


**A5. OBSERVATIONS BRUTES**

- 4 minutes sur 5 sont acheteuses YES ; l'imbalance de session est 71,7% (B-F4).
- La seule minute vendeuse (41,6%, t=120,0-180,0s) précède exactement le creux du carnet à 0,56$ (A-G1, t=186,1s).


**A6. SPÉCIFIQUE À LA SESSION**
Le pic final à 89,3% sur 30 s reflète la course au règlement d'une session déjà décidée par l'oracle : niveau non généralisable, seul le sens l'est éventuellement.

## A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS (ÉVÉNEMENTIELS)

**A1. IDENTITÉ**
Graphique 5, «Spreads YES et NO absolus (événementiels)». Axe X : t=0,0s à t=300,0s ; axe Y : spread en $. Mêmes zooms.

**A2. QUESTION**
Combien coûte l'exécution, et quand devient-elle chère ?

**A3. COMMENT je l'ai exploité**
Lecture des pics de spread et du niveau de base entre les pics.

**A4. VALEURS EXTRAITES**

- t=1,6s : spread maximum de la session, 0,08$
- t=25,9s : pic secondaire à 0,07$, juste avant mon point d'entrée
- t=27,4s : spread 0,01$ au moment de l'entrée (0,74/0,75$)
- t=186,1s : spread élargi à 0,06$ (0,56/0,62$) au creux du carnet
- spread moyen de session : 0,0197$ (B-F3)


**A5. OBSERVATIONS BRUTES**

- Le spread de base est de 0,01-0,02$ sur l'essentiel de la session : l'exécution au ask coûte peu.
- Les élargissements à 0,05$ et plus sont concentrés sur les 26 premières secondes et sur le stress de t=186,1s.


**A6. SPÉCIFIQUE À LA SESSION**
Le pic de 0,08$ à t=1,6s appartient à la phase de formation du carnet, présente dans toute ouverture : le timestamp est spécifique, le phénomène d'ouverture large ne l'est pas.

## A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ**
Graphique 6, «Écart inter-carnets YES vs NO (incohérents : 22)». Axe X : t=0,0s à t=300,0s ; axe Y : écart entre le mid YES et (1 − mid NO) en $. Mêmes zooms.

**A2. QUESTION**
Existe-t-il un arbitrage interne entre les deux carnets, exploitable avec 5,00$ ?

**A3. COMMENT je l'ai exploité**
Recherche du plus grand écart absolu entre mid YES et 1 − mid NO sur les 678 lignes complètes.

**A4. VALEURS EXTRAITES**

- écart moyen : 0,0000$ (0,0000074$ exactement, B-F3)
- écart maximum : 0,0050$ à t=294,6s
- nombre de lignes à écart non nul : 1 sur 678
- croisements bid>ask : 4, à t=32,6s, t=203,0s (deux fois) et t=237,1s
- lignes incomplètes (un côté absent) : 2 sur 680


**A5. OBSERVATIONS BRUTES**

- Les carnets YES et NO sont le miroir exact l'un de l'autre à 0,005$ près : aucun arbitrage interne n'existe sur cette session.


**A6. SPÉCIFIQUE À LA SESSION**
L'unique écart de 0,0050$ (t=294,6s) et les 4 croisements sont des artefacts de mise à jour asynchrone à la milliseconde : bruit. Ce graphe n'apporte aucune règle de trading ; je le garde comme contrôle de qualité des données, il valide que le prix YES suffit à tout décrire.

---

# PARTIE B — LES FICHIERS CSV

## B-F1 — ORACLE.CSV

**B1. IDENTITÉ**
oracle.csv, colonnes t_ms, price, ts_src ; 288 lignes de données ; période t=0,0s à t=298,0s ; cadence de 1 tick par seconde avec 11 trous supérieurs à 1 s (trou max 2,0s). Colonne ts_src : valeur unique «payload» sur les 288 lignes — utilisée uniquement comme contrôle de source, écartée ensuite car sans variance.

**B2. APPORT**
La seule série qui décide du règlement : la marge exacte oracle − strike à chaque seconde, illisible à cette précision sur le graphe 2.

**B3. COMMENT**
Strike = price du premier tick (t=0,0s). Marge(t) = price(t) − strike. Moyenne arithmétique, écart-type de population, min/max avec timestamps, comptages de seuils sur les 288 lignes, colonnes t_ms et price.

**B4. RÉSULTATS CHIFFRÉS**
Le tableau des mesures oracle, ligne par ligne :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Strike (tick t=0,0s) | 64 250,16$ | price | 1
| Minimum / timestamp | 64 249,41$ / t=8,0s | t_ms, price | 288
| Maximum / timestamp | 64 271,98$ / t=101,0s | t_ms, price | 288
| Moyenne | 64 258,99$ | price | 288
| Écart-type | 6,46$ | price | 288
| Valeur finale / marge finale | 64 265,66$ / +15,50$ | t_ms, price | 1
| Ticks sous le strike | 2 (t=2,0s ; t=8,0s) | t_ms, price | 288
| Ticks à marge ≥ +10$ | 95 sur 288 ; premier à t=27,0s | t_ms, price | 288
| Marge min après t=28,0s | +3,29$ à t=283,0s | t_ms, price | 271


**B5. OBSERVATIONS BRUTES**

- L'oracle est au-dessus du strike sur 286 ticks sur 288 ; les 2 exceptions tiennent dans les 8 premières secondes.
- La marge atteint +10$ dès t=27,0s et s'y maintient sur 2 ticks consécutifs à t=28,0s.
- La marge atteint +15$ sur 2 ticks consécutifs à t=59,0s.
- Après t=28,0s, la marge ne redescend jamais sous +3,29$ (t=283,0s) : elle ne remenace jamais le strike.


**B6. SPÉCIFIQUE À LA SESSION**
Les 11 trous d'échantillonnage (max 2,0s) sont du bruit de flux, sans impact : aucun ne coïncide avec un mouvement de marge supérieur à l'écart-type. Le fait que la marge finale (+15,50$) soit confortable est un résultat de session, pas une propriété de la stratégie.

## B-F2 — SPOT.CSV

**B1. IDENTITÉ**
spot.csv, colonnes t_ms, price ; 3 611 lignes ; période t=0,7s à t=299,9s ; 460 timestamps uniques (plusieurs ticks par seconde). Toutes colonnes utilisées.

**B2. APPORT**
La confirmation indépendante : un deuxième flux de prix BTC qui dit si l'oracle est isolé ou soutenu.

**B3. COMMENT**
Min/max/moyenne/écart-type sur price ; basis(t) = spot(t) − dernier tick oracle ≤ t, calculé sur les 3 611 lignes appariées.

**B4. RÉSULTATS CHIFFRÉS**
Les mesures spot :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Minimum / timestamp | 64 304,64$ / t=8,4s | t_ms, price | 3 611
| Maximum / timestamp | 64 330,00$ / t=82,0s | t_ms, price | 3 611
| Moyenne | 64 317,55$ | price | 3 611
| Écart-type | 7,26$ | price | 3 611
| Premier / dernier prix | 64 304,65$ / 64 323,61$ | price | 2
| Écart min spot − strike | +54,48$ | price (+ strike B-F1) | 3 611
| Basis spot − oracle : min / max | +52,90$ (t=80,1s) / +64,55$ (t=294,6s) | price ×2 fichiers | 3 611
| Basis moyen | +57,51$ | price ×2 fichiers | 3 611


**B5. OBSERVATIONS BRUTES**

- Le spot ne s'approche jamais à moins de 54,48$ du strike : le deuxième flux ne conteste jamais le règlement YES.
- Le basis reste dans un couloir de 11,65$ de large (52,90$ à 64,55$) : les deux sources bougent ensemble.


**B6. SPÉCIFIQUE À LA SESSION**
Le niveau absolu du basis (+57,51$ en moyenne) est propre à ce couple de flux ce jour-là : verdict bruit pour sa valeur, signal pour sa stabilité. Aucun trou de données notable (460 secondes couvertes sur 300 s de session en timestamps multiples).

## B-F3 — BBO.CSV

**B1. IDENTITÉ**
bbo.csv, colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 680 lignes dont 678 complètes (2 lignes avec un côté NO absent) ; période t=0,2s à t=296,1s ; événementiel, non cadencé. Toutes colonnes utilisées.

**B2. APPORT**
Les prix exécutables exacts — ce que ni le graphe 1 (lecture visuelle) ni les trades ne donnent : le ask au moment précis où je veux acheter.

**B3. COMMENT**
Spread = yes_ask − yes_bid sur les 678 lignes complètes ; cohérence = |mid YES − (1 − mid NO)| ; extraction du dernier quote ≤ t pour t=27,4s et t=58,8s ; min de yes_bid après t=28,0s.

**B4. RÉSULTATS CHIFFRÉS**
Les mesures de carnet :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Premier quote YES | 0,49/0,50$ (t=0,2s) | yes_bid, yes_ask | 1
| Dernier bid YES | 0,99$ (t=296,1s) | yes_bid | 1
| Spread moyen / max | 0,0197$ / 0,08$ (t=1,6s) | yes_bid, yes_ask | 678
| Croisements bid>ask | 4 (t=32,6s ; t=203,0s ×2 ; t=237,1s) | yes_bid, yes_ask | 678
| Erreur de cohérence YES vs 1−NO : max | 0,0050$ (t=294,6s) | 4 colonnes | 678
| Quote au signal 1 (t=27,4s) | 0,74/0,75$ | yes_bid, yes_ask | 1
| Quote au signal 2 (t=58,8s) | 0,74/0,75$ | yes_bid, yes_ask | 1
| Min yes_bid après t=28,0s | 0,56$ (t=186,1s) | t_ms, yes_bid | 616
| Premier ask ≥ 0,95$ | t=272,7s | t_ms, yes_ask | 680


**B5. OBSERVATIONS BRUTES**

- Au moment où l'oracle affiche +13,09$ de marge (t=28,0s, B-F1), le marché ne cote YES qu'à 0,75$ : le carnet paie 0,25$ de gain potentiel pour un règlement que l'oracle désigne déjà.
- Cette décote persiste : à t=58,8s, même quote 0,74/0,75$ pour une marge oracle de +15,53$.
- Le carnet ne rejoint la quasi-certitude (ask ≥ 0,95$) qu'à t=272,7s, 245 s après l'oracle.


**B6. SPÉCIFIQUE À LA SESSION**
2 lignes incomplètes (t=0,2s et t=296,1s : côté NO absent) : bordures de session, bruit. Les 4 croisements négatifs sont des asynchronismes de mise à jour, bruit également.

## B-F4 — TRADES.CSV

**B1. IDENTITÉ**
trades.csv, colonnes t_ms, usd, direction ; 1 218 lignes ; période t=0,3s à t=298,4s. Toutes colonnes utilisées.

**B2. APPORT**
Le comportement réel des autres participants : volumes exécutés et sens, invisibles dans le BBO.

**B3. COMMENT**
Somme de usd par direction (+1 acheteur YES, −1 vendeur) ; imbalance = volume(+1)/volume total ; découpes par fenêtres t_ms.

**B4. RÉSULTATS CHIFFRÉS**
Les mesures de flux :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Volume total | 14 968,71$ | usd | 1 218
| Trades / volume côté YES | 781 / 10 731,56$ | usd, direction | 781
| Trades / volume côté NO | 437 / 4 237,16$ | usd, direction | 437
| Imbalance de session | 71,7% YES | usd, direction | 1 218
| Taille moyenne | 12,29$ | usd | 1 218
| Plus gros trade | 2 011,16$ (t=297,1s, YES) | t_ms, usd, direction | 1
| Volume premiers 60 s | 3 249,18$ | t_ms, usd | 260
| Volume / imbalance derniers 30 s | 4 683,78$ / 89,3% | t_ms, usd, direction | 1 218


**B5. OBSERVATIONS BRUTES**

- Le flux confirme l'oracle sur 4 minutes sur 5 : imbalance YES de 68,5%, 69,0%, 64,8% et 87,8% par minute, contre une seule minute vendeuse à 41,6% (t=120,0-180,0s).
- Le tiers du volume de session (4 683,78$ sur 14 968,71$) s'exécute dans les 30 dernières secondes, à 89,3% côté YES.


**B6. SPÉCIFIQUE À LA SESSION**
Le trade de 2 011,16$ à t=297,1s est une anomalie de taille (164 fois la taille moyenne) : verdict bruit, non exploitable. [FRAGILE - 1 SOURCE]

---

# PARTIE C — LA STRATÉGIE «ANCRAGE À L'ORACLE»

## C0 — INTERPRÉTATION OPÉRATIONNELLE

Voici les cinq faits les plus solides des Parties A et B, ceux sur lesquels tout repose :

1. OBS-1 : l'oracle est au-dessus du strike sur 286 ticks sur 288, et sa marge finale est +15,50$ (B-F1, B5).
2. OBS-2 : la marge atteint +10$ sur 2 ticks consécutifs dès t=28,0s et ne redescend jamais sous +3,29$ ensuite (B-F1, B5).
3. OBS-3 : à cet instant, le marché ne cote YES qu'à 0,74/0,75$, et encore 0,74/0,75$ à t=58,8s pour +15,53$ de marge (B-F3, B5).
4. OBS-4 : le flux exécuté confirme l'oracle — 71,7% du volume côté YES, 4 minutes acheteuses sur 5 (B-F4, B5).
5. OBS-5 : le spot Binance ne s'approche jamais à moins de 54,48$ du strike — le second flux ne conteste jamais l'oracle (B-F2, B5).


PRINCIPE DIRECTEUR : l'oracle Chainlink est la seule source de vérité du règlement ; s'y ancrer signifie prendre position dans le sens de la marge oracle − strike dès qu'elle est durablement établie, en ignorant le bruit du carnet, et ne sortir que si l'oracle lui-même reperd son ancrage.

La traduction en règles chiffrées, chacune avec sa source :

| Règle | Formulation chiffrée | Observations sources (A5/B5)
|-----|-----|-----|-----
| Entrée | Acheter YES au ask quand marge oracle ≥ +10$ sur 2 ticks consécutifs | OBS-2 (B-F1 B5), OBS-3 (B-F3 B5)
| Renforcement | 2e tranche quand marge ≥ +15$ sur 2 ticks consécutifs | OBS-2, OBS-3 (quote identique 0,75$ à t=58,8s)
| Sortie de protection | Vendre tout au bid si marge < +2$ sur 2 ticks consécutifs | OBS-2 (plancher observé +3,29$) ; A-G2 A5 (2 ticks sous strike en ouverture)
| Sortie normale | Tenir jusqu'au règlement, payé 1,00$ si oracle final > strike | OBS-1 [HYPOTHÈSE n°3]
| Dimensionnement | 2 tranches de 2,50$ chacune, capital engagé max 5,00$ | contrainte de capital + OBS-3 (décote 0,25$ disponible)
| Exécution | Achat au yes_ask du dernier BBO ≤ signal, fill intégral | B-F3 B4 (spread 0,01$ aux deux signaux) [HYPOTHÈSE n°2]
| Frais | 0,00$ par trade | aucune colonne frais dans aucun CSV [HYPOTHÈSE n°1]


Hypothèses non dérivées des données, une par ligne :

- [HYPOTHÈSE n°1] : frais de transaction nuls — aucun des 4 CSV ne contient de colonne frais.
- [HYPOTHÈSE n°2] : exécution intégrale au ask affiché — aucune donnée de profondeur de carnet disponible.
- [HYPOTHÈSE n°3] : règlement du contrat YES à 1,00$ si oracle final > strike — règle de marché binaire, non présente dans les fichiers.


## C1-ADAPT — JOURNAL DES ADAPTATIONS

Trois calibrations, toutes décidées sur les chiffres A/B, toutes avant simulation :

| # | Paramètre/Règle d'origine | Adaptation | Justification chiffrée (source)
|-----|-----|-----|-----
| 1 | «Marge durablement établie» (non chiffré) | Seuil fixé à +10$ sur 2 ticks consécutifs [ADAPTATION n°1] | +10$ = 1,5 fois l'écart-type oracle de 6,46$ (B-F1 B4) ; 95 ticks sur 288 le franchissent, premier à t=27,0s (B-F1 B4)
| 2 | Entrée en une fois de 5,00$ | Entrée fractionnée : 2,50$ à +10$, 2,50$ à +15$ [ADAPTATION n°2] | 2 ticks sous le strike existent en ouverture (B-F1 B4) : engager 100% sur le premier signal expose tout le capital à un retournement précoce
| 3 | «Sortir si l'ancrage est perdu» (non chiffré) | Stop : marge < +2$ sur 2 ticks consécutifs [ADAPTATION n°3] | +2$ est sous le plancher post-entrée observé +3,29$ (B-F1 B4) tout en restant au-dessus du strike : le stop ne se déclenche que si l'oracle menace réellement le règlement


Aucune de ces trois adaptations ne contredit le PRINCIPE DIRECTEUR : toutes lisent exclusivement l'oracle.

## C2 — LA STRATÉGIE ADAPTÉE EN PSEUDO-CODE

```plaintext
strike = oracle.price[t=0]                     # B-F1 B4 : 64 250,16$
cash = 5,00$ ; position = 0 ; tranche = 2,50$
lock = FAUX                                    # verrou d'ordre (RACE a)

POUR chaque tick oracle t (cadence 1 s, B-F1 B1) :
  marge = oracle.price[t] − strike

  # ENTRÉE 1 [ADAPTATION n°1]
  SI tranche1 non prise ET marge ≥ 10 ET marge_precedente ≥ 10 :
      ask = dernier yes_ask ≤ t               # B-F3 B4 : 0,75$ à t=27,4s
      ACHETER qty = tranche/ask               # fill intégral [HYPOTHÈSE n°2]
      cash −= tranche                          # frais 0 [HYPOTHÈSE n°1]

  # ENTRÉE 2 [ADAPTATION n°2]
  SI tranche2 non prise ET marge ≥ 15 ET marge_precedente ≥ 15 :
      ask = dernier yes_ask ≤ t               # B-F3 B4 : 0,75$ à t=58,8s
      ACHETER qty = tranche/ask ; cash −= tranche

  # STOP [ADAPTATION n°3]
  SI position > 0 ET marge < 2 ET marge_precedente < 2 :
      VENDRE tout au dernier yes_bid ≤ t

# RÈGLEMENT [HYPOTHÈSE n°3]
SI oracle.final > strike :                     # B-F1 B4 : +15,50$
    cash += position × 1,00$
```

## C3 — TABLE DE CONVERGENCE

Chaque règle, avec les pièces pour et contre :

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| Entrée à marge ≥ +10$ ×2 ticks | B-F1 (95 ticks ≥ +10$) ; A-G2 (oracle décollé du strike) ; B-F4 (flux 71,7% YES) ; B-F2 (spot jamais sous strike) | aucune | CONFIRMÉE - 4 SOURCES
| Renforcement à ≥ +15$ | B-F1 (marge tenue) ; B-F3 (quote inchangée 0,75$ : décote intacte) | aucune | CONFIRMÉE - 2 SOURCES
| Stop à marge < +2$ ×2 ticks | B-F1 (plancher +3,29$ : seuil jamais touché, donc jamais de fausse sortie) | aucune pièce ne le teste réellement (jamais déclenché) | [FRAGILE - 1 SOURCE]
| Tenue jusqu'au règlement | B-F1 (marge finale +15,50$) ; A-G1 (bid 0,99$ à t=296,1s) ; B-F4 (89,3% YES sur les 30 dernières s) | A-G1/B-F3 : creux MTM à 0,56$ (t=186,1s) — coût de portage psychologique, pas de contradiction du règlement | CONFIRMÉE - 3 SOURCES
| Exécution au ask sans profondeur | B-F3 (spread 0,01$ aux deux signaux) | aucune donnée de profondeur | [FRAGILE - 1 SOURCE]


Je vous le dis explicitement : le stop et l'exécution sont marqués FRAGILES, une seule source les soutient chacun.

## C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

### ÉTAT DU PORTEFEUILLE

- t=0,0s : cash 5,00$ ; positions : aucune ; valeur totale 5,00$.
- t=28,0s : signal 1 (marge +13,09$, B-F1 B4). Achat 3,3333 parts YES à 0,75$ (B-F3 B4) = 2,50$. Cash 2,50$ ; position 3,3333 parts à 0,75$ ; valeur (au bid 0,74$) 4,97$.
- t=59,0s : signal 2 (marge +15,53$, B-F1 B4). Achat 3,3333 parts à 0,75$ = 2,50$. Cash 0,00$ ; position 6,6667 parts ; valeur (au bid 0,74$) 4,93$.
- t=186,1s : creux mark-to-market — bid 0,56$ (B-F3 B4), valeur 3,73$. Stop non déclenché : la marge oracle vaut alors plus de +3,29$ (B-F1 B4), au-dessus du seuil de +2$.
- t=300,0s : règlement — oracle final 64 265,66$ > strike (B-F1 B4). Encaissement 6,6667 parts × 1,00$ = 6,6667$. Cash final 6,67$.


### RACE CONDITIONS

(a) Signal d'achat pendant un ordre en cours : chaque tick oracle est traité séquentiellement à cadence 1 s, et un verrou d'actif est posé à l'émission d'un ordre. RÈGLE : verrou par actif — tout signal reçu pendant qu'un ordre est vivant est rejeté, jamais mis en file. Dans cette session, les signaux tombent à t=28,0s et t=59,0s, séparés de 31,0s : le verrou n'a jamais été sollicité.

(b) Signal de vente sur une position non confirmée : la vente ne peut porter que sur des parts confirmées en portefeuille. RÈGLE : le stop ne s'applique qu'à la quantité confirmée ; s'il tombe pendant un achat en vol, il annule d'abord l'achat non confirmé puis vend le confirmé. Cas jamais rencontré ici : le stop ne s'est jamais déclenché (marge plancher +3,29$ > seuil +2$, B-F1 B4).

(c) Deux signaux simultanés sur le même actif : les conditions d'entrée 1, d'entrée 2 et de stop sont évaluées dans un ordre fixe à chaque tick. RÈGLE : priorité déterministe stop > entrée 2 > entrée 1 ; un seul ordre émis par tick. À t=59,0s, seule l'entrée 2 était éligible (entrée 1 déjà consommée) : aucun conflit réel.

(d) Fill partiel : les quotes BBO ne donnent pas la profondeur [HYPOTHÈSE n°2]. RÈGLE : tout ordre est simulé en fill intégral au prix affiché ; en cas de fill partiel réel, le reliquat serait annulé (immediate-or-cancel), jamais reporté. Les deux ordres de 2,50$ représentent 0,02% du volume de session de 14 968,71$ (B-F4 B4) : l'hypothèse de fill intégral est la moins coûteuse possible.

### JOURNAL DE TRADES

Le journal, ligne par ligne :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=28,0s | 0,75$ | 3,3333 | t=300,0s (règlement) | 1,00$ | 0,00$ | +0,8333$ | 2,50$ (après achat)
| 2 | t=59,0s | 0,75$ | 3,3333 | t=300,0s (règlement) | 1,00$ | 0,00$ | +0,8333$ | 0,00$ (après achat)
| — | — | — | 6,6667 | encaissement règlement | — | 0,00$ | — | 6,67$


### RÉSULTATS

Le récapitulatif final :

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 2 / +1,67$ (somme exacte 0,8333$ + 0,8333$) | lignes 1-2
| Trades perdants / pertes cumulées | 0 / 0,00$ | —
| Frais totaux | 0,00$ + 0,00$ = 0,00$ | lignes 1-2 [HYPOTHÈSE n°1]
| BÉNÉFICE NET | +1,67$ | lignes 1-2
| Capital final vs initial | 6,67$ vs 5,00$ (+33,33%) | ligne d'encaissement
| Pire perte unitaire | aucune (0 trade perdant) | journal complet
| Drawdown maximum (mark-to-market) | 1,27$ — valeur 3,73$ à t=186,1s, retour à 5,00$ dépassé avant t=272,7s (B-F3 B4) | états du portefeuille


## C4 — VERDICT ET PORTÉE

VERDICT : [VALIDÉE SUR SESSION] — 2 trades, 2 gagnants, bénéfice net +1,67$ sur 5,00$ engagés, aucun stop touché, drawdown MTM limité à 1,27$.

Ce qui relève de la stratégie elle-même : l'ancrage à l'oracle a capté une décote réelle — le marché cotait 0,75$ ce que l'oracle désignait déjà. Ce qui relève de mes adaptations : le fractionnement et le stop n'ont rien coûté ni rapporté sur cette session (stop jamais déclenché) ; le rendement vient intégralement de la règle d'entrée.

- Le marché sous-cote durablement un règlement que l'oracle désigne dès t=28,0s (0,75$ payé pour 1,00$ livré 272 s plus tard). [PATTERN CANDIDAT]
- La marge finale confortable de +15,50$ qui a rendu la tenue jusqu'au règlement indolore. [SESSION-SPÉCIFIQUE]
- L'absence de retour de l'oracle sous +3,29$ après l'entrée, qui a laissé le stop inutile. [SESSION-SPÉCIFIQUE]
- Le flux acheteur YES à 71,7% aligné avec l'oracle. [SESSION-SPÉCIFIQUE]
- Le stop à +2$ protège le capital sans fausse sortie, mais n'a jamais été testé en conditions réelles. [FRAGILE - 1 SOURCE]


# AUTO-CONTRÔLE FINAL

- Contrôle 1 → CONFORME : 6 valeurs en A-G1, 6 en A-G2, 5 en A-G3, 6 en A-G4, 5 en A-G5, 5 en A-G6 ; 9 métriques en B-F1, 8 en B-F2, 9 en B-F3, 8 en B-F4.
- Contrôle 2 → CONFORME : chaque règle de C0/C1/C2/C3 cite une rubrique A4/B4 ou porte un marqueur [HYPOTHÈSE].
- Contrôle 3 → CONFORME : 6,67$, +1,67$, +33,33%, 2 trades, 3 adaptations identiques en ouverture, corps et clôture.
- Contrôle 4 → CONFORME : 5,00 − 2,50 − 2,50 + 6,6667 = 6,6667$, arrondi 6,67$ — égal au capital final annoncé, au centime près.
- Contrôle 5 → 3 [HYPOTHÈSE] utilisées, dans la limite : frais (aucune colonne frais), fill intégral (aucune profondeur), règlement à 1,00$ (règle de marché hors fichiers).
- Contrôle 6 → 3 [ADAPTATION] utilisées, toutes au journal C1-ADAPT avec justification chiffrée ; aucune ne contredit le PRINCIPE DIRECTEUR (toutes lisent l'oracle seul) ; aucune décidée après C-SIM.
- Contrôle 7 → CONFORME : verdict en ouverture, ordre F1 respecté, tableaux F4 fournis, identifiants A-G1 à A-G6 et B-F1 à B-F4, clôture en 3 lignes sans chiffre nouveau.


# CLÔTURE DU SPEECH

Vous avez vu chaque chiffre et sa source : le bénéfice net est de +1,67$, le capital final de 6,67$ contre 5,00$ de départ.
«ANCRAGE À L'ORACLE», telle que je l'ai adaptée, est [VALIDÉE SUR SESSION].
La décote du marché face à l'oracle est le seul élément que je vous propose de retester ailleurs — le reste appartient à cette session.

---

## Source : 14ancrage.md

# SPEECH — TEST DE «ANCRAGE À L'ORACLE» SUR LA SESSION POLYMARKET BTC 5MIN DU 6 AOÛT (7:50PM–7:55PM ET)

## 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Vous voulez savoir ce que vaut «ANCRAGE À L'ORACLE» sur cette session ; voici le verdict avant toute chose.

- Stratégie testée : «ANCRAGE À L'ORACLE» — je m'ancre à l'écart signé entre l'oracle Chainlink (seule source de règlement) et le strike, et j'achète le côté que l'oracle désigne quand le carnet le sous-price.
- Capital final : 8,4727$ contre 5,00$ de départ, soit un rendement net de +69,45%.
- Trades : 1 trade, 1 gagnant, 0 perdant.
- Adaptations apportées à la stratégie : 4.
- VERDICT SUR LA STRATÉGIE : [VALIDÉE SUR SESSION] — étiquette de portée [SESSION-SPÉCIFIQUE]


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

## 1. CADRE D'ANALYSE CHOISI

Je retiens un cadre microstructurel événementiel : comparaison tick par tick de trois horloges — le spot Binance (5841 ticks, précision ms), l'oracle Chainlink (270 ticks, pas de 1 s) et le carnet Polymarket (279 événements BBO). Sur un horizon de 5 minutes, il n'y a ni fondamentaux ni saisonnalité exploitables : la seule information est QUI SAIT QUOI EN PREMIER, et cette granularité milliseconde est exactement ce que les fichiers fournissent. C'est le cadre adapté à ces données précises, pas un choix de convention.

---

## PARTIE A — LES 6 GRAPHES DU PDF

### A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (279 ÉVÉNEMENTS BBO)

**A1. IDENTITÉ**
Graphique 1, «Carnet YES/NO complet — 279 événements BBO (précision ms) | incohérents : 19». Axe X : temps depuis l'ouverture (mm:ss.mmm), de t=00:00.168 à t=03:21.941. Axe Y gauche : prix en $, de 0,0 à 1,0 ; axe Y droit : probabilité. Six séries (bid/ask/mid YES et NO) plus la probabilité consensus (YES, 1−NO). Vue 300 s et trois zooms 15 s.

**A2. QUESTION**
Que fait le prix du carnet quand l'information extérieure arrive ? À quelle vitesse la probabilité consensus converge-t-elle vers 0 ou 1 ?

**A3. COMMENT je l'ai exploité**
Lecture des tooltips ask YES et ask NO aux points de rupture de pente, comparaison des paliers avant/après chaque accélération, et repérage du dernier événement coté.

**A4. VALEURS EXTRAITES**

- t=00:00.168 : YES ask = 0,52$
- t=00:09.244 : YES ask = 0,65$
- t=01:00.685 : YES ask = 0,82$
- t=01:00.710 : NO bid = 0,16$
- t=01:48.927 : YES ask = 0,92$
- t=03:02.452 : YES ask = 0,99$
- t=03:21.941 : dernier événement coté, YES ask = 0,98$


**A5. OBSERVATIONS BRUTES**

- Le YES passe de 0,52$ (t=00:00.168) à 0,65$ (t=00:09.244) en 9,076 s : le carnet réagit par rafales, pas en continu.
- Entre t=01:00.366 et t=01:01.623, le YES passe de 0,71$ à 0,89$ : +0,18$ en 1,257 s.
- Après t=03:02.452 (0,99$), plus aucun retour sous 0,98$ : la convergence est terminale à 60% de la session.
- Le carnet cesse d'émettre des événements après t=03:21.941 : 98 s de silence avant la clôture.


**A6. SPÉCIFIQUE À LA SESSION**
La rupture à t=01:00.366 (+0,18$ en 1,257 s) est datée sur un mouvement spot unique (voir A-G2) : c'est un événement de cette session, pas une propriété du carnet. Le silence final de 98 s est propre à une issue résolue tôt ; sur une session serrée, le carnet coterait jusqu'au bout.

### A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT

**A1. IDENTITÉ**
Graphique 2, «Spot Binance (flux direct) vs Oracle Chainlink — strike exact (dernier tick ≤ t0)». Axe X : temps (mm:ss.mmm), 300 s ; axe Y gauche : prix BTC en $, 64 180 à 64 300 ; panneau bas : basis = spot − oracle en $, 0 à 80. Strike d'ouverture : 64 172,00$. 5841 ticks Binance, 270 ticks Chainlink, fallback ts 0/270 (0,0%).

**A2. QUESTION**
Lequel des deux flux bouge en premier, et de combien ? L'oracle menace-t-il jamais le strike ?

**A3. COMMENT je l'ai exploité**
Lecture des tooltips des deux courbes aux mêmes fenêtres temporelles (zooms 00:53–01:08 et 03:18–03:33), mesure des décalages, et lecture de la distance verticale oracle − strike.

**A4. VALEURS EXTRAITES**

- t=00:00.937 : spot = 64 228,51$
- t=00:16.000 : oracle = 64 175,32$ (premier tick, +3,32$ au-dessus du strike 64 172,00$)
- t=01:01.631 : spot = 64 271,88$
- t=01:02.000 : oracle = 64 208,65$
- t=03:25.876 : spot = 64 300,00$
- t=04:59.815 : spot = 64 304,65$


**A5. OBSERVATIONS BRUTES**

- L'oracle est au-dessus du strike dès son premier tick : +3,32$ à t=00:16.000.
- Le spot atteint 64 271,88$ à t=01:01.631 quand l'oracle n'affiche que 64 208,65$ à t=01:02.000 : le spot précède.
- Le basis spot − oracle est positif sur toute la vue (bande 53–65$ lue sur le panneau bas) : les deux flux ne se croisent jamais.
- L'oracle ne revient jamais toucher le strike sur les 283 s couvertes.


**A6. SPÉCIFIQUE À LA SESSION**
Le basis constamment positif de 53–65$ est une caractéristique de calibration des deux flux ce jour-là (indices composites vs une seule bourse) ; son niveau absolu n'est pas transposable. L'absence totale de menace sur le strike (marge minimale +3,32$) est le tirage de cette session.

### A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ YES/NO (817 TRADES)

**A1. IDENTITÉ**
Graphique 3, «Flux directionnel normalisé YES/NO (817 trades)». Axe X : temps, 300 s ; axe Y : volume en $ par fenêtre 30 s, flux haussier (BUY YES + SELL NO) en positif, baissier en négatif, échelle −1 000 à 5 000.

**A2. QUESTION**
Quand l'argent entre-t-il, et de quel côté ? Le flux suit-il ou précède-t-il le prix ?

**A3. COMMENT je l'ai exploité**
Lecture des tooltips des trades individuels aux pics de chaque rafale, et comparaison des amplitudes haussières/baissières par fenêtre.

**A4. VALEURS EXTRAITES**

- t=00:26.079 : +200,00$
- t=00:50.426 : −284,54$
- t=01:33.140 : +300,80$
- t=03:01.428 : +724,06$
- t=03:02.757 : +3 294,11$
- t=03:12.008 : −135,65$


**A5. OBSERVATIONS BRUTES**

- Le plus gros trade de la session (+3 294,11$ à t=03:02.757) arrive quand le YES cote déjà 0,99$ (A-G1, A4) : ce flux confirme, il ne découvre pas.
- Les rafales haussières de t=01:00.594 à t=01:02.783 suivent le saut spot de t=01:00.261 (A-G2) : le flux réagit au spot avec un délai inférieur à 1 s.
- Les flux baissiers ne dépassent jamais −284,54$ : aucune conviction vendeuse.


**A6. SPÉCIFIQUE À LA SESSION**
Le bloc de +3 294,11$ est un ordre isolé d'un acteur unique : non reproductible. La quasi-absence de flux après t=03:23.474 (dernier tooltip) reflète l'issue déjà résolue.

### A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ**
Graphique 4, «Imbalance directionnelle (YES/NO normalisés)», Imbalance = Haussier / (Haussier + Baissier), 817 événements, ligne d'équilibre à 0,5. Axe X : temps, 300 s ; axe Y : 0,0 à 1,0.

**A2. QUESTION**
La pression acheteuse domine-t-elle durablement, ou oscille-t-elle autour de 0,5 ?

**A3. COMMENT je l'ai exploité**
Lecture des tooltips aux extrêmes et aux croisements de la ligne 0,5 sur la vue 300 s et les trois zooms.

**A4. VALEURS EXTRAITES**

- t=00:01.097 : imbalance = 1,00
- t=00:02.500 : imbalance = 0,17
- t=01:00.394 : imbalance = 0,38
- t=01:07.042 : imbalance = 0,74
- t=03:18.147 : imbalance = 0,95
- t=04:01.427 : imbalance = 0,00


**A5. OBSERVATIONS BRUTES**

- L'imbalance passe de 0,38 (t=01:00.394) à 0,74 (t=01:07.042) : bascule haussière en 6,648 s, synchrone du saut spot de A-G2.
- À t=03:18.147 elle tient 0,95 : le flux est unanime en fin de convergence.
- La valeur 0,00 à t=04:01.427 porte sur un volume résiduel (4 trades sur 240–300 s, voir B-F4) : dénominateur quasi vide.


**A6. SPÉCIFIQUE À LA SESSION**
Les extrêmes 1,00 (t=00:01.097) et 0,00 (t=04:01.427) sont des artefacts de fenêtre 30 s presque vide — classés bruit, car calculés sur 1 à 4 trades.

### A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS

**A1. IDENTITÉ**
Graphique 5, «Spreads YES et NO absolus (événementiels)», ask − bid pour chaque carnet, 279 événements. Axe X : temps, 300 s ; axe Y : 0,00 à 0,08$.

**A2. QUESTION**
Que coûte l'exécution à chaque instant, et quand le spread s'élargit-il ?

**A3. COMMENT je l'ai exploité**
Lecture des tooltips aux pics de spread et aux retours à 0,01$, sur la vue 300 s et les zooms.

**A4. VALEURS EXTRAITES**

- t=00:09.093 : spread = 0,08$
- t=00:09.244 : spread = 0,01$
- t=00:49.647 : spread = 0,07$
- t=01:00.376 : spread = 0,06$
- t=03:18.749 : spread = 0,00$


**A5. OBSERVATIONS BRUTES**

- Le spread saute à 0,08$ pendant la rafale de t=00:09.081–00:09.171 puis revient à 0,01$ à t=00:09.244 : élargissements de moins de 200 ms.
- Hors rafales, le spread se tient à 0,01–0,02$ : le coût d'exécution de base est de 1 à 2 cents par part.
- Les élargissements coïncident avec les accélérations de prix de A-G1 (t=00:09, t=00:49, t=01:00).


**A6. SPÉCIFIQUE À LA SESSION**
Le spread 0,00$ à t=03:18.749 correspond au carnet dégénéré de fin de convergence (YES à 0,99$) : spécifique à une issue résolue tôt.

### A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ**
Graphique 6, «Écart inter-carnets YES vs NO (incohérents : 19)», écart = yes_mid − (1 − no_mid), 279 événements, référence «cohérence parfaite» à 0, échelle Y ±1,0$ avec facteur 1e−16, de t=00:00.168 à t=03:21.941.

**A2. QUESTION**
Les deux carnets racontent-ils la même probabilité ? Existe-t-il des fenêtres d'arbitrage YES/NO ?

**A3. COMMENT je l'ai exploité**
Lecture de l'échelle (le facteur 1e−16 signifie que la courbe est numériquement nulle) et comptage des marqueurs creux «suspects».

**A4. VALEURS EXTRAITES**

- t=00:00.168 à t=03:21.941 : écart = 0,00$ (à 1×10⁻¹⁶ près) sur les 279 événements
- Nombre de suspects «carnet croisé» marqués : 19
- Fallback ts (ts_src≠payload) : 0/270 (0,0%)
- Timestamps individuels des 19 marqueurs suspects : DONNÉE NON DISPONIBLE (aucun tooltip dans le PDF)
- Bornes de l'axe : −1,0$ à +1,0$


**A5. OBSERVATIONS BRUTES**

- yes_mid = 1 − no_mid sur la totalité des 279 événements : les deux carnets sont miroir exact au niveau du mid.
- Les incohérences signalées (19 selon le titre du graphe) ne se voient pas sur le mid : elles sont donc portées par les bid/ask, pas par le mid.


**A6. SPÉCIFIQUE À LA SESSION**
Le comptage «19» du titre n'est pas reconstructible depuis le CSV BBO, qui n'exhibe que 2 timestamps de carnet croisé (voir B-F3, B6) : je classe l'écart de comptage comme artefact de méthode (comptage par mise à jour intermédiaire vs par timestamp distinct), verdict bruit.

---

## PARTIE B — LES FICHIERS CSV

### B-F1 — SPOT.CSV (BINANCE BTC/USDT)

**B1. IDENTITÉ**
Fichier spot, colonnes t_ms et price, toutes deux utilisées. 5841 lignes, de t=00:00.937 à t=04:59.815, échantillonnage événementiel (rafales de ticks au même t_ms).

**B2. APPORT**
La vitesse exacte du spot, que le graphe 2 ne donne qu'en lecture visuelle : c'est ici que se mesure l'avance du spot sur l'oracle.

**B3. COMMENT**
Min/max/moyenne/écart-type sur price (5841 lignes) ; plus forte hausse sur fenêtre glissante de 1 000 ms (balayage des 5841 ticks) ; valeur au dernier tick ≤ t pour t=01:00.000 et t=01:02.000.

**B4. RÉSULTATS CHIFFRÉS**
Les huit mesures, colonne par colonne :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Premier prix | 64 228,51$ | t_ms, price | 1
| Dernier prix | 64 304,65$ | t_ms, price | 1
| Minimum | 64 228,50$ | price | 5841
| Maximum | 64 305,98$ | price | 5841
| Moyenne | 64 264,49$ | price | 5841
| Écart-type | 24,83$ | price | 5841
| Plus forte hausse en 1 s | +26,92$ (t=01:00.261 → t=01:01.243) | t_ms, price | 5841
| Prix au dernier tick ≤ t=01:00.000 | 64 243,38$ (t=00:57.947) | t_ms, price | 1


**B5. OBSERVATIONS BRUTES**

- Le spot gagne +26,92$ en 982 ms à partir de t=01:00.261 : c'est l'événement directeur de la session.
- Le spot ne repasse jamais sous son ouverture : minimum 64 228,50$ pour une ouverture à 64 228,51$.
- Amplitude totale 77,48$ (64 305,98$ − 64 228,50$) pour un écart-type de 24,83$ : une session directionnelle, pas oscillante.


**B6. SPÉCIFIQUE À LA SESSION**
Rafales de dizaines de ticks au même t_ms (exemple : t=00:00.8690 et suivants au prix 64 228,51$) : artefact d'agrégation du flux, verdict bruit. La monotonie haussière quasi parfaite est le tirage du jour, verdict signal non transposable.

### B-F2 — ORACLE.CSV (CHAINLINK BTC/USD)

**B1. IDENTITÉ**
Fichier oracle, colonnes t_ms, price, ts_src, toutes utilisées. 270 lignes, de t=00:16.000 à t=04:58.000, pas nominal de 1 s.

**B2. APPORT**
La série de règlement elle-même — le seul prix qui décide UP ou DOWN — et sa qualité d'horodatage (ts_src), invisibles dans les graphes au tick près.

**B3. COMMENT**
Min/max/moyenne sur price (270 lignes) ; marge = price − 64 172,00$ ligne par ligne ; écarts t_ms consécutifs (269 paires) ; pas de prix consécutifs (269 paires) ; comptage ts_src ≠ payload ; basis = spot(dernier tick ≤ t) − oracle sur les 270 t de l'oracle.

**B4. RÉSULTATS CHIFFRÉS**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Premier tick | 64 175,32$ à t=00:16.000 | t_ms, price | 1
| Marge minimale vs strike 64 172,00$ | +3,32$ (t=00:17.000) | price | 270
| Marge finale | +78,27$ (t=04:58.000) | price | 1
| Plus grand pas haussier | +15,85$ (t=01:02.000) | t_ms, price | 269 paires
| Plus grand pas baissier | −3,75$ (t=01:49.000) | t_ms, price | 269 paires
| Trous d'échantillonnage > 1 s | 13 (12 trous de 2 s, 1 trou de 3 s après t=03:27.000) | t_ms | 269 paires
| Fallback ts_src ≠ payload | 0/270 (0,0%) | ts_src | 270
| Basis spot − oracle : min / max / moyenne | 53,42$ / 64,84$ (t=01:01.000) / 57,76$ | price + spot.price | 270


**B5. OBSERVATIONS BRUTES**

- L'oracle n'approche jamais le strike à moins de 3,32$ sur 270 ticks : l'issue UP n'est jamais menacée à partir de t=00:16.000.
- Le saut oracle de +15,85$ arrive à t=01:02.000, soit au moins 757 ms après le début du saut spot à t=01:00.261 (B-F1, B4) : l'oracle est en retard structurel sur le spot.
- Le pire recul en un pas est −3,75$ : un seuil de sécurité au-dessus de 3,75$ absorbe tout pas isolé observé.
- Le basis reste dans la bande 53,42–64,84$ : l'oracle et le spot ne convergent jamais, ils se translatent.


**B6. SPÉCIFIQUE À LA SESSION**
Aucune donnée oracle entre t=00:00.000 et t=00:16.000 : trou de démarrage, verdict signal (il interdit tout ancrage sur les 16 premières secondes). Les 13 trous d'1 s supplémentaire sont réguliers et sans impact directionnel, verdict bruit.

### B-F3 — BBO.CSV (CARNET POLYMARKET)

**B1. IDENTITÉ**
Fichier bbo, colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask, toutes utilisées. 279 lignes, de t=00:00.168 à t=03:23.301, échantillonnage événementiel.

**B2. APPORT**
Les prix exécutables exacts — c'est dans ce fichier que ma simulation prend ses fills, ce que les graphes ne permettent pas au cent près.

**B3. COMMENT**
Spread YES = yes_ask − yes_bid sur les 272 lignes complètes côté YES ; test de cohérence (yes_bid+yes_ask)/2 − (1 − (no_bid+no_ask)/2) sur les 271 lignes complètes des deux côtés ; comptage yes_bid + no_bid > 1,00$ ; comptage des lignes à côté manquant ; premier/dernier mid.

**B4. RÉSULTATS CHIFFRÉS**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Mid YES premier | 0,515$ (t=00:00.168) | yes_bid, yes_ask | 1
| Mid YES dernier | 0,985$ (t=03:21.941) | yes_bid, yes_ask | 1
| Spread YES moyen | 0,0205$ | yes_bid, yes_ask | 272
| Spread YES maximum | 0,08$ (t=00:09.093) | yes_bid, yes_ask | 272
| Événements à spread ≥ 0,05$ | 20 | yes_bid, yes_ask | 272
| Carnets croisés (yes_bid + no_bid = 1,01$) | 2 (t=00:48.737 et t=01:48.821) | yes_bid, no_bid | 271
| Écart de cohérence mid YES vs 1 − mid NO | 0,0000$ sur les 271 lignes complètes | 4 colonnes | 271
| Lignes à côté manquant | 8 | 4 colonnes | 279


**B5. OBSERVATIONS BRUTES**

- À t=00:14.946, le YES s'achète à 0,59$ alors que l'oracle affiche +3,32$ de marge dès t=00:16.000 (B-F2, B4) : le carnet price 59% un événement que la série de règlement montre acquis à 3,32$ près.
- Le spread de base est de 0,01$ : entrer au ask coûte 1 cent par part hors rafale.
- Les 2 carnets croisés offrent chacun 0,01$ d'arbitrage théorique — trop bref (une seule mise à jour) et trop petit pour être exploité avec 5,00$.


**B6. SPÉCIFIQUE À LA SESSION**
Le PDF annonce 19 incohérents, le CSV n'exhibe que 2 timestamps croisés distincts : écart de méthode de comptage, verdict bruit (cf. A-G6, A6). Les 8 lignes à côté manquant sont toutes en fin de session (le NO cesse d'être coté après t=03:03), verdict bruit lié à la convergence.

### B-F4 — TRADES.CSV (FLUX D'ORDRES POLYMARKET)

**B1. IDENTITÉ**
Fichier trades, colonnes t_ms, usd, direction, toutes utilisées. 817 lignes, de t=00:01.097 à t=04:59.899.

**B2. APPORT**
Le volume signé réel — les graphes 3 et 4 en montrent l'agrégat 30 s, le CSV donne chaque ordre.

**B3. COMMENT**
Somme et comptage de usd par direction (817 lignes) ; imbalance = Σ(usd | direction=1) / Σ(usd total) ; découpage par tranches de 60 s sur t_ms ; médiane et maximum de usd.

**B4. RÉSULTATS CHIFFRÉS**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Trades haussiers (direction=1) | 560, pour 11 446,90$ | usd, direction | 817
| Trades baissiers (direction=−1) | 257, pour 2 488,71$ | usd, direction | 817
| Volume total | 13 935,62$ | usd | 817
| Imbalance session | 0,8214 | usd, direction | 817
| Plus gros trade | 3 294,11$ haussier à t=03:02.757 | t_ms, usd, direction | 1
| Médiane des tailles | 4,67$ | usd | 817
| Imbalance par minute (0-60/60-120/120-180/180-240/240-300 s) | 0,545 / 0,851 / 0,875 / 0,943 / 0,000 | t_ms, usd, direction | 817
| Volume 240–300 s | 9,14$ (4 trades) | t_ms, usd | 4


**B5. OBSERVATIONS BRUTES**

- L'imbalance passe de 0,545 (minute 1) à 0,851 (minute 2) : la conviction acheteuse s'installe après le saut spot de t=01:00.261 (B-F1, B4), pas avant.
- La médiane à 4,67$ montre un marché de petits ordres : mes 5,00$ sont une taille de marché normale, pas une anomalie.
- Le marché meurt après 240 s : 9,14$ de volume sur la dernière minute.


**B6. SPÉCIFIQUE À LA SESSION**
Le bloc de 3 294,11$ à t=03:02.757 représente 23,6% du volume total en un ordre : valeur extrême d'un acteur unique, verdict bruit. Les 4 trades résiduels après t=04:00 sont du nettoyage de position, verdict bruit.

---

## PARTIE C — LA STRATÉGIE «ANCRAGE À L'ORACLE»

### C0. INTERPRÉTATION OPÉRATIONNELLE

Voici les cinq observations les plus fortes des Parties A et B, celles sur lesquelles tout le reste s'appuie :

- OBS-1 : l'oracle reste au-dessus du strike sur toute sa période, marge minimale +3,32$ à t=00:17.000 (B-F2, B5).
- OBS-2 : le spot précède l'oracle — +26,92$ en 982 ms dès t=01:00.261 côté spot (B-F1, B5), contre un pas de +15,85$ seulement à t=01:02.000 côté oracle (B-F2, B5).
- OBS-3 : le carnet est en retard sur l'oracle — YES à 0,59$ à t=00:14.946 quand la marge oracle est de +3,32$ (B-F3, B5).
- OBS-4 : le basis spot − oracle reste dans la bande 53,42–64,84$ : seul l'oracle fait foi pour le règlement, le spot n'est qu'un indicateur avancé (B-F2, B5).
- OBS-5 : le flux converge vers l'issue oracle — imbalance de 0,545 à 0,943 entre la minute 1 et la minute 4 (B-F4, B5).


PRINCIPE DIRECTEUR : L'oracle Chainlink est l'unique source de règlement ; je m'ancre à l'écart signé oracle − strike et j'achète le côté que cet écart désigne tant que le carnet le price en dessous de sa valeur d'ancrage. Je ne sors que si l'oracle lui-même revient menacer le strike — jamais sur le bruit du carnet.

Le tableau de traduction, règle par règle :

| Règle | Formulation chiffrée | Observations sources (A5/B5)
|-----|-----|-----|-----
| Entrée | Acheter YES au yes_ask dès que oracle − strike > 0,00$ | OBS-1 (B-F2, B5) ; OBS-3 (B-F3, B5)
| Prix d'exécution | Fill au yes_ask du dernier BBO ≤ t_signal | B-F3, B5 (spread de base 0,01$)
| Sortie de protection | Vendre au yes_bid si oracle − strike < 0,00$ | OBS-1 (B-F2, B5)
| Sortie nominale | Tenir jusqu'à résolution, payout 1,00$ par part | A-G1, A5 (convergence terminale à 0,99$) ; OBS-5
| Dimensionnement | 100% du cash disponible | B-F4, B5 (médiane du marché 4,67$ : 5,00$ est une taille standard)


Les hypothèses que les données ne fournissent pas, je les marque :

- Frais de transaction = 0,00$ : aucun fichier ne contient de barème de frais. [HYPOTHÈSE n°1]
- Fill intégral au BBO affiché pour un notional de 5,00$ : les tailles aux meilleures limites ne sont pas dans le fichier bbo. [HYPOTHÈSE n°2]


### C1-ADAPT. JOURNAL DES ADAPTATIONS

J'ai calibré quatre points avant toute simulation — chacun tracé, chiffré, et aucun ne touche au principe directeur :

| # | Paramètre/Règle d'origine | Adaptation | Justification chiffrée (source)
|-----|-----|-----|-----
| 1 | Entrée si marge > 0,00$ | Entrée si marge ≥ +3,00$ | Le pire pas oracle isolé est −3,75$ (B-F2, B4) : sous 3,00$ de marge, un seul pas peut invalider le signal ; la marge réelle à l'entrée est de +3,32$ (B-F2, B4) [ADAPTATION n°1]
| 2 | Entrée à tout prix yes_ask | Filtre : yes_ask ≤ 0,70$ | Au-dessus de 0,70$, le gain résiduel ≤ 0,30$/part ne couvre plus 3 fois le pire pas oracle ramené en probabilité ; le ask franchit 0,71$ dès t=00:59.992 (B-F3, B4) [ADAPTATION n°2]
| 3 | Sortie si marge < 0,00$ | Sortie si marge < +1,00$ | Avec un pas oracle possible de −3,75$ (B-F2, B4), attendre le franchissement du strike sort trop tard ; la zone morte à 1,00$ déclenche avant [ADAPTATION n°3]
| 4 | Stratégie active dès t=00:00.000 | Aucun trade avant t=00:16.000 | L'oracle n'existe pas avant t=00:16.000 (B-F2, B1) : sans oracle, pas d'ancrage possible [ADAPTATION n°4]


### C2. LA STRATÉGIE ADAPTÉE EN PSEUDO-CODE

```plaintext
CONSTANTES:
  STRIKE        = 64172.00        # graphe 2, A1
  SEUIL_ENTREE  = +3.00           # [ADAPTATION n°1] — pire pas oracle -3.75$ (B-F2, B4)
  CAP_PRIX      = 0.70            # [ADAPTATION n°2] — ask > 0.70$ dès t=00:59.992 (B-F3, B4)
  SEUIL_SORTIE  = +1.00           # [ADAPTATION n°3] — zone morte vs pas -3.75$ (B-F2, B4)
  T_MIN         = 16000 ms        # [ADAPTATION n°4] — premier tick oracle (B-F2, B1)
  FRAIS         = 0.00            # [HYPOTHÈSE n°1]

ÉTAT: cash = 5.00 ; position = 0 ; verrou_ordre = LIBRE

POUR CHAQUE tick oracle (t, prix) AVEC t >= T_MIN:      # ancrage: horloge = oracle
  marge = prix - STRIKE
  SI position == 0 ET verrou_ordre == LIBRE:
    bbo = dernier BBO avec t_bbo <= t                   # fills réels (B-F3)
    SI marge >= SEUIL_ENTREE ET bbo.yes_ask <= CAP_PRIX:
      verrou_ordre = OCCUPÉ
      qty  = arrondi(cash / bbo.yes_ask, 2)             # fill intégral [HYPOTHÈSE n°2]
      cash = cash - qty * bbo.yes_ask
      position = qty ; verrou_ordre = LIBRE
  SINON SI position > 0:
    SI marge < SEUIL_SORTIE:
      vendre position au yes_bid du dernier BBO <= t    # sortie de protection
FIN
SI position > 0 À t = 300000 ms: payout = position * 1.00   # A-G1, A5
```

### C3. TABLE DE CONVERGENCE

Chaque règle, avec ce qui la soutient et ce qui la contredit :

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| Entrée à marge ≥ +3,00$ | A-G2 (A4) ; B-F1 (B4) ; B-F2 (B4) | aucune | Soutenue — 3 sources
| Filtre yes_ask ≤ 0,70$ | A-G1 (A4) ; B-F3 (B4) | aucune | Soutenue — 2 sources
| Sortie à marge < +1,00$ | B-F2 (B4) | aucune (jamais déclenchée sur la session) | [FRAGILE - 1 SOURCE]
| Tenir jusqu'à résolution | A-G1 (A5) ; A-G4 (A4) ; B-F4 (B4) | aucune | Soutenue — 3 sources
| Dimensionnement 100% du cash | B-F4 (B4, médiane 4,67$) | aucune | [FRAGILE - 1 SOURCE]


Je le dis clairement : la règle de sortie et le dimensionnement ne tiennent chacun qu'à une seule pièce — je les marque FRAGILES, et la sortie n'a même jamais été testée par les données puisque la marge n'est jamais repassée sous 3,32$.

### C-SIM. SIMULATION ALGORITHMIQUE DÉTAILLÉE

**ÉTAT DU PORTEFEUILLE**

- t=00:00.000 : cash 5,0000$, positions : aucune, valeur totale 5,0000$.
- t=00:16.000 : signal d'entrée — oracle 64 175,32$, marge +3,32$ ≥ 3,00$ (B-F2, B4) ; BBO de référence : t=00:14.946, yes_ask 0,59$ ≤ 0,70$ (B-F3, B4). Achat de 8,47 parts YES à 0,59$ = 4,9973$ ≤ cash disponible. Cash restant : 0,0027$. Valeur mark-to-market au yes_bid 0,58$ : 4,9153$.
- t=00:16.000 → t=04:58.000 : marge oracle minimale +3,32$ (B-F2, B4), jamais < +1,00$ : aucune sortie de protection déclenchée. Position tenue.
- t=05:00.000 : résolution ▲ UP (titre du PDF). Payout 8,47 × 1,00$ = 8,4700$. Cash final : 8,4700$ + 0,0027$ = 8,4727$.


**RACE CONDITIONS**
(a) Signal d'achat pendant un ordre en cours : le verrou par actif passe à OCCUPÉ à l'émission de l'ordre ; tout signal reçu pendant ce temps est ignoré, pas mis en file. RÈGLE : un verrou par actif rejette tout signal d'achat tant qu'un ordre est en vol.
(b) Signal de vente sur une position non confirmée : la vente ne peut porter que sur `position`, qui n'est incrémentée qu'au fill confirmé ; un signal de vente antérieur au fill trouve position = 0 et est rejeté. RÈGLE : aucune vente n'est émise tant que le fill d'achat n'a pas incrémenté l'état position.
(c) Deux signaux simultanés sur le même actif : les ticks oracle sont traités en file FIFO stricte par t_ms croissant ; à t_ms égal, la première ligne du fichier prime et la seconde est rejetée par le verrou. RÈGLE : file FIFO sur t_ms, le second signal simultané est rejeté par le verrou de l'actif.
(d) Fill partiel : la quantité confirmée devient la position, le cash non consommé est recrédité immédiatement, et aucun re-ordre n'est émis avant le tick oracle suivant. RÈGLE : un fill partiel fige la position à la quantité exécutée et recrédite le solde sans re-soumission automatique. Sur cette session, le cas ne s'est pas présenté sous [HYPOTHÈSE n°2].

**JOURNAL DE TRADES**
Le journal, ligne par ligne — il n'y en a qu'une :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=00:16.000 | 0,59$ | 8,47 parts YES | t=05:00.000 (résolution) | 1,00$ | 0,00$ | +3,4727$ | 8,4727$


**RÉSULTATS**

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 1 / +3,4727$ | ligne 1
| Trades perdants / pertes cumulées | 0 / 0,00$ | —
| Frais totaux | 0,00$ [HYPOTHÈSE n°1] | ligne 1
| BÉNÉFICE NET | +3,4727$ | ligne 1
| Capital final vs initial | 8,4727$ vs 5,0000$ (+69,45%) | ligne 1
| Pire perte unitaire | 0,00$ | —
| Drawdown maximum (mark-to-market) | −0,0847$ (5,0000$ → 4,9153$, entre t=00:16.000 et t=00:17.489) | ligne 1, valorisée au yes_bid 0,58$ (B-F3)


### C4. VERDICT ET PORTÉE

VERDICT : [VALIDÉE SUR SESSION] — bénéfice net +3,4727$, 1 trade sur 1 gagnant, drawdown maximal −0,0847$, uniquement sur les chiffres du tableau C-SIM.

Chaque conclusion, avec son étiquette :

- L'oracle en retard sur le spot d'au moins 757 ms (B-F1/B-F2, B4) crée une fenêtre où le carnet sous-price l'issue : [PATTERN CANDIDAT]
- Un carnet à 0,59$ face à une marge oracle de +3,32$ dès t=00:16.000 : [PATTERN CANDIDAT]
- La marge oracle jamais inférieure à +3,32$ — donc une sortie de protection jamais testée et un trade sans adversité : [SESSION-SPÉCIFIQUE]
- Le rendement de +69,45% en un trade : [SESSION-SPÉCIFIQUE]
- Ce qui relève de la stratégie elle-même : l'entrée ancrée à la marge oracle et la tenue jusqu'à résolution ; ce qui relève de mes adaptations : le seuil +3,00$, le cap 0,70$, la zone morte +1,00$ et l'exclusion des 16 premières secondes — sans elles, la même stratégie serait entrée plus tôt, plus cher, ou sans données.


## AUTO-CONTRÔLE FINAL

- Contrôle 1 → conforme : 5 valeurs minimum par graphe (7, 6, 6, 6, 5, 5) ; 8 métriques en tableau B4 pour chacun des 4 CSV ; une DONNÉE NON DISPONIBLE déclarée (timestamps des 19 suspects, A-G6).
- Contrôle 2 → conforme : chaque règle de C0/C1/C2 pointe vers une rubrique A4, A5, B4 ou B5 nommée ; aucune affirmation orpheline conservée.
- Contrôle 3 → conforme : 8,4727$, +69,45%, +3,4727$, 0,59$, 8,47 parts, +3,32$, −0,0847$ identiques dans l'ouverture, le corps et la clôture.
- Contrôle 4 → conforme : 5,0000$ − 4,9973$ = 0,0027$ ; 0,0027$ + 8,4700$ = 8,4727$ = capital final annoncé, au centime près.
- Contrôle 5 → 2 [HYPOTHÈSE] utilisées (frais nuls ; fill intégral) : sous le plafond de 3, pas de justification supplémentaire requise.
- Contrôle 6 → 4 [ADAPTATION] utilisées, toutes au journal C1-ADAPT avec justification chiffrée ; aucune ne contredit le principe directeur (toutes resserrent l'ancrage à l'oracle, aucune n'y substitue un autre signal) ; aucune décidée après C-SIM.
- Contrôle 7 → conforme : verdict dès l'ouverture, ordre F1 respecté, tableaux au format F4, identifiants A-G1…B-F4, adresse directe à l'auditeur, clôture en 3 lignes sans chiffre nouveau.


## CLÔTURE

Vous avez vu chaque chiffre et sa source : «ANCRAGE À L'ORACLE», adaptée en quatre points, transforme 5,00$ en 8,4727$, soit +3,4727$ nets (+69,45%), en un seul trade gagnant. Le verdict est [VALIDÉE SUR SESSION] — mais sur une session sans adversité, où la sortie de protection n'a jamais été mise à l'épreuve. Je la reteste sur des sessions où l'oracle repasse le strike avant de la qualifier de pattern.


