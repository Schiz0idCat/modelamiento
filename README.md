# Proceso General
Un barco, de ahora en adelante **Entidad Transportadora Exterior** (ETE) para generalizar,
llevan un **manifiesto** que contiene toda la información con respecto a la **carga** (**archivo xml**).
**La ETE le envía el manifiesto a aduana** para ingresar al país.

Acá *Beetracer* hace el primer movimiento. Descarga el xml y lo almacena en su base de datos.

Cuando un barco llega a puerto, de ahora en adelante **Punto de Acceso Nacional** (PAN) hay 2 alternativas.
- Abrir contenedor y que cada cliente que lleve su **carga** (en desuso por crecimiento de comercio exterior y pequeño tamaño de puertos).
- Camiones se llevan los contenedores a **Terminales extraportuarios** (TEP), acá están las oficinas aduaneras. Luego cada cliente se lleva su **carga**.

Una vez el camión con la **carga** llega al **TEP**, se hace el **desconsolidado** (una faena), donde se abre el contenedor y se extrae su **carga**.
En el proceso de **desconsolidado**, un **operador** usa la **app movil de Beetracer** para **validar la carga esperada**.
Acá aparece el listado **producto-cantidad-dueño** (PCD),
el **operador debe sacar una foto de comprobación de la carga** y el camión se lo lleva al cliente.

# Proceso Interno
## 1 Analizar el Contexto
Hay dos caminos iniciales desde los que *Beetracer* empieza a trabajar:
1. Que el **ETE** sea un barco internacional con un **manifiesto** con toda la información de la **carga**.
2. Que la **carga** llegue por un camión intranacional (ETN) directamente al cliente.

El primer camino es más sencillo en términos de que está estandarizado, todos los **ETE** usan el mismo modelo de **manifiesto**.
Sin embargo, en el caso de la **ETN**, cada proveedor usa su propio formato de **manifiesto**,
por lo que el sistema se debe adaptar a todos estos casos.
Teniendo especial cuidado cuando el proveedor no tiene sistema, o no tiene uno al que *Beetracer* se pueda ajustar.

## 2 Gestión del Manifiesto
## 2.1 Descarga del Manifiesto
La **ETE** de entrega los **manifiestos** aduanas, y esta los expone en un *servicio web de acceso privilegiado*.
**Beetracer tiene permisos** para descargar los xml de los manifiestos y guardarlos en su base de datos.

Esta información se guarda sobre la **carga** (esto es lo mínimo extraído de la charla):
1. Qué es.
2. Cuánto hay.
3. De dónde viene.
4. A quién pertenece.
5. Cuándo llega.

## 2.2 Manejo de Manifiesto Diferente o Ausente
En el caso de que la **carga** sea entregado por una **ETN**, el *manifiesto* no está estandarizado.
Por lo que pueden haber multiples formatos diferentes, o directamente no haber.

> [!NOTE]
> Nota del editor:
> A nivel de programación, esto es una complejidad un tanto ilusioria.
> En un escenario limpio, el programa debería ser lo más agnóstico posible.
> Da igual si el manifiesto viene de un barco o de un camión.
> Da igual si tiene un formato u otro.
> Incluso es un tanto indiferente si el manifiesto siquiera existe o no.
> 
> Para todo esto, la solución es el patrón Adapter.
> Acá se define un único manifiesto que es la fuente de verdad para nuestro programa.
> Y a este manifiesto se le crean adaptadores para traducir desde cualquier formato a nuestro tipo de manifiesto.
> ¿Y si no hay manifiesto?
> Esto es tan sencillo como dos clases que hereden de un padre común, una que considere la existencia de un manifiesto y la otra no.

## 3 Desconsolidado
El desconsolidado se puede hacer con o sin **manifiesto**.
Si existe información previe en la base de datos, lo descrito a continuación debe ser interpretado como un proceso de validación de la **carga**.
En caso contrario, el proceso de desconsolidado no es más que la creación a tiempo real del **manifiesto** faltante.

