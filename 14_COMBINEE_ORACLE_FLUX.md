# 14 — COMBINÉE ORACLE + FLUX

**Fichiers sources regroupés :** 300Combinée_oracle+flux.md

---

## Source : 300Combinée_oracle+flux.md

ici jai tester plusieurs strategie :


## 1. Stratégie oracle pure « aveugle » (achat NO et hold) — meilleure sur le papier, mais c'est de la chance

| Stratégie | Entrée | Prix NO | P&L | Pire latent
|-----|-----|-----|-----|-----
| O1 : 1er tick oracle < strike | 00:00 | 0,53$ | **+4,43$** | 3,58$ (−28%)
| O4 : 15 ticks consécutifs < strike | 00:15 | 0,51$ | **+4,80$** | 3,73$ (−25%)
| S2 (baseline suiveur de flux) | 02:42.961 | 0,64$ | +2,81$ | 3,12$


Ces variantes battent S2 **uniquement par rétrospective** : elles entrent à un prix quasi fair (0,51–0,53$ quand le marché cotait 50/50) sans aucun avantage informationnel — l'oracle n'était que 2–15$ sous le strike et il l'a **recroisé 10 fois** dans la session (dernier passage au-dessus : 03:21). Sur une session UP, elles perdent tout. Le drawdown latent (−25 à −28% pendant le pic YES à 0,61$) le confirme. **[SESSION-SPÉCIFIQUE]**

## 2. Stratégie oracle « honnête » (avec règle de sortie) — nettement pire

Dès qu'on ajoute une sortie réaliste, les 10 croisements oracle/strike + le spread de 0,02$ par rotation détruisent le résultat :

- O2 flip YES/NO sur signe de l'oracle : 11 rotations, **−3,53$**
- O3 flip avec buffer ±10$ : 5 rotations, **−2,63$**
- O5 NO-only (achat sous strike, vente au-dessus) : **−1,17$**
- O6 idem avec confirmation 5 ticks : **−2,17$**


L'oracle collé au strike est un signal de whipsaw ; son seul vrai avantage est le **retard du marché après 03:22** (OBS-4), déjà capturé par S4 (+1,02$, entrée tardive à 0,83$).

## 3. Combinaison oracle + flux — oui, légèrement meilleure que S2

La meilleure combinaison testée : **entrer NO quand l'oracle est confirmé sous le strike (5 ticks) ET que le flux 30 s est ≤ 0 ; stop si le flux repasse > +800$** :

- Entrée : 01:14 à 0,63$ (le filtre flux bloque l'entrée pendant la minute 0 haussière — le stop n'est jamais déclenché ensuite)
- **P&L : +2,94$ contre +2,81$ pour S2** (+4,7% de mieux), pire latent 3,17$ vs 3,12$ (quasi identique)


## Verdict

- **Oracle seul avec sortie réaliste : non**, c'est la pire famille (−1,17$ à −3,53$).
- **Oracle seul en hold aveugle : +4,43$ mais c'est un pari, pas une stratégie** — aucun edge à l'entrée, ruine totale en cas d'UP.
- **Combinaison oracle + flux : oui, marginalement mieux** (+2,94$ vs +2,81$) — l'oracle donne la direction plus tôt (01:14 vs 02:42, prix 0,63$ vs 0,64$) et le flux sert de filtre anti-faux-départ et de stop. L'écart de +0,13$ est toutefois inférieur au critère de départage de 0,10$ + variabilité d'exécution : sur un seul échantillon de 5 minutes, S2 et la combinaison sont statistiquement indiscernables. **[SESSION-SPÉCIFIQUE]**








j'ai testé le **mécanisme** générique (« acheter le côté désigné par l'oracle quand le marché le vend avec une décote ») sur votre session, avec plusieurs calibrations. Verdict rapide :

