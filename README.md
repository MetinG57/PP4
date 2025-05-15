# PP4

## Goal

In this exercise you will:

* Use SSH to connect to remote servers from WSL, macOS, or Linux shells, understanding the handshake and authentication process.
* Generate an Ed25519 SSH key pair and explain the concept of digital signatures.
* Configure your local SSH client via the `~/.ssh/config` file for streamlined access.
* Securely copy files between local and remote hosts using `scp`, including local-to-remote, remote-to-local, and remote-to-remote transfers.
* Automate startup tasks on the remote server by writing a shell script that runs at login and explaining the role of `~/.bashrc` vs. `~/.profile`.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. Once time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository
2. **Modify & commit** your solution
3. **Submit your link for Review**

---

## Prerequisites

* Several starter repos are available here:
  [https://github.com/orgs/STEMgraph/repositories?q=SSH%3A](https://github.com/orgs/STEMgraph/repositories?q=SSH%3A)
* Consult the SSH and SCP man-pages for detailed options and explanations:

  * `man ssh`
  * `man scp`

---

## Tasks

### Task 1: SSH Login

**Objective:** Establish an SSH connection and observe each stage of the process.

1. From your local shell (WSL, macOS Terminal, or Linux), log into the `vorlesungsserver` (or any other remote machine of your choice, e.g. your own raspberry pi):

   ```bash
   ssh youruser@remotehost
   ```
2. Carefully observe and note each step:

   * **TCP connection** to port 22 on `remotehost`.
   * **SSH protocol handshake**: key exchange and algorithm negotiation.
   * **Authentication**: public-key or password exchange.
   * **Shell allocation**: your remote session starts.
3. After login, exit the session with `exit`.

**Provide:**

```bash
# 1) The exact ssh command you ran
# 2) A detailed, step-by-step explanation of what happened at each stage
```
Task 1
1) bash
    ssh -v metin@localhost
Ich habe diesen Befehl in meinem WSL-Terminal (Ubuntu) ausgeführt. Da ich noch keinen SSH-Schlüssel eingerichtet habe, wurde ich zur Passworteingabe aufgefordert.
---
2)
# SSH Connection to Remote Server – Step-by-Step Explanation

## Step 1. SSH Command Used
Beim Ausführen des Befehls stellte mein Computer eine Verbindung zu localhost (IP-Adresse 127.0.0.1) auf Port 22 her, dem Standardport für SSH. Dies erfolgt durch einen TCP-Handshake, der aus drei Schritten besteht: SYN, SYN-ACK und ACK. Die Verbindung wurde erfolgreich aufgebaut, wie in der Ausgabe sichtbar:
debug1: Connecting to localhost [127.0.0.1] port 22.
debug1: Connection established.

### Step 2: SSH Protocol Handshake
Client und Server einigten sich auf sichere Algorithmen für Verschlüsselung (z. B. chacha20- poly1305@openssh.com), Schlüsselaustausch (z. B. sntrup761x25519-sha512@openssh.com) und Host-Schlüssel (ssh-ed25519). Der Server sendete seinen Host-Schlüssel, der beim ersten Kontakt akzeptiert wurde, da keine known_hosts-Datei vorhanden war. Anschließend wurde ein gemeinsamer Sitzungsschlüssel zur Verschlüsselung der Kommunikation erzeugt.
Wichtige Ausgabe:
debug1: kex: algorithm: sntrup761x25519-sha512@openssh.com
debug1: kex: host key algorithm: ssh-ed25519

### Step 3: password-based authentication
Ohne eingerichteten SSH-Schlüssel nutzte ich die Passwort-Authentifizierung:
Der Server forderte mein Passwort an:

metin@localhost's password:
Ich gab mein WSL-Benutzerpasswort für metin ein.

Nach Überprüfung des Passworts wurde ich eingeloggt:
Authenticated to localhost ([127.0.0.1]:22) using "password".

### Step 4: Shell Allocation
Nach der Authentifizierung öffnete der Server eine Bash-Shell, und ich war als Benutzer metin eingeloggt. Die Eingabeaufforderung lautete:

metin@LAPTOP-621KHKJV:~$

In dieser Shell konnte ich Befehle ausführen.

### Step 5: Exit the Session
Nach dem Test der Verbindung schloss ich die Sitzung mit:
exit

---
---

### Task 2: Ed25519 Key Pair

**Objective:** Create a secure key pair and explain how digital signatures verify identity.

1. Generate an Ed25519 SSH key pair:

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

   * Accept the default file location (`~/.ssh/id_ed25519`). Or provide the `-f <filepath>` option additionally.
   * Enter a passphrase when prompted (optional).
