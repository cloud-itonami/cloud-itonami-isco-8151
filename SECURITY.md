# Security Policy

This project handles fibre preparing, spinning and winding machine operator
operating workflows. Treat vulnerabilities as potentially high impact even
when the demo data is synthetic — this domain's failure modes include real
entanglement hazard from rotating spindles, drive belts and high-speed
winding mechanisms, and fibre-dust exposure, alongside physical
worker-safety risk.

## Do Not Disclose Publicly

Report privately before opening public issues for:

- credential exposure
- real spinner, mill or operator data exposure
- authorization bypass
- Fibre Preparing, Spinning and Winding Mill Scheduling Coordination Governor bypass
- audit-ledger tampering
- over-disclosure in reports or exports
- unsafe robot action dispatch
- any path that lets a proposal reach a machine-operation-execution
  decision, a mill-safety-clearance decision, or a
  mill-safety-officer-override decision

## Reporting

Use GitHub private vulnerability reporting when available for the repository.
If that is unavailable, contact the repository maintainers through the
cloud-itonami organization before publishing details.

Include:

- affected commit or version
- reproduction steps
- expected and actual behavior
- impact on spinner/mill data, policy enforcement or audit logging
- suggested fix, if known

## Production Guidance

- Store secrets outside Git.
- Keep real spinner/mill/operator data outside this repository.
- Run policy tests before deployment.
- Export and review audit logs regularly.
- Use least privilege for operators and service accounts.