**Décote fixe (le mécanisme naïf) : catastrophique.** Quel que soit le seuil, il entre à 00:42 sur le **YES** à 0,61$ (l'oracle était alors +25$ au-dessus du strike) → **−5,00$**. Le piège du mécanisme : en début de fenêtre, l'oracle désigne un côté avec conviction... qui peut se retourner. Ici il s'est retourné.

**Décote dégressive dans le temps (exiger beaucoup de décote tôt, peu tard) : bon.** La marge oracle de +25$ à 00:42 n'offre pas assez de décote pour justifier l'entrée précoce, donc il attend et entre **NO à 01:12 à 0,62$ → +3,06$**, mieux que S2 (+2,81$) et que la combinée oracle+flux (+2,94$).

| Mécanisme | P&L
|-----|-----
| Ancrage décote fixe (toutes variantes) | **−5,00$**
| Ancrage dégressif dans le temps | **+3,06$**
| Combinée oracle+flux | +2,94$
| S2 suiveur de flux | +2,81$


Donc : oui, le mécanisme d'ancrage oracle peut battre le suiveur — **mais uniquement si l'exigence de décote diminue avec le temps restant**. Sans cette pondération temporelle, il fait exactement l'erreur fatale de cette session (all-in YES à 00:42). Et l'écart de +0,25$ sur un seul échantillon reste dans le bruit. **[SESSION-SPÉCIFIQUE]**







# SPEECH — POLYMARKET BTC 5MIN « BITCOIN UP OR DOWN — AUGUST 7, 10:05–10:10AM ET »

## 0. ACCROCHE ET SYNTHÈSE EXÉCUTIVE

Le verdict d'abord, la démonstration ensuite.

- Stratégie retenue : **« Suiveur de flux 30 s »** — achat de NO quand le flux net agressif sur 30 s passe sous −800,00$.
- Capital final : **7,81$** contre 5,00$ de départ, soit un rendement net de **+56,25%** (bénéfice net +2,81$).
- Nombre de trades : **1**, dont **1 gagnant, 0 perdant**.
- Critère de sélection : **bénéfice net en $ sur la session, départage par pire perte latente** — annoncé avant toute comparaison.
- Étiquette globale : **[SESSION-SPÉCIFIQUE]** — un seul échantillon de 5 minutes ne valide aucun pattern.


Je vais vous montrer d'où vient chacun de ces chiffres, pièce par pièce.

## 1. CADRE D'ANALYSE CHOISI

J'analyse cette session par **microstructure de marché** : flux d'ordres agressifs, carnets BBO milliseconde, et écart entre le sous-jacent rapide (Binance, 25 750 ticks) et le sous-jacent de règlement lent (Chainlink, 284 ticks). Ce cadre est le seul adapté à un horizon de 300 s où toute l'information exploitable est dans la vitesse relative des quatre flux fournis. Aucune donnée fondamentale n'existe à cette échelle.

---

## PARTIE A — LES 6 GRAPHES DU PDF

### A-G1 — GRAPHE 1 : CARNET YES/NO COMPLET (1503 ÉVÉNEMENTS BBO)

**A1. IDENTITÉ.** Bid/ask/mid des jetons YES et NO, prix 0,00$–1,00$ en ordonnée, temps mm:ss.mmm de t=00:00.000 à t=05:00.000, plus 3 zooms 15 s ([00:43.000→00:58.000], [03:49.000→04:04.000], [03:13.000→03:28.000]). 1503 événements, 112 marqués incohérents.

**A2. QUESTION.** Que fait la probabilité de marché au fil de la fenêtre, et à quelle vitesse absorbe-t-elle l'information ?

**A3. COMMENT.** Lecture des étiquettes d'extrema et des séquences milliseconde des zooms ; je suis le YES mid et le NO mid en parallèle.

**A4. VALEURS EXTRAITES.**

- t=00:00.237 : YES = 0,47$ ; NO = 0,53$
- t=00:36.207 : YES = 0,61$ (maximum de session) ; NO = 0,39$
- t=03:21.451 : NO = 0,61$ (zoom 3, montée continue)
- t=03:56.548 : NO = 0,82$ (zoom 2)
- t=04:02.845 : YES = 0,13$ ; NO = 0,87$
- t=04:37.067 : YES = 0,01$ ; NO = 0,99$ (convergence terminale)


**A5. OBSERVATIONS BRUTES.**

- Le marché ouvre indécis à 0,47$/0,53$ (t=00:00.237) et clôt convergé à 0,01$/0,99$ (t=04:37.067).
- La conviction DOWN se construit par paliers : NO = 0,61$ à t=03:21.451, 0,82$ à t=03:56.548, 0,99$ à t=04:37.067.
- Les zooms montrent des glissements de 0,01$ par événement, sans saut supérieur à 0,08$.


**A6. SPÉCIFIQUE À LA SESSION.** Le pic YES à 0,61$ (t=00:36.207) suivi d'un effondrement complet est le scénario particulier de cette fenêtre ; rien n'indique que le faux départ haussier se répète. [SESSION-SPÉCIFIQUE]

### A-G2 — GRAPHE 2 : SPOT BINANCE VS ORACLE CHAINLINK, STRIKE EXACT

**A1. IDENTITÉ.** Prix BTC ($) : Binance BTC/USDT (25 750 ticks), Chainlink BTC/USD (284 ticks), strike d'ouverture 64 814,00$, panneau inférieur basis = spot − oracle (échelle 0$–60$). Vue 300 s + 3 zooms 15 s.

**A2. QUESTION.** Lequel des deux flux bouge en premier, et de combien le sous-jacent s'écarte-t-il du strike ?

**A3. COMMENT.** Comparaison des timestamps des extrema des deux courbes et lecture des croisements du strike.

**A4. VALEURS EXTRAITES.**

- t=00:37.266 : spot Binance = 64 892,00$ (maximum de session)
- t=01:05.550 : spot Binance = 64 808,00$ (minimum local)
- t=01:10.752 : spot Binance = 64 814,00$ (retour exact au strike)
- t=00:00.000 : oracle = 64 811,45$ ; t=04:58.000 : oracle = 64 776,27$
- t=03:27.000 : oracle = 64 780,13$ (chute de 41$ en 14 s depuis t=03:13.000 : 64 812,80$)
- Panneau basis : plage 0$–60$ affichée, courbe stable en zone 40$–60$


**A5. OBSERVATIONS BRUTES.**

- Le spot Binance porte un basis positif permanent sur l'oracle (courbe basis en zone 40$–60$ toute la session).
- L'oracle finit à 64 776,27$ (t=04:58.000), 37,73$ sous le strike de 64 814,00$ : le DOWN est acquis côté sous-jacent bien avant la clôture.
- Le spot imprime ses extrema avant l'oracle : max spot t=00:37.266 contre max oracle t=00:39.000 (B-F4).


**A6. SPÉCIFIQUE À LA SESSION.** L'amplitude totale spot de 84,00$ (64 892,00$ − 64 808,00$) est propre à la volatilité de cette fenêtre. [SESSION-SPÉCIFIQUE]

### A-G3 — GRAPHE 3 : FLUX DIRECTIONNEL NORMALISÉ YES/NO (3097 TRADES)

**A1. IDENTITÉ.** Volume agressif 30 s en $ : haussier (BUY YES + SELL NO) positif, baissier (SELL YES + BUY NO) négatif. Échelle −6 000$ à +4 000$, vue 300 s + 3 zooms.

**A2. QUESTION.** Où le gros argent pousse-t-il, et à quels instants ?

**A3. COMMENT.** Lecture des étiquettes d'extrema et des rafales dans les zooms.

**A4. VALEURS EXTRAITES.**

- t=00:07.135 : +1 362,31$ (plus gros ordre haussier de la session)
- t=00:43.229 : +1 205,56$ ; t=00:43.540 : +820,72$
- t=04:01.505 : −440,00$ ; t=04:01.544 : −430,00$ ; t=04:01.773 : −440,00$ (rafale vendeuse de 1 310,00$ en 268 ms)
- t=04:07.151 : −599,64$
- t=03:56.685 : −98,40$ ; t=03:56.791 : −94,30$


**A5. OBSERVATIONS BRUTES.**

- Les gros ordres haussiers sont concentrés avant t=00:44.000 ; les gros ordres baissiers après t=03:49.000.
- La rafale t=04:01.505→04:01.773 (3 ordres, 1 310,00$ vendeurs) précède la convergence NO 0,87$ (A-G1, t=04:02.845).


**A6. SPÉCIFIQUE À LA SESSION.** Les deux blocs haussiers isolés de t=00:43.229 et t=00:43.540 (2 026,28$ cumulés) n'ont pas empêché la baisse : un flux isolé n'est pas un signal ici. [SESSION-SPÉCIFIQUE]

### A-G4 — GRAPHE 4 : IMBALANCE DIRECTIONNELLE

**A1. IDENTITÉ.** Imbalance = Haussier / (Haussier + Baissier), échelle 0–1, ligne d'équilibre 0,5, 3097 événements, vue 300 s + 3 zooms.

**A2. QUESTION.** De quel côté penche le rapport de forces acheteur/vendeur, en proportion et non en montant ?

**A3. COMMENT.** Lecture des étiquettes et de la position de la courbe par rapport à 0,5 dans chaque zoom.

**A4. VALEURS EXTRAITES.**

- t=00:01.084 : 1,00 (initialisation, 1 seul trade)
- t=00:01.242 : 0,09
- t=00:57.241 : 0,75
- t=03:49.013 : 0,37
- t=04:03.498 : 0,28
- t=04:57.506 : 0,30


**A5. OBSERVATIONS BRUTES.**

- L'imbalance est au-dessus de 0,5 dans le zoom minute 1 (0,75 à t=00:57.241) et durablement sous 0,4 dans les zooms des minutes 3–4 (0,37 ; 0,28).
- La dérive de l'imbalance précède la dérive du prix NO (A-G1) dans les deux zooms tardifs.


**A6. SPÉCIFIQUE À LA SESSION.** Les valeurs 1,00 et 0,09 des 250 premières millisecondes sont des artefacts de fenêtre quasi vide (1–5 trades). [SESSION-SPÉCIFIQUE]

### A-G5 — GRAPHE 5 : SPREADS YES ET NO ABSOLUS

**A1. IDENTITÉ.** Spread ask − bid des deux jetons, échelle −0,025$ à 0,150$, 1503 événements, vue 300 s + 3 zooms.

**A2. QUESTION.** Combien coûte l'entrée-sortie, et le carnet se disloque-t-il par moments ?

**A3. COMMENT.** Lecture des extrema étiquetés et du régime courant dans les zooms.

**A4. VALEURS EXTRAITES.**

- t=00:00.237 : 0,01$
- t=03:07.256 : 0,15$ (maximum de session)
- t=03:07.309 : −0,03$ (spread négatif = carnet croisé)
- t=03:20.665 : 0,08$ ; t=03:25.351 : −0,03$
- t=04:37.067 : 0,00$
- Régime courant dans les 3 zooms : 0,01$–0,02$


**A5. OBSERVATIONS BRUTES.**

- Le spread de régime est de 0,01$–0,02$ : le coût de friction d'un aller-retour est de l'ordre de 2% du prix.
- Les dislocations (0,15$ puis −0,03$ en 53 ms à t=03:07.256→03:07.309) sont brèves et se referment seules.


**A6. SPÉCIFIQUE À LA SESSION.** Le pic 0,15$ est un événement unique en 1503 ; sa datation exacte relève de cette session. [SESSION-SPÉCIFIQUE]

### A-G6 — GRAPHE 6 : ÉCART INTER-CARNETS YES VS NO

**A1. IDENTITÉ.** Écart = yes_mid − (1 − no_mid), échelle −0,010$ à +0,004$, ligne de cohérence parfaite à 0, 112 suspects marqués, vue 300 s + 3 zooms.

**A2. QUESTION.** Les deux carnets racontent-ils la même probabilité, et l'écart est-il monnayable ?

**A3. COMMENT.** Le PDF n'étiquette aucune valeur ponctuelle sur ce graphe ; je lis l'échelle et la position de la courbe, et je chiffre l'écart par le CSV bbo (B-F1, B4).

**A4. VALEURS EXTRAITES.**

- Borne basse de l'échelle : −0,010$ ; borne haute : +0,004$
- Valeur ponctuelle maximale lisible sur le graphe : DONNÉE NON DISPONIBLE (aucune étiquette)
- Timestamp du pire écart lisible sur le graphe : DONNÉE NON DISPONIBLE
- Nombre de suspects affiché : 112
- Écart moyen recalculé sur bbo.csv : 0,00002$ ; écart absolu maximal : 0,01$ (B-F1, B4)


**A5. OBSERVATIONS BRUTES.**

- La courbe reste collée à 0 sur toute l'échelle visible : les deux carnets sont cohérents à 0,01$ près au pire (B-F1, B4).


**A6. SPÉCIFIQUE À LA SESSION.** Ce graphe n'apporte aucun signal exploitable : l'écart est trop petit pour couvrir le spread de 0,01$–0,02$ (A-G5, A4). Je le garde comme contrôle de qualité des données uniquement.

---

## PARTIE B — LES FICHIERS CSV

### B-F1 — bbo.csv (CARNET BBO)

**B1. IDENTITÉ.** Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 1503 lignes de données (1501 complètes, 2 avec côté YES vide) ; période t=00:00.237 → t=04:37.067 ; échantillonnage événementiel milliseconde. Toutes les colonnes sont utilisées.

**B2. APPORT.** Les prix exécutables exacts (ask à l'achat, bid à la vente) que les graphes ne donnent qu'en étiquettes éparses. C'est le fichier d'exécution de toute simulation.

**B3. COMMENT.** Mids = (bid+ask)/2 ; spreads = ask−bid ; détection carnet croisé par yes_ask+no_ask < 1,00$ et par spread négatif ; comptages sur les 1501 lignes complètes.

**B4. RÉSULTATS CHIFFRÉS.** Le tableau, colonne par colonne :

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| YES mid initial / final | 0,475$ / 0,010$ | yes_bid, yes_ask | 1501
| YES mid maximum | 0,615$ à t=00:36.207 | yes_bid, yes_ask | 1501
| Spread YES moyen / médian | 0,019$ / 0,020$ | yes_bid, yes_ask | 1501
| Spread NO moyen / médian | 0,019$ / 0,020$ | no_bid, no_ask | 1501
| Événements yes_ask+no_ask < 1,00$ | 9 (somme min 0,97$ à t=03:07.309) | yes_ask, no_ask | 1501
| Événements à spread négatif | 9 | 4 colonnes de prix | 1501
| Écart inter-carnets moyen / max | 0,00002$ / 0,01$ | 4 colonnes de prix | 1501
| Dernier yes_mid ≥ 0,50$ | t=03:48.645 | yes_bid, yes_ask | 1501
| Premier no_mid ≥ 0,60$ / ≥ 0,80$ / ≥ 0,90$ | t=01:00.407 / t=03:33.643 / t=04:12.459 | no_bid, no_ask | 1501


**B5. OBSERVATIONS BRUTES.**

- L'aller-retour coûte 0,02$ médian : toute stratégie à plus de 2–3 rotations part avec un handicap supérieur à 0,06$ sur 5,00$ de capital.
- Il existe 9 fenêtres où acheter YES et NO ensemble coûte moins de 1,00$ — un gain sans risque de 0,01$ à 0,03$ par paire.
- La conviction DOWN monte par seuils datés : 0,60$ à t=01:00.407, 0,80$ à t=03:33.643, 0,90$ à t=04:12.459.


**B6. SPÉCIFIQUE À LA SESSION.** Les 2 lignes à côté YES vide (t=00:00.237) sont un artefact d'initialisation — bruit. Le PDF annonce 112 incohérents ; mon recomptage sur les colonnes brutes en trouve 9 par somme des asks et 9 par spread négatif : le critère exact des 112 n'est pas reproductible depuis ce fichier — bruit de définition, pas signal.

### B-F2 — spot.csv (BINANCE BTC/USDT)

**B1. IDENTITÉ.** Colonnes t_ms, price ; 25 750 lignes ; période t=00:00.115 → t=04:59.151 ; flux tick par tick (86 ticks/s en moyenne). Les deux colonnes sont utilisées.

**B2. APPORT.** La granularité milliseconde du sous-jacent, 91 fois plus dense que l'oracle : c'est le seul fichier qui montre le prix AVANT que l'oracle et le carnet ne réagissent.

**B3. COMMENT.** Min/max/moyenne/écart-type sur price ; datation des extrema ; alignement au dernier tick ≤ t pour le calcul du basis contre l'oracle (B-F4).

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre de ticks | 25 750 | t_ms | 25 750
| Premier / dernier prix | 64 857,91$ / 64 824,47$ | price | 25 750
| Maximum | 64 892,00$ à t=00:37.266 | t_ms, price | 25 750
| Minimum | 64 808,00$ à t=01:05.550 | t_ms, price | 25 750
| Moyenne / écart-type | 64 844,57$ / 19,11$ | price | 25 750
| Amplitude totale | 84,00$ | price | 25 750
| Basis moyen spot − oracle | +48,00$ (écart-type 4,33$) | price + oracle.price | 284 alignements
| Basis min / max | +32,79$ / +63,11$ | price + oracle.price | 284 alignements


**B5. OBSERVATIONS BRUTES.**

- Le basis +48,00$ est stable (écart-type 4,33$) : spot − 48,00$ est un estimateur exploitable de l'oracle en avance de phase.
- Le max spot (t=00:37.266) précède le max oracle (t=00:39.000, B-F4) de 1,734 s ; le min spot (t=01:05.550) précède le min oracle (t=01:06.000) de 0,450 s.


**B6. SPÉCIFIQUE À LA SESSION.** Aucun trou de données ; la valeur du basis (+48,00$) dépend de la paire USDT/USD du jour — le niveau est session-spécifique, la stabilité est le fait intéressant. [FRAGILE - 1 SOURCE]

### B-F3 — trades.csv (FLUX AGRESSIF POLYMARKET)

**B1. IDENTITÉ.** Colonnes t_ms, usd, direction (+1 haussier, −1 baissier) ; 3097 lignes ; période t=00:01.084 → t=04:57.506 ; événementiel. Les trois colonnes sont utilisées.

**B2. APPORT.** Le sens et la taille de l'argent réellement engagé — ce que le carnet (B-F1) ne dit pas, puisqu'une cotation n'est pas un engagement.

**B3. COMMENT.** Sommes par direction ; découpage par minute ; flux net glissant 30 s = somme de usd×direction sur ]t−30 s, t] ; quantiles de taille ; comptage des ordres ≥ 100,00$.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Volume total | 43 355,06$ | usd | 3097
| Volume haussier / baissier | 18 693,17$ (1420 trades) / 24 661,89$ (1677 trades) | usd, direction | 3097
| Flux net de session | −5 968,72$ | usd, direction | 3097
| Flux net minute 0 → 4 | +3 632,03$ / −225,12$ / −1 353,33$ / −1 888,55$ / −6 133,76$ | t_ms, usd, direction | 3097
| Imbalance minute 0 → 4 | 0,649 / 0,484 / 0,370 / 0,392 / 0,200 | t_ms, usd, direction | 3097
| Flux 30 s extrêmes | +2 724,37$ à t=00:58.000 ; −5 125,60$ à t=04:30.000 | t_ms, usd, direction | 3097
| Taille médiane / moyenne / p99 | 3,60$ / 14,00$ / 112,80$ | usd | 3097
| Ordres ≥ 100,00$ | 53, dont 38 baissiers (6 775,40$) et 15 haussiers (5 963,24$) | usd, direction | 3097


