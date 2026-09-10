# Ã‰TAPE 5 â€” Formules de vente fondamentales

## Objet et rÃ¨gle de lecture

Cette Ã©tape transpose la mÃ©thode des Ã©tapes 1 Ã  4 au **cÃ´tÃ© sortie**. Une formule de vente n'est pas une formule d'achat inversÃ©e : elle rÃ©pond Ã  une autre question.

> **Vendre si, et seulement si, la liquidation maintenant offre une valeur nette supÃ©rieure Ã  la valeur attendue de la conservation de la position, ou si une condition de sÃ©curitÃ© invalide la conservation.**

Le mot important est *suffit*. Une condition de vente doit Ãªtre assez forte pour dÃ©clencher la sortie, pas seulement signaler qu'une sortie serait envisageable.

Le corpus disponible est celui dÃ©crit dans `ETAP4.MD` : trois sessions BTC 5 minutes, 543 instants analysÃ©s, mais seulement trois rÃ¨glements indÃ©pendants. Les donnÃ©es confirment surtout les mÃ©canismes de prix, de volatilitÃ© et de filtrage. Elles **ne valident pas encore une stratÃ©gie de vente**, car aucune vente anticipÃ©e n'a Ã©tÃ© observÃ©e dans le rejeu final.

---

## 1. Corpus, conventions et limites

### 1.1 Sources

- `ETAP4.MD`, calibration et audit quantitatif indÃ©pendant.
- PDF de l'Ã©tape 4, cartographie structurelle du systÃ¨me et provenance des variables.
- DonnÃ©es normalisÃ©es Ã©voquÃ©es dans `ETAP4.MD` : carnet, oracle, spot et trades, rÃ©solution nominale d'une seconde.

Le PDF indique notamment : oracle Ã  cadence nominale de 1 seconde, trous pouvant atteindre 7 secondes, carnet limitÃ© aux meilleurs niveaux `yes_bid / yes_ask / no_bid / no_ask`, absence de profondeur exploitable, moteur d'appariement et carnets concurrents non observables.

### 1.2 Notation

- `Ï„` : secondes Ã©coulÃ©es depuis l'ouverture de la fenÃªtre.
- `T = 300 s` : Ã©chÃ©ance thÃ©orique.
- `K` : strike de rÃ¨glement.
- `p_t` : probabilitÃ© estimÃ©e que le rÃ©sultat UP survienne Ã  l'Ã©chÃ©ance.
- `q_t` : probabilitÃ© estimÃ©e que **la position dÃ©tenue** gagne. Pour une position YES, `q_t = p_t`; pour une position NO, `q_t = 1 - p_t`.
- `b_t` : meilleur bid du cÃ´tÃ© dÃ©tenu, donc le prix immÃ©diatement encaissable en vendant.
- `a_t` : meilleur ask du cÃ´tÃ© dÃ©tenu.
- `s_t = a_t - b_t` : spread observable au meilleur niveau.
- `f^sell_t` : coÃ»t effectif de vente, incluant frais, impact et Ã©ventuel coÃ»t de transfert de prix. Il ne doit pas Ãªtre supposÃ© Ã©gal au coÃ»t aller-retour si seule la vente est exÃ©cutÃ©e.
- `V^hold_t` : valeur nette de conservation jusqu'au rÃ¨glement ou jusqu'Ã  une sortie ultÃ©rieure.
- `V^sell_t = b_t - f^sell_t` : valeur nette de vente immÃ©diate par unitÃ©.

### 1.3 Alerte sur les incohÃ©rences de provenance

`ETAP4.MD` contient deux sÃ©ries de chiffres pour `Ïƒâ‚ƒâ‚€` et deux conventions de fenÃªtre. Les rÃ©sultats principaux citent d'une part des mÃ©dianes oracle de 4,59 / 1,77 / 2,64 $, et d'autre part 11,48 / 5,75 / 3,05 $. Le mÃªme fichier cite aussi des strikes reconstruits diffÃ©rents selon le bloc consultÃ©. Ces divergences ne doivent pas Ãªtre Ã©crasÃ©es : **la formule de vente doit recevoir un strike validÃ© et une volatilitÃ© calculÃ©e avec une dÃ©finition unique**.

---

