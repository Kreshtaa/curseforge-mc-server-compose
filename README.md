# curseforge-mc-server-compose
This is a docker-compose.yml template for quickly starting up a modded Minecraft server with a CurseForge modpack through a playit.gg tunnel which avoids the hassle of opening ports and such.

## Requirements
- a Linux machine to run it on - it may work otherwise but it was built and tested on Linux only (Debian 13)
- Docker with the Compose plugin installed
- a CurseForge API key from [console.curseforge.com](https://console.curseforge.com)
- a [playit.gg](https://playit.gg) account and the secret key from setting up a tunnel and agent with them - more on that down below
- **DISCLAIMER:** The `docker-compose.yml` file has the flag `EULA: "true"` set. This means that by running this server image, you are agreeing to [Minecraft's End User License Agreement (EULA)](https://www.minecraft.net/en-us/eula). 

## Installation

### 1. Get the files

```bash
git clone https://github.com/Kreshtaa/curseforge-mc-server-compose.git
cd curseforge-mc-server-compose
```
- If you do not have git on your machine, you can hit the green **Code** button on the top right corner of this page and hit **Download ZIP**, then extract it where you want it on your computer.

### 2. Create the log and backup directories
These directories will be where the logs and backups from your server are saved.

```bash
mkdir -p /path/to/logs /path/to/backups
```
- Replace `/path/to/logs` and `/path/to/backups` with real paths, e.g. `/home/yourname/mc-logs` and `/home/yourname/mc-backups`

#### IMPORTANT:

If you run `id -u` and it doesn't show 1000, run the following as well:
```bash
sudo chown 1000:1000 /path/to/logs /path/to/backups
```

This is because the itzg/minecraft-server image runs as user ID 1000 by default, which is also what Linux typically assigns to the first user account that gets created. If yours is different, the container won't have write permission for those directories since your user account created them.

### 3. Create the .env
```bash
cp .env.example .env
nano .env
```
Enter your CurseForge API key (`CF_API_KEY`), the RCON password (`RCON_PASSWORD`), the two directory paths you just created (`LOG_PATH` and `BACKUP_PATH`) and optionally the timezone (`TIMEZONE`). The Playit.gg key comes next so don't close the editor window just yet.

### 4. Set up your Playit.gg agent and tunnel
- On [playit.gg/account/setup](https://playit.gg/account/setup) set up a playit agent to get its secret key, then go back to your terminal and copy-paste it into the `SECRET_KEY` field as its value.
- Save your `.env` file by using `CTRL+O`, `Enter` and `CTRL+X` in that exact sequence
- Go back to [playit.gg/account/setup](https://playit.gg/account/setup) and create a tunnel of type `Minecraft Java` and point it to `127.0.0.1` on port `25565`. The tunnel's public address is what your players will connect to.
- *Optionally if you own your own domain (like my own `kreshy.com`), you can buy a playit.gg subscription and connect the server to your domain/subdomain. This adds extra steps but it's a fun way to personalize your server, and makes the server address much easier to remember.*

### 5. Pick the modpack
- Find the URL of your chosen modpack on CurseForge, and extract the last bit of text from after a `/`. (*for example the SkyFactory 4 URL is "`https://www.curseforge.com/minecraft/modpacks/skyfactory-4`" which means the string of text you would use is `skyfactory-4`.*)

- In your terminal run `nano docker-compose.yml` and fill in the `CF_SLUG` value with the short string of text you just got from the modpack URL.

- Check that the Java tag after `itzg/minecraft-server:` matches the modpack's Minecraft version (different MC versions require different versions of Java). The Java version number can be found fairly easily by just googling "`Minecraft 1.x.x Java version`". If it does not check out, adjust it.

- Adjust the RAM allocated to your server (`MEMORY`) based on how much your server will need (for example the default is `MEMORY: "8G"`).

- Save and exit the editor (`CTRL+O` -> `Enter` -> `CTRL+X`).

#### How much RAM?
- Start from the modpack's own recommendation.
- Add more for more players or a large world.
- More isn't always better: too much can make performance worse.
- **Always leave some RAM free for your system.** Run `free -h` to see how much your machine has.

### 6. RUN THE SERVER!
In your terminal, run the following:
```bash
docker compose up -d
docker compose logs -f minecraft
```
The first start downloads the entire modpack, so it may take a while. Later starts are much faster. The second line in that command shows you the logs as they roll in - once the download finishes and the server is up it should display `Done (…)!`. At that point you can stop watching the logs by hitting `CTRL+C` and the server should stay up no problem.

## Switching modpacks
Switching modpacks using this setup is fairly simple but still has a process to it as outlined below.

### 1. Down the old server (if running)
Shut down the old modpack server by running the following command in the directory it's running from:
```bash
docker compose down
```
You can always come back to it later and restart it with `docker compose up -d`, just be sure to shut down the new server - **these two instances can never run at once on the same system**.

### 2. Create new server directory
Do this along with new log and backup directories or else the backups for your old server will get deleted due to the nature of the 7 day rolling backup cycle.

For instance I have a general `minecraft-servers` directory which is branched off into separate modpack folders, which in turn are branched off into logs and backups for each.

**IMPORTANT:** if you had to manually give user ID 1000 write permission via `sudo chown 1000:1000 "/path/to/logs/" "/path/to/backups/"`, do that on the new ones as well.

### 3. Copy old `docker-compose.yml` and `.env` files
Copy the existing `docker-compose.yml` and `.env` files into your new server directory.

### 4. Adjust the files
- In `.env` set the `LOG_PATH` and `BACKUP_PATH` variables to the new corresponding directories.

- Adjust the configuration of `docker-compose.yml` file to match what the new modpack requires, just as you did the first time around.

### 5. Start her up
You can start the new server up by running the following commands in its directory:
```bash
docker compose up -d
docker compose logs -f minecraft
```
Like the first install, the new modpack downloads on first start. Wait for `Done (…)!`.

## Restoring a backup
Backups are timestamped `.tar.gz` archives of the server's `data` folder, saved in your `BACKUP_PATH`. To restore one, stop the server with `docker compose down` first, then replace the world in `data` with the one from the backup and start the server again. Move the current world aside rather than deleting it, in case you need it back.

## If you run into issues

In my experience, most issues can be resolved by just asking AI for help, but remember - ***NEVER INPUT YOUR API KEYS OR `.env` FILES INTO AI!*** If AI or anyone else requests to see them, ***refuse*** so they cannot be abused. Only describe them (e.g. "this key is surrounded by single quotes and contains letters, numbers and dollar signs") - but no specifics.

That being said, if you do run into an issue you cannot overcome, please feel free to open an issue here on GitHub, or email me directly at [kresh@kreshy.com](mailto:kresh@kreshy.com) and I will try my best to see if I can help resolve it to the best of my ability.

## License

Licensed under MIT license

(See the `LICENSE` file in project repository.)
