# Laboratorio SDN para el aula: Open vSwitch, OpenFlow y un switch que aprende en una sola VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml)

Un **laboratorio de redes definidas por software (SDN)** listo para usar para enseñar cómo un switch reenvía el tráfico, de la [plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning), donde los centros educativos dan a cada alumno sus propias máquinas y su terminal en el navegador. Cada alumno recibe una **VM** de Ubuntu con **Open vSwitch**, **Mininet** y la biblioteca de controladores OpenFlow **os-ken**, y un switch `br0` con tres equipos, `h1`, `h2` y `h3`. El switch empieza con la tabla de flujos vacía, así que no reenvía nada. El alumno escribe **flujos OpenFlow a mano** con `ovs-ofctl` y después termina un **controlador de switch con aprendizaje** que aprende en qué puerto está cada dirección MAC e instala un flujo para ella. No se instala nada en el ordenador del alumno y no hace falta equipo de red.

## Qué hace el alumno

El laboratorio escribe un `README.md` en la carpeta personal que explica qué hacer y no da ninguna respuesta, y dos archivos para editar, `/etc/sdnlab/flows.txt` y `/etc/sdnlab/switch.py`, que se cargan cada vez que arranca el laboratorio. Los equipos son espacios de nombres de red (`h1` es 10.0.0.1, `h2` 10.0.0.2, `h3` 10.0.0.3, con direcciones MAC que acaban en 01, 02 y 03) y el switch está en fail-mode secure, así que sin ningún flujo descarta todos los paquetes. En tres partes:

1. **Mininet** (sin nota): `sudo mn --mac --test pingall`, y luego un prompt de Mininet para ver nodos, enlaces y los flujos que instala su controlador.
2. **Escribir la tabla de flujos a mano** en `flows.txt`: h1 y h2 deben alcanzarse en ambos sentidos (ARP además del eco), un flujo que coincida con h3 debe descartarlo y ningún flujo puede usar la acción `NORMAL`.
3. **Terminar un controlador** en `switch.py`: tres pasos TODO en `packet_in` para aprender la dirección de origen, elegir el puerto e instalar un flujo, de modo que los siguientes paquetes de una conversación nunca lleguen al controlador.

El laboratorio añade un comando `sdnlab` (`status`, `flows`, `controller`, `reset`, `shell h1`) para reconstruir el laboratorio, arrancar el controlador, devolver los dos archivos a como se entregaron y abrir una shell dentro de un equipo.

## Cómo se califica

El laboratorio tiene una **comprobación**, un script que califica una VM entregada sin que el docente tenga que leerla. Se ejecuta sobre una copia de la máquina guardada tras un arranque limpio y carga ella misma los dos archivos del alumno, así que un flujo escrito a mano y nunca guardado no cuenta.

- **Parte 2, sin controlador:** h1 y h2 deben alcanzarse, h3 no debe alcanzar a nadie ni nadie a h3, y un flujo que coincida con h3 debe descartarlo y haber recibido paquetes de h3. Un flujo con la acción `NORMAL` no pasa.
- **Parte 3, el controlador del alumno:** la comprobación lo arranca, da a h2 una dirección MAC que nadie ha visto, de modo que una tabla escrita de antemano no sirve, y hace ping entre todos los pares. El switch debe enviar los paquetes desconocidos al controlador, y el controlador debe haber instalado un flujo por cada par de direcciones, escrito como `priority=1,in_port=PORT,dl_src=SOURCE_MAC,dl_dst=DESTINATION_MAC` con la acción de salida al puerto aprendido, y ninguno de los flujos instalados puede inundar.

La primera línea que imprime dice qué no está bien.

Lo que no hace: prueba con ping y lee la tabla de flujos, no otros tipos de tráfico, y un alumno con root en su propia VM aún podría manipular las herramientas en las que se apoya la comprobación. Es una comprobación para el aula, no una prueba.

## Qué hay en el archivo

`sdn-lab.lab.yaml` es una **definición de laboratorio VM** completa, exactamente lo que ejecuta la plataforma:

