# NetSec — Práctica FortiGate: Topología de Seguridad Perimetral

**Estudiante:** Enmanuel Fernandez — Matrícula 20251402  
**Curso:** Seguridad en Redes — Sección 5 (Lunes) — ITLA  
**Profesor:** Jonathan Esteban Rondón Corniel  
**Plataforma:** FortiGate-VM64-KVM v7.6.7

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
- [x] WAF en servidor Web

---

## 🗺️ Topología

| Interfaz | Nombre | Rol | IP | Máscara |
|---|---|---|---|---|
| port1 | WAN | Salida a Internet | 10.0.0.202 | /24 |
| port2 | LAN-USERS | LAN de usuarios | 14.2.10.1 | /25 |
| port3 | LAN-SERVERS | LAN de servidores | 14.2.20.1 | /28 |

<!-- CAP-00: Diagrama de topología general (opcional, si se genera en draw.io / Visio) -->
> <img width="1461" height="819" alt="image" src="https://github.com/user-attachments/assets/8930112e-d2cd-44ca-87ae-12ee8bb83f2e" />


---

## ⚙️ 1. Configuración de interfaces y direccionamiento

### 1.1 Dashboard general del sistema
> <img width="1528" height="873" alt="image" src="https://github.com/user-attachments/assets/4260055c-d35e-42db-aa31-5c5377cce573" />


### 1.2 Interfaces de red
Configuración de port1 (WAN), port2 (LAN-USERS) y port3 (LAN-SERVERS).

> <img width="1456" height="840" alt="preview (1)" src="https://github.com/user-attachments/assets/0ae33fd8-0983-44f5-a728-087dad7251da" />


### 1.3 DHCP en LAN de usuarios
- Rango: `14.2.10.10 – 14.2.10.126`
- Máscara: `255.255.255.128`
- Gateway: `14.2.10.1`
- DNS: `1.1.1.1` / `14.2.10.1` / `8.8.8.8`
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


### 3.1 Prueba funcional: solo HTTP permitido

Desde el cliente se confirmó: ping a `1.1.1.1` exitoso (Internet), ping a la IP del servidor y a su gateway con **timeout** (bloqueado), mientras que el acceso HTTP al servidor **sí funcionó** desde el navegador. Esto valida que solo el protocolo HTTP está permitido entre LANs.

> <img width="770" height="645" alt="image" src="https://github.com/user-attachments/assets/ad0a530d-12db-484f-b123-21a5f21869dd" />


---

## 🛡️ 4. Perfiles de seguridad

### 4.1 Web Filter — Bloqueo de itla.edu.do
Perfil `Block-ITLA`, Static URL Filter con reglas Wildcard: `itla.edu.do` y `*.itla.edu.do` → Block.

> <img width="1456" height="839" alt="preview (7)" src="https://github.com/user-attachments/assets/d25f6cc8-278d-43d4-8d80-67958806a68e" />

Bloqueo confirmado en tres pruebas reales: `itla.edu.do`, `aulavirtual.itla.edu.do` y `campusvirtual.itla.edu.do`.

> <img width="1520" height="950" alt="image" src="https://github.com/user-attachments/assets/347e27f2-362f-4f5e-9607-bcbf77958733" />
> <img width="1426" height="918" alt="image" src="https://github.com/user-attachments/assets/4d61bd53-3c6e-4831-90ce-be9ae5dde738" />
> <img width="1518" height="836" alt="image" src="https://github.com/user-attachments/assets/dc57e3ee-6d85-4cb8-9eab-9bd8c2b797df" />


### 4.2 Application Control — Redes sociales y WhatsApp
Perfil `APP-SOCIAL-BLOCK`: categoría **Social Media** bloqueada + override `WhatsApp_VoIP.Call` → Block. Aplicado en la política de salida a Internet (LAN-USERS → port1). **Verificado en vivo.**

> <img width="1456" height="838" alt="preview (8)" src="https://github.com/user-attachments/assets/63b67b76-16a1-4279-bd6d-aa1c784e05ec" />


