# Ansible Role: LLDP

This role will install and configure the lldpad service which will enable LLDP on the specified interfaces.

This role will also set the LLDP TLVs to enable on all interfaces.

## Requirements

This role has to be executed as user 'root'.

## Variables

| Variable            | Type   | Required | Default                            | Comment                                                   |
|---------------------|--------|----------|------------------------------------|-----------------------------------------------------------|
| lldp_interfaces     | list() | Yes      | N/A                                | Interfaces to configure LLDP on                           |
| lldp_admin_status   | string | No       | rxtx                               | LLDP Admin Status for interfaces (rx, tx, rxtx, disabled) |
| lldp_enabled_tlvs   | list() | No       | see defaults                       | TLVs to enable on interfaces                              |
| lldp_disabled_tlvs  | list() | No       | []                                 | TLVs to disable on interfaces where TLVs were enabled     |
| lldp_mgmt_addr_ipv4 | string | No       | {{ ansible_default_ipv4.address }} | TLV Management address (mngAddr TLV disabled when empty)  |
| lldp_reset          | bool   | No       | false                              | Reset complete LLDP configuration                         |

**Note:** A 'lldp_reset' will remove all configuration and reconfigure everything with the specified configuration.
          This will normally be used as an extra vars variable: ansible-playbook -e "lldp_reset=true".

### Interfaces

The LLDP admin status will be configured on the specified interfaces.

When interfaces are removed from the interfaces list its best and easiest to also execute a 'lldp_reset'.

### TLV

The enabled TLV IDs will be added to all configured interfaces. 
These TLV IDs can be specified by their type number or type keyword.

When an enabled TLV ID needs to be disabled this can be accomplished with the 'lldp_disabled_tlvs' list.
Once the change is executed this list can be emptied again.

A 'lldp_reset' will also remove all TLV configuration on all interfaces.

### TLV mngAddr (8)

The mngAddr (8) TLV ID can not be specified in the enabled or disabled TLV list!

This TLV ID will be enabled if a management address 'lldp_mgmt_addr_ipv4' is configured or disabled when this
string is empty.

## Example

```yaml
lldp_interfaces:
  - eno8303
  - eno8403

lldp_enabled_tlvs:
  - 1
  - 2
  - 3
  - 4
  - 5

lldp_mgmt_addr_ipv4: 192.168.5.10
```

## Management

### LLDP Tool

The 'lldptool' CLI tool can be used to query an interface its LLDP status and its local or neighbor TLVs. 

```shell
# Check if LLDP is enabled on interface (adminStatus != disabled)
lldptool get-lldp -i eno8303 -c adminStatus

# Query local TLVs
lldptool get-tlv -i eno8303

# Query neighbor TLVs
lldptool get-tlv -i eno8303 -n

# Query local mngAddr TLV
lldptool get-tlv -i eno8303 -V 8
lldptool get-tlv -i eno8303 -V mngAddr

# Check if TLV is enabled
lldptool get-tlv -i eno8303 -V 5 -c enableTx
```
