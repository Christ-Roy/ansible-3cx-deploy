# 🚨 3CX Licence Gratuite Terminée - Solutions Alternatives

## ❌ Problème : 3CX ne propose plus de licence gratuite self-hosted

**Depuis début 2023**, 3CX a arrêté les licences gratuites on-premise :

- ❌ **Nouvelles installations** : Plus de licence gratuite self-hosted
- ⚠️ **Licences existantes** : Valides jusqu'à fin 2025 (4SC) puis 2SC en 2026
- ⚠️ **V18** : Doit upgrader vers V20 avant 31 mars 2025
- ✅ **Seule option gratuite** : StartUP Free (hébergé par 3CX uniquement)

### Coût minimal 3CX on-premise (2025):
- **4SC PRO** : $145/an (~$12/mois) - minimum pour self-hosted
- Prix augmentés en février 2025

---

## ✅ Solutions Alternatives Gratuites (Open Source)

### **🏆 1. FreePBX (RECOMMANDÉ)**

**Le meilleur remplacement pour 3CX**

#### Avantages:
- ✅ **100% Gratuit et Open Source**
- ✅ **Basé sur Asterisk** (le plus populaire)
- ✅ **Interface Web GUI** (comme 3CX)
- ✅ **Communauté énorme** + support actif
- ✅ **Installation facile** (ISO disponible)
- ✅ **Extensions illimitées** (gratuit)
- ✅ **Modules gratuits** : Voicemail, IVR, Call Recording, Conference
- ✅ **Intégrations** : CRM, SIP trunks, WebRTC

#### Inconvénients:
- Moins "clé en main" que 3CX
- Configuration plus technique
- Interface moins moderne

#### Déploiement:
```bash
# ISO FreePBX disponible
wget https://www.freepbx.org/downloads/
# Ansible role disponible
ansible-galaxy install dallen1.freepbx
```

**Équivalent Ansible:**
- Notre rôle 3CX peut être adapté pour FreePBX
- Communauté active : https://community.freepbx.org/

---

### **2. FusionPBX**

**Alternative multi-tenant basée sur FreeSWITCH**

#### Avantages:
- ✅ **Multi-tenant** (plusieurs entreprises sur 1 serveur)
- ✅ **Basé sur FreeSWITCH** (plus moderne qu'Asterisk)
- ✅ **Interface web moderne**
- ✅ **Support WebRTC natif**
- ✅ **Gratuit et Open Source**

#### Inconvénients:
- Communauté plus petite que FreePBX
- Moins de modules disponibles

#### Installation:
```bash
# Debian 12 (comme 3CX)
wget -O - https://raw.githubusercontent.com/fusionpbx/fusionpbx-install.sh/master/debian/pre-install.sh | sh
cd /usr/src/fusionpbx-install.sh/debian && ./install.sh
```

---

### **3. Asterisk (Brut)**

**Pour les experts qui veulent tout contrôler**

#### Avantages:
- ✅ **Maximum de contrôle**
- ✅ **Performance optimale**
- ✅ **Personnalisation illimitée**
- ✅ **Standard de l'industrie**

#### Inconvénients:
- ❌ **Pas d'interface GUI** (sauf si vous installez FreePBX dessus)
- ❌ **Configuration texte uniquement**
- ❌ **Courbe d'apprentissage très raide**

**Verdict:** Utilisez plutôt FreePBX qui inclut Asterisk + GUI

---

### **4. Kamailio**

**Pour les très gros volumes (carrier-grade)**

#### Avantages:
- ✅ **Milliers d'appels/seconde**
- ✅ **Très sécurisé** (TLS, auth avancée)
- ✅ **WebRTC natif**
- ✅ **Load balancing intégré**

#### Inconvénients:
- ❌ **Complexe à configurer**
- ❌ **Nécessite un développeur**
- ❌ **Pas d'interface web**

**Use case:** Opérateurs télécom, très grosses installations

---

## 📊 Comparaison : 3CX vs Alternatives

| Critère | 3CX (Payant) | FreePBX | FusionPBX | Kamailio |
|---------|--------------|---------|-----------|----------|
| **Prix** | $145/an | ✅ Gratuit | ✅ Gratuit | ✅ Gratuit |
| **Interface Web** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ❌ |
| **Facilité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐ |
| **Communauté** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Features** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Performance** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Support** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

---

## 🎯 Recommandation Finale

### **Pour remplacer 3CX : FreePBX**

**Raisons:**
1. ✅ Interface GUI similaire à 3CX
2. ✅ Gratuit à 100% (pas de limite)
3. ✅ Communauté énorme (forums actifs)
4. ✅ Documentation complète en français
5. ✅ Modules disponibles pour tout
6. ✅ Compatible avec notre infrastructure Ansible/KVM

---

## 🚀 Migration 3CX → FreePBX

### **Ce qui change:**
| Fonctionnalité | 3CX | FreePBX |
|----------------|-----|---------|
| Extensions | ✅ | ✅ Identique |
| Trunks SIP | ✅ | ✅ Identique |
| IVR | ✅ | ✅ Plus flexible |
| Voicemail | ✅ | ✅ Identique |
| Recording | ✅ | ✅ Gratuit vs payant |
| WebRTC | ✅ | ✅ Via modules |
| Mobile Apps | ✅ 3CX apps | Softphones tiers |

### **Avantages de la migration:**
- 💰 **$0 au lieu de $145/an**
- 🔓 **Pas de limitations arbitraires**
- 🛠️ **Contrôle total du système**
- 📈 **Scalabilité illimitée**

---

## 🔧 Adaptation de notre Playbook Ansible

**Notre rôle ansible-3cx-deploy peut être adapté pour FreePBX:**

```yaml
# Changements nécessaires:
1. ISO: 3cx-debian.iso → freepbx.iso
2. Wizard: /usr/sbin/3CXWizard → FreePBX web wizard
3. Config: SetupConfig.xml → FreePBX backup/restore
4. Ports: Identiques (5060 SIP, etc.)
```

**Temps d'adaptation estimé:** 2-3 heures

---

## 📚 Ressources

### FreePBX:
- Site officiel: https://www.freepbx.org/
- Download ISO: https://www.freepbx.org/downloads/
- Wiki: https://wiki.freepbx.org/
- Forums: https://community.freepbx.org/
- Modules: https://www.freepbx.org/downloads/freepbx-distro/

### FusionPBX:
- Site: https://www.fusionpbx.com/
- GitHub: https://github.com/fusionpbx/fusionpbx
- Docs: https://docs.fusionpbx.com/

### Comparaisons:
- 3CX vs FreePBX: https://getvoip.com/blog/3cx-vs-freepbx/
- Migration guide: https://community.freepbx.org/t/migration-from-3cx-to-freepbx/106374

---

## ❓ FAQ

**Q: FreePBX est-il vraiment gratuit ?**
A: Oui, 100% open source. Sangoma vend du support/hardware mais le logiciel est gratuit.

**Q: Puis-je utiliser mes téléphones 3CX avec FreePBX ?**
A: Oui ! Tout téléphone SIP fonctionne (Yealink, Grandstream, etc.)

**Q: La qualité audio est-elle identique ?**
A: Oui, même protocoles SIP/RTP. Performance identique.

**Q: Ai-je besoin de formation ?**
A: Oui, interface différente de 3CX. Comptez 1-2 jours d'apprentissage.

**Q: Le playbook Ansible fonctionnera ?**
A: Avec adaptations mineures, oui (2-3h de travail).

---

**Conclusion:** 3CX n'étant plus gratuit, **FreePBX est le meilleur choix** pour du self-hosted gratuit en 2025.
