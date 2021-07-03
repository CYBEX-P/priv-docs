# Bind interfaces and port

## KMS and backend

To change bind interface and port edit `docker-compose.yml` for each corresponding module, under `port`. They will be ending in `:5000`, this is required as it binds to internal container port `5000`, for more information on dbinding correctly refer to [Docker's network documentation](https://docs.docker.com/config/containers/container-networking/). For information editing docker compose file refer to [docker compose port documentation](https://docs.docker.com/compose/compose-file/compose-file-v3/#ports). The correct notation is `IPADDR:HOSTPORT:CONTAINERPORT`.

```{admonition} locahost does not work
:class: caution

`localhost` is not a valid interface, please use `127.0.0.1` if wanting to bind locally only. A local bind will not allow outide of the machine access, perfect for setting up a proxy with TLS and basic authentication infront of it.
```

## Collector 

The collector's bind interface is controller by the `./collector.sh` script. By default it binds to all network interfaces (0.0.0.0) at port 6000. If this is not the desired behaviour please change, as it could dbe a security risk because data is not yet encrpyted. Please refer to the help output `./collector.sh --help`. When using specify interface and port, like so `<interface>:<port>`, e.g. `127.0.0.1:6000`.

```{admonition} locahost does not work
:class: caution

`localhost` is not a valid interface, please use `127.0.0.1` if wanting to bind locally only. A local bind will not allow outide of the machine access, perfect for setting up a proxy with TLS and basic authentication infront of it.
```