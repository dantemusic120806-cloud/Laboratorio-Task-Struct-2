# Laboratorio-Task-Struct-2
```markdown
# Informe de Laboratorio: Exploración Autónoma - Anatomía de un Proceso en Linux

## 1. Introducción

En un sistema operativo basado en Linux, la diferencia fundamental entre una entidad pasiva (un programa almacenado en disco) y una entidad activa (un proceso en ejecución en la memoria RAM) radica en el control que el núcleo ejerce sobre sus recursos. Para gestionar los registros de la CPU, la memoria virtual, la jerarquía y los descriptores de archivos, el kernel de Linux utiliza un descriptor de proceso universal conocido como el **Bloque de Control de Proceso (PCB)**. En el código fuente de C, este PCB se implementa mediante la estructura `struct task_struct`.

---

## 2. Entorno de Trabajo y Localización

* **Entorno de Trabajo:** Linux Mint virtualizado en Oracle VM VirtualBox.
* **Ruta de Cabeceras del Kernel:** `/usr/src/linux-headers-$(uname -r)/include/linux/sched.h`
* **Archivo Principal de Análisis:** `sched.h` (donde reside la definición de `struct task_struct`).

---

## 3. Análisis Estructural: 10 Subcampos Clave de `struct task_struct`

A partir de la inspección del archivo de cabecera `sched.h`, se identificaron y documentaron los 10 subcampos clave requeridos por la guía de laboratorio:

| # | Nombre del Campo | Tipo en C | Subsistema Asociado | Propósito / Descripción |
|---| :--- | :--- | :--- | :--- |
| 1 | **`pid`** | `pid_t` | Identificación | Identificador único del hilo o proceso (*Process ID*). |
| 2 | **`tgid`** | `pid_t` | Identificación | Identificador del grupo de hilos (*Thread Group ID*). Para el usuario y herramientas como `ps` o `top`, equivale al PID visible del proceso. |
| 3 | **`comm`** | `char [16]` | Identificación | Almacena el nombre del ejecutable que dio origen al proceso (truncado a 15 caracteres legibles). |
| 4 | **`__state` / `state`** | `volatile long` / `unsigned int` | Planificador (*Scheduler*) | Variable donde se registran las transiciones de estado del ciclo de vida del proceso (`TASK_RUNNING`, `TASK_INTERRUPTIBLE`, `EXIT_ZOMBIE`, etc.). |
| 5 | **`prio`** | `int` | Planificador (*Scheduler*) | Define la prioridad dinámica asignada por el algoritmo de planificación para el uso de la CPU. |
| 6 | **`real_parent`** | `struct task_struct *` | Jerarquía de Procesos | Puntero que señala directamente al proceso padre legítimo que creó la tarea mediante `fork()`. |
| 7 | **`mm`** | `struct mm_struct *` | Memoria Virtual | Referencia la estructura encargada de administrar el espacio de direcciones virtuales del proceso (tablas de páginas, heap, stack). |
| 8 | **`files`** | `struct files_struct *` | Sistema de Archivos (`fs`) | Administra la tabla de descriptores de archivos abiertos, tuberías (*pipes*) y sockets asignados. |
| 9 | **`signal`** | `struct signal_struct *` | Comunicación IPC | Gestiona el estado de las señales compartidas, manejadores de señales y señales pendientes. |
| 10| **`cred`** | `struct cred *` | Seguridad | Contiene las credenciales de seguridad del proceso, incluyendo UID, GID y *capabilities*. |

---

## 4. Verificación Empírica (`/proc/self/status`)

Mediante la ejecución del comando `cat /proc/self/status` directamente en la terminal de la máquina virtual local, se obtuvieron los siguientes valores reales en tiempo de ejecución:

```text
Name:   cat
State:  R (running)
Tgid:   3804
Pid:    3804
PPid:   3763
Uid:    1000    1000    1000    1000

```

### Mapeo entre Salida de Terminal y el Código C:

| Salida en Terminal (`/proc/self/status`) | Campo correspondiente en `struct task_struct` | Interpretación Empírica |
| --- | --- | --- |
| **`Name: cat`** | `comm` | Nombre del comando o ejecutable en curso ejecutado en el proceso. |
| **`State: R (running)`** | `__state` / `state` | Estado activo de ejecución (*Running*) dentro del planificador del sistema operativo. |
| **`Pid: 3804` / `Tgid: 3804**` | `pid` / `tgid` | Identificador único numérico e identificador de grupo de hilos otorgado por el kernel al lanzar la tarea. |
| **`PPid: 3763`** | `real_parent` | Identificador del proceso padre (*Parent PID*), correspondiente a la terminal o shell desde la que se invocó el comando. |
| **`Uid: 1000`** | `cred` | Credenciales de usuario real y efectivo que determinan los permisos de lectura, escritura y ejecución. |

---

## 5. Conclusión

El desarrollo de este laboratorio permite comprobar que un sistema operativo Linux funciona internamente como una gran base de datos relacional interconectada en memoria RAM. La estructura `struct task_struct` consolida en C todos los datos necesarios para administrar recursos, garantizando que el planificador pueda pausar, reanudar y aislar la ejecución de procesos sin perder el contexto del sistema.

```

```
