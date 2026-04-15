# Arquitectura Ofensiva con VPS y VPNs para Bug Bounty

> **TL;DR:** Si alguna vez te levantaste sin poder entrar a Infobae, Netflix o tu *scope* porque el WAF te metió en una *blacklist* por tirar `ffuf` sin protección, este artículo es para vos. Acá está el *blueprint* real de cómo los *top hunters* de Bug Bounty Argentina montan infraestructura ofensiva distribuida, barata y letal.

---

## Prólogo: El día que me bloquearon de medio internet

Hace un tiempo, un miembro de la comunidad compartió su experiencia:

> *"Bug Bounty Tip: Si van a fuzzear desde su casa, doble check si la VPN está activa. Al amigo de un amigo le pasó que se despertó y tiene media internet bloqueada porque no activó la VPN 🤣🤣😭😭😭"*

Y no es chiste. Cuando empezás a hacer *Recon* en serio, te das cuenta de que tirar herramientas como `amass`, `ffuf` o `nuclei` desde tu máquina física es como ir a una guerra con una escopeta de feria. Te exponés a:

- **Baneos masivos de IP:** Akamai, Cloudflare y otros CDNs te meten en *blacklist* y perdés acceso no solo al *target*, sino a todos los sitios que usan esa infraestructura.
- **Limitaciones de ancho de banda:** Tu proveedor de internet hogareño te da 10 Mbps de subida si tenés suerte.
- **Inestabilidad:** Se te corta la luz, se cae el WiFi, se reinicia Windows para actualizar... y perdiste 8 horas de escaneo.

La solución no es "ser más cuidadoso". La solución es **construir una arquitectura distribuida**.

---

## La arquitectura del hunter profesional

Los *hunters* que más facturan no tienen un *setup* mágico. Tienen **capas de separación** entre el análisis manual y la fuerza bruta automatizada.

### Capa 1: El comando local (tu máquina)

Acá es donde hacés el trabajo fino:

- **OS:** Mac, Windows, Linux (lo que te resulte cómodo).
- **Tool Principal:** Burp Suite Professional. Es tu bisturí.
- **VPN Cliente:** Para cambiar de región cuando te banean manualmente.
- **Browser con Proxy:** Firefox/Chrome con FoxyProxy configurado a `127.0.0.1:8080`.

Tu máquina local es tu **centro de operaciones**. Acá hacés *spidering*, armás *payloads* quirúrgicos, analizás respuestas en el *Repeater* y *Intruder* de Burp.

### Capa 2: La nube de ejecución (VPS en la nube)

Acá es donde delegás todo lo pesado, ruidoso y que requiere tiempo:

- **OS:** Debian/Ubuntu Server (*headless*, sin GUI).
- **Herramientas:** `subfinder`, `httpx`, `nuclei`, `ffuf`, `amass`, `gau`, `waybackurls`.
- **Orquestación:** `tmux` o `screen` para mantener procesos vivos aunque te desconectes.
- **Conectividad:** 100-500 Mbps simétricos. Ancho de banda real.

El *workflow* es simple:

1. Te conectás por SSH al VPS.
2. Levantás un `tmux` con 3 panes: uno para *Recon* pasivo, otro para *fuzzing*, otro para monitoreo.
3. Lanzás comandos y te desconectás. El VPS sigue trabajando 24/7.
4. Cuando volvés, te attachás al `tmux` y revisás los resultados.

---

## La cruda realidad de los costos

Uno de los mitos más grandes en la comunidad es que necesitás un presupuesto millonario para hacer Bug Bounty. Acá va la verdad:

### Opción 1: El setup minimalista ($0 - $6 USD/mes)

**Proveedores:**
- **Oracle Cloud Free Tier:** 4 vCPU + 24GB RAM (gratis forever). Ideal para empezar.
- **AWS Free Tier:** 1 Core, 1GB RAM, gratis por 12 meses.
- **Digital Ocean / Linode (con referral):** $200 USD de crédito inicial.

**Para qué alcanza:**
- Correr `subfinder`, `httpx` y `nuclei` en *targets* medianos.
- Tener una IP externa limpia para evitar baneos.

**Limitación:**
- 1GB de RAM se te queda corto si querés hacer *permutaciones* masivas de subdominios o correr `amass` en modo *brute force*.

### Opción 2: El setup intermedio ($6 - $12 USD/mes)

**Proveedores:**
- **Contabo:** 4GB RAM, 4 vCore, 200GB SSD por $6.99 USD/mes.
- **Digital Ocean:** 2GB RAM, 1 CPU por $12 USD/mes (más estable para producción).
- **Hetzner:** 2GB RAM, 2 vCore por $4.51 EUR/mes (~$5 USD).

