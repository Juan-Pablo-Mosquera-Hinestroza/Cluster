# Implementación de un Clúster de Conmutación por Error en Windows Server 2025 con SQL Server 2022

## Introducción

Los sistemas informáticos empresariales modernos exigen altos niveles de disponibilidad y fiabilidad, especialmente en servicios críticos como bases de datos. La interrupción de estos servicios puede causar pérdidas económicas, daño reputacional y baja productividad. En este contexto, los clusters de conmutación por error se han vuelto esenciales para infraestructuras de TI robustas.

Windows Server 2022 ofrece capacidades avanzadas para implementar clusters de conmutación por error, permitiendo que los servicios críticos sigan funcionando incluso si uno de los nodos falla. Este trabajo explora la implementación práctica de esta tecnología para garantizar la alta disponibilidad de servicios SQL Server, mostrando cómo configurar, administrar y mantener un entorno de cluster eficiente.

La relevancia de este estudio radica en proporcionar una guía aplicada para profesionales de TI que buscan fortalecer la resiliencia de sus infraestructuras de servidor y bases de datos, contribuyendo así a la continuidad del negocio en entornos donde el tiempo de inactividad no es una opción viable.

## Objetivos

**Objetivo General:**

Implementar y configurar un cluster de conmutación por error en Windows Server 2022 para garantizar la alta disponibilidad y tolerancia a fallos de servicios críticos de bases de datos SQL Server.

**Objetivos Específicos:**

1. Analizar los conceptos fundamentales y componentes de un cluster de conmutación por error en entornos Windows Server.
2. Configurar el entorno de laboratorio con dos servidores físicos y dos máquinas virtuales, estableciendo el almacenamiento compartido necesario para el funcionamiento del cluster.
3. Implementar la función de conmutación por error de Windows Server y crear la estructura del cluster con su configuración de quorum apropiada.
4. Instalar y configurar SQL Server en las máquinas virtuales del cluster para habilitar la conmutación por error de las bases de datos.
5. Realizar pruebas de conmutación por error para verificar la funcionalidad y eficacia del cluster implementado.
6. Establecer protocolos de monitoreo y resolución de problemas para garantizar el mantenimiento efectivo del cluster en un entorno de producción.

## 1. Preparación del Entorno

### 1.1 Creación de la primera máquina virtual (Directorio Activo)

1. **Crear la máquina virtual:** Será el Directorio Activo donde se conectarán las otras dos máquinas virtuales.
2. **Cambiar el nombre y asignar IP estática:**
   - Cambia el nombre de la máquina y asigna una IP estática.
   - ![Nombre de AD](Imagenes/Samuel/nombre_de_AD.png)
   - ![IP estática máquina 1](Imagenes/Samuel/Ip_estatica_maquina_1.png)

3. **Agregar roles y características:**
   - Ve a "Agregar roles y características".
   - ![Roles y características](Imagenes/Samuel/Roles_y_caracteristicas.png)
   - Selecciona "Servicios de Dominios de Active Directory".
   - ![Roles del servidor](Imagenes/Samuel/Roles_del_servidor.png)

4. **Instalación y configuración del dominio:**
   - Espera la confirmación de la instalación.
   - ![Proceso de instalación](Imagenes/Samuel/5._proceso_de_instalacion.png)
   - Configura el dominio desde el icono de la bandera.
   - ![Configuraciones del Dominio](Imagenes/Samuel/Ilustracion_7_Configuraciones_del_Dominio.png)
   - ![Propiedades del servidor](Imagenes/Samuel/Ilustracion_8_Propiedades_del_servidor.png)
   - Revisa las propiedades del servidor.
   - ![Disco ISCSI](Imagenes/Samuel/Ilustracion_9_Disco_ISCSI.png)

### 1.2 Configuración del almacenamiento compartido (ISCSI)

1. **Crear almacenamiento ISCSI:**
   - Apaga la máquina, ve a VirtualBox > Configuración > Almacenamiento.
   - Selecciona el controlador LSIlogic y crea un nuevo disco duro.

2. **Inicializar y preparar el disco:**
   - Ve a "Administrador de equipos" > "Almacenamiento".
   - Inicializa el disco con la opción GPT.
   - ![Herramientas del servidor](Imagenes/Samuel/Ilustracion_10_Herramientas_del_servidor.png)
   - ![ISCSI 1](Imagenes/Samuel/Ilustracion_11_ISCSI_1.png)
   - ![ISCSI 2](Imagenes/Samuel/Ilustracion_12_ISCSI_2.png)
   - ![ISCSI 3](Imagenes/Samuel/Ilustracion_13_ISCSI_3.png)

