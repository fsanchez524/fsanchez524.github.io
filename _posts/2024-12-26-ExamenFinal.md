---
title: EXAMEN FINAL DE CIBERSEGURIDAD
date: 2024-12-29 00:00:00 -05:00
categories: [sysmon, procmon, cibersecurity, pentesting]
tags: [caldera, sysmon, procmon, kali, metasploit]  # TAG names should always be lowercase
---

<style>
ol.ieee {
  list-style: none; /* Elimina el estilo de lista predeterminado */
  counter-reset: item; /* Inicializa el contador */
}

ol.ieee li {
  counter-increment: item; /* Incrementa el contador para cada elemento */
}

ol.ieee li::before {
  content: "[" counter(item) "] "; /* Inserta el número entre corchetes */
}
</style>

<marquee behavior="alternate" style="font-size: 32px; font-weight: 800; font-family: Algerian" >EXAMEN FINAL</marquee>

<hr style="border-top: 10px double lightgreen">

## 1) **¿Cómo podrías utilizar Procmon y Sysmon juntos para investigar la actividad de un proceso sospechoso? (5 ptos)**

### **Explica los tipos de eventos que Procmon y Sysmon pueden capturar de forma complementaria.**

Con _System Monitor (Sysmon)_ y _Process Monitor (Procmon)_ se activan las fuentes de información cada vez que se utiliza una aplicación o se realiza una función que requiera recursos del sistema (hardware y software) y se va a obtener la información de los eventos y procesos del sistema. Cuando se ejecuta un proceso sospechoso como un malware o un proceso de ataque, los programas se utilizan con las siguientes finalidades:

_**PROCMON :**_

Procmon inicia la captura de eventos en tiempo real, relaciondos con la creación/modificación de archivos, registros y procesos en general. En este se puede observar la captura específica de procesos con características específicas como operación de escritura de archivo, lectura, rutas, cambios en registros, etc.


_**SYSMON :**_

Por otro lado, *Sysmon* se encarga de la captura de eventos realacionados con el proceso en cuestión y determina algunos eventos etiquetados como por ejemplo: *ProcessCreate*, *FileCreate* y *NetworkConnect*, que indican la creación de un proceso, creación de un archivo y conexión de red implementada respectivamente. Estos eventos pueden ser almacenados y analizados luego mediante un software o código para aplicar Machine Learning o algún método de detección de procesos anormales.

![alt text](/assets/images/images_Post_007/part1/image-39.png)

![alt text](/assets/images/images_Post_007/part1/image-38.png)

Cuadro de relación entre operaciones y eventos de **Procmon** y **Sysmon** para captura de eventos maliciosos:

|**Evento/Proceso**|**PROCMON**|**SYSMON**|
|Creación de proceso|ProcessStart|ProcessCreate(Event ID 1)|
|Conexión de red|TCPConnect, UDP Send, UDP Receive|NetworkConnection (Event ID 3|
|Acceso a proceso|No lo captura directamente|ProcessAcess (Event ID 10)|
|Inicio de flujo de datos en archivos|No capturado por Procmon|FileCreateStreamHash (Event ID 15)​|


### **EJEMPLO DE USO DE SYSMON Y PROCMON**
**1) Captura de eventos con Sysmon:** Se puede iniciar la captura de eventos específicos utilizando algún filtro, como por ejemplo, para la captura de la creación de procesos y accesos a memoria (**ProcessCreate** y **ProcessAccess**, *Event ID 1* y *10* respectivamente) y de estos eventos capturados, se puede determinar algún proceso sospechoso o inesperado que comprometa algún archivo importante del sistema.

**2) Uso de Procmon:** Una vez determinado un evento sospechoso, se puede emplear Procmon para monitorear las operaciones y se puede emplear algún código en **Python** para descubrir alguna modificación en algún archivo crítico. Por ejmplo: **C:\Windows\System32**.

**3) Seguimiento y persistencia de las operaciones:** Empleando ambos programas, se puede verificar si un proceso sobreescribe o modifica un registro crítico y genera flujo de datos persistentes, conexiones de red externas y desconocidas, etc.

Referencias: [7], [8]


<hr style="border-top: 10px double lightgreen">

## 2) **En Sysmon, ¿qué diferencias existen entre los eventos <code class="language-plaintext highlighter-rouge" style="background-color: black; color: lightgreen; font-size: 16px;">ProcessCreate</code> y <code class="language-plaintext highlighter-rouge" style="background-color: black; color: lightgreen; font-size: 16px;">ProcessAccess</code>, y qué utilidad tienen cada uno para un analista de seguridad? (5 ptos)**

### **- Describe los atributos principales de ambos eventos.**

Cada evento de Sysmon tiene diferentes atributos o características. Por ejemplo: El proceso ProcessCreate (Event ID 1) y algunos otros procesos tienen los atributos: RuleNanme, UtcTime (marca de tiempo en que sucedió), ProcessGuid (identificador único del proceso), ProcessId, etc. Hay atributos que no varian en el tiempo, por ejemplo Image (ruta del ejecutable).

