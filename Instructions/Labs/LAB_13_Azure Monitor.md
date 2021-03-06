---
lab:
  title: 13 - Azure Monitor
  module: Module 04 - Manage security operations
ms.openlocfilehash: cf654eb2550d7daf9f416953746e0e432f3d2662
ms.sourcegitcommit: 2eb153f2856445e5afaa218a012cb92e3d48f24b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625706"
---
# <a name="lab-13-azure-monitor"></a>Laboratorio 13: Azure Monitor
# <a name="student-lab-manual"></a>Manual de laboratorio para alumnos

## <a name="lab-scenario"></a>Escenario del laboratorio

Se le ha pedido que cree una prueba de concepto de supervisión del rendimiento de la máquina virtual. En concreto, quiere:

- Configurar una máquina virtual para que se puedan recopilar datos de telemetría y registros.
- Mostrar la telemetría y los registros que se pueden recopilar.
- Mostrar cómo se pueden utilizar y consultar los datos. 

> Para todos los recursos de este laboratorio, se usa la región **Este de EE. UU.** Compruebe con el instructor que esta es la región que se va a usar para la clase. 

## <a name="lab-objectives"></a>Objetivos del laboratorio

En este laboratorio completará el ejercicio siguiente:

- Ejercicio 1: Recopilar datos de una máquina virtual de Azure con Azure Monitor

### <a name="exercise-1-collect-data-from-an-azure-virtual-machine-with-azure-monitor"></a>Ejercicio 1: Recopilar datos de una máquina virtual de Azure con Azure Monitor

### <a name="exercise-timing-20-minutes"></a>Tiempo del ejercicio: 20 minutos

En este ejercicio completará las tareas siguientes: 

- Tarea 1: Implementar una máquina virtual de Azure 
- Tarea 2: Crear un área de trabajo de Log Analytics
- Tarea 3: Habilitar la extensión de máquina virtual de Log Analytics
- Tarea 4: Recopilar datos de rendimiento y eventos de máquina virtual
- Tarea 5: Ver y consultar los datos recopilados 

#### <a name="task-1-deploy-an-azure-virtual-machine"></a>Tarea 1: Implementar una máquina virtual de Azure

1. Inicie sesión en Azure Portal **`https://portal.azure.com/`** .

    >**Nota**: Inicie sesión en Azure Portal con una cuenta que tenga el rol Propietario o Colaborador en la suscripción de Azure que usa para este laboratorio.

1. Haga clic en el primer icono de la esquina superior derecha de Azure Portal para abrir Cloud Shell. Si se le solicita, seleccione **PowerShell** y **Crear almacenamiento**.

1. Asegúrese de que **PowerShell** esté seleccionado en el menú desplegable superior izquierdo del panel de Cloud Shell.

1. En la sesión de PowerShell en el panel de Cloud Shell, ejecute lo siguiente para crear el grupo de recursos que se usará en este laboratorio:
  
    ```powershell
    New-AzResourceGroup -Name AZ500LAB131415 -Location 'EastUS'
    ```

    >**Nota**: Este grupo de recursos se usará para los laboratorios 13, 14 y 15. 

1. En la sesión de PowerShell en el panel de Cloud Shell, ejecute lo siguiente para crear una máquina virtual de Azure. 

    ```powershell
    New-AzVm -ResourceGroupName "AZ500LAB131415" -Name "myVM" -Location 'EastUS' -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName   "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress" -OpenPorts 80,3389
    ```

1.  Cuando se le pidan las credenciales:

    |Configuración|Valor|
    |---|---|
    |Nombre de usuario|**localadmin**|
    |Contraseña|**Pa55w.rd1234**|

    >**Nota**: Espere a que la implementación se complete. 

1. En la sesión de PowerShell en el panel de Cloud Shell, ejecute lo siguiente para confirmar que se creó la máquina virtual denominada **myVM** y que su valor de **provisioningState** es **Correcto**.

    ```powershell
    Get-AzVM -Name 'myVM' -ResourceGroupName 'AZ500LAB131415' | Format-Table
    ```