## 2. Inventaire exhaustif des sorties

Pour une position dÃ©jÃ  ouverte, quatre sorties normales et une situation pathologique couvrent l'espace logique :

1. **Vente anticipÃ©e exÃ©cutable** : un ordre de vente est rÃ©ellement publiable et remplit au bid observÃ©.
2. **RÃ¨glement gagnant** : conservation jusqu'Ã  l'Ã©chÃ©ance, paiement 1 par unitÃ©.
3. **RÃ¨glement perdant** : conservation jusqu'Ã  l'Ã©chÃ©ance, paiement 0.
4. **Expiration opÃ©rationnelle sans dÃ©cision** : plus de nouvelle donnÃ©e fiable ou plus de possibilitÃ© de traiter; la politique de sÃ©curitÃ© conserve, annule ou clÃ´t selon ce qui est techniquement possible.
5. **LiquiditÃ© absente ou non observable** : le meilleur bid disparaÃ®t, la profondeur est inconnue ou le remplissage n'est pas confirmÃ©. Ce n'est pas une vente rÃ©ussie; c'est un Ã©tat `SUSPENDU`.

La formule de vente ne doit donc jamais produire seulement `SELL` ou `HOLD`. Le codage minimal est : `SELL_FILLED`, `HOLD`, `SETTLE_WIN`, `SETTLE_LOSS`, `SUSPEND_DATA`, `SUSPEND_LIQUIDITY`, `UNKNOWN`.

---

## 3. Conditions fondamentales de vente

### V0 â€” ValiditÃ© de l'Ã©tat et fraÃ®cheur des donnÃ©es

**DÃ©finition.** DÃ©clencher une sortie de sÃ©curitÃ© si le timestamp oracle est absent, si le dernier tick dÃ©passe le dÃ©lai maximal admis, si le strike n'est pas rÃ©solu sans ambiguÃ¯tÃ©, ou si les champs bid/ask ne sont pas cohÃ©rents.

**Justification.** Une probabilitÃ© calculÃ©e Ã  partir d'une donnÃ©e morte n'est pas une probabilitÃ© exploitable. Le PDF signale des trous oracle allant jusqu'Ã  7 secondes et l'absence d'heure source; cette condition est mesurable seulement par rapport Ã  l'horloge de capture.

**Codage proposÃ©.** `STALE_ORACLE` aprÃ¨s `Î”oracle > 3 s` en rÃ©gime normal, et `STALE_ORACLE_HARD` Ã  `> 7 s`. Ces seuils sont des paramÃ¨tres de sÃ©curitÃ©, non des constantes empiriquement validÃ©es.

**Verdict.** **Ã€ conserver comme veto opÃ©rationnel, non comme signal de marchÃ©.** Si le systÃ¨me ne peut pas vendre, V0 donne `SUSPEND_DATA` plutÃ´t que de prÃ©tendre qu'une sortie a eu lieu.

### V1 â€” Vente Ã  valeur nette supÃ©rieure Ã  la conservation

**DÃ©finition.** Vendre si :

```text
V^sell_t = b_t - f^sell_t
V^sell_t >= V^hold_t + Îµ
```

Pour une approximation de conservation Ã  l'Ã©chÃ©ance, `V^hold_t â‰ˆ q_t`. La condition devient :

```text
b_t - q_t >= f^sell_t + Îµ
```

oÃ¹ `Îµ` est une marge de dÃ©cision contre l'erreur de modÃ¨le.

**Justification.** C'est la condition fondamentale de sortie par valeur. Elle ne vend pas parce que le prix est Â« haut Â»; elle vend parce que le bid net dÃ©passe la valeur attendue de la position conservÃ©e.

**Codage proposÃ©.** `D_sell = b_t - q_t`. Mesurer sÃ©parÃ©ment `D_sell`, le coÃ»t de vente et le taux de remplissage. Ne pas utiliser `p_t - ask_t`, qui est la dÃ©cote d'achat et non la condition de sortie.

**Verdict.** **Condition centrale, mais non validÃ©e par le corpus.** Les donnÃ©es donnent des dÃ©cotes d'achat, pas une sÃ©rie de ventes remplies. Le seuil doit rester dÃ©sactivÃ© tant que les frais de vente et le remplissage ne sont pas mesurÃ©s sÃ©parÃ©ment.

