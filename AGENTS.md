# AGENTS.md

## Cursor Cloud specific instructions

This is a **pure static website** — no build step, no package manager, no dependencies, no test framework.

### Running the site locally

Serve the production site from the `prod/` directory using any HTTP server:

```
python3 -m http.server 8080 --directory prod
```

Then open `http://localhost:8080/` in a browser. The site contains inline CSS/JS with interactive elements (rotating hero text, tabbed CLI/chat demo, FAQ accordion, smooth-scroll nav).

### Repo layout

See `README.md` for full details. Key paths:
- `prod/` — production site content (deploy target)
- `staging/` — staging variant
- `infra/cloudformation/` — AWS IaC
- `.github/workflows/` — CI/CD pipeline

### No lint/test/build tooling

There are no linters, test runners, or build tools configured. Validation is visual — open the HTML in a browser and inspect.

### Deployment

Production deploys happen via GitHub Actions on push to `main` affecting `prod/**`. No manual deploy steps are needed locally.
