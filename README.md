# DevOps_seg4

# Realizado por: Sebastián Muñoz Zapata

En este proyecto se despliega una aplicación (se usa la misma aplicación que se ha venido trabajando durante el semestre) y luego se expone como un servicio, para finalmente escalarla. Utilizando Google Cloud como nube y su servicio Google Kubernetes Engine (GKE) para crear el clúster de Kubernetes.

# Pasos

# 1. Preparación del entorno

Para este paso se debe crear una cuenta de Google Cloud en https://cloud.google.com/, luego realizar la instalación de Google Cloud SDK y configurarlo mediante la siguiente guía: https://cloud.google.com/sdk/docs/install. Por último, configurar el acceso a Google Cloud desde el CLI de Google Cloud ejecutando: gcloud init.
Este paso ya se había realizado en la entrega anterior en la que se creó infraestructura en la nube usando Terraform.

# 2. Crear un cluster de Kubernetes en Google Cloud

Para este paso se accede a la consola de Google Cloud desde el siguiente enlace y ruta: [Google Cloud Console](https://console.cloud.google.com/) y se busca Kubernetes Engine.
Se da click en "Crear un nuevo clúster", nos da la posibilidad de crear un clúster de Autopilot o un clúster estándar, se selecciona clúster estándar y se le aplica la siguiente configuración:

- **Cluster name**: mi-cluster-gke
- **Location**: Selecciona la zona o región más cercana.
- **Machine type**: Puedes usar el tipo predeterminado `e2-medium` para los nodos.
- **Node count**: Comienza con 2 nodos.

Se crea el clúster y se configura kubectl para acceder al cluster GKE, para esto primero se debe instalar un plugin de auth de GCP con el siguiente comando: gcloud components install gke-gcloud-auth-plugin, y luego se configura que kubectl use el plugin con el comando: gcloud config set container/use_client_certificate False.
Por último, se obtienen las credenciales de acceso al clúster GKE con el comando: gcloud container clusters get-credentials mi-cluster-gke --zone <zona> --project <tu-proyecto> (Se ocultan valores sensibles).
Se verifica la conexión al clúster mediante: kubectl get nodes, y así podremos ver que nos muestra los 2 nodos creados previamente.

# 3. Desplegar una aplicación simple

Para este paso primero se crea un archivo YAML para el Deployment (deployment.yaml), el cual es el siguiente:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mi-api
  template:
    metadata:
      labels:
        app: mi-api
    spec:
      containers:
      - name: mi-api
        image: sebas807/dev_ops:test
        ports:
        - containerPort: 3000 
        envFrom:
        - secretRef:
            name: api-env-secret 

Se crea con 2 Pods (instancias) y se escoge la imagen que está en un repositorio en Docker Hub de la aplicación que se ha venido trabajando durante todo el semestre (sebas807/dev_ops:test), con el puerto que se expone en esa aplicación (3000), y con un secret llamado api-env-secret que se configura de la siguiente manera desde la consola de Google Cloud SDK Shell:

Se accede primero a la carpeta donde está ubicado el archivo .env que contiene las variables de entorno necesarias para que la aplicación se ejecute correctamente, para esto se ejecuta el comando: cd "Ruta a la carpeta", y luego se crea el secret con el comando: kubectl create secret generic api-env-secret --from-env-file=.env

Luego de eso, se procede a aplicar el archivo .yaml, accediendo primero a la carpeta donde se encuentra y ejecutando el comando: kubectl apply -f deployment.yaml

Se puede comprobar que todo salió correctamente mediante el comando: kubectl get pods, con este podremos ver que los 2 pods creados están en correcto estado y ejecutándose.

Y para verificar que la aplicación se está ejecutando correctamente en cada pod lo podemos hacer accediendo a sus logs, con el comando: kubectl logs <nombre-del-pod>

Ahora se procede a exponer el Deployment como un Servicio, para esto se crea otro archivo .yaml (service.yaml) con lo siguiente:

apiVersion: v1
kind: Service
metadata:
  name: mi-api-service
spec:
  selector:
    app: mi-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer

Podemos ver que el targetPort es el mismo puerto que se usa en el contenedor al hacer el Deployment (containerPort).
Luego aplicamos el archivo .yaml con el comando: kubectl apply -f service.yaml. Y podemos ejecutar el comando: kubectl get services para comprobar que todo salió correctamente, nos mostrará el servicio creado con el nombre mi-api-service y una EXTERNAL-IP que podemos usar para acceder y probar la API desde un navegador o Postman, siguiendo la ruta: http://EXTERNAL-IP/api/v2/leagues.

# 4. Escalar la aplicación

Para este paso se van a escalar el número de Pods (instancias) de la aplicación ejecutando el comando: kubectl scale deployment mi-aplicacion --replicas=4, lo que hará que aumente la cantidad de instancias del contenedor que están corriendo (aplicación corriendo) de 2 (valor que pusimos al crear el deployment) a 4. Y luego comprobamos que se haya aplicado correctamente con el comando:  kubectl get pods, con esto podremos comprobar que ahora en vez de 2 pods corriendo veremos 4.


# 5. Limpiar recursos

El paso final simplemente consiste en eliminar los recursos creados previamente, para eliminar el Deployment y el Servicio, se ejecutan los comandos: kubectl delete -f deployment.yaml y kubectl delete -f service.yaml. 
Y para eliminar el clúster completo se puede ejecutar el comando: gcloud container clusters delete mi-cluster-gke --zone <zona> --project <tu-proyecto>, o eliminarlo manualmente desde la consola de Google Cloud.

# Conclusiones

- Aprendizaje práctico de Kubernetes: Se comprendió cómo crear y gestionar un clúster, desplegar aplicaciones en contenedores usando Deployments y exponerlas como Servicios.
- Manejo de escalabilidad: Se aprendió a escalar aplicaciones dinámicamente aumentando o disminuyendo el número de pods para ajustar la capacidad según la demanda.
- Gestión segura de configuraciones: Se aplicaron buenas prácticas para manejar variables sensibles con Secrets en Kubernetes, evitando exponer datos confidenciales en los archivos de configuración.
- Automatización y limpieza: Se practicó la aplicación y eliminación de recursos con kubectl y la gestión del ciclo de vida del clúster en Google Cloud, facilitando la administración eficiente y ordenada del entorno.


