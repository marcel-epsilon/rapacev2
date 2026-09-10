A. Résumé de la stratégie finale
Le principe, en langage courant

Le marché sur lequel opère le bot est un pari binaire : « le prix de BTC sera-t-il au-dessus ou en dessous d'un niveau K (le "strike", le niveau de référence fixé au départ) dans 5 minutes ? ». Chaque session dure exactement 300 secondes. À la fin, le contrat vaut 1 ou 0.


Le bot ne prédit rien. Il exploite une seule grandeur connue d'avance : la quantité de mouvement que le prix peut encore faire avant la fin. À 60 secondes du départ, il reste beaucoup de chemin possible. À 250 secondes, presque plus. Cette réduction est mécanique, arithmétique, et ne dépend d'aucune opinion sur la direction.


Le bot propose donc d'acheter le côté qui est actuellement gagnant, à un prix légèrement inférieur à sa valeur réelle. La différence entre les deux (la décote, notée δ) est sa rémunération. Il la calibre exactement sur le risque restant : plus le temps restant est court, plus la décote peut être fine.


Pourquoi personne d'en face ne corrige ce prix ? Parce que le temps qui passe est un événement silencieux. Aucune ligne du carnet ne bouge quand une seconde s'écoule. C'est précisément ce silence qui est l'information.
Les trois principes structurels

    Non-directionnalité par miroir. Le bot ne parie jamais sur une direction. Il regarde de quel côté du strike se trouve l'oracle, et achète ce côté. Si l'oracle était de l'autre côté, il ferait exactement l'inverse avec les mêmes chiffres. Le signe sert à choisir le côté, jamais à décider s'il faut agir.
    Refus par défaut (fail-closed). Toute donnée manquante, périmée ou incohérente produit un refus écrit dans le journal, jamais une valeur de remplacement. Le bot est inactif la plupart du temps, et c'est un résultat, pas un échec.
    Aucune constante gelée. Chaque seuil qui peut être mesuré en direct est mesuré en direct. C'est l'erreur unique qui a stérilisé toutes les versions précédentes.


B. Formule d'achat finale
Les grandeurs, avec leur définition unique

Chaque grandeur a une seule définition dans tout le code. Le mot « unique » est littéral : une seule fonction, aucune variante.


τ        = horloge_marché() − (endDate − 300)
           # temps écoulé dans la session, en secondes.
           # endDate = heure officielle de fin. Jamais time(), jamais un temps relatif de fichier.

σ₃₀      = écart_type(incréments_1s(oracle, sur les 30 dernières secondes)) × √30
           # "combien l'oracle bouge en 30 secondes", en dollars. Mesuré, jamais posé.

σ_T(τ)   = σ₃₀ · √((300 − τ) / 30)
           # combien l'oracle peut ENCORE bouger d'ici la fin. C'est le coeur du système.

λ(τ)     = 0,5513 · σ_T(τ)
           # facteur de conversion dollars → probabilité. 0,5513 = √3/π, identité mathématique,
           # non réglable, non optimisable.

m*(τ)    = oracle_dernier(τ) − K
           # de combien de dollars l'oracle est au-dessus (+) ou en dessous (−) du strike.

z(τ)     = |m*(τ)| / σ_T(τ)
           # "combien d'écarts-types d'avance". z = 1 signifie : il faudrait tout le mouvement
           # restant, dans le mauvais sens, pour renverser l'issue.

côté     = OUI si m*(τ) ≥ 0, sinon NON        # transformation miroir, PAS un pari

p_ancré  = Φ(z(τ))          # score borné entre 0 et 1. PAS une probabilité calibrée.

e(τ)     = |p_ancré − ask_côté(τ)|            # écart entre l'ancrage et le carnet. TOUJOURS absolu.

X₆₀(τ)   = nombre de changements de signe de m* sur les 60 dernières secondes
P(τ)     = durée cumulée de même signe avec marge
v(τ)     = |m*(τ) − m*(τ−15)| / 15            # vitesse, en $/s, en valeur absolue
N        = plancher(5,00 / (ask + fee_sizing(ask) + 0,01))    # nombre de parts

La porte d'entrée, dans l'ordre exact

L'évaluation s'arrête au premier faux, et ce premier faux est écrit dans le journal avec son code. C'est la donnée qui permettra de réviser les seuils.


#
	

Condition
	

Code de refus
	

Sens en clair

1
	

Tous les blocs de validité (voir plus bas)
	

code du bloc
	

données propres

2
	

180 ≤ τ ≤ 270
	

HORS_FENETRE
	

seule fenêtre où l'information existe

3
	

z(τ) ≥ 1,0
	

MARGE_INSUFFISANTE
	

il faut un écart-type complet d'avance

4
	

X₆₀(τ) = 0
	

CYCLICITE
	

l'oracle ne doit pas osciller autour du strike

5
	

P(τ) ≥ 5 s
	

PERSISTANCE
	

même côté depuis 5 secondes au moins

6
	

v(τ) ≤ 0,45 $/s
	

RETOURNEMENT
	

garde symétrique, les deux sens refusés

7
	

e(τ) > δ_min de façon continue depuis ≥ 2 s
	

PAS_D_EDGE / ECART_NON_CONFIRME
	

l'écart doit survivre à sa naissance

8
	

(1 − ask) > fee_settlement(ask) + 0,02
	

PRIX_MAX
	

test d'espérance nette, symétrique

9
	

e(τ) ≤ D_max hors conjonction, en alerte seule
	

DECOTE_ABERRANTE
	

voir point de décision D3
Les blocs de validité (préalable absolu)

Portée session (refus de toute la session) : endDate présent et officiel · K présent avec k_source == "officiel", aucune tolérance, aucun proxy · ratio de régime σ₃₀_session / médiane_20_sessions ∈ [0,4 ; 2,5].


