# 🔐 k3s-secrets-cluster

Automatización completa para instalar y configurar **Sealed Secrets** de Bitnami en un clúster **K3s** usando **Ansible**. Esta herramienta permite gestionar secretos cifrados seguros dentro de Kubernetes, compatibles con flujos GitOps y control de versiones en Git.

---

## 📦 Características

- Instalación de **Sealed Secrets Controller** vía Helm.
- Plantillas de secretos (`Opaque`) para cifrado.
- Generación de manifiestos sellados con `kubeseal`.
- Reutilización de secretos en Jenkins, ArgoCD, Prometheus, Grafana, etc.
- Compatible con arquitectura HA de K3s.
- Preparado para CI/CD y uso con ArgoCD y GitHub.

---

## 🚀 Requisitos

- Ansible >= 2.14
- Helm y kubectl instalados en el nodo de control
- Acceso válido al clúster K3s vía kubeconfig
- kubeseal instalado localmente

---

## 🧪 Instalación del Controller

Ejecutar el siguiente comando para instalar el Sealed Secrets Controller:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/install_sealed_secrets.yml
```

---

## ✍️ Ejemplo de Cifrado Manual

### Obtener el certificado público del controlador

```bash
kubeseal --fetch-cert \
  --controller-name=sealed-secrets \
  --controller-namespace=kube-system \
  > pub-cert.pem
```

### Renderizar el secreto con Ansible

```bash
ansible -i inventory/hosts.ini localhost \
  -m template -a "src=templates/secret-example.yaml.j2 dest=/tmp/my-secret.yaml" \
  -e "secret_name=my-basic-auth secret_namespace=default password=admin123"
```

### Cifrar con kubeseal

```bash
kubeseal --cert pub-cert.pem --format yaml < /tmp/my-secret.yaml > my-secret-sealed.yaml
```

### Aplicar en el clúster

```bash
kubectl apply -f my-secret-sealed.yaml
```

---

## 💡 Casos de Uso

- Proteger accesos de Jenkins, ArgoCD, Grafana, etc.
- Seguridad para contraseñas, tokens y credenciales.
- Flujo GitOps con secretos versionados sin exponer datos sensibles.

---

## 📘 Referencias

- [Bitnami Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- [Kubeseal CLI](https://github.com/bitnami-labs/sealed-secrets#kubeseal)

---

## ⚠️ Notas de Seguridad

- Nunca guardes Secret sin cifrar en repositorios públicos.
- El archivo `pub-cert.pem` puede publicarse, pero nunca la clave privada.

---
✅ PASOS TEÓRICOS ORDENADOS
1️⃣ Instalar Sealed Secrets Controller (una sola vez)
🔧 Dónde: Dentro del clúster Kubernetes (K3s)
🧰 Cómo: Usando Helm y un playbook de Ansible
🎯 Qué hace: Despliega un pod que contiene las claves para descifrar secretos

2️⃣ Obtener la clave pública del controlador
📜 Por qué: Para cifrar secretos desde tu máquina o CI sin exponer claves privadas
🛠️ Comando:

bash
Copiar
Editar
kubeseal --fetch-cert --controller-namespace=kube-system > sealed-secrets-public-cert.pem
Esta clave se usa para cifrar, pero solo el clúster puede descifrar.

3️⃣ Crear un Secret Kubernetes como plantilla (NO aplicar)
Ejemplo:

yaml
Copiar
Editar
apiVersion: v1
kind: Secret
metadata:
  name: traefik-dashboard-secret
  namespace: kube-system
type: Opaque
data:
  traefik-dashboard-user: <base64>
  traefik-dashboard-pass: <base64>
Esto representa el secreto que quieres proteger.

4️⃣ Sellar el secreto (convertirlo en SealedSecret)
⚙️ Herramienta: kubeseal
🔒 Comando:

bash
Copiar
Editar
kubeseal --format=yaml \
  --cert sealed-secrets-public-cert.pem \
  < my-secret.yaml > my-sealed-secret.yaml
Ahora tienes un SealedSecret seguro que puedes subir a Git.

5️⃣ Subir el SealedSecret al repositorio Git
📦 Este archivo my-sealed-secret.yaml es seguro para GitHub, GitLab o cualquier CI/CD
🎯 Será aplicado por ArgoCD automáticamente

6️⃣ ArgoCD aplica el SealedSecret y lo convierte en un Secret real
🧠 El controlador dentro de K3s detecta los SealedSecret, los descifra, y crea un Secret real automáticamente.

7️⃣ Tu servicio (ej: Traefik) lo usa normalmente
En values.yaml o ingress, usas el secreto como siempre:

yaml
Copiar
Editar
traefik.ingress.kubernetes.io/auth-secret: traefik-dashboard-secret
🧩 EJEMPLOS DE USO TÍPICO
jenkins-admin-secret

grafana-admin-secret

argocd-secret

traefik-dashboard-secret

webhook-github-token

smtp-password-secret

🔒 VENTAJAS
Puedes hacer GitOps seguro (guardar secretos en Git sin riesgo)

Solo el clúster puede leer los secretos

Integración nativa con ArgoCD, Jenkins, etc.

## 🧑‍💻 Autor

Desarrollado por Victor H. Gálvez