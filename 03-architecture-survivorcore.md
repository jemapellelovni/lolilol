# Architecture SurvivorCore — le socle modulaire de MaSurvie V2

> Document de contexte pour une IA assistante. Décrit l'architecture réelle du plugin
> `SurvivorCore` (package `fr.masurvie.survivorcore`), son pattern modulaire, chacun de
> ses ~30 modules, le bus d'événements **MaSurvieEventHub**, et la convention complète des
> placeholders PlaceholderAPI. Toutes les valeurs ci-dessous reflètent le code source réel
> (`masurvie_mini_core/`) et le wiki, qui fait foi.

---

## 1. Vue d'ensemble

SurvivorCore est un **plugin Paper / Purpur 1.21, Java 21**, conçu comme un *socle unique* qui a
progressivement **absorbé une vingtaine de plugins autrefois standalone** (LootRegion, RegionStats,
TerritoryManager, InfectedConvoy, HordeSpawner, RadiationCore, StreamCode, EventHub, AxGraves,
AxTrade, etc.). Chacun est devenu un **module** interne plutôt qu'un jar séparé.

Avantages de cette absorption :

- **Une seule connexion BDD partagée** (pool HikariCP `CoreDataSource`) au lieu d'une par plugin.
- **Un seul serveur HTTP** mutualisé (`CoreHttpService`, port `25580`, auth Bearer).
- **Un seul fichier de langue** central `lang_fr.yml` (≈ 70 KB, sections namespacées par module).
- **Un seul hook PlaceholderAPI** (`SurvivorCorePlaceholders`) — bien que chaque module enregistre
  souvent sa propre *expansion* avec son préfixe (`%lootregion_*%`, `%radiationcore_*%`, etc.).
- **Couplage inter-modules sans dépendance dure** via le bus EventHub.

Version actuelle : `SurvivorCore-2.0.0.jar` (30 modules au total au 2026-05-29).

---

## 2. Le pattern modulaire

### 2.1 L'interface `Module`

Tout module implémente `fr.masurvie.survivorcore.modules.Module` :

```java
public interface Module {
    String getName();        // nom lisible (logs, /sc status)
    String getConfigKey();   // clé dans config.yml sous "modules.<key>"
    void enable();           // enregistre listeners, tâches, hooks
    void disable();          // annule proprement listeners et tâches
    void reload();           // recharge la config sans full restart
    boolean isEnabled();
}
```

Le contrat est volontairement minimal : chaque module est **indépendant** et peut être activé /
désactivé via `config.yml` sous `modules.<key>: true|false`.

### 2.2 Le bootstrap `SurvivorCore.java`

La classe principale (`extends JavaPlugin`, singleton via `SurvivorCore.get()`) orchestre tout
dans `onEnable()`, dans cet ordre précis :

1. **Logging** — force le niveau de `slf4j-simple` (shadé/relocalisé) par propriétés *système*
   (`System.setProperty`), car sur Paper le Thread-Context-ClassLoader ne lit pas
   `simplelogger.properties`. Ces deux lignes doivent rester les toutes premières.
