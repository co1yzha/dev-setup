### Steps

- Create a new SSH key (ed25519)
```bash
ssh-keygen -t ed25519 -a 100 -C "<note>" -f ~/.ssh/id_ed25519_<project>
```
- Start agent and add the key (macOS)
```bash
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_<project>
```

- Copy the public key to the server
  - If ssh-copy-id is available:
    ```bash
    ssh-copy-id -i ~/.ssh/id_ed25519_<project>.pub <username>@<server_ip_or_dns>
    ```
  - If not, use this fallback:
    ```bash
    cat ~/.ssh/id_ed25519_<project>.pub | ssh <username>@<server_ip_or_dns> 'mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys'
    ```

- Configure your `~/.ssh/config`
```sshconfig
Host <host_alias>
  HostName <server_ip_or_dns>
  User <username>
  IdentityFile ~/.ssh/id_ed25519_<project>



  <!-- 
  IdentitiesOnly yes
  AddKeysToAgent yes
  UseKeychain yes
  PubkeyAuthentication yes 
  -->
```

- Test
```bash
ssh <host_alias>
```

Notes:
- Replace placeholders: `<project>`, `<note>`, `<host_alias>`, `<server_ip_or_dns>`, `<username>`.
- The first connection will still ask for the server password to install the key; subsequent logins should be passwordless.
- If login still prompts for a password, verify remote permissions:
  - On the server: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`
  - Ensure `PubkeyAuthentication yes` is enabled in `/etc/ssh/sshd_config` (server-side) and restart sshd if changed.