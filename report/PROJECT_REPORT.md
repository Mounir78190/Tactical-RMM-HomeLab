# Rapport de projet - Tactical RMM dans le HomeLab

## Resume executif

L'objectif etait d'ajouter une capacite de Remote Monitoring and Management au
HomeLab sans perturber Wazuh, deja exploite comme SIEM. L'etude a ecarte
l'installation directe sur l'hote Ubuntu, car l'installateur traditionnel de
Tactical RMM attend un serveur vierge et configure notamment son propre reverse
proxy.

Une VM Debian 12 a donc ete deployee avec KVM/QEMU et libvirt sur l'hote
existant. La plateforme Tactical RMM v1.5.2 utilise trois noms DNS et un
certificat wildcard Let's Encrypt obtenu par challenge DNS-01. Deux endpoints
Windows ont ete enroles. Un Ping Check natif execute depuis `DMZ_WEB` supervise
desormais l'interface pfSense avec un intervalle de cinq minutes et une alerte
apres trois echecs consecutifs.

Le POC est fonctionnel. L'agent Linux, l'envoi SMTP, le renouvellement TLS
automatise et la supervision avancee pfSense restent explicitement hors de la
validation actuelle.

## Besoin initial

Le HomeLab comprend plusieurs zones administrees par pfSense :

- VLAN 20 ADMIN ;
- VLAN 30 DEV ;
- VLAN 40 KALI ;
- acces distant WireGuard ;
- Wazuh All-in-One sur le serveur Ubuntu Admin.

Le besoin fonctionnel etait de centraliser la disponibilite, l'inventaire,
l'etat des ressources et les actions de maintenance, tout en conservant Wazuh
pour la detection et l'investigation de securite.

## Etude des options

| Option | Analyse | Decision |
|---|---|---|
| Installation TRMM directe sur Ubuntu | conflits possibles avec Wazuh, Nginx et TCP 443 ; serveur non vierge | Rejetee |
| Conteneurs Docker sur l'hote | isolation partielle et exploitation plus complexe pour ce POC | Non retenue |
| VM dediee sur le meme hote | isolation, snapshot, restauration et OS compatible | Retenue |

## Audit de faisabilite

L'hote dispose de quatre coeurs, 15 Gio de RAM et d'un stockage largement
disponible. Wazuh etait actif et stable ; son indexeur Java representait le
principal consommateur de memoire. Le support VT-x, les modules KVM et
`/dev/kvm` ont ete confirmes.

L'allocation initiale de la VM a ete volontairement limitee a deux vCPU, 4 Gio
de RAM et 60 Gio de disque afin de laisser une marge au SIEM et a l'hote.

## Mise en oeuvre

### Virtualisation

KVM/QEMU, libvirt, Virt-Manager et les outils clients ont ete installes. Le
service libvirt et le reseau NAT par defaut ont ete valides avant la creation de
la VM Debian 12 minimale.

### Reseau

La configuration de l'hote repose sur NetworkManager et `enp3s0`. La presence
de plusieurs fichiers Netplan a conduit a eviter une bascule immediate vers un
bridge physique. Un reseau virtuel prive a permis de preparer la VM sans couper
SSH, le Dashboard Wazuh ou les flux inter-VLAN existants.

### DNS et TLS

Trois noms ont ete reserves pour le frontend, l'API et MeshCentral. Plusieurs
incidents de resolution et de validation ACME ont montre l'importance de
distinguer DNS interne et DNS public autoritaire. Le certificat final est un
wildcard Let's Encrypt obtenu par DNS-01 puis fourni a l'installateur.

### Installation applicative

L'installation a ete effectuee sur la VM dediee avec les chemins du certificat
et de la cle. Une interruption a necessite la restauration de l'image QCOW2,
ce qui a valide l'utilite du point de retour pris avant l'operation.

### Enrolement

Le client et le site du HomeLab ont ete crees. Deux agents Windows sont apparus
en ligne apres verification du DNS, de TLS, des protections endpoint et de la
connectivite MeshCentral.

## Supervision de pfSense

Un test `Test-Connection` depuis `DMZ_WEB` a confirme quatre reponses ICMP sans
perte. Le check natif a ensuite ete configure :

| Parametre | Valeur |
|---|---|
| Nom | `PFSENSE - ICMP Availability` |
| Cible | `192.168.20.1` |
| Intervalle | 300 secondes |
| Severite | Warning |
| Seuil | 3 echecs consecutifs |

L'execution forcee a retourne trois paquets transmis, trois recus et 0 % de
perte.

## Incidents et enseignements

| Incident | Enseignement |
|---|---|
| DNS public non resolu | verifier la zone autoritaire, pas seulement le DNS local |
| Challenge ACME en NXDOMAIN | controler le FQDN produit par l'interface du fournisseur |
| Erreur Certbot | verifier versions, logs et staging avant production |
| Installation interrompue | toujours creer et tester un point de restauration |
| Agent Windows bloque | diagnostiquer DNS, TLS et EDR avant toute exception |
| Agent Linux refuse | respecter le modele de signature et de sponsoring officiel |
| Script PowerShell sans prerequis | declarer interpreteur, version et rollback |

## Resultat

Le HomeLab dispose maintenant d'une plateforme RMM isolee et accessible en
HTTPS, de deux endpoints Windows enroles et d'un premier monitoring reseau de
pfSense. Le serveur Wazuh est reste separe et operationnel.

Le resultat demontre une demarche complete : audit, choix d'architecture,
virtualisation, DNS/TLS, installation, restauration, enrolement, monitoring et
documentation des limites.

## Prochaines etapes

1. Configurer SMTP et un Alert Template pour e-mail et pop-up Dashboard.
2. Tester l'alerte sur une cible factice, sans interrompre pfSense.
3. Automatiser et tester le renouvellement DNS-01.
4. Ajouter des checks Wazuh, DNS et HTTPS.
5. Ajouter une seconde sonde pour reduire la dependance a `DMZ_WEB`.
6. Evaluer SNMPv3 et l'integration des alertes TRMM dans Wazuh.
7. Formaliser un plan de sauvegarde et de reprise avec RTO/RPO.