Como bonus, si existe el **manifiesto**, *Beetracer* sabe qué **carga** esperar, y puede programar el **desconsolidado** con fecha inicial y final.

Para hacer el **desconsolidado** (sea para validar o para generar el manifiesto), se deben asignar dos roles:
1. Quien descarga: todos los bienes desde el contenedor.
2. Quien valida: desde la app movil para validar o registrar los bienes del contenedor.

La **carga** puede venir en diferentes estados (estos estados se declaran desde la app movil):
1. Perfecto estado.
2. Dañado.
3. Faltante. (este estado no se menciona en la charla. Se entiende que si partimos sin manifiesto inicial, este estado no es posible).

Una vez se termina el desconsolidado, se cierra el contenedor y se devuelve por donde vino.

Cuando ya se tiene constancia de todos los vienes puede pasar una de dos cosas:
1. El cliente llega a buscar su **carga**.
2. El cliente no llega a buscar su **carga** y se manda a bodega del **TEP**. Cobran por día de uso.

Luego de todo esto, desde la app movil de cierra el proceso del desconsolidado con observaciones finales.
Todo este registro se envía a la base de datos de *Beetracer* para poder comparar lo esperado con lo que llegó.
A partir de esto, se genera un reporte en pdf que contiene las fotos, fechas, trabajadores, información sobre la **carga**, etc.
Este reporte se hace individualmente por cliente, exclusivamente con la **carga** que le corresponde, esto para poder enviarselo por email.

> [!IMPORTANT]
> Solamenta la validación de la descarga se hace desde la app movil.
> La gestión del manifiesto y la asignación de roles es desde la vista web.

> [!NOTE]
> Por estándares de seguridad, desde la app movil se saca fotos en los siguientes momentos.
> 1. Al sello que asegura el contenedor.
> 2. Al abrir el contenedor, se saca fotos al estado inicial interior. Esto antes de sacar nada.
> 3. Cuando se hace el desconsolidado, se saca foto a cada bien individualmente.
> 4. Foto al contenedor cerrado cuando se termina la faenación.
> Además, las fotos se suben una por una al servidor mientras se realizan.

# Reporte
Con toda la información recopilada se envía un informe (pdf) al cliente.

Se especifica lo siguiente:
1. En caso de haber manifiesto, se declara la diferencia entre lo esperado y lo que llegó.
2. Todas las imágenes descritas anteriormente.
3. El estado de cada paquete.

> [!CAUTION]
> Por seguridad, los informes no son envíados automáticamente.
> Pasan por un proceso de revisión en la vista web.
> Una vez el reporte es validado, se envía manualmente al cliente.

> [!NOTE]
> Si bien el informe tiene un formato estándar,
> hay veces que los clientes exigen uno personalizado.

# Vocabulario
- ***Carga***: Son todos los bienes que traen las *ETE*.
- ***Consolidado***: La acción de abrir el contenedor para validar su *manifiesto*, hacer procedimientos legales y enviar *carga* a los clientes.
- ***ETE***: Entidad Transportadora Exterior. Es el barco o cualquier otro transporte que lleva *carga* al país para ser recepcionado por aduanas.
- ***ETN***: Entidad Transportadora Nacional. Es el camión o cualquier otro transporte que lleva *carga* a otro almacen dentro del país.
- ***Manifiesto***: Detalle de *carga* (archivo xml) que lleva una *ETE*, es enviado a aduanas y descargado por *Beetracer* en su DB.
- ***PAN***: Punto de Acceso Nacional. Son puertos o similares donde llegan las *ETE* para el *desconsolidado*.
- ***PCD***: Producto-Cantidad-Dueño. Formato en la que la app movil de *Beetracer* muestra el *manifiesto* descargado para validar la *carga* y actualizar la logística.
- ***TEP***: Terminal Extraportuario. Donde la *carga* llega luego de pasar por los *PAN*, acá se usa la app movil de *Beetracer* para validar la *carga*.
