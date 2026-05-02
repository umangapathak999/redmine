# Redmine Docker Setup

This repository contains a Docker Compose setup for running [Redmine](https://www.redmine.org/) with PostgreSQL.

The stack includes:

- Redmine `6.0`
- PostgreSQL `16`
- Persistent Redmine uploads in `./files`
- Local plugin support in `./plugins`
- Local theme support in `./themes`
- Included `bleuclair` theme under `themes/bleuclair`

## Requirements

- Docker
- Docker Compose

## Project Structure

```text
.
|-- docker-compose.yml   # Redmine and PostgreSQL services
|-- env.example          # Example environment configuration
|-- .env                 # Local environment values, ignored by Git
|-- files/               # Redmine uploaded files
|-- plugins/             # Redmine plugins
|-- themes/              # Redmine themes
`-- backup/              # Suggested location for manual backups
```

## Setup

Create a local environment file:

```powershell
Copy-Item env.example .env
```

Edit `.env` and replace the default secrets:

```env
POSTGRES_PASSWORD=change_this_database_password
REDMINE_SECRET_KEY_BASE=change_this_to_a_very_long_random_secret_key_for_redmine
```

You can generate a strong Redmine secret with PowerShell:

```powershell
[guid]::NewGuid().ToString("N") + [guid]::NewGuid().ToString("N")
```

## Run

Start Redmine and PostgreSQL:

```powershell
docker compose up -d
```

Open Redmine in your browser:

```text
http://localhost:3000
```

The port is controlled by `REDMINE_PORT` in `.env`.

Default Redmine login:

```text
Username: admin
Password: admin
```

Change the admin password after the first login.

## Stop

Stop the containers:

```powershell
docker compose down
```

Stop the containers and remove the PostgreSQL volume:

```powershell
docker compose down -v
```

Use `-v` carefully because it deletes the database volume.

## Data Persistence

PostgreSQL data is stored in the Docker volume:

```text
redmine-db-data
```

Redmine files, plugins, and themes are mounted from local folders:

```text
./files   -> /usr/src/redmine/files
./plugins -> /usr/src/redmine/plugins
./themes  -> /usr/src/redmine/themes
```

## Plugins

Place Redmine plugins inside `plugins/`, then restart Redmine:

```powershell
docker compose restart redmine
```

Some plugins require migrations:

```powershell
docker compose exec redmine bundle exec rake redmine:plugins:migrate RAILS_ENV=production
docker compose restart redmine
```

## Themes

Place Redmine themes inside `themes/`.

The repository currently includes the `bleuclair` theme. To enable a theme:

1. Log in as an administrator.
2. Go to `Administration -> Settings -> Display`.
3. Select the theme from the `Theme` dropdown.
4. Save changes.

## Backup

Create a PostgreSQL backup:

```powershell
docker compose exec -T redmine-db pg_dump -U redmine redmine > backup/redmine.sql
```

Back up uploaded files separately:

```powershell
Compress-Archive -Path files -DestinationPath backup/redmine-files.zip
```

If you changed `POSTGRES_USER` or `POSTGRES_DB` in `.env`, use those values in the `pg_dump` command.

## Restore

Restore a database backup into a running stack:

```powershell
Get-Content backup/redmine.sql | docker compose exec -T redmine-db psql -U redmine redmine
```

Restore uploaded files by extracting the archive back into `files/`.

## Logs

View all logs:

```powershell
docker compose logs -f
```

View Redmine logs only:

```powershell
docker compose logs -f redmine
```

View database logs only:

```powershell
docker compose logs -f redmine-db
```

## Upgrade

To upgrade Redmine, change the image tag in `docker-compose.yml`, then pull and restart:

```powershell
docker compose pull
docker compose up -d
```

Create a database and files backup before upgrading.
