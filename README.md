# kamal-deploy skill

A Claude Code skill that walks Claude through setting up [Kamal](https://kamal-deploy.org) end-to-end — local install on macOS, remote Docker setup on China-based servers (with cloud-mirror handling for `get.docker.com` blockage), and a team-standard `deploy.yml` template.

Built for the Keqin team's deploy workflow, but the structure is straightforward to fork and adapt.

## What it does

When a teammate asks Claude things like:
- *"set up kamal on my new Mac"*
- *"how do I get docker on this Tencent server"*
- *"gem install kamal keeps failing with permission errors"*
- *"prepare a server in Aliyun Beijing for kamal deploys"*

…this skill triggers and walks them through the right steps in the right order, with explanations of *why* each step matters.

### Coverage

1. **Local (macOS)** — verify Homebrew → install latest Ruby (not the system one) → install Kamal gem
2. **Remote (China server)** — install Docker via Aliyun or Tencent apt mirror → configure Docker Hub registry mirror → verify
3. **Deploy** — team-standard `deploy.yml` template with `keqin/*` image namespace, `/up` healthcheck, push to `private-registry.tealight.uk:8443`, env-var-based secrets

## Repository layout

```
kamal-deploy/                 ← the skill itself (install this)
├── SKILL.md                  main flow Claude follows
├── references/
│   ├── kamal-commands.md     command cheat sheet (loaded on demand)
│   ├── china-mirrors.md      Docker mirror options for China servers
│   └── accessories.md        Postgres / MySQL / Redis configs for deploy.yml
└── evals/
    └── evals.json            test prompts used while developing the skill

example-deploy/               ← minimal nginx app showing a working deploy.yml
├── Dockerfile
├── config/deploy.yml
└── .kamal/secrets            (template; real values come from env)
```

The `kamal-deploy-workspace/` directory (gitignored) holds test-run outputs from skill iteration — not relevant for users.

## Install

### Option 1: copy into your Claude Code skills directory

```bash
git clone https://github.com/<your-org>/kamal-deploy-skill.git
cp -r kamal-deploy-skill/kamal-deploy ~/.claude/skills/
```

### Option 2: package as a `.skill` file (Claude Code's skill-creator)

```
/skill-creator
> "package the kamal-deploy skill"
```

Outputs `kamal-deploy.skill` which can be shared and double-clicked to install.

## Adapting for your own team

Defaults baked into the skill that you'll likely want to change:

| What                          | Where                                                                  |
| ----------------------------- | ---------------------------------------------------------------------- |
| Image namespace `keqin/*`     | `kamal-deploy/SKILL.md` Step 6 template                                |
| Docker Hub mirror URL         | `kamal-deploy/SKILL.md` Step 5c + `references/china-mirrors.md`        |
| Private registry URL          | `kamal-deploy/SKILL.md` Step 6 + `references/china-mirrors.md`         |
| Default registry username     | `kamal-deploy/SKILL.md` Step 6 ("First-time setup" section)            |

Search the SKILL.md files for `tealight`, `KeqinHQ`, and `keqin/` — those are the team-specific bits.

## License

MIT — see [LICENSE](LICENSE).
