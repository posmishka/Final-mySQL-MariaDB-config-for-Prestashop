## Optimized MariaDB 10.11 Config for PrestaShop 1.7 (8GB RAM)

This repository contains a battle-tested MariaDB configuration file (`50-server.cnf`) specifically optimized for **PrestaShop 1.7** running heavy batch imports (like 1C synchronization) on modern MariaDB engines.

## 🖥️ Target Environment
* **OS:** Ubuntu 22.04 LTS
* **Database:** MariaDB 10.11.x
* **Hardware:** 2 CPU Cores / 8 GB RAM
* **Application:** PrestaShop 1.7.x with heavy XML/1C product imports

## 🛑 The Problems This Solves
1. **Linux OOM (Out of Memory) Crashes:** Default or overtuned `innodb_buffer_pool_size` values cause the server to run out of RAM during heavy PHP/DB spikes, resulting in the Linux Kernel terminating MariaDB (`mariadbd`) mid-import.
2. **"Got an error reading communication packets" / Timeout Drops:** PHP scripts dropping database connections because massive unindexed JOINs exhaust the database thread buffers.
3. **Frontend Freezes (The `rowid_filter` bug):** MariaDB 10.6+ introduced a new query optimizer that notoriously conflicts with PrestaShop 1.7's complex category and product JOINs, causing massive full table scans and 100% CPU usage on the storefront.

## 🛠️ Key Optimizations Included
* **OOM Prevention:** Caps the `innodb_buffer_pool_size` at `3584M` (3.5 GB). This reserves 50% of the server's 8 GB RAM for PHP-FPM, the OS, and MariaDB per-thread connections.
* **Bulk Import Acceleration:** Sets `innodb_flush_log_at_trx_commit = 2`, providing a massive speedup for `INSERT`/`UPDATE` heavy operations like 1C imports while remaining crash-safe.
* **PrestaShop 1.7 Optimizer Fix:** Disables the problematic MariaDB row ID filter (`optimizer_switch = 'rowid_filter=off'`) to restore instant load times on storefront category pages.
* **Frontend Speed:** Re-enables the Query Cache (`query_cache_type = 1`), which PrestaShop 1.7 relies on heavily to prevent recalculating hooks and theme modules on every page load.
* **Cache Eviction Fixes:** Bumps table caches to `2000` to cover all 1,100+ standard PrestaShop tables, preventing heavy disk I/O.

## 🚀 How to Apply
1. Back up your existing configuration file:
   ```bash
   sudo cp /etc/mysql/mariadb.conf.d/50-server.cnf /etc/mysql/mariadb.conf.d/50-server.cnf.backup
Replace the contents of /etc/mysql/mariadb.conf.d/50-server.cnf with the configuration below.

Restart MariaDB to apply the changes safely:

```Bash
sudo systemctl restart mariadb
Verify the database is running smoothly:

```Bash
sudo systemctl status mariadb
📝 Additional Recommendations for 1C Imports
To prevent PHP from severing the connection before the database finishes importing your products, ensure your php.ini (e.g., /etc/php/8.1/fpm/php.ini) has adequate timeouts:

```Ini, TOML
max_execution_time = 3600
max_input_time = 3600
default_socket_timeout = 600
memory_limit = 512M

