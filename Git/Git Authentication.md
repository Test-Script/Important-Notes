# Git Authentication Guide

## Introduction

Git often prompts for credentials when using HTTPS without a credential helper, especially since GitHub deprecated password authentication in favor of Personal Access Tokens (PATs) or SSH keys. This guide provides secure methods to authenticate with Git repositories, particularly GitHub, to avoid repeated prompts.

A **Personal Access Token (PAT)** is a secure alternative to passwords, generated in your GitHub account settings under "Developer settings > Personal access tokens."

## Prerequisites

- Git installed on your system.
- A GitHub account.
- For PAT: Generate a token with appropriate scopes (e.g., `repo` for private repositories).
- For SSH: Basic knowledge of SSH key generation.

## Method 1: Git Credential Manager (Recommended for HTTPS)

The Git Credential Manager stores credentials securely in your system's credential store (Windows Credential Manager on Windows, Keychain on macOS, or a file on Linux).

### Steps:

1. Install Git Credential Manager if not already installed (available via Git for Windows or as a separate download).
2. Configure Git to use the credential manager:

   ```bash
   git config --global credential.helper manager
   ```

   On Windows, this uses the Windows Credential Manager. On Linux/macOS, it may use `store` or `cache`.

3. The next Git operation (e.g., push) will prompt for username and PAT once. Enter your GitHub username and PAT.
4. Credentials are stored securely, and you won't be prompted again.

**Note:** On Windows, ensure Git for Windows is installed with the credential manager option.

## Method 2: SSH Keys (Recommended for Security)

SSH keys provide passwordless authentication and are more secure for DevOps environments.

### Steps:

1. **Generate an SSH key:**

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   Press Enter to accept defaults. This creates `id_ed25519` and `id_ed25519.pub` in `~/.ssh/`.

2. **Add the public key to GitHub:**

   - Copy the contents of `~/.ssh/id_ed25519.pub`:

     ```bash
     cat ~/.ssh/id_ed25519.pub
     ```

   - Go to GitHub > Settings > SSH and GPG keys > New SSH key.
   - Paste the key and save.

3. **Update your repository to use SSH:**

   ```bash
   git remote set-url origin git@github.com:YOUR_USERNAME/YOUR_REPO.git
   ```

4. **Test the connection:**

   ```bash
   ssh -T git@github.com
   ```

   You should see a success message.

**Security Note:** Never share your private key. Use a passphrase for added security.

## Method 3: GitHub CLI (`gh`)

The GitHub CLI simplifies authentication with browser-based login.

### Steps:

1. Install GitHub CLI (`gh`) from [github.com/cli/cli](https://github.com/cli/cli).

2. Authenticate:

   ```bash
   gh auth login
   ```

   - Select HTTPS.
   - Choose GitHub.com.
   - Follow the browser prompt to authorize.

3. This configures your Git credential helper automatically.

## Troubleshooting

- **Still prompting?** Check your Git config: `git config --global credential.helper`.
- **PAT issues:** Ensure the token has the correct scopes and hasn't expired.
- **SSH issues:** Verify the key is added correctly and the remote URL is SSH.
- **On Azure DevOps Agents:** Use `System.AccessToken` in pipelines or set up PATs for agents.

## Additional Resources

- [GitHub PAT Documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [SSH Key Setup](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager)