# net-inventory — Inventário de rede via SNMP

Ferramenta de linha de comandos, em **Python**, que varre uma sub-rede, consulta os equipamentos por **SNMP** e gera um inventário em **CSV** e **JSON**.

Para cada dispositivo que responde, recolhe: nome (`sysName`), descrição/modelo (`sysDescr`), tempo de atividade (uptime), contacto, localização e número de interfaces.

> Projeto de portefólio nas áreas de **redes e automação**. Funciona em Linux, Windows ou macOS — só precisa de conseguir alcançar os equipamentos pela rede (SNMP, UDP 161).

---

## Porquê

Manter um inventário atualizado dos equipamentos de rede costuma ser manual e desatualiza-se depressa. Este script automatiza a recolha: aponta para uma sub-rede e obténs, em segundos, uma lista limpa do que está ligado e a responder — pronta para abrir no Excel ou alimentar outra ferramenta (um CMDB, o Zabbix, etc.).

---

## Requisitos

- Python 3.8+
- Dependência: [`pysnmp`](https://pypi.org/project/pysnmp/)

```bash
pip install -r requirements.txt
```

---

## Utilização

A community SNMP **nunca** deve ficar escrita no código. Passa-se por argumento ou pela variável de ambiente `SNMP_COMMUNITY`.

```bash
# um único equipamento
python net_inventory.py 192.168.0.1 --community minhacomunidade

# uma sub-rede inteira, lendo a community do ambiente (mais seguro)
export SNMP_COMMUNITY=minhacomunidade
python net_inventory.py 192.168.0.0/24

# porta, timeout e ficheiros de saída personalizados
python net_inventory.py 10.0.0.0/28 -c publica --port 161 --output inventario_piso2
```

### Opções

| Opção | Predefinição | Descrição |
|---|---|---|
| `alvo` | (obrigatório) | IP único (`192.168.0.1`) ou sub-rede CIDR (`192.168.0.0/24`) |
| `-c`, `--community` | `$SNMP_COMMUNITY` ou `public` | Community SNMP de leitura |
| `--port` | `161` | Porta SNMP |
| `--timeout` | `1.5` | Timeout por host, em segundos |
| `--retries` | `1` | Tentativas por host |
| `--workers` | `50` | Consultas em paralelo |
| `--output` | `inventario` | Prefixo dos ficheiros gerados (`.csv` e `.json`) |

---

## Exemplo de saída

```
A varrer 254 endereço(s) com SNMP v2c na porta 161...

  [OK] 192.168.0.1      SW-CORE-01    (Switch Modelo X1000, IOS v15.2)
  [OK] 192.168.0.2      SW-ACCESS-02  (Switch Modelo X500, IOS v15.0)
  [OK] 192.168.0.10     RTR-EDGE-01   (Router Modelo R900, RouterOS 7.8)

3 equipamento(s) inventariado(s).
  CSV : inventario.csv
  JSON: inventario.json
```

O CSV resultante ([exemplo](examples/inventario_exemplo.csv)):

| ip | sysName | sysDescr | uptime | sysContact | sysLocation | ifNumber |
|---|---|---|---|---|---|---|
| 192.168.0.1 | SW-CORE-01 | Switch Modelo X1000, IOS v15.2 | 98d 7h 10m | admin@exemplo.local | Datacenter - Rack 3 | 48 |

---

## Como testar sem equipamento real

Não precisas de uma rede montada para experimentar. Usa um **simulador SNMP** ([`snmpsim`](https://pypi.org/project/snmpsim/)) que finge ser um switch numa porta local:

```bash
pip install snmpsim pysmi
mkdir -p dados && cat > dados/teste.snmprec << 'EOF'
1.3.6.1.2.1.1.1.0|4|Switch Simulado X1000
1.3.6.1.2.1.1.3.0|67|849302100
1.3.6.1.2.1.1.5.0|4|SW-TESTE-01
1.3.6.1.2.1.1.6.0|4|Lab
1.3.6.1.2.1.2.1.0|2|3
EOF
snmpsim-command-responder --data-dir=./dados --agent-udpv4-endpoint=127.0.0.1:1161 &

python net_inventory.py 127.0.0.1 --community teste --port 1161
```

---

## Segurança

- A community **nunca** é versionada — usa argumento ou variável de ambiente.
- O `.gitignore` impede que inventários reais (`inventario.csv`/`.json`) sejam enviados para o repositório.
- Nos exemplos usam-se sempre dados fictícios.

---

## Melhorias futuras

- Suporte a **SNMPv3** (utilizador, autenticação e cifra) — mais seguro que a community v2c em texto simples.
- Recolha da tabela completa de interfaces (estado, velocidade, tráfego).
- Descoberta de VLANs e endereços IP por equipamento.
- Exportação direta para outras ferramentas (Zabbix, CMDB).

---

## Licença

MIT — ver [LICENSE](LICENSE).
