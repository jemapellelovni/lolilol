# 02 — Stack de plugins MaSurvie V2

> Inventaire complet de ce qui tourne sur le serveur, croisé entre les **jars de prod**
> (`deploy/`) et les **cartes du hub wiki** (`wiki/README.md`). Trois catégories :
> (a) SurvivorCore + ses modules (code maison), (b) plugins/services maison autonomes,
> (c) plugins tiers / premium (décrits seulement, pas de code copié).

**État de référence au 2026-05-29.** L'évolution majeure du projet est l'**absorption**
progressive d'anciens plugins standalone en **modules de SurvivorCore**. Les jars de
prod sont aujourd'hui très peu nombreux ; presque tout le gameplay vit dans
`SurvivorCore.jar`.

---

## Jars réellement présents dans `deploy/` (prod)

| Jar | Catégorie | Rôle court |
|---|---|---|
| `SurvivorCore-2.0.0.jar` | maison (socle) | Plugin central, ~24 modules absorbés. **C'est le cœur.** |
| `EpicAchievements-2.10.0-multitask.jar` | maison (fork) | Système de succès / carnet (fork modifié multi-tâches). |
| `BlueBridgeCore-2.2.jar` | tiers (fork local) | Pont BlueMap (carte web), base. |
| `BlueBridgeWG-2.2.jar` | tiers (fork local) | Affiche les régions WorldGuard sur BlueMap. |
| `SuperiorSkyblock2-missions/MaSurvieSSB2Missions.jar` | maison | Missions SSB2 (va dans `plugins/SuperiorSkyblock2/missions/`, **pas** `plugins/`). |

> Note : `deploy/` ne contient **que les jars maison + les forks locaux**. Tous les
> plugins **tiers** (SuperiorSkyblock2, WorldGuard, PlaceholderAPI, Vault, MythicMobs,
> ModelEngine, ItemsAdder, zAuctionHouse, AxTrade…) sont installés à la main côté
> serveur et ne transitent pas par `deploy/`. `server-patch/` contient des jars du
> stack eco historique (eco/libreforge/EcoItems/EcoScrolls) **désormais retiré du
> gameplay** (remplacé par le module **ItemModule** de SurvivorCore) — à ne pas considérer comme actif.

---

## (a) SurvivorCore + ses modules — code maison

**SurvivorCore** (`SurvivorCore-2.0.0.jar`, jar ~11,8 MB) — Purpur 1.21.x, Java 21.
Plugin socle modulaire. **Hard depend : PlaceholderAPI + Vault** (se désactive si
absents). Tout le reste est softdepend (réflexion ou compileOnly). Storage dual
MySQL/SQLite, config DB unifiée dans `plugins/SurvivorCore/bdd.yml` (source unique,
pool partagé `getSharedDataSource()`). Lang centralisé `lang_fr.yml`.

Modules (chaque ancien plugin est devenu un module ; package d'origine souvent
préservé pour ne pas casser les API consommatrices) :

