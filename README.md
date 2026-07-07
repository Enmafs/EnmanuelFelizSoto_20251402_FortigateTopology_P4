# NetSec — Práctica FortiGate: Topología de Seguridad Perimetral

**Estudiante:** Enmanuel Fernandez — Matrícula 20251402  
**Curso:** Seguridad en Redes — Sección 5 (Lunes) — ITLA  
**Profesor:** Jonathan Esteban Rondón Corniel  
**Plataforma:** FortiGate-VM64-KVM v7.6.2 (modo evaluación)

---

## 📌 Descripción

Este repositorio documenta la implementación de una topología de red segura en **FortiGate**, configurada íntegramente desde la GUI, que incluye segmentación de red, DHCP, NAT, control de acceso entre LANs, y perfiles de seguridad (Web Filter, Application Control, IPS y WAF).

## 🎯 Objetivos de la práctica

- [x] Configuración exclusiva por GUI
- [x] Acceso a Internet
- [x] LAN de usuarios (/25)
- [x] LAN de servidores (/28)
- [x] IP en interfaces
- [x] DHCP en LAN de usuarios
- [x] Ruta por defecto
- [x] NAT
- [x] Solo tráfico HTTP permitido: LAN Usuarios → LAN Servidores (resto bloqueado)
- [x] Bloqueo de redes sociales
- [x] Bloqueo de llamadas de WhatsApp
- [x] Bloqueo de dominios y subdominios de itla.edu.do
- [x] Detección y bloqueo de escáneres de red (IPS)
- [ ] WAF en servidor Web *(pendiente)*

---

## 🗺️ Topología

| Interfaz | Nombre | Rol | IP | Máscara |
|---|---|---|---|---|
| port1 | WAN | Salida a Internet | 10.0.0.173 | /24 |
| port2 | LAN USERS | LAN de usuarios | 14.2.20.1 | /25 |
| port3 | LAN SERVERs | LAN de servidores | 14.2.10.1 | /28 |

<!-- CAP-00: Diagrama de topología general (opcional, si se genera en draw.io / Visio) -->
> <img width="1461" height="819" alt="image" src="https://github.com/user-attachments/assets/8930112e-d2cd-44ca-87ae-12ee8bb83f2e" />


---

## ⚙️ 1. Configuración de interfaces y direccionamiento

### 1.1 Dashboard general del sistema
> <img width="1528" height="873" alt="image" src="https://github.com/user-attachments/assets/4260055c-d35e-42db-aa31-5c5377cce573" />


### 1.2 Interfaces de red
Configuración de port1 (WAN), port2 (LAN USERS) y port3 (LAN SERVERs).

> <img width="1456" height="840" alt="preview (1)" src="https://github.com/user-attachments/assets/0ae33fd8-0983-44f5-a728-087dad7251da" />


### 1.3 DHCP en LAN de usuarios
- Rango: `14.2.20.10 – 14.2.20.126`
- Máscara: `255.255.255.128`
- Gateway: `14.2.20.1`
- DNS: `1.1.1.1` / `14.2.20.1`
- Lease time: `604800s`

> <img width="1456" height="836" alt="preview (2)" src="https://github.com/user-attachments/assets/355cc75e-5b46-478e-be0d-9b8f1a70170d" />


### 1.4 Validación desde el cliente
Cliente recibe IP por DHCP y confirma salida a Internet (`ip a` + `ping 1.1.1.1`).

> <img width="1459" height="812" alt="preview (3)" src="https://github.com/user-attachments/assets/e8d198c3-45d7-42a3-9f34-6593cd89b40b" />


---

## 🧭 2. Enrutamiento y NAT

### 2.1 Ruta por defecto
`0.0.0.0/0` vía `10.0.0.1` por `port1`.

> <img width="1456" height="839" alt="preview (5)" src="https://github.com/user-attachments/assets/2bc4fdb4-7e0d-488e-956b-7118714b783b" />