2. `saveDefaultConfig()` + extraction des ressources manquantes : `timeslots.yml`, `schedules.yml`,
   `lang_fr.yml`, `zones.yml`, `zonespawn.yml` (via `saveResourceIfMissing`, jamais d'écrasement).
3. `DebugLogger.init(this)` — logger de debug central, piloté par `debug.modules.<module>` dans
   `config.yml` (zéro overhead si désactivé, test précoce avant toute concaténation de string).
4. `LangManager` — charge `lang_fr.yml` (doit précéder ZonesRegistry et les modules).
5. `ZonesRegistry` — référentiel central des **21 zones WG** (display / tier / theme / lore
   partagés), lu depuis `zones.yml`.
6. **Hook Vault** (hard depend) — si absent, le plugin se désactive.
7. **Hook MythicMobs** (soft) — simple flag `mythicMobsHooked`.
8. `loadBddConfig()` — provisionne et charge **`bdd.yml`** (voir §2.4).
9. `DatabaseManager` — base SQLite historique du core.
10. `CoreDataSource` — pool **HikariCP mutualisé** des modules absorbés
    (`SurvivorCore.get().getSharedDataSource().getConnection()`), convention de table
    `survivor_<module>_*`.
11. `CoreHttpService` — créé avant les modules (tickets s'y enregistre à l'enable).
12. **Enregistrement des modules** : `modules.add(new XxxModule(this))` pour chacun (~30).
13. **Activation conditionnelle** : pour chaque module, si `isModuleEnabled(getConfigKey())`,
    appelle `module.enable()` dans un try/catch (une erreur d'un module n'abat pas le serveur).
14. `httpService.start()` — n'ouvre le port que si au moins un module a enregistré des routes.
15. **Hook PlaceholderAPI** (hard depend) — `SurvivorCorePlaceholders.register()`.
16. **Hook Plan** (soft, Player Analytics) — `PlanHook.register` en réflexion FQN.
17. Enregistre la commande principale `/survivorcore` (alias `/sc`, `/survcore`).

`onDisable()` parcourt les modules actifs et appelle `disable()`, puis ferme placeholders, HTTP,
pool partagé et DB.

`reloadAll()` (commande `/sc reload`) recharge config + lang + zonesRegistry, puis pour chaque
module : `reload()` s'il est actif, ou `enable()` s'il vient d'être activé en config.

### 2.3 Accès typé aux modules

`getModule(Class<T>)` retourne l'instance typée d'un module (ou null). C'est ainsi que les modules
se trouvent entre eux *sans import circulaire dur*.

### 2.4 Configuration : `config.yml`, `bdd.yml`, et les YAML par module

- **`config.yml`** : régénéré à chaque mise à jour. Contient la section `modules:` (toggles),
  `settings:` (timezone `Europe/Paris`, langue, log-level), et la config des modules « historiques »
  (prime-tracker, time-slot-manager, spawn-control, schedule-manager, death-messages,
  regionambiance, etc.).
- **`bdd.yml`** : fichier **jamais écrasé** lors des MAJ (clés racine `type`, `table_prefix`,
  `mysql.*`, `sqlite.*`). À la première création, `loadBddConfig()` migre automatiquement l'ancien
  bloc `storage:` de `config.yml` vers `bdd.yml`. **C'est le fichier de secrets** — il n'est pas
  reproduit ici. Ne plus mettre de section `storage:` dans `config.yml`.
- **YAML par module** : les gros modules ont leur propre sous-dossier dans
  `plugins/SurvivorCore/` (ex. `horde/`, `radiation/`, `lootregion/`, `regionstats/`, `territory/`,
  `streamcode/`, `eventhub/`, `webhookreport/`, `trophydisplay/`, `graves/`, `airdrop/`), pour
  éviter d'alourdir le `config.yml` (la config LootRegion fait ~98 KB).

### 2.5 Storage

Dual-driver **MySQL / SQLite**. Un proxy JDK traduit au vol le SQL « SQLite » (`ON CONFLICT`,
`AUTOINCREMENT`, `TEXT`, `REAL`) en équivalents MySQL — pas de duplication de code. Convention de
temps du repo : colonnes `created_at` en **`BIGINT` epoch-ms** (`System.currentTimeMillis()`), jamais
de `TIMESTAMP` SQL.

---

## 3. Les modules, un par un

> Numérotation = ordre d'enregistrement dans `onEnable()`. Clé config = clé sous `modules.`.

### Modules « cœur » historiques (1–5)

**1. PrimeTracker** (`prime-tracker`)
- Lit le solde **Vault** (read-only) et calcule un *tier* selon des bornes → bonus % + label coloré.
- Cache rafraîchi toutes les 30 s ; annonce du top économie toutes les 30 min (snapshot en DB).
- Le top ne retient que les joueurs **en ligne** (`online-only: true`).
- Table : `prime_history`.
- Placeholders : `%survivorcore_balance%`, `_bonus_percent%`, `_bonus_multiplier%`, `_tier%`,
  `_tier_label%`, `_tier_next%`, `_tier_next_missing%`, `_rank%`, `_top_<pos>%` (+ `_amount/_tier/_tier_label`),
  `_prime_top1_name/_amount%`, `_players_in_survival%`.

**2. TimeSlotManager** (`time-slot-manager`)
- Créneaux horaires globaux (`timeslots.yml`) qui activent des multiplicateurs (spawn, drop).
- Plages `HH:mm` avec wrap minuit, filtrage `days`, `playercount-threshold`, priorités, annonces.
- Placeholders : `%survivorcore_slot_active%`, `_label%`, `_spawn_multiplier%`, `_drop_multiplier%`,
  `_remaining%`, `_duration%`, `_next%`, `_next_start%`, `_playercount%`, `_threshold_met%`,
  `_none%`, `_count_today%`, plus `%survivorcore_is_weekend%`, `_current_time_fr%`.

**3. SpawnControl** (`spawn-control`)
- Bloque les spawns vanilla naturels dans des mondes listés ; option `allow-mythicmobs`.
- Compteur quotidien en DB : `spawn_blocked_count` (PK `day`).
- Placeholders : `%survivorcore_spawns_blocked_today%`, `_spawn_control_active%`.

**4. ScheduleManager** (`schedule-manager`)
- Planifie des commandes console selon 5 types : `FIXED`, `RANDOM_IN_RANGE`, `RANDOM_INTERVAL`,
  `RANDOM_WEEKDAY`, `RANDOM_CHANCE_INTERVAL` (`schedules.yml`). `max-per-day` cap quotidien.
- Table : `schedule_history`.
- Placeholders : `%survivorcore_schedule_next%`, `_next_time%`, `_last%`, `_last_at%`, `_count_today%`.

**5. DeathMessages** (`death-messages`)
- Messages de mort custom par cause (PvP / mob / environnement), killstreaks à seuils.
- Table : `death_history`.
- Placeholders : `%survivorcore_deaths_total%`, `_deaths_by_pvp%`, `_deaths_by_mob_<id>%`.

### Modules services / IA (6–11)

**6. Tickets** (`tickets`) — ex-plugin SurvivorTickets absorbé. GUIs staff/joueur, catégories,
système de tickets, route HTTP `TicketsHttpRoutes` sur le CoreHttpService, hook DiscordSrv.
Publie vers EventHub via `EventHubReportBridge` (cf. WebhookReport). Cache `TicketCache`.

**7. DiscordLink** (`discordlink`) — remplace **DiscordSRV**. Liaison Discord ⇄ MC maison :
- Code à **6 chiffres** généré in-game (`/link`), vérifié côté bot Node via `POST /api/link/verify`.
- API **REST + WebSocket** sur le CoreHttpService (`:25580`, Bearer `api-shared.token`).
- Moteur de récompenses anti-abus (`RewardsManager`, `RewardQueueService`), perks continus,
  suivi des régions WG (`RegionMovementListener`, rôles de région live + persistants), synchro d'état
  Discord (boost, server-tag).
- Hooks `LuckPermsHook`, `WorldGuardHook`. Config sous `discordlink:` dans `config.yml`.
- Embarque l'expansion **ReactTracker** : `%reacttracker_is_linked%`, `_discord_id%`,
  `_total_reactions_tracked%`, `_active_channels_count%`, `_total_points_distributed_<period>%`,
  `_top_<…>%`.
- Tables : `discord_*` / liaison + stats de réactions (`ReactionStatsDao`).

**8. AiBudget** (`aibudget`) — garde-fou central des dépenses IA. **Doit s'enable AVANT Narrator /
Flavortext**.
- Comptabilise chaque appel Claude (1 ligne dans la table **`ai_costs`**), calcule le coût USD→EUR,
  applique une limite mensuelle (10 € défaut) avec paliers **50 % (log) / 75 % (warn+webhook) /
  90 % (throttle, coupe Flavortext) / 100 % (stop, coupe Narrator+Flavortext)**.
- Reset mensuel auto. Endpoint `POST /ai-cost` (le bot Narrator y reporte son coût).
- Méthode pivot : `aiAllowed("<source>")` interrogée par les modules IA avant chaque dépense.

**9. Narrator** (`narrator`) — narrateur IA d'événements (embed Discord ± broadcast in-game).
- **Zéro listener**, 100 % command-driven : `/svnarrate [--broadcast] <texte PAPI>`,
  `/svnarrate-bilan` (console, 22 h via schedule FIXED), `/svnarrate-reload`.
- Résout les placeholders PAPI puis `POST <bot-url>/narrate` (Bearer). Le **bot Node** appelle Claude
  (Haiku ponctuel / Sonnet bilan), 6 personas pondérés, mémoire 24 h, embed coloré par l'IA.
- Seul levier créatif = `narrator.yml` (sections `lore` / `style` / `color-guidance` / `personas`),
  **partagé avec Flavortext**. Prompt caching sur le bloc système stable.

**10. Flavortext** (`flavortext`) — reformule un message **pour l'afficher en jeu** (vs Narrator qui
narre sur Discord). Appel **direct à Claude** depuis Java (`java.net.http`, `anthropic-beta`).
- `/svflavor <cible> <mode> <texte PAPI>` — cibles `<player>` / `@all` / `@console` ; modes
  `chat` / `actionbar` / `title` / `subtitle` / `bossbar`.
