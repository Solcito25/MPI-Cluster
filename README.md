# MPI-Cluster
###  Configuración de Virtual box
1.- Configurar en todas las máquinas el adaptador de red de la siguiente forma:

![Screenshot](imagen1.jfif)

2.- Configurar otro adaptador de red tipo NAT para tener acceso a internet:

![Screenshot](imagen2.jfif)

### Configuración en Ubuntu
1.- Configurar en todas las máquina direcciones IP estáticas en el menú de configuración de red:

![Screenshot](imagen3.jfif)

### Configuración  MPI
#### Paso 1 Instalar MPICH2 o OpenMPI
Instalar MPICH2 en todos los sistemas
<pre><code> sudo apt install -y mpich </code></pre>
O instalar OpenMPI 
<pre><code> sudo apt-get install libopenmpi-dev openmpi-bin openmpi-doc </code></pre>
#### Paso 2 Configurar el Host file
Aquí asignaremos las direcciones IP a los nombres de host para que no tengamos que escribir las direcciones IP una y otra vez.

*Master*:
<pre><code> sudo nano /etc/hosts </code></pre>
127.0.0.1 localhost
#MPI SETUP

100.96.100.1 master

100.96.100.2 client1

100.96.100.2 client2

100.96.100.2 client3

**Client1**:
<pre><code> sudo nano /etc/hosts </code></pre>
127.0.0.1 localhost

#MPI SETUP

100.96.100.1 master

100.96.100.2 client1

**Client2**:
<pre><code> sudo nano /etc/hosts </code></pre>
127.0.0.1 localhost

#MPI SETUP

100.96.100.1 master

100.96.100.2 client2

**Client3**:
<pre><code> sudo nano /etc/hosts </code></pre>
127.0.0.1 localhost

#MPI SETUP

100.96.100.1 master

100.96.100.2 client3


Guardar todos los archivos y salir

#### Paso 3 Crear un nuevo usuario 
Crear un nuevo usuario con el mismo nombre en todos los dispositivos
<pre><code> sudo adduser mpiuser </code></pre>
#### Paso 4 Configurar SSH
Los sistemas se comunicarán a través de SSH y compartirán datos a través de NFS 

Instalar OpenSSH en todos los sistemas
<pre><code> sudo apt install openssh-server </code></pre>
 **Cambiar el usuario**
 <pre><code> su -mpiuser </code></pre> 
 **#generacion de key**
 <pre><code> ssh-keygen -t rsa </code></pre> 
  **#crear un directorio .ssh en el cliente**
  <pre><code> ssh mpiuser@client1 mkdir -p .ssh </code></pre> 
  Escribir yes cuando lo solicite
  
  Escribir el password del client1
  
  **#Actualizar las claves publicas generadas al cliente**
  <pre><code> cat .ssh/id_rsa.pub | ssh mpiuser@client1 'cat >> .ssh/authorized_keys' </code></pre> 
  
  Ingresar el password del client1 cuando lo solicite
  
  **#Configurar el permiso en el cliente**
    <pre><code> cssh mpiuser@client1 "chmod 700 .ssh; chmod 640 .ssh/authorized_keys" </code></pre> 
    **#Iniciar sesión en el cliente sin contraseña**
    <pre><code> ssh mpiuser@client1 </code></pre> 
**Nota:** Hacer lo mismo para los otros clientes desde la master

**Nota:** Hacer lo mismo para la master desde todos los clientes

#### Paso 5 Configurar NFS
NFS se utiliza para compartir el archivo de objeto entre todos los sistemas y los datos compartibles.
*Master*:

Instalar el servidor NFS en la master para montar la carpeta compartida
**#NFS para la instalación del servidor**
<pre><code> sudo apt install nfs-kernel-server </code></pre> 

**#Crear una carpeta compartida**
<pre><code> mkdir storage </code></pre>
**#Crear una entrada en  /etc/exports**
<pre><code> cat /etc/exports
/home/mpiuser/storage *(rw,sync,no_root_squash,no_subtree_check) </code></pre>
**#Ejecutar el comando después de cualquier cambio en  /etc/exports**
<pre><code> exportfs -a </code></pre>
**#Reiniciar el servidor NFS**
<pre><code> sudo service nfs-kernel-server restart</code></pre>

*Client*:

**#Instalar NFS para el cliente**
<pre><code> sudo apt-get install nfs-common </code></pre>
**#Crear una carpeta compartible con el mismo nombre que la master**
<pre><code> mkdir storage </code></pre>
**#Montar la carpeta master en el cliente**
<pre><code> sudo mount -t nfs master:/home/mpiuser/storage ~/storage </code></pre>
**#Comprobar si el montaje es exitoso**
<pre><code> df -h </code></pre>
**#Reiniciar el sistema cliente**
**#Agregar la entrada a la tabla del sistema de archivos**
<pre><code> cat /etc/fstab </code></pre>

#MPI CLUSTER SETUP

master:/home/mpiuser/storage /home/mpiuser/storage nfs

#### Paso 6 Ejecutar el programa
Cambiar el directorio en el master node
<pre><code> cd storage/ </code></pre>
<pre><code> pwd
/home/mpiuser/storage </code></pre>
Crear un programa Hello World MPI en C con el nombre helloworld_MPI.c

**#Compilar el codigo**
<pre><code> mpicc helloworld_MPI.c </code></pre>

**#Ejecutar el codigo**
<pre><code> mpirun -np 4 -hosts master,client1,client2,client3 ./a.out </code></pre>
