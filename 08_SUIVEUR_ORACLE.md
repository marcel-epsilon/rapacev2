# 08 — SUIVEUR D'ORACLE

**Fichiers sources regroupés :** 291SUIVEUR_D'ORACLE.md, 286_SOP_—_Suiveur_d'Oracle_Persistant.md

---

## Source : 291SUIVEUR_D'ORACLE.md

# SPEECH STRATÈGE-AUDITEUR — SESSION POLYMARKET BTC 5MIN, 7 AOÛT, 9:20–9:25AM ET

---

## 0. SYNTHÈSE EXÉCUTIVE

Stratégie retenue : **SUIVEUR D'ORACLE** — achat taker du côté désigné par l'oracle Chainlink reconstitué (Binance − basis médian), seuil de déviation 25$ soutenu 10 s.
Capital final : **5.5163$** vs 5.00$, soit **+10.33% net réel** (après latence 250 ms, slippage vérifié, frais taker crypto 0.07).
Trades : **1** — 1 gagnant, 0 perdant.
Changement Polymarket le plus impactant : la résolution des marchés crypto est passée à un **TWAP Chainlink 30 s** (5-min markets), documenté sur docs.polymarket.com/market-data/chainlink-twap (RTDS actif 4 août 2026), le sniping de dernière seconde est mort.
VERDICT DE FIABILITÉ : **[FRAGILE]**.
Étiquette : **[PATTERN CANDIDAT]** (mécanisme reproductible, validé sur 1 seule session).

---

## 1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET (consulté le 09/08/2026)

| Règle | Valeur en vigueur | Source (URL) | Date | A changé récemment ?
|-----|-----|-----|-----|-----
| P0-1 Frais taker crypto | fee = C × **0.07** × p × (1−p) ; maker **0** ; rebate maker 20% ; arrondi 5 décimales | docs.polymarket.com/trading/fees | consulté 09/08/2026 | Oui (extension à tous les marchés crypto le 06/03/2026, changelog)
| P0-1 Frais gas / résolution | Aucun frais de gas utilisateur documenté côté CLOB ; redemption non tarifée dans la doc | docs.polymarket.com/trading/fees | consulté 09/08/2026 | DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : 0$, impact borné < 0.05$
| P0-2 Types d'ordres | Limit GTC/GTD (GTD : expiration ≥ 3 min), Market, FAK, FOK, Post-Only | docs.polymarket.com/trading/place-orders | consulté 09/08/2026 | Oui (CLOB V2 le 28/04/2026 : ordre re-signé, pUSD, feeRateBps supprimé)
| P0-2 Tick size / taille min | tick 0.01 typique (0.1→0.0001 possibles) ; `min_order_size` = 5 parts (exemple doc) | docs.polymarket.com/trading/place-orders | consulté 09/08/2026 | Non
| P0-3 Résolution crypto 5-min | **TWAP Chainlink 30 s** (Data Streams, rapport signé V2), relayé par RTDS `crypto_prices_twap_thirty` | docs.polymarket.com/market-data/chainlink-twap | RTDS lancé 04/08/2026 | **OUI — settlement TWAP effectif 07/08/2026** (30 s pour 5-min, 60 s pour 15-min/4h ; source secondaire corroborée par la page officielle TWAP)
| P0-3 Délai taker | Le délai taker de 500 ms antérieur est retiré | recherche web (source secondaire) | ~07/08/2026 | DONNÉE NON VÉRIFIABLE en doc datée — hypothèse conservatrice : aucun délai favorable supposé
| P0-4 Rate limits | CLOB général 9,000 req/10s ; `POST /order` 5,000/10s burst, 120,000/10min ; `/book` 1,500/10s | docs.polymarket.com/api-reference/rate-limits | consulté 09/08/2026 | Oui (limites relevées, changelog)
| P0-4 Latence propagation carnet | Non publiée | — | — | DONNÉE NON DISPONIBLE — hypothèse conservatrice : 31 ms (latence CLOB du tableau)


**Contrainte dure P0-3** : toute stratégie dépendant du prix instantané au close est invalide ; seul le TWAP 30 s final compte.

## 2. CADRE D'ANALYSE

Je lis d'abord les 6 graphes comme faits bruts, puis je recalcule tout depuis les 4 CSV. Toute règle de stratégie devra remonter à une observation A5/B5 et respecter P0. Le P&L de référence est le net réel après latences du tableau, slippage et frais P0-1.

---

## 3. PARTIE A — LES 6 GRAPHES

### A-G1 — CARNET YES/NO COMPLET, VUE 300 s

A1. Axes : temps mm:ss.mmm (0→05:00) ; prix $ (0→1). 6 séries BBO + prob consensus. 545 événements, 34 incohérents signalés.
A2. Le marché a-t-il pricé la baisse avant la résolution ?
A3. Lecture des étiquettes d'extrema et de la trajectoire du mid consensus.
A4. Valeurs : t=00:00.199 : YES 0.54$ ; t=01:17.475 : YES 0.64$ (max session) ; t=04:12.738 : YES 0.01$ ; t=00:00.199 : NO 0.46$ ; t=04:12.738 : NO 0.99$.
A5. OBSERVATION : le consensus reste dans la bande 0.36–0.64 pendant ~150 s alors que le résultat final est DOWN ; l'effondrement YES→0.01 n'est acté qu'à 04:12.738.
A6. Spécifique session : le pic YES 0.64$ à 01:17.475 est propre à ce flux d'ordres.

### A-G2 — CARNET, ZOOMS 15 s [01:53–02:08], [00:10–00:25], [04:45–05:00]

A1. Mêmes axes, fenêtres 15 s.
A2. Quelle est la vitesse et la granularité des re-pricings ?
A3. Lecture séquentielle des étiquettes milli-secondes.
A4. Valeurs : t=01:54.401 : YES 0.38$ ; t=02:01.287 : YES 0.54$ ; t=02:04.525 : YES 0.36$ ; t=00:17.816 : YES 0.47$ ; t=00:23.195 : YES 0.55$.
A5. OBSERVATION : oscillations de ±0.18$ en < 10 s (0.38→0.54→0.36 entre 01:54 et 02:05) — le carnet sur-réagit puis revient ; des rafales de mises à jour < 50 ms d'écart existent (02:04.470 ×3).
A6. Spécifique session : l'amplitude exacte de ces allers-retours.

### A-G3 — SPOT BINANCE vs ORACLE CHAINLINK, VUE 300 s

A1. Axes : temps ; prix BTC $ (65,140→65,280). Binance 8,252 ticks ; Chainlink 287 ticks ; strike 65,221$ ; sous-graphe basis 0→50$.
A2. Lequel des deux flux mène, et de combien ?
A3. Comparaison des deux trajectoires et du sous-graphe basis.
A4. Valeurs : Binance t=00:00.216 : 65,270.00$ ; t=01:32.307 : 65,277.77$ (max) ; t=04:57.570 : 65,178.70$ ; Oracle t=00:00.000 : 65,223.40$ ; t=04:58.000 : 65,132.45$.
A5. OBSERVATION : le basis spot−oracle est en permanence positif (bande ~27–53$) ; l'oracle — la source de résolution — évolue ~45$ sous Binance et décroche sous le strike bien avant la fin.
A6. Spécifique session : la valeur absolue du basis (~45$) est propre à cette fenêtre.

### A-G4 — SPOT vs ORACLE, ZOOMS 15 s

A1. Mêmes axes, fenêtres [01:53–02:08], [00:10–00:25], [04:45–05:00].
A2. L'oracle suit-il Binance tick à tick ou par paliers ?
A3. Lecture des étiquettes des deux séries.
A4. Valeurs : Binance t=01:53.094 : 65,260.01$ ; t=02:02.464 : 65,268.91$ ; t=04:45.016 : 65,210.91$ ; Oracle t=01:54.000 : 65,214.14$ ; t=04:58.000 : 65,132.45$.
A5. OBSERVATION : l'oracle avance par pas ~1 s (paliers visibles), Binance en continu ; ts fallback 0/287 (0.0%) — horodatage payload fiable.
A6. Spécifique session : le décrochage final (65,145→65,132 sur les 5 dernières secondes) est un événement de cette fenêtre.

### A-G5 — FLUX DIRECTIONNEL YES/NO, VUE 300 s