_NOTA: En SysInternals solo se documenta los tags, descripciones, y otras características de los eventos de Sysmon, pero no hay información sobre los **atributos**._

**ProcessCreate:** Se genera cuando se crea un nuevo proceso. Proporciona información sobre el proceso padre, el proceso hijo, la línea de comandos utilizada y otros detalles relevantes.

**ProcessAccess:** Se captura cuando un proceso intenta acceder a otro proceso. Proporciona información sobre el proceso que intenta acceder y el proceso al que se accede.

|**Característica**|**ProcessCreate**|**ProcessAccess**|
|Event ID|N° 1|N° 10|
|Registro del evento|Se registra cada vez que se inicia un proceso en el sistema| Se captura cada vez que un proceso interactúa con otro|
|Atributos|RuleName, CommandLine, PPID, Hashes, *características o atributos del proceso padre*|nombre del proceso PID, objetivo, permisos, seguimiento de llamados|


### **- Investiga y menciona al menos dos escenarios donde cada evento podría ser clave en la detección de amenazas.**

- **ProcessCreate:** Para detección de procesos iniciados en el sistema con líneas de comando inusuales y para detección de scripts maliciosos como **cmd.exe** que ejecutan comandos inesperados.

- **ProcessAccess:** Para detectar algún proceso que sobreescribe en otro (cambios en registros), para detectar intentos de inyección de código o manipulación de procesos y para detectar *escalamiento de privilegios* debido a la modificación de archivos desde menor nivel para acceder a privilegios con cambios críticos en el sistema.

<hr style="border-top: 10px double lightgreen">

## 3) **En Procmon, ¿qué operación(es) corresponde(n) al evento <code class="language-plaintext highlighter-rouge" style="background-color: black; color: lightgreen; font-size: 16px;">FileCreateStreamHash</code> en Sysmon, y cómo podrías configurarlo en Sysmon para detectar un posible uso malicioso de Alternate Data Streams (ADS)? (5 ptos)**

### **- Alternate Data Streams (ADS) y su uso para ataques**

Los ADS son una funcionalidad de los sistemas de archivos **NTFS** *(New Technology File System)* que permite almacenar datos adicionales dentro de un archivo o carpeta sin afectar su contenido principal (metadatos).

Se puede emplear el siguiente filtro en **Sysmon** para detección de malware o proceso sospechoso con ADS:

```xml

<Sysmon schemaversion="4.22">
  <EventFiltering>
    <RuleGroup name="" groupRelation="or">
      <FileCreateStreamHash onmatch="include">
        <TargetFilename condition="contains">:Zone.Identifier</TargetFilename>
      </FileCreateStreamHash>
    </RuleGroup>
  </EventFiltering>
</Sysmon>

```

Otro ejemplo: 

```xml

<!--Para registrar eventos con ADS en rutas sospechosas-->


<RuleGroup name="Detectar ADS">
    <FileCreateStreamHash onmatch="include">
        <Image condition="is">C:\Windows\System32\cmd.exe</Image>
        <TargetFilename condition="contains">:</TargetFilename>
    </FileCreateStreamHash>
</RuleGroup>

```

Los atacantes lo emplean para ocultar datos maliciosos, como código ejecutable o configuraciones críticas , evitando la detección de herramientas tradicionales 

### **- Especifica qué operaciones de Procmon están relacionadas con este tipo de actividad.**

Procmon detecta los cambios de ADS como operaciones de archivo, como las operaciones **CreateFile** y **SetInformationFile**, que pueden indicar algun cambio o manipulación ADS, los cuales incluyen algun nombre de archivo con el formato: **archivo:stream** cuando se trata de un flujo alternativo.

<hr style="border-top: 10px double lightgreen">

## 4) **En Sysmon, ¿qué ventajas ofrece el uso de filtros avanzados en comparación con capturar todos los eventos de forma indiscriminada? (5 ptos)**

### **- Investiga cómo un mal diseño de filtros podría afectar el desempeño del sistema y la calidad de los logs.**

Un mal diseño de filtro en las capturas puede generar una sobrecarga en el sistema, debido al almacenamiento y procesamiento de información innecesaria, dando lugar a un bajo rendimiento del sistema y al aumento el ruido (captura de procesos innecesarios o normales sin enfoque óptimo hacia procesos importantes), haciendo difícil la detección de patrones anormales o sospechosos en el sistema.

Si se emplea un buen filtro, se puede reducir las cantidades de información sobre los procesos y eventos, se mejora el rendimiento del captura y se da lugar a un análisis óptimo sobre los eventos críticos en el sistema.

### **- Ejemplo de un filtro efectivo para reducir ruido en un entorno de producción.**

