# Limites et feuille de route

## Limites du POC

- Le dimensionnement vise quelques agents, pas une production MSP.
- Seuls deux agents Windows ont ete valides.
- L'agent Linux/macOS n'a pas ete deploye en raison du prerequis de signature
  de code et de sponsoring affiche par l'instance.
- Le check pfSense valide uniquement ICMP depuis `DMZ_WEB`.
- L'absence de redondance de la sonde peut produire une ambiguite si DMZ_WEB
  est hors ligne.
- Les notifications SMTP et le pop-up Dashboard restent a tester de bout en
  bout.
- Le renouvellement automatique du certificat wildcard DNS-01 n'est pas valide.
- La supervision SNMP de pfSense n'est pas configuree.
- L'essai d'un script PowerShell tiers n'a pas abouti et reste hors perimetre.
- La haute disponibilite et la reprise sur un second hote ne sont pas traitees.

## Feuille de route priorisee

| Priorite | Action | Critere de sortie |
|---|---|---|
| P1 | Configurer un Alert Template | pop-up et e-mail recus sur un check de recette |
| P1 | Tester sauvegarde et restauration TRMM | restauration documentee sur VM isolee |
| P1 | Automatiser le renouvellement TLS | renouvellement en staging puis production |
| P2 | Ajouter une seconde sonde ICMP | correlation entre deux zones independantes |
| P2 | Superviser Wazuh par checks reseau | ICMP, HTTPS et service attendus visibles |
| P2 | Etudier SNMPv3 pour pfSense | metriques sans communaute SNMPv2 publique |
| P2 | Mettre en place des politiques d'automatisation | seuils CPU, RAM, disque et services pilotes |
| P3 | Evaluer l'agent Linux officiel | decision documentee sur sponsoring et signature |
| P3 | Integrer les alertes TRMM a Wazuh | webhook ou collecte testee sans boucle d'alertes |
| P3 | Documenter une PRA complete | RTO/RPO, sauvegardes et exercice de reprise |

## Ameliorations de securite

- separer les comptes d'administration et de service ;
- limiter les fonctions `Run script on Server` aux roles strictement necessaires ;
- mettre les secrets SMTP dans un coffre ;
- definir une retention des historiques adaptee au stockage ;
- revoir periodiquement les agents, comptes et tokens inactifs ;
- creer des alertes Wazuh sur les actions sensibles du serveur RMM.
