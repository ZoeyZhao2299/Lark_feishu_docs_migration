# 🚀 Lark to Feishu Docs Migration Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)
[![Lark](https://img.shields.io/badge/Platform-Lark%20%7C%20Feishu-blue)](https://open.feishu.cn/)

This repository provides a reusable migration workflow for moving selected cloud documents, Sheets, and Base/Bitable resources from Lark into Feishu.

The workflow is designed for practical, reviewed migrations: inventory the source workspace, filter the migration scope, export supported document types, import them into Feishu, move the imported resources into Feishu `My Document Library`, and clean up temporary Drive folders afterward.

---

## 📂 Repository Structure

* 📄 **[README.md](README.md)** - Main documentation and project overview.
* 🛠️ **[Lark_feishu_docs_migration_SKILL.md](Lark_feishu_docs_migration_SKILL.md)** - The core, reusable step-by-step migration workflow.

---

## ✨ Features & Scope

The migration skill covers the full lifecycle of a controlled Lark-to-Feishu document migration:

- [x] **App Provisioning:** Guide for Lark and Feishu developer app setup.
- [x] **Authentication:** User authorization workflow using `lark-cli`.
- [x] **Discovery:** Source resource inventory generation through Drive search.
- [x] **Granular Control:** Scope filtering before bulk migration, such as owner, folder, wiki space, or reviewed allowlist.
- [x] **Data Transformation:** Export/import type mapping for Docx, Sheets, and Base/Bitable resources.
- [x] **Organization:** Moving imported files into Feishu `My Document Library`.
- [x] **Housekeeping:** Post-migration cleanup of temporary Drive shortcuts and folders.
- [x] **Verification:** Final checks for migrated resources and temporary folder cleanup.

---

## 🔒 Safety & Best Practices

> [!IMPORTANT]
> To ensure data security and a smooth migration experience, review credentials, permissions, and migration scope before running any bulk operation.

* **Keep Secrets Safe:** Never commit app secrets (`app_secret`), access tokens, device codes, OAuth token files, or private customer data to version control.
* **Double Check Scope:** Always review the generated migration scope and inventory list *before* running bulk operations.
* **Context Awareness:** Use the `--as user` flag when interacting with and migrating personal cloud documents to ensure proper permission inheritance.
* **Avoid Accidental Over-Migration:** Do not migrate shared, external, cross-tenant, or other-owner resources unless that scope is explicitly approved.
* **Clean Up Carefully:** Before deleting temporary folders or shortcuts, verify that they were created by the migration workflow and no longer contain needed files.

---

## 🛠️ Quick Start

To begin using this workflow, read the detailed migration guide:

👉 **[Read the Migration Workflow](Lark_feishu_docs_migration_SKILL.md)**

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome. Feel free to open an issue or improvement suggestion in [ZoeyZhao2299/Lark_feishu_docs_migration](https://github.com/ZoeyZhao2299/Lark_feishu_docs_migration).
