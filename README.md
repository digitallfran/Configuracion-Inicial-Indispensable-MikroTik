<div align="center">
  <img src="https://digitallfran.co/logo.svg" alt="Logo DigitAllFran" width="200"/>
  <h1 align="center">Script de Configuración Inicial Indispensable</h1>
  <h3>Aseguramiento y Puesta en Marcha Básica para RouterOS v7</h3>
  <p>
    <img src="https://img.shields.io/badge/RouterOS-v7.x-0099D5?style=for-the-badge&logo=mikrotik&logoColor=white" alt="RouterOS v7+"/>
    <img src="https://img.shields.io/badge/Configuración-Inicial-27ae60?style=for-the-badge" alt="Configuración Inicial"/>
    <img src="https://img.shields.io/badge/Seguridad-Básica-e74c3c?style=for-the-badge" alt="Seguridad Básica"/>
    <img src="https://img.shields.io/badge/Red-LAN%2FSOHO-3498db?style=for-the-badge" alt="Red LAN/SOHO"/>
  </p>
</div>

<hr>

> [!NOTE]<div style="border-left: 5px solid #007bff; padding: 10px 15px; background-color: #f8f9fa; margin-top: 20px; margin-bottom: 20px;">
>  <p><strong>Punto de Partida Seguro para tu Router</strong><br>
>  Este script ejecuta la configuración fundamental para cualquier RouterBOARD nuevo. Automatiza la creación de un usuario administrador seguro, nombra interfaces, configura el acceso a Internet (WAN), crea una red local (LAN) con DHCP, establece un servicio de DNS eficiente y asegura el equipo con un firewall base.</p>
></div>

> [!WARNING]
><div style="border-left: 5px solid #ffc107; padding: 10px 15px; background-color: #fff3cd; margin-bottom: 20px;">
>  <p><strong>¡ADVERTENCIA!</strong><br>
>  Este script utiliza un nombre de usuario (<code>Digitallfran</code>) y una contraseña (<code>TuCl@veSegura</code>) de ejemplo. Es <strong>crucial</strong> que los modifiques por tus propias credenciales seguras antes de aplicar el script.</p>
>  <p>Luego de aplicar el script asegurate de colocar luego la ip de tu RB y el puerto 10.0.0.1:9000 para entrar en Winbox</p>
></div>

<h3>Paso 1: Crear Usuario Administrador y Deshabilitar 'admin'</h3>
<p>La práctica más segura es crear un nuevo usuario con privilegios completos y deshabilitar el usuario <code>admin</code> por defecto. Esto reduce la superficie de ataque al eliminar un nombre de usuario predecible.</p>
<pre><code>/user add name="Digitallfran" password="TuCl@veSegura" group=full comment="Usuario Administrador"
/user disable [find name=admin]
</code></pre>

<h3>Paso 2: Identificar el Router y Configurar la Hora</h3>
<p>Se le da un nombre único al router para identificarlo en la red y se configura la zona horaria correcta, un paso importante para la precisión de los registros (logs).</p>
<pre><code>/system identity set name="Router Core - Administrador"
/system clock set time-zone-name=America/Bogota
</code></pre>

<h3>Paso 3: Nombrar las Interfaces Físicas</h3>
<p>Se asignan nombres descriptivos a las interfaces. Usar comillas es <strong>obligatorio</strong> porque los nombres contienen espacios.</p>
<pre><code>/interface ethernet
set [ find default-name=ether1 ] name="1 - WAN"
set [ find default-name=ether2 ] name="2 - LAN"
set [ find default-name=ether3 ] name="3 - LAN"
set [ find default-name=ether4 ] name="4 - LAN"
set [ find default-name=ether5 ] name="5 - LAN"
</code></pre>

<h3>Paso 4: Crear Listas de Interfaces</h3>
<p>Usar listas de interfaces es una buena práctica en RouterOS v7. Permite crear reglas de firewall más limpias y fáciles de mantener, ya que agrupamos las interfaces por su función (LAN o WAN).</p>
<pre><code>/interface list
add name=WAN comment="Interfaces conectadas a Internet"
add name=LAN comment="Interfaces de la red local"
</code></pre>

