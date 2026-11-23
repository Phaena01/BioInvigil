# BioInvigil: offline-first biometric exam invigilation

COEN838 Advanced Software Engineering. A hub enrols students and issues each hall terminal a sealed, signed
bundle; the terminal verifies candidates at the door with no network, logs every decision in a hash chain, and syncs
to the hub later. The fingerprint scanner is **simulated** (128-d feature vectors); error rates are model-based.

## Quick start

```bash
python3 -m venv .venv && source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements-dev.txt && pip install -e .
make check            # ruff, 170+ tests with coverage (>= 90 %), bandit, pip-audit
```

Python 3.11 to 3.15. Tests take about 80 s. 
## Run the demo

Three terminals, from the repository root.

```bash
mkdir -p demo && export BIOINVIGIL_DB=demo/hub.db BIOINVIGIL_ADMIN_PASSWORD='Admin-Demo-Passw0rd!'

# 1. hub
python -m bioinvigil.central create-admin admin
python -m bioinvigil.central serve --port 8000

# 2. provision a session (40 students, one hall, one terminal); prints staff passwords and the keystore passphrase
python scripts/seed_demo.py --out demo/term --students 40

# 3. hall terminal with the simulated scanner, then open http://127.0.0.1:8001
export BIOINVIGIL_KEY_PASSPHRASE='<passphrase printed in step 2>' BIOINVIGIL_SIMULATOR=1
python -m bioinvigil.edge --bundle demo/term/bundle.json --keys demo/term/keys.enc \
    --hub-pub demo/term/hub_pub.txt --db demo/terminal.db --hub-url http://127.0.0.1:8000
```

Sign in as `invig1` (password from step 2). Verify `U20CS1001` (leave "finger of" empty); try `U20CS1002` with the finger
of `U20CS1003` three times; approve the override as `chief1`. The demo script is in `docs/07_Technical_Report.docx`, §9;
the full staff workflow (users, halls, rosters, enrolment, terminal provisioning) is in
`docs/08_User_Manual_and_Installation_Guide.docx`.

## Layout

| Path | Content |
| --- | --- |
| `src/bioinvigil/` | `central/` hub (FastAPI + SQLite), `edge/` terminal (engine, event store, keystore, sync, UI), `admin.py` hub client, `crypto.py`, `audit.py`, `biometric.py` |
| `tests/` | unit, integration, system (BDD), security, performance, traceability gate |
| `scripts/` | `simulate_thresholds.py` (FAR/FRR study), `seed_demo.py` |
| `reports/` | latest test, coverage, performance, threshold, bandit and pip-audit results |
| `docs/` | the eight Word documents, `RTM.csv`, `figures/`; `docs/build/` regenerates them (`make docs`) |

## Documents

01 SRS · 02 Design Document · 03 Threat Model and Security · 04 Test Plan and Report · 05 Requirements Traceability
Matrix · 06 Risk Register and Maintenance Plan · 07 Technical Report · 08 User Manual and Installation Guide.

## Known limits

Simulated scanner and model-based error rates; no liveness detection; TLS is expected at a reverse proxy; no data-subject
deletion job yet; DPIA and legal review pending. Full list: Technical Report §7.
