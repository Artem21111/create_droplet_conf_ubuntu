Role for configuring security for Ubuntu-based systems. 
Using sshd settings, fail2ban, iptables with ipset and iptables for docker.

Need to work: 
    - json file with description iptables rules configuration
    Example: 
        { 
    "firewall_status": {
        "allow": {
            "hosts_for_firewall": [
                {
                    "name": "ssh",
                    "ip_address": "127.0.0.1",
                    "protocol": "tcp",
                    "chain": "INPUT",
                    "rule": "ACCEPT",
                    "port": "22", 
                    "description": "ip connect to ssh port"
                }
            ]
        }
    }
}
    - how include file: 
        vars_files: ./vars/<filename>.json

Usage example:
    1) playbook: include_role: 
                   name: config_security_ubuntu
    2) ansible-playbook <your playbook name> 

Variables list:

# Variables for sshd configuration
ssh_port:               ""                      ( New sshd port )
local_user:             ""                      ( Localhost user )

# Variables for fail2ban configuration
path_for_logs:          "/var/log/auth.log"     ( Pass to your sshd log file ) 
max_trys:               "3"                     ( Max attempts before ban )
ban_time:               "180"                   ( Ban time )
ignore_ip:                                      ( Ip which should be ignored )
  - 127.0.0.1
find_time:              "600"                   ( Login attempts detection time )

# Variables for configuration docker iptables 
install_docker:         ""                      ( If you need to install docker-ce write "yes" )
conf_docker:            ""                      ( Write "yes" if you want configuration docker iptables MASQUERADE )
docker_vars: 
    - { in_interface: "docker0" }
    - { chain: "POSTROUTING", source: "172.17.0.0/16", table: "nat", jump: "MASQUERADE" }
    - { ctstate: "RELATED,ESTABLISHED", out_interface: "docker0" }
    - { in_interface: "docker0", out_interface: "!docker0" }
    - { in_interface: "docker0", out_interface: "docker0" }
    - { jump: "DROP", comment: "Custom DROP rule" }
