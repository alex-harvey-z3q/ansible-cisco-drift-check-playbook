# Ansible + Cisco DevNet Sandbox – Config Drift Detection

This project sets up a minimal Python virtual environment and Ansible configuration
to compare a Cisco IOS-XE device running configuration against a golden config.

If any drift is detected, the playbook fails.

---

## Prerequisites

- macOS (or Linux)
- Homebrew (macOS only)
- Python 3
- Working SSH access to the Cisco DevNet Sandbox device

---

## Initial Setup

Create the virtual environment and install Ansible:

    make

This will create a virtual environment (venv).

Install required Ansible collections:

    ansible-galaxy collection install cisco.ios ansible.netcommon

---

## Configure LibSSH (macOS only)

On macOS, ansible-pylibssh requires Homebrew libssh headers.

    export LIBSSH_PREFIX="$(brew --prefix libssh)"
    export CPPFLAGS="-I${LIBSSH_PREFIX}/include"
    export LDFLAGS="-L${LIBSSH_PREFIX}/lib"
    export PKG_CONFIG_PATH="${LIBSSH_PREFIX}/lib/pkgconfig"

---

## Create Sandbox in Cisco DevNet

1. Log in:
   https://developer.cisco.com/site/sandbox/

2. Launch Sandbox
3. Launch the Catalyst 8000 Always-on Sandbox
4. Once provisioning finishes, click on the provisioned device
5. Write down the hostname, username, and password

Export these:

    export CISCO_HOST=devnetsandboxiosxec8k.cisco.com
    export CISCO_PASS=xxxxxxxxxxxx
    export CISCO_USER=alexharv074

---

## Project Structure

    .
    ├── inventory.yml
    ├── check_config_drift.yml
    └── goldens/
        └── sandbox.cfg   (your golden config)

---

## Create Initial Golden Config

To make the current running config your golden baseline:

    ansible -i inventory.yml sandbox \
      -m cisco.ios.ios_command \
      -a '{"commands":["show running-config"]}' \
      -o | awk -F'=> ' '{print $2}' | jq -r '.stdout[0]' > goldens/sandbox.cfg

---

## Run Drift Check

    ansible-playbook -i inventory.yml check_config_drift.yml --diff

If no drift exists → playbook succeeds.

If drift exists → diff is shown and playbook fails.

---

## Forcing a Test Drift

Edit goldens/sandbox.cfg and add a harmless line such as:

    banner motd ^THIS IS A TEST BANNER^

Re-run the playbook. It should now fail with drift detected.

---

## Licence

MIT