Portée instant (refus de l'instant) : âge oracle ≤ 2,5 s · trou oracle ≤ 8 s · trou carnet ≤ 3 s · 4 champs de carnet présents (booléen, jamais une valeur sentinelle) · HALT présent et faux · |bid_OUI + ask_NON − 1| ≤ 1 tick · basis dans la bande à 3σ · σ₃₀ ∈ [0,5 ; 40] $ et fenêtre de 30 s pleine · spread ∈ [0 ; 0,04] bornes strictes des deux côtés · ask ≥ méd₃ₛ(ask) − q95(chute_3s) · max(|Δask| sur 3 s) ≤ q95(|Δask|_3s) · profondeur cumulée ≥ plancher(N) · position vide, zéro ordre en vol, réconciliation OK, plancher(N) ≥ 5.
Le prix et la taille

δ_min(τ)   = fee_settlement(ask) + 0,01          ≈ 0,0132     (voir décision D4)
δ_guard(τ) = +∞      si τ < 180
             0,0157  si 180 ≤ τ < 240
             0,0107  si 240 ≤ τ < 270
             +∞      si τ ≥ 270
δ(τ)       = max(δ_min(τ), δ_guard(τ))

prix_limite = min( arrondi_tick_bas(p_ancré − δ), bid_côté )
              ordre POST-ONLY obligatoire. Un rejet pour croisement est un SUCCÈS attendu.

taille      = min(plancher(N), profondeur_disponible), plancher 5 parts
validité    = 2 secondes


La décote est une enveloppe par paliers, jamais une fonction continue. Raison mesurée : la décote de milieu de session varie d'un facteur 37 entre sessions. Un ajustement continu fabriquerait une fausse précision.


Vérification que ça se tient : σ_T(180)/σ_T(255) = 1,67 et δ_guard(180)/δ_guard(255) = 1,47. Les deux ratios sont du même ordre : c'est bien le risque restant qui pilote la décote. La dégressivité est une conséquence arithmétique, pas un réglage.
Ce que la porte ne lit JAMAIS

Le score de Platt p, la décote signée D, le décalage lag_signé, la pente signée de m*, le momentum du spot, le marqueur de flux, le signe du basis. Ces grandeurs sont journalisées et jamais lues par une condition, vérifié par un test qui inspecte le code lui-même.


C. Formule de vente finale
Politique par défaut : sortie unique au règlement

Pas de position                                    → NO_POSITION
Données périmées ou strike non résolu               → SUSPEND_DATA
bid ≤ 0, spread hors [0 ; 0,02], fill non confirmable → SUSPEND_LIQUIDITY
τ ≥ 300                                            → SETTLE
τ ≥ 262 (τ_lock)                                   → HOLD_TO_SETTLEMENT
côté ancré inversé pendant h secondes consécutives → SELL_INVALIDATION
sinon                                              → HOLD


Justification de l'absence de sortie anticipée : la thèse est que le temps travaille pour la position. Sortir tôt revient à racheter l'incertitude qu'on vient de vendre. La friction aller-retour mesurée est de 0,044 à 0,047 par unité autour de ask = 0,50 à 0,71, soit environ 0,5 à 0,6 $ de mouvement d'oracle. Toute sortie plus fine que cette bande morte détruit de la valeur mécaniquement.
Les états de sortie, non réductibles

SELL_FILLED · SELL_PARTIAL · HOLD · HOLD_TO_SETTLEMENT · SETTLE_WIN · SETTLE_LOSS · SUSPEND_DATA · SUSPEND_LIQUIDITY · NO_POSITION · UNKNOWN


Réduire ça à SELL / HOLD est une faute : un HOLD silencieux alors que la sortie était impossible masque exactement l'incident qu'il faut mesurer.
Les sept contradictions interdites, codées en assertions

Le code doit lever une erreur, pas journaliser un avertissement, si : SELL_FILLED et SUSPEND_LIQUIDITY au même instant · SELL_FILLED après τ_lock sans sortie de sécurité journalisée · vendre une position qui n'existe plus · déclarer une vente au bid en utilisant l'ask · déclarer un gain sans confirmation de remplissage · utiliser p pour une position NON sans convertir en 1−p · mélanger un strike inféré et un strike officiel dans le même rejeu.
Les conditions désactivées, et pourquoi

V1 (vente à la valeur) et V5 (prise de bénéfice) exigent une probabilité calibrée de gagner. Cette grandeur n'existe pas : le score échoue au test de calibration par sous-groupe. Elles sont codées, derrière un drapeau, inertes. Activation conditionnée à un score de Brier meilleur que celui du carnet dans les 4 sous-groupes sur au moins 50 sessions étiquetées. Le seuil « vendre à 0,92 » est interdit en dur : le corpus mesure 68 instants où l'ask dépassait 0,92, pas un seul bid rempli.


D. Stratégie d'ancrage finale
Ce qui constitue l'ancrage

L'ancrage n'est pas le carnet. C'est un prix théorique construit à partir de trois choses, et de trois choses seulement :


    K, le strike officiel, lu une seule fois à l'ouverture.
    L'oracle, la source unique qui décide de l'issue.
    Le temps restant, converti en risque par la loi en racine carrée.

ancrage = Φ( |oracle − K| / (σ₃₀ · √((300 − τ)/30)) )


En clair : « à quelle distance du strike suis-je, mesurée en unités du mouvement qui reste possible ». Une distance de 20 $ ne veut rien dire dans l'absolu. Elle vaut beaucoup à 250 secondes et rien à 40 secondes.
Quand il est créé, comment il évolue

    À l'ouverture (τ = 0) : K est lu, certifié, gelé pour toute la session. Si k_source ≠ "officiel", la session entière est refusée. Aucun proxy : un écart de 12,25 $ sur le strike inverse le règlement d'une session du corpus.
    De τ = 0 à τ = 30 : rien n'est calculable. La fenêtre de 30 secondes n'est pas pleine, donc σ₃₀ n'existe pas, donc l'ancrage n'existe pas. Le bot refuse, il n'attend pas. La distinction compte : un refus s'écrit, une attente ne s'écrit pas.
    De τ = 30 à τ = 180 : il mesure et il se tait. Il calcule tout, il journalise tout, il ne cote pas. Raison arithmétique, pas prudentielle : le mouvement résiduel médian de l'oracle y vaut 11,62 $ = 1,6 σ₃₀, et la décote qui protégerait contre ce mouvement vaudrait 0,677 point, six fois le plafond de sécurité.
    De τ = 180 à τ = 270 : fenêtre d'action. Trois choses changent simultanément à 180 secondes, mesurées par trois familles indépendantes : le mouvement résiduel tombe à 1,50 $ (0,21 σ₃₀), la décote nécessaire tombe de 0,677 à 0,0157 point (facteur 43), et l'écart entre le carnet et l'ancrage, lui, ne tombe pas.
    Après 270 : le carnet cesse de publier vers 273 secondes. Toute logique au-delà travaille sur des données mortes. Annulation générale, aucune ouverture.

Ce qui est comparé à l'ancrage

Une seule chose : l'ask du côté ancré. L'écart est e(τ) = |p_ancré − ask_côté|, toujours en valeur absolue.


Pourquoi absolu et pas signé ? Parce que mesuré signé, les trois plus grosses sous-évaluations détectées dans le corpus étaient toutes du mauvais côté. L'écart signé est anti-corrélé au résultat. Le signe change de camp avec l'issue.


Le prix du côté NON n'est jamais lu séparément : il est dérivé du côté OUI par l'invariant de complémentarité (vérifié exact sur 1 025 lignes sur 1 025). Ça divise par deux la surface d'erreur.
Frontière observé / déduit / confirmé / hypothèse

Statut
	

Contenu

Directement observé, comptage explicite
	

Absence de décalage oracle→carnet (0/27 dans la bande postulée) · σ₃₀ mesuré à 7,20 $ médian contre 26,50 $ posé (29/29) · formule à 22 tests stérile (0 entrée, 19/19 agents) · invariant de complémentarité exact (1 025/1 025) · basis structurel +42,1 $ positif sur 100 % des lignes · verrouillage de l'issue à 262 s (8/8) · aucun prédicteur, R² max 0,207 · 11 champs sur 24 absents de 41 sessions sur 41

Déduit, vérifiable arithmétiquement
	

La porte telle qu'écrite produit un ordre agressif, pas passif · la décote retenue viole son propre plancher · le coupe-circuit exclut la phase que le mécanisme cible · deux formules de taille coexistent · trois modèles de frais sans règle d'affectation

Confirmé par plusieurs sessions
	

Le déplacement de fenêtre vers [180;270], réclamé par trois familles de mesures indépendantes · le basis est un décalage de source, jamais une inefficience (12 sessions)

Hypothèse, à ne jamais traiter comme un fait
	

Le tick vaut 0,01 (inféré du carnet, pas un champ de schéma) · Φ est la loi normale standard · la plateforme accepte les ordres post-only · τ_lock = 262 s (mesuré sur une seule session) · z_min = 1,0 (explicitement PROVISOIRE)


E. Mécanisme de l'oracle
Ce que l'oracle fournit, et quand

Une valeur de prix, environ une fois par seconde, avec des trous pouvant atteindre 7 à 8 secondes. C'est la source unique de résolution : c'est lui, et lui seul, qui décide de l'issue à la fin, par une moyenne sur les 30 dernières secondes.


Champ inutilisable à signaler : ts_src contient littéralement la chaîne payload sur 269 lignes sur 269. Il n'existe donc aucune heure de publication source. Toute latence causale sous la seconde est non mesurable avec ces données.
Le décalage que tu décris : ce qu'il est devenu

Je dois être précis ici, parce que c'est le coeur de ta question et que la réponse n'est pas celle du postulat de départ.


Le postulat initial était : l'oracle publie une information, le carnet met 5 à 60 secondes à l'intégrer, on achète dans cet intervalle. C'était la justification écrite de toute la stratégie.


La mesure : décalage médian −0,75 seconde. P10 à −28 s, P90 à 0 s. 0 mesure sur 27 dans la bande [+5 ; +60]. 26 sur 27 avec un décalage négatif ou nul, c'est-à-dire : c'est le carnet qui mène l'oracle, pas l'inverse. La corrélation la plus forte de tout l'échantillon (r = +0,424) est à −2 secondes. Dans la bande exigée, la corrélation moyenne est négative sur les trois sessions instrumentées.


Le test qui vérifiait cette condition a refusé 543 instants sur 543. C'est lui, à lui seul, qui explique le PnL nul. Ce n'est pas un seuil à élargir : élargir une bande jusqu'à ce qu'elle contienne la mesure est une tautologie, pas une correction.
Le décalage qui existe vraiment, et qui est exploité

Ton intuition d'un décalage entre deux instants de l'oracle est correcte. Simplement, le décalage utile n'est pas entre l'oracle et le carnet. Il est entre l'oracle et lui-même, dans le temps. Quatre comparaisons à deux instants survivent dans la version finale, et ce sont elles qui portent tout le mécanisme :


Comparaison
	

Deux instants comparés
	

Ce que ça mesure
	

Rôle

σ₃₀
	

chaque incrément sur 30 s
	

l'agitation récente
	

fabrique l'échelle de tout le système

X₆₀
	

signe de m* sur 60 s
	

l'oracle a-t-il traversé le strike
	

condition d'entrée, doit valoir 0

P
	

durée de même signe
	

depuis combien de temps il tient
	

condition d'entrée, ≥ 5 s

v
	

m*(τ) contre m*(τ−15)
	

à quelle vitesse il bouge
	

garde symétrique, ≤ 0,45 $/s

TRAVERSEE_SEUIL
	

signe(m* maintenant) contre signe(m* à l'envoi de l'ordre)
	

le côté s'est-il inversé
	

annulation immédiate, priorité 1


Et surtout : σ_T(τ) est une comparaison entre maintenant et la fin. C'est un décalage temporel pur. C'est la seule grandeur du système connue à l'avance, et c'est pour ça qu'elle est exploitable : personne ne la re-cote, parce que son écoulement ne produit aucun événement dans le carnet.
Comment le décalage devient un signal, puis une décision

ÉVÉNEMENT           : un tick d'oracle arrive
INFORMATION REÇUE   : un prix, à un instant de capture
INFORMATION GARDÉE  : la fenêtre glissante de 30 s (pour σ₃₀), celle de 60 s (pour X₆₀),
                      la valeur d'il y a 15 s (pour v), le compteur de persistance,
                      le signe au moment de l'envoi de l'ordre
COMPARAISON         : σ₃₀ contre son historique 20 sessions (régime)
                      |m*| contre σ_T (c'est z)
                      p_ancré contre ask (c'est e)
                      signe maintenant contre signe à l'envoi (c'est la traversée)
DÉCISION            : une seule action par événement, par priorité stricte
ACTION              : annuler tout, suspendre, sortir, entrer, ou tenir


F. Chronologie d'exécution
Le déclencheur

L'union des événements oracle et carnet. Jamais une horloge fixe. Cadence plancher : 4 Hz sur le carnet, obligatoire.


Pourquoi ce n'est pas négociable : la cadence médiane réelle du carnet est de 0,013 seconde, celle de l'oracle de 1 seconde. Rapport 77 pour 1. À 1 Hz, 98 % des mises à jour du carnet sont invisibles. Le krach d'ask de 0,279 s et la traversée de seuil de 0,300 s sont manqués 70 à 72 % du temps. Les gardes censées détecter ces événements étaient aveugles à l'événement même qu'elles devaient détecter.
Les 17 étapes, à chaque tick

#
	

Étape
	

Module
	

Ce qui se passe

1
	

Réception
	

Intake
	

événement typé, horodaté sur son horloge de capture propre. t_decision pas encore posé

2
	

Fraîcheur
	

Intake, Warden
	

âge ≤ 2,5 s · trou oracle ≤ 8 s · trou carnet ≤ 3 s · 4 champs BBO · HALT faux

3
	

Nettoyage
	

Intake
	

déduplication, détection d'ordre inversé, séparation stricte des deux horloges

4
	

Mesures brutes
	

Oracle, Garnet
	

sur oracle : σ₃₀, σ_T, λ, m*, TWAP30. Sur carnet : spread, mid, médiane 3 s, quantiles, invariants, basis

5
	

Termes observés
	

Oracle, Garnet
	

z, côté, p_ancré, e, X₆₀, P, v, δ_min, δ_guard, δ, N

6
	

Microstructure
	

Prism
	

spread borné 2 côtés, anti-couteau, stabilité d'ask, vitesse symétrique, profondeur

7
	

Validité
	

Warden
	

ordre déterministe, arrêt au premier faux, un code atomique

8
	

Signal
	

Signal
	

seulement si 7 est verte. Produit une intention, pas un ordre

9
	

Positions
	

Positions, Exit
	

réconciliation avec l'API avant toute décision. Position existante → entrée interdite

10
	

Risque
	

Risk, Fees
	

N, plancher 5 parts, garde de frais, budget de mesure, kill-switch

11
	

Décision
	

Arbiter
	

une seule action, priorité absolue (voir ci-dessous)

12
	

Validation finale
	

Arbiter
	

7 assertions, prix dans [0;1] et multiple du tick, vérification que le prix limite ne dépasse pas le bid. t_decision posé ici

13
	

Envoi
	

Executor
	

post-only, validité 2 s. Jamais de modification en place : annuler puis recoter. t_envoi posé

14
	

Suivi
	

Executor
	

t_ack, t_fill, prix et quantité, maker/taker, codes de rejet

15
	

État
	

Positions
	

agrégation des fills, réconciliation, affectation du sous-groupe, PnL contrefactuel

16
	

Journal
	

Chronicle
	

24 champs + 8 horodatages + sous-groupe + code + valeur mesurée et seuil pour chaque test. Un échec d'écriture arrête le bot

17
	

Dégradations
	

tous
	

tout est fail-closed, journalisé, et annule les ordres en vol
L'ordre de priorité, absolu

1. TRAVERSEE_SEUIL          → ANNULER TOUT, avant tout autre traitement (budget < 300 ms)
2. Rupture d'intégrité      → ANNULER TOUT + SUSPENDRE
3. Sortie de sécurité       → SUSPEND_DATA / SUSPEND_LIQUIDITY
4. Sortie discrétionnaire   → seulement si le mode est activé
5. Entrée
6. HOLD

Ce que la séquence garantit

Aucune décision sans un τ exact (étape 2 avant 4). Aucune mesure sur des données sales (3 avant 4). Aucune économie évaluée avant l'intégrité (7 avant 8). Aucun ordre sans état réconcilié (9 avant 13). Aucun double ordre (verrou en 9, action unique en 11, assertion en 12). Aucune décision non traçable (16 obligatoire).


La contrainte la plus dure et la moins connue : le budget de latence. La fenêtre utile d'une cotation est de 2 secondes, l'annulation cible ≤ 500 ms, la détection de traversée < 300 ms. Le budget réel est aujourd'hui non déterminable, et c'est exactement ce que les 8 horodatages servent à mesurer. Tant qu'ils ne sont pas mesurés, aucun mécanisme ne peut être déclaré atteignable.


G. Variables : quand lues, écrites, utilisées

Variable
	

Apparaît
	

Lue
	

Mémorisée
	

Comparée à
	

Déclencheur
	

Décision influencée

endDate
	

ouverture
	

1 fois
	

pour la session
	

rien
	

ouverture de session
	

absente → session refusée

K, k_source
	

ouverture
	

1 fois
	

pour la session
	

oracle, à chaque tick
	

ouverture
	

non officiel → session refusée

oracle.price
	

~1 Hz
	

chaque tick
	

fenêtres 30 s et 60 s
	

K (donne m*)
	

tick oracle
	

tout

σ₃₀
	

τ ≥ 30 s
	

chaque tick oracle
	

valeur courante + historique 20 sessions
	

bornes [0,5 ; 40], ratio de régime
	

tick oracle
	

échelle de tout le système

σ_T
	

τ ≥ 30 s
	

chaque tick
	

non
	

rien
	

recalculé
	

z, λ, δ_guard

m*
	

τ ≥ 0
	

chaque tick
	

signe + valeur d'il y a 15 s
	

son propre signe passé
	

tick oracle
	

côté, z, X₆₀, P, v

z
	

τ ≥ 30 s
	

chaque tick
	

valeur à l'envoi de l'ordre
	

z_min = 1,0 ; z à l'envoi
	

tick oracle
	

entrée, recotation

X₆₀
	

τ ≥ 60 s
	

chaque tick
	

compteur glissant
	

0
	

tick oracle, traversée
	

entrée. Forcé à 1 pendant 60 s après traversée

P
	

continu
	

chaque tick
	

compteur cumulé
	

5 s
	

tick oracle
	

entrée. Gelé sur trou, remis à zéro sur changement de signe

ask, bid, spread
	

≥ 4 Hz
	

chaque snapshot
	

médiane 3 s, quantiles glissants
	

bornes, quantiles mesurés
	

tick carnet
	

e, N, prix, microstructure

e
	

τ ≥ 30 s
	

chaque snapshot
	

durée de dépassement de δ_min
	

δ_min pendant ≥ 2 s
	

tick carnet
	

entrée

δ_guard
	

par palier
	

à chaque évaluation
	

palier courant
	

δ_min
	

franchissement de palier à 240 s
	

prix, recotation forcée

basis
	

événementiel
	

chaque ligne spot
	

médiane glissante 60 s
	

bande 3σ
	

tick spot
	

intégrité seule, jamais un signal

position, ordres en vol
	

continu
	

avant chaque décision
	

état vérité + API
	

état API
	

chaque décision
	

divergence → refus total

signe à l'envoi
	

à l'envoi
	

à chaque tick
	

jusqu'à la fin de vie de l'ordre
	

signe actuel
	

tick oracle
	

annulation prioritaire
Les deux horloges, jamais mélangées

τ-marché : origine imposée de l'extérieur, endDate − 300, identique pour tous, pilote tout ce qui dépend du risque restant. Horloge de capture : propre à chaque flux, pilote la fraîcheur et les trous.


Ce sont deux types incompatibles dans le code : toute addition entre elles échoue à la compilation. Ce n'est pas de la coquetterie. L'erreur inverse a déjà été commise : un temps relatif pris pour absolu déplaçait la fenêtre de règlement de 126 secondes après la fermeture du marché.
Les décalages temporels, explicitement

Décalage
	

Valeur mesurée
	

Conséquence de code

oracle → carnet
	

−0,75 s médian, jamais positif
	

le postulat de départ est mort. Journalisé en indicateur de santé, jamais lu

spot → oracle
	

+1,15 s dans le seul cas mesuré
	

le basis arbitre, l'oracle n'est jamais corrigé par le spot

cadence oracle contre cadence carnet
	

1 s contre 0,013 s, rapport 77:1
	

déclenchement par l'union, 4 Hz plancher

fermeture de fenêtre contre verrouillage de l'issue
	

240 s contre 262 s, écart 22 s
	

la fenêtre passe à [180;270]. C'est le déplacement le plus important

dernier carnet contre dernier oracle
	

273 s contre 298 s
	

τ_max = 270 s


H. Architecture des fichiers

42 fichiers de source, 30 de test, 72 au total. Deux invariants vérifiés par test automatique : execution/broker.py est le seul fichier autorisé à ouvrir une connexion sortante, et journal/ est le seul autorisé à écrire sur disque. Tout le reste est pur, donc rejouable sans réseau ni disque.
Réponses directes à tes questions de repérage

Tu cherches
	

Fichier
	

Précision

la récidive du marché (le côté s'inverse)
	

anchor/anchor.py détecte · decision/arbiter.py donne la priorité 1 · strategy/exit.py gère la position déjà remplie
	

après une traversée, X₆₀ := 1 pendant 60 s, donc en pratique fin de session, et c'est voulu

la répétition / oscillation
	

measure/oracle.py (x60, persistence)
	

un oracle qui oscille est une session où la bonne décision est de ne rien faire

la récidive des erreurs passées
	

tests/static/ (7 fichiers)
	

échouent à la présence d'un motif interdit dans le code, pas à un résultat numérique

la récidive entre sessions
	

regime/regime.py
	

médiane glissante 20 sessions, ratio ∈ [0,4 ; 2,5]

l'oracle
	

measure/oracle.py
	

une seule implémentation de σ₃₀ dans toute la base de code

le temps
	

clock/chronos.py
	

seule source de temps du système

la décision
	

decision/arbiter.py
	

une action par événement, 7 assertions

l'achat
	

strategy/signal.py (intention) → execution/executor.py (envoi)
	

Signal ne produit jamais un ordre, seulement une intention

la vente
	

strategy/exit.py → execution/executor.py
	

idem

l'état
	

state/positions.py
	

réconciliation avant chaque décision

l'analyse et la validation
	

dry_run/report.py, gate/gate.py
	

11 critères, verdict binaire
Arborescence complète

bot/
├── pyproject.toml                dépendances épinglées, seuils de couverture
├── README.md                     thèse, limites, checklist de passage en live
│
├── config/
│   ├── constants.py              NON configurables : TICK, K_LAMBDA=0,5513, T=300, SIGMA_WINDOW=30, PARTS_MIN=5
│   ├── params.yaml               configurables et versionnés : z_min (PROVISOIRE), δ_guard, D_max, seuils, C
│   ├── params.py                 chargement, validation de bornes. Refuse de démarrer si hors bornes
│   └── modes.py                  drapeaux et préconditions : PASSIVE_ONLY/TAKER_ASSUMED, SETTLE_ONLY/EARLY_EXIT
│
├── core/                         (Kernel)
│   ├── types.py                  toutes les structures immuables
│   ├── codes.py                  25 codes de refus, 10 états, 4 sous-groupes
│   ├── units.py                  Dollar, PointDeContrat, SecondeMarché, SecondeCapture — types DISJOINTS
│   └── tick.py                   arrondi, prix complémentaire, côté miroir
│
├── journal/                      (Chronicle) — RANG 1 D'IMPLÉMENTATION
│   ├── chronicle.py              append-only. Échec d'écriture = arrêt du bot
│   ├── schema.py                 24 champs + 8 horodatages + sous-groupe + code
│   └── health.py                 agrégats hors ligne. Consomme ce que la décision n'a PAS le droit de lire
│
├── clock/chronos.py              (Chronos) τ₀, τ, phase, paliers, w(τ), τ_lock
├── ingest/
│   ├── intake.py                 union des événements, 4 Hz, âge, trou, disponibilité
│   ├── dedup.py                  doublons et désordre
│   └── streams.py                adaptateurs oracle, carnet, spot, trades, officiel
├── anchor/anchor.py              (Anchor) K, k_source, m*, côté miroir, traversée
├── measure/
│   ├── oracle.py                 (Oracle) σ₃₀ estimateur unique, σ_T, λ, z, Φ(z), X₆₀, P, v, TWAP30
│   └── garnet.py                 (Garnet) ask, bid, spread, e, méd₃ₛ, quantiles, invariants, δ
├── fees/fees.py                  3 fonctions NON interchangeables + garde + bande morte
├── microstructure/prism.py       (Prism) 5 filtres, chacun avec valeur mesurée ET seuil appliqué
├── regime/regime.py              (Regime) ratio 20 sessions, volatilité annualisée implicite
├── validation/warden.py          (Warden) pile de vetos, arrêt au premier faux
├── strategy/
│   ├── signal.py                 porte d'entrée, prix limite, taille
│   ├── exit.py                   V0 à V8 dans l'ordre imposé, verrou terminal
│   └── mechanisms.py             M1, M2, M3 avec préconditions. Ordre imposé : M2 → M3 → M1
├── risk/risk.py                  N, capacité, budget, kill-switch, détection d'adaptation
├── decision/arbiter.py           (Arbiter) priorités, idempotence, 7 assertions
├── execution/
│   ├── executor.py               verrou d'unicité, post_only, validité 2 s, annulation
│   ├── lifecycle.py              machine à états d'un ordre, 8 horodatages
│   └── broker.py                 SEUL point réseau sortant du projet
├── state/positions.py            (Positions) état vérité, réconciliation, sous-groupe, PnL
├── dry_run/
│   ├── harness.py                rejeu déterministe bit-à-bit, horloge simulée
│   ├── mirror.py                 génération de jeux miroir : l'outil du test de non-directionnalité
│   ├── fill_sim.py               contact du carnet et remplissage réel : DEUX ÉTIQUETTES DISTINCTES
│   ├── sweep.py                  balayage de configurations et de δ
│   └── report.py                 rapport PAR SESSION, PnL par sous-groupe, intervalles de confiance
├── gate/gate.py                  11 critères de passage en live, critères de retrait
├── main/
│   ├── run_replay.py             rejeu hors ligne, aucun réseau
│   ├── run_dry.py                temps réel, aucun ordre envoyé
│   └── run_live.py               refuse de démarrer si Gate n'est pas vert
└── tests/
    ├── static/  (7)              aucune sentinelle · σ₃₀ jamais constante · une seule implémentation ·
    │                             pas de réutilisation croisée des frais · aucun terme directionnel lu ·
    │                             constantes supprimées absentes · aucune valeur magique
    ├── unit/  (19)               un par module
    ├── integration/  (6)         17 étapes · 19 dégradations · zéro double ordre ·
    │                             symétrie miroir · régression sur le corpus · couverture du registre
    └── fixtures/                 sessions réelles, jeux miroir, cas limites synthétiques

Les trois tests qui comptent vraiment

    test_corpus_regression.py : le harnais avant la stratégie. Le rejeu du corpus sous l'ancienne configuration doit reproduire 0 entrée, décalage médian ≤ 0 s, σ₃₀ médian ≈ 7,20 $. S'il ne les reproduit pas, le harnais est faux, et aucun résultat de stratégie ne vaut rien. C'est le premier test à écrire après le journal.
    test_mirror_symmetry.py : sur un jeu de données réfléchi autour de K, le bot doit produire des décisions rigoureusement miroir. Une asymétrie révèle un pari caché. C'est le seul test qui pourra un jour prouver la non-directionnalité, et c'est celui que le corpus ne permettait pas de faire (3 sessions, toutes du même côté).
    test_no_double_order.py : rafale de 1 000 événements en 1 seconde, avec doublons, désordre, accusés retardés, remplissages partiels et déconnexions. Zéro double ordre. Un seul échec suffit à invalider.


I. Évolution de la stratégie : les ajustements, du plus lourd au plus fin
Les six corrections structurelles

1. La volatilité gelée → mesurée en ligne. Avant : σ₃₀ = 26,50 $, constante. Problème : mesure réelle à 7,20 $ médian. Facteur d'erreur ×3,68. C'était une erreur d'unité, pas un réglage prudent. Effet : la barre de conviction était placée à 25,78 $ à τ=60 quand la dérive réellement observée était de 8,55 $. Trois fois au-dessus de la distribution qu'elle était censée filtrer. Résultat : 0 entrée sur la totalité du corpus, PnL 0,00 $. Correction : mesure fenêtre par fenêtre, bornes de plausibilité [0,5 ; 40] $, refus si la fenêtre de 30 s n'est pas pleine. C'est l'ajustement le plus important de tout le projet.


2. Le mécanisme économique remplacé. Avant : latence de propagation oracle → carnet. Problème : elle n'existe pas (0/27). Correction : la décroissance déterministe de σ_T. Effet : la dégressivité de la décote devient une conséquence arithmétique et non un réglage arbitraire.


3. La fenêtre déplacée de [60;240] vers [180;270]. Problème : l'issue se verrouille à 262 s, soit 22 s après la fermeture de l'ancienne fenêtre, et 16 à 18 % du chemin de prix est posté dans les 20 % finaux. Le postulat cherchait à décider avant que l'issue existe. Avant 180 s, toute position est à 1,6 σ₃₀ d'exposition au sens. Trois familles de mesures indépendantes réclament ce déplacement. C'est une condition de non-directionnalité, pas un réglage de performance.


4. 22 tests → 6 dimensions. Problème : la conjonction complète donnait 0 entrée sur 610 instants. Les tests se recouvraient massivement (R² 0,96 entre mid et ask, 0,90 entre ask et N, 0,90 entre ratio et p). Correction : validité de la mesure, temps restant, détermination z, écart carnet/ancrage, qualité de carnet, état interne. Ne pas coder 22 conditions comme 22 informations.


5. Tous les signaux directionnels sortis de la décision. L'écart signé était anti-corrélé au résultat. La pente n'autorisait qu'un sens. Le test de basis était vrai 1 943 fois sur 1 943 : il n'a jamais rien filtré. Correction : tout devient valeur absolue ou garde symétrique. Journalisé, jamais lu.


6. Le journal passe en rang 1. Aucune ligne de code de trading avant lui. Un échec d'écriture arrête le bot. Sous 100 % de journalisation, tout le reste est invalide.
Les petits ajustements qui décident de tout

Ce sont ceux-là que tu voulais. Chacun paraît mineur. Chacun est la différence entre un bot qui cote et un bot qui ne cote jamais.


Ajustement
	

Avant
	

Après
	

Pourquoi c'est déterminant

Borne basse de spread
	

spread ≤ 0,04
	

spread ∈ [0 ; 0,04], strict des deux côtés
	

deux carnets croisés passaient à tort. Un spread négatif était interprété comme une opportunité

Suppression de la valeur sentinelle
	

−1 pour « indisponible »
	

booléen de disponibilité séparé
	

−1 ≤ 0,04 est vrai. C'était un fail-open accidentel : le bot cotait sur des champs absents

Seuil de cyclicité durci
	

X₆₀ < 3
	

X₆₀ = 0
	

X₆₀ = 0 mesuré à 100 % de réussite (n=335) contre 99,5 %. Et surtout : c'est le marqueur d'oracle décroché, seul régime où l'écart médian est positif

Quantiles mesurés au lieu de constantes
	

saut d'ask ≤ 0,01
	

quantile 95 mesuré en ligne, plancher 0,02, plafond 0,30
	

le p95 réel est 0,10 à 0,20. Le pas médian du carnet est 2 cents. Le test refusait le marché normal, seuil 10 à 20 fois trop serré

Garde de vitesse symétrisée
	

pente ≥ −0,40 $/s
	

`
	

Δm*

Persistance raccourcie
	

10 s
	

5 s cumulées, avec remise à zéro sur changement de signe et gel sur trou > 8 s
	

10 s est trop long face à une cadence d'oracle de 1 s. Réduisait le nombre d'événements éligibles sans rien protéger

Tolérance sur le strike supprimée
	

±5,00 $
	

zéro tolérance
	

une tolérance de 5 $ égale à la marge minimale exigée annulait la condition de marge. Elle la vidait de son sens

Plafond de prix remplacé
	

min(0,70 + 0,00075τ ; 0,92)
	

(1 − ask) > frais + 0,02
	

refusait 198 instants sur 198 selon la session, et le plafond de 0,92 était inerte (atteint à 293 s, hors fenêtre). Aucune régression n'étayait ces trois constantes

Coupe-circuit déplacé
	

e ≤ 0,114 en condition d'entrée
	

alerte de journalisation seule
	

l'écart médian mesuré vaut 0,4642 en [240;270[. Le coupe-circuit refusait précisément la phase que le mécanisme cible. Exactement l'erreur reprochée à la version précédente, réintroduite

Cadence forcée à 4 Hz
	

1 Hz
	

4 Hz plancher, union des événements
	

à 1 Hz, 98 % des mises à jour sont invisibles, et la traversée de seuil de 0,300 s est manquée 70 % du temps

Gel du carnet borné
	

seul le trou d'oracle l'était
	

3 s
	

un carnet gelé passait inaperçu. P90 mesuré 1,19 s, max 6,5 à 9,2 s

Une seule formule de taille
	

deux versions coexistaient
	

celle du document d'origine
	

écart de ±1 part selon l'ask, ce qui peut faire basculer la contrainte de 5 parts minimum, donc l'éligibilité elle-même

Trois modèles de frais nommés
	

réutilisation croisée
	

3 fonctions, réutilisation interdite et vérifiée
	

pénaliser l'entrée d'un aller-retour alors que la position va au règlement crée un biais de comparaison

k_λ = 0,5513 requalifié
	

hyperparamètre à optimiser
	

identité √3/π, non configurable
	

un balayage donnait un optimum de 8,00 sur une session et 0,15 sur une autre, facteur 53. Ça ne réfute pas l'identité : ça démontre que le score n'est pas calibré. Deux grandeurs différentes confondues

Recotation forcée au palier 240 s
	

rien
	

annulation et recotation systématiques
	

même si z n'a pas bougé, δ change. Sans ça, l'ordre reste coté à l'ancienne décote dans le nouveau régime de risque

Garde de régime ajoutée
	

rien
	

ratio σ₃₀ / médiane 20 sessions ∈ [0,4 ; 2,5]
	

le corpus est à 6,9 % de volatilité annualisée contre 40 à 60 % typiques. Une décote de 2 ticks calibrée sur ce calme est très inférieure au mouvement adverse d'un régime normal


J. Validation historique
Ce que je peux rejouer, et ce que je ne peux pas

Sois clair sur le périmètre, parce que c'est là que se joue l'honnêteté de l'exercice.


Les documents décrivent 41 blocs d'analyse couvrant environ 25 fenêtres, mais 3 issues indépendantes seulement sont chiffrées instant par instant avec strike, oracle final et distribution d'écart : A_1450, B_1455, C_1500. Les autres sessions sont citées en agrégat (médianes, comptages) sans le détail seconde par seconde nécessaire à un rejeu chronologique.


Ce qui manque précisément pour un rejeu complet, et où le trouver : les 12 CSV normalisés (data/), le fichier brut de 9 988 lignes, et endDate par session. Donne-les moi et je rejoue les 25 fenêtres avec le code exact, sans aucune information future.
Le rejeu des trois sessions, strictement chronologique

Méthode : à chaque τ, je n'utilise que ce qui est disponible à τ. Je calcule z = |m*(τ)| / (σ₃₀ · √((300−τ)/30)) avec le σ₃₀ de la session, et je vérifie la porte finale.
C_1500: entrée, gain, robuste

Élément
	

Valeur disponible à l'instant

σ₃₀ mesuré (médian session)
	

3,05 $

σ_T(180)
	

3,05 × √4 = 6,10 $

|m*| en fin de fenêtre
	

≈ 22 $ (final −22,09)

z(180)
	

≈ 3,6 ≫ 1,0 ✅

X₆₀
	

concordance de signe 100 % dès [30;60[, aucune traversée tardive ✅

Écart contre frais
	

139 instants sur 241 (57,7 %) avec écart > frais ✅

Décote médiane
	

+0,0824, positive ✅


Décision : ENTRÉE sur le côté NON, dans [180;270], cotation passive. Règlement DOWN. GAIN.

C'est la seule session du lot où l'écart est réellement là, et l'ancienne formule l'a refusée : 159 refus imputés au seul test de latence. C'est la faute la plus coûteuse du corpus.
B_1455: refus, robuste

Élément
	

Valeur

σ₃₀ mesuré
	

5,75 $

|m*| final
	

0,84 $

σ_T(270)
	

5,75 $

z(270)
	

0,84 / 5,75 = 0,15 ≪ 1,0 ❌

Décote médiane
	

−0,0547, négative

Écart > frais
	

29 instants sur 241 (12 %)


Décision : REFUS, code MARGE_INSUFFISANTE. Aucune position. Résultat 0,00 $.


C'est le résultat le plus important de cette validation. L'ancienne formule est entrée sur B à τ=122 s, ask 0,73, et a gagné +1,54 $. Elle a gagné avec 84 cents de marge finale sur un strike à 64 735 $. C'est de la chance, pas un écart. Et surtout : si le strike inféré est le bon plutôt que le strike ajusté, B change de camp et devient une perte. Un écart de 12,25 $ sur le strike inverse le règlement de cette session.


La version finale refuse B pour la bonne raison, avec la bonne règle, sans connaître l'issue. Et le refus est robuste : pour n'importe quelle valeur plausible de σ₃₀, z reste sous 1,0.
A_1450: indéterminé, et je te dis exactement quelle donnée manque

τ
	

|m*| disponible
	

σ_T avec σ₃₀ = 11,48
	

z
	

Verdict

90
	

7,94 (signe positif, UP)
	

22,96
	

0,35
	

❌ hors fenêtre de toute façon

120
	

1,38
	

21,0
	

0,07
	

❌ hors fenêtre

150
	

17,15
	

19,4
	

0,88
	

❌ hors fenêtre

210
	

18,42
	

19,88
	

0,93
	

❌ refus, de justesse

240
	

≈ 18,4
	

16,24
	

1,13
	

✅ passe, de justesse


A est borderline, et le point de bascule est à z = 1,0. Avec σ₃₀ = 11,48 (médiane de session), A entre vers τ=240 sur le côté NON, qui est le côté gagnant : GAIN. Avec σ₃₀ au maximum intra-session (19,89), σ_T(240) = 28,1 et z = 0,65 : REFUS. Avec σ₃₀ au minimum (3,01), z = 7,0 : entrée franche.


La donnée manquante est précisément celle-ci : σ₃₀ mesuré sur la fenêtre [180;270] de A_1450, pas sa médiane de session. Le corpus ne donne que la médiane et les extrêmes (facteur 6,6 d'amplitude intra-session). Elle se lit en trois lignes dans oracle-*.csv de A_1450. Sans elle, je ne peux pas te dire si A entre, et je ne vais pas choisir la valeur qui m'arrange.


Ce qui est certain sur A : la version finale ne peut pas se faire piéger par le faux départ. Sur [60;120[, le signe est juste 41,7 % du temps, pire qu'une pièce, et les deux seules pertes de toutes les stratégies naïves testées sont là (achat UP à τ=90 → −4,58 $, à τ=120 → −4,68 $). La fenêtre [180;270] exclut structurellement ces deux instants. Le déplacement de fenêtre n'est pas une optimisation rétroactive : il est justifié par trois mesures indépendantes qui ignorent l'issue.
Verdict de la validation

Session
	

Ancienne formule
	

Version finale
	

Perte évitée

A_1450
	

aucune entrée, 0,00 $
	

entrée tardive probable sur le côté gagnant, indéterminé faute d'un σ₃₀ local
	

oui : le piège de [60;120[ est hors fenêtre par construction

B_1455
	

entrée, +1,54 $ par chance
	

refus, 0,00 $
	

oui : refuse le pari à 84 cents de marge, et refuse le risque d'inversion du strike

C_1500
	

refusée par le test de latence
	

entrée, gain
	

non applicable : c'est le gain que l'ancienne version a raté


Ce que je peux affirmer : sur les trois sessions dont j'ai les données instant par instant, la version finale ne produit aucune perte. Zéro. Et elle capture la seule session où l'écart existait vraiment.


Ce que je refuse d'affirmer, et pourquoi tu ne devrais pas me le demander : que ce soit une garantie. Trois issues indépendantes, un marqueur à 3/3, intervalle de confiance de Clopper-Pearson [29,2 % ; 100 %]. Un taux réel de 30 % est statistiquement compatible avec ces données. Il faut 29 sessions consécutives sans faute pour rejeter l'hypothèse d'un taux ≤ 90 %. Et les trois sessions sont du même jour, dans le même sens, à 6,9 % de volatilité annualisée. Un filtre qui pariait systématiquement dans ce sens afficherait un sans-faute sans rien prédire, et je ne peux pas distinguer la stratégie de ce cas-là avec ces données.


Une FAU qui gagne 100 % des sessions passées est facile à écrire : il suffit de regarder les réponses et de placer les seuils autour. C'est exactement le mécanisme qui a produit les échecs précédents. Ce que je te livre est différent : chaque seuil est justifié par une mesure qui ignore l'issue, et c'est pour ça qu'il a une chance de tenir sur la session suivante.
Le critère de retrait, non négociable

PnL strictement positif dans chacun des 4 sous-groupes (oracle en hausse / en baisse × OUI gagne / NON gagne), sur ≥ 200 sessions avec ≥ 100 par sous-groupe. Un seul sous-groupe négatif entraîne le retrait immédiat, quel que soit l'agrégat. Un PnL positif avec un sous-groupe négatif est un pari. Et si aucune valeur de z_min, δ_guard ou D_max ne garde les quatre positifs, le mécanisme est retiré, pas réajusté.


K. Feuille de route d'implémentation
Ordre imposé par les dépendances. Aucune étape ne commence avant que la précédente soit verte.

Étape
	

Fichiers
	

Erreur à ne pas refaire
	

Critère de passage

0. Journal (rang 1 absolu)
	

core/*, journal/*, config/*
	

sentinelles numériques, horloges mélangées, code de refus non atomique, littérales dans le code métier
	

100 % des décisions et des refus journalisés avec un code unique. Zéro ligne écrite après confirmation

1. Temps et frais
	

clock/chronos.py, fees/fees.py
	

time() comme origine, temps relatif pris pour absolu, réutilisation croisée des frais
	

refus de session sans endDate. Les 3 fonctions de frais prouvées distinctes

2. Ingestion
	

ingest/*
	

évaluation à 1 Hz, recalcul rétroactif, doublon qui avance un compteur
	

cadence effective ≥ 4 Hz mesurée. Le krach de 0,279 s est vu

3. Oracle et ancrage
	

anchor/anchor.py, measure/oracle.py
	

toute la stérilité passée est ici : σ₃₀ gelée, deux estimateurs, un proxy de strike
	

une seule implémentation de σ₃₀ dans la base. Strike officiel sur ≥ 99 % des sessions

4. Termes observés
	

measure/garnet.py
	

écart signé, quantiles en constantes, score traité en probabilité
	

écart toujours ≥ 0. Prix NON dérivé = observé à 1 tick près sur 1 025 lignes

5. Régime
	

regime/regime.py
	

appliquer un calibrage de régime calme à un régime agité
	

volatilité annualisée publiée par session. Historique < 20 sessions → refus

6. Microstructure
	

microstructure/prism.py
	

spread sans borne basse, seuils absolus, garde unilatérale
	

taux de refus par test entre 5 % et 30 %. Aucun à 0 % ni à 100 %

7. Positions (avancé)
	

state/positions.py
	

position fantôme tolérée, PnL calculé avec les frais d'entrée
	

réconciliation 100 %, sous-groupe sur 100 % des événements

8. Validité
	

validation/warden.py
	

le piège principal : reconstruire un filtre stérilisant
	

zéro cotation sur données incohérentes. Un test qui refuse 100 % est faux par construction

9. Risque
	

risk/risk.py
	

deux formules de taille, branche morte
	

une seule formule. Budget borné à 400 $ sur 100 sessions

10. Signal
	

strategy/signal.py, mechanisms.py
	

appeler « passif » un ordre qui traverse le spread
	

≥ 0,5 événement éligible par session. En mode passif, aucun prix limite au-dessus du bid

11. Sortie
	

strategy/exit.py
	

HOLD silencieux, ask utilisé pour une vente au bid, empiler 9 conditions sur 20
	

les 7 contradictions lèvent. V1 et V5 inertes

12. Décision
	

decision/arbiter.py
	

deux actions par événement, traiter entrée et sortie comme deux filtres empilés
	

zéro état incohérent sur 500 sessions rejouées

13. Exécution
	

execution/*
	

modification en place, compter un gain avant confirmation
	

8 horodatages croissants. Annulation ≤ 500 ms au P90. Zéro remplissage sur ordre périmé

14. Dry run et instrumentation
	

dry_run/*, main/run_*.py
	

compter un contact du carnet comme un remplissage
	

rejeu reproduisant 0 entrée sous l'ancienne config. Puis instrumentation sur ≥ 100 sessions

15. Passage en live
	

gate/gate.py, main/run_live.py
	

traiter un critère non évaluable comme neutre au lieu de rouge
	

11 critères verts, versionnés. PnL positif dans chacun des 4 sous-groupes
L'ordre des mécanismes, non négociable

M2 (intégrité)  →  M3 (instrumentation, 5 parts)  →  M1 (cotation, taille réelle)
aucun ordre        ordres minimaux, ≤ 400 $         ordres réels
PnL 0,00 $         espérance ≈ 0                    PnL à démontrer


L'instrumentation avant la cotation n'est pas une précaution : c'est le seul mécanisme qui transforme les onze champs manquants en données. Sans elle, la table de décote restera pour toujours une table de contacts du carnet, jamais de remplissages, et aucun PnL par sous-groupe ne sera jamais calculable.


Les cinq décisions qui bloquent le codage de la stratégie

Le journal, les structures, le temps, les frais et l'ingestion (étapes 0 à 2) peuvent démarrer maintenant. La stratégie, non.


#
	

Question
	

Ma recommandation

D1
	

Cotation passive ou achat assumé ? La porte telle qu'écrite garantit un prix limite au-dessus du meilleur ask, donc un ordre agressif à chaque déclenchement. Ce n'est pas +0,075 $ par événement, c'est +1,18 $ ou −4,80 $ avec un point mort à 80-96 % de réussite directionnelle
	

Passif, post-only, sans hésitation. L'option agressive parie que le score bat le carnet, alors que le carnet réplique déjà l'oracle à R² = 0,904

D2
	

Sortie unique au règlement, ou sortie anticipée ?
	

Règlement unique, conséquence directe de D1

D3
	

Le coupe-circuit reste-t-il dans la porte ?
	

Alerte de journalisation seule. Le garder en dur reproduit la stérilité passée

D4
	

Quel plancher de décote réel ?
	

Frais de règlement + 1 tick ≈ 0,0132, donc δ = 0,02 admissible. L'instrumentation balaiera les deux de toute façon

D5
	

Que faire d'une position remplie dont le côté s'inverse ?
	

Journaliser seul d'abord, avec le PnL des deux branches sur 100 sessions. C'est la seule question où mesurer coûte moins cher que choisir


Et deux garde-fous que je ne lèverai pas, même si tu me le demandes : aucun proxy de strike (12,25 $ d'écart inversent un règlement, et un backtest sur proxy produit des résultats qui semblent signifier quelque chose), et aucune constante gelée là où une mesure en ligne est possible (c'est l'erreur unique qui a stérilisé tout le travail précédent).


Dis-moi « pars sur tes recommandations » et je commence à l'étape 0 dans l'ordre du plan, avec chaque choix par défaut journalisé comme hypothèse révisable dans params.yaml.


Et donne-moi les 12 CSV plus ta FAU. Je rejoue les 25 fenêtres en chronologie stricte, je te sors le PnL ventilé par sous-groupe, et je te dis exactement quelle règle décide sur chaque session. Si ta FAU gagne partout, on regardera ensemble par où l'information de l'issue est entrée dans les seuils. C'est comme ça qu'on saura si elle est solide ou si elle est jolie