2. Locate and inspect your `id_ed25519` (private key) and `id_ed25519.pub` (public key).
3. Install your key on the remote machine (e.g. `vorlesungsserver`.
4. Explain in writing:

   * How the **private key** is used to sign challenges.
   * How the **public key** on the server verifies signatures without revealing the private key.
   * Why Ed25519 is preferred (performance, security).

**Provide:**

```bash
# 1) The ssh-keygen command you ran
# 2) The file paths of the generated keys
# 3) Your written explanation (3–5 sentences) of the signature process
```
# SSH Key Generation and Signature Process Explanation

## 1. SSH Key Generation Command Used
1. Verwendeter Befehl
ssh-keygen -t ed25519 -C "metin@LAPTOP-621KHKJV"
Ich führte diesen Befehl in meinem WSL-Terminal aus, um ein Ed25519-Schlüsselpaar zu generieren. Ich akzeptierte den Standardpfad (/home/metin/.ssh/id_ed25519) und setzte keine Passphrase, um die Anmeldung zu vereinfachen.

## 2. File Paths of the Generated Keys
Die Schlüssel wurden wie folgt gespeichert:
Privater Schlüssel: /home/metin/.ssh/id_ed25519
Öffentlicher Schlüssel: /home/metin/.ssh/id_ed25519.pub


## 3. Signature Process – Written Explanation
Der private Schlüssel auf meinem Rechner erzeugt eine digitale Signatur für eine Server-Herausforderung. Der öffentliche Schlüssel in ~/.ssh/authorized_keys prüft diese Signatur, um meine Identität zu bestätigen – ganz ohne den privaten Schlüssel preiszugeben. Dieses Verfahren basiert auf asymmetrischer Kryptografie. Ed25519 wird bevorzugt, da es kompakte, schnelle und sehr sichere Schlüssel durch elliptische Kurven bietet, die gegen viele Angriffe resistent sind.

---

### Task 3: SSH Config File

**Objective:** Simplify SSH commands via `~/.ssh/config`.

1. Open (or create) `~/.ssh/config` in `vim`.
2. Add entries for your hosts, for example:

   ```text
   Host my-remote
       HostName remote.example.com
       User youruser
       IdentityFile ~/.ssh/id_ed25519

   Host backup-server
       HostName backup.example.com
       User backupuser
       Port 2222
       IdentityFile ~/.ssh/id_ed25519_backup
   ```
3. Save and close the file, then test:

   ```bash
   ssh my-remote
   ssh backup-server
   ```
4. Explain:

   * How SSH reads `~/.ssh/config` and matches hosts.
   * The difference between `HostName` and `Host`.
   * How aliases prevent long commands.

**Provide:**

```text
# 1) The full contents of your ~/.ssh/config
# 2) A short explanation (3–4 sentences) of how the config simplifies connections
```
# SSH Config File – Simplifying SSH Connections

## 1. Full Contents of `~/.ssh/config`

```ssh-config
Host local
    HostName 127.0.0.1
    User metin
    Port 22
    IdentityFile ~/.ssh/id_ed25519
Ich habe diese Konfiguration erstellt, um SSH-Verbindungen zu localhost mit dem Alias local zu vereinfachen.

## 2. Explanation – How `~/.ssh/config` Simplifies Connections
Die ~/.ssh/config-Datei speichert vordefinierte Einstellungen für SSH-Verbindungen, die über Aliase aufgerufen werden. Beim Befehl ssh local liest SSH die Datei, erkennt den Host-Eintrag „local“ und nutzt HostName (127.0.0.1), User, Port und IdentityFile für die Verbindung. So ersetzt der Alias lange Befehle wie ssh metin@127.0.0.1 -i ~/.ssh/id_ed25519 und macht die Eingabe effizienter.

Bishierhin geschafft
---

### Task 4: SCP File Transfers

**Objective:** Practice copying files securely using `scp`.

1. **Local → Remote**:

   ```bash
   scp /path/to/localfile.txt youruser@remotehost:~/destination/
   ```
2. **Remote → Local**:

   ```bash
   scp youruser@remotehost:~/remotefile.log ./local_destination/
   ```
3. **Remote → Remote** (between two directories on the same remote host):

   ```bash
   scp -r youruser@remotehost:/path/dir1 youruser@remotehost:/path/dir2
   ```
4. For each command:

   * Verify file timestamps and sizes after transfer, using `ls -la`
   * Note any flags you used (e.g., `-r`, `-P` for port).
5. Explain:

   * How `scp` initiates an SSH session for each transfer.
   * The role of encryption in protecting data in transit.

**Provide:**

```bash
# 1) Each scp command you ran
# 2) Any flags or options used
# 3) A brief explanation (2–3 sentences) of scp’s mechanism
```

---

### Task 5: Login Shell Script & Profile Explanation

**Objective:** Automate commands at login and understand shell initialization files.

1. On the **remote** server, create a script `~/login_tasks.sh` containing at least three commands you find useful (e.g., `echo "Welcome $(whoami)"`, `uptime`, `ls ~/projects`). You may either use `vim` or try the following to create a file from your commandline directely:

   ```bash
   cat << 'EOF' > ~/login_tasks.sh
   #!/usr/bin/env bash
   echo "Welcome $(whoami)! Today is $(date)."
   uptime
   ls ~/projects
   EOF
   chmod +x ~/login_tasks.sh
   ```

> The files content should be something akin to:
> ```bash
> #!/usr/bin/env bash
> echo "Welcome $(whoami)! Today is $(date)."
> uptime
> ls ~/projects
> ```

2. Append to your `~/.bashrc` (or `~/.profile` if using a login shell) a line to source this script on each new session:

   ```bash
   echo "source ~/login_tasks.sh" >> ~/.bashrc
   ```
3. Log out and log back in to trigger the script.
4. Explain:

   * The difference between `~/.bashrc` and `~/.profile` (interactive vs. login shells).
   * Why and when each file is read.
   * How sourcing differs from executing.

**Provide:**

```bash
# 1) The contents of login_tasks.sh
# 2) The lines you added to ~/.bashrc or ~/.profile
# 3) Your explanation (3–5 sentences) of shell init files and sourcing vs. executing
```

---

**Remember:** Stop working after **90 minutes** and record where you stopped.

Total used time:92
