# Ansible Strongswan Role

## Description

Ansible role which installs and configures [Strongswan](https://www.strongswan.org/).

## Role Variables

Setup declaration to configure charon
```yml
strongswan_config_setup: charondebug="ike 1, knl 1, cfg 1"
```

Default configuration which will be used for every declared tunnel
```yml
strongswan_config_default: |
  leftauth=psk
  rightauth=psk
  ike=aes256-sha256-modp2048s256,aes128-sha1-modp1024!
  ikelifetime=28800s
  aggressive=no
  esp=aes128-sha256-modp2048s256,aes128-sha1-modp1024!
  lifetime=3600s
  dpddelay=10s
  dpdtimeout=30s
  keyexchange=ikev1
  rekey=yes
  reauth=no
  dpdaction=restart
  closeaction=restart
  installpolicy=yes
  compress=no
  mobike=no
  type=tunnel
  rightupdown={{ strongswan_config_dir }}/ipsec-vti.sh
  mark=%unique
  auto=start
  leftsubnet=0.0.0.0/0
  rightsubnet=0.0.0.0/0
```

You may want to build the peer config dict like this:
```yml
- name: ansible | render strongswan config
  vars:
    strongswan_config_peer_name: "{{ vpn_connection_config.0.vpn_connection['@id'] }}-{{ vpn_connection_index }}"
    strongswan_config_peer_right_inside_ip: "{{ vpn_connection_config.1.customer_gateway.tunnel_inside_address.ip_address }}"
    strongswan_config_peer_right_outside_ip: "{{ vpn_connection_config.1.customer_gateway.tunnel_outside_address.ip_address }}"
    strongswan_config_peer_left_inside_ip: "{{ vpn_connection_config.1.vpn_gateway.tunnel_inside_address.ip_address }}"
    strongswan_config_peer_left_outside_ip: "{{ vpn_connection_config.1.vpn_gateway.tunnel_outside_address.ip_address }}"
    strongswan_config_peer_network_cidr: "{{ vpn_connection_config.1.vpn_gateway.tunnel_inside_address.network_cidr }}"
    strongswan_config_peer_psk: "{{ vpn_connection_config.1.ike.pre_shared_key }}"
  set_fact:
    strongswan_config_peers: "{{ strongswan_config_peers | combine(lookup('template', 'templates/strongswan_config_peer.j2')) }}"
  loop: "{{ query('subelements', vpn_connection_configs, 'vpn_connection.ipsec_tunnel', {'skip_missing': true}) }}"
  loop_control:
    loop_var: vpn_connection_config
    index_var: vpn_connection_index
```

You only need to use a jinja template to build the dict entry dynamically
```jinja
{
  "{{ strongswan_config_peer_name }}": {
    "right_inside_ip": "{{ strongswan_config_peer_right_inside_ip }}",
    "right_outside_ip": "{{ strongswan_config_peer_right_outside_ip }}",
    "left_inside_ip": "{{ strongswan_config_peer_left_inside_ip }}",
    "left_outside_ip": "{{ strongswan_config_peer_left_outside_ip }}",
    "network_cidr": "{{ strongswan_config_peer_network_cidr }}",
    "psk": "{{ strongswan_config_peer_psk }}"
  }
}
```
