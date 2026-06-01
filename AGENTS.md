# Technical Tutorial Writing Guide

Use this repository to create and revise technical tutorials. Write for readers
who want to complete a concrete task and understand the important decisions
along the way.

## Core Principles

- Define the tutorial outcome early. State what the reader will build, configure,
  or learn.
- Identify the intended audience and assumed knowledge.
- Prefer direct, active language and concise sentences.
- Explain why a step matters when the reason is not obvious.
- Keep the tutorial focused. Link to background material instead of adding long
  tangents.
- Do not invent commands, output, API behavior, configuration keys, or version
  requirements. Verify technical details before publishing.

## Tutorial Structure

Use this order unless the topic requires a different flow:

1. Title that describes the task.
2. Short introduction with the outcome and use case.
3. Prerequisites, including required tools, versions, accounts, and permissions.
4. Step-by-step instructions in the order the reader must perform them.
5. Verification steps that confirm the result works.
6. Troubleshooting notes for likely failure modes when useful.
7. Cleanup steps when the tutorial creates billable, persistent, or local
   resources.
8. Short conclusion that summarizes the result and points to a logical next
   topic.

## Writing Steps

- Use headings that describe reader goals, such as `Configure the server`.
- Start each step with the action the reader should take.
- Keep one main action per numbered step.
- Introduce code blocks before showing them.
- Explain placeholders immediately, such as `<PROJECT_ID>` or `<YOUR_TOKEN>`.
- Distinguish commands the reader runs from output the reader should expect.
- Use notes and warnings sparingly. Put critical information before the action
  it affects.

## Code And Commands

- Use fenced code blocks with an appropriate language identifier.
- Make examples minimal, runnable, and internally consistent.
- Include filenames before code blocks when the reader must edit a file.
- Prefer complete commands that readers can run as written.
- Avoid shell prompts such as `$` in command blocks unless they are necessary
  to distinguish commands from output.
- Never include real credentials, secrets, or private endpoints.
- Test commands and code examples when the environment permits it. If testing
  is not possible, state that limitation during review.

## Links And References

- Use descriptive link text instead of phrases such as `click here`.
- Prefer primary sources for technical claims, including official
  documentation, specifications, and release notes.
- Check whether version-sensitive details are current before publishing.
- Link to prerequisite concepts rather than re-explaining unrelated topics.

## Writing Agent Maintenance

- Treat `.claude/agents/*.md` as the editable source of truth for writing-agent
  profiles.
- Do not edit `.codex/agents/*.toml` directly. Codex uses generated TOML copies
  because it cannot load the Claude Code Markdown format.
- After editing, adding, or removing a writing-agent profile, regenerate the
  Codex copies:

  ```bash
  python3 scripts/sync-agents.py
  ```

- Before finishing agent-profile changes, confirm the generated files are
  current:

  ```bash
  python3 scripts/sync-agents.py --check
  ```

## Review Checklist

Before considering a tutorial complete, confirm that:

- The reader can identify the outcome and prerequisites.
- The steps follow a reproducible order with no missing transitions.
- Every code block and command is accurate, formatted, and explained.
- Verification instructions prove that the tutorial succeeded.
- Version-sensitive statements and external links have been checked.
- Terminology is consistent throughout the tutorial.
- The content has been reviewed with the local style-guide skills when
  applicable.
