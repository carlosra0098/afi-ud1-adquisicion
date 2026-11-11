USB encontrado a las 11:44 a fecha de 11/11/2026.

# Adquisición en vivo

Estado inicial del ordenador:
Hora del sistema --> 11:44
Hora real --> 11:44
Desfase --> Nada.
![[Pasted image 20261111120853.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111120853.png)

## Memoria RAM del equipo usando DumpIt. Hora de realización --> 12:13
Tras usar DumpIt, el programa nos da el hash en el archivo FORENSE-06-20261111-120148.raw.
![[Pasted image 20261111131036.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111131036.png)

## Datos volátiles usando el script respuesta-en-vivo.ps1. Hora de realización --> 13:10.
Tras utilizar el script crea una carpeta llamada "salida" donde se encuentran todos los datos volátiles dentro de la carpeta ADI-UD1-AlmadrabaDiseno.
![[Pasted image 20261111132827.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111132827.png)
![[Pasted image 20261111132913.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111132913.png)

## Tráfico de red usando los siguientes comandos. Hora de realización --> 13:30:
``` bash
VBoxManage controlvm "FORENSIC_10" nictracefile1 /ruta/caso/red.pcap
VBoxManage controlvm "FORENSIC_10" nictrace1 on     # y "off" al terminar
```

Tras usar esos dos comandos nos genera un archivo llamado red.pcap donde aparece todo el tráfico de red que tiene el equipo.
![[Pasted image 20261111133812.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111133812.png)

## Triaje usando KAPE. Hora de realización --> 13:38.

Al no poder usar KAPE ni Velociraptor, he tenido que usar herramientas manuales para poder realizarlo usando los siguientes comandos mediante powershell:

``` Bash
1.Extraer colmenas del Registro:Usuarios, USBs y Sistema.Ejecuta estos tres comandos uno a uno para volcar el registro en vivo: reg save HKLM\SAM "D:\salida\SAM.hiv" reg save HKLM\SYSTEM "D:\salida\SYSTEM.hiv" reg save HKLM\SOFTWARE "D:\salida\SOFTWARE.hiv" Verificación: La consola devolverá el mensaje "La operación finalizó correctamente" para cada colmena. 
2.Extraer Visor de Eventos:Inicios de sesión y errores.Utiliza el comando nativo wevtutil para exportar los logs críticos: wevtutil epl System "D:\salida\System.evtx" wevtutil epl Security "D:\salida\Security.evtx" 
3.Extraer Prefetch:Rastro de programas ejecutados.El directorio Prefetch no suele estar bloqueado por el sistema, por lo que puedes copiarlo directamente: Copy-Item -Path "C:\Windows\Prefetch*" -Destination "D:\salida\Prefetch" -Recurse
```
![[Pasted image 20261111135203.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111135203.png)

## Comprobación de cifrado de discos. Hora de realización --> 13:56

Tras usar el siguiente comando _manage-bde -status_, aparece  que el disco esta descifrado completamente y sin BitLocker activo.
``` bash
manage-bde -status
```
![[Pasted image 20261111135923.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111135923.png)

## Cómo apagar el equipo. Hora de realización --> 14:00

Al no haber volúmenes cifrados, se opta por hacer un apagado forzado (corte de corriente), ya que no se pierde acceso a ningún dato al cortar la corriente. Un apagado ordenado ejecutaría scripts de cierre, tareas programadas y servicios que podrían modificar o borrar evidencias (p. ej. mecanismos de autodestrucción). Antes del apagado ya se había adquirido todo lo volátil (RAM, datos volátiles, red, triaje). 

Riesgo asumido: posible inconsistencia del sistema de ficheros por el corte, aceptable porque se adquiere la imagen completa del disco después.

Para la máquina virtual se utilizaría un comando por powershell externo para simular el corte de corriente, que sería el siguiente:
``` bash
VBoxManage controlvm "FORENSIC_10" poweroff
```

# Adquisición Post-Mortem

## ¿Qué discos tiene FORENSIC_10? 

Tras mirar en Almacenamiento, solamente tiene un disco llamado FORENSIC_10_disk0.vdi. Hora de realización --> 14:04
![[Pasted image 20261111142506.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111142506.png)

## ¿Hay alguna instantánea? 

Tras mirar en Instantáneas, podemos ver que no hay ninguna instantánea tomada. Hora de realización --> 14:05
![[Pasted image 20261111142727.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111142727.png)

## ¿Cuál es la ruta de la carpeta de la VM? 

C:\Users\clrg0\VirtualBox VMs\FORENSIC_10 Hora de realización --> 14:06
![[Pasted image 20261111142832.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111142832.png)

## Hashes. Hora de realización --> 14:30

El comando usado para poder obtener todos los hashes es el siguiente:
``` bash
get-childitem -Path . -File -Recurse | get-filehash
```
Se tiene que usar abriendo una terminal Powershell desde la ruta de la máquina virtual. 
## Hash del disco .vdi

Algorithm: SHA256

Hash: 36DC40371E4DF95945262C6BF5B83016C84C4275A6A298A74938990371ACFB74  

Path: C:\Users\clrg0\VirtualBox VMs\FORENSIC_10\FORENSIC_10_disk0.vdi

## Hash del disco .vbox
Algorithm: SHA256 

Hash: 8FC7014663AAB75066838697CC72968411FA042767E16BE1D383F954F25B73C8

Path: C:\Users\clrg0\VirtualBox VMs\FORENSIC_10\FORENSIC_10.vbox

## Hashes de los logs

Algorithm: SHA256

Hash: 9BF02386C38403C0A4AE4A9D5669F55CC01B3F8E5B8A0888824F34275DE0961A

Path: C:\Users\clrg0\VirtualBox VMs\FORENSIC_10\Logs\VBox.log

Algorithm: SHA256

Hash: A13BFC0C301E3E6065F08367065D4A467E7EC187B7714CF9CC391FD5EA2C08DC

Path: C:\Users\clrg0\VirtualBox VMs\FORENSIC_10\Logs\VBox.log.1

Algorithm: SHA256

Hash: 61C2083C5D97F7C3B76AB2B961A2D837CC73B4B7E98E9C5174B2215B71E74FB2

Path: C:\Users\clrg0\VirtualBox VMs\FORENSIC_10\Logs\VBox.log.2

Algorithm: SHA256

Hash: 91BF0C7C0944CF78E87E2AD02F238C40E0F0E92749567943BBE763D06AC029F0

Path: C:\Users\clrg0\VirtualBox VMs\FORENSIC_10\Logs\VBoxHardening.log

## Imagen del disco. Hora de realización --> 16:58

Para realizar la imagen del disco he utilizado FTKImager para así poder crearla y verificarla. 

En la siguiente captura se puede ver la creación completa:
![[Pasted image 20261111170215.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111170215.png)

En la siguiente captura se puede ver la verificación de la creación:
![[Pasted image 20261111173252.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111173252.png)

## Mismo hash del .vdi. Hora de realización --> 17:30

Tras calcular el hash del disco copiado, podemos ver en la siguiente captura que da exactamente el mismo hash al original el cual se mencionó antes en el apartado _Hash del disco .vdi_.
![[Pasted image 20261111173725.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111173725.png)
## Copia de configuración y registros. Hora de realización --> 17:35

Tras haberlos copiado junto a la imagen y comprobar los hashes, en la siguiente captura se ve que tienen el mismo hash que anteriormente se mencionó en el apartado _Hash del disco .vbox_ y _Hashes de los Logs_.

Hash de .vbox:
![[Pasted image 20261111174543.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111174543.png)

Hash de los Logs:
![[Pasted image 20261111174634.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111174634.png)

## Pendrive encontrado. Hora de realización --> 18:00
![[WhatsApp Image 2026-11-11 at 18.24.14.jpeg|287]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/WhatsApp%20Image%202026-11-11%20at%2018.24.14.jpeg)

El pendrive encontrado es un 4GB TDK TRANS IT USB Flash Drive 4GB TDK USB Stick de 4Gb de capacidad de almacenamiento. Su número de referencia es Ref-4198.

Tras haber usado Caine con el pendrive, obtengo la copia física de este mismo y la información en un .info.

# Busca tu marca. Hora de realización --> 20:20

Tras ejecutar el comando nos aparece la marca de la RAM.
``` bash
strings -el FORENSE-06-20261111-120148.raw | grep "AFI-"
```
![[Pasted image 20261111203109.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111203109.png)

Tras montar la imagen del disco en solo lectura y con Administrador, podemos encontrar el fichero:
![[Pasted image 20261111204511.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111204511.png)

Al hacerlo con el pendrive, da un error que desconozco y no lo pude solucionar:
![[Pasted image 20261111210520.png]](https://github.com/carlosra0098/afi-ud1-adquisicion/blob/main/capturas/Pasted%20image%2020261111210520.png)