1. Cierre el panel de Cloud Shell. 

#### <a name="task-2-create-a-log-analytics-workspace"></a>Tarea 2: Crear un área de trabajo de Log Analytics

En esta tarea, creará un área de trabajo de Log Analytics. 

1. En Azure Portal, use el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página de Azure Portal, escriba **Áreas de trabajo de Log Analytics** y presione la tecla **Entrar**.

1. En la página **Áreas de trabajo de Log Analytics**, haga clic en **+ Crear**.

1. En la pestaña **Aspectos básicos** de la hoja **Crear un área de trabajo de Log Analytics**, especifique las siguientes opciones (deje las demás con los valores predeterminados):

    |Configuración|Value|
    |---|---|
    |Subscription|nombre de la suscripción de Azure que usa en este laboratorio|
    |Resource group|**AZ500LAB131415**|
    |Name|Cualquier nombre válido y único globalmente|
    |Region|**(EE. UU.) Este de EE. UU.**|

1. Haga clic en **Siguiente: Plan de tarifa >** ; en la pestaña **Plan de tarifa** de la hoja **Crear área de trabajo de Log Analytics**, acepte el plan de tarifa predeterminado **Pago por uso (por GB 2018)** y haga clic en **Revisar y crear**.

1. En la pestaña **Revisar y crear** de la hoja **Crear área de trabajo de Log Analytics**, haga clic en **Crear**.

#### <a name="task-3-enable-the-log-analytics-virtual-machine-extension"></a>Tarea 3: Habilitar la extensión de máquina virtual de Log Analytics

En esta tarea, habilitará la extensión de máquina virtual de Log Analytics. Esta extensión instala el agente de Log Analytics en máquinas virtuales Windows y Linux. Este agente recopila datos de la máquina virtual y los transfiere al área de trabajo de Log Analytics que designe. Una vez instalado el agente, se actualizará automáticamente para garantizar que siempre tenga las características y correcciones más recientes. 

1. En Azure Portal, vuelva a la hoja **Áreas de trabajo de Log Analytics** y, en la lista de áreas de trabajo, haga clic en la entrada que representa el área de trabajo que creó en la tarea anterior.

1. En la hoja Área de trabajo de Log Analytics, en la sección **Conectar un origen de datos**, haga clic en la entrada **Máquinas virtuales**.

    >**Nota**: Para que el agente se instale correctamente, la máquina virtual debe estar en ejecución.

1. En la lista de máquinas virtuales, busque la entrada que representa la VM de Azure **myVM** que implementó en la primera tarea de este ejercicio y observe que aparece como **No conectada**.

1. Haga clic en la entrada **myVM** y, a continuación, en la hoja **myVM**, haga clic en **Conectar**. 

1. Espere a que la máquina virtual se conecte al área de trabajo de Log Analytics.

    >**Nota**: Esta operación puede tardar unos minutos. El **Estado** que se muestra en la hoja **myVM** cambiará de **Conectando** a **Esta área de trabajo**. 

#### <a name="task-4-collect-virtual-machine-event-and-performance-data"></a>Tarea 4: Recopilar datos de rendimiento y eventos de máquina virtual

En esta tarea, configurará la recopilación del registro del sistema Windows y varios contadores de rendimiento comunes. También revisará otros orígenes que están disponibles.

1. En Azure Portal, vuelva al área de trabajo de Log Analytics que creó anteriormente en este ejercicio.

1. En la hoja Área de trabajo de Log Analytics, en la sección **Configuración**, haga clic en **Configuración de agentes**.

1. En la hoja **Configuración de agentes**, revise las opciones configurables, incluidas Registros de eventos de Windows, Contadores de rendimiento de Windows, Contadores de rendimiento de Linux, Registros de IIS y Syslog. 

