## Description
<!-- Provide a brief description of the changes in this PR -->

## Type of Change
<!-- Mark the relevant option with an 'x' -->

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Infrastructure change (changes to cluster configuration, Flux resources, etc.)
- [ ] Documentation update
- [ ] Security improvement

## Security Checklist
<!-- All items must be checked before merging -->

- [ ] No secrets or credentials are included in this PR
- [ ] No hardcoded API keys, tokens, or passwords
- [ ] GitHub Actions are pinned to commit SHAs (if applicable)
- [ ] Minimal permissions requested for workflows (if applicable)
- [ ] Changes follow the principle of least privilege
- [ ] Security implications have been considered and documented

## Testing
<!-- Describe the tests you ran to verify your changes -->

- [ ] Local Kustomize build validation (`kubectl kustomize`)
- [ ] Flux validation script (`./scripts/validate.sh`)
- [ ] Tested in development environment
- [ ] End-to-end testing completed

## Deployment Impact
<!-- Describe any impact this change might have on the running cluster -->

- [ ] This change requires cluster reconciliation
- [ ] This change may cause service interruption
- [ ] This change updates critical infrastructure components
- [ ] No impact on running services

## Checklist

- [ ] My code follows the style guidelines of this project
- [ ] I have performed a self-review of my own code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings or errors
- [ ] I have reviewed the SECURITY.md file and followed security best practices

## Related Issues
<!-- Link any related issues here -->

Closes #
