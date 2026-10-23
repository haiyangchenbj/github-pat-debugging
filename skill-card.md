## Description:

Helps diagnose GitHub personal access token authentication failures by checking request details, credential handling, and independent client results before attributing failures to token status or permissions.

This skill is ready for commercial/non-commercial use.

## Publisher:

[haiyangchenbj](https://clawhub.ai/user/haiyangchenbj)

### License/Terms of Use:

MIT-0

## Use Case:

Developers and engineers troubleshooting GitHub API authentication and repository publishing use this skill to distinguish local credential-passing mistakes from permission, revocation, and repository-visibility issues.

### Deployment Geography for Use:

Global

## Known Risks and Mitigations:

Risk: A GitHub personal access token could be exposed in chat or logs during troubleshooting.

Mitigation: Do not paste or print full tokens; rotate any token that has already been exposed.

Risk: A diagnostic command could send credentials to an unintended endpoint.

Mitigation: Confirm that credential-bearing commands target GitHub-owned endpoints before running them.

## Reference(s):

- [GitHub PAT Debugging on ClawHub](https://clawhub.ai/haiyangchenbj/skills/github-pat-debugging)
- [Token Failure Mode Decision Tree](references/token-failure-modes.md)

## Skill Output:

**Output Type(s):** [Guidance, Shell commands, Code]

**Output Format:** [Markdown with diagnostic steps and command examples]

**Output Parameters:** [1D]

**Other Properties Related to Output:** [Recommends comparing independent authentication checks without disclosing the full token.]

## Skill Version(s):

1.0.4 (source: release evidence, frontmatter)

## Ethical Considerations:

Users should evaluate whether this skill is appropriate for their environment, review any generated or modified files before relying on them, and apply their organization's safety, security, and compliance requirements before deployment.
