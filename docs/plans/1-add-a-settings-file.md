# SIP: Add a settings file

- **Issue:** [#1 — Add a settings file](https://github.com/chrissysemens/foundry-pilot/issues/1)
- **Revision:** 2
- **Status:** Ready for Review (not approved or authorized for implementation)

## Objective and user outcome

Provide a standard-library-only Python 3 command-line preferences utility whose
settings persist between invocations. Users can set the supported preferences
and inspect their current values.

## Original issue request

> The application should save its settings so they're remembered next time.

The issue does not provide additional acceptance criteria.

## Confirmed requirements and decisions

The following details were clarified with the human and are requirements for
this plan; they are not stated in the original issue:

- Build a standalone Python 3 command-line utility with `set` and `show`
  commands.
- Require Python 3.10 or later.
- Use only the Python standard library.
- Support macOS, Windows, and Linux.
- Persist settings in a per-user JSON file.
- Support exactly two preferences:
  - `display-name`: a string, defaulting to `User`.
  - `output-format`: either `text` or `json`, defaulting to `text`.
- Use `set <name> <value>` to change a preference and `show` to display both
  preferences. The saved `output-format` controls the output of `show`.
- If no settings file exists, `show` displays the defaults:
  `display-name = User` and `output-format = text`.
- If a preference key is missing from the file, use that preference's default.
- A successful `set` saves immediately; a later, separate `show` invocation
  loads the saved values.
- Unknown preference names, invalid values, and malformed or unreadable files
  must produce clear errors. A malformed or unreadable file must not be silently
  overwritten.

The human confirmed Python 3.10+ during Grill Me. The human also clarified that
empty `display-name` values were not explicitly agreed. Allowing an empty value
remains a proposed assumption for review, not a confirmed requirement.

## Scope

### In scope

- Command-line argument parsing and validation for the two commands and two
  supported preferences.
- Cross-platform resolution and use of a per-user settings-file location.
- JSON persistence, defaults, text output, JSON output, and explicit error
  reporting.
- Focused automated tests and concise usage documentation.

### Out of scope

- Adding more preferences, commands, output formats, or configuration sources.
- Dependencies outside the Python standard library.
- Integration with a GUI or another application not present in this repository.
- Creating a PR, implementing this plan, or merging changes as part of this
  planning task.

## Proposed approach

The repository currently contains only its README, which describes this as a
disposable workflow pilot. There is no existing application structure, test
suite, or project convention to extend.

Implement a small CLI module using standard-library facilities such as
`argparse`, `json`, and `pathlib`/`os`. Keep the preference defaults and
validation in one shared implementation path so `set` and `show` interpret the
same settings consistently. `show` should emit human-readable `name = value`
lines in text mode and a JSON object in JSON mode.

**Recommended settings-file paths** (proposed implementation details, not
requirements confirmed in the issue):

- macOS: `~/Library/Application Support/foundry-pilot/settings.json`.
- Windows: `%APPDATA%\foundry-pilot\settings.json`. If `APPDATA` is unset,
  empty, or not an absolute path, fall back to
  `~/AppData/Roaming/foundry-pilot/settings.json`.
- Linux: `$XDG_CONFIG_HOME/foundry-pilot/settings.json` when
  `XDG_CONFIG_HOME` is set to a non-empty absolute path; otherwise use
  `~/.config/foundry-pilot/settings.json`. A relative `XDG_CONFIG_HOME` is
  ignored in favor of the default.

The recommended application directory is `foundry-pilot`, and the filename is
`settings.json`. These paths are based on the platform's conventional per-user
configuration locations and do not require a third-party library. Do not fall
back to a working-directory file if the selected per-user path cannot be
resolved or accessed; report a clear error instead. Create the application
directory only when saving a setting, not when `show` uses defaults.

On `show`, load the file if present, overlay its keys on the defaults, and
validate present values. If it is absent, use defaults without creating it.
On `set`, load and validate existing data before updating one preference.
Write the complete settings document immediately after successful validation.
Use a temporary file and atomic replacement where supported to avoid leaving a
partially written settings file. Report filesystem and parse failures clearly;
do not treat them as defaults or overwrite unreadable/malformed data.

## Proposed implementation steps

1. Define the two defaults, accepted values, settings-file schema, and the
   recommended cross-platform paths and fallback behavior above.
2. Implement `set` and `show`, shared load/default/validation logic, text and
   JSON rendering, and immediate JSON persistence.
3. Add standard-library automated tests for command behavior, persistence
   across separate invocations, defaults, missing keys, validation, and file
   errors.
4. Document installation or invocation, supported platforms, commands,
   defaults, output formats, and settings-file location in the README.
5. Run the focused test suite and verify the CLI's help and representative
   command flows on available environments.

## Acceptance criteria and verification

| Criterion | Verification |
|---|---|
| **Original issue:** Settings are saved so they are remembered next time. | Run `set` to change a value, then run `show` in a separate process and verify the saved value is reported. |
| A missing file makes `show` display `display-name = User` and `output-format = text`, without creating a file. | Run `show` with an isolated, nonexistent settings location; assert both defaults and that the file remains absent. |
| Missing preference keys use their documented defaults. | Load a valid JSON file containing only one supported preference and verify the other appears with its default. |
| A successful `set` saves immediately, and `show` loads those values on a later invocation. | Exercise `set display-name "..."` and `set output-format json`, then inspect results in separate `show` processes and verify persisted JSON. |
| `show` follows the saved `output-format`. | Verify text output for `text` and parseable JSON output for `json`, including both preference names and values. |
| Only the two supported preferences and valid values can be set. | Verify unknown names and invalid `output-format` values return clear errors and do not report success. Under the proposed empty-name assumption, verify an empty `display-name` is accepted. |
| Malformed or unreadable settings files result in a clear error and are not silently overwritten. | Use malformed JSON and a controlled unreadable-file case; assert a non-success result, an actionable error, and unchanged file contents. |
| The utility requires Python 3.10+, uses only the standard library, and supports macOS, Windows, and Linux. | Confirm the documented minimum version, no third-party runtime dependencies, and run the focused suite on all supported CI platforms when available. |

## Risks, assumptions, dependencies, and open details

- **Proposed config-path details:** The human requested explicit path guidance;
  the platform paths, `foundry-pilot` directory, `settings.json` filename, and
  fallback rules above are recommendations for review, not details independently
  stated in the issue.
- **Proposed empty-name behavior:** Empty `display-name` values are allowed in
  the current proposal, but this was not explicitly agreed. Human review should
  confirm or change this assumption before implementation.
- **CLI execution shape:** The repository has no packaging or entry-point
  convention. The implementation should document a direct invocation that works
  from a checkout; whether to add a packaged console entry point should be
  decided only if packaging is introduced within scope.
- **Dependencies:** No third-party dependencies are planned. Cross-platform
  verification depends on access to macOS, Windows, and Linux test environments.

## Review status

This is revision 2 and is ready for human review. It has not been approved, and
no implementation authorization is implied by this document or its status.
