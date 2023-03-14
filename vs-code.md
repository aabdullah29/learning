

### reset all setting in mac:
```
rm -rfv "$HOME/.vscode"
rm -rfv "$HOME/Library/Application Support/Code"
rm -rfv "$HOME/Library/Caches/com.microsoft.VSCode"
rm -rfv "$HOME/Library/Saved Application State/com.microsoft.VSCode.savedState"
```

### User settings location :
[link](https://stackoverflow.com/questions/71838032/how-to-remove-the-visual-studio-code-cloud-settings-from-github)
- Windows: %APPDATA%\Code\User
- macOS: $HOME/Library/Application\ Support/Code/User
- Linux: $HOME/.config/Code/User


### Workspace settings location :
- Windows : %USERPROFILE%\.vscode
- macOS : ~/.vscode
- Linux : ~/.vscode


### Files location :
- Settings : <user-settings>\Code\User\ settings.json
- Keyboard Shortcuts : <user-settings>\Code\User\ keybindings.json
- User snippets : <user-settings>\Code\User\snippets\ <language-name>.json
- User tasks : <user-settings>\Code\User\ tasks.json
- Extensions : <workspace-settings>\ extensions.json
- UI State : <user-settings>\Code\User\globalStorage\ state.vscdb
(vscdb file is a SQLite database, tool like sqlitebrowser is required)


