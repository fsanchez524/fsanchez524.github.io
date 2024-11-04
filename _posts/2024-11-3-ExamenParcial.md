---
title: Examen Parcial de Ciberseguridad
date: 2024-11-3 00:00:00 -05:00
categories: [buffer overflow, cibersecurity, kill chain, pentesting]
tags: [buffer overflow, windows server vulnerability, crafted packet, kali, metasploit]  # TAG names should always be lowercase
---

Pasos preliminares:
- Firewall apagado en el Metasploitable (Win2008):
![alt text](/assets/images/parcial/firewall_apagado.png)

- Servidor creado
![alt text](/assets/images/parcial/files_server_creado.png)





### Instrucciones:
1) **Escaneo de Red y Enumeración de Servicios (2 puntos)**
*Utiliza Nmap desde tu máquina atacante para descubrir servicios activos en la máquina objetivo. Debes identificar los puertos relacionados con SMB y verificar si SMBv1 está habilitado.
Tip: Investiga cuáles son los 02 puertos asociados con el servicio SMB. Luego, investiga como puedes usar nmap con el flag --script para una enumeración detallada. Al analizar el output del comando, buscar específicamente el término SMBv1 o el protocolo NT LM 0.12 en la lista de protocolos. Ello indicará la presencia del servicio SMBv1.
Escribe el comando que usaste en Nmap y explica brevemente cómo determina si SMBv1 está activo.*

- Usando el siguiente comando en Windows (Metasploitable) para verificar el puerto 445 de SMB en escucha:
```bash
netstat -an
```
![alt text](/assets/images/parcial/image-5.png)

- Utilizando el siguiente comando para determinar los puertos abiertos en la red, teniendo en cuenta que se tiene una conexiòn con el servidor con IP 10.0.2.4 (víctima) con los servicios de File Services. Para determinar si existe conexión con SMB se debe verificar el puerto 445.
```bash
nmap -A -p 445 10.0.2.4
```
![alt text](/assets/images/parcial/image-1.png) 
Se observa que el puerto 445 del protocolo SMB está abierto.

2) **Exploración de la Vulnerabilidad (2 puntos)**
*Utiliza los resultados del escaneo para identificar si el servicio SMBv1 presenta alguna vulnerabilidad conocida. Indica si el servidor es vulnerable al exploit EternalBlue (MS17-010).
Tip: Para explotar esta vulnerabilidad, usa el framework Metasploit y selecciona el módulo específico exploit/windows/smb/ms17_010_eternalblue, con el objetivo de establecer una conexión reverse shell hacia la máquina atacante.
Explica brevemente el funcionamiento del script exploit/windows/smb/ms17_010_eternalblue y por qué se selecciona este módulo en particular para obtener acceso remoto.*

- Iniciamos abriendo el mòdulo de Metasploit usando el siguiente comando, con la bandera -q para saltear la presentación de la herramienta:
```bash
msfconsole -q
```
Y posteriormente ingresamos el módulo indicado:
```bash
msf6 > use exploit/windows/smb/ms17_010_eternalblue
```
![alt text](/assets/images/parcial/image-6.png)

Y revisamos las opciones de la herramienta:
![alt text](/assets/images/parcial/image-7.png)

Verificamos la dirección IP del host atacante (Kali Linux):
![alt text](/assets/images/parcial/image-8.png)

#### Funcionamiento del exploit **ms17_010_eternalblue**
El script ms17_010_eternalblue.rb es un exploit para la vulnerabilidad EternalBlue, que se aprovecha de una falla en el protocolo SMBv1 para permitir la ejecución remota de código en sistemas Windows vulnerables. Se puede encontrar en el siguiente link de github:
*https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/windows/smb/ms17_010_eternalblue.rb*

**Funciones más importantes del script ms17_010_eternalblue.rb:**

- **Configuración de la Vulnerabilidad:**

**Rex::Proto::SMB::Session.create:** Esta función se utiliza para establecer una sesión SMB con el host objetivo. Es crucial para la comunicación inicial con el sistema vulnerable.
Exploración de la Vulnerabilidad:

**check:** Esta función verifica si el host objetivo es vulnerable a la explotación de EternalBlue. Realiza una serie de comprobaciones para determinar si el sistema puede ser explotado.
Preparación del Payload:

**generate_payload:** Esta función genera el payload que se enviará al sistema objetivo. El payload puede incluir código malicioso que se ejecutará en el sistema vulnerable.
Envío del Payload:

**send_payload:** Esta función envía el payload generado al sistema objetivo. Utiliza el protocolo SMB para transmitir el payload al sistema vulnerable.
Explotación de la Vulnerabilidad:

**exploit:** Esta es la función principal que orquesta la explotación de la vulnerabilidad. Combina las funciones de configuración, verificación, generación de payload y envío para llevar a cabo la explotación completa.

- **Manejo de la Sesión:**

**session:** Esta función maneja la sesión establecida con el sistema objetivo después de la explotación. Permite la ejecución de comandos y la interacción con el sistema comprometido.
Limpieza y Cierre:

**cleanup:** Esta función realiza cualquier limpieza necesaria después de la explotación. Esto puede incluir el cierre de conexiones y la eliminación de archivos temporales.