**Para qué alcanza:**
- Correr múltiples herramientas en paralelo.
- Hacer *crawling* pesado con `gospider` o `katana`.
- Dejar `nuclei` corriendo con todas las plantillas durante días.

**Comentario de la comunidad:**
> *"OVH tiene VPS baratos, no sé por qué todos se van a DigitalOcean que es más caro. Hetzner está súper bien también."*

### Opción 3: La granja distribuida (usuarios avanzados)

Acá entramos en territorio *hardcore*. Una estrategia avanzada es alquilar múltiples servidores baratos para distribuir el escaneo y evitar baneos.

**El concepto:**
- Alquilar VPS de $2-5 USD/mes en proveedores como **Racknerd**, **OVH** o **Hetzner**.
- Cada VPS tiene su propia IP pública.
- Dividir el diccionario de subdominios o *paths* entre los nodos.
- Usar **pssh** (Parallel SSH) o **Axiom** para orquestación distribuida.

**Ejemplo con pssh:**

```bash
# Archivo hosts.txt con las IPs de los VPS
vps1.example.com
vps2.example.com
vps3.example.com

# Ejecutar subfinder en paralelo en todos los VPS
pssh -h hosts.txt -i "subfinder -d target.com -silent"
```

**Resultado:**
- Múltiples IPs distintas golpeando el *target*.
- Si te banean una, tenés las otras funcionando.
- *Throughput* masivo combinado.

**Costo total:** ~$50-200 USD al mes dependiendo de la escala.

---

## VPNs: No es paranoia, es munición

Una VPN no es para "ocultarte de la NSA". Es para **rotar IPs** cuando te bloquean y seguir avanzando.

### El workflow con VPN

1. Instalás un cliente OpenVPN en tu VPS o máquina local.
2. Contratás un proveedor que permita **múltiples conexiones simultáneas**:
   - **NordVPN:** Unlimited devices (~$3-4 USD/mes con plan largo).
   - **Surfshark:** Unlimited devices (~$2.50 USD/mes con plan largo).
   - **Mullvad:** 5 devices (€5/mes, ~$5.50 USD).
   - **Private Internet Access (PIA):** Unlimited devices (~$2 USD/mes con plan largo).

3. Levantás *dockers* o *tmux panes* que ruteen su tráfico por diferentes túneles de la VPN.

**Ejemplo con Docker:**

```bash
# Levantar 10 contenedores, cada uno conectado a una VPN distinta
for i in {1..10}; do
  docker run -d --name vpn-worker-$i \
    -v /path/to/openvpn-config-$i.ovpn:/vpn/config.ovpn \
    my-vpn-image
done
```

Cada *worker* sale con una IP diferente. Si uno se banea, los otros 9 siguen operativos.

### El truco con Squid Proxy

Una configuración avanzada de la comunidad:

> *"Uso múltiples conexiones VPN con Squid Proxy, así que tengo en el puerto 8081 la IP1, en el 8082 la IP2, etc."*

**¿Qué logra con esto?**
- Puede configurar Burp Suite para rotar entre proxies.
- Si está *fuzzeando* con `ffuf`, puede usar `--proxy http://localhost:8081` y cambiar de puerto cuando le conviene.

---

## La magia de X11 Forwarding: GUI remota sin VNC

Uno de los *pain points* más comunes es: *"¿Cómo corro Burp Suite en un VPS si no tiene interfaz gráfica?"*

La respuesta amateur es instalar VNC o RDP. La respuesta profesional es **X11 Forwarding**.

### ¿Qué es X11 Forwarding?

Es un protocolo que permite que la **interfaz gráfica de un programa corra en el servidor**, pero se **renderice en tu pantalla local**.

**Ventajas:**
- Latencia baja (50-100 ms si el VPS está cerca).
- No necesitás instalar un *Desktop Environment* completo en el VPS.
- El proceso pesado (Burp haciendo HTTP requests) corre en el VPS con su ancho de banda de 100 Mbps.

**Ejemplo:**

```bash
# Conectarse al VPS con X11 Forwarding habilitado
ssh -X root@ip_del_vps

# Una vez dentro, ejecutar Burp
java -jar burpsuite_pro.jar
```

La ventana de Burp se abre en tu pantalla, pero **las peticiones HTTP salen desde la IP del VPS**.

### Alternativa moderna: Xpra

Si la latencia de X11 es alta, podés usar **Xpra**, que es como "screen para aplicaciones gráficas":

