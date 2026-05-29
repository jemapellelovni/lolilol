# 07 — Lore & univers (MaSurvie V2)

Cadre narratif, ton d'écriture, zones, rangs et events. Destiné à une IA qui doit **écrire dans l'univers** (lore, dialogues, annonces, descriptions d'items, narration).

---

## 1. Le cadre narratif

**MaSurvie** est un serveur de **survie post-apocalyptique** : un monde **contaminé par la radiation**, infesté de **zombies** et de **créatures mutantes**. Le joueur est un **survivant** lâché dans ce monde. Le gameplay (Skyblock + survie) se traduit en fiction par : on **pille des zones dangereuses** pour s'équiper, on **conquiert des territoires** avec sa faction, et surtout on **survit à la radiation** qui ronge le corps à chaque expédition profonde. Règle d'or de l'univers : **plus on va loin, meilleur est le loot — et plus le monde veut votre peau.**

### Ambiance et ton

- **Tendu, crépusculaire, mais avec une étincelle d'espoir** et un **humour noir occasionnel**. Jamais purement désespéré, jamais rigolo.
- Les survivants se regroupent en **îles fortifiées / colonies**. La **nuit, l'Infection rôde**.
- Esthétique mêlant **post-apo zombie** et **STALKER / Tchernobyl** (radiation, dosimètre Geiger, masques à gaz, anomalies, artefacts).

### Plume (style d'écriture imposé)

Issu du lore partagé (`narrator.yml`, commun aux modules Narrator et Flavortext) :

- **Phrases courtes et imagées.** Vocabulaire de survie.
- **Jamais de méta** : on ne dit pas « ce joueur », « le serveur », « le plugin ». On dit **« un survivant »**, **« la colonie »**, **« les ruines »**.
- **2 à 4 phrases max** pour un événement ponctuel.
- **Pas d'emoji dans le corps** de la prose immersive (les emojis sont pour les titres/fields d'annonce).
- L'argent s'appelle **« crédits »** (alias narratif de la monnaie / prime). Les boss de donjon sont des **« aberrations biologiques »**.

### Vocabulaire récurrent (à employer)

`survivant`, `colonie`, `ruines`, `contamination`, `dose` / `mSv`, `signature`, `mutants`, `infectés`, `horde`, `convoi`, `zone`, `tier`, `radiation`, `décontamination`, `iode`, `filtre`, `combi NBC`, `exosquelette`, `masque à gaz`, `Geiger`, `hotspot`, `artefact`, `anomalie`, `pilleur`, `scroll` / `parchemin`, `inscription`, `crédits`, `prime`, `territoire`, `faction`, `taxe`, `créneau`, `rankup`.

### Les 6 voix narratives (personas du bot)

Le narrateur IA tire au sort une des 6 voix (poids relatif) — chacune a un ton distinct, utile pour varier les annonces et les dialogues :

| Persona | Poids | Ton |
| --- | --- | --- |
| **Radio Survivants** | 3 | Radio pirate qui émet encore dans les ruines. Urgent, factuel, espoir fragile, comme un bulletin crachotant. |
| **Dr. Mercier** | 2 | Ancien médecin, journal clinique de l'apocalypse. Posé, analytique, froideur scientifique laissant filtrer l'effroi. |
| **Journal de Camille** | 2 | Survivante, journal intime à la bougie. Personnel, émotionnel, introspectif, première personne. |
| **Caporal Vasquez** | 2 | Militaire, rapports de situation laconiques. Sec, codifié, opérationnel, sans fioritures. |
| **Le Vieux** | 1 | Survivant âgé, sagesse fataliste. Lent, philosophique, métaphores rurales, proverbes amers, conteur au coin du feu. |
| **Graffiti urbain** | 1 | Tag anonyme sur un mur en ruine. Brut, lapidaire, slogans, fragments, presque poétique dans sa violence. |

Le **nom de la persona** sert de **footer** d'embed de narration. Le **bilan quotidien de 22h** ("Chroniques du jour — \<date FR\>") synthétise les événements du cycle.

---

## 2. Outils narratifs en jeu (Narrator, Flavortext)

> **Typewriter : NON utilisé.** Aucune trace de Typewriter (le plugin de dialogues/cinématiques/quêtes) dans le dépôt. La narration n'est **pas** gérée par Typewriter mais par deux modules maison à base d'IA :

- **Narrator** (côté Discord) — transforme un fait brut (« X a tué le boss ») en message d'ambiance posté en embed Discord (± broadcast en jeu). 100 % piloté par commande (`/svnarrate`), pas de listener : l'admin colle `svnarrate …` dans les *reward commands* des autres plugins.
- **Flavortext** (côté jeu) — reformule un texte pour l'**afficher au joueur en jeu** (chat / actionbar / title / subtitle / bossbar) via `/svflavor`. Même lore partagé que Narrator.

