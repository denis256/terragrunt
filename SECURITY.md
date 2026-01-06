# Security

## Verifying Release Signatures

Starting from version X.X.X, Terragrunt releases include cryptographic signatures for the checksums file. You can verify downloads using either GPG or Cosign.

### GPG vs Cosign: Understanding the Difference

| Aspect | GPG (GNU Privacy Guard) | Cosign (Sigstore) |
|--------|-------------------------|-------------------|
| **Type** | Traditional PGP/OpenPGP | Modern keyless signing |
| **How it works** | Uses asymmetric key pairs (public/private) | Uses OIDC identity + transparency log |
| **Key management** | You must import our public key | No key needed - verifies via Sigstore |
| **Trust model** | Web of trust / manual key verification | GitHub Actions OIDC identity verification |
| **Verification tool** | `gpg --verify` (pre-installed on most Linux) | `cosign verify-blob` (requires install) |
| **Offline capable** | Yes - once you have the public key | No - requires internet to check transparency log |
| **Output files** | `.gpgsig` (detached signature) | `.sig` (signature) + `.pem` (certificate) |
| **Best for** | Traditional verification, air-gapped systems | CI/CD pipelines, automated verification |

### How GPG Signing Works

1. **Key generation**: A 4096-bit RSA key pair is created
2. **Private key**: Stored as GitHub secret, used to create signatures
3. **Public key**: Published in repo, users import it to verify
4. **Signing**: `gpg --detach-sign` creates `.gpgsig` file
5. **Verification**: User runs `gpg --verify` with our public key

### How Cosign (Sigstore) Works

1. **No static keys**: Uses ephemeral keys generated per-signing
2. **OIDC identity**: GitHub Actions provides identity token proving the workflow ran in our repo
3. **Transparency log**: Signature recorded in Rekor (public audit log)
4. **Certificate**: `.pem` contains X.509 cert with workflow identity
5. **Verification**: Cosign checks cert chain + transparency log entry

### Download Verification Files

```bash
VERSION=vX.X.X
BASE_URL="https://github.com/gruntwork-io/terragrunt/releases/download/${VERSION}"

# Download binary and verification files
curl -sLO "${BASE_URL}/terragrunt_linux_amd64"
curl -sLO "${BASE_URL}/SHA256SUMS"
curl -sLO "${BASE_URL}/SHA256SUMS.gpgsig"
curl -sLO "${BASE_URL}/SHA256SUMS.sig"
curl -sLO "${BASE_URL}/SHA256SUMS.pem"
```

### Verify with GPG

```bash
# Import Terragrunt public key (one time)
curl -sL https://raw.githubusercontent.com/gruntwork-io/terragrunt/master/.github/keys/terragrunt-release-key.asc | gpg --import

# Verify the signature
gpg --verify SHA256SUMS.gpgsig SHA256SUMS

# If valid, verify the binary checksum
sha256sum -c SHA256SUMS --ignore-missing
```

**Expected output:**

```text
gpg: Signature made ...
gpg: Good signature from "Terragrunt Release Signing Key <security@gruntwork.io>"
```

### Verify with Cosign

```bash
# Install cosign if needed: https://docs.sigstore.dev/cosign/system_config/installation/

# Verify the signature (no key import needed)
cosign verify-blob SHA256SUMS \
  --signature SHA256SUMS.sig \
  --certificate SHA256SUMS.pem \
  --certificate-identity-regexp=".*github.com/gruntwork-io/terragrunt.*" \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com

# If valid, verify the binary checksum
sha256sum -c SHA256SUMS --ignore-missing
```

**Expected output:**

```text
Verified OK
```

---

# Reporting Security Issues

Gruntwork takes security seriously, and we value the input of independent security researchers. If you're reading this because you're looking to engage in responsible disclosure of a security vulnerability, we want to start with thanking you for your efforts. We appreciate your work and will make every effort to acknowledge your contributions.

To report a security issue, please use the GitHub Security Advisory ["Report a vulnerability"](https://github.com/gruntwork-io/terragrunt/security/advisories/new) button in the ["Security"](https://github.com/gruntwork-io/terragrunt/security) tab.

After receiving the report, we will investigate the issue and inform you of next steps. After the initial reply, we may ask for additional information, and will endeavor to keep you informed of our progress.

If you are reporting a bug related to an associated tool that Terragrunt integrates with, we ask that you report the issue directly to the maintainers of that tool.

Please do not disclose the issue publicly until we have had a chance to address it.

## Expectations on timelines

You can expect that Gruntwork will take any report of a security vulnerability seriously, but we ask that you also respect that it can take time to investigate and address issues given the size of the team maintaining Terragrunt. We will do our best to keep you informed of our progress, and provide insight into the timeline for addressing the issue.

## Thank you

We appreciate your help in making Terragrunt more secure. Thank you for your efforts in responsibly disclosing security issues, and for your patience as we work to address them.
