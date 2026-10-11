# Procédure de déploiement

Cette procédure reconstitue les étapes du POC. Elle doit être adaptée à la
version de Tactical RMM et à l'architecture cible. La documentation officielle
reste la référence avant toute exécution.

## 1. Auditer l'hôte existant

Les contrôles initiaux ont porté sur le système, les ressources, Wazuh, les
ports et la virtualisation :

```bash
hostnamectl
uname -m
nproc
free -h
df -h /
uptime

sudo systemctl is-active wazuh-manager
sudo systemctl is-active wazuh-indexer
sudo systemctl is-active wazuh-dashboard
ps -eo pid,comm,%cpu,%mem,rss --sort=-rss | head -20
sudo ss -tulpen
```

Constats principaux :

- l'hôte dispose de ressources suffisantes pour un petit POC ;
- Wazuh utilise déjà TCP 443 et plusieurs ports de collecte sur l'hôte ;
- l'installation TRMM directe aurait donc été risquée ;
- la VM permet d'isoler les services, les sauvegardes et les mises à jour.

## 2. Valider KVM

```bash
grep -Eoc '(vmx|svm)' /proc/cpuinfo
lsmod | grep kvm
ls -l /dev/kvm
lscpu | grep -E 'Model name|Virtualization|CPU\(s\)'
```

Le POC a confirmé les extensions Intel VT-x, les modules `kvm` et `kvm_intel`
ainsi que la présence de `/dev/kvm`.

## 3. Installer l'hyperviseur

```bash
sudo apt update
sudo apt install -y \
  qemu-kvm \
  libvirt-daemon-system \
  libvirt-clients \
  virt-manager \
  virtinst \
  bridge-utils

sudo usermod -aG libvirt,kvm "$USER"
```

Ouvrir une nouvelle session avant de tester les groupes, puis vérifier :

```bash
systemctl status libvirtd --no-pager
qemu-system-x86_64 --version
virsh --version
sudo virsh net-list --all
sudo virsh nodeinfo
```

Le POC a validé QEMU 8.2.2, libvirt 10.0.0 et le réseau NAT `default`.

## 4. Contrôler le réseau avant modification

```bash
ip -br addr
ip route
nmcli -f NAME,TYPE,DEVICE connection show --active
sudo netplan get
```

L'hôte utilise NetworkManager et l'interface Ethernet `enp3s0`. Plusieurs
fichiers Netplan étaient présents ; aucune conversion précipitée en bridge n'a
été réalisée. Le déploiement a d'abord utilisé un réseau virtuel privé pour ne
pas interrompre Wazuh ou SSH.

> `sudo netplan get` peut afficher des clés Wi-Fi en clair. Ne jamais publier sa
> sortie brute. Les clés exposées pendant le diagnostic doivent être changées.

## 5. Créer la VM Debian 12

Dans Virt-Manager :

| Paramètre | Valeur du POC |
|---|---|
| Nom | `TRMM-SRV` |
| ISO | Debian 12 amd64 |
| vCPU | 2 |
| RAM | 4096 Mio |
| Disque | 60 Gio QCOW2 |
| Réseau initial | libvirt NAT |
| Paquets installés sur l'invité | serveur SSH et outils standard uniquement |

Appliquer les mises à jour, définir une adresse stable et vérifier DNS, NTP et
HTTPS sortant. Créer ensuite un snapshot ou une copie cohérente du QCOW2 avant
l'installation de Tactical RMM.

## 6. Préparer DNS et TLS

Créer trois noms au même niveau :

```text
rmm.homelab.example.tld
api.homelab.example.tld
mesh.homelab.example.tld
```

Vérifier depuis un agent et depuis le réseau d'administration :

```bash
getent hosts rmm.homelab.example.tld
getent hosts api.homelab.example.tld
getent hosts mesh.homelab.example.tld
```

Pour le certificat wildcard, l'installateur demande un TXT sous :

```text
_acme-challenge.homelab.example.tld
```

La valeur est temporaire et ne doit jamais être archivée. Attendre sa
propagation et la vérifier depuis un résolveur public avant de poursuivre :

```bash
dig TXT _acme-challenge.homelab.example.tld @1.1.1.1
```

Le POC a finalement utilisé un certificat Let's Encrypt valide fourni à
l'installateur avec son `fullchain.pem` et sa clé privée. Aucun certificat ou
fichier PEM n'est inclus dans ce dépôt.

## 7. Installer Tactical RMM

La documentation officielle demande un serveur vierge Debian 11/12 ou Ubuntu
22.04 LTS avec au moins 4 Gio de RAM. Toujours récupérer l'installateur depuis
le dépôt officiel et le relire avant exécution :

```bash
wget https://raw.githubusercontent.com/amidaware/tacticalrmm/master/install.sh
chmod +x install.sh
./install.sh --use-own-cert
```

Valeurs fonctionnelles attendues :

| Question | Exemple |
|---|---|
| Backend | `api.homelab.example.tld` |
| Frontend | `rmm.homelab.example.tld` |
| MeshCentral | `mesh.homelab.example.tld` |
| Root domain | `homelab.example.tld` |
| Certificat | chemin local vers `fullchain.pem` |
| Clé privée | chemin local vers `privkey.pem` |

Ne pas employer `--insecure` pour un environnement durable. Conserver les
identifiants générés dans un gestionnaire de secrets, pas dans Git.

## 8. Initialiser l'application

Depuis le frontend HTTPS :

1. Créer le client du laboratoire.
2. Créer le site `HomeLab - Infrastructure`.
3. Sélectionner le fuseau horaire adapté.
4. Activer l'authentification multifacteur.
5. Créer un compte nominatif distinct du compte d'urgence.
6. Appliquer le principe du moindre privilège.

## 9. Enrôler les agents Windows

Le POC a enregistré deux machines Windows : un poste d'administration distant
et `DMZ_WEB`. Les contrôles ont inclus :

- résolution des trois noms TRMM ;
- accès TCP 443 ;
- validation de la chaîne TLS ;
- vérification de l'antivirus et de Smart App Control ;
- installation avec élévation de privilèges ;
- apparition de l'agent et actualisation de `Last Response` ;
- test des fonctions autorisées de MeshCentral.

Ne jamais inclure dans un commit l'installateur généré, son jeton d'enrôlement
ou sa ligne de commande complète.

## 10. Valider le service

Les critères de recette sont :

- interface RMM accessible sans alerte TLS ;
- API et MeshCentral joignables par leur nom DNS ;
- deux agents Windows en ligne ;
- inventaire visible ;
- contrôle ICMP pfSense au vert ;
- historique du check alimenté ;
- Wazuh toujours opérationnel sur l'hôte ;
- restauration de la VM documentée.

## Références

- [Installation traditionnelle Tactical RMM](https://docs.tacticalrmm.com/install_server/)
- [Installation des agents](https://docs.tacticalrmm.com/install_agent/)
- [Sauvegarde Tactical RMM](https://docs.tacticalrmm.com/backup/)
- [Restauration Tactical RMM](https://docs.tacticalrmm.com/restore/)