Si on doit écrire du contenu narratif, le format attendu est donc : **un fait concret** (nom, lieu, montant) → l'IA habille en prose immersive selon les personas et le mapping émotion→couleur.

---

## 3. Les zones (tiers 0 → 5)

La carte est découpée en **~20 régions WorldGuard** classées par **tier de danger**. Plus le tier est haut : mobs plus forts, **radiation plus élevée**, loot plus rare et meilleur.

### Échelle réelle (zones × étoiles × radiation)

| ★ | `radiation_level` | Régions WorldGuard (slugs réels) |
| --- | --- | --- |
| Sûre (tier 0) | `0` | `spawn` |
| 1★ | `1` | `port_touristique`, `station_radar` |
| 2★ | `2` | `camp_abandonne`, `plage_deserte`, `camp_sauvage` |
| 3★ | `4` | `station_epuration`, `parc_eolien`, `ferme_infectee`, `qg_police`, `secteur_est`, `prison` |
| 4★ | `6` | `aerodrome_ravage`, `port_industriel`, `navire_fantome`, `complexe_nucleaire`, `secteur_ouest` |
| 5★ | `9` | `base_militaire`, `la_plateforme`, `mine_effondree` |

> Les slugs ci-dessus sont les **vrais IDs WorldGuard**. Côté affichage humain (salons vocaux Discord, audit), ils sont rendus jolis (ex. `station_epuration` → « Station d'Épuration », `qg_police` → « QG de Police »). Pour nommer des zones dans du contenu, utilise la forme lisible : Port Touristique, Station Radar, Camp Abandonné, Plage Déserte, Camp Sauvage, Station d'Épuration, Parc Éolien, Ferme Infectée, QG de Police, Secteur Est, Prison, Aérodrome Ravagé, Port Industriel, Navire Fantôme, Complexe Nucléaire, Secteur Ouest, Base Militaire, La Plateforme, Mine Effondrée.

### Ce qui change par tier

- **Mobs** plus nombreux et plus coriaces (hordes adaptées au tier).
- **Radiation** : `radiation_level × 0,6 mSv/s` avant protection. Tier 0 = sûr ; à partir de 3★ la radiation devient un mur sans équipement ; tier 5 = exosquelette + filtre carbone obligatoires.
- **Loot** : coffres LootRegion plus riches, EcoItems/EcoScrolls plus rares, artefacts au tier 5.
- **Loot gating par protection** : niveaux `bronze` (0) → `iron` (masque+filtre) → `gold` (combi NBC) → `diamond` (exosquelette, tier 5).

### Premiers pas canon (utile pour tutos / dialogues d'accueil)

Spawn (tier 0, sûr) → kit de départ → sortie ouest vers **Port Touristique** (1★) → premier coffre LootRegion (fouille animée) → inscription d'un scroll via `/inscribe` → tuer 3 zombies → rentrer déposer le butin → lire `/radhelp` → ouvrir le menu de rankup.

---

## 4. Système de radiation (pilier central)

- **Dose (mSv)** : monte en zone irradiée, par paliers `500` (symptômes légers) → `2000` (modérés) → `4000` (sévères, dégâts) → `6000` (critique) → `8000` = **létal**, mais **mort LENTE** (fenêtre pour fuir et décontaminer, jamais one-shot). Hors zone, la dose s'élimine (~2,5 mSv/s) ; décontamination en zone safe / douche.
- **Protection** (cumulative) : masque + filtre (le filtre s'use) → **combinaison NBC** (intégrité qui se dégrade) → **exosquelette** (alimenté par une **cellule/capsule**, requis tier 5).
- **Signature / contamination** : « niveau de saleté » radioactif, calculé à partir de la dose et du temps d'exposition continu. *Historiquement* elle attirait des mutants de chasse (Bloodsucker, Chimera) — **ce spawn dynamique a été retiré (2026-05-18)** : la signature est toujours calculée et exposée, mais n'a plus d'effet mécanique. Pour du lore, on peut quand même évoquer la traque des mutants par la contamination (ambiance), c'est cohérent avec l'univers.
- **Iode** : pris en préventif, réduit l'absorption avant une sortie.
- **Geiger** : compteur qui crépite selon le niveau de la zone et détecte les **hotspots** (poches de radiation intense).
- **Artefacts** : objets rares récoltés en zones profondes ; très convoités, dangereux à porter sans protection.
- **Anomalies STALKER** : pièges invisibles détectés au boulon — **système retiré du plugin** mais reste un élément d'ambiance évocable.

---

## 5. Items, scrolls et économie (vocabulaire de loot)

- **EcoItems** : objets custom (pioches du pilleur, armes, armures, gadgets, combi NBC…), souvent livrés **pré-inscrits** de leurs scrolls.
- **EcoScrolls** : parchemins qui ajoutent une capacité à un objet. **Pas de table d'enchantement ni d'enclume** sur le serveur : toute la puissance vient des scrolls, gravés via **`/inscribe`** (ou glisser-déposer le parchemin sur l'objet).
- **Crédits / prime** : la monnaie. Le module PrimeTracker lit le solde et calcule un **palier (tier)** avec un **label coloré** (ex. Argent, Or) et un bonus %. Un **top** est annoncé toutes les ~30 min.
- **Coffres LootRegion** : pillables par zone, avec cooldown, animation de fouille, pity/streaks. Exemple d'item canon : `pickaxe_pilleur_t2` (pioche du pilleur tier 2).
- **Forgeron** : atelier de **réparation** d'équipement (préserve les inscriptions). Commande `/forgeron`.

---

## 6. Rangs / grades — le rankup du survivant

- Le **survivant** possède un **grade** qui monte via le **rankup** (la "boucle longue", le fil rouge end-game). On monte de grade en accomplissant des objectifs de progression.
- Distinct du **tier de prime** (PrimeTracker) qui est un palier **économique** (basé sur le solde de crédits, avec label coloré type Argent / Or) donnant un bonus de gains.
- Distinct aussi du **tier de zone** (0→5, danger).

> Il y a donc **trois notions de "tier/rang"** à ne pas confondre :
> 1. **Tier de zone** (0→5) — danger / radiation de la région.
> 2. **Tier de prime** (économique) — palier de richesse, label coloré + bonus %.
> 3. **Grade de survivant** (rankup) — progression de personnage / end-game.

L'end-game (boucle longue) : rankup → capturer un territoire avec sa faction → monter l'exosquelette + filtre carbone → accéder au tier 5 → récolter les artefacts → débloquer des **mutations** (bonus permanents au prix d'expositions).

---

## 7. Events spéciaux & menaces dynamiques

### Créneaux dynamiques (fenêtres de temps qui changent les règles)

Gérés par le module **TimeSlot / créneaux** de SurvivorCore (`timeslots.yml`). Le créneau de plus haute priorité qui matche est actif ; certains effets d'objets ne s'activent **« que pendant un créneau »**.

- **Nuit Creuse** — la nuit : **loot ET danger boostés**. Hordes plus grosses, radiation accrue. Moment fort pour piller gros ou mourir.
- **Rush Weekend** — le week-end : gains accrus (multiplicateurs spawn / drop / économie).

### Convois zombies (module ConvoyModule / InfectedConvoy)

Des **convois d'infectés** traversent la map à intervalles. Mobiles et coriaces : il faut **les suivre et les intercepter** pour un loot d'événement. Persistants et autonomes.

### Hordes dynamiques (module HordeModule / HordeSpawner)

Le monde n'est jamais vide : illusion de **horde infinie**. Où que va le survivant, des silhouettes émergent. En coulisses on triche (recyclage des mobs lointains, ~15 mobs réels autour d'un groupe). La horde grossit avec : le **nombre de joueurs** (pas linéairement), la **nuit**, le **tier de zone**, un **convoi** en cours, ou la **contamination** du groupe. Les mobs ciblent plus le joueur **faible ou contaminé**. Les **boss MythicMobs** sont gérés à part (période de grâce, jamais recyclés).

### Mutants & boss

Créatures mutantes nommées dans le lore : **Bloodsucker**, **Chimera**. Les boss sont des **aberrations biologiques** (MythicMobs). Ils ne disparaissent pas si on s'éloigne un instant.

### Anomalies STALKER & mutations

- **Anomalies** : pièges invisibles de zones radioactives (détectés au boulon, mortels, gardent des artefacts). Système mécanique **retiré** mais élément d'ambiance valide.
- **Mutations** : bonus permanents débloqués en end-game, payés par l'exposition à la radiation.

### Territoires / factions

Les **factions** capturent des régions WorldGuard. Posséder un **territoire** = bonus passifs + **taxe** prélevée sur les pilleurs qui y entrent. Les zones se **contestent**. Capture = bon sujet d'annonce collective (couleur verte).

---

## 8. Ce qu'il faut éviter (cohérence)

- Ne pas parler de **table d'enchantement / enclume** (désactivées) ; la puissance vient des **scrolls**.
- Ne pas écrire de **mort instantanée** à la radiation : c'est une **mort lente** volontairement progressive (tension de fin d'expédition).
- Ne pas confondre les trois "tiers" (zone / prime / grade) — voir §6.
- Ne pas employer de méta (« joueur », « serveur », « plugin ») dans la prose immersive — dire « survivant », « colonie », « les ruines ».
- Typewriter n'existe pas ici : ne pas inventer de système de dialogues/cinématiques basé dessus.