A1. Axes : temps ; volume $ 30 s glissant (−4,000→+2,000), baissier en négatif. 1,464 trades.
A2. Le flux agrégé précède-t-il le re-pricing du carnet ?
A3. Lecture des extrema étiquetés.
A4. Valeurs : t=00:00.427 : +27.00$ ; t=03:13.752 : +992.64$ ; t=04:12.504 : −1,000.00$ ; t=04:55.366 : +14.98$ ; plancher visuel ~−4,000$ vers 04:10–04:15.
A5. OBSERVATION : le flux devient structurellement baissier après ~02:00 et culmine en valeur absolue autour de 04:12, exactement quand YES touche 0.01$ (A-G1) — le flux accompagne, il ne précède pas la résolution.
A6. Spécifique session : le trade haussier isolé de 992.64$ à 03:13.752 (contre-tendance perdant).

### A-G6 — FLUX DIRECTIONNEL, ZOOMS 15 s

A1. Mêmes axes, fenêtres 15 s.
A2. Le flux fin est-il exploitable à l'échelle de la seconde ?
A3. Lecture trade par trade.
A4. Valeurs : t=02:00.883 : −497.52$ ; t=01:54.563 : +90.54$ ; t=02:07.250 : −100.00$ ; t=00:11.652 : +100.00$ ; t=00:23.558 : +66.00$.
A5. OBSERVATION : les gros ordres taker (≥100$) sont immédiatement suivis d'un déplacement du BBO de 1–2 ticks (−497.52$ à 02:00.883 précède le repli YES 0.53→0.48 sur 02:01.890–02:02.417, A-G2).
A6. Spécifique session : la séquence exacte de ces ordres.

---

## 4. PARTIE B — LES CSV

### B-F1 — bbo.csv

B1. Colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 545 lignes ; 00:00.199→04:39.790 ; événementiel (ms).
B2. Apport : chiffre exact des spreads et des états croisés que le graphe ne fait que marquer.
B3. Calculs : spread = yes_ask − yes_bid ; croisement = yes_ask+no_ask `< 1 ou yes_bid+no_bid >` 1 ; sur toutes lignes complètes.
B4.

| Métrique | Valeur | Colonnes source | Nb lignes
|-----|-----|-----|-----|-----
| Événements BBO | 545 | toutes | 545
| Spread YES moyen | 0.0230$ | yes_bid, yes_ask | 543
| Spread YES médian | 0.0200$ | yes_bid, yes_ask | 543
| Spread YES max | 0.1000$ | yes_bid, yes_ask | 543
| Spread YES min | −0.0100$ (croisé) | yes_bid, yes_ask | 543
| États yes_ask+no_ask < 1 | 1 (t=02:29.562, somme 0.99) | yes_ask, no_ask | 545
| États yes_bid+no_bid > 1 | 1 (t=02:29.562, somme 1.01) | yes_bid, no_bid | 545
| NO ask stable 0.90$ | de 02:57.969 à 03:00.548 (2.58 s) | no_ask | 4
| B5. OBSERVATION (OBS-3) : une seule fenêtre d'arbitrage croisé, à t=02:29.562, refermée à la mise à jour suivante t=02:29.594 → durée de vie **32 ms**.
| B6. Anomalie : le PDF annonce 34 événements incohérents ; le CSV agrégé n'en montre que 2 (états transitoires intra-mise à jour probablement comptés côté PDF) — bruit, pas signal.


### B-F2 — trades.csv

B1. Colonnes t_ms, usd, direction ; 1,464 lignes ; 00:00.427→04:56.164.
B2. Apport : volumes exacts et asymétrie directionnelle.
B3. Calculs : sommes par direction ; flux net 30 s glissant ; percentiles de taille.
B4.

| Métrique | Valeur | Colonnes source | Nb lignes
|-----|-----|-----|-----|-----
| Volume total | 27,636.64$ | usd | 1,464
| Volume haussier | 9,565.86$ (584 trades) | usd, direction | 584
| Volume baissier | 18,070.78$ (880 trades) | usd, direction | 880
| Taille médiane | 4.70$ | usd | 1,464
| Taille moyenne | 18.88$ | usd | 1,464
| Plus gros trade | 1,000.00$ baissier à 04:12.504 | usd, direction | 1
| 1er flux net 30 s ≤ −500$ | t=01:48.000 (−501.78$) | usd, direction | fenêtre 30 s
| Volume ±5 s autour de 03:00.4 | 925.65$ (81 trades) | usd | 81
| B5. OBSERVATION (OBS-4) : dominance baissière 65.4% du volume ; le flux net 30 s casse −500$ dès 01:48.000, 144 s avant l'effondrement final.
| B6. Anomalies : timestamps localement non monotones (ex. lignes 5266→5182) — ordre d'arrivée réseau, bruit.


### B-F3 — spot.csv (Binance)

B1. Colonnes t_ms, price ; 8,252 lignes ; 00:00.216→04:59.548 ; rafales intra-ms.
B2. Apport : trajectoire continue haute fréquence que l'oracle n'a pas.
B3. Calculs : min/max ; basis = spot − dernier tick oracle ≤ t, sur les 8,252 ticks.
B4.

| Métrique | Valeur | Colonnes source | Nb lignes
|-----|-----|-----|-----|-----
| Ticks | 8,252 | toutes | 8,252
| Premier / dernier | 65,270.00$ / 65,178.70$ | price | 2
| Max / min session | 65,277.77$ / 65,178.70$ | price | 8,252
| Basis moyen | +44.81$ | avec B-F4 | 8,252
| Basis médian | +45.37$ | avec B-F4 | 8,252
| Basis min / max | +26.62$ / +53.11$ | avec B-F4 | 8,252
| Basis écart-type | 3.58$ | avec B-F4 | 8,252
| Occurrences basis < 0 | **0** | avec B-F4 | 8,252
| B5. OBSERVATION (OBS-1) : le basis Binance−Chainlink est strictement positif sur 8,252 points (bande 26.62–53.11$, σ=3.58$) — l'oracle de résolution se reconstruit depuis Binance à ±10$ près.
| B6. Anomalie : rafales de dizaines de ticks au même t_ms (ex. t=00:01.629) — agrégation de niveau, bruit.


### B-F4 — oracle.csv (Chainlink)

B1. Colonnes t_ms, price, ts_src ; 287 lignes ; 00:00.000→04:58.000 ; cadence ~1 s.
B2. Apport : c'est la série qui résout le marché ; cadence et fiabilité d'horodatage.
B3. Calculs : gaps entre ticks ; position vs strike 65,221$ ; moyenne des 30 dernières secondes (proxy TWAP P0-3).
B4.

| Métrique | Valeur | Colonnes source | Nb lignes
|-----|-----|-----|-----|-----
| Ticks | 287 | toutes | 287
| Cadence médiane / max | 1,000 ms / 3,000 ms | t_ms | 286
| ts fallback | 0/287 (0.0%) | ts_src | 287
| Premier / dernier | 65,223.40$ / 65,132.45$ | price | 2
| Dernier tick ≥ strike | t=02:01.000 (65,215.87$ < strike dès 01:46) | price | 287
| Oracle à 03:00.000 | 65,190.75$ (strike −30.25$) | price | 1
| Moyenne 30 dernières s (proxy TWAP) | 65,162.71$ (n=31) | price | 31
| Marge TWAP vs strike | −58.29$ → DOWN | price | 31
| B5. OBSERVATION (OBS-2) : l'oracle passe durablement sous le strike dès 01:46.000 (65,218.17$) et ne le retouche plus ; à 03:00 la marge est −30.25$ alors que le carnet price encore NO à 0.90$ (B-F1). OBSERVATION (OBS-5) : le proxy TWAP 30 s final (65,162.71$) confirme DOWN avec 58.29$ de marge — la conformité P0-3 est acquise très tôt.
| B6. Anomalie : strike affiché 65,221$ (PDF) vs premier tick CSV 65,223.40$ — écart 2.40$, probable tick antérieur à t0 absent du CSV ; hypothèse conservatrice : strike = 65,221$ (valeur PDF).


---

## 5. PARTIE C — STRATÉGIES ET SIMULATION

### C0 — CANDIDATES

Observations sources : **OBS-1** (basis strictement positif, σ=3.58$, B-F3), **OBS-2** (oracle sous strike dès 01:46, carnet en retard, B-F4/B-F1), **OBS-3** (arbitrage croisé unique, vie 32 ms, B-F1), **OBS-4** (flux net −500$ dès 01:48, B-F2), **OBS-5** (TWAP 30 s final −58.29$ vs strike, B-F4).

Latences de boucle (tableau imposé) :

