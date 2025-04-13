# `git-sops`

Git integration for the [SOPS](https://github.com/mozilla/sops) secrets manager using Git clean/smudge filters. Automatically decrypt secrets during checkout and re-encrypt them during staging.

Ideal for `.env`, `.yaml`, and similar configuration files that should remain encrypted in Git.

## Features

- Transparent encryption/decryption of tracked secrets
- Configurable via standard `.gitattributes`
- Supports JSON, YAML, INI, dotenv, and binary formats
- Pluggable: works with SOPS backends (e.g., `age`, GCP KMS, AWS KMS)

---

## Quick Start

> [!CAUTION]
> **DO NOT** use this tool unless you know how to test and verify its efficacy in your environment! Your secrets are on the line, and I claim no responsibility.

### Initialize

Assuming you have installed this integration script into your `$PATH` by whichever method you prefer:

```bash
# Create a repository
mkdir my-repo
cd my-repo
git init

# Initialize the Git filter
git-sops init
```

### Populate `.gitattributes`

To enable encryption on specific files, add your desired inclusion patterns to `.gitattributes`:

```gitattributes
*.env filter=crypt
*.yaml filter=crypt
```

### Example usage

When using `encrypted_comment_regex: 'enc!'` in your `sops.yaml`, lines after an `#enc!` comment will be encrypted, and others will be left in plaintext.

Example `.env` file:

```env
MY_DATABASE_USER=myuser

#enc!
MY_DATABASE_PASS=this-is-secret
```

After committing, the password line is encrypted in Git, while the username remains readable:

```env
MY_DATABASE_USER=myuser

#enc!
MY_DATABASE_PASS=ENC[AES256_GCM,data: ...]
```

> [!NOTE]  
> For now, this tool detects whether a file is encrypted by scanning for the string `ENC[`, which is how SOPS marks encrypted values. While a bit fragile, and obviously subject to future improvement, this does **not** affect how SOPS encrypts data—only how this tool identifies files as already encrypted.

---

## Prerequisites

- [`sops`](https://github.com/mozilla/sops) must be installed and available in your `$PATH`
- A valid SOPS config (`sops.yaml`)
  - Default location: `~/.config/sops/sops.yaml`
  - Override using `SOPS_CONFIG=/path/to/config.yaml`

Example `~/.config/sops/sops.yaml`:

```yaml
creation_rules:
  - path_regex: .*\.(env.*|yml.*|yaml.*)
    encrypted_comment_regex: 'enc!'
    age: &default_age >-
      AGE-SECRET-KEY-...
  - age: *default_age
```

---

## Usage

```bash
git-sops init           # Sets up filter and prompts to decrypt
git-sops clean FILE     # Encrypts file before staging
git-sops smudge FILE    # Decrypts file during checkout
```

Git runs the clean and smudge operations automatically based on `.gitattributes`.

---

## Troubleshooting

- **Nothing happens on commit?**  
  Make sure the file is listed in `.gitattributes` and that Git filters are initialized (`init`).
  
- **Not decrypting?**  
  Ensure the file contains SOPS-encrypted data (e.g., `ENC[AES256...`) and that your SOPS config is valid.

- **Filter not triggering?**  
  Run `git config --local --get-regexp 'filter\.crypt\..*'` to confirm the
  filter is set.
