# Synoptiques : conventions de dessin

Un synoptique est un dessin SVG (fait par exemple dans Inkscape) importé dans l'app. Il s'anime en direct selon l'état réel de l'installation : capteurs, cycles, modules, conditions.

Rien n'est à programmer : tout passe par **l'id des objets** du dessin (Inkscape : *Objet > Propriétés de l'objet > Id*, ou l'éditeur XML `Ctrl+Maj+X`).

Exemple complet : [bac_200l_double_turbine.svg](bac_200l_double_turbine.svg) (version commentée).

---

## Sommaire

1. [Principe : la clé](#1-principe--la-clé)
2. [Récapitulatif des préfixes](#2-récapitulatif-des-préfixes)
3. [Éléments interactifs et affectations](#3-éléments-interactifs-et-affectations)
4. [Valeur affichée : `s_element_value_`](#4-valeur-affichée--s_element_value_)
5. [Animations pilotées](#5-animations-pilotées)
6. [États : `s_state_`](#6-états--s_state_)
7. [Affectation Condition](#7-affectation-condition)
8. [Repères de configuration : `s_cfg_` et `s_link_`](#8-repères-de-configuration--s_cfg_-et-s_link_)
9. [Import, ré-import et limites](#9-import-ré-import-et-limites)
10. [Conseils Inkscape](#10-conseils-inkscape)

---

## 1. Principe : la clé

Chaque objet spécial a un id de la forme `s_<type>_<clé>`. La **clé** relie entre eux tous les objets qui concernent une même chose :

```
s_element_tank1          ← zone tactile, porte l'affectation (ex. cycle « Remplissage »)
s_element_value_tank1    ← texte où s'affiche la valeur
s_wave_tank1-a           ← vague animée quand l'affectation est active
s_hide_tank1             ← surface plane visible à l'arrêt
```

**Règles de la clé**

- Caractères autorisés : lettres, chiffres, `_` et `-`. Pas de `.` ni de `$`.
- Le suffixe `-…` est réservé aux copies. Inkscape ajoute `-1`, `-2`… quand on duplique un objet, et ce suffixe est ignoré : `s_flow_air1-left-core` se rattache à la clé `air1`. On peut donc avoir **autant d'objets que voulu par clé**.
- Préférez des clés **sans `-`** (`tank1`, `pompe`, `00001`). Si deux clés existent, la plus longue qui correspond l'emporte.

---

## 2. Récapitulatif des préfixes

| Id | Rôle | Configuration | Supervision |
|---|---|---|---|
| `s_element_<clé>` | Zone tactile, porte l'affectation | Toucher : ouvre l'affectation | Respire si affecté ; toucher : détail |
| `s_element_value_<clé>` | Texte de valeur | Texte d'origine | Valeur en direct (masqué si non affecté) |
| `s_flow_<clé>` | Flux en tirets le long d'un tracé | Figé | Défile si actif, masqué sinon |
| `s_spin_<clé>` | Rotation (turbine, ventilateur) | Figé | Tourne si actif |
| `s_blink_<clé>` | Clignotement | Figé | Clignote si actif, normal sinon |
| `s_flash_<clé>` | Voyant qui clignote | Figé | Clignote si actif, masqué sinon |
| `s_show_<clé>` | Visible en marche | Figé | Visible seulement si actif |
| `s_hide_<clé>` | Visible à l'arrêt | Figé | Visible seulement si inactif |
| `s_wave_<clé>` | Surface de liquide qui ondule | Figé | Glisse si actif, masqué sinon |
| `s_state_<clé>-<valeur>` | Variante d'un état | Figé | Visible seulement si l'état vaut `<valeur>` |
| `s_cfg_<clé>` | Nom lisible de l'élément | Affiché, nomme la ligne | Masqué |
| `s_link_<clé>` | Repère pour toucher l'élément | Affiché, clignote si sélectionné | Masqué |

---

## 3. Éléments interactifs et affectations

### `s_element_<clé>`

C'est l'objet central : sans `s_element_<clé>`, la clé n'existe pas et rien d'autre ne peut s'y rattacher. Ce peut être une forme ou un groupe entier (un bac complet par exemple).

- Chaque `s_element_` détecté devient une ligne dans la liste « Éléments interactifs » du mode Configuration.
- Si deux objets portent le même id `s_element_`, le premier du document gagne.
- En **supervision**, un élément affecté **respire** (son opacité pulse doucement, toutes les 2,4 s), pour indiquer qu'on peut le toucher.

### Les types d'affectation

En mode Configuration, on choisit pour chaque élément l'une de ces affectations :

| Choix | Cible | Valeur affichée | « Actif » quand… | Valeur d'état (`s_state_`) |
|---|---|---|---|---|
| **Cycle** (cycle, groupe) | un cycle | avancement `25 %` si durée, sinon *En cours* / *Arrêté* | il est en cours | `on` / `off` |
| **Cycle** (module) | un module | *ON* / *OFF* | il est en cours | `on` / `off` |
| **Cycle** (proportionnel) | une vanne proportionnelle | ouverture `35 %` | elle est en cours | `on` / `off` |
| **Capteur** | un capteur | dernière mesure + unité `21.5 °C` | sa valeur est > 0 (débit, pression…) | valeur brute (`21.5`, `rain`…) |
| **Synoptique** | un autre synoptique | son nom | jamais | — |
| **Condition** | une expression de conditions | *Vrai* / *Faux* | l'expression est vraie | `true` / `false` |

Cycles et modules sont réunis sous le choix « Cycle ». Le bon type est déduit de la cible choisie.

### Toucher un élément en supervision

| Affectation | Effet |
|---|---|
| Cycle / module | Carte du cycle : marche/arrêt, durée forcée, passage de séquence… |
| Capteur | Carte du capteur et date de la dernière mesure |
| Synoptique | Ouvre directement le synoptique cible |
| Condition | Détail de l'expression : résultat global, puis chaque condition avec sa valeur actuelle |

Une cible supprimée depuis s'affiche « Cible introuvable » (en rouge en configuration), et sa valeur affiche `—`.

---

## 4. Valeur affichée : `s_element_value_`

Un objet `s_element_value_<clé>` reçoit la valeur en direct de l'affectation de `s_element_<clé>`.

- **Texte** (`<text>`) : son texte est remplacé. Le premier `<tspan>` reçoit la valeur et garde sa police, sa taille et sa position. Les lignes suivantes sont vidées.
- **Groupe** (fond + texte + unité…) : seul le **premier** `<text>` du groupe est remplacé, les autres textes restent intacts.
- Une forme sans texte (`rect`…) n'est pas modifiée.
- **Copies** : un texte copié dans le groupe (`s_element_value_tank1-1`) est aussi rempli.
- **Sans affectation**, la valeur est **masquée en supervision**. En configuration, le texte d'origine du dessin reste affiché (mettez `--` par exemple).
- Un `s_element_value_` sans `s_element_` correspondant est ignoré, et masqué en supervision.

---

## 5. Animations pilotées

Un objet `s_<animation>_<clé>` s'anime selon l'état **actif / inactif** de l'affectation de `s_element_<clé>` (voir la colonne « Actif quand… » du [tableau des affectations](#les-types-daffectation)). Les animations ne jouent qu'en supervision.

| Préfixe | Actif | Inactif | Période |
|---|---|---|---|
| `s_flow_` | Tirets qui défilent dans le sens du tracé | Masqué | 1 s |
| `s_spin_` | Rotation horaire autour du centre de rotation | Immobile | 1,2 s / tour |
| `s_blink_` | Clignote | Affichage normal | 1 s |
| `s_flash_` | Clignote | Masqué | 1 s |
| `s_show_` | Visible | Masqué | — |
| `s_hide_` | Masqué | Visible | — |
| `s_wave_` | Glisse horizontalement en boucle | Masqué | 6 s |

### Détails par animation

- **`s_flow_`** : si le tracé est en trait plein, des tirets `12, 12` sont posés automatiquement. Avec vos propres tirets (`stroke-dasharray`), leur motif est conservé. Pour inverser le sens : *Chemin > Inverser*.
- **`s_spin_`** : l'objet tourne autour de son **centre de rotation Inkscape** (la croix déplaçable en mode rotation). Pour tourner dans l'autre sens : *Objet > Retourner horizontalement*.
- **`s_flash_`** : posez une copie recolorée (couleur « en marche ») par-dessus l'objet d'origine. La couleur vient du dessin.
- **`s_hide_` + `s_flash_`** : combinaison classique pour un voyant (couleur d'arrêt fixe, couleur de marche qui clignote).
- **`s_wave_`** : le tracé doit contenir **2 motifs identiques côte à côte**, chacun plus large que le bac. Il glisse de la moitié de sa largeur, sans saut visible. Pour avoir une surface stable à l'arrêt, posez dessous un `s_hide_` à bord plat. Plusieurs couches (`s_wave_tank1-a`, `-b`, `-c`) avec des longueurs d'onde différentes donnent un rendu plus naturel.

### À savoir

- Un objet **masqué dans Inkscape** (pour alléger le dessin) **apparaît** quand il doit être visible : `display` est forcé.
- Les `<animate>` éventuellement restés dans le fichier sont ignorés sur ces objets.

---

## 6. États : `s_state_`

`s_state_<clé>-<valeur>` est visible **seulement** quand la valeur d'état de `s_element_<clé>` vaut `<valeur>`, et masqué dans tous les autres cas. Cela permet d'afficher une **variante parmi plusieurs** : météo, mode, position, alarme…

| Affectation | Valeur d'état | Exemples d'id |
|---|---|---|
| Capteur | valeur brute de la mesure | `s_state_meteo-rain`, `s_state_meteo-sun`, `s_state_mode-2` |
| Cycle / module | `on` / `off` | `s_state_pompe-on`, `s_state_pompe-off` |
| Condition | `true` / `false` | `s_state_alarme-true`, `s_state_alarme-false` |

- La comparaison **ignore la casse** (`-RAIN` = `-rain`).
- La valeur ne contient pas de `-` : `s_state_meteo-rain-2` est une **copie** de l'état `rain`. On peut donc avoir plusieurs objets par état.
- Si la valeur n'existe pas (capteur sans mesure, condition indéterminée), **toutes** les variantes sont masquées.
- Un dessin qui masque ses variantes avec `visibility="hidden"` fonctionne aussi : la bonne variante est rendue visible.

### `s_state_` ou `s_show_` / `s_hide_` ?

Pour un cycle ou une condition, `s_show_` correspond presque à `-on`/`-true`, et `s_hide_` à `-off`/`-false`. La différence apparaît quand la valeur est **inconnue** :

| Situation | `s_show_` | `s_hide_` | `s_state_…-true` | `s_state_…-false` |
|---|---|---|---|---|
| Vrai / en marche | visible | masqué | visible | masqué |
| Faux / à l'arrêt | masqué | visible | masqué | visible |
| Inconnu | masqué | **visible** | masqué | **masqué** |

Utilisez `s_state_` si vous voulez trois rendus distincts (vrai, faux, rien), et `s_show_` / `s_hide_` pour un simple marche/arrêt.

---

## 7. Affectation Condition

Une affectation **Condition** ne cible pas une ressource, mais une **expression de conditions**, construite comme celle des déclencheurs (triggers). Son résultat **vrai / faux** pilote l'élément.

### Saisie

Mode Configuration → toucher l'élément → **Condition**. Le constructeur est le même que pour les triggers :

- une condition = **Capteur** (opérateur `<`, `>`, `=`, `<=`, `>=` + valeur numérique) ou **Cycle** (`ON` / `OFF`, ou ouverture en % pour une vanne proportionnelle) ;
- toucher **ET / OU** entre deux conditions inverse le connecteur ;
- **Sous-groupe** = une paire de parenthèses ;
- « ET » est prioritaire sur « OU » : `A OU B ET C` = `A OU (B ET C)`.

Une condition incomplète bloque la validation (✓) : la ligne se déplie et les champs manquants passent en rouge. Retirer toutes les conditions revient à supprimer l'affectation.

### Effet sur le dessin

| Résultat | Actif (`s_show_`, `s_flow_`…) | `s_element_value_` | `s_state_<clé>-…` |
|---|---|---|---|
| Vrai | oui | *Vrai* | `-true` visible |
| Faux | non | *Faux* | `-false` visible |
| Indéterminé | non | `—` | aucune variante |

**Indéterminé** : aucune condition n'est évaluable, par exemple parce que tous les éléments comparés ont été supprimés. Une condition dont l'élément a disparu est **ignorée** : `A ET (élément supprimé)` = `A`.

Un cycle dont aucun statut n'a encore été reçu compte comme **OFF**.

### Exemple : voyant d'alarme

Affectation de `s_element_alarme` : `Température > 30 OU (Pompe = OFF ET Niveau < 20)`.

```
s_element_alarme          zone tactile (le cadre du voyant)
s_state_alarme-false      voyant vert
s_state_alarme-true       voyant rouge
s_flash_alarme            halo rouge qui clignote pendant l'alarme
s_element_value_alarme    texte « Vrai » / « Faux »
```

En touchant le voyant en supervision, on voit quelle condition est vraie ou fausse, avec les valeurs actuelles.

---

## 8. Repères de configuration : `s_cfg_` et `s_link_`

Ces objets ne servent **qu'en mode Configuration** et sont masqués en supervision.

### `s_cfg_<clé>` : nom lisible

Un **texte** `s_cfg_…` donne son nom à l'élément dans la liste de configuration (« Bac gauche » au lieu de `tank1`).

- Rattachement, par ordre de priorité :
  1. le `s_element_` qui **contient** le texte, quel que soit le reste de son id (pratique après une duplication Inkscape, qui produit des ids comme `s_cfg_00001-1-0-7`) ;
  2. sinon la clé lue dans l'id (`s_cfg_tank1`, ou `s_cfg_tank1-1`).
- Le premier texte non vide trouvé pour une clé l'emporte. Les lignes (`<tspan>`) sont jointes par une espace.

### `s_link_<clé>` : repère tactile

Un repère (pastille, flèche…) posé près d'un élément petit ou difficile à toucher.

- Toucher le repère déplie l'affectation de `<clé>`.
- Il **clignote** tant que cette affectation est dépliée, pour montrer quel élément on configure.
- Il est figé : aucune animation, y compris dans son contenu.

---

## 9. Import, ré-import et limites

- **Fichier** : SVG avec une racine `<svg>`, **5 Mo maximum**.
- **Nom** : obligatoire, 20 caractères maximum, unique (la casse est ignorée).
- **Remplacer le SVG** : les affectations des clés **toujours présentes** sont conservées. Celles des clés disparues sont perdues, et l'app les liste avant validation. On peut donc retoucher le dessin sans tout réaffecter, à condition de **garder les clés**.
- **Supprimer un synoptique** retire aussi les affectations « Synoptique » qui pointaient vers lui.
- Les affectations sont enregistrées sur la box à la validation (✓). Le retour arrière demande confirmation si des modifications sont en cours.

---

## 10. Conseils Inkscape

### Rendu

L'app corrige automatiquement plusieurs différences entre Inkscape et le moteur de rendu mobile :

- les dégradés Inkscape chaînés (`xlink:href`) sont résolus ;
- l'attribut `style` l'emporte sur les attributs de présentation, comme dans Inkscape ;
- `paint-order: stroke` (contour sous le remplissage) est émulé ;
- `<use>` d'un `<symbol>` sans taille prend la taille de la zone parente.

### Performance

Les parties fixes du dessin sont dessinées une seule fois, et seuls les objets animés sont redessinés. Pour que ce découpage fonctionne :

- **évitez `mask` et `filter`** (flou, ombre portée) sur un groupe qui contient des objets animés ;
- **évitez `<use>`** (clones Inkscape) vers des objets dessinés, et préférez une vraie copie (`Ctrl+D`) ;
- ne mettez pas d'animation dans `<defs>`.

Dans ces cas, tout le dessin est redessiné à chaque image, et le téléphone chauffe sur un synoptique très animé.

Un dessin bien plus large que haut s'affiche petit en portrait : l'app conseille alors de passer en paysage.

### Checklist avant import

- [ ] Chaque objet interactif a un `s_element_<clé>` avec une clé sans `-`.
- [ ] Les textes de valeur sont des `s_element_value_<clé>` (avec un texte provisoire `--`).
- [ ] Les objets animés réutilisent la **même clé** que leur `s_element_`.
- [ ] Sens des `s_flow_` vérifié (*Chemin > Inverser*), centre des `s_spin_` placé.
- [ ] Les `s_wave_` contiennent 2 motifs identiques, avec un `s_hide_` plat dessous.
- [ ] Un `s_cfg_` nomme chaque élément.
- [ ] Pas de flou ni de clone sur les parties animées.
