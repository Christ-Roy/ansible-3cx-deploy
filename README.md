# 3CX Automated Deployment with Ansible

Rôle Ansible pour déployer automatiquement 3CX PBX sur KVM/libvirt avec configuration complète via `SetupConfig.xml`.

## 🚀 Fonctionnalités

- ✅ Création automatique de VM KVM
- ✅ Installation Debian 12 automatisée (preseed)
- ✅ Installation 3CX automatique
- ✅ Configuration complète via SetupConfig.xml
- ✅ Vérification de l'état des services
- ✅ Gestion complète du cycle de déploiement

## 📋 Prérequis

### Packages requis sur l'hôte KVM

```bash
sudo apt install -y \
  libvirt-daemon \
  libvirt-clients \
  python3-libvirt \
  qemu-kvm \
  virtinst \
  genisoimage \
  sshpass
```

### Collections Ansible

```bash
ansible-galaxy collection install community.libvirt
```

## 📁 Structure du rôle

```
ansible-3cx-deploy/
├── defaults/
│   └── main.yml              # Variables par défaut
├── tasks/
│   ├── main.yml              # Point d'entrée principal
│   ├── preflight.yml         # Vérifications préalables
│   ├── create_vm.yml         # Création de la VM
│   ├── wait_install.yml      # Attente fin d'installation
│   └── configure_3cx.yml     # Configuration 3CX
├── meta/
│   └── main.yml              # Métadonnées du rôle
├── playbook.yml              # Playbook d'exemple
├── inventory                 # Fichier d'inventaire
└── ansible.cfg               # Configuration Ansible
```

## ⚙️ Configuration

### 1. Préparer votre SetupConfig.xml

Créez ou utilisez votre fichier `SetupConfig.xml` existant avec vos paramètres 3CX.

Exemple: `/home/brunon5/___3CX@Admin2025/SetupConfig.xml`

### 2. Configurer les variables

Éditez `defaults/main.yml` ou passez les variables dans le playbook :

```yaml
# Configuration VM
vm_name: "3cx-veridian"
vm_memory: 4096              # MB
vm_vcpus: 2
vm_disk_size: 40             # GB
vm_ip_address: "192.168.122.93"

# Chemins
iso_path: "/home/brunon5/ISOs/3cx-debian.iso"
setupconfig_source: "/home/brunon5/___3CX@Admin2025/SetupConfig.xml"

# Credentials SSH (définis pendant l'install Debian)
ssh_user: "root"
ssh_password: "ChangeMe123!"
```

### 3. Ajuster le réseau

Choisissez le type de réseau :

**Option A - Réseau NAT par défaut :**
```yaml
vm_network: "default"
vm_network_type: "network"
```

**Option B - Bridge (accès direct au réseau) :**
```yaml
vm_network: "br0"
vm_network_type: "bridge"
```

## 🎯 Utilisation

### Déploiement standard

```bash
cd /home/brunon5/ansible-3cx-deploy
ansible-playbook -i inventory playbook.yml
```

### Avec variables personnalisées

```bash
ansible-playbook -i inventory playbook.yml \
  -e "vm_name=3cx-prod" \
  -e "vm_ip_address=192.168.1.100" \
  -e "vm_memory=8192"
```

### Mode verbose (debug)

```bash
ansible-playbook -i inventory playbook.yml -vvv
```

## 📊 Processus de déploiement

Le rôle exécute les étapes suivantes :

1. **Pré-vérifications** (1-2 min)
   - Vérification ISO et SetupConfig.xml
   - Installation des packages requis
   - Nettoyage des VM existantes

2. **Création VM** (2-3 min)
   - Création du disque virtuel
   - Lancement de la VM avec l'ISO
   - Démarrage de l'installation automatisée

3. **Installation** (15-30 min)
   - Installation Debian 12 (preseed automatique)
   - Reboot automatique
   - Installation 3CX
   - Démarrage des services

4. **Configuration** (2-5 min)
   - Copie du SetupConfig.xml
   - Exécution du wizard 3CX
   - Vérification des services
   - Test de l'interface web

