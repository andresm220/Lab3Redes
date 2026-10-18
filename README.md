# Laboratorio 3 — Algoritmos de Enrutamiento

Planificación IP con VLSM, VLANs y enrutamiento OSPF / EIGRP con redistribución mutua, balanceo de carga y sumarización, implementado en Cisco Packet Tracer.

| | |
| --- | --- |
| **Curso** | Redes — Sección 10, Universidad del Valle de Guatemala |
| **Docente** | Kevin Velásquez |
| **Integrantes** | Andrés Mazariegos (21749) · June Herrera (231038) |

## Contenido del repositorio

| Archivo | Descripción |
| --- | --- |
| `Lab3_Enrutamiento.pkt` | Topología funcional en Packet Tracer: 5 routers, 4 switches, VLANs y protocolos configurados |
| `PDF1_Calculos_y_Conceptos.pdf` | Cálculos VLSM a mano, tabla de direccionamiento y conceptos investigados |
| `PDF2_Evidencias.pdf` | Tablas de enrutamiento, evidencia de balanceo OSPF y EIGRP, y prueba de conectividad extremo a extremo |

## Topología

```
 SW-OSPF-1                                   SW-EIGRP-1
     |                                            |
 R-OSPF-1 ---L2---.                  .---L5--- R-EIGRP-1
     |             \                /             |
     L1             R-CENTRAL                     L4
     |             /                \             |
 R-OSPF-2 ---L3---'                  '---L6--- R-EIGRP-2
     |                                            |
 SW-OSPF-2                                   SW-EIGRP-2
```

| Router | Protocolo | Rol |
| --- | --- | --- |
| R-OSPF-1, R-OSPF-2 | OSPF área 0 | Sitios de acceso |
| R-EIGRP-1, R-EIGRP-2 | EIGRP AS 100 | Sitios de acceso |
| R-CENTRAL | OSPF + EIGRP | Frontera: redistribución mutua y sumarización |

## Direccionamiento

| Segmento | Red | Hosts requeridos | Gateway |
| --- | --- | --- | --- |
| R-OSPF-2 LAN | 192.168.0.0/24 | 200 | 192.168.0.1 |
| R-OSPF-1 VLAN 10 | 192.168.1.0/26 | 50 | 192.168.1.1 |
| R-OSPF-1 VLAN 30 | 192.168.1.64/28 | 10 | 192.168.1.65 |
| R-EIGRP-2 LAN | 172.16.0.0/25 | 100 | 172.16.0.1 |
| R-EIGRP-1 VLAN 10 | 172.16.0.128/26 | 60 | 172.16.0.129 |
| R-EIGRP-1 VLAN 20 | 172.16.0.192/27 | 30 | 172.16.0.193 |
| Enlaces L1–L3 (lado OSPF) | 10.0.0.0/30, .4/30, .8/30 | 2 c/u | — |
| Enlaces L4–L6 (lado EIGRP) | 10.0.0.16/30, .20/30, .24/30 | 2 c/u | — |

Bloques base: 192.168.0.0/20 (VLANs OSPF), 172.16.0.0/20 (VLANs EIGRP) y 10.0.0.0/24 (enlaces). El detalle completo está en el PDF1.

## Decisiones de diseño

- **Enrutamiento entre VLANs:** router-on-a-stick (subinterfaces 802.1Q) en R-OSPF-1 y R-EIGRP-1. Las interfaces hacia usuarios son pasivas.
- **Balanceo OSPF (ECMP):** `ip ospf cost 128` en ambos extremos del enlace directo L1, para igualarlo con el camino por R-CENTRAL (64 + 64). Ambos caminos cuestan 129.
- **Balanceo EIGRP (≈ 75/25):** ajuste de `delay` (2000 en L4, 17900 hacia R-CENTRAL desde R-EIGRP-1/2, 1000 en R-CENTRAL) y `variance 3`. Métricas: 2 172 416 (successor) y 6 498 816 (feasible successor), razón 2,99 ≈ 3 : 1.
- **Redistribución hacia EIGRP:** métrica semilla `1544 2000 255 1 1500`.
- **Redistribución hacia OSPF:** rutas externas tipo **E1**, para que cada router vea el costo real del camino completo.
- **Sumarización:** R-CENTRAL anuncia **192.168.0.0/23** hacia EIGRP y **172.16.0.0/24** hacia OSPF.

### Limitación de Packet Tracer

Packet Tracer no soporta `summary-address`, `distribute-list` ni `route-map` dentro de `router ospf`. El resumen hacia OSPF se implementó con una ruta estática `172.16.0.0/24 → Null0` redistribuida como E1, que es lo que `summary-address` hace internamente en un router real. Hacia EIGRP se usó `ip summary-address eigrp 100` en las interfaces, que sí es compatible.

## Verificación

- Ping y tracert desde PC-OSPF1-V10 (192.168.1.2) hacia PC-EIGRP2 (172.16.0.2): 0 % de pérdida.
- Ruta del tracert: 192.168.1.1 → 10.0.0.6 → 10.0.0.25 → 172.16.0.2.

## Cómo abrir

Abrir `Lab3_Enrutamiento.pkt` con Cisco Packet Tracer 8.x. Las configuraciones ya están guardadas en la NVRAM de cada equipo.