- Fallback robuste : toute défaillance (timeout, budget épuisé, réponse vide) → affiche le texte brut
  résolu, jamais d'erreur joueur. Coût reporté in-process à AiBudget (`aiAllowed("flavortext")`).

**11. BlockInteractGuard** (`blockinteractguard`, **opt-in `false` par défaut**) — empêche
l'ouverture (clic-droit) de certains blocs de stockage dans une liste de mondes (forcer les coffres
vanilla en Skyblock). Pas de DB/API/PAPI. Bypass : `survivorcore.blockguard.bypass`.

### Modules de jeu absorbés (12–22)

**12. ZoneSpawn** (`zonespawn`) — points d'extraction *cuboid* (WorldEdit softdep). Stats en DB
(`zonespawn_stats`). Émet `ZoneSpawnSuccessEvent` (consommé par EventHub pour les events `EXTRACT_*`).
Placeholders `%zonespawn_<stat>%` + suffixe `_z_<zone>` (longest-match), `_global_*`, `_top_*`, `_rank_*`.

**13. ItemModule** (`itemmodule`) — moteur générique d'items à capacités, piloté par YAML
(`itemmodule.yml`). Item = `Material` + abilities (trigger + effets). Clé PDC
`survivorcore:itemmodule_id`. Inclut boutons d'inventaire + **poche sécurisée** (4 slots, table
`pocket_contents`, items sauvés à la mort, déverrouillage WG+potions). Softdep RadiationCore /
LootRegion / WorldGuard / LuckPerms. Commande `/itemmodule`, perm `survivorcore.itemmodule.admin`.