### V2 â€” Invalidation directionnelle persistante

**DÃ©finition.** Vendre si le signal relatif Ã  la position change de signe et reste dÃ©favorable pendant une durÃ©e `h` :

```text
sign(m*_t) != side(position)
pendant h secondes consÃ©cutives
```

Le seuil ne doit pas Ãªtre un simple tick. La valeur de travail est `h = 20 Ã  30 s`, Ã  recalibrer.

**Justification empirique.** Dans A_1450, la concordance du signe avec le rÃ¨glement tombe Ã  41,7 % dans `[60;120[`, avec `m* = +7,94 $` Ã  `Ï„=90`, puis `m* = -17,15 $` Ã  `Ï„=150`. Une invalidation persistante aurait une justification Ã©conomique claire dans cette zone. Un flip instantanÃ©, en revanche, risque le whipsaw.

**Verdict.** **Ã€ retenir comme stop de thÃ¨se, confiance faible Ã  moyenne.** Une seule session montre le scÃ©nario utile; aucune session UP ne permet de tester la symÃ©trie.

### V3 â€” Ã‰rosion de marge normalisÃ©e

**DÃ©finition.** Vendre si la marge directionnelle ne couvre plus une borne de bruit en ligne :

```text
|m*_t| < k_exit Â· Ïƒ_T(t)
```

avec une hystÃ©rÃ©sis : `k_exit` de sortie doit Ãªtre infÃ©rieur au seuil de rÃ©entrÃ©e, afin d'Ã©viter les bascules immÃ©diates.

**Justification.** Une position qui n'a plus de marge statistique ne mÃ©rite pas d'Ãªtre conservÃ©e par inertie. Mais `Ïƒâ‚ƒâ‚€ = 26,50 $` est rÃ©futÃ© par l'audit : le postulat surestime fortement la volatilitÃ© des sessions.

**Codage proposÃ©.** Calculer `Ïƒ_T(t)` Ã  partir d'incrÃ©ments oracle rÃ©cents, avec une dÃ©finition documentÃ©e et identique entre entrÃ©e et sortie. Interdire une constante hÃ©ritÃ©e de l'Ã©tape 3.

**Verdict.** **Diagnostic utile, dÃ©clencheur suspendu.** Il devient testable seulement aprÃ¨s rÃ©solution de la dÃ©finition de volatilitÃ© et validation hors Ã©chantillon.

### V4 â€” Explosion de volatilitÃ© avec invalidation

**DÃ©finition.** Vendre ou rÃ©duire si `Ïƒ_T(t)` dÃ©passe un multiple de sa mÃ©diane locale, par exemple `2Ã—`, **et** si le signal directionnel devient dÃ©favorable ou si `D_sell` est favorable.

**Justification.** Sur A_1450, l'amplitude de `Ïƒâ‚ƒâ‚€` varie d'un facteur 6,6 et l'explosion accompagne la zone oÃ¹ le signe devient trompeur. La volatilitÃ© seule n'indique pas le sens de la prochaine variation.

**Verdict.** **Ne pas en faire une vente autonome.** V4 peut augmenter la prudence, allonger `h`, rÃ©duire la taille ou autoriser V2; il ne doit pas dÃ©clencher seul une vente sans bid Ã©conomiquement acceptable.

### V5 â€” Prise de bÃ©nÃ©fice Ã  probabilitÃ© Ã©levÃ©e

**DÃ©finition.** Vendre si le bid atteint un niveau Ã©levÃ©, mais seulement si la valeur abandonnÃ©e en conservant est infÃ©rieure au risque et au coÃ»t de vente :

```text
b_t - f^sell_t >= q_t - L_tail
```

oÃ¹ `L_tail` est la perte de valeur maximale acceptable pour Ã©liminer le risque rÃ©siduel.

Une rÃ¨gle pratique candidate est `b_t >= 0,92`, mais elle n'est **pas** une formule validÃ©e : l'audit rapporte `ask > 0,92` sur 68 instants, pas des ventes au bid remplies.

