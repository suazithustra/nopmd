# nopmd (No Port Mapper Daemon)

If you run distributed services such like pubsubs or message brokers between regions with EPMD, all your nodenames and ports are exposed over the internet, and you've probably got more ports open in your firewalls than actual services running behind them. Yuck.

Don't you wish you could just decide on a port, append it to the nodename and call it a day?

## Elixir

    # mix.exs
    {:nopmd, "> 0.0.0"}

    # nodename example (--sname not supported, obviously)
    --name mynode@127.0.0.1:8080

    # ipv6 nodename example
    --name mynode@::1:8080

    # erl flags (put in --erl "..." when running with iex)
    -pa _build/(dev|prod)/lib/nopmd/ebin \
    -proto_dist nopmd_(inet|inet6)_(tcp|tls)_dist \
    -no_epmd

## Erlang (Hex)

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
    -no_epmd

## Build & Test Locally

    $ rebar3 compile
    $ epmd -kill

    $ erl -name mynode@127.0.0.1:8080 \
    -pa _build/default/lib/nopmd/ebin \
    -proto_dist nopmd_inet_tcp \
    -no_epmd

    $ erl -name mynode@::1:8080 \
    -pa _build/default/lib/nopmd/ebin \
    -proto_dist nopmd_inet6_tcp \
    -no_epmd

    $ erl -name mynode@127.0.0.1:8080 \
    -pa _build/default/lib/nopmd/ebin \
    -proto_dist nopmd_inet_tls \
    -ssl_dist_opt server_certfile cert.pem \
    -ssl_dist_opt server_secure_renegotiate true client_secure_renegotiate true \
    -no_epmd

    $ erl -name mynode@::1:8080 \
    -pa _build/default/lib/nopmd/ebin \
    -proto_dist nopmd_inet6_tls \
    -ssl_dist_opt server_certfile cert.pem \
    -ssl_dist_opt server_secure_renegotiate true client_secure_renegotiate true \
    -no_epmd
