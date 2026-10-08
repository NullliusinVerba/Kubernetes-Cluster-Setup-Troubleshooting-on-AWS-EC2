# Clúster Kubernetes de Alta Resiliencia y Resolución de Incidentes (Troubleshooting) en AWS

## 1. Resumen Ejecutivo y Alcance Operativo
Este repositorio documenta la implementación, configuración avanzada, administración y resolución de incidentes en tiempo real de un clúster de Kubernetes aprovisionado desde cero sobre instancias Amazon EC2 (Ubuntu 24.04) en la nube de AWS. 

El proyecto demuestra competencias clave en la orquestación de contenedores, diseño de redes seguras, gestión de almacenamiento persistente nativo, aplicación estricta de políticas de control de acceso y mitigación de fallos críticos en producción.

---

## 2. Arquitectura de la Infraestructura
* **Entorno Cloud:** Amazon Web Services (AWS) - 3 instancias EC2 interconectadas.
* **Topología de Nodos:** 
  * `k8s-master` (Control Plane)
  * `k8s-worker1` (Nodo Trabajador 1)
  * `k8s-worker2` (Nodo Trabajador 2)
* **Sistema Operativo:** Ubuntu 24.04 LTS con optimización de recursos y kernel reforzado.
* **Orquestación:** Kubernetes v1.30 (`kubeadm`, `kubelet`, `kubectl`).
* **Runtime de Contenedores:** `containerd` configurado con soporte para `systemd cgroup`.
* **Red de Contenedores (CNI):** Calico, asegurando el enrutamiento inter-nodo y políticas de red avanzadas.

---

## 3. Procedimiento de Implementación y Hardening
1. **Preparación del Sistema Operativo:**
   * Desactivación de *swap* (requisito indispensable para la estabilidad del `kubelet`).
   * Carga de módulos de kernel requeridos (`netfilter`, entre otros).
   * Aplicación de parámetros `sysctl` para el correcto reenvío de paquetes en el puente de red (*bridge*).
2. **Aprovisionamiento y Configuración de Red:**
   * Configuración de reglas de seguridad estrictas (Security Groups y puertos específicos para API Server, Kubelet, etcd y Calico).
   * Inicialización del plano de control mediante `kubeadm` e integración del CNI Calico.
   * Unión segura de los nodos *workers* al clúster mediante tokens cifrados.

---

## 4. Almacenamiento Persistente y Gestión de Bloques (AWS EBS)
* **Integración Nativa:** Creación y asociación manual de volúmenes de almacenamiento en bloque Amazon EBS a los nodos del clúster.
* **Operación de Montaje:** Formateo de sistemas de archivos (`ext4`) a nivel de host y estructuración de directorios dedicados para garantizar la persistencia de datos ante fallas de infraestructura.

---

## 5. Seguridad y Control de Acceso (RBAC)
Para cumplir con los principios de mínimo privilegio y seguridad defensiva:
* **ServiceAccounts:** Creación de identidades de servicio aisladas dentro del clúster.
* **Roles y RoleBindings:** Implementación de políticas de control de acceso basado en roles (`RBAC`) para restringir permisos operativos (por ejemplo, otorgando exclusivamente privilegios de lectura sobre *pods* a usuarios o servicios específicos, limitando el radio de acción ante posibles compromisos de seguridad).

---

## 6. Resolución de Incidentes Críticos (Troubleshooting en Producción)
El repositorio incluye la simulación, diagnóstico y mitigación de escenarios de falla complejos:

* **Incidente 1: Errores de Ciclo de Vida e Imágenes (`ImagePullBackOff` y `CrashLoopBackOff`)**
  * *Diagnóstico:* Uso de comandos de inspección profunda (`kubectl describe pod`, análisis de logs del runtime).
  * *Mitigación:* Corrección de sintaxis en rutas de imágenes, validación de credenciales y resolución de bloqueos de red perimétrica.
* **Incidente 2: Saturación de Recursos (`OOMKilled`)**
  * *Diagnóstico:* Identificación de procesos terminados abruptamente por el kernel debido al consumo excesivo de memoria RAM sin límites definidos.
  * *Mitigación:* Establecimiento de solicitudes y límites (*requests & limits*) de recursos en los manifiestos de los contenedores.
* **Incidente 3: Estabilización y Monitorización del Metrics-Server**
  * *Desafío:* El componente de métricas presentaba fallos de comunicación debido a restricciones de red estrictas en el entorno.
  * *Solución de Ingeniería:* Aplicación de un parche avanzado utilizando `nodeSelector`, reubicación del puerto de escucha y habilitación de parámetros de conectividad segura (`--kubelet-insecure-tls` adaptado al entorno de laboratorio controlado), permitiendo la correcta ejecución de comandos de rendimiento (`kubectl top`).

---

## 7. Operación y Mantenimiento del Clúster
* **Escalabilidad Dinámica:** Ejecución de escalamiento horizontal de despliegues (*deployments*) para absorber mayores cargas de trabajo.
* **Mantenimiento Preventivo:** Aplicación de protocolos de vaciado de nodos (`kubectl drain` y `uncordon`) para garantizar la continuidad operacional y alta disponibilidad durante tareas de actualización o mantenimiento de infraestructura.

---

## 8. Evidencias Documentales
Este repositorio contiene los siguientes recursos de validación:
* `01-rbac.yaml`: Políticas de control de acceso implementadas.
* `03-metrics-server-patch.yaml`: Solución aplicada al incidente crítico de monitorización.
* `Screenshots.pdf`: Registro detallado de capturas de consola y validación de estado operativo del clúster.

*Nota: Todos los archivos pueden ser desplegados en entornos AWS compatibles siguiendo los procedimientos documentados.*
