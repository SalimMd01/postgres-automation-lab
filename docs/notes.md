# Learning Log

## Week 0 - Setup
- Windows 10 Home 22H2, WSL2 Ubuntu moved to D:\WSL\Ubuntu
- Docker Engine installed inside Ubuntu (not Docker Desktop)
- Data folders in ~/data (stored on D:)
- Problems: GitHub rate limit on login, waited and retried

## Week 0 - Completed
- VS Code with WSL extension working
- AWS skipped (paid). Terraform will use Docker and PostgreSQL providers

## Week 1 - Monitoring stack

### Architecture
Postgres 16 -> postgres_exporter (9187) -> Prometheus (9090, 7d retention) -> Grafana (3000)
- Containers talk by service name (prometheus:9090), not localhost
- Data folders are mounted as volumes so data survives restarts

### Concepts
- Gauge: goes up and down. Example: pg_stat_database_numbackends
- Counter: only increases. Example: pg_stat_database_xact_commit
- rate() turns a counter into a per-second value
- Cache hit ratio = blks_hit / (blks_hit + blks_read). Healthy OLTP is above 0.99
- My ratio was 0.9837: pgbench -i had just loaded about 1M rows into a cold cache, and shared_buffers is only 128 MiB
- The ratio is cumulative since the last stats reset. rate() over a window shows the recent ratio

### PromQL queries I ran
- pg_up -> 1
- pg_stat_database_numbackends{datname="labdb"} -> 2
- rate(pg_stat_database_xact_commit{datname="labdb"}[5m]) -> about 1.96 commits/sec idle
- Cache hit ratio -> 0.9837

### Dashboard 9628 panels
| Panel | Source | Bad value |
|---|---|---|
| Active sessions | pg_stat_activity | Close to max_connections (100) |
| Transactions | pg_stat_database | Rollbacks rising sharply |
| Update data | pg_stat_database (tup_updated) | Unexplained spikes |
| Max Connections | pg_settings | Too low, or sessions near it |
| Shared Buffers | pg_settings | 128 MiB default with low cache hit ratio |

### Problems and fixes
- GitHub "too many requests": waited, retried once
- code . not found: installed VS Code on Windows plus WSL extension
- Grafana Start Time panel N/A: known issue, ignored
- Ubuntu hang, wsl --shutdown gave CO_E_SERVER_EXEC_FAILURE: ran net start LxssManager as admin

### Interview lines
- Monitoring chain: Postgres stats views, exporter, Prometheus scrape, Grafana dashboards
- Cache hit ratio from blks_hit and blks_read, expected above 99% for OLTP
