# 🔐 k3s-secrets-cluster

Automatización completa para instalar y configurar **Sealed Secrets** de Bitnami en un clúcster **K3s** usando **Ansible**. Esta herramienta permite gestionar secretos cifrados seguros dentro de Kubernetes, compatibles con flujos GitOps y control de versiones en Git.

---

## 📦 Características

* Instalación de **Sealed Secrets Controller** vía Helm.
* Plantillas de secretos (`Opaque`) para cifrado.
* Generación de manifiestos sellados con `kubeseal`.
* Reutilización de secretos en Jenkins, ArgoCD, Prometheus, Grafana, etc.
* Compatible con arquitectura HA de K3s.
* Preparado para CI/CD y uso con ArgoCD y GitHub.

---

## 🗂️ Estructura Recomendada

```
k3s-secrets-cluster/
├── group_vars/
│   └── all.yml
├── inventory/
│   └── hosts.ini
├── playbooks/
│   └── install_sealed_secrets.yml
├── roles/
│   ├── sealed_secrets/
│   │   └── tasks/main.yml
│   └── kubeseal_installer/
│       └── tasks/main.yml
├── templates/
│   └── secret-example.yaml.j2
└── README.md
```

---

## 🚀 Instalación del Controller y Kubeseal

Ejecutar:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/install_sealed_secrets.yml
```

Esto instala el controlador en el clúcster y la herramienta `kubeseal` localmente.

---

## 🔧 Ejemplo de Cifrado Manual

### 1. Obtener el certificado público

```bash
kubeseal --fetch-cert \
  --controller-name=sealed-secrets \
  --controller-namespace=kube-system > pub-cert.pem
```

### 2. Crear plantilla de Secret

Archivo: `templates/secret-example.yaml.j2`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ secret_name }}
  namespace: {{ secret_namespace }}
type: Opaque
data:
  password: {{ password | b64encode }}
```

### 3. Renderizar y cifrar

```bash
ansible -i inventory/hosts.ini localhost \
  -m template -a "src=templates/secret-example.yaml.j2 dest=/tmp/my-secret.yaml" \
  -e "secret_name=admin-secret secret_namespace=default password=MiPassword"

kubeseal --cert pub-cert.pem --format yaml < /tmp/my-secret.yaml > my-secret-sealed.yaml
```

### 4. Aplicar al clúcster

```bash
kubectl apply -f my-secret-sealed.yaml
```

---

## 🔄 Flujo GitOps con ArgoCD

1. Instala `sealed-secrets` controller con Ansible + Helm.
2. Obtén `pub-cert.pem` y cifra secretos localmente.
3. Versiona `my-secret-sealed.yaml` en Git.
4. ArgoCD detecta cambios y aplica el manifiesto.
5. El controlador crea el Secret real dentro del clúcster.
6. Tu app lo consume normalmente.

---

## 🔐 Integración con Ingress

Traefik, por ejemplo, puede usar un Secret cifrado:

```yaml
metadata:
  annotations:
    traefik.ingress.kubernetes.io/auth-secret: traefik-dashboard-secret
```

---

## 🧩 Secretos Típicos a Cifrar

* `jenkins-admin-secret`
* `grafana-admin-secret`
* `argocd-secret`
* `traefik-dashboard-secret`
* `smtp-password-secret`

---

ansible localhost -m template \
  -a "src=templates/secret-example.yaml.j2 dest=jenkins-secret.yaml" \
  -e '{"secret_name": "jenkins-admin-secret", "secret_namespace": "kube-system", "secret_data": {"username": "admin", "password": "s3cret"}}'



## 🛡️ Seguridad y Buenas Prácticas

* Solo almacena en Git los `SealedSecret`, no `Secret` normales.
* Publica `pub-cert.pem`, **nunca** la clave privada.
* Usa Sealed Secrets desde el inicio para servicios internos (ingress, dashboards).

---

## 📘 Referencias

* [https://github.com/bitnami-labs/sealed-secrets](https://github.com/bitnami-labs/sealed-secrets)
* [https://github.com/bitnami-labs/sealed-secrets#kubeseal](https://github.com/bitnami-labs/sealed-secrets#kubeseal)

---

## 👨‍💻 Autor

Desarrollado por Victor H. Gálvez



