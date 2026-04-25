# Security Policy

## Overview

This document outlines security practices and mitigations implemented to protect against attacks targeting GitHub Personal Access Tokens (PATs) and cross-cloud lateral movement, as documented in [Wiz Research on GitHub PAT Attacks](https://www.wiz.io/blog/github-attacks-pat-control-plane).

## Threat Model

### Primary Threats

1. **GitHub PAT Compromise**: Leaked or stolen Personal Access Tokens used for cross-cloud lateral movement
2. **GitHub Actions Supply Chain Attacks**: Compromised third-party actions exfiltrating secrets
3. **Secret Exposure**: Credentials stored in code repositories or workflow files
4. **Insufficient Access Controls**: Overly permissive workflow permissions

### Attack Vectors

- 45% of organizations store plaintext cloud keys in private repositories
- 73% of organizations store CSP credentials in GitHub Action Secrets
- Audit log gaps: code-search API calls are not logged
- GitHub Actions can access secrets and move laterally to cloud control planes

## Security Controls Implemented

### 1. GitHub Actions Hardening

#### Pinned Action Versions
All GitHub Actions are pinned to specific commit SHAs (not mutable tags) to prevent supply chain attacks:
- ✅ Protects against compromised action repositories
- ✅ Ensures reproducible builds
- ✅ Allows controlled updates with security review

#### Minimal Permissions
Workflows follow the principle of least privilege:
- `permissions:` blocks explicitly define required permissions
- Read-only by default
- Write permissions only when necessary and scoped

#### Secret Access Controls
- Secrets are never logged or echoed
- Workflow conditionals restrict execution to authorized users
- Environment-based secret isolation where possible

### 2. Secret Management

#### Current State
- ❌ **HIGH RISK**: PAT stored in GitHub secrets (`secrets.PAT`)
- ❌ **HIGH RISK**: KUBECONFIG stored in GitHub secrets
- ⚠️ **MEDIUM RISK**: Tailscale OAuth credentials in secrets
- ⚠️ **MEDIUM RISK**: Claude Code OAuth token in secrets

#### Recommended Mitigations

**Immediate Actions:**
1. **Rotate all GitHub PATs** and scope them minimally:
   - Use fine-grained PATs instead of classic PATs
   - Scope to specific repositories only
   - Grant only required permissions (e.g., `contents:write` only)
   - Set expiration dates (max 90 days)

2. **Implement OIDC for Cloud Access**:
   - Replace long-lived credentials with OpenID Connect
   - Use GitHub's OIDC provider for AWS/Azure/GCP authentication
   - Eliminates need to store cloud credentials in secrets

3. **Use Environment Secrets**:
   - Move secrets to environment-specific protection rules
   - Require manual approval for production deployments
   - Add deployment branch restrictions

4. **Implement SOPS or External Secrets Operator**:
   - Encrypt secrets at rest in the repository
   - Use cloud KMS for key management
   - Limit decryption keys to production environments only

**Long-term Strategy:**
- Deploy HashiCorp Vault or similar secret management
- Implement short-lived, dynamically generated credentials
- Use Kubernetes service accounts with IRSA/Workload Identity
- Enable audit logging for all secret access

### 3. Code Repository Security

#### Implemented Controls
- ✅ `.gitignore` configured to exclude secret files
- ✅ No hardcoded credentials in codebase (verified via automated scanning)
- ✅ Kustomize validation in CI pipeline

#### Recommended Additional Controls
- [ ] Enable GitHub secret scanning (free for public repos)
- [ ] Implement pre-commit hooks with tools like:
  - `gitleaks` - Detect secrets in commits
  - `tfsec` - Security scanning for IaC
  - `trivy` - Vulnerability scanning
- [ ] Enable GitHub push protection to prevent secret commits
- [ ] Implement branch protection rules requiring:
  - Code review before merge
  - Status checks to pass
  - Signed commits

### 4. Workflow Security Best Practices

#### User Authorization
Workflows use conditional execution to restrict access:
```yaml
if: (github.event.sender.id == '12449626')
```

**Recommendation**: Replace hardcoded user IDs with GitHub team membership checks:
```yaml
if: contains(fromJSON('["OWNER","MEMBER"]'), github.event.sender.association)
```

#### Network Security
- Workflows accessing internal infrastructure use Tailscale VPN
- Kubernetes access requires valid kubeconfig
- Consider implementing:
  - IP allowlisting for sensitive operations
  - Network policies in Kubernetes
  - mTLS for service-to-service communication

### 5. Audit and Monitoring

#### Current Gaps
- Limited visibility into secret access patterns
- No alerting on suspicious workflow behavior
- GitHub audit log gaps (code-search API not logged)

#### Recommended Monitoring
1. **Enable GitHub Advanced Security** (if available):
   - Secret scanning alerts
   - Code scanning (CodeQL)
   - Dependency review

2. **Implement SIEM Integration**:
   - Forward GitHub audit logs to centralized SIEM
   - Alert on:
     - Workflow failures accessing secrets
     - New workflow files created
     - Changes to workflow files with secret access
     - Unusual API access patterns

3. **Regular Security Reviews**:
   - Quarterly review of all PATs and their scopes
   - Monthly audit of workflow permissions
   - Review GitHub Actions supply chain (dependabot updates)

## Vulnerability Disclosure

If you discover a security vulnerability in this repository:

1. **DO NOT** open a public issue
2. Email security details to repository maintainers
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested remediation (if any)

## Security Checklist for Contributors

Before submitting a PR:
- [ ] No secrets or credentials committed
- [ ] GitHub Actions pinned to commit SHAs
- [ ] Minimal workflow permissions requested
- [ ] New secrets documented in this file
- [ ] Security implications of changes considered

## Compliance

This repository implements security controls aligned with:
- **CIS Kubernetes Benchmark**
- **NIST Cybersecurity Framework**
- **GitOps Security Best Practices**

## References

- [Wiz Research: GitHub PAT Control Plane Attacks](https://www.wiz.io/blog/github-attacks-pat-control-plane)
- [GitHub Actions Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [FluxCD Security Best Practices](https://fluxcd.io/flux/security/)
- [OWASP CI/CD Security Risks](https://owasp.org/www-project-top-10-ci-cd-security-risks/)

## Last Updated

December 2025 - Initial security assessment and hardening