**14. RegionAmbiance** (`regionambiance`) — force, **côté client seulement**, météo claire + heure
verrouillée pour les joueurs dans une liste de régions WG. Tâche périodique (pas de PlayerMoveEvent),
reset à la sortie. WG softdep. Pas de DB/PAPI.

**15. Forge** (`forge`) — forgeron de réparation 2-slots (refonte de MaSurvie_Forgeron). Dispatch vers
backends ItemsAdder (réflexion) et ItemModule (API publique). Émet `FORGE_*` (via EventHub).

**16. Horde** (`horde`) — horde dynamique 50-100 joueurs (ex-HordeSpawner Kotlin). Refuse de s'enable
sans MythicMobs. Configs dans `horde/`. NamespacedKey `hordespawner` préservé pour les boss PDC.
Placeholders : `%horde_active_total%`, `_pool_usage_percent%`, `_tps_state%`, `_avg_eval_ms%`,
`_active_bosses%`, `_cluster_count%`, `_mobs_around_me%`, `_my_quota%`, `_cluster_size%`.

**17. Radiation** (`radiation`) — radioactivité MaSurvie (ex-RadiationCore Kotlin). WG softdep.
Configs dans `radiation/`. Préfixe placeholder `%radiationcore_*%` (≈ 50 placeholders : `_dose%`,
`_dose_percent/bar/color/status%`, `_zone_*%`, `_protection_*%`, `_filter_*%`, `_suit_integrity%`,
`_exoskeleton_*%`, `_signature_*%`, `_accessible_tier%`, `_can_loot_<tier>%`, `_mutations_*%`,
`_has_mutation_<id>%`, `_target_<player>_<topic>%`, etc.).

**18. StreamCode** (`streamcode`) — codes streamer + parrainage 1→N + récompenses + classement.
Implémente lui-même `Module` (conserve `getInstance()` / API legacy). Tables `stream_*` (8 tables).
Commande `/code`, perms `streamcode.*`. Placeholders `%streamcode_total_codes_active%`,
`_total_referrals_today%`, `_top_<…>%`.

**19. LootRegion** (`lootregion`) — moteur de loot par région WG (ex-LootRegion). 8 features
(Pity / Heat / Streak / Trap / Pool / Chain / MythicMobs / TeamBonus / Webhook) + BlockLoot (blocs
farmables, table `lr_blockloot_stats`). Pas de coordonnées stockées (containers = région WG +
Material, scan tile-entities). Émet 14 events `LR_*`. Package API préservé `fr.vrombi.lootregion.api.*`.
Placeholders `%lootregion_chests_opened%` (+ `_<region>`, `_variant_<v>`), `_loot_streak*%`, `_heat*%`,
`_pity_*%`, `_blockloot_total_<type>%`, `_reward/loot/record/session_<key>%`, `_top_chests_<N>%`,
`_zone_total_chests_<region>%`, `_container_count_<region>%`.

