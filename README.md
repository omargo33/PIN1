# Grupo 13
## Ejercicio práctico de Jenkins

Ejecutar proyecto de Jenkins con Docker y levantar flujo de trabajo de CI/CD desde un repositorio de GitHub.

## Requisitos
- GitHub
    - Cuenta de GitHub
    - Fork
- Docker
- Docker-compose

## Instrucciones

### GitHub - Cuenta
Se debe crear una cuenta en [github.com](https://www.github.com) de la manera habitual que se crea una cuenta en cualquier sitio web.

### GitHub Fork repositorio
Fork el repositorio de GitHub en su cuenta personal. (Fork -> Bifurcación) desde el siguiente [link](https://github.com/EducacionMundose/PIN1)

### Docker y Docker-compose
Para instalar docker en Ubuntu

• Actualizar el sistema operativo
```bash
    sudo apt-get update && upgrade
```

• Instalar servidor docker-ce y docker-compose
```bash
    sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

• Comprobamos el estado de docker
```bash
    sudo systemctl status docker
```

• Para validar que está correctamente instalado se ejecuta:
```bash
    docker --version; 
    docker compose version;
```

## Lanzar Jenkins
Para lanzar Jenkins se debe ejecutar el siguiente comando:
```bash
    docker run -dit -p 8080:8080 --network=host -v /var/run/docker.sock:/var/run/docker.sock \
    --name jenkins-curso docker.io/mguazzardo/pipe-seg
```
Descargando
![Docker Download Jenkings](/imagenes/download.png)

Lunch - Lanzado Jenkins
Usuario y contraseña: admin
URL: http://localhost:8080
![Docker Download Jenkings](/imagenes/jenkings.png)

# Prácticas de Jenkins

## Práctica 1

Se desea realizar un flujo de trabajo de CI/CD con Jenkins, para ello se debe realizar lo siguiente:

### Crear un nuevo proyecto en Jenkins

1. Seleccionar la opción de "Nueva tarea"
2. Poner nombre a la tarea
3. Seleccionar la opción de "Pipeline"
4. Agregas una descripción
5. Seleccionar la opción de "Advanced Project Options"
6. Tomar la opción de "Pipeline script from SCM" y seleccionar "Git" y agregar la dirección del git Fork a su usuario de GitHub, en mi caso es github.com/omargo33/PIN1
![Data](/imagenes/config.png) 
7. Guardar

### Primera ejecución

Volvemos a la pantalla principal de Jenkins y seleccionamos la tarea que acabamos de crear, en la parte izquierda seleccionamos la opción de "Build Now" y esperamos a que se ejecute el flujo de trabajo.

Y esta nos dará el siguiente resultado:
![Data](/imagenes/errorInicial.png)

Como la corrida fue errónea, se procede a revisar el log.

Sobre la tarea creada, seleccionamos en la columna de estado de ejecución, nube con lluvia, y seleccionamos la opción de "Console Output" para ver el log de la ejecución.

![Data](/imagenes/salidaConsola.png)

En la que vemos que la carpeta webapp no existe, por lo que se procede a retirarla del archivo Jenkinsfile del repositorio y se vuelve a ejecutar el flujo de trabajo.

### Segunda ejecución
En una segunda ejecución, se procede a ejecutar el flujo de trabajo y se obtiene el siguiente resultado:

![Data](/imagenes/error2.png)

Como se puede ver nos acaba de dar un error, en el que nos dice que no se puede ejecutar el comando de npm, por lo que se procede a revisar el log.

![Data](/imagenes/error3.png)

En este se describe que no se puede hacer push a un servidor local sobre el puerto 5000, por lo que se procede a ejecutar el comando docker para levantar dicho servidor.
 
### Tercera ejecución

En este momento se ve que hace falta levantar un nuevo contenedor docker con el puerto 5000 activos con el siguiente comando:

```bash    
    docker run --name testOv001  -p 5000:5000 registry:2
```
![Data](/imagenes/docker5000.png)

Y se realiza una nueva ejecución del flujo de trabajo, en la que se obtiene el siguiente resultado:

![Data](/imagenes/OkJenkings.png)

Y con eso se da por terminado el flujo de trabajo.
 
## Práctica 2

### Crear un nuevo proyecto en Jenkins

1. Seleccionar la opción de "Nueva tarea"
2. Poner nombre a la tarea
3. Seleccionar la opción de "Pipeline"
4. Agregas una descripción
5. Seleccionar la opción de "Advanced Project Options"
6. Tomar la opción de "Pipeline script from SCM" y seleccionar "Git" y agregar la dirección del git Fork a su usuario de GitHub, en mi caso es github.com/omargo33/PIN1
![Data](/imagenes/tarea2-01.png)
7. Se cambia el nombre del archivo Jenkinsfile.all
![Data](/imagenes/tarea2-02.png)
8. Guardar

### Primera ejecución

Volvemos a la pantalla principal de Jenkins y seleccionamos la tarea que acabamos de crear, en la parte izquierda seleccionamos la opción de "Build Now" y esperamos a que se ejecute el flujo de trabajo.

Y esta nos dará el siguiente resultado:
![Data](/imagenes/tarea2-03.png)

### Problemática

En este punto nos dimos cuenta de que el apartado "Pass To K8s" no se ejecutaba correctamente porque este hacía mención a un servidor de kubernets que no tenemos aún acceso o instalado por lo que se decidió instalar un servidor de pruebas en nuestro equipo local, y optamos por un servidor minikube. Ver más en [minikube](https://minikube.sigs.k8s.io/)

#### Instalación de Minikube

Para lo cual usamos los siguientes comandos: 

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

![Data](/imagenes/tarea2-05.png)

Una vez instalado lo iniciamos con el siguiente comando:

```bash
minikube start
```

Luego de esto lanzamos su dashboard con el siguiente comando:

```bash
minikube dashboard
```

Que nos permite visualizar nuestro cluster de kubernets. (de un unico nodo)

![Data](/imagenes/tarea2-04.png)


#### Cambios en la tarea k8s

El archivo original de la tarea k8s es el siguiente:

```bash
    stage('Pass To K8s'){

        steps {

        sh '''
        sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl create deployment testapp --image=127.0.0.1:5000/mguazzardo/testapp"
       echo "Wait"
       sleep 10
       sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl expose deployment testapp --port=3000"
       sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "wget https://raw.githubusercontent.com/tercemundo/platzi-scripts-integracion/master/webapp/nodePort.yml"
       sshpass -p 'master' ssh 172.17.0.1 -l root -o StrictHostKeyChecking=no "kubectl apply -f nodePort.yml" 

           '''

            }
        }
```
En la que se puede apreciar que se hace uso de un servidor de kubernets en la ip 172.17.0.1 con el usuario root, cuya clave de acceso es master y para interactuar con el mismo se usa los comandos de kubectl.


Mientras que para nuestra instancia local de minikube se uso el siguiente comando:

```bash
    minikube kubectl
```
Dando como resultado el siguiente archivo:

```bash
    stage('Pass To K8s'){

        steps {

        sh '''
	sshpass -p 12341234s ssh 172.18.5.35 -p 21212 -l colaborador -o StrictHostKeyChecking=no "minikube kubectl -- create deployment testapp --image=127.0.0.1:5000/mguazzardo/testapp"
       echo "Wait"
       sleep 10
       sshpass -p 12341234s ssh 172.18.5.35 -p 21212 -l colaborador -o StrictHostKeyChecking=no "minikube kubectl -- expose deployment testapp --port=3000"
       sshpass -p 12341234s ssh 172.18.5.35 -p 21212 -l colaborador -o StrictHostKeyChecking=no "wget https://raw.githubusercontent.com/tercemundo/platzi-scripts-integracion/master/webapp/nodePort.yml"
       sshpass -p 12341234s ssh 172.18.5.35 -p 21212 -l colaborador -o StrictHostKeyChecking=no "minikube kubectl -- apply -f nodePort.yml" 

           '''

            }
        }
```
En la que se puede apreciar que se hace uso de un servidor de kubernets en la ip 172.18.5.35 y que el puerto ssh es el 21212, con el usuario colaborador, cuya clave de acceso es 12341234s y para interactuar con el mismo se usa los comandos de minikube kubectl.

### Ejecución 18

Luego de el trabajo de configuración del servidor, revisión de servicio ssh y la configuración de la tarea k8s, se procede a ejecutar el flujo de trabajo y se obtiene el siguiente resultado:

![Data](/imagenes/tarea2-06.png)

, su correspondiente despliegue en el dashboard de minikube:

![Data](/imagenes/tarea2-07.png)

Y; el log de la ejecución:

![Data](/imagenes/tarea2-08.png)


### Integrantes
Manuel Tinajero   manuel93tc@hotmail.com
Sebastián Peña    penasantiago346@gmail.com
Omar Vélez        omargo33@gmail.com
Susy Robalino     susana.robalino@gmail.com
Joel Cittar       joelcittar@gmail.com   