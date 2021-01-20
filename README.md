# Dockerized LGSM Rust Dedicated Server

This project combines Docker, [Rust][rust] Dedicated Server, and [Linux
GSM][lgsm] all in one!  Self-hosted Rust dedicated server management made easy.

- [Play on your server](#play-on-your-server)
- [Playing multiplayer](#playing-multiplayer)
- [Prerequisites](#prerequisites)
- [Getting started](#getting-started)
- [Server power management](#server-power-management)
  - [Starting the server](#starting-the-server)
  - [Graceful shutdown](#graceful-shutdown)
  - [Uninstallation](#uninstallation)
- [Game Server Administration](#game-server-administration)
  - [Login shell](#login-shell)
  - [RCON: Remote Admin Console](#rcon-remote-admin-console)
  - [Limiting server resources](#limiting-server-resources)
  - [Easy Anti-Cheat](#easy-anti-cheat)
- [Server Mods](#server-mods)
  - [Oxide mods](#oxide-mods)
  - [Custom mods](#custom-mods)
  - [Updating mod configuration](#updating-mod-configuration)
- [Customize Map](#customize-map)
  - [Generated Maps](#generated-maps)
  - [Custom Maps](#custom-maps)
    - [Self-hosted custom maps](#self-hosted-custom-maps)
    - [Remotely hosted custom maps](#remotely-hosted-custom-maps)

# Play on your server

By default your server is forwarding port `28015/UDP` to all interfaces so that
you can use router port forwarding to play multiplayer.

If you're playing on the same machine from Proton on Linux, then press F1 to
open console and connect with:

    client.connect 127.0.0.1:28015

You may need to enter a domain name or alternate IP address if you're playing
Rust from a different computer.

# Playing multiplayer

Enable port forwarding on your router for the following ports.

- `28015/udp` (required for game clients)
- `8000/tcp` (optional: only required if self-hosting a custom map)

# Prerequisites

- 4GB of RAM if the server is remotely hosted (total memory including OS).
  Alternately, 16GB of RAM if you're going to host a dedicated server and play
  on the same machine.  These memory recommendations are just estimates and
  minimum requirements for memory could be much lower.
- You have [Git installed][git] and cloned this repository to work with locally.

  ```
  git clone https://github.com/samrocketman/docker-compose-lgsm-rust-dedicated-server
  ```

- Install [Docker on Linux][docker].  Docker on Windows or Mac would probably
  work but is entirely untested.  Docker for Mac has known performance issues
  unrelated to Rust.
- Install [docker-compose][compose].  This typically comes separate from Docker.

# Getting started

If you don't want to customize anything, then start the server.  It will
generate a 3K map using a random seed which will persist when restarting the
server.

    docker-compose up -d

It may take 15 minutes or longer for the server to start the first time
depending on your internet connection.  This is because it has to download Rust
among other server setup tasks.  You can monitor the server logs at any time (to
see progress) with the following command.

    docker-compose logs -f

Press `CTRL+C` to exit logs.

# Server power management

### Starting the server

    docker-compose up -d

It may take at 5 minutes or longer to start depending on your internet
connection.

See logs with the following command (`CTRL+C` to cancel).

    docker-compose logs -f

### Graceful shutdown

    docker-compose down

### Uninstallation

To completely uninstall and delete all Rust data run the following command.

    docker-compose down -v --rmi all

Remove this Git repository for final cleanup.

# Game Server Administration

### Login shell

If you want a shell login to your server, then run the following command from
the root of this repository.

```bash
./admin/shell.sh

# alternately if you need root shell access
./admin/shell.sh root
```

### RCON: Remote Admin Console

You can access the Rust RCON interface using any RCON client.  I recommend one
of the following clients.

- https://facepunch.github.io/webrcon Facepunch official client
- http://rcon.io/login community RCON client

The RCON interface is password protected.  Reveal the password using the
following command.

    ./admin/get-rcon-pass.sh

The script will output your RCON password as well as additional instructions for
your web browser to access the RCON console.

By default, the RCON interface is only accessible from `localhost`.  However, if
you require remote access, then you can set the `RUST_RCON_INTERFACE` variable
before starting the server.

```bash
docker-compose down
export RUST_RCON_INTERFACE=0.0.0.0
docker-compose up -d
```

### Limiting server resources

In the [`docker-compose.yml`](docker-compose.yml) file, there's two settings you
can adjust to limit how much CPU and memory the dedicated server is allowed.  By
default, it is set to dedicated server recommended values for extremly high
populations:

```yaml
cpu_count: 2
mem_limit: 8gb
```

You can adust the resources to your liking.  Generally, I recommend to not set
the server below `2` CPUs and  `2gb` of memory (RAM).  These policies ensure the
server can't use more than these limits.

### Easy Anti-Cheat

By default, EAC is disabled for Linux clients.  Enable EAC with the following
shell variable in [`rust-environment.sh`](rust-environment.sh).


```bash
export ENABLE_RUST_EAC=1
```

If EAC is enabled, Linux clients will not be able to connect and your server
will be listed in the in-game server browser.

# Server Mods

[Oxide plugins](https://umod.org/) and custom mods are supported.  I use the
term mods and plugins interchangeably.

Oxide mods are installed and updated automatically on every server start.
However, if you have a custom mod that has a name conflict with an official
Oxide mod, then the custom mod will be used.

### Oxide mods

To automatically install mods, create a new text file:
[`mod-configs/plugins.txt`](mod-configs).  Add what plugins you would like
in your rust server; one plugin per line.

For example, let's say you want the following plugins:

* [Backpacks](https://umod.org/plugins/backpacks)
* [Chest Stacks](https://umod.org/plugins/chest-stacks)

The download links for both of those plugins would be `Backpacks.cs` and
`ChestStacks.cs`.  Your `mod-configs/plugins.txt` would need to have the
following contents.

```bash
# example mod-configs/plugins.txt
# code comments are supported along with blank lines
Backpacks
ChestStacks
```

When the server boots, the mods will be automatically downloaded from uMod.  If
they already exist, then updates will be checked instead.

If you edit `mod-configs/plugins.txt`, then you can reload plugins without
restarting the server.  Run the following command.

    ./admin/get-or-update-oxide-plugins.sh

If you remove a plugin from `mod-configs/plugins.txt`, then it will be
deleted from your server automatically.

You can also remove mods from `mod-configs/plugins.txt` by starting the line
with a `#`.  This allows you to delete the plugin from the server but also keep
it around in case you want to re-enable it later.

### Custom mods

If you're writing your own custom mod, then the file name must end with `.cs`.
For example, `MyCustomMod.cs`.  Place any `.cs` files into the
[`custom-mods/`](custom-mods) directory.  Next time your server


You can edit your mods as much as you want without affecting the mod used by the
server.  You can copy and update to your custom mod to the server using the
following command.

    ./admin/get-or-update-oxide-plugins.sh

If you delete a custom mod from `custom-mods/` folder, then it will be removed
from the server automatically.

### Updating mod configuration

Oxide Mods automatically generate plugin config which is accessible by editing
files in the [`mod-configs/`](mod-configs/) directory.  If you edit a JSON
config you can open up the web management RCON console to reload the plugin (See
[RCON: Remote Admin Console](#rcon-remote-admin-console) for how to access RCON
console).

Use console command:

    oxide.reload plugin_name

Or to reload all plugins:

    oxide.reload *

Use the uMod download name of the plugin.  For example, if you download from
uMod `Backpacks.cs`, then you need only add `Backpacks` to
`mod-configs/plugins.txt`.

# Customize Map

You can have a randomly generated map with a seed or a custom map.

### Generated Maps

> Note: Generated Map settings are completely ignored if you've configured a
> Custom Map.

You can uncomment and change the following variables in
[`rust-environment.sh`](rust-environment.sh).

- seed
- salt
- worldsize

### Custom Maps

Please note:

> _Your map file should be hosted on a public web-site that works 24/7, since
> new players of your server will download the map from that URL, not from your
> Rust server. If your URL link doesn't work then players that haven't
> downloaded the map yet won't be able to join the server._
> - [facepunch Wiki: Hosting a custom map][fp-custom-maps]

There's two ways a custom map is supported.

1. Self-hosted
2. Remotely hosted

##### Self-hosted custom maps

If you're playing multi-player and self hosting your Map, then there's two extra
configuration items you must toggle.

- Port forward port `8000/tcp` to your router.
- Set `MAP_BASE_URL` variable in [`rust-environment.sh`](rust-environment.sh) to
  your public IP where clients can download the map.

Your custom map file name must end with `.map`.  Download your custom map
locally to your computer and place it in the [`custom-maps/`](custom-maps/)
directory.

If you have more than one map, then the first map alphabetically is used.  If
you would like to have multiple custom maps and change the map, then it is
recommended you prefix all maps with a 4 digit number.  For example,

```
0001_custom-map.map
0002_custom-map.map
0003_custom-map.map
... etc
```

If you wish to use a Generated Map, then all files ending with `.map` must be
removed from `custom-maps/` directory.  Just having the custom map file in that
directory is what enables the Custom Map logic.

##### Remotely hosted custom maps

If you're using a public file serving service separate from your game server
(such as Dropbox), then set `CUSTOM_MAP_URL` variable in
[`rust-environment.sh`](rust-environment.sh).  No extra configuration is
required.

# Road Map

- :heavy_check_mark: Initial working vanilla server
- :heavy_check_mark: Basic admin actions like shell login and RCON access
- :heavy_check_mark: Support for adding server mods and automatic mod updates
- :heavy_check_mark: Limit server resources
- :heavy_check_mark: Support for customizing initial Map generation on first
  time startup.
- :heavy_check_mark: Support for custom server mods (Oxide plugins)
- :heavy_check_mark: Support for custom Maps.
- :heavy_check_mark: Improve documentation

[compose]: https://docs.docker.com/compose/install/
[docker]: https://docs.docker.com/engine/install/
[git]: https://git-scm.com/
[lgsm]: https://linuxgsm.com/
[rust]: https://rust.facepunch.com/
[fp-custom-maps]: https://wiki.facepunch.com/rust/Hosting_a_custom_map
