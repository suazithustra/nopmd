# nopmd (no-port-mapper-daemon)

**NOTE:** Tested only on OTP 23+

If you run distributed services such like pubsubs or message brokers between regions with EPMD, all your nodenames and ports are either exposed over the internet or you've written scripts to forward ports via ssh, and you've probably got more ports open in your firewalls than actual services running behind them. Yuck.

Don't you wish you could just decide on a port, append it to the nodename and call it a day?

## Elixir

```shell
# mix.exs
{:nopmd, "> 0.0.0"}

# nodename example (--sname not supported, obviously)
--name mynode@127.0.0.1:8080

# ipv6 nodename example
--name mynode@::1:8080

# erl flags (go in --erl "...")
-pa _build/(dev|prod)/lib/nopmd/ebin \
-proto_dist nopmd_(inet|inet6)_(tcp|tls)_dist \
-start_epmd false
```

### Elixir Releases

Uncomment `RELEASE_DISTRIBUTION` and `RELEASE_NODE` in `env.sh.eex`, then add the following and modify as needed:

```shell
export RELEASE_PORT=${RELEASE_PORT:-"8080"}
export RELEASE_AUX_PORT=${RELEASE_AUX_PORT:-"8000"}
export RELEASE_PROTO_DIST=nopmd_inet_tcp
export ELIXIR_ERL_OPTIONS="-pa lib/nopmd-0.1.0/ebin -proto_dist $RELEASE_PROTO_DIST -start_epmd false"

case $RELEASE_COMMAND in
  start*|daemon*)
    export RELEASE_NODE=$RELEASE_NODE:$RELEASE_PORT
    ;;
  *)
  export ELIXIR_ERL_OPTIONS="$ELIXIR_ERL_OPTIONS -name $RELEASE_NODE:$RELEASE_AUX_PORT -kernel logger_level error"
  export RELEASE_NODE=$RELEASE_NODE:$RELEASE_PORT
    ;;
esac
```

**NOTE:** When using ssl/tls, you must add all your ssl dist opts to the `ELIXIR_ERL_OPTIONS` line in this example.

### Distillery Releases

Make all the necessary modifications to `vm.args`. Since Distillery doesn't provide any way to add flags when instantiating commands, the only command that will work is `start`, if you want to stop, rpc remsh, etc, you will have to write scripts for them manually and use overlays to overwrite the existing scripts for those commands in `libexec/commands/` (or just use elixir releases).

**NOTE:** Distillery doesn't seem to care about distribution encryption 😕, if you're using encryption, you must add `:ssl` to `extra_applications` in `mix.exs` and include all relevent ssl opts in your overlays.

## Erlang (Hex)

```shell
# rebar.config
{deps, [nopmd]}.
{plugins, [rebar3_hex]}.

# nodename example (-sname not supported, obviously)
-name mynode@127.0.0.1:8080

# ipv6 nodename example
-name mynode@::1:8080

# flags
-pa _build/default/lib/nopmd/ebin
-proto_dist nopmd_(inet|inet6)_(tcp|tls)_dist
-start_epmd false
```

### Relx Releases

Add `-name ${NODE_NAME}` to `vm.args.src`. The following commands will have to be used as such:

```shell
NODE_NAME=myapp@127.0.0.1:8080 \_build/default/rel/myapp/bin/myapp daemon
NODE_NAME=myapp@127.0.0.1 ERL_DIST_PORT=8080 \_build/default/rel/myapp/bin/myapp pid
NODE_NAME=myapp@127.0.0.1 ERL_DIST_PORT=8080 \_build/default/rel/myapp/bin/myapp stop
```

Remsh will have to be done manually or the generated `mynode` script modified.

## Build & Test Locally

```shell
$ rebar3 compile
$ epmd -kill

$ erl -name mynode@127.0.0.1:8080 \
-pa _build/default/lib/nopmd/ebin \
-proto_dist nopmd_inet_tcp \
-start_epmd false

$ erl -name mynode@::1:8080 \
-pa _build/default/lib/nopmd/ebin \
-proto_dist nopmd_inet6_tcp \
-start_epmd false

$ erl -name mynode@127.0.0.1:8080 \
-pa _build/default/lib/nopmd/ebin \
-proto_dist nopmd_inet_tls \
-ssl_dist_opt server_certfile cert.pem \
-start_epmd false

$ erl -name mynode@::1:8080 \
-pa _build/default/lib/nopmd/ebin \
-proto_dist nopmd_inet6_tls \
-ssl_dist_opt server_certfile cert.pem \
-start_epmd false
```
