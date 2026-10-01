# pi-runbook-skills

Pi package containing a `find-runbook` skill for selecting the best runbook Markdown file for an error, stacktrace, alert, or incident description.

## Install

```sh
pi install git:github.com/aleury/pi-runbook-skills@main
```

## Requirements

For the Jev-powered ranking path, configure Pi with codemode and make a TypeSafe API key available to the Pi process:

```json
{
  "defaultTools": ["+codemode"]
}
```

Pi expects the key in the `TYPESAFE_API_KEY` environment variable. Set it using the environment management approach for your shell, operating system, or secret manager.

The skill still works without Jev by ranking candidates from filenames, headings, and snippets.

## Usage

```text
/skill:find-runbook sqlx::PoolTimedOut
```

or ask naturally:

```text
Find the best runbook for this error: sqlx::PoolTimedOut
```

By default, the skill searches dedicated runbook locations only:

- `runbooks/**/*.md`
- `docs/runbooks/**/*.md`