- A : âge signal Binance 219 ms + envoi CLOB 31 ms = **250 ms** ; vie du signal ≥ 10,000 ms (persistance imposée) → faisable.
- B : flux CLOB 31 ms + envoi 31 ms = **62 ms** ; vie du signal ~secondes → faisable.
- C : lecture book 31 ms + envoi 31 ms = **62 ms** ; vie du signal **32 ms** (OBS-3) → **[STRUCTURELLEMENT IMPOSSIBLE]**, éliminée avant comparaison (en outre gain brut 0.01$/paire < frais ~0.026$/paire, P0-1).
- D : 62 ms ; vie du signal ~secondes → faisable.


**CRITÈRE DE SÉLECTION : P&L net réel de session, avec condition d'éligibilité préalable — le signal doit être causalement lié à la source de résolution P0-3 (oracle Chainlink) ; une candidate au signal non causal ne peut être retenue, quel que soit son P&L.**

| Stratégie | Obs. sources | Conforme P0 ? | Nb trades | P&L brut | P&L net réel | Verdict
|-----|-----|-----|-----|-----
| A. Suiveur d'oracle (déviation implicite ≥ 25$, 10 s) | OBS-1, OBS-2, OBS-5 | Oui | 1 | +0.5550$ | **+0.5163$** | **RETENUE**
| B. Momentum de flux (flux 30 s ≤ −500$) | OBS-4 | Oui | 1 | +3.5154$ | +3.3727$ | Écartée : signal auto-référent, non causal vs P0-3
| C. Arbitrage carnet croisé | OBS-3 | Oui (mécanique) | 0 | +0.01$/paire | — | [STRUCTURELLEMENT IMPOSSIBLE]
| D. Scalp de bande YES 0.44/0.53 | A-G2 (A5) | Oui | 4 | +2.30$ puis blocage | **−3.0741$** | Éliminée : P&L négatif


Décision : B affiche le meilleur P&L de session (+3.3727$) mais son signal (le flux des autres traders) n'a aucun lien causal avec l'oracle de résolution — sur une session calme il se déclenche sur du bruit à ~0.50$ avec espérance négative après frais. A est la seule candidate positive ET causalement ancrée sur P0-3. D est détruite par son propre mécanisme : 3 scalps gagnants (+0.7144, +0.6046, +0.6046$) puis position YES 10.93 parts à 0.44$ non débouclée → −4.9977$ à la résolution.

### C1 — STRATÉGIE RETENUE : SUIVEUR D'ORACLE

```plaintext
# Capital: 5.00$ | Marché: BTC 5min | Côtés: YES/NO
CONSTANTES:
  SEUIL_DEV   = 25$        # > 6σ du basis (σ=3.58$, OBS-1) [B-F3]
  PERSIST     = 10,000 ms  # >> boucle 250 ms (C0)
  CALIBRATION = 60,000 ms  # fenêtre basis médian roulant [B-F3]
  PRIX_MAX    = 0.92$      # borne d'entrée [HYPOTHÈSE n°1: au-delà, gain < frais+risque]

BOUCLE (à chaque tick Binance, âge 219 ms):
  basis_med = médiane(spot - oracle) sur 60 s glissants          [OBS-1]
  implied   = spot(t) - basis_med                                 [OBS-1]
  D         = implied - strike
  SI t >= CALIBRATION ET |D| >= SEUIL_DEV soutenu PERSIST:        [OBS-2]
     côté = NO si D < 0, YES si D > 0
     SI ask(côté) <= PRIX_MAX ET aucune position:
        ACHETER taker, taille C: C*p + C*0.07*p*(1-p) <= cash     [P0-1]
  TENIR jusqu'à résolution TWAP 30 s                              [OBS-5, P0-3]
  # Pas de sortie anticipée: le TWAP verrouille l'issue quand |D| >> 25$ [FRAGILE - 1 SOURCE]
```

### C-SIM — SIMULATION EN CONDITIONS RÉELLES

**ÉTAT DU PORTEFEUILLE** : cash initial 5.0000$, aucune position, aucun ordre.

**RACE CONDITIONS (règles déterministes)** :
(a) RÈGLE : signal pendant ordre en cours → ignoré (1 position max).
(b) RÈGLE : vente sur position non confirmée → interdite ; ici aucune vente (tenue à résolution).
(c) RÈGLE : deux signaux simultanés → priorité au côté de |D| max ; l'autre est jeté.
(d) RÈGLE : fill partiel (FAK) → conservé, reliquat annulé, pas de re-chasse au-delà de PRIX_MAX.

**JOURNAL DE TRADES** (r1 : +250 ms appliqué ; r2 : âge Binance 219 ms inclus ; r3 : profondeur non publiée dans bbo.csv = DONNÉE NON DISPONIBLE → contrôle par volume : 5$ ≤ 0.54% des 925.65$ traités à ±5 s (B-F2), et le niveau 0.90$ est resté affiché 2.58 s (B-F1) → exécution au BBO retenue ; r4 : frais P0-1 ; r5 : respecté) :

| # | Ts signal | Ts exécution réelle | Prix entrée | Quantité | Ts sortie | Prix sortie | Slippage | Frais | P&L net | Cash après
|-----|-----|-----|-----|-----
| 1 | 03:00.188 | 03:00.438 | NO 0.90$ | 5.51 parts (4.9590$) | résolution 05:00 | 1.00$ | 0.00$ | 0.0347$ | +0.5163$ | 0.0063$ → 5.5163$


Contexte d'entrée : spot Binance 65,239.53$, basis médian 44.95$, implied 65,194.58$ = strike −26.42$ ; BBO inchangé entre 02:57.969 et 03:00.548 → prix à t+250 ms identique (0.90$).

**RÉSULTATS** :

- 1 gagnant (+0.5163$), 0 perdant ; frais totaux 0.0347$ ; slippage 0.00$.
- BÉNÉFICE NET : **+0.5163$** ; capital final **5.5163$** (+10.33%).
- Pire perte latente : NO bid 0.88$ à 03:00.565 → valorisation 4.8551$, drawdown max **−2.90%**.
- P&L THÉORIQUE (sans r1–r4) : 5.55 parts à 0.90$, +0.5550$ → coût du réalisme = **0.0387$** (frais 0.0347$ + arrondi de taille 0.0040$ ; latence 0$ car BBO stable).
- Contre-scénario UP : 0 déclenchement haussier erroné sur les 8,252 ticks (B-F3).


---

## 6. PARTIE D — VERDICT DE FIABILITÉ

| Facteur | Hypothèse de la stratégie | Réalité (source) | Impact P&L | Verdict
|-----|-----|-----|-----|-----
| D1 Latence de boucle | 250 ms n'altère pas le prix | BBO 0.90$ stable 02:57.969→03:00.548, soit 2,579 ms (B-F1) | 0.00$ | [SANS IMPACT]
| D2 Fraîcheur signal | 219 ms d'âge négligeable | persistance imposée 10,000 ms = 45× l'âge (C1) | 0.00$ | [SANS IMPACT]
| D3 Slippage/profondeur | fill 5$ au BBO | profondeur non publiée ; ordre = 0.54% du volume ±5 s (B-F2) ; borne +1 tick → P&L +0.4601$ | −0.0562$ max | [DÉGRADE]
| D4 Frais complets | 0.07 taker (P0-1) | fee 0.0347$ intégrée ; gas 0$ (hypothèse conservatrice P0-1) | −0.0347$ | [DÉGRADE]
| D5 Niveau disparu/fill partiel | improbable | niveau tenu 2,579 ms vs boucle 250 ms ; règle (d) borne au pire à P&L 0 | 0 à −0.5163$ d'opportunité | [SANS IMPACT] (prouvé par la tenue du niveau)
| D6 Défaillances techniques | ordre non confirmé = trade raté | pas de RPC dans la boucle (CLOB 31 ms seul) ; échec → P&L 0, jamais négatif | borné à 0$ | [SANS IMPACT]
| D7 Passage à l'échelle | valable pour ~5$ | le taker de 497.52$ à 02:00.883 précède un repli de 2 ticks en 300 ms (B-F2/B-F1) → au-delà de ~500$/ordre le prix fuit | s'éteint vers ~500$ | [DÉGRADE]
| D8 Dépendance session | session calme → pas de trade | seuil = déviation ≥ 25$ ≈ 7σ (σ=3.58$, B-F3) ; jamais atteint côté UP ici ; fréquence inter-sessions inconnue | P&L 0$ sur session calme | [DÉGRADE] (validation mono-session)
| D9 Conformité TWAP (P0-3) | l'entrée à −120 s survit au TWAP 30 s | proxy TWAP final 65,162.71$ = strike −58.29$ (B-F4) ; la stratégie n'exploite aucun mécanisme supprimé | 0.00$ | [SANS IMPACT]