**20. RegionStats** (`regionstats`) — stats par région WG (ex-RegionStats). Temps de présence, kills
(PvP / mythic / vanilla), drops, rewards, records, équipes. Émet 24 events `RS_*`. Package préservé
`fr.masurvie.regionStats.api.*`. ≈ 90 placeholders `%regionstats_*%` (`_zone_tag%`, `_time_*%`,
`_kills_*%`, `_mythic_*%`, `_playerkills_*%`, `_vanilla_*%`, `_streak_*%`, `_zone_*%`, `_team_*%`,
`_reward_*%`, `_rewards_*%`, `_top_<…>%`) + un sous-système **Drop** `%regionstats_drop_*%`
(`_pity_*%`, `_streak_*%`, `_count_<region>%`, `_money_*%`, `_legendary_count%`, `_last_legendary%`).

**21. Territory** (`territory`) — capture de zones WG via un Trophée de Capture (ex-TerritoryManager).
Émet 7 events `TM_*`. Package préservé `fr.vrombi.territorymanager.api.*`. Placeholders
`%territory_owner_<zone>%` (+ `_id`), `_owned_since_<zone>%`, `_is_owned_<zone>%`,
`_trophy_location_<zone>%`, `_capture_*_<zone>%`, `_top_team_<N>%`, `_player_team_zones_count/list%`,
`_player_in_owned/enemy/own_zone%`, `_team_zones_owned/captures_total/trophies_destroyed_<team>%`.

**22. Convoy** (`convoy`) — convois persistants de survivants (ex-InfectedConvoy). Vault + WG requis.
Émet 8 events `CONVOY_*`. Consomme l'API LootRegion par event (pas de couplage dur). Commande
`/convoi`. Placeholders `%infectedconvoy_active%`, `_active_size%`, `_active_duration%`,
`_active_pool_received%`, `_active_mvp%`, `_count_member_of/leader_of%`, `_total_capsules_all%`.

### Bus + modules récents (23–30)

**23. EventHub** (`eventhub`) — le **bus d'events unifié** (voir §4 en détail). Aucune hard depend,
sources & sinks s'auto-activent. Compteurs SQLite dans `eventhub/`. Placeholders `%eventhub_*%`.

**24. WebhookReport** (`webhookreport`, package `fr.masurvie.webhookreport`) — signalements / bugs /
avis / suggestions. SQLite locale dans `webhookreport/`. Module bot custom `webhookbridge`. Publie
vers EventHub `REPORT_BUG_VALIDATED`, `REPORT_AVIS_SENT`, `REPORT_SUGGESTION_SENT`. Placeholders
`%webhook_reports_sent%`, `_bugs_sent[_<cat>]%`, `_avis_sent/avg_rating%`, `_suggestions_sent%`,
`_total_suggestions%`, `_cooldown_<type>%`, `_can_<type>%`, `_last_<TYPE>_at%`, `_notif_*%`,
`_blacklisted/blacklist_reason%`.

**25. TrophyDisplay** (`trophydisplay`, package `fr.masurvie.trophydisplay`) — trophées soulbound,
MySQL counter transactionnel (`FOR UPDATE`), FancyHolograms / ItemsAdder / SSB2 par réflexion. PDC
`trophydisplay:*`. Si MySQL injoignable, le module ne s'enable pas (le reste continue). Publie vers
EventHub `TROPHY_OBTAINED` / `TROPHY_PLACED` (+ dérivés). Placeholders `%trophe_total%`, `_posed%`,
`_stored%`, `_last[_date]%`, `_etat_<id>%`, `_rang_<id>%`, `_total_distribues_<id>%`.

**26. Graves** (`graves`) — tombes natives (ex-AxGraves, réécrit sans AxAPI). ArmorStand Bukkit +
FancyHolograms (réflexion) + storage YAML (`graves/graves.yml`). Bridges SuperiorSkyblock / MythicMobs
/ MythicLib. Commande `/grave`. Émet 5 events publics + 7+ HubEventType `TOMB_*` (`TOMB_CREATED_OWN_ISLAND`,
`TOMB_CREATED_WILD`, `TOMB_INFECTED_LOOTED`, `TOMB_EXPIRED`…). Revente HDV à l'expiration via le module
Pricing.

