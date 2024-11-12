### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

![image](https://github.com/user-attachments/assets/5c0ade05-feb4-4141-8d41-8677f8c01860)


7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000
  

![image](https://github.com/user-attachments/assets/bb217809-4f55-4c5a-9fb0-65584b74cc87)


![image](https://github.com/user-attachments/assets/b83ca1c5-8773-44bf-8620-fa9be2ac2587)


![image](https://github.com/user-attachments/assets/f3aaceb0-e004-4863-ade1-50346ddaf353)


8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)



![image](https://github.com/user-attachments/assets/4d91f747-83d8-477b-a41b-2a28b640f4d7)


9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

![image](https://github.com/user-attachments/assets/9cb2005b-5327-4c6f-b969-4dacd03f249a)

![image](https://github.com/user-attachments/assets/777f270b-a80f-4f3e-ab13-b859df79fa89)

![image](https://github.com/user-attachments/assets/56232fff-b240-49f7-9024-8d095db14af3)


![image](https://github.com/user-attachments/assets/83f2eaa2-39ec-48e1-a2be-3755c4c75b23)



10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

![image](https://github.com/user-attachments/assets/93ce5d9f-3e92-4cf7-8645-3c6fd2ea7cbe)

![image](https://github.com/user-attachments/assets/d1573a56-b15a-4f5b-bca0-812c4291a8b6)

![image](https://github.com/user-attachments/assets/991e7720-f03c-4263-9867-fc6626368b83)

![image](https://github.com/user-attachments/assets/8d950313-d82e-4a65-9aad-586ad0374f1c)

![image](https://github.com/user-attachments/assets/87eb46f4-9ab8-473e-822a-eb1c2796ab61)

![image](https://github.com/user-attachments/assets/f162f987-5df4-4f60-b798-e4fa165671da)

![image](https://github.com/user-attachments/assets/5794ab0c-8e34-4039-a07b-d9d07daac8cf)


13. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

Si se evidencia una reducción significativa del uso de recursos luego de realizar el escalamiento.
   
15. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

Virtual Machine, Resource Group, Public and Private IP address.

2. ¿Brevemente describa para qué sirve cada recurso?

Virtual Machine (VM)
Una máquina virtual es un entorno de computación simulado que actúa como un ordenador físico, permitiendo ejecutar aplicaciones y sistemas operativos. En Azure, se usa para alojar aplicaciones y servicios con recursos configurables como CPU, RAM y almacenamiento.

Resource Group
Un resource group en Azure es un contenedor lógico que organiza y administra recursos relacionados, como máquinas virtuales, redes y bases de datos. Permite manejar y monitorear recursos como una unidad cohesiva.

Public IP Address
Una dirección IP pública permite que recursos en la nube sean accesibles desde Internet. Es esencial para servicios que requieren conexión externa, como APIs o aplicaciones web.

Private IP Address
Una dirección IP privada es utilizada para la comunicación interna dentro de una red virtual. Estos recursos no son accesibles desde Internet, ofreciendo mayor seguridad para sistemas internos.

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

Cuando se ejecuta una aplicación con npm FibonacciApp.js dentro de la conexión SSH, esta queda asociada a la sesión del terminal activo. Al cerrar la conexión SSH:

El proceso se termina automáticamente porque el terminal que lo inició deja de existir.
Esto ocurre porque no se está ejecutando el proceso en segundo plano ni usando una herramienta que lo desacople de la sesión SSH. Para esto es necesario el forever

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.


La función presenta un tiempo de ejecución significativo, y este tiende a incrementarse progresivamente a medida que se realizan más iteraciones. Esto se debe a que cada iteración agrega un conjunto adicional de operaciones, lo que acumula más procesamiento en cada ciclo. Por lo tanto, el desempeño de la función está directamente influenciado por el número de iteraciones, haciendo que el tiempo total de ejecución aumente de manera proporcional o incluso exponencial dependiendo de la complejidad de las operaciones internas. Este comportamiento sugiere que el diseño y la optimización de la función son críticos, especialmente cuando se trabaja con un alto volumen de iteraciones, para garantizar un desempeño más eficiente y consistente


5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

![image](https://github.com/user-attachments/assets/29494bcf-9a14-4dc5-aa4a-74ce32a2ad15)

Vemos una reducción entre las ejecuciones de diferentes tamaños en la parte derecha mayor el uso de la cpu.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.

![image](https://github.com/user-attachments/assets/eb57aaea-1ea6-44c9-9ffc-35d6e07338f3)

![image](https://github.com/user-attachments/assets/cc821bca-d427-419b-b54e-f5a404c4d09a)

La segunda prueba muestra un mejor desempeño, con una reducción del tiempo promedio de respuesta de aproximadamente 1.8 segundos (de 9.3s a 7.5s).

Ambas pruebas muestran una desviación estándar baja (126ms en la primera y 83ms en la segunda), lo que indica que los tiempos de respuesta son consistentes en cada prueba.

No se presentan fallos.

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

1. Recursos Asignados:

B1ls:

vCPU: 1
Memoria: 0.5 GiB
Almacenamiento Temporal: 4 GiB
Costo Aproximado: $3.80/mes 

B2ms:

vCPU: 2
Memoria: 8 GiB
Almacenamiento Temporal: 16 GiB
Costo Aproximado: $58.40/mes 

2. Casos de Uso:

B1ls:

Ideal para tareas muy ligeras, como servidores de desarrollo, pruebas básicas o aplicaciones con requerimientos mínimos de recursos.
Adecuada para procesos que no demandan alto rendimiento y pueden tolerar latencias.
B2ms:

Apta para aplicaciones web pequeñas, servidores de bases de datos ligeros o entornos de desarrollo que requieren más memoria y capacidad de procesamiento.
Mejor opción para cargas de trabajo que, aunque no son intensivas, necesitan más recursos que los ofrecidos por B1ls.
3. Rendimiento y Flexibilidad:

B1ls:

Ofrece el costo más bajo entre las VM de Azure, pero con recursos muy limitados.
Es adecuada para aplicaciones donde el costo es una prioridad y el rendimiento no es crítico.
B2ms:

Proporciona un equilibrio entre costo y rendimiento, ofreciendo más flexibilidad para manejar cargas de trabajo variables.
Permite acumular y utilizar créditos de CPU de manera más eficiente, beneficiando aplicaciones con patrones de uso intermitente.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

Se observó que el tiempo de respuesta se redujo al incrementar la capacidad, aunque los resultados no mostraron una mejora significativa. Por esta razón, el incremento en capacidad no resulta justificado desde la perspectiva de costo-beneficio. A pesar del ajuste, la aplicación continúa demandando un alto nivel de procesamiento.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

El precio aumenta en gran medida.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

Si porque los tiempos se redujeron y la cpu tuvo menos uso en el segundo intento.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)
Realizado:

![image](https://github.com/user-attachments/assets/e70ef3ac-20f7-44f8-a280-dfe5a850018e)
![image](https://github.com/user-attachments/assets/a58433f9-2a9e-442a-9e26-60f390b5ab2b)


2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