Se puede emplear los siguientes filtros para detectar procesos críticos (como cambios o flujo de datos en aplicaciones administrativas como PowerShell o CMD)

```xml

<RuleGroup name="Filtros Corporativos">
    <ProcessCreate onmatch="include">
        <Image condition="is">C:\Windows\System32\powershell.exe</Image>
        <CommandLine condition="contains">-EncodedCommand</CommandLine>
    </ProcessCreate>
</RuleGroup>

```

```xml 

<Sysmon schemaversion="4.22">
  <EventFiltering>
    <RuleGroup name="" groupRelation="or">
      <ProcessCreate onmatch="include">
        <Image condition="contains">C:\Windows\System32\cmd.exe</Image>
      </ProcessCreate>
      <FileCreate onmatch="include">
        <TargetFilename condition="contains">.exe</TargetFilename>
      </FileCreate>
    </RuleGroup>
  </EventFiltering>
</Sysmon>

```

<hr style="border-top: 10px double lightgreen">


## **REFERENCIAS**

<ol class="ieee">
  <li id="art1">R. Thomas, S. Steiner & D. Conte de Leon (2022). "HESPIDS: A Hierarchical and Extensible System for Process Injection Detection using Sysmon". 10.24251/HICSS.2022.905. [Online] <a href="https://scholarspace.manoa.hawaii.edu/server/api/core/bitstreams/1d5579e9-c7d4-411f-b166-6ef2de97cbd2/content">https://scholarspace.manoa.hawaii.edu/server/api/core/bitstreams/1d5579e9-c7d4-411f-b166-6ef2de97cbd2/content</a></li>

  <li id="art2">Smiliotopoulos, C., Kambourakis, G. & Barbatsalou, K. "On the detection of lateral movement through supervised machine learning and an open-source tool to create turnkey datasets from Sysmon logs". Int. J. Inf. Secur. 22, 1893-1919 (2023). https://doi.org/10.1007/s10207-023-00725-8[Online] <a href="https://link.springer.com/content/pdf/10.1007/s10207-023-00725-8.pdf">https://link.springer.com/content/pdf/10.1007/s10207-023-00725-8.pdf</a></li>

  <li id="art3">R. -V. Mahmoud, M. Anagnostopoulos, S. Pastrana and J. M. Pedersen, "Redefining Malware Sandboxing: Enhancing Analysis Through Sysmon and ELK Integration," in IEEE Access, vol. 12, pp. 68624-68636, 2024, doi: 10.1109/ACCESS.2024.3400167.[Online] <a href="https://ieeexplore.ieee.org/ielx7/6287639/6514899/10529261.pdf">https://ieeexplore.ieee.org/ielx7/6287639/6514899/10529261.pdf</a></li>

  <li id="art4">H. Brunner. "Detecting privacy leaks in the private browsing mode of modern web browsers through process monitoring". (Doctoral dissertation, Technische Universität Wien, 2014). [Online] <a href="https://web.archive.org/web/20220128143407id_/https://repositum.tuwien.at/bitstream/20.500.12708/8355/2/Brunner%20Herbert%20-%202014%20-%20Detecting%20privacy%20leaks%20in%20the%20private%20browsing%20mode%20of...pdf"> https://web.archive.org/web/20220128143407id_/https://repositum.tuwien.at/bitstream/20.500.12708/8355/2/Brunner%20Herbert%20-%202014%20-%20Detecting%20privacy%20leaks%20in%20the%20private%20browsing%20mode%20of...pdf</a></li>

  <li id="art5">U. E., E., & S. I., E. (2024). "Cuckoo Sandbox and Process Monitor (Procmon) Performance Evaluation in Large-Scale Malware Detection and Analysis". British Journal of Computer, Networking and Information Technology. [Online] <a href="abjournals.org/bjcnit/wp-content/uploads/sites/11/journal/published_paper/volume-7/issue-4/BJCNIT_FCEDOOMY.pdf">abjournals.org/bjcnit/wp-content/uploads/sites/11/journal/published_paper/volume-7/issue-4/BJCNIT_FCEDOOMY.pdf</a></li>

  <li id="art6">JACOBSEN, M. Douglas. "Procmon: Real-time process monitoring on the Cray XC-30". [Online] <a href="https://cug.org/proceedings/cug2015_proceedings/includes/files/pap175-file1.pdf">https://cug.org/proceedings/cug2015_proceedings/includes/files/pap175-file1.pdf</a></li>

  <li id="art7">Microsoft. SysInternals. PROCMON [Online] <a href="https://learn.microsoft.com/en-us/sysinternals/downloads/procmon">https://learn.microsoft.com/en-us/sysinternals/downloads/procmon</a></li>

  <li id="art8"> Microsoft. SysInternals. SYSMON [Online] <a href="https://learn.microsoft.com/es-es/sysinternals/downloads/sysmon">https://learn.microsoft.com/es-es/sysinternals/downloads/sysmon</a></li>

</ol>
