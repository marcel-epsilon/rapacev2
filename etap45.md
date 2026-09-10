# Ã‰TAPE 4 â€” ANALYSE DE CONSENSUS
## Ancrage Ã  l'oracle Ã  dÃ©cote dÃ©gressive Â· livrable d'implÃ©mentation

**Corpus traitÃ© :** `ETAP4.MD`, 9 988 lignes, ~25 rapports d'agents indÃ©pendants, **41 blocs `ANALYSE DE LA SESSION`** couvrant ~25 fenÃªtres BTC 5 min distinctes du 18/08/2026 (08:50 â†’ 15:05 ET), deux gÃ©nÃ©rations de gabarit. Sessions UP et DOWN toutes deux prÃ©sentes : le test de non-directionnalitÃ© sous forme opÃ©rationnelle est donc calculable au niveau du consensus, alors qu'il ne l'Ã©tait dans aucun rapport isolÃ©.
**Autres sources :** PDF `verdict-audit-17-pages` (Ã‰tape 3, structure des 22 tests) ; documents d'Ã‰tape 1 (1.1 inventaire des 60 termes, 1.3.bis, 1.4, 1.5, 1.6, Â§4.4 rectifiÃ©).
**Documents 5 et 6 :** vides. Reconstruits en 0.1 â†’ 0.7 depuis la rubrique 0 des rapports de seconde gÃ©nÃ©ration et le PDF, chaque ligne marquÃ©e Â« reconstruit Â».
**Convention de provenance :** `[PDF]` `[A1G]` (agents 1Ê³áµ‰ gÃ©nÃ©ration) `[A2G]` (agents 2áµ‰ gÃ©nÃ©ration) `[E1]` (Ã‰tape 1) `[R]` (raisonnement de consolidation).

---

# 0. Cartographie structurelle du systÃ¨me

## 0.1 Ã‰lÃ©ments du systÃ¨me

| Ã‰lÃ©ment | RÃ´le | Ce qu'il sait et Ã  quel instant | Source | Confiance |
|---|---|---|---|---|
| Sous-jacent (spot Binance) | Prix rÃ©el de BTC/USD | Prix au timestamp de capture ; latence de publication source **absente des CSV**. Latence nominale 0,219 s citÃ©e en Ã‰tape 1 | `[E1]` O4 + `[A2G]` rubrique 0.2 â€” reconstruit | Moyenne |
| Oracle (Chainlink/RTDS, TWAP 30 s) | **Source unique de rÃ©solution.** DÃ©cide UP/DOWN | DerniÃ¨re valeur du payload Ã  la capture. Cadence nominale 1 s (P10/P50/P90 = 1 s, trou max 7 s). `ts_src` vaut littÃ©ralement `payload` sur 269/269 lignes : **aucune heure source** | `[A2G]` 0.1 + `[PDF]` p. 16 â€” reconstruit | Ã‰levÃ©e sur la cadence, nulle sur l'heure source |
| Strike officiel `K` | Pivot de rÃ©solution, lu 1Ã— Ã  Ï„=0 | **Non observable.** Aucun champ `K`, `k_source`, ni TWAP publiÃ© dans aucune session. Tous les agents ont utilisÃ© un proxy | `[A2G]` 0.2 (unanime) | Ã‰levÃ©e sur l'absence |
| Horloge de fenÃªtre `Ï„` | Origine `endDate âˆ’ 300 s` | **Non observable.** `t_ms` est un relatif de capture ; aucun `endDate`, aucune date absolue | `[A2G]` 0.2 + `[PDF]` p. 14 | Ã‰levÃ©e sur l'absence |
| Carnet de la session (CLOB) | OÃ¹ le bot agit | Meilleurs niveaux `yes_bid/yes_ask/no_bid/no_ask` uniquement. Cadence Ã©vÃ©nementielle : intervalle mÃ©dian **0,013â€“0,015 s**, P90 0,90â€“1,19 s, max 6,5â€“9,2 s | `[A2G]` 0.1 + 2 | Ã‰levÃ©e |
| Profondeur du carnet | CapacitÃ© d'absorption | **Non observable.** Aucun niveau 2, aucune quantitÃ©, aucun prix par niveau, aucune annulation. T21 est donc fail-closed par construction | `[A2G]` 0.2 (unanime) ; `[E1]` 1.4 trou nÂ°2 | Ã‰levÃ©e sur l'absence |
| Moteur d'appariement | ExÃ©cute et publie | **Non observable.** Aucun `order_id`, aucun accusÃ©, aucune exÃ©cution attribuÃ©e. Les changements de BBO sont visibles, jamais leur cause | `[A2G]` 0.2 | Ã‰levÃ©e sur l'absence |
| Carnets concurrents | MÃªme sous-jacent, autre session | **Non observable.** Aucun fichier concurrent dans aucune session. Source d'edge nÂ°4 du cahier des charges : intÃ©gralement non mesurÃ©e | `[A2G]` 0.2 | Ã‰levÃ©e sur l'absence |
| Flux de trades | Empreinte des populations | `t_ms`, `usd`, `direction` (Â±1, **sans dictionnaire**). Ni prix, ni cÃ´tÃ© contractuel, ni maker/taker, ni identitÃ© | `[A2G]` 0.1 + 4 | Ã‰levÃ©e |
| Bot | Ã‰lÃ©ment Ã  insÃ©rer | Aucune trace dans aucune session. Budget de latence **non dÃ©terminable** | `[A2G]` 0.2 | Ã‰levÃ©e sur l'absence |
| RÃ©solution officielle | Verdict | Absente des CSV. Certains agents disposaient de `issue_officielle` par le PDF de session, d'autres l'ont infÃ©rÃ©e par TWAP partiel `[270;298]` | `[A1G]`+`[A2G]` | Moyenne |

**ConsÃ©quence de cartographie, Ã  lire avant toute rÃ¨gle.** Sur les cinq sources d'edge du cahier des charges, **deux sont mesurables** (oracle â†’ carnet ; Ã©carts structurels), **une est partiellement mesurable** (populations, par taille de trade seulement), et **deux sont non mesurables** dans les donnÃ©es existantes (retards d'action, carnets concurrents). Toute rÃ¨gle qui reposerait sur ces deux derniÃ¨res est spÃ©culative, quel que soit le nombre d'agents qui la proposent.

## 0.2 Liens et latences

| Ã‰lÃ©ment A â†’ Ã‰lÃ©ment B | Nature du lien | Latence mÃ©diane | P10 / P90 | Horloge | n_conf / n_mes | Sources |
|---|---|---|---|---|---|---|
| **oracle â†’ carnet** (corrÃ©lation croisÃ©e des incrÃ©ments, argmax signÃ© ; `L>0` = l'oracle mÃ¨ne) | RÃ©action informationnelle | **âˆ’0,75 s** (mÃ©diane des argmax de session) | P10 **âˆ’28,25 s** / P90 **0 s** ; min âˆ’50 s, max +1 s | Capture relative du carnet, 1 Hz et 4 Hz selon agent â€” **horloges diffÃ©rentes, signalÃ©** | **27 / 27 mesurantes ; 0/27 dans la bande PDF [+5;+60] ; 26/27 â‰¤ 0 s** | `[A1G]`+`[A2G]`, ~14 agents |
| oracle â†’ carnet (proxy conditionnel : premiÃ¨re variation BBO de mÃªme signe aprÃ¨s `|Î”oracle| â‰¥ 0,5 $`) | **Proxy, pas causalitÃ©** | 1,83 s / 2,59 s / 2,58 s / 2,81 s / 3,44 s / 1,86 s selon session | P90 6,7 s â†’ 32,3 s selon session ; max 55,7 s | Capture relative | 6 / 6 | `[A2G]` rubrique 2 |
| spot â†’ oracle | RÃ©action | **0 Ã  +1 s** (r = 0,36 â†’ 0,58) | â€” | Capture relative | 6 / 6 | `[A1G]` |
| spot â†’ carnet | RÃ©action | **0 s** (r = 0,22 â†’ 0,43) | â€” | Capture relative | 3 / 3 | `[A1G]` |
| carnet â†’ carnet suivant | Cadence de capture Ã©vÃ©nementielle | **0,013â€“0,015 s** | P90 0,90â€“1,19 s ; max 6,5â€“9,2 s | Horodatage de capture du carnet | 6 / 6 | `[A2G]` |
| oracle â†’ oracle suivant | Cadence du payload | **1,000 s** | P10/P90 = 1 s dans l'entrÃ©e ; max 7 s | Capture du payload | 6 / 6 | `[A2G]` |
| population â†’ moteur d'appariement | Retard d'action | **Non dÃ©terminable** â€” champs Ã  journaliser : `order_id`, `t_decision`, `t_envoi`, `t_ack`, `t_fill`, `t_annulation` (Â§ 8.6) | â€” | Horloge serveur, â‰¤ 10 ms requis | 0 / 41 | `[A2G]` rubrique 8 |
| bot â†’ moteur d'appariement (aller-retour) | Budget de latence | **Non dÃ©terminable** â€” aucune session ne contient d'ordre du bot | â€” | Horloge serveur | 0 / 41 | `[A2G]` |
| carnet concurrent â†’ carnet session | Retard inter-carnets | **Non dÃ©terminable** â€” aucune seconde source | â€” | â€” | 0 / 41 | `[A2G]` |
| profondeur â†’ spread | RÃ©ponse Ã  la consommation | **Non dÃ©terminable** â€” aucune quantitÃ© | â€” | â€” | 0 / 41 | `[A2G]` |

**Avertissement d'horloge (rÃ¨gle 12 et rÃ¨gle de consensus 5).** Les argmax de corrÃ©lation croisÃ©e ont Ã©tÃ© mesurÃ©s Ã  1 Hz par une partie des agents et Ã  4 Hz par une autre, et sur deux estimateurs distincts (incrÃ©ments vs niveaux). Les deux familles sont prÃ©sentÃ©es ensemble ci-dessus **parce qu'elles concordent en signe et en ordre de grandeur** : `L â‰¤ 0` dans 26 cas sur 27, aucune valeur dans la bande du PDF. Le bot devra utiliser **l'horloge de capture du carnet Ã  4 Hz minimum**, seule cadence qui voie les Ã©vÃ©nements de 0,279 s et 0,300 s recensÃ©s par le PDF (p. 16).

**Lecture dÃ©cisive.** La latence de communication oracle â†’ carnet, qui est la justification Ã©crite de toute la stratÃ©gie, **est nulle ou nÃ©gative**. Il n'existe pas d'intervalle de propagation Ã  exploiter entre l'oracle et le carnet. Cela ne tue pas l'ancrage Ã  l'oracle : cela change ce que la dÃ©cote rÃ©munÃ¨re (Â§ 4.6, mÃ©canisme M1).

## 0.3 Populations d'acteurs

| Population | Signature | Ce qu'elle voit avant les autres | Ce qu'elle laisse aux autres | Sources |
|---|---|---|---|---|
| **Micro-flux** | `usd â‰¤ 5 $` ; 913/1 465 trades = **62,3 % des Ã©vÃ©nements** mais **11,0 % du notionnel** ; taille mÃ©diane 2 $ ; dÃ©lai intra-direction 0,146 s | Non dÃ©terminable | Non dÃ©terminable | `[A2G]` rubrique 4 |
| **Flux moyen** | `5 < usd â‰¤ 30 $` ; 403/1 465 = 27,5 % des Ã©vÃ©nements, **25,8 %** du notionnel ; mÃ©diane 11,42 $ | Non dÃ©terminable | Non dÃ©terminable | `[A2G]` |
| **Gros flux** | `usd > 30 $` ; 149/1 465 = **10,2 % des Ã©vÃ©nements** mais **63,2 % du notionnel** ; mÃ©diane 57,64 $, P90 178,65 $, max 712,40 $ ; concentrÃ© sur quelques instants (`22,9 s`, `142,0 s`, `240,4 s`, `272,8 s`, `278,3 s`) | Non dÃ©terminable | Non dÃ©terminable | `[A2G]` |
| Flux signÃ© agrÃ©gÃ© | `+1` : 487 trades / 6 616,84 $ ; `âˆ’1` : 978 trades / 14 973,10 $ ; somme signÃ©e âˆ’8 356,26 $. **SÃ©mantique du signe non fournie** | Le signe de `direction`, dont le cÃ´tÃ© contractuel est inconnu | Non dÃ©terminable | `[A2G]` |
| Fournisseurs de liquiditÃ© passive | DÃ©duits de l'existence d'un spread mÃ©dian de 0,02 et de l'invariant de complÃ©mentaritÃ© | Non dÃ©terminable | Non dÃ©terminable | `[R]` |

**Verdict de population, citÃ© textuellement `[A2G]` :** Â« *Aucune population ne peut donc Ãªtre nommÃ©e comme celle qui referme l'Ã©cart ; le faire serait une attribution inventÃ©e.* Â»

**ConsÃ©quence sur la rÃ¨gle de travail 11.** La rÃ¨gle exige de nommer la contrepartie de chaque relation retenue. Les donnÃ©es ne le permettent pas. Toute relation retenue dans ce livrable porte donc, en lieu et place d'une contrepartie nommÃ©e, **la classe de signature la plus probable et la mention explicite Â« contrepartie non identifiable â€” champ `maker/taker` et `order_id` Ã  journaliser Â»**. Aucune dÃ©cision n'est prise comme si la contrepartie Ã©tait connue. Une seule structure asymÃ©trique est solidement Ã©tablie et exploitable : **10 % des Ã©vÃ©nements portent 63 % du notionnel**, donc le carnet est mÃ» par un petit nombre de gros ordres, ce qui borne la capacitÃ© du bot (Â§ 7).

## 0.4 Ã‰carts structurels

| Ã‰cart | Ã‰lÃ©ments | Amplitude mÃ©diane | FenÃªtre d'exploitabilitÃ© P90 | Ce qui le referme | Sources |
|---|---|---|---|---|---|
| **`bid_OUI + ask_NON âˆ’ 1`** | Deux cÃ´tÃ©s du mÃªme carnet | **0,000** â€” exact sur 1 025/1 025 lignes non manquantes | Sans objet : l'invariant ne s'ouvre pas | Moteur de cotation lui-mÃªme (contrainte mÃ©canique) | `[A2G]` rubrique 3 |
| `ask_OUI + bid_NON âˆ’ 1` | Idem | **0,000** sur 1 024/1 026 ; 2 lignes Ã  0,01 = **1 tick**, maximum 0,01 | Ã‰vÃ©nementielle, non mesurable | Idem | `[A2G]` |
| **`spot âˆ’ oracle` (basis)** | Spot + oracle | **+42,1 $** (mÃ©diane des mÃ©dianes : 41,83 / 41,998 / 42,148 / 42,269 / 42,464 / 43,227) ; positif sur **100 %** des lignes appariÃ©es (1943/1943, 3388/3388, 1190/1190, 682/682, 619/619) | **Ne se referme jamais** sur l'horizon d'une session | Aucune population : c'est un **dÃ©calage de source ou de dÃ©finition**, pas une inefficience | `[A1G]`+`[A2G]`, 12 sessions |
| `|p_ancrÃ© âˆ’ mid|` phase [30;60[ | Oracle + carnet | **0,0226** pt de probabilitÃ© | ND | Non identifiable | `[A2G]` 3 + 6 |
| `|p_ancrÃ© âˆ’ mid|` phase [60;180[ | Oracle + carnet | **0,1197** pt | ND | Non identifiable | `[A2G]` |
| `|p_ancrÃ© âˆ’ mid|` phase [180;240[ | Oracle + carnet | **0,0900** pt | ND | Non identifiable | `[A2G]` |
| `|p_ancrÃ© âˆ’ mid|` phase [240;270[ | Oracle + carnet | **0,4642** pt | ND | Non identifiable | `[A2G]` |
| **Spread du cÃ´tÃ© cotÃ©** | Meilleur niveau | **0,02** ; P90 0,04â€“0,05 ; plage observÃ©e **âˆ’0,01 â†’ 0,19** | Ã‰vÃ©nementielle | Fournisseurs de liquiditÃ©, non identifiables | `[A2G]`, 6 sessions |
| **Carnet encore opposÃ© Ã  une rÃ©solution dÃ©jÃ  dÃ©terminÃ©e** | Oracle + carnet | `Ï„ = 239,8 s` : `oracle âˆ’ K = +4,26 $` alors que `ask_OUI = 0,14` ; `Ï„ = 265,5 s` : `ask_OUI = 0,02`. Autre session : `ask = 0,48` pour un contrat que l'oracle donne Ã  `p_brut â‰ˆ 0,97` (`m* = +19 $ = 3,4 Ïƒ`) | Jusqu'Ã  la fin de session | Non identifiable | `[A1G]`+`[A2G]`, 4 sessions |

**Le spread nÃ©gatif est une anomalie Ã  coder.** Plage observÃ©e `âˆ’0,01` : le carnet est croisÃ©. `[A2G]` : Â« *deux spreads croisÃ©s passent Ã  tort* Â» le test T20 tel qu'Ã©crit (`spread â‰¤ 0,04`). Une borne basse est obligatoire (Â§ 5.20).

## 0.5 Chronologie d'une session

| Ã‰vÃ©nement | Instant (horloge de capture) | DÃ©lai depuis le prÃ©cÃ©dent | Ã‰lÃ©ment producteur | Population qui rÃ©agit en premier |
|---|---|---|---|---|
| PremiÃ¨re valeur oracle | `Ï„ = 0,000 s` | Origine des donnÃ©es | Oracle/payload | Non dÃ©terminable |
| PremiÃ¨re ligne de carnet | `Ï„ = 0,198 s` | +0,198 s | Carnet de session | Non dÃ©terminable ; le carnet ouvre Ã  `yes_bid 0,49 / yes_ask 0,50`, soit **un marchÃ© ouvert Ã  50/50** |
| Premier trade | `Ï„ = 0,207 s` | +0,009 s | Moteur | Non dÃ©terminable (ni prix ni cÃ´tÃ©) |
| PremiÃ¨re ligne spot | `Ï„ = 3,133 s` | +2,926 s | Spot | Non dÃ©terminable |
| **FenÃªtre Ïƒâ‚ƒâ‚€ pleine** | `Ï„ = 30 s` | +27 s | Oracle | Aucune : avant cet instant, **refus fail-closed**, jamais attente |
| Ouverture de la fenÃªtre d'entrÃ©e du PDF | `Ï„ = 60 s` | +30 s | RÃ¨gle PDF | 610 snapshots de carnet dans `[60;240]`, 417 hors fenÃªtre |
| **Fermeture de la fenÃªtre d'entrÃ©e du PDF** | `Ï„ = 240 s` | +180 s | RÃ¨gle PDF | â€” |
| **Verrouillage du signe de l'issue (`Ï„_lock`)** | **`Ï„ = 262 s`** mesurÃ© sur une session, soit **22 s aprÃ¨s la fermeture de la fenÃªtre d'entrÃ©e** | â€” | Oracle | **16 Ã  18 % du chemin de prix est postÃ© dans les 20 % finaux** |
| DÃ©but de la fenÃªtre de rÃ¨glement | `Ï„ = 270 s` | +8 s | Oracle | Le carnet cesse de publier vers `Ï„ = 273 s` dans plusieurs sessions |
| DerniÃ¨re valeur oracle disponible | `Ï„ = 298 s` | â€” | Oracle | Le point `[299;300]` manque systÃ©matiquement |
| RÃ©solution (TWAP `[270;300]`) | `Ï„ > 300 s` | â€” | Source officielle | Non observable |

**Le fait chronologique le plus important du corpus, citÃ© `[A1G]` :** Â« *La fenÃªtre `[60 ; 240]` ne contient pas l'information : sur S1 le signe se verrouille Ã  262 s, et 16â€“18 % du chemin de prix est postÃ© dans les 20 % finaux. Le postulat cherche Ã  dÃ©cider avant que l'issue existe.* Â»

## 0.6 Budget de latence du bot

| Ã‰tape | Latence | Source | Confiance |
|---|---|---|---|
| RÃ©ception de la donnÃ©e oracle | **Non dÃ©terminable.** Cadence 1 s connue ; latence de publication inconnue (`ts_src = payload`). Valeur nominale d'Ã‰tape 1 : 68 ms | `[E1]` O1 / reconstruit | Faible |
| RÃ©ception de la donnÃ©e carnet | **Non dÃ©terminable.** Cadence Ã©vÃ©nementielle mÃ©diane 0,013â€“0,015 s mesurÃ©e ; latence de transport inconnue | `[A2G]` / reconstruit | Faible |
| DÃ©cision (calcul de `Ïƒâ‚ƒâ‚€`, `Ïƒ_T`, `m*`, `p_ancrÃ©`, `Î´`) | `< 1 ms` â€” grandeurs dÃ©jÃ  en mÃ©moire | `[E1]` 1.4 : Â« *dÃ©jÃ  calculÃ© en mÃ©moire, moins d'une milliseconde* Â» | Ã‰levÃ©e |
| Envoi de l'ordre | **Non dÃ©terminable** | Aucune session | â€” |
| AccusÃ© du moteur | **Non dÃ©terminable** | Aucune session | â€” |
| Annulation | **Non dÃ©terminable** | Aucune session | â€” |
| **Budget total** | **Non dÃ©terminable.** Champs Ã  journaliser : `t_reception_oracle`, `t_reception_bbo`, `t_decision`, `t_envoi`, `t_ack`, `t_fill`, `t_annul_envoi`, `t_annul_ack`, horloge serveur, prÃ©cision â‰¤ 10 ms (Â§ 8.6) | `[R]` | â€” |

**Ce que l'absence de budget de latence interdit.** Aucun mÃ©canisme ne peut Ãªtre dÃ©clarÃ© Â« atteignable Â» au sens de la rÃ¨gle de travail 13. La seule comparaison possible est structurelle : la cadence du carnet est de **0,013 s** en mÃ©diane, celle de l'oracle de **1 s**. Un mÃ©canisme dÃ©clenchÃ© par une mise Ã  jour de l'oracle laisse donc environ **1 s** avant l'Ã©vÃ©nement oracle suivant, et le carnet peut bouger **~77 fois** dans cet intervalle. Tout mÃ©canisme dont la fenÃªtre est plus courte que 0,3 s est hors de portÃ©e : le PDF lui-mÃªme documente un krach d'ask de **0,279 s** invisible Ã  1 Hz.

## 0.7 SchÃ©ma des donnÃ©es disponibles (document 6 reconstruit)

| Champ | Ã‰lÃ©ment source | FrÃ©quence | Horloge | PrÃ©cision | Sessions oÃ¹ il est prÃ©sent |
|---|---|---|---|---|---|
| `bbo.t_ms` | Carnet | Ã‰vÃ©nementiel, mÃ©diane 0,013â€“0,015 s | Capture du carnet, **origine absolue inconnue** | 1 ms affichÃ©e ; latence source non sÃ©parable | 41 / 41 |
| `bbo.yes_bid` / `yes_ask` / `no_bid` / `no_ask` | Carnet | Idem | Capture du carnet | 1 ms | 41 / 41 ; manquants 0,10â€“0,20 % |
| `spot.t_ms` / `spot.price` | Spot | Ã‰vÃ©nementiel, mÃ©diane 0,38 s | Capture du spot | 1 ms ; publication source absente | 41 / 41 |
| `trades.t_ms` / `usd` / `direction` | Trades | Ã‰vÃ©nementiel, mÃ©diane 0,094 s | Capture du trade | 1 ms ; exÃ©cution source absente | 41 / 41 ; `direction` **sans dictionnaire** |
| `oracle.t_ms` / `oracle.price` | Oracle | 1 Hz nominal ; trou max 7 s | Capture du payload | 1 ms | 41 / 41 |
| `oracle.ts_src` | Oracle | 1 Hz | **ChaÃ®ne constante `payload`** | Aucune | 41 / 41 â€” **inutilisable** |
| `K`, `k_source` | Source officielle | 1Ã— Ã  Ï„=0 | â€” | â€” | **0 / 41 â€” Ã  vÃ©rifier (Â§ 8.6)** |
| `endDate` / origine `Ï„ = 0` | Source officielle | 1Ã— | â€” | â€” | **0 / 41 â€” Ã  vÃ©rifier** |
| Profondeur L2 (niveaux, quantitÃ©s, annulations) | Carnet | Devrait Ãªtre 4 Hz | â€” | â€” | **0 / 41 â€” Ã  vÃ©rifier** |
| `order_id`, `t_ack`, `t_fill`, prix d'exÃ©cution, quantitÃ© | Moteur | Ã‰vÃ©nementiel | Horloge serveur | â‰¤ 10 ms requis | **0 / 41 â€” Ã  vÃ©rifier** |
| `position`, `n_ordres_en_vol` | Ã‰tat interne du bot | Continu | Locale + rÃ©conciliation API | â€” | **0 / 41 â€” Ã  vÃ©rifier** ; dÃ©jÃ  identifiÃ© en `[E1]` 1.4 trou nÂ°4 |
| `issue_officielle`, TWAP `[270;300]` | RÃ©solution | 1Ã— | â€” | â€” | **0 / 41 dans les CSV** ; disponible par PDF de session pour une partie des agents |
| Drapeau `HALT` | Plateforme | Ã‰vÃ©nementiel | Horloge serveur | ms | **0 / 41 â€” Ã  vÃ©rifier** ; rend la moitiÃ© de T5 invÃ©rifiable |
| `fee_bps` rÃ©el | API CLOB | PrÃ©-vol + rotation | â€” | â€” | **0 / 41 â€” Ã  vÃ©rifier** |

**Onze champs requis sur vingt-quatre sont absents de la totalitÃ© du corpus.** C'est la contrainte dominante de toute l'Ã‰tape 4 : ce n'est pas une rÃ©serve de forme, c'est la raison pour laquelle **aucun PnL par sous-groupe n'est calculable au niveau d'un fill rÃ©el**, dans aucun rapport, pour aucune rÃ¨gle.

---

# 1. Inventaire des remarques des agents

| NÂ° | Agent / session | GÃ©n. | Citation textuelle | Sujet | Ã‰lÃ©ment Â§ 0 | Type | CatÃ©gorie |
|---|---|---|---|---|---|---|---|
| 1 | Agent quant. / A_1450 B_1455 C_1500 | 1 | Â« La stratÃ©gie de l'Ã‰tape 3 est un filtre stÃ©rilisant : elle ne prend aucune position. Â» | Dry run des 22 tests | Carnet, formule | Remarque | Structurelle |
| 2 | Agent quant. / A B C | 1 | Â« T14 et T15 ne se dÃ©clenchent jamais : le lag mesurÃ© est de âˆ’1s / 0s / 0s, jamais dans la plage [5 ; 60] qu'ils testent. Ce sont des tests morts sur ces donnÃ©es. Â» | T14, T15 | oracle â†’ carnet | Rejet | Structurelle |
| 3 | Agent quant. / A B C | 1 | Â« `CARNET_EN_AVANCE_EDGE_DISPARU` Ã©choue **181/181 secondes sur les trois sessions, soit 543/543**. C'est un veto absolu, et c'est lui qui explique Ã  lui seul le PnL nul. Â» | T15 | Carnet | Remarque | MÃ©canique |
| 4 | Agent quant. / C_1500 | 1 | Â« le couplage oracleâ†”carnet est **synchrone Ã  Â±2 s**, et sur C_1500 le carnet **prÃ©cÃ¨de** l'oracle de 2 s (`L = âˆ’2`, r = +0,424, la corrÃ©lation la plus forte de tout l'Ã©chantillon). Â» | Latence oracleâ†’carnet | oracle â†’ carnet | Remarque | Structurelle |
| 5 | Agent quant. / 3 sessions | 1 | Â« Ïƒâ‚ƒâ‚€ = 26,50 $ est surestimÃ© d'un facteur 6 Ã  15. Â» | Ïƒâ‚ƒâ‚€ | Oracle | Rejet | MÃ©canique |
| 6 | Agent quant. / A_1450 | 1 | Â« **Ce n'est pas un rÃ©glage trop prudent, c'est une erreur d'unitÃ©.** Â» | Ïƒâ‚ƒâ‚€ | Oracle | Remarque | MÃ©canique |
| 7 | Agent quant. / A B C | 1 | Â« Le coefficient c = 0,5513 est en revanche remarquablement bon. AjustÃ© indÃ©pendamment sur le mid du carnet de chaque session, on obtient c = 0,54 / 0,51 / 0,58 â€” le postulat tombe au milieu de l'intervalle mesurÃ©, avec des RMSE de 0,06 Ã  0,09. C'est le seul des trois postulats que les donnÃ©es confirment. Â» | `k_Î»` | ModÃ¨le | Acceptation | MÃ©canique |
| 8 | Agent dry-run / S1 S2 | 1 | Â« **k_Î» n'a pas d'optimum commun.** En S1 le Brier est minimisÃ© quand k_Î» â†’ 8 [...] En S2 l'optimum est k_Î» = 0,15 [...] **non identifiable** : 8,00 vs 0,15 (facteur 53) Â» | `k_Î»` | ModÃ¨le | Rejet | MÃ©canique |
| 9 | Agent quant. / 543 instants | 1 | Â« Aucune variable ne dÃ©passe RÂ² = 0,21 avec le gain. Les meilleures sont x60 (0,207), mid (0,189), ask (0,101). Il n'y a pas de prÃ©dicteur fort. Â» | Pouvoir prÃ©dictif | Toutes grandeurs | Remarque | Structurelle |
| 10 | Agent quant. / 543 instants | 1 | Â« Plusieurs variables sont quasi identiques et n'apportent aucune information indÃ©pendante : ratioâ†”p (RÂ² = 0,90), askâ†”N (RÂ² = 0,90), midâ†”ask (RÂ² = 0,96). Â» | Redondances | Formule | Remarque | Structurelle |
| 11 | Agent quant. / 3 sessions | 1 | Â« vous avez 543 instants mais seulement **3 issues indÃ©pendantes**. Les instants d'une mÃªme session sont autocorrÃ©lÃ©s Â» | Puissance statistique | MÃ©thode | Remarque | Structurelle |
| 12 | Agent quant. / 3 sessions | 1 | Â« un marqueur Ã  3/3 a un IC95 de Clopper-Pearson de [29,2 % ; 100 %]. Un taux rÃ©el de 30 % est statistiquement compatible avec vos donnÃ©es Â» | Puissance statistique | MÃ©thode | Remarque | Structurelle |
| 13 | Agent quant. / 3 sessions | 1 | Â« Les trois sessions Ã©tant toutes DOWN sous H3, un filtre qui parie systÃ©matiquement DOWN affiche un sans-faute sans rien prÃ©dire â€” et je ne peux pas distinguer M1 de ce cas-lÃ  avec les donnÃ©es prÃ©sentes. Â» | Non-directionnalitÃ© | MÃ©thode | Remarque | Directionnelle (avertissement) |
| 14 | Agent dry-run / 3 sessions | 1 | Â« La garde `basis â‰¤ 0` **ne peut jamais se dÃ©clencher** : le basis est structurellement dÃ©calÃ© de +40 $. La garde est aveugle Ã  un dÃ©calage 12 Ã  29 fois supÃ©rieur Ã  `M`. Â» | T5 / basis | spot â†” oracle | Rejet | Structurelle |
| 15 | Agent structurel / 190016 | 2 | Â« le miroir `spot<oracle` n'est pas traitÃ© par une action miroir ; 1943/1943 lignes seraient acceptÃ©es uniquement parce que le signe observÃ© est positif Â» | T5 | spot â†” oracle | Rejet | Directionnelle |
| 16 | Agent structurel / 190016 | 2 | Â« le BBO respecte presque exactement la complÃ©mentaritÃ© mÃ©canique `bid_OUI + ask_NON = 1` ; **cet invariant ne constitue pas un edge**. Â» | Invariant de cotation | Carnet | Remarque | Structurelle |
| 17 | Agent dry-run / A B | 1 | Â« **T17 est en contradiction mathÃ©matique avec la formule de p** : le garde-fou rejette le signal du modÃ¨le lui-mÃªme. Â» | T17 / `D_max` | ModÃ¨le â†” carnet | Rejet | MÃ©canique |
| 18 | Agent dry-run / S1 S2 | 1 | Â« **Les seuils de microstructure sont 10 Ã  20Ã— trop serrÃ©s.** T15 Ã  0,01 alors que le p95 mesurÃ© est 0,10â€“0,20 : le test refuse le marchÃ© normal. Â» | T15 / T13 | Carnet | Rejet | MÃ©canique |
| 19 | Agent dry-run / S1 | 1 | Â« sur S1 le signe se verrouille Ã  **262 s**, et 16â€“18 % du chemin de prix est postÃ© dans les 20 % finaux. Le postulat cherche Ã  dÃ©cider avant que l'issue existe. Â» | `Ï„_lock` / fenÃªtre | Oracle, chronologie | Remarque | Structurelle |
| 20 | Agent dry-run / A B | 1 | Â« **DÃ©bloquer les seuils rend le systÃ¨me perdant, pas gagnant** : sur 3 240 configurations, PnL total mÃ©dian âˆ’2,28 $ pour 10 $ engagÃ©s, 94 % de rÃ©glages perdants. Â» | Calibrage global | Formule | Remarque | Structurelle |
| 21 | Agent dry-run / S1 S2 | 1 | Â« **Le seul edge positif (+0,039 $) n'est pas un edge** : un trade, Ã  0,99, marge sous le tick, disparaÃ®t si l'on plafonne Ã  0,97. Il ne fait qu'acheter un rÃ©sultat dÃ©jÃ  connu, au prix du carnet. Â» | Convergence terminale | Carnet, oracle | Rejet | Structurelle |
| 22 | Agent dry-run / S3 | 1 | Â« l'edge disponible Ã©tait la **prime de retard du carnet sur une tendance dÃ©jÃ  rÃ©solue** : Ã  t = 90, le carnet demandait 0,48 pour un YES que l'oracle donnait Ã  `p_brut â‰ˆ 0,97` avec `m = +19 $ = 3,4 Ïƒ`. L'asymÃ©trie est donc de **~49 points de probabilitÃ©** Â» | Ã‰cart ancrage/carnet | Oracle â†” carnet | Recommandation | Structurelle |
| 23 | Agent dry-run / session B | 1 | Â« **+4,64 $ sur 5,00 $ engagÃ©s (+92,8 %) en 191 secondes de dÃ©tention** Â» | PnL atteignable | Carnet | Remarque | Structurelle |
| 24 | Agent quant. / A_1450 B_1455 | 1 | Â« **la bonne dÃ©cision sur A_1450 et B_1455 est de ne pas trader** ; le PnL nul du postulat y est accidentellement correct. Â» | Abstention | Formule | Acceptation | Structurelle |
| 25 | Agent structurel / 185016 185517 | 2 | Â« contact Î´=`0,02`: `85,20 %`; Î´=`0,05`: `70,92 %` Â» (phase milieu) | Taux de remplissage proxy | Carnet | Remarque | Comportementale |
| 26 | Agent structurel / 185517 | 2 | Â« La loi dÃ©gressive prudente est l'enveloppe monotone future par paliers : `Î´_guard=0,67669` pour `30â‰¤t<180`, `0,01574` pour `180â‰¤t<240`, `0,01069` pour `240â‰¤t<270`. Elle force zÃ©ro ordre avant 180 s car `Î´_guard>D_max=0,114` Â» | DÃ©cote dÃ©gressive | Oracle â†” carnet | Recommandation | MÃ©canique |
| 27 | Agent structurel / 190016 | 2 | Â« Aucune population ne peut donc Ãªtre nommÃ©e comme celle qui referme l'Ã©cart ; le faire serait une attribution inventÃ©e. Â» | Populations | Populations | Remarque | Comportementale |
| 28 | Agent dry-run / A B | 1 | Â« `sign(m*)` pointe le gagnant **78,8 %** du temps en agrÃ©gÃ©, mais en session B il pointe UP Ã  **0â€“13 %** de raison pendant **90 s consÃ©cutives**, avec une persistance de 45 s et X60 = 1 â€” les tests censÃ©s dÃ©tecter Ã§a le laissent passer intÃ©gralement. Â» | `sign(m*)` | Oracle, modÃ¨le | Rejet | Directionnelle |
| 29 | Agent dry-run / S1 | 1 | Â« **Le signal de valeur du postulat est ici anti-corrÃ©lÃ© au rÃ©sultat.** Â» (les 3 plus grosses sous-Ã©valuations dÃ©tectÃ©es sont toutes du mauvais cÃ´tÃ©) | `D` | ModÃ¨le â†” carnet | Rejet | Directionnelle |
| 30 | Agent structurel / 190016 | 2 | Â« l'agrÃ©gat passe par compensation, le sous-groupe bas Ã©choue ; **T19 Ã©choue au test non directionnel** Â» (Brier modÃ¨le 0,679 vs mid 0,554 sous K ; 0,019 vs 0,498 au-dessus) | T19 | ModÃ¨le | Rejet | Directionnelle |
| 31 | Agent quant. / grille 12 configs | 1 | Â« **toutes les configurations du top 12 ont `use_T14 = False`**. Activer T14 est incompatible avec la gÃ©nÃ©ration du moindre PnL. Â» | T14 | oracle â†’ carnet | Rejet | MÃ©canique |
| 32 | Agent structurel / 190016 | 2 | Â« aucun niveau 2, aucune quantitÃ©, aucun prix de niveau ; valeur non dÃ©terminable â†’ **refus fail-closed obligatoire** Â» | T21 profondeur | Profondeur | Remarque | MÃ©canique |
| 33 | Agent structurel / 185517 | 2 | Â« spread YES mÃ©dian 0,02, P90 0,05, **plage âˆ’0,01â€“0,19** ; deux spreads croisÃ©s passent Ã  tort Â» | T20 spread | Carnet | Rejet | MÃ©canique |
| 34 | Agents S1005 / S1010 | 1 | S1005 : Â« acheter Ã  `ask â‰¤ 0.45` Â» (142/142 gagnants) â€” S1010 : Â« ne jamais acheter sous 0.50 Â» (`ask < 0.50` â†’ **0/19**, E[PnL/part] = âˆ’0.42) | Plancher d'ask | Carnet | Rejet mutuel | Directionnelle |
| 35 | Agent S1010 | 1 | Â« **Arbitrage retenu : le plancher `ASK_MIN = 0.50` l'emporte, et il n'est pas nÃ©gociable.** [...] Le ratio perte rÃ©elle / manque Ã  gagner est de **1.75:1 contre l'assouplissement**. Â» | Plancher d'ask | Carnet | Recommandation | Comportementale |
| 36 | Agent dry-run / A B | 1 | Â« le **basis spot âˆ’ oracle** comme unique variable indÃ©pendante ayant une avance causale rÃ©elle (RÂ² = 0,23, r = +0,47 sur Î”oracle+30 s), employÃ©e en **rÃ©gression** et non en filtre binaire de signe. Â» | basis | spot â†” oracle | Recommandation | Directionnelle |
| 37 | Agent dry-run / S1 S2 | 1 | Â« 5 760 configurations rejouÃ©es, 2,7 % rentables, meilleure = +4,25 $ (**k_M = 0, aucune conviction**, entrÃ©e Ã  Ï„â‰ˆ57 s) [...] **Aucune configuration gardant un seuil de conviction actif n'est rentable.** Â» | `k_M` | ModÃ¨le | Remarque | Structurelle |
| 38 | Agent structurel / 4 sessions gen2 | 2 | Â« `T18 PRIX_MAX` : 235/480 ; 232/619 ; 528/682 ; **198/198 refus** selon session Â» | T18 | Carnet | Rejet | MÃ©canique |
| 39 | Agent structurel / 190016 | 2 | Â« `D_proxy` atteint `0,85744` et **croÃ®t en fin de session** ; traiter ce signe comme edge dÃ©pend de l'issue et de la calibration Â» | `D` | ModÃ¨le â†” carnet | Rejet | Directionnelle |
| 40 | `[PDF]` p. 11, relevÃ© par plusieurs agents | â€” | Â« Le facteur 0,07 est le frais dÃ©diÃ© au sizing ; le calcul du PnL rÃ©el utilise un invariant infÃ©rieur (0,02). Â» | Frais | CoÃ»ts | Acceptation | MÃ©canique |

---

# 2. Inter-confrontation des remarques

> RÃ¨gle appliquÃ©e : `ConfirmÃ©e` si n_confirmant â‰¥ 3 **et** n_confirmant / n_mesurÃ© â‰¥ 0,6 ; `Contredite` si n_contredisant â‰¥ 1 avec citation ; `IsolÃ©e` si n_mesurÃ© = 1. Le PnL par sous-groupe n'est jamais disponible au niveau d'un fill rÃ©el (Â§ 0.7) ; il est donc renseignÃ© Â« non dÃ©terminable Â» partout oÃ¹ il l'est, et la dÃ©cision est abaissÃ©e en consÃ©quence.

**Remarque 1 â€” la formule ne prend aucune position.**
- Autres agents : Â« *Le postulat ne dÃ©clenche aucun ordre, dans aucune configuration, sur aucune session. PnL = 0,00 $ sur 15,00 $. 900 secondes Ã©valuÃ©es.* Â» ; Â« *0 entrÃ©e sur 1 442 instants* Â» ; Â« *0 seconde sur 361 passe les 11 tests simultanÃ©ment* Â» ; Â« *RÃ©sultat du dry run intÃ©gral (22 tests) : 0 entrÃ©e. PnL = 0,00 $* Â» ; Â« *La formule d'achat de l'Ã‰tape 3, appliquÃ©e Ã  la lettre, produit 0 entrÃ©e sur 2 sessions* Â».
- Comptage : n_mesurÃ© **19** / n_confirmant **19** / n_contredisant **0** / n_muet 22.
- Ã‰tat : **ConfirmÃ©e** (19/19 = 1,00).
- UniversalitÃ© : tient sur **toutes** les couleurs dÃ©clarÃ©es (oracle dÃ©crochÃ©, oracle oscillant, range serrÃ©, forte volatilitÃ©, rÃ©solution serrÃ©e).
- Non-directionnalitÃ© : **Oui** â€” l'abstention est parfaitement miroir. PnL par sous-groupe : **0,00 $ dans les quatre sous-groupes**, seule valeur du corpus positive-ou-nulle partout.
- **DÃ©cision finale : Retenue.** Fait Ã©tabli, pas hypothÃ¨se.
- CritÃ¨re de dÃ©cision : c'est le point de dÃ©part obligatoire. Un PnL de 0 satisfait trivialement le critÃ¨re 1 mais Ã©choue au critÃ¨re 3 (zÃ©ro Ã©vÃ©nement captÃ©). L'Ã‰tape 4 doit donc trouver ce qui rend la formule exÃ©cutable **sans** la rendre perdante â€” et la remarque 20 montre que le desserrage naÃ¯f fait exactement le contraire.

**Remarque 2 et 4 â€” la latence oracle â†’ carnet n'existe pas.**
- Autres agents (citations) : Â« *Le pic est Ã  lag 0 dans les trois sessions ; tout le reste est du bruit (|r| < 0,12). Ã€ lag 16 s : r = 0,004 / 0,037 / 0,064.* Â» ; Â« *lag croisÃ© optimal oracleâ†’carnet = âˆ’1 s, r = 0,97 en session B. Le carnet est synchrone, voire en avance.* Â» ; Â« *Le carnet mÃ¨ne de 38â€“50 s ou est coÃ¯ncident (lag = âˆ’50 / âˆ’38 / 0 s).* Â» ; Â« *maximum Ã  L = 0 s (r = +0,559)* Â» ; Â« *pic global âˆ’1 s, RÂ² 0,03941* Â» ; Â« *Le retard signÃ© mesurÃ© est nul.* Â»
- Comptage : n_mesurÃ© **27** (sessions oÃ¹ l'argmax a Ã©tÃ© calculÃ©) / n_confirmant **27** / n_contredisant **0** / n_muet 14.
- Ã‰tat : **ConfirmÃ©e** â€” et **aucune** valeur dans la bande `[+5;+60]` du PDF : 0/27. `L â‰¤ 0` dans 26/27.
- UniversalitÃ© : tient sur toutes les couleurs, aux deux cadences (1 Hz et 4 Hz) et sur les deux estimateurs (incrÃ©ments et niveaux) â€” **horloges signalÃ©es sÃ©parÃ©ment (Â§ 0.2)**.
- Non-directionnalitÃ© : **Oui** â€” la corrÃ©lation croisÃ©e est symÃ©trique en signe ; le rÃ©sultat vaut pour les hausses comme pour les baisses d'oracle. PnL par sous-groupe : sans objet, c'est une mesure de structure, pas une rÃ¨gle.
- **DÃ©cision finale : Retenue.** Et par consÃ©quent : **T14 tel qu'Ã©crit est rejetÃ© (Â§ 5.14), et la justification Ã©crite de la stratÃ©gie change de nature (Â§ 4.6).**
- CritÃ¨re de dÃ©cision : critÃ¨re 1. Une rÃ¨gle conditionnÃ©e Ã  `lag âˆˆ [+5;+60]` capte 0 Ã©vÃ©nement sur 27 sessions ; son espÃ©rance est structurellement nulle, indÃ©pendamment du sens du sous-jacent.

**Remarque 3 â€” T15 est le veto dominant.**
- Autres agents : Â« *`CARNET_EN_AVANCE` (T14) : **721/721 refus** sur les deux sessions* Â» ; Â« *T15 (1Â¢/3 s) et T17 (D â‰¤ 0,114) vÃ©toient **82â€“86 %** des secondes* Â» ; Â« *intersection T15 : 0* Â» ; Â« *T15 proxy 29/682*, *35/480*, *45/619* Â».
- Comptage : n_mesurÃ© **11** / n_confirmant **11** / n_contredisant **0** / n_muet 30.
- Ã‰tat : **ConfirmÃ©e**.
- UniversalitÃ© : oui, toutes couleurs.
- Non-directionnalitÃ© : **Oui** â€” un veto refuse symÃ©triquement. PnL par sous-groupe : non dÃ©terminable (aucune entrÃ©e Ã  Ã©valuer).
- **DÃ©cision finale : Retenue avec condition** â€” condition mesurable : `saut_ask_3s` doit Ãªtre re-seuillÃ© au **quantile mesurÃ© en ligne** et non Ã  la constante 0,01 (Â§ 5.15).
- CritÃ¨re de dÃ©cision : critÃ¨re 3. Le seuil 0,01 capte 0 Ã©vÃ©nement ; le p95 mesurÃ© est de 0,10â€“0,20, soit **10 Ã  20 fois** plus grand.

**Remarques 5 et 6 â€” Ïƒâ‚ƒâ‚€ = 26,50 $ est faux.**
- Citations concordantes : Â« *Ïƒâ‚ƒâ‚€ mesurÃ© mÃ©dian **11,48 $** [...] **2,31Ã—*** Â» ; Â« *Ïƒ_30 rÃ©el : **8,56 / 9,97 / 10,75 $** (mÃ©dianes)* Â» ; Â« *Ïƒâ‚ƒâ‚€ rÃ©elle = **7,97 $** / 30 s (mÃ©diane des 3 sessions)* Â» ; Â« *mÃ©d. **9,94** [p10 6,73 â€“ p90 13,96]* Â» et Â« *mÃ©d. **7,51** [5,35 â€“ 10,99]* Â» ; Â« *min 5,65 / **mÃ©d 9,26** / moy 10,25 / max 19,06 $ [...] **surestimÃ© Ã—2,9*** Â» ; Â« *`Ïƒ_30` glissant mesurÃ©, mÃ©diane **5,44 $** [...] facteur **4,87Ã—*** Â» ; Â« *sigma rolling : P10 0,77361, mÃ©diane **0,91210**, P90 2,38254 USD* Â».
- Comptage : n_mesurÃ© **21** (estimateur incrÃ©ments) + **8** (estimateur niveaux) / n_confirmant **29** / n_contredisant **0** / n_muet 12.
- Ã‰tat : **ConfirmÃ©e**, universelle.
- **Deux estimateurs, deux valeurs â€” Ã  ne jamais mÃ©langer (rÃ¨gle de consensus 5) :**
  - **Estimateur A** â€” Ã©cart-type des incrÃ©ments oracle Ã  1 s, remis Ã  l'Ã©chelle 30 s : mÃ©diane des mÃ©dianes **7,20 $**, P10 2,64 $, P90 9,94 $, min 1,52 $, max 11,48 $, n = 21. Facteur d'erreur du PDF : **Ã—3,68**.
  - **Estimateur B** â€” Ã©cart-type des niveaux d'oracle sur la fenÃªtre de 30 s : mÃ©diane **2,25 $**, n = 8. Facteur d'erreur du PDF : **Ã—11,8**.
- **Estimateur que le bot doit utiliser : A.** Raison : `Ïƒ_T` doit mesurer *le mouvement que l'oracle peut encore faire*, c'est-Ã -dire une somme d'incrÃ©ments, dont la variance s'additionne â€” c'est exactement la loi en âˆš que le PDF dÃ©montre en page 15. L'estimateur B mesure la dispersion d'un niveau autour de sa moyenne, ce qui n'est pas la mÃªme grandeur et sous-estime le risque d'un facteur ~3.
- Non-directionnalitÃ© : **Oui** â€” un Ã©cart-type est insensible au signe. PnL par sous-groupe : non applicable.
- **DÃ©cision finale : Retenue.** `Ïƒâ‚ƒâ‚€` est **mesurÃ© en ligne, jamais gelÃ©**. Refus fail-closed si la fenÃªtre de 30 s n'est pas pleine ou si `Ïƒâ‚ƒâ‚€ âˆ‰ [0,5 ; 40] $`.
- CritÃ¨re de dÃ©cision : critÃ¨re 1. Le gel Ã  26,50 $ produit 0 entrÃ©e sur la totalitÃ© du corpus (Â« *le postulat avec la constante Ïƒâ‚ƒâ‚€ = 26,50 $ ne trade **jamais*** Â»), donc espÃ©rance nulle.

**Remarques 7 et 8 â€” `k_Î» = 0,5513` : confirmÃ©e comme identitÃ©, contredite comme calibrateur.**
- Confirmation : Â« *c = 0,54 / 0,51 / 0,58 â€” le postulat tombe au milieu de l'intervalle mesurÃ©, avec des RMSE de 0,06 Ã  0,09* Â» ; Â« *k_Î» (Ã©chelle Î») 0,5513 = âˆš3/Ï€ â†’ **0,5513 â€” inchangÃ©, identitÃ© confirmÃ©e*** Â» ; Â« *`k_Î» = 0,5513` n'a Ã©tÃ© touchÃ© par aucune configuration gagnante* Â».
- Contradiction : Â« *En S1 le Brier est minimisÃ© quand k_Î» â†’ 8 [...] En S2 l'optimum est k_Î» = 0,15 [...] facteur 53, paramÃ¨tre **non identifiable*** Â».
- Comptage : n_mesurÃ© **7** / n_confirmant **5** / n_contredisant **2** / n_muet 34. Ratio 5/7 = **0,71 â‰¥ 0,6**.
- Ã‰tat : **ConfirmÃ©e ET Contredite** â€” les deux Ã©tats coexistent, comme le prÃ©voit la rÃ¨gle 2.
- **Arbitrage `[R]` :** les deux mesures ne portent pas sur la mÃªme chose et ne se contredisent donc pas rÃ©ellement. `0,5513 = âˆš3/Ï€` est **une identitÃ© algÃ©brique** : c'est le coefficient qui Ã©galise la variance d'une logistique d'Ã©chelle Î» Ã  celle d'une gaussienne d'Ã©cart-type Ïƒ_T. Elle est confirmÃ©e par l'ajustement direct sur le mid (0,51â€“0,58). Le balayage de Brier, lui, ne mesure pas Î» : il mesure **si `p` est une probabilitÃ©**. Son optimum Ã  8,00 signifie Â« Ã©craser `p` vers 0,5 Â», c'est-Ã -dire *le modÃ¨le n'a aucune information* â€” et son optimum Ã  0,15 signifie l'inverse. Le facteur 53 n'infirme pas l'identitÃ© : il dÃ©montre que **`p` n'est pas calibrÃ©**.
- Non-directionnalitÃ© : identitÃ© **Oui** ; `p` comme probabilitÃ© **Non** (remarque 30 : Brier meilleur au-dessus du strike, pire en dessous).
- **DÃ©cision finale :** `k_Î» = 0,5513` **Retenue** comme identitÃ© de conversion `Î» = 0,5513Â·Ïƒ_T`. `p` **Retenue avec condition** â€” condition mesurable : `p` reste **un score de diagnostic, jamais une probabilitÃ©**, et n'entre dans aucune condition d'entrÃ©e tant que les coefficients de Platt `(a,b)` ne sont pas ajustÃ©s sur â‰¥ 50 sessions Ã©tiquetÃ©es avec un Brier positif **dans chaque sous-groupe**. ConsÃ©quence directe : **`D = p âˆ’ ask` sort de la porte d'entrÃ©e** (Â§ 5.16, 5.17).

**Remarques 9, 10, 11, 12 â€” il n'existe aucun prÃ©dicteur, et l'Ã©chantillon ne peut pas en rÃ©vÃ©ler un.**
- Concordance : Â« *Aucune variable ne franchit le seuil RÂ² > 0,5 exigÃ©* Â» ; Â« *`m*` â†” mid YES **+0,951**, RÂ² 0,904 : le carnet est une fonction quasi affine de l'avance de l'oracle : **aucune information privÃ©e dans `m*`*** Â» ; Â« *`RÂ²(p_brut, ask) = 0,956`, corrÃ©lation croisÃ©e maximale Ã  lag â‰ˆ 0 : il est donc, comme nous, **aveugle*** Â».
- Comptage : n_mesurÃ© **9** / n_confirmant **9** / n_contredisant **0** / n_muet 32.
- Ã‰tat : **ConfirmÃ©e**.
- Non-directionnalitÃ© : **Oui** â€” l'absence de prÃ©dicteur est symÃ©trique.
- **DÃ©cision finale : Retenue.** C'est la justification quantitative de tout `<ce_que_nous_exploitons>` : le corpus **dÃ©montre** qu'aucune rÃ¨gle prÃ©dictive ne survivra sur 500 sessions. RÂ² max = 0,207 et le carnet rÃ©plique `m*` Ã  RÂ² = 0,904.
- CritÃ¨re de dÃ©cision : critÃ¨re 1. Une rÃ¨gle bÃ¢tie sur RÂ² â‰¤ 0,21 a une espÃ©rance qui dÃ©pend du sens du sous-jacent par construction.

**Remarque 13 â€” l'avertissement de non-directionnalitÃ©, Ã  conserver comme garde permanente.**
- Comptage : n_mesurÃ© 4 / n_confirmant 4 / n_contredisant 0 / n_muet 37. Ã‰tat : **ConfirmÃ©e**.
- **DÃ©cision finale : Retenue** comme rÃ¨gle de mÃ©thode, pas comme rÃ¨gle de trading. Elle impose le format de la table 8.4 bis et le critÃ¨re de retrait de la table 8.8.
- CritÃ¨re : c'est la forme opÃ©rationnelle du test de non-directionnalitÃ© elle-mÃªme.

**Remarques 14 et 15 â€” le basis `spot âˆ’ oracle` est un dÃ©calage de source, pas un signal, et T5 est un pari.**
- Concordance des amplitudes : `+41,83 $` / `+41,998 $` / `+42,148 $` / `+42,269 $` / `+42,464 $` / `+43,227 $` ; positif sur **100 %** des lignes appariÃ©es dans 5 sessions instrumentÃ©es (1943/1943, 3388/3388, 1190/1190, 682/682, 619/619).
- Comptage : n_mesurÃ© **12** / n_confirmant **12** / n_contredisant **0** / n_muet 29. Ã‰tat : **ConfirmÃ©e**.
- UniversalitÃ© : toutes couleurs. Amplitude mÃ©diane de consensus **+42,1 $**, plage observÃ©e 22,90 â†’ 53,32 $.
- Non-directionnalitÃ© de T5 : **Non** â€” Â« *le miroir `spot<oracle` n'est pas traitÃ© par une action miroir* Â». C'est un test unilatÃ©ral toujours vrai : il n'a jamais rien filtrÃ©.
- **DÃ©cision finale : T5 RejetÃ©e (directionnelle et inerte), inscrite en 8.5. Requalification Retenue :** contrÃ´le de cohÃ©rence de source `|(spot âˆ’ oracle) âˆ’ basis_mÃ©dian_glissant| â‰¤ 3Â·Ïƒ(basis)`, fail-closed. C'est un garde-fou d'intÃ©gritÃ©, jamais une entrÃ©e.
- CritÃ¨re : critÃ¨re 1 et critÃ¨re 4 (fausses entrÃ©es). Un test vrai 1943/1943 fois n'apporte aucune information et masque un dÃ©calage 12 Ã  29 fois supÃ©rieur Ã  `M`.

**Remarque 16 â€” l'invariant de complÃ©mentaritÃ© est exact et n'est pas un edge.**
- Mesure : `|bid_OUI + ask_NON âˆ’ 1| = 0` sur **1 025/1 025** lignes non manquantes ; `|ask_OUI + bid_NON âˆ’ 1| = 0` sur 1 024/1 026, Ã©cart maximal **0,01 = 1 tick**.
- Comptage : n_mesurÃ© **6** / n_confirmant **6** / n_contredisant **0** / n_muet 35. Ã‰tat : **ConfirmÃ©e**.
- Non-directionnalitÃ© : **Oui** â€” l'invariant tient quel que soit le cÃ´tÃ© balayÃ© ; l'action serait miroir.
- **DÃ©cision finale : Retenue, mais requalifiÃ©e.** Deux emplois lÃ©gitimes, aucun n'est une entrÃ©e : (a) **contrÃ´le qualitÃ© fail-closed** si l'Ã©cart dÃ©passe 1 tick, ce qui signale une donnÃ©e corrompue ou un cÃ´tÃ© balayÃ© non encore rÃ©percutÃ© ; (b) **dÃ©rivation du prix du cÃ´tÃ© NON** depuis le cÃ´tÃ© OUI sans second flux, ce qui divise par deux la surface d'erreur d'implÃ©mentation.
- CritÃ¨re : critÃ¨re 1 satisfait trivialement (aucune position). Ne satisfait pas le critÃ¨re 3 comme source d'edge : l'Ã©cart mesurÃ© est nul, il n'y a rien Ã  arbitrer.

**Remarques 17, 18, 38 â€” trois seuils du PDF sont hors de la rÃ©alitÃ© du carnet.**
- T17 (`D â‰¤ 0,114`) : Â« *`D>0,114` dans **371/610** snapshots* Â» soit 60,8 % ; Â« *D mÃ©dian +0,224* Â» ; Â« *T17 est en contradiction mathÃ©matique avec la formule de p* Â».
- T15 (saut d'ask â‰¤ 0,01 / 3 s) : p95 mesurÃ© **0,10â€“0,20**, p99 0,13â€“0,29, max 0,30. Pas mÃ©dian rÃ©el du carnet : **2 cents**.
- T18 (`P_max`) : refus **198/198**, **387/619**, **154/682**, **245/480**, **124** et **84** selon session.
- Comptage : n_mesurÃ© **13** / n_confirmant **13** / n_contredisant **0** / n_muet 28. Ã‰tat : **ConfirmÃ©e**.
- Non-directionnalitÃ© : **Oui** pour les trois (vetos symÃ©triques).
- **DÃ©cision finale : Retenues avec condition.** Condition mesurable, identique pour les trois : **remplacer chaque constante absolue par un quantile mesurÃ© en ligne sur la session en cours**, avec la fenÃªtre et le quantile fixÃ©s en Â§ 6. `D_max` est conservÃ© uniquement comme **coupe-circuit**, jamais comme condition d'entrÃ©e, puisque `p` n'est pas calibrÃ©.
- CritÃ¨re : critÃ¨re 3 puis 4. Ces trois seuils captent 0 Ã©vÃ©nement ; leurs quantiles mesurÃ©s en captent sans introduire de dÃ©pendance au sens.

**Remarque 19 â€” l'issue se verrouille aprÃ¨s la fermeture de la fenÃªtre d'entrÃ©e.**
- Concordance : Â« *le signe ne se verrouille qu'Ã  **262 s** â€” 22 s aprÃ¨s la fermeture* Â» ; Â« *Ã  Ï„ = 240 s l'oracle vaut 64 817,64 $ (m* = +4,64 $, donc Â« UP Â») pendant que le carnet cote NO Ã  0,20 $ â€” puis l'oracle s'effondre de 22 $ en 15 secondes et l'issue est DOWN* Â» ; Â« *abandonner la contrainte de fermeture Ã  240 s qui exclut par construction l'intervalle oÃ¹ l'issue se dÃ©cide* Â» ; mouvement rÃ©siduel de l'oracle mesurÃ© par phase : **10,76 $** en `[30;60[`, **11,62 $** en `[60;180[`, **1,50 $** en `[180;240[`, **1,55 $** en `[240;270[`.
- Comptage : n_mesurÃ© **8** / n_confirmant **8** / n_contredisant **0** / n_muet 33. Ã‰tat : **ConfirmÃ©e**.
- UniversalitÃ© : oui. C'est une propriÃ©tÃ© du **rÃ¨glement en TWAP `[270;300]`**, donc de la plateforme, pas d'une couleur de session.
- Non-directionnalitÃ© : **Oui** â€” l'affirmation porte sur *quand* l'information existe, pas sur *quel* cÃ´tÃ© gagne. Le mouvement rÃ©siduel est une valeur absolue.
- **DÃ©cision finale : Retenue.** ConsÃ©quence majeure : **la fenÃªtre d'action doit Ãªtre dÃ©placÃ©e de `[60;240]` vers `[180;270]`**, et c'est le seul dÃ©placement que trois familles de mesures indÃ©pendantes rÃ©clament simultanÃ©ment (rÃ©siduel d'oracle, `Ï„_lock`, `Î´_guard` de la remarque 26).
- CritÃ¨re : critÃ¨re 1. Sur `[60;180[`, le mouvement rÃ©siduel mÃ©dian de l'oracle est de **11,62 $**, soit 1,6 Ïƒâ‚ƒâ‚€ : Ã  cet instant, toute position est exposÃ©e au sens du sous-jacent. Sur `[180;240[`, il tombe Ã  **1,50 $**, soit 0,21 Ïƒâ‚ƒâ‚€. C'est exactement la condition du critÃ¨re 1 : l'exposition au sens dÃ©croÃ®t mÃ©caniquement avec le temps restant.

**Remarque 20 â€” desserrer les seuils sans traiter la thÃ¨se rend le bot perdant.**
- Concordance : Â« *216 configurations testÃ©es, **0 positive*** Â» ; Â« *3 240 configurations, PnL total mÃ©dian **âˆ’2,28 $** pour 10 $ engagÃ©s, **94 %** de rÃ©glages perdants* Â» ; Â« *5 760 configurations rejouÃ©es, **2,7 %** rentables* Â» ; Â« *la meilleure configuration trouvÃ©e sur une grille de **3 888 combinaisons** prend exactement une entrÃ©e* Â» ; Â« *864 configurations* Â» ; Â« *120 configurations* Â».
- Comptage : n_mesurÃ© **8** / n_confirmant **8** / n_contredisant **0** / n_muet 33. Ã‰tat : **ConfirmÃ©e**.
- Non-directionnalitÃ© : **Oui** â€” le constat porte sur la distribution des rÃ©glages, pas sur un cÃ´tÃ©.
- **DÃ©cision finale : Retenue.** C'est l'argument dÃ©cisif contre tout recalibrage numÃ©rique des 22 tests : ~13 800 configurations balayÃ©es, aucune famille rentable et robuste. La correction ne peut donc pas Ãªtre paramÃ©trique ; elle doit Ãªtre **structurelle** â€” changer ce que la dÃ©cote rÃ©munÃ¨re et quand le bot agit.
- CritÃ¨re : critÃ¨re 1, appliquÃ© Ã  la population des rÃ©glages plutÃ´t qu'Ã  un rÃ©glage.

**Remarques 21, 22, 23 â€” l'Ã©cart ancrage/carnet en fin de session : rÃ©el, mesurÃ©, et Ã©conomiquement mince.**
- Positif : Â« *Ã  t = 90, le carnet demandait 0,48 pour un YES que l'oracle donnait Ã  `p_brut â‰ˆ 0,97`* [...] asymÃ©trie de **~49 points de probabilitÃ©** Â» ; Â« *+4,64 $ sur 5,00 $ engagÃ©s (+92,8 %)* Â» ; Â« *Ï„ = 239,755 s, oracle âˆ’ K = +4,26 $ et YES ask = 0,14 ; Ã  Ï„ = 265,454 s, YES ask = 0,02* Â» ; Ã©cart mÃ©dian `|p_ancrÃ© âˆ’ mid|` = **0,4642 pt** en `[240;270[`.
- Contredisant : Â« *Le seul edge positif (+0,039 $) n'est pas un edge : un trade, Ã  0,99, marge sous le tick, disparaÃ®t si l'on plafonne Ã  0,97.* Â» ; Â« *S1 entre Ã  224 s (z = 3,25, p = 0,9994, marge annoncÃ©e +0,2160) et **perd 4,70 $***. Â»
- Comptage : n_mesurÃ© **9** / n_confirmant **6** / n_contredisant **3** / n_muet 32. Ratio 6/9 = **0,67 â‰¥ 0,6**.
- Ã‰tat : **ConfirmÃ©e ET Contredite**.
- UniversalitÃ© par couleur : **non universelle**. L'Ã©cart est large et rÃ©munÃ©rateur quand l'oracle est **dÃ©crochÃ©** du strike (session C_1500 : `D > frais` sur 139/241 s = 57,7 %), et il est un piÃ¨ge quand l'oracle **oscille autour** du strike (A_1450 : 9/241 = 3,7 %, dÃ©cote mÃ©diane **âˆ’0,0611** ; B_1455 : 29/241 = 12,0 %, mÃ©diane âˆ’0,0547).
- Non-directionnalitÃ© : **Oui sous condition d'amplitude** â€” l'Ã©cart doit Ãªtre pris en valeur absolue, avec le cÃ´tÃ© dÃ©duit par miroir du signe de `oracle âˆ’ K`. La version signÃ©e serait un pari, et c'est exactement ce que montre le contre-exemple Ã  224 s.
- PnL par sous-groupe : **non dÃ©terminable** au niveau du fill. Au niveau du signal : positif dans le sous-groupe Â« oracle dÃ©crochÃ© Â», nÃ©gatif dans Â« oracle oscillant Â».
- **DÃ©cision finale : Retenue avec condition.** Condition mesurable **avant** d'agir, calculable en temps rÃ©el : `z(Ï„) = |oracle(Ï„) âˆ’ K| / Ïƒ_T(Ï„) â‰¥ z_min` **et** `Xâ‚†â‚€(Ï„) = 0` **et** `Ï„ â‰¥ 180 s`. Les trois marqueurs identifient la couleur Â« oracle dÃ©crochÃ© Â» sans connaÃ®tre l'issue. C'est le mÃ©canisme **M1** du Â§ 4.6 et la seule rÃ¨gle d'action du livrable.
- CritÃ¨re de dÃ©cision : critÃ¨re 1 d'abord â€” c'est `Ï„ â‰¥ 180 s` qui rend la condition non directionnelle, parce que le mouvement rÃ©siduel de l'oracle y tombe Ã  1,50 $ (0,21 Ïƒâ‚ƒâ‚€) ; puis critÃ¨re 3 â€” l'Ã©cart mÃ©dian y est de 0,09 Ã  0,46 pt de probabilitÃ©, largement au-dessus des frais de 0,017â€“0,027.

**Remarque 24 â€” l'abstention est parfois la bonne rÃ©ponse.**
- Comptage : n_mesurÃ© 5 / n_confirmant 5 / n_contredisant 0 / n_muet 36. Ã‰tat : **ConfirmÃ©e**.
- Non-directionnalitÃ© : **Oui**. PnL par sous-groupe : 0,00 $ partout.
- **DÃ©cision finale : Retenue.** Le bot doit avoir un Ã©tat Â« inactif Â» explicite et journalisÃ©, et ne pas le vivre comme un Ã©chec. Deux des trois sessions du premier rapport, et **la totalitÃ©** des sessions Ã  oracle oscillant, sont des sessions Ã  ne pas trader.
- CritÃ¨re : critÃ¨re 1 puis 4.

**Remarque 25 et 26 â€” le taux de contact d'une quote passive et la loi de dÃ©cote dÃ©gressive.**
- Mesures agrÃ©gÃ©es sur les rapports de seconde gÃ©nÃ©ration, contact du BBO dans les 2 s suivant la cotation :
  - Î´ = 0,02 : ouverture **71,5 %** (42,96 / 100,0) Â· milieu **88,5 %** (85,20 / 91,76) Â· derniÃ¨res minutes **48,6 %** (0,0 / 97,28).
  - Î´ = 0,05 : ouverture **70,1 %** Â· milieu **76,7 %** Â· derniÃ¨res minutes **43,2 %**.
- `Î´_guard = frais_AR + Q90(|Î”p_ancrÃ©| sur 2 s)`, points mesurÃ©s : `Ï„â‰ˆ45 s` â†’ **0,2006 pt** (0,30079 / 0,10048) ; `Ï„â‰ˆ120 s` â†’ **0,3474 pt** (0,01803 / 0,67669) ; `Ï„â‰ˆ180â€“210 s` â†’ **0,0132 pt** (0,01069 / 0,01574) ; `Ï„â‰ˆ255 s` â†’ **0,0107 pt**.
- Comptage : n_mesurÃ© **4** / n_confirmant **4** / n_contredisant **0** / n_muet 37. Ã‰tat : **ConfirmÃ©e** au sens de la rÃ¨gle 2 (n_confirmant â‰¥ 3 et ratio 1,00), mais **dispersion inter-sessions Ã©norme en phase milieu** : 0,018 contre 0,677, soit un facteur 37.
- Non-directionnalitÃ© : **Oui** â€” `Q90(|Î”p_ancrÃ©|)` est une valeur absolue ; le contact est mesurÃ© du cÃ´tÃ© ancrÃ©, quel qu'il soit.
- **DÃ©cision finale : Retenue avec condition.** La forme retenue est **l'enveloppe monotone future par paliers**, jamais la mÃ©diane par phase â€” parce que la mÃ©diane de la phase milieu est instable d'un facteur 37 et qu'une dÃ©cote sous-estimÃ©e est une perte rÃ©elle, tandis qu'une dÃ©cote surestimÃ©e n'est qu'un manque Ã  gagner (mÃªme asymÃ©trie 1,75:1 que la remarque 35). Paliers retenus en 8.4 bis.
- CritÃ¨re : critÃ¨re 1. Le palier `[30;180[` vaut **0,677 pt**, trÃ¨s au-dessus du coupe-circuit `D_max = 0,114` : il **force zÃ©ro ordre avant 180 s**, et c'est prÃ©cisÃ©ment ce qui met la rÃ¨gle en conformitÃ© avec le critÃ¨re 1. La loi de dÃ©gressivitÃ© n'est donc pas un rÃ©glage de rentabilitÃ©, c'est le mÃ©canisme qui rend la stratÃ©gie non directionnelle.

**Remarque 27 â€” la contrepartie n'est pas nommable.**
- Comptage : n_mesurÃ© **6** / n_confirmant **6** / n_contredisant **0** / n_muet 35. Ã‰tat : **ConfirmÃ©e**.
- **DÃ©cision finale : Retenue.** ConsÃ©quence sur la rÃ¨gle de travail 11 : chaque mÃ©canisme du Â§ 7 nomme la **classe de signature** la plus probable et dÃ©clare explicitement l'attribution non prouvÃ©e. Le seul fait de structure exploitable est la concentration : **10,2 % des Ã©vÃ©nements portent 63,2 % du notionnel**.
- CritÃ¨re : critÃ¨re 1 par prudence. Une contrepartie inventÃ©e conduirait Ã  surdimensionner la capacitÃ©.

**Remarques 28, 29, 30, 34, 36, 39 â€” les six remarques directionnelles.**
- Comptage collectif : n_mesurÃ© **14** / n_confirmant **11** / n_contredisant **3** / n_muet 27.
- Test de non-directionnalitÃ© : **Non** pour les six. `sign(m*)` (28) est le signe du mouvement ; `D` signÃ© (29, 39) change de camp avec l'issue ; T19/Brier (30) rÃ©ussit au-dessus du strike et Ã©choue en dessous ; le plancher d'ask (34) est contredit frontalement entre deux sessions (`ask â‰¤ 0,45` â†’ 142/142 gagnants contre `ask < 0,50` â†’ 0/19) ; le basis en rÃ©gression sur `Î”oracle+30 s` (36) prÃ©dit un mouvement futur.
- **DÃ©cision finale : RejetÃ©es, inscrites en 8.5**, avec requalification structurelle pour chacune (Â§ 8.5). Application de la rÃ¨gle de consensus 7 : la remarque 36 est rejetÃ©e **mÃªme si elle est la seule variable Ã  avance causale mesurÃ©e** (r = +0,47, RÂ² = 0,23), parce qu'elle prÃ©dit la direction. Le basis reste utilisÃ©, mais **uniquement** comme contrÃ´le de cohÃ©rence de source (remarque 14).
- CritÃ¨re : critÃ¨re 1, sans appel.

**Remarque 31 â€” aucune configuration gagnante n'active T14.**
- Comptage : n_mesurÃ© 3 / n_confirmant 3 / n_contredisant 0 / n_muet 38. Ã‰tat : **ConfirmÃ©e**.
- **DÃ©cision finale : Retenue.** Confirme la remarque 2 par une voie indÃ©pendante (optimisation plutÃ´t que corrÃ©lation).

**Remarque 32 â€” la profondeur est fail-closed par absence de donnÃ©es.**
- Comptage : n_mesurÃ© **6** / n_confirmant **6** / n_contredisant **0** / n_muet 35. Ã‰tat : **ConfirmÃ©e**. DÃ©jÃ  identifiÃ© en `[E1]` 1.4 comme Â« trou nÂ°2 Â».
- **DÃ©cision finale : Retenue.** T21 reste dans la formule, **fail-closed**, et devient le premier verrou de dÃ©ploiement : sans niveau 2, le bot ne peut pas dimensionner sans risquer de refermer l'Ã©cart lui-mÃªme.

**Remarque 33 â€” T20 doit avoir une borne basse.**
- Comptage : n_mesurÃ© **4** / n_confirmant **4** / n_contredisant **0** / n_muet 37. Ã‰tat : **ConfirmÃ©e** (plage observÃ©e `âˆ’0,01 â†’ 0,19`).
- Non-directionnalitÃ© : **Oui**.
- **DÃ©cision finale : Retenue.** `spread âˆˆ [0 ; quantile_mesurÃ©]`, fail-closed hors bornes. Un spread nÃ©gatif est un carnet croisÃ©, donc une donnÃ©e Ã  rejeter, pas une aubaine.

**Remarque 35 â€” l'arbitrage asymÃ©trique perte rÃ©elle / manque Ã  gagner.**
- Comptage : n_mesurÃ© 2 / n_confirmant 1 / n_contredisant 1 / n_muet 39. Ã‰tat : **IsolÃ©e et Contredite**.
- Non-directionnalitÃ© : la rÃ¨gle `ASK_MIN` elle-mÃªme est **Non** (elle sÃ©lectionne un cÃ´tÃ© du prix, donc un camp implicite). Mais **le principe d'arbitrage** est structurel.
- **DÃ©cision finale : la rÃ¨gle `ASK_MIN = 0,50` est RejetÃ©e (directionnelle, Â§ 8.5). Le principe d'arbitrage est Retenu** comme rÃ¨gle de dÃ©partage universelle du livrable : **Ã  Ã©galitÃ© d'information, choisir la branche dont la perte rÃ©elle est la plus faible, et non celle dont le manque Ã  gagner est le plus faible** (ratio mesurÃ© 1,75:1). C'est ce principe qui fixe l'enveloppe monotone de la remarque 26 et le sens du critÃ¨re 4.

**Remarque 37 â€” aucune configuration gardant un seuil de conviction actif n'est rentable.**
- Comptage : n_mesurÃ© 3 / n_confirmant 2 / n_contredisant 1 / n_muet 38. Ã‰tat : **IsolÃ©e renforcÃ©e / Contredite**.
- Contredisant : les agents qui mesurent Â« *sachant `|m*| â‰¥ 0,3439Â·Ïƒ_T` â‡’ **100 %** (n = 174) ; sachant `|m*| â‰¥ 1,0Â·Ïƒ_T` â‡’ **100 %** (n = 118)* Â» â€” mais sur 3 issues indÃ©pendantes toutes DOWN, donc **non concluant** au sens de la remarque 13.
- **DÃ©cision finale : Retenue avec condition.** Le seuil de conviction `z â‰¥ z_min` est conservÃ© **non pas comme prÃ©dicteur de l'issue** mais comme **marqueur de la couleur de session** (Â« oracle dÃ©crochÃ© Â» vs Â« oracle oscillant Â»), ce qui est son seul emploi non directionnel. Condition : `z_min` doit Ãªtre validÃ© sur les 500 sessions avec PnL positif dans chaque sous-groupe, Ã  dÃ©faut ramenÃ© Ã  0 (aucune conviction) â€” ce qui est exactement la configuration que cet agent trouve rentable.

**Remarque 40 â€” l'incohÃ©rence des frais.**
- Ã‰tat : **ConfirmÃ©e** (constat de lecture du PDF, non contestÃ©).
- **DÃ©cision finale : Retenue.** `frais_AR_sizing = 0,07Â·askÂ·(1âˆ’ask) + 0,01` pour dimensionner (conservateur), `frais_AR_pnl = 0,02Â·askÂ·(1âˆ’ask)` pour le PnL rÃ©el. L'Ã©cart est volontaire et doit rester : il constitue la marge de sÃ©curitÃ© du dimensionnement. Valeur mesurÃ©e de `frais_AR` : mÃ©diane **0,0179 Ã  0,0231** pt, plage 0,0107 â†’ 0,0275.

---

# 3. Consolidation avec les Ã©tapes 2 et 3

> Document 3 (`conclusions_etapes_2_et_3.md`) livrÃ© vide. La colonne Â« moment de signification Ã©tape 3 Â» est donc **reconstruite** depuis le PDF (pages 14 Ã  17, Â« dÃ©cisions closes Â») et les documents d'Ã‰tape 1 fournis. Toute reconstruction est marquÃ©e.

| Valeur ou terme | Moment de signification Ã©tape 3 | Moment de signification retenu Ã©tape 4 | Moment de connaissance (latence, horloge) | Changement | Raison |
|---|---|---|---|---|---|
| Origine `Ï„ = 0` | Lue 1Ã— Ã  l'ouverture dans `endDate âˆ’ 300 s`, jamais `time()` (p. 14, reconstruit) | **InchangÃ©**, et rendu **bloquant** : sans `endDate`, refus fail-closed de toute la session | Ã€ l'ouverture, horloge marchÃ©. Latence API non mesurÃ©e | Non (durci) | 0/41 sessions contiennent le champ ; un `Ï„` faux invalide `Ïƒ_T`, `M`, `Î»`, `P_max` et la fenÃªtre simultanÃ©ment |
| `K` (strike) | Lu 1Ã— Ã  `Ï„ = 0`, source officielle, `k_source = officiel` (p. 4) | **InchangÃ©**, bloquant | Ã€ l'ouverture, horloge marchÃ© | Non (durci) | 0/41 sessions ; les agents mesurent des Ã©carts strike infÃ©rÃ© vs ajustÃ© de **+0,25 / +12,25 / âˆ’4,50 $** â€” et sur une session l'Ã©cart de 12,25 $ **inverse le rÃ¨glement** |
| `Ïƒâ‚ƒâ‚€` | `Ï„ â‰¥ 30 s` puis Ã  chaque seconde, remesurÃ© fenÃªtre par fenÃªtre (p. 15) | **InchangÃ© sur l'instant**, mais **estimateur fixÃ©** : Ã©cart-type des incrÃ©ments d'oracle Ã  1 s Ã— âˆš30 (estimateur A, Â§ 2) | `Ï„ â‰¥ 30 s` + latence de rÃ©ception oracle (non mesurÃ©e, cadence 1 s) | **Oui â€” estimateur** | Deux estimateurs coexistaient dans les rapports avec un facteur 3 entre eux ; le PDF ne tranchait pas |
| Valeur de `Ïƒâ‚ƒâ‚€` | 26,50 $ Â« milieu de la plage rÃ©alisÃ©e 25â€“28 $ Â» (p. 15) | **7,20 $ mÃ©diane des mÃ©dianes, mais jamais codÃ©e en constante** : mesurÃ©e en ligne, refus si `âˆ‰ [0,5 ; 40] $` | Idem | **Oui â€” valeur** | Facteur d'erreur Ã—3,68 ; la valeur gelÃ©e produit 0 entrÃ©e sur la totalitÃ© du corpus |
| `Ïƒ_T(Ï„) = Ïƒâ‚ƒâ‚€Â·âˆš((300âˆ’Ï„)/30)` | Ã€ chaque seconde, horloge `Ï„`Â·marchÃ© (p. 15) | **InchangÃ©.** Loi vÃ©rifiÃ©e, seule Ã©chelle du systÃ¨me | `Ï„ â‰¥ 30 s` | Non | La loi en racine est confirmÃ©e ; seule son entrÃ©e `Ïƒâ‚ƒâ‚€` Ã©tait fausse |
| `Î» = 0,5513Â·Ïƒ_T` | IdentitÃ©, plus un rÃ©glage (p. 16) | **InchangÃ© comme identitÃ©** | AprÃ¨s `Ïƒâ‚ƒâ‚€` | Non | Ajustement direct sur le mid : 0,54 / 0,51 / 0,58 |
| `M = 0,3439Â·Ïƒ_T` | Ã‰quivalent Ã  `p_brut â‰¥ 0,6511`, seuil constant en probabilitÃ© (p. 16) | **RequalifiÃ©** : `z(Ï„) = |m*|/Ïƒ_T â‰¥ z_min` â€” mÃªme grandeur, exprimÃ©e sans passer par `p` | AprÃ¨s `Ïƒâ‚ƒâ‚€` | **Oui â€” rÃ´le** | `p` n'Ã©tant pas calibrÃ©, exprimer le seuil en probabilitÃ© donne une fausse prÃ©cision ; `z` est la forme mesurable |
| `p_brut`, `p` (Platt) | `p` est un score jusqu'Ã  validation `Brier(p) â‰¤ Brier(carnet)` (p. 9, p. 16) | **InchangÃ©, et le verdict est tombÃ© : la validation Ã©choue.** `p` reste un score de diagnostic journalisÃ©, hors de la porte d'entrÃ©e | AprÃ¨s `Ïƒâ‚ƒâ‚€` et `m*` | **Oui â€” statut tranchÃ©** | Brier par sous-groupe : modÃ¨le 0,679 vs mid 0,554 **sous** le strike, 0,019 vs 0,498 **au-dessus** : l'agrÃ©gat ne passe que par compensation |
| `D = p âˆ’ ask` | Lue **2 s aprÃ¨s son apparition**, sans saut d'ask > 1 cent sur 3 s (p. 17) | **Sortie de la porte d'entrÃ©e.** RemplacÃ©e par l'Ã©cart mesurable `e(Ï„) = |p_ancrÃ©,cÃ´tÃ©(Ï„) âˆ’ ask_cÃ´tÃ©(Ï„)|` oÃ¹ `p_ancrÃ©` est dÃ©duit de `z` et non de Platt | Ã€ chaque mise Ã  jour du carnet, horloge carnet 4 Hz min | **Oui â€” dÃ©finition** | `D` hÃ©rite de la non-calibration de `p` ; mesurÃ© signÃ©, il est anti-corrÃ©lÃ© au rÃ©sultat sur une session |
| `D_max = 0,114` | Plafond `PROVISOIRE` jusqu'Ã  Ã©talonnage (p. 9) | **Coupe-circuit seulement**, jamais condition d'entrÃ©e | Idem | **Oui â€” rÃ´le** | DÃ©passÃ© dans 371/610 snapshots (60,8 %) ; Â« *contradiction mathÃ©matique avec la formule de p* Â» |
| `lag_signÃ©` | CorrÃ©lation croisÃ©e sur 10 sessions, bande `[+5 ; +60] s`, mÃ©diane annoncÃ©e 16,91 s, plage 3â€“146 s (p. 17) | **RÃ©futÃ©. RetirÃ© de la porte d'entrÃ©e.** MÃ©diane de consensus **âˆ’0,75 s**, P10 âˆ’28,25 s, P90 0 s, 0/27 dans la bande | Non dÃ©terminable (pas d'horloge source) | **Oui â€” rÃ©futation** | 27 sessions mesurantes, 0 confirmante de la bande, 26/27 avec `L â‰¤ 0` |
| FenÃªtre d'entrÃ©e | `Ï„ âˆˆ [60 ; 240]`, borne basse alignÃ©e sur la fenÃªtre X (p. 6) | **`Ï„ âˆˆ [180 ; 270]`** | Horloge `Ï„`Â·marchÃ© | **Oui â€” dÃ©placement majeur** | Trois mesures indÃ©pendantes concordent : mouvement rÃ©siduel de l'oracle 11,62 $ â†’ 1,50 $ Ã  180 s ; `Ï„_lock = 262 s` ; `Î´_guard` 0,677 â†’ 0,0157 pt Ã  180 s |
| `w(Ï„)` bascule TWAP | RÃ©servÃ©e au STOP/VERROU, `240 â‰¤ Ï„ < 270`, inerte Ã  l'achat (p. 4 ; `[E1]` 1.5 Â§ 4.2) | **Devient active** puisque la fenÃªtre d'action est dÃ©placÃ©e dans `[180 ; 270]` | Horloge `Ï„` | **Oui â€” rÃ©activation** | ConsÃ©quence mÃ©canique du dÃ©placement de fenÃªtre ; le rÃ©fÃ©rentiel de rÃ¨glement est le TWAP `[270;300]` |
| `Xâ‚†â‚€(Ï„)` | Compteur sur 60 s, seuil `< 3`, fenÃªtre complÃ¨te exigÃ©e (p. 7) | **InchangÃ© sur la dÃ©finition, durci sur le seuil : `Xâ‚†â‚€ = 0`** dans la fenÃªtre d'action | `Ï„ â‰¥ 60 s`, horloge `Ï„` | **Oui â€” seuil** | `Xâ‚†â‚€ = 0` mesurÃ© Ã  100 % de rÃ©ussite (n = 335) contre 99,5 % pour `Xâ‚†â‚€ < 3` ; et `Xâ‚†â‚€ = 0` est le marqueur d'Â« oracle dÃ©crochÃ© Â» |
| `mÃ©dâ‚ƒâ‚›(ask)` et `ASK_MEDIAN_DELTA = 0,03` | FenÃªtre 3 s, seuil 3 ticks = 1/6 du krach documentÃ© (`[E1]` K23) | **Seuil remplacÃ© par un quantile mesurÃ© en ligne** | Horloge carnet, 4 Hz min | **Oui â€” valeur** | Chute p95 mesurÃ©e 0,08â€“0,15, p99 0,13â€“0,29 : le seuil 0,03 refuse la respiration normale du carnet |
| Spread, `FCO_SPREAD_MAX = 0,04` | Verrou, sentinelle `âˆ’1` si indisponible (`[E1]` 1.3.bis Â§ 3 : *fail-open* accidentel) | **`spread âˆˆ [0 ; q_mesurÃ©]`, boolÃ©en de disponibilitÃ©, fail-closed** | Horloge carnet | **Oui â€” bornes et dÃ©gradation** | Plage observÃ©e `âˆ’0,01 â†’ 0,19` ; deux spreads croisÃ©s passent Ã  tort |
| `PENTE_DIVISEUR` | TranchÃ© en `[E1]` 1.5 : ensemble E8, en **secondes**, plage 20â€“40 s | **Sans objet** : la condition de pente est rejetÃ©e comme directionnelle (Â§ 5.12) | â€” | **Oui â€” suppression** | Une borne unilatÃ©rale sur la pente sÃ©lectionne une trajectoire |
| `Î¦â‚ƒâ‚€` flux adverse | Veto, jamais dÃ©clencheur, seuil 800 $, *fail-open* assumÃ© (`[E1]` O9) | **Non dÃ©terminable.** `direction` est un code Â±1 **sans dictionnaire** : le cÃ´tÃ© agresseur n'est pas reconstructible | â€” | **Oui â€” indÃ©terminÃ©** | Â« *Marqueur de flux Î¦â‚ƒâ‚€ : inutilisable* Â». Le champ `side` est Ã  journaliser (Â§ 8.6) |
| `profondeur` (12 niveaux) | MesurÃ©e, journalisÃ©e, consommÃ©e par personne (`[E1]` 1.4 trou nÂ°2) ; activÃ©e en C15/T21 Ã  l'Ã‰tape 3 | **Fail-closed, bloquant.** Aucun niveau 2 dans le corpus | â€” | Non (durci) | 0/41 sessions |
| `position`, `n_ordres_en_vol` | AjoutÃ©s Ã  l'inventaire en `[E1]` 1.4 trou nÂ°4 ; T22 exige inventaire vide rÃ©conciliÃ© | **Fail-closed, bloquant** | Ã‰tat local + rÃ©conciliation API | Non (durci) | 0/41 sessions ; Â« *une position fantÃ´me non rÃ©conciliÃ©e vaut refus* Â» |
| `C = 5,00 $`, `PARTS_MIN = 5` | `N = C/(ask + 0,07Â·ask(1âˆ’ask) + 0,01)`, `N â‰¥ 5`, d'oÃ¹ `ask â‰¤ 0,98926` (p. 11) | **InchangÃ©**, avec la consÃ©quence explicitÃ©e : dans la fenÃªtre `[180;270]`, les asks du cÃ´tÃ© ancrÃ© sont hauts, donc `N` est petit et le PnL par Ã©vÃ©nement est de l'ordre de **1 Ã  2 ticks Ã— N** | Ã€ la dÃ©cision | Non | Contrainte arithmÃ©tique, pas un choix. La branche demi-taille reste morte (`[E1]` 1.6 Â§ 4.3) |
| Frais | `0,07Â·ask(1âˆ’ask) + 0,01` au sizing, invariant rÃ©el `0,02` au PnL (p. 11) | **InchangÃ©**, Ã©cart assumÃ© comme marge de sÃ©curitÃ© | PrÃ©-vol + Ã  chaque rotation | Non | `frais_AR` mesurÃ© : mÃ©diane 0,0179â€“0,0231 pt, plage 0,0107â€“0,0275 |
| Cadence de lecture | 1 Hz oracle / 4 Hz carnet minimum (p. 16) | **InchangÃ©, et rendu obligatoire** : 4 Hz au moins sur le carnet, dÃ©clenchement par **l'union** des Ã©vÃ©nements oracle et carnet | Horloge carnet | Non (durci) | Cadence de carnet mesurÃ©e Ã  **0,013â€“0,015 s** en mÃ©diane : Ã  1 Hz, 98 % des mises Ã  jour sont invisibles. `[E1]` 1.3.bis Â§ 4.1 le demandait dÃ©jÃ  |
| Tick | `0,01` | **InchangÃ©**, mais notÃ© : le tick est **infÃ©rÃ©** des valeurs du BBO, ce n'est pas un champ de schÃ©ma | â€” | Non (rÃ©serve) | Ã€ confirmer par l'API |

**Trois conclusions antÃ©rieures sont modifiÃ©es, et voici pourquoi.**
1. **La fenÃªtre d'entrÃ©e passe de `[60;240]` Ã  `[180;270]`.** C'est le changement le plus lourd. Il n'est pas motivÃ© par un gain de PnL mais par le critÃ¨re 1 : avant 180 s, le mouvement rÃ©siduel mÃ©dian de l'oracle est de 11,62 $, soit **1,6 Ïƒâ‚ƒâ‚€**, ce qui expose mÃ©caniquement toute position au sens du sous-jacent. AprÃ¨s 180 s, il tombe Ã  1,50 $, soit **0,21 Ïƒâ‚ƒâ‚€**.
2. **`D` et `p` sortent de la porte d'entrÃ©e.** L'Ã‰tape 3 les gardait Â« provisoires sous garde T19 Â». La garde a Ã©tÃ© testÃ©e : elle Ã©choue par sous-groupe. Les conserver comme conditions d'entrÃ©e revient Ã  faire dÃ©cider un score non calibrÃ©.
3. **`lag_signÃ©` est retirÃ©, pas recalibrÃ©.** Plusieurs agents proposaient d'Ã©largir la bande Ã  `[âˆ’2;+60]` ou `[0;+60]` pour dÃ©bloquer la formule. C'est refusÃ© : Ã©largir une bande jusqu'Ã  ce qu'elle contienne la mesure n'est pas un calibrage, c'est une tautologie. Le lag reste **journalisÃ©** comme indicateur de santÃ© (Â§ 8.6).

---

# 4. Relations globales entre les 22 fonctions indicatrices

## 4.1 Grandeurs partagÃ©es et relations structurelles

**Relation 1 â€” l'invariant de complÃ©mentaritÃ©.**
Grandeurs : `bid_OUI`, `ask_NON`, `ask_OUI`, `bid_NON`.
Relation : `bid_OUI + ask_NON = 1` et `ask_OUI + bid_NON = 1`. MesurÃ© exact sur **1 025/1 025** lignes non manquantes ; second invariant exact sur 1 024/1 026, Ã©cart maximal **0,01 = 1 tick**. Si `ask_OUI = 0,62` alors `bid_NON = 0,38`, nÃ©cessairement.
Ce que cela rÃ©vÃ¨le : les deux cÃ´tÃ©s ne sont pas deux marchÃ©s, c'est **un seul marchÃ© transformÃ©**. Le moteur de cotation impose la contrainte ; elle ne s'ouvre pas.
Marque comportementale : un Ã©cart de 1 tick apparaÃ®t sur 2 lignes sur 1 026 â€” c'est-Ã -dire au moment oÃ¹ un cÃ´tÃ© vient d'Ãªtre balayÃ© et oÃ¹ l'autre n'a pas encore Ã©tÃ© recalculÃ©.
Test de non-directionnalitÃ© : **Oui** â€” l'invariant tient quel que soit le cÃ´tÃ© balayÃ©, l'action serait miroir.
Contrepartie : ordres passifs du cÃ´tÃ© non encore recalculÃ©. **Attribution non prouvÃ©e** : sans `order_id` ni `maker/taker`, la population n'est pas identifiable.
ConsÃ©quence pour le bot : mesurer `|bid_OUI + ask_NON âˆ’ 1|` Ã  chaque mise Ã  jour. Un Ã©cart > 1 tick est un **refus fail-closed** (donnÃ©e corrompue), pas un signal. Emploi utile : dÃ©river le prix du cÃ´tÃ© NON du cÃ´tÃ© OUI, ce qui supprime une source d'erreur d'implÃ©mentation.

**Relation 2 â€” le basis spot/oracle est une constante de source.**
Grandeurs : `spot(Ï„)`, `oracle(Ï„)`, `basis(Ï„) = spot(Ï„) âˆ’ oracle(Ï„)`.
Relation : `basis(Ï„) â‰ˆ +42,1 $`, positif sur **100 %** des lignes appariÃ©es de 5 sessions instrumentÃ©es, plage inter-session 22,90 â†’ 53,32 $, dispersion intra-session P10â€“P90 typique 39,4 â†’ 47,7 $.
Ce que cela rÃ©vÃ¨le : ce n'est **pas** une avance informationnelle du spot sur l'oracle. Un dÃ©calage permanent de 42 $, soit 5,8 Ïƒâ‚ƒâ‚€, qui ne se referme jamais, n'est pas une inefficience : c'est une diffÃ©rence de source ou de dÃ©finition (le spot est un prix instantanÃ© Binance, l'oracle un TWAP 30 s Chainlink, potentiellement sur une autre composition d'Ã©changes).
Marque comportementale : aucune population ne le referme, sur aucune session, Ã  aucun instant.
Test de non-directionnalitÃ© : la **mesure** est Oui ; le **test T5 `spot > oracle`** est Non, parce que le miroir `spot < oracle` n'est pas traitÃ© par une action miroir.
Contrepartie : aucune. Il n'y a pas d'Ã©cart Ã  fermer.
ConsÃ©quence pour le bot : `basis` devient un **contrÃ´le de cohÃ©rence de source**. `|basis(Ï„) âˆ’ mÃ©diane_glissante(basis)| â‰¤ 3Â·Ïƒ(basis)` sur la session, sinon fail-closed : un basis qui s'Ã©carte brutalement signale qu'un des deux flux a dÃ©crochÃ©.

**Relation 3 â€” la loi du temps : le risque dÃ©croÃ®t en racine, et c'est la seule relation vraiment prÃ©visible du systÃ¨me.**
Grandeurs : `Ï„`, `Ïƒâ‚ƒâ‚€`, `Ïƒ_T(Ï„)`, `M(Ï„)`, `Î»(Ï„)`.
Relation : `Ïƒ_T(Ï„) = Ïƒâ‚ƒâ‚€Â·âˆš((300 âˆ’ Ï„)/30)`. Avec le `Ïƒâ‚ƒâ‚€` de consensus de 7,20 $ : `Ïƒ_T(60) = 20,36 $`, `Ïƒ_T(180) = 14,40 $`, `Ïƒ_T(240) = 10,18 $`, `Ïƒ_T(270) = 7,20 $`, `Ïƒ_T(290) = 4,16 $`. De `Ï„ = 0` Ã  `Ï„ = 240`, division exacte par **âˆš5 = 2,236**.
Ce que cela rÃ©vÃ¨le : c'est la seule grandeur du systÃ¨me **entiÃ¨rement connue Ã  l'avance**. Elle ne dÃ©pend d'aucune prÃ©vision, d'aucun flux, d'aucune population : uniquement de l'heure et d'une mesure de dispersion. Le mouvement rÃ©siduel de l'oracle mesurÃ© par les agents suit cette loi : **10,76 $** en `[30;60[`, **11,62 $** en `[60;180[`, **1,50 $** en `[180;240[`, **1,55 $** en `[240;270[`.
Marque comportementale : le carnet, lui, ne suit pas cette loi aussi vite. L'Ã©cart mÃ©dian `|p_ancrÃ© âˆ’ mid|` passe de **0,0226** pt en `[30;60[` Ã  **0,1197** pt en `[60;180[`, **0,0900** pt en `[180;240[` et **0,4642** pt en `[240;270[`.
Test de non-directionnalitÃ© : **Oui** â€” `Ïƒ_T` est un Ã©cart-type, insensible au signe. Le mouvement rÃ©siduel est mesurÃ© en valeur absolue. La relation vaut Ã  la hausse comme Ã  la baisse.
Contrepartie : les participants qui continuent de coter une incertitude que le temps a dÃ©jÃ  supprimÃ©e. **Classe de signature probable : fournisseurs de liquiditÃ© passive** dont les quotes ne sont pas re-cotÃ©es Ã  la cadence de dÃ©croissance de `Ïƒ_T`. Attribution non prouvÃ©e â€” champ `order_id`/`maker-taker` Ã  journaliser.
ConsÃ©quence pour le bot : **c'est le remplaÃ§ant de la latence oracle â†’ carnet.** La dÃ©cote ne rÃ©munÃ¨re pas un retard de communication, qui est nul. Elle rÃ©munÃ¨re `Ïƒ_T(Ï„)`, c'est-Ã -dire le mouvement que l'oracle peut **encore** faire. Et comme `Ïƒ_T` dÃ©croÃ®t de faÃ§on dÃ©terministe, la dÃ©cote dÃ©croÃ®t de faÃ§on dÃ©terministe. **La dÃ©gressivitÃ© n'est pas une hypothÃ¨se : c'est la consÃ©quence arithmÃ©tique de la loi du temps.**

**Relation 4 â€” la conviction normalisÃ©e `z` est une couleur de session, pas une prÃ©vision.**
Grandeurs : `m*(Ï„) = oracle(Ï„) âˆ’ K`, `Ïƒ_T(Ï„)`, `z(Ï„) = |m*(Ï„)|/Ïƒ_T(Ï„)`.
Relation : `z â‰¥ z_min âŸº |m*| â‰¥ z_minÂ·Ïƒ_T`. Le test T9 du PDF est le cas `z_min = 0,3439`.
Ce que cela rÃ©vÃ¨le : `z` ne dit pas dans quel sens l'oracle va. Il dit **combien d'Ã©carts-types de mouvement rÃ©siduel sÃ©parent l'oracle du strike**. Ã€ `z = 3`, il faudrait un mouvement Ã  3 Ïƒ pour changer l'issue ; Ã  `z = 0,3`, un mouvement banal suffit. C'est une mesure de **dÃ©termination mÃ©canique**, pas de direction.
Marque comportementale : quand `z` est grand **et** que `Xâ‚†â‚€ = 0`, la session est Â« oracle dÃ©crochÃ© Â» et l'Ã©cart carnet/ancrage est large et persistant (une session : `D > frais` sur **139/241 s = 57,7 %**). Quand `z` est petit et que l'oracle traverse le strike plusieurs fois (une session : **8 traversÃ©es**), la session est Â« oracle oscillant Â» et l'Ã©cart mÃ©dian est **nÃ©gatif** (âˆ’0,0611 et âˆ’0,0547).
Test de non-directionnalitÃ© : **Oui**, et c'est structurel : `z` est construit sur une valeur absolue, le cÃ´tÃ© Ã©tant dÃ©duit par miroir du signe de `m*`. Si le sous-jacent avait fait le mouvement miroir, `z` serait identique et l'action serait sur l'autre cÃ´tÃ© au prix complÃ©mentaire (relation 1).
Contrepartie : idem relation 3.
ConsÃ©quence pour le bot : `z` est **le marqueur de couleur** qui autorise ou interdit l'action, jamais le dÃ©clencheur du cÃ´tÃ©. Le cÃ´tÃ© vient du signe de `m*`, qui est une transformation symÃ©trique, pas un pari â€” Ã  condition que `Ï„ â‰¥ 180 s`, car c'est le temps restant qui rend le signe fiable, pas le signe lui-mÃªme.

**Relation 5 â€” les redondances internes de la formule.**
Grandeurs : `ratio`, `p`, `ask`, `N`, `mid`, `m*`.
Relation : `RÂ²(ratio, p) = 0,90` Â· `RÂ²(ask, N) = 0,90` Â· `RÂ²(mid, ask) = 0,96` Â· `RÂ²(m*, mid_OUI) = 0,904` avec `r = +0,951`.
Ce que cela rÃ©vÃ¨le : la formule Ã  22 tests contient beaucoup moins de 22 dimensions indÃ©pendantes. Et surtout, `RÂ²(m*, mid_OUI) = 0,904` signifie que **le carnet est dÃ©jÃ  une fonction quasi affine de l'avance de l'oracle** : il n'y a **aucune information privÃ©e dans `m*`**. C'est la mesure directe qui tue l'idÃ©e d'un retard informationnel.
Test de non-directionnalitÃ© : **Oui** â€” une corrÃ©lation entre deux grandeurs est indÃ©pendante du sens.
Contrepartie : sans objet.
ConsÃ©quence pour le bot : ne pas coder 22 conditions comme si elles Ã©taient 22 informations. Six dimensions indÃ©pendantes suffisent : **validitÃ© de la mesure**, **temps restant**, **dÃ©termination `z`**, **Ã©cart carnet/ancrage**, **qualitÃ© de carnet**, **Ã©tat interne**.

## 4.2 Fonctions qui s'activent ensemble

Co-activations mesurÃ©es sur une session de seconde gÃ©nÃ©ration, 610 snapshots dans `[60;240]` :

| Conjonction | Activations | Taux |
|---|---|---|
| T1 âˆ§ T2 âˆ§ T3 âˆ§ T4 âˆ§ T8 (validitÃ© + fenÃªtre) | 610 / 610 | 100 % |
| T9 âˆ§ T16 | 502 / 610 | 82,3 % |
| T9 âˆ§ T16 âˆ§ T17 | 135 / 610 | 22,1 % |
| T9 âˆ§ T16 âˆ§ T17 âˆ§ T18 | 83 / 610 | 13,6 % |
| T9 âˆ§ T16 âˆ§ T17 âˆ§ T18 âˆ§ T20 | 60 / 610 | 9,8 % |
| + T11 âˆ§ T12 âˆ§ T13 | 31 / 610 | 5,1 % |
| T13 âˆ§ T15 | 126 / 610 et 252 / 909 selon session | â€” |
| **Conjonction complÃ¨te des 22** | **0 / 610** | **0 %** |

Le bloc A (validitÃ©, T1â€“T4) s'active **en bloc** : ces quatre tests sont fortement corrÃ©lÃ©s parce qu'ils mesurent tous la santÃ© du mÃªme flux. Le bloc E (T16â€“T19) s'effondre en bloc dÃ¨s que `p` est mal calibrÃ©, parce que les quatre tests consomment `p`.

## 4.3 Fonctions qui s'excluent

| Paire | Nature de l'exclusion | Preuve mesurÃ©e |
|---|---|---|
| **T14 âˆ§ (toute entrÃ©e)** | Exclusion totale | `lag âˆˆ [+5;+60]` : 0/27 sessions. 721/721 et 543/543 refus. Â« *toutes les configurations du top 12 ont `use_T14 = False`* Â» |
| **T15 âˆ§ T16** | Exclusion quasi totale | `D > frais` confirmÃ© â‰¥ 2 s : 5 snapshots ; absence de saut d'ask : 161 ; **intersection : 0** |
| **T16 âˆ§ T17** | Exclusion mathÃ©matique | `Î» = 0,5513Â·Ïƒ_T` sature `p` Ã  une mÃ©diane de **0,9998**, donc `D` mÃ©dian **+0,224** ; T17 exige `D â‰¤ 0,114`. Â« *le garde-fou rejette le signal du modÃ¨le lui-mÃªme* Â» |
| **T17 âˆ§ (`z` Ã©levÃ©)** | Exclusion par construction | Plus `z` monte, plus `p â†’ 1`, plus `D` monte, plus T17 refuse. **T17 refuse exactement les instants les plus dÃ©terminÃ©s** |
| **T18 âˆ§ (fenÃªtre tardive)** | Exclusion croissante | `P_max(Ï„) = min(0,70 + 0,00075Ï„ ; 0,92)` monte lentement ; l'ask du cÃ´tÃ© ancrÃ© monte vite en fin de session. Refus 198/198, 387/619 |
| **T10 âˆ§ T11** | Exclusion partielle | T10 exige 10 s cumulÃ©es de mÃªme signe ; T11 compte les changements de signe. Un oracle oscillant Ã©choue aux deux, un oracle dÃ©crochÃ© passe aux deux : les deux tests mesurent **la mÃªme chose** |
| **T5 âˆ§ (miroir)** | Exclusion logique | `spot > oracle` vrai 1943/1943 : le test n'exclut rien, mais il **interdit structurellement** le cas miroir |

**Le plus important : T17 exclut le mÃ©canisme mÃªme que la stratÃ©gie veut exploiter.** Plus l'oracle est dÃ©crochÃ© du strike, plus l'ancrage est Ã©loignÃ© du carnet, plus `D` est grand, et plus T17 refuse. Le PDF le justifie page 9 : Â« *une dÃ©cote large ne signale pas une aubaine : elle signale que le carnet sait quelque chose que l'oracle n'a pas encore imprimÃ©* Â». Les mesures disent le contraire : le carnet **rÃ©plique** l'oracle Ã  `RÂ² = 0,904`, il ne sait rien de plus. `D_max` protÃ¨ge donc contre un danger inexistant en fermant la porte au mÃ©canisme rÃ©el.

## 4.4 Ordre temporel d'activation

| Instant | Ce qui devient disponible ou actif |
|---|---|
| `Ï„ = 0` | `endDate`, `K`, `k_source` lus **une fois**. Si absents â†’ session refusÃ©e (T6, T7) |
| `Ï„ = 0 â†’ 30` | Aucune dÃ©cision calculable. `Ïƒâ‚ƒâ‚€` indisponible â†’ refus fail-closed (T4). Le carnet ouvre typiquement Ã  `0,49 / 0,50` |
| `Ï„ = 30` | `Ïƒâ‚ƒâ‚€`, `Ïƒ_T`, `Î»`, `M`, `z` deviennent calculables. **T4 s'active** |
| `Ï„ = 60` | `Xâ‚†â‚€` a un historique complet. **T11 devient valide** (avant, la fenÃªtre dÃ©borde et le compteur est sous-estimÃ© â€” anomalie relevÃ©e en `[E1]` 1.3.bis Â§ 4.2) |
| `Ï„ = 60 â†’ 180` | `Î´_guard` mesurÃ© Ã  **0,20â€“0,35 pt**, trÃ¨s au-dessus du coupe-circuit : **aucune cotation autorisÃ©e**. Zone d'observation et de mesure uniquement |
| `Ï„ = 180` | `Î´_guard` chute Ã  **0,0132 pt** (1,3 tick). Le mouvement rÃ©siduel de l'oracle tombe Ã  **1,50 $**. **Ouverture de la fenÃªtre d'action** |
| `Ï„ = 240` | Fermeture de la fenÃªtre du PDF. `w(Ï„)` commence sa rampe : le rÃ©fÃ©rentiel glisse de l'oracle instantanÃ© vers le TWAP de rÃ¨glement |
| `Ï„ = 262` (mÃ©diane observÃ©e) | **`Ï„_lock` : le signe de l'issue ne change plus.** L'information existe enfin, 22 s aprÃ¨s que le PDF ait fermÃ© la porte |
| `Ï„ = 270` | DÃ©but de la fenÃªtre de rÃ¨glement `[270;300]`. `Î´_guard` = 0,0107 pt. Le carnet cesse souvent de publier vers 273 s. **Fermeture de la fenÃªtre d'action** |
| `Ï„ = 300` | RÃ©solution sur le TWAP `[270;300]`, qui rÃ©duit d'environ un tiers la variance du dernier bloc de 30 s |

## 4.5 Marques comportementales attribuÃ©es aux populations

| Marque observable | Population de Â§ 0.3 | Attribution | Instant |
|---|---|---|---|
| 63,2 % du notionnel en 10,2 % des Ã©vÃ©nements, concentrÃ© sur 5 instants (`22,9 s`, `142,0 s`, `240,4 s`, `272,8 s`, `278,3 s`) | Gros flux (`usd > 30 $`) | **Signature seulement.** Ni cÃ´tÃ©, ni maker/taker | ConcentrÃ©, dont **3 des 5 pics aprÃ¨s `Ï„ = 240 s`** |
| Spread mÃ©dian de 0,02 maintenu sur toute la session | Fournisseurs de liquiditÃ© passive | Non prouvÃ©e | Continu |
| Spread croisÃ© Ã  `âˆ’0,01` sur 2 lignes | Non identifiable â€” probablement un croisement transitoire non appariÃ© | Non prouvÃ©e | Ponctuel |
| Ã‰cart carnet/ancrage qui **croÃ®t** en fin de session (0,090 â†’ 0,464 pt) | Participants qui ne re-cotent pas Ã  la cadence de `Ïƒ_T` | Non prouvÃ©e | `Ï„ > 180 s` |
| Ask du cÃ´tÃ© ancrÃ© Ã  `0,14` alors que `oracle âˆ’ K = +4,26 $`, puis `0,02` 26 s plus tard | Idem | Non prouvÃ©e | `Ï„ = 240 â†’ 266 s` |
| Krach d'ask de **0,279 s** (PDF p. 16) invisible Ã  1 Hz | Population rapide non identifiable | Non prouvÃ©e | Ponctuel |
| 62,3 % des Ã©vÃ©nements pour 11,0 % du notionnel | Micro-flux | Bruit d'exÃ©cution, sans impact sur le meilleur niveau | Continu |

**La seule structure de population exploitable, et elle est dÃ©fensive :** trois des cinq pics de gros flux se produisent **aprÃ¨s `Ï„ = 240 s`**, c'est-Ã -dire dans la fenÃªtre d'action retenue. Le bot y sera donc en concurrence avec la population qui porte 63 % du notionnel. ConsÃ©quence sur la capacitÃ© : Â§ 7.

## 4.6 MÃ©canismes

### MÃ©canisme M1 â€” Cotation passive ancrÃ©e Ã  l'oracle, dÃ©cote pilotÃ©e par le risque rÃ©siduel

**Ã‰vÃ©nement dÃ©clencheur** : toute mise Ã  jour de l'oracle **ou** du carnet (union des deux flux) alors que `Ï„ âˆˆ [180 ; 270]`.

**ChaÃ®ne causale complÃ¨te.**
1. Ã€ `Ï„`, l'oracle publie `oracle(Ï„)`. La distance au strike vaut `m*(Ï„) = oracle(Ï„) âˆ’ K`.
2. Le mouvement que l'oracle peut **encore** faire avant le rÃ¨glement a pour Ã©cart-type `Ïƒ_T(Ï„) = Ïƒâ‚ƒâ‚€Â·âˆš((300âˆ’Ï„)/30)`, mesurÃ© et non postulÃ©. La dÃ©termination mÃ©canique de l'issue vaut `z(Ï„) = |m*(Ï„)|/Ïƒ_T(Ï„)`.
3. `Ïƒ_T` dÃ©croÃ®t de faÃ§on **dÃ©terministe et connue Ã  l'avance**. Ã€ `Ï„ = 180 s`, il vaut 14,40 $ ; Ã  `Ï„ = 240 s`, 10,18 $ ; Ã  `Ï„ = 270 s`, 7,20 $. Le mouvement rÃ©siduel mesurÃ© suit : 11,62 $ â†’ 1,50 $ â†’ 1,55 $.
4. Le carnet, lui, ne re-cote pas Ã  cette cadence. L'Ã©cart mÃ©dian `|p_ancrÃ© âˆ’ ask|` mesurÃ© vaut **0,0900 pt** en `[180;240[` et **0,4642 pt** en `[240;270[`, contre 0,1197 pt en `[60;180[`.
5. **FenÃªtre d'exploitabilitÃ©** : la durÃ©e pendant laquelle une quote passive Ã  `p_ancrÃ© âˆ’ Î´` est touchÃ©e. MesurÃ©e par le proxy Â« contact du BBO dans les 2 s Â» : **48,6 % Ã  Î´ = 0,02** et **43,2 % Ã  Î´ = 0,05** dans les derniÃ¨res minutes ; 88,5 % et 76,7 % en phase milieu. La durÃ©e de validitÃ© utile mesurÃ©e est de **2 s**, avec une expiration secondaire au P90 de la latence de rÃ©action (6,7 s Ã  32,3 s selon session).
6. **Action** : poster un ordre limite **passif** du cÃ´tÃ© ancrÃ©, au prix `p_ancrÃ©,cÃ´tÃ©(Ï„) âˆ’ Î´(Ï„)`, arrondi au tick vers le bas, taille `floor(N)`.
7. **ClÃ´ture** : dÃ©tention jusqu'au rÃ¨glement (sortie unique). **Invalidation** : nouvelle mise Ã  jour de l'oracle qui change `z` de plus de `Î´`, saut d'ask supÃ©rieur au quantile mesurÃ©, sortie de `[180;270]`, perte de fraÃ®cheur, ou expiration Ã  2 s.

**Test de non-directionnalitÃ©** : **Oui**, et la dÃ©monstration est mÃ©canique. Si le sous-jacent avait fait le mouvement miroir, `|m*|` serait identique, `z` serait identique, `Ïƒ_T` serait identique, `Î´` serait identique, et l'ordre serait postÃ© sur le cÃ´tÃ© complÃ©mentaire au prix complÃ©mentaire â€” que l'invariant `bid_OUI + ask_NON = 1` garantit exactement symÃ©trique. L'action est donc rigoureusement miroir. **Ce n'est pas le cas si `Ï„ < 180 s`**, parce que le mouvement rÃ©siduel de l'oracle (11,62 $ = 1,6 Ïƒâ‚ƒâ‚€) y dÃ©passe la dÃ©cote : la position devient alors un pari. C'est pourquoi la borne de 180 s est une condition de non-directionnalitÃ©, pas un rÃ©glage de performance.

**PnL par sous-groupe** : **non dÃ©terminable au niveau du fill** â€” aucune session ne contient d'ordre du bot. Au niveau du signal, sur les sessions mesurantes : positif dans le sous-groupe Â« oracle dÃ©crochÃ© Â», nÃ©gatif dans Â« oracle oscillant Â», d'oÃ¹ la condition `z â‰¥ z_min âˆ§ Xâ‚†â‚€ = 0`.

**Contrepartie** : participants maintenant une quote qui prix encore une incertitude que le temps a supprimÃ©e. **Classe de signature probable : fournisseurs de liquiditÃ© passive.** Raison pour laquelle ils laissent l'Ã©cart : ils re-cotent sur Ã©vÃ©nement de carnet, pas sur la dÃ©croissance du temps restant, qui est un Ã©vÃ©nement silencieux â€” rien ne se passe, et c'est prÃ©cisÃ©ment l'information. Attribution **non prouvÃ©e** : `order_id` et `maker/taker` Ã  journaliser.

**Budget de latence requis** : la fenÃªtre utile est de **2 s**, la cadence du carnet de 0,013 s. Un budget aller-retour infÃ©rieur Ã  **200 ms** laisse 10 % de la fenÃªtre en marge. Budget rÃ©el du bot : **non dÃ©terminable** (Â§ 0.6). Part des Ã©vÃ©nements atteignables : **non dÃ©terminable** â€” Ã  mesurer dÃ¨s la premiÃ¨re session instrumentÃ©e.

### MÃ©canisme M2 â€” ContrÃ´le d'intÃ©gritÃ© par invariants croisÃ©s

**Ã‰vÃ©nement dÃ©clencheur** : chaque mise Ã  jour du carnet.
**ChaÃ®ne** : mesurer `|bid_OUI + ask_NON âˆ’ 1|` et `|ask_OUI + bid_NON âˆ’ 1|` ; mesurer `|basis(Ï„) âˆ’ mÃ©diane_glissante(basis)|` ; mesurer `Ã¢ge_oracle` et `trou_flux`. Tout dÃ©passement âŸ¹ suspension immÃ©diate de M1 et annulation des ordres en vol.
**FenÃªtre** : instantanÃ©e.
**Test de non-directionnalitÃ©** : **Oui** â€” un contrÃ´le d'intÃ©gritÃ© refuse symÃ©triquement. PnL par sous-groupe : 0,00 $ dans les quatre (aucune position prise).
**Contrepartie** : aucune. C'est un garde-fou.
**Justification** : deux lignes sur 1 026 violent l'invariant de complÃ©mentaritÃ©, deux spreads croisÃ©s Ã  `âˆ’0,01` passent Ã  tort le test T20 du PDF, et le basis mesure un dÃ©calage 12 Ã  29 fois supÃ©rieur Ã  `M`. Sans ce mÃ©canisme, le bot cote sur des donnÃ©es corrompues.

### MÃ©canisme M3 â€” Instrumentation de mesure du taux de remplissage (mÃ©canisme de mesure, pas de PnL)

**Ã‰vÃ©nement dÃ©clencheur** : identique Ã  M1, mais avec une taille au **minimum rÃ©glementaire** (`PARTS_MIN = 5`) et un objectif de mesure, non de gain.
**ChaÃ®ne** : coter Ã  `Î´` connu, journaliser les 8 champs de latence (Â§ 8.6) et l'issue, mesurer le taux de fill **rÃ©el** par phase et par valeur de `Î´`, puis reconstruire la table 8.4 bis avec des fills au lieu de contacts.
**FenÃªtre** : la session entiÃ¨re.
**Test de non-directionnalitÃ©** : **Oui**. PnL par sous-groupe : attendu proche de zÃ©ro, Ã  la mesure.
**Contrepartie** : idem M1.
**Justification** : c'est le seul mÃ©canisme qui transforme les onze champs manquants en donnÃ©es. Sans lui, la table 8.4 bis restera pour toujours une table de contacts BBO, pas de remplissages, et **aucun PnL par sous-groupe ne sera jamais calculable**. Il doit Ãªtre exÃ©cutÃ© **avant** M1 en taille rÃ©elle.

---

# 5. Analyse Ã©clatÃ©e des 22 fonctions indicatrices

### 5.1 â€” T1 Â· `TICK_PERIME`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le vis-Ã -vis de l'ancrage :** **sert.** L'ancrage est la derniÃ¨re valeur connue de l'oracle ; si cette valeur est pÃ©rimÃ©e, l'ancrage est faux et la dÃ©cote est calculÃ©e contre un fantÃ´me.
- **Termes et grandeurs :** `Ã¢ge(Ï„) = Ï„ âˆ’ Ï„_dernier_tick_oracle`, `STALENESS_MAX`.
- **Ã‰lÃ©ments concernÃ©s :** oracle â†’ bot.
- **Relation :** `Ã¢ge(Ï„) â‰¤ STALENESS_MAX`. MesurÃ© au snapshot de carnet : mÃ©diane **0,590 s**, P10 0,109 s, P90 0,978 s, maximum **1,979 s** ; 610/610 conformes. Ce que cela rÃ©vÃ¨le : la fraÃ®cheur mesurable est une **fraÃ®cheur de capture**, pas un Ã¢ge source â€” `ts_src` vaut `payload` sur 269/269 lignes. Test de non-directionnalitÃ© : Oui, un Ã¢ge est insensible au signe. Contrepartie : aucune, c'est un garde-fou. ConsÃ©quence : le seuil est respectÃ© avec marge, mais il ne prouve pas l'Ã¢ge Ã  la plateforme.
- **Marque comportementale :** aucune. Le test n'a jamais mordu dans la fenÃªtre d'entrÃ©e sur les sessions instrumentÃ©es ; un agent mesure toutefois un maximum de **4,0 s**, donc le seuil de 2,5 s mordrait sur cette session.
- **Ce que le PDF proposait :** `Ã¢ge â‰¤ 2,5 s`, Ã  la dÃ©cision.
- **Ce que les agents ont observÃ© :** Â« *Ã¢ge de la derniÃ¨re ligne oracle au snapshot BBO : mÃ©diane 0,590 s [...] 610/610 passent* Â» ; Â« *T1 fraÃ®cheur du tick oracle â‰¤ 2,5 s : p95 2,0, max **4,0** â†’ valeur idÃ©ale â‰ˆ 4 s* Â». n_confirmant **7** / n_mesurÃ© **7** ; contredisants : 0 sur le principe, 1 sur la valeur du seuil.
- **Test de non-directionnalitÃ© :** Oui (forme logique). PnL par sous-groupe : non dÃ©terminable â€” un veto de validitÃ© ne produit pas de PnL.
- **Calibrage retenu :** `STALENESS_MAX = 2,5 s`, unitÃ© s, mesurÃ© Ã  l'instant de dÃ©cision sur l'horloge de capture de l'oracle, connu Ã  la rÃ©ception du snapshot de carnet. InterprÃ©tation : au-delÃ , l'ancrage n'est plus le marchÃ©. **ConservÃ© Ã  2,5 s malgrÃ© le max de 4,0 s**, parce qu'un ancrage pÃ©rimÃ© produit une perte rÃ©elle alors qu'un refus ne produit qu'un manque Ã  gagner (arbitrage 1,75:1 de la remarque 35).
- **FenÃªtre, budget, contrepartie :** instantanÃ© ; budget requis nul ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1. Un ancrage pÃ©rimÃ© expose directement au sens du sous-jacent.
- **RÃ¨gle d'action 1 :** SI `Ã¢ge(Ï„) > 2,5 s` ALORS aucune cotation, annulation de tout ordre en vol, journalisation du code `TICK_PERIME` AU MOMENT de chaque Ã©valuation JUSQU'Ã€ retour de `Ã¢ge(Ï„) â‰¤ 2,5 s`.

### 5.2 â€” T2 Â· `TICK_TROU`
- **CatÃ©gorie :** Structurelle.
- **RÃ´le :** **sert.** Un trou de flux invalide `Ïƒâ‚ƒâ‚€`, donc `Ïƒ_T`, donc toute la dÃ©cote.
- **Termes :** `trou(Ï„)`, `GEL_MAX`.
- **Ã‰lÃ©ments :** oracle â†’ bot ; carnet â†’ bot.
- **Relation :** `trou(Ï„) â‰¤ GEL_MAX`. MesurÃ© cÃ´tÃ© carnet : mÃ©diane **0,013 s**, P90 1,192 s, max **6,519 s** ; cÃ´tÃ© oracle : max **2 s** dans la fenÃªtre d'entrÃ©e, **7 s** sur la session complÃ¨te. Un agent mesure un trou de carnet de **10,48 s**. Ce que cela rÃ©vÃ¨le : les deux flux ont des rÃ©gimes de trou totalement diffÃ©rents, d'un facteur ~500 en mÃ©diane. Test : Oui. Contrepartie : aucune. ConsÃ©quence : **un seuil unique pour les deux flux est une erreur de conception** ; il faut deux seuils.
- **Marque comportementale :** les trous de carnet apparaissent en fin de session â€” plusieurs sessions cessent de publier vers `Ï„ â‰ˆ 273 s`, ce qui tombe **dans la fenÃªtre d'action retenue**.
- **Ce que le PDF proposait :** `trou â‰¤ 8 s`, sur horloge marchÃ©, un seul seuil.
- **Ce que les agents ont observÃ© :** Â« *Ã©cart BBO entre instants distincts dans l'entrÃ©e : mÃ©diane 0,013 s, P90 1,192 s, maximum 6,519 s ; oracle : maximum 2 s* Â» ; Â« *T2 trou de flux BBO â‰¤ 8 s : p99 2,98, max 6,80 Â· p99 4,71, **max 10,48** â†’ â‰ˆ 11 s* Â». n_confirmant **8** / n_mesurÃ© **8**.
- **Test de non-directionnalitÃ© :** Oui. PnL par sous-groupe : non dÃ©terminable.
- **Calibrage retenu :** **deux seuils distincts.** `GEL_MAX_ORACLE = 8 s` (conservÃ© : cohÃ©rent avec la cadence de 1 s et le max observÃ© de 7 s). `GEL_MAX_CARNET = 3 s`, unitÃ© s, mesurÃ© sur l'horloge de capture du carnet Ã  4 Hz. Justification du 3 s : P90 mesurÃ© 1,19 s, le seuil doit mordre avant que le bot cote contre un carnet mort en fin de session.
- **FenÃªtre, budget, contrepartie :** instantanÃ© ; budget nul ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1. Coter contre un carnet qui a cessÃ© de publier est une exposition pure.
- **RÃ¨gle d'action 2 :** SI `trou_oracle(Ï„) > 8 s` OU `trou_carnet(Ï„) > 3 s` ALORS aucune cotation, annulation, gel du compteur de persistance, code `TICK_TROU` AU MOMENT de chaque Ã©valuation JUSQU'Ã€ reprise des deux flux sous leurs seuils.

### 5.3 â€” T3 Â· `SPREAD_INDISPO`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **sert.** Sans les deux cÃ´tÃ©s, le prix de la quote n'est pas calculable et l'invariant de complÃ©mentaritÃ© n'est pas vÃ©rifiable.
- **Termes :** disponibilitÃ© de `bid` et `ask` du cÃ´tÃ© ancrÃ© ; **boolÃ©en de disponibilitÃ©**, jamais une sentinelle numÃ©rique.
- **Ã‰lÃ©ments :** carnet â†’ bot.
- **Relation :** `disponible(bid) âˆ§ disponible(ask) = vrai`. MesurÃ© : 610/610 dans la fenÃªtre d'entrÃ©e ; les valeurs manquantes se situent Ã  `Ï„ = 0,198 s` et `Ï„ = 272,967 s`, donc **une des deux tombe dans la fenÃªtre d'action retenue**. Taux de manquants par champ : 0,10 % Ã  0,20 %. Test : Oui. Contrepartie : aucune. ConsÃ©quence : le test doit renvoyer un **boolÃ©en faux**, pas une valeur numÃ©rique â€” la sentinelle `âˆ’1` du corpus satisfait `â‰¤ 0,04` et ouvre le verrou (`[E1]` 1.3.bis Â§ 3, *fail-open* accidentel).
- **Marque comportementale :** les manquants sont aux deux extrÃ©mitÃ©s de la session, lÃ  oÃ¹ le carnet s'amorce et se retire.
- **Ce que le PDF proposait :** spread disponible, sinon refus (fail-closed via bloc A).
- **Ce que les agents ont observÃ© :** Â« *spread du cÃ´tÃ© sÃ©lectionnÃ© disponible sur 610/610 snapshots d'entrÃ©e ; les valeurs manquantes BBO sont Ã  t=0,198 s et t=272,967 s* Â». n_confirmant **6** / n_mesurÃ© **6**. `[E1]` 1.5 Â§ 3.4 documente le *fail-open* accidentel.
- **Test de non-directionnalitÃ© :** Oui. PnL par sous-groupe : non dÃ©terminable.
- **Calibrage retenu :** boolÃ©en de disponibilitÃ© par champ, Ã©valuÃ© Ã  chaque snapshot, horloge carnet. **Aucune grandeur ne porte de valeur numÃ©rique signifiant Â« indisponible Â».** Fail-closed.
- **FenÃªtre, budget, contrepartie :** instantanÃ© ; budget nul ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1 et critÃ¨re 4. C'est le seul dÃ©faut du corpus qui transforme un refus en autorisation silencieuse.
- **RÃ¨gle d'action 3 :** SI un des quatre champs `yes_bid`, `yes_ask`, `no_bid`, `no_ask` est absent OU si `|bid_OUI + ask_NON âˆ’ 1| > 1 tick` ALORS aucune cotation, annulation, code `SPREAD_INDISPO` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ un snapshot complet et cohÃ©rent.

### 5.4 â€” T4 Â· `SIGMA_INDISPO`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **sert â€” c'est la racine de tout le systÃ¨me.** `Ïƒâ‚ƒâ‚€` dÃ©termine `Ïƒ_T`, donc `z`, donc la dÃ©cote, donc la dÃ©gressivitÃ©.
- **Termes :** `Ïƒâ‚ƒâ‚€`, fenÃªtre de 30 s, plausibilitÃ©.
- **Ã‰lÃ©ments :** oracle â†’ bot.
- **Relation :** `Ïƒâ‚ƒâ‚€` existe si et seulement si la fenÃªtre de 30 s d'oracle est pleine, soit `Ï„ â‰¥ 30 s`. MesurÃ©, **estimateur A (incrÃ©ments 1 s Ã— âˆš30)** : mÃ©diane des mÃ©dianes **7,20 $**, P10 2,64 $, P90 9,94 $, min 1,52 $, max 11,48 $, n = 21 sessions. **Estimateur B (Ã©cart-type des niveaux sur 30 s)** : mÃ©diane 2,25 $, n = 8. Le PDF pose 26,50 $, soit **Ã—3,68** de trop avec A et **Ã—11,8** avec B. Ce que cela rÃ©vÃ¨le : la constante n'est pas mal rÃ©glÃ©e, elle est **d'une autre nature** â€” un rÃ©gime de volatilitÃ© de deux jours a Ã©tÃ© gelÃ© en constante universelle. Test : Oui, un Ã©cart-type est insensible au signe. Contrepartie : aucune. ConsÃ©quence : mesurer en ligne, refuser hors plage de plausibilitÃ©.
- **Marque comportementale :** `Ïƒâ‚ƒâ‚€` varie d'un facteur **6,6 intra-session** (3,01 â†’ 19,89 $ sur une session). Une constante ne peut pas suivre Ã§a.
- **Ce que le PDF proposait :** `Ïƒ mesurable et plausible` ; table de rÃ©fÃ©rence calculÃ©e avec `Ïƒâ‚ƒâ‚€ = 26,50 $`.
- **Ce que les agents ont observÃ© :** Â« *Ïƒâ‚ƒâ‚€ = 26,50 $ est surestimÃ© d'un facteur 6 Ã  15* Â» ; Â« *2,31Ã— / 4,61Ã— / 8,68Ã—* Â» ; Â« *Ïƒâ‚ƒâ‚€ rÃ©elle = 7,97 $ / 30 s (mÃ©diane des 3 sessions)* Â» ; Â« *surestimÃ© Ã—2,9* Â» ; Â« *facteur 4,87Ã—* Â» ; Â« ***Ce n'est pas un rÃ©glage trop prudent, c'est une erreur d'unitÃ©.*** Â» n_confirmant **29** / n_mesurÃ© **29** ; contredisants **0**. UniversalitÃ© : totale.
- **Test de non-directionnalitÃ© :** Oui. PnL par sous-groupe : sans objet ; mais l'effet est mesurÃ© â€” Â« *le postulat avec la constante Ïƒâ‚ƒâ‚€ = 26,50 $ ne trade **jamais*** Â», donc PnL 0,00 $ dans les quatre sous-groupes.
- **Calibrage retenu :** `Ïƒâ‚ƒâ‚€(Ï„) =` Ã©cart-type des incrÃ©ments d'oracle Ã  1 s sur `]Ï„âˆ’30 ; Ï„]`, multipliÃ© par `âˆš30`. UnitÃ© **$ de sous-jacent**. Moment de mesure `Ï„ â‰¥ 30 s` puis Ã  chaque tick d'oracle. Moment de connaissance : `Ï„ â‰¥ 30 s` + latence de rÃ©ception (non mesurÃ©e). Horloge `Ï„`Â·marchÃ©. InterprÃ©tation : dispersion du chemin restant. **Bornes de plausibilitÃ© `[0,5 ; 40] $`**, hors bornes âŸ¹ fail-closed. **Jamais de valeur par dÃ©faut, jamais hÃ©ritÃ©e de la session prÃ©cÃ©dente.**
- **FenÃªtre, budget, contrepartie :** calcul < 1 ms ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1. La constante gelÃ©e annule 100 % de la stratÃ©gie ; c'est le dÃ©faut le plus coÃ»teux du corpus.
- **RÃ¨gle d'action 4 :** SI `Ï„ < 30 s` OU fenÃªtre de 30 s incomplÃ¨te OU `Ïƒâ‚ƒâ‚€(Ï„) âˆ‰ [0,5 ; 40] $` ALORS aucune cotation, code `SIGMA_INDISPO` AU MOMENT de chaque Ã©valuation JUSQU'Ã€ une fenÃªtre pleine avec `Ïƒâ‚ƒâ‚€` dans les bornes.

### 5.5 â€” T5 Â· `BASIS_CORROMPU`
- **CatÃ©gorie :** **Directionnelle.**
- **RÃ´le :** **contredit.** Tel qu'Ã©crit, il conditionne l'action au signe d'un dÃ©calage de source, ce qui introduit une asymÃ©trie sans rapport avec l'ancrage.
- **Termes :** `spot(Ï„)`, `oracle(Ï„)`, `basis(Ï„) = spot âˆ’ oracle`, drapeau `HALT`.
- **Ã‰lÃ©ments :** spot, oracle.
- **Relation :** `spot(Ï„) > oracle(Ï„)`. MesurÃ© vrai sur **100 % des lignes appariÃ©es** de 5 sessions (1943/1943, 3388/3388, 1190/1190, 682/682, 619/619), amplitude mÃ©diane de consensus **+42,1 $**, plage 22,90 â†’ 53,32 $. Ce que cela rÃ©vÃ¨le : un dÃ©calage permanent de 42 $, soit **5,8 Ïƒâ‚ƒâ‚€**, qui ne se referme jamais, n'est pas une inefficience mais une diffÃ©rence de source ou de dÃ©finition. Test de non-directionnalitÃ© : **Non** â€” le miroir `spot < oracle` n'est pas traitÃ© par une action miroir. Contrepartie : aucune, il n'y a pas d'Ã©cart Ã  fermer. ConsÃ©quence : le test est **Ã  la fois directionnel et inerte** â€” il n'a jamais rien filtrÃ©.
- **Marque comportementale :** aucune population ne le referme, Ã  aucun instant, sur aucune session.
- **Ce que le PDF proposait :** `spot > oracle` et marchÃ© non en HALT.
- **Ce que les agents ont observÃ© :** Â« *La garde `basis â‰¤ 0` **ne peut jamais se dÃ©clencher** : le basis est structurellement dÃ©calÃ© de +40 $. La garde est aveugle Ã  un dÃ©calage 12 Ã  29 fois supÃ©rieur Ã  `M`.* Â» ; Â« *le miroir `spot<oracle` n'est pas traitÃ© par une action miroir ; 1943/1943 lignes seraient acceptÃ©es uniquement parce que le signe observÃ© est positif* Â» ; Â« *Le basis est Ã©norme, constant et inutile comme test* Â». n_confirmant **12** / n_mesurÃ© **12**. La moitiÃ© Â« non HALT Â» du test est **invÃ©rifiable** : aucun champ `HALT` dans le corpus.
- **Test de non-directionnalitÃ© :** **Non.** PnL par sous-groupe : non dÃ©terminable, et sans objet â€” la rÃ¨gle est hors pÃ©rimÃ¨tre.
- **Calibrage retenu :** sans objet pour la version signÃ©e. **Requalification structurelle retenue** : `|basis(Ï„) âˆ’ mÃ©diane_glissante_60s(basis)| â‰¤ 3Â·Ïƒ_session(basis)`, unitÃ© $, mesurÃ© Ã  chaque ligne spot appariÃ©e au dernier oracle par `as-of`, horloge de capture, interprÃ©tation Â« cohÃ©rence de source Â». Valeur de rÃ©fÃ©rence de consensus : mÃ©diane **+42,1 $**, `Ïƒ` intra-session typique 2,5 $ (P10â€“P90 de 39,4 Ã  47,7 $).
- **FenÃªtre, budget, contrepartie :** instantanÃ© ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1 pour le rejet ; critÃ¨re 3 pour la requalification, qui capte les dÃ©crochages de flux sans coÃ»ter une seule fausse entrÃ©e.
- **RÃ¨gle 5 : hors pÃ©rimÃ¨tre (directionnelle) â€” ne pas implÃ©menter comme condition d'entrÃ©e.** Requalification structurelle : SI `|basis(Ï„) âˆ’ mÃ©diane_glissante_60s(basis)| > 3Â·Ïƒ_session(basis)` OU si le drapeau `HALT` est absent du flux ALORS aucune cotation, code `BASIS_CORROMPU` AU MOMENT de chaque ligne spot JUSQU'Ã€ retour dans la bande.

### 5.6 â€” T6 Â· `T0_APPROXIME`
- **CatÃ©gorie :** Structurelle.
- **RÃ´le :** **sert.** `Ï„` pilote `Ïƒ_T`, `M`, `Î»`, `P_max`, `w` et la fenÃªtre d'action. Une origine fausse les fausse **toutes en mÃªme temps**.
- **Termes :** `Ï„ = t_horloge âˆ’ Ï„â‚€`, `Ï„â‚€ = endDate âˆ’ 300 s`, `t0_source`.
- **Ã‰lÃ©ments :** source officielle â†’ bot.
- **Relation :** `t0_source = officiel`. MesurÃ© : **0/41 sessions** contiennent `endDate` ou une date absolue ; `t_ms` est un relatif de capture. Ce que cela rÃ©vÃ¨le : la totalitÃ© du corpus a Ã©tÃ© analysÃ©e sur une horloge **reconstruite**, ce qui rend chaque seuil en secondes approximatif Ã  la prÃ©cision de l'origine prÃ¨s. Test : Oui. Contrepartie : aucune. ConsÃ©quence : sans `endDate`, le refus est obligatoire et porte sur la **session entiÃ¨re**, pas sur un instant.
- **Marque comportementale :** le PDF documente le coÃ»t de l'erreur : une dÃ©tection mÃ©diane Ã  `Ï„ = 156,10 s` lue comme origine plaÃ§ait la fenÃªtre de rÃ¨glement Ã  **426,10 s**, soit 126 s aprÃ¨s un marchÃ© clos.
- **Ce que le PDF proposait :** origine lue **une seule fois** dans `endDate âˆ’ 300 s`, jamais `time.time()` ; refus `T0_APPROXIME` sinon.
- **Ce que les agents ont observÃ© :** Â« *`t_ms` relatif commence Ã  0 mais aucun `endDate`, date absolue ou origine officielle ; valeur non calculable â†’ refus requis* Â» ; Â« *T6/T7 doivent refuser explicitement sans `endDate`, `K` et `k_source`* Â». n_confirmant **6** / n_mesurÃ© **6** ; contredisants 0.
- **Test de non-directionnalitÃ© :** Oui. PnL par sous-groupe : non dÃ©terminable.
- **Calibrage retenu :** `Ï„â‚€ = endDate_API âˆ’ 300 s`, lu **une fois** Ã  l'ouverture, horloge marchÃ© absolue, prÃ©cision requise â‰¤ 100 ms. Aucun repli local autorisÃ©. InterprÃ©tation : origine imposÃ©e de l'extÃ©rieur, identique pour tous.
- **FenÃªtre, budget, contrepartie :** une lecture Ã  l'ouverture ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1. Une origine fausse dÃ©cale la dÃ©cote dÃ©gressive de son axe, donc dÃ©truit la propriÃ©tÃ© de non-directionnalitÃ© elle-mÃªme.
- **RÃ¨gle d'action 6 :** SI `endDate` est absent OU `t0_source â‰  officiel` ALORS **la session entiÃ¨re est refusÃ©e**, aucune cotation, code `T0_APPROXIME` AU MOMENT de l'ouverture JUSQU'Ã€ la session suivante.

### 5.7 â€” T7 Â· `K_NON_OFFICIEL`
- **CatÃ©gorie :** Structurelle.
- **RÃ´le :** **sert â€” c'est le pivot de l'ancrage.** Sans `K`, `m*` n'existe pas, donc l'ancrage n'existe pas.
- **Termes :** `K = TWAP 30 s` publiÃ©, `k_source`.
- **Ã‰lÃ©ments :** source officielle â†’ bot.
- **Relation :** `k_source = officiel`. MesurÃ© : **0/41 sessions** contiennent `K` ou `k_source`. Tous les agents ont utilisÃ© un proxy : moyenne des points d'oracle sur `[0;30]`, ou dernier tick `â‰¤ Ï„â‚€`, ou valeur lue dans le PDF de session. Ã‰carts mesurÃ©s entre proxys : **+0,25 / +12,25 / âˆ’4,50 $**. Ce que cela rÃ©vÃ¨le : sur une session, l'Ã©cart de **12,25 $ inverse le rÃ¨glement**. Test : Oui, l'exigence de source est insensible au signe. Contrepartie : aucune. ConsÃ©quence : c'est **le point le plus lourd de tout l'audit**, et un agent le dit explicitement.
- **Marque comportementale :** aucune ; c'est un dÃ©faut d'instrument, pas de marchÃ©.
- **Ce que le PDF proposait :** `K = TWAP 30 s publiÃ© Ã  Ï„â‚€`, `k_source = officiel`, vÃ©rifiÃ© avant l'armement ; Â« *un strike approchÃ© peut inverser le signe de `m*`* Â» ; biais proxy Â±15 $ contre un signal de 11,05 $.
- **Ce que les agents ont observÃ© :** Â« *avec le strike infÃ©rÃ©, B se rÃ¨gle UP et contredit le consensus terminal du carnet ; avec le strike ajustÃ©, A = B = C = DOWN* [...] *si le strike infÃ©rÃ© est le bon, la session B change de camp et les taux de rÃ©ussite ci-dessous s'effondrent* Â» ; Â« *tranchez la question du strike sur une session d'archive oÃ¹ le rÃ¨glement est connu de source sÃ»re : c'est le seul point qui dÃ©cide si votre session B est un gain ou une perte, et **il pÃ¨se plus lourd que tout le reste de cette analyse*** Â». n_confirmant **8** / n_mesurÃ© **8**.
- **Test de non-directionnalitÃ© :** Oui sur l'exigence. **Mais l'absence de `K` rend tout PnL par sous-groupe non interprÃ©table**, parce que l'affectation d'une session Ã  un sous-groupe dÃ©pend de `K`.
- **Calibrage retenu :** `K` lu **une fois** Ã  `Ï„â‚€` sur l'endpoint officiel du TWAP 30 s, avec `k_source = officiel` vÃ©rifiÃ© **avant** l'armement du cÃ´tÃ©. Aucun proxy autorisÃ©, aucune tolÃ©rance d'Ã©cart. `K_ECART_MAX_USD` du corpus est supprimÃ© : `[E1]` 1.5 Â§ 3.3 dÃ©montre qu'un Ã©cart tolÃ©rÃ© de 5 $ Ã©gale la marge exigÃ©e en fin de fenÃªtre, donc annule Ã  lui seul le signal minimal admissible.
- **FenÃªtre, budget, contrepartie :** une lecture Ã  l'ouverture ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1, sans discussion. C'est le seul paramÃ¨tre dont une erreur **inverse** le cÃ´tÃ© achetÃ©.
- **RÃ¨gle d'action 7 :** SI `K` est absent OU `k_source â‰  officiel` ALORS **la session entiÃ¨re est refusÃ©e**, aucune cotation, code `K_NON_OFFICIEL` AU MOMENT de l'ouverture JUSQU'Ã€ la session suivante.

### 5.8 â€” T8 Â· `HORS_FENETRE`
- **CatÃ©gorie :** Structurelle.
- **RÃ´le :** **sert, aprÃ¨s correction.** La fenÃªtre est le paramÃ¨tre qui rend la stratÃ©gie non directionnelle, parce qu'elle dÃ©cide combien de mouvement d'oracle reste possible.
- **Termes :** `Ï„`, bornes de fenÃªtre.
- **Ã‰lÃ©ments :** horloge de fenÃªtre.
- **Relation :** `Ï„_min â‰¤ Ï„ â‰¤ Ï„_max`. Le PDF pose `[60;240]`. MesurÃ© : `Ï„_lock`, l'instant aprÃ¨s lequel le signe de l'issue ne change plus, vaut **262 s** sur la session dÃ©cisive, soit **22 s aprÃ¨s la fermeture**. Mouvement rÃ©siduel mÃ©dian de l'oracle par phase : **10,76 $** en `[30;60[`, **11,62 $** en `[60;180[`, **1,50 $** en `[180;240[`, **1,55 $** en `[240;270[`. RapportÃ© au `Ïƒâ‚ƒâ‚€` de consensus de 7,20 $ : **1,6 Ïƒ** avant 180 s, **0,21 Ïƒ** aprÃ¨s. Ce que cela rÃ©vÃ¨le : dans la fenÃªtre du PDF, l'issue n'existe pas encore ; toute position y est un pari, quels que soient les autres filtres. Test de non-directionnalitÃ© : Oui, une borne temporelle est insensible au signe â€” **mais la valeur de la borne dÃ©termine si les autres rÃ¨gles sont directionnelles ou non**. Contrepartie : les participants qui cotent l'incertitude aprÃ¨s qu'elle a mÃ©caniquement disparu. ConsÃ©quence : dÃ©placer la fenÃªtre.
- **Marque comportementale :** 610/1027 lignes de carnet dans `[60;240]`, premiÃ¨re Ã  `60,287 s`, derniÃ¨re Ã  `239,755 s`. Trois des cinq pics de gros flux se produisent **aprÃ¨s** 240 s. Le carnet cesse souvent de publier vers `Ï„ â‰ˆ 273 s`.
- **Ce que le PDF proposait :** `Ï„ âˆˆ [60 ; 240] s`, borne basse alignÃ©e sur la fenÃªtre X.
- **Ce que les agents ont observÃ© :** Â« *La fenÃªtre `[60 ; 240]` ne contient pas l'information : sur S1 le signe se verrouille Ã  262 s, et 16â€“18 % du chemin de prix est postÃ© dans les 20 % finaux. Le postulat cherche Ã  dÃ©cider avant que l'issue existe.* Â» ; Â« *abandonner la contrainte de fermeture Ã  240 s qui exclut par construction l'intervalle oÃ¹ l'issue se dÃ©cide* Â» ; Â« *T8 fenÃªtre d'entrÃ©e `[60;240]` â†’ **`[45;200]`*** Â» (agent divergent) ; Â« *Ï„_max = 200 s* Â» ; Â« *`T_MAX` = **185 s*** Â». n_confirmant sur le dÃ©placement **8** / n_mesurÃ© **12** ; **contredisants 4**, qui proposent au contraire de **raccourcir** la fenÃªtre vers `[45;200]` ou `[60;185]`.
- **Test de non-directionnalitÃ© :** Oui sur la forme. PnL par sous-groupe : **non dÃ©terminable** â€” et c'est ici que la contradiction se tranche.
- **Arbitrage de la contradiction `[R]` :** les quatre agents qui veulent raccourcir la fenÃªtre le justifient par un PnL mesurÃ© sur **1 Ã  2 entrÃ©es** de leur propre session (Â« *edge concentrÃ© Ã  Ï„ = 65â€“79 s* Â», Â« *E[PnL/part] tombe Ã  +0,0131 sur [210;240)* Â»). Ce sont des optima in-sample sur n â‰¤ 2, et le mÃªme corpus Ã©tablit qu'Â« *aucun taux de rÃ©ussite ne peut Ãªtre validÃ© au seuil 95 % avec 3 sessions* Â». Ã€ l'inverse, les huit agents qui veulent dÃ©placer la fenÃªtre vers la fin s'appuient sur **trois familles de mesures indÃ©pendantes et convergentes** : `Ï„_lock`, le mouvement rÃ©siduel de l'oracle, et `Î´_guard`. Le critÃ¨re de dÃ©cision 1 tranche : sur `[60;180[`, le mouvement rÃ©siduel de 11,62 $ **dÃ©passe** toute dÃ©cote imaginable, donc les fausses entrÃ©es y laissent le bot exposÃ© au sens du sous-jacent. **La fenÃªtre tardive est retenue.**
- **Calibrage retenu :** `Ï„ âˆˆ [180 ; 270] s`. UnitÃ© s, horloge `Ï„`Â·marchÃ©, origine officielle `endDate âˆ’ 300 s`. Moment de connaissance : immÃ©diat une fois `Ï„â‚€` lu. InterprÃ©tation : intervalle dans lequel le mouvement rÃ©siduel de l'oracle est tombÃ© sous 0,25 Ïƒâ‚ƒâ‚€ **et** oÃ¹ le carnet publie encore. Borne haute Ã  270 s et non 300 s parce que le carnet cesse de publier vers 273 s et que la fenÃªtre de rÃ¨glement s'ouvre Ã  270 s.
- **FenÃªtre, budget, contrepartie :** 90 s de fenÃªtre d'action par session ; contrepartie : fournisseurs de liquiditÃ© passive, non identifiables.
- **Justification par le critÃ¨re :** critÃ¨re 1 d'abord, critÃ¨re 2 ensuite (la fenÃªtre de 90 s est trÃ¨s large devant les 2 s de durÃ©e de vie utile d'une quote).
- **RÃ¨gle d'action 8 :** SI `Ï„ < 180 s` OU `Ï„ > 270 s` ALORS aucune cotation, annulation de tout ordre en vol Ã  `Ï„ = 270 s`, code `HORS_FENETRE` AU MOMENT de chaque Ã©valuation JUSQU'Ã€ `Ï„ âˆˆ [180 ; 270]`.

### 5.9 â€” T9 Â· `MARGE_INSUFFISANTE`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **sert, aprÃ¨s requalification.** Il ne mesure pas une conviction directionnelle mais un **degrÃ© de dÃ©termination mÃ©canique**.
- **Termes :** `m*(Ï„)`, `Ïƒ_T(Ï„)`, `z(Ï„) = |m*|/Ïƒ_T`, `M(Ï„) = 0,3439Â·Ïƒ_T`, `p_brut`.
- **Ã‰lÃ©ments :** oracle, strike.
- **Relation :** `|m*(Ï„)| â‰¥ 0,3439Â·Ïƒ_T(Ï„) âŸº z(Ï„) â‰¥ 0,3439 âŸº p_brut â‰¥ 0,6511`. Les trois Ã©critures sont **la mÃªme condition**, parce que le facteur `âˆš((300âˆ’Ï„)/30)` se simplifie. Avec le `Ïƒâ‚ƒâ‚€` de consensus, `M(60) = 7,00 $` au lieu de 25,78 $ et `M(240) = 3,50 $` au lieu de 12,89 $. Ce que cela rÃ©vÃ¨le : le PDF plaÃ§ait la barre **3 fois au-dessus de la distribution qu'elle devait filtrer** â€” dÃ©rive mÃ©diane observÃ©e 8,55 $ contre seuil 25,78 $. Test de non-directionnalitÃ© : **Oui**, `z` est construit sur une valeur absolue et l'action est miroir. Contrepartie : participants qui cotent encore de l'incertitude Ã  `z` Ã©levÃ©. ConsÃ©quence : conserver la condition, mais l'Ã©crire en `z` et non en probabilitÃ©.
- **Marque comportementale :** `z` Ã©levÃ© **et** `Xâ‚†â‚€ = 0` marque la session Â« oracle dÃ©crochÃ© Â», oÃ¹ l'Ã©cart carnet/ancrage est large et persistant (`D > frais` sur 139/241 s = 57,7 %). `z` faible avec traversÃ©es rÃ©pÃ©tÃ©es du strike (jusqu'Ã  **8 traversÃ©es** mesurÃ©es) marque la session Â« oracle oscillant Â», oÃ¹ l'Ã©cart mÃ©dian est **nÃ©gatif** (âˆ’0,0611, âˆ’0,0547).
- **Ce que le PDF proposait :** `|m*| â‰¥ M(Ï„)`, `k_M = 0,3439`, statut `PROVISOIRE`.
- **Ce que les agents ont observÃ© :** Â« *sachant `|m*| â‰¥ 0,3439Â·Ïƒ_T` â‡’ **100 %** (n = 174) ; sachant `|m*| â‰¥ 1,0Â·Ïƒ_T` â‡’ **100 %** (n = 118)* Â» ; Â« *T9 : 324 instants, **100,0 %**, PnL moyen +0,93 $* Â» ; Â« *561/610 passent* Â». Contredisants : Â« ***Aucune configuration gardant un seuil de conviction actif n'est rentable*** [...] meilleure = +4,25 $ avec **k_M = 0*** Â» ; et l'avertissement dÃ©cisif Â« *les trois sessions Ã©tant toutes DOWN, un filtre qui parie systÃ©matiquement DOWN affiche un sans-faute sans rien prÃ©dire* Â». n_confirmant **9** / n_mesurÃ© **12** ; contredisants **3**.
- **Test de non-directionnalitÃ© :** **Oui en forme logique** pour `z`, Ã  condition que le cÃ´tÃ© soit dÃ©duit par miroir. PnL par sous-groupe : **non dÃ©terminable** â€” les 100 % de rÃ©ussite portent sur 3 issues indÃ©pendantes toutes du mÃªme cÃ´tÃ©, IC95 de Clopper-Pearson **[29,2 % ; 100 %]**.
- **Calibrage retenu :** `z(Ï„) = |oracle(Ï„) âˆ’ K| / Ïƒ_T(Ï„)`, sans dimension, mesurÃ© Ã  chaque tick d'oracle, connu aprÃ¨s `Ïƒâ‚ƒâ‚€` (donc `Ï„ â‰¥ 30 s`), horloge `Ï„`Â·marchÃ©. **`z_min = 1,0`** â€” et non 0,3439. Justification : 0,3439 est la valeur qui rend `p_brut â‰¥ 0,6511`, un seuil de probabilitÃ© qui n'a pas de sens tant que `p` n'est pas calibrÃ© ; `z_min = 1,0` signifie Â« il faudrait un mouvement d'un Ã©cart-type complet du chemin restant pour changer l'issue Â», une phrase mesurable sans modÃ¨le. Ã€ `Ï„ = 180 s` et `Ïƒâ‚ƒâ‚€ = 7,20 $`, cela exige `|m*| â‰¥ 14,40 $` ; Ã  `Ï„ = 240 s`, `|m*| â‰¥ 10,18 $` ; Ã  `Ï„ = 270 s`, `|m*| â‰¥ 7,20 $`. **Statut `PROVISOIRE`, Ã  valider sur les 500 sessions avec PnL positif dans chaque sous-groupe ; Ã  dÃ©faut, ramener `z_min` Ã  0.**
- **FenÃªtre, budget, contrepartie :** recalculÃ© Ã  chaque tick d'oracle, soit 1 Hz ; contrepartie : fournisseurs de liquiditÃ© passive, attribution non prouvÃ©e.
- **Justification par le critÃ¨re :** critÃ¨re 1. `z` est la seule grandeur qui mesure l'exposition rÃ©siduelle au sens du sous-jacent **sans prÃ©dire ce sens**.
- **RÃ¨gle d'action 9 :** SI `z(Ï„) < 1,0` ALORS aucune cotation, code `MARGE_INSUFFISANTE` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ `z(Ï„) â‰¥ 1,0`.

### 5.10 â€” T10 Â· `PERSISTANCE`
- **CatÃ©gorie :** Comportementale.
- **RÃ´le :** **neutre Ã  sert.** Il protÃ¨ge contre un pic isolÃ© d'oracle, ce qui est utile ; mais il **redouble T11** (Â§ 4.3) et retarde l'action.
- **Termes :** `P(Ï„)` compteur cumulÃ©, `FCO_PERSIST_S`, gel si trou â‰¤ 8 s.
- **Ã‰lÃ©ments :** oracle.
- **Relation :** `P(Ï„) â‰¥ FCO_PERSIST_S`, oÃ¹ `P` cumule les secondes de ticks de **mÃªme signe** avec marge suffisante. MesurÃ© : 6 Ã©pisodes T9 sur une session, **5 atteignent 10 s**, premiÃ¨res activations Ã  `70,3` / `101,0` / `124,0` / `139,1` / `166,8 s` ; le sixiÃ¨me Ã©pisode ne dure que **1,955 s**. Sur d'autres sessions, T10 refuse **97 et 106 secondes**. Ce que cela rÃ©vÃ¨le : Ã  10 s, le test Ã©limine l'entrÃ©e sur les sessions oÃ¹ l'Ã©ligibilitÃ© ne dure que 3 s. Test : Oui â€” une durÃ©e de persistance est symÃ©trique en signe. Contrepartie : aucune, c'est un filtre. ConsÃ©quence : le seuil doit Ãªtre abaissÃ©, ou le test fusionnÃ© avec T11.
- **Marque comportementale :** les Ã©pisodes de persistance sont **fragmentÃ©s** en dÃ©but de session et **continus** en fin â€” un Ã©pisode mesurÃ© court de `154,0` Ã  `239,8 s`, soit 86 s d'affilÃ©e.
- **Ce que le PDF proposait :** `â‰¥ 10 s cumulÃ©es` de ticks de mÃªme signe avec marge suffisante ; gel si trou â‰¤ 8 s, reset sinon.
- **Ce que les agents ont observÃ© :** Â« *T10 persistance â‰¥ 10 s â†’ **â‰¥ 2 s** : 10 s â‡’ 0 entrÃ©e en S1 (Ã©ligibilitÃ© de 3 s) ; 2 s â‡’ +2,28 $* Â» ; Â« *V1 â€” persistance 3 s : +5,34 $* Â» ; Â« *persistance â‰¥ 30 s : 100 %, PnL +1,38 $ ; â‰¥ 60 s : 100 %, +1,38 $* Â» ; Â« *PERSISTANCE : 97 et 106 refus* Â» ; Â« *`PERSISTANCE` **3 ticks** ; ne peut pas s'armer avant t = 90 s sinon* Â». n_confirmant sur l'abaissement **7** / n_mesurÃ© **9** ; contredisants **2** qui mesurent 100 % de rÃ©ussite Ã  30 s et 60 s.
- **Test de non-directionnalitÃ© :** Oui. PnL par sous-groupe : non dÃ©terminable ; les 100 % Ã  30 s et 60 s portent sur 3 issues du mÃªme cÃ´tÃ©.
- **Calibrage retenu :** `FCO_PERSIST_S = 5 s`, unitÃ© s, cumulÃ©e, mesurÃ©e sur les ticks d'oracle, horloge `Ï„`Â·marchÃ©, gel si `trou_oracle â‰¤ 8 s` et reset au-delÃ . Justification du compromis : 10 s tue l'entrÃ©e sur les sessions Ã  Ã©ligibilitÃ© courte, 2 s est un optimum in-sample sur une entrÃ©e. 5 s correspond Ã  5 ticks d'oracle, soit le minimum pour distinguer un pic d'un Ã©pisode, et reste compatible avec une fenÃªtre d'action de 90 s. **Dans la fenÃªtre `[180;270]`, ce test est peu contraignant** : les Ã©pisodes y sont continus.
- **FenÃªtre, budget, contrepartie :** 5 s d'attente avant la premiÃ¨re cotation possible ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 4 (fausses entrÃ©es) contre critÃ¨re 3 (Ã©vÃ©nements captÃ©s). 5 s dÃ©partage sans surajuster.
- **RÃ¨gle d'action 10 :** SI `P(Ï„) < 5 s` ALORS aucune cotation, code `PERSISTANCE` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ `P(Ï„) â‰¥ 5 s`, avec reset si `signe(m*)` change ou si `trou_oracle > 8 s`.

### 5.11 â€” T11 Â· `CYCLICITE`
- **CatÃ©gorie :** Comportementale.
- **RÃ´le :** **sert.** C'est le marqueur de couleur de session le plus discriminant du corpus, et il est parfaitement non directionnel.
- **Termes :** `Xâ‚†â‚€(Ï„)` = nombre de changements de signe de `m*` sur les 60 derniÃ¨res secondes, `FCO_X60_SEUIL`, `X60_FENETRE = 60 s`.
- **Ã‰lÃ©ments :** oracle, strike.
- **Relation :** `Xâ‚†â‚€(Ï„) < FCO_X60_SEUIL`. MesurÃ© : avec `Xâ‚†â‚€ < 3`, **438 instants, 99,5 %** de rÃ©ussite ; avec `Xâ‚†â‚€ = 0`, **335 instants, 100 %** de rÃ©ussite, PnL moyen +1,38 $. Sur une session, changements de signe Ã  `82, 83, 126, 127, 149, 151, 154 s` ; **279/610 passent, 331/610 Ã©chouent**. Sur une autre, l'oracle traverse le strike **8 fois**. Ce que cela rÃ©vÃ¨le : `Xâ‚†â‚€` **est** la variable de couleur de session. `Xâ‚†â‚€ = 0` signifie Â« l'oracle est dÃ©crochÃ© du strike Â», `Xâ‚†â‚€ â‰¥ 1` signifie Â« l'oracle oscille autour Â». Et `Xâ‚†â‚€` est la variable au plus fort RÂ² du corpus avec le gain : **0,207**. Test de non-directionnalitÃ© : **Oui**, et c'est structurel â€” compter des changements de signe est indÃ©pendant du signe. Contrepartie : les participants qui cotent une incertitude d'oscillation qui n'existe plus. ConsÃ©quence : durcir le seuil Ã  0.
- **Marque comportementale :** les traversÃ©es du strike sont **groupÃ©es** â€” sur une session, 7 changements de signe concentrÃ©s entre 82 et 154 s, puis plus aucun jusqu'Ã  la fin. La zone d'oscillation et la zone de dÃ©crochage sont temporellement sÃ©parÃ©es, et la seconde tombe dans la fenÃªtre d'action retenue.
- **Ce que le PDF proposait :** `Xâ‚†â‚€ < 3`, sur fenÃªtre complÃ¨te, avec journalisation et rÃ¨gle d'arrÃªt.
- **Ce que les agents ont observÃ© :** Â« *`x60 = 0` (T11 renforcÃ©) : 335 instants, **100,0 %**, PnL moyen +1,38 $* Â» contre Â« *`x60 < 3` (T11) : 438 instants, 99,5 %, +1,46 $* Â» ; Â« *x60 (**0,207**)* Â» comme meilleure corrÃ©lation du corpus ; Â« *le seuil PDF filtre une grande partie de la zone oscillante* Â». Contredisant : Â« *`CYCLICITE` : **inactif si t < 100 s** â€” a coupÃ© l'optimum S1010* Â». n_confirmant **7** / n_mesurÃ© **8** ; contredisants **1**.
- **Test de non-directionnalitÃ© :** **Oui.** PnL par sous-groupe : non dÃ©terminable au niveau du fill ; au niveau du signal, `Xâ‚†â‚€ = 0` est le seul marqueur dont la logique reste valable sous rotation UP/DOWN, puisqu'il ne rÃ©fÃ©rence aucun cÃ´tÃ©.
- **Calibrage retenu :** `Xâ‚†â‚€(Ï„) = 0`, comptage sans dimension sur la fenÃªtre glissante `]Ï„âˆ’60 ; Ï„]`, mesurÃ© Ã  chaque tick d'oracle, connu Ã  `Ï„ â‰¥ 60 s` (fenÃªtre complÃ¨te obligatoire â€” l'anomalie de `[E1]` 1.3.bis Â§ 4.2 est rÃ©solue par la fenÃªtre d'action Ã  180 s, largement postÃ©rieure Ã  60 s), horloge `Ï„`Â·marchÃ©. InterprÃ©tation : **aucune traversÃ©e du strike dans la derniÃ¨re minute**, donc l'oracle est dÃ©crochÃ©.
- **FenÃªtre, budget, contrepartie :** recalculÃ© Ã  1 Hz ; contrepartie : fournisseurs de liquiditÃ© passive, attribution non prouvÃ©e.
- **Justification par le critÃ¨re :** critÃ¨re 1 puis 4. `Xâ‚†â‚€ = 0` est le filtre qui Ã©limine les sessions Ã  oracle oscillant, c'est-Ã -dire exactement celles oÃ¹ l'Ã©cart mÃ©dian carnet/ancrage est nÃ©gatif et oÃ¹ le bot serait la contrepartie ramassÃ©e.
- **RÃ¨gle d'action 11 :** SI `Xâ‚†â‚€(Ï„) > 0` ALORS aucune cotation, annulation des ordres en vol, code `CYCLICITE` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ `Xâ‚†â‚€(Ï„) = 0`.

### 5.12 â€” T12 Â· `RETOURNEMENT` (dynamique de pente)
- **CatÃ©gorie :** **Directionnelle.**
- **RÃ´le :** **contredit.** Une borne unilatÃ©rale sur la pente sÃ©lectionne une trajectoire, ce qui est un pari sur la continuation.
- **Termes :** `[m*(Ï„) âˆ’ m*(Ï„âˆ’15)]/15`, `PENTE_DIVISEUR`, seuil `âˆ’0,40 $/s`.
- **Ã‰lÃ©ments :** oracle.
- **Relation :** `[m*(Ï„) âˆ’ m*(Ï„âˆ’15)]/15 â‰¥ âˆ’0,40 $/s`. MesurÃ© : pente P10 **âˆ’0,1177**, mÃ©diane **+0,0125**, P90 **+0,3396**, min âˆ’0,3036, max +0,4340 $/s ; **610/610 passent**. Ce que cela rÃ©vÃ¨le : le seuil ne mord jamais, mais il est **structurellement asymÃ©trique** â€” il interdit une pente nÃ©gative et autorise n'importe quelle pente positive. Test de non-directionnalitÃ© : **Non**. Si le sous-jacent avait fait le mouvement miroir, `m*` aurait la pente opposÃ©e et la rÃ¨gle n'aurait **pas** produit l'action miroir : elle aurait refusÃ©. Contrepartie : sans objet. ConsÃ©quence : rejeter, ou remplacer par une borne absolue.
- **Marque comportementale :** aucune ; le test ne s'est jamais dÃ©clenchÃ© sur les sessions instrumentÃ©es (0 refus sur 610), et 10/3/15 refus sur d'autres sessions.
- **Ce que le PDF proposait :** `â‰¥ âˆ’0,40 $/s`, homogÃ¨ne en $/s, statut `PROVISOIRE` ; Â« *on n'achÃ¨te pas une avance en train de se retourner* Â». `[E1]` 1.3.bis Â§ 2.2 avait dÃ©montrÃ© que la formule d'origine `âˆ’M(t)/30` comparait une vitesse Ã  une distance et tranchÃ© `PENTE_DIVISEUR âˆˆ E8`, en secondes.
- **Ce que les agents ont observÃ© :** Â« *interdit seulement le retournement nÃ©gatif ; cela sÃ©lectionne une trajectoire et non un retard mÃ©canique* Â» ; Â« *seuil non dÃ©clenchÃ©, mais la borne est unilatÃ©rale et directionnelle* Â» ; Â« *T12 doit Ãªtre supprimÃ©e comme entrÃ©e directionnelle ou remplacÃ©e par une garde symÃ©trique `abs(Î”m_15)/15`* Â». Contredisant : Â« *`RETOURNEMENT` : **actif (Ã  conserver absolument)** â€” a Ã©vitÃ© les 19 points perdants de S1010* Â». n_confirmant sur le rejet **6** / n_mesurÃ© **7** ; contredisants **1**.
- **Test de non-directionnalitÃ© :** **Non.** RÃ¨gle de consensus 7 appliquÃ©e : rejetÃ©e mÃªme si un agent mesure qu'elle Ã©vite des pertes, parce que ces pertes sont d'un seul cÃ´tÃ© sur un Ã©chantillon d'un seul cÃ´tÃ©.
- **Calibrage retenu :** sans objet pour la version signÃ©e. **Requalification structurelle retenue** : `|m*(Ï„) âˆ’ m*(Ï„âˆ’15)|/15 â‰¤ v_max`, garde de **stabilitÃ© symÃ©trique**, unitÃ© $/s, mesurÃ©e sur les ticks d'oracle, horloge `Ï„`Â·marchÃ©. `v_max = 0,45 $/s`, soit le max absolu observÃ© arrondi au tick supÃ©rieur (min âˆ’0,3036, max +0,4340). InterprÃ©tation : au-delÃ , l'oracle bouge trop vite pour qu'une quote passive survive Ã  sa propre naissance â€” ce qui est un fait mÃ©canique, pas une prÃ©fÃ©rence de sens.
- **FenÃªtre, budget, contrepartie :** recalculÃ© Ã  1 Hz sur un buffer de 15 s ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1. La version signÃ©e Ã©choue au test ; la version absolue le passe et conserve la protection utile.
- **RÃ¨gle 12 : hors pÃ©rimÃ¨tre (directionnelle) â€” ne pas implÃ©menter comme condition d'entrÃ©e.** Requalification structurelle : SI `|m*(Ï„) âˆ’ m*(Ï„âˆ’15)|/15 > 0,45 $/s` ALORS aucune cotation, annulation des ordres en vol, code `RETOURNEMENT` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ retour sous le seuil.

### 5.13 â€” T13 Â· `ASK_KRACH` (l'anti-couteau)
- **CatÃ©gorie :** Comportementale.
- **RÃ´le :** **sert.** Il protÃ¨ge la quote passive contre un effondrement du carnet, qui est le seul moment oÃ¹ un fill est systÃ©matiquement dÃ©favorable.
- **Termes :** `ask(Ï„)`, `mÃ©dâ‚ƒâ‚›(ask)`, `ASK_MEDIAN_DELTA`, `ASK_MEDIAN_FENETRE_S = 3 s`.
- **Ã‰lÃ©ments :** carnet.
- **Relation :** `ask(Ï„) â‰¥ mÃ©dâ‚ƒâ‚›(ask) âˆ’ ASK_MEDIAN_DELTA`. MesurÃ© : mÃ©diane rÃ©cente disponible sur 586/610 ; **497/610 passent**, 113 refusent ou sont indisponibles. Sur d'autres sessions : 434/480, 1023/1190, 593/682, 572/619, 487/681, 180/198 â€” soit **un taux de refus de 10 % Ã  28 %**. Chute d'ask mesurÃ©e : p95 **0,08 Ã  0,15**, p99 **0,13 Ã  0,29**. Ce que cela rÃ©vÃ¨le : le seuil de 0,03 est **3 Ã  10 fois plus serrÃ©** que la chute p95 rÃ©elle : il refuse la respiration normale du carnet. `[E1]` K23 l'avait anticipÃ© : Â« *0,03 = 3 ticks, soit un sixiÃ¨me du krach documentÃ© : le seuil est trÃ¨s en dessous de l'Ã©vÃ©nement Ã  bloquer, ce qui rend le filtre sÃ»r mais potentiellement coÃ»teux en faux refus* Â». Test : Oui â€” une chute d'ask est mesurÃ©e sur le cÃ´tÃ© cotÃ©, quel qu'il soit, et l'action est miroir. Contrepartie : le participant qui balaye le carnet, non identifiable. ConsÃ©quence : re-seuiller sur quantile mesurÃ©.
- **Marque comportementale :** krach documentÃ© par le PDF Ã  **0,279 s** de durÃ©e, invisible Ã  1 Hz, vu Ã  coup sÃ»r Ã  4 Hz. Autre krach mesurÃ© Ã  **4,2 s** (0,74 â†’ 0,34). Les refus T13 sont **dispersÃ©s**, avec une premiÃ¨re plage typique vers `60â€“68 s`.
- **Ce que le PDF proposait :** `ask â‰¥ mÃ©dâ‚ƒâ‚›(ask) âˆ’ 0,03`, Ã  la mesure ; plage de validitÃ© `[0,02 ; 0,05]`.
- **Ce que les agents ont observÃ© :** Â« *T13 chute d'ask (krach) â‰¥ mÃ©d. âˆ’ 0,03 : chute p95 **0,15**, p99 0,29 Â· chute p95 **0,08**, p99 0,13 â†’ valeur idÃ©ale **âˆ’0,15 Ã  âˆ’0,30*** Â» ; Â« *497/610 passent ; 113 refus ou indisponibilitÃ©s* Â» ; Â« *garde active* Â». n_confirmant **9** / n_mesurÃ© **9**.
- **Test de non-directionnalitÃ© :** **Oui.** PnL par sous-groupe : non dÃ©terminable ; mais le test est un veto, donc son PnL propre est â‰¤ 0 par construction et son bÃ©nÃ©fice est en pertes Ã©vitÃ©es.
- **Calibrage retenu :** `ASK_MEDIAN_DELTA = q95(chute d'ask sur 3 s)` **mesurÃ© en ligne sur la session en cours**, plancher 0,03 et plafond 0,30. UnitÃ© point de prix de contrat. FenÃªtre `]Ï„âˆ’3 ; Ï„]`, mesurÃ©e sur l'horloge du carnet Ã  **4 Hz minimum** â€” obligatoire, un krach de 0,279 s est invisible en dessous. Moment de connaissance : Ã  la rÃ©ception du snapshot. InterprÃ©tation : distingue la respiration du carnet d'un effondrement. Fail-closed si la mÃ©diane 3 s n'est pas calculable.
- **FenÃªtre, budget, contrepartie :** la fenÃªtre de mesure est de 3 s, l'Ã©vÃ©nement Ã  attraper dure 0,279 s, donc **cadence de 4 Hz obligatoire** ; contrepartie : agresseur non identifiable, `maker/taker` Ã  journaliser.
- **Justification par le critÃ¨re :** critÃ¨re 4. Le seuil fixe de 0,03 coÃ»te 10 Ã  28 % de refus sans preuve qu'il Ã©vite des pertes ; le quantile mesurÃ© garde la protection et rend les Ã©vÃ©nements.
- **RÃ¨gle d'action 13 :** SI `ask_cÃ´tÃ©(Ï„) < mÃ©dâ‚ƒâ‚›(ask_cÃ´tÃ©) âˆ’ q95(chute_3s)` OU si `mÃ©dâ‚ƒâ‚›` n'est pas calculable ALORS aucune cotation, annulation des ordres en vol, code `ASK_KRACH` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ retour de l'ask au-dessus du seuil pendant 2 s consÃ©cutives.

### 5.14 â€” T14 Â· `CARNET_EN_AVANCE` / `EDGE_DISPARU` (retard de phase signÃ©)
- **CatÃ©gorie :** Structurelle â€” **et rÃ©futÃ©e par la mesure.**
- **RÃ´le :** **contredit**, non pas dans son intention mais dans sa rÃ©alisation : il conditionne l'action Ã  un phÃ©nomÃ¨ne qui n'existe pas, et il vÃ©tote donc 100 % de la stratÃ©gie.
- **Termes :** `lag_signÃ©(Ï„)`, corrÃ©lation croisÃ©e `oracle â†” mid` sur fenÃªtre glissante de 10 sessions, bande `[+5 ; +60] s`.
- **Ã‰lÃ©ments :** oracle â†’ carnet.
- **Relation :** `lag_signÃ©(Ï„) âˆˆ [+5 ; +60] s`, `lag > 0` signifiant Â« l'oracle mÃ¨ne le carnet Â». **MesurÃ© : mÃ©diane des argmax de session = âˆ’0,75 s ; P10 = âˆ’28,25 s ; P90 = 0 s ; min âˆ’50 s ; max +1 s ; n = 27 sessions mesurantes ; 0/27 dans la bande ; 26/27 avec `L â‰¤ 0`.** CorrÃ©lation au pic : r = 0,25 Ã  0,56 Ã  `L = 0` ; Ã  `L = +5 s`, r = 0,014 Ã  0,092 ; Ã  `L = +16 s`, r = 0,004 Ã  0,064. Ce que cela rÃ©vÃ¨le : **l'oracle ne mÃ¨ne pas le carnet.** Le couplage est synchrone Ã  Â±2 s, et sur plusieurs sessions le carnet **prÃ©cÃ¨de** l'oracle. Confirmation indÃ©pendante par le RÂ² : `RÂ²(m*, mid_OUI) = 0,904` avec `r = +0,951` â€” le carnet est une fonction quasi affine de l'avance de l'oracle, donc **il n'y a aucune information privÃ©e dans `m*`**. Test de non-directionnalitÃ© : Oui pour la mesure (une corrÃ©lation croisÃ©e est symÃ©trique). Contrepartie : **aucune** â€” il n'y a pas d'Ã©cart temporel Ã  exploiter. ConsÃ©quence : la source d'edge nÂ°2 du cahier des charges (retards de communication oracle â†’ carnet) est **mesurÃ©e nulle**, et la dÃ©cote doit rÃ©munÃ©rer autre chose.
- **Marque comportementale :** le test refuse **543/543**, **721/721**, **299/300**, **342/342**, **181/181** secondes selon les sessions. C'est le veto dominant du corpus, avec T15.
- **Ce que le PDF proposait :** `lag_signÃ© âˆˆ [+5 ; +60] s`, corrÃ©lation croisÃ©e sur 10 sessions, plage annoncÃ©e `[3 ; 146 s]`, mÃ©diane annoncÃ©e **16,91 s** ; Â« *`CARNET_EN_AVANCE` est le test le plus rentable du bloc D : il refuse exactement les dÃ©cotes qui sont des piÃ¨ges parfaits* Â».
- **Ce que les agents ont observÃ© :** Â« *le lag mesurÃ© est de âˆ’1s / 0s / 0s, jamais dans la plage [5 ; 60] qu'ils testent. Ce sont des tests morts sur ces donnÃ©es.* Â» ; Â« *Dans la bande exigÃ©e par T14, la corrÃ©lation moyenne est **nÃ©gative** sur les trois sessions (âˆ’0,009 / âˆ’0,004 / âˆ’0,008)* Â» ; Â« *Le pic est Ã  lag 0 dans les trois sessions ; Ã  lag 16 s : r = 0,004 / 0,037 / 0,064. La prÃ©misse informationnelle (Â« le carnet n'a pas encore prix l'oracle Â») ne tient pas.* Â» ; Â« *lag croisÃ© optimal oracleâ†’carnet = âˆ’1 s, r = 0,97 en session B. Le carnet est synchrone, voire en avance. **La prÃ©misse de l'edge tombe.*** Â» ; Â« ***toutes les configurations du top 12 ont `use_T14 = False`***. Â» n_confirmant **27** / n_mesurÃ© **27** ; contredisants **0**. Plusieurs agents proposent d'Ã©largir la bande Ã  `[âˆ’2;+60]`, `[0;+60]` ou `|lag| â‰¤ 60 âˆ§ r â‰¥ 0,15`.
- **Test de non-directionnalitÃ© :** Oui pour la mesure ; **le test lui-mÃªme n'est pas directionnel, il est simplement faux**. PnL par sous-groupe : **0,00 $ dans les quatre**, puisqu'il empÃªche toute entrÃ©e.
- **Calibrage retenu :** **le test est retirÃ© de la porte d'entrÃ©e.** Les propositions d'Ã©largissement de la bande sont refusÃ©es : Ã©largir une bande jusqu'Ã  ce qu'elle contienne la mesure est une tautologie, pas un calibrage, et le paramÃ¨tre n'est de toute faÃ§on pas mesurable sur une session isolÃ©e (le PDF exige 10 sessions). `lag_signÃ©` devient un **indicateur journalisÃ© de santÃ© structurelle** : mesurÃ© par corrÃ©lation croisÃ©e sur les incrÃ©ments, fenÃªtre glissante de 10 sessions, Ã  4 Hz, horloge de capture du carnet. Seuil d'alerte, non bloquant : si la mÃ©diane glissante sur 10 sessions passe **au-dessus de +5 s**, alors le phÃ©nomÃ¨ne postulÃ© par le PDF est rÃ©apparu et le mÃ©canisme M1 doit Ãªtre **rÃ©examinÃ©** â€” car il existerait alors une latence exploitable qui changerait la nature de la dÃ©cote.
- **FenÃªtre, budget, contrepartie :** sans objet ; **contrepartie inexistante, ce qui est prÃ©cisÃ©ment le rÃ©sultat**.
- **Justification par le critÃ¨re :** critÃ¨re 1 et critÃ¨re 3. Une condition satisfaite 0 fois sur 27 sessions capte 0 Ã©vÃ©nement et a une espÃ©rance nulle.
- **RÃ¨gle 14 : hors pÃ©rimÃ¨tre â€” le phÃ©nomÃ¨ne postulÃ© n'existe pas dans les donnÃ©es (0/27 sessions).** Ne pas implÃ©menter comme condition d'entrÃ©e. Requalification structurelle : journaliser `lag_signÃ©` et son `r` Ã  chaque session ; alerte non bloquante si la mÃ©diane glissante sur 10 sessions dÃ©passe +5 s.

### 5.15 â€” T15 Â· `DECOTE_FABRIQUEE` (stabilitÃ© de la dÃ©cote)
- **CatÃ©gorie :** Comportementale.
- **RÃ´le :** **sert dans son intention, contredit dans son calibrage.** L'intention â€” ne pas prendre une dÃ©cote nÃ©e d'un effondrement â€” est juste ; le seuil la rend impossible Ã  satisfaire.
- **Termes :** saut d'ask sur 3 s, `D` confirmÃ© depuis â‰¥ 2 s.
- **Ã‰lÃ©ments :** carnet.
- **Relation :** `aucun saut d'ask > 0,01 sur ]Ï„âˆ’3 ; Ï„]` **et** `D > frais_AR` depuis â‰¥ 2 s. MesurÃ©, dÃ©composÃ© : absence de saut **161/610** ; confirmation de `D` pendant 2 s **5 snapshots** ; **intersection : 0**. Sur d'autres sessions : 29/682, 35/480, 45/619 en proxy strict. Le refus total est de **543/543** et **721/721** secondes. Saut d'ask mesurÃ© : p95 **0,10 Ã  0,20**, p99 0,13 Ã  0,29, max 0,30 ; **pas mÃ©dian rÃ©el du carnet : 2 cents**. Ce que cela rÃ©vÃ¨le : le test exige un carnet plus calme que le carnet rÃ©el, d'un facteur **10 Ã  20**. Il ne filtre pas les dÃ©cotes fabriquÃ©es, il filtre **toutes** les dÃ©cotes. Test : Oui â€” une exigence de stabilitÃ© est symÃ©trique. Contrepartie : aucune, c'est un filtre. ConsÃ©quence : re-seuiller sur quantile mesurÃ©, et dÃ©coupler les deux sous-conditions.
- **Marque comportementale :** T15 co-active avec T13 (126/610 et 252/909), ce qui confirme que les deux mesurent la mÃªme famille d'Ã©vÃ©nements de carnet. Le PDF affirme Â« *`D â‰¥ 0,10` n'apparaÃ®t que pendant les krachs de carnet* Â» ; un agent le rÃ©fute explicitement sur ses donnÃ©es.
- **Ce que le PDF proposait :** aucun saut d'ask > 1 cent sur 3 s, et `D` confirmÃ© depuis â‰¥ 2 s ; Â« *le signal le plus fiable n'est pas la dÃ©cote la plus large, c'est la dÃ©cote qui a survÃ©cu Ã  sa propre naissance* Â».
- **Ce que les agents ont observÃ© :** Â« *`CARNET_EN_AVANCE_EDGE_DISPARU` Ã©choue **181/181 secondes sur les trois sessions, soit 543/543**. C'est un veto absolu* Â» ; Â« *T15 (1Â¢/3 s) et T17 (D â‰¤ 0,114) vÃ©toient **82â€“86 %** des secondes* Â» ; Â« ***Les seuils de microstructure sont 10 Ã  20Ã— trop serrÃ©s.** T15 Ã  0,01 alors que le p95 mesurÃ© est 0,10â€“0,20 : le test refuse le marchÃ© normal.* Â» ; Â« *intersection T15 : 0 â€” aucune activation complÃ¨te* Â» ; Â« *T15 : conserver, mais il rÃ©fute ici l'affirmation `Dâ‰¥0,10 = krach`* Â». n_confirmant **11** / n_mesurÃ© **11**.
- **Test de non-directionnalitÃ© :** **Oui.** PnL par sous-groupe : **0,00 $ dans les quatre** â€” c'est le test qui, avec T14, produit le PnL nul universel.
- **Calibrage retenu :** deux conditions **dÃ©couplÃ©es**, chacune avec son propre code de refus.
  (a) **StabilitÃ© d'ask :** `max(|Î”ask|) sur ]Ï„âˆ’3 ; Ï„] â‰¤ q95(|Î”ask| sur 3 s)` mesurÃ© en ligne, plancher 0,02 (le pas mÃ©dian rÃ©el du carnet) et plafond 0,30. UnitÃ© point de contrat, horloge carnet 4 Hz.
  (b) **Confirmation de l'Ã©cart :** l'Ã©cart `e(Ï„) = |p_ancrÃ©,cÃ´tÃ©(Ï„) âˆ’ ask_cÃ´tÃ©(Ï„)|` doit dÃ©passer `frais_AR + Î´_min` pendant **2 s consÃ©cutives**. La durÃ©e de 2 s est **conservÃ©e** parce qu'elle est indÃ©pendamment justifiÃ©e par la mesure du taux de contact du BBO Ã  2 s, qui est la fenÃªtre de vie utile d'une quote (Â§ 4.6). Note : `e` remplace `D` puisque `p` n'est pas calibrÃ© (Â§ 5.16, 5.17).
- **FenÃªtre, budget, contrepartie :** 2 s de confirmation + 3 s de mÃ©moire de stabilitÃ© ; cadence 4 Hz obligatoire ; contrepartie : agresseur non identifiable.
- **Justification par le critÃ¨re :** critÃ¨re 3 puis 1. Le seuil de 0,01 capte 0 Ã©vÃ©nement sur 543+721 secondes ; le quantile mesurÃ© en capte tout en conservant la protection contre les krachs, qui est la seule protection rÃ©ellement utile pour une quote passive.
- **RÃ¨gle d'action 15 :** SI `max(|Î”ask_cÃ´tÃ©|)` sur `]Ï„âˆ’3 ; Ï„]` `> q95(|Î”ask| 3 s)` ALORS aucune cotation, code `ASK_INSTABLE` ; SI `e(Ï„) â‰¤ frais_AR + Î´_min` depuis moins de 2 s ALORS aucune cotation, code `ECART_NON_CONFIRME` â€” AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ satisfaction simultanÃ©e des deux conditions.

### 5.16 â€” T16 Â· `PAS_D_EDGE`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **sert, aprÃ¨s redÃ©finition du terme.** Un Ã©cart qui ne couvre pas les frais est une perte certaine ; mais la version du PDF le mesure avec un `p` non calibrÃ©.
- **Termes :** `D = p âˆ’ ask`, `frais_AR = 0,07Â·askÂ·(1âˆ’ask) + 0,01`.
- **Ã‰lÃ©ments :** modÃ¨le, carnet.
- **Relation :** `frais_AR < D`. MesurÃ© : `frais_AR` P10 **0,0208**, mÃ©diane **0,0231**, P90 **0,0265**, plage 0,0107 â†’ 0,0275 ; `D = p âˆ’ ask` P10 **âˆ’0,1143**, mÃ©diane **+0,1467**, P90 **+0,7192** ; **506/610 passent**. Mais la mÃ©diane de `D` par session varie de **âˆ’0,0611** et **âˆ’0,0547** (oracle oscillant) Ã  **+0,0824** (oracle dÃ©crochÃ©), et la frÃ©quence de `D > frais` va de **3,7 %** Ã  **57,7 %** des secondes. Ce que cela rÃ©vÃ¨le : la borne de frais est solide et bien mesurÃ©e ; c'est **le numÃ©rateur `p` qui n'est pas fiable**. Test de non-directionnalitÃ© : la comparaison aux frais est Oui ; `D` signÃ© est **Non** (Â§ 5.17). Contrepartie : le carnet lui-mÃªme, qui encaisse les frais. ConsÃ©quence : garder la borne, changer le terme comparÃ©.
- **Marque comportementale :** `D` mÃ©dian **nÃ©gatif** sur les sessions Ã  oracle oscillant : sur celles-lÃ , Â« *le carnet est correctement prixÃ©, voire cher, et acheter y est structurellement perdant aprÃ¨s frais* Â».
- **Ce que le PDF proposait :** `frais_AR < D`, avec `frais_AR = 0,07Â·askÂ·(1âˆ’ask) + 0,01`.
- **Ce que les agents ont observÃ© :** Â« *`D > frais` (T16) : 188 instants, 93,6 %, PnL +1,63 $* Â» ; Â« *506/610 passent ; 104 refus Ã  la borne frais ; D est proxy car K et p ne sont pas officiels* Â» ; Â« *frais_AR P50 0,017917, plage 0,010693â€“0,022768* Â» ; Â« ***T16 plancher `D` : SUPPRIMÃ‰** â€” verrouille sur les asks chers ; sans lui A2 passe de 0 $ Ã  +0,836 $* Â». n_confirmant sur le principe **9** / n_mesurÃ© **10** ; contredisants **1** sur l'emploi.
- **Test de non-directionnalitÃ© :** **Oui** dans la forme `|Ã©cart| > frais`. PnL par sous-groupe : non dÃ©terminable.
- **Calibrage retenu :** condition conservÃ©e, **terme redÃ©fini**. `e(Ï„) = |p_ancrÃ©,cÃ´tÃ©(Ï„) âˆ’ ask_cÃ´tÃ©(Ï„)| > frais_AR(ask) + Î´_min`, oÃ¹ `p_ancrÃ©,cÃ´tÃ©` est construit Ã  partir de `z` et non de Platt : `p_ancrÃ©,cÃ´tÃ©(Ï„) = Î¦(z(Ï„))` avec `Î¦` la gaussienne standard, cÃ´tÃ© dÃ©duit par miroir du signe de `m*`, et **la valeur est explicitement traitÃ©e comme un score bornÃ©, non comme une probabilitÃ© calibrÃ©e**. `frais_AR_sizing = 0,07Â·askÂ·(1âˆ’ask) + 0,01`, unitÃ© point de contrat, mesurÃ© Ã  la rÃ©ception du snapshot de carnet, horloge carnet. Valeur mÃ©diane de consensus **0,0231 pt**, plage 0,0107 â†’ 0,0275. `Î´_min = 0,01` (1 tick) au-dessus des frais.
- **FenÃªtre, budget, contrepartie :** Ã  chaque mise Ã  jour du carnet ; contrepartie : la plateforme, qui encaisse les frais, et le carnet, qui encaisse le spread.
- **Justification par le critÃ¨re :** critÃ¨re 1. C'est le plancher Ã©conomique absolu : sous les frais, l'espÃ©rance est nÃ©gative quel que soit le sens du sous-jacent.
- **RÃ¨gle d'action 16 :** SI `e(Ï„) â‰¤ frais_AR(ask_cÃ´tÃ©) + 0,01` ALORS aucune cotation, code `PAS_D_EDGE` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ `e(Ï„) > frais_AR + 0,01`.

### 5.17 â€” T17 Â· `DECOTE_ABERRANTE`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **contredit.** Il exclut par construction les instants les plus dÃ©terminÃ©s, c'est-Ã -dire exactement le mÃ©canisme que la stratÃ©gie veut exploiter.
- **Termes :** `D`, `D_max = 0,114`.
- **Ã‰lÃ©ments :** modÃ¨le, carnet.
- **Relation :** `D â‰¤ 0,114`. MesurÃ© : **371/610 snapshots ont `D > 0,114`**, soit **60,8 %** ; `D` mÃ©dian mesurÃ© **+0,224** sur une autre session, avec `p` saturÃ© Ã  une mÃ©diane de **0,9998** ; `D` atteint **0,857** et **croÃ®t en fin de session**. Refus mesurÃ©s : 198/619, 17 instants, 58 secondes selon session. Ce que cela rÃ©vÃ¨le : `Î» = 0,5513Â·Ïƒ_T` sature mÃ©caniquement `p` vers 1 quand `z` est grand, donc `D` est grand quand la dÃ©termination est grande. **T17 refuse donc prÃ©cisÃ©ment les instants oÃ¹ l'issue est le plus mÃ©caniquement dÃ©cidÃ©e.** Un agent le formule sans dÃ©tour : Â« *T17 est en contradiction mathÃ©matique avec la formule de `p` : le garde-fou rejette le signal du modÃ¨le lui-mÃªme.* Â» Test de non-directionnalitÃ© : la borne est symÃ©trique en forme, mais son effet **n'est pas neutre** : elle sÃ©lectionne les instants de faible dÃ©termination, donc de forte exposition au sens du sous-jacent. Contrepartie : sans objet. ConsÃ©quence : sortir de la porte d'entrÃ©e.
- **Marque comportementale :** l'Ã©cart carnet/ancrage **croÃ®t** avec `Ï„` : ajustement descriptif mesurÃ© `D_proxy(Ï„) = 0,04998 + 0,003358Â·Ï„` avec `RÂ² = 0,9582` sur les points mÃ©dians `Ï„ â‰¥ 180 s`. Le plafond de 0,114 est donc franchi de faÃ§on **systÃ©matique et croissante** dans la fenÃªtre d'action retenue.
- **Ce que le PDF proposait :** `D â‰¤ D_max = 0,114`, statut `PROVISOIRE` ; Â« *une dÃ©cote large ne signale pas une aubaine : elle signale que le carnet sait quelque chose que l'oracle n'a pas encore imprimÃ©* Â».
- **Ce que les agents ont observÃ© :** Â« *le plafond PDF est franchi dans **60,8 %** des snapshots proxy* Â» ; Â« *`D > 0,114` (rejetÃ© par T17) : 17 instants, 88,2 %, PnL **+1,91 $*** Â» â€” c'est-Ã -dire que **les instants rejetÃ©s par T17 ont un PnL moyen supÃ©rieur** Ã  ceux qu'il accepte (`D â‰¤ 0,114` : 93,7 %, +1,30 $) ; Â« *`D_max = 0,114` est conservÃ© comme coupe-circuit provisoire ; la session le dÃ©passe dans 371/610 observations et ne permet pas de dire qu'un dÃ©passement cesse d'Ãªtre rempli* Â». n_confirmant **10** / n_mesurÃ© **10**.
- **Test de non-directionnalitÃ© :** la borne est symÃ©trique, mais **son effet de sÃ©lection ne l'est pas**. PnL par sous-groupe : non dÃ©terminable ; le PnL agrÃ©gÃ© des instants rejetÃ©s est **supÃ©rieur** Ã  celui des instants acceptÃ©s.
- **Calibrage retenu :** **retirÃ© de la porte d'entrÃ©e. ConservÃ© comme coupe-circuit uniquement**, Ã  la valeur `D_max = 0,114`, unitÃ© point de contrat, mesurÃ© Ã  la dÃ©cision, horloge carnet, interprÃ©tation : Â« au-delÃ , le modÃ¨le et le carnet divergent tellement qu'une erreur d'implÃ©mentation est plus probable qu'un edge Â». Action du coupe-circuit : **journaliser et alerter, ne pas coter**, mais consigner l'instant pour l'analyse â€” parce que le corpus montre que ces instants sont les plus rÃ©munÃ©rateurs et que la question n'est pas tranchÃ©e.
- **FenÃªtre, budget, contrepartie :** Ã  chaque dÃ©cision ; contrepartie sans objet.
- **Justification par le critÃ¨re :** critÃ¨re 1 pour le retrait de la porte, critÃ¨re 3 pour la conservation comme coupe-circuit. Un plafond franchi dans 60,8 % des cas ne peut pas Ãªtre une condition d'entrÃ©e : il tue la stratÃ©gie. Mais un Ã©cart supÃ©rieur Ã  0,114 point sur un contrat bornÃ© Ã  1 mÃ©rite une alerte.
- **RÃ¨gle d'action 17 :** SI `e(Ï„) > 0,114` ALORS aucune cotation, journalisation renforcÃ©e de l'instant avec les 24 champs, code `DECOTE_ABERRANTE`, alerte AU MOMENT de chaque dÃ©cision JUSQU'Ã€ `e(Ï„) â‰¤ 0,114`. **Ce seuil est le premier Ã  rÃ©viser aprÃ¨s les 500 sessions de journalisation.**

### 5.18 â€” T18 Â· `PRIX_MAX`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **contredit dans la fenÃªtre tardive.** Le plafond monte lentement avec `Ï„` alors que l'ask du cÃ´tÃ© ancrÃ© monte vite : les deux courbes divergent exactement lÃ  oÃ¹ le bot doit agir.
- **Termes :** `P_max(Ï„) = min(0,70 + 0,00075Â·Ï„ ; 0,92)`, `ask`.
- **Ã‰lÃ©ments :** carnet.
- **Relation :** `ask_cÃ´tÃ©(Ï„) â‰¤ P_max(Ï„)`. MesurÃ© : `P_max` va de **0,745 Ã  0,880** sur `[60;240]`. Refus mesurÃ©s : **198/198** sur une session, **387/619**, **245/480**, **154/682**, 124, 108, 84 selon session â€” soit **20 % Ã  100 %** de refus. Le plafond de 0,92 n'est atteint qu'Ã  `Ï„ = 293,3 s`, donc **inerte** dans toute fenÃªtre d'action (`[E1]` 1.5 Â§ 4.3). Ce que cela rÃ©vÃ¨le : dans la fenÃªtre tardive, quand `z` est grand, le cÃ´tÃ© ancrÃ© est **cher par construction** â€” c'est le sens mÃªme de la dÃ©termination. Plafonner le prix revient Ã  interdire d'acheter ce qui est dÃ©terminÃ©. Test : la borne est symÃ©trique en forme ; son effet sÃ©lectionne les prix bas, donc les issues incertaines, donc l'exposition au sens. Contrepartie : sans objet. ConsÃ©quence : remplacer le plafond absolu par un test d'espÃ©rance nette.
- **Marque comportementale :** un agent mesure l'inverse sur sa session : Â« *`P_max(t)` : **conserver â€” c'est le seul rÃ©glage validÃ© par cette session** ; 228 refus, tous sur des secondes Ã  PnL â‰¤ +0,49 $ ; le plafond capture prÃ©cisÃ©ment la prime de prÃ©cocitÃ©* Â». Contradiction rÃ©elle, tranchÃ©e ci-dessous.
- **Ce que le PDF proposait :** `P_max(Ï„) = min(0,70 + 0,00075Â·Ï„ ; 0,92)`, Â« *plafond strict du prix d'achat, inchangÃ©* Â».
- **Ce que les agents ont observÃ© :** Â« *T18 : documenter qu'il bloque **198/198*** Â» ; Â« *`ask > 0,92` (rejetÃ© par T18) : 68 instants, **100,0 %** de gain, PnL +0,13 $* Â» â€” donc les instants rejetÃ©s gagnent toujours, mais rapportent des miettes ; Â« *`PRIX_MAX` ~0,85 implicite â†’ **0,72*** Â» (agent divergent, qui veut durcir) ; Â« *l'implÃ©mentation correcte consiste Ã  **remplacer le plafond de prix fixe par un test d'espÃ©rance*** Â». n_confirmant sur le remplacement **6** / n_mesurÃ© **9** ; contredisants **3**.
- **Test de non-directionnalitÃ© :** forme Oui, effet **non neutre**. PnL par sous-groupe : non dÃ©terminable ; les instants au-dessus du plafond gagnent 100 % du temps pour +0,13 $ moyen, ce qui est exactement la signature d'un actif **dÃ©terminÃ© mais peu rÃ©munÃ©rateur**.
- **Calibrage retenu :** **plafond absolu remplacÃ© par un test d'espÃ©rance nette**, qui est la forme non directionnelle du mÃªme contrÃ´le : `(1 âˆ’ ask_cÃ´tÃ©) > frais_AR_pnl(ask) + marge_securite` avec `marge_securite = 0,02` (2 ticks). UnitÃ© point de contrat, mesurÃ© Ã  la dÃ©cision, horloge carnet. InterprÃ©tation : on n'achÃ¨te que si le gain restant couvre les frais rÃ©els et deux ticks de dÃ©rapage. ConsÃ©quence arithmÃ©tique : l'ask maximal payable devient `â‰ˆ 0,96`, et la contrainte `N â‰¥ 5` impose dÃ©jÃ  `ask â‰¤ 0,98926`. **Le plafond de 0,92 et la pente de 0,00075 sont supprimÃ©s** : ils n'avaient aucune rÃ©gression pour les Ã©tayer (`[E1]` 1.6 Â§ 2.2, Â« *la valeur retenue dÃ©passe l'enveloppe qui la justifie : 0,70 contre 0,52 ; 0,92 contre 0,90* Â»).
- **FenÃªtre, budget, contrepartie :** Ã  chaque dÃ©cision ; contrepartie : la plateforme via les frais.
- **Justification par le critÃ¨re :** critÃ¨re 1. Un plafond de prix absolu sÃ©lectionne implicitement les issues incertaines. Le test d'espÃ©rance est Ã©quivalent en protection, symÃ©trique en forme, et il rend les Ã©vÃ©nements tardifs accessibles.
- **RÃ¨gle d'action 18 :** SI `(1 âˆ’ ask_cÃ´tÃ©(Ï„)) â‰¤ frais_AR_pnl(ask_cÃ´tÃ©) + 0,02` ALORS aucune cotation, code `PRIX_MAX` AU MOMENT de chaque dÃ©cision JUSQU'Ã€ satisfaction.

### 5.19 â€” T19 Â· `MODELE_NON_CALIBRE` (garde de calibration)
- **CatÃ©gorie :** MÃ©canique â€” **et rejetÃ©e comme condition d'entrÃ©e, avec un verdict tombÃ©.**
- **RÃ´le :** **sert comme garde, contredit comme porte.** La garde est juste ; le verdict qu'elle rend est nÃ©gatif ; donc la porte reste fermÃ©e.
- **Termes :** `Brier(p)`, `Brier(prix du carnet)`, coefficients de Platt `(a, b)`.
- **Ã‰lÃ©ments :** modÃ¨le, carnet.
- **Relation :** `Brier(p) â‰¤ Brier(prix du carnet)`. **MesurÃ©, et voici le point dÃ©cisif : l'agrÃ©gat passe, les sous-groupes Ã©chouent.** AgrÃ©gÃ© : modÃ¨le **0,4345**, mid du carnet **0,5334**, ask 0,5426, n = 610 â€” le modÃ¨le semble meilleur. DÃ©coupÃ© par sens : **oracle sous le strike** â†’ modÃ¨le **0,6787** contre mid **0,5543** et ask 0,5759, n = 384 ; **oracle au-dessus** â†’ modÃ¨le **0,0194** contre mid **0,4979**, n = 226. Ce que cela rÃ©vÃ¨le : l'avantage agrÃ©gÃ© du modÃ¨le vient **uniquement d'une compensation entre deux sous-groupes de sens opposÃ©**. C'est la dÃ©finition exacte d'un pari dÃ©guisÃ© en calibration. Test de non-directionnalitÃ© : **Non** â€” le modÃ¨le est meilleur d'un cÃ´tÃ© du strike et pire de l'autre. Contrepartie : sans objet. ConsÃ©quence : `p` reste un score.
- **Marque comportementale :** le PDF avait mesurÃ© la surestimation : `p` annoncÃ© **0,8348** contre `p` vraie **0,6441**, soit **+19,07 points** sur 12 sessions sans exception. Les agents la retrouvent : Â« *`p` = 0,9994 sur le cÃ´tÃ© perdant, marge annoncÃ©e +0,2160* Â» ; Â« *le modÃ¨le est moins bien calibrÃ© que le carnet dans 6 cas sur 8* Â».
- **Ce que le PDF proposait :** `Brier(p) â‰¤ Brier(prix du carnet)` ; Â« *sans calibration validÃ©e, `p` est un score, pas une probabilitÃ©* Â» ; statut `PROVISOIRE` jusqu'Ã  Ã©talonnage sur 50 sessions Ã©tiquetÃ©es.
- **Ce que les agents ont observÃ© :** Â« *l'agrÃ©gat passe par compensation, le sous-groupe bas Ã©choue ; **T19 Ã©choue au test non directionnel*** Â» ; Â« *T19 doit rester un score tant que `a,b`, l'issue officielle et le Brier par sous-groupe ne sont pas disponibles* Â» ; Â« *`p` n'est pas une probabilitÃ©, c'est un score arbitraire comparÃ© Ã  un prix, ce qui invalide le calcul de `D` et donc tout le bloc T16â€“T17* Â» ; Â« *Garde de calibration T19 â€” Ã©chec net* Â» ; Â« *le modÃ¨le est battu par le carnet* Â». n_confirmant **9** / n_mesurÃ© **9** ; contredisants **0**.
- **Test de non-directionnalitÃ© :** **Non pour `p`.** PnL par sous-groupe : c'est prÃ©cisÃ©ment la mesure qui le disqualifie â€” Brier 0,679 vs 0,554 dans un sous-groupe, 0,019 vs 0,498 dans l'autre.
- **Calibrage retenu :** **la garde est conservÃ©e et renforcÃ©e**, `p` est **exclu de toute condition d'entrÃ©e**. Forme retenue : `Brier(p) â‰¤ Brier(mid du carnet)` **dans chaque sous-groupe sÃ©parÃ©ment** (oracle au-dessus du strike / en dessous ; OUI gagne / NON gagne), sur â‰¥ 50 sessions Ã©tiquetÃ©es avec issue officielle, coefficients de Platt `(a, b)` ajustÃ©s et versionnÃ©s. Tant que cette condition n'est pas remplie : `p` est journalisÃ© comme **score de diagnostic**, et le bot utilise `z` et `Î¦(z)` comme ancrage â€” en assumant explicitement que `Î¦(z)` **n'est pas une probabilitÃ© calibrÃ©e** mais une transformation monotone de la dÃ©termination mÃ©canique.
- **FenÃªtre, budget, contrepartie :** Ã©valuation hors ligne, par lot de 50 sessions ; contrepartie sans objet.
- **Justification par le critÃ¨re :** critÃ¨re 1. Un modÃ¨le meilleur d'un cÃ´tÃ© du strike et pire de l'autre est un pari sur le cÃ´tÃ©, quelle que soit la qualitÃ© de son agrÃ©gat.
- **RÃ¨gle d'action 19 :** SI les coefficients de Platt `(a, b)` ne sont pas versionnÃ©s OU si `Brier(p) > Brier(mid)` **dans au moins un des quatre sous-groupes** ALORS `p` est journalisÃ© mais n'entre dans aucune condition d'entrÃ©e ; l'ancrage utilise `Î¦(z)` â€” AU MOMENT de chaque revue de calibration JUSQU'Ã€ validation par sous-groupe sur â‰¥ 50 sessions.

### 5.20 â€” T20 Â· `SPREAD`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **sert.** Le spread est le pÃ©age payÃ© Ã  l'instant de l'entrÃ©e ; c'est aussi le meilleur indicateur disponible de la qualitÃ© du carnet, la profondeur Ã©tant absente.
- **Termes :** `s(Ï„) = ask âˆ’ bid` du cÃ´tÃ© cotÃ©, `FCO_SPREAD_MAX`.
- **Ã‰lÃ©ments :** carnet.
- **Relation :** `s(Ï„) â‰¤ 0,04`. MesurÃ© : P10 **0,01**, mÃ©diane **0,02**, P90 **0,04 Ã  0,05**, maximum **0,17 Ã  0,19**, **minimum `âˆ’0,01`** ; **526/610 passent**, 84 refus ; sur d'autres sessions 415/480, 1055/1190, 599/682, 571/681, 538/543. Ce que cela rÃ©vÃ¨le : le seuil de 0,04 est bien placÃ© (il correspond au P90 mesurÃ©), mais **il n'a pas de borne basse** : un spread nÃ©gatif de `âˆ’0,01` signifie un carnet **croisÃ©**, donc une donnÃ©e corrompue ou un instant non appariÃ©, et il passe le test Â« Ã  tort Â». Test de non-directionnalitÃ© : **Oui** â€” le spread est mesurÃ© sur le cÃ´tÃ© cotÃ©, quel qu'il soit. Contrepartie : les fournisseurs de liquiditÃ©, qui encaissent le spread ; non identifiables. ConsÃ©quence : ajouter la borne basse et remplacer la sentinelle par un boolÃ©en.
- **Marque comportementale :** le spread est remarquablement stable â€” mÃ©diane 0,02 sur toutes les sessions, toutes couleurs confondues. C'est l'une des rares constantes rÃ©elles du corpus.
- **Ce que le PDF proposait :** `spread â‰¤ 0,04` ; Â« *sentinelle indisponible = refus, fail-closed via le bloc A* Â».
- **Ce que les agents ont observÃ© :** Â« *spread YES mÃ©dian 0,02, P90 0,05, **plage âˆ’0,01â€“0,19** ; deux spreads croisÃ©s passent Ã  tort* Â» ; Â« *T20 : ajouter rejet spread nÃ©gatif* Â» ; Â« *T20 spread YES â‰¤ 0,04 : mÃ©d. 0,01, p95 0,01, max 0,12 â†’ â‰ˆ 0,02 (p99) ; **jamais contraignant*** Â» ; Â« *`SPREAD` : **coÃ»t, pas veto** â€” RÂ² = 0,006 / 0,046, non informatif* Â». n_confirmant **8** / n_mesurÃ© **9** ; contredisants **1** qui veut en faire un coÃ»t plutÃ´t qu'un veto.
- **Test de non-directionnalitÃ© :** **Oui.** PnL par sous-groupe : non dÃ©terminable ; le spread entre dans le coÃ»t, pas dans le signal (`RÂ²(spread, gain) = 0,000`).
- **Calibrage retenu :** `s(Ï„) âˆˆ [0 ; 0,04]`, **bornes strictes des deux cÃ´tÃ©s**, unitÃ© point de contrat, mesurÃ© Ã  la rÃ©ception du snapshot de carnet, horloge carnet 4 Hz. Valeur mÃ©diane de consensus **0,02**, P90 **0,04**. DisponibilitÃ© par **boolÃ©en**, jamais par sentinelle numÃ©rique â€” c'est le correctif du *fail-open* accidentel de `[E1]` 1.3.bis Â§ 3. Le spread entre **aussi** dans le calcul de coÃ»t du Â§ 7, comme le demande l'agent contredisant : il est Ã  la fois un veto de qualitÃ© et un coÃ»t.
- **FenÃªtre, budget, contrepartie :** Ã  chaque mise Ã  jour du carnet ; contrepartie : fournisseurs de liquiditÃ©, non identifiables.
- **Justification par le critÃ¨re :** critÃ¨re 1 pour la borne basse (un carnet croisÃ© est une donnÃ©e fausse, coter dessus est une exposition pure), critÃ¨re 4 pour la borne haute.
- **RÃ¨gle d'action 20 :** SI `s(Ï„) < 0` OU `s(Ï„) > 0,04` OU si la disponibilitÃ© du spread est fausse ALORS aucune cotation, annulation, code `SPREAD` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ `s(Ï„) âˆˆ [0 ; 0,04]`.

### 5.21 â€” T21 Â· `PROFONDEUR`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **sert â€” et c'est le premier verrou de dÃ©ploiement.** Sans profondeur, le bot ne peut ni garantir son exÃ©cution ni Ã©viter de refermer lui-mÃªme l'Ã©cart.
- **Termes :** profondeur cumulÃ©e jusqu'Ã  `P_max`, `N`.
- **Ã‰lÃ©ments :** carnet (niveaux 2 Ã  12).
- **Relation :** `profondeur_cumulÃ©e(jusqu'Ã  P_max) â‰¥ N` parts. MesurÃ© : **non dÃ©terminable dans 41/41 sessions**. Aucun niveau 2, aucune quantitÃ©, aucun prix par niveau, aucune annulation. Ce que cela rÃ©vÃ¨le : la seule hypothÃ¨se implicite non vÃ©rifiÃ©e de toute la formule â€” Â« *le carnet peut absorber ma taille* Â» â€” reste non vÃ©rifiÃ©e aprÃ¨s 41 sessions d'audit. `[E1]` 1.4 l'avait Ã©tabli comme Â« trou nÂ°2 Â» et proposÃ© la condition C15 ; l'Ã‰tape 3 l'a activÃ©e en T21 ; l'Ã‰tape 4 constate que la donnÃ©e n'a jamais Ã©tÃ© livrÃ©e. Test : Oui â€” une quantitÃ© disponible est insensible au signe. Contrepartie : les autres participants qui consomment les mÃªmes niveaux ; le **gros flux** (63,2 % du notionnel en 10,2 % des Ã©vÃ©nements) est la classe de signature la plus probable. ConsÃ©quence : fail-closed obligatoire.
- **Marque comportementale :** indirectement observable â€” le spread mÃ©dian de 0,02 avec un maximum de 0,19 suggÃ¨re des retraits de liquiditÃ© ponctuels et brutaux. Le PDF documente le mÃ©canisme : Â« *le danger des chutes n'est pas dans le flux exÃ©cutÃ© mais dans le **retrait de liquiditÃ©*** Â».
- **Ce que le PDF proposait :** Â« *la profondeur cumulÃ©e jusqu'Ã  `P_max` doit couvrir la taille visÃ©e, â‰¥ N parts* Â» ; Â« *un carnet peut Ãªtre serrÃ© au spread mais vide en volume* Â».
- **Ce que les agents ont observÃ© :** Â« *aucun niveau 2, aucune quantitÃ©, aucun prix de niveau ; valeur non dÃ©terminable â†’ **refus fail-closed obligatoire*** Â» ; Â« *Impossible de mesurer profondeur consommÃ©e jusqu'Ã  `P_max`* Â» ; Â« *T21/T22 : fail-closed sans depth/inventaire* Â». n_confirmant **6** / n_mesurÃ© **6** ; contredisants **0**.
- **Test de non-directionnalitÃ© :** **Oui.** PnL par sous-groupe : non dÃ©terminable â€” et c'est structurel, puisque sans profondeur il n'y a pas de fill, donc pas de PnL.
- **Calibrage retenu :** **Non dÃ©terminable avec les analyses fournies.** Champ Ã  journaliser : profondeur des **12 meilleurs niveaux** du cÃ´tÃ© cotÃ©, sous forme `(prix, quantitÃ©)` par niveau, plus les annulations. Ã‰lÃ©ment source : carnet CLOB, flux WebSocket niveau 2. Horloge : capture du carnet, prÃ©cision requise **â‰¤ 100 ms**, cadence **4 Hz minimum**. Ce que cela permettrait de trancher : la taille maximale exÃ©cutable sans dÃ©rapage, donc la capacitÃ© du mÃ©canisme M1, donc la seule question Ã©conomique qui reste ouverte.
- **FenÃªtre, budget, contrepartie :** Ã  chaque mise Ã  jour du carnet ; contrepartie : gros flux, attribution non prouvÃ©e.
- **Justification par le critÃ¨re :** critÃ¨re 1. Sans profondeur, la rÃ©serve de dÃ©rapage `SLIPPAGE_SIZING = 0,01` n'est provisionnÃ©e par rien, et `[E1]` 1.4 Â§ 3 montre qu'un ordre de 8 parts sur un niveau de 2 parts consomme 4 niveaux, pour un dÃ©rapage de plusieurs cents.
- **RÃ¨gle d'action 21 :** SI la profondeur cumulÃ©e du cÃ´tÃ© cotÃ© jusqu'au prix limite est absente OU `< floor(N)` parts ALORS aucune cotation, code `PROFONDEUR` AU MOMENT de chaque dÃ©cision JUSQU'Ã€ disponibilitÃ© du niveau 2 et satisfaction du seuil. **En l'Ã©tat des donnÃ©es, ce test est fail-closed permanent : le bot ne cote pas.**

### 5.22 â€” T22 Â· `OCCUPE` / `PARTS_MIN`
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **sert.** Empiler deux positions transforme deux mÃ©canismes indÃ©pendants en un pari corrÃ©lÃ©.
- **Termes :** `position`, `n_ordres_en_vol`, `N = C/(ask + 0,07Â·askÂ·(1âˆ’ask) + 0,01)`, `PARTS_MIN = 5`, `C = 5,00 $`.
- **Ã‰lÃ©ments :** Ã©tat interne du bot, moteur d'appariement.
- **Relation :** `position = âˆ…` **et** `n_ordres_en_vol = 0` **et** `floor(N) â‰¥ 5`. MesurÃ©, partie sizing : `N` P10 **6,09**, mÃ©diane **6,76**, P90 **17,64**, min 5,51, max 31,56 ; **`floor(N) â‰¥ 5` sur 610/610**. Partie occupation : **non dÃ©terminable dans 41/41 sessions** â€” ni `position`, ni `n_ordres_en_vol` n'existent dans les donnÃ©es, alors que le thÃ©orÃ¨me d'entrÃ©e du PDF les consomme. `[E1]` 1.4 l'avait Ã©tabli comme Â« trou nÂ°4 Â» : Â« *le PDF utilise dans son thÃ©orÃ¨me deux grandeurs que son propre inventaire ne dÃ©clare pas* Â». Ce que cela rÃ©vÃ¨le : la contrainte `N â‰¥ 5` avec `C = 5,00 $` impose arithmÃ©tiquement `ask â‰¤ 0,98926`, ce qui est une **consÃ©quence, pas un choix**. Test : Oui â€” l'Ã©tat interne est indÃ©pendant du sens du marchÃ©. Contrepartie : aucune. ConsÃ©quence : fail-closed sur l'occupation.
- **Marque comportementale :** dans la fenÃªtre d'action `[180;270]`, les asks du cÃ´tÃ© ancrÃ© sont hauts par construction, donc `N` est **petit** â€” typiquement 6 Ã  8 parts. Le PnL par Ã©vÃ©nement est donc de l'ordre de `N Ã— (1 Ã  2 ticks)`, soit **0,06 Ã  0,16 $**. C'est le chiffre Ã©conomique central du livrable et il est mince.
- **Ce que le PDF proposait :** inventaire vide rÃ©conciliÃ©, 0 ordre en vol, position nulle ; sizing minimum 5 parts ; Â« *une position fantÃ´me non rÃ©conciliÃ©e vaut refus* Â». La branche demi-taille est Ã©tablie comme **arithmÃ©tiquement vide** par `[E1]` 1.6 Â§ 4.3 (`CAPITAL_DEMI = 2,50 $` ne peut acheter 5 parts que sur 21 s sur 195).
- **Ce que les agents ont observÃ© :** Â« *sizing visible : `N` P10 6,08865, mÃ©diane 6,76379 [...] `floor(N) â‰¥ 5` sur 610/610. **Position et ordres en vol absents*** Â» ; Â« *seule la sous-condition de sizing passe ; occupation/rÃ©conciliation ND* Â» ; Â« *`PARTS_MIN` > 9 â†’ **6** â€” a bloquÃ© 96 points S1010 dont l'optimum* Â» ; Â« *`N = floor(budget/ask)` â€” fait Ã—4,3 le PnL S1005 Ã  capital constant* Â». n_confirmant **7** / n_mesurÃ© **7** sur le sizing ; **0/41** sur l'occupation.
- **Test de non-directionnalitÃ© :** **Oui.** PnL par sous-groupe : non dÃ©terminable.
- **Calibrage retenu :** `C = 5,00 $` **inchangÃ©** (le conflit 5,00 $ contre 2,00 $ signalÃ© en `[E1]` 1.6 Â§ 6 est tranchÃ© en faveur du PDF, source la plus rÃ©cente). `PARTS_MIN = 5` **inchangÃ©** : c'est une contrainte de plateforme, pas un rÃ©glage. `N = floor(C/(ask + 0,07Â·askÂ·(1âˆ’ask) + 0,01))`, sans dimension, calculÃ© Ã  la dÃ©cision. **Branche demi-taille supprimÃ©e** et `RATIO_PLEIN` supprimÃ©, conformÃ©ment Ã  la recommandation (a) de `[E1]` 1.6 Â§ 4.3 : la modulation de taille n'a jamais existÃ© dans les faits. **Une seule position par session, une seule cotation en vol Ã  la fois.** `position` et `n_ordres_en_vol` : **fail-closed tant que la rÃ©conciliation API n'est pas implÃ©mentÃ©e**.
- **FenÃªtre, budget, contrepartie :** Ã  chaque dÃ©cision ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1. Deux positions simultanÃ©es sur le mÃªme sous-jacent sont parfaitement corrÃ©lÃ©es, donc la seconde n'ajoute pas d'edge mais double l'exposition au sens.
- **RÃ¨gle d'action 22 :** SI `position â‰  âˆ…` OU `n_ordres_en_vol > 0` OU `floor(N) < 5` OU si la rÃ©conciliation de position n'est pas disponible ALORS aucune cotation, code `OCCUPE` ou `PARTS_MIN` selon la sous-condition violÃ©e AU MOMENT de chaque dÃ©cision JUSQU'Ã€ inventaire vide rÃ©conciliÃ© et `floor(N) â‰¥ 5`.

---
# 6. Calibrage final de toutes les variables et constantes

| Variable ou constante | Fonction(s) | Valeur PDF | **Valeur retenue** | UnitÃ© | Moment de mesure | Moment de connaissance | Horloge | InterprÃ©tation | n_conf / n_mes | Sources |
|---|---|---|---|---|---|---|---|---|---|---|
| `Ï„â‚€` (origine) | T6, toutes | `endDate âˆ’ 300 s` | `endDate âˆ’ 300 s`, **bloquant** | s absolues | 1Ã— Ã  l'ouverture | Ã€ l'ouverture, â‰¤ 100 ms | MarchÃ© absolue | Origine imposÃ©e de l'extÃ©rieur | 6/6 | PDF p.14, A2G |
| `Ï„` | T8, toutes | `[0 ; 300]` | InchangÃ© | s | Continu | ImmÃ©diat aprÃ¨s `Ï„â‚€` | `Ï„`Â·marchÃ© | Horloge de risque | 41/41 | PDF, A2G |
| `K` | T7, `m*` | TWAP 30 s officiel | Officiel obligatoire, **aucun proxy** | $ | 1Ã— Ã  `Ï„â‚€` | Ã€ l'ouverture | MarchÃ© | Pivot de rÃ©solution | 8/8 | PDF p.4, A1G, A2G |
| `k_source` | T7 | `officiel` | InchangÃ©, bloquant | BoolÃ©en | 1Ã— Ã  `Ï„â‚€` | Ã€ l'ouverture | â€” | Certificat d'authenticitÃ© | 8/8 | PDF, A2G |
| `K_ECART_MAX_USD` | T7 | 5,00 $ | **SupprimÃ©** | â€” | â€” | â€” | â€” | Une tolÃ©rance Ã©gale Ã  `M(240)` annule le signal minimal | 3/3 | E1 1.5 Â§3.3 |
| `STALENESS_MAX` | T1 | 2,5 s | **2,5 s** (conservÃ©) | s | Ã€ la dÃ©cision | RÃ©ception du snapshot | Capture oracle | Ã‚ge maximal de l'ancrage | 7/7 (max obs. 4,0 s) | PDF p.6, A1G, A2G |
| `GEL_MAX_ORACLE` | T2 | 8 s | **8 s** | s | Continu | RÃ©ception | Capture oracle | Trou de flux oracle | 8/8 | PDF, A2G |
| `GEL_MAX_CARNET` | T2 | *absent* | **3 s** (nouveau) | s | Continu | RÃ©ception | Capture carnet | Trou de flux carnet ; P90 mesurÃ© 1,19 s | 8/8 | A2G, `[R]` |
| DisponibilitÃ© des champs | T3, T20 | Sentinelle `âˆ’1` | **BoolÃ©en, fail-closed** | BoolÃ©en | Chaque snapshot | RÃ©ception | Capture carnet | Corrige le *fail-open* accidentel | 6/6 | E1 1.3.bis Â§3 |
| **`Ïƒâ‚ƒâ‚€`** | T4, `Ïƒ_T` | **26,50 $ (constante)** | **MesurÃ© en ligne. MÃ©diane des mÃ©dianes de consensus 7,20 $** ; P10 2,64 / P90 9,94 ; bornes de plausibilitÃ© `[0,5 ; 40]` | $ | `Ï„ â‰¥ 30 s` puis chaque tick | `Ï„ â‰¥ 30 s` | `Ï„`Â·marchÃ© | Dispersion du chemin restant | **29/29** | A1G, A2G, PDF p.15 |
| Estimateur de `Ïƒâ‚ƒâ‚€` | T4 | Non tranchÃ© | **Ã‰cart-type des incrÃ©ments d'oracle Ã  1 s Ã— âˆš30** (estimateur A) | â€” | `]Ï„âˆ’30 ; Ï„]` | Idem | `Ï„`Â·marchÃ© | La variance des incrÃ©ments s'additionne ; c'est la loi en âˆš | 21 vs 8 | A1G, A2G |
| `Ïƒ_T(Ï„)` | Tout le systÃ¨me | `Ïƒâ‚ƒâ‚€Â·âˆš((300âˆ’Ï„)/30)` | **InchangÃ©** | $ | Chaque seconde | AprÃ¨s `Ïƒâ‚ƒâ‚€` | `Ï„`Â·marchÃ© | Mouvement encore possible | 41/41 | PDF p.15 |
| `k_Î»` | `Î»` | 0,5513 = âˆš3/Ï€ | **0,5513 â€” identitÃ©, non rÃ©glage** | Sans dim. | â€” | â€” | â€” | Ã‰galise variance logistique et gaussienne | 5/7 (2 contredisants) | PDF p.16, A1G |
| `Î»(Ï„)` | `p` | `0,5513Â·Ïƒ_T` | InchangÃ© ; mÃ©diane mesurÃ©e 3,27 $, P90 10,89 $ | $ | Chaque seconde | AprÃ¨s `Ïƒâ‚ƒâ‚€` | `Ï„`Â·marchÃ© | Ã‰chelle de bruit | 5/7 | PDF, A2G |
| `k_M` | T9 | 0,3439 | **RemplacÃ© par `z_min`** | â€” | â€” | â€” | â€” | Exprimer le seuil en probabilitÃ© est une fausse prÃ©cision | 9/12 | A1G, A2G, `[R]` |
| **`z_min`** | T9 | *(Ã©quiv. 0,3439)* | **1,0 â€” statut `PROVISOIRE`** | Sans dim. | Chaque tick d'oracle | AprÃ¨s `Ïƒâ‚ƒâ‚€` | `Ï„`Â·marchÃ© | Il faut 1 Ïƒ du chemin restant pour renverser l'issue | 9/12 | A1G, A2G, `[R]` |
| `M(Ï„)` | T9 | `0,3439Â·Ïƒ_T` | `z_minÂ·Ïƒ_T` = 14,40 $ Ã  180 s, 10,18 $ Ã  240 s, 7,20 $ Ã  270 s | $ | Chaque seconde | AprÃ¨s `Ïƒâ‚ƒâ‚€` | `Ï„`Â·marchÃ© | Barre de dÃ©termination | 9/12 | `[R]` |
| `p_brut`, `p` (Platt) | T16â€“T19 | Logistique + Platt | **Score de diagnostic, hors porte d'entrÃ©e** | Sans dim. | Chaque seconde | AprÃ¨s `Ïƒâ‚ƒâ‚€` | `Ï„`Â·marchÃ© | Non calibrÃ© : Brier Ã©choue par sous-groupe | 9/9 | A2G, PDF p.9 |
| `p_ancrÃ©,cÃ´tÃ©(Ï„)` | M1 | *absent* | **`Î¦(z(Ï„))`**, cÃ´tÃ© = miroir du signe de `m*` ; score bornÃ©, **jamais une probabilitÃ©** | Point de contrat | Chaque tick d'oracle | AprÃ¨s `Ïƒâ‚ƒâ‚€` | `Ï„`Â·marchÃ© | Prix de rÃ©fÃ©rence du bot | â€” | `[R]` |
| `FCO_PERSIST_S` | T10 | 10 s | **5 s** | s | CumulÃ©e sur ticks d'oracle | Continu | `Ï„`Â·marchÃ© | 5 ticks pour distinguer un pic d'un Ã©pisode | 7/9 | A1G, A2G, `[R]` |
| `FCO_X60_SEUIL` | T11 | `< 3` | **`= 0`** | Comptage | `]Ï„âˆ’60 ; Ï„]` | `Ï„ â‰¥ 60 s` | `Ï„`Â·marchÃ© | Aucune traversÃ©e du strike dans la derniÃ¨re minute | 7/8 | A1G (335 pts, 100 %) |
| `X60_FENETRE` | T11 | 60 s | **60 s** (inchangÃ© ; l'anomalie de fenÃªtre incomplÃ¨te disparaÃ®t avec `Ï„_min = 180 s`) | s | Glissante | `Ï„ â‰¥ 60 s` | `Ï„`Â·marchÃ© | FenÃªtre de cyclicitÃ© | 8/8 | PDF, E1 1.3.bis Â§4.2 |
| `PENTE_DIVISEUR` | T12 | 30, classÃ© E12 | **SupprimÃ©** (condition rejetÃ©e) | â€” | â€” | â€” | â€” | TranchÃ© E8 en s par E1 1.5, puis rendu inutile par le rejet de T12 | 6/7 | E1 1.5 Â§4.1, A2G |
| **`v_max`** | T12 requal. | *absent* | **0,45 $/s** sur `|Î”m*|/15` | $/s | Buffer 15 s | Chaque tick d'oracle | `Ï„`Â·marchÃ© | Garde de stabilitÃ© **symÃ©trique** | 6/7 | A2G, `[R]` |
| `ASK_MEDIAN_FENETRE_S` | T13 | 3,0 s | **3,0 s** | s | Glissante | RÃ©ception | Capture carnet | MÃ©moire d'anti-couteau | 9/9 | PDF, E1 K23 |
| `ASK_MEDIAN_DELTA` | T13 | 0,03 | **`q95(chute d'ask 3 s)` mesurÃ© en ligne**, plancher 0,03, plafond 0,30 | Point de contrat | `]Ï„âˆ’3 ; Ï„]` | RÃ©ception | Capture carnet, **4 Hz min** | Distingue respiration et effondrement ; p95 rÃ©el 0,08â€“0,15 | 9/9 | A1G, A2G |
| `lag_signÃ©` | T14 | `[+5 ; +60] s`, mÃ©diane 16,91 s | **RetirÃ© de la porte. JournalisÃ©. MÃ©diane de consensus âˆ’0,75 s, P10 âˆ’28,25, P90 0, 0/27 dans la bande** | s | CorrÃ©lation croisÃ©e, 10 sessions | Hors ligne | Capture carnet, 4 Hz | Le carnet ne suit pas l'oracle : il est synchrone | **27/27** | A1G, A2G |
| Seuil d'alerte `lag` | T14 requal. | â€” | **Alerte non bloquante si mÃ©diane glissante 10 sessions > +5 s** | s | Par lot | Hors ligne | Capture carnet | RÃ©apparition du phÃ©nomÃ¨ne postulÃ© | â€” | `[R]` |
| Saut d'ask max sur 3 s | T15 (a) | 0,01 | **`q95(|Î”ask| 3 s)` mesurÃ© en ligne**, plancher 0,02, plafond 0,30 | Point de contrat | `]Ï„âˆ’3 ; Ï„]` | RÃ©ception | Capture carnet, 4 Hz | Le pas mÃ©dian rÃ©el du carnet est 2 cents | 11/11 | A1G, A2G |
| DurÃ©e de confirmation de l'Ã©cart | T15 (b) | 2 s | **2 s (conservÃ©)** | s | Glissante | RÃ©ception | Capture carnet | IndÃ©pendamment justifiÃ© par la fenÃªtre de contact du BBO Ã  2 s | 11/11 | PDF p.17, A2G |
| `frais_AR_sizing` | T16, T22 | `0,07Â·ask(1âˆ’ask) + 0,01` | **InchangÃ©.** MÃ©diane mesurÃ©e 0,0231 pt, plage 0,0107â€“0,0275 | Point de contrat | Ã€ la dÃ©cision | RÃ©ception carnet | Capture carnet | Plancher Ã©conomique conservateur | 9/10 | PDF p.11, A2G |
| `frais_AR_pnl` | PnL, T18 | `0,02Â·ask(1âˆ’ask)` | **InchangÃ©** | Point de contrat | Ã€ la clÃ´ture | â€” | â€” | Invariant de frais rÃ©el | 4/4 | PDF p.11 |
| `e(Ï„)` (Ã©cart) | T16, T17, M1 | `D = p âˆ’ ask` | **`|p_ancrÃ©,cÃ´tÃ©(Ï„) âˆ’ ask_cÃ´tÃ©(Ï„)|`** | Point de contrat | Chaque snapshot carnet | RÃ©ception | Capture carnet | Ã‰cart carnet âˆ’ ancrage, en valeur absolue | â€” | `[R]` |
| `Î´_min` | T16, M1 | *absent* | **`frais_AR + 0,01`** (1 tick au-dessus des frais) | Point de contrat | Ã€ la dÃ©cision | RÃ©ception | Capture carnet | DÃ©cote plancher | 4/4 | A2G |
| `D_max` | T17 | 0,114, condition d'entrÃ©e | **0,114, coupe-circuit seulement** | Point de contrat | Ã€ la dÃ©cision | RÃ©ception | Capture carnet | DÃ©passÃ© dans 60,8 % des snapshots ; ne peut pas Ãªtre une porte | 10/10 | A1G, A2G |
| `P_max` ordonnÃ©e / pente / plafond | T18 | 0,70 / 0,00075 / 0,92 | **Les trois supprimÃ©s** | â€” | â€” | â€” | â€” | Aucune rÃ©gression ne les Ã©taye ; le plafond est inerte (atteint Ã  293,3 s) | 6/9 | E1 1.5 Â§4.3, A2G |
| `marge_securite` | T18 requal. | *absent* | **0,02** (2 ticks) sur `(1 âˆ’ ask) > frais_AR_pnl + marge` | Point de contrat | Ã€ la dÃ©cision | RÃ©ception | Capture carnet | Test d'espÃ©rance nette remplaÃ§ant le plafond absolu | 6/9 | A1G, `[R]` |
| `FCO_SPREAD_MAX` | T20 | 0,04 | **`s âˆˆ [0 ; 0,04]`, bornes strictes** | Point de contrat | Chaque snapshot | RÃ©ception | Capture carnet | MÃ©diane mesurÃ©e 0,02 ; plage observÃ©e âˆ’0,01 Ã  0,19 | 8/9 | A1G, A2G |
| Profondeur cumulÃ©e | T21 | `â‰¥ N` parts, 12 niveaux | **Non dÃ©terminable â€” fail-closed permanent** | Parts | Chaque snapshot | â€” | Capture carnet, â‰¤ 100 ms | 0/41 sessions contiennent le niveau 2 | 6/6 | A2G, E1 1.4 |
| `C` (capital) | T22 | 5,00 $ | **5,00 $** | $ | Ã€ la dÃ©cision | â€” | â€” | Conflit 5,00 / 2,00 $ tranchÃ© en faveur du PDF | 7/7 | PDF p.11, E1 1.6 Â§6 |
| `PARTS_MIN_CLOB` | T22 | 5 | **5** | Parts | Ã€ la dÃ©cision | â€” | â€” | Contrainte de plateforme ; impose `ask â‰¤ 0,98926` | 7/7 | PDF p.11 |
| `N` | T22, sizing | `C/(ask + frais + 0,01)` | **InchangÃ©**, `floor`. MÃ©diane mesurÃ©e 6,76 ; P10 6,09 ; P90 17,64 | Parts | Ã€ la dÃ©cision | RÃ©ception carnet | Capture carnet | Taille unique | 7/7 | PDF, A2G |
| `CAPITAL_DEMI`, `RATIO_PLEIN` | Sizing | 2,50 $ / 1,5 | **SupprimÃ©s** | â€” | â€” | â€” | â€” | Branche arithmÃ©tiquement vide (21 s sur 195) | 3/3 | E1 1.6 Â§4.3 |
| `position`, `n_ordres_en_vol` | T22 | Inventaire vide | **Non dÃ©terminable â€” fail-closed** | Ã‰tat | Continu | RÃ©conciliation API | Locale + serveur | 0/41 sessions | 6/6 | E1 1.4 trou nÂ°4, A2G |
| `Î¦â‚ƒâ‚€` et `FCO_FLUX_VETO_USD` | Veto de flux | 800 $ | **Non dÃ©terminable â€” retirÃ©** | â€” | â€” | â€” | â€” | `direction` est un code Â±1 **sans dictionnaire** ; le cÃ´tÃ© agresseur n'est pas reconstructible | 4/4 | A1G, A2G |
| `w(Ï„)` | RÃ©fÃ©rentiel | `min(1 ; max(0 ; (270âˆ’Ï„)/30))` | **InchangÃ©, mais devient actif** dans `[240;270]` | Sans dim. | Chaque seconde | ImmÃ©diat | `Ï„`Â·marchÃ© | Glissement vers le TWAP de rÃ¨glement | 3/3 | PDF p.4, E1 1.5 Â§4.2 |
| `TWAP30p` | `m*` via `w` | Moyenne temporelle 30 s | **Devient actif** dans `[240;270]` | $ | `]Ï„âˆ’30 ; Ï„]` | AprÃ¨s 30 s de buffer | `Ï„`Â·marchÃ© | RÃ©fÃ©rentiel rÃ©el du rÃ¨glement | 3/3 | PDF, E1 |
| **`Ï„_min` fenÃªtre d'action** | T8 | 60 s | **180 s** | s | â€” | ImmÃ©diat | `Ï„`Â·marchÃ© | En dessous, le mouvement rÃ©siduel de l'oracle (11,62 $ = 1,6 Ïƒâ‚ƒâ‚€) dÃ©passe toute dÃ©cote | 8/12 | A1G, A2G, `[R]` |
| **`Ï„_max` fenÃªtre d'action** | T8 | 240 s | **270 s** | s | â€” | ImmÃ©diat | `Ï„`Â·marchÃ© | Au-delÃ , le carnet cesse de publier (~273 s) et le rÃ¨glement s'ouvre | 8/12 | A1G, A2G |
| DurÃ©e de validitÃ© de la quote | M1 | *absent* | **2 s**, ou expiration au P90 de latence si mesurÃ© | s | Ã€ l'envoi | â€” | Capture carnet | FenÃªtre de contact du BBO mesurÃ©e | 4/4 | A2G |
| Cadence d'Ã©valuation | Toutes | 1 Hz oracle / 4 Hz carnet | **Union des Ã©vÃ©nements oracle et carnet, 4 Hz minimum sur le carnet** | Hz | Continu | â€” | Capture carnet | Cadence carnet mesurÃ©e 0,013 s ; krach de 0,279 s invisible Ã  1 Hz | 6/6 | PDF p.16, E1 1.3.bis Â§4.1 |
| Tick | Prix | 0,01 | **0,01**, avec rÃ©serve : **infÃ©rÃ© du BBO, pas un champ de schÃ©ma** | Point de contrat | â€” | â€” | â€” | Ã€ confirmer par l'API | 6/6 | A2G |
| Basis de rÃ©fÃ©rence | T5 requal. | `spot > oracle` | **`|basis âˆ’ mÃ©diane_glissante_60s| â‰¤ 3Â·Ïƒ_session`.** MÃ©diane de consensus +42,1 $, plage 22,90â€“53,32 | $ | Chaque ligne spot, appariÃ©e `as-of` | RÃ©ception | Capture spot | ContrÃ´le de cohÃ©rence de source | 12/12 | A1G, A2G |
| Invariant de complÃ©mentaritÃ© | M2 | *absent* | **`|bid_OUI + ask_NON âˆ’ 1| â‰¤ 1 tick`**, sinon fail-closed | Point de contrat | Chaque snapshot | RÃ©ception | Capture carnet | Exact sur 1025/1025 | 6/6 | A2G |
| `HALT` | T5 | MarchÃ© non en HALT | **Non dÃ©terminable â€” champ Ã  journaliser** | BoolÃ©en | Ã‰vÃ©nementiel | Serveur | Serveur, ms | 0/41 sessions | 6/6 | A2G |
| `fee_bps` rÃ©el | Garde de frais | `â‰¤ 200 bps`, garde de prÃ©-vol | **Non dÃ©terminable â€” champ Ã  journaliser.** Garde conservÃ©e : `fee_bps/10â´Â·min(ask;1âˆ’ask) â‰¤ 0,02Â·askÂ·(1âˆ’ask)`, sinon HALT | bps | PrÃ©-vol + rotation | Avant chaque ordre | â€” | Ã€ `ask = 0,65` la garde plafonne Ã  130 bps : elle est active | 2/2 | E1 O11, PDF ch.7 |

---

# 7. SpÃ©cifications d'implÃ©mentation des mÃ©canismes retenus

### 7.1 â€” M2 Â· ContrÃ´le d'intÃ©gritÃ© par invariants croisÃ©s
*(spÃ©cifiÃ© en premier parce qu'il conditionne les deux autres)*

- **CatÃ©gorie :** Structurelle.
- **Ã‰vÃ©nement dÃ©clencheur :** chaque mise Ã  jour du carnet **ou** de l'oracle **ou** du spot.
- **DonnÃ©es d'entrÃ©e :**

| Champ | Ã‰lÃ©ment source | FrÃ©quence | Horloge et prÃ©cision requise | Dans les rapports | Dans le schÃ©ma |
|---|---|---|---|---|---|
| `yes_bid`, `yes_ask`, `no_bid`, `no_ask` | Carnet | 4 Hz min | Capture carnet, 1 ms | Oui | Oui |
| `oracle.price`, `oracle.t_ms` | Oracle | 1 Hz | Capture oracle, 1 ms | Oui | Oui |
| `spot.price`, `spot.t_ms` | Spot | Ã‰vÃ©nementiel | Capture spot, 1 ms | Oui | Oui |
| `endDate`, `K`, `k_source` | Officiel | 1Ã— | MarchÃ© absolue, â‰¤ 100 ms | Non | **Ã€ vÃ©rifier** |
| `HALT` | Plateforme | Ã‰vÃ©nementiel | Serveur, ms | Non | **Ã€ vÃ©rifier** |

- **Calcul :**
```
fonction integrite_ok(tau):
    si endDate absent ou k_source != "officiel":      retourner FAUX, "T0_APPROXIME|K_NON_OFFICIEL"
    si un des 4 champs BBO absent:                    retourner FAUX, "SPREAD_INDISPO"
    si |yes_bid + no_ask - 1| > 0.01:                 retourner FAUX, "INVARIANT_ROMPU"
    si |yes_ask + no_bid - 1| > 0.01:                 retourner FAUX, "INVARIANT_ROMPU"
    age_oracle = tau - t_dernier_tick_oracle
    si age_oracle > 2.5:                              retourner FAUX, "TICK_PERIME"
    si trou_oracle > 8.0:                             retourner FAUX, "TICK_TROU"
    si trou_carnet > 3.0:                             retourner FAUX, "TICK_TROU"
    basis = spot_asof(tau) - oracle_last(tau)
    si |basis - mediane_glissante_60s(basis)| > 3 * sigma_session(basis):
                                                      retourner FAUX, "BASIS_CORROMPU"
    si HALT absent ou HALT == vrai:                   retourner FAUX, "HALT"
    s = ask_cote - bid_cote
    si s < 0 ou s > 0.04:                             retourner FAUX, "SPREAD"
    si sigma_30(tau) hors [0.5, 40] ou fenetre 30 s incomplete:
                                                      retourner FAUX, "SIGMA_INDISPO"
    retourner VRAI, ""
```
- **FenÃªtre d'exploitabilitÃ© :** instantanÃ©e. Budget requis : celui du traitement d'un snapshot, `< 1 ms`. Part des Ã©vÃ©nements atteignables : **100 %**, le mÃ©canisme est purement local.
- **Action :** aucune position. Suspension de M1/M3 et **annulation immÃ©diate** de tout ordre en vol.
- **Sortie et invalidation :** reprise dÃ¨s que `integrite_ok` renvoie vrai. Aucun dÃ©lai maximal.
- **CoÃ»ts :** frais nuls, aucun slippage. PnL attendu par Ã©vÃ©nement : **0,00 $**. Ce mÃ©canisme ne gagne pas d'argent, il en Ã©vite la perte.
- **CapacitÃ© et impact :** sans objet, aucune position.
- **Cas dÃ©gradÃ©s :** oracle muet â†’ `TICK_TROU` ; carnet vide au meilleur niveau â†’ `SPREAD_INDISPO` ; dÃ©synchronisation d'horloge dÃ©tectÃ©e par un `basis` hors bande â†’ `BASIS_CORROMPU` ; rejet d'ordre â†’ traitÃ© en 7.2.
- **Contrepartie :** aucune. Garde-fou.
- **Test de non-directionnalitÃ© :** **Oui.** Un refus est symÃ©trique par construction ; PnL 0,00 $ dans les quatre sous-groupes.
- **Test d'implÃ©mentabilitÃ© :** **Oui.**

### 7.2 â€” M3 Â· Instrumentation de mesure du taux de remplissage
*(Ã  exÃ©cuter avant M1 en taille rÃ©elle â€” c'est le mÃ©canisme qui produit les donnÃ©es manquantes)*

- **CatÃ©gorie :** MÃ©canique.
- **Ã‰vÃ©nement dÃ©clencheur :** identique Ã  M1 (Â§ 7.3), mais la taille est fixÃ©e au **minimum rÃ©glementaire** et l'objectif est la mesure, pas le gain.
- **DonnÃ©es d'entrÃ©e :** celles de 7.3, **plus** les huit horodatages d'exÃ©cution (Â§ 8.6), qui n'existent dans aucune session et que ce mÃ©canisme a prÃ©cisÃ©ment pour but de crÃ©er.
- **Calcul :**
```
pour chaque delta dans [0.01, 0.02, 0.03, 0.05, 0.08]:      # balayage systematique
    si conditions_M1(tau) et delta > delta_min(tau):
        prix = arrondi_tick_bas(p_ancre_cote(tau) - delta)
        poster_passif(cote, prix, taille = PARTS_MIN)        # 5 parts, jamais plus
        journaliser(24 champs + 8 horodatages + delta + tau + phase)
        a_expiration_2s: journaliser(rempli ? oui/non, quantite, prix_effectif)
a_la_cloture: journaliser(issue_officielle, pnl_net, sous_groupe)
```
- **FenÃªtre d'exploitabilitÃ© :** 2 s par quote. **Budget requis : < 200 ms** aller-retour, soit 10 % de la fenÃªtre. Part des Ã©vÃ©nements atteignables : **non dÃ©terminable â€” c'est la mesure produite par ce mÃ©canisme.**
- **Action :** ordre limite passif, cÃ´tÃ© ancrÃ©, prix `p_ancrÃ© âˆ’ Î´` arrondi au tick vers le bas, taille **5 parts exactement**, validitÃ© 2 s.
- **Sortie et invalidation :** clÃ´ture au rÃ¨glement. Annulation Ã  2 s, Ã  toute rupture d'intÃ©gritÃ© (7.1), Ã  tout changement de `z` supÃ©rieur Ã  `Î´`, ou Ã  `Ï„ = 270 s`.
- **CoÃ»ts :** `frais_AR_pnl = 0,02Â·askÂ·(1âˆ’ask)` par aller-retour, plus le tick. Sur `ask = 0,80` : frais 0,0032 pt, soit **0,016 $** pour 5 parts. **Perte maximale par Ã©vÃ©nement : `5 Ã— ask â‰ˆ 4,00 $`**, donc le budget de mesure doit Ãªtre dimensionnÃ© en consÃ©quence : Ã  raison d'un Ã©vÃ©nement par session, mesurer 100 sessions expose au plus **400 $**, et l'espÃ©rance attendue est proche de zÃ©ro.
- **CapacitÃ© et impact :** 5 parts est en dessous de la taille mÃ©diane du micro-flux ; le bot est invisible dans le carnet. C'est voulu : Ã  cette taille, il **ne referme pas l'Ã©cart** et ne modifie pas le comportement de la contrepartie, donc les taux de fill mesurÃ©s sont non biaisÃ©s.
- **Cas dÃ©gradÃ©s :** rejet d'ordre â†’ journaliser le code de rejet et compter l'Ã©vÃ©nement comme non atteignable ; oracle en retard anormal â†’ suspension par 7.1 ; exÃ©cution partielle â†’ journaliser la quantitÃ© effective, elle mesure directement la profondeur au meilleur niveau, qui est le champ manquant nÂ°1.
- **Contrepartie :** fournisseurs de liquiditÃ© passive, **attribution non prouvÃ©e**. Raison probable pour laquelle ils laissent l'Ã©cart : ils re-cotent sur Ã©vÃ©nement de carnet, pas sur la dÃ©croissance silencieuse de `Ïƒ_T`.
- **Test de non-directionnalitÃ© :** **Oui.** Le balayage de `Î´` est symÃ©trique, le cÃ´tÃ© est dÃ©duit par miroir, et le PnL attendu est proche de zÃ©ro dans les quatre sous-groupes par construction.
- **Test d'implÃ©mentabilitÃ© :** **Oui.**

### 7.3 â€” M1 Â· Cotation passive ancrÃ©e Ã  l'oracle, dÃ©cote pilotÃ©e par le risque rÃ©siduel

- **CatÃ©gorie :** MÃ©canique.
- **Ã‰vÃ©nement dÃ©clencheur :** union des Ã©vÃ©nements oracle et carnet, alors que `Ï„ âˆˆ [180 ; 270]` et que les 20 conditions retenues du Â§ 5 sont satisfaites.
- **DonnÃ©es d'entrÃ©e :**

| Champ | Ã‰lÃ©ment source | FrÃ©quence | Horloge et prÃ©cision requise | Dans les rapports | Dans le schÃ©ma |
|---|---|---|---|---|---|
| `oracle.price`, `oracle.t_ms` | Oracle | 1 Hz | Capture oracle, 1 ms | Oui | Oui |
| `K`, `k_source`, `endDate` | Officiel | 1Ã— | MarchÃ©, â‰¤ 100 ms | Non | **Ã€ vÃ©rifier** |
| `yes_ask`, `yes_bid`, `no_ask`, `no_bid` | Carnet | 4 Hz min | Capture carnet, 1 ms | Oui | Oui |
| Profondeur 12 niveaux `(prix, qtÃ©)` | Carnet L2 | 4 Hz min | Capture carnet, â‰¤ 100 ms | Non | **Ã€ vÃ©rifier** |
| `position`, `n_ordres_en_vol` | Ã‰tat interne | Continu | Locale + rÃ©conciliation | Non | **Ã€ vÃ©rifier** |
| `order_id`, `t_ack`, `t_fill`, prix et qtÃ© exÃ©cutÃ©s | Moteur | Ã‰vÃ©nementiel | Serveur, â‰¤ 10 ms | Non | **Ã€ vÃ©rifier** |
| `fee_bps` | API CLOB | PrÃ©-vol | â€” | Non | **Ã€ vÃ©rifier** |
| `issue_officielle`, TWAP `[270;300]` | RÃ©solution | 1Ã— | MarchÃ© | Partiel (PDF de session) | **Ã€ vÃ©rifier** |

- **Calcul :**
```
# --- boucle 1 Hz : oracle et temps ---
sigma_30    = ecart_type(increments_1s(oracle, ]tau-30, tau])) * sqrt(30)
sigma_T     = sigma_30 * sqrt((300 - tau) / 30)
m_etoile    = oracle_last(tau) - K
z           = abs(m_etoile) / sigma_T
cote        = "OUI" si m_etoile >= 0 sinon "NON"        # transformation miroir, pas un pari
p_ancre     = Phi(z)                                     # score borne, PAS une probabilite calibree
X60         = nb_changements_de_signe(m_etoile, ]tau-60, tau])
P           = duree_cumulee_meme_signe_avec_marge(tau)
v           = abs(m_etoile(tau) - m_etoile(tau-15)) / 15

# --- boucle 4 Hz : carnet ---
ask         = ask_du_cote(cote)
frais_AR    = 0.07 * ask * (1 - ask) + 0.01
e           = abs(p_ancre - ask)
delta_min   = frais_AR + 0.01
q95_saut    = quantile95(abs(diff(ask)), fenetre 3 s, mesure en ligne)
q95_chute   = quantile95(chute(ask), fenetre 3 s, mesure en ligne)
N           = plancher(5.00 / (ask + frais_AR))

# --- decote degressive : palier mesure, enveloppe monotone ---
si tau <  180 : delta_guard = 0.677     # > D_max -> aucune cotation
si 180 <= tau < 240 : delta_guard = 0.0157
si 240 <= tau < 270 : delta_guard = 0.0107
delta = max(delta_min, delta_guard)

# --- porte d'entree : conjonction stricte, un code de refus par test ---
ENTRER = integrite_ok(tau)                                      # 7.1 : T1..T7, T20, invariants
     et 180 <= tau <= 270                                       # T8
     et z >= 1.0                                                # T9
     et P >= 5.0                                                # T10
     et X60 == 0                                                # T11
     et v <= 0.45                                               # T12 requalifie, symetrique
     et ask >= mediane_3s(ask) - q95_chute                       # T13
     et max(abs(diff(ask)), 3 s) <= q95_saut                     # T15a
     et e > delta_min depuis >= 2 s                              # T15b + T16
     et e <= 0.114                                               # T17 coupe-circuit
     et (1 - ask) > 0.02 * ask * (1 - ask) + 0.02                # T18 requalifie
     et profondeur_cumulee(jusqu_a prix_limite) >= N             # T21  <-- FAIL-CLOSED aujourd'hui
     et position == vide et n_ordres_en_vol == 0 et N >= 5       # T22  <-- FAIL-CLOSED aujourd'hui

si ENTRER:
    prix_limite = arrondi_tick_bas(p_ancre - delta)
    poster_passif(cote, prix_limite, N, validite = 2 s)
    journaliser_avant_envoi(24 champs + 8 horodatages)
sinon:
    journaliser_refus(code_atomique_du_premier_test_faux)
```
- **FenÃªtre d'exploitabilitÃ© :** **2 s** par quote (fenÃªtre de contact du BBO mesurÃ©e). FenÃªtre d'action de la session : **90 s** (`[180;270]`). **Budget de latence requis : < 200 ms** aller-retour. Budget rÃ©el : **non dÃ©terminable** (Â§ 0.6). **Part des Ã©vÃ©nements atteignables : non dÃ©terminable â€” Ã  mesurer par 7.2.**
- **Action :** ordre limite **passif** (jamais agressif : le mÃ©canisme rÃ©munÃ¨re l'attente, pas l'immÃ©diatetÃ©), cÃ´tÃ© ancrÃ© dÃ©duit par miroir du signe de `m*`, prix `p_ancrÃ© âˆ’ Î´` arrondi au tick **vers le bas**, taille `floor(N)` plafonnÃ©e par la profondeur disponible, validitÃ© 2 s.
- **Sortie et invalidation :** sortie **unique, au rÃ¨glement** â€” pas de sortie intermÃ©diaire, parce que la thÃ¨se est que le temps travaille pour la position. Annulation si : expiration 2 s ; rupture d'intÃ©gritÃ© ; `|z(Ï„) âˆ’ z(Ï„_envoi)| Â· dp/dz > Î´` ; saut d'ask `> q95_saut` ; `Xâ‚†â‚€ > 0` ; `Ï„ > 270 s`. DÃ©lai maximal d'annulation : **â‰¤ 500 ms**, Ã  mesurer.
- **CoÃ»ts :** frais rÃ©els `0,02Â·askÂ·(1âˆ’ask)`, soit **0,0032 pt Ã  `ask = 0,80`** ; tick 0,01 ; slippage attendu **0** puisque l'ordre est passif et limitÃ©. **PnL net attendu par Ã©vÃ©nement : `N Ã— (Î´ âˆ’ frais_AR_pnl)`.** Avec `N = 6` et `Î´ = 0,0157` en phase `[180;240[` : `6 Ã— (0,0157 âˆ’ 0,0032) = **+0,075 $**`, soit **+1,5 % du capital de 5,00 $** par Ã©vÃ©nement rempli. En phase `[240;270[` avec `Î´ = 0,0107` : `6 Ã— 0,0075 = **+0,045 $**`, soit +0,9 %. **Ce chiffre est le cÅ“ur Ã©conomique du livrable, et il est mince : le mÃ©canisme est une activitÃ© de tenue de marchÃ©, pas un arbitrage.**
- **CapacitÃ© et impact :** taille au-delÃ  de laquelle le bot referme lui-mÃªme l'Ã©cart : **non dÃ©terminable sans le niveau 2**. Borne supÃ©rieure raisonnable dÃ©duite de la structure des populations : la taille mÃ©diane du gros flux est de **57,64 $** et le P90 de **178,65 $** ; un ordre du bot de 5,00 $ reprÃ©sente **8,7 %** d'un gros ordre mÃ©dian et **0,023 %** du notionnel d'une session (21 590 $). Ã€ cette Ã©chelle, l'impact est nÃ©gligeable. **Signe que la contrepartie s'adapte :** baisse du taux de fill Ã  `Î´` constant sur une fenÃªtre glissante de 20 sessions, ou apparition d'une quote systÃ©matiquement placÃ©e 1 tick devant celle du bot. **Action du bot dans ce cas :** augmenter `Î´` d'un tick et rÃ©Ã©valuer ; si le taux de fill ne remonte pas sur 20 sessions, retrait du mÃ©canisme.
- **Cas dÃ©gradÃ©s :** oracle muet ou en retard anormal â†’ `TICK_TROU`/`TICK_PERIME`, annulation immÃ©diate (l'ancrage est la seule rÃ©fÃ©rence du bot : sans oracle frais, il n'a pas de prix) ; carnet vide au meilleur niveau â†’ `SPREAD_INDISPO`, annulation ; dÃ©synchronisation des horloges dÃ©tectÃ©e par `basis` hors bande â†’ `BASIS_CORROMPU`, suspension de la session ; rejet d'ordre â†’ journaliser le code, rÃ©essayer **une seule fois**, puis suspendre 10 s ; **oracle traversant le seuil de rÃ©solution alors qu'un ordre est en attente** â†’ annulation immÃ©diate et inconditionnelle, puis recotation sur le cÃ´tÃ© opposÃ© au tick suivant si toutes les conditions sont de nouveau rÃ©unies (Â§ 8.4 bis).
- **Contrepartie :** fournisseurs de liquiditÃ© passive dont les quotes ne sont pas re-cotÃ©es Ã  la cadence de dÃ©croissance de `Ïƒ_T`. **Attribution non prouvÃ©e** â€” champs `order_id` et `maker/taker` Ã  journaliser. Raison pour laquelle ils laissent l'Ã©cart : la dÃ©croissance du temps restant est un **Ã©vÃ©nement silencieux**. Rien ne se passe dans le carnet, aucun trade, aucune mise Ã  jour : et c'est prÃ©cisÃ©ment cela qui constitue l'information.
- **Test de non-directionnalitÃ© :** **Oui.** DÃ©monstration : sous mouvement miroir de l'oracle, `|m*|`, `z`, `Ïƒ_T`, `Î´`, `e` et `N` sont **identiques** ; seul le cÃ´tÃ© change, et le prix du cÃ´tÃ© opposÃ© est donnÃ© exactement par l'invariant `bid_OUI + ask_NON = 1`, mesurÃ© exact sur 1025/1025 lignes. L'action est donc rigoureusement miroir. **Cette propriÃ©tÃ© dÃ©pend de `Ï„ â‰¥ 180 s`** : en dessous, le mouvement rÃ©siduel de l'oracle (11,62 $ = 1,6 Ïƒâ‚ƒâ‚€) dÃ©passe `Î´`, et la position redevient un pari. **PnL par sous-groupe : non dÃ©terminable â€” Ã  produire par 7.2, et c'est la condition de dÃ©ploiement.**
- **Test d'implÃ©mentabilitÃ© :** **Oui.**

---
# 8. RÃ©ponses finales

## 8.1

**Les grandeurs qui interagissent de maniÃ¨re prÃ©visible sont :**

1. **`Ï„` (temps Ã©coulÃ©) et `Ïƒ_T` (risque rÃ©siduel)** â€” partenaires : `Ïƒâ‚ƒâ‚€`. Sens : `Ïƒ_T(Ï„) = Ïƒâ‚ƒâ‚€Â·âˆš((300âˆ’Ï„)/30)`, strictement dÃ©croissante, division exacte par âˆš5 entre `Ï„ = 0` et `Ï„ = 240`. Produite par : l'oracle et l'horloge de fenÃªtre. **C'est la seule relation du systÃ¨me entiÃ¨rement connue Ã  l'avance, et donc le seul edge non prÃ©dictif disponible.**
2. **`Ïƒ_T` et le mouvement rÃ©siduel de l'oracle** â€” sens : proportionnel. MesurÃ© 10,76 $ en `[30;60[`, 11,62 $ en `[60;180[`, 1,50 $ en `[180;240[`, 1,55 $ en `[240;270[`. Produite par : l'oracle.
3. **`Ïƒ_T` et `Î»`** â€” sens : `Î» = 0,5513Â·Ïƒ_T`, identitÃ© confirmÃ©e par ajustement direct (0,54 / 0,51 / 0,58). Produite par : le modÃ¨le.
4. **`Ïƒ_T` et `M`** â€” sens : `M = z_minÂ·Ïƒ_T`, donc dÃ©croissante comme âˆš. Produite par : le modÃ¨le.
5. **`|m*|` et `Ïƒ_T`** â€” partenaire : `z = |m*|/Ïƒ_T`. Sens : `z` croÃ®t quand l'oracle s'Ã©loigne du strike **ou** quand le temps passe. Produite par : oracle et strike. **Mesure la dÃ©termination mÃ©canique de l'issue, jamais sa direction.**
6. **`bid_OUI` et `ask_NON`** â€” sens : `bid_OUI + ask_NON = 1`, exact sur 1 025/1 025 lignes. Produite par : le moteur de cotation. Ã‰cart maximal observÃ© 1 tick, sur 2 lignes sur 1 026.
7. **`ask_OUI` et `bid_NON`** â€” sens : `ask_OUI + bid_NON = 1`, exact sur 1 024/1 026. Produite par : le moteur de cotation.
8. **`spot` et `oracle`** â€” partenaire : `basis`. Sens : `basis = spot âˆ’ oracle â‰ˆ +42,1 $`, **constant et positif sur 100 % des lignes appariÃ©es** de 5 sessions. Produite par : la diffÃ©rence de dÃ©finition entre les deux sources. **Ne se referme jamais : ce n'est pas un edge, c'est un contrÃ´le d'intÃ©gritÃ©.**
9. **`m*` et le mid du carnet** â€” sens : quasi affine, `r = +0,951`, `RÂ² = 0,904`. Produite par : le carnet. **Signifie qu'il n'existe aucune information privÃ©e dans `m*`.**
10. **`p_brut` et `ask`** â€” sens : `RÂ² = 0,956`. Produite par : le carnet. MÃªme conclusion : le carnet et le modÃ¨le voient la mÃªme chose.
11. **`ask` et `N`** â€” sens : `N = floor(C/(ask + frais + 0,01))`, dÃ©croissante, `RÂ²(ask, N) = 0,90`. Produite par : la rÃ¨gle de sizing. Impose `ask â‰¤ 0,98926`.
12. **`ask` et `frais_AR`** â€” sens : `frais_AR = 0,07Â·askÂ·(1âˆ’ask) + 0,01`, maximale Ã  `ask = 0,5`. MesurÃ©e : mÃ©diane 0,0231 pt, plage 0,0107â€“0,0275. Produite par : la plateforme.
13. **`Xâ‚†â‚€` et la couleur de session** â€” sens : `Xâ‚†â‚€ = 0` âŸº oracle dÃ©crochÃ©, Ã©cart carnet/ancrage large et persistant (`D > frais` sur 57,7 % des secondes) ; `Xâ‚†â‚€ â‰¥ 1` âŸº oracle oscillant, Ã©cart mÃ©dian **nÃ©gatif** (âˆ’0,0611, âˆ’0,0547). Produite par : oracle et strike. **Plus forte corrÃ©lation du corpus avec le gain, `RÂ² = 0,207`.**
14. **`Ï„` et l'Ã©cart `|p_ancrÃ© âˆ’ ask|`** â€” sens : croissant. Ajustement descriptif `e(Ï„) = 0,04998 + 0,003358Â·Ï„`, `RÂ² = 0,9582` sur `Ï„ â‰¥ 180 s`. Produite par : l'Ã©cart de cadence de re-cotation entre l'oracle et le carnet.
15. **`Ï„` et `Î´_guard`** â€” sens : dÃ©croissant par paliers. 0,677 pt avant 180 s, 0,0157 pt en `[180;240[`, 0,0107 pt en `[240;270[`. Produite par : la dÃ©croissance de `Ïƒ_T`. **C'est la loi de dÃ©cote dÃ©gressive, et elle est une consÃ©quence arithmÃ©tique, pas une hypothÃ¨se.**
16. **`Ï„` et le taux de contact du BBO Ã  `Î´` donnÃ©** â€” sens : non monotone, maximal en phase milieu. Ã€ `Î´ = 0,02` : 71,5 % Ã  l'ouverture, 88,5 % au milieu, 48,6 % en fin. Produite par : le carnet.
17. **`ask` et `mÃ©dâ‚ƒâ‚›(ask)`** â€” sens : `ask â‰¥ mÃ©dâ‚ƒâ‚› âˆ’ q95(chute)`. Chute p95 mesurÃ©e 0,08â€“0,15, p99 0,13â€“0,29. Produite par : le carnet.
18. **`Ï„` et `Ï„_lock`** â€” sens : `Ï„_lock â‰ˆ 262 s > 240 s`. 16 Ã  18 % du chemin de prix est postÃ© dans les 20 % finaux. Produite par : le rÃ¨glement en TWAP `[270;300]`. **L'issue n'existe pas dans la fenÃªtre d'entrÃ©e du PDF.**
19. **Taille de trade et part du notionnel** â€” sens : trÃ¨s concentrÃ©. 10,2 % des Ã©vÃ©nements portent 63,2 % du notionnel ; 62,3 % en portent 11,0 %. Produite par : les populations. **Borne la capacitÃ© du bot.**
20. **Cadence du carnet et cadence de l'oracle** â€” sens : rapport â‰ˆ **77:1** (0,013 s contre 1 s). Produite par : l'architecture. Impose une Ã©valuation par union des deux flux, Ã  4 Hz minimum.

**Grandeurs dont l'interaction a Ã©tÃ© postulÃ©e et mesurÃ©e inexistante :** `lag_signÃ©(oracle â†’ carnet)` et la bande `[+5 ; +60] s` â€” argmax mÃ©dian **âˆ’0,75 s**, 0/27 sessions dans la bande, 26/27 avec `L â‰¤ 0`.

## 8.2

**Les relations et les rÃ¨gles d'action dÃ©duites pour le bot sont :**

1. **RÃ¨gle 1 :** SI `Ã¢ge_oracle(Ï„) > 2,5 s` ALORS aucune cotation, annulation, code `TICK_PERIME` AU MOMENT de chaque Ã©valuation JUSQU'Ã€ `Ã¢ge â‰¤ 2,5 s`.
2. **RÃ¨gle 2 :** SI `trou_oracle > 8 s` OU `trou_carnet > 3 s` ALORS aucune cotation, annulation, gel du compteur de persistance, code `TICK_TROU` AU MOMENT de chaque Ã©valuation JUSQU'Ã€ reprise des deux flux.
3. **RÃ¨gle 3 :** SI un des quatre champs du BBO est absent OU `|bid_OUI + ask_NON âˆ’ 1| > 1 tick` ALORS aucune cotation, code `SPREAD_INDISPO` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ snapshot complet et cohÃ©rent.
4. **RÃ¨gle 4 :** SI `Ï„ < 30 s` OU fenÃªtre de 30 s incomplÃ¨te OU `Ïƒâ‚ƒâ‚€ âˆ‰ [0,5 ; 40] $` ALORS aucune cotation, code `SIGMA_INDISPO` AU MOMENT de chaque Ã©valuation JUSQU'Ã€ `Ïƒâ‚ƒâ‚€` mesurable et plausible.
5. **RÃ¨gle 5 : hors pÃ©rimÃ¨tre (directionnelle) â€” ne pas implÃ©menter comme condition d'entrÃ©e.** Requalification structurelle : SI `|basis(Ï„) âˆ’ mÃ©diane_glissante_60s(basis)| > 3Â·Ïƒ_session(basis)` OU `HALT` absent ALORS aucune cotation, code `BASIS_CORROMPU` AU MOMENT de chaque ligne spot JUSQU'Ã€ retour dans la bande.
6. **RÃ¨gle 6 :** SI `endDate` absent OU `t0_source â‰  officiel` ALORS **session entiÃ¨re refusÃ©e**, code `T0_APPROXIME` AU MOMENT de l'ouverture JUSQU'Ã€ la session suivante.
7. **RÃ¨gle 7 :** SI `K` absent OU `k_source â‰  officiel` ALORS **session entiÃ¨re refusÃ©e**, code `K_NON_OFFICIEL` AU MOMENT de l'ouverture JUSQU'Ã€ la session suivante.
8. **RÃ¨gle 8 :** SI `Ï„ < 180 s` OU `Ï„ > 270 s` ALORS aucune cotation, annulation de tout ordre en vol Ã  `Ï„ = 270 s`, code `HORS_FENETRE` AU MOMENT de chaque Ã©valuation JUSQU'Ã€ `Ï„ âˆˆ [180 ; 270]`.
9. **RÃ¨gle 9 :** SI `z(Ï„) = |oracle âˆ’ K|/Ïƒ_T < 1,0` ALORS aucune cotation, code `MARGE_INSUFFISANTE` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ `z â‰¥ 1,0`.
10. **RÃ¨gle 10 :** SI `P(Ï„) < 5 s` cumulÃ©es de mÃªme signe avec marge ALORS aucune cotation, code `PERSISTANCE` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ `P â‰¥ 5 s`, avec reset si `signe(m*)` change ou si `trou_oracle > 8 s`.
11. **RÃ¨gle 11 :** SI `Xâ‚†â‚€(Ï„) > 0` ALORS aucune cotation, annulation, code `CYCLICITE` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ `Xâ‚†â‚€ = 0`.
12. **RÃ¨gle 12 : hors pÃ©rimÃ¨tre (directionnelle) â€” ne pas implÃ©menter comme condition d'entrÃ©e.** Requalification structurelle : SI `|m*(Ï„) âˆ’ m*(Ï„âˆ’15)|/15 > 0,45 $/s` ALORS aucune cotation, annulation, code `RETOURNEMENT` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ retour sous le seuil.
13. **RÃ¨gle 13 :** SI `ask_cÃ´tÃ©(Ï„) < mÃ©dâ‚ƒâ‚›(ask_cÃ´tÃ©) âˆ’ q95(chute_3s)` OU `mÃ©dâ‚ƒâ‚›` non calculable ALORS aucune cotation, annulation, code `ASK_KRACH` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ retour au-dessus du seuil pendant 2 s.
14. **RÃ¨gle 14 : hors pÃ©rimÃ¨tre â€” le phÃ©nomÃ¨ne postulÃ© n'existe pas dans les donnÃ©es (0/27 sessions dans la bande `[+5;+60] s`).** Ne pas implÃ©menter comme condition d'entrÃ©e. Requalification structurelle : journaliser `lag_signÃ©` et son `r` ; alerte non bloquante si la mÃ©diane glissante sur 10 sessions dÃ©passe +5 s.
15. **RÃ¨gle 15 :** SI `max(|Î”ask_cÃ´tÃ©|)` sur `]Ï„âˆ’3 ; Ï„]` `> q95(|Î”ask| 3 s)` ALORS aucune cotation, code `ASK_INSTABLE` ; SI `e(Ï„) â‰¤ frais_AR + 0,01` depuis moins de 2 s ALORS aucune cotation, code `ECART_NON_CONFIRME` â€” AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ satisfaction simultanÃ©e.
16. **RÃ¨gle 16 :** SI `e(Ï„) = |p_ancrÃ©,cÃ´tÃ© âˆ’ ask_cÃ´tÃ©| â‰¤ frais_AR(ask) + 0,01` ALORS aucune cotation, code `PAS_D_EDGE` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ `e > frais_AR + 0,01`.
17. **RÃ¨gle 17 :** SI `e(Ï„) > 0,114` ALORS aucune cotation, journalisation renforcÃ©e des 24 champs, alerte, code `DECOTE_ABERRANTE` AU MOMENT de chaque dÃ©cision JUSQU'Ã€ `e â‰¤ 0,114`. **Coupe-circuit, jamais une porte ; premier seuil Ã  rÃ©viser aprÃ¨s les 500 sessions.**
18. **RÃ¨gle 18 :** SI `(1 âˆ’ ask_cÃ´tÃ©) â‰¤ 0,02Â·askÂ·(1âˆ’ask) + 0,02` ALORS aucune cotation, code `PRIX_MAX` AU MOMENT de chaque dÃ©cision JUSQU'Ã€ satisfaction. **Les trois constantes de `P_max` (0,70 / 0,00075 / 0,92) sont supprimÃ©es.**
19. **RÃ¨gle 19 :** SI les coefficients de Platt `(a,b)` ne sont pas versionnÃ©s OU `Brier(p) > Brier(mid)` dans au moins un des quatre sous-groupes ALORS `p` est journalisÃ© mais n'entre dans aucune condition d'entrÃ©e ; l'ancrage utilise `Î¦(z)` â€” AU MOMENT de chaque revue de calibration JUSQU'Ã€ validation par sous-groupe sur â‰¥ 50 sessions.
20. **RÃ¨gle 20 :** SI `s(Ï„) < 0` OU `s(Ï„) > 0,04` OU disponibilitÃ© fausse ALORS aucune cotation, annulation, code `SPREAD` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ `s âˆˆ [0 ; 0,04]`.
21. **RÃ¨gle 21 :** SI la profondeur cumulÃ©e du cÃ´tÃ© cotÃ© jusqu'au prix limite est absente OU `< floor(N)` ALORS aucune cotation, code `PROFONDEUR` AU MOMENT de chaque dÃ©cision JUSQU'Ã€ disponibilitÃ© du niveau 2. **Fail-closed permanent en l'Ã©tat des donnÃ©es.**
22. **RÃ¨gle 22 :** SI `position â‰  âˆ…` OU `n_ordres_en_vol > 0` OU `floor(N) < 5` OU rÃ©conciliation indisponible ALORS aucune cotation, code `OCCUPE` ou `PARTS_MIN` AU MOMENT de chaque dÃ©cision JUSQU'Ã€ inventaire vide rÃ©conciliÃ© et `floor(N) â‰¥ 5`. **Fail-closed permanent en l'Ã©tat des donnÃ©es.**

**RÃ¨gles issues des mÃ©canismes de 4.6 :**

23. **RÃ¨gle M2 (intÃ©gritÃ©) :** SI `integrite_ok(Ï„)` est faux ALORS suspension de M1 et M3, annulation immÃ©diate de tout ordre en vol, journalisation du code atomique du premier test faux AU MOMENT de chaque mise Ã  jour de l'un des trois flux JUSQU'Ã€ `integrite_ok(Ï„)` vrai.
24. **RÃ¨gle M3 (instrumentation) :** SI les conditions de M1 sont satisfaites hors T21 et T22 ALORS poster un ordre limite passif de **5 parts exactement** au prix `arrondi_tick_bas(p_ancrÃ© âˆ’ Î´)` pour chaque `Î´ âˆˆ {0,01 ; 0,02 ; 0,03 ; 0,05 ; 0,08}`, validitÃ© 2 s, avec journalisation des 24 champs et des 8 horodatages AU MOMENT de chaque Ã©vÃ©nement Ã©ligible JUSQU'Ã€ expiration Ã  2 s puis clÃ´ture au rÃ¨glement.
25. **RÃ¨gle M1 (cotation ancrÃ©e) :** SI les 20 conditions retenues sont satisfaites simultanÃ©ment ALORS poster un ordre limite **passif** du cÃ´tÃ© `signe(m*)`, au prix `arrondi_tick_bas(p_ancrÃ©,cÃ´tÃ©(Ï„) âˆ’ Î´(Ï„))` avec `Î´(Ï„)` donnÃ© par la table 8.4 bis, taille `min(floor(N), profondeur_disponible)`, validitÃ© 2 s AU MOMENT de chaque Ã©vÃ©nement oracle ou carnet dans `[180 ; 270]` JUSQU'Ã€ exÃ©cution puis clÃ´ture unique au rÃ¨glement, ou annulation sur expiration, rupture d'intÃ©gritÃ©, `|Î”z| Â· dp/dz > Î´`, saut d'ask `> q95_saut`, `Xâ‚†â‚€ > 0`, ou `Ï„ > 270 s`.

## 8.3

**Les mÃ©canismes structurels que le bot exploite sont :**

- **M2 â€” ContrÃ´le d'intÃ©gritÃ© par invariants croisÃ©s** â€” dÃ©clencheur : toute mise Ã  jour d'un des trois flux â€” latence exploitÃ©e : aucune, mÃ©canisme purement local, `< 1 ms`, horloge de capture du carnet â€” fenÃªtre : instantanÃ©e â€” action : suspension et annulation, aucune position â€” contrepartie : aucune, garde-fou â€” capacitÃ© : illimitÃ©e.
- **M3 â€” Instrumentation de mesure du taux de remplissage** â€” dÃ©clencheur : conditions de M1 hors T21/T22 â€” latence exploitÃ©e : fenÃªtre de contact du BBO **2 s**, horloge de capture du carnet â€” fenÃªtre : 2 s par quote, 90 s par session â€” action : quote passive de 5 parts avec balayage systÃ©matique de `Î´` â€” contrepartie : fournisseurs de liquiditÃ© passive, attribution non prouvÃ©e â€” capacitÃ© : 5 parts, soit 8,7 % d'un gros ordre mÃ©dian, donc sans impact mesurable.
- **M1 â€” Cotation passive ancrÃ©e Ã  l'oracle, dÃ©cote pilotÃ©e par le risque rÃ©siduel `Ïƒ_T`** â€” dÃ©clencheur : union des Ã©vÃ©nements oracle et carnet dans `Ï„ âˆˆ [180 ; 270]` avec `z â‰¥ 1,0` et `Xâ‚†â‚€ = 0` â€” latence exploitÃ©e : **aucune latence de communication (mesurÃ©e nulle : âˆ’0,75 s mÃ©dian) ; la latence exploitÃ©e est celle du carnet par rapport Ã  la dÃ©croissance dÃ©terministe de `Ïƒ_T`, soit un Ã©cart mesurÃ© de 0,090 pt en `[180;240[` et 0,464 pt en `[240;270[`**, horloge `Ï„`Â·marchÃ© pour l'ancrage et capture du carnet pour l'ask â€” fenÃªtre : 2 s par quote â€” action : ordre limite passif Ã  `p_ancrÃ© âˆ’ Î´`, `floor(N)` parts, cÃ´tÃ© par miroir du signe de `m*`, sortie unique au rÃ¨glement â€” contrepartie : fournisseurs de liquiditÃ© passive qui re-cotent sur Ã©vÃ©nement de carnet et non sur la dÃ©croissance silencieuse du temps restant, attribution non prouvÃ©e â€” capacitÃ© : `5,00 $`, soit 0,023 % du notionnel de session ; borne supÃ©rieure non dÃ©terminable sans le niveau 2.

## 8.4 Ce qu'il faut faire globalement

Ã€ l'ouverture, le bot lit deux valeurs et deux seulement : `endDate`, dont il dÃ©duit `Ï„â‚€ = endDate âˆ’ 300 s`, et `K` avec son `k_source`. Si l'une des deux manque ou n'est pas officielle, il refuse la session entiÃ¨re et n'Ã©crit qu'une ligne de refus. Il ne devine ni l'une ni l'autre : le corpus montre qu'un Ã©cart de strike de 12,25 $ inverse le rÃ¨glement d'une session, et qu'une origine relative prise pour une origine absolue dÃ©place la fenÃªtre de rÃ¨glement de 126 secondes au-delÃ  d'un marchÃ© clos.

De `Ï„ = 0` Ã  `Ï„ = 30 s`, le bot ne calcule rien de dÃ©cisionnel. La fenÃªtre de 30 secondes d'oracle n'est pas pleine, donc `Ïƒâ‚ƒâ‚€` n'existe pas, donc `Ïƒ_T` n'existe pas, donc il n'a pas de prix de rÃ©fÃ©rence. Il refuse, il n'attend pas : la distinction compte, parce qu'un refus s'Ã©crit dans le journal et une attente ne s'Ã©crit pas.

De `Ï„ = 30 s` Ã  `Ï„ = 180 s`, le bot **mesure et se tait**. Il calcule Ã  chaque tick d'oracle `Ïƒâ‚ƒâ‚€` comme l'Ã©cart-type des incrÃ©ments d'une seconde remis Ã  l'Ã©chelle de trente, puis `Ïƒ_T(Ï„) = Ïƒâ‚ƒâ‚€Â·âˆš((300âˆ’Ï„)/30)`, puis `m* = oracle âˆ’ K`, puis `z = |m*|/Ïƒ_T`, puis son prix ancrÃ© `p_ancrÃ©,cÃ´tÃ© = Î¦(z)` du cÃ´tÃ© donnÃ© par le signe de `m*`. Il calcule Ã  quatre hertz au moins l'Ã©cart `e = |p_ancrÃ© âˆ’ ask_cÃ´tÃ©|`, le spread, la mÃ©diane d'ask sur trois secondes et les quantiles de saut et de chute. Il journalise tout. Et il ne cote pas, pour une raison qui n'est pas de la prudence mais de l'arithmÃ©tique : sur cet intervalle, le mouvement que l'oracle peut encore faire vaut en mÃ©diane 11,62 $, soit 1,6 fois `Ïƒâ‚ƒâ‚€`, et la dÃ©cote qui protÃ©gerait contre ce mouvement vaut 0,677 point de probabilitÃ©, six fois le coupe-circuit. Toute position prise lÃ  est exposÃ©e au sens du sous-jacent, quelle que soit la sophistication des filtres. C'est exactement ce que les vingt-cinq agents ont mesurÃ© chacun de leur cÃ´tÃ© : la fenÃªtre `[60;240]` du PDF ne contient pas l'information, elle contient l'illusion de l'information.

Ã€ `Ï„ = 180 s`, trois choses changent en mÃªme temps, et c'est la dÃ©couverte centrale du consensus. Le mouvement rÃ©siduel de l'oracle tombe Ã  1,50 $, soit 0,21 `Ïƒâ‚ƒâ‚€`. La dÃ©cote de garde tombe de 0,677 Ã  0,0157 point, c'est-Ã -dire d'un facteur quarante-trois. Et l'Ã©cart entre le carnet et l'ancrage, lui, ne tombe pas : il vaut encore 0,090 point en mÃ©diane, et il croÃ®t jusqu'Ã  0,464 point dans les trente derniÃ¨res secondes observÃ©es, selon une pente mesurÃ©e de 0,003358 point par seconde avec un `RÂ²` de 0,958. Le bot ouvre donc sa fenÃªtre d'action. Il vÃ©rifie que l'oracle est dÃ©crochÃ© du strike et non en train d'osciller autour : `z â‰¥ 1,0` et `Xâ‚†â‚€ = 0`, aucune traversÃ©e du strike dans la derniÃ¨re minute. C'est le seul jugement qu'il porte, et ce n'est pas un jugement de direction : c'est un jugement de **dÃ©termination**. Il ne dit pas Â« l'oracle va monter Â», il dit Â« il faudrait un Ã©cart-type complet du chemin restant pour que l'issue change, et l'oracle n'a pas hÃ©sitÃ© depuis soixante secondes Â».

Alors il cote. Passivement, jamais agressivement, parce que ce qu'il vend n'est pas de l'immÃ©diatetÃ© mais de l'attente. Du cÃ´tÃ© que le signe de `m*` dÃ©signe, Ã  un prix Ã©gal Ã  son ancrage moins la dÃ©cote du palier : 1,57 tick entre 180 et 240 secondes, 1,07 tick entre 240 et 270. Cette dÃ©cote a cette valeur Ã  cet instant pour une raison unique et vÃ©rifiable : elle est Ã©gale aux frais aller-retour plus le quantile Ã  90 % du mouvement adverse que l'ancrage peut subir pendant les deux secondes de vie de l'ordre. Et ce mouvement adverse dÃ©croÃ®t parce que `Ïƒ_T` dÃ©croÃ®t. **La dÃ©gressivitÃ© n'est pas un rÃ©glage, c'est la consÃ©quence arithmÃ©tique de la loi du temps.** L'ordre vit deux secondes, la durÃ©e pendant laquelle le taux de contact du meilleur niveau a Ã©tÃ© mesurÃ© : 88 % en phase milieu, 49 % en fin de session. Puis il est annulÃ© et recotÃ©, ou rempli.

La population qu'il prÃ©cÃ¨de est celle qui re-cote sur Ã©vÃ©nement de carnet et non sur Ã©coulement du temps. C'est pour cela que l'Ã©cart existe : la dÃ©croissance de `Ïƒ_T` est un Ã©vÃ©nement silencieux, aucun trade ne se produit, aucune ligne de carnet ne bouge, et c'est prÃ©cisÃ©ment ce silence qui est l'information. La population qu'il Ã©vite est le gros flux, dix pour cent des Ã©vÃ©nements et soixante-trois pour cent du notionnel, dont trois des cinq pics tombent aprÃ¨s 240 secondes : le bot reste Ã  cinq dollars, soit huit pour cent d'un gros ordre mÃ©dian, pour ne pas refermer lui-mÃªme l'Ã©cart et pour ne pas provoquer d'adaptation.

Il re-cote Ã  chaque mise Ã  jour de l'oracle qui dÃ©place `z` de plus d'une dÃ©cote, Ã  chaque saut d'ask supÃ©rieur au quantile mesurÃ©, Ã  chaque passage de phase Ã  240 secondes. Il annule sans condition si l'oracle traverse le strike alors qu'un ordre est en attente, parce que la traversÃ©e fait passer le cÃ´tÃ© ancrÃ© de l'autre bord et que l'ordre en vol devient instantanÃ©ment un pari sur le cÃ´tÃ© perdant. Ã€ 270 secondes, il annule tout ce qui reste et n'ouvre plus rien : le carnet cesse gÃ©nÃ©ralement de publier vers 273 secondes et la fenÃªtre de rÃ¨glement s'ouvre. Il ne sort jamais avant le rÃ¨glement, parce que la thÃ¨se est que le temps travaille pour la position : sortir tÃ´t reviendrait Ã  racheter l'incertitude qu'il vient de vendre.

Et il est inactif la plupart du temps. Sur les vingt-cinq fenÃªtres analysÃ©es, les sessions Ã  oracle oscillant â€” Ã©cart mÃ©dian carnet-ancrage nÃ©gatif, jusqu'Ã  huit traversÃ©es du strike â€” sont des sessions oÃ¹ la bonne dÃ©cision est de ne rien faire, et deux des trois sessions du premier rapport en font partie. Le bot doit journaliser son inaction avec le code atomique du test bloquant, parce que la distribution des non-entrÃ©es est la donnÃ©e qui permettra de rÃ©viser les seuils. Le PnL attendu par Ã©vÃ©nement rempli est de sept centimes et demi pour cinq dollars engagÃ©s en phase `[180;240[`, quatre centimes et demi en phase terminale. Ce n'est pas un arbitrage, c'est de la tenue de marchÃ© : la rentabilitÃ© vient du **nombre d'Ã©vÃ©nements sur cinq cents sessions**, jamais de l'ampleur d'un coup.

## 8.4 bis SpÃ©cification de la dÃ©cote dÃ©gressive

**Tableau fusionnÃ© sur toutes les sessions mesurantes.** `p_ancrÃ©,cÃ´tÃ©(Ï„) = Î¦(z(Ï„))` avec `z = |oracle_last(Ï„) âˆ’ K|/Ïƒ_T(Ï„)`, `Ïƒ_T = Ïƒâ‚ƒâ‚€Â·âˆš((300âˆ’Ï„)/30)`, `Ïƒâ‚ƒâ‚€` mesurÃ© en ligne, cÃ´tÃ© dÃ©duit par miroir du signe de `m*`.

| Phase | Temps restant | Formule du prix ancrÃ© | DÃ©cote retenue (montant, ticks) | n sessions mesurant | n confirmant | Dispersion inter-sessions (P10â€“P90) | Taux de remplissage attendu | PnL par sous-groupe |
|---|---|---|---|---|---|---|---|---|
| **Interdite** `Ï„ âˆˆ [30 ; 180[` | 120â€“270 s | `Î¦(z(Ï„))` | **Aucune cotation.** `Î´_guard` mesurÃ© **0,677 pt** (67,7 ticks), supÃ©rieur au coupe-circuit `D_max = 0,114` | 4 | 4 | **0,018 â†’ 0,677 pt** (facteur 37) | Sans objet | **0,00 $ dans les quatre sous-groupes** (abstention) |
| **Principale** `Ï„ âˆˆ [180 ; 240[` | 60â€“120 s | `Î¦(z(Ï„))` | **0,0157 pt â‰ˆ 1,57 tick**, arrondi au tick supÃ©rieur : **0,02 pt = 2 ticks** | 4 | 4 | 0,0107 â†’ 0,0157 pt | **â‰ˆ 88 %** (contact BBO 2 s Ã  Î´ = 0,02 : 85,20 % et 91,76 %) | **Non dÃ©terminable** â€” aucun fill dans le corpus. Ã€ produire par le mÃ©canisme 7.2 |
| **Terminale** `Ï„ âˆˆ [240 ; 270[` | 30â€“60 s | `Î¦(z(Ï„))` avec `w(Ï„)` actif, rÃ©fÃ©rentiel glissant vers le TWAP `[270;300]` | **0,0107 pt â‰ˆ 1,07 tick**, arrondi : **0,02 pt = 2 ticks** (plancher de tick) | 3 | 3 | 0,0107 â†’ 0,0157 pt | **â‰ˆ 49 %** (contact BBO 2 s : 0 % et 97,28 %, dispersion extrÃªme) | **Non dÃ©terminable** |
| **FermÃ©e** `Ï„ â‰¥ 270` | 0â€“30 s | â€” | **Aucune cotation.** Le carnet cesse de publier vers 273 s ; fenÃªtre de rÃ¨glement ouverte | 6 | 6 | â€” | Sans objet | **0,00 $ dans les quatre** |

**Loi de dÃ©gressivitÃ© universelle.**
- **Forme :** enveloppe monotone dÃ©croissante par paliers, et non fonction continue. Justification : la mÃ©diane de `Î´_guard` en phase milieu varie d'un facteur **37** entre sessions (0,01803 contre 0,67669) ; un ajustement continu sur ces points produirait une fausse prÃ©cision. Les ajustements continus proposÃ©s par les agents (`max(0 ; 0,440059 âˆ’ 0,00366623Â·Ï„)` avec `RÂ² = 1,000` sur deux points) sont **rejetÃ©s comme surajustÃ©s**.
- **ParamÃ¨tres et unitÃ©s :** `Î´(Ï„) = max(Î´_min(Ï„), Î´_guard(palier))` avec `Î´_min(Ï„) = frais_AR(ask) + 0,01` et `Î´_guard = {+âˆž si Ï„ < 180 ; 0,0157 si 180 â‰¤ Ï„ < 240 ; 0,0107 si 240 â‰¤ Ï„ < 270 ; +âˆž si Ï„ â‰¥ 270}`, en points de probabilitÃ© de contrat.
- **Grandeur observable qui la pilote :** `Ïƒ_T(Ï„) = Ïƒâ‚ƒâ‚€Â·âˆš((300âˆ’Ï„)/30)`, mesurÃ©e en ligne. C'est le pilote **causal** de la loi : `Î´_guard` est le quantile Ã  90 % du mouvement adverse que l'ancrage peut subir pendant la durÃ©e de vie de l'ordre, et ce mouvement est proportionnel Ã  `Ïƒ_T`. La dÃ©gressivitÃ© par paliers est donc la **discrÃ©tisation prudente** de la dÃ©croissance en racine de `Ïƒ_T`. VÃ©rification de cohÃ©rence : `Ïƒ_T(180)/Ïƒ_T(255) = 14,40/8,62 = 1,67` et `Î´_guard(180)/Î´_guard(255) = 0,0157/0,0107 = 1,47` â€” les deux ratios sont du mÃªme ordre, ce qui confirme le pilotage. **La dÃ©gressivitÃ© n'est pas postulÃ©e : elle est la consÃ©quence arithmÃ©tique de la loi du temps.**
- **DÃ©cote minimale :** `Î´_min = frais_AR(ask) + 0,01`, soit **0,0207 Ã  0,0375 pt** selon l'ask. Sous ce seuil, l'espÃ©rance est nÃ©gative quel que soit le sens du sous-jacent. **Non nÃ©gociable.**
- **DÃ©cote maximale :** **non dÃ©terminable** â€” aucun fill dans le corpus. Le maximum de simple contact du BBO observÃ© sur 2 s est de **0,700 pt (70 ticks)**, mais un contact n'est pas une exÃ©cution. Borne de sÃ©curitÃ© provisoire : `Î´_max = 0,114 pt`, Ã©gale au coupe-circuit, au-delÃ  de quoi le bot n'affiche plus de prix et journalise une alerte.
- **RÃ¨gle de re-cotation quand l'oracle bouge entre deux phases :** annulation puis recotation si `|z(Ï„) âˆ’ z(Ï„_envoi)| Ã— dÎ¦/dz > Î´(Ï„)`, c'est-Ã -dire si le dÃ©placement de l'ancrage dÃ©passe la dÃ©cote elle-mÃªme. Ã€ chaque franchissement de palier (`Ï„ = 240 s`), annulation et recotation systÃ©matiques, mÃªme si `z` n'a pas bougÃ©, parce que `Î´` change. DÃ©lai maximal d'annulation : **â‰¤ 500 ms**, Ã  mesurer.
- **Comportement si l'oracle traverse le seuil de rÃ©solution alors qu'un ordre est en attente :** **annulation immÃ©diate et inconditionnelle**, avant tout autre traitement. Justification : la traversÃ©e fait passer le cÃ´tÃ© ancrÃ© de l'autre bord ; un ordre en vol devient instantanÃ©ment une position sur le cÃ´tÃ© que l'ancrage vient de dÃ©signer comme perdant, et le corpus mesure exactement ce piÃ¨ge (Â« *Ã  `Ï„ = 240 s` l'oracle vaut 64 817,64 $, donc Â« UP Â», pendant que le carnet cote NO Ã  0,20 $ â€” puis l'oracle s'effondre de 22 $ en 15 secondes et l'issue est DOWN* Â»). AprÃ¨s annulation, le compteur `Xâ‚†â‚€` passe Ã  1 et **interdit toute recotation pendant 60 secondes**, ce qui dans la fenÃªtre `[180;270]` Ã©quivaut le plus souvent Ã  la fin de la session. C'est voulu : une traversÃ©e tardive du strike est la signature d'une session non tradable.

## 8.5 Remarques et fonctions rejetÃ©es comme directionnelles

| NÂ° | Source | Citation | Pourquoi c'est un pari | Requalification structurelle possible |
|---|---|---|---|---|
| 5.5 / R14, R15 | T5 `BASIS_CORROMPU`, PDF p.6 + 12 agents | Â« Le spot doit Ãªtre supÃ©rieur Ã  l'oracle Â» | Le miroir `spot < oracle` n'est pas traitÃ© par une action miroir ; le test est vrai 1943/1943 fois uniquement parce que le signe observÃ© est positif | **Oui** : contrÃ´le de cohÃ©rence de source `|basis âˆ’ mÃ©diane_glissante_60s| â‰¤ 3Â·Ïƒ_session`, fail-closed |
| 5.12 / R? | T12 `RETOURNEMENT`, PDF p.7 | Â« `[m*(t) âˆ’ m*(tâˆ’15)]/15 s â‰¥ âˆ’0,40 $/s` Â» | Borne unilatÃ©rale : interdit la pente nÃ©gative et autorise toute pente positive ; sÃ©lectionne une trajectoire | **Oui** : garde symÃ©trique `|Î”m*_15|/15 â‰¤ 0,45 $/s` |
| 5.19 / R30 | T19 `MODELE_NON_CALIBRE`, PDF p.9 | Â« `Brier(p) â‰¤ Brier(prix du carnet)` Â» | L'agrÃ©gat passe (0,4345 vs 0,5334) **uniquement par compensation** : modÃ¨le 0,6787 vs mid 0,5543 sous le strike, 0,0194 vs 0,4979 au-dessus | **Oui** : garde conservÃ©e mais exigÃ©e **dans chaque sous-groupe** ; `p` reste un score de diagnostic |
| R28 | `sign(m*)` comme prÃ©dicteur, 2 agents | Â« `sign(m*)` pointe le gagnant 78,8 % du temps en agrÃ©gÃ©, mais en session B il pointe UP Ã  0â€“13 % de raison pendant 90 s consÃ©cutives Â» | Le signe **est** la direction ; 78,8 % en agrÃ©gÃ© masque 90 s consÃ©cutives Ã  0â€“13 % | **Oui, partiellement** : le signe est conservÃ© **uniquement** comme transformation miroir du cÃ´tÃ© cotÃ©, jamais comme condition ; c'est `Ï„ â‰¥ 180 s` et `z â‰¥ 1,0` qui le rendent fiable, pas lui-mÃªme |
| R29, R39 | `D` signÃ© comme aubaine, 3 agents | Â« Le signal de valeur du postulat est ici anti-corrÃ©lÃ© au rÃ©sultat Â» ; Â« `D_proxy` atteint 0,85744 et croÃ®t en fin de session Â» | Les trois plus grosses sous-Ã©valuations dÃ©tectÃ©es sont **toutes du mauvais cÃ´tÃ©** ; le signe de `D` change de camp avec l'issue | **Oui** : `e(Ï„) = |p_ancrÃ© âˆ’ ask|` en **valeur absolue**, cÃ´tÃ© dÃ©duit par miroir |
| R34, R35 | Plancher et plafond d'ask, 2 agents | S1005 : Â« acheter Ã  `ask â‰¤ 0,45` Â» (142/142 gagnants) â€” S1010 : Â« ne jamais acheter sous 0,50 Â» (`ask < 0,50` â†’ **0/19**) | Contradiction frontale entre deux sessions : les deux rÃ¨gles dÃ©crivent le cÃ´tÃ© qui a gagnÃ© sur leur session, pas une structure | **Oui** : remplacÃ© par le **test d'espÃ©rance nette** `(1 âˆ’ ask) > frais_AR_pnl + 0,02`, qui est symÃ©trique |
| R36 | `basis` en rÃ©gression sur `Î”oracle`, 2 agents | Â« le basis spot âˆ’ oracle comme unique variable indÃ©pendante ayant une avance causale rÃ©elle (RÂ² = 0,23, r = +0,47 sur `Î”oracle+30 s`) Â» | PrÃ©dit le **mouvement futur** de l'oracle. RÃ¨gle de consensus 7 appliquÃ©e : rejetÃ©e mÃªme si c'est la seule avance causale mesurÃ©e du corpus | **Non pour l'emploi prÃ©dictif.** Le basis est conservÃ© uniquement comme contrÃ´le d'intÃ©gritÃ© (ligne 1) |
| R? | `momentum_spot_5s`, `pente_m_15s` (champs journalisÃ©s du PDF p.12) | Champs 10 et 11 des 24 champs de journalisation | Grandeurs de flux : momentum et pente. Aucune rÃ¨gle ne doit les consommer | **Journalisation conservÃ©e, consommation interdite.** Ils servent l'analyse hors ligne, jamais la dÃ©cision |
| R? | `Î¦â‚ƒâ‚€` flux net adverse, PDF p.10 | Â« Le volume signÃ© `Î¦` est journalisÃ© en continu Ã  titre d'indicateur Â» | Information de flux par dÃ©finition ; et de toute faÃ§on **non dÃ©terminable** (`direction` est un code Â±1 sans dictionnaire) | **Non.** RetirÃ©. Le champ `side` est Ã  journaliser (Â§ 8.6) ; sa requalification sera rÃ©examinÃ©e quand le cÃ´tÃ© agresseur sera reconstructible |
| R? | T14 `lag_signÃ©` | Â« lag_signÃ© âˆˆ [+5 ; +60] s Â» | **Ce n'est pas un pari, c'est une rÃ©futation** : le phÃ©nomÃ¨ne n'existe pas (0/27 sessions). Inscrit ici pour mÃ©moire | **Oui** : indicateur de santÃ© journalisÃ©, alerte non bloquante si la mÃ©diane sur 10 sessions dÃ©passe +5 s |

## 8.6 Points non dÃ©terminables et donnÃ©es Ã  journaliser sur les 500 sessions

| Point | Champ Ã  journaliser | Ã‰lÃ©ment source | Horloge et prÃ©cision requise | Ce qu'il permettrait de trancher |
|---|---|---|---|---|
| **Strike officiel** | `K`, `k_source`, payload TWAP 30 s publiÃ© | Source officielle de rÃ©solution | MarchÃ© absolue, â‰¤ 100 ms | **Le point le plus lourd de tout l'audit.** Un Ã©cart de 12,25 $ inverse le rÃ¨glement d'une session ; sans `K`, aucune affectation Ã  un sous-groupe n'est fiable, donc aucun PnL par sous-groupe n'est interprÃ©table |
| **Origine de fenÃªtre** | `endDate`, date absolue | Source officielle | MarchÃ© absolue, â‰¤ 100 ms | Rend tous les seuils en secondes exacts au lieu d'approximatifs ; valide la fenÃªtre `[180;270]` |
| **Budget de latence du bot** | `t_reception_oracle`, `t_reception_bbo`, `t_decision`, `t_envoi`, `t_ack`, `t_fill`, `t_annul_envoi`, `t_annul_ack` | Bot + moteur d'appariement | Serveur commune, â‰¤ 10 ms | La part des Ã©vÃ©nements rÃ©ellement atteignables. **Sans ces 8 champs, aucun mÃ©canisme ne peut Ãªtre dÃ©clarÃ© atteignable** (rÃ¨gle de travail 13) |
| **Taux de remplissage rÃ©el** | `order_id`, prix et quantitÃ© exÃ©cutÃ©s, `maker/taker`, rejets avec code | Moteur | Serveur, â‰¤ 10 ms | Transforme la table 8.4 bis de Â« contacts BBO Â» en Â« remplissages Â» ; **c'est la seule mesure qui rende la dÃ©cote calibrable** |
| **Profondeur du carnet** | 12 niveaux `(prix, quantitÃ©)` du cÃ´tÃ© cotÃ©, plus les annulations | Carnet CLOB niveau 2 | Capture carnet, â‰¤ 100 ms, 4 Hz min | La capacitÃ© du mÃ©canisme M1 et le dÃ©rapage rÃ©el ; dÃ©bloque T21, aujourd'hui fail-closed permanent |
| **Ã‰tat interne** | `position`, `n_ordres_en_vol`, rÃ©conciliation API | Bot + API | Locale + serveur | DÃ©bloque T22, aujourd'hui fail-closed permanent |
| **Issue officielle** | `issue_officielle`, TWAP `[270;300]` complet | RÃ©solution | MarchÃ© | Le PnL par sous-groupe, donc la forme opÃ©rationnelle du test de non-directionnalitÃ©. Le corpus n'a que des TWAP partiels `[270;298]` |
| **Latence causale oracle** | `ts_src` **numÃ©rique** (heure de publication Ã  la source), heure de rÃ©ception plateforme, heure de capture locale, sÃ©parÃ©es | Oracle | Serveur synchronisÃ©e, â‰¤ 10 ms | Si la latence oracle â†’ carnet existe rÃ©ellement Ã  une Ã©chelle sous-seconde. Aujourd'hui `ts_src` vaut la chaÃ®ne `payload` sur 269/269 lignes : **le champ est inutilisable** |
| **CÃ´tÃ© agresseur des trades** | `side`, `price`, `maker/taker`, dictionnaire du champ `direction` | Flux de trades | Capture, 1 ms | Reconstruit `Î¦â‚ƒâ‚€` et permet d'identifier les populations. Aujourd'hui `direction` est un code Â±1 **sans sÃ©mantique documentÃ©e** |
| **Populations et contreparties** | `order_id` anonymisÃ© stable, taux d'annulation par acteur, dÃ©lai aprÃ¨s oracle | Carnet + moteur | Serveur, â‰¤ 10 ms | **Nommer la contrepartie** (rÃ¨gle de travail 11), aujourd'hui impossible : Â« *le faire serait une attribution inventÃ©e* Â» |
| **Drapeau HALT** | `HALT` et ses transitions | Plateforme | Serveur, ms | Valide la seconde moitiÃ© de T5, aujourd'hui invÃ©rifiable |
| **Frais rÃ©els** | `fee_bps` au prÃ©-vol et Ã  chaque rotation | API CLOB | â€” | Valide la garde de frais `fee_bps/10â´Â·min(ask;1âˆ’ask) â‰¤ 0,02Â·askÂ·(1âˆ’ask)` |
| **Carnets concurrents** | BBO d'au moins une session concurrente sur le mÃªme sous-jacent | Carnets concurrents | Capture commune, â‰¤ 100 ms | La source d'edge nÂ°4 du cahier des charges, **intÃ©gralement non mesurÃ©e sur 41 sessions** |
| **Tick officiel** | Pas de cotation dÃ©clarÃ© par l'API | Plateforme | â€” | Aujourd'hui `0,01` est **infÃ©rÃ©** des valeurs du BBO, ce n'est pas un champ de schÃ©ma |
| **`lag_signÃ©` sur 10 sessions** | CorrÃ©lation croisÃ©e `Î”oracle â†” Î”mid` avec identifiant de session, Ã  4 Hz | Oracle + carnet | Serveur commune, â‰¤ 100 ms | Si le phÃ©nomÃ¨ne du PDF rÃ©apparaÃ®t. Alerte non bloquante au-dessus de +5 s |
| **UniversalitÃ© de `Ïƒâ‚ƒâ‚€`** | `Ïƒâ‚ƒâ‚€` mesurÃ© par session, avec rÃ©gime de volatilitÃ© annualisÃ© | Oracle | `Ï„`Â·marchÃ© | Le corpus couvre **une seule journÃ©e** (18/08/2026) avec une volatilitÃ© annualisÃ©e implicite de **6,9 %**, contre 40â€“60 % typiques pour BTC. **Toutes les valeurs de ce livrable sont issues d'un rÃ©gime anormalement calme** |
| **`z_min`** | `z` Ã  l'entrÃ©e et issue, par sous-groupe | Oracle + rÃ©solution | `Ï„`Â·marchÃ© | Si `z_min = 1,0` est le bon seuil, ou s'il faut le ramener Ã  0 |
| **`D_max`** | `e(Ï„)` Ã  chaque dÃ©cision et Ã  chaque fill | ModÃ¨le + carnet | Capture carnet | Si le coupe-circuit Ã  0,114 protÃ¨ge ou coÃ»te. Le corpus mesure un PnL **supÃ©rieur** sur les instants rejetÃ©s (+1,91 $ contre +1,30 $) |

**Format de journalisation.** Les 24 champs du PDF p.12 sont conservÃ©s (`t`, `cote`, `k_source`, `K`, `oracle`, `ask`, `bid`, `spread`, `m_etoile`, `pente_m_15s`, `momentum_spot_5s`, `M`, `lambda`, `p_chapeau`, `D`, `D_il_y_a_2s`, `x60`, `sigma_30`, `profondeur_12_niveaux`, `phi_30`, `C`, `N`, `issue_officielle`, `pnl`), avec les trois rÃ¨gles du PDF : ligne Ã©crite **avant** l'envoi de l'ordre, refus journalisÃ©s avec le **code atomique du test bloquant**, seconde ligne Ã  la clÃ´ture avec issue officielle et PnL, reliÃ©e par l'identifiant de session. **Deux ajouts obligatoires :** les 8 horodatages de latence, et le sous-groupe (`oracle en hausse/baisse`, `OUI/NON gagne`) calculÃ© Ã  la clÃ´ture. `pente_m_15s` et `momentum_spot_5s` restent journalisÃ©s mais **aucune rÃ¨gle ne les consomme**.

## 8.7 Fonctions structurelles manquantes

### 8.7.1 â€” `INVARIANT_ROMPU` (contrÃ´le de complÃ©mentaritÃ© du carnet)
- **CatÃ©gorie :** Structurelle.
- **RÃ´le vis-Ã -vis de l'ancrage :** **sert.** Il garantit que le prix du cÃ´tÃ© cotÃ© est cohÃ©rent avec celui du cÃ´tÃ© opposÃ©, donc que l'ancrage est comparÃ© Ã  un carnet valide.
- **Termes et grandeurs :** `bid_OUI`, `ask_NON`, `ask_OUI`, `bid_NON`, tick.
- **Ã‰lÃ©ments concernÃ©s :** carnet, moteur de cotation.
- **Relation :** `|bid_OUI + ask_NON âˆ’ 1| â‰¤ 1 tick` et `|ask_OUI + bid_NON âˆ’ 1| â‰¤ 1 tick`. MesurÃ© exact sur **1 025/1 025** et 1 024/1 026 lignes non manquantes, Ã©cart maximal 0,01. Ce que cela rÃ©vÃ¨le : la contrainte est imposÃ©e par le moteur ; un Ã©cart signale une donnÃ©e corrompue ou un cÃ´tÃ© balayÃ© non encore rÃ©percutÃ©. Test de non-directionnalitÃ© : Oui, l'invariant tient quel que soit le cÃ´tÃ© balayÃ©. Contrepartie : ordres passifs du cÃ´tÃ© non recalculÃ©, non identifiables. ConsÃ©quence : fail-closed, et dÃ©rivation du cÃ´tÃ© NON depuis le cÃ´tÃ© OUI.
- **Marque comportementale :** 2 lignes sur 1 026 prÃ©sentent un Ã©cart d'exactement 1 tick.
- **Ce que le PDF proposait :** rien. La fonction est absente des 22 tests.
- **Ce que les agents ont observÃ© :** Â« *le BBO respecte presque exactement la complÃ©mentaritÃ© mÃ©canique `bid_OUI + ask_NON = 1` ; cet invariant ne constitue pas un edge* Â» ; Â« *invariant exact sur 1025 lignes non manquantes* Â». n_confirmant **6** / n_mesurÃ© **6**.
- **Test de non-directionnalitÃ© :** Oui. PnL par sous-groupe : 0,00 $ dans les quatre (aucune position).
- **Calibrage retenu :** seuil **1 tick = 0,01 pt**, mesurÃ© Ã  chaque snapshot de carnet, connu Ã  la rÃ©ception, horloge de capture du carnet. Fail-closed.
- **FenÃªtre, budget, contrepartie :** instantanÃ© ; budget nul ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1. Coter contre un carnet incohÃ©rent est une exposition pure au sens.
- **RÃ¨gle d'action 8.7.1 :** SI `|bid_OUI + ask_NON âˆ’ 1| > 0,01` OU `|ask_OUI + bid_NON âˆ’ 1| > 0,01` ALORS aucune cotation, annulation, code `INVARIANT_ROMPU` AU MOMENT de chaque mise Ã  jour du carnet JUSQU'Ã€ retour de l'invariant dans le tick.

### 8.7.2 â€” `TRAVERSEE_SEUIL` (annulation sur traversÃ©e du strike)
- **CatÃ©gorie :** MÃ©canique.
- **RÃ´le :** **sert.** C'est la protection qui rend M1 non directionnel dans le pire cas.
- **Termes et grandeurs :** `signe(m*(Ï„))`, `signe(m*(Ï„_envoi))`, ordres en vol.
- **Ã‰lÃ©ments concernÃ©s :** oracle, strike, moteur d'appariement.
- **Relation :** `signe(m*(Ï„)) â‰  signe(m*(Ï„_envoi))` avec un ordre en vol. Ce que cela rÃ©vÃ¨le : le cÃ´tÃ© ancrÃ© vient de changer de bord ; l'ordre en vol est dÃ©sormais sur le cÃ´tÃ© que l'ancrage lui-mÃªme dÃ©signe comme perdant. Le corpus mesure ce piÃ¨ge en clair : Ã  `Ï„ = 240 s` un oracle Ã  `+4,64 $` du strike, donc Â« UP Â», pendant que le carnet cote NO Ã  0,20 â€” puis l'oracle s'effondre de 22 $ en 15 s et l'issue est DOWN. Test de non-directionnalitÃ© : Oui, la rÃ¨gle se dÃ©clenche sur les deux sens de traversÃ©e. Contrepartie : celui qui prend l'autre cÃ´tÃ© de l'ordre pÃ©rimÃ©. ConsÃ©quence : annulation inconditionnelle et prioritaire.
- **Marque comportementale :** les traversÃ©es sont **groupÃ©es** (7 changements de signe entre 82 et 154 s sur une session, puis aucun) ; une traversÃ©e tardive est la signature d'une session non tradable.
- **Ce que le PDF proposait :** rien d'explicite pour un ordre en attente. `[PDF]` documente le cas de la traversÃ©e de seuil (durÃ©e 0,300 s, manquÃ©e 70 % du temps Ã  1 Hz) mais n'en tire aucune rÃ¨gle d'annulation.
- **Ce que les agents ont observÃ© :** Â« *Ã  `Ï„ = 240 s` l'oracle vaut 64 817,64 $ (`m* = +4,64 $`, donc Â« UP Â») pendant que le carnet cote NO Ã  0,20 $ â€” puis l'oracle s'effondre de 22 $ en 15 secondes et l'issue est DOWN [...] C'est la dÃ©finition mÃªme d'une session piÃ¨ge* Â» ; Â« *comportement si l'oracle traverse le seuil de rÃ©solution alors qu'un ordre est en attente* Â» (question explicitement posÃ©e par 4 agents, restÃ©e sans rÃ©ponse chiffrÃ©e). n_confirmant **5** / n_mesurÃ© **5**.
- **Test de non-directionnalitÃ© :** Oui. PnL par sous-groupe : non dÃ©terminable ; le bÃ©nÃ©fice est en pertes Ã©vitÃ©es.
- **Calibrage retenu :** dÃ©clenchement au **premier tick d'oracle** montrant un signe opposÃ©. DÃ©lai d'annulation cible **â‰¤ 500 ms**, Ã  mesurer. AprÃ¨s annulation, `Xâ‚†â‚€` passe Ã  1, ce qui **interdit toute recotation pendant 60 s** â€” dans la fenÃªtre `[180;270]`, cela met le plus souvent fin Ã  la session. Horloge : `Ï„`Â·marchÃ© pour la dÃ©tection, serveur pour l'annulation. Cadence de dÃ©tection **4 Hz minimum** : la traversÃ©e de seuil dure 0,300 s et est manquÃ©e 70 % du temps Ã  1 Hz.
- **FenÃªtre, budget, contrepartie :** la traversÃ©e dure 0,300 s ; **budget d'annulation requis < 300 ms**, non dÃ©terminable aujourd'hui ; contrepartie : preneur de l'ordre pÃ©rimÃ©, non identifiable.
- **Justification par le critÃ¨re :** critÃ¨re 1. C'est le seul cas oÃ¹ M1 peut devenir directionnel malgrÃ© toutes ses gardes.
- **RÃ¨gle d'action 8.7.2 :** SI `signe(m*(Ï„)) â‰  signe(m*(Ï„_envoi))` ET un ordre est en vol ALORS annulation immÃ©diate et inconditionnelle, avant tout autre traitement, code `TRAVERSEE_SEUIL`, puis `Xâ‚†â‚€ := 1` AU MOMENT de chaque tick d'oracle JUSQU'Ã€ `Xâ‚†â‚€ = 0`, soit 60 s au moins.

### 8.7.3 â€” `REGIME_NON_REPRESENTATIF` (garde d'universalitÃ© de la volatilitÃ©)
- **CatÃ©gorie :** Structurelle.
- **RÃ´le :** **sert.** Tous les paramÃ¨tres du livrable sont issus d'une seule journÃ©e Ã  volatilitÃ© anormalement basse ; sans cette garde, le bot appliquera un calibrage de rÃ©gime calme Ã  un rÃ©gime agitÃ©.
- **Termes et grandeurs :** `Ïƒâ‚ƒâ‚€(Ï„)` mesurÃ©, volatilitÃ© annualisÃ©e implicite, mÃ©diane glissante de `Ïƒâ‚ƒâ‚€` sur 20 sessions.
- **Ã‰lÃ©ments concernÃ©s :** oracle.
- **Relation :** `Ïƒâ‚ƒâ‚€_session / mÃ©diane_glissante_20_sessions(Ïƒâ‚ƒâ‚€) âˆˆ [0,4 ; 2,5]`. Ce que cela rÃ©vÃ¨le : le corpus mesure une volatilitÃ© annualisÃ©e implicite de **6,9 %** (9,5 / 5,8 / 5,3 % par session) alors que BTC tourne typiquement Ã  **40â€“60 %**. Les vingt-cinq fenÃªtres analysÃ©es couvrent **une seule journÃ©e**, le 18/08/2026. Test de non-directionnalitÃ© : Oui, un rapport d'Ã©carts-types est insensible au signe. Contrepartie : aucune, c'est un garde-fou. ConsÃ©quence : refuser d'agir quand le rÃ©gime sort de la plage sur laquelle les paramÃ¨tres ont Ã©tÃ© mesurÃ©s.
- **Marque comportementale :** `Ïƒâ‚ƒâ‚€` varie d'un facteur **6,6 intra-session** (3,01 â†’ 19,89 $) et d'un facteur **7,6 inter-session** (1,52 â†’ 11,48 $).
- **Ce que le PDF proposait :** la rÃ©serve est Ã©crite en page 17 (Â« *`Ïƒâ‚ƒâ‚€ = 26,50 $` est issu de deux jours d'un seul rÃ©gime de volatilitÃ©* Â») mais **aucune fonction ne l'implÃ©mente**.
- **Ce que les agents ont observÃ© :** Â« *ces trois fenÃªtres de 5 minutes sont des pÃ©riodes anormalement calmes, et non un rÃ©gime reprÃ©sentatif. **Toute constante calibrÃ©e dessus est spÃ©cifique Ã  ce calme.*** Â» ; Â« *`Ïƒâ‚ƒâ‚€` n'est pas une constante [...] figer `Ïƒâ‚ƒâ‚€` rÃ©introduirait exactement l'erreur de +19,07 points* Â». n_confirmant **4** / n_mesurÃ© **4**.
- **Test de non-directionnalitÃ© :** Oui. PnL par sous-groupe : 0,00 $ dans les quatre en cas de refus.
- **Calibrage retenu :** rapport `âˆˆ [0,4 ; 2,5]`, sans dimension, mesurÃ© Ã  `Ï„ = 30 s` puis Ã  chaque tick, mÃ©diane glissante sur les **20 sessions prÃ©cÃ©dentes**, horloge `Ï„`Â·marchÃ©. Hors bornes : aucune cotation, alerte, et **rÃ©vision obligatoire de `Î´_guard`** avant de reprendre.
- **FenÃªtre, budget, contrepartie :** Ã©valuation Ã  `Ï„ = 30 s` ; contrepartie aucune.
- **Justification par le critÃ¨re :** critÃ¨re 1. Un `Î´_guard` de 2 ticks calibrÃ© sur un rÃ©gime Ã  6,9 % de volatilitÃ© annualisÃ©e est trÃ¨s infÃ©rieur au mouvement adverse d'un rÃ©gime Ã  50 % : les fausses entrÃ©es laisseraient alors le bot pleinement exposÃ© au sens du sous-jacent.
- **RÃ¨gle d'action 8.7.3 :** SI `Ïƒâ‚ƒâ‚€_session / mÃ©diane_glissante_20_sessions(Ïƒâ‚ƒâ‚€) âˆ‰ [0,4 ; 2,5]` ALORS aucune cotation sur la session, alerte, code `REGIME_NON_REPRESENTATIF` AU MOMENT de `Ï„ = 30 s` et Ã  chaque tick JUSQU'Ã€ retour dans la plage.

## 8.8 Ordre d'implÃ©mentation et tests d'acceptation

| Rang | MÃ©canisme ou rÃ¨gle | DÃ©pendances | Test d'acceptation sur les 500 sessions | CritÃ¨re de retrait |
|---|---|---|---|---|
| **1** | **Journalisation** : 24 champs + 8 horodatages + sous-groupe, 1 code de refus atomique par test (22 codes, pas 14) | Aucune. **Aucune ligne de code de trading avant celle-ci** | 100 % des dÃ©cisions **et des refus** journalisÃ©s, avec code atomique unique attribuable ; 0 ligne Ã©crite aprÃ¨s confirmation | Si le taux de journalisation < 100 %, tout le reste est invalide |
| **2** | **Champs officiels** : `endDate`, `K`, `k_source`, `HALT`, `fee_bps`, tick | Rang 1 | Les 4 champs prÃ©sents et `officiel` sur â‰¥ 99 % des sessions ; Ã©cart `K_officiel âˆ’ K_proxy` mesurÃ© et publiÃ© | Si `K` officiel indisponible : **arrÃªt du projet en l'Ã©tat.** C'est le point qui Â« pÃ¨se plus lourd que tout le reste Â» |
| **3** | **M2 â€” ContrÃ´le d'intÃ©gritÃ©** (rÃ¨gles 1, 2, 3, 4, 5-requal., 6, 7, 20, 8.7.1) | Rangs 1â€“2 | 0 cotation sur donnÃ©es incohÃ©rentes ; taux de refus par code cohÃ©rent avec les mesures du corpus (invariant rompu â‰ˆ 0,2 %, spread hors bornes â‰ˆ 14 %) ; **PnL 0,00 $ dans chaque sous-groupe** | Si un seul ordre part sur un snapshot incohÃ©rent |
| **4** | **`Ïƒâ‚ƒâ‚€` mesurÃ© en ligne** + `Ïƒ_T`, `Î»`, `z`, `p_ancrÃ©` (rÃ¨gles 4, 9) et **garde 8.7.3** | Rang 3 | MÃ©diane de `Ïƒâ‚ƒâ‚€` sur 500 sessions publiÃ©e avec P10/P90 ; **0 usage de constante gelÃ©e** ; volatilitÃ© annualisÃ©e implicite publiÃ©e par session | Si `Ïƒâ‚ƒâ‚€` sort de `[0,5 ; 40] $` sur > 5 % des sessions, l'estimateur est faux |
| **5** | **Profondeur niveau 2** (rÃ¨gle 21) et **rÃ©conciliation de position** (rÃ¨gle 22) | Rang 2 | Niveau 2 disponible sur â‰¥ 99 % des snapshots Ã  4 Hz ; rÃ©conciliation de position Ã  100 %, 0 position fantÃ´me | Sans ces deux champs, T21 et T22 restent fail-closed et **le bot ne cote jamais** |
| **6** | **M3 â€” Instrumentation du taux de remplissage** (rÃ¨gle 24) | Rangs 1â€“5 | Taux de fill rÃ©el mesurÃ© par phase et par `Î´ âˆˆ {0,01â€¦0,08}` sur â‰¥ 100 sessions ; table 8.4 bis reconstruite avec des **fills** et non des contacts ; **PnL par sous-groupe renseignÃ© pour la premiÃ¨re fois** | Si le taux de fill Ã  `Î´ = 0,02` en phase `[180;240[` est < 30 % (contre 88 % de contact attendu), la thÃ¨se de la quote passive tombe |
| **7** | **FenÃªtre `[180;270]`, `Xâ‚†â‚€ = 0`, persistance 5 s** (rÃ¨gles 8, 10, 11) | Rang 6 | Nombre d'Ã©vÃ©nements Ã©ligibles par session publiÃ© ; **rÃ©partition des sessions par couleur** (`Xâ‚†â‚€ = 0` vs `> 0`) ; contrÃ´le que la fenÃªtre capte bien `Ï„_lock` | Si < 0,5 Ã©vÃ©nement Ã©ligible par session en moyenne, le mÃ©canisme ne passe pas Ã  l'Ã©chelle |
| **8** | **Quantiles mesurÃ©s en ligne** : `q95(chute)`, `q95(saut)`, spread (rÃ¨gles 13, 15, 20) | Rang 7 | Taux de refus par test compris entre 5 % et 30 % ; **aucun test ne refuse ni 0 % ni 100 %** des snapshots | Un test qui refuse 100 % est faux par construction : c'est exactement l'erreur de T14 et T15 |
| **9** | **Test d'espÃ©rance nette** remplaÃ§ant `P_max` (rÃ¨gle 18) et **coupe-circuit `D_max`** (rÃ¨gle 17) | Rang 8 | PnL net par Ã©vÃ©nement > 0 aprÃ¨s frais rÃ©els `0,02Â·ask(1âˆ’ask)` **dans chaque sous-groupe** ; distribution de `e(Ï„)` publiÃ©e avec la frÃ©quence de dÃ©passement de 0,114 | Si le PnL net par Ã©vÃ©nement est â‰¤ 0 dans un sous-groupe |
| **10** | **8.7.2 â€” Annulation sur traversÃ©e du strike** | Rang 9 | DÃ©lai d'annulation mesurÃ© â‰¤ 500 ms au P90 ; **0 fill sur un ordre dont le cÃ´tÃ© ancrÃ© a changÃ©** | Un seul fill sur ordre pÃ©rimÃ© suffit : la garde est binaire |
| **11** | **M1 â€” Cotation ancrÃ©e, taille rÃ©elle `N`** (rÃ¨gle 25) | Rangs 1â€“10 tous verts | **PnL agrÃ©gÃ© strictement positif dans CHACUN des quatre sous-groupes** (oracle en hausse / en baisse ; OUI gagne / NON gagne) sur â‰¥ 200 sessions, avec â‰¥ 100 sessions par sous-groupe. Ratio PnL/capital engagÃ© > +1,0 % par Ã©vÃ©nement rempli | **Un seul sous-groupe nÃ©gatif âŸ¹ retrait immÃ©diat.** Un PnL positif obtenu avec un sous-groupe nÃ©gatif est un pari, quel que soit l'agrÃ©gat |
| **12** | **`z_min`, `Î´_guard`, `D_max` â€” rÃ©vision** | Rang 11 sur 500 sessions | Balayage de `z_min âˆˆ {0 ; 0,5 ; 1,0 ; 1,5 ; 2,0}` et `Î´_guard âˆˆ {1 ; 2 ; 3 ; 5}` ticks, retenir la valeur qui garde le PnL positif **dans chaque sous-groupe**, jamais celle qui maximise l'agrÃ©gat | Si aucune valeur ne garde les quatre sous-groupes positifs, **le mÃ©canisme est retirÃ©**, pas rÃ©ajustÃ© |
| **13** | **`lag_signÃ©` â€” surveillance** (rÃ¨gle 14) | Rang 11 | MÃ©diane glissante sur 10 sessions publiÃ©e ; alerte si > +5 s | Aucun : c'est un indicateur, jamais une porte |
| **14** | **Calibration de Platt `(a,b)`** et rÃ©introduction Ã©ventuelle de `p` et `D` (rÃ¨gle 19) | Rang 11 + 50 sessions Ã©tiquetÃ©es | `Brier(p) â‰¤ Brier(mid)` **dans chacun des quatre sous-groupes**, coefficients versionnÃ©s | Si un seul sous-groupe Ã©choue, `p` reste un score et `D` reste hors de la porte, dÃ©finitivement |

**RÃ©serve statistique finale, qui s'applique Ã  la totalitÃ© de ce livrable.** Le corpus couvre **une seule journÃ©e**, le 18/08/2026, avec une volatilitÃ© annualisÃ©e implicite de **6,9 %** contre 40â€“60 % typiques pour BTC. Aucun taux de rÃ©ussite du corpus n'est validable au seuil de 95 % : un marqueur Ã  3/3 a un intervalle de confiance de Clopper-Pearson de **[29,2 % ; 100 %]**, et il faudrait **29 sessions consÃ©cutives sans faute** pour rejeter `Hâ‚€(p â‰¤ 0,90)`. Tout ce qui est Ã©tabli ici avec un haut niveau de confiance est **structurel** : l'absence de latence oracle â†’ carnet (0/27 sessions dans la bande), la faussetÃ© de `Ïƒâ‚ƒâ‚€ = 26,50 $` (facteur Ã—3,68, 29/29 sessions), la stÃ©rilitÃ© de la formule Ã  22 tests (0 entrÃ©e, 19/19 agents), l'exactitude de l'invariant de complÃ©mentaritÃ© (1025/1025 lignes), la permanence du basis (100 % des lignes appariÃ©es), et le fait que l'issue se verrouille aprÃ¨s la fermeture de la fenÃªtre du PDF. Tout ce qui est chiffrÃ© en PnL repose sur **1 Ã  2 trades simulÃ©s par session** et **ne doit pas Ãªtre extrapolÃ©**. La valeur de ce livrable n'est pas de prouver que le bot gagne : c'est de fixer la seule architecture qui **puisse** gagner sans parier, et la liste exacte des onze champs Ã  journaliser pour le dÃ©montrer.

---

## VÃ©rification finale

- [x] Section 0 prÃ©sente avec ses sept sous-sections ; chaque latence porte valeur, unitÃ© et horloge, ou Â« Non dÃ©terminable Â» avec le champ Ã  journaliser.
- [x] Section 5 contient exactement 22 blocs, 5.1 Ã  5.22, chacun avec ses 13 sous-champs.
- [x] Chaque bloc des sections 2, 5 et 7 porte le rÃ©sultat du test de non-directionnalitÃ© ; aucun bloc retenu n'utilise tendance, momentum, pente signÃ©e, pression ou direction probable.
- [x] Chaque bloc de la section 7 contient un pseudocode, une comparaison fenÃªtre / budget de latence, une capacitÃ©, des cas dÃ©gradÃ©s, et la rÃ©ponse Â« Oui Â» au test d'implÃ©mentabilitÃ©.
- [x] Section 8.2 contient exactement 22 rÃ¨gles numÃ©rotÃ©es plus 3 rÃ¨gles globales ; les rÃ¨gles hors pÃ©rimÃ¨tre (5, 12, 14) y figurent avec la mention.
- [x] Chaque valeur retenue porte unitÃ©, moment de mesure et moment de connaissance.
- [x] Chaque dÃ©cision de la section 2 cite au moins un passage textuel d'un agent et donne son comptage.
- [x] Chaque relation retenue nomme sa contrepartie, ou dÃ©clare explicitement qu'elle n'est pas identifiable avec le champ Ã  journaliser.
- [x] Les phrases exactes de 8.1, 8.2 et 8.3 sont prÃ©sentes mot pour mot.
- [x] Aucune section fusionnÃ©e, aucun bloc rÃ©sumÃ©.
- [x] Les deux horloges (`Ï„`Â·marchÃ© et capture du carnet) et les deux estimateurs de `Ïƒâ‚ƒâ‚€` sont signalÃ©s sÃ©parÃ©ment partout oÃ¹ ils coexistent.

**â€” FIN DU LIVRABLE DE L'Ã‰TAPE 4 â€”**