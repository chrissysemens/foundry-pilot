# SIP: Add an export command

- **Issue:** [#2 — Add an export command](https://github.com/chrissysemens/foundry-pilot/issues/2)
- **Revision:** 1
- **Status:** Ready for Review (not approved or authorized for implementation)

## Objective and user outcome

Let users export their preferences to a portable JSON file so they can use
their settings elsewhere.

## Original issue request

> The application should let users export their settings so they can use them
> elsewhere.

The issue has no additional acceptance criteria or comments.

## Confirmed requirements and decisions

The following details were clarified with the human; the original issue does not
state them:

- Add `export <path>` to the preferences CLI planned in [SIP #1](./1-add-a-settings-file.md).
- Write the export as JSON, independently of the `output-format` preference used
  by `show`.
- Export both supported preferences using their effective values. Missing keys
  are represented by their documented defaults.
- If the destination already exists, fail without changing it.
- Create missing parent directories for the destination.
- A failed export must not leave a partial destination file.
- Issue #2 depends on the settings model and persistence behavior defined by
  issue #1; it is an additive command, not a second settings utility.

## Scope

### In scope

- Implementing `export <path>` in the same CLI as issue #1.
- Reading the current effective preferences from the shared settings model.
- Writing a complete JSON representation to the requested destination.
- Clear errors for source read/parse failures, existing destinations, and
  destination path or write failures.
- Focused tests and documentation of the command and its file behavior.

### Out of scope

- Importing exported settings, adding other export formats, or exporting
  arbitrary user-selected subsets.
- Changing `set`, `show`, preference names, defaults, or the per-user settings
  schema defined by issue #1.
- Implementing issue #1, creating a PR, or merging changes as part of this
  planning task.

## Proposed approach

Issue #1 is planned but has not been implemented. This feature should be built
only after the issue #1 CLI and shared settings loader are available. Reuse that
loader so defaults and validation are consistent between `show` and `export`.

The export should be a JSON object containing exactly the two preference keys,
`display-name` and `output-format`, with their effective values. For example,
when no settings file exists, the output is equivalent to:

```json
{
  "display-name": "User",
  "output-format": "text"
}
```

The `output-format` value controls `show` only; export always uses JSON. The
destination argument should accept an absolute path or a path relative to the
current working directory. Create missing parent directories. Refuse to replace
any existing destination. Write to a temporary file in the destination
directory and install it without clobbering an existing path, cleaning up
temporary files on failure, so an unsuccessful export does not leave a partial
destination. Report failures clearly and return a non-success status.

## Proposed implementation steps

1. Complete issue #1's settings model and shared loading/validation behavior;
   resolve any remaining settings-file schema decisions noted below.
2. Add `export <path>` and serialize the two effective settings in JSON, using
   the shared loader and defaults.
3. Implement safe destination handling: create parent directories, reject
   existing destinations, avoid partial output, and report errors.
4. Add standard-library tests for exported contents, defaults, persisted values,
   path handling, no-overwrite behavior, and failure cleanup.
5. Document command syntax, JSON shape, destination behavior, and examples in
   the README.
6. Run the focused suite and verify representative CLI flows.

## Acceptance criteria and verification

| Criterion | Verification |
|---|---|
| **Original issue:** Users can export settings for use elsewhere. | Run `export <path>` and verify a valid JSON file is created at the requested destination. |
| Export always writes JSON and contains both supported preference keys with effective values. | Parse the output and compare its exact key set and values with the current effective settings. |
| If settings are absent or a supported key is missing, its default is included in the exported object. | Test export with no settings file and with a valid settings file containing one preference. |
| Export uses current saved values. | Set both preferences, run export in a separate invocation, and compare the JSON values with those saved by `set`. |
| The `output-format` preference does not change export encoding. | Set `output-format` to `text` and `json` in separate cases; verify export is valid JSON in both. |
| Missing destination parent directories are created. | Export to a nested path whose parent does not exist and verify the resulting file. |
| An existing destination is not overwritten. | Create a destination with known contents, run export to that path, and verify a clear error and unchanged contents. |
| Failed exports do not leave partial destination files. | Induce a controlled destination write/install failure and verify no partial destination is present; also verify temporary-file cleanup where feasible. |
| Source settings read or parse failures are reported and do not result in a misleading export. | Use malformed and unreadable source settings and verify a non-success result and no destination file. |

## Risks, assumptions, dependencies, and unresolved details

- **Prerequisite:** Issue #1 must establish the CLI, effective-value defaults,
  settings loader, and supported JSON schema before this feature can be
  implemented reliably.
- **Inherited schema-validation gap:** The issue #1 plan clearly covers malformed
  or unreadable files and missing keys, but does not yet fully specify behavior
  for valid JSON with an invalid root type, wrong value types, or unknown stored
  keys. Resolve this in issue #1's SIP before implementing the shared loader;
  export must not silently produce a misleading file from invalid source data.
- **Proposed path convention:** Relative export paths are interpreted from the
  current working directory. This is a conventional CLI behavior proposed here,
  not explicitly stated in issue #2.
- **Inherited empty-name assumption:** Whether empty `display-name` values are
  accepted remains an assumption in issue #1's plan. Export should preserve the
  effective value accepted by the finalized settings model rather than define
  separate validation rules.
- **No implementation or verification has occurred:** This plan describes
  proposed work and tests; it does not claim that any tests have passed.

## Review status

This is revision 1 and is ready for human review. It has not been approved, and
no implementation authorization is implied by this document or its status.
