# iptables

Ansible role to setup or unset iptables.

This role is a piece of (*yet another*)
[Ansible Quick Starter](/aqs-common) (**AQS**).

## Requirements

None.

## Role Variables

The main action the role is called for. Choices are `setup` (the default) and
`unset`, that may apply to the same `iptables__rules`, as setup means the listed
rules are present, and unset means they are absent. Other valid choices are:
`flush` and `reset`.
```yaml
iptables__action: unset
```

This variable defines a ruleset to apply in a loop within a single task:
```yaml
iptables__rules:
  - chain: FORWARD
    policy: DROP
  - chain: INPUT
    action: insert
    rule_num: 5
    protocol: tcp
    match: tcp
    syn: negate
    ctstate: NEW
    comment: "bad NEWs"
    jump: DROP
```


## Dependencies

None.

## Installation

To make use of this role as a galaxy role, put this in `requirements.yml`:

```yaml
- name: "iptables"
  src: "https://github.com/quidame/aqs-role-iptables.git"
  scm: "git"
  version: master
```

And then run:

```bash
ansible-galaxy install -r requirements.yml
```

## Example Playbook

Setup a basic firewall from scratch:
```yaml
- hosts: servers
  roles:
    - role: iptables
      iptables__rules:
        - { chain: "OUTPUT", policy: "ACCEPT" }
        - { chain: "INPUT", policy: "ACCEPT" }
        - { chain: "FORWARD", policy: "DROP" }
        - { flush: true }
        - { chain: "OUTPUT", ctstate: "INVALID", jump: "DROP" }
        - { chain: "OUTPUT", ctstate: "RELATED,ESTABLISHED", jump: "ACCEPT" }
        - { chain: "INPUT", ctstate: "INVALID", jump: "DROP" }
        - { chain: "INPUT", ctstate: "RELATED,ESTABLISHED", jump: "ACCEPT" }
        - { chain: "INPUT", in_interface: "lo", jump: "ACCEPT" }
        - { chain: "INPUT", protocol: "icmp", jump: "ACCEPT" }
        - { chain: "INPUT", protocol: "udp", match: "udp", source_port: "0:1023", comment: "bad source port", jump: "DROP" }
        - { chain: "INPUT", protocol: "tcp", match: "tcp", source_port: "0:1023", comment: "bad source port", jump: "DROP" }
        - { chain: "INPUT", protocol: "tcp", match: "tcp", syn: "negate", ctstate: "NEW", comment: "bad NEWs", jump: "DROP" }
        - { chain: "INPUT", protocol: "tcp", match: "tcp", destination_port: "{{ ansible_ssh_port | default(22) }}", comment: "SSH", jump: "ACCEPT" }
        - { chain: "INPUT", policy: "DROP" }
```

## License

GPLv3

## Author Information

<quidame@poivron.org>