#### ¿Por qué emplear este módulo?
El módulo ms17_010_eternalblue permite explotar la vulnerabilidad MS17-010 en el protocolo SMBv1 de Windows y requiere que la máquina objetivo esté ejecutando una versión vulnerable de Windows (Windows 7, Windows Server 2008, etc.). Nosotros al emplear **Windows Server 2008 R2** (víctima) con el firewall **apagado** nos facilita debida a que este servidor y version cuenta con la vulnerabilidad de **SMB** y podemos explotar el sistema. Cabe resaltar que el exploit cuenta con facilidades como control del sistema, propagación de malware, acceso a redes internas y *backdoors*.

3) **Configuración del Exploit en el Framework Metasploit (2 puntos)**
*Configura el exploit mencionado anteriormente, especificando los parámetros necesarios, como las direcciones IP del atacante y la víctima.
Tip: Configura los parámetros RHOSTS para la IP de la máquina objetivo y LHOST para tu máquina atacante. Estos son esenciales para establecer la conexión.
Tip adicional: También debes configurar el payload para el reverse shell. Busca en el metasploit un payload (que ya anteriormente hemos usado en clase) para obtener un reverse shell connection.
Explica brevemente cada parámetro crítico que configuraste, como LHOST y RHOSTS, y justifica por qué esos valores son importantes para el ataque.*

#### Configuración de los parámetros del sploit:
- Ingresamos los datos de dirección de host local (LHOST), host remoto (RHOST) y PAYLOAD (windows/x64/meterpreter/reverse_tcp) para posteriormente ejecutar con el comando **exploit** o **run**. Se debe configurar ambas direcciones de víctima y atacante para definir el ataque, ya que si no se especifica los hosts, el sploit podrìa realizarse hacia otros *hosts*, servidores o *gateways* presentes en la red.
![alt text](/assets/images/parcial/image-2.png)
![alt text](/assets/images/parcial/image-3.png)


4) **Ejecución del Exploit (2 puntos)**
*Ejecuta el exploit para obtener una conexión Meterpreter con el servidor. Si la explotación es exitosa, documenta qué mensaje o salida te confirma que la conexión ha sido establecida.
Tip: Al ejecutar el exploit, Metasploit intentará establecer una sesión Meterpreter en la máquina víctima si el ataque es exitoso.
Tip adicional: Documenta cualquier mensaje en la salida de Metasploit que indique éxito, como Meterpreter session X opened. Esto confirma la conexión.*

- Luego de ejecutar el exploit, se observa el proceso de conexión, envío de paquetes y otros procedimientos. Al final se observará la conexión y sesión con *meterpreter*.
![alt text](/assets/images/parcial/image-4.png)

5) **Exfiltración de Archivos SAM y SYSTEM (2 puntos)**
*Utilizando la sesión de Meterpreter establecida, explica los pasos para infiltrar un script o herramienta que te permita crear una copia de volumen de sombra (Volume Shadow Copy) de la unidad principal (C:) del servidor.
Describe brevemente cómo utilizarías esta copia de sombra para acceder a los archivos SAM y SYSTEM. Enumera los comandos necesarios en Meterpreter o la shell de Windows para:
Crear la copia de sombra.
Copiar los archivos SAM y SYSTEM a un directorio accesible.
Descargar los archivos a tu máquina atacante.
Tip: Utiliza el enlace de descarga proporcionado para obtener el script vssown.vbs e infíltralo en la máquina víctima usando Meterpreter.
Tip adicional: Después de ejecutar el script vssown.vbs, toma nota de la ubicación de la copia de volumen de sombra que se crea (ruta similar a \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopyX).*

- Verificamos la descarga del script *vssown.vbs*
![alt text](/assets/images/parcial/image-9.png)

- Y también verificamos en el terminal usando *change directory* o **cd**:
![alt text](/assets/images/parcial/image-10.png)

#### Procedimiento en Meterpreter para copiar una archivo al host remoto
Recordar que la ID de la sesión actual es *4*.
- 

6) Análisis de los Archivos Exfiltrados (2 puntos)
Una vez que hayas descargado los archivos SAM y SYSTEM, describe el procedimiento que seguirías en tu máquina atacante para extraer los hashes de contraseñas. Especifica qué herramientas emplearías para realizar esta tarea.

Tip: Una vez descargados los archivos SAM y SYSTEM a tu máquina Kali Linux, usa herramientas como samdump2 para extraer los hashes.

Tip adicional: Con los hashes extraídos, puedes intentar descifrarlos con herramientas como John the Ripper o Hashcat.

7) Requisitos de Documentación
Documentar en su respectivo blog.

Para cada paso, proporciona capturas de pantalla y explica en tus propias palabras qué estás haciendo y por qué es relevante en un proceso de auditoría de seguridad.

Asegúrate de usar herramientas comunes en entornos de pentesting (por ejemplo, Nmap, Metasploit, samdump2, John the Ripper).

Tip: Asegúrate de capturar cada paso clave con capturas de pantalla y explica en tu blog lo que hace cada comando o herramienta.








