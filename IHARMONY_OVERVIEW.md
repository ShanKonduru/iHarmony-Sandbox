<p align="center">
  <img src="https://raw.githubusercontent.com/ShanInnominds/innoHarmony-assets/main/iHarmony/hero.png" alt="iHarmony™ by Innominds — Unified Quality Engineering Framework" width="100%">
</p>

# iHarmony™ — Project Overview

This project was created by `iharmony init`. It is a ready-to-extend **iHarmony™** test suite:
UI, API, data and device tests on **pytest + Playwright**, sharing one configuration model and one
set of reports. iHarmony™ is engineered by **Innominds**.

> Check the installed framework version with `iharmony --version`, and read any guide offline
> with `iharmony docs <topic>` — the guides ship with the package and always match your version.

---

## What `iharmony init` created

| Path | Purpose |
|---|---|
| `conftest.py`, `pytest.ini` | pytest wiring and marker registration (the framework fixtures load automatically) |
| `config/config.yaml` | Framework defaults — browser, execution, reporting, resilience, quality gate, plugins … |
| `config/environments.yaml` | Base URLs and settings per environment (`dev`, `qa`, `uat`, `prod`, …) |
| `config/projects/<name>.yaml` | Per-project overrides for the multi-project layout |
| `config/org.example/` | Sample org → business-unit → project governance policy with locked keys; inactive until renamed to `config/org/` |
| `config/devices.yaml` | Devices under test, benches and simulated twins |
| `pages/` | Your page objects (`login_page.py` is a working example) |
| `test_data/` | Test data (`${ENV_VAR}` placeholders resolve at load time), JSON schemas, Pact files, a demo SQLite database |
| `tests/<project>/functional/{ui,api,database}/` | Where your tests go — one folder per project, feature and domain |
| `tests/demo/` | Runnable reference tests for UI, API, database, file comparison and devices |

## First run

```bash
# demo login for the public saucedemo.com app (PowerShell: $env:NAME = "value")
export IHARMONY_TEST_USERNAME=standard_user
export IHARMONY_TEST_PASSWORD=secret_sauce

pytest tests/demo/functional/api tests/demo/functional/ui
```

| Demo | Extra it needs |
|---|---|
| `tests/demo/functional/api`, `tests/demo/functional/ui` | none |
| `tests/demo/functional/database` (Excel, SQLite, file comparison) | `pip install "iharmony[data-comparison]"` |
| `tests/demo/functional/database/postgres_src_tgt` | `[postgresql]` and Docker (see its README) |
| `tests/demo/functional/device` (simulated thermostat) | `pip install "iharmony[devices-serial]"` |

## Capabilities and how to switch them on

Everything is opt-in: install the extra, then enable the feature in `config/config.yaml` or on the
command line.

