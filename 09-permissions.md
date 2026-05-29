# 09 — Permissions

> Référence des permissions des plugins MaSurvie. Utile quand tu génères des configs de
> grades/rangs (LuckPerms), des kits, ou que tu décris quelles features sont gardées.
> Page sœur de [03-architecture-survivorcore.md](03-architecture-survivorcore.md) (placeholders & modules).

## Conventions

- **Défaut** : `op` (opérateurs uniquement), `true` (tout le monde), `false` (personne par défaut).
- Les `*.admin` regroupent souvent des sous-permissions (children).
- Préfixes hétérogènes hérités des plugins absorbés : `survivorcore.*`, `survivor.*`, `survie.*`, et préfixes d'origine conservés (`lootregion.*`, `regionstats.*`, `radiationcore.*`, `hordespawner.*`, `streamcode.*`, `territorymanager.*`, `infectedconvoy.*`, `eventhub.*`, `webhook.*`, `trophe.*`, `graves.*`, `airdrop.*`, `pricing.*`).

## Core
| Nœud | Rôle | Défaut |
|---|---|---|
| `survivorcore.admin` | Toutes les commandes `/sc` (reload, status, slot, schedule, blockguard, **afk**). | op |
| `survivorcore.admin.reload` / `.status` / `.slot` / `.schedule` | Sous-actions admin. | op |

## Joueur & staff
| Nœud | Rôle | Défaut |
|---|---|---|
| `survivor.tickets.use` | GUI `/tickets` perso. | true |
| `survivor.tickets.staff.view` / `.claim` / `.close` / `.stats` / `.profile` | Outils staff tickets. | op |
| `survivorcore.forge.use` | Forge sur soi (`/forgeron open`). | true |
| `survivorcore.forge.others` / `.admin` | Forge d'autrui / reload. | op |
| `survivorcore.itemmodule.admin` | Admin ItemModule (give, durability, reload). | op |
| `trophe.use` / `trophe.view` / `trophe.tp` | Trophées : accès / voir autrui / TP. | true |
| `trophe.admin` / `trophe.modo` | Admin / modérateur trophées. | op |
| `graves.allow` / `graves.teleport` | Créer une tombe / s'y TP. | true |
| `graves.admin` | Admin tombes. | op |
| `survivorcore.blockguard.bypass` | Outrepasser les restrictions d'ouverture de blocs. | op |

## Gameplay & zones
| Nœud | Rôle | Défaut |
|---|---|---|
| `lootregion.use` | Commandes LootRegion de base. | true |
| `lootregion.admin` | Admin (children : stats.view.others, stats.reset, stats.wipe, block.admin). | op |
| `lootregion.legendary` | Recevoir les drops légendaires. | op |
| `lootregion.stats.view.self` | Voir ses stats BlockLoot. | true |
| `regionstats.use` | Commandes joueur RegionStats. | true |
| `regionstats.admin` (+ `.drops`) | Admin RegionStats. | op |
| `radiationcore.use` / `.stats.self` / `.geiger.advanced` / `.particles` | Radiation joueur. | true |
| `radiationcore.stats.others` / `.admin` | Stats d'autrui / admin. | op |
| `radiationcore.bypass` | Immunité radiation. | false |
| `hordespawner.bypass` | Pas de horde autour du joueur. | false |
| `hordespawner.admin` / `.stats` | Admin / stats horde. | op |
| `territorymanager.use` / `.admin` | Territoires joueur / admin. | true / op |
| `infectedconvoy.create` / `.use` | Créer / utiliser un convoi. | true |
| `infectedconvoy.admin` | Admin convois. | op |
| `survie.zonespawn.admin` | Admin ZoneSpawn. | op |
| `airdrop.use` / `airdrop.admin` | Airdrops joueur / admin. | true / op |
| `pricing.use` | `/ah-price`. | true |
| `eventhub.admin` | Admin EventHub. | op |

## Discord & reporting
| Nœud | Rôle | Défaut |
|---|---|---|
| `survivorcore.link` | `/link` `/unlink` sur soi. | true |
| `survivorcore.link.admin` / `survivorcore.discordlink.admin` | Forcer link / admin perks. | op |
| `streamcode.use` | `/code`, `/code info`, `/code top`. | true |
| `streamcode.stats` / `.mod` / `.admin` | Streamers / mod / admin. | op |
| `survivorcore.narrator.*` / `.flavortext.*` / `.aibudget.*` | Modules IA (use/admin/stats). | op |
| `webhook.report.player` / `.report.bug` / `webhook.avis` / `webhook.suggestion` | Reports & avis joueur. | true |
| `webhook.admin` / `webhook.staff.*` / `webhook.validate` / `webhook.publish` / `webhook.history` / `webhook.search` / `webhook.blacklist` / `webhook.bypass.cooldown` | Outils staff WebhookReport. | op |

## Plugins maison annexes
| Nœud | Rôle | Défaut |
|---|---|---|
| `achievements.admin` / `achievements.tracker` | EpicAchievements admin / tracker. | op |
| `masurvie.quetes.use` | Carnet d'exploration (`/quetes`). | true |
| `masurvie.quetes.admin` | Admin carnet. | op |
| `masurvie.quetes.teleport.bypass-cartography` / `-cooldown` / `-tier` | Contournements RTP. | op |
| `axtrade.trade` / `axtrade.toggle` | Échange / toggle sur soi. | true |
| `axtrade.admin` / `axtrade.toggle.other` | Admin / toggle d'autrui. | op |

---

_~103 nœuds, dérivés des `plugin.yml` et des `hasPermission()` du code. 2026-05-29._