| Module | Origine | Rôle | Intégration |
|---|---|---|---|
| **PrimeTracker** | natif core | Économie / tiers / primes, top prime (joueurs en ligne uniquement). | PAPI `%survivorcore_*%` (balance en entier). |
| **TimeSlots** | natif core | Créneaux dynamiques (Nuit Creuse, Rush Weekend), days-of-week + wrap minuit, TZ Europe/Paris. | Conditionne effets « pendant un créneau ». |
| **SpawnControl** | natif core | Contrôle du spawn (blocages, comptes). | Table `spawn_blocked_count`. |
| **ScheduleManager** | natif core | 5 types de schedules (FIXED, RANDOM_*…). | Déclenche bilans IA, events. |
| **DeathMessages** | natif core | Messages de mort + killstreaks. | Table `death_history`. |
| **tickets** | ex-SurvivorTickets | Tickets support ; bot Discord (panel/modals/transcripts). | CoreHttpService :25580 ; MySQL partagée ; module bot `tickets`. |
| **discordlink** | maison | Liaison Discord⇄MC (remplace DiscordSRV) : codes 6 chiffres, perks continus, rôles de région. | REST+WS :25580 Bearer ; LuckPerms/WG soft. |
| **narrator** | maison | Narrateur IA (fait brut → message d'ambiance Claude), embed Discord ± broadcast. | Bot Node module `narrator` ; lore `narrator.yml`. |
| **flavortext** | maison | Reformule un texte brut en message immersif via Claude (appel Java direct). | Lore partagé `narrator.yml` ; coût à AiBudget. |
| **aibudget** | maison | Garde-fou coûts IA (table `ai_costs`), limite mensuelle 10 €, throttle/stop. | `POST /ai-cost` ; alertes webhook. |
| **zonespawn** | maison | Zones d'extraction (cuboids WorldEdit), stats joueur. | Table `zonespawn_stats` ; `ZoneSpawnSuccessEvent`. |
| **itemmodule** | maison | Moteur d'items à capacités YAML (remplace EcoItems) : 11 effets, boutons inventaire, poche sécurisée 4 slots, furtivité BlueMap. | Bridges RadiationCore/LootRegion/WG ; `/inscribe`. |
| **forge** | ex-MaSurvie_Forgeron | Forge de réparation (GUI native), durabilité ItemsAdder CustomStack. | depend ItemsAdder ; 8 events. |
| **horde** | ex-HordeSpawner | Hordes dynamiques optimisées (Kotlin), boss MythicMobs gérés à part. | NamespacedKey `hordespawner` préservé ; MythicMobs/ModelEngine soft. |
| **radiation** | ex-RadiationCore | Dose/protection/signature/anomalies STALKER (Kotlin), loot gating. | `RadiationAPI` ; PAPI `%radiationcore_*%` ; ProtocolLib soft. |
| **streamcode** | ex-StreamCode | Codes streamer / parrainage, milestones, anti-spam. | 8 tables `stream_*` ; PAPI 40+. |
| **lootregion** | ex-LootRegion | Loot régional (variants, pity, heat, streaks, chains, BlockLoot). | Packages `fr.vrombi.lootregion.api.*` ; 12 events Bukkit ; PAPI 28. |
| **regionstats** | ex-RegionStats | Stats par région (temps, kills Mythic/PvP/vanilla, streaks, morts) + DropAPI natif (remplace MythicMobs drops). | Packages `fr.masurvie.regionStats.api.*` ; PAPI 80-90+. |
| **territory** | ex-TerritoryManager | Capture de zones WG par faction, taxe, classements. | Packages `fr.vrombi.territorymanager.api.*` ; PAPI 20+ ; SSB2/MMOItems réflexion. |
| **convoy** | ex-InfectedConvoy | Convois de survivants (taille 4, RTP vote, pool Vault same-region). | Packages `fr.vrombi.infectedconvoy.api.*` ; 8 events ; PAPI 8. |
| **eventhub** | ex-MaSurvieEventHub | **Hub d'events unifié** : 1 listener par source → N sinks ; 74 HubEventType. | Sinks ClueScrolls + EpicAchievements + PAPI `%eventhub_counter_*%`. |
| **webhookreport** | ex-WebhookReport | 4 systèmes Discord : `/reportplayer`, `/reportbug`, `/avis`, `/suggestion`. | SQLite `webhook_*` ; module bot `webhookbridge` :25591. |
| **airdrop** | ex-plugin LukeMccon/Airdrop | Airdrops (drop/packages), orchestrateur par région. | `/airdrop`, `/package`, `/packages`. |
| **trophydisplay** | ex-TrophyDisplay | Trophées soulbound (item ItemsAdder, hologramme FancyHolograms, GUI). | Pool partagé ; MySQL counter `SELECT FOR UPDATE` ; PAPI 8. |
| **graves** | natif (T1) | Tombes à la mort (hologramme, téléportation). | MythicMobsBridge multi-version. |
| **pricing** | natif (T3) | `PriceService` HDV : moyenne+médiane des prix (`/ah-price`). | Lit l'HDV (zAuctionHouse/AxTrade). |
| **blockinteractguard** | natif (E) | Restrictions d'ouverture de blocs/coffres. | Perm `survivorcore.blockguard.bypass`. |
| **trade** | ex-AxTrade | `/trade` natif (framework Lamp), 45 fichiers + 16 shims AxAPI. | Hooks Vault + XP gardés. |

> **Comment ça communique** : le pattern central est **MaSurvieEventHub**. Chaque
> source (lootregion, regionstats, territory, convoy, forge, radiation, survivorcore…)
> émet un `HubEvent` normalisé ; le hub redistribue à des **sinks** (ClueScrolls pour
> les ClueScrolls, EpicAchievements en mode PLACEHOLDER, compteurs SQLite exposés en
> `%eventhub_counter_<TYPE>%`, et MaSurvieSSB2Missions qui lit les compteurs). **Aucune
> dépendance dure** entre sources et consommateurs : tout est softdepend / réflexion.

---

## (b) Plugins & services maison autonomes (hors SurvivorCore)

| Élément | Type | Rôle | Intégration |
|---|---|---|---|
| **EpicAchievements** (`EpicAchievements-2.10.0-multitask.jar`) | jar plugin (fork modifié) | Système de succès maison étendu : carnet (302 succès / 19 zones), `/quetes` GUI 3 niveaux, multi-tâches ALL/ANY/COUNT, condition `zone_tier`, RTP carto, récompenses Vault, **0 table SQL** (tout dérivé de placeholders). Catégorie `effondrement` (~150 succès). | Lit `%eventhub_counter_*%` et PAPI de tous les modules ; intègre ClueScrolls + ZoneSpawn. Configs dans `deploy/configs/EpicAchievements/`. |
| **MaSurvieSSB2Missions** (`MaSurvieSSB2Missions.jar`) | jar mission SSB2 | Bridge SuperiorSkyblock2 ↔ EventHub (+ ClueScrolls) : missions LOOT/STAT/CLUE + composite AND/OR. | Va dans `plugins/SuperiorSkyblock2/missions/` ; lit le hub (1 listener). |
| **Bot Discord MaSurvie** (`bot/`) | service Node.js | Bot custom (discord.js 14, Node 20+, MySQL partagée). Modules : `tickets`, `discordlink`, `reactiontracker`, `narrator`, `internal`, `autoreact`, `embeds`, `autorole`, `mcAnnounce`, `welcome`, `webhookbridge`. | Parle au plugin via HTTP/WS (CoreHttpService :25580, BotReceiver :25591). |
| **ReactionTracker** | module bot + expansion PAPI | Points par réactions emoji Discord → `%reacttracker_*%` in-game ; 3 classements, anti-abus, paliers. | Côté bot `reactiontracker.json` ; MySQL partagée. |
| **SurvivorPanel** (`survivor-panel/`) | service web Node.js | Panneau d'admin web (Fastify, Discord OAuth + 2FA TOTP obligatoire, RBAC, audit). Pilote bot + plugin (Pterodactyl en Phase 2). | MySQL partagée ; ne modifie jamais le code du bot. Owner Discord codé en dur. |

> **MaSurvieOutil n'existe plus.** L'ancien plugin/catalogue d'items a été remplacé par le module **`itemmodule`** de SurvivorCore (voir catégorie (a) et le fichier 03). Toute la logique d'items et de scrolls vit désormais dans `itemmodule`. Ne plus référencer « MaSurvieOutil » comme un composant actif.

---

## (c) Plugins tiers / premium (décrits, jamais leur code)

Installés côté serveur, consommés par les modules maison en hard/soft depend ou par
réflexion. **Aucun code de ces plugins n'est reproduit ici** ; pour les premiums
forkés, le dépôt ne contient que des shims / des descriptions.

| Plugin | Type | Rôle sur MaSurvie | Lien d'intégration |
|---|---|---|---|
| **SuperiorSkyblock2 (SSB2)** | tiers | Moteur Skyblock : îles, équipes/factions, missions. | Hard pour SSB2Missions ; soft pour tickets/territory/regionstats. |
| **WorldGuard** | tiers | Régions = les zones tier 0→5, territoires, gating. | Hard pour lootregion/regionstats/territory/radiation. |
| **WorldEdit / FastAsyncWorldEdit** | tiers | Sélections/cuboids (zonespawn, extractions). | Soft (transitif WG + zonespawn). |
| **PlaceholderAPI** | tiers | Tous les `%...%` du serveur. | **Hard depend** de SurvivorCore. |
| **Vault** | tiers | Économie (argent, primes, taxe, pool convoi). | **Hard depend** de SurvivorCore. |
| **MythicMobs** | tiers | Mobs & mutants & boss. | Soft : horde (variantes), radiation (`radiation_mob`), regionstats (kills). |
| **ModelEngine** | tiers | Modèles 3D des mobs custom. | Soft : horde (validation visuelle best-effort). |
| **ItemsAdder** | tiers | Items custom + durabilité native (`CustomStack`). | Soft : forge (durabilité), trophydisplay (item soulbound). |
| **FancyHolograms** | tiers | Hologrammes (trophées posés, tombes). | Réflexion pure (trophydisplay, graves). |
| **ClueScrolls** | tiers | Énigmes/clues. | Sink `CluescrollsSink` du hub (remplace l'ex-InfectedSkyblockClues). |
| **LuckPerms** | tiers | Permissions / groupes / perks. | Soft : discordlink (groupes/perks). |
| **Lands** | tiers | Régions de claim (alternative). | Soft : discordlink (best-effort). |
| **LiteBans** | tiers | Bannissements. | Soft : tickets (statut ban). |
| **CoinsEngine** | tiers | Monnaie alternative. | Soft : discordlink (rewards). |
| **Plan** | tiers | Analytics joueurs. | Soft : 10+ modules exposent un `DataExtension`. |
| **ProtocolLib** | tiers | Paquets bas niveau. | Soft : radiation (particules best-effort). |
| **BlueMap** | tiers | Carte web 2D/3D du serveur. | Cible des forks BlueBridgeCore/WG. |
| **zAuctionHouse** | premium (fork local présent) | Hôtel des ventes (HDV). | Module `pricing` lit ses prix ; **code premium non reproduit**. |
| **AxTrade / AxAPI** | premium | Échange joueur-à-joueur (origine du module `trade`). | Module `trade` natif réimplémente `/trade` via shims AxAPI ; **code premium non reproduit**. |
| **eco / libreforge / EcoItems / EcoScrolls** (Auxilor) | tiers (stack historique) | Ancien moteur d'items/enchants. **Retiré du gameplay** (remplacé par le module **ItemModule** de SurvivorCore) ; jars conservés dans `server-patch/` pour historique. | N'est plus actif. |
| **DiscordSRV** | tiers (legacy) | Ancien pont Discord. | Remplacé par le module `discordlink` (s'enregistre comme service si présent). |

---

## Sources lues pour ce fichier

- `deploy/` (liste des jars de prod) + `deploy/README.md`
- `wiki/README.md` (cartes plugins + changelog d'absorptions)
- `masurvie_mini_core/src/main/resources/plugin.yml` (modules, commandes, permissions, softdepend)
- `bot/src/modules/*` (modules du bot Node)
- `server-patch/README.txt` (statut stack eco)
- mémoire projet (absorptions successives : trade/AxTrade, lootregion/regionstats, territory/convoy, etc.)
