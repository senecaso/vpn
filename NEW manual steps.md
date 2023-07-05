1. install latest armbian image
2. ssh in, add new user
3. allow password-less sudo for your newly created user

   ```bash
   visudo
   ```

   change the line `%sudo   ALL=(ALL:ALL) ALL` to `%sudo   ALL=(ALL:ALL) NOPASSWD:ALL`

   Save the file and e

4. update the image: `apt-get update && apt-get dist-upgrade`
5. install ansible: `apt-get install ansible`
6. install os-hardening: `ansible-galaxy collection install devsec.hardening`
7. create `main.yml` with the following content:

   ```yaml
   - hosts: localhost
     become: true
     connection: local
     collections:
       - devsec.hardening
     roles:
       - ssh_hardening

   - hosts: localhost
     become: true
     connection: local
     collections:
       - devsec.hardening
     roles:
       - os_hardening
     vars:
       os_user_pw_ageing: false
       sysctl_overwrite:
         net.ipv4.ip_forward: 1
         net.ipv4.conf.all.forwarding: 1
         net.ipv6.conf.all.forwarding: 0
         net.ipv6.conf.all.disable_ipv6: 0
   ```

8. run ansible: `ansible-playbook main.yml`
9. install git: `apt-get install git`
10. clone [writeguard-install](https://github.com/angristan/wireguard-install/tree/master): `git clone https://github.com/angristan/wireguard-install.git`
11. set up wireguard-install:

    ```bash
    cd wireguard-install
    chmod +x wireguard-install.sh
    ```

12. follow steps to install DuckDNS dynamic DNS: [here](https://www.duckdns.org/install.jsp)
13. install wireguard: `sudo ./wireguard-install.sh`
14. to add/remove/list users: `sudo ~/wireguard-install/wireguard-install.sh`