3. **Crear volumen simple:**
   - Asigna la letra E y el nombre "Sharing".
   - ![ISCSI 4](Imagenes/Samuel/Ilustracion_14_ISCSI_4.png)
   - ![ISCSI 5](Imagenes/Samuel/Ilustracion_15_ISCSI_5.png)
   - ![ISCSI 6](Imagenes/Samuel/Ilustracion_16_ISCSI_6.png)
   - ![ISCSI 7](Imagenes/Samuel/Ilustracion_17_ISCSI_7.png)

### 1.3 Configuración de las máquinas virtuales clientes

#### Segunda máquina virtual

1. Cambia el nombre y asigna IP estática.
2. Configura el DNS con la IP del Directorio Activo.
3. Une la máquina al dominio principal.
   - ![IP máquina 2](Imagenes/Samuel/Ilustracion_18_Ip_maquina2.png)
   - ![Nombre y Dominio máquina 2](Imagenes/Samuel/Ilustracion_19_Nombre_y_Dominio_maquina_2.png)
   - ![Usuario y contraseña](Imagenes/Samuel/Ilustracion_20_Usuario_y_contrasena_de_la_cuenta_del_DC.png)

#### Tercera máquina virtual

1. Cambia el nombre y asigna IP estática.
2. Configura el DNS con la IP del Directorio Activo.
3. Une la máquina al dominio principal.
   - ![IP estática máquina 3](Imagenes/Samuel/Ilustracion_21_IP_estatica_maquina_3.png)
   - ![Nombre y Dominio máquina 3](Imagenes/Samuel/Ilustracion_22_Nombre_y_Dominio_maquina_3.png)
   - ![Dominio máquina 3](Imagenes/Samuel/Ilustracion_23_entra_al_Dominio_la_maquina_3.png)
   - ![Nuevo rol](Imagenes/Samuel/Ilustracion_24_Nuevo_rol.png)

### 1.4 Configuración del rol ISCSI y discos compartidos

1. **Desplegar el rol ISCSI:**
   - Instala el rol "Servidor de Destino".
   - ![Opciones del rol](Imagenes/Samuel/Ilustracion_25_Opciones_del_rol.png)
   - ![Proceso de instalación](Imagenes/Samuel/Ilustracion_26_Proceso_de_instalacion.png)

2. **Crear y asignar destino ISCSI:**
   - Selecciona la partición creada y asígnale un tamaño de 1GB.
   - ![Nuevo Destino](Imagenes/Samuel/Ilustracion_27_Nuevo_Destino.png)
   - ![IQN](Imagenes/Samuel/Ilustracion_31_IQN.png)

3. **Agregar destino ISCSI a las máquinas:**
   - ![Sharing (E)](Imagenes/Samuel/Ilustracion_28_Sharing__E_.png)
   - ![Nombre del disco](Imagenes/Samuel/Ilustracion_29_Nombre_del_disco.png)
   - ![Tamaño del disco](Imagenes/Samuel/Ilustracion_30_Tamano_del_disco.png)
   - Añade los nombres de las máquinas para obtener el IQN.
   - ![Resultados](Imagenes/Samuel/Ilustracion_32_Resultados.png)
   - ![Inicializador ISCSI](Imagenes/Samuel/Ilustracion_33_Inicializador_ISCSI.png)

4. **Inicializar ISCSI en los clientes:**
   - Abre "Herramientas de Windows" > "Inicializador ISCSI".
   - ![ISCSI de Microsoft](Imagenes/Samuel/Ilustracion_34_ISCSI_de_Microsoft.png)
   - En destino, pon la IP de la máquina principal y autoconfigura.
   - ![Iniciador ISCSI](Imagenes/Samuel/Ilustracion_35_Iniciador_ISCSI.png)
   - ![Autoconfigurar](Imagenes/Samuel/Ilustracion_36_Autoconfigurar.png)
   - Inicializa y asigna letra y nombre al volumen.
   - ![Espacio ISCSI](Imagenes/Samuel/Ilustracion_37_Espacio_ISCSI.png)
   - ![Nombre del volumen simple](Imagenes/Samuel/Ilustracion_38_Nombre_del_volumen_simple.png)

