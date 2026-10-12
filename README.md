# Palworld server hosting: RAM and CPU sizing for 32 players, real plan prices, and one-click setup without touching Linux

Palworld left Early Access on July 10, 2026, and the dedicated-server question came back with it. Steam peaked at 855,525 concurrent players on July 12, roughly a sixteen-fold jump from the ~58,000 the game was averaging two months earlier, and total players across PC, PS5, Xbox, Game Pass and Mac have passed 40 million. Public servers were never built for that.

So you look at hosting. Then you hit the wall every Palworld host hits: a flat "16GB recommended" answer that doesn't tell you whether your five friends need a ¥65/month box or a ¥360/month one, and hosting pages that quote specs without saying which CPU line they're actually selling.

This is the practical version. What the server really eats, which machine lines make sense, what they cost on RainYun, and how far the "one-click" promise actually gets you.

## What 1.0 changed for server owners

The 1.0 build isn't a patch. It added the World Tree endgame region, Sunreach, 72 new Pals (287 in total), an Awakening system, mutation breeding, aquatic bases and a raised level cap. On the server side, crossplay is on by default, and a single dedicated server now hosts PC, Xbox, PlayStation and Game Pass players together — which means your player list is bigger than your Steam friends list.

Two consequences matter for hardware:

- **The map got bigger.** More terrain, more entities, more disk I/O as players spread out and worlds generate. World generation still spikes RAM temporarily when new chunks load.
- **Bases got heavier.** More structures, more working Pals, more automation. Pal AI, breeding timers and base logic are largely single-threaded, so clock speed does more for you than core count past a point.

## How much RAM and CPU a Palworld world actually eats

Pocketpair's own dedicated-server documentation lists 8GB as the minimum and 16GB as recommended. The 8GB figure will boot a server, but it's also flagged as raising the risk of out-of-memory crashes. Here's how that translates to real player counts:

| Players | Practical RAM | Notes |
| --- | --- | --- |
| 2–4 | 8GB | Works with a nightly restart; fine for a starter camp |
| 5–8 | 12–16GB | The comfortable zone for a friend group with two or three active bases |
| 8–16 | 16GB | Pocketpair's own recommendation; treat it as the anchor, not an upsell |
| 16–32 | 24–32GB | Full 32-slot worlds, big base counts, or any modpack |

Two things the table doesn't capture. First, player count is not the main driver — base activity is. Working Pals keep simulating whether their owner is logged in, so a five-player server with sprawling production bases can out-eat a ten-player server where everyone stays mobile. BaseCampWorkerMaxNum in your config file is the direct lever on that load.

Second, Palworld has a well-documented memory leak. RAM climbs the longer the process stays up, regardless of activity. The fix isn't more RAM, it's a scheduled restart — a cron entry that bounces the server at 4 a.m. is the standard workaround, and most hosts running Palworld do it for you.

> On CPU: 4 cores is the floor, but high-clock beats high-core for Palworld. A 4-core Ryzen or i9 will generally hold tick rate better than a slower 8-core Xeon. Storage should be NVMe or SSD, with at least 20–30GB free once saves and backups are counted — the server files alone run around 12GB.

## Three ways to host, and who each one fits

**Official and public servers** cost nothing, but they're shared, crowded at peak hours, and you get zero control over rates, drops or death penalties. Fine for a first look at the game, useless for a persistent world.

**A managed game panel host** gives you a web console: pick Palworld from a list, click start, and manage files in a browser. No SSH, no SteamCMD, no user permission headaches.

**A bare VPS or cloud server** means you're running SteamCMD yourself on Ubuntu. More work, more control — mods, extra services alongside the game, custom ports.

RainYun (雨云) covers the first two properly and the third as a byproduct: it's a Chinese cloud provider founded in 2018 that runs game-specific machines on high-clock CPUs, and its own documentation ships a Palworld deployment walkthrough. Worth knowing the funding situation before you commit: the panel interface, the docs, and support are in Chinese.

## The machine lines that matter for Palworld

RainYun splits its game cloud into a **panel server** (their own panel or MCSM, fully browser-managed, one-click game deployment) and a **game cloud VPS** (full root access, install anything). Both come with 30GB system disk plus 20GB data disk, and both are sold by CPU line.

For Palworld specifically, the low-clock Xeon E5 2666v3 tier is the one to skip — RainYun's own tutorial recommends 4 cores / 8GB or higher on a 5900X, Xeon Gold 6146 or i9 14900KF machine, and notes the official recommendation is 4 cores / 16GB. Current lines with published pricing include:

