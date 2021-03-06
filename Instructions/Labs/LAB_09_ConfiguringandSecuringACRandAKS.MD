---
lab:
  title: 09 - Configuración y protección de ACR y AKS
  module: Module 02 - Implement Platform Protection
ms.openlocfilehash: d4b8e21e2bc795f4e9147ddc1ca81db2ddbc52cd
ms.sourcegitcommit: 2eb153f2856445e5afaa218a012cb92e3d48f24b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625708"
---
# <a name="lab-09-configuring-and-securing-acr-and-aks"></a>Laboratorio 09: Configuración y protección de ACR y AKS
# <a name="student-lab-manual"></a>Manual de laboratorio para alumnos

## <a name="lab-scenario"></a>Escenario del laboratorio

Se le ha pedido que implemente una prueba de concepto con Azure Container Registry y Azure Kubernetes Service. En concreto, la prueba de concepto debe demostrar lo siguiente:

- Uso de Dockerfile para compilar una imagen.
- Uso de Azure Container Registry para almacenar imágenes.
- Configuración de una instancia de Azure Kubernetes Service.
- Protección de aplicaciones de contenedor y acceso a estas tanto de forma interna como externa. 

> Para todos los recursos de este laboratorio, se usa la región **Este de EE. UU.** Compruebe con el instructor que esta es la región que se va a usar para la clase. 

## <a name="lab-objectives"></a>Objetivos del laboratorio

En este laboratorio completará el ejercicio siguiente:

- Ejercicio 1: Configuración y protección de ACR y AKS

## <a name="lab-files"></a>Archivos de laboratorio:

- **\\Allfiles\\Labs\\09\\nginxexternal.yaml**
- **\\Allfiles\\Labs\\09\\nginxinternal.yaml**

### <a name="exercise-1-configuring-and-securing-acr-and-aks"></a>Ejercicio 1: Configuración y protección de ACR y AKS

### <a name="estimated-timing-45-minutes"></a>Tiempo estimado: 45 minutos

> Para todos los recursos de este laboratorio, se usa la región **Este de EE. UU.** Compruebe con el instructor que esta es la región que se va a usar para la clase. 

En este ejercicio completará las tareas siguientes:

- Tarea 1: Crear una instancia de Azure Container Registry
- Tarea 2: Crear un Dockerfile, compilar un contenedor e insertarlo en Azure Container Registry
- Tarea 3: Crear un clúster de Azure Kubernetes Service
- Tarea 4: Conceder permisos de clúster de AKS para acceder a ACR
- Tarea 5: Implementar un servicio externo en AKS
- Tarea 6: Comprobar que puede acceder a un servicio hospedado en AKS externo
- Tarea 7: Implementar un servicio interno en AKS
- Tarea 8: Comprobar que puede acceder a un servicio hospedado en AKS interno

#### <a name="task-1-create-an-azure-container-registry"></a>Tarea 1: Crear una instancia de Azure Container Registry

En esta tarea, creará un grupo de recursos para el laboratorio en Azure Container Registry.

1. Inicie sesión en Azure Portal **`https://portal.azure.com/`** .

    >**Nota**: Inicie sesión en Azure Portal con una cuenta que tenga el rol Propietario o Colaborador en la suscripción de Azure que usa para este laboratorio y el rol Administrador global en el inquilino de Azure AD asociado a esa suscripción.

1. Haga clic en el primer icono de la esquina superior derecha de Azure Portal para abrir Cloud Shell. Si se le solicita, haga clic en **Bash** y en **Crear almacenamiento**.

1. Asegúrese de que **Bash** está seleccionado en el menú desplegable de la esquina superior izquierda del panel de Cloud Shell.

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para crear un grupo de recursos para este laboratorio:

    ```sh
    az group create --name AZ500LAB09 --location eastus
    ```

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para comprobar que se ha creado el grupo de recursos:

    ```
    az group list --query "[?name=='AZ500LAB09']" -o table
    ```

1. En la sesión de Bash en el panel Cloud Shell, ejecute lo siguiente para crear una instancia de Azure Container Registry (ACR) (el nombre de la instancia de ACR debe ser único globalmente): 

    ```sh
    az acr create --resource-group AZ500LAB09 --name az500$RANDOM$RANDOM --sku Basic
    ```

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para confirmar que se ha creado la nueva instancia de ACR:

    ```sh
    az acr list --resource-group AZ500LAB09
    ```

    >**Nota**: Registre el nombre del ACR. Lo necesitará en la próxima tarea.

#### <a name="task-2-create-a-dockerfile-build-a-container-and-push-it-to-azure-container-registry"></a>Tarea 2: Crear un Dockerfile, compilar un contenedor e insertarlo en Azure Container Registry

