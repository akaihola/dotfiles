branch: fix/shell-completions-options-forward
title: fix(shell-completions): forward extra args in getSuggestions for pi-tui 0.63.x

## Problem

Pi TUI 0.63.0 (commit `0406b41a` in `badlogic/pi-mono`) added a required 4th
`options: { signal: AbortSignal; force?: boolean }` parameter to
`AutocompleteProvider.getSuggestions()`. The `wrapWithShellCompletion` wrapper in
`pi-shell-completions@0.1.0` only declares and forwards 3 arguments:

```typescript
getSuggestions(lines, cursorLine, cursorCol) {
    // ...
    return baseProvider.getSuggestions(lines, cursorLine, cursorCol);
}
```

The 4th argument is silently dropped, so `CombinedAutocompleteProvider.getSuggestions`
receives `undefined` as `options` and crashes on `options.force` when the user types `/`.

```
TypeError: Cannot read properties of undefined (reading 'force')
    at CombinedAutocompleteProvider.getSuggestions (autocomplete.js:202)
    at Object.getSuggestions (pi-shell-completions/index.ts:217)
    at ShellCompletionEditor.runAutocompleteRequest (editor.js:1792)
```

## Fix

Use `...rest` to forward any extra arguments the caller passes:

```typescript
getSuggestions(lines, cursorLine, cursorCol, ...rest) {
    // ...
    return baseProvider.getSuggestions(lines, cursorLine, cursorCol, ...rest);
}
```

This is compatible with both the old (3-arg) and new (4-arg) interface.

## Reproduction

1. Install `pi@0.63.1` globally via bun
2. Add `npm:pi-shell-completions` to `~/.pi/agent/settings.json` packages
3. Launch `pi`
4. Type `/` – crash

With `-ne` (no extensions) or with this fix applied, no crash.
