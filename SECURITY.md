# Security policy

This registry only contains manifests: links to release files, plus their checksums. The security risk is a manifest that points users at a **malicious or compromised** file.

## Reporting a problem

Please **don't open a public issue**. Report it privately instead: go to this repository's **Security** tab and click **Report a vulnerability**.

Report it if, for example:
- a package's release file was replaced or tampered with upstream;
- a manifest points at a file that isn't from the package's real project;
- a package itself is malicious.

Include the package name, the version, and what you found. A compromised entry is removed or reverted first, and investigated after.

Security problems in the hako program itself belong in [hako-source](https://github.com/j-kop/hako-source/security/advisories/new).
