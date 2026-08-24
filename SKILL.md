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
  中文摘要：GitHub PAT 认证排障。先查命令、环境变量传递与请求参数，再用 curl/Node/Python
  交叉验证，最后才判断权限、撤销或网络原因。触发词：GitHub token 失效排查、401 Bad
  credentials、PAT 认证失败、推送失败诊断.
description_zh: GitHub PAT 认证排障——先查命令与环境变量传递，再交叉验证，最后才判断 token 状态
description_en: GitHub PAT Debugging
version: 1.0.2
agent_created: true
not_for:
  - GitHub Actions workflow failures (different failure domain)
  - SSH key authentication issues (PAT is HTTPS token-based)
  - OAuth app or GitHub App token flows (different token types)
  - Rate limiting (HTTP 429) rather than authentication (HTTP 401) errors
  - Git merge or rebase conflicts (not auth-related)
---

# github-pat-debugging

## When to use
- A GitHub API or Contents API request returns `401 Bad credentials`.
- A token is shown as active or non-expiring in GitHub, or another GitHub workflow has just succeeded.
- Different runtimes or shells are being mixed, especially Bash, Node.js, Python, PowerShell, curl, or Git.

## Steps

> Diagnostic skill: steps 1–3 are `[Deterministic]` (shell commands and file inspection); steps 4–8 mix `[Deterministic]` probes with `[LLM]` interpretation.

1. **[LLM]** Do not conclude that the token is expired or revoked from one failed request. Record the exact endpoint, HTTP status, auth scheme, and runtime.
2. **[Deterministic]** Inspect the token file without printing the token: byte count, prefix, suffix, and trailing newline. Do not expose the full secret.
3. **[Deterministic]** Test the same token with an independent client. In Bash, use direct expansion for curl:
   ```bash
   TOKEN=$(cat "$HOME/.github-token")
   curl -sS -D - -o /dev/null \
     -H "Authorization: Bearer $TOKEN" \
     -H "User-Agent: token-probe" \
     -H "Accept: application/vnd.github+json" \
     https://api.github.com/user
   ```
4. **[Deterministic]** When handing the token to a child process, export it explicitly. This is a critical Bash distinction:
   - Wrong for a later command: `TOKEN=$(cat file) && node script.js` (shell variable is not exported).
   - Correct: `export TOKEN="$(cat file)" && node script.js`.
   - Also correct for one process: `TOKEN="$(cat file)" node script.js`.
5. **[Deterministic]** In Node.js, check `process.env.TOKEN` only as a boolean/presence signal; never print the value. Test both `Bearer` and `token` schemes if needed.
6. **[LLM]** Compare the results. If curl is `200` with `X-OAuth-Scopes` and Node is `401`, inspect environment propagation before token state, proxy, or GitHub account hypotheses.
7. **[LLM]** Only after independent clients using the same secret both fail, investigate GitHub-side causes using the failure-mode decision tree in `references/token-failure-modes.md`: manual deletion/revocation, secret-scanning revocation, third-party credential revocation, OAuth-app token limits, organization/enterprise policy, or expiration.
8. **[Deterministic]** After fixing the auth path, fetch the current remote blob SHA, update through the Contents API with the SHA, and verify the raw file contains the intended content.

## Hard Rules

1. Never conclude token expiry or revocation from a single failed request in a single runtime.
2. Never print, commit, log, or paste a full PAT; inspect only byte count, prefix, and suffix.
3. Local causes (variable propagation, wrong shell syntax, unset env) are ruled out before any GitHub-side hypothesis.
4. Two independent clients must fail with the same secret before investigating revocation.
5. A token exposed in logs or chat is rotated regardless of whether it still works.

## Pitfalls
- `VAR=value command` exports the variable only to that command; `VAR=value && command` does not export it to the later command.
- A `401` from Node with `process.env.TOKEN` unset is a local process bug, not evidence of a revoked PAT.
- `Never used` or a stale "last used" label is weaker evidence than a live authenticated `GET /user`; use the latter for runtime validation.
- Do not print, commit, or paste a full PAT. If a token has been exposed, rotate it after completing the needed deployment.
- Do not overwrite a remote file without first retrieving its current SHA.

## Failure Handling

| Scenario | Action |
|---|---|
| curl succeeds but Node fails | Environment propagation bug — inspect `export` usage before touching the token |
| Both clients fail with 401 | Walk `references/token-failure-modes.md` decision tree; check security log events |
| Token file unreadable or empty | Fix file access first; an unreadable token is not a revoked token |
| Intermittent failures | Suspect proxy, rate limiting, or SSO enforcement before token state |
| Token confirmed exposed | Rotate immediately after completing the critical deployment |

## Output Format

```markdown
# GitHub PAT Diagnosis

## 1. Symptom (endpoint, status, runtime, exact command form)
## 2. Local-cause check (variable propagation, env, shell syntax)
## 3. Cross-validation results (curl / Node / Python, status codes)
## 4. Root cause (with evidence strength per references/token-failure-modes.md)
## 5. Fix applied (exact command correction)
## 6. Post-fix verification (GET /user 200 + write SHA + raw file check)
```

## Verification
- Run an authenticated `GET /user` with the corrected runtime and confirm HTTP 200 plus the expected login, without printing the token.
- Confirm the write response is HTTP 200/201 and record only the commit SHA.
- Read the public raw file and verify the new marker is present and stale markers are absent.
- Record the precise root cause and command correction in the project log.
