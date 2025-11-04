summary: Bastionado del arranque seguro en Linux (Debian)
id: bastionado-arranque-seguro-linux-debian
categories: seguridad, hardening
environments: Linux Debian
status: Published
authors: Nerea
feedback link: https://github.com/IES-Rafael-Alberti/hardening-ARRANQUE-2526
analytics account: UA-XXXXX

# Bastionado del arranque seguro en Linux (Debian)

## Introducción
Duration: 0:02:00

En esta guía aprenderás a proteger el arranque de un sistema Debian usando GRUB 2, configurando contraseñas cifradas, ocultando el menú y realizando otras configuraciones de seguridad.

---

## Fundamentos del gestor de arranque GRUB 2
Duration: 0:02:00

GRUB 2 es el gestor de arranque más utilizado en sistemas Linux. Utiliza archivos de configuración y scripts:

- `/boot/grub/grub.cfg`: Archivo principal generado automáticamente
- `/etc/default/grub`: Configuración general
- `/etc/grub.d/`: Scripts de configuración
- `/boot/grub/custom.cfg`: Archivo opcional para personalización

**Importante:** Nunca edites directamente `grub.cfg`, siempre modifica los archivos fuente y regenera la configuración.

---

## Paso 1: Editar archivo de configuración personalizado
Duration: 0:02:00

Primero accedemos al archivo donde configuraremos la protección:
sudo nano /etc/grub.d/40_custom


---

## Paso 2: Añadir superusuario y contraseña
Duration: 0:03:00

Dentro del archivo `/etc/grub.d/40_custom`, añade al final estas líneas:
set superusers="root"
password root TuContraseña

![Añadir configuración de contraseña](image/arranque1.png)


**Para mayor seguridad, usa contraseña cifrada:**

1. Genera el hash con:
sudo grub-mkpasswd-pbkdf2


2. Introduce tu contraseña dos veces

3. Copia el hash generado

*Referencia visual:*
![Añadir configuración de contraseña](image/arranque2.png)



---

## Paso 3: Configurar contraseña cifrada
Duration: 0:02:00

Reemplaza la línea de contraseña en texto plano por:
set superusers="root"
password_pbkdf2 root grub.pbkdf2.sha512.10000.[TU_HASH_COMPLETO]


Donde `[TU_HASH_COMPLETO]` es el hash que generaste en el paso anterior.

*Referencia visual:*
![Configuración con hash](image/arranque3.png)


---

## Paso 4: Proteger permisos del archivo
Duration: 0:01:00

Cambia los permisos para que solo root pueda modificar el archivo:

sudo chmod 700 /etc/grub.d/40_custom


*Referencia visual:*
![Cambiar permisos](image/arranque5.png)

---

## Paso 5: Ocultar menú de arranque 
Duration: 0:02:00

Para ocultar el menú GRUB y acelerar el arranque:

1. Edita el archivo de configuración:
sudo nano /etc/default/grub

2. Busca la línea `GRUB_TIMEOUT=5` y cámbiala a:
GRUB_TIMEOUT=0


3. Guarda y cierra

*Referencia visual:*
![Configurar timeout](image/arranque6.png)

---

## Paso 6: Aplicar cambios
Duration: 0:01:00

Actualiza la configuración de GRUB para aplicar todos los cambios:

sudo update-grub


Este comando regenera el archivo `/boot/grub/grub.cfg` con las nuevas configuraciones.

*Referencia visual:*
![Actualizar GRUB](image/arranque7.png)

---

## Comprobaciones y pruebas
Duration: 0:02:00

### Verificar timeout configurado

grep GRUB_TIMEOUT /etc/default/grub


Debe mostrar `GRUB_TIMEOUT=0` si configuraste ocultación del menú.

![Mostrar GRUB_TIMEOUT=0](image/arranque8.png)

### Verificar Secure Boot 
mokutil --sb-state


**Nota:** En máquinas virtuales sin UEFI, verás "EFI variables are not supported".

### Verificar cifrado de disco
lsblk -o NAME,FSTYPE,MOUNTPOINT | grep crypt


### Prueba de arranque

Reinicia el sistema y verifica:
- El menú GRUB pide contraseña para modificar opciones
- Sin contraseña correcta, no se puede acceder al menú
- Si configuraste timeout=0, el menú está oculto y arranca directamente

---

## Paso 0 (antes de modificar): Copia de seguridad (muy importante)
Antes de editar cualquier archivo importante, haz una copia de seguridad para poder restaurar la configuración original si algo falla:

sudo cp /etc/default/grub /etc/default/grub.bak
sudo cp /etc/grub.d/40_custom /etc/grub.d/40_custom.bak

También puedes hacer un respaldo completo así:

sudo tar czvf grub_backup_$(date +%F).tar /etc/default/grub /etc/grub.d/ /boot/grub/grub.cfg

## Recomendaciones adicionales
Duration: 0:01:00

- **Secure Boot:** Es una característica de la  que evita que se cargue software no autorizado durante el arranque, protegiendo contra malware y rootkits. Se activa en la configuración del BIOS/UEFI.

- **Actualización de firmware:** Mantén el sistema actualizado para corregir vulnerabilidades.

- **Cifrado de disco:**  Protege todos los datos del disco solicitando una contraseña al arrancar el sistema. Es una opción que se puede seleccionar durante la instalación o configurar posteriormente.

- **Backup de configuración:**  Siempre guarda copias de tus archivos de configuración antes de modificar.


---

## Referencias

- [Documentación oficial GRUB](https://www.gnu.org/software/grub/manual/)
- [Guía Debian Secure Boot](https://wiki.debian.org/SecureBoot)
- [GRUB Password Protection](https://help.ubuntu.com/community/Grub2/Passwords)

---

## Conclusión

Has configurado exitosamente el bastionado del arranque en Debian protegiendo GRUB con contraseña cifrada, 
ocultando el menú y aplicando buenas prácticas de seguridad.