**B5. OBSERVATIONS BRUTES.**

- Le flux net se dégrade de façon monotone : +3 632,03$ en minute 0, puis −225,12$, −1 353,33$, −1 888,55$, −6 133,76$.
- Le franchissement du seuil −800,00$ de flux 30 s se produit à t=02:42.905 (−842,03$) et le flux ne repasse jamais au-dessus de +80,00$ ensuite (max +80,00$ à t=03:22.000).
- 38 des 53 gros ordres sont vendeurs ; les 3 plus gros vendeurs hors clôture tombent en 268 ms à t=04:01.505/04:01.544/04:01.773 (440,00$ + 430,00$ + 440,00$).


**B6. SPÉCIFIQUE À LA SESSION.** Le trade de 1 362,31$ haussier à t=00:07.135 est la plus grosse ligne du fichier et a été du mauvais côté — signal que la taille unitaire seule ne prédit rien ici. Verdict : bruit en unitaire, signal en cumulé 30 s.

### B-F4 — oracle.csv (CHAINLINK BTC/USD)

**B1. IDENTITÉ.** Colonnes t_ms, price, ts_src ; 284 lignes ; période t=00:00.000 → t=04:58.000 ; cadence 1 s avec 15 trous > 1 s (trou max 2 s après t=04:16.000). Les trois colonnes sont utilisées — ts_src sert au contrôle qualité : 284/284 « payload », 0 fallback.

**B2. APPORT.** Le prix de règlement. C'est lui, et non Binance, qui décide UP ou DOWN contre le strike de 64 814,00$ (A-G2).

**B3. COMMENT.** Min/max/datation ; comptage des ticks au-dessus/en dessous du strike ; datation du dernier passage au-dessus du strike.

**B4. RÉSULTATS CHIFFRÉS.**

| Métrique | Valeur | Colonnes source | Nb de lignes utilisées
|-----|-----|-----|-----
| Nombre de ticks / fallback ts | 284 / 0 | t_ms, ts_src | 284
| Premier / dernier prix | 64 811,45$ / 64 776,27$ | price | 284
| Maximum | 64 843,00$ à t=00:39.000 | t_ms, price | 284
| Minimum | 64 759,99$ à t=01:06.000 | t_ms, price | 284
| Ticks > strike / < strike (64 814,00$) | 56 / 228 | price | 284
| Dernier tick > strike | t=03:21.000 | t_ms, price | 284
| Premier tick < strike ensuite | t=03:22.000 (définitif) | t_ms, price | 284
| Écart final au strike | −37,73$ | price | 284


**B5. OBSERVATIONS BRUTES.**

- L'oracle passe 228 ticks sur 284 (80,3%) sous le strike : le DOWN domine presque toute la session.
- Après t=03:22.000, l'oracle ne repasse plus jamais au-dessus du strike — le résultat est verrouillé 98 s avant la clôture, alors que NO ne cote encore que 0,80$ à t=03:33.643 (B-F1, B4).


**B6. SPÉCIFIQUE À LA SESSION.** Les 15 trous > 1 s sont des absences de rafraîchissement oracle normales — bruit. L'écart final −37,73$ est confortable ; une session finissant à ±2$ du strike rendrait ce fichier beaucoup moins lisible. [SESSION-SPÉCIFIQUE]

---

## PARTIE C — STRATÉGIES ET SIMULATION

### C0 — STRATÉGIES CANDIDATES

Les cinq observations les plus fortes, d'abord :

- **OBS-1** : le flux net agressif se dégrade de façon monotone, de +3 632,03$ (minute 0) à −6 133,76$ (minute 4), et le flux 30 s franchit −800,00$ à t=02:42.905 sans jamais remonter au-dessus de +80,00$ (B-F3, B5).
- **OBS-2** : le spot Binance précède l'oracle de 0,450 s à 1,734 s sur les extrema, avec un basis stable de +48,00$ (écart-type 4,33$) (B-F2, B5).
- **OBS-3** : 9 événements de carnet permettent d'acheter YES + NO pour moins de 1,00$, minimum 0,97$ à t=03:07.309 (B-F1, B5 ; A-G5, A5).
- **OBS-4** : l'oracle est sous le strike définitivement dès t=03:22.000, alors que NO ne cote 0,80$ qu'à t=03:33.643 et 0,90$ qu'à t=04:12.459 (B-F4, B5 ; B-F1, B4).
- **OBS-5** : 38 des 53 ordres ≥ 100,00$ sont vendeurs, concentrés en fin de session (rafale 1 310,00$ en 268 ms à t=04:01.505–04:01.773) (B-F3, B5 ; A-G3, A5).


De ces observations, quatre familles de candidates à mécanismes réellement distincts :

- **S1 — Arbitrage bi-carnet** (OBS-3) : acheter simultanément YES et NO quand yes_ask + no_ask < 1,00$ ; encaisser 1,00$ par paire à la résolution quel que soit le résultat. Entrée : somme des asks ≤ 0,99$ ; sortie : résolution. Simulation sommaire : 1 exécution (t=00:21.930, somme 0,99$, tout le capital), P&L net +0,05$.
- **S2 — Suiveur de flux 30 s** (OBS-1, OBS-5) : acheter NO quand le flux net 30 s `< −800,00$ ; revendre si le flux repasse >` +800,00$ ; sinon tenir jusqu'à résolution. Simulation sommaire : 1 trade (achat NO 0,64$ à t=02:42.905), P&L net +2,81$.
- **S3 — Ancre spot−strike** (OBS-2) : estimer l'oracle par spot − 48,00$ ; acheter NO si l'estimateur `< strike − 10,00$ et no_ask ≤ 0,80$ ; revendre si l'estimateur >` strike + 10,00$. Simulation sommaire : 5 ordres (3 achats, 2 ventes), P&L net +1,30$.
- **S4 — Verrouillage tardif** (OBS-4) : à t ≥ 04:00.000, acheter le côté que l'oracle désigne contre le strike si son ask ≤ 0,90$ ; tenir jusqu'à résolution. Simulation sommaire : 1 trade (achat NO 0,83$ à t=04:00.432), P&L net +1,02$.


