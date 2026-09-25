# LanguageCSVExporter
A [ProcessWire](https://processwire.com) module that lets you export a module's translations for a selected language as a downloadable CSV file directly from the admin.

## Why this module?

ProcessWire's core translation tools work great for editing translations of *one* language file at a time in the admin, but there is no built-in, one-click way to export all registered translations of a specific module — especially awkward when a module's translatable strings are spread across many files (fieldtypes, inputfields, validation rules, and so on).

Language CSV Exporter adds a simple admin page: pick a module, pick a language, click download — you get a single CSV file with every registered translation for that module and language.

This module is specifically designed for module developers who want to ship their modules with language files. Instead of manually copying all translation files into a single CSV file, you can export them with just one click using this module.

## Features

- Simple, one-page admin interface under **Setup → Language CSV Exporter**
- Select any installed module and any configured language
- Downloads a single CSV file (`{module}-{language}-translations.csv`)
- **Correctly separates translations of modules that share the same folder.** If two modules live in the same directory (a common pattern for module "bundles", e.g. a main module and a companion admin/manager module shipped together), the export only ever contains the translations that actually belong to the selected module — not its neighbor's.
- Dedicated permission (`language-csv-exporter`) so you can control who is allowed to use it
- No third-party dependencies — uses ProcessWire's own `LanguagePorter` API under the hood

## Requirements

- ProcessWire 3.0.264 or newer
- The core **LanguageSupport** module installed (i.e. your site has multi-language support enabled)

## Installation

1. Download or clone this repository into `site/modules/LanguageCSVExporter/`
2. In the admin, go to **Modules → Refresh**
3. Find **Language CSV Exporter** on the **Site** tab and click **Install**

## Usage

1. Go to **Setup → Language CSV Exporter**
2. Select the module whose translations you want to export
3. Select the language
4. Click **Download translations as CSV**

Your browser will download a CSV file named after the module and language, for example `frontendforms-de-translations.csv`.

## The CSV format

The exported file uses ProcessWire's standard translation CSV format (as produced by `LanguagePorter::exportCsv()`), with the following columns:

| Column | Description |
|---|---|
| `en` | The original, English source string |
| `de` (or whichever language was exported) | The translated string |
| `description` | An optional translator note, if one was set |
| `file` | The relative path of the source file the string was found in |
| `hash` | An internal hash ProcessWire uses to match the string back to its source |

This is the same format used by ProcessWire's own language import/export tools, so the resulting file can be handed off for translation and re-imported normally through **Setup → Languages**.

## How the folder-sharing detection works

Some modules ship as a pair — a main module plus a companion module — both living in the same folder (for example `site/modules/FrontendForms/` might contain both `FrontendForms.module` and `FrontendFormsManager.module`). Since ProcessWire's own CSV export only supports filtering by folder, exporting either one of these modules on its own would normally also pull in the other's translations.

This module works around that by looking at every module (installed or not) whose file lives inside the selected module's folder:

- **Modules in a subfolder** (e.g. `site/modules/FrontendForms/SubModules/FooBar/FooBar.module`) own that whole subfolder. It is always excluded — export that module separately.
- Among the modules **directly in the same folder**, one is picked as the **primary** module. Its export includes everything in the folder *except* the other modules' own files and subfolders. It is determined in this order:
  1. it is the only module in the folder,
  2. the folder name equals its class name (case-insensitive),
  3. the folder name starts with its class name (e.g. a GitHub download `FrontendForms-main/`),
  4. its class name is the prefix of all other class names in the folder (e.g. `FrontendForms` vs. `FrontendFormsManager`).
- Every other module in that folder is a **guest** module. Its export is restricted to *only* its own file.

If no primary module can be determined, every module in that folder only gets its own file, and a warning is shown.

This detection is automatic and requires no configuration.

## Troubleshooting

**"No translations were found …"** — the export only contains *registered* translation files, i.e. files that have been opened at least once for the selected language in **Setup → Languages → (language) → Translate files**. Register the module's files there first, then export again.

## Permissions

This module adds a `language-csv-exporter` permission during installation. Only users (or roles) with this permission can access the export page. The permission is removed again automatically on uninstall.

## License

MIT
