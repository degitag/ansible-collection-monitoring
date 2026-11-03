# AGENTS.md

Dieses Repository enthaelt die Ansible Collection `degitag.monitoring`.

## Ziele

- Rollen fuer Debian 13, Ubuntu 22.04/24.04/26.04 und Rocky/RHEL 9 konsistent halten
- neue Rollen nur mit sauberem Standard-Geruest aufnehmen
- lokale Checks auf macOS reproduzierbar halten

## Pflicht fuer neue oder geaenderte Rollen

- `README.md` pro Rolle
- `meta/main.yml`
- `meta/argument_specs.yml`
- `tests/test.yml`
- Plattform-Support explizit dokumentieren
- keine impliziten Distribution-Annahmen ohne `assert` oder klares `when`

## Rollen-Standard

- Defaults muessen sicher und ohne versteckte Fremdvariablen nutzbar sein
- Rollen mit Diensten sollen Konfiguration vor Restart oder Enable validieren, wenn ein Validator verfuegbar ist
- Plattformspezifische Paketnamen, Pfade und Services gehoeren in `vars/`
- Neue Rollen sollen moeglichst `ansible.builtin.*` verwenden
- Secrets gehoeren nicht in Defaults, sondern muessen leer oder optional sein
- Service-, Firewall- und Netzwerkrrollen sollen offensichtliche Schalter wie `*_manage_firewall`, `*_service_enabled` oder gleichwertige Steuerung haben
- Rollen mit generierten Dateien sollen idempotent bleiben und veraltete Artefakte nach Moeglichkeit bereinigen

## Scaffold und Formatierung

- Rollen-Geruest wird ueber `tools/generate_role_scaffold.py` gepflegt
- YAML wird ueber `tools/format_yaml.py` vereinheitlicht
- Nach Struktur-Aenderungen immer ausfuehren:
  - `.venv/bin/python tools/generate_role_scaffold.py`
  - `.venv/bin/python tools/format_yaml.py`

## Lokale Toolchain

- Auf macOS die lokale `.venv` mit Homebrew `python@3.12` bauen
- Homebrew nicht mit `sudo` ausfuehren
- Beispiel:
  - `/opt/homebrew/opt/python@3.12/bin/python3.12 -m venv .venv`
  - `.venv/bin/pip install -r requirements-test.txt`

## Mindestchecks vor Commit

- `.venv/bin/python tools/generate_role_scaffold.py --check`
- `.venv/bin/pytest -q tests/unit`
- `PATH="$PWD/.venv/bin:$PATH" bash tests/run_role_syntax_checks.sh`
- `PATH="$PWD/.venv/bin:$PATH" ansible-lint`
- bei CI-Aenderungen auch `bash -n tests/run_container_integration_smoke.sh`

## Plattform-Regeln

- `netplan`, `interfaces`, `ufw` nur Debian-Familie
- `ifcfg`, `firewalld` nur RedHat-Familie
- Rollen mit distributionsspezifischem Verhalten sollen das frueh validieren
- Beispiele sammeln Facts auf Play-Ebene; Rollen laden nur fehlende benoetigte Facts nach
- Rollen verwenden `ansible_facts`; Syntax-/Beispiel-Playbooks sammeln Facts mit `gather_facts: true`
- Wenn eine Rolle nicht sinnvoll plattformuebergreifend ist, lieber klar eingrenzen als implizit brechen

## Doku- und Namespace-Regeln

- Collection-Referenzen immer als `degitag.monitoring.*`
- keine neuen Verweise auf den alten Namespace oder alte Repository-Namen
- Beispiel-Playbooks und Rollen-READMEs nach jeder Namespace-Aenderung regenerieren

## Version 2 architecture

Alloy and node_exporter are owned by this collection. Grafana Agent and Promtail are removed without migration. Use generated preflight/configuration entries, layered platform variables, ansible_facts, guarded service handlers and artifact-based tests. Linux targets are Debian 13, Ubuntu 22.04/24.04/26.04 and Rocky/RHEL 9. Windows Alloy is a separate implementation and test target. No production rollout is implied by this refactor.

## Änderungsvertrag und Abnahme

- Wiederverwendbare Rollen und Module pflegen; kundenspezifische Hostwerte, Ziele und Secrets gehören in nutzende Inventories beziehungsweise deren Secret-Store.
- Öffentliche Variablen dokumentieren, sinnvoll und sicher vorbelegen sowie typisiert validieren. Fachlich erforderliche Eingaben vor Eingriffen prüfen. Bei kompatiblen Änderungen bestehende Nutzer und Variablenverträge erhalten; notwendige Inkompatibilitäten ausdrücklich mit Migrationsweg dokumentieren.
- Änderungen gegen die deklarierte Distributions- und Ansible-Supportmatrix aus README, Metadaten und Preflight prüfen; keine Unterstützung allein aus einem Syntaxlauf ableiten. Rollenbezogene Einschränkungen bleiben bestehen.
- Idempotenz auf geeigneten Testsystemen durch einen unveränderten zweiten Lauf mit `changed=0` nachweisen. Check-Modus soweit unterstützt berücksichtigen und seine Grenzen benennen. Echtes Dienstverhalten einschließlich systemd/journald auf geeigneten Systemen prüfen; Container allein belegen diese Funktion nicht.
- Den gemeinsamen Prüfeinstieg `bash tests/run_checks.sh` für Implementierungsänderungen und die zur Änderung passenden zusätzlichen Tests verwenden. Bei reinen Arbeitsanweisungsänderungen Markdown, Diff und Konsistenz prüfen und die ausgelöste erforderliche CI vollständig abwarten; keine künstlichen Implementierungs- oder Hosttests dafür anlegen.
- Changelog und Version passend zu funktionaler Änderung und Kompatibilität pflegen. Collection bauen und Artefakt prüfen; bei tatsächlichen Releases die Release-Prüfungen bis zum erfolgreichen Endzustand verfolgen. Reine Anweisungsänderungen begründen keinen Release-Tag. Eine Collection-Veröffentlichung autorisiert keinen Rollout auf nutzende Systeme.

## Auftragsabschluss und Dokumentation

Der globale Abschlussstandard gilt auch hier: Eigene Änderungen committen, pushen und per Pull Request (bei GitLab Merge Request) nach erfolgreichen erforderlichen Prüfungen integrieren; auftragsrelevante Nach-Merge-Prüfungen abwarten und erst danach eigene integrierte Arbeitsbranches bereinigen. Bestehende unabhängige Änderungen, Branches und Worktrees erhalten. Schutzregeln und erforderliche Reviews nicht umgehen; echte Blocker mit offenem Schritt benennen.

Repository-Integration, Veröffentlichung und produktiver Betrieb sind getrennte Ergebnisse. Eine erfolgreiche Prüfung oder ein Merge erteilt keine zusätzliche Produktionsfreigabe. Bestehende konkrete Freigaberegeln bleiben verbindlich; eine pauschale automatische Produktionsfreigabe wird hier nicht eingeführt.

In Anleitungen `vim` verwenden. Dokumentation aus Sicht von DevOps und Systemarchitektur schreiben. Technische Nachweise im Repository und bestehende Betriebsdokumentation in Outline erhalten; Outline nur im beauftragten Umfang bearbeiten. Secrets aus geschützten Variablenquellen beziehen, niemals ausgeben oder ins Repository übernehmen.
