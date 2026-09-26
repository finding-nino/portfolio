# PHPStorm Database Browser Setup

This connects PHPStorm's built-in **Database** tool window to the same database your Symfony app uses, so you can browse tables, run queries, and get SQL autocomplete inside Doctrine repository code.

## 0. Prerequisites

- PHPStorm Ultimate (the Database tool is not in the free/Community edition).
- A locally installed and running database server. On macOS with Homebrew:

```bash
brew install mysql
brew services start mysql
```

(Swap `mysql` for `postgresql@16` if your project uses PostgreSQL instead — commands below adjust accordingly.)

Then create the database and user your app will use:

```bash
mysql -u root
```

(Homebrew's MySQL has no root password set by default on a fresh install — if `-u root` alone doesn't work, try `mysql -u root -p` and hit enter for an empty password.)

```sql
CREATE DATABASE app_db;
CREATE USER 'app'@'localhost' IDENTIFIED BY 'app';
GRANT ALL PRIVILEGES ON app_db.* TO 'app'@'localhost';
FLUSH PRIVILEGES;
```

- Your `.env` `DATABASE_URL` matching that database, e.g.:

```
DATABASE_URL="mysql://app:app@127.0.0.1:3306/app_db?serverVersion=8.0.32&charset=utf8mb4"
```

## 1. Add the data source

1. Open the tool window: **View → Tool Windows → Database** (or click the small "Database" tab on the right edge of the IDE).
2. Click **+ → Data Source →** pick your driver (MySQL, PostgreSQL, SQLite, etc. — match whatever's in your `DATABASE_URL`).
3. Fill in:
   - **Host**: `127.0.0.1`
   - **Port**: `3306` (MySQL default) / `5432` (PostgreSQL default)
   - **User** / **Password**: from your `DATABASE_URL`
   - **Database**: the DB name from your `DATABASE_URL`
4. If PHPStorm shows "Download missing driver files" at the bottom of the dialog, click it.
5. Click **Test Connection** — should show a green check.
6. Click **OK**.

Your tables now appear in the Database tool window, and you get SQL autocomplete in `.sql` files, in Doctrine query strings, and in the **Database Console** (right-click a data source → **New → Query Console**).

## 2. Keep credentials out of git, but share the connection setup

PHPStorm splits the data source into two files:

- `.idea/dataSources.xml` — the connection **structure** (driver, host, port, DB name). No password. **Safe to commit** — this is what makes the setup reusable across the team/template.
- `.idea/dataSources.local.xml` — your actual **credentials**. Must **not** be committed.

Add these to `.gitignore` if they aren't already covered:

```gitignore
# PHPStorm
.idea/workspace.xml
.idea/dataSources.local.xml
.idea/dataSources/
```

With `dataSources.xml` committed, anyone who clones the repo (or a project generated from this template) opens PHPStorm and already sees the data source defined — they just need to add their own password and hit **Test Connection**.

## 3. Optional but useful

- **Introspect entities as SQL**: right-click a table → **Diagrams → Show Visualization** to see relationships.
- **Doctrine integration**: PHPStorm can resolve entity ↔ table mappings once the data source is attached — go to **Settings → Languages & Frameworks → PHP → Symfony** and make sure Symfony support is enabled, then **Settings → Languages & Frameworks → PHP → Doctrine** to point it at the same connection for entity autocomplete.
- **Run migrations from the DB console**: you can drag `.sql` files (e.g. generated Doctrine migrations) straight into the Query Console to preview/run them against the connected database.

## Troubleshooting

- **"Test Connection" fails with "Unknown database"**: the database itself doesn't exist yet. Create it first, e.g. `bin/console doctrine:database:create`.
- **Driver download fails / no internet in a sandboxed environment**: manually download the JDBC driver `.jar` and point PHPStorm to it via **Data Source Properties → Driver → Go to Driver Manager**.
- **Connection refused**: confirm the DB service is actually running with `brew services list` (look for `mysql`/`postgresql` as `started`), and that nothing else is already bound to the same port — `lsof -i :3306` will show what's using it.