En esta tarea, creará un Dockerfile, compilará una imagen a partir del Dockerfile e implementará la imagen en la instancia de ACR. 

1. En la sesión de Bash del panel de Cloud Shell, ejecute lo siguiente para crear un Dockerfile para crear una imagen basada en Nginx: 

    ```sh
    echo FROM nginx > Dockerfile
    ```

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para compilar una imagen a partir del Dockerfile e insertar la imagen en la nueva instancia de ACR. 

    >**Nota**: El punto final es obligatorio al final de la línea de comandos. Designa el directorio actual como la ubicación de Dockerfile. 

    ```sh
    ACRNAME=$(az acr list --resource-group AZ500LAB09 --query '[].{Name:name}' --output tsv)

    az acr build --image sample/nginx:v1 --registry $ACRNAME --file Dockerfile .
    ```

    >**Nota**: Espere a que el comando se complete correctamente. Esto puede tardar unos 2 minutos.

1. Cierre el panel de Cloud Shell.

1. En Azure Portal, vaya al grupo de recursos **AZ500Lab09** y, en la lista de recursos, haga clic en la entrada que representa la instancia de Azure Container Registry que aprovisionó en la tarea anterior.

1. En la hoja Registro de contenedor, en la sección **Servicios**, haga clic en **Repositorios**. 

1. Compruebe que la lista de repositorios incluye la nueva imagen de contenedor denominada **sample/nginx**.

1. Haga clic en la entrada **sample/nginx** y compruebe la presencia de la etiqueta **v1** que identifica la versión de la imagen.

1. Haga clic en la entrada **v1** para ver el manifiesto de imagen.

    >**Nota**: El manifiesto incluye la síntesis del mensaje sha256, la fecha de creación del manifiesto y las entradas de la plataforma. 

#### <a name="task-3-create-an-azure-kubernetes-service-cluster"></a>Tarea 3: Crear un clúster de Azure Kubernetes Service

En esta tarea, creará un servicio de Azure Kubernetes y revisará los recursos implementados. 

1. En Azure Portal, en el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página, escriba **Servicios de Kubernetes** y presione la tecla **Entrar**.

1. En la hoja **Servicios de Kubernetes**, haga clic en **+ Crear** y, en el menú desplegable, haga clic en **+ Add Kubernetes cluster** (+ Agregar clúster de Kubernetes).

1. En la pestaña **Aspectos básicos** de la hoja **Crear un clúster de Kubernetes**, configure las siguientes opciones (deje las demás con los valores predeterminados):

    |Configuración|Value|
    |----|----|
    |Subscription|nombre de la suscripción de Azure que usa en este laboratorio|
    |Resource group|**AZ500LAB09**|
    |Nombre del clúster de Kubernetes|**MyKubernetesCluster**|
    |Region|**(EE. UU.) Este de EE. UU.**|
    |Zonas de disponibilidad |**None**|
    |Recuento de nodos|**1**|

1. Haga clic en **Siguiente: Grupos de nodos >** y, en la pestaña **Grupos de nodos** de la hoja **Crear un clúster de Kubernetes**, configure las siguientes opciones (deje las demás con los valores predeterminados):

    |Configuración|Value|
    |----|----|
    |Habilitar nodos virtuales|Casilla desactivada|
    |VM Scale Sets|Casilla desactivada|
    
1. Haga clic en **Siguiente: Autenticación >** , en la pestaña **Autenticación** de la hoja **Crear un clúster de Kubernetes**, acepte los valores predeterminados y haga clic en **Siguiente: Redes >** . 

1. En la pestaña **Redes** de la hoja **Crear un clúster de Kubernetes**, especifique las siguientes opciones (deje las demás con los valores predeterminados):

    |Configuración|Value|
    |----|----|
    |Configuración de red|**Azure CNI**|
    |Prefijo del nombre DNS|Cualquier nombre de host DNS válido y único globalmente|

    >**Nota**: AKS se puede configurar como un clúster privado. Se asigna una IP privada al servidor de API para garantizar que el tráfico de red entre el servidor de API y los grupos de nodos permanece solo en la red privada. Para obtener más información, visite la página [Creación de un clúster privado de Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/aks/private-clusters).

1. Haga clic en **Siguiente: Integraciones >** y, en la pestaña **Integraciones** de la hoja **Crear un clúster de Kubernetes**, establezca **Container monitoring** (Supervisión de contenedores) en **Deshabilitado**. 

    >**Nota**: En escenarios de producción, le gustaría habilitar la supervisión. La supervisión está deshabilitada en este caso, ya que no se trata en el laboratorio. 

1. Haga clic en **Revisar + crear** y, después, en **Crear**.

    >**Nota**: Espere a que la implementación se complete. Esto puede tardar unos 10 minutos.

