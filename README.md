# tokimo-package-pg

Prebuilt portable PostgreSQL + pgvector for embedding in desktop applications.

## Artifacts

Each build produces platform-specific relocatable zip archives:

| Platform | Arch | File |
|---|---|---|
| Windows | x64 (MSVC) | `pg-{version}-windows-x64.zip` |
| macOS | arm64 (Apple Silicon) | `pg-{version}-macos-arm64.zip` |

The zip contains a standalone PostgreSQL installation (`bin/`, `lib/`, `share/`). Unzip anywhere and run — no installer, no system dependencies.

## CI Parameters

Trigger manually from [Actions](https://github.com/tokimo-lab/tokimo-package-pg/actions):

| Parameter | Default | Description |
|---|---|---|
| `pg_version` | `REL_18_3` | PostgreSQL source git tag |
| `pgvector_version` | `v0.8.2` | pgvector source git tag |
| `ssl` | `false` | Build with OpenSSL support |

Each build auto-generates a `pg{ver}-vec{ver}` git tag and publishes a Release.

Every push to `master` triggers a CI build (nightly).

## Verified

CI runs these checks per platform, per build:

1. `initdb` — initialize a database cluster
2. `pg_ctl start` — start PostgreSQL
3. `CREATE EXTENSION vector` — load pgvector
4. Vector insert, L2 distance query, IVFFlat index creation
5. `pg_ctl stop` — clean shutdown

## Tauri Integration

```
tauri-app/
  src-tauri/
    binaries/
      pg-windows-x64/    ← unzip here
        bin/postgres.exe
        bin/initdb.exe
        lib/
        share/
```

Configure as an external binary in `tauri.conf.json` and invoke `initdb`, `postgres`, etc. via `Command::new_sidecar`.

## Local Build

```bash
# Requires: meson, ninja, C compiler

# Build PostgreSQL
git clone --depth 1 --branch REL_18_3 https://github.com/postgres/postgres.git
cd postgres
meson setup build --prefix=/path/to/install --buildtype=release -Dssl=none
ninja -C build
ninja -C build install

# Build pgvector
git clone --depth 1 --branch v0.8.2 https://github.com/pgvector/pgvector.git
cd pgvector
export PATH=/path/to/install/bin:$PATH
make
make install
```
