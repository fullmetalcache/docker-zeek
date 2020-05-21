
This project is meant to run a single-system Zeek cluster inside of a docker container. It is based on, but differs from [blacktop/zeek:zeekctl](https://hub.docker.com/r/blacktop/zeek) in that it focuses on running multiple Zeek processes with `zeekctl`. To that end, there are several helpful features included:

- A configuration wizard for generating a `node.cfg` cluster configuration
- Will automatically run `zeekctl` on start and print a diagnostic report if it fails
- Cron will periodically ensure that all Zeek processes are running and restart any that have crashed
- Zeek's package maanger is included, allowing you to easily install zeek plugins
- Performance improvement with AF_Packet plugin installed and enabled by default in the configuration wizard
- Performance improvement by using `ethtool` to disable certain interface features by default

## Configuring

If you don't already have a `node.cfg` file you can use the following commands to generate one.

```bash
touch node.cfg
docker run --rm -it --network host \
    --mount source=$(pwd)/node.cfg,destination=/node.cfg,type=bind \
    activecm/zeek \
    zeekcfg -o /node.cfg
```

## Starting

```bash
docker run --cap-add net_raw --cap-add net_admin --network host --detach \
    --name zeek \
    --mount source=/etc/localtime,destination=/etc/localtime,type=bind,readonly \
    --mount source=YOURLOGS,destination=/usr/local/zeek/logs/,type=bind \
    --mount source=YOURCFG,destination=/usr/local/zeek/etc/node.cfg,type=bind \
    activecm/zeek
```

Replace `YOURLOGS` with the absolute path on your host system you want Zeek logs written to. Replace `YOURCFG` with the absolute path to your `node.cfg` file. For example:

```bash
docker run ---cap-add net_raw --cap-add net_admin --network host --detach \
    --name zeek \
    --mount source=/etc/localtime,destination=/etc/localtime,type=bind,readonly \
    --mount source=$(pwd)/logs,destination=/usr/local/zeek/logs/,type=bind \
    --mount source=$(pwd)/node.cfg,destination=/usr/local/zeek/etc/node.cfg,type=bind \
    activecm/zeek
```

Here are several locations in the container that you may want to customize by mounting your own files or directories:

* `/usr/local/zeek/logs/` - Directory where Zeek's logs are written.
* `/usr/local/zeek/etc/node.cfg` - Zeek's cluster configuration including how many workers and which interfaces to capture from. You need to customize this file with your desired sniffing network interface name at the very least.
* `/usr/local/zeek/etc/networks.cfg` - Internal network range definitions. If you have internal network ranges other than the standard RFC1918 (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) you can add them here.
* `/usr/local/zeek/etc/zeekctl.cfg` - Zeekctl settings. If you don't have a dedicated sniffing interface you'll want to disable the `interfacesetup.enabled` in this file.
* `/usr/local/zeek/share/zeek/site/local.zeek` - Determines which Zeek scripts are loaded. Any

## Stopping

```
docker stop zeek -t 90
```

## Updating

```
docker pull activecm/zeek

# Don't forget to recreate your container
```

## Installing a Plugin

```
# Container must be running already
docker exec zeek zkg install ja3
```

## Diagnosing Issues

If Zeek crashes right after starting you can check the log output.

```
docker logs zeek
```

If Zeek is successfully capturing and you want to see if there are any issues:

```
# Container must be running already
docker exec zeek zeekctl doctor
```

## Credits

Dockerfile based on [blacktop/docker-zeek](https://github.com/blacktop/docker-zeek/tree/master/zeekctl).
