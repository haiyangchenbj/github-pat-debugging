## Description:

Diagnoses GitHub Personal Access Token authentication failures by checking command syntax, environment-variable propagation, token-file formatting, and independent client responses before investigating permissions or revocation.

This skill is ready for commercial/non-commercial use.

## Publisher:

[haiyangchenbj](https://clawhub.ai/user/haiyangchenbj)

### License/Terms of Use:

MIT-0

## Use Case:

Developers and engineers use this skill to diagnose GitHub PAT failures across shells, runtimes, and GitHub API requests. It helps distinguish local token-handling problems from permissions, revocation, or account-policy issues before making repository changes.

### Deployment Geography for Use:

Global

## Known Risks and Mitigations:

Risk: The workflow includes repository changes that may be insufficiently scoped.

Mitigation: Keep troubleshooting read-only unless the user explicitly authorizes the exact repository change; confirm the target account and token file first.

Risk: An exposed access token may remain active during deployment if replacement is delayed.

Mitigation: Replace exposed tokens before continuing deployment, and check dependent integrations before revoking a token.

## Reference(s):

- [GitHub PAT Debugging on ClawHub](https://clawhub.ai/haiyangchenbj/skills/github-pat-debugging)
- [Token Failure Mode Decision Tree](artifact/references/token-failure-modes.md)

## Skill Output:

**Output Type(s):** [Guidance, Shell commands, API calls]

**Output Format:** [Markdown with inline shell commands and code examples]

**Output Parameters:** [1D]

**Other Properties Related to Output:** [Troubleshooting guidance; do not print or expose full token values.]

## Skill Version(s):

1.0.5 (source: frontmatter)

## Ethical Considerations:

Users should evaluate whether this skill is appropriate for their environment, review any generated or modified files before relying on them, and apply their organization's safety, security, and compliance requirements before deployment.
