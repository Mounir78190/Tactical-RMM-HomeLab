# Limites et feuille de route

## Limites du POC

- Le dimensionnement vise quelques agents, pas une production MSP.
- Seuls deux agents Windows ont été validés.
- L'agent Linux/macOS n'a pas été déployé en raison du prérequis de signature
  de code et de parrainage affiché par l'instance.
- Le check pfSense valide uniquement ICMP depuis `DMZ_WEB`.
- L'absence de redondance de la sonde peut produire une ambiguïté si DMZ_WEB
  est hors ligne.
- Les notifications SMTP et le pop-up Dashboard restent à tester de bout en
  bout.
- Le renouvellement automatique du certificat wildcard DNS-01 n'est pas validé.
- La supervision SNMP de pfSense n'est pas configurée.
- L'essai d'un script PowerShell tiers n'a pas abouti et reste hors périmètre.
- La haute disponibilité et la reprise sur un second hôte ne sont pas traitées.

## Feuille de route priorisée

| Priorité | Action | Critère de sortie |
|---|---|---|
| P1 | Configurer un Alert Template | pop-up et e-mail reçus sur un check de recette |
| P1 | Tester sauvegarde et restauration TRMM | restauration documentée sur VM isolée |
| P1 | Automatiser le renouvellement TLS | renouvellement en staging puis production |
| P2 | Ajouter une seconde sonde ICMP | corrélation entre deux zones indépendantes |
| P2 | Superviser Wazuh par des checks réseau | ICMP, HTTPS et service attendus visibles |
| P2 | Étudier SNMPv3 pour pfSense | métriques sans communauté SNMPv2 publique |
| P2 | Mettre en place des politiques d'automatisation | seuils CPU, RAM, disque et services pilotes |
| P3 | Évaluer l'agent Linux officiel | décision documentée sur le parrainage et la signature |
| P3 | Intégrer les alertes TRMM à Wazuh | webhook ou collecte testée sans boucle d'alertes |
| P3 | Documenter une PRA complète | RTO/RPO, sauvegardes et exercice de reprise |

## Améliorations de sécurité

- séparer les comptes d'administration et de service ;
- limiter les fonctions `Run script on Server` aux rôles strictement nécessaires ;
- mettre les secrets SMTP dans un coffre ;
- définir une rétention des historiques adaptée au stockage ;
- revoir périodiquement les agents, comptes et jetons inactifs ;
- créer des alertes Wazuh sur les actions sensibles du serveur RMM.
