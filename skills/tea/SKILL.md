---
name: tea
description: Use tea to manage Gitea issues, pull requests, labels, and issue dependencies when the user specifies Gitea or the project tracks the task there. Also use it to troubleshoot tea login and connection failures.
---

# Use tea with Gitea

## Choose the repository and login

Use the host and repository from the issue, pull request (PR), or repository URL the user provides. If the user asks to file an issue without naming a tracker, follow the project's tracking rules. If those rules specify Gitea but leave the repository unspecified, use the Git remote. For a PR across forks, target the repository that will receive the PR.

Use `tea logins list` to choose the login for that host. Pass `--login "$login"` and `--repo "$repo"` to repository commands. Set `$repo` to `owner/repo`.

If the project docs and Git remote name different repositories, run this command for each name. If both names return the same `id` on the same host, use the returned `full_name`. If the IDs differ and the intended write target is still unclear, ask the user.

```sh
tea api --login "$login" --repo "$repo" 'repos/{owner}/{repo}'
```

## Read issues, PRs, and labels

```sh
tea issues list --login "$login" --repo "$repo" --state open --fields index,title,labels
```

Use `tea pulls` for PRs and `tea labels list` for labels. Read every page when the user asks for the full list. Report a failed query if tea exits with a nonzero status or prints an empty table with `Failed to ...` on stderr.

## Write and verify changes

Submit Chinese or multiline bodies from a file. To create an issue, read the full body from a UTF-8 Markdown file and serialize `title` and `body` to a UTF-8 JSON file. Set `$payload` to the JSON file path so tea reads it directly. If the issue needs labels, look them up in the target repository and put their numeric IDs in the `labels` array. Follow the project's label conventions and body templates.

```sh
tea api --login "$login" --repo "$repo" --method POST --data "@$payload" 'repos/{owner}/{repo}/issues'
```

The paths below are relative to `repos/{owner}/{repo}/`.

| Action | Request | Fields to send |
| --- | --- | --- |
| Create a PR | `POST pulls` | `head`, `base`, `title`, `body` |
| Edit an issue or PR | `PATCH issues/<number>` or `PATCH pulls/<number>` | Only the fields to change |
| Comment on an issue or PR | `POST issues/<number>/comments` | `body` |

After a write, issue a separate GET by number or ID and verify the fields you changed. When publishing a body, compare the full remote text with the source file. For a new PR, also check the head branch, base branch, and head commit SHA. For batch writes, verify each item by number.

If a write times out, check whether the change reached the server before retrying. If the write returned a number but the GET fails, keep the number and report which fields still need verification.

## Link PRs and record blockers

- To close an issue that the PR fully resolves, put `Closes #123` on its own line in the PR body as plain text with no trailing punctuation. Use one line per issue. For another repository on the same host, use `Closes owner/repo#123`. Use `Refs #123` for a reference or a partial fix. To confirm that Gitea recorded the closing reference, check the target issue's timeline for the PR's `ref_action=closes`. After merge, read the issue state.
- If issue B blocks issue A, send `POST issues/<blocked-number>/dependencies` using A's number in the path and B's `owner`, `repo`, and `index` in the JSON body. GET the same path and confirm that B appears in A's list of blockers.

## Troubleshoot connections

- If the shell cannot find `tea`, locate the installed executable. On Windows, look for a winget installation of `tea.exe` under `%LOCALAPPDATA%\Microsoft\WinGet\Packages\Gitea.tea_*\`. If Git also fails to find `tea login helper`, add the executable's directory to the current process's PATH.
- If DNS resolution times out and the host requires a proxy, set `HTTP_PROXY` and `HTTPS_PROXY` to the machine's proxy in the process that runs tea. If the request still fails, use stderr to distinguish proxy connection, DNS, and authentication errors.
