# aMiNo Skills

Real skill packages for [aMiNo](https://github.com/AMINODa/aMiNo) — the on-device Android manager agent (Shizuku + Debian PRoot).

## Skills

| Skill | What it does | Import URL |
|-------|--------------|-----------|
| `android-system-doctor` | Full device health check (battery / storage / memory / CPU / network) with real commands and an honest verdict | see below |
| `smart-app-launcher` | Open any installed app with ONE command — resolve real package from pm list (never memory), monkey launch (never guessed activity names), disabled check, topResumedActivity verification, honest NOT INSTALLED verdict | see below |

## smart-app-launcher

**Import URL (Skills Center → 🧩 Skills → Import → GitHub):**

```
https://github.com/AMINODa/aMiNo-skills/tree/main/smart-app-launcher
```

**Run it** by asking in chat:

> افتح تيك توك باستعمال مهارة smart-app-launcher

or any app: "افتح واتساب" / "lance instagram" / "open youtube".

## How to import into aMiNo 1.4+

**In the Skills Center** (drawer → 🧩 Skills → Import → GitHub):

```
https://github.com/AMINODa/aMiNo-skills/tree/main/<skill-folder>
```

**Or just tell the agent in chat:**

> استورد مهارة من https://github.com/AMINODa/aMiNo-skills/tree/main/android-system-doctor

Preview the parsed package, then install. The skill is a playbook: the agent loads it with `skill_use` and executes it with its own real tools (`shell_command`, `screen_read`).

**Run it** by asking in chat:

> افحص جهازي باستعمال مهارة android-system-doctor

## Adding your own skills

Create a folder with a `SKILL.md` (YAML frontmatter: `name`, `description`, optional `version`, `tags`, `allowed-tools`) or a `manifest.json`, open a PR, or keep it in your own repo and import from there — aMiNo imports SKILL.md, JSON manifests, ZIP packages, local folders, GitHub URLs, or pasted text.
