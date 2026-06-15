# Nasra — Odoo HR & Payroll Suite — Plemo Repo Guide

> Custom Odoo HR/Payroll/Attendance addon suite for the customer **Nasra** (Arabic–Syria locale), built by Centione. 32 in-house `nasra_*` modules layered on Odoo's `hr`, `hr_payroll`, `hr_attendance`, and `hr_holidays`.

## Overview
This repo is a single Odoo addons directory: every top-level folder is one custom module. Together they form an end-to-end Human Resources system — employee profiles, contracts, work schedules, biometric attendance ingestion, late/early/absence penalty calculation, leaves with multi-level approval, payroll structures and salary rules, income tax, social insurance, loans, custody, medical insurance, end-of-service, and overtime. The modules are designed to be dropped into an Odoo `addons` path and installed; there is no application server, container, or CI tooling in the repo. Localisation files target `ar_SY` (Arabic — Syria), so this is a Syrian deployment.

## Tech Stack
- **Platform:** Odoo (server-side addons). Manifests use the modern `__manifest__.py` convention and depend on `hr_payroll`, `hr_work_entry`, `hr_work_entry_contract`, and `hr_work_entry_contract_enterprise` — these are **Odoo Enterprise** payroll apps, so the suite requires the **Enterprise** edition (roughly the Odoo 14/15 era based on the module surface). Two legacy modules still use the old `__openerp__.py` manifest name.
- **Language:** Python (models, wizards, controllers) + Odoo XML (views, data, security) + CSV (`ir.model.access.csv`). Compiled `.pyc` artifacts target CPython 3.8/3.9.
- **External Python deps (for `nasra_zk_attendance` only):** `xmltodict`, `pytz`, `python-dateutil`.
- **Hardware integration:** ZK biometric attendance machines (see `nasra_zk_attendance`).
- **Localisation:** `ar_SY` translations in attendance modules; `nasra_hr_public_holidays` carries many community locales (ar, de, es, fr, it, pl, pt_BR, etc.).

## Repository Structure
Every top-level directory is a self-contained Odoo addon (standard layout: `models/`, `views/`, `security/`, `wizard/`, `controllers/`, `data/`, `demo/`, `i18n/`). There is no `addons/` parent folder — modules sit at the repo root.
- `nasra_hr*/` — the HR/payroll/attendance modules (see table below).
- `README.md` — stub only (`# nasra`); not informative.
- `.idea/` — JetBrains/PyCharm project metadata (not part of the deployable code).
- Note: many folders also contain committed `__pycache__/*.pyc` build artifacts; ignore them.

## Modules / Addons
32 addons total; all detailed below. `author` is **Centione** except `nasra_hr_attendance_base` and `nasra_hr_contract_work_hours` (author **Mussder**, legacy `__openerp__.py`) and `nasra_salary_rules` (placeholder author). Dependency direction matters for install order; the foundational modules are `nasra_hr`, `nasra_hr_payroll_base`, `nasra_hr_public_holidays`, `nasra_hr_contract_work_hours`, and `nasra_hr_attendance_base`.

