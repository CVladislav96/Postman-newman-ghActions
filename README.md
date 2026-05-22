# Postman + Newman + GitHub Actions

Automated API tests for the [Swagger Petstore](https://petstore.swagger.io/) Postman collection, run with [Newman](https://github.com/postmanlabs/newman) in GitHub Actions, with HTML reports published to GitHub Pages.

## Repository layout

```
.
├── .github/workflows/newman.yml   # CI: Newman run and report deploy
├── postman/
│   ├── PetStore.postman_collection.json
│   └── PetStore_Environment.postman_environment.json
└── report/                        # generated locally (in .gitignore)
```

## Collection

| Section | Requests | Description |
|---------|----------|-------------|
| **Pet** | 9 | Pet CRUD, find by status, negative 404 cases |
| **Store** | 6 | Inventory and orders |
| **User** | 8 | Sign-up, login, profile, delete |

Base URL is set in the environment: `https://petstore.swagger.io` (`baseUrl` variable).

## Run locally

Requires [Node.js](https://nodejs.org/) 18+ and npm.

```bash
# Install Newman and the HTML reporter (one-time)
npm install -g newman newman-reporter-htmlextra

# Run the collection
newman run postman/PetStore.postman_collection.json \
  -e postman/PetStore_Environment.postman_environment.json \
  --insecure
```

Without a global install:

```bash
npx newman@6 run postman/PetStore.postman_collection.json \
  -e postman/PetStore_Environment.postman_environment.json \
  --insecure
```

HTML report:

```bash
newman run postman/PetStore.postman_collection.json \
  -e postman/PetStore_Environment.postman_environment.json \
  -r htmlextra \
  --reporter-htmlextra-export ./report/index.html \
  --insecure
```

The `--insecure` flag disables SSL verification (same as in CI). It is not required for the public Petstore API but kept for parity with the workflow.

## CI/CD (GitHub Actions)

Workflow: [`.github/workflows/newman.yml`](.github/workflows/newman.yml)

- **Triggers:** push to `main`, manual run (`workflow_dispatch`)
- **Steps:** checkout → Node 20 → install Newman → run tests → deploy `./report` to GitHub Pages (`peaceiris/actions-gh-pages`)

If tests fail, the job is marked as failed, but the report deploy step still runs (`if: always()`) so you can open the HTML report for details.

### GitHub Pages setup (one-time)

1. Repository → **Settings** → **Pages**
2. **Source:** Deploy from a branch → branch `gh-pages` → `/ (root)`
3. After the first successful workflow, the report is available at:  
   `https://<username>.github.io/Postman-newman-ghActions/`

The workflow sets `permissions: contents: write` so `GITHUB_TOKEN` can push to the `gh-pages` branch.

## Import into Postman

1. **Import** → select `postman/PetStore.postman_collection.json` and `postman/PetStore_Environment.postman_environment.json`
2. Select the **PetStore Environment** in the top-right environment dropdown
3. Run the collection via **Runner** or individual requests

## API notes

- Petstore is a demo service with shared data; request order in the collection matters (e.g. create pet with `id: 1000` before GET/DELETE).
- Some negative scenarios rely on fixed IDs (`1000`, `999999999`, order `9`, etc.).
- API `message` fields may differ in casing (e.g. `Order Not Found`); tests compare messages case-insensitively where needed.

## Repository

- Remote: `https://github.com/CVladislav96/Postman-newman-ghActions.git`
- Default branch: `main`
