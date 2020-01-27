# Installing

1. setup the python3 virtualenv

    ```bash
    sudo apt install -y python3-virtualenv
    ```

2. Install remaining dependencies

    ```bash
    python3 -m virtualenv --python="$(command -v python3)" .env &&
      source .env/bin/activate &&
      python3 -m pip install -U pip virtualenv &&
      python3 -m pip install -r requirements.txt

      ansible-galaxy install dev-sec.os-hardening
      ansible-galaxy install dev-sec.ssh-hardening
    ```

#  About

This repo is based on a stripped down version of (algo)[https://github.com/trailofbits/algo]