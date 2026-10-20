# Reading the frontend, backend and design sources

Skills that read the app's source code (`test-cases-generator`, `test-script-generator`, `test-script-healer`) follow this file. It's framework-agnostic: the local clone folders come from the **Frontend / backend / design sources** row of the Project profile.

## Where to look

1. **Local clones first.** If the folders from the profile exist, search them.
2. **Otherwise, GitHub.** If the user hasn't pulled the dev repos into this repo:
   - **Ask the user** which GitHub repo (`owner/name`) to use for each source you need (frontend, backend, design), and which **branch**, which should match the release under test. Ask once, and reuse the answers for the rest of the run. If the user isn't sure, you can list the candidates with `gh repo list <owner>`.
   - **Use** the GitHub MCP tools if they're connected, otherwise the `gh` CLI. For example, `gh search code "<label text>" --repo <owner>/<repo>` to find a file, then `gh api repos/<owner>/<repo>/contents/<path>?ref=<branch>` to read one.
3. **Neither available:** tell the user and ask how to proceed. Never guess what the source says.

## Rules

- **Keep it small:** search first, then read only the files or parts you need. Never clone or download a whole repo.
- **Code search only covers the default branch.** Use it to find the file path, then always read the file from the chosen branch (`?ref=<branch>`), because it can differ.
- **Read-only:** never create branches, commits, pull requests, issues or comments.
