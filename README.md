<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:E3342F,50:7f0000,100:0d1f33&height=220&section=header&text=roboat&fontSize=85&fontColor=ffffff&animation=fadeIn&fontAlignY=42&desc=python+%C2%B7+roblox+%C2%B7+built+different&descAlignY=63&descAlign=50&descColor=ffcccc&descSize=18" width="100%"/>

<br/>

<img src="https://bitter-forest-7924.blammervale.workers.dev/roboat_logo.svg" width="220" height="160" alt="roboat"/>

<br/>

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=700&size=17&pause=1000&color=E3342F&center=true&vCenter=true&width=600&lines=sail+through+the+roblox+api;no+cookie.+no+hassle.+just+oauth.;async+%2B+sync.+your+choice.;datastores%2C+leaderboards%2C+bans+%E2%80%94+all+covered;rap+tracking+%26+market+analysis;real-time+events+that+actually+work;a+terminal+you+will+actually+use" alt="Typing SVG" />

<br/><br/>

[![Python](https://img.shields.io/badge/python-3.8%2B-E3342F?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![MIT](https://img.shields.io/badge/license-MIT-991B1B?style=for-the-badge)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.1.0-E3342F?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Addi9000/roboat/releases)
[![Site](https://img.shields.io/badge/roboat.pro-visit-7f0000?style=for-the-badge&logo=googlechrome&logoColor=white)](https://roboat.pro)
[![Stars](https://img.shields.io/github/stars/Addi9000/roboat?style=for-the-badge&color=E3342F&logo=github&logoColor=white&label=stars)](https://github.com/Addi9000/roboat/stargazers)

<br/>

> **roboat** is a Python library for the Roblox API that doesn't suck.
> OAuth auth, typed responses, async support, a built-in database, Open Cloud integration,
> DataStore read/write, ordered leaderboards, marketplace tools, and a terminal you'll actually want to use.

<br/>

</div>

---

## install

```bash
pip install roboat

# with async support
pip install "roboat[async]"

# everything
pip install "roboat[all]"
```

---

## quick start

```python
from roboat import RoboatClient

client = RoboatClient()

user = client.users.get_user(156)
print(user)  # Builderman (@builderman) ✓ [ID: 156]

game = client.games.get_game(2753915549)
print(f"{game.name} — {game.visits:,} visits, {game.playing:,} playing")

for item in client.catalog.search(keyword="fedora", sort_type="Sales"):
    print(f"{item.name} — {item.price}R$")
```

No setup. No cookie extraction. Just works.

---

## auth

roboat uses Roblox OAuth 2.0. Open a browser, log in, done.

```python
from roboat import OAuthManager, RoboatClient

manager = OAuthManager(timeout=120)
token = manager.authenticate()  # opens browser, waits up to 120s

client = RoboatClient(oauth_token=token)
```

In the terminal, type `auth`. Browser opens, you log in, live countdown while it waits.

---

## datastores

This is where roboat really shines for game developers. Full read/write/delete access to Roblox DataStores from Python — no in-game script needed.

```python
API_KEY  = "roblox-KEY-xxxxx"   # from create.roblox.com/dashboard/credentials
UNIVERSE = 123456789

# write player data
client.develop.set_datastore_entry(
    UNIVERSE, "PlayerData", "player_156",
    value={"coins": 1000, "level": 25, "wins": 42},
    api_key=API_KEY,
    user_ids=[156],   # link to Roblox user for GDPR
)

# read it back
data = client.develop.get_datastore_entry(UNIVERSE, "PlayerData", "player_156", API_KEY)
print(data)  # {"coins": 1000, "level": 25, "wins": 42}

# increment a counter atomically
client.develop.increment_datastore_entry(UNIVERSE, "Stats", "total_deaths", 1, API_KEY)

# list all keys with a prefix
keys = client.develop.list_datastore_keys(UNIVERSE, "PlayerData", API_KEY, prefix="player_")

# delete stale data
client.develop.delete_datastore_entry(UNIVERSE, "Sessions", "expired_key", API_KEY)

# see version history for any key
versions = client.develop.list_datastore_versions(UNIVERSE, "PlayerData", "player_156", API_KEY)
```

### leaderboards — ordered datastores

Sorted stores for global rankings, scores, and anything you want in order.

```python
# update a player's score (creates if doesn't exist)
client.develop.set_ordered_datastore_entry(
    UNIVERSE, "GlobalLeaderboard", "player_156",
    value=9500, api_key=API_KEY, allow_missing=True,
)

# add to their score
client.develop.increment_ordered_datastore(
    UNIVERSE, "GlobalLeaderboard", "player_156",
    amount=100, api_key=API_KEY,
)

# get top 10 players
top = client.develop.list_ordered_datastore(
    UNIVERSE, "GlobalLeaderboard", API_KEY,
    max_page_size=10, order_by="desc",
)
for i, entry in enumerate(top.get("entries", []), 1):
    print(f"#{i}  {entry['id']}  {entry['value']:,}")
```

### list all datastores in a universe

```python
stores = client.develop.list_datastores(UNIVERSE, API_KEY)
for s in stores:
    print(s.name, s.created)
```

### broadcast to all live game servers

```python
# sends to every server currently running via MessagingService
client.develop.announce(UNIVERSE, API_KEY, "double xp event starts now!")
client.develop.publish_message(UNIVERSE, "Rewards", "daily_bonus_active", API_KEY)
client.develop.broadcast_shutdown(UNIVERSE, API_KEY, "server restart in 60s")
```

### ban and unban players

```python
# temporary ban — 24 hours
client.develop.ban_user(
    UNIVERSE, user_id=1234, api_key=API_KEY,
    duration_seconds=86400,
    display_reason="You have been temporarily banned.",
    private_reason="exploiting — auto-detected",
)

# permanent ban
client.develop.ban_user(
    UNIVERSE, user_id=5678, api_key=API_KEY,
    duration_seconds=None,
    display_reason="Permanently banned.",
)

# lift a ban
client.develop.unban_user(UNIVERSE, user_id=1234, api_key=API_KEY)

# see all active bans
bans = client.develop.list_bans(UNIVERSE, API_KEY, max_page_size=100)
```

---

## local database

Built-in SQLite layer for persisting Roblox data between Python runs. Great for caching, inventory diffs, analytics, and tracking changes over time.

```python
from roboat import SessionDatabase

db = SessionDatabase.load_or_create("myproject")

# save API objects directly — no manual serialisation
db.save_user(client.users.get_user(156))
db.save_game(client.games.get_game(2753915549))

# key-value store for anything JSON-serialisable
db.set("tracked_users", [156, 261, 1234])
db.set("last_poll",     "2024-06-01T12:00:00")
db.set("config",        {"interval": 30, "rap_threshold": 5000})

val = db.get("tracked_users")   # [156, 261, 1234]

# retrieve cached data
user = db.get_user(156)
game = db.get_game(2753915549)

print(db.stats())
# {"users": 10, "games": 5, "session_keys": 3, "log_entries": 42}
db.close()
```

### inventory diff — detect what changed

Run this on a schedule to track limited item changes:

```python
from roboat import RoboatClient, SessionDatabase
from roboat.utils import Paginator

client = RoboatClient()
db     = SessionDatabase.load_or_create("inventory_tracker")

USER_ID = 156

# fetch current collectibles
assets   = Paginator(lambda c: client.inventory.get_collectibles(USER_ID, cursor=c)).collect()
current  = {a.asset_id: a.recent_average_price for a in assets}
saved    = db.get(f"inv:{USER_ID}") or {}

gained = {k: v for k, v in current.items() if k not in saved}
lost   = {k: v for k, v in saved.items()   if k not in current}

for aid, rap in gained.items():
    print(f"+ gained asset {aid}  RAP: {rap:,}R$")
for aid, rap in lost.items():
    print(f"- lost asset {aid}  RAP: {rap:,}R$")

db.set(f"inv:{USER_ID}", current)
db.close()
```

---

## the terminal

```bash
roboat
```

```
  ____       _     _            _    ____ ___
 |  _ \ ___ | |__ | | _____  __/ \  |  _ \_ _|
 | |_) / _ \| '_ \| |/ _ \ \/ / _ \ | |_) | |
 |  _ < (_) | |_) | | (_) >  < ___ \|  __/| |
 |_| \_\___/|_.__/|_|\___/_/\_/_/  \_|_|  |___|

  roboat v2.1.0  ·  roboat.pro

» start 156
» auth
» game 2753915549
» inventory 156
» rap 156
» owns 156 1365767
» universe 2753915549
» newdb tracker
» save user 156
```

| command | what it does |
|---|---|
| `start <id>` | begin session (required first) |
| `auth` | oauth login via browser |
| `game <id>` | game stats + votes |
| `friends <id>` | friends list |
| `inventory <id>` | limiteds + RAP |
| `rap <id>` | total RAP |
| `catalog <kw>` | search avatar shop |
| `presence <id>` | online status |
| `trades` | your trades |
| `servers <placeid>` | active servers |
| `universe <id>` | dev universe info |
| `owns <uid> <aid>` | ownership check |
| `newdb / loaddb` | local database |
| `watch <id>` | visit milestone alerts |

---

## clients

### sync

```python
from roboat import RoboatClient, ClientBuilder

client = (
    ClientBuilder()
    .set_oauth_token("TOKEN")
    .set_timeout(15)
    .set_cache_ttl(60)    # cache responses for 60s
    .set_rate_limit(10)   # throttle to 10 req/s
    .build()
)
```

### async

```python
import asyncio
from roboat import AsyncRoboatClient

async def main():
    async with AsyncRoboatClient() as client:
        game, votes, icon = await asyncio.gather(
            client.games.get_game(2753915549),
            client.games.get_votes([2753915549]),
            client.thumbnails.get_game_icons([2753915549]),
        )
        # auto-chunks into batches of 100
        users = await client.users.get_users_by_ids(list(range(1, 501)))

asyncio.run(main())
```

---

## groups

```python
group = client.groups.get_group(7)
roles = client.groups.get_roles(7)

# requires auth
client.groups.set_member_role(7, user_id=1234, role_id=role.id)
client.groups.kick_member(7, user_id=1234)
client.groups.post_shout(7, "hey everyone")
client.groups.accept_all_join_requests(7)
client.groups.pay_out(7, user_id=1234, amount=500)
```

---

## marketplace & RAP

```python
from roboat.marketplace import MarketplaceAPI

market = MarketplaceAPI(client)

data   = market.get_limited_data(1365767)
print(f"{data.name}  {data.recent_average_price:,}R$  {data.price_trend}")

profit = market.estimate_resale_profit(1365767, purchase_price=12000)
print(f"net: {profit.estimated_profit:,}R$  roi: {profit.roi_percent}%")

tracker = market.create_rap_tracker([1365767, 1028606])
tracker.snapshot()
tracker.snapshot()
print(tracker.summary())

deals = market.find_underpriced_limiteds([1365767, 1028606, 19027209])
```

---

## events

```python
from roboat import EventPoller

poller = EventPoller(client, interval=30)

@poller.on_friend_online
def online(user):
    print(f"{user.display_name} just came online")

poller.track_game(2753915549, milestone_step=1_000_000)

@poller.on("visit_milestone")
def milestone(game, count):
    print(f"{game.name} hit {count:,} visits")

poller.start()
```

---

## social graph

```python
from roboat.social import SocialGraph

sg = SocialGraph(client)

mutuals  = sg.mutual_friends(156, 261)
snap     = sg.presence_snapshot([156, 261, 1234])
online   = sg.who_is_online([156, 261, 1234])
nodes    = sg.most_followed_in_group([156, 261, 1234])
suggests = sg.follow_suggestions(156, limit=10)
```

---

## errors

```python
from roboat import UserNotFoundError, RateLimitedError, RoboatAPIError
from roboat.utils import retry

@retry(max_attempts=3, backoff=2.0)
def get(uid):
    return client.users.get_user(uid)

try:
    user = get(99999999999)
except UserNotFoundError:
    print("doesn't exist")
except RateLimitedError:
    print("slow down")
except RoboatAPIError as e:
    print(e)
```

---

## pagination

```python
from roboat.utils import Paginator

for follower in Paginator(
    lambda c: client.friends.get_followers(156, limit=100, cursor=c)
):
    print(follower)

top_500 = Paginator(
    lambda c: client.friends.get_followers(156, limit=100, cursor=c),
    max_items=500,
).collect()
```

---

## api coverage

```
users · games · catalog · groups · friends · badges
economy · presence · avatar · trades · messages · inventory
thumbnails · develop · opencloud · analytics · marketplace
social · notifications · publish · moderation · events · database
```

24 modules. Every major Roblox endpoint.

---

## cli tools

```bash
python tools/bulk_lookup.py --ids 1 156 261 --format csv
python tools/rap_snapshot.py --user 156 --diff
python tools/game_monitor.py --universe 2753915549 --interval 60
python scripts/check_env.py
```

---

## license

```
MIT License — Copyright (c) 2024 roboat contributors

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the "Software"),
to deal in the Software without restriction, including without limitation
the rights to use, copy, modify, merge, publish, distribute, sublicense,
and/or sell copies of the Software, and to permit persons to whom the
Software is furnished to do so.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND.
```

---

<div align="center">

<br/>

[![roboat.pro](https://img.shields.io/badge/%F0%9F%8C%90%20roboat.pro-E3342F?style=for-the-badge)](https://roboat.pro)
&nbsp;
[![github](https://img.shields.io/badge/github-Addi9000%2Froboat-0d1f33?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Addi9000/roboat)
&nbsp;
[![issues](https://img.shields.io/badge/report%20a%20bug-991B1B?style=for-the-badge)](https://github.com/Addi9000/roboat/issues)

<br/>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1f33,50:7f0000,100:E3342F&height=140&section=footer" width="100%"/>

</div>