**27. Pricing** (`pricing`) — `PriceService` réutilisable + `/ah-price [Nd|N]` (alias `/price`,
`/ahprice`). Sonde **zAuctionHouse V4 par réflexion** (zéro compileOnly ; service en veille si absent).
Stats avg/median/count/min/max calculées en async. Placeholders `%zah_avg_price_inhand%`,
`_median_price_inhand%`, `_count_inhand%` (+ variantes listings), **cache TTL 60 s** (jamais de calcul
à la résolution).

**28a. Airdrop** (`com.airdropmc.Airdrop`) — airdrop natif absorbé (LukeMccon/Airdrop, MIT), package
préservé `com.airdropmc.*`. Configs dans `airdrop/`. S'enable avant l'orchestrateur.

**28b. AirdropModule** (`airdrop`) — couche d'auto-airdrop par région WG : `PlayerCountService`
(snapshot O(players), filtre vanish/AFK) + scheduler pondéré + tier + recheck. Émet `AIRDROP_*`.

**29. Afk** (`afk`) — détection + zone AFK natives. Source de vérité unique `IN_AFK_ZONE` (présence
dans la région WG). Anti-boucle TP, récompenses passives, leaderboards reset Europe/Paris, filtre
commandes. Branche `PlayerCountService` (Airdrop ignore les AFK). Placeholders `%survivorcore_afk%`,
`_afk_zone%`, `_afk_time_total/day/week/month%`, `_afk_top_<n>_name/time%`, `_afk_rank%` (tous O(1),
caches in-memory / async 60 s).

**30. Trade** (`trade`) — pont réflexif vers **AxTrade** (jar standalone officiel). Écoute
`AxTradeCompleteEvent` par réflexion. Publie `TRADE_COMPLETED` (2 joueurs) et `TRADE_IN_DANGER_ZONE`
si la région est listée.

---

## 4. Le bus MaSurvieEventHub

### 4.1 Rôle et principe

Avant le hub, plusieurs bridges faisaient le même travail mécanique : écouter LootRegion / RegionStats
/ … et relayer ailleurs (ClueScrolls, EpicAchievements, SSB2 missions). Le hub remplace ça par :

```
 sources (plugins/modules) ──(1 listener par source)──► MaSurvieEventHub
                                                         │ HubEvent(type, player?, region?, ctx)
                                                         │ compteurs SQLite (sinks stateless)
                                                         ▼
                  ┌──────────────────┬───────────────────┬────────────────────┐
            CluescrollsSink    EpicAchievementsSink   EventHubExpansion   (SSB2Missions
            (remplace ISC)     (TaskType PLACEHOLDER)  %eventhub_*% PAPI    lit getCounter)
```

Principes clés :

- **Un seul listener Bukkit par source**, en `EventPriority.MONITOR`, qui normalise l'event natif en
  un `HubEvent` immuable `(type, playerUuid?, region?, ctx)` et le dispatche en interne vers **N sinks**.
- **Sinks stateless** : chacun stocke sa propre progression (ClueScrolls côté CS, EA en SQLite/mongo).
  Le hub ne persiste que les **compteurs** servant `getCounter` / PAPI / SSB2.
