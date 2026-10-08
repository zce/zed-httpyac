# Zed httpYac

A minimal [Zed](https://zed.dev) extension that executes `.http` and `.rest` requests with [httpYac](https://github.com/AnWeber/httpyac).

The extension only supplies syntax highlighting and Zed tasks. httpYac owns request parsing, environment resolution, scripting, and HTTP execution. No Rust, LSP, bundled engine, or project-specific dependencies.

## Requirements

Install **Node.js >= 18** and httpYac, and ensure `httpyac` is on Zed's `PATH`:

```sh
npm install --global httpyac
httpyac --version
```

## Try it

1. Clone this repository.
2. In Zed, run **zed: install dev extension** and select the repository root.
3. Open [examples/basic.http](examples/basic.http).
4. Use the gutter run button beside a request, or select **httpYac: Run all requests** from **task: spawn**.

Disable other Zed extensions registering the `HTTP` language while testing (including zed-resterm), as they can conflict with this extension's `.http`/`.rest` association.

## Running an unsaved request

For **httpYac: Run request**, Zed captures the complete current editor buffer through its runnable query, passes the text in a task environment variable, and writes it to a temporary `.http` file **in the same directory as the source file**. Then it executes:

```sh
httpyac send --line "$HTTPYAC_LINE" "$tmp"
```

The current buffer is **not saved**. A temporary file is necessary because httpYac's CLI reads files rather than stdin. Creating the file alongside the source preserves relative file references and allows nearby `http-client.env.json` / `http-client.private.env.json` files to be discovered.

The temporary file is created with mode 0600 and removed when the task's shell exits normally (including most interruptions). An uncatchable termination such as SIGKILL or a system crash can leave it behind. It may be briefly visible to file watchers. The directory must be writable. Because the editor content is passed through the process environment, the OS's environment size limit applies.

**httpYac: Run all requests** remains a file-based task; it saves the current buffer and calls `httpyac send --all "$ZED_FILE"`.

## Notes

- Every click invokes a separate httpYac process. In-memory named-response variables from a previous click are **not** automatically retained. To exercise chained requests in a single process, use **Run all requests** (review any mutating requests first).
- Keep shared `.http` files as portable as possible across VS Code REST Client, JetBrains HTTP Client, and Zed; httpYac-specific syntax is optional, not required.
- The grammar is pinned to [feapps/tree-sitter-http](https://github.com/feapps/tree-sitter-http).

This repo was adapted from [zce/zed-resterm](https://github.com/zce/zed-resterm).