**CRITÈRE DE SÉLECTION : bénéfice net en $ sur la session ; en cas d'écart inférieur à 0,10$, départage par la pire perte latente en cours de position.** Ce critère est fixé avant lecture du tableau.

Le tableau comparatif :

| Stratégie | Observations sources (A5/B5) | Nb trades | P&L net | Verdict
|-----|-----|-----|-----
| S1 Arbitrage bi-carnet | OBS-3 (B-F1 B5, A-G5 A5) | 1 paire | +0,05$ | Rejetée : gain plafonné à 1% par rareté (9 événements, somme min 0,97$)
| S2 Suiveur de flux 30 s | OBS-1, OBS-5 (B-F3 B5, A-G3 A5) | 1 | +2,81$ | **RETENUE**
| S3 Ancre spot−strike | OBS-2 (B-F2 B5, A-G2 A5) | 5 ordres | +1,30$ | Rejetée : 2 sorties prématurées payent 0,06$ de spread et rendent 1,51$ de moins que S2
| S4 Verrouillage tardif | OBS-4 (B-F4 B5, B-F1 B4) | 1 | +1,02$ | Rejetée : entre à 0,83$, soit 0,19$ de marge résiduelle contre 0,36$ pour S2


Décision : S2 domine avec +2,81$ contre +1,30$ (S3), +1,02$ (S4) et +0,05$ (S1) — aucun départage nécessaire, l'écart minimal est de 1,51$.

### C1 — TABLE DE CONVERGENCE DE LA STRATÉGIE RETENUE

| Règle | Pièces qui confirment | Pièces qui contredisent | Statut
|-----|-----|-----|-----
| R1 : entrer NO quand flux 30 s < −800,00$ | B-F3 (B5 : franchissement t=02:42.905, dégradation monotone) ; A-G3 (A5 : gros vendeurs tardifs) ; A-G4 (A4 : imbalance 0,37 puis 0,28) | A-G3 (A6 : +2 026,28$ haussiers à t=00:43 sans suite haussière — un flux peut mentir en unitaire) | Confirmée (3 pièces)
| R2 : tenir tant que flux 30 s ≤ +800,00$ | B-F3 (B5 : max +80,00$ après entrée) ; A-G3 (A4 : extrema tous négatifs après t=03:49.000) | Aucune | Confirmée (2 pièces)
| R3 : sortie de secours au bid si flux 30 s > +800,00$ | B-F1 (B5 : spread médian 0,02$, sortie bon marché) | Aucune | [FRAGILE - 1 SOURCE] — je le dis à voix haute : une seule pièce soutient le coût de sortie
| R4 : engager 100% du cash en un ordre | B-F1 (B4 : profondeur au BBO non fournie — taille exécutable non vérifiable) | Aucune pièce ne valide la profondeur | [FRAGILE - 1 SOURCE]


### C2 — LA STRATÉGIE RETENUE EN PSEUDO-CODE EXÉCUTABLE

```plaintext
CONSTANTES:
  SEUIL_ENTREE = -800.00 $   # B-F3 B5 : franchi une seule fois, t=02:42.905, jamais annulé
  SEUIL_SORTIE = +800.00 $   # B-F3 B4 : max post-entrée +80.00 $ -> jamais déclenché
  FENETRE     = 30 000 ms    # définition du flux du graphe 3 (A-G3 A1)
  FRAIS       = 0.00 $       # [HYPOTHÈSE n°1] marché Polymarket sans frais de trading

ETAT: cash = 5.00 $ ; position_NO = 0 ; verrou_ordre = LIBRE

POUR CHAQUE événement BBO b (bbo.csv, 1501 lignes complètes, B-F1 B1):
  flux30 = somme(usd * direction) sur ]b.t - 30 000 ms, b.t]   # trades.csv (B-F3 B3)

  SI position_NO == 0 ET flux30 < SEUIL_ENTREE ET verrou_ordre == LIBRE:
      verrou_ordre = PRIS                       # règle race condition (a)
      qty  = cash / b.no_ask                    # exécution à l'ask (B-F1 B2)
                                                # [HYPOTHÈSE n°2] fill intégral au BBO, sans slippage
                                                # [HYPOTHÈSE n°3] parts fractionnaires autorisées
      cash = cash - qty * b.no_ask
      position_NO = qty ; verrou_ordre = LIBRE

  SI position_NO > 0 ET flux30 > SEUIL_SORTIE ET verrou_ordre == LIBRE:
      verrou_ordre = PRIS
      cash = cash + position_NO * b.no_bid      # sortie au bid (B-F1 B2)
      position_NO = 0 ; verrou_ordre = LIBRE

A LA RÉSOLUTION (t=05:00.000): cash += position_NO * 1.00 $ si DOWN, sinon 0.00 $
```

### C-SIM — SIMULATION ALGORITHMIQUE DÉTAILLÉE

**ÉTAT DU PORTEFEUILLE.** Départ : cash 5,00$, aucune position, valeur totale 5,00$. À t=02:42.905, le premier événement BBO où flux30 < −800,00$ (−842,03$, B-F3 B5) : achat de 7,8125 parts NO à 0,64$ (no_ask, B-F1) pour 5,00$ exactement — le coût n'excède pas le cash disponible. État après : cash 0,00$, position 7,8125 NO à 0,64$, valeur au marché 7,8125 × 0,62$ (no_bid) = 4,84$. Aucun autre signal d'achat n'est finançable (cash 0,00$) et le signal de sortie n'est jamais déclenché (flux30 max post-entrée : +80,00$ à t=03:22.000). À la résolution DOWN : cash 0,00$ + 7,8125 × 1,00$ = 7,81$.

**RACE CONDITIONS.**
(a) Signal d'achat pendant un ordre en cours : le verrou par actif est pris à l'émission et libéré au fill ; tout signal reçu verrou pris est ignoré, pas mis en file. RÈGLE : verrou par actif, rejet du signal concurrent.
(b) Signal de vente sur une position non confirmée à l'achat : la vente n'est recevable que si position_NO > 0 à l'état confirmé ; un signal de vente antérieur au fill d'achat est rejeté. RÈGLE : aucune vente sans position confirmée.
(c) Deux signaux simultanés sur le même actif (même événement BBO déclenchant entrée et sortie) : l'ordre d'évaluation est déterministe — entrée testée avant sortie dans la boucle, un seul ordre par événement. RÈGLE : un événement BBO produit au plus un ordre.
(d) Fill partiel : la simulation suppose le fill intégral [HYPOTHÈSE n°2] ; si un fill partiel survenait, la quantité restante serait annulée et non re-soumise. RÈGLE : partiel = annulation du reliquat.
Dans la session simulée, aucun des cas (a)–(c) ne s'est matérialisé : un seul signal, un seul ordre.

**JOURNAL DE TRADES.** Le journal, ligne par ligne :

| # | Timestamp entrée | Prix entrée | Quantité | Timestamp sortie | Prix sortie | Frais | P&L net | Cash après
|-----|-----|-----|-----
| 1 | t=02:42.905 | 0,64$ | 7,8125 NO | t=05:00.000 (résolution DOWN) | 1,00$ | 0,00$ | +2,81$ | 7,81$


**RÉSULTATS.** Le récapitulatif :

| Indicateur | Valeur | Source (ligne(s) du journal)
|-----|-----|-----|-----
| Trades gagnants / gains cumulés | 1 / +2,81$ | ligne 1
| Trades perdants / pertes cumulées | 0 / 0,00$ | —
| Frais totaux | 0,00$ [HYPOTHÈSE n°1] | ligne 1
| BÉNÉFICE NET | +2,81$ | ligne 1
| Capital final vs initial | 7,81$ vs 5,00$ (+56,25%) | ligne 1
| Pire perte unitaire réalisée | 0,00$ (aucun trade perdant) | —
| Drawdown maximum (latent) | −1,88$ : valeur au marché 3,13$ (no_bid 0,40$ à t=03:07.642) contre 5,00$ investis à t=02:42.905 | ligne 1 + B-F1


### C4 — PORTÉE

- Le flux net 30 s < −800,00$ a précédé et accompagné la résolution DOWN de cette session. [SESSION-SPÉCIFIQUE]
- La dégradation monotone du flux par minute (+3 632,03$ → −6 133,76$) comme structure de signal cumulé mérite un test multi-sessions. [PATTERN CANDIDAT]
- L'avance du spot Binance sur l'oracle (0,450 s à 1,734 s) avec basis stable est un mécanisme structurel de la paire de flux. [PATTERN CANDIDAT]
- Le retard du carnet sur l'oracle verrouillé (NO à 0,80$ 11,6 s après le dernier passage sous strike définitif) mérite un test multi-sessions. [PATTERN CANDIDAT]
- Le rendement de +56,25% en un trade tenu à terme dépend d'un dénouement sans retour au-dessus du strike. [SESSION-SPÉCIFIQUE]
- L'arbitrage bi-carnet existe mais est plafonné à 9 fenêtres et 3% de marge maximale ici. [SESSION-SPÉCIFIQUE]


---

## AUTO-CONTRÔLE FINAL