**Justification.** Vendre Ã  0,92 abandonne au maximum 0,08 de valeur brute par unitÃ© si la position aurait gagnÃ©. En contrepartie, on supprime le risque de retournement final. Le PnL moyen observÃ© sur ces instants est seulement `+0,13 $` dans le rÃ©sumÃ© d'audit, mais ce chiffre dÃ©crit des instants, pas des transactions de vente.

**Verdict.** **Candidate prometteuse, non confirmÃ©e.** Ã€ tester avec `bid`, frais de vente, remplissage et rÃ©sultat contrefactuel jusqu'au rÃ¨glement. Ne pas confondre `ask > 0,92` avec `bid >= 0,92`.

### V6 â€” Verrou terminal de conservation

**DÃ©finition.** Au-delÃ  d'un temps `Ï„_lock`, ne plus vendre discrÃ©tionnairement; laisser la position aller au rÃ¨glement, sauf veto de sÃ©curitÃ© ou impossibilitÃ© technique.

La valeur de travail issue de l'audit est `Ï„_lock = 240 s`, soit les 60 derniÃ¨res secondes.

**Justification.** Ã€ l'approche de l'Ã©chÃ©ance, la valeur thÃ©orique converge vers 0 ou 1. Une vente tardive paie encore le spread et peut cÃ©der la quasi-totalitÃ© de la valeur terminale pour une protection devenue inutile. La concordance signe/rÃ¨glement est de 100 % dans `[180;240[` et `[240;300[` pour les trois sessions, mais cela repose sur trois issues.

**Verdict.** **Ã€ conserver comme rÃ¨gle de prudence, mais pas comme vÃ©ritÃ© statistique.** Le verrou est rationnel sous coÃ»t de transaction; il doit Ãªtre dÃ©sactivable pour un stop V0 ou V2 trÃ¨s fort.

### V7 â€” Spread et liquiditÃ© minimale

**DÃ©finition.** Une vente de marchÃ© est autorisÃ©e seulement si le spread est infÃ©rieur au seuil et si le bid reste prÃ©sent :

```text
s_t <= s_max  ET  b_t > 0  ET  fill_confirmed = true
```

La valeur `s_max = 0,02` est cohÃ©rente avec le corpus : le spread est infÃ©rieur ou Ã©gal Ã  0,02 sur 538 des 543 instants.

**Limite.** La profondeur, la quantitÃ© disponible et le moteur d'appariement ne sont pas observables. Un bid affichÃ© ne prouve donc pas qu'une taille de 5 $ sera remplie.

**Verdict.** **Ã€ conserver comme garde d'exÃ©cution.** Il est peu sÃ©lectif, mais sa fonction est la qualitÃ© d'exÃ©cution, pas la prÃ©diction.

### V8 â€” Annulation de la vente si le bid est en train de disparaÃ®tre

**DÃ©finition.** Ne pas exÃ©cuter une vente si le bid a chutÃ© ou a Ã©tÃ© retirÃ© entre l'observation et l'envoi, sauf scÃ©nario de sÃ©curitÃ© explicitement autorisÃ©.

**Justification.** Vendre dans un bid qui s'effondre transforme une alerte de marchÃ© en mauvaise exÃ©cution. Le PDF ne permet pas de distinguer une vraie absorption d'un ordre isolÃ©, car les quantitÃ©s et niveaux 2 manquent.

**Verdict.** **Codage `SUSPEND_LIQUIDITY`, jamais `HOLD` silencieux.** Le systÃ¨me doit indiquer qu'il n'a pas pu sortir.

---

## 4. ExhaustivitÃ© et cohÃ©rence logique

### 4.1 Formule de vente candidate

La formule fondamentale proposÃ©e est :

```text
SELL si
  data_valid
  ET execution_possible
  ET Ï„ < Ï„_lock
  ET (
       V^sell_t >= V^hold_t + Îµ
       OU invalidation_persistante
       OU prise_de_bÃ©nÃ©fice_avec_risque_acceptable
      )
```

Avec les exceptions suivantes :

```text
SUSPEND si data_stale OU liquiditÃ©_non_confirmÃ©e
SETTLE si Ï„ >= T et le marchÃ© rÃ¨gle
```

### 4.2 Ordre d'Ã©valuation

