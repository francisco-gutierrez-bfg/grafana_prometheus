1. Desplegar Kubernetes Dashboard
Descarga y aplica el manifiesto oficial del Kubernetes Dashboard:
 kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

Verifica que los recursos hayan sido creados correctamente:
 kubectl get all -n kubernetes-dashboard

Asegúrese de que los pods y servicios estén en estado Running.

2. Configurar el Servicio como NodePort
Por defecto, el servicio del Dashboard es de tipo ClusterIP, lo que limita el acceso al clúster. Para exponerlo, cambia su tipo a NodePort.

Editar el Servicio
Ejecutar el siguiente comando para editar el servicio:
 kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard

En el archivo que se abre, cambia el campo type de ClusterIP a NodePort:

spec:
  type: NodePort

Guarda los cambios y sal del editor.

Verificar el Puerto NodePort
Confirmar el puerto asignado por Kubernetes:

kubectl get svc -n kubernetes-dashboard

El resultado será algo como esto:

NAME                        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard        NodePort   10.96.0.1      <none>        443:32412/TCP   5m

En este caso, el puerto NodePort asignado es 32412.

3. Configurar Port-Forward (opcional)
Para acceder al Dashboard localmente, usa port-forward:
 kubectl port-forward -n kubernetes-dashboard svc/kubernetes-dashboard 8443:443

4. Configurar un ingress (opcional)
Utilice el ingress de su elección.

Esto redirige el puerto 443 del servicio al puerto 8443 en tu máquina local.

5. Crear un Usuario Administrador
Para acceder al Dashboard, necesitas un token. Crea un archivo YAML con las siguientes configuraciones para un ServiceAccount y un ClusterRoleBinding:

Archivo admin-user.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard

Aplicar este archivo:

kubectl apply -f admin-user.yaml

Generar el Token
Obtener el token del usuario administrador:
 kubectl -n kubernetes-dashboard create token admin-user

Copia el token para usarlo más adelante.

6. Acceder al Dashboard
Desde la Máquina Local (Port-Forward)
Acceder al Dashboard localmente desde tu navegador:

https://localhost:8443
Desde la Red (NodePort)
Obtener la IP de uno de los nodos de tu clúster:
 kubectl get nodes -o wide

Luego, abrir el navegador y accede a:

https://<NODE_IP>:<NODE_PORT>
Reemplazar <NODE_IP> con la IP del nodo y <NODE_PORT> con el puerto NodePort (por ejemplo, 32412).

7. Instalar metrics server:
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.1/components.yaml

8. (Opcional) Mejorar la Seguridad
Certificados SSL: Configura certificados válidos en el Dashboard para evitar advertencias de seguridad.
Restricciones de Acceso: Usa reglas de firewall o configuraciones específicas para limitar quién puede acceder al Dashboard.

