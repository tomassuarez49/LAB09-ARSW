### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Autores: Camilo Nicolas Murcia Espinosa y Tomas Suarez Piratova

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

Realizado:
![image](https://github.com/user-attachments/assets/9afa7867-ba94-4ee3-9328-426d07363f98)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)
Realizado:
![image](https://github.com/user-attachments/assets/88407bb4-eed7-43be-a94f-087c4d8d7f44)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)
Realizado:
![image](https://github.com/user-attachments/assets/ed89dec4-cae0-416c-a529-8a4aaae42546)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)
Realizado:
![image](https://github.com/user-attachments/assets/cce402a3-ee1e-4ce5-a06d-684a6763c163)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)
Realizado:
![image](https://github.com/user-attachments/assets/6e140b8f-e1a2-4dc9-b4cf-be6c6df363bf)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)
Realizado: 
![image](https://github.com/user-attachments/assets/bf6e5cfa-22fd-440b-8169-0f57f1ea15d5)


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

![Hello World](https://github.com/user-attachments/assets/a3f4b575-40af-4c0c-b000-30fabc6fdffa)

![Fibonacci1](https://github.com/user-attachments/assets/26f8c823-0336-4ff1-9307-891567f11058)


2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

![newman1](https://github.com/user-attachments/assets/dad3086b-7a83-44ca-a278-b61c4c125476)

![vm1](https://github.com/user-attachments/assets/5effc068-14d0-41fe-8b1f-556aaa376356)


![newman2](https://github.com/user-attachments/assets/01e3300e-824c-45be-b7b4-775d95c61447)

![vm2](https://github.com/user-attachments/assets/b6d2c0c0-8848-4ca8-82eb-a23063e73b45)


3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

Debido a que se ha alcanzado el límite máximo de recursos asignados a esta suscripción, no es posible añadir una cuarta máquina virtual.
Al incrementar el número de máquinas virtuales, se duplica la potencia de procesamiento (CPU) y la capacidad de almacenamiento temporal (RAM), lo que resulta en un rendimiento significativamente mayor y, en consecuencia, en una tasa de éxito más elevada al procesar las solicitudes.

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

   Existen tres tipos de balanceadores de carga que se diferencian por la capa de red en la que operan:

      - Balanceador de Carga de Nivel de Aplicación (Application Gateway): Funciona en la capa 7 del modelo OSI (capa de aplicación), donde dirige el tráfico en función de detalles específicos de la aplicación, como URL o encabezados HTTP. También incluye medidas de seguridad avanzadas, como cifrado SSL y protección contra ataques DDoS.
      
      - Balanceador de Carga de Nivel de Transporte (Load Balancer): Trabaja en la capa 4 del modelo OSI (capa de transporte) y distribuye equitativamente el tráfico entre los servidores de back-end según IP, puerto y protocolo.
      
      - Gateway VPN: Sirve para establecer una conexión segura a través de VPN entre una red virtual de Azure y una red local, permitiendo el flujo de tráfico seguro entre ambas redes para facilitar la integración de servicios locales y en la nube.

Los SKU (Stock Keeping Units) son identificadores únicos de recursos en Azure que permiten definir sus características, capacidades y precios. Estos SKU permiten elegir la opción más adecuada según capacidad, rendimiento y disponibilidad. Algunos tipos son:

      - SKU de Máquina Virtual: Define características como núcleos de CPU, RAM, almacenamiento y rendimiento de red, con opciones para uso general, optimización de cómputo, memoria y almacenamiento.
      - SKU de Base de Datos: Determina el tamaño de almacenamiento, rendimiento y disponibilidad de bases de datos, con opciones optimizadas para uso general, memoria y alta disponibilidad.
      - SKU de Almacenamiento: Especifica características como capacidad de almacenamiento, rendimiento y durabilidad, con categorías optimizadas para uso general, rendimiento o archivos.
      - SKU de Servicio de Red: Define aspectos como ancho de banda, disponibilidad y seguridad, con opciones para aplicaciones web y empresariales.

Para permitir que los clientes accedan a recursos de Azure detrás de un balanceador de carga, es fundamental asignarle una dirección IP pública. Esta dirección sirve como punto de entrada para dirigir el tráfico entrante a los recursos del conjunto de escalado. Sin una IP pública, no habría una ruta para el tráfico externo, impidiendo el acceso a los servicios en la nube. En resumen, la IP pública en el balanceador es esencial para habilitar la conectividad y acceso adecuado a los servicios.
  
* ¿Cuál es el propósito del *Backend Pool*?

   El Backend Pool permite que el balanceador de carga redirija el tráfico entrante hacia los recursos correctos que están detrás de él. Dentro de este conjunto, se incluyen recursos como máquinas virtuales o instancias de contenedor configuradas en el grupo de escalado. El balanceador de carga aplica algoritmos de enrutamiento de tráfico, como Round Robin o Hash de IP, para distribuir equitativamente el tráfico hacia los recursos de destino en el Backend Pool. Este mecanismo garantiza una distribución eficiente y balanceada de la carga entre los recursos disponibles.
  
* ¿Cuál es el propósito del *Health Probe*?

   El Health Probe es una funcionalidad esencial del balanceador de carga que permite monitorear y verificar el estado de los recursos en el Backend Pool. Su objetivo principal es asegurarse de que solo los recursos disponibles y en buen funcionamiento reciban el tráfico entrante.

   Para lograr esto, el Health Probe envía periódicamente solicitudes, como pings, a los recursos del Backend Pool para comprobar su disponibilidad y respuesta adecuada. Estas solicitudes pueden ser de distintos tipos, como HTTP o TCP. Si el recurso responde correctamente, se considera saludable y sigue disponible para recibir tráfico. Si, en cambio, no responde o muestra un error, se marca como no saludable y se excluye de la lista de recursos disponibles. De esta manera, el Health Probe asegura que el balanceador de carga dirija el tráfico solo hacia los recursos que están en buen estado, garantizando la eficiencia y confiabilidad en la distribución del tráfico.
  
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

   La Load Balancing Rule (Regla de Balanceo de Carga) establece cómo el balanceador de carga debe direccionar el tráfico entrante hacia los recursos configurados en el Backend Pool. Estas reglas definen los criterios y condiciones para distribuir la carga entre los recursos disponibles, e incluyen detalles como el puerto de destino, el protocolo empleado y el algoritmo de balanceo de carga a utilizar.

La persistencia de sesión es clave para mantener la coherencia en las interacciones de los usuarios, incluso cuando ocurren interrupciones o cambios en la infraestructura subyacente. Dos enfoques comunes para lograr esta persistencia son:
   - Sesión basada en cookies:
   La información de sesión se almacena en una cookie enviada al cliente y recuperada en solicitudes posteriores.
   El balanceador de carga de Azure puede configurarse para distribuir el tráfico según la cookie de sesión.
   Asegura que las solicitudes posteriores del mismo usuario se dirijan siempre al mismo servidor, manteniendo la continuidad de la sesión.
   - Sesión basada en IP:
   La información de sesión se almacena en un servidor de sesión dedicado y se asocia con la dirección IP del cliente.
   El balanceador de carga de Azure puede configurarse para distribuir el tráfico según la dirección IP del cliente.
   Garantiza que las solicitudes posteriores del mismo cliente se dirijan siempre al mismo servidor de sesión, proporcionando persistencia. La persistencia de sesión es crucial en aplicaciones web o móviles que requieren autenticación del usuario o retienen información significativa del usuario, como el carrito de compras en un sitio de comercio electrónico. Al garantizar que las interacciones del usuario se mantengan con coherencia a lo largo del tiempo, se mejora la experiencia del usuario y se evitan problemas asociados con la pérdida de datos de sesión.
  
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

   Una Virtual Network (VNet) es un servicio que permite crear una red virtual aislada en la nube. Al configurar una VNet, se puede asignar un rango propio de direcciones IP, establecer subredes, reglas de seguridad y puertas de enlace para conectarse con otras redes, ya sea en Internet o en redes locales.

   Una Subnet es una subdivisión de una VNet que permite segmentar la red en partes más pequeñas. Cada Subnet tiene su propio rango de direcciones IP dentro del espacio de direcciones IP de la VNet y puede contar con sus propias reglas de seguridad y puertas de enlace, facilitando un mayor control y segmentación de la red.
   
   El Address Space es el rango de direcciones IP privadas disponibles para una VNet. Al crear una VNet, se define este espacio de direcciones, que estará disponible para la VNet y sus subredes.
   
   El Address Range es el rango de direcciones IP asignado a una Subnet dentro de la VNet. Al configurar una Subnet, se especifica un Address Range dentro del Address Space de la VNet, y los recursos implementados en esa Subnet recibirán direcciones IP de este rango.
  
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

   Una Availability Zone (Zona de Disponibilidad) en Azure es un grupo de centros de datos interconectados dentro de una misma región, donde cada zona está ubicada en un sitio físico independiente y funciona de forma autónoma, manteniéndose aislada de las fallas en otras zonas de la región. Esto ofrece mayor disponibilidad y resiliencia a las aplicaciones alojadas en Azure.

   La distribución de los recursos de una aplicación en tres zonas de disponibilidad distintas permite mejorar su disponibilidad y resiliencia, ya que asegura que la aplicación pueda seguir operando incluso si una o dos zonas fallan. Además, esta estrategia optimiza el rendimiento al distribuir el tráfico entre las zonas.
   
   La IP zone-redundant es una dirección IP pública que se puede asignar a un recurso de Azure, como una máquina virtual o un balanceador de carga, y está disponible en todas las zonas de disponibilidad de una región. Si ocurre una falla en una zona, esta dirección IP sigue accesible desde las demás zonas, garantizando así la continuidad de la aplicación en situaciones adversas.
  
* ¿Cuál es el propósito del *Network Security Group*?

   El Network Security Group (NSG) proporciona una capa adicional de seguridad en la red virtual, funcionando como un conjunto de reglas de filtrado de tráfico. Estas reglas definen qué tráfico puede entrar o salir de la red virtual, basándose en factores como la dirección IP de origen y destino, los puertos de origen y destino, el protocolo utilizado y otros criterios relevantes.   

* Informe de newman 1 (Punto 2)

   ![image](https://github.com/user-attachments/assets/bacc338a-0007-4eee-9cf9-ea1c76465b22)

* Presente el Diagrama de Despliegue de la solución.

![image](https://github.com/user-attachments/assets/13f1112f-3a58-462f-a2ef-29df4e9b1097)


