---
name: github-pat-debugging
slug: github-pat-debugging
displayName: GitHub PAT Debugging
description: >
  Diagnose GitHub Personal Access Token failures — 401 Bad credentials, push
  failures, tokens that appear valid but fail — before declaring them expired
  or revoked. Checks the command, variable propagation, and request parameters
  first, then cross-validates with curl, Node.js, Python, or PowerShell, and
  only then investigates permission, revocation, or network causes. Covers the
  GitHub REST API, Contents API, and file-push workflows.
description_zh: GitHub PAT 认证排障——先查命令与环境变量传递，再交叉验证，最后才判断 token 状态
description_en: GitHub PAT Debugging
version: 1.0.0
agent_created: true
---

# github-pat-debugging

## When to use
- A GitHub API or Contents API request returns `401 Bad credentials`.
- A token is shown as active or non-expiring in GitHub, or another GitHub workflow has just succeeded.
- Different runtimes or shells are being mixed, especially Bash, Node.js, Python, PowerShell, curl, or Git.

## Steps
1. Do not conclude that the token is expired or revoked from one failed request. Record the exact endpoint, HTTP status, auth scheme, and runtime.
2. Inspect the token file without printing the token: byte count, prefix, suffix, and trailing newline. Do not expose the full secret.
3. Test the same token with an independent client. In Bash, use direct expansion for curl:
   ```bash
   TOKEN=$(cat "$HOME/.github-token")
   curl -sS -D - -o /dev/null \
     -H "Authorization: Bearer $TOKEN" \
     -H "User-Agent: token-probe" \
     -H "Accept: application/vnd.github+json" \
     https://api.github.com/user
   ```
4. When handing the token to a child process, export it explicitly. This is a critical Bash distinction:
   - Wrong for a later command: `TOKEN=$(cat file) && node script.js` (shell variable is not exported).
   - Correct: `export TOKEN="$(cat file)" && node script.js`.
   - Also correct for one process: `TOKEN="$(cat file)" node script.js`.
5. In Node.js, check `process.env.TOKEN` only as a boolean/presence signal; never print the value. Test both `Bearer` and `token` schemes if needed.
6. Compare the results. If curl is `200` with `X-OAuth-Scopes` and Node is `401`, inspect environment propagation before token state, proxy, or GitHub account hypotheses.
7. Only after independent clients using the same secret both fail, investigate GitHub-side causes using the failure-mode decision tree in `references/token-failure-modes.md`: manual deletion/revocation, secret-scanning revocation, third-party credential revocation, OAuth-app token limits, organization/enterprise policy, or expiration.
8. After fixing the auth path, fetch the current remote blob SHA, update through the Contents API with the SHA, and verify the raw file contains the intended content.

## Pitfalls
- `VAR=value command` exports the variable only to that command; `VAR=value && command` does not export it to the later command.
- A `401` from Node with `process.env.TOKEN` unset is a local process bug, not evidence of a revoked PAT.
- `Never used` or a stale "last used" label is weaker evidence than a live authenticated `GET /user`; use the latter for runtime validation.
- Do not print, commit, or paste a full PAT. If a token has been exposed, rotate it after completing the needed deployment.
- Do not overwrite a remote file without first retrieving its current SHA.

## Verification
- Run an authenticated `GET /user` with the corrected runtime and confirm HTTP 200 plus the expected login, without printing the token.
- Confirm the write response is HTTP 200/201 and record only the commit SHA.
- Read the public raw file and verify the new marker is present and stale markers are absent.
- Record the precise root cause and command correction in the project log.