1. Cuando se complete la implementación, en Azure Portal, en el cuadro de texto **Buscar recursos, servicios y documentos** de la parte superior de la página, escriba **Grupos de recursos** y presione la tecla **Entrar**.

1. En la hoja **Grupos de recursos**, en la lista de grupos de recursos, observe el nuevo grupo de recursos denominado **MC_AZ500LAB09_MyKubernetesCluster_eastus**, que contiene los componentes de los nodos de AKS. Revise los recursos de este grupo de recursos. 
    
1. Vuelva a la hoja **Grupos de recursos** y haga clic en la entrada **AZ500LAB09**. 

    >**Nota**: En la lista de recursos, observe el clúster de AKS y la red virtual correspondiente.

1. En Azure Portal, abra una sesión de Bash en Cloud Shell. 

    >**Nota**: Asegúrese de que **Bash** esté seleccionado en el menú desplegable superior izquierdo del panel de Cloud Shell.

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para conectarse al clúster de Kubernetes:

    ```sh
    az aks get-credentials --resource-group AZ500LAB09 --name MyKubernetesCluster
    ```

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para enumerar los nodos del clúster de Kubenetes: 

    ```sh
    kubectl get nodes
    ```

    >**Nota**: Compruebe que el **Estado** del nodo del clúster aparece como **Listo**.

#### <a name="task-4-grant-the-aks-cluster-permissions-to-access-the-acr-and-manage-its-virtual-network"></a>Tarea 4: Conceder permisos al clúster de AKS para acceder a ACR y administrar su red virtual

En esta tarea, concederá al clúster de AKS permiso para acceder a ACR y administrar su red virtual. 

1. En la sesión de Bash del panel de Cloud Shell, ejecute lo siguiente para configurar el clúster de AKS para usar la instancia de Azure Container Registry que creó anteriormente en este laboratorio. 

    ```sh
    ACRNAME=$(az acr list --resource-group AZ500LAB09 --query '[].{Name:name}' --output tsv)

    az aks update -n MyKubernetesCluster -g AZ500LAB09 --attach-acr $ACRNAME
    ```

    >**Nota**: Este comando concede la asignación de roles "acrpull" a la instancia de ACR. 

    >**Nota**: Este comando puede tardar unos minutos en completarse. 

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para conceder al clúster de AKS el rol Colaborador para su red virtual. 

    ```sh
    RG_AKS=AZ500LAB09
    
    AKS_VNET_NAME=AZ500LAB09-vnet
    
    AKS_CLUSTER_NAME=MyKubernetesCluster
    
    AKS_VNET_ID=$(az network vnet show --name $AKS_VNET_NAME --resource-group $RG_AKS --query id -o tsv)
    
    AKS_MANAGED_ID=$(az aks show --name $AKS_CLUSTER_NAME --resource-group $RG_AKS --query identity.principalId -o tsv)
    
    az role assignment create --assignee $AKS_MANAGED_ID --role "Contributor" --scope $AKS_VNET_ID
    ```

#### <a name="task-5-deploy-an-external-service-to-aks"></a>Tarea 5: Implementar un servicio externo en AKS

En esta tarea, descargará los archivos de manifiesto, editará el archivo YAML y aplicará los cambios al clúster. 

1. En la sesión de Bash en el panel de Cloud Shell, haga clic en el icono **Cargar/Descargar archivos**, en el menú desplegable, haga clic en **Cargar**; en el cuadro de diálogo **Abrir**, navegue hasta la ubicación donde descargó los archivos de laboratorio, seleccione **\\Allfiles\\Labs\\09\\nginxexternal.yaml** y haga clic en **Abrir**. A continuación, seleccione **\\Allfiles\\Labs\\09\\nginxinternal.yaml** y haga clic en **Abrir**.

1. En la sesión de Bash del panel de Cloud Shell, ejecute lo siguiente para identificar el nombre de la instancia de Azure Container Registry:

    ```sh
    echo $ACRNAME
    ```

    >**Nota**: Registre el nombre de la instancia de Azure Container Registry. Lo necesitará más adelante en esta tarea.

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para abrir el archivo nginxexternal.yaml, para poder editar su contenido. 

    ```sh
    code ./nginxexternal.yaml
    ```

    >**Nota**: Este es el archivo .yaml *externo*.

1. En el panel del editor, desplácese hacia abajo hasta la **línea 24** y reemplace el marcador de posición **`<ACRUniquename>`** por el nombre de la instancia de ACR.

1. En el panel del editor, en la esquina superior derecha, haga clic en el icono de **puntos suspensivos**, haga clic en **Guardar** y, a continuación, haga clic en **Cerrar editor**. 

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para aplicar el cambio en el clúster:

    ```sh
    kubectl apply -f nginxexternal.yaml
    ```

