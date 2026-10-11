# Rapport de projet - Tactical RMM dans le HomeLab

## Résumé exécutif

L'objectif était d'ajouter une capacité de Remote Monitoring and Management au
HomeLab sans perturber Wazuh, déjà exploité comme SIEM. L'étude a écarté
l'installation directe sur l'hôte Ubuntu, car l'installateur traditionnel de
Tactical RMM attend un serveur vierge et configure notamment son propre proxy
inverse.

Une VM Debian 12 a donc été déployée avec KVM/QEMU et libvirt sur l'hôte
existant. La plateforme Tactical RMM v1.5.2 utilise trois noms DNS et un
certificat wildcard Let's Encrypt obtenu par challenge DNS-01. Deux postes
Windows ont été enrôlés. Un Ping Check natif exécuté depuis `DMZ_WEB` supervise
désormais l'interface pfSense avec un intervalle de cinq minutes et une alerte
après trois échecs consécutifs.

Le POC est fonctionnel. L'agent Linux, l'envoi SMTP, le renouvellement TLS
automatisé et la supervision avancée de pfSense restent explicitement hors de la
validation actuelle.

## Besoin initial

Le HomeLab comprend plusieurs zones administrées par pfSense :

- VLAN 20 ADMIN ;
- VLAN 30 DEV ;
- VLAN 40 KALI ;
- accès distant WireGuard ;
- Wazuh All-in-One sur le serveur Ubuntu Admin.

Le besoin fonctionnel était de centraliser la disponibilité, l'inventaire,
l'état des ressources et les actions de maintenance, tout en conservant Wazuh
pour la détection et l'investigation de sécurité.

## Étude des options

| Option | Analyse | Décision |
|---|---|---|
| Installation TRMM directe sur Ubuntu | conflits possibles avec Wazuh, Nginx et TCP 443 ; serveur non vierge | Rejetée |
| Conteneurs Docker sur l'hôte | isolation partielle et exploitation plus complexe pour ce POC | Non retenue |
| VM dédiée sur le même hôte | isolation, snapshot, restauration et OS compatible | Retenue |

## Audit de faisabilité

L'hôte dispose de quatre cœurs, 15 Gio de RAM et d'un stockage largement
disponible. Wazuh était actif et stable ; son indexeur Java représentait le
principal consommateur de mémoire. Le support VT-x, les modules KVM et
`/dev/kvm` ont été confirmés.

L'allocation initiale de la VM a été volontairement limitée à deux vCPU, 4 Gio
de RAM et 60 Gio de disque afin de laisser une marge au SIEM et à l'hôte.

## Mise en œuvre

### Virtualisation

KVM/QEMU, libvirt, Virt-Manager et les outils clients ont été installés. Le
service libvirt et le réseau NAT par défaut ont été validés avant la création de
la VM Debian 12 minimale.

### Réseau

La configuration de l'hôte repose sur NetworkManager et `enp3s0`. La présence
de plusieurs fichiers Netplan a conduit à éviter une bascule immédiate vers un
bridge physique. Un réseau virtuel privé a permis de préparer la VM sans couper
SSH, le Dashboard Wazuh ou les flux inter-VLAN existants.

### DNS et TLS

Trois noms ont été réservés pour le frontend, l'API et MeshCentral. Plusieurs
incidents de résolution et de validation ACME ont montré l'importance de
distinguer DNS interne et DNS public autoritaire. Le certificat final est un
wildcard Let's Encrypt obtenu par DNS-01 puis fourni à l'installateur.

### Installation applicative

L'installation a été effectuée sur la VM dédiée avec les chemins du certificat
et de la clé. Une interruption a nécessité la restauration de l'image QCOW2,
ce qui a validé l'utilité du point de retour pris avant l'opération.

### Enrôlement

Le client et le site du HomeLab ont été créés. Deux agents Windows sont apparus
en ligne après vérification du DNS, de TLS, des protections des postes et de la
connectivité MeshCentral.

## Supervision de pfSense

Un test `Test-Connection` depuis `DMZ_WEB` a confirmé quatre réponses ICMP sans
perte. Le check natif a ensuite été configuré :

| Paramètre | Valeur |
|---|---|
| Nom | `PFSENSE - ICMP Availability` |
| Cible | `192.168.20.1` |
| Intervalle | 300 secondes |
| Sévérité | Warning |
| Seuil | 3 échecs consécutifs |

L'exécution forcée a retourné trois paquets transmis, trois reçus et 0 % de
perte.

## Incidents et enseignements

| Incident | Enseignement |
|---|---|
| DNS public non résolu | vérifier la zone autoritaire, pas seulement le DNS local |
| Challenge ACME en NXDOMAIN | contrôler le FQDN produit par l'interface du fournisseur |
| Erreur Certbot | vérifier les versions, les journaux et la préproduction avant la production |
| Installation interrompue | toujours créer et tester un point de restauration |
| Agent Windows bloqué | diagnostiquer DNS, TLS et EDR avant toute exception |
| Agent Linux refusé | respecter le modèle officiel de signature et de parrainage |
| Script PowerShell sans prérequis | déclarer l'interpréteur, la version et le retour arrière |

## Résultat

Le HomeLab dispose maintenant d'une plateforme RMM isolée et accessible en
HTTPS, de deux postes Windows enrôlés et d'une première supervision réseau de
pfSense. Le serveur Wazuh est resté séparé et opérationnel.

Le résultat démontre une démarche complète : audit, choix d'architecture,
virtualisation, DNS/TLS, installation, restauration, enrôlement, supervision et
documentation des limites.

## Prochaines étapes

1. Configurer SMTP et un Alert Template pour e-mail et pop-up Dashboard.
2. Tester l'alerte sur une cible factice, sans interrompre pfSense.
3. Automatiser et tester le renouvellement DNS-01.
4. Ajouter des checks Wazuh, DNS et HTTPS.
5. Ajouter une seconde sonde pour réduire la dépendance à `DMZ_WEB`.
6. Évaluer SNMPv3 et l'intégration des alertes TRMM dans Wazuh.
7. Formaliser un plan de sauvegarde et de reprise avec RTO/RPO.