**VERDICT GLOBAL : [FRAGILE].** Zéro [DÉTRUIT], P&L net réel positif, conformité P0 totale — mais la condition [FIABLE] "D8 ≠ dépendance à un événement unique" n'est pas démontrable : la fréquence des déviations ≥ 7σ hors de cette session est DONNÉE NON DISPONIBLE, et l'asymétrie de gain (risque 4.9937$ pour gagner 0.5163$) exige un taux de réussite > 90.6% que 1 session ne peut établir. Étiquette : [PATTERN CANDIDAT].

---

## 7. AUTO-CONTRÔLE FINAL

1. Phase 0 complète → P0-1 à P0-4 sourcés ; changement TWAP identifié ; 3 items marqués NON VÉRIFIABLE avec hypothèse conservatrice.
2. Minimum de valeurs → 5+/graphe (A-G1…A-G6), 8/CSV (B-F1…B-F4) : respecté.
3. Traçabilité des règles → chaque règle C1 référence OBS-1/2/5, B-F3, P0-1/P0-3 ; 1 règle marquée [FRAGILE - 1 SOURCE].
4. Latence de boucle → 219+31 = 250 ms, reconstituée en C0 et appliquée à l'unique entrée du journal.
5. Cohérence comptable → 5.0000 − 4.9590 − 0.0347 = 0.0063 ; + 5.5100 = 5.5163$ : conforme au centime.
6. D1–D9 traités → 4 [SANS IMPACT] chacun prouvé par un chiffre, 4 [DÉGRADE], 0 [DÉTRUIT] ; verdict conforme aux règles mécaniques.
7. Unicité des chiffres → +0.5163$, 5.5163$, +10.33%, 1 trade, −2.90% identiques en ouverture, corps et clôture.


---

## 8. CLÔTURE

Vous avez sous les yeux la seule stratégie de cette session qui soit à la fois positive après réalisme et causalement ancrée sur l'oracle qui résout le marché. Elle gagne peu, ne trade que sur déviation extrême, et son verdict restera [FRAGILE] tant qu'elle n'aura pas été rejouée sur d'autres sessions. Je recommande une campagne de validation multi-sessions avant tout capital supérieur à 5.00$.

---

## Source : 286_SOP_—_Suiveur_d'Oracle_Persistant.md



SPEECH DU STRATÈGE-AUDITEUR — SESSION POLYMARKET BTC 5MIN DU 7 AOÛT, 8:55–9:00AM ET
0. SYNTHÈSE EXÉCUTIVE
Voici le verdict avant toute chose, chaque chiffre étant repris à l'identique dans le corps.

Stratégie retenue : SOP — Suiveur d'Oracle Persistant (achat NO sur persistance confirmée de l'oracle Chainlink sous le strike).
Capital final : 9,25256$ contre 5,00$ de départ, soit +85,05% net après latences, slippage et frais réels.
Nombre de trades : 1 — 1 gagnant / 0 perdant.
Changement Polymarket le plus impactant : depuis le 7 août 2026, les marchés crypto Up/Down se résolvent sur un TWAP Chainlink de 30 secondes (marchés 5 min) et non plus sur un tick unique (source : docs.polymarket.com, annonces relayées, 07/08/2026).
VERDICT DE FIABILITÉ : [FRAGILE].
Étiquette : [PATTERN CANDIDAT].
1. PHASE 0 — ÉTAT ACTUEL DE POLYMARKET
Vous avez exigé une vérification documentaire préalable ; voici l'état en vigueur, sourcé, avec le changement récent mis en évidence.

Règle	Valeur en vigueur	Source (URL)	Date	A changé récemment ?
P0-1 Frais taker crypto	fee = C × 0,07 × p × (1−p) ; maker : 0 ; rebate maker 20% ; arrondi 5 décimales, min 0,00001 USDC	docs.polymarket.com/polymarket-learn/trading/fees	consultée 10/08/2026	Non (structure stable ; sports modifiés le 10/07/2026)
P0-1 Frais dépôt/retrait/résolution	0$ côté Polymarket ; gas de rédemption pris en charge via relayer	docs.polymarket.com/polymarket-learn/trading/fees ; changelog 14/07/2026 (Neg Risk Adapter v2)	consultée 10/08/2026	Non
P0-2 Types d'ordres	GTC, GTD, FAK, FOK ; makers jamais facturés ; depuis le 24/07/2026, POST /order retourne tradeIDs (plus de transactionHashes inline)	docs.polymarket.com/changelog (entrée 17/07/2026)	17/07/2026	Oui (24/07/2026)
P0-2 Tick size marché 5 min crypto	DONNÉE NON VÉRIFIABLE dans une source datée — hypothèse conservatrice appliquée : 0,01$ (confirmée empiriquement par les 564 lignes BBO, toutes multiples de 0,01$ sauf 0,001$ observé à t=277 803 ms en zone extrême)	—	—	Décimalisation 0,0025$ limitée aux marchés World Cup (changelog 02/07/2026)
P0-2 Taille minimale d'ordre	DONNÉE NON VÉRIFIABLE — hypothèse conservatrice : ordre ≥ 1,00$ et ≥ 1 share (notre ordre de 4,59$ / 9 shares est valide sous toute hypothèse raisonnable)	—	—	—
P0-3 Résolution marchés 5 min	TWAP Chainlink 30 secondes (fenêtre de retour en arrière avant la clôture) ; 60 s pour 15 min et 4 h ; valeur signée à consommer via RTDS WebSocket ou Chainlink Data Streams — le TWAP fait foi, pas le dernier tick	docs.polymarket.com (crypto markets), annonces du 07/08/2026	07/08/2026	OUI — LE changement critique, entré en vigueur le jour même de la session
P0-4 Rate limits	CLOB POST /order : 5 000 req/10 s burst, 120 000 req/10 min sustained ; /book 1 500 req/10 s ; général CLOB 9 000 req/10 s	docs.polymarket.com/api-reference/rate-limits	consultée 10/08/2026	Oui (relevés le 01/06/2026)
P0-4 Latence de propagation carnet	DONNÉE NON VÉRIFIABLE (non publiée) — hypothèse conservatrice : mise à jour poussée par WebSocket avec la latence réseau du tableau (31 ms vers le CLOB)	—	—	Pipeline async du 24/07/2026 « réduit la latence de matching » sans chiffre publié
Ce qui a changé et ce que cela implique ici : l'ancienne résolution au tick unique rendait la dernière seconde manipulable et une stratégie « sniper du dernier tick » envisageable. Depuis le 07/08/2026, seule la moyenne des 30 dernières secondes compte pour un marché 5 min. Toute stratégie exploitant un tick isolé de fin de fenêtre est invalide par défaut ; toute stratégie fondée sur la persistance de l'oracle est mécaniquement renforcée. Contrainte dure pour la suite.

2. CADRE D'ANALYSE
J'analyse les 6 graphes puis les 4 CSV sans stratégie préconçue, en horodatage relatif à l'ouverture (mm:ss.mmm), prix en $ à 2 décimales minimum. Toute valeur provient d'une étiquette du PDF, d'un calcul CSV explicite, ou de la Phase 0 ; sinon elle est marquée DONNÉE NON DISPONIBLE. La session : « Bitcoin Up or Down — August 7, 8:55AM–9:00AM ET », résultat ▼ DOWN, strike 65 295,00$ (dernier tick oracle ≤ t0, étiquette G2).

PARTIE A — LES 6 GRAPHES
A-G1 — CARNET YES/NO COMPLET (564 ÉVÉNEMENTS BBO)
A1. IDENTITÉ : axe X temps 00:00.000→05:00.000 ; axe Y gauche prix 0,0–1,0$ ; axe droit probabilité ; 6 séries (bid/ask/mid YES et NO) + consensus (YES, 1−NO) ; 564 événements, précision ms ; 37 points « incohérents » marqués ; vue 300 s + 3 zooms 15 s.

A2. QUESTION : à quelle vitesse le carnet intègre-t-il l'information directionnelle, et laisse-t-il des fenêtres de prix erroné ?

A3. MÉTHODE : lecture des étiquettes d'extrema et des zooms, croisée avec le CSV BBO (B-F1).

A4. VALEURS EXTRAITES :