1. RÃ©soudre l'Ã©tat : strike, orientation, timestamp, prix et side dÃ©tenu.
2. Si donnÃ©es pÃ©rimÃ©es ou incohÃ©rentes : `SUSPEND_DATA`.
3. Si bid absent, spread excessif ou remplissage non confirmable : `SUSPEND_LIQUIDITY`.
4. Si `Ï„ >= T` : `SETTLE`.
5. Si `Ï„ >= Ï„_lock` : interdire les sorties discrÃ©tionnaires.
6. Tester en premier la prise de bÃ©nÃ©fice V5, car elle est indÃ©pendante de l'invalidation directionnelle.
7. Tester V2 avec persistance.
8. Tester V1, puis V3 comme confirmation diagnostique.
9. Sinon : `HOLD`.

Cet ordre Ã©vite la paire circulaire T14/T15 observÃ©e Ã  l'Ã©tape 4. Aucun test de vente ne doit dÃ©pendre d'un retard oracle-carnet supposÃ©; ce retard est mesurÃ© comme nul Ã  environ Â±2 secondes.

### 4.3 Contradictions Ã  interdire

- `SELL_FILLED` et `SUSPEND_LIQUIDITY` au mÃªme timestamp.
- `SELL_FILLED` aprÃ¨s `Ï„_lock`, sauf sortie de sÃ©curitÃ© explicitement journalisÃ©e.
- Vendre une position qui n'existe plus.
- DÃ©clarer une vente au bid en utilisant seulement l'ask.
- DÃ©clarer un gain de vente sans confirmation de remplissage.
- Utiliser `p_t` pour une position NO sans convertir en `q_t = 1-p_t`.
- Utiliser un strike infÃ©rÃ© et un strike ajustÃ© dans le mÃªme rejeu.

---

## 5. Verdict de la formule de vente

### Verdict global

**La formule de vente est logiquement dÃ©finissable mais empiriquement non dÃ©montrÃ©e.** La meilleure version actuelle est une formule Ã  trois motifs, encadrÃ©e par des gardes :

```text
SELL_FILLED â‡
  donnÃ©es fraÃ®ches
  ET bid rÃ©ellement exÃ©cutable
  ET spread acceptable
  ET Ï„ < 240 s
  ET (
       b_t - f^sell_t >= V^hold_t + Îµ
       OU signal opposÃ© persistant 20â€“30 s
       OU bid Ã©levÃ© avec perte de valeur abandonnÃ©e acceptable
     )
```

La sortie Ã  l'Ã©chÃ©ance reste le cas par dÃ©faut. Le systÃ¨me ne doit pas prÃ©tendre qu'une sortie anticipÃ©e est rentable tant que le bid, le coÃ»t de vente et le remplissage n'ont pas Ã©tÃ© enregistrÃ©s transaction par transaction.

---

## 6. Analyse empirique par condition

