# Montando o template VLAN/Subnet no Zabbix

Em vez de configurar item por item em cada host, agrupe tudo num **template** e vincule aos seus switches. Faça uma vez, use em todos.

## 1. Criar o template

**Data collection → Templates → Create template**

- **Template name**: `Template Net VLAN e Subnet`
- **Groups**: `Templates/Network devices`

## 2. Macros do template

Na aba **Macros** do template:

| Macro | Valor padrão | Descrição |
|---|---|---|
| `{$SNMP_COMMUNITY}` | `public` | Community SNMP (sobrescreva no host) |
| `{$VLAN.OID}` | `1.3.6.1.2.1.17.7.1.4.3.1.1` | OID base das VLANs |

## 3. Discovery rule de VLANs

- **Key**: `vlan.discovery`
- **Type**: `SNMPv2 agent`
- **SNMP OID**: `discovery[{#VLAN_NAME},{$VLAN.OID}]`

### Item prototype

- **Name**: `VLAN {#SNMPINDEX} - {#VLAN_NAME}`
- **Key**: `vlan.name[{#SNMPINDEX}]`
- **SNMP OID**: `{$VLAN.OID}.{#SNMPINDEX}`
- **Type of information**: `Character`

## 4. Discovery rule de Subnets (script externo)

- **Key**: `descobrir_subnets.sh["{HOST.IP}","{$SNMP_COMMUNITY}"]`
- **Type**: `External check`

### Item prototype

- **Name**: `Subnet {#SUBNET} (IP {#IP})`
- **Key**: `subnet.info[{#IP}]`

## 5. Vincular ao host

No host do switch: **Templates → Link new templates → `Template Net VLAN e Subnet`**.
Sobrescreva `{$SNMP_COMMUNITY}` na aba **Macros** do host.

## Dica: exportar o template

Depois de montar, exporte em **Templates → (selecione) → Export** para gerar um `.yaml`/`.xml`.
Esse arquivo pode entrar no repositório — assim qualquer pessoa importa o template pronto em
**Templates → Import**, sem refazer nada na mão.
