# Copilot instructions for postgress-eda

Short, actionable guidance for AI coding agents working in this repo.

## Big picture
- Purpose: This repo holds small Ansible artifacts for Event-Driven Ansible (EDA) workflows that react to PostgreSQL NOTIFY channels and trigger AWX/Tower job templates or playbooks.
- Key components:
  - `rulebooks/postgress.yml`: an EDA rulebook that uses `ansible.eda.pg_listener` to subscribe to Postgres channels and run `run_job_template` actions.
  - `high_cpu.yml`: a simple Ansible playbook used by the rulebook's job template (example runner/playbook).

## Data flow / service boundaries
- `ansible.eda.pg_listener` connects to PostgreSQL using `postgres_params` and listens on `channels` (e.g., `ops_alerts`).
- When a rule condition matches, it invokes an action such as `run_job_template` which delegates work to AWX/Tower (job template `Investigate_High_CPU` in `Default` org in the example).

Example snippet (from `rulebooks/postgress.yml`):
```yaml
sources:
  - ansible.eda.pg_listener:
      postgres_params:
        host: 192.168.64.41
        port: 5432
        dbname: awx
        user: postgres
        password: primod123
      channels:
        - ops_alerts

rules:
  - name: R1 - High CPU usage detected
    condition: true
    action:
      run_job_template:
        name: Investigate_High_CPU
        organization: Default
```

## Developer workflows (commands)
- Run the example playbook locally (Ansible):
  - `ansible-playbook high_cpu.yml`
- Start the EDA rulebook runner against the rulebook file:
  - `ansible-rulebook rulebooks/postgress.yml`
  - Add `-vv` for verbose logging while debugging.

## Repo-specific patterns & conventions
- Rulebooks are structured as EDA documents (list of rules with `sources`, `rules`, `action` blocks). Look for `sources: - ansible.eda.*` and `action: run_job_template` to find integration points.
- Hosts are frequently `localhost` because the runner/service interacts with external systems (Postgres, AWX) rather than remote Nodes.
- Credentials and sensitive parameters may appear inline in example files (e.g., `password` in `rulebooks/postgress.yml`). Treat these as examples only — follow project secret management conventions (Vault or env vars) when editing.

## Integration points and external dependencies
- PostgreSQL NOTIFY channels (e.g., `ops_alerts`) — the `pg_listener` source subscribes here.
- AWX/Tower job templates — the rulebook triggers job templates via the `run_job_template` action (ensure AWX collections and authentication are available in the runtime environment).
- Ansible collections required at runtime: `ansible.eda`, AWX/collection for job template execution.

## What to look for when changing behavior
- If adding new triggers, mirror the `sources` pattern in `rulebooks/*` and wire the correct `channels` and `postgres_params`.
- Avoid committing real credentials — replace with placeholders or use Ansible Vault/lookup for secrets.
- For debugging, increase verbosity on the runner and inspect Postgres notification traffic.

## Files to inspect first
- [rulebooks/postgress.yml](../rulebooks/postgress.yml)
- [high_cpu.yml](../high_cpu.yml)

If anything here is unclear or you'd like these instructions to include more examples (e.g., AWX auth setup, required collections, or a validated run example), tell me which area to expand.