<table>
<tr><th>Code</th><th>Test</th><th>Mesurable dans le corpus</th><th>RÃ©sultat</th><th>Verdict</th></tr>
<tr><td>V0</td><td>FraÃ®cheur / validitÃ©</td><td>Partiellement</td><td>Trous oracle jusqu'Ã  7 s; heure source absente</td><td>Garde obligatoire</td></tr>
<tr><td>V1</td><td>Bid net supÃ©rieur Ã  conservation</td><td>Non, coÃ»t de vente sÃ©parÃ© absent</td><td>Aucune vente observÃ©e</td><td>Formule centrale, non validÃ©e</td></tr>
<tr><td>V2</td><td>Flip persistant</td><td>Oui, sur le signal</td><td>A_1450: concordance 41,7 % dans [60;120[</td><td>Candidate, faible Ã©chantillon</td></tr>
<tr><td>V3</td><td>Marge sous volatilitÃ©</td><td>Partiellement</td><td>Ïƒ posÃ© faux de 2,3Ã— Ã  8,7Ã— ou davantage selon la dÃ©finition</td><td>Recalibrer avant activation</td></tr>
<tr><td>V4</td><td>Explosion de volatilitÃ©</td><td>Oui</td><td>Facteur intra-session jusqu'Ã  6,6</td><td>Confirmation, pas dÃ©clencheur seul</td></tr>
<tr><td>V5</td><td>Prise de bÃ©nÃ©fice haute</td><td>Ask oui, bid non</td><td>68 instants avec ask &gt; 0,92</td><td>Ã€ tester avec bid et fills</td></tr>
<tr><td>V6</td><td>Verrou terminal</td><td>Oui</td><td>Concordance tardive Ã©levÃ©e, 3 sessions</td><td>RÃ¨gle prudente</td></tr>
<tr><td>V7</td><td>Spread / bid prÃ©sent</td><td>Oui, partiellement</td><td>538/543 avec spread â‰¤ 0,02</td><td>Garde d'exÃ©cution</td></tr>
<tr><td>V8</td><td>Bid qui disparaÃ®t</td><td>Non, profondeur absente</td><td>Pas de niveau 2 ni quantitÃ©</td><td>Suspendre, ne pas inventer</td></tr>
</table>

La puissance statistique reste le verrou principal : 3/3 issues favorables donnent un IC95 de Clopper-Pearson `[29,2 %; 100 %]`. Pour rejeter `H0: p â‰¤ 0,90` avec un sans-faute, il faut 29 sessions consÃ©cutives sans erreur. Les taux de 100 % de l'audit sont donc descriptifs, pas prÃ©dictifs.

---

## 7. MÃ©thodes existantes et interprÃ©tation

La littÃ©rature sur l'arrÃªt optimal modÃ©lise la dÃ©cision comme une sÃ©paration entre une **zone de continuation** et une **zone d'arrÃªt**. C'est exactement la structure requise ici : `HOLD` dans la continuation, `SELL` dans la stopping region. Les modÃ¨les d'arrÃªt optimal avec coÃ»ts de transaction dÃ©rivent gÃ©nÃ©ralement des frontiÃ¨res de sortie qui se dÃ©placent avec le temps restant, la volatilitÃ© et le coÃ»t de liquidation.

Trois rÃ©sultats sont directement utiles :

1. **Sans friction, conserver jusqu'Ã  l'Ã©chÃ©ance domine souvent une sortie anticipÃ©e** pour un payoff binaire si la valeur de marchÃ© est cohÃ©rente avec l'espÃ©rance conditionnelle.
2. **Avec spread, frais, financement ou risque de liquiditÃ©, une sortie anticipÃ©e peut devenir rationnelle** lorsqu'elle dÃ©passe la valeur de continuation nette.
3. **Une stratÃ©gie d'entrÃ©e et de sortie doit Ãªtre traitÃ©e comme un problÃ¨me de double arrÃªt**, et non comme deux filtres indÃ©pendants empilÃ©s.

RÃ©fÃ©rences de mÃ©thode :

- Merton, rÃ©sultat classique de non-exercice anticipÃ© dans certains cadres sans dividende, utile ici comme contre-hypothÃ¨se : sans friction et sans bÃ©nÃ©fice de liquidation, la sortie anticipÃ©e n'est pas automatiquement justifiÃ©e.
- Leung et Li, *Optimal Mean Reversion Trading with Transaction Costs and Stop-Loss Exit*, formulation explicite d'un problÃ¨me de double arrÃªt avec coÃ»ts et frontiÃ¨re de sortie.
- Liu et Mu, *Optimal Stopping Methods for Investment Decisions*, revue des modÃ¨les d'arrÃªt, de continuation et de dÃ©cision sÃ©quentielle.
- Travaux rÃ©cents sur l'entrÃ©e/sortie optimale avec coÃ»ts de transaction : ils confirment qu'il faut estimer sÃ©parÃ©ment la valeur de continuation, le coÃ»t de sortie et le risque de remplissage.

Ces mÃ©thodes justifient la forme de V1 et V2; elles ne valident pas leurs paramÃ¨tres sur les trois sessions du corpus.

---

## 8. DÃ©cision de codage

### 8.1 Ã‰tats et Ã©vÃ©nements

```text
POSITION_OPEN
  -> HOLD
  -> SELL_SIGNAL
  -> SELL_SUBMITTED
  -> SELL_FILLED
  -> SELL_PARTIAL
  -> SUSPEND_DATA
  -> SUSPEND_LIQUIDITY
  -> SETTLED_WIN / SETTLED_LOSS
```

Chaque dÃ©cision doit journaliser : `Ï„`, side, `K`, `p_t`, `q_t`, `bid`, `ask`, spread, frais estimÃ©s, `V_hold`, `D_sell`, signal actif, raison principale, rÃ©sultat du remplissage et rÃ©sultat contrefactuel Ã  l'Ã©chÃ©ance.

### 8.2 Pseudocode normatif

```python
def sell_decision(state):
    if not state.position_open:
        return "NO_POSITION"
    if not state.strike_valid or state.oracle_age > 3:
        return "SUSPEND_DATA"
    if state.tau >= 300:
        return "SETTLE"
    if state.bid <= 0 or state.spread > 0.02:
        return "SUSPEND_LIQUIDITY"
    if state.tau >= 240:
        return "HOLD_TO_SETTLEMENT"

    q = state.win_probability_for_held_side
    v_sell = state.bid - state.sell_fee
    v_hold = state.continuation_value

    if v_sell >= v_hold + state.epsilon:
        return "SELL_VALUE"
    if state.opposite_signal_persistent_seconds >= 20:
        return "SELL_INVALIDATION"
    if state.bid >= 0.92 and state.tail_loss_acceptable:
        return "SELL_PROFIT"
    return "HOLD"
```

Le `SELL_*` produit un signal, pas un gain. Le gain n'est comptÃ© qu'aprÃ¨s confirmation du fill.

### 8.3 ComptabilitÃ© des frais

L'Ã©tape 4 utilise `frais_AR(ask) = 0,07Â·askÂ·(1âˆ’ask) + 0,01` comme coÃ»t aller-retour dans le filtre d'achat. Cela ne doit pas Ãªtre rÃ©utilisÃ© automatiquement pour une sortie seule. Deux scÃ©narios doivent Ãªtre calculÃ©s sÃ©parÃ©ment :

- **Sortie anticipÃ©e** : frais d'entrÃ©e dÃ©jÃ  payÃ©s + frais de vente rÃ©ellement applicables + spread/impact.
- **RÃ¨glement** : frais d'entrÃ©e seulement, sauf preuve d'un coÃ»t de rÃ¨glement.

Cette distinction est importante : pÃ©naliser l'achat avec un aller-retour alors que la simulation conserve jusqu'au rÃ¨glement crÃ©e un biais de comparaison.

---

## 9. Moments optimaux de dÃ©clenchement

### 9.1 FenÃªtre de protection

La zone `[60;120[` est la fenÃªtre de risque de retournement identifiÃ©e sur A_1450. V2 peut Ãªtre Ã©valuÃ©e ici, mais avec persistance de 20 Ã  30 secondes. Aucun flip instantanÃ© ne doit dÃ©clencher seul une vente.

### 9.2 FenÃªtre de sortie par valeur

La zone candidate est `[120;240[`, car les entrÃ©es M1 observÃ©es se concentrent vers `Ï„ â‰ˆ 122 s`, alors que le marchÃ© gagne progressivement en conviction. V1 et V5 doivent rester guidÃ©es par le bid net, non par une horloge fixe.

### 9.3 Verrou terminal

Ã€ partir de `Ï„ = 240 s`, la recommandation est `HOLD_TO_SETTLEMENT`, sauf V0, V8 ou Ã©vÃ©nement de sÃ©curitÃ©. Cette rÃ¨gle limite les ventes tardives qui abandonnent le payoff terminal pour quelques centimes de liquiditÃ©.

### 9.4 Bande morte

Avec un spread de 0,02 et des frais aller-retour de l'ordre de 0,024 Ã  0,028 au voisinage de `ask = 0,50â€“0,71`, la zone de friction achat/vente est de l'ordre de 0,044 Ã  0,047 par unitÃ©. Une variation plus petite que cette bande ne justifie pas un aller-retour.

Sur les points M1 de B et C, la bande Ã©quivaut approximativement Ã  0,5â€“0,6 $ de mouvement oracle selon le spread. Elle est donc non nÃ©gligeable, mais faible face aux marges directionnelles de plusieurs dollars observÃ©es Ã  `Ï„ â‰ˆ 122 s`.

---

## 10. Relations entre conditions

| Relation | Analyse | DÃ©cision |
|---|---|---|
| V1 â†” V5 | Deux sorties fondÃ©es sur la valeur du bid; V5 est un cas de seuil pratique de V1 | V1 maÃ®tre, V5 proxy testable |
| V2 â†” V4 | Une explosion de volatilitÃ© accompagne parfois un flip; risque de double comptage | V2 dÃ©clenche, V4 confirme ou rÃ©duit |
| V3 â†” V1 | La marge et le prix de marchÃ© sont liÃ©es, mais pas identiques | V3 diagnostic, V1 exÃ©cutable |
| V6 â†” V1/V5 | Le verrou terminal bloque les sorties discrÃ©tionnaires | V6 a prioritÃ© sur les signaux ordinaires |
| V0 â†” tous | Une donnÃ©e pÃ©rimÃ©e invalide les autres conditions | V0 est le premier veto |
| V7 â†” tous | Le signal n'est pas exÃ©cutable si le bid ou le spread ne conviennent pas | V7 doit prÃ©cÃ©der `SELL_FILLED` |
| V8 â†” V7 | Un bid visible peut disparaÃ®tre avant le fill | Retour `SUSPEND_LIQUIDITY`, jamais faux fill |

La formule finale doit Ãªtre parcimonieuse. Empiler tous les tests recrÃ©erait le problÃ¨me T14/T15 : un filtre apparemment prudent, mais impossible Ã  dÃ©clencher ou fondÃ© sur une hypothÃ¨se non mesurÃ©e.

---

## 11. Verdict par condition et plan de falsification

### Ã€ activer maintenant

- V0, fraÃ®cheur et cohÃ©rence.
- V7, spread et prÃ©sence du bid.
- V6, verrou terminal comme politique prudente.
- Journalisation exhaustive de V1, V2 et V5 sans exÃ©cution automatique.

### Ã€ tester en simulation puis paper trading

- V1, vente si le bid net dÃ©passe la valeur de conservation.
- V2, flip persistant 20â€“30 secondes.
- V5, seuil de prise de bÃ©nÃ©fice autour de 0,92, uniquement sur le bid.

### Ã€ suspendre

- V3 tant que la volatilitÃ© n'a pas une dÃ©finition unique.
- V4 comme dÃ©clencheur autonome.
- Toute rÃ¨gle utilisant la profondeur, le moteur ou les carnets concurrents non observables.

### ExpÃ©rience minimale de l'Ã©tape suivante

Collecter au moins 29 sessions indÃ©pendantes, rÃ©parties entre UP et DOWN, avec :

1. strike officiel et source temporelle validÃ©s;
2. bid/ask Ã  chaque seconde et quantitÃ© disponible;
3. ordre de vente, acceptation, fill partiel ou complet, et timestamp d'exÃ©cution;
4. frais d'entrÃ©e et de sortie sÃ©parÃ©s;
5. valeur contrefactuelle au rÃ¨glement;
6. comparaison V1/V2/V5 contre `HOLD_TO_SETTLEMENT`;
7. rapport par session, pas seulement par seconde;
8. validation hors Ã©chantillon et test de symÃ©trie UP/DOWN.

Le critÃ¨re principal ne sera pas le taux de rÃ©ussite instantanÃ©, mais le **gain net par position**, le drawdown, le coÃ»t d'opportunitÃ© de la sortie et la proportion de signaux rÃ©ellement remplis.

---

## Conclusion

La formule de vente fondamentale n'est pas Â« vendre quand le prix est haut Â» ni Â« vendre quand le signal change une fois Â». C'est une **rÃ¨gle de comparaison entre liquidation immÃ©diate et continuation**, renforcÃ©e par une invalidation persistante et limitÃ©e par la qualitÃ© d'exÃ©cution.

Le verdict actuel est donc : **V1 est la dÃ©finition correcte; V2 est le garde-fou directionnel le plus dÃ©fendable; V5 est la meilleure candidate de prise de bÃ©nÃ©fice; V6 et V7 encadrent l'action; aucune de ces conditions n'est encore validÃ©e comme performance attendue.** Le prochain progrÃ¨s ne consiste pas Ã  ajouter des tests, mais Ã  enregistrer de vraies sorties et Ã  rÃ©soudre dÃ©finitivement le strike, les frais et le remplissage.