1. Asegúrese de que la opción **Registros de eventos de Windows** está seleccionada, haga clic en **+ Add windows event log** (+ Agregar registro de eventos de Windows). En la lista de tipos de registro de eventos, seleccione **Sistema**.

    >**Nota**: Esta es la forma de agregar registros de eventos al área de trabajo. Otras opciones son, por ejemplo, **Eventos de hardware** o **Servicio de administración de claves**.  

1. Desactive la casilla **Información** y deje activadas las casillas **Error** y **Advertencia**.

1. Haga clic en **Contadores de rendimiento de Windows**, haga clic en **+ Add performance counter** (+ Agregar contador de rendimiento), revise la lista de contadores de rendimiento disponibles y agregue los siguientes:

    - Procesador (\*)\% Tiempo del procesador
    - Seguimiento de eventos para Windows\Uso de memoria total --- Bloque no paginado
    - Seguimiento de eventos para Windows\Uso de memoria total --- Bloque paginado

    >**Nota**: Los contadores se agregan y configuran con un intervalo de ejemplo de colección de 60 segundos.
  
1. En la hoja **Agents configuration** (Configuración de agentes), haga clic en **Aplicar**.

#### <a name="task-5-view-and-query-collected-data"></a>Tarea 5: Ver y consultar los datos recopilados

En esta tarea, ejecutará una búsqueda de registros en la recopilación de datos. 

1. En Azure Portal, vuelva al área de trabajo de Log Analytics que creó anteriormente en este ejercicio.

1. En la hoja Área de trabajo de Log Analytics, en la sección **General**, haga clic en **Registros**.

1. Si es necesario, cierre la ventana **Bienvenido a Log Analytics**. 

1. En el panel **Consultas**, en la columna **Todas las consultas**, desplácese hacia abajo hasta la parte inferior de la lista de tipos de recursos y haga clic en **Máquinas virtuales**.
    
1. Revise la lista de consultas predefinidas, seleccione **Memory and CPU usage** (Uso de memoria y CPU) y haga clic en el botón **Ejecutar** correspondiente.

    >**Nota**: Puede empezar con la consulta **Memoria disponible de la máquina virtual**. Si no obtiene ningún resultado, compruebe que el ámbito está establecido en máquina virtual.

1. La consulta se abrirá automáticamente en una nueva pestaña de consulta. 

    >**Nota**: Log Analytics usa el lenguaje de consulta Kusto. Puede personalizar las consultas existentes o crear las suyas propias. 

    >**Nota**: Los resultados de la consulta seleccionada se muestran automáticamente debajo del panel de consulta. Para volver a ejecutar la consulta, haga clic en **Ejecutar**.

    >**Nota**: Dado que esta máquina virtual se acaba de crear, es posible que aún no haya ningún dato. 

    >**Nota**: Tiene la opción de mostrar datos en formatos diferentes. También tiene la opción de crear una regla de alertas basada en los resultados de la consulta.

    >**Nota**: Puede generar alguna carga adicional en la VM de Azure que implementó anteriormente en este laboratorio mediante los pasos siguientes:

    1. Vaya a la hoja VM de Azure.
    1. En la hoja VM de Azure, en la sección **Operaciones**, seleccione **Ejecutar comando**; en la hoja **Ejecutar script de comando**, escriba el siguiente script y haga clic en **Ejecutar**:
    2. 
       ```cmd
       cmd
       :loop
       dir c:\ /s > SWAP
       goto loop
       ```
       
    1. Vuelva a la hoja Log Analytics y ejecute de nuevo la consulta. Es posible que tenga que esperar unos minutos a que se recopilen los datos y volver a ejecutar la consulta.

> Resultados: ha usado un área de trabajo de Log Analytics para configurar orígenes de datos y registros de consultas. 

**Limpieza de recursos**

>**Nota**: No quite los recursos de este laboratorio, ya que son necesarios para los laboratorios de Azure Security Center y Azure Sentinel.
 
