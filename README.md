# ARKA TARA shared documentation

This repository versions the single canonical project/business documentation in
[docs/](docs/) and the [shared agent operating guide](AGENTS.md). Start with
[BUSINESS_CONTEXT.md](docs/BUSINESS_CONTEXT.md), then consult
[TASKS.md](docs/TASKS.md) for implementation status and
[DECISIONS.md](docs/DECISIONS.md) for approved and unresolved decisions.

## Repository boundaries

The workspace has three independent Git repositories:

| Location | Contents |
| --- | --- |
| `ARKA TARA/` | This documentation repository: nine canonical documents, `AGENTS.md`, this README, `.gitignore` and `.gitattributes`. |
| `ARKA TARA/arkatara_backend/` | Django/DRF backend and its own Git history/remote. |
| `ARKA TARA/arkatara_frontend/` | Next.js/React frontend and its own Git history/remote. |

The root `.gitignore` allows only the listed documentation/setup files. Both
application directories, `.tools/`, local databases, environment files and other
unlisted files are excluded. The applications are ordinary independent clones;
they are not submodules. Run application Git commands from the corresponding
application directory. Root Git commands operate on shared documentation.

Shared content belongs only in `docs/`. Application `AGENTS.md` files reference it;
application READMEs contain their own setup instructions. Do not duplicate the
shared documents inside either application repository.

## Reconstructing the workspace

The owner published all three repositories. A new developer can use these commands
from a parent directory:

```powershell
git clone https://github.com/AmirtharajMuthukrishnan/arkatara_docs.git "ARKA TARA"
Set-Location "ARKA TARA"
git clone git@github.com:AmirtharajMuthukrishnan/arkatara_backend.git arkatara_backend
git clone git@github.com:AmirtharajMuthukrishnan/arkatara_frontend.git arkatara_frontend
```

This layout makes the applications' `../docs/` and `../AGENTS.md` references work.
A standalone application clone does not include the canonical context. Obtain
the shared repository before work that depends on the business rules. Local
tools and dependencies are installed separately using the application READMEs.

## Documentation changes and review

The initial documentation baseline is committed on `main`; `development` starts
from that baseline. Use focused documentation branches from `development` and
the same review/promotion approach described in
[ARCHITECTURE.md](docs/ARCHITECTURE.md). The documentation remote is
[arkatara_docs](https://github.com/AmirtharajMuthukrishnan/arkatara_docs).

Keep implementation status and decision evidence current. Record owner approvals
without deleting earlier decision history; implementation does not resolve open
business, CA or legal reviews. Reference the companion documentation commit or
pull request in application changes that depend on it. This repository does not
automatically synchronize the two application histories.

Before committing, check `git diff --check` and `git diff --cached --name-only`.
Keep the root tracking list narrow; review any deliberate expansion before adding
new paths. BD-14 records approval of this layout; publication was verified on
2026-09-24.