```bash
# En el VPS
xpra start :100 --start-child="java -jar burpsuite_pro.jar"

# Desde tu PC local
xpra attach ssh:root@ip_del_vps:100
```

---

## Proveedores de VPS: Tier list para Bug Bounty (actualizado 2026)

No todos los proveedores son iguales. Algunos te cierran la cuenta a la primera queja de *Abuse*. Acá la experiencia de la comunidad:

| Proveedor | Precio (RAM/CPU) | Ancho de Banda | Política de TOS | Uso Recomendado |
|-----------|------------------|----------------|-----------------|-----------------|
| **Digital Ocean** | $6 (1GB/1CPU) | 1000 Mbps | Permisivo con *pentesting* | *Burn & Destroy*: levantar, escanear, destruir |
| **Linode (Akamai)** | $5 (1GB/1CPU) | 1000 Mbps | Muy permisivo | Servidor principal de *Recon* 24/7 |
| **Contabo** | $6.99 (4GB/4vCore) | 200 Mbps | Permisivo | Tareas pesadas (Amass, Nuclei, Dalfox) |
| **Hetzner** | $4.51 EUR (2GB/2vCore) | Hasta 20 TB incluido | Muy permisivo | Granjas distribuidas |
| **Vultr** | $6 (1GB/1CPU) | 2000 Mbps | Permisivo | *Recon* de alta velocidad |
| **Oracle Cloud Free** | $0 (24GB/4vCore) | Variable | Restrictivo | Testing y aprendizaje |
| **AWS Lightsail** | $3.50 (512MB/1CPU) | 1 TB incluido | Moderado | Microservicios y automatización |

### Comentarios de la comunidad

> *"Algunos proveedores aceptan crypto para pago, lo cual es útil si querés más anonimato en tus pruebas."*

> *"Linode nunca me falló, y una sola vez me contactaron porque pensaban que me habían comprometido. Un ticket y a la media hora estaba todo OK."*

> *"DigitalOcean sabe qué es Bug Bounty. Tienen un programa propio en HackerOne."*

> *"Hetzner tiene una relación precio/performance increíble. Para Europa es imbatible."*

---

## Axiom: Automatización distribuida en la nube

**Axiom** es un framework de automatización dinámica que permite crear, gestionar y destruir flotas de VPS bajo demanda específicamente para *Recon* y *Bug Bounty*.

### ¿Qué es Axiom?

Desarrollado por **@pry0cc**, Axiom te permite:
- Desplegar múltiples VPS en segundos en Digital Ocean, Linode, Azure, AWS.
- Distribuir tareas de *Recon* automáticamente entre todos los nodos.
- Escalar horizontalmente: necesitás más poder, creás más instancias.
- Destruir todo al terminar para minimizar costos.

### Ventajas de Axiom

1. **Costo-efectivo:** Solo pagás por el tiempo que usás los VPS.
2. **Escalable:** Podés pasar de 1 a 50 instancias en minutos.
3. **Modular:** Integraciones pre-construidas con herramientas populares.
4. **Rotación de IPs:** Cada instancia tiene su propia IP pública.

### Instalación básica

```bash
# Clonar Axiom
git clone https://github.com/pry0cc/axiom
cd axiom

# Ejecutar instalador
./interact/axiom-configure

# Configurar provider (ejemplo: Digital Ocean)
# Necesitás tu API token del proveedor
```

### Workflow típico con Axiom

```bash
# 1. Crear una flota de 10 instancias
axiom-fleet my-recon-fleet -i 10

# 2. Distribuir subdomain enumeration
axiom-scan domains.txt -m subfinder -o subdomains.txt

# 3. Validar hosts activos en paralelo
axiom-scan subdomains.txt -m httpx -o active-hosts.txt

# 4. Destruir la flota para dejar de pagar
axiom-rm my-recon-fleet -f
```

### Casos de uso reales

**Scenario 1: Recon Masivo en Live Hacking Event**
```bash
# Crear 20 instancias
axiom-fleet lhe-2026 -i 20

# Escanear 10,000 subdominios en paralelo
axiom-scan massive-scope.txt -m nuclei -T lhe-2026 -o results/
```

**Scenario 2: Fuzzing Distribuido**
```bash
# Distribuir fuzzing de directorios entre 5 VPS
axiom-scan endpoints.txt -m ffuf -w /path/to/wordlist.txt -T my-fleet
```

**Scenario 3: Screenshot de Miles de Hosts**
```bash
# Capturar screenshots de todos los hosts activos
axiom-scan active-hosts.txt -m gowitness -T fleet
```

### Módulos populares de Axiom

