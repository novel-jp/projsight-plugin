# ProjSight Plugin

ProjSight is an AI-native project management SaaS where AI coding agents record tasks, risks, and design decisions over MCP as they code.

This plugin connects Claude Code and Cursor to the ProjSight MCP server and adds the skills and workflow rules that go with it. Engineers keep working in their coding agent; product owners, project managers, and tech leads follow the project on the ProjSight web dashboard at [projsight.com](https://projsight.com).

日本語の説明は [projsight.com/lp/](https://projsight.com/lp/) と [よくある質問](https://projsight.com/lp/faq/) をご覧ください。

## Before you start

The plugin talks to a hosted service, so you need an account and a project first:

1. Sign up at [projsight.com](https://projsight.com/auth/signup/). The Free plan needs no credit card.
2. Create a project on the web dashboard.
3. Note the project ID. It is in the dashboard URL and on the project overview page.

## Install

### Claude Code

```bash
claude plugin marketplace add novel-jp/projsight-plugin
claude plugin install projsight
```

Then run `/mcp` in Claude Code, pick `projsight`, and sign in with your ProjSight account in the browser window that opens.

### Cursor

```
/add-plugin novel-jp/projsight-plugin
```

### Tell the agent which project to use

Add the project ID to your repository instructions (`CLAUDE.md` for Claude Code, `.cursor/rules/projsight.md` for Cursor):

```markdown
## ProjSight

- **Project ID**: `<your-project-id>`
```

After that you can work in plain language, for example "start this task" or "the work is done". The agent records the task, its progress, and any decisions in ProjSight.

## What the plugin sets up

| Part            | What it is                                                                                                   |
| --------------- | ------------------------------------------------------------------------------------------------------------ |
| MCP server      | A remote MCP server at `https://projsight.com/mcp` (Streamable HTTP, OAuth sign-in). Nothing runs locally.   |
| Skills          | Guided workflows such as `start-task`, `design-review`, `product-review`, `health-check`, and `risk-review`. |
| Workflow rules  | `rules/projsight-workflow.md`, which tells the agent when to record tasks, decisions, risks, and issues.     |
| Learning skills | The `learn-*` skills, a hands-on curriculum for working with AI coding agents on a ProjSight project.        |

## MCP tools

The default connection exposes 38 tools:

| Area                             | Tools                                                                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Projects and context             | `list_projects`, `get_project_context`, `list_orgs`, `list_members`, `list_phases`, `list_milestones`                    |
| Tasks and work                   | `upsert_task`, `list_tasks`, `start_work`, `complete_work`, `cancel_work`, `attach_pr`, `detach_pr`                      |
| Deliverables                     | `upsert_deliverable`, `list_deliverables`                                                                                |
| Decision records and design docs | `upsert_dr`, `list_drs`, `get_dr`, `add_design_doc`                                                                      |
| Risks                            | `upsert_risk`, `get_risk`, `list_risks`                                                                                  |
| Issues                           | `upsert_issue`, `get_issue`, `list_issues`                                                                               |
| Questions                        | `upsert_question`, `answer_question`, `add_question_comment`, `list_question_comments`, `get_question`, `list_questions` |
| Links between records            | `create_link`, `remove_link`, `list_links`                                                                               |
| Attachments                      | `attach_file`, `list_attachments`, `read_attachment`, `delete_attachment`                                                |

Tech leads and administrators can add a second connection for management operations such as inviting members, changing scope, and managing phases:

```bash
claude mcp add --transport http --scope project projsight-admin "https://projsight.com/mcp?profile=admin"
```

Run `/mcp` again and sign in to `projsight-admin` after adding it.

## Access and roles

Every tool call is checked against your project membership. Roles are `owner`, `admin`, `member`, and `viewer`; you only see and change projects you belong to.

## Plans

- **Free**: 1 project and up to 5 organization members.
- **Pro**: unlimited projects and members, billed per seat.

All MCP tools are available on both plans. See [pricing](https://projsight.com/lp/#pricing) for current prices.

## Data and privacy

Project data is stored on Amazon Web Services in the Tokyo region. See the [privacy policy](https://projsight.com/lp/privacy/) and [terms of service](https://projsight.com/lp/terms/) for details.

## Support

Email [contact-us@support.novel.jpn.com](mailto:contact-us@support.novel.jpn.com).

## License

The contents of this repository (plugin manifests, skills, and rules) are released under the [MIT License](LICENSE). ProjSight itself is a commercial hosted service operated by Novel Inc.