**Durée totale estimée : 20-40 minutes**

## 🔍 Vérification post-déploiement

### Vérifier l'état de la VM

```bash
sudo virsh list --all
sudo virsh dominfo 3cx-veridian
```

### Se connecter à la VM

```bash
ssh root@192.168.122.93
```

### Vérifier les services 3CX

```bash
ssh root@192.168.122.93 'systemctl status 3cxpbx'
```

### Accéder à l'interface web

- Interface utilisateur : `https://192.168.122.93`
- Interface admin : `https://192.168.122.93:5001`

## 🛠️ Troubleshooting

### La VM ne démarre pas

```bash
# Vérifier les logs libvirt
sudo journalctl -u libvirtd -f

# Vérifier la console de la VM
sudo virsh console 3cx-veridian
```

### L'installation prend trop de temps

```bash
# Augmenter le timeout
ansible-playbook -i inventory playbook.yml \
  -e "install_timeout=3600"
```

### Impossible de se connecter en SSH

1. Vérifier que la VM a bien l'IP configurée :
   ```bash
   sudo virsh domifaddr 3cx-veridian
   ```

2. Vérifier le mot de passe SSH configuré

3. Tester manuellement :
   ```bash
   ssh -o StrictHostKeyChecking=no root@192.168.122.93
   ```

### Le wizard 3CX échoue

1. Vérifier le SetupConfig.xml :
   ```bash
   xmllint --noout /home/brunon5/___3CX@Admin2025/SetupConfig.xml
   ```

2. Se connecter à la VM et lancer manuellement :
   ```bash
   ssh root@192.168.122.93
   /usr/sbin/3CXWizard --setupconfig /etc/3cxpbx/setupconfig.xml
   ```

## 🔄 Redéploiement

Pour redéployer une VM existante :

```bash
# Le rôle détruit automatiquement la VM existante
ansible-playbook -i inventory playbook.yml

# Ou forcer manuellement :
sudo virsh destroy 3cx-veridian
sudo virsh undefine 3cx-veridian
sudo rm /var/lib/libvirt/images/3cx-veridian.qcow2
ansible-playbook -i inventory playbook.yml
```

## 📝 Variables disponibles

| Variable | Défaut | Description |
|----------|--------|-------------|
| `vm_name` | `3cx-veridian` | Nom de la VM |
| `vm_memory` | `4096` | RAM en MB |
| `vm_vcpus` | `2` | Nombre de vCPUs |
| `vm_disk_size` | `40` | Taille disque en GB |
| `vm_ip_address` | `192.168.122.93` | Adresse IP de la VM |
| `iso_path` | - | Chemin vers l'ISO 3CX Debian |
| `setupconfig_source` | - | Chemin vers SetupConfig.xml |
| `ssh_user` | `root` | Utilisateur SSH |
| `ssh_password` | `ChangeMe123!` | Mot de passe SSH |
| `install_timeout` | `1800` | Timeout installation (sec) |
| `post_install_wait` | `120` | Attente post-install (sec) |

## 🔒 Sécurité

⚠️ **Important :**

1. Changez le mot de passe SSH par défaut
2. Utilisez Ansible Vault pour les credentials sensibles :
   ```bash
   ansible-vault encrypt_string 'MySecurePassword' --name 'ssh_password'
   ```
3. Limitez l'accès réseau à la VM
4. Configurez un firewall sur la VM 3CX

## 📦 Exemple de déploiement avec Vault

```bash
# Créer un fichier vault
ansible-vault create vars/secrets.yml

# Contenu du fichier secrets.yml :
ssh_password: "SecurePassword123!"
setupconfig_source: "/path/to/SetupConfig.xml"

# Lancer avec vault
ansible-playbook -i inventory playbook.yml \
  --vault-password-file ~/.vault_pass
```

## 🤝 Support

Pour toute question ou problème :
- Vérifiez les logs : `/var/log/3cxpbx/`
- Documentation 3CX : https://www.3cx.com/docs/
- Forums 3CX : https://www.3cx.com/community/

## 📄 Licence

MIT

## 👤 Auteur

Robert Brunon - Veridian
