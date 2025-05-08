# Manual de cargas de comisiones de Nevasa 

## Resumen

- A solicitud de Nevasa nos debemos empezar a encargar de sus procesos de carga manuales correspondiente a la información de comisiones de sus clientes externos, la cual debe ser llevada a cabo una vez al mes.
- Este proceso se ejecuta el tercer día hábil de cada mes, y corresponde al ingreso de información del mes anterior.
- Todos los procesos de los diferentes clientes se reducen a:
    1) Copiar la información del mes desde la planilla de cada cliente, la cual va a ser suministrada por Sebastián Chamorro el segundo día hábil de cada mes.
    2) Pegar la información dentro de su planilla correspondiente dentro de la carpeta "_cubo_" en la red de Nevasa.
    3) Ejecutar el _job_ de carga desde su base de datos SQl Server, llamado *Carga_Única_DM_Gestión*.
- El proceso de carga demora 90 minutos en completarse, y se puede ejecutar de forma manual como esperar a que corra automáticamente, lo cual ocurre todos los días, en la mañana y después de almuerzo.

### Tabla resumen

<center>

| Nombre del proceso | Archivo de entrada | Planilla de destino | Requiere procesamiento adicional? |
|:------------------:|:------------------:|:-------------------:|:---------------------------------:|
| [APV](#apv) | Carga_APV.xlsx | cubo/Apv/Carga_APV.xlsx | NO |
| [Fondos Mutuos](#fondos-mutuos) | Carga_RemuFinal_FFMM.xlsx | cubo/Rem_FM/Carga_RemuFinal_FFMM.xlsx | NO |
| [Insigneo](#insigneo) | COMISIONES_CONSOLIDADO.xlsx | cubo/Internacional/Patrimonios.xlsx | SI |
| [CFI](#cfi) | Cierre Mensual CFI - Edición - copia.xlsx | cubo/Otros_Ingresos/Colocaciones.xlsx | SI |
| [Fondos de Inversión](#fondos-de-inversión) | Correo _Remuneración Ahorro_  | cubo/Rem_FIP/Rem_FIP.xlsx | SI |

</center>

### Errores comunes

- Es posible que el proceso de carga se caída por incongruencia en los datos de los clientes, de ocurrir esto seremos contactados por Sebastián Chamorro para revisar las planillas y corregir cualquier dato errado.
    - Esto se suele dar cuando los clientes no envían la información final del mes, y es responsabilidad de ellos suministrar los datos corregidos.
    - Nuestro deber es corregir la carga y re-procesar.

<div class="page"/>

## Procesos por cliente

### APV

- **Archivo de entrada**: Carga_APV.xlsx
- **Planilla de destino**: cubo/Apv/Carga_APV.xlsx
- **Consideraciones adicionales**:
    - Hay que traspasar los datos de cada hoja del libro de Excel a su contraparte correspondiente en la planilla del cubo.
    - Hoy en día solamente va apareciendo información nueva en las hojas _Base2_ y *CA_BICE*.
    - El archivo que se recibe es incremental, contiene un histórico que crece mes a mes, pero a nosotros solo nos interesa la información del mes anterior a la fecha de proceso.
    - El mes de la data viene especificado en la columna llamada "_PERIODO_", la cual puede ser tanto la A como la K, dependiendo de la hoja.
    - Nos recomiendan darle un color diferente a las celdas de cada mes para facilitar su diferenciación.
    - Dentro de la carpeta _cubo/Apv/_ hay un .txt que pareciera tener información sobre como se lleva a cabo la carga, para revisarlo en el futuro.

### Fondos Mutuos

- **Archivo de entrada**: Carga_RemuFinal_FFMM.xlsx
- **Planilla de destino**: cubo/Rem_FM/Carga_RemuFinal_FFMM.xlsx
- **Consideraciones adicionales**:
    - Al igual que con APV, el archivo que recibiremos es incremental, solo nos interesa la información del mes anterior a la fecha de proceso.
    - El mes de la data viene especificado en la columna H, "_PERIODO_".

<div class="page"/>

### Insigneo

- **Archivo de entrada**: COMISIONES_CONSOLIDADO.xlsx
- **Planilla de destino**: cubo/Internacional/Patrimonios.xlsx
- **Consideraciones adicionales**:
    - Este archivo trae varias pestañas, pero solo nos interesa la que contiene la información de las comisiones, la cual en el ejemplo que nos presentaron se llamó "*COM_INSIGNEO*", pero a veces le cambian el nombre.
    - La hoja de comisiones se identifica por las dos columnas que tiene al final, "_validación_" y "_T/P_", y de aquí nos interesan los datos desde las columnas A (_Broker Month_) hasta la Z (_Rep Total Payout_).
    - Es necesario procesar estos datos mediante un Excel aparte, **cierre mes-año insigneo.xlxs**, pegando los datos en la hoja _Transacciones Blotter_, y recuperando los resultados de las columnas en celeste, desde la AC hasta la AR.
        - Esta planilla de procesamiento incluye una hoja de mapeo, **agente**, donde se deben especificar los agentes por su nombre, su código interno de Insigneo (el cual se especifica entre paréntesis al lado de su nombre en la columna G, "_Rep Name_") y su código CMR, el cual va a ser proporcionado por Sebastián Chamorro.
        - Se puede detectar cuando un agente no esta dado de alta mirando la columna AC, la cual tendrá un valor nulo (_#N/D_) en caso de no encontrarlo en la hoja de mapeo.
        - Al finalizar el procesamiento, es necesario modificar a mano la fecha de carga en la columna AO, para que coincida con la del fin de mes en proceso.
    - La planilla de destino utiliza un _short name_ para cada cliente en la columna Q, el cual, en caso de que aparezca un cliente nuevo, tiene que ser mapeado a mano en la hoja _Hoja2_.
        - Esta es una convención antigua que ya no se utiliza, por lo que el nombre a ser asignado no importa, solo tenemos que asegurarnos que sea único.
        - Recomiendan usar una mezcla entre el nombre y los apellidos del cliente.

<div class="page"/>

### CFI

- **Archivo de entrada**: Cierre Mensual CFI - Edición - copia.xlsx
- **Planilla de destino**: cubo/Otros_Ingresos/Colocaciones.xlsx
- **Consideraciones adicionales**:
    - Los resultados se terminan colocando en el modulo de colocaciones, el cual acepta cualquier tipo de ingreso, esto debido a que CFI entro después y no se le hizo un desarrollo propio.
    - Del Excel de entrada nos interesa la hoja _Consolidado_, y de aquí solo tenemos que copiar las columnas _Contraparte_ (B), _Nemo_ (D) y la ultima columna de la hoja, cuyo nombre sigue la convención _Facturado en {mes} {año} Custodia de {mes-1}_.
    - Estos datos deben ser procesados aparte, para lo cual hacemos una copia del Excel y creamos dos pestañas nuevas dentro del mismo, _Consolidado2_ y _custodia_.
        - Comenzamos pegando las columnas copiadas en la hoja _Consolidado2_, en el siguiente orden: _Nemo_ - _Fact_ - _emisor_.
        - Luego extraemos datos del mes de custodia (el mes anterior al de los datos) mediante la ejecución de una query en su SQL Server, la cual esta especificado en el Jupyter Notebook que nos harán llegar, especificando en el campo `cod_per` de la clausula `where` el año y el mes a procesar. Pegamos la tabla con los resultados de esta query en la pestaña _custodia_.
    - Una vez guardado el Excel modificado debemos ejecutar el script de Python que se encuentra dentro del Jupyter Notebook que nos proveerán, el cual termina generando un nuevo Excel de salida, **CargaFigCAST0225.xlsx**, el que contiene la información que de hecho debe ser cargada al cubo, dentro de la hoja _Carga_ de la planilla de colocaciones.

<div class="page"/>

### Fondos de Inversión

- **Archivo de entrada**: Correo _Remuneración Ahorro_, el cual recopila los multiples correos con la información de los diferentes fondos.
- **Planilla de destino**: cubo/Rem_FIP/Rem_FIP.xlsx
- **Consideraciones adicionales**:
    - La data es enviada por correo como imágenes, por lo que tiene que ser digitada a mano en cada caso.
    - Tenemos que considerar la columna _Mensual_ de cada correo.
    - El mapeo entre los nombres de los fondos de inversión y sus mnemotécnicos es el siguiente:

        | FONDO | NEMO | SERIE |
        |:-----:|:----:|:-----:|
        | VISION | CFINHVISION | A |
        | VISION | CFINHVISION | B |
        | AHORRO | CFINHRFL | A |
        | AHORRO | CFINHRFL | A |
        | AHORRO | CFINHRFL | B |
        | AHORRO | CFINHRFL | B |
        | AHORRO | CFINHRFL | I |
        | AHORRO | CFINHRFL | I |
        | DEUDA PRIVADA | CFINVSDP | R |
        | GESTIÓN VII | CFINGVII | R |
        | MVP | CFIPIPO-E | A |
        | MVP | CFIPIPO-E | C |
        | PROTECCIÓN | CFI-NVPRT | A |
        | AXION | CFINVCHX | A |
        | AXION | CFINVCHX | B |
        | AXION | CFINVCHX | C |
        | GESTIÓN VIII | CFINGVIII | B |
        | GESTIÓN VIII | CFINGVIII | R |
        | GESTIÓN VIII | CFINGVIII | I |
    
    - El archivo del cubo contiene la información de todos los fondos de inversión de Nevasa, incluidos algunos que ya están cerrados, por lo que tenemos que tener cuidado de solo cargar datos a los que de hecho vienen con valores en el correo.
        - Concretamente copiamos la info del mes anterior, la pegamos en la celda siguiente con un nuevo color para distinguirla, y reemplazamos los valores según la info que nos llegue del correo.
    - Los montos llegan con IVA, por lo que debemos dividir su valor en pesos por 1.19 al incluirlos en la planilla.
    - En el caso de los montos en dolares tenemos que transformarlos a pesos considerando el dolar observado al primer día hábil del mes siguiente.
    - Hay dos fondos MVP, los cuales tienen una regla propia de calculo, por lo cual no se tocan directamente en el listado, en cambio se debe modificar el valor de la columna G con el monto que se recibe, y esto automáticamente modificará el monto de estos fondos.
        - Nos dejaron estos campos en rojo en la hoja para que tengamos ojo con esto.
        - En concreto la regla de calculo es que solo se les debe considerar el 39.44% de sus valores.