| Machine line | Location & network | Entry config | Monthly (CNY) | Notes |
| --- | --- | --- | --- | --- |
| Xeon E5 2666v3 | Suqian / Shiyan | 2 cores · 4GB · 10Mbps | ¥65 | Cheapest; 2.9GHz base, dated silicon — fine for Minecraft, weak for Palworld |
| Xeon Gold 6146 | Suqian / Shiyan | 2 cores · 4GB · 10Mbps | ¥80 | 3.2GHz base / 4.2GHz boost; the value pick for Palworld |
| Ryzen 5900X | Suqian / Shiyan | 1 core · 2GB · 5Mbps | ¥125 | 3.7GHz base / 4.8GHz boost, 64MB cache; strong single-thread |
| Ryzen 7950X | Suqian / Shiyan | 1 core · 2GB · 5Mbps | ¥139 | 4.5GHz base; highest clock-per-yuan of the AMD lines |
| i9 13900KF | Suqian / Shiyan | 1 core · 2GB · 5Mbps | ¥145 | Overkill for Palworld |
| i9 14900KF | Suqian / Shiyan | 1 core · 2GB · 5Mbps | ¥145 | Fastest line; 6.3GHz boost |

Read the prices as entry points, not fixed SKUs. Cores, RAM, bandwidth and disk are all adjustable at purchase, each of which moves the number, and the game cloud lines carry 100G–200G DDoS protection with higher tiers on the Shiyan network. Ports are open on the panel servers via NAT mapping — you share a public IP and get a mapped address like `hm.rainplay.cn:31740` to hand out. Dedicated public IPs are available as a paid add-on on some lines.

