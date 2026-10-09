# Module: Editor tooling

For editor extensions and Asset Store editor tools.

- **No emoji in IMGUI.** Unity's editor GUI cannot render emoji; they show as empty boxes. Use `EditorGUIUtility.IconContent("<builtin icon name>")` for icons, or plain ASCII text.
- Editor code lives under an `Editor` folder or an editor-only `.asmdef`, so it never ships in a player build.
