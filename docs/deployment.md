# Procedure de deploiement

Cette procedure reconstitue les etapes du POC. Elle doit etre adaptee a la
version de Tactical RMM et a l'architecture cible. La documentation officielle
reste la reference avant toute execution.

## 1. Auditer l'hote existant

Les controles initiaux ont porte sur le systeme, les ressources, Wazuh, les
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

- l'hote dispose de ressources suffisantes pour un petit POC ;
- Wazuh utilise deja TCP 443 et plusieurs ports de collecte sur l'hote ;
- l'installation TRMM directe aurait donc ete risquee ;
- la VM permet d'isoler les services, les sauvegardes et les mises a jour.

## 2. Valider KVM

```bash
grep -Eoc '(vmx|svm)' /proc/cpuinfo
lsmod | grep kvm
ls -l /dev/kvm
lscpu | grep -E 'Model name|Virtualization|CPU\(s\)'
```

Le POC a confirme les extensions Intel VT-x, les modules `kvm` et `kvm_intel`
ainsi que la presence de `/dev/kvm`.

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

Ouvrir une nouvelle session avant de tester les groupes, puis verifier :

```bash
systemctl status libvirtd --no-pager
qemu-system-x86_64 --version
virsh --version
sudo virsh net-list --all
sudo virsh nodeinfo
```

Le POC a valide QEMU 8.2.2, libvirt 10.0.0 et le reseau NAT `default`.

## 4. Controler le reseau avant modification

```bash
ip -br addr
ip route
nmcli -f NAME,TYPE,DEVICE connection show --active
sudo netplan get
```

L'hote utilise NetworkManager et l'interface Ethernet `enp3s0`. Plusieurs
fichiers Netplan etaient presents ; aucune conversion precipitee en bridge n'a
ete realisee. Le deploiement a d'abord utilise un reseau virtuel prive pour ne
pas interrompre Wazuh ou SSH.

> `sudo netplan get` peut afficher des cles Wi-Fi en clair. Ne jamais publier sa
> sortie brute. Les cles exposees pendant le diagnostic doivent etre changees.

## 5. Creer la VM Debian 12

Dans Virt-Manager :

| Parametre | Valeur du POC |
|---|---|
| Nom | `TRMM-SRV` |
| ISO | Debian 12 amd64 |
| vCPU | 2 |
| RAM | 4096 Mio |
| Disque | 60 Gio QCOW2 |
| Reseau initial | libvirt NAT |
| Paquets invites | serveur SSH et outils standard uniquement |

Appliquer les mises a jour, definir une adresse stable et verifier DNS, NTP et
HTTPS sortant. Creer ensuite un snapshot ou une copie coherente du QCOW2 avant
l'installation de Tactical RMM.

## 6. Preparer DNS et TLS

Creer trois noms au meme niveau :

```text
rmm.homelab.example.tld
api.homelab.example.tld
mesh.homelab.example.tld
```

Verifier depuis un agent et depuis le reseau d'administration :

```bash
getent hosts rmm.homelab.example.tld
getent hosts api.homelab.example.tld
getent hosts mesh.homelab.example.tld
```

Pour le certificat wildcard, l'installateur demande un TXT sous :

```text
_acme-challenge.homelab.example.tld
```

La valeur est temporaire et ne doit jamais etre archivee. Attendre sa
propagation et la verifier depuis un resolveur public avant de poursuivre :

```bash
dig TXT _acme-challenge.homelab.example.tld @1.1.1.1
```

Le POC a finalement utilise un certificat Let's Encrypt valide fourni a
l'installateur avec son `fullchain.pem` et sa cle privee. Aucun certificat ou
fichier PEM n'est inclus dans ce depot.

## 7. Installer Tactical RMM

La documentation officielle demande un serveur vierge Debian 11/12 ou Ubuntu
22.04 LTS avec au moins 4 Gio de RAM. Toujours recuperer l'installateur depuis
le depot officiel et le relire avant execution :

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
| Cle privee | chemin local vers `privkey.pem` |

Ne pas employer `--insecure` pour un environnement durable. Conserver les
identifiants generes dans un gestionnaire de secrets, pas dans Git.

## 8. Initialiser l'application

Depuis le frontend HTTPS :

1. Creer le client du laboratoire.
2. Creer le site `HomeLab - Infrastructure`.
3. Selectionner le fuseau horaire adapte.
4. Activer l'authentification multifacteur.
5. Creer un compte nominatif distinct du compte d'urgence.
6. Appliquer le principe du moindre privilege.

## 9. Enroler les agents Windows

Le POC a enregistre deux machines Windows : un poste d'administration distant
et `DMZ_WEB`. Les controles ont inclus :

- resolution des trois noms TRMM ;
- acces TCP 443 ;
- validation de la chaine TLS ;
- verification de l'antivirus et de Smart App Control ;
- installation avec elevation de privileges ;
- apparition de l'agent et actualisation de `Last Response` ;
- test des fonctions autorisees de MeshCentral.

Ne jamais committer l'installateur genere, son jeton d'enrolement ou sa ligne de
commande complete.

## 10. Valider le service

Les criteres de recette sont :

- interface RMM accessible sans alerte TLS ;
- API et MeshCentral joignables par leur nom DNS ;
- deux agents Windows en ligne ;
- inventaire visible ;
- controle ICMP pfSense au vert ;
- historique du check alimente ;
- Wazuh toujours operationnel sur l'hote ;
- restauration de la VM documentee.

## References

- [Installation traditionnelle Tactical RMM](https://docs.tacticalrmm.com/install_server/)
- [Installation des agents](https://docs.tacticalrmm.com/install_agent/)
- [Sauvegarde Tactical RMM](https://docs.tacticalrmm.com/backup/)
- [Restauration Tactical RMM](https://docs.tacticalrmm.com/restore/)
