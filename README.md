# Epmdless

If you run distributed services such like pubsubs or message brokers between datacenters with EPMD, all your nodenames and ports are exposed over the internet, and you've probably got more ports open in your firewalls than actual services running behind them. Yuck.

Don't you wish you could just decide on a port, append it to the nodename and ditch EPMD altogether?

## Elixir

    # mix.exs
    {:epmdless, "> 0.0.0"}

    # flags (put in --erl "..." when running with iex)
    -pa _build/(dev|prod)/lib/epmdless/ebin \
    -proto_dist epmdless_(inet|inet6)_(tcp|tls)_dist \
    -no_epmd

## Erlang (Hex)

    # rebar.config
    {deps, [epmdless]}.
    {plugins, [rebar3_hex]}.

    # flags
    -pa _build/default/lib/epmdless/ebin
    -proto_dist epmdless_(inet|inet6)_(tcp|tls)_dist
    -no_epmd

## Build & Test Locally

    $ rebar3 compile
    $ epmd -kill

    $ erl --name mynode@0.0.0.0:8080 \
    -pa _build/default/lib/epmdless/ebin \
    -proto_dist epmdless_inet_tcp \
    -no_epmd

    $ erl --name mynode@::1:8080 \
    -pa _build/default/lib/epmdless/ebin \
    -proto_dist epmdless_inet6_tcp \
    -no_epmd

    $ erl --name mynode@0.0.0.0:8080 \
    -pa _build/default/lib/epmdless/ebin \
    -proto_dist epmdless_inet_tls \
    -ssl_dist_opt server_certfile cert.pem \
    -ssl_dist_opt server_secure_renegotiate true client_secure_renegotiate true \
    -no_epmd

    $ erl --name mynode@::1:8080 \
    -pa _build/default/lib/epmdless/ebin \
    -proto_dist epmdless_inet6_tls \
    -ssl_dist_opt server_certfile cert.pem \
    -ssl_dist_opt server_secure_renegotiate true client_secure_renegotiate true \
    -no_epmd