<h3>Paso 5: Crear el Bridge LAN y Asignar Miembros a Listas</h3>
<p>Unimos los puertos LAN en un solo switch virtual (Bridge). Luego, agregamos las interfaces correctas a las listas que creamos en el paso anterior.</p>
<pre><code>/interface bridge add name=Bridge-LAN comment="Bridge principal para unificar la red local"
/interface bridge port
add bridge=Bridge-LAN interface="2 - LAN"
add bridge=Bridge-LAN interface="3 - LAN"
add bridge=Bridge-LAN interface="4 - LAN"
add bridge=Bridge-LAN interface="5 - LAN"
/interface list member
add interface="1 - WAN" list=WAN
add interface=Bridge-LAN list=LAN
</code></pre>

<h3>Paso 6: Configurar WAN, NTP y DNS</h3>
<p>Configuramos el cliente DHCP para obtener IP del proveedor, el cliente NTP para la hora automática y los servidores DNS públicos para la resolución de nombres.</p>
<pre><code>/ip dhcp-client add interface="1 - WAN" use-peer-dns=no use-peer-ntp=no add-default-route=yes comment="Configuracion automatica para la WAN del ISP"
/system ntp client set enabled=yes servers=time.google.com
/ip dns set allow-remote-requests=yes cache-size=4096KiB servers=1.1.1.1,8.8.8.8,9.9.9.9
</code></pre>

<h3>Paso 7: Configurar la Red Local (IP, Pool y DHCP)</h3>
<p>Asignamos una IP al router que servirá de puerta de enlace, creamos el rango de IPs para los clientes y activamos el servidor DHCP.</p>
<pre><code>/ip address add address=10.0.0.1/24 interface=Bridge-LAN comment="Gateway para la red LAN"
/ip pool add name=Pool-DHCP-LAN ranges=10.0.0.2-10.0.0.254 comment="Rango de IPs para clientes de la LAN"
/ip dhcp-server add address-pool=Pool-DHCP-LAN interface=Bridge-LAN name=DHCP-Server-LAN comment="Servidor DHCP para la red LAN"
/ip dhcp-server network add address=10.0.0.0/24 dns-server=10.0.0.1 gateway=10.0.0.1
</code></pre>

<h3>Paso 8: Configurar NAT</h3>
<p>Creamos la regla de enmascarado (NAT) que permite a todos los dispositivos de la red LAN salir a Internet usando la IP pública de la interfaz WAN.</p>
<pre><code>/ip firewall nat add action=masquerade chain=srcnat out-interface-list=WAN comment="NAT para que la LAN salga a Internet"
</code></pre>

<h3>Paso 9: Aplicar Firewall de Seguridad</h3>
<p>Este es el conjunto de reglas de firewall esencial. Protege al router y a la red LAN de accesos no solicitados desde Internet, mientras permite que tus usuarios naveguen sin problemas.</p>
<pre><code>/ip firewall filter
add action=fasttrack-connection chain=forward connection-state=established,related in-interface-list=LAN out-interface-list=WAN comment="Acelerar trafico LAN -> WAN"
add action=accept chain=forward connection-state=established,related comment="Permitir establecido y relacionado"
add action=drop chain=forward connection-state=invalid comment="Bota conexiones invalidas"
add action=accept chain=forward connection-state=new in-interface-list=LAN out-interface-list=WAN comment="Permitir nuevas conexiones desde LAN"
add action=drop chain=forward comment="Bota el resto del trafico forward"
add action=accept chain=input connection-state=established,related comment="Permitir establecido y relacionado al router"
add action=drop chain=input connection-state=invalid comment="Bota conexiones invalidas al router"
add action=accept chain=input in-interface-list=LAN comment="Permitir administracion desde LAN"
add action=drop chain=input comment="Bota el resto del trafico al router"
</code></pre>

<h3>Paso 10: Asegurar y Limpiar Servicios</h3>
<p>Finalmente, deshabilitamos servicios innecesarios o inseguros y cambiamos el puerto de Winbox para evitar ataques automatizados.</p>
<pre><code>/ip service
disable [find where name="telnet" or name="ftp" or name="www" or name="ssh" or name="www-ssl"]
set [find name=winbox] port=9000
</code></pre>

