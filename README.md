# 📓 Obsidian Vault

This repository contains the data for a personal [Obsidian](https://obsidian.md/) vault, used to store and sync notes, ideas, and study/work material in Markdown format.

## 📁 Structure

```
.
├── 00-Hubs/           # Main of Concept 
├── 01-Resource/       # Concept's Resource 
├── 10-Concepts/       # Coding Concept
├── 20-Projects/       # Project using 10-Concept
├── 30-Todo List/      # Todo list everyday
├── 90-Templates/      # Templates for Hubs, Projects, Todo List, Concepts
└── .obsidian/         # Obsidian configuration (theme, plugins, hotkeys...)
```

> The structure above is just an example — adjust it to match your actual vault.

## 🚀 Usage

1. Clone this repository:
   ```fish
   git clone https://github.com/wickyhien18/Obsidian-Vault
   ```
2. Open [Obsidian](https://obsidian.md/) and choose **Open folder as vault**.
3. Point it to the cloned folder.

## 🔌 Plugins Used

- (List the community plugins you use, e.g. Dataview, Templater, Calendar...)

## 🔄 Syncing

The vault is synced manually via Git:

```fish
git add .
git commit -m "update notes"
git push
```

## ⚠️ Notes

- Some notes may contain personal information — consider this before making the repo public.
- The `.obsidian/` folder contains local machine settings; you may want to `.gitignore` parts of it if you don't want them synced (e.g. `.obsidian/workspace.json`).
