# Services

Definitions for all services and applications running in the hosts.

## `common`

Services deployed across two or more hosts.
- Databases
- Message brokers
- Proxies
- Telemetry services

Each service here *could* potentially be shared across hosts, but not
necessarily. It is upon each host's configuration to decide which services to
*include* from `common`.