### 4.3 Intrusion Prevention — Detección de escáneres
Perfil `Scanner-Block`: más de 75 firmas de escaneo y explotación web (Acunetix.Web.Vulnerability.Scanner, DirBuster.Web.Server.Scanner, Apache.APR.PSPrintf.Memory.Corruption, entre otras) → Block, con Packet Logging habilitado.

> <img width="1456" height="834" alt="preview (9)" src="https://github.com/user-attachments/assets/114d5485-706a-455d-be48-9ae8a003c778" />

**Verificado en vivo:** el log de Security Events registró la detección y bloqueo (`dropped`) de un ataque `HTTP.URI.SQL.Injection`, severidad alta, desde el cliente hacia el servidor web.

> <img width="1483" height="580" alt="image" src="https://github.com/user-attachments/assets/98ff33e7-ce3c-4237-9556-446e3ea3647c" />


### 4.4 Web Application Firewall (WAF)
Perfil `WAF-ServidorWEB` con las siguientes firmas en modo Block: Cross Site Scripting, Cross Site Scripting (Extended), SQL Injection, SQL Injection (Extended), Known Exploits, Information Disclosure y Credit Card Detection. Restricciones (Constraints) en Block: Header Length, Number of URL Parameters, Malformed Request.

Aplicado en la política `LAN-USERS → LAN-SERVERS`, protegiendo directamente al servidor web. **Verificado en vivo** con payloads de SQL Injection y XSS, confirmando bloqueo correcto.

> <img width="1536" height="877" alt="image" src="https://github.com/user-attachments/assets/9e117cff-7758-4f63-b6ed-6befad87155e" />
> <img width="1536" height="876" alt="image" src="https://github.com/user-attachments/assets/9a7dd67a-66a7-4275-9411-578b13c22471" />


---

## ✅ Evidencia de funcionamiento

A diferencia de un despliegue anterior en modo evaluación (donde IPS aparecía como `Not Supported`), en esta VM (FortiOS v7.6.7) todos los motores de seguridad están operativos y fueron verificados con tráfico real:

- **IPS:** ataque `HTTP.URI.SQL.Injection` detectado y bloqueado (`dropped`), registrado en `Log & Report → Security Events`.
- **Web Filter:** bloqueo confirmado en 3 pruebas reales contra `itla.edu.do`, `aulavirtual.itla.edu.do` y `campusvirtual.itla.edu.do`.
- **Application Control:** bloqueo de redes sociales y llamadas de WhatsApp verificado en vivo.
- **WAF:** bloqueo de payloads SQLi/XSS verificado en vivo.

**Nota sobre SSL Deep Inspection:** para que estos perfiles inspeccionen tráfico HTTPS, se activó Deep Inspection en las políticas de salida a Internet. Esto genera una advertencia de certificado en el navegador del cliente (certificado sustituto emitido por FortiGate), comportamiento esperado que se resolvería distribuyendo el certificado CA a los clientes en un entorno de producción.

> <img width="1486" height="860" alt="image" src="https://github.com/user-attachments/assets/c7a63d32-7790-40b1-88fa-1548cd2e5b4e" />


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
    ├── 01_interfaces.png
    ├── 02_static_routes.png
    ├── 03_dhcp_config.png
    ├── 04_dhcp_dns.png
    ├── 05_cliente_dhcp.png
    ├── 06_cliente_ping.png
    ├── 07_server_ping_ip.png
    ├── 08_http_only_proof.png
    ├── 09_firewall_policy_final.png
    ├── 10_block_aulavirtual.png
    ├── 11_block_campusvirtual.png
    ├── 12_block_itla_root.png
    ├── 13_ssl_deepinspection_warning.png
    ├── 14_waf_signatures.png
    ├── 15_waf_constraints.png
    ├── 16_scanner_block_final.png
    └── 17_ips_log_sqli.png
```

## 🔗 Enlaces relacionados

- Repositorio padre del curso: [github.com/Enmafs/NetSec](https://github.com/Enmafs/NetSec)

---

*Última actualización: práctica de topología FortiGate — Seguridad en Redes, ITLA.*