| Módulo | Herramienta | Uso |
|--------|-------------|-----|
| `subfinder` | ProjectDiscovery | Enumeración de subdominios |
| `httpx` | ProjectDiscovery | Validación de hosts activos |
| `nuclei` | ProjectDiscovery | Escaneo de vulnerabilidades |
| `ffuf` | ffuf | Directory/Parameter fuzzing |
| `gowitness` | GoWitness | Screenshots de aplicaciones web |
| `amass` | OWASP Amass | Recon pasivo/activo |

### Best Practices con Axiom

1. **Usa Snapshots:** Creá un snapshot con todas tus tools instaladas para deploy rápido.
2. **Monitoreo de Costos:** Destruí las flotas cuando no las usás.
3. **Rate Limiting:** Configurá `-rate` en tus escaneos para no saturar targets.
4. **Almacenamiento:** Sincronizá resultados a tu máquina local frecuentemente.

**Documentación oficial:** [https://github.com/pry0cc/axiom](https://github.com/pry0cc/axiom)

---

## Scripts de automatización: El bootstrap en 5 minutos

Cuando levantás un VPS nuevo, no podés perder 30 minutos instalando herramientas a mano. Necesitás un **script de bootstrap**.

### Script base (Debian/Ubuntu)

```bash
#!/bin/bash
set -e

echo "[+] Actualizando sistema..."
apt update && apt upgrade -y

echo "[+] Instalando dependencias base..."
apt install -y git wget curl tmux jq libpcap-dev build-essential python3-pip

echo "[+] Instalando Go..."
wget https://go.dev/dl/go1.22.1.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.22.1.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:~/go/bin' >> ~/.bashrc
source ~/.bashrc

echo "[+] Instalando herramientas de ProjectDiscovery..."
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install -v github.com/projectdiscovery/katana/cmd/katana@latest

echo "[+] Instalando herramientas de Tomnomnom..."
go install -v github.com/tomnomnom/httprobe@latest
go install -v github.com/tomnomnom/waybackurls@latest
go install -v github.com/tomnomnom/gf@latest
go install -v github.com/tomnomnom/qsreplace@latest

echo "[+] Instalando ffuf..."
go install -v github.com/ffuf/ffuf/v2@latest

echo "[+] Instalando Amass..."
go install -v github.com/owasp-amass/amass/v4/...@master

echo "[+] Clonando wordlists..."
git clone https://github.com/danielmiessler/SecLists.git ~/wordlists

echo "[+] Setup completo. Bienvenido al VPS."
```

### Uso

```bash
# Copiar el script al VPS
scp bootstrap.sh root@ip_del_vps:/root/

# Conectarse y ejecutar
ssh root@ip_del_vps
chmod +x bootstrap.sh
./bootstrap.sh
```

En 5 minutos tenés un VPS listo para *Recon*.

---

## Casos de uso reales: Cómo lo usa la comunidad

### Caso 1: El hunter solo con 1 VPS

**Setup:**
- 1 VPS de Contabo ($6.99 USD/mes, 4GB RAM).
- VPN de Surfshark ($2.50 USD/mes).
- Burp Suite Pro en la máquina local.

**Workflow:**
1. Hace *Recon* pasivo desde el VPS: `subfinder`, `httpx`, `waybackurls`.
2. Analiza los resultados en su máquina local con Burp.
3. Si necesita *fuzzear*, activa la VPN en su máquina y tira `ffuf` desde ahí.

**Costo total:** ~$10 USD/mes.

### Caso 2: El hunter con arquitectura distribuida

**Setup:**
- 1 VPS principal en Hetzner (4GB RAM) para orquestar.
- Axiom configurado con Digital Ocean para deploy dinámico.
- VPN con múltiples conexiones.

**Workflow:**
1. Deploy de flota con Axiom cuando hay un LHE o *scope* grande.
2. Distribución automática de tareas con módulos de Axiom.
3. Resultados centralizados en VPS principal.
4. Destrucción de flota post-evento.

**Costo total:** $5 USD/mes (VPS fijo) + costos variables de Axiom (solo cuando se usa).

### Caso 3: El setup avanzado con granja permanente

**Setup:**
- 10 VPS baratos en Hetzner/Racknerd ($3-5 USD cada uno).
- 1 servidor de control con base de datos para orquestación.
- Script personalizado con `pssh` para distribución de tareas.

**Workflow:**
1. Carga dominios a escanear en una cola/base de datos.
2. Cada VPS consulta, toma trabajo, ejecuta y reporta.
3. Sistema centralizado detecta *duplicados*.

**Costo total:** ~$40-60 USD/mes.

**Lección aprendida:**
> *"La automatización masiva sirve para *Recon*, pero los *bounties* grandes los sacás con trabajo manual y creatividad."*

---

## Errores comunes y cómo evitarlos

### Error 1: Tirar Amass sin VPS

> *"Si no corrés Amass en un VPS, en tu PC local te mata la conexión. Es de las mejores herramientas para encontrar subdominios."*

**Solución:** Siempre correr herramientas pesadas en VPS.

### Error 2: Olvidarse de activar la VPN

> *"Me banearon de Akamai, ahora no puedo entrar ni a varios sitios importantes. Ahora a usar VPN hasta que me cambie la IP."*

**Solución:** Crear un *alias* que valide si la VPN está activa antes de ejecutar herramientas:

```bash
# ~/.bashrc
function safe_ffuf() {
  if ! curl -s https://api.ipify.org | grep -q "IP_DE_TU_VPN"; then
    echo "[!] VPN no está activa. Activala primero."
    return 1
  fi
  ffuf "$@"
}
```

### Error 3: Usar Ubuntu en VPS con 1GB de RAM

> *"Ubuntu consume 600 MB de RAM por defecto. En un VPS de 1GB te mata. Debian usa la mitad, creo 200-300 MB."*

**Solución:** Siempre usar Debian en VPS de bajos recursos.

### Error 4: No limitar el *rate* de requests

> *"Una vez esperé semanas con la IP de mi casa bloqueada... ni la PlayStation funcionaba. Desde entonces, nunca más tiré escaneos desde otro lado que no fuera un VPS."*

**Solución:** Siempre usar `--rate-limit` en herramientas como `ffuf` y `nuclei`:

```bash
ffuf -u https://target.com/FUZZ -w wordlist.txt -rate 50
nuclei -l targets.txt -rl 150
```

---

## Conclusión: La infraestructura es tu ventaja competitiva

El Bug Bounty no es solo encontrar vulnerabilidades. Es **optimizar tu flujo de trabajo** para cubrir más *scope*, más rápido, sin quemar tu infraestructura.

**Resumen del setup definitivo:**

1. **VPS principal:** Contabo/Hetzner (4-8GB RAM) para *Recon* 24/7.
2. **VPN comercial:** NordVPN/Surfshark para rotación de IPs.
3. **Burp Suite Pro local:** Para análisis manual quirúrgico.
4. **Axiom (opcional):** Para escalado dinámico en eventos.
5. **Script de bootstrap:** Para levantar VPS nuevos en 5 minutos.
6. **tmux/screen:** Para mantener procesos vivos.

---

## Bonus: Recursos mencionados por la comunidad

**Proveedores de VPS:**
- [Digital Ocean](https://www.digitalocean.com/)
- [Linode (Akamai)](https://www.linode.com/)
- [Contabo](https://contabo.com/)
- [Hetzner](https://www.hetzner.com/)
- [Vultr](https://www.vultr.com/)
- [Oracle Cloud](https://www.oracle.com/cloud/free/) (Free tier permanente)

**VPNs recomendadas:**
- [Mullvad VPN](https://mullvad.net/) (Enfoque en privacidad)
- [NordVPN](https://nordvpn.com/) (Unlimited devices)
- [Surfshark](https://surfshark.com/) (Unlimited devices)
- [Private Internet Access](https://www.privateinternetaccess.com/)
- [ProtonVPN](https://protonvpn.com/) (Free tier disponible)

**Frameworks de Automatización:**
- [Axiom](https://github.com/pry0cc/axiom) - Automatización distribuida dinámica
- [ReconFTW](https://github.com/six2dez/reconftw) - Framework all-in-one
- [Osmedeus](https://github.com/j3ssie/osmedeus) - Automated Recon & Scanning
- [VPS Docker for Pentest](https://github.com/aaaguirrep/vps-docker-for-pentest)

**Scripts de provisioning:**
- [Terraform + Ansible para VPS](https://github.com/aaaguirrep/vps-docker-for-pentest)

---

## Palabras finales

Este *writeup* está basado en **mensajes reales** de la comunidad de **Bug Bounty Argentina**, extraídos de años de conversaciones sobre infraestructura, automatización y *tooling*.

No es teoría. Es la experiencia cruda de *hunters* que facturan y comparten sus *workflows* reales.

Si te sirvió, compartilo. Si tenés un *setup* distinto, contalo en los comentarios.

*Happy Hunting*

---

**#BugBounty #VPS #VPN #Automation #Recon #InfrastructuraOfensiva #BugBountyArgentina**