- Contrôle 1 (comptes de valeurs) → A-G1 : 6 ; A-G2 : 6 ; A-G3 : 5+ ; A-G4 : 6 ; A-G5 : 6+ ; A-G6 : 5 lignes dont 2 « DONNÉE NON DISPONIBLE » explicites ; B-F1 : 9 métriques ; B-F2 : 8 ; B-F3 : 8 ; B-F4 : 8. Conforme.
- Contrôle 2 (traçabilité de C) → chaque règle et chaque candidate cite ses rubriques A4/A5/B4/B5 ; aucune affirmation orpheline restante. Conforme.
- Contrôle 3 (unicité des chiffres) → 7,81$, +2,81$, +56,25%, 1 trade, −842,03$, 0,64$, 7,8125 identiques en ouverture, corps et clôture. Conforme.
- Contrôle 4 (cohérence comptable) → 5,00$ − 7,8125 × 0,64$ = 0,00$ ; 0,00$ + 7,8125 × 1,00$ = 7,81$ = capital final annoncé, au centime près. Conforme.
- Contrôle 5 (hypothèses) → 3 au total : n°1 frais 0,00$ (structure de frais absente des pièces), n°2 fill intégral au BBO (profondeur absente des pièces), n°3 parts fractionnaires (granularité minimale absente des pièces). Trois, la limite ; chacune couvre une donnée matériellement absente du dossier.
- Contrôle 6 (antériorité des observations) → R1 et R2 remontent à OBS-1/OBS-5, elles-mêmes issues de B-F3 B5 et A-G3 A5, énoncées avant C0 ; R3 et R4 sont marquées [FRAGILE - 1 SOURCE]. Conforme.
- Contrôle 7 (format du speech) → verdict en ouverture, ordre F1 respecté, tableaux F4 non paraphrasés, clôture ci-dessous sans chiffre nouveau. Conforme.


## CLÔTURE DU SPEECH

Vous avez vu chaque chiffre sortir d'une pièce du dossier : le Suiveur de flux 30 s transforme 5,00$ en 7,81$, soit +2,81$ de bénéfice net.
Un trade, tenu jusqu'à la résolution, sans perte réalisée.
Étiquette globale : [SESSION-SPÉCIFIQUE] — la mécanique est candidate, la preuve exigera d'autres sessions.








## 1er : Le suiveur de flux (S2) — le plus probable de généraliser

- **P&L ici : +2,81$** (pas le meilleur, mais solide)
- Son edge est structurel : il ne prédit rien, il **suit l'argent réel** déjà engagé par les autres. Un flux net de −800$ en 30s est un fait, pas une opinion.
- Il fonctionne sur n'importe quel marché à carnet, même **sans accès à l'oracle**. C'est sa vraie force : il a le moins d'hypothèses.
- Faiblesse connue : sessions sans tendance (whipsaw de flux) → il paiera le spread.


## 2e : La combinée oracle+flux (+2,94$)

- Légèrement meilleure ici, et le mécanisme est sain : **l'oracle donne la direction, le flux sert de filtre anti-faux-départ et de stop**. Deux signaux indépendants qui doivent être d'accord.
- Mais elle exige un oracle fiable et un strike connu — applicable seulement sur des marchés type « prix au-dessus/en-dessous de X ». Moins portable que S2.


## 3e : L'ancrage dégressif (+3,06$, le gagnant local)

- Meilleur P&L ici, mais **un seul paramètre mal calibré (décote fixe au lieu de dégressive) le fait passer de +3,06$ à −5,00$**. Une stratégie dont le signe du résultat dépend d'un détail de calibration est fragile.
- Et ANCRAGE ORACLE version stricte ne trade que les paniques de fin de fenêtre : rentable seulement si ces anomalies se reproduisent.


## Verdict

**Pour appliquer ailleurs : le suiveur de flux, éventuellement renforcé par le filtre oracle quand il est disponible.** C'est le seul dont l'edge (suivre les positions réelles, pas les prédire) ne repose ni sur une calibration fine, ni sur un type de marché particulier, ni sur une anomalie rare. Les +0,13$ à +0,25$ d'écart des autres sur cette session sont du bruit statistique sur 1 échantillon — la robustesse, elle, ne l'est pas. **[1 SESSION = AUCUNE PREUVE ; à valider sur 30+ sessions avant tout engagement réel.]**




# SPEECH — AUDIT STRATÉGIQUE, SESSION POLYMARKET BTC 5 MIN DU 7 AOÛT (10:05–10:10 ET)

## 0. SYNTHÈSE EXÉCUTIVE

Vous avez le PDF, les 4 CSV et le tableau des latences sous les yeux ; voici mon verdict d'entrée.

- Stratégie retenue : **ORACLE-ANCHOR DOWN** — achat tardif de NO ancré sur le flux Chainlink, pas sur Binance.
- Capital final : **6,02 $ vs 5,00 $**, soit **+1,02 $ net réel (+20,43 %)**, après latences, slippage borné et frais P0.
- Trades : **1 trade, 1 gagnant, 0 perdant**.
- Changement Polymarket le plus impactant : depuis le **7 août 2026**, les marchés crypto up/down se résolvent sur un **TWAP Chainlink de 30 s** (fenêtre 5 min), et non plus sur un prix instantané (docs.polymarket.com/market-data/chainlink-twap, consulté le 09/08/2026).
- VERDICT DE FIABILITÉ : **[FRAGILE]**.
- Étiquette : **[PATTERN CANDIDAT]** — mécanisme structurel (écart Binance/Chainlink), mais validé sur une seule session.


---

## 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET (VÉRIFIÉ)

Je n'ai retenu aucune connaissance interne : tout ce qui suit sort de la documentation officielle consultée le 09/08/2026.

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × 0,07 × p × (1−p), en USDC, arrondi à 5 décimales | docs.polymarket.com/trading/fees | consulté 09/08/2026 | Oui — frais étendus à tous les marchés crypto depuis mars 2026 (changelog)
| P0-1 Frais maker | 0 (rebate 20 % des frais taker crypto) | docs.polymarket.com/trading/fees | consulté 09/08/2026 | Non
| P0-1 Gas / résolution | 0 $ par ordre (signature EIP-712 hors chaîne, règlement opérateur) ; frais de résolution : DONNÉE NON VÉRIFIABLE → hypothèse conservatrice 0,00 $ | docs.polymarket.com/trading/place-orders | consulté 09/08/2026 | —
| P0-2 Types d'ordres | Limit GTC/GTD (GTD : vie effective min. de 2 min env. bornée par la règle « expiration ≥ 3 min »), FOK, FAK, post-only | docs.polymarket.com/trading/place-orders + changelog | consulté 09/08/2026 | Oui — CLOB V2 depuis le 28/04/2026 (pUSD, nouveaux contrats, ordres V1 invalides)
| P0-2 Tick / taille min | tick_size 0,01 (confirmé par la grille des 1503 BBO du CSV) ; min_order_size 5 shares | docs.polymarket.com/trading/place-orders | consulté 09/08/2026 | Non pour ces marchés
| P0-3 Résolution crypto | **TWAP Chainlink Data Streams : fenêtre 30 s pour les marchés 5 min** (60 s pour 15 min/4 h) ; RTDS relaie le flux depuis le 04/08/2026 | docs.polymarket.com/market-data/chainlink-twap | consulté 09/08/2026 ; entrée en vigueur annoncée le 07/08/2026 | **OUI — c'est LE changement des dernières 48 h**
| P0-4 Rate limits | POST /order : 5 000 req/10 s (burst), 120 000/10 min ; /book et /price : 1 500 req/10 s ; WSS market channel temps réel ; heartbeat obligatoire pour bots | docs.polymarket.com/api-reference/rate-limits | consulté 09/08/2026 | Oui — limites relevées (changelog)


Ce qui a changé et ce que cela implique : jusqu'au 6 août, un snapshot de prix au moment de l'expiration décidait du résultat — un « sniping » de la dernière seconde était concevable. Depuis le 7 août (jour de cette session), le résultat est la **moyenne pondérée temps des 30 dernières secondes du flux Chainlink BTC/USD**, pas de Binance. Deux conséquences dures : (1) toute stratégie de dernière seconde est morte, il faut une moyenne soutenue ; (2) **le prix qui compte est Chainlink, pas Binance** — et vous allez voir que cette session punit précisément ceux qui regardaient Binance.

## 2. CADRE D'ANALYSE

Je travaille en timestamps relatifs à l'ouverture (t0 = 10:05:00 ET), précision milliseconde. Strike de la fenêtre : 64 814 $ (dernier tick oracle ≤ t0, PDF G2). Résultat officiel : ▼ DOWN. Tous les chiffres ci-dessous sont recalculés depuis les CSV ou lus sur les graphes, jamais estimés.

---

## PARTIE A — LES 6 GRAPHES

### A-G1 — CARNET YES/NO COMPLET (1503 ÉVÉNEMENTS BBO)

- A1. IDENTITÉ : x = temps mm:ss.mmm (0→300 s), y = prix $ (0→1) ; 6 séries (bid/ask/mid YES et NO) + proba consensus ; 112 événements marqués « carnet croisé — suspects ».
- A2. QUESTION : le carnet a-t-il « su » avant la fin que DOWN gagnait, et à quel prix pouvait-on encore l'acheter ?
- A3. MÉTHODE : lecture des extrema étiquetés et des trois zooms 15 s (00:43–00:58, 03:13–03:28, 03:49–04:04).
- A4. VALEURS EXTRAITES :

- t=00:00.237 : YES 0,47 $ / NO 0,53 $
- t=00:36.207 : YES 0,61 $ (sommet de la session) / NO 0,39 $
- t=03:20.665 : YES passe de 0,52 à 0,49 $ en 1 événement (zoom 2)
- t=03:56.387 : YES 0,18 $ / NO ask 0,82 $ (zoom 3)
- t=04:01.535 : YES 0,12 $ / NO 0,88 $
- t=04:37.067 : YES 0,01 $ / NO 0,99 $ — le carnet est terminal



