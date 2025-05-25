🔐 Guía completa para usar Sealed Secrets con K3s (desde cero)
🧩 ¿Qué necesitas?
Recurso	Uso
✅ Clúster K3s funcionando	Donde se desplegará el Sealed Secrets Controller
✅ kubectl instalado	Para administrar el clúster
✅ Acceso a kubeconfig	Archivo que permite a kubectl acceder al clúster
✅ helm instalado	Para instalar el controlador con Helm
✅ kubeseal instalado	Para cifrar tus secretos fuera del clúster
✅ ansible (opcional)	Para automatizar todo fácilmente

🚀 Paso 1 – Instalar kubeseal en tu máquina local
kubeseal es una herramienta que cifra los secretos para el controlador en K3s. Necesitas instalarla en tu PC o servidor de gestión.

✅ Forma automática con Ansible
Ansible ya lo instala si usas el playbook install_sealed_secrets.yml.

Pero si quieres instalarlo manualmente:

✅ Forma manual (Linux)
bash
Copiar
Editar
# 1. Descargar
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.6/kubeseal-0.24.6-linux-amd64.tar.gz

# 2. Extraer
tar -xvf kubeseal-0.24.6-linux-amd64.tar.gz

# 3. Instalar en el sistema
sudo install -m 755 kubeseal /usr/local/bin/kubeseal

# 4. Verificar
kubeseal --version
⚙️ Paso 2 – Instalar el Sealed Secrets Controller en K3s
Este pod vive dentro del clúster y es el único que puede descifrar los secretos.

✅ Opción 1: Con Ansible (recomendado)
bash
Copiar
Editar
ansible-playbook -i inventory/hosts.ini playbooks/install_sealed_secrets.yml
✅ Opción 2: Manual con Helm
bash
Copiar
Editar
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install sealed-secrets bitnami/sealed-secrets \
  --namespace kube-system \
  --create-namespace
🔑 Paso 3 – Obtener la clave pública del Controller
Tu máquina necesita esta clave para cifrar secretos. Puedes publicarla en tu repositorio.

bash
Copiar
Editar
kubeseal --fetch-cert \
  --controller-namespace kube-system \
  --controller-name sealed-secrets \
  > sealed-secrets-public-cert.pem
🛠️ Paso 4 – Crear y cifrar un Secret
A. Crear un Secret como plantilla
Ejemplo YAML:

yaml
Copiar
Editar
apiVersion: v1
kind: Secret
metadata:
  name: grafana-admin-secret
  namespace: monitoring
type: Opaque
data:
  admin-password: YWRtaW4=   # Base64(admin)
Guárdalo como grafana-secret.yaml.

B. Cifrarlo con kubeseal
bash
Copiar
Editar
kubeseal --cert sealed-secrets-public-cert.pem --format yaml \
  < grafana-secret.yaml > grafana-secret-sealed.yaml
Ahora este archivo es seguro y puedes subirlo a GitHub.

☸️ Paso 5 – Aplicar el SealedSecret
bash
Copiar
Editar
kubectl apply -f grafana-secret-sealed.yaml
El controlador en el clúster lo descifra y crea el Secret real.

✅ ¿Dónde se usa el Secret?
Tu aplicación (como Grafana, ArgoCD, Jenkins, etc.) lo usará como siempre, por ejemplo:

yaml
Copiar
Editar
env:
  - name: ADMIN_PASSWORD
    valueFrom:
      secretKeyRef:
        name: grafana-admin-secret
        key: admin-password
📌 ¿Por qué usar Sealed Secrets?
🔐 Seguridad total: el secreto está cifrado incluso en Git.

✅ GitOps nativo: puedes aplicar todo con ArgoCD.

🔄 Automatizable con Ansible, CI/CD, etc.