<hr>

> [!NOTE]<h2>Script Limpio (para Copiar y Pegar)</h2>
><p>Esta es la versión final del script con todos los comandos en secuencia y <strong>sin comentarios</strong>, lista para ser ejecutada directamente en el router.</p>
<pre><code>/user add name="Digitallfran" password="TuCl@veSegura" group=full comment="Usuario Administrador"
/user disable [find name=admin]
/system identity set name="Router Core - Administrador"
/system clock set time-zone-name=America/Bogota
/interface list
add name=WAN comment="Interfaces conectadas a Internet"
add name=LAN comment="Interfaces de la red local"
/interface ethernet
set [ find default-name=ether1 ] name="1 - WAN" comment="ISP conexion a Internet"
set [ find default-name=ether2 ] name="2 - LAN" comment="Red LAN"
set [ find default-name=ether3 ] name="3 - LAN" comment="Red LAN"
set [ find default-name=ether4 ] name="4 - LAN" comment="Red LAN"
set [ find default-name=ether5 ] name="5 - LAN" comment="Red LAN"
/interface bridge add name=Bridge-LAN comment="Bridge principal para unificar la red local"
/interface bridge port
add bridge=Bridge-LAN interface="2 - LAN"
add bridge=Bridge-LAN interface="3 - LAN"
add bridge=Bridge-LAN interface="4 - LAN"
add bridge=Bridge-LAN interface="5 - LAN"
/interface list member
add interface="1 - WAN" list=WAN
add interface=Bridge-LAN list=LAN
/ip dhcp-client add interface="1 - WAN" use-peer-dns=no use-peer-ntp=no add-default-route=yes comment="Configuracion automatica para la WAN del ISP"
/system ntp client set enabled=yes servers=time.google.com
/ip dns set allow-remote-requests=yes cache-size=4096KiB servers=1.1.1.1,8.8.8.8,9.9.9.9
/ip address add address=10.0.0.1/24 interface=Bridge-LAN comment="Gateway para la red LAN"
/ip pool add name=Pool-DHCP-LAN ranges=10.0.0.2-10.0.0.254 comment="Rango de IPs para clientes de la LAN"
/ip dhcp-server add address-pool=Pool-DHCP-LAN interface=Bridge-LAN name=DHCP-Server-LAN comment="Servidor DHCP para la red LAN"
/ip dhcp-server network add address=10.0.0.0/24 dns-server=10.0.0.1 gateway=10.0.0.1
/ip firewall nat add action=masquerade chain=srcnat out-interface-list=WAN comment="NAT para que la LAN salga a Internet"
/ip firewall filter
add action=fasttrack-connection chain=forward connection-state=established,related in-interface-list=LAN out-interface-list=WAN comment="Acelerar trafico LAN -> WAN"
add action=accept chain=forward connection-state=established,related comment="Permitir establecido y relacionado"
add action=drop chain=forward connection-state=invalid comment="Bota conexiones invalidas"
add action=accept chain=forward connection-state=new in-interface-list=LAN out-interface-list=WAN comment="Permitir nuevas conexiones desde LAN"
add action=drop chain=forward comment="Bota el resto del trafico forward"
add action=accept chain=input connection-state=established,related comment="Permitir establecido y relacionado al router"
add action=drop chain=input connection-state=invalid comment="Bota conexiones invalidas al router"
add action=accept chain=input in-interface-list=LAN comment="Permitir administracion desde LAN"
add action=drop chain=input comment="Bota el resto del trafico al router"
/ip service
disable [find where name="telnet" or name="ftp" or name="www" or name="ssh" or name="www-ssl"]
set [find name=winbox] port=9000
:log info "Script de Configuracion Inicial Aplicado Correctamente."
</code></pre>

<hr>

<div align="center" style="margin-top: 40px;">
  <img src="https://digitallfran.co/logo.svg" alt="Logo DigitAllFran" width="100"/>
  <p><strong>¿Necesitas soporte, consultoría o quieres llevar tu comunicación al siguiente nivel?</strong></p>
  <a href="https://digitallfran.co" target="_blank" rel="noopener noreferrer">¡Visita digitallfran.co y agenda tu consulta!</a>
</div>