| Addon | Summary | Key dependencies |
| --- | --- | --- |
| `nasra_hr` | Base HR customizations (employee grade, employee view tweaks). | `hr` |
| `nasra_hr_contract` | Customized `hr_contract`; prevents multiple running contracts per employee. | `base`, `hr_contract` |
| `nasra_hr_contract_work_hours` | Adds work-hours config to contracts (legacy `__openerp__.py`). | `hr_contract` |
| `nasra_hr_payroll_base` | Foundation payroll structures: defines payroll structure type + default structure. | `hr_payroll`, `hr_work_entry` |
| `nasra_hr_payroll` | Payroll extensions incl. payslip-batch report. | `hr_payroll`, `nasra_hr_payroll_base`, `nasra_hr_attendance_base` |
| `nasra_salary_rules` | Salary rule definitions (data + views). | `base`, `hr`, `hr_payroll`, `nasra_hr_payroll_base`, `hr_contract` |
| `nasra_hr_public_holidays` | Manage public holidays (v10-era OCA-style module; broad i18n). | `hr`, `hr_holidays` |
| `nasra_hr_attendance_base` | Core attendance analysis engine (analyzed intervals/periods, holiday-aware calculations). Foundational. | `hr`, `nasra_hr_public_holidays`, `hr_attendance`, `nasra_hr_contract_work_hours` |
| `nasra_zk_attendance` | Integration with ZK biometric attendance machines; reads attendance logs from a text file. | `base`, `hr`, `hr_attendance`, `nasra_hr` |
| `nasra_make_attendance_log` | Generates/feeds attendance log entries for the ZK pipeline. | `base`, `nasra_zk_attendance` |
| `nasra_attendance_excel` | Excel attendance import/export. | `base`, `hr_attendance`, `nasra_hr_self_service` |
| `nasra_attendance_manual_recalculate` | Wizard to manually recalculate attendance. | `base`, `nasra_zk_attendance` |
| `nasra_absence_manual_recalculate` | Wizard to manually recalculate absences. | `base`, `nasra_hr_late_early_absence`, `nasra_attendance_manual_recalculate` |
| `nasra_hr_late_early_absence` | Late / early-leave / absence attendance penalty logic. | `resource`, `hr_attendance`, `hr_holidays`, `nasra_hr_payroll_base`, `nasra_hr_public_holidays` |
| `nasra_repair_late_early_penality` | Repair/recompute utility for late–early penalties. | `base`, `hr`, `nasra_hr_late_early_absence` |
| `nasra_hr_work_schedule` | Work schedule management tied to attendance. | `base`, `hr`, `resource`, `nasra_hr_late_early_absence` |
| `nasra_over_time` | Overtime calculation and payroll integration. | `base`, `hr`, `nasra_hr_contract`, `nasra_hr_payroll_base`, `hr_work_entry_contract`, `hr_work_entry_contract_enterprise`, `hr_attendance` |
| `nasra_hr_self_service` | Employee self-service portal (leaves, payroll, recruitment touchpoints). | `hr_holidays`, `hr_payroll`, `hr_work_entry_contract`, `hr_recruitment` |
| `nasra_hr_holidays_multi_levels_approval` | Multi-level approval workflow for leave requests. | `hr`, `hr_holidays` |
| `nasra_hr_multi_approval_chain` | Generic multi-step approval chain across self-service, loans, and leaves. | `base`, `hr`, `nasra_hr_self_service`, `nasra_hr_loan_correct`, `nasra_hr_holidays_multi_levels_approval` |
| `nasra_hr_loan_correct` | Employee loan management; deducts loans in payslips; accounting + mail integration. | `hr`, `hr_payroll`, `account`, `nasra_hr_payroll_base`, `mail` |
| `nasra_income_tax` | Income-tax configuration for payroll. | `hr`, `hr_payroll`, `nasra_hr_payroll_base` |
| `nasra_insurance` | Social insurance configuration + payroll integration. | `hr`, `nasra_hr_contract`, `hr_work_entry_contract`, `nasra_hr_payroll_base` |
| `nasra_hr_medical_insurance` | Medical insurance management. | `nasra_hr`, `hr_payroll` |
| `nasra_hr_variable_allowance_deduction` | Variable allowances & deductions on payroll. | `hr_payroll`, `hr_work_entry_contract_enterprise`, `nasra_hr_payroll_base`, `nasra_hr_contract` |
| `nasra_hr_custody` | Employee custody (assets handed to employees). | `hr` |
| `nasra_hr_end_service` | End-of-service / final settlement calculation. | `hr`, `nasra_hr_custody`, `hr_contract`, `nasra_hr_payroll_base` |
| `nasra_employee_profile` | Consolidated employee profile view across custody, contract, payroll, attendance, leaves, EoS. | `base`, `nasra_hr_custody`, `hr_contract`, `hr_payroll`, `hr_attendance`, `hr_holidays`, `nasra_hr_end_service` |
| `nasra_hr_employee_document` | Employee document management. | `hr` |
| `nasra_hr_employee_customizations` | Misc employee-model field/view customizations. | `base`, `hr` |
| `nasra_hr_organization_chart` | Organization chart view. | `hr` |
| `nasra_top_managment_group` | Defines a top-management security group / contract access (manifest name mislabeled "Nasra HR contract"). | `base`, `hr_contract` |

