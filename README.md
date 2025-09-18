# Ask-My-DB (Laravel App)

Query your database using natural language via LLMs. This Laravel app integrates a local package (`packages/askmydb/laravel-askmydb`) that converts plain-English prompts into safe SQL SELECT statements, executes them against your configured database, and shows the results with optional charts.

## Background and attribution
- Inspired by the original Python project: [Ask-My-DB (Python)](https://github.com/Msalways/Ask-My-DB)
- Sample data: Chinook database (SQLite): [lerocha/chinook-database](https://github.com/lerocha/chinook-database/tree/master)
- This is a fresh Laravel 12 implementation with OpenAI-compatible and Ollama providers and a simple demo UI.

## Features
- Natural language to SQL via LLM
- Safe by default: only SELECTs, with enforced LIMIT
- Database-agnostic (Laravel connections); demo configured for SQLite Chinook
- Schema introspection with limits to stay within LLM context
- Demo UI at `/askmydb` with SQL preview, JSON, and charts (Bar, Line, Area, Pie, Donut, Scatter)
- Auto-render charts with axis selectors

## Requirements
- PHP ^8.2, Laravel ^12
- Composer, Node (only if you build your own frontend assets)

## Installation
```bash
composer install
cp .env.example .env  # if you don't have one
php artisan key:generate
```

## Database (Chinook sample)
- Place `Chinook_Sqlite.sqlite` in the project root (already present).
- Configure SQLite in `.env` using a relative path:
```env
DB_CONNECTION=sqlite
DB_DATABASE=Chinook_Sqlite.sqlite
```
- The app resolves relative paths against the project root.
- Clear caches after changes:
```bash
php artisan config:clear && php artisan optimize:clear
```

Sessions/cache
- For a read-only DB like Chinook, file drivers are used:
```env
SESSION_DRIVER=file
CACHE_DRIVER=file
```

## Ask-My-DB package
- Local package path: `packages/askmydb/laravel-askmydb`
- Service provider, routes, controllers, and Blade view are auto-registered via the package.

### LLM provider configuration
Set one provider in `.env`:
```env
ASKMYDB_PROVIDER=openai   # or ollama, or dummy

# OpenAI-compatible (OpenAI or OpenRouter)
OPENAI_API_KEY=YOUR_KEY
# For OpenAI: https://api.openai.com/v1
# For OpenRouter: https://openrouter.ai/api/v1
OPENAI_BASE_URL=https://openrouter.ai/api/v1
OPENAI_MODEL=openai/gpt-4o-mini
OPENAI_TEMPERATURE=0.2

# Ollama (local)
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=llama3.1

# Introspection limits (optional)
ASKMYDB_MAX_TABLES=50
ASKMYDB_MAX_COLUMNS=60
```
Notes:
- Temperature must be numeric (e.g., `0.2`).
- To use a different DB connection, set `ASKMYDB_CONNECTION` to a Laravel connection name; otherwise it uses the default.

## Usage
- Visit: `/askmydb`
- Enter prompts like:
  - "List the 10 latest invoices with total and customer name"
  - "From Invoice and InvoiceLine, monthly totals for 2024 and 2023"
- The UI shows generated SQL and JSON results, and auto-renders charts.

### Charts
- Chart types: Bar, Line, Area (line+fill), Pie, Donut, Scatter
- Select X axis and one or more Y axes; chart updates automatically
- Pie/Donut use the first selected Y series; Scatter requires two Y series

## Troubleshooting
- Only README appears after pushing the package repo
  - You likely created the repo with an initial README. Do `git push --force-with-lease origin main` from the package directory, or pull with `--allow-unrelated-histories` then push.
- OpenRouter 400 "Expected number, received string"
  - Ensure `OPENAI_TEMPERATURE` is numeric; clear config cache.
- Context length exceeded
  - Lower `ASKMYDB_MAX_TABLES` / `ASKMYDB_MAX_COLUMNS` or choose a larger-context model.
- Model generates invalid SQL (e.g., missing FROM)
  - The app sanitizes and falls back to a harmless query. Provide concrete table names from `/askmydb/schema.json` to guide the model.
- Session table missing (SQLite)
  - Sessions use `file` driver in `.env`. Clear cookies and retry if needed.

## Security
- The package sanitizes SQL to allow only SELECT and injects a LIMIT when missing.
- Prefer read-only DB users/replicas for safety.

## Development
- Clear caches when tweaking `.env`:
```bash
php artisan config:clear && php artisan optimize:clear
```
- Quick tinker check:
```bash
php artisan tinker --execute="dump(AskMyDB\\Laravel\\Facades\\AskMyDB::ask('list 5 invoices with total and date'));"
```

## License
- MIT for this app and package
- Chinook database licensing per upstream repository
