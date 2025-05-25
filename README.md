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

## 🧑‍💻 Autor

Desarrollado por Victor H. Gálvez