## Key Files & Entry Points
- `nasra_hr_attendance_base/models/hr_attendance.py` — the heart of attendance: `get_attendance_dates()` and period analysis; consumes the `classes/analyzed_interval.py` and `classes/analyzed_period.py` helpers (constants `P_VE`, `N_VE`, `LEAVE_COVERED`). Start here to understand how raw punches become payable/penalized days.
- `nasra_hr_attendance_base/classes/analyzed_period.py`, `analyzed_interval.py` — pure-Python analysis primitives (note: duplicate copies also exist at the module root — prefer the `classes/` versions).
- `nasra_zk_attendance/__manifest__.py` — documents the ZK machine integration; expects a logs text file at `/opt/odoo/pharos/custom/zkLogs.txt` and the `xmltodict`/`pytz`/`python-dateutil` packages.
- `nasra_hr_payroll_base/data/hr_payroll_structure_type.xml` + `hr_payroll_structure.xml` — seed the default payroll structure (`custom_default_payroll_structure`); most payroll modules build on this.
- `nasra_salary_rules/data/salary_rules.xml` — salary rule definitions.
- Per-module `security/ir.model.access.csv` — access rights; `nasra_attendance_manual_recalculate/security/sec_groups.xml` and `nasra_top_managment_group` define groups.
- Wizards (`*/wizard/*.py` + matching `views/*wizard*.xml`) drive the manual recalculation flows in `nasra_attendance_manual_recalculate` and `nasra_absence_manual_recalculate`.

## Working in This Repo
- **Install/run:** No Docker, requirements file, or CI is committed. Deploy by placing the `nasra_*` folders on the Odoo addons path of an **Odoo Enterprise** server, then install via Apps. Install foundational modules first (`nasra_hr`, `nasra_hr_payroll_base`, `nasra_hr_public_holidays`, `nasra_hr_contract_work_hours`, `nasra_hr_attendance_base`) — Odoo resolves the rest from `depends`. The `pharos` path in the ZK manifest implies an on-prem `/opt/odoo/pharos/custom/` deployment.
- **Conventions & gotchas observed:**
  - Two modules use the legacy `__openerp__.py` manifest name (`nasra_hr_attendance_base`, `nasra_hr_contract_work_hours`) — keep that name or Odoo won't detect them.
  - Manifest metadata is thin: most `summary`/`description` fields are empty, `category` is often `Uncategorized`, and every `version` is `0.1` except `nasra_hr_public_holidays` (`10.0.1.0.0`) and a couple of `1.0`s. Don't trust version strings to infer the Odoo target — infer it from the enterprise `hr_work_entry*` deps instead.
  - `nasra_top_managment_group/__manifest__.py` has a copy-pasted `name` of "Nasra HR contract" — the folder, not the manifest name, is authoritative.
  - Several manifests have unusual comma-first `depends` formatting (`, 'depends': [...]`) — valid Python, just stylistically odd.
  - Committed `__pycache__/*.pyc` artifacts (cpython-38/39) are noise; the deployment Python is 3.8/3.9.
  - XML data-load order matters for payroll: structure type → structure → salary rules. The attendance base notes "Order of files matters, be careful."
  - This is a customer-specific suite, not a generic library — many calculations encode Syrian HR/payroll rules (income tax, insurance, public holidays).

## Repository Notes
- **Default branch:** `main` (this guide documents `main`).
- `main` contains the full codebase (32 addons, 824 blobs); it is **not** a near-empty/README-only branch.
- `README.md` is a one-line stub (`# nasra`); this PLEMO.md is the authoritative map. (README was left untouched.)
- No tests, CI pipelines, Dockerfiles, or `requirements.txt` are present in the repo.