- A5. OBSERVATIONS BRUTES : (i) la conviction DOWN monte par paliers dès 03:20, pas en un choc ; (ii) à 03:56 le carnet offre encore 18 % de probabilité UP alors que la fin est à 41 s ; (iii) les 112 « suspects » n'apparaissent jamais comme des fenêtres durables sur les zooms.
- A6. SPÉCIFIQUE À LA SESSION : le sommet YES 0,61 $ à 00:36.207 coïncide avec le pic Binance 64 892 $ (00:37.266, A-G2) — le marché pricait Binance, pas l'oracle.


### A-G2 — SPOT BINANCE VS ORACLE CHAINLINK + BASIS

- A1. IDENTITÉ : y = prix BTC $ ; Binance 25 750 ticks (flux direct), Chainlink 284 ticks ; ligne de strike 64 814 $ ; panneau bas : basis = spot − oracle (échelle 0→60 $).
- A2. QUESTION : les deux flux racontent-ils la même histoire par rapport au strike ?
- A3. MÉTHODE : extrema étiquetés + série Chainlink seconde par seconde.
- A4. VALEURS EXTRAITES :

- t=00:00.115 : Binance 64 857,91 $ (déjà 43,91 $ au-dessus du strike)
- t=00:00.000 : Chainlink 64 811,45 $ (2,55 $ SOUS le strike)
- t=00:37.266 : Binance max 64 892,00 $
- t=03:56.000 : Chainlink 64 788,81 $ (déficit 25,19 $)
- t=04:59.151 : Binance 64 824,47 $ — **au-dessus du strike à 1 s de la fin**
- t=04:58.000 : Chainlink 64 776,27 $ — 37,73 $ sous le strike



- A5. OBSERVATIONS BRUTES : le basis est positif sur toute la session, jamais nul, borne visible 32–63 $ sur le panneau bas.
- A6. SPÉCIFIQUE À LA SESSION : la ligne d'arrivée est la contradiction pure — Binance dit « UP possible » (64 824,47 > 64 814), l'oracle qui résout dit DOWN avec 37,73 $ de marge. C'est le cœur exploitable de la session.


### A-G3 — FLUX DIRECTIONNEL NORMALISÉ (3097 TRADES)

- A1. IDENTITÉ : y = volume $ signé fenêtré 30 s (haussier +, baissier −), échelle −6 000/+4 000 $.
- A2. QUESTION : qui pousse, quand, et en quelle taille ?
- A3. MÉTHODE : extrema étiquetés + zooms.
- A4. VALEURS EXTRAITES :

- t=00:07.135 : +1 362,31 $ (plus gros trade haussier de la session)
- t=00:43.229 : +1 205,56 $ puis t=00:43.540 : +820,72 $
- t=04:01.505 : −440,00 $ ; t=04:01.544 : −430,00 $ ; t=04:01.773 : −440,00 $ (rafale baissière)
- t=04:07.151 : −599,64 $
- t=03:56.685 : −98,40 $ ; t=03:56.791 : −94,30 $



- A5. OBSERVATIONS BRUTES : les gros tickets haussiers sont concentrés dans la minute 0 ; les gros tickets baissiers dans les minutes 4–5.
- A6. SPÉCIFIQUE À LA SESSION : la rafale −440/−430/−440 $ en 268 ms (04:01.505→04:01.773) est un acteur unique probable qui charge NO à 0,87–0,88 — après mon point d'entrée.


### A-G4 — IMBALANCE DIRECTIONNELLE

