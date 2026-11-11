# resolve_path fails for paths without template keys

## Problem
`ScriptService.resolve_path` (client/ayon_aftereffects/api/scripts.py:136) returns `None` as the resolved path when given a plain path with no template keys, so the script is reported as broken / fails to run.

## Reproduction
1. Configure a script in project settings with a plain absolute path, e.g. `C:/scripts/foo.jsx` (no `{root[...]}` or other keys).
2. Open AE scripts list / run scripts.
3. Expected: path resolves to itself. Actual: resolution fails (`None` path).

## Root cause
`StringTemplate.format_template` solves fine for key-less strings. Then `anatomy.path_remapper(result.normalized())` is called (line 155). `AnatomyRoots.path_remapper` returns `None` when the path doesn't start with a known root. `None` is returned as `resolved`, and `list_items` passes it to `_has_supported_extension` / `os.path.isfile`.

## Fix approach
In `resolve_path`: `resolved = anatomy.path_remapper(normalized) or normalized`.

## Scope
In: `resolve_path` fallback only.
Out: refactors, changes to list_items, the module-level wrapper, root remapping behaviour.

## Regression test plan (manual, no test suite in repo)
- Plain absolute path to an existing .jsx: resolves to itself, runs.
- Path using `{root[work]}/...`: still resolved/remapped.
- Path using an env var key: still resolved.
- Path with an unknown key: still reports unresolved template variables.

## Open questions
None.