- **Isolation** : une exception dans un sink est attrapée et loggée — les autres sinks et la source
  continuent. Aucun sink ne peut annuler un event source (il ne reçoit qu'un `HubEvent` immuable).
- **Aucune hard depend** : le hub démarre toujours ; sources/sinks absents sont ignorés (log au boot).
- **Perf** : aucune I/O dans le thread d'event ; compteurs en mémoire + flush async (60 s).

### 4.2 Comment les modules communiquent SANS couplage dur

Deux patterns coexistent :

1. **Plugins/modules avec API Bukkit** (LootRegion, RegionStats, Territory, Convoy, Forge…) : le hub a
   un *HubSource* dédié qui écoute leurs events natifs.
2. **Publishers in-process** (`EffondrementHubBridges`) : pour les modules SurvivorCore sans event
   Bukkit dédié (Graves, Pocket d'ItemModule, TrophyDisplay, PrimeTracker bounty…). Pattern :

```java
EffondrementHubBridges bridge = EffondrementHubBridges.get();
if (bridge != null) {
    bridge.publishPocketStored(player.getUniqueId(), itemId, "legendary");
}
```

3. **Plugins externes** (WebhookReport) : `EventHubReportBridge.publish("REPORT_BUG_VALIDATED", uuid, null)`
   par réflexion sur le `ServicesManager`.

API Java consommateur (artefact `fr.masurvie:eventhub-api`, pur Java sans dépendance) :

```java
MaSurvieEventHubAPI hub = MaSurvieEventHubProvider.get();   // null si absent
long n = hub.getCounter(uuid, HubEventType.LR_CHEST_OPENED, null); // null = toutes régions
hub.subscribe(HubEventType.RS_MYTHIC_KILL, ev -> { /* ev.ctxString("mob_id") */ });
```

### 4.3 Taxonomie `HubEventType`

Enum **stable et versionné** publié dans `eventhub-api`. Ajout = non-breaking, suppression = breaking.
Familles (préfixes) :

| Préfixe | Source | Exemples |
|---|---|---|
| `LR_*` (14) | LootRegion | `LR_CHEST_OPENED`, rare, legendary, money, item, trap, pity, chain, streak, heat, pool |
| `RS_*` (24) | RegionStats | enter/exit, kills, drops, rewards, records, team, trap, `RS_MYTHIC_KILL`… |
| `TM_*` (7) | Territory | claimed/lost/contested, trophy, capture, owner |
| `II_*` (8) | InfectedItems (historique) | used/broken/repaired, jammer, chest sealed, smuggler |
| `FORGE_*` (7) | Forge | instant/batch/ticket/completed/cancelled/open |
| `CONVOY_*` (8) | Convoy | created, member join/left/died, session, contribution |
| `DL_*` (6) | DiscordLink (WS, off par défaut v1) | events de liaison |

Plus les familles « Effondrement » (publishers in-process) :

| Préfixe / type | Catégorie |
|---|---|
| `BM_STEALTH_*` (USED/CHEST/KILL/EXTRACT) | BlueMap Stealth |
| `POCKET_*` (ITEM_STORED, FULL_STORED, LEGENDARY_STORED, SAVED_ON_DEATH, BUFF_ACTIVE_FULL, BUFF_EXTRACT) | Poche sécurisée |
| `TROPHY_*` (OBTAINED, PLACED, PLACED_ALL, RARE_OBTAINED, FIRST_ON_SERVER) | TrophyDisplay |
| `BOUNTY_*` (BALANCE_TICK, TIER_UP, TOP1_REACHED) | PrimeTracker |
| `REPORT_*` (BUG_VALIDATED, AVIS_SENT, SUGGESTION_SENT) | WebhookReport |
| `TIMESLOT_*` (CHEST_OPENED, KILL, EXTRACT) | TimeSlot |
| `EXTRACT_*` (SUCCESS, NO_DEATH, WITH_LEGENDARY, RICH, NO_KILL) | Extraction composite (ZoneSpawn) |
| `TOMB_*` (CREATED_OWN_ISLAND, CREATED_WILD, INFECTED_LOOTED, EXPIRED, …) | Graves |
| `AIRDROP_*` (SPAWNED…) | Airdrop |
| `TRADE_*` (COMPLETED, IN_DANGER_ZONE) | Trade |
| `META_ACHIEVEMENT_UNLOCKED` | Méta |

> Note : les events « zone / équipe » (capture, team) ont `playerUuid == null` et **ne sont pas
> comptés** par `getCounter`.

### 4.4 Les placeholders `%eventhub_counter_*%`

Exposés par `EventHubExpansion` (préfixe `eventhub`). Conventions exactes :

| Placeholder | Renvoie |
|---|---|
| `%eventhub_total%` | nombre de compteurs distincts trackés (diagnostic) |
| `%eventhub_counter_<HubEventType>%` | compteur du joueur pour ce type, **toutes régions** |
| `%eventhub_counter_<HubEventType>_<region>%` | compteur du joueur pour ce type **dans une région donnée** |

Le parseur cherche le **plus long préfixe de tokens** qui matche un `HubEventType` connu ; le reste est
interprété comme nom de région. Exemples :

```
%eventhub_counter_LR_CHEST_OPENED%
%eventhub_counter_RS_MYTHIC_KILL_spawn%
%eventhub_counter_BM_STEALTH_CHEST_port_touristique%
```

### 4.5 Sinks

- **CluescrollsSink** : reprend les 17 clues d'InfectedSkyblockClues (absorbé). Enregistre via
  `registerCustomClue(...)` avec le préfixe `infectedskyblockclues` → les identifiants de clue restent
  `infectedskyblockclues_<name>`, donc les configs ClueScrolls existantes fonctionnent sans changement.
- **EpicAchievementsSink** (mode `api`, défaut) : EA pilote la progression via son TaskType natif
  `PLACEHOLDER` qui sonde `%eventhub_counter_<TYPE>%` (toutes les 20 ticks) et appelle `TaskProgress.set`.
  **Zéro dépendance compile sur EA**. Mode `command` = push manuel `/eventhub progress`.
- **EventHubExpansion** : l'expansion PAPI ci-dessus.
- **SSB2Missions v1.1** : consomme le hub par réflexion (`getCounter`) — un seul listener alimente
  toutes les missions LOOT/STAT.

Commandes : `/eventhub status` / `reload` / `progress <joueur> <TYPE> [montant]` (perm `eventhub.admin`).

---

## 5. Récap — placeholders exposés par SurvivorCore et ses modules

| Préfixe | Module / source | Ce que renvoie (résumé) |
|---|---|---|
| `%survivorcore_*%` | PrimeTracker, TimeSlot, SpawnControl, Schedule, DeathMessages, **Afk** | Solde/tier/rang primes, créneau actif & multiplicateurs, spawns bloqués, prochaines tâches, morts, état/temps/leaderboard AFK |
| `%eventhub_*%` | EventHub | `_total%` (diag) + `_counter_<HubEventType>[_<region>]%` compteurs unifiés |
| `%lootregion_*%` | LootRegion | Coffres ouverts (par région/variant), streak, heat, pity, blockloot, records, sessions, tops |
| `%regionstats_*%` | RegionStats | Temps de présence, kills (pvp/mythic/vanilla), streaks, stats de zone & d'équipe, rewards |
| `%regionstats_drop_*%` | RegionStats (Drop) | Pity/streak/count de drops par région, money dropée, légendaires |
| `%radiationcore_*%` | Radiation | Dose, zone, protection, filtre/suit/exosquelette, signature, tiers de loot, mutations, médication |
| `%territory_*%` | Territory | Propriétaire de zone, capture en cours, tops d'équipe, zones possédées |
| `%infectedconvoy_*%` | Convoy | Convoi actif (taille, durée, pool, MVP), nombre de convois membre/leader, capsules |
| `%streamcode_*%` | StreamCode | Codes actifs, parrainages du jour, tops |
| `%reacttracker_*%` | DiscordLink | Lié ou non, Discord ID, réactions trackées, points distribués, tops |
| `%horde_*%` | Horde | Mobs actifs, usage du pool, mode TPS, boss, clusters, quota du joueur |
| `%zonespawn_*%` | ZoneSpawn | Stats d'extraction par zone (joueur/global/top/rang), suffixe `_z_<zone>` |
| `%trophe_*%` | TrophyDisplay | Trophées total/posés/stockés, dernier, état & rang par trophée |
| `%webhook_*%` | WebhookReport | Reports/bugs/avis/suggestions envoyés, cooldowns, notifs, blacklist |
| `%zah_*_inhand%` | Pricing | Prix moyen/médian/count HDV de l'item en main (cache 60 s) |

> Convention PAPI : tout retourne une **string** ; booléens = `"true"`/`"false"` ; valeurs
> absentes / joueur hors-ligne = `"0"`, `""` ou `"N/A"` selon le placeholder.
>
> **Règle de performance** (auditée T1→T6) : **aucun placeholder ne calcule à la résolution**. Tous
> les agrégats / leaderboards sont précalculés en tâche async périodique (Pricing 60 s, Afk 60 s,
> RegionStats tops warmed 30 s, PrimeTracker cache Vault 30 s) ; les compteurs EventHub sont des
> lookups HashMap O(1).

---

## 6. Points d'attention pour créer du contenu cohérent

- **Toujours passer par EventHub** pour faire réagir EA / ClueScrolls / SSB2 missions à un nouvel
  event — ne pas recréer un bridge. Ajouter un `HubEventType` (non-breaking) plutôt qu'un listener
  parallèle.
- **Lang** : tout message de module vit dans `lang_fr.yml` (section namespacée du module) ; point
  d'accès `LangManager.getModuleString(mod, key)`. Plus de `messages.yml` par module.
- **DB** : utiliser le pool partagé `getSharedDataSource()` (convention `survivor_<module>_*`), temps
  en epoch-ms BIGINT. Les secrets de connexion vivent dans **`bdd.yml`** (jamais commit, jamais écrasé).
- **Timezone** : `Europe/Paris` par défaut, pilote schedules + créneaux + resets leaderboards.
- **Hooks tiers en réflexion** quand c'est un plugin premium/externe (zAuctionHouse, AxTrade,
  FancyHolograms, ItemsAdder) — pas de `compileOnly`, dégradation gracieuse si absent.