5. **Repetir proceso en la segunda máquina cliente.**

6. **Crear un nuevo disco virtual ISCSI:**
   - Repite el proceso, asigna nombre y tamaño (23GB).
   - ![Nuevo Disco](Imagenes/Samuel/Ilustracion_39_Nuevo_Disco.png)
   - ![Asignación de disco](Imagenes/Samuel/Ilustracion_40_Asignacion_de_disco.png)
   - ![Nombre del disco](Imagenes/Samuel/Ilustracion_41_Nombre_del_disco.png)
   - ![Espacio del disco](Imagenes/Samuel/Ilustracion_42_Espacio_del_disco.png)
   - ![Asignar destino](Imagenes/Samuel/Ilustracion_43_Asignar_destino.png)
   - ![Propiedades del disco Datos](Imagenes/Samuel/Ilustracion_44_Propiedades_del_disco_Datos.png)
   - ![Agregar Rol nuevo](Imagenes/Samuel/Ilustracion_45_Agregar_Rol_nuevo.png)
   - ![Rol del cluster](Imagenes/Samuel/Ilustracion_46_Rol_del_cluster.png)
   - ![Características del rol](Imagenes/Samuel/Ilustracion_47_Caracteristicas_del_rol.png)

7. **Inicializar y montar el disco en los clientes:**
   - Repite el proceso de inicialización y asignación de nombre "Datos".

### 1.5 Instalación del rol de clúster y despliegue

1. **Instalar el rol de clúster:**
   - En una de las máquinas, instala el rol "Cluster de Conmutación por Error".
   - ![Proceso de instalación](Imagenes/Samuel/Ilustracion_48_Proceso_de_instalacion.png)
   - ![Gui del cluster](Imagenes/Samuel/Ilustracion_49_Gui_del_cluster.png)
   - ![Despliegue del Cluster](Imagenes/Samuel/Ilustracion_50_Despliegue_del_Cluster.png)
   - ![Nodos del Cluster](Imagenes/Samuel/Ilustracion_51_Nodos_del_Cluster.png)

2. **Crear el clúster:**
   - Abre el Administrador de clústeres de conmutación por error.
   - Selecciona "Crear clúster" y sigue el asistente:
     - Selecciona los nodos.
     - Valida la configuración.
     - Asigna nombre y dirección IP al clúster.
     - Configura el quorum (elige el disco testigo).

> **Nota:** Antes de crear el clúster, asegúrate de:
>
> - Tener al menos dos nodos unidos al mismo dominio.
> - Haber configurado almacenamiento compartido (iSCSI).
> - Contar con una red confiable entre los nodos.
> - Tener habilitada la característica "Clúster de conmutación por error".
---

## Configuración avanzada del Quorum y adaptadores de red

### Configuración del Quorum en el clúster

1. En el clúster, accede a las opciones adicionales en la parte izquierda del administrador y elige la opción "Configurar los ajustes del Quorum del clúster".
   - ![Configuración del Quorum - Paso 1](Imagenes/JuanPablo/image1.png)
2. Elige la configuración avanzada de Quorum.
   - ![Configuración avanzada de Quorum](Imagenes/JuanPablo/image2.png)
3. Asigna los nodos que van a votar en el clúster.
   - ![Asignación de nodos votantes](Imagenes/JuanPablo/image3.png)
4. Elige la opción de configurar un testigo de disco.
   - ![Configuración de testigo de disco](Imagenes/JuanPablo/image4.png)
5. Selecciona el disco Quorum (en este caso, el disco de clúster 1).
   - ![Selección de disco Quorum](Imagenes/JuanPablo/image5.png)
6. En la siguiente pantalla, solo confirma y haz clic en "Next". Ya habrás configurado el Quorum de forma satisfactoria. Para finalizar, haz clic en "Finalizar".
   - ![Finalización de la configuración del Quorum](Imagenes/JuanPablo/image6.png)

---

## Instalación de SQL Server 2022 y SQL Server Management Studio (SSMS)

