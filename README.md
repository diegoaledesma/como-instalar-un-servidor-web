# Cómo instalar un servidor web en la nube.

Este tutorial está realizado sobre una VM de Google cloud con Ubuntu 18.

### 1 - Permitir la conexión por ssh

Utilizamos Vim para ésta tarea.

'''
$ sudo vim /etc/ssh/sshd_config
'''

Dentro del archivo cambiamos la siguiente línea

'''
PasswordAuthentication no
'''

por

'''
PasswordAuthentication yes
'''

Guardamos los cambios y reiniciamos el servicio de ssh con el siguiente comando

'''
$ sudo systemctl restart ssh.service
'''

### 2 - Creación de un usuario en la VM 

Debemos crear un usuario en la VM para poder conectarnos por ssh.

'''
$ sudo adduser dev
'''

Luego agregamos el usuario al grupo sudo

'''
$ sudo gpasswd -a dev sudo
'''

### 3 - Copiar nuestra key de ssh en la VM

En nuestro equipo generamos la key de ssh que vamos a utilizar para conectarnos a la VM sin la necesidad de ingresar nuestro password cada vez que vayamos a conectarnos.

'''
$ ssh-keygen -t rsa -b 4096
''' 

Una vez generada la copiamos en la VM con el siguiente comando.

'''
$ ssh-copy-id dev@IP_ADDRESS
'''

Antes de responder a la pregunta Are you sure you want to continue connecting (yes/no)? podemos ejecutar el siguiente comando en la VM para verificar que el fingerprint es el correcto.

'''
$ ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
'''

Una vez realizado ésto ya estamos en condiciones de acceder a la VM a traves de ssh sin la necesidad de ingresar el passwd.

### 4 - Abrimos tmux

En una terminal de nuestro equipo abrimos tmux para poder dividir la terminal y poder ejecutar distinto comandos al mismo tiempo.

'''
$ tmux
'''

En el caso de no tenerlo instalado ejecutamos lo siguiente.

'''
$ sudo apt install -y tmux
'''

