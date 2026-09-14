# RedeemMaster

<p align="center">
  <img src="https://raw.githubusercontent.com/Dndakrm/Redeem-Master/main/banner.png" alt="RedeemMaster Banner" width="100%" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Paper%20%2F%20Purpur%20%2F%20Spigot-00D1B2.svg" alt="Platform" />
  <img src="https://img.shields.io/badge/Minecraft-1.21%2B-blue.svg" alt="Version" />
  <img src="https://img.shields.io/badge/JDK-21-orange.svg" alt="JDK" />
  <img src="https://img.shields.io/badge/Dependencies-Zero-brightgreen.svg" alt="Dependencies" />
  <a href="https://discord.gg/VAVKTKNBYB"><img src="https://img.shields.io/badge/Discord-ZEFTY%20STUDIO-5865F2.svg" alt="Discord" /></a>
</p>

A dependency-free referral and milestone reward system built for Paper 1.21+ networks. Instead of hard-binding to specific economy, token, or crate APIs, RedeemMaster runs configurable console dispatches with variable placeholders (`<player>`, `<owner>`, `<count>`).

---

## Features

- **Dependency-Free Architecture:** Plug-and-play with any crate (ExcellentCrates, GoldenCrates), currency (EssentialsX, Vault), or token plugin (PlayerPoints) through native console command strings.
- **Dual Anti-Alt Protection:**
  - **IP Subnet Guard:** Blocks players on the same network or IP from claiming each other's codes.
  - **Playtime Verification Gate:** Requires newcomers to meet active playtime criteria before redeeming codes.
- **Milestone & Modulo Rewards:** Trigger linear rewards per invite (e.g., +1 Point) alongside modulo milestone cycles (e.g., Rare Crate Key every 5th invite) and static targets (e.g., 25 invites).
- **Fast Storage & In-Memory Cache:** Inverted UUID hash map lookups ensure $O(1)$ code resolution during runtime, backed up by flat YAML storage (`invite.yml`).
- **Geyser / Floodgate Friendly:** Supports Bedrock player prefixes (e.g. `.Username`) and offline UUID string matching without parsing exceptions.

---

## Installation

1. Download the latest compiled release from the **Releases** tab.
2. Drop `RedeemMaster-1.0.0.jar` into your server's `plugins/` directory.
3. Start or restart your server (Java 21+ required).
4. Configure reward commands inside `plugins/RedeemMaster/config.yml`.

---

## Commands & Permissions

Root command: `/redeemmaster`  
Registered aliases: `/redeem`, `/code`, `/ref`

### Player Commands (`redeemmaster.use`)
| Command | Description |
| :--- | :--- |
| `/redeem <code>` | Redeems an active referral code for starter bonuses. |
| `/redeem create [code]` | Generates a custom alphanumeric code (3–12 chars) or creates a random 6-character code. |

### Admin Commands (`redeemmaster.admin`)
| Command | Description |
| :--- | :--- |
| `/redeem info <player>` | Inspects player referral stats, creator IP, total invites, and pending counts. |
| `/redeem revoke <player>` | Revokes an active referral code. |
| `/redeem reactivate <player>` | Re-enables a revoked referral code. |
| `/redeem verify <player> [amount]` | Manually verifies pending referrals and fires milestone reward checks. |
| `/redeem reload` | Reloads `config.yml` and hot-refreshes `invite.yml` memory caches. |

---

## Configuration (`config.yml`)

```yaml
settings:
  # Anti-Alt and Abuse Prevention
  anti-alt:
    prevent-same-ip: true
    # Minimum active playtime in minutes required before a player can redeem a code (0 to disable)
    min-playtime-minutes: 15

  # Actions dispatched to the referee (the new player)
  referee-rewards:
    - "[console]eco give <player> 5000"
    - "[console]crates key give <player> starter 1"

  # Linear base reward dispatched to the code owner per referral
  owner-per-referral-rewards:
    - "[console]points give <owner> 1"

  # Tiered / Milestone actions for the code owner
  owner-milestones:
    5:
      type: "EVERY_X" # Triggers at 5, 10, 15, 20...
      commands:
        - "[console]crates key give <owner> rare 1"
        - "[console]broadcast &a[RedeemMaster] &e<owner> &7reached &b<count> &7referrals and earned a &6Rare Key&7!"
    25:
      type: "EXACT"   # One-time trigger at referral #25
      commands:
        - "[console]points give <owner> 25"
        - "[console]crates key give <owner> legendary 1"
        - "[console]broadcast &6&l[RedeemMaster] &e<owner> &6has reached a grand milestone of &e25 &6referrals!"

messages:
  prefix: "&8[&bRedeemMaster&8] &r"
  redeemed-success: "&aSuccessfully redeemed code &e<code-name>&a! Starter bonus awarded."
  already-redeemed: "&cYou have already redeemed a referral code on this server."
  self-redeem: "&cYou cannot redeem your own referral code!"
  code-not-found: "&cReferral code not found!"
  code-revoked: "&cThis referral code has been revoked by administrators."
  code-created: "&aReferral code created: &e<code-name>"
  already-has-code: "&cYou already have a referral code: &e<code-name>"
  invalid-code-format: "&cCustom code must be 3-12 alphanumeric characters (no spaces)."
  code-taken: "&cThat referral code is already taken by someone else!"
  no-permission: "&cYou do not have permission to execute this command."
  same-ip-blocked: "&cYou cannot redeem a referral code created from your own IP address/network."
  playtime-requirement: "&cYou need at least <required> minutes of playtime to redeem a code! (Current: <current>m)"
  reload-success: "&aConfiguration and data records reloaded successfully."
