# What

Ansible scripts to deploy a semi-hardended VPN appliance

# Why

This started out as cheap way to create a stable, reliable VPN appliance using WireGuard. There are now many other options for operating your own
WireGuard server, so something like this is likely no longer necessary. However, if you find yourself in need of a standalone VPN appliance, this
may be a viable option for you as it has been for me.

# How

This has been deployed and tested on a [FriendlyElec NanoPi NEO2](https://wiki.friendlyelec.com/wiki/index.php/NanoPi_NEO2) using [Armbian](https://www.armbian.com/nanopi-neo-2/). It
should work on any modern Debian/Ubuntu host though, and would likely work on many other SBCs. During the install, the OS and `sshd` are hardened using [devsec.hardening](https://github.com/dev-sec/ansible-collection-hardening). Also
uses a modified version of [wireguard-install](https://github.com/angristan/wireguard-install) for VPN user management. For users that don't already have a domain registered that they can use to
track the IP of their server, [DuckDNS](https://www.duckdns.org/) can be enabled to support dynamic DNS for free.

# Alternatives

- [pfSense](https://docs.netgate.com/pfsense/en/latest/vpn/wireguard/index.html)
- [OPNsense](https://docs.opnsense.org/manual/how-tos/wireguard-client.html)
- [OpenWrt](https://openwrt.org/docs/guide-user/services/vpn/wireguard/start)
- [Firewalla Gold](https://help.firewalla.com/hc/en-us/articles/1500004087521-WireGuard-VPN-Server-)
- [Algo](https://github.com/trailofbits/algo)
- likely many others

# Installing

1. install the OS on your target device. This process has been tested on a NanoPi NEO2, using [Armbian_23.5.2_Nanopineo2_jammy_current_6.1.30_minimal](https://www.armbian.com/nanopi-neo-2/)

2. ssh into your new host

   ```bash
   ssh root@<IP of host here>
   ```

   you can get the IP of your host by checking the DHCP logs on your router to see which IP address is was assigned. The device should appear as `nanopi` (or similar)

3. follow the prompts. This will create a non-root user with sudo access

   if you dont have a password manager that can generate good passwords for you, I suggest you use the following to generate a secure password. Use as many characters as you can. Do the same for the new user you create. Dont worry, you wont need to use that password since you will be uploading an ssh key

   ```bash
   openssl rand 128 -base64
   ```

4. allow password-less sudo for your newly created user

   ```bash
   visudo
   ```

   change the line `%sudo   ALL=(ALL:ALL) ALL` to `%sudo   ALL=(ALL:ALL) NOPASSWD:ALL`

   Save the file and exit

5. logout of your device

6. Copy your ssh key to your device

   ```bash
   ssh-copy-id <username>@<IP of nanopi device here>
   ```

7. [optional] ssh back into your devide, then update host timezone to UTC

   ```bash
   sudo dpkg-reconfigure tzdata
   ```

   When prompted, select `None of the above`, then select `UTC` from the next prompt

8. setup the python3 virtualenv

   ```bash
   sudo apt install -y python3-virtualenv
   ```

9. Install remaining dependencies

   ```bash
   python3 -m virtualenv --python="$(command -v python3)" .env &&
     source .env/bin/activate &&
     python3 -m pip install -U pip &&
     pip install -r requirements.txt

     ansible-galaxy collection install devsec.hardening
   ```

10. update `config.yml` to meet your requirements. The main values you will want to change are `server_public_ip` and `duckdns.domain` (if you want to enable
    DuckDNS).

    - `server_public_ip` should be set to either the public IP address or hostname that your VPN clients will be connecting to.
    - `duckdns.domain` should be set to the domain you register on the [DuckDNS](https://www.duckdns.org/) website

11. kick off ansible to deploy to your device

    ```bash
    ansible-playbook -e 'ansible_user=<non-root username on device>' -i <IP of your device>, main.yml
    ```

    NB: the `,` after the host IP is important!

    Follow the prompts, and wait for the procedure to complete

12. ssh to your new VPN host, then change your root and non-root user's password now that the password hashing strategy has been hardened

    ```bash
    sudo passwd
    passwd
    ```

13. add some users to the VPN

    ```bash
    sudo wireguard-admin.sh
    ```

14. configure your router/firewall to forward incoming UDP traffic to the IP of your new VPN server, using the port you specified in `config.cfg`.  
    This is left as an exercise for the reader, since every router and firewall is slightly different, you will need to sort that out on your own.
