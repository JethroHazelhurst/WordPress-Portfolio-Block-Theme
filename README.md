# Portfolio WordPress starter

Local development with Docker Desktop, WSL2 and VS Code. WordPress 7.1.2 / PHP 8.4 / MySQL 8.4. The block theme uses theme.json version 3 and needs no Node build step or custom PHP.

## Start

1. Enable Docker Desktop's WSL2 engine and integration for your Linux distribution.
2. Keep this project in the WSL Linux filesystem (`~/projects/portfolio`), rather than `/mnt/c`.
3. From a WSL terminal, open the project with `code .`. VS Code should show a WSL connection.
4. If `.env` does not exist, run `cp .env.example .env`. Set a local database password before the first startup. An existing `.env` is left untouched.
5. Run `docker compose up -d --wait`.
6. Open http://localhost:8080 in Windows, finish the WordPress installer, and choose your own administrator credentials. The database credentials are separate from the administrator account.
7. In Appearance → Themes, activate **Portfolio**.
8. Create a Home page with portfolio blocks; set it as the homepage under Settings → Reading. Create About and Contact pages and add them to Navigation in Appearance → Editor.

## Edit

Edit `wp-content/themes/portfolio/` in VS Code. The directory is bind mounted into WordPress, so saved files are available immediately. `theme.json` controls editor and frontend design; `templates/*.html` contain block markup; `parts/` contains the header and footer. `style.css` supplies the theme metadata; custom CSS can be added through theme.json or enqueued later.

Templates and styles saved in Appearance → Editor live in the database and may override the theme files. Export the theme from the Site Editor and copy reviewed changes into this folder to track them in Git. Reset a customized template in the editor when you want its file version to take effect. Pages, navigation and media are content and are not captured by a Git commit.

WordPress core, installed plugins and uploads persist in `wordpress_data`; MySQL persists in `db_data`. Install plugins in the admin UI; bind mount a separate source directory if you later develop a custom plugin. Do not recursively change ownership of your theme to the container user: keep source files owned by your WSL account.

## Useful commands

```bash
docker compose ps
docker compose logs --tail=100 wordpress db
docker compose exec wordpress tail -n 100 wp-content/debug.log
docker compose down
docker compose up -d --wait
```

Debugging is enabled locally, with messages logged rather than displayed. `debug.log` appears after the first logged event. To disable debugging, set WORDPRESS_DEBUG to an empty value in `.env` (the official image treats a nonempty `0` as enabled), and recreate the container.

`docker compose down` preserves data. **`docker compose down -v` deletes the database, uploads, plugins and installation volume.** Back up the database and uploads separately before resets or migrations. Changing `.env` database passwords after initialization does not change existing MySQL user passwords.

This Compose file is for local development: HTTP is bound to localhost, the database has no published port, and development logging is enabled. Production needs its own hosting, TLS and backup configuration.

## Git

Yes: version the theme, Compose file, README, `.env.example` and editor configuration. `.env`, logs and database exports are ignored. Initialize from WSL:

```bash
git init -b main
git add .
git diff --cached
git commit -m "Add WordPress Docker and portfolio block theme starter"
```

Never commit real credentials or rely on Git for WordPress content backups.

## Updates

The WordPress image is pinned to a verified patch release; MySQL tracks the 8.4 maintenance line. Review new official tags and deliberately update the WordPress pin. Pull and recreate after backing up:

```bash
docker compose pull
docker compose up -d --wait
```

The WordPress volume retains installed core across container replacement, so replacing the image alone does not upgrade existing core. Apply core security updates through WordPress as well; keep themes and plugins current.

References: [official WordPress Docker image](https://hub.docker.com/_/wordpress), [block theme structure](https://developer.wordpress.org/themes/core-concepts/theme-structure/), [theme.json reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/theme-json-living/), [Docker WSL practices](https://docs.docker.com/desktop/features/wsl/best-practices/).