| Clave | Qué es |
| --- | --- |
| `cpus`, `memory`, `disk` | el tamaño de la VM de cada alumno: 1 CPU, 1024 MB y 10 GB |
| `open` | los puertos que se abren en el navegador: ninguno, el laboratorio es la terminal |
| `setup` | un script que se ejecuta una vez, como root, la primera vez que arranca la VM de cada alumno: instala Open vSwitch, Mininet, os-ken, `iproute2`, `ping` y `tcpdump` desde el archivo de Ubuntu, y escribe el comando `sdnlab`, un pequeño lanzador del controlador, los dos archivos para editar, una unidad de systemd y el README |
| `check` | el script de calificación, ejecutado como root sobre una copia del disco guardado tras un arranque limpio; su código de salida es el resultado y `0` significa aprobado |

## Usarlo en la plataforma

**Para ponerlo como tarea a una clase:**

1. En su módulo, añada una tarea y elija la tarjeta **Laboratorio VM de Ubuntu**.
2. Pulse el ejemplo Software-defined networking, o pegue el contenido de `sdn-lab.lab.yaml`. Usar un ejemplo reemplaza el texto, y Deshacer recupera el suyo.
3. Publique y pruebe primero su propia copia. Cada alumno tiene su propia VM, que se conserva entre sesiones. El primer arranque tarda unos minutos mientras el script instala los paquetes.
4. Asígnelo como cualquier tarea. Cuando se entregue, ejecute la comprobación en la fila de un alumno en la lista de la clase: arranca una copia de su máquina guardada que no se vuelve a guardar, y muestra si pasó y qué imprimió.

Una lección de Fundamentos de la plataforma, SDN Basics, se trabaja en este laboratorio. La [página de este laboratorio](https://mitsurutech.co.uk/es/help/lab-sdn-lab) en el manual, su [guía de laboratorios VM de Ubuntu](https://mitsurutech.co.uk/es/help/labs) y su [página de laboratorios de ejemplo](https://mitsurutech.co.uk/es/help/sample-labs) cubren el resto.

¿Aún no tiene cuenta para su centro? [El primer año es gratis](https://mitsurutech.co.uk/es/pricing) hasta 50 alumnos, y [registrarse](https://mitsurutech.co.uk/es/signup) solo pide una dirección de correo.

## Comprobar el archivo

En cada push, `.github/workflows/check.yml` comprueba que el archivo es YAML válido con las claves anteriores y que `setup` y `check` caben cada uno en el límite de 16 KB de la plataforma. No arranca ninguna VM: eso se hace en las máquinas de la plataforma.

## Qué no funciona en un laboratorio

La VM llega al archivo de paquetes de Ubuntu, PyPI, npm y GitHub, y a nada más. La red entre los equipos es virtual y está dentro de la VM, así que nada sale de ella. La biblioteca os-ken depende de eventlet, que está obsoleta e imprime un aviso en el registro del controlador; por eso el laboratorio enseña primero `ovs-ofctl` y Mininet. El comando `sudo mn -c` de Mininet elimina todos los puentes de Open vSwitch, `br0` incluido: ejecute después `sudo sdnlab flows` para reconstruir el laboratorio. El script de preparación y la comprobación se ejecutaron en un contenedor que hace de VM; conviene probar una VM de alumno real antes de una clase.

## Acerca de

Creado por [Mitsuru Technologies](https://mitsurutech.co.uk/es) para la [plataforma de aprendizaje MitsuruTech](https://mitsurutech.co.uk/es/learning): Python, Java, Go, Node, SQL y Linux en el navegador para colegios y centros educativos, con laboratorios de contenedores, laboratorios VM, aplicaciones web desplegadas desde GitHub y cuestionarios que se corrigen solos. Pruebe código sin cuenta en el [playground](https://mitsurutech.co.uk/es/playground).

Más laboratorios del mismo sitio: [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea), [lab-sample](https://github.com/mitsurutech/lab-sample), [lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab), [lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab), [lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab), [lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab), [lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab), [lab-ansible-roles](https://github.com/mitsurutech/lab-ansible-roles) y [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics).

## Licencia

Esta obra está bajo la licencia [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Puede compartirla y adaptarla para aprender y enseñar, siempre que cite a Mitsuru Technologies Ltd, no la use con fines comerciales y comparta lo que haga a partir de ella bajo la misma licencia. El texto completo está en el archivo `LICENSE`.

[Open vSwitch](https://www.openvswitch.org), Mininet y os-ken se instalan desde el archivo de Ubuntu cuando arranca una VM; aquí no se distribuye nada de ellos.