- A1. IDENTITÉ : y = haussier/(haussier+baissier) sur 30 s, référence 0,5.
- A2. QUESTION : le flux confirme-t-il le carnet ?
- A3. MÉTHODE : extrema + zooms.
- A4. VALEURS EXTRAITES : t=00:01.084 : 1,00 ; t=00:01.242 : 0,09 (bruit d'ouverture) ; t=03:49.013 : 0,37 ; t=04:03.498 : 0,28 ; t=04:57.506 : 0,30.
- A5. OBSERVATIONS BRUTES : après 03:49, l'imbalance ne repasse plus au-dessus de 0,5 sur les zooms.
- A6. SPÉCIFIQUE À LA SESSION : imbalance ≤ 0,37 sur toute la période 03:49–04:04, cohérente avec le déficit oracle croissant (A-G2).


### A-G5 — SPREADS YES ET NO

- A1. IDENTITÉ : y = ask − bid en $, échelle −0,025→0,150.
- A2. QUESTION : combien coûte l'immédiateté, et quand devient-elle chère ?
- A3. MÉTHODE : extrema + zooms.
- A4. VALEURS EXTRAITES : t=00:00.237 : 0,01 $ ; t=03:07.256 : 0,15 $ (max de session) ; t=03:07.309 : −0,03 $ (croisé, 53 ms après le max) ; t=03:20.665 : 0,08 $ ; t=03:56.387→04:02.845 : oscillation 0,01–0,02 $.
- A5. OBSERVATIONS BRUTES : le spread revient à 0,01–0,02 $ en moins de 100 ms après chaque choc ; le spread négatif ne survit jamais à l'événement suivant.
- A6. SPÉCIFIQUE À LA SESSION : au moment de mon entrée (03:57), le spread est 0,01 $ — le coût d'immédiateté est minimal précisément quand le signal est actif.


### A-G6 — ÉCART INTER-CARNETS YES VS NO

- A1. IDENTITÉ : y = yes_mid − (1 − no_mid), échelle −0,010→+0,004 ; 112 suspects marqués.
- A2. QUESTION : les carnets YES et NO offrent-ils un arbitrage interne ?
- A3. MÉTHODE : lecture de l'enveloppe sur la vue 300 s et les 3 zooms.
- A4. VALEURS EXTRAITES : borne basse −0,010 $ ; borne haute +0,004 $ ; écart = 0 sur l'essentiel des 1503 événements ; 112 suspects (7,45 % des événements) ; 0/284 fallback de timestamp.
- A5. OBSERVATIONS BRUTES : l'écart maximal (−0,010 $) est inférieur au spread courant (0,01–0,02 $) : l'incohérence est plus petite que le coût pour la capturer.
- A6. SPÉCIFIQUE À LA SESSION : aucune dérive persistante — les 112 suspects sont du bruit de séquencement, pas une poche d'arbitrage (confirmé en B-F1 : durée mesurée 0 ms).


---

## PARTIE B — LES 4 CSV

### B-F1 — bbo.csv (carnet best bid/offer)

- B1. IDENTITÉ : 5 colonnes (t_ms, yes_bid, yes_ask, no_bid, no_ask), 1503 lignes de données, t = 237 → 277 067 ms, événementiel (cadence irrégulière).
- B2. APPORT : les valeurs exactes du carnet à la milliseconde — c'est le fichier d'exécution.
- B3. CALCULS : spread = ask − bid ; somme des asks = yes_ask + no_ask ; complémentarité = yes_bid + no_ask ; persistance des croisements = durée entre l'événement croisé et le suivant.
- B4. RÉSULTATS :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Spread YES moyen | 0,0194 $ | yes_ask, yes_bid | 1 501
| Spread YES médian | 0,0200 $ | yes_ask, yes_bid | 1 501
| Spread YES max / min | 0,15 $ / −0,03 $ | yes_ask, yes_bid | 1 501
| Événements somme-des-asks < 1 | 9 (min 0,97) | yes_ask + no_ask | 1 501
| Durée de persistance de ces 9 événements | 0 ms chacun | t_ms | 9
| Lignes non complémentaires (yes_bid+no_ask ≠ 1) | 5 / 1 501 | yes_bid, no_ask | 1 501
| Mid YES max | 0,615 à t=36 207 ms | yes_bid, yes_ask | 1 501
| NO ask à t=236 566 ms | 0,82 $ | no_ask | 1
| Dernier état (t=277 067 ms) | YES 0,01 / NO 0,99 | toutes | 1


- B5. OBSERVATIONS BRUTES : le NO ask est encore à 0,82 $ à 03:56.566, à 0,88 $ à 04:01.535, à 0,99 $ à 04:37.067 — la certitude se paie de plus en plus cher, mais lentement.
- B6. ANOMALIES : les 9 croisements (t = 21 930 ; 72 037 ; 92 443 ; 187 309 ; 205 351 ×3 ; 220 474 ; 244 323 ms) partagent tous leur timestamp avec l'événement de correction : durée exploitable mesurée = 0 ms. Bruit de séquencement, pas signal.


### B-F2 — spot.csv (Binance BTC/USDT, flux direct)

- B1. IDENTITÉ : 2 colonnes (t_ms, price), 25 750 lignes, t = 115 → 299 151 ms, cadence tick (jusqu'à 60+ ticks partageant la même milliseconde).
- B2. APPORT : ce que voyait un trader branché sur Binance — la « fausse » référence.
- B3. CALCULS : min/max, position vs strike 64 814 $, comptage de croisements du strike.
- B4. RÉSULTATS :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Premier prix | 64 857,91 $ (t=115 ms) | price | 1
| Dernier prix | 64 824,47 $ (t=299 151 ms) | price | 1
| Max | 64 892,00 $ à t=37 266 ms | price, t_ms | 25 750
| Min | 64 808,00 $ à t=65 550 ms | price, t_ms | 25 750
| Croisements du strike | 8 | price | 25 750
| Écart dernier prix − strike | +10,47 $ | price | 1
| Amplitude de session | 84,00 $ | price | 25 750
| Part du temps au-dessus du strike | majoritaire en minutes 0–1, y compris le dernier tick | price | 25 750


- B5. OBSERVATIONS BRUTES : Binance ne descend jamais sous 64 808 $ — soit 6 $ sous le strike au pire. Vu de Binance, cette session est un coin-flip serré.
- B6. ANOMALIES : rafales de 50+ ticks par milliseconde (t=261, 2 151, 2 675 ms…) — bursts de matching Binance, sans incidence sur les conclusions.


### B-F3 — oracle.csv (Chainlink BTC/USD, 284 ticks)

- B1. IDENTITÉ : 3 colonnes (t_ms, price, ts_src), 284 lignes, t = 0 → 298 000 ms, cadence médiane 1 000 ms (max 2 000 ms), ts_src = payload sur 284/284 (0 fallback).
- B2. APPORT : le seul prix qui compte pour la résolution depuis le 7 août (P0-3).
- B3. CALCULS : déficit = strike − oracle ; TWAP par paliers sur 270 000→300 000 ms ; basis = Binance(t) − oracle(t) (dernier tick Binance ≤ t) ; max de remontée sur toute fenêtre glissante de 30 s.
- B4. RÉSULTATS :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Premier / dernier prix | 64 811,45 $ / 64 776,27 $ | price | 2
| Min / Max | 64 759,99 $ / 64 843,00 $ | price | 284
| **TWAP 270–300 s (résolution)** | **64 775,68 $ → DOWN (déficit 38,32 $)** | t_ms, price | 30
| Basis Binance − Chainlink : moyenne | +48,11 $ | les 2 fichiers | 284
| Basis : médiane / min / max | +48,08 $ / +32,79 $ / +63,11 $ | les 2 fichiers | 284
| Max de remontée oracle sur 30 s | +58,13 $ (à t=95 000 ms) | t_ms, price | 284
| P95 des remontées sur 30 s | +47,08 $ | t_ms, price | 284
| Déficit à t=236 000 / 237 000 ms | 25,19 $ / 30,06 $ | price | 2
| Croisements du strike | 10 | price | 284


- B5. OBSERVATIONS BRUTES : (i) le basis ne passe JAMAIS sous +32,79 $ — Binance surestime systématiquement le prix de résolution ; (ii) à partir de t=232 000 ms (03:52), le déficit reste ≥ 18,76 $ sans interruption jusqu'à la fin.
- B6. ANOMALIES : 15 secondes sans tick (intervalles de 2 000 ms, ex. t=124 000→126 000 ms) — trous de publication d'1 s, sans fallback de timestamp.


### B-F4 — trades.csv (3 097 trades du marché)

- B1. IDENTITÉ : 3 colonnes (t_ms, usd, direction ±1), 3 097 lignes, t = 1 084 → 297 506 ms.
- B2. APPORT : la profondeur réellement consommée — ma seule référence de slippage, le BBO n'ayant pas de tailles.
- B3. CALCULS : sommes par direction, par minute, percentiles de taille, flux autour de mon point d'entrée.
- B4. RÉSULTATS :


| Métrique | Valeur | Colonnes source | Nb lignes utilisées
|-----|-----|-----|-----|-----
| Volume total | 43 355,06 $ | usd | 3 097
| Volume haussier / baissier | 18 693,17 $ / 24 661,89 $ | usd, direction | 3 097
| Imbalance globale | 0,4312 | usd, direction | 3 097
| Nb trades up / down | 1 420 / 1 677 | direction | 3 097
| Taille médiane / moyenne / p90 | 3,60 $ / 14,00 $ / 30,45 $ | usd | 3 097
| Net par minute (0→4) | +3 632,03 / −225,12 / −1 353,33 / −1 888,55 / −6 133,76 $ | usd, direction | 3 097
| Imbalance minute 4 | 0,2003 | usd, direction | segment
| Flux 235–245 s | 112 trades, 2 568,16 $ dont 2 367,68 $ côté DOWN | t_ms, usd, direction | 112


- B5. OBSERVATIONS BRUTES : 2 367,68 $ s'exécutent côté DOWN dans les 10 s qui entourent mon entrée — un ordre de 4,94 $ y représente 0,21 % du flux.
- B6. ANOMALIES : timestamps localement non monotones (ex. lignes 3 263 ms après 3 284 ms) — arrivée réseau désordonnée, amplitude < 50 ms, sans impact sur les agrégats.


---

## PARTIE C — STRATÉGIES ET SIMULATION

### C0 — CANDIDATES

Mes cinq observations les plus fortes, chacune tracée à sa source :

- **OBS-1** (B-F3/B4) : basis Binance − Chainlink positif sur 100 % de la session — moyenne +48,11 $, jamais sous +32,79 $.
- **OBS-2** (B-F2/B-F3, A-G2/A4) : au dernier tick, Binance 64 824,47 $ > strike 64 814 $ alors que Chainlink 64 776,27 $ → un lecteur Binance croit UP possible ; la résolution TWAP (64 775,68 $) dit DOWN.
- **OBS-3** (B-F3/B5) : déficit oracle ≥ 18,76 $ sans interruption de t=232 s à la clôture ; ≥ 25,19 $ dès t=236 s.
- **OBS-4** (B-F1/B4, A-G1/A4) : NO ask encore à 0,82 $ à t=236 566 ms — le carnet vend 18 % de probabilité UP alors qu'annuler le déficit exigerait une remontée soutenue supérieure au max observé sur 30 s (58,13 $, OBS-5) maintenue sur la moyenne.
- **OBS-5** (B-F1/B6, A-G6/A5) : les 9 carnets croisés (somme des asks 0,97–0,99) durent 0 ms mesurée.


**Candidate C-A — Arbitrage de carnet croisé** (acheter YES ask + NO ask quand la somme `< 1). Logique : gain sans risque de 1 − somme, max 3 % (somme 0,97). Conformité P0 : OK (FAK, tick 0,01). **Faisabilité structurelle : latence de boucle = 31 ms (lecture WSS CLOB) + 31 ms (envoi) + 31 ms (confirmation) = 93 ms ; vie du signal = 0 ms (OBS-5). Boucle >` signal → [STRUCTURELLEMENT IMPOSSIBLE]. Éliminée.**

**Candidate C-B — Momentum de latence Binance→CLOB** (le spot Binance précède le mid YES : corrélation 0,248 entre le retour spot 2 s et le mouvement du mid 500 ms plus tard, calcul B-F1×B-F2). Logique : front-runner le carnet sur les mouvements spot. Conformité P0 : OK. Faisabilité : boucle = 219 ms (âge du signal Binance) + 31 ms (envoi) + 31 ms (confirmation) = 281 ms, vie du signal 500–1 000 ms : possible. **Mais l'économie est morte : gain espéré par trade ≤ 1 tick (0,01 $/share), frais taker à p≈0,50 = 0,07 × 0,50 × 0,50 = 0,0175 $/share (P0-1), plus un spread médian de 0,02 $ (B-F1). Frais + spread > edge sur chaque trade → P&L net réel structurellement négatif. Éliminée au critère.**

**Candidate C-C — ORACLE-ANCHOR DOWN** (mécanisme : divergence de référentiel, ni arbitrage de carnet ni momentum). Logique : la foule price Binance (OBS-1, OBS-2, A-G1/A6) alors que la résolution est un TWAP Chainlink 30 s (P0-3) ; quand le déficit Chainlink est profond et tardif, le NO coté 0,82 $ est structurellement sous-évalué. Règles chiffrées : entrée si t ≥ 230 s ET déficit ≥ 25 $ sur deux ticks oracle consécutifs (OBS-3), exécution FAK au NO ask (OBS-4), tenue jusqu'à résolution. Conformité P0 : FAK autorisé, taille 6,02 shares ≥ min 5, tick respecté, frais intégrés. Faisabilité : boucle = 68 ms (âge Chainlink on-chain) + 31 ms (envoi CLOB) + 31 ms (confirmation) = **130 ms** ; vie du signal = 62 s (déficit ≥ 25 $ de t=236 s à t=298 s, B-F3). **Boucle/signal = 0,2 %. Faisable.**

**CRITÈRE DE SÉLECTION : P&L net réel sur la session (après latences, slippage borné, frais P0-1), sous condition de faisabilité structurelle.** Annoncé avant comparaison.

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----|-----
| C-A Arbitrage croisé | OBS-5 | Oui | 0 possible | +0,15 $ théorique | 0,00 $ (inexécutable) | [STRUCTURELLEMENT IMPOSSIBLE]
| C-B Momentum latence | corr. 0,248 (B-F1×B-F2) | Oui | 10–30 | ≤ +0,01 $/share | négatif (frais 0,0175 > edge 0,01) | Éliminée
| C-C Oracle-Anchor DOWN | OBS-1,2,3,4 | Oui | 1 | +1,0962 $ | **+1,0214 $** | **RETENUE**


### C1 — STRATÉGIE RETENUE : ORACLE-ANCHOR DOWN

```plaintext
# ORACLE-ANCHOR DOWN — 1 trade max par fenêtre de 5 min
PARAMÈTRES:
  SEUIL   = 25 $      # déficit strike − chainlink (OBS-3, B-F3)  [FRAGILE - 1 SOURCE]
  T_MIN   = 230 s     # zone tardive: 70 s avant clôture          [FRAGILE - 1 SOURCE]
  BOUCLE  = 130 ms    # 68 (âge Chainlink) + 31 (envoi CLOB) + 31 (confirmation) — tableau des latences
  PLAFOND_PRIX = 0.90 # au-delà, edge net < risque résiduel (D7)

À CHAQUE TICK oracle (cadence 1 000 ms, B-F3):
  SI t >= T_MIN
     ET (strike - oracle[t])   >= SEUIL          # réf. B4/B-F3
     ET (strike - oracle[t-1]) >= SEUIL          # 2 ticks consécutifs anti-bruit
     ET aucun ordre en vol ET position vide:
        prix_ref = NO ask courant (WSS CLOB)     # réf. B4/B-F1
        SI prix_ref <= PLAFOND_PRIX:
           ENVOYER FAK BUY NO, taille = max C tel que C×ask + C×0.07×ask×(1-ask) <= cash   # P0-1
           [HYPOTHÈSE n°1] le meilleur ask absorbe 5 $ (justifié: 2 367,68 $ de flux DOWN en 10 s, B-F4/B5)
TENIR jusqu'à résolution TWAP (P0-3). Aucune sortie anticipée: le signal est terminal.
```

Chaque condition remonte à une observation antérieure : SEUIL et T_MIN à OBS-3, l'exécution au ask à OBS-4, la légitimité du référentiel oracle à OBS-1/OBS-2 et P0-3.

### C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE** : cash initial 5,00 $, aucune position, aucun ordre.

**RACE CONDITIONS** (règles déterministes) :

- (a) Signal pendant ordre en vol → **RÈGLE : ignoré** (drapeau in_flight, un seul ordre vivant).
- (b) Vente sur position non confirmée → **RÈGLE : interdite** ; ici aucune vente n'existe (tenue à résolution).
- (c) Deux signaux simultanés → **RÈGLE : priorité au timestamp oracle le plus ancien, 1 trade par fenêtre.**
- (d) Fill partiel → **RÈGLE : le FAK conserve le rempli, le reste est annulé ; pas de re-envoi.**


**JOURNAL DE TRADES** (réalisme r1–r5 intégré) :

| # | Ts entrée signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----
| 1 | 03:57.000 (tick Chainlink 64 783,94 $, déficit 30,06 $ ; tick précédent 25,19 $ ≥ 25) | 03:57.130 (+130 ms de boucle, r1–r2) | NO ask 0,82 $ (BBO de 03:56.566, inchangé jusqu'à 03:57.505 — B-F1) | 6,02 shares (coût 4,9364 $, r5 : 4,9986 $ ≤ 5,00 $) | résolution (t ≥ 300 s, TWAP 64 775,68 $ → DOWN) | 1,00 $ | 0,00 $ (r3 : niveau présent à 03:57.130) | 0,0622 $ (P0-1 : 6,02×0,07×0,82×0,18) | **+1,0214 $** | 0,0014 $ puis 6,0214 $ après règlement


Vérification r3 énoncée à l'avance : si le niveau 0,82 avait disparu, re-pricing unique au ask suivant 0,83 → 5,95 shares, frais 0,0588 $, P&L net +0,9527 $ (+19,05 %) — borne basse du slippage.

**RÉSULTATS** :

- Trades : 1 gagnant (+1,0214 $), 0 perdant. Frais totaux : 0,0622 $.
- **BÉNÉFICE NET : +1,0214 $. Capital final : 6,0214 $ vs 5,00 $ (+20,43 %).**
- Pire perte réalisée : 0,00 $. Drawdown latent max : NO bid touche 0,78 $ à 03:58.033 (B-F1) → valorisation 4,6956 $ contre 4,9986 $ engagés = −0,3030 $ (−6,06 % du capital), résorbé à partir de 04:00.432.
- **P&L THÉORIQUE (sans r1–r4)** : 6,09 shares à 0,82 $, sans frais → +1,0962 $ (+21,92 %). **Coût du réalisme : 0,0748 $**, dont 0,0622 $ de frais et 0,0126 $ d'arrondi de taille ; la latence a coûté 0,00 $ sur ce trade (même prix à t_signal et t_signal+130 ms).


---

## PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----
| D1 Latence de boucle | 130 ms n'altère pas le prix | NO ask = 0,82 $ à t_signal ET à t_signal+130 ms (B-F1, BBO 03:56.566 stable jusqu'à 03:57.505) | 0,00 $ — prouvé | [SANS IMPACT]
| D2 Fraîcheur du signal | un tick oracle vieux de ≤ 1 068 ms (68 ms + cadence 1 000 ms) suffit | avec 1 tick de retard (entrée 03:58.130), le NO ask est 0,80 $ (B-F1, BBO 03:57.533) : prix meilleur | borné ±0,02 $/share ; 0,00 $ réalisé | [SANS IMPACT]
| D3 Slippage / profondeur | le meilleur ask absorbe 5 $ | tailles absentes du BBO ; 2 367,68 $ de flux DOWN exécuté en 235–245 s (B-F4) ; borne re-pricing 0,83 $ | −0,0687 $ au pire (+0,9527 $ au lieu de +1,0214 $) | [DÉGRADE]
| D4 Frais complets + gas | frais P0-1, gas 0 | 0,0622 $ (formule officielle) ; 0 gas par ordre (P0-2) | −0,0622 $ intégré | [DÉGRADE]
| D5 Niveau disparu / fill partiel | FAK conserve le rempli | fill 50 % → +0,51 $ ; fill 0 % → 0,00 $ (capital intact) | proportionnel, jamais négatif | [DÉGRADE]
| D6 Défaillances techniques | ordre non confirmé → pas de re-envoi aveugle | statut `delayed`/heartbeat CLOB (P0-2, P0-4) ; échec = 0 trade, 5,00 $ intact | perte d'opportunité, pas de perte en capital | [DÉGRADE]
| D7 Passage à l'échelle | valable pour 5 $ | flux DOWN disponible 235–245 s = 2 367,68 $ (B-F4) ; au-delà de quelques centaines de $, l'ask migre vers 0,85–0,88 (B-F1, 04:01) et l'edge net tend vers 0 au-dessus de 0,90 (plafond C1) | extinction estimée au-delà de la centaine de $ par fenêtre | [DÉGRADE]
| D8 Dépendance à la session | ne trade que si déficit ≥ 25 $ tardif | sur session calme (déficit < 25 $) : 0 trade, capital 5,00 $ intact ; MAIS le gain observé provient d'un seul trade sur une seule session, et exige la persistance du basis +48,11 $ (OBS-1) | résultat = 1 événement ; reproductibilité non démontrée | [DÉGRADE]
| D9 Conformité au nouveau Polymarket | la stratégie EXPLOITE la résolution TWAP Chainlink 30 s | changement vérifié P0-3 (docs.polymarket.com/market-data/chainlink-twap, consulté 09/08/2026) ; le TWAP recalculé (64 775,68 $) reproduit le résultat officiel DOWN | conformité totale ; la stratégie dépend du NOUVEAU mécanisme, pas d'un mécanisme supprimé | [SANS IMPACT]


**Point de risque assumé, chiffré** : au moment de l'entrée, le déficit (30,06 $) était INFÉRIEUR au max de remontée oracle observé sur 30 s (58,13 $, B-F3). Le trade n'était donc pas sans risque — c'était un achat de probabilité ~0,90–0,95 payé 0,82, pas un arbitrage. Ce chiffre m'interdit de plaider le « sûr ».

**VERDICT GLOBAL (règles mécaniques)** : zéro [DÉTRUIT] ; P&L net réel positif (+1,0214 $) ; conformité P0 totale. Mais D8 constate une dépendance à une configuration observée une seule fois (un seul trade, une seule session, un basis non expliqué causalement). La condition « D8 ≠ dépendance exclusive à un événement unique » du label [FIABLE] n'est pas remplie. **VERDICT : [FRAGILE]** — mécanisme économiquement cohérent et structurellement exécutable, preuve statistique insuffisante.

---

## AUTO-CONTRÔLE FINAL

1. Phase 0 complète → P0-1 à P0-4 sourcés (fees, place-orders, chainlink-twap, rate-limits — docs.polymarket.com, consultés 09/08/2026) ; changement TWAP du 07/08/2026 identifié ; gas de résolution marqué DONNÉE NON VÉRIFIABLE avec hypothèse 0,00 $. ✔
2. Minimum 5 valeurs par graphe (A-G1 : 6 ; A-G2 : 6 ; A-G3 : 5+ ; A-G4 : 5 ; A-G5 : 5 ; A-G6 : 5) et 8+ par CSV (tableaux B4 : 9, 8, 9, 8). ✔
3. Chaque règle de C1 référencée → SEUIL/T_MIN : OBS-3 ; exécution : OBS-4 ; référentiel : OBS-1/OBS-2/P0-3 ; deux paramètres marqués [FRAGILE - 1 SOURCE]. ✔
4. Latence de boucle reconstituée chemin par chemin (68 + 31 + 31 = 130 ms ; C-A : 93 ms ; C-B : 281 ms) et appliquée à l'unique entrée du journal. ✔
5. Cohérence comptable → 5,00 − 4,9364 − 0,0622 = 0,0014 ; 0,0014 + 6,02 = 6,0214 $ = capital final annoncé, au centime (au dixième de centime) près. ✔
6. D1–D9 traités ; les deux [SANS IMPACT] portent leur preuve chiffrée (0,82 $ = 0,82 $ pour D1 ; 0,80 $ ≤ 0,82 $ pour D2) ; verdict global conforme aux règles mécaniques (D8 bloque [FIABLE]). ✔
7. Unicité des chiffres → 6,0214 $ / +20,43 % / 1 trade / 0,0622 $ / 64 775,68 $ / +48,11 $ identiques en ouverture et dans le corps ; la clôture n'introduit rien. ✔


## CLÔTURE

Le changement de résolution a déplacé la vérité du prix de Binance vers l'oracle, et cette session montre un carnet qui ne l'avait pas encore intégré. La stratégie qui en découle a gagné ici, exécutable dans vos latences réelles, mais une session ne fait pas une preuve. Je recommande la collecte des prochaines fenêtres avant tout engagement au-delà du capital de test.

---

**Note de méthode** : tous les chiffres ont été recalculés par scripts sur les 4 CSV copiés dans `/vercel/share/v0-project/data/` (bbo.csv, spot.csv, oracle.csv, trades.csv) — spread, basis, TWAP par paliers, fenêtres glissantes 30 s, flux par minute et journal de simulation sont reproductibles depuis ces fichiers.


