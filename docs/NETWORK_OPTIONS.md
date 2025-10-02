# Options réseau pour 3CX VM

## 🌐 Problème actuel

Par défaut, le playbook utilise `--network network=default` qui:
- ✅ Crée la VM avec une IP NAT interne (192.168.122.x)
- ❌ N'est PAS accessible depuis l'extérieur de l'hôte
- ❌ Pas de port forwarding configuré

## 🔧 Solutions

### **Option A: Bridge Network (RECOMMANDÉ pour production)**

La VM obtient une IP sur votre réseau local, directement accessible.

#### Configuration:

1. **Créer un bridge sur l'hôte:**
```bash
# Voir les bridges existants
sudo virsh net-list --all

# Créer un bridge (si pas déjà fait)
sudo nmcli connection add type bridge ifname br0
sudo nmcli connection add type bridge-slave ifname eth0 master br0
```

2. **Modifier le playbook:**
```yaml
vars:
  vm_network: "br0"           # Nom de votre bridge
  vm_network_type: "bridge"   # Type: bridge au lieu de network
  vm_ip_address: "192.168.1.100"  # IP sur votre LAN
```

3. **Lancer le déploiement:**
```bash
ansible-playbook -i inventory playbook.yml
```

✅ **Résultat:** VM accessible depuis tout le réseau local via 192.168.1.100

---

### **Option B: NAT avec Port Forwarding**

Garde l'IP interne mais redirige les ports vers l'hôte.

#### Ports 3CX à exposer:
- **5015** - Web configuration wizard
- **5001** - Web admin interface
- **5000** - Web client
- **443** - HTTPS web interface
- **80** - HTTP redirect
- **5060** - SIP (UDP/TCP)
- **5061** - SIP TLS
- **9000-10999** - RTP (media/audio)

#### Configuration automatique:

Ajoutez à `tasks/create_vm.yml` après la création de VM:

```yaml
- name: Setup NAT port forwarding for 3CX
  ansible.builtin.shell: |
    # Web interfaces
    iptables -t nat -I PREROUTING -p tcp --dport 5015 -j DNAT --to {{ vm_ip_address }}:5015
    iptables -t nat -I PREROUTING -p tcp --dport 5001 -j DNAT --to {{ vm_ip_address }}:5001
    iptables -t nat -I PREROUTING -p tcp --dport 5000 -j DNAT --to {{ vm_ip_address }}:5000
    iptables -t nat -I PREROUTING -p tcp --dport 443 -j DNAT --to {{ vm_ip_address }}:443
    iptables -t nat -I PREROUTING -p tcp --dport 80 -j DNAT --to {{ vm_ip_address }}:80

    # SIP
    iptables -t nat -I PREROUTING -p tcp --dport 5060 -j DNAT --to {{ vm_ip_address }}:5060
    iptables -t nat -I PREROUTING -p udp --dport 5060 -j DNAT --to {{ vm_ip_address }}:5060
    iptables -t nat -I PREROUTING -p tcp --dport 5061 -j DNAT --to {{ vm_ip_address }}:5061

    # RTP Media
    iptables -t nat -I PREROUTING -p udp --dport 9000:10999 -j DNAT --to {{ vm_ip_address }}

    # Save rules
    iptables-save > /etc/iptables/rules.v4
  become: true
  when: vm_network_type == "network"
```

✅ **Résultat:** Accès via IP de l'hôte, ports redirigés vers la VM

---

### **Option C: Libvirt NAT Hooks (Persistant)**

Configuration via libvirt hooks pour rendre le port forwarding permanent.

#### 1. Créer le hook script:
```bash
sudo mkdir -p /etc/libvirt/hooks
sudo nano /etc/libvirt/hooks/qemu
```

```bash
#!/bin/bash
VM_NAME="3cx-veridian"
VM_IP="192.168.122.93"

if [ "$1" = "$VM_NAME" ]; then
  if [ "$2" = "started" ] || [ "$2" = "reconnect" ]; then
    # Web interfaces
    iptables -t nat -A PREROUTING -p tcp --dport 5015 -j DNAT --to $VM_IP:5015
    iptables -t nat -A PREROUTING -p tcp --dport 5001 -j DNAT --to $VM_IP:5001
    iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to $VM_IP:443

    # SIP
    iptables -t nat -A PREROUTING -p tcp --dport 5060 -j DNAT --to $VM_IP:5060
    iptables -t nat -A PREROUTING -p udp --dport 5060 -j DNAT --to $VM_IP:5060

    # RTP
    iptables -t nat -A PREROUTING -p udp --dport 9000:10999 -j DNAT --to $VM_IP

    iptables -A FORWARD -d $VM_IP -j ACCEPT
  fi

  if [ "$2" = "stopped" ] || [ "$2" = "release" ]; then
    # Cleanup rules
    iptables -t nat -D PREROUTING -p tcp --dport 5015 -j DNAT --to $VM_IP:5015 2>/dev/null
    # ... (répéter pour tous les ports)
  fi
fi
```

```bash
sudo chmod +x /etc/libvirt/hooks/qemu
sudo systemctl restart libvirtd
```

---

## 📊 Comparaison des options

| Option | Simplicité | Performance | Production | Use Case |
|--------|-----------|-------------|------------|----------|
| **Bridge** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ | LAN accessible |
| **NAT + iptables** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⚠️ | Test/Dev |
| **Libvirt Hooks** | ⭐⭐ | ⭐⭐⭐⭐ | ✅ | NAT persistant |

---

## 🎯 Recommandation finale

**Pour production:** Utilisez **Bridge Network** (Option A)
- Plus simple à maintenir
- Meilleures performances réseau
- Pas de gestion de règles iptables

**Pour test/dev:** Utilisez **NAT avec port forwarding** (Option B)
- Isolation réseau
- Rapide à configurer
- Pas besoin de modifier la config réseau de l'hôte