### 2.2 NAT
Habilitado en políticas de salida a Internet; deshabilitado en tráfico interno LAN Usuarios → LAN Servidores.

> <img width="1456" height="833" alt="preview (6)" src="https://github.com/user-attachments/assets/22a7b4c4-216c-4765-a220-69ccfee544d7" />


---

## 🔒 3. Políticas de Firewall

| Política | Origen | Destino | Servicio | Acción | NAT |
|---|---|---|---|---|---|
| NAT (2) | LAN SERVERs | port1 (WAN) | ALL | ACCEPT | Enabled |
| LAN USERS (3) | LAN USERS | LAN SERVERs | **HTTP** | ACCEPT | Disabled |
| NAT (2) | LAN USERS | port1 (WAN) | ALL | ACCEPT | Enabled |
| Implicit Deny | all | all | ALL | DENY | — |

> <img width="1456" height="833" alt="preview (6)" src="https://github.com/user-attachments/assets/356f1d88-91cf-4545-a08d-d7d35a7c9b2b" />


---

## 🛡️ 4. Perfiles de seguridad

### 4.1 Web Filter — Bloqueo de itla.edu.do
Perfil `Block-ITLA`, Static URL Filter con wildcard `*itla.edu.do` → Block.

> <img width="1456" height="839" alt="preview (7)" src="https://github.com/user-attachments/assets/d25f6cc8-278d-43d4-8d80-67958806a68e" />


### 4.2 Application Control — Redes sociales y WhatsApp
Perfil `default`: categoría **Social Media** bloqueada + override `WhatsApp_VoIP.Call` → Block.

> <img width="1456" height="838" alt="preview (8)" src="https://github.com/user-attachments/assets/63b67b76-16a1-4279-bd6d-aa1c784e05ec" />


### 4.3 Intrusion Prevention — Detección de escáneres
Perfil `Scanner-Block`: firmas de Acunetix, DirBuster, Apache Tomcat scanner, entre otras (16 en total) → Block.

> <img width="1456" height="834" alt="preview (9)" src="https://github.com/user-attachments/assets/114d5485-706a-455d-be48-9ae8a003c778" />


### 4.4 Web Application Firewall (WAF)
*(Pendiente de configurar/capturar)*

> 

---

## ⚠️ Limitación técnica: Licencia FortiGuard

El FortiGate-VM opera en modo de **evaluación**, por lo que el motor de **IPS** aparece como `Not Supported` y los servicios de **Web Filter** y **Application Control** no descargan las bases de datos de categorías/firmas actualizadas de FortiGuard. Los perfiles están correctamente creados y aplicados vía GUI, pero **no ejecutan bloqueo real de tráfico** en este entorno de laboratorio.

> 📸 **[CAP-12]** — Dashboard → Licenses (IPS: Not Supported)

---

## 📄 Documentación completa

El informe técnico detallado con todas las capturas embebidas está disponible en:
- [`Practica_FortiGate_Documentacion.docx`](./Practica_FortiGate_Documentacion.docx)
- [`Practica_FortiGate_Documentacion.md`](./Practica_FortiGate_Documentacion.md)

## 📁 Estructura del repositorio

```
NetSec-FortiGate-Practica/
├── README.md
├── Practica_FortiGate_Documentacion.docx
├── Practica_FortiGate_Documentacion.md
└── images/
    ├── 01_dashboard.png
    ├── 02_interfaces.png
    ├── 03_dhcp_interface.png
    ├── 04_cliente_ping_dhcp.png
    ├── 05_static_routes.png
    ├── 06_firewall_policy.png
    ├── 07_webfilter.png
    ├── 08_appcontrol.png
    ├── 09_ips.png
    └── 10_waf.png          # pendiente
```

## 🔗 Enlaces relacionados

- Repositorio padre del curso: [github.com/Enmafs/NetSec](https://github.com/Enmafs/NetSec)

---

*Última actualización: práctica de topología FortiGate — Seguridad en Redes, ITLA.*
