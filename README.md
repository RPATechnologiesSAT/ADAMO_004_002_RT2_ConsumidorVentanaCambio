### FrameWork RT2 ###

![#](/Documentation/img/0.png)


**Resumen**

Framework para procesos de tipo transactional de tipo items cola cargados en una ejecución previa.
Para el consumidor o performer del modelo tipico Productor-Consumidor.


##### Argumentos
    **in_ConfigPath** - Default value: "Data\Config.json"

##### Librerias


Dentro de la carpeta Librerias se deja espacio para incluir aquellos ".xaml" de acciones que puedan ser reutilizados en diferentes procesos.

![#](/Documentation/img/0.2.png)


##### Guía básica de uso 
1) Descarga directamente la última release [RPATechnologies_RT2_Template.zip]
2) Descomprime el .zip en tu directorio Documents\UiPath\
3) Renombra el proyecto a NombreCliente_AliasProcesoGeneral_RT1_0X_NombreDelProceso (Por ejemplo NIMOGORDILLO_CVN_RT1_01_DescargarExpedientes)
4) Incluye las variables y assets necesarios para el proceso en el archivo de configuración "Data\Config.json" respetando la estructura del ejemplo.
5) Desarrolla en un xaml independiente "Process.xaml" las acciones necesarias para la ejecución del proceso.(Usa el modelo Sequence, Flowchart o State machineque mejor se ajuste a la necesidad.)
6) Invoca el xaml "Process.xaml" dentro del Process State.



**1. Configuration State**

![#](/Documentation/img/1.png)

##### ConfigFramework
Variables de configuración del Framework
-ProjectName
-InitTime
-Resolution
-UserName
-Machine
-ResultMessage
-ResultStatus
-TotalSuccessTransactions
-TotalBusinessTransactions
-TotalSystemTransactions
-dt_Ejecucion

##### ConfigProcess
Variables de configuración necesarias para la ejecución. <br>
El fichero Config contiene la información de Settings, Assets y Credentials. <br>
Debe respetarse la estructura de JArray del ejemplo de archivo "Config.json" para que tanto las variables como los assets del orquestador puedan ser obtenidos al inicio de la ejecución.

![#](/Documentation/img/1.1.png)

**¡IMPORTANTE!** Si el proceso no requiere del uso de assets, y/o Credentials, será necesario eliminar el correspondiente array del archivo "config.json" para evitar error de obtención al inicio del proceso.
Quedará de la siguiente manera:

![#](/Documentation/img/6.png)



**2. Get transaction Data**

![#](/Documentation/img/2.png)

- Stop Requested: ResultStatus = 0
- Handle SE:  ResultStatus = -1
- Handle BE:  ResultStatus = 0 (No Data Remaining)

Se comprueba si se ha ordenado parar el proceso desde el orquestador.

![#](/Documentation/img/7.4.png)

Se comprueba que el número de excepciones de tipo system acumuladas en toda la ejecucion no supere el maximo indicado en el config.

![#](/Documentation/img/7.1.png)


Antes de recoger el siguiente elemento a tratar:
Se comprueba el numero de items pendientes en la cola de trabajo.
En caso de que no haya mas items pendientes.

![#](/Documentation/img/2.1.png)
![#](/Documentation/img/2.2.png)

Sin elementos

![#](/Documentation/img/7.2.png)

último elemento tratado

![#](/Documentation/img/7.3.png)


En caso de quedar elementos pendientes, se obtiene el siguiente para procesarlo.


**3. Process Transaction**

![#](/Documentation/img/3.png)

En este state será donde invocaremos el "Process.xaml" que ha sido desarrollado con las acciones que debe realizar el proceso. <br>

En caso de excepción de tipo System o Business dentro de este xaml, la excepción debe ser relanzada con un throw o rethrow para que pueda ser capturada por el framework en este Process State.
Una vez levantada la excepción desde el "Process.xaml", su gestión queda contemplada por el framework y no requiere ninguna modificación.

<br>Business RuleException:

![#](/Documentation/img/3.1.png)


<br>System Exception:

![#](/Documentation/img/3.2.png)

![#](/Documentation/img/3.4.png)

Tener en cuenta que en caso de system, al finalizar la transaccion el entorno será limpiado y será necesario volver a prepararlo (InitAllApplications) de cara al siguiente item, por ejemplo, haciendo de nuevo el login o navegando a la página que corresponda.

<br>Finally

![#](/Documentation/img/3.3.png)

Independientemente del resultado de la transaccion procesada, en el finally se realizarán las siguientes acciones para dejar traza del elemento tratado y de la ejecución completa. 

Tras completar la ejecución unitaria, se resetean valores para procesar el siguiente elemento pasando nuevamente por el Get transaction Data en todos los casos.

Por cada elemento tratado se irá llevando el control del resumen de ejecucion con el numero de elementos procesados y sus distintos desenlaces.

Por cada elemento tratado se añadirá un nuevo registro sobre la tabla de resumen dt_execution.

![#](/Documentation/img/0.1.png)



**4. Final State**

![#](/Documentation/img/4.png)

Tanto en caso de ejecución correcta como de excepción, el proceso pasará por esta fase antes de terminar la ejecución. <br>

El objetivo es liberar el entorno de ejecución mediante el cierre de aplicaciones independientemente del desenlace. <br>

Si fuera necesario realizar alguna acción en caso de error, por ejemplo, enviar un correo de notificación, dicha acción será incluida en este punto del framework. <br>

![#](/Documentation/img/4.1.png)

En caso de error durante esta acción adicional, la excepción será también capturada y el mensaje de error concatenado al mensaje previo de error del process. 
Una vez limpiado el entorno y llevado a cabo las posibles acciones que fueran necesarias, la excepción será levantada para setear el job como Failed en el orquestador de cara a la gestión de su mantenimiento.



En caso de ejecucion correcta, por defecto, se escribirá el dt con el detalle resumen de cada transacción procesada en la ejecución como en el ejemplo. 

![#](/Documentation/img/8.png)


![#](/Documentation/img/8.1.png)