1. En la sesión de Bash en el panel de Cloud Shell, revise la salida del comando que ejecutó en la tarea anterior para comprobar que se han creado la implementación y el servicio correspondiente. 

    ```
    deployment.apps/nginxexternal created
    service/nginxexternal created
    ```

#### <a name="task-6-verify-the-you-can-access-an-external-aks-hosted-service"></a>Tarea 6: Comprobar que puede acceder a un servicio hospedado en AKS externo

En esta tarea, compruebe que se puede acceder externamente al contenedor mediante la dirección IP pública.

1. En la sesión de Bash del panel de Cloud Shell, ejecute lo siguiente para recuperar la información sobre el servicio nginxexternal, incluidos el nombre, el tipo, las direcciones IP y los puertos. 

    ```sh
    kubectl get service nginxexternal
    ```

1. En la sesión de Bash en el panel de Cloud Shell, revise la salida y registre el valor en la columna External-IP. Esta información la necesitará en el siguiente paso. 

1. Abra Internet Explorer y vaya a la dirección IP que identificó en el paso anterior.

1. Asegúrese de la página **Bienvenido a nginx** se muestra. 

#### <a name="task-7-deploy-an-internal-service-to-aks"></a>Tarea 7: Implementar un servicio interno en AKS

En esta tarea, implementará el servicio de acceso interno en AKS. 

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para abrir el archivo nginxinternal.yaml, para poder editar su contenido. 

    ```sh
    code ./nginxinternal.yaml
    ```

    >**Nota**: Este es el archivo .yaml *interno*.

1. En el panel del editor, desplácese hacia abajo hasta la línea que contiene la referencia a la imagen de contenedor y reemplace el marcador de posición **`<ACRUniquename>`** por el nombre de la instancia de ACR.

1. En el panel del editor, en la esquina superior derecha, haga clic en el icono de **puntos suspensivos**, haga clic en **Guardar** y, a continuación, haga clic en **Cerrar editor**. 

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para aplicar el cambio en el clúster:

    ```sh
    kubectl apply -f nginxinternal.yaml
    ```

1.  En la sesión de Bash en el panel de Cloud Shell, revise la salida para comprobar que se han creado la implementación y el servicio:

    ```
    deployment.apps/nginxinternal created
    service/nginxinternal created
    ```

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para recuperar la información sobre el servicio nginxinternal, incluidos el nombre, el tipo, las direcciones IP y los puertos. 

    ```sh
    kubectl get service nginxinternal
    ```

1. En la sesión de Bash en el panel de Cloud Shell, revise la salida. La dirección de External-IP es, en este caso, una dirección IP privada. 

    >**Nota**: Registre esta dirección IP. Lo necesitará en la próxima tarea. 

    >**Nota**: Para acceder al punto de conexión de servicio interno, se conectará de manera interactiva a uno de los pods que se ejecutan en el clúster. 

    >**Nota**: También puede usar la dirección de CLUSTER-IP.

#### <a name="task-8-verify-the-you-can-access-an-internal-aks-hosted-service"></a>Tarea 8: Comprobar que puede acceder a un servicio hospedado en AKS interno

En esta tarea, usará uno de los pods que se ejecutan en el clúster de AKS para acceder al servicio interno. 

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para enumerar los pods del espacio de nombres predeterminado en el clúster de AKS:

    ```sh
    kubectl get pods
    ```

1. En la lista de pods, copie la primera entrada en la columna **NAME**.

    >**Nota**: Este es el pod que usará en los pasos posteriores.

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para conectarse de manera interactiva al primer pod (reemplace el marcador de posición `<pod_name>` por el nombre que copió en el paso anterior):

    ```sh
    kubectl exec -it <pod_name> -- /bin/bash
    ```

1. En la sesión de Bash en el panel de Cloud Shell, ejecute lo siguiente para comprobar que el sitio web de nginx está disponible a través de la dirección IP privada del servicio (reemplace el marcador de posición `<internal_IP>` por la dirección IP que registró en la tarea anterior):

    ```sh
    curl http://<internal_IP>
    ```

1. Cierre el panel de Cloud Shell.

> Resultado: ha configurado y protegido ACR y AKS.


**Limpieza de recursos**

> No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no se generarán costes inesperados.

1. Haga clic en el primer icono de la esquina superior derecha de Azure Portal para abrir Cloud Shell. 

1. En el menú desplegable superior izquierdo del panel de Cloud Shell, seleccione **PowerShell** y, cuando se le solicite, haga clic en **Confirmar**.

1. En la sesión de PowerShell en el panel de Cloud Shell, ejecute lo siguiente para quitar los grupos de recursos que creó en este laboratorio:
  
    ```powershell
    Remove-AzResourceGroup -Name "AZ500LAB09" -Force -AsJob
    ```

1.  Cierre el panel de **Cloud Shell**. 
