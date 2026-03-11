# 03-file-operations

Rust backend exposes Tauri commands for reading files, saving files, listing directories, and counting words. All file I/O goes through `std::fs`. Frontend calls these via `invoke()`.

## System Diagram

```mermaid
flowchart TB
    FE[Frontend invoke] --> IPC[Tauri IPC]
    IPC --> RF[read_file]
    IPC --> SF[save_file]
    IPC --> WC[word_count]
    IPC --> LD[list_directory]
    IPC --> HD[get_home_dir]
    RF --> FS[std::fs]
    SF --> FS
    LD --> FS
```

## 1. Tauri Commands

| Command | Input | Output | Purpose |
|---------|-------|--------|---------|
| `read_file` | `path: String` | `FileInfo { path, content, size }` | Read file + metadata |
| `save_file` | `path: String, content: String` | `()` | Write content to path |
| `word_count` | `text: String` | `WordCount { chars, words, lines }` | Count chars/words/lines |
| `list_directory` | `path: String` | `Vec<DirEntry>` | List directory entries |
| `get_home_dir` | none | `String` | Return `$HOME` |

## 2. Directory Listing

`list_directory` filters dotfiles, sorts directories first then alphabetically (case-insensitive). Returns `DirEntry` with `name`, `path`, `is_dir`, `extension`.

## 3. Frontend File Flow

| Action | Trigger | Dialog |
|--------|---------|--------|
| Open file | `Cmd+O` or toolbar | `@tauri-apps/plugin-dialog` open |
| Save file | `Cmd+S` or toolbar | Save-as dialog if no current path |
| Drop file | Drag `.md` onto window | None (Tauri drag-drop event) |
| Browse folder | Toolbar "Folder" button | None (loads sidebar) |

## 4. State Tracking

`currentFilePath` tracks the open file. `isModified` flips to `true` on any content change, resets on save. The modified indicator (`●`) shows in the toolbar.

## File Reference

| File | Purpose |
|------|---------|
| `src-tauri/src/lib.rs:28-42` | `read_file`, `save_file` |
| `src-tauri/src/lib.rs:44-51` | `word_count` |
| `src-tauri/src/lib.rs:53-83` | `list_directory` |
| `src/main.ts:130-175` | Frontend file open/save logic |
| `src/main.ts:273-282` | Drag-and-drop handler |

## Cross-References

| Doc | Relation |
|-----|----------|
| [00-architecture-overview](00-architecture-overview.md) | IPC model |
| [05-ui-layout](05-ui-layout.md) | Sidebar file tree |
| [04-pdf-export](04-pdf-export.md) | Also uses file I/O |