| Capability area | How to use it | Guide |
|---|---|---|
| **Web UI Automation** | Page objects inherit `BasePage` (`from iharmony import BasePage`); Chromium, Firefox, WebKit; `--iharmony-device "iPhone 13"` for mobile emulation; WCAG checks with the `[accessibility]` extra and the `accessibility:` section; visual regression with `assert_visual_snapshot` | `quickstart`, `qe-visual-guide` |
| **Mobile & Desktop Apps** *(preview)* | `[apps]` extra, then `apps.enabled: true` in `config/apps.yaml`; screen objects inherit `BaseScreen` (`from iharmony.apps import BaseScreen, Locator`); `app_session` fixture, `@pytest.mark.mobile` / `desktop` / `app(name)`; Android, iOS, Windows through Appium | `app-automation` |
| **API & Contract Testing** | `ApiClient` (retries, OAuth2, multipart, cookies) + `ResponseValidator`; GraphQL and SOAP via `[graphql-client]` / `[soap-client]`; contracts via `[contracts]` / `[pact]` | `api-testing` |
| **Data & Database Validation** | SQLite built in; PostgreSQL, MySQL, SQL Server, Oracle, DB2, Informix, Snowflake, MongoDB via extras (`[all-databases]`); `compare_excel_files`, `compare_csv_files` and DB-to-file comparisons with key columns, tolerances and column mappings; interactive HTML mismatch reports | `data-validation`, `database` |
| **Device & IoT Testing** | `dut` fixture, `@pytest.mark.device` / `requires_dut`; serial, SSH, Telnet, HTTP, MQTT, CoAP, Modbus, CAN, BLE, ADB; twins (`--iharmony-dut-mode sim`) or hardware (`hil`); fault injection, device pool | `iharmony devices catalog` |
| **Test Data Management** | `tdm` fixture: seeded synthetic data, 7 masking strategies, provisioning and reset; `tdm.enabled: true` masks reports and logs | `test-data` |
| **Self-Healing & Resilience** | `resilience: enabled: true` — fallback selector chains, DOM retries, confidence score, recovery audit log; `@pytest.mark.flaky` re-runs | `resilience` |
| **Observability & Reporting** | Automatic failure classification, flaky-test detection, KPI and persona reports; `iharmony observability-report` renders them to HTML | `observability` |
| **Release Quality Gate** | `pytest --iharmony-quality-gate` — PASS / WARN / BLOCK verdict and risk score; exits 1 on BLOCK; `iharmony-mcp get-release-recommendation` | `cli` |
| **Enterprise Governance** | Org → business unit → project policy with locked keys; environment overlays; secrets from environment variables, Vault, AWS or Azure (`[vault]`, `[aws]`, `[azure]`) | `secrets-governance` |
| **Security Built In** | `scripts/run_security_reports.sh` (in the repository): seven dependency, code and secret scans in one consolidated report | `ci` |
| **Scale, CI/CD & Integrations** | `pytest -n auto`; `--iharmony-orchestration` for dependency-ordered suites; plugins with lifecycle hooks; GitHub Actions, Jenkins and Azure DevOps templates | `plugins`, `ci` |
| **AI-Ready (MCP Server)** | `[mcp]` extra, then `iharmony-mcp serve`; 24 tools, each also a CLI command | `mcp` |

## Everyday commands

| Goal | Command |
|---|---|
| Run one project | `pytest tests/<project>` |
| Run a suite | `pytest -m smoke` · `pytest -m regression` |
| Choose environment and browser | `pytest --iharmony-env qa --iharmony-browser firefox` |
| Watch the browser | `pytest --iharmony-headed` |
| Run in parallel | `pytest -n auto` |
| Check the environment first | `pytest --iharmony-readiness-check` |
| Add projects | `iharmony init . --multi --project-names=shop,admin` |
| Inspect the device bench | `iharmony devices list` · `iharmony devices check` |

## Ground rules that keep suites maintainable

- **Locators live in page objects**, never in test files.
- **No hard-coded URLs, credentials or browser settings** — use `config/` and environment variables.
- **No `sleep()`** — rely on Playwright's auto-waiting and the framework's `wait_for_visible()`.
- **Log, don't print** — use the `logger` fixture.
- **Keep tests independent** — every test gets a fresh browser context.

## Where results go

| Output | Location |
|---|---|
| HTML test report | `reports/report.html` |
| Screenshots and videos of failures | `reports/screenshots/`, `reports/videos/` |
| Observability, KPI and persona reports | `reports/observability/` |
| Data-comparison reports | `reports/data_validation/` |
| Device transcripts and simulator state | `reports/devices/<test>/` |

## Documentation

```bash
iharmony docs                 # every topic
iharmony docs quickstart      # print one
```

Topics: `quickstart` · `install` · `cli` · `run-tests` · `multi-project` · `test-organization` ·
`qe-visual-guide` · `api-testing` · `database` · `data-validation` · `app-automation` · `architecture` · `resilience` ·
`observability` · `secrets-governance` · `plugins` · `ci` · `mcp`

## Support

iHarmony™ is developed and supported by **Innominds**. For onboarding, enterprise support or
custom integrations, contact [iHarmony@innominds.com](mailto:iHarmony@innominds.com) or visit
[innominds.com](https://www.innominds.com).

<sub>iHarmony™ is a trademark of Innominds. © Innominds.</sub>