👉 [Check current RainYun game cloud pricing and availability](https://www.rainyun.com/MzA2MTc2_)

If your group is outside Asia, the mainland endpoints are the wrong choice — pick a Hong Kong or Los Angeles cloud server instead. RainYun's regular cloud server (RCS) line runs KVM virtualisation with configurable disk and unlimited traffic, and pricing on standard plans is lower than the game lines:

| Cloud server plan | Location | CPU · RAM · Bandwidth | Monthly (CNY) |
| --- | --- | --- | --- |
| KVM entry | Hong Kong | 1 core · 1GB · 10Mbps up/10Mbps down | ¥28 |
| KVM standard | Hong Kong | 2 cores · 2GB · 20Mbps | ¥35 |
| KVM mid | Hong Kong | 2 cores · 4GB · 20Mbps | ¥48 |
| KVM high | Hong Kong | 4 cores · 4GB · 30Mbps | ¥68 |
| KVM top | Hong Kong | 4 cores · 8GB · 30Mbps | ¥108 |
| KVM flagship | Hong Kong | 8 cores · 16GB · 50Mbps | ¥188 |
| KVM entry | US (Los Angeles) | 1 core · 1GB · 20/30Mbps | ¥25 |
| KVM standard | US | 2 cores · 2GB · 30/50Mbps | ¥34 |
| KVM mid | US | 2 cores · 4GB · 40/60Mbps | ¥44 |
| KVM advanced | US | 4 cores · 4GB · 60/80Mbps | ¥60 |
| KVM high | US | 4 cores · 8GB · 80/100Mbps | ¥113 |
| KVM flagship | US | 8 cores · 16GB · 100/150Mbps | ¥197 |

For a Palworld world of 8–16 players, that puts you looking at the 4-core / 8GB rows in either region. Billing is in CNY, and neither the Hong Kong nor US nodes require ICP filing — which matters if you also want a web map or status page alongside the game server.

👉 [Compare Hong Kong and US cloud server plans](https://www.rainyun.com/MzA2MTc2_)

## Fixed billing vs dynamic billing, for a game that never fully sleeps

RainYun sells game cloud on two billing models. **Fixed** is a flat monthly rent with no usage charges. **Dynamic** charges a lower base rate and then adds cost based on CPU consumption — cheap while idle, full power when busy.

The usual rule of thumb from the host community is that if you play more than two hours a day, go fixed; less than that, go dynamic. For Palworld, tilt further toward fixed than you would for Minecraft. A Palworld world keeps simulating working Pals and automation whether or not anyone is connected, and the server process holds a growing RAM footprint even when idle. There's much less of the "empty server costs nothing" effect to exploit. If two or three of you log in most evenings, dynamic billing rarely wins.

## Discounts, trials and refunds worth knowing about

- **¥1 one-day trial** on cloud servers and game cloud products — one trial per product type, and the pricier machines cost a bit more than ¥1 to test.
- **7-day no-reason refund**, limited to three times per account.
- **30% off annual billing** across the board — a four-core Palworld machine on annual terms lands in the ¥70–90/month equivalent range rather than the list price.
- **First-month 50% off coupon** when you register through a referral link and bind WeChat to the account.
- **Stackable coupons from the points mall.** Points accumulate from routine account activity and can be swapped for coupons; the common 20%-off new-purchase coupon stacks with annual billing, landing around 56% of list price.
- **Free game cloud time via points.** RainYun runs a points-redemption route to free game cloud instances, which community write-ups cover in detail if you want to run Palworld at zero cost for a trial period.

👉 [Register and pick up the new-user coupons](https://www.rainyun.com/MzA2MTc2_)

## Getting a Palworld server online on a panel plan

This is the part that justifies the panel plan over a bare VPS. The whole flow, per RainYun's own Palworld guide:

1. Buy a game cloud **panel server**, choose Palworld as the game type, and pick a high-clock machine with 4 cores / 8GB or more.
2. Choose fixed or dynamic billing, then pay — annual gets the 30% discount automatically.
3. Open your game cloud list in the console and hit manage on the instance.
4. Click the panel address, log in with the credentials shown, and start the instance.
5. Copy the external address and port from the panel console. That string is what you hand to friends.
6. In Palworld, choose **Join Multiplayer Game (Dedicated Server)** and paste it in.
7. To change server settings, open the file manager, go to `pal/Pal/Saved/Config/LinuxServer/PalWorldSettings.ini`, and paste in the contents of `DefaultPalWorldSettings.ini` as a template before editing. The file ships empty.
8. Edit, save, restart the instance. Settings apply on restart.

The values most people touch first: `ServerPlayerMaxNum` (default 32), `ServerName`, `ServerPassword`, `AdminPassword`, `DeathPenalty`, `ExpRate`, `BaseCampWorkerMaxNum`, and `PublicPort` (default 8211). You can also switch the instance to a different game later from the same panel, and resize the config without rebuilding.

## Five things that trip people up

**Language.** Panel, documentation and support are Chinese-first, with support hours advertised as 7×22 rather than 24/7. If you can't read Chinese and can't lean on a community guide, budget extra time.

**Region.** Mainland nodes (Suqian, Shiyan, and the newer Ningbo and Zhenjiang game lines) are built for Asian players — domestic latency in the sub-30ms range on the good lines. A server in Shiyan is a bad host for a guild spread across Europe. Pick Hong Kong or Los Angeles instead.

**Shared IP on panel plans.** Panel servers use NAT and port mapping, so your address looks like a subdomain with a port, not a clean IP. Fine for Palworld, slightly awkward if you're also hosting a web service that needs port 80 or 443. Dedicated IPs are add-ons.

**Restart discipline.** The memory leak is real. Whatever plan you buy, set a nightly restart. If RAM sits pinned near the ceiling after restarts, that's your signal to move up a tier, not to restart more often.

**The refund window.** Seven days is enough to run a real test with your group, confirm the ping is acceptable, and confirm the config survives a full session. Use it before you commit to annual billing.

> Ports and connectivity: Palworld's default is UDP 8211, and some guides also open 27015 for Steam query. If friends can't connect, check the panel's port mapping before you start debugging the config file.

## Quick answers

**How much RAM do I need for a Palworld server?** 8GB for 2–4 players with scheduled restarts, 16GB for 8–16 players, 24–32GB if you're running 32 slots or mods.

**Can I host Palworld on a cheap 2-core box?** You can boot it, but base raids and Pal-heavy bases will stutter and the server may OOM. RainYun's own guide recommends 4 cores / 8GB minimum, and Pocketpair recommends 4 cores / 16GB.

**Is a panel server or a VPS better for Palworld?** Panel if you want the server live in minutes and don't want to touch SSH. VPS if you want mods, extra services, or full control over the install and ports.

**Which RainYun line should I buy for Palworld?** Xeon Gold 6146 at ¥80/month for a small group, Ryzen 5900X or 7950X for the best single-thread value, i9 14900KF if you want headroom and don't mind paying for it. Skip the E5 2666v3 tier.

**When's the cheapest time to commit?** Annual billing with a stacked points-mall coupon — roughly 56% of list — once you've confirmed the plan works through the ¥1 trial or a short monthly cycle.

👉 [Start a ¥1 trial and see how your world performs](https://www.rainyun.com/MzA2MTc2_)

Palworld hosting in 2026 is a sizing problem more than a shopping problem. Get the RAM right, keep the restart schedule honest, pick a region near your actual players, and almost any mid-tier high-clock machine will hold a 32-slot world. Get those wrong and no amount of extra cores will save the tick rate.
