# Screen Time

Daily budgets, bedtime, parent controls, and optional arithmetic rewards.

A community plugin for **Omarchy Quattro with the Quickshell plugin system**. It works on a regular Omarchy installation; an Omarchy Kids ISO or fork is not required. The plugin ID is `io.github.peterholko.screen-time`.

## Install

Run these commands in the intended user's Omarchy desktop session:

```bash
omarchy plugin add https://github.com/peterholko/omarchy-screen-time --enable
omarchy bar put io.github.peterholko.screen-time --section right
```

### Set up the background service

Adding the shell plugin alone does not install or authorize a privileged service. Review `setup` and `service/`, then run this in a terminal. Replace `CHILD_USERNAME` with the local account to enroll (for example, `linnea`):

```bash
omarchy pkg add python
sudo "$HOME/.config/omarchy/plugins/io.github.peterholko.screen-time/setup" --user CHILD_USERNAME
```

Setup asks for a new **controls parent password** of at least eight characters. Screen Time and School Mode share this password and the `omarchy-kids-controls.service` service. Setup copies only this repository's local, reviewed payload; it does not download code. Installing the second plugin preserves the first plugin's settings and enrollments. Use matching plugin releases; mismatched service versions require an explicit `--upgrade`, and unknown files or locally modified installed service files stop setup.

Only the named account is enrolled. Root owns the password hash, schedules, budgets and reward checks. The UI sends passwords over stdin, and the local service authenticates callers by their Unix socket peer credentials. It rate limits failed parent-password attempts. The controls password is separate from the login, administrator and disk passwords.

These are desktop controls for a cooperative family setup. An account that retains administrator access can disable the service, and user-controlled shell plugins are not an application sandbox. This installer does not convert or demote OS accounts. It refuses to enroll an account already configured for the original Omarchy Kids backend, to prevent two services enforcing different policies.

Click the screen-time widget to inspect remaining time, configure weekday budgets and bedtime, or grant extra time with the controls parent password. Only active, unlocked graphical-session time is counted. School Mode, when separately enabled for the account, pauses the free-time budget during school hours.

Install [Math Time](https://github.com/peterholko/omarchy-math-time) to earn time through arithmetic. Without it, daily budgets and bedtime still work. At zero time the service locks the session. After a normal OS unlock, the Screen Time plugin opens Math Time; the service retains its lock fallback if the overlay cannot open. The controls parent password can grant a short bypass in Math Time. The stock OS lock screen continues to use the account's normal unlock credentials.

### Manage enrollment and password

```bash
sudo omarchy-kids-controls password
sudo omarchy-kids-controls disable time --user CHILD_USERNAME
sudo omarchy-kids-controls enable time --user CHILD_USERNAME
```

### Service paths and dependencies

The shared service uses Python 3's standard library, systemd/logind and Omarchy's shell/lock/notification commands. School desktop effects also use Bash 5, Hyprland's Lua IPC, jq and flock, supplied by Omarchy. No pip packages, network services or API keys are needed.

- Code: `/usr/lib/omarchy-kids-controls/`
- Commands: `/usr/bin/omarchy-kids-controls` and `omarchy-kids-controls-{time,school,grove}-client`
- Unit: `/etc/systemd/system/omarchy-kids-controls.service`
- Private configuration and password: `/etc/omarchy-kids-controls/`
- Private service state and per-user read-only status: `/var/lib/omarchy-kids-controls/`
- Local socket: `/run/omarchy-kids-controls/sock`

## Update

```bash
omarchy plugin update io.github.peterholko.screen-time
```

Review any service changes, then update the root-owned copy explicitly:

```bash
sudo "$HOME/.config/omarchy/plugins/io.github.peterholko.screen-time/setup" --user CHILD_USERNAME --upgrade
```

Updating the user-owned shell checkout never silently replaces the installed privileged service.

## Remove

Disable this module's enrollments and remove its service installation before removing the shell plugin:

```bash
sudo omarchy-kids-controls remove time
omarchy plugin remove io.github.peterholko.screen-time
```

If the other module is installed, its shared service, password and settings remain. Removing the last module stops and removes the service, unit and owned command wrappers. Configuration, password and history are retained for a deliberate reinstall; inspect `/etc/omarchy-kids-controls/` and `/var/lib/omarchy-kids-controls/` before deleting that data yourself. Modified or unexpected installed files stop automatic removal for review.

## License and source

MIT. See [LICENSE](LICENSE) and [ATTRIBUTION.md](ATTRIBUTION.md) for retained copyright notices and asset provenance. [SOURCE.json](SOURCE.json) records the source revision and reproducible exporter in [Omarchy Kids](https://github.com/peterholko/omarchy-kids).

## Validation

```bash
omarchy plugin validate .
```

These packages are checked with the upstream manifest validator and local source tests. Full desktop enforcement, systemd installation and removal require validation on an actual Omarchy laptop. There are no GitHub Actions workflows in this repository.
