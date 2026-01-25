---

# Zabbix Hytale Server Template (Linux)

## Overview

Zabbix template for monitoring a **Hytale server on Linux** using **Zabbix Agent (active checks)** by parsing local Hytale JSON files.

Compatible with **Zabbix 7.0 and newer**.

---

## Requirements

* **Linux server** running the Hytale server
* Zabbix Server **7.0+**
* Zabbix Agent or Zabbix Agent 2 (**active checks enabled**)
* `acl` package (for `setfacl`)
---

## Monitored values

* Player discovery (per player JSON file)
* Player name
* Player UUID
* Player world
* Player position (X/Y/Z)
* Player game mode
* Player health (incl. armor)
* Player stamina
* Player stamina regeneration delay
* Player oxygen
* Player ammo
* Player immunity
* Player gliding state
* Player magic charges
* Player signature energy
* Player signature charges
* Active effect remaining duration
* Player inventory (items, quantity, durability)
* Discovered zones
* Respawn points (per world)
* Death markers (positions)
* Ban discovery
* Ban status
* Ban reason
* Ban type
* Ban issuer
* Ban timestamp
* Whitelist discovery
* Whitelisted players

---
---

## Template Macro

The Hytale base path is configured via:

```text
{$PLAYER_JSON_PATH}
```

Default value in the template:

```text
/home/hytale/
```

If your server directory is different, **adjust the macro** and also the ACL paths below.

---

## Low-Level Discovery (LLD)

### Player discovery

Source:

* `{$PLAYER_JSON_PATH}Server/universe/players/*.json`

LLD macros:

* `{#PATHNAME}` = full path to the player JSON file
* `{#UUID}` = player UUID (derived from filename)

Per-player item prototypes (examples):

* `player.name["{#PATHNAME}"]`
* `player.world["{#PATHNAME}"]`
* `player.position["{#PATHNAME}"]`
* `player.gamemode["{#PATHNAME}"]` *(Value map: GameMode)*
* `player.healthy["{#PATHNAME}"]`
* `player.stamina["{#PATHNAME}"]`
* `player.oxygen["{#PATHNAME}"]`
* `player.ammo["{#PATHNAME}"]`
* `player.immunity["{#PATHNAME}"]`
* `player.gliding["{#PATHNAME}"]`
* `player.magiccharges["{#PATHNAME}"]`
* `player.signatureenergy["{#PATHNAME}"]`
* `player.signaturecharges["{#PATHNAME}"]`
* `player.staminaregendelay["{#PATHNAME}"]`
* `player.remaining.duration["{#PATHNAME}"]` *(if active effects exist)*
* `player.items["{#PATHNAME}"]` *(inventory list)*
* `player.discoveredzones["{#PATHNAME}"]`
* `player.respawnpoints["{#PATHNAME}"]`
* `player.deathmarker["{#PATHNAME}"]`

### Ban discovery

Source:

* `{$PLAYER_JSON_PATH}Server/bans.json`

LLD macros:

* `{#UUID}` = banned player UUID (`target`)

Per-ban item prototypes (examples):

* `player.target["{#UUID}"]`
* `player.reason["{#UUID}"]`
* `player.by["{#UUID}"]`
* `player.type["{#UUID}"]`
* `player.timestamp["{#UUID}"]` *(converted to unixtime)*

### Whitelist discovery

Source:

* `{$PLAYER_JSON_PATH}Server/whitelist.json`

LLD macros:

* `{#UUID}` = whitelisted player UUID

Per-whitelist item prototypes:

* `player.whitelist["{#UUID}"]`

---

## Triggers

* **Player banned** (prototype)
* **Player in Creative mode** (prototype)
* **Player whitelisted** (prototype)

---

## Value Maps

* `GameMode`

  * `1` → Adventure
  * `2` → Exploration
  * `3` → Creative

---

## File permissions (ACL)

The Zabbix agent must be able to **traverse directories** and **read** the JSON files.

**Important:** The commands below use `/home/hytale/` as example.
If `{$PLAYER_JSON_PATH}` points somewhere else, **adjust the paths** accordingly.

### Directory traversal

```bash
setfacl -m u:zabbix:x /home/hytale
setfacl -m u:zabbix:x /home/hytale/Server
setfacl -m u:zabbix:x /home/hytale/Server/universe
```

### Player files (existing + new)

```bash
setfacl -R -m u:zabbix:rx /home/hytale/Server/universe/players
```

```bash
setfacl -R -d -m u:zabbix:rx /home/hytale/Server/universe/players
```

### Ban + whitelist + server files

```bash
setfacl -m u:zabbix:r /home/hytale/Server/bans.json
setfacl -m u:zabbix:r /home/hytale/Server/whitelist.json
setfacl -m u:zabbix:r /home/hytale/Server/config.json
```

---
