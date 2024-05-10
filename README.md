# V Rising Dedicated Server (in a container)

[![Docker Pulls](https://img.shields.io/docker/pulls/whalybird/vrising-server.svg)](https://hub.docker.com/r/whalybird/vrising-server/)
[![Docker stars](https://img.shields.io/docker/stars/whalybird/vrising-server.svg)](https://hub.docker.com/r/whalybird/vrising-server)

This image is based on [didstopia/vrising-server](https://hub.docker.com/r/didstopia/vrising-server), I just updated the base image in use and rebuilt the container to get up-to-date dependencies so **this image works with V Rising 1.0**. The source code is [available on GitHub](https://github.com/C0Nd3Mnd/vrising-server/).

This image will always install/update to the latest steamcmd and V Rising server, all you have to do to update your server is to redeploy the container.

Also note that the entire `/steamcmd/vrising` folder can be mounted on the host system, which would avoid having to reinstall the game when updating or recreating the container.

## Usage

### Docker Compose

Minimal example `compose.yml` / `docker-compose.yml`:

```yaml
services:
  vrising:
    image: whalybird/vrising-server:latest
    container_name: vrising # Optionally name your container.
    restart: unless-stopped
    environment:
      V_RISING_SERVER_NAME: My V Rising Server
    ports:
      - 9876:9876/udp # Game port
      - 9877:9877/udp # Query port
      - 9878:9878/tcp # RCON port - change the default V_RISING_RCON_PASSWORD if you expose this port!
    volumes:
      - ./data:/app/vrising # Folder containing saves and settings. Should be owned by 1000:1000.
```

### Switching from `didstopia/vrising-server` to this image

Simply replace `image: didstopia/vrising-server:latest` with `image: whalybird/vrising-server:latest` in your existing Compose file and run `docker compose up -d`.

### Environment variables

*(Place these under the `environment:` section in your Compose file)*

* `V_RISING_SERVER_PERSISTENT_DATA_PATH` Path where V Rising should store settings and save games. I recommend leaving this at the default value and changing the volume binding instead (default: `"/app/vrising"`)
* `V_RISING_SERVER_BRANCH` Server version to use (currently the only known version is `public`) (default: `"public"`)
* `V_RISING_SERVER_START_MODE` Changes what happens on container start. `"0"` - update server and start afterward, `"1"` - update only (don't start), `"2"` - start only (don't update even if a newer version is available) (default: `"0"`)
* `V_RISING_SERVER_UPDATE_MODE` Changes when pending updates are applied. `"0"` - update only when the container (re)starts, `"1"` - update as soon as an update is found and restart the server automatically (default: `"0"`)
* `V_RISING_SERVER_DEFAULT_HOST_SETTINGS` Use default host settings. I recommend leaving this enabled and changing host settings via the environment variables described here instead (default: `true`)
* `V_RISING_SERVER_DEFAULT_GAME_SETTINGS` Use default game settings. I recommend leaving this enabled and overriding the game settings you want to change via a custom preset instead (see **Custom preset** below) (default: `true`)
* `V_RISING_SERVER_NAME` Name of your server (default: `"V Rising Docker Server"`)
* `V_RISING_SERVER_DESCRIPTION` A description for your server (default: `"V Rising server running inside a Docker container."`)
* `V_RISING_SERVER_BIND_IP` IP to bind to. Leave it empty if you don't know what this means (default: `""`)
* `V_RISING_SERVER_BIND_IP_AUTO_DETECT` Auto-detect IP to bind to. Leave this disabled unless you know what this means (default: `false`)
* `V_RISING_SERVER_GAME_PORT` Game port (UDP) the server listens to. You can leave this at default and change the port on the host instead (default: `9876`)
* `V_RISING_SERVER_QUERY_PORT` Query port (UDP) the server listens to. You can leave this at default and change the port on the host instead (default: `9877`)
* `V_RISING_SERVER_RCON_PORT` RCON port (TCP) the server listens to. You can leave this at default and change the port on the host instead (default: `9878`)
* `V_RISING_SERVER_RCON_ENABLED` Whether RCON should be enabled. If you disable this the server won't shut down cleanly when stopping the container (default: `true`)
* `V_RISING_SERVER_RCON_PASSWORD` RCON password. Change this from the default if you expose the RCON port (default: `"s3cr3t_rc0n_p455w0rd"`)
* `V_RISING_SERVER_MAX_CONNECTED_USERS` Amount of concurrent players allowed on your server (default: `40`)
* `V_RISING_SERVER_MAX_CONNECTED_ADMINS` Amount of reserved admin slots (default: `4`)
* `V_RISING_SERVER_SAVE_NAME` Name of the save game (default: `"docker"`)
* `V_RISING_SERVER_PASSWORD` Server password. Leave empty to allow connecting without a password (default: `""`)
* `V_RISING_SERVER_LIST_ON_MASTER_SERVER` List the server on the master server list. You can still connect to the server using direct connect (default: not set)
* `V_RISING_SERVER_LIST_ON_STEAM` List the server on Steam. You can still connect to the server using direct connect (default: `true`)
* `V_RISING_SERVER_LIST_ON_EPIC_EOS` List the server on Epic. You can still connect to the server using direct connect (default: `false`)
* `V_RISING_SERVER_AUTO_SAVE_COUNT` How many of the last n auto saves to keep (default: `50`)
* `V_RISING_SERVER_AUTO_SAVE_INTERVAL` Seconds between auto saves (default: `300` = 5 minutes)
* `V_RISING_SERVER_GAME_SETTINGS_PRESET` Game settings preset to use. Existing presets include `"StandardPvE"` or `"StandardPvP"` and many more. A custom preset can be created, see **Custom preset** below (default: `""`)
* `V_RISING_SERVER_GAME_ENABLE_PVP`, `V_RISING_SERVER_GAME_DISABLE_BLOOD_DRAIN`, `V_RISING_SERVER_GAME_DISABLE_DECAY` Modifies game settings for PvP, blood drain and decay. I recommend using a **Custom preset** (see below) to set these instead (default: `false`)

### Custom preset

If you don't want to play using the default game settings or alternatively a default preset, you can create your own preset. To do that, create a JSON file on your server and mount it into the `GameSettingPresets` folder in the container:

```yaml
services:
  image: whalybird/vrising-server:latest
  # ...
  volumes:
    - ./CustomPreset.json:/steamcmd/vrising/VRisingServer_Data/StreamingAssets/GameSettingPresets/CustomPreset.json
```

Make sure that your file (`CustomPreset.json`) is readable by UID 1000. In this `CustomPreset.json`, you can override any of the default settings. You can find the default settings in the `ServerGameSettings.json` file, which is located at `./data/Settings/ServerGameSettings.json` if you used the bind mount from the Compose example under **Docker Compose**. If you want to increase the maximum amount of clan members allowed per clan from 4 to 10 for example, your `CustomPreset.json` should look like this:

```json
{
  "ClanSize": 10
}
```

If you want to additionally increase the sun damage modifier from 1x to 1.5x, it should look like this:

```json
{
  "ClanSize": 10,
  "SunDamageModifier": 1.5
}
```

There are many other settings like `"GameDifficulty"` and `"GameModeType"`, but it would be too much work to list them all here, but most of them are self-explanatory when looking them up in the default configuration file.

## License

See [LICENSE](https://github.com/C0Nd3Mnd/vrising-server/blob/master/LICENSE)
