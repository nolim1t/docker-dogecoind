# Much container.. wow

This is a dockerized version of [dogecoin](https://github.com/dogecoin/dogecoin) built the [LNCM way](https://github.com/lncm). 

Not directly affliated with LNCNM, but more a lighthearted version of crypto (and this stuff is still better than ETH or BCASH)

Also, setting this up helps me understand [docker-bitcoind](https://github.com/lncm/docker-bitcoind) which is used by many people to ensure quality.

Also, I think it would be fun to run a dogecoin node in 2021.

## Changes / Differences from bitcoin

* No verification (reckless), because dogecoin. nuff said. Doesn't seem to be a PGP key I can use
* binary files are different
* port number would be different

## Repo Mirrors

* [Github](https://github.com/nolim1t/docker-dogecoind)
* [Gitlab](https://gitlab.com/nolim1t/docker-dogecoind)

## Building

```bash
# Replace 'arm64v8' with github arch matrix type
# Replace '1.14' with version to build
docker build --build-arg "ARCH=arm64v8" \
	-t nolim1t/dogecoin \
	./1.14
```

## Configuring

At a bare minimum, dogecoind requires that the rpcpassword setting be set when running as a daemon. If the configuration file does not exist or this setting is not set, dogecoind will shutdown promptly after startup.

This password does not have to be remembered or typed as it is mostly used as a fixed token that dogecoind and client programs read from the configuration file, however it is recommended that a strong and secure password be used as this password is security critical to securing the wallet should the wallet be enabled.

If dogecoind is run with the "-server" flag (set by default), and no rpcpassword is set, it will use a special cookie file for authentication. The cookie is generated with random content when the daemon starts, and deleted when it exits. Read access to this file controls who can access it through RPC.

By default the cookie is stored in the data directory, but it's location can be overridden with the option '-rpccookiefile'.

This allows for running dogecoind without having to do any manual configuration.

## Running

```bash
docker run --it \
	--name dogecoin \
	-v $HOME:/data \
	nolim1t/dogecoin
```