t=00:00.246 : YES 0,47$ / NO 0,53$ (ouverture quasi 50/50)
t=00:34.401 : YES 0,78$ / NO 0,23$ (pic haussier de la session)
t=00:30.302 : séquence de carnet croisé (marqueur creux, zoom 22–37 s)
t=03:30.686 : YES 0,01$ / NO 0,99$ (convergence terminale)
t=04:10.166 : YES 0,01$ / NO 0,98$ (plancher tenu)
A5. OBSERVATIONS BRUTES : (i) le carnet a pricé la HAUSSE jusqu'à 0,78$ à 00:34.401 alors que le résultat fut DOWN ; (ii) la descente YES 0,78$→0,01$ s'étale sur ~176 s (00:34.401→03:30.686) — une convergence lente, pas un choc ; (iii) les incohérences se concentrent dans les phases de re-pricing rapide (00:28–00:31).

A6. SPÉCIFIQUE À LA SESSION : le pic à 0,78$ et son timing exact sont propres à cette session ; la lenteur de convergence (ordre de la minute) est potentiellement reproductible mais non prouvable sur une seule session.

A-G2 — SPOT BINANCE VS ORACLE CHAINLINK, STRIKE 65 295,00$
A1. IDENTITÉ : axe X temps 300 s ; axe Y prix BTC 65 200–65 400$ ; 2 séries : Binance BTC/USDT (15 140 ticks), Chainlink BTC/USD (289 ticks) ; ligne de strike 65 295$ ; sous-graphe basis = spot − oracle (échelle 0–60$) ; ts fallback : 0/289 (0,0%).

A2. QUESTION : lequel des deux flux gouverne le résultat, et avec quel écart ?

A3. MÉTHODE : étiquettes d'extrema + croisement avec B-F2/B-F3.

A4. VALEURS EXTRAITES :

t=00:00.000 : oracle 65 296,64$ (1,64$ au-dessus du strike)
t=00:35.252 : spot 65 390,99$ (max spot session)
t=00:57.000 : oracle 65 291,17$ (premier tick du passage définitif sous strike)
t=03:17.000 : oracle 65 208,20$ (voisinage du minimum oracle)
t=03:42.537 : spot 65 269,99$ (minimum spot, zoom 3:41–3:56)
A5. OBSERVATIONS BRUTES : (i) le basis spot−oracle est structurellement positif, ~+40 à +70$ sur toute la session (sous-graphe borné 0–60$ avec dépassements) ; (ii) l'oracle franchit le strike à la baisse vers 00:57 et n'y revient jamais ; (iii) le spot, lui, reste au-dessus de 65 295$ jusqu'à ~01:35 (étiquette 01:35.201 → 65 295,12$).

A6. SPÉCIFIQUE À LA SESSION : l'amplitude du basis (prime USDT Binance vs agrégat Chainlink) est un régime de marché du jour ; sa positivité systématique invalide toute comparaison directe spot/strike, ce qui est un enseignement structurel.

A-G3 — FLUX DIRECTIONNEL NORMALISÉ (2 049 TRADES)
A1. IDENTITÉ : axe X 300 s ; axe Y volume $ en fenêtre glissante 30 s, flux haussier (BUY YES + SELL NO) positif, baissier négatif ; échelle −6 000 à +4 000$.

A2. QUESTION : le flux agrégé anticipe-t-il ou suit-il le prix ?

A3. MÉTHODE : étiquettes des extrema + agrégation par seaux de 30 s dans B-F4.

A4. VALEURS EXTRAITES :

t=00:01.435 : −31,80$ (premier trade baissier significatif)
t=00:31.013 : −675,26$ (gros vendeur pendant le pic YES)
t=03:26.953 : −1 000,00$ (plus gros trade baissier)
t=03:35.941 : +970,12$ (plus gros contre-flux haussier)
t=04:59.587 : +34,97$ (dernier trade étiqueté)
A5. OBSERVATIONS BRUTES : (i) le flux net 30 s est haussier en début de session (+1 772$ sur [0;30 s]) — à contresens du résultat ; (ii) il devient franchement baissier de [60 s;210 s] avec un minimum de −4 210$ sur [150 s;180 s] — c'est-à-dire APRÈS le basculement de l'oracle (00:57) ; (iii) le flux suit, il n'anticipe pas.

A6. SPÉCIFIQUE À LA SESSION : les montants exacts le sont ; le caractère retardataire du flux vs l'oracle est un candidat structurel, appuyé par une seule session.

A-G4 — IMBALANCE DIRECTIONNELLE
A1. IDENTITÉ : axe X 300 s ; axe Y imbalance = haussier/(haussier+baissier) ∈ [0;1], ligne d'équilibre 0,5.

A2. QUESTION : l'imbalance donne-t-elle un signal exploitable en avance sur le prix ?

A3. MÉTHODE : étiquettes des zooms (extrema seuls en vue 300 s).

A4. VALEURS EXTRAITES :

t=00:00.304 : 0,00 (initialisation)
t=00:02.058 : 0,29 (bref épisode baissier initial)
t=00:03.591 : 0,78 (bascule haussière immédiate)
t=00:22.132 : 0,67 (imbalance haussière pendant le rallye)
t=04:52.076 : 1,00 (saturation terminale)
A5. OBSERVATIONS BRUTES : (i) l'imbalance était haussière (0,67–0,78) précisément quand le résultat se préparait à être DOWN ; (ii) elle sature à 1,00 en fin de session quand tout est déjà joué (NO à 0,98–0,99). Signal contrariant au mieux, bruyant au pire.

A6. SPÉCIFIQUE À LA SESSION : oui — l'imbalance terminale à 1,00 reflète des achats de NO à 0,98, comportement de fin de fenêtre générique mais non testable ici au-delà de cette session.

A-G5 — SPREADS YES ET NO ABSOLUS
A1. IDENTITÉ : axe X 300 s ; axe Y spread (ask−bid) en $, échelle −0,04 à +0,10$ ; 2 séries (YES, NO) sur 564 événements.

A2. QUESTION : quand le coût de traversée du carnet explose-t-il ?

A3. MÉTHODE : étiquettes des extrema et du zoom 22–37 s, croisées avec B-F1.

A4. VALEURS EXTRAITES :