Instalamos SQL Server 2022 como una instancia del clúster seleccionando "New SQL Server failover Cluster installation" en el instalador de SQL Server.
   - ![Instalador SQL Server - Paso 1](Imagenes/Canizales/image1.png)
   - ![Instalador SQL Server - Paso 2](Imagenes/Canizales/image2.png)
   - ![Instalador SQL Server - Paso 3](Imagenes/Canizales/image3.png)
   - ![Instalador SQL Server - Paso 4](Imagenes/Canizales/image4.png)
   - ![Instalador SQL Server - Paso 5](Imagenes/Canizales/image5.png)
   - ![Instalador SQL Server - Paso 6](Imagenes/Canizales/image6.png)
   - ![Instalador SQL Server - Paso 7](Imagenes/Canizales/image7.png)
   - ![Instalador SQL Server - Paso 8](Imagenes/Canizales/image8.png)
   - ![Instalador SQL Server - Paso 9](Imagenes/Canizales/image9.png)
   - ![Instalador SQL Server - Paso 10](Imagenes/Canizales/image10.png)
   - ![Instalador SQL Server - Paso 11](Imagenes/Canizales/image11.png)
   - ![Instalador SQL Server - Paso 12](Imagenes/Canizales/image12.png)

Damos permisos a todos para actuar en el almacenamiento compartido que se usó en el SQL Server 2022.
   - ![Permisos almacenamiento compartido](Imagenes/Canizales/image13.png)

Descargamos e instalamos SQL Server Management Studio (SSMS) en los nodos.
   - ![Instalación de SSMS](Imagenes/Canizales/image14.png)

Nos conectamos al clúster con SQL Server Management Studio (SSMS).
   - ![Conexión al clúster con SSMS](Imagenes/Canizales/image15.png)
   - ![Conexión exitosa al clúster](Imagenes/Canizales/image16.png)

Le damos permisos a los usuarios desde SQL Server Management Studio (SSMS).
   - ![Permisos a usuarios en SSMS](Imagenes/Canizales/image17.png)

---

### Configuración de adaptadores de red y creación de la base de datos

1. Crea otro adaptador de red en VirtualBox:
   - ![Creación de adaptador de red](Imagenes/JuanPablo/image7.png)
2. Asigna una IP estática al nuevo adaptador:
   - ![Asignación de IP estática](Imagenes/JuanPablo/image8.png)
3. Verifica la configuración en el clúster:
   - ![Verificación en el clúster](Imagenes/JuanPablo/image9.png)
4. Crea la base de datos Escuela_Futbol:
   - ![Creación de la base de datos Escuela_Futbol](Imagenes/JuanPablo/image10.png)
5. En el otro nodo, realiza la instalación "Add node to a SQL Server failover cluster":
   - ![Instalación Add node to a SQL Server failover cluster](Imagenes/JuanPablo/image11.png)
6. Proceso de instalación en el nodo:
   - ![Instalación nodo - paso 1](Imagenes/JuanPablo/image12.png)
   - ![Instalación nodo - paso 2](Imagenes/JuanPablo/image13.png)
   - ![Instalación nodo - paso 3](Imagenes/JuanPablo/image14.png)
   - ![Instalación nodo - paso 4](Imagenes/JuanPablo/image15.png)

---

Como se ve, se puede mover el rol del SQLCLUSTERSERVER entre los nodos sin problemas, por lo que si uno falla el rol puede moverse al otro.
   - ![Mover rol SQLCLUSTERSERVER - paso 1](Imagenes/JuanPablo/image13.png)
   - ![Mover rol SQLCLUSTERSERVER - paso 2](Imagenes/JuanPablo/image14.png)
   - ![Mover rol SQLCLUSTERSERVER - paso 3](Imagenes/JuanPablo/image15.png)

---
## Conclusión

La implementación del clúster de conmutación por error en Windows Server 2025 con SQL Server 2022 permitió garantizar la alta disponibilidad y la tolerancia a fallos de los servicios críticos de bases de datos. Se logró configurar correctamente el almacenamiento compartido, el Quorum, la instalación de SQL Server en modo clúster, la gestión de usuarios y la creación de la base de datos, así como la validación de la conmutación de roles entre nodos. El sistema resultante asegura la continuidad operativa y minimiza el tiempo de inactividad ante fallos, cumpliendo con los objetivos de resiliencia y robustez requeridos en entornos empresariales.

---

**Autores:**

- Samuel David Montenegro Gómez
- Juan Pablo Mosquera Hinestroza
- Juan David Vidal Canizales

**Institución Universitaria Antonio José Camacho**
Facultad de Ingenierías, Ingeniería en Sistemas, 2025
