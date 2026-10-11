# Tactical RMM dans un HomeLab segmenté

<p align="center">
  <img src="report/cover.svg" alt="Tactical RMM HomeLab" width="620">
</p>

[![RMM](https://img.shields.io/badge/RMM-Tactical%20RMM-16a3c7)](https://github.com/amidaware/tacticalrmm)
[![Version](https://img.shields.io/badge/TRMM-v1.5.2-334155)](VERSION)
[![Hyperviseur](https://img.shields.io/badge/Hyperviseur-KVM%20%2F%20libvirt-7c3aed)](docs/architecture.md)
[![Statut](https://img.shields.io/badge/Statut-POC%20fonctionnel-16a34a)](report/PROJECT_REPORT.md)
[![Classification](https://img.shields.io/badge/TLP-CLEAR-ffffff)](docs/security.md)
[![Licence](https://img.shields.io/badge/Licence-MIT-green)](LICENSE)

## Présentation

Ce dépôt documente l'intégration de Tactical RMM dans un HomeLab protégé par
pfSense et segmenté en VLAN. Le serveur RMM est isolé dans une VM Debian 12 sur
KVM/libvirt afin de préserver le serveur Wazuh déjà en production sur l'hôte
Ubuntu.

Le POC valide l'accès HTTPS, l'enrôlement de deux agents Windows et la
supervision indirecte de pfSense par un contrôle ICMP exécuté depuis la DMZ.
Wazuh reste le SIEM du laboratoire ; Tactical RMM assure la supervision
opérationnelle et l'administration des postes clients.

![Agents Windows supervisés](evidence/screenshots/01-windows-agents.png)

## Synthèse

| Élément | Valeur |
|---|---|
| Projet | `Tactical RMM HomeLab` |
| Solution | Tactical RMM v1.5.2 |
| Virtualisation | KVM/QEMU et libvirt sur Ubuntu 24.04 LTS |
| VM | Debian 12, 2 vCPU, 4 Gio de RAM, disque QCOW2 de 60 Gio |
| Sécurité réseau | pfSense, VLAN, routage contrôlé et accès WireGuard |
| TLS | Certificat Let's Encrypt wildcard par challenge DNS-01 |
| Postes validés | 2 agents Windows |
| Supervision sans agent | Disponibilité ICMP de pfSense |
| Statut | POC fonctionnel, notifications e-mail à finaliser |
| Classification | `TLP:CLEAR` |

## Livrables

- [Rapport complet du projet](report/PROJECT_REPORT.md)
- [Architecture et flux](docs/architecture.md)
- [Procédure de déploiement](docs/deployment.md)
- [Supervision de pfSense](docs/monitoring.md)
- [Notifications Dashboard et e-mail](docs/notifications.md)
- [Incidents et dépannage](docs/troubleshooting.md)
- [Limites et feuille de route](docs/limitations.md)
- [Sécurité de la publication](docs/security.md)
- [Inventaire des preuves](evidence/README.md)

## Architecture logique

```text
                         accès privé / WireGuard
                                   |
                            pfSense 192.168.20.1
                         routage et filtrage inter-VLAN
                         /            |             \
              VLAN 20 ADMIN     VLAN 30 DEV     VLAN 40 KALI
                    |                 |                |
        Ubuntu Admin / Wazuh       DMZ_WEB           Kali
        192.168.20.10                 |
              | KVM/libvirt           | agent Windows + Ping Check
              |                       +-----------> pfSense
              v
        VM Debian 12 TRMM
        réseau virtuel privé
              |
      rmm / api / mesh en HTTPS
```

Le schéma détaillé et les choix de cloisonnement sont présentés dans
[`docs/architecture.md`](docs/architecture.md).

## Résultats validés

- KVM, QEMU, libvirt et `/dev/kvm` opérationnels sur l'hôte Ubuntu.
- VM Debian 12 dédiée au RMM, sans cohabitation avec Wazuh.
- Trois noms DNS distincts pour l'interface web, l'API et MeshCentral.
- Certificat TLS wildcard obtenu par challenge DNS-01.
- Interface Tactical RMM accessible en HTTPS.
- Deux postes Windows enrôlés et visibles en ligne.
- Contrôle `PFSENSE - ICMP Availability` exécuté toutes les 300 secondes.
- Alerte configurée après trois échecs consécutifs, sévérité `Warning`.

![Contrôle ICMP pfSense opérationnel](evidence/screenshots/02-pfsense-check-ok.png)

## Choix structurants

| Décision | Justification |
|---|---|
| VM dédiée plutôt qu'installation sur l'hôte | Le script traditionnel exige un serveur vierge et pourrait casser Wazuh |
| Debian 12 | Système pris en charge par la documentation TRMM utilisée pendant le POC |
| 4 Gio de RAM | Minimum documenté et compromis adapté aux 15 Gio de l'hôte |
| DNS séparé `rmm`, `api`, `mesh` | Prérequis de routage applicatif et de TLS de Tactical RMM |
| Certificat public par DNS-01 | HTTPS valide sans exposer temporairement un service HTTP |
| Ping depuis DMZ_WEB | pfSense n'est pas une cible classique de l'agent TRMM |
| TRMM et Wazuh complémentaires | Opérations et maintenance d'un côté, détection de sécurité de l'autre |

## Reproduction

La procédure complète est dans [`docs/deployment.md`](docs/deployment.md). Les
grandes phases sont :

1. Auditer les ressources, les ports et le support KVM de l'hôte.
2. Installer KVM/QEMU, libvirt et Virt-Manager.
3. Créer une VM Debian 12 minimale et prendre un snapshot avant TRMM.
4. Configurer le routage, le DNS interne et les règles pfSense minimales.
5. Préparer `rmm`, `api` et `mesh` au même niveau DNS.
6. Obtenir un certificat wildcard par challenge DNS-01.
7. Installer Tactical RMM avec le certificat validé.
8. Créer le client, le site et le fuseau horaire du laboratoire.
9. Enrôler les agents Windows pilotes.
10. Créer et valider le contrôle ICMP de pfSense.
11. Configurer puis tester les notifications Dashboard et e-mail.

> Ne jamais réutiliser les valeurs de mot de passe, jetons, clés privées ou
> challenges ACME d'une capture. Ces données sont exclues du dépôt.

## Compétences démontrées

- audit de capacité et analyse de compatibilité ;
- virtualisation KVM/QEMU et administration libvirt ;
- segmentation VLAN et filtrage pfSense ;
- DNS interne, TLS et challenge ACME DNS-01 ;
- déploiement et exploitation d'une plateforme RMM ;
- enrôlement et supervision de postes Windows ;
- supervision sans agent et stratégie d'alerte ;
- restauration QCOW2 et analyse d'incidents d'installation ;
- articulation entre RMM et SIEM ;
- documentation technique et publication responsable des preuves.

## Auteur

**Mounir KOUSKOUS**<br>
Opérateur Cybersécurité | SOC Junior | Administration Système et Réseau<br>
[LinkedIn](https://www.linkedin.com/in/mounir-kouskous/)

## Avertissement

Ce projet décrit un laboratoire contrôlé. Les adresses privées appartiennent aux
plages RFC 1918. Tactical RMM donne des capacités d'administration puissantes :
son utilisation doit être limitée aux équipements possédés ou explicitement
autorisés, avec authentification forte, droits minimaux et journalisation.