t=00:00.246 : 0,01$ (spread d'ouverture)
t=00:28.769 : 0,06$ (élargissement pendant le re-pricing)
t=00:30.302 : −0,04$ (spread négatif = carnet croisé)
t=00:30.379 : 0,11$ (spread maximal de la session, 11 ticks)
t=04:10.166 : 0,01$ (retour au serré en fin de session)
A5. OBSERVATIONS BRUTES : (i) le spread moyen YES est de 0,0184$ (calcul B-F1) — le carnet est serré ~95% du temps ; (ii) les explosions de spread (0,11$) durent moins d'une seconde et coïncident avec les phases de re-pricing violentes (00:30.379→00:30.428, retour à 0,01$ en 49 ms).

A6. SPÉCIFIQUE À LA SESSION : les timestamps des explosions le sont ; l'ordre de grandeur du spread de croisière (1–2 ticks) est cohérent avec un marché 5 min activement tenu par des market makers.

A-G6 — ÉCART INTER-CARNETS YES VS 1−NO
A1. IDENTITÉ : axe X 300 s ; axe Y écart = yes_mid − (1 − no_mid), échelle ±0,004$ ; ligne de cohérence parfaite à 0 ; 37 suspects en marqueur creux ; aucune étiquette de valeur individuelle (étiquettes absentes du PDF).

A2. QUESTION : existe-t-il un désalignement exploitable entre les deux jambes du marché ?

A3. MÉTHODE : lecture de l'échelle et des marqueurs ; croisement avec le calcul direct en B-F1.

A4. VALEURS EXTRAITES (bornes lues sur l'axe, faute d'étiquettes) :

Échelle max de l'écart affiché : +0,004$
Échelle min : −0,004$
Cohérence de référence : 0,000$
Nombre de suspects marqués : 37
ts fallback : 0/289 (0,0%)
A5. OBSERVATIONS BRUTES : l'écart inter-carnets reste confiné dans ±0,004$, soit moins d'un demi-tick — les deux jambes sont synchronisées par l'infrastructure elle-même. Les 37 « suspects » du PDF sont des micro-désalignements de mid ; mon calcul B-F1 n'identifie que 2 croisements durs de carnet (bid>ask).

A6. SPÉCIFIQUE À LA SESSION : non — cette cohérence quasi parfaite est structurelle (le CLOB apparie mécaniquement BUY YES et SELL NO) ; c'est une contrainte, pas une opportunité.

PARTIE B — LES 4 CSV
B-F1 — bbo.csv (carnet YES/NO)
B1. IDENTITÉ : colonnes t_ms, yes_bid, yes_ask, no_bid, no_ask ; 564 lignes ; période 246→277 803 ms ; fréquence événementielle (par changement de BBO).

B2. APPORT : les valeurs exactes du carnet à la milliseconde, là où G1/G5/G6 ne donnent que des extrema étiquetés. Aucune colonne de taille : la profondeur est DONNÉE NON DISPONIBLE.

B3. CALCULS : spread = yes_ask − yes_bid par ligne ; croisement dur = yes_bid > yes_ask ; arbitrage théorique = yes_ask+no_ask < 1 ou yes_bid+no_bid > 1 ; durée de vie d'un état = t du prochain événement − t de l'état.

B4. RÉSULTATS :

Métrique	Valeur	Colonnes source	Nb lignes utilisées
Événements BBO	564	toutes	564
Spread YES moyen	0,0184$	yes_ask−yes_bid	564
Spread YES max	0,11$ (t=30 379 ms)	yes_ask−yes_bid	564
Croisements durs (bid>ask)	2 (t=30 302 et 42 880 ms)	yes_bid, yes_ask	564
Somme des bids max	1,04 (t=30 302 ms)	yes_bid+no_bid	564
Somme des asks min	0,96 (t=30 302 ms)	yes_ask+no_ask	564
Durée de vie des croisements	19 ms et 69 ms	t_ms	564
BBO stable [56 976;60 010 ms]	YES 0,50/0,51 — NO 0,49/0,50	4 colonnes	6
NO ask atteint 0,85 à	t=100 631 ms	no_ask	564
NO ask atteint 0,99 à	t=189 560 ms	no_ask	564
B5. OBSERVATIONS BRUTES : (i) OBS clé — entre 56 976 et 60 010 ms le carnet cote encore NO à 0,50$ ask, fenêtre stable de 3 034 ms ; (ii) le NO ne dépasse 0,85$ qu'à 100 631 ms, soit 43,6 s après le basculement définitif de l'oracle (B-F2) ; (iii) les 2 seuls croisements durs vivent 19 et 69 ms.

B6. ANOMALIES : t=30 302 ms (yes_bid 0,67 > yes_ask 0,63, somme bids 1,04) et t=42 880 ms (0,75>0,74, somme 1,01) — bruit de séquencement des mises à jour, pas signal exploitable vu leur durée de vie ; lignes 1 et 562–564 avec champs vides (carnet unilatéral en extrême de prix), cohérentes avec le tick 0,001$ en zone >0,96.

B-F2 — oracle.csv (Chainlink BTC/USD)
B1. IDENTITÉ : colonnes t_ms, price, ts_src ; 289 lignes ; période 0→298 000 ms ; cadence moyenne 1 034,7 ms, intervalle max 3 000 ms ; ts_src = payload sur 289/289 (0 fallback).

B2. APPORT : la seule série qui compte pour la résolution (P0-3) ; G2 n'en étiquette que 9 points.

B3. CALCULS : comparaison de chaque tick au strike 65 295,00$ ; détection des croisements ; moyenne arithmétique des ticks de [270 000;300 000 ms] comme proxy du TWAP 30 s de résolution.

B4. RÉSULTATS :

Métrique	Valeur	Colonnes source	Nb lignes utilisées
Ticks oracle	289	t_ms	289
Premier tick	65 296,64$ (t=0)	price	1
Dernier tick ≥ strike	65 298,33$ (t=56 000 ms)	price	289
Premier tick du régime bas définitif	65 291,17$ (t=57 000 ms)	price	289
Ticks < strike après t=57 000 ms	229/229 (100%)	price	229
Minimum oracle	65 208,20$ (t=197 000 ms)	price	289
Proxy TWAP [270 s;300 s]	65 236,32$ (28 ticks)	price	28
Marge TWAP sous strike	−58,68$	price	28
B5. OBSERVATIONS BRUTES : (i) OBS clé — après 57 000 ms, l'oracle ne repasse jamais au-dessus du strike : 229 ticks consécutifs en dessous pendant 243 s ; (ii) dès t=59 000 ms on dispose de 3 ticks consécutifs sous strike−3$ (65 291,17 / 65 291,17 / 65 291,15, marges −3,83/−3,83/−3,85$) ; (iii) le proxy TWAP final est 58,68$ sous le strike — la résolution DOWN était insensible à toute manipulation de dernière seconde.

B6. ANOMALIES : deux trous de cadence à 3 000 ms (heartbeat/déviation Chainlink) ; un faux départ à t=1 000 ms (65 294,73$, 0,27$ sous strike) suivi d'un retour au-dessus — c'est exactement le cas que le filtre « marge + persistance » doit ignorer.

B-F3 — spot.csv (Binance BTC/USDT)
B1. IDENTITÉ : colonnes t_ms, price ; 15 140 lignes ; période 88→299 847 ms ; fréquence tick par tick avec rafales horodatées identiques (jusqu'à plusieurs centaines de ticks au même t_ms).

B2. APPORT : mesure du basis vs l'oracle, invisible à cette précision dans G2.

B3. CALCULS : basis = spot(t) − oracle(t) échantillonné à chaque tick oracle (dernier tick spot ≤ t) ; franchissements du strike par le spot.

B4. RÉSULTATS :

Métrique	Valeur	Colonnes source	Nb lignes utilisées
Ticks spot	15 140	t_ms	15 140
Max spot	65 390,99$	price	15 140
Min spot	65 261,39$	price	15 140
Basis moyen	+50,76$	price (2 fichiers)	289 points
Basis min / max	+38,12$ / +73,56$	price (2 fichiers)	289 points
Points avec basis > 40$	288/289	price (2 fichiers)	289
Premier tick spot < strike	t=95 201 ms	price	15 140
Retard spot vs oracle sur le franchissement	38,2 s (95 201 − 57 000 ms)	price (2 fichiers)	—
B5. OBSERVATIONS BRUTES : (i) le spot Binance ne franchit le strike qu'à 95 201 ms, 38,2 s après l'oracle — non parce qu'il est « en retard », mais parce qu'il cote une autre chose (USDT, agrégation différente) avec +50,76$ de prime moyenne ; (ii) le minimum spot (65 261,39$) reste 53,19$ au-dessus du minimum oracle (65 208,20$).

B6. ANOMALIES : rafales massives de ticks au même t_ms (322 ticks à t=1 910 ms) — artefact de batch du flux ; sans impact sur les conclusions car seul le dernier tick ≤ t est utilisé.

B-F4 — trades.csv (flux d'exécutions)
B1. IDENTITÉ : colonnes t_ms, usd, direction (+1 haussier, −1 baissier) ; 2 049 lignes ; période 304→~299 600 ms ; fréquence événementielle.

B2. APPORT : la profondeur RÉVÉLÉE (montants réellement exécutés), seule preuve de liquidité disponible puisque le BBO n'a pas de colonne de taille.

B3. CALCULS : volumes totaux et par direction ; flux net par seau de 30 s ; distribution des tailles.

B4. RÉSULTATS :

Métrique	Valeur	Colonnes source	Nb lignes utilisées
Trades	2 049	t_ms	2 049
Volume total	37 098,17$	usd	2 049
Volume haussier / baissier	15 397,34$ / 21 700,83$	usd, direction	2 049
Trade médian	4,00$	usd	2 049
Plus gros trade	1 000,00$ baissier (t=206 953 ms)	usd, direction	1
Flux net [0;30 s]	+1 772$	usd, direction	seau 1
Flux net [150 s;180 s]	−4 210$	usd, direction	seau 6
Trade de 761,95$ exécuté à	t=57 414 ms (baissier)	usd, direction	1
B5. OBSERVATIONS BRUTES : (i) le flux net est haussier (+1 772$) sur les 30 premières secondes puis baissier de [60 s;210 s] — confirmation chiffrée que le flux SUIT l'oracle avec retard ; (ii) preuve de profondeur : un ordre de 761,95$ s'exécute à t=57 414 ms, soit dans la fenêtre exacte où ma stratégie devra exécuter 4,59$ — 166 fois plus petit ; (iii) le marché absorbe des trades de 600–1 000$ sans assécher le carnet.

B6. ANOMALIES : trades de 0,0042 à 0,0146$ (poussière, ~15 occurrences) — probablement des fills résiduels ; quelques timestamps non strictement croissants (ex. 822 ms après 1 139 ms) — désordre d'horodatage source, sans impact sur les agrégats.

PARTIE C — STRATÉGIES ET SIMULATION
C0. CANDIDATES
Voici d'abord mes cinq observations les plus fortes, chacune sourcée.

OBS-1 : l'oracle passe définitivement sous le strike à t=57 000 ms et y reste 229/229 ticks pendant 243 s (B-F2/B5).
OBS-2 : le carnet cote encore NO à 0,50$ ask sur la fenêtre stable [56 976;60 010 ms] (3 034 ms), et n'atteint 0,85$ qu'à t=100 631 ms — 43,6 s de retard (B-F1/B5).
OBS-3 : le basis spot−oracle est de +50,76$ en moyenne (min +38,12$, max +73,56$) ; le spot franchit le strike 38,2 s après l'oracle (B-F3/B5).
OBS-4 : le flux directionnel suit le prix au lieu de l'anticiper : +1 772$ haussier sur [0;30 s], puis −4 210$ au pire sur [150 s;180 s] (B-F4/B5, A-G3/A5).
OBS-5 : les seuls croisements de carnet durs vivent 19 ms et 69 ms (B-F1/B5) ; le TWAP 30 s de résolution est 58,68$ sous le strike (B-F2/B4, P0-3).
CRITÈRE DE SÉLECTION : P&L NET RÉEL sur la session (après latence de boucle, âge du signal, slippage +1 tick, frais P0-1), à capital initial 5,00$, une stratégie [STRUCTURELLEMENT IMPOSSIBLE] étant éliminée avant toute lecture de P&L. Ce critère est posé avant la comparaison.

Les quatre candidates (familles à mécanismes réellement distincts) :

SOP — Suiveur d'Oracle Persistant. Logique : la résolution ne dépend QUE de Chainlink (P0-3) ; quand l'oracle affiche N ticks consécutifs au-delà d'une marge M sous le strike et que le carnet cote encore l'issue perdante près de 0,50$, acheter l'issue gagnante. Règles : N=3 ticks, M=3,00$ (OBS-1), NO ask ≤ 0,60$ (OBS-2). Conformité P0 : ordre FAK taker, frais 0,07 appliqués, renforcée par le TWAP (P0-3) — conforme.
ARB — Arbitrage de carnet croisé. Logique : acheter YES+NO quand yes_ask+no_ask < 1 (0,96 à t=30 302 ms), gain sans risque de 0,04$/paire. Règles : 2 ordres FOK simultanés (OBS-5). Conformité P0 : conforme aux types d'ordres — mais voir filtre structurel.
FLUX — Momentum de flux directionnel. Logique : suivre le flux net 30 s (OBS-4) ; à la clôture d'un seau, prendre la direction du flux. Règles : seuil ±1 000$ de flux net. Conformité P0 : conforme.
SPOT — Suiveur de spot Binance. Logique : utiliser Binance comme signal avancé du franchissement de strike. Règles : spot < strike → acheter NO (OBS-3 l'invalide déjà sur le fond). Conformité P0 : conforme aux règles, invalide sur la source (la résolution n'utilise pas Binance, P0-3).
FILTRE DE FAISABILITÉ STRUCTURELLE (latence totale de boucle = lecture signal + décision + envoi + confirmation, uniquement avec les latences du tableau ; décision locale bornée à 5 ms [HYPOTHÈSE n°1]) :

Candidate	Boucle	Détail	Durée de vie du signal	Verdict structurel
SOP	135 ms	Chainlink 68 + décision 5 + CLOB 31 + confirmation 31	243 000 ms (OBS-1)	VIABLE (rapport 1 800:1)
ARB	98 ms	WSS carnet 31 + décision 5 + 2 ordres parallèles 31 + confirmation 31	19 ms et 69 ms (OBS-5)	[STRUCTURELLEMENT IMPOSSIBLE] — boucle 98 ms > signal 69 ms, éliminée avant P&L
FLUX	98 ms	data-api 31 + décision 5 + CLOB 31 + confirmation 31	~30 000 ms (seau)	VIABLE
SPOT	286 ms	Binance 219 + décision 5 + CLOB 31 + confirmation 31	~38 000 ms	VIABLE mécaniquement
Tableau comparatif (simulation identique en réalisme r1–r5 pour chaque survivante ; détail de la retenue en C-SIM) :

Stratégie	Obs. sources	Conforme P0 ?	Nb trades	P&L brut	P&L net réel	Verdict
SOP	OBS-1, OBS-2, OBS-5	Oui	1	+5,00$	+4,25256$	RETENUE
ARB	OBS-5	Oui	0	+0,04$/paire théorique	—	ÉLIMINÉE (structurel)
FLUX	OBS-4	Oui	1 (BUY YES à t≈30,1 s, 7 shares @0,64$, frais 0,11290$)	−4,48$	−4,59290$	ÉLIMINÉE (P&L négatif)
SPOT	OBS-3	Oui (règles), source invalide	1 (BUY NO à t≈95,5 s, 5 shares @0,85$, frais 0,04463$)	+0,75$	+0,70537$	ÉLIMINÉE (6× inférieure à SOP, et ne « gagne » que parce que l'oracle avait tranché 38,2 s plus tôt)
Décision par les chiffres : SOP maximise le P&L net réel (+4,25256$ contre +0,70537$ pour la meilleure alternative survivante). « Ne pas trader » est battu par SOP.

C1. LA STRATÉGIE RETENUE — SOP
# SOP — Suiveur d'Oracle Persistant (marché BTC 5min, capital 5,00$)
CONSTANTES:
  STRIKE        = strike du marché (dernier tick Chainlink <= t0)   # P0-3, A-G2/A1
  MARGE         = 3,00$                                             # OBS-1 via B-F2/B4 (marges -3,83$ dès t=59s)
  N_TICKS       = 3 ticks oracle consécutifs                        # B-F2/B6 (élimine le faux départ de t=1 000 ms)
  PRIX_MAX      = 0,60$ sur l'ask de l'issue gagnante               # OBS-2 (NO ask 0,50$ disponible 3 034 ms) [FRAGILE - 1 SOURCE]
  T_MIN, T_MAX  = 30 s, 240 s                                       # évite l'ouverture bruitée et la zone TWAP finale (P0-3)
  SLIPPAGE_MAX  = +0,01$ (1 tick, P0-2) ; au-delà: annuler          # règle r3 énoncée à l'avance

BOUCLE (à chaque tick Chainlink, âge de lecture 68 ms):
  si T_MIN <= t <= T_MAX
     et les N_TICKS derniers ticks sont tous < STRIKE - MARGE       # (symétrique pour > STRIKE + MARGE -> YES)
     et ask(NO) <= PRIX_MAX                                         # lu via WSS CLOB (31 ms)
     et aucun ordre en vol et aucune position:
        C = floor( cash / (ask+0,01 + 0,07*(ask+0,01)*(1-ask-0,01)) )   # frais P0-1
        envoyer FAK BUY NO, C shares, limite ask+0,01               # boucle totale 135 ms (C0)
  TENIR jusqu'à résolution (pas de sortie anticipée: le TWAP 30 s   # P0-3
  rend la position terminale dès lors que la persistance tient)     # [HYPOTHÈSE n°2: pas de stop — assumée et discutée en D8]
Chaque règle remonte à une observation antérieure ; la règle PRIX_MAX repose sur la seule session étudiée : [FRAGILE - 1 SOURCE].

C-SIM. SIMULATION EN CONDITIONS RÉELLES
ÉTAT DU PORTEFEUILLE : cash initial 5,00000$ ; 0 position ; aucun ordre en vol.

RACE CONDITIONS (règles déterministes) :

(a) Signal d'achat pendant un ordre en cours → RÈGLE : ignoré ; un seul ordre en vol à tout instant.
(b) Vente sur position non confirmée → RÈGLE : interdite ; aucune action tant que le fill n'est pas confirmé (confirmation +31 ms).
(c) Deux signaux simultanés → RÈGLE : priorité à l'oracle (source de résolution, P0-3) ; tout signal non-oracle est jeté.
(d) Fill partiel → RÈGLE : conserver la part exécutée, annuler le reliquat (FAK le fait nativement) ; re-tenter une seule fois à ask+0,01$ max.
JOURNAL DE TRADES (r1 : entrée décalée de la boucle 135 ms ; r2 : le tick oracle t=59 000 ms est lu à 59 068 ms ; r3 : exécution à ask+0,01$ = 0,51$ faute de profondeur mesurable — la profondeur top-of-book est DONNÉE NON DISPONIBLE, bornée par le trade de 761,95$ exécuté à t=57 414 ms, 166× notre taille ; r4 : frais P0-1 ; r5 : 4,74744$ ≤ 5,00000$ de cash) :

#	Ts entrée signal	Ts exécution réelle	Prix entrée	Quantité	Ts sortie	Prix sortie	Slippage	Frais	P&L net	Cash après
1	00:59.000 (3ᵉ tick < strike−3$)	00:59.104 (signal 59 000 + lecture 68 + décision 5 + envoi 31 ms ; BBO stable jusqu'à 60 010 ms)	0,51$ (NO)	9 shares	05:00.000 (résolution DOWN)	1,00$	+0,01$/share (−0,09$)	0,15744$ (9×0,07×0,51×0,49)	+4,25256$	entrée : 0,25256$ ; après résolution : 9,25256$
Vérification comptable ligne à ligne : 5,00000 − (9×0,51 = 4,59000) − 0,15744 = 0,25256$ ; résolution : 0,25256 + 9×1,00 = 9,25256$, au centime (et au centième de centime) près.

RÉSULTATS :

Trades : 1 gagnant (+4,25256$) / 0 perdant ; frais totaux 0,15744$ ; slippage total −0,09$.
BÉNÉFICE NET : +4,25256$ ; capital final 9,25256$ vs 5,00$ soit +85,05%.
Pire perte réalisée : 0,00$ ; drawdown latent max : −0,45$ (no_bid tombe à 0,46$ sur [61 724;62 026 ms], 9×(0,51−0,46)), soit 9,00% du capital, résorbé à t=69 308 ms (no_bid 0,60$).
P&L THÉORIQUE sans r1–r4 : 10 shares à 0,50$ sans frais ni slippage → 10,00$, soit +5,00$ (+100,00%). Le coût du réalisme est de 0,74744$ (0,09$ slippage + 0,15744$ frais + 0,50$ de perte de taille par arrondi de shares).
PARTIE D — VERDICT DE FIABILITÉ
Facteur	Hypothèse de la stratégie	Réalité (source)	Impact P&L	Verdict
D1 Latence de boucle	135 ms tolérables	BBO identique sur [56 976;60 010 ms] = 3 034 ms ≫ 135 ms (B-F1/B4)	0,00$ — prix inchangé à l'exécution	[SANS IMPACT] (prouvé)
D2 Fraîcheur du signal	tick oracle âgé ≤ 68 ms + cadence 1 034,7 ms absorbés par N=3	229/229 ticks sous strike après 57 s : aucune inversion à rattraper (B-F2/B4)	0,00$	[SANS IMPACT] (prouvé)
D3 Slippage / profondeur	fill à ask+0,01$	profondeur BBO DONNÉE NON DISPONIBLE ; bornée par trade de 761,95$ à t=57 414 ms (B-F4/B4)	−0,09$ intégré ; borne si fill à 0,52$ : capital final 9,16275$ (−0,08981$)	[DÉGRADE] (borné)
D4 Frais complets	taker 0,07, gas 0$	frais P0-1 exacts : 0,15744$ ; gas de rédemption via relayer (P0-1) [HYPOTHÈSE n°3]	−0,15744$ intégré	[DÉGRADE] (chiffré)
D5 Niveau disparu / fill partiel	règle (d) : re-try unique à +1 tick	6 événements BBO seulement dans la fenêtre de 3 034 ms : carnet calme (B-F1)	borné à −0,08981$ (cas 0,52$) ; fill partiel garde l'espérance positive par share	[DÉGRADE] (borné)
D6 Défaillances techniques	1 re-émission possible	fenêtre exploitable de 243 s = ~1 800 boucles de 135 ms disponibles (B-F2/B4, C0)	0,00$ si re-émission ≤ 243 s ; prix d'entrée se dégrade de ~0,10$/40 s de retard (0,60$ à t=69 s, B-F1)	[SANS IMPACT] sur l'issue, dégradation bornée sur le prix
D7 Passage à l'échelle	taille 4,59$ invisible	volume total session 37 098,17$ ; plus gros trade unitaire 1 000,00$ (B-F4/B4)	mécanisme intact ≤ ~500$/ordre [borne : plus gros fill observé sans décalage = 1 000,00$ ; au-delà, DONNÉE NON DISPONIBLE]	[DÉGRADE] au-delà de quelques centaines de $
D8 Dépendance à la session	filtre N=3/M=3$ ne déclenche pas sur session calme	sur cette session le filtre déclenche 1 fois ; un oracle oscillant autour du strike ne produit jamais 3 ticks < strike−3$ → 0 trade, 0 perte ; MAIS le gain de +85,05% exige un carnet en retard de 43,6 s (OBS-2), constaté 1 seule fois	sur session calme : 0,00$ ; l'ampleur du gain dépend d'un unique événement de convergence lente	[DÉGRADE]
D9 Conformité au nouveau fonctionnement	la persistance sous strike gouverne la résolution	TWAP 30 s en vigueur depuis le 07/08/2026 (P0-3) ; proxy TWAP de la session : 65 236,32$, soit 58,68$ sous strike (B-F2/B4) — le TWAP renforce la thèse de persistance et tue les stratégies de dernier tick, pas celle-ci	0,00$ — la marge TWAP est 19,6× la marge d'entrée exigée (3,00$)	[SANS IMPACT] (prouvé)
VERDICT GLOBAL (application mécanique) : zéro [DÉTRUIT] ; P&L net réel positif (+4,25256$) ; conformité P0 totale ; mais D8 établit que l'ampleur du profit dépend d'un événement unique de convergence lente observé sur une seule session — la condition « D8 ≠ dépendance exclusive à un événement unique » exigée pour [FIABLE] n'est pas satisfaite. Verdict : [FRAGILE]. Le mécanisme (marché lent vs oracle de résolution) est un [PATTERN CANDIDAT] : falsifiable à faible coût puisque sur session non conforme la stratégie ne trade pas.

AUTO-CONTRÔLE FINAL
Contrôle 1 → Phase 0 : P0-1 à P0-4 sourcés (fees, changelog, rate-limits : docs.polymarket.com, consultés le 10/08/2026 ; TWAP : 07/08/2026) ; tick size et taille minimale marqués DONNÉE NON VÉRIFIABLE avec hypothèses conservatrices énoncées. OK
Contrôle 2 → 5 valeurs minimum par graphe (A-G1 à A-G6 : 5, 5, 5, 5, 5, 5) ; 8 minimum par CSV (B-F1 : 10, B-F2 : 8, B-F3 : 8, B-F4 : 8). OK
Contrôle 3 → traçabilité des règles : MARGE→OBS-1/B-F2, N_TICKS→B-F2/B6, PRIX_MAX→OBS-2/B-F1 [FRAGILE - 1 SOURCE], SLIPPAGE→P0-2, T_MAX→P0-3. OK
Contrôle 4 → boucle reconstituée chemin par chemin : Chainlink 68 + décision 5 [HYPOTHÈSE n°1] + CLOB 31 + confirmation 31 = 135 ms, appliquée à l'unique entrée du journal (59 000 → 59 104 ms pour l'envoi, confirmation à 59 135 ms). OK
Contrôle 5 → comptabilité : 5,00000 − 4,59000 − 0,15744 = 0,25256 ; 0,25256 + 9,00000 = 9,25256$ = capital final annoncé. OK
Contrôle 6 → D1 à D9 traités ; chaque [SANS IMPACT] porte sa preuve chiffrée (D1 : 3 034 ms vs 135 ms ; D2 : 229/229 ; D9 : 58,68$ vs 3,00$) ; verdict global conforme aux règles mécaniques. OK
Contrôle 7 → cohérence des chiffres entre ouverture, corps et clôture : 9,25256$, +85,05%, 1 trade, 0,15744$, 07/08/2026, [FRAGILE], [PATTERN CANDIDAT] — identiques partout. OK
CLÔTURE
Vous avez sous les yeux une stratégie qui n'invente rien : elle lit la source de résolution avant que le carnet ne la lise. Elle a transformé 5,00$ en 9,25256$ sur cette session, et elle sait surtout ne pas trader quand les conditions ne sont pas réunies. Je la qualifie de [FRAGILE] tant qu'une seconde session n'a pas confirmé le retard de convergence du carnet.





