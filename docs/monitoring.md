# Monitoring de pfSense

## Objectif

pfSense n'etant pas une cible classique de l'agent Tactical RMM, sa
disponibilite est controlee indirectement depuis l'agent Windows `DMZ_WEB`,
situe dans le HomeLab.

Avant l'automatisation, quatre requetes ICMP ont confirme la connectivite de
`DMZ_WEB` vers `192.168.20.1`, avec des temps de reponse de 2 a 3 ms lors du
test ponctuel.

## Check valide

| Parametre | Valeur |
|---|---|
| Type | Ping Check natif |
| Nom | `PFSENSE - ICMP Availability` |
| Cible | `192.168.20.1` |
| Agent executant | `DMZ_WEB` |
| Intervalle | 300 secondes |
| Severite | `Warning` |
| Echecs avant alerte | 3 |

![Parametres du check](../evidence/screenshots/03-pfsense-check-settings.png)

## Creation dans Tactical RMM

1. Ouvrir le client et le site du HomeLab.
2. Selectionner l'agent `DMZ_WEB`.
3. Ouvrir `Checks`.
4. Choisir `New > Ping Check`.
5. Renseigner les parametres du tableau.
6. Enregistrer puis utiliser `Run Checks Now`.
7. Confirmer la coche verte et consulter `Show Run History`.

Le resultat valide affiche trois paquets emis, trois recus et aucune perte.

## Interpretation

Un etat vert confirme uniquement que l'interface pfSense cible repond depuis
`DMZ_WEB`. Il ne prouve pas que :

- l'acces WAN fonctionne ;
- le tunnel WireGuard est disponible ;
- le DNS repond correctement ;
- toutes les interfaces VLAN sont operationnelles ;
- le pare-feu dispose de ressources suffisantes.

## Dependances et faux positifs

Le check depend de l'agent `DMZ_WEB`, de Windows, du VLAN DEV et du routage
inter-VLAN. Si cet agent est hors ligne, le controle ne permet pas de conclure
que pfSense est en panne.

Une supervision plus robuste doit utiliser une seconde sonde independante et
correler :

- le statut de l'agent executant ;
- un second ping depuis une autre zone ;
- un controle HTTPS de l'interface d'administration, si autorise ;
- des metriques SNMP restreintes ;
- les journaux pfSense deja exploites par Wazuh.

## Controles suivants proposes

| Controle | Cible | Statut |
|---|---|---|
| ICMP Ubuntu Admin | `192.168.20.10` | A planifier |
| HTTPS Wazuh | hote Admin, TCP 443 | A planifier |
| Wazuh Manager | hote Admin, TCP 1514 | A planifier |
| ICMP Kali | `192.168.40.10` | A planifier |
| DNS pfSense | requete DNS reelle | A planifier |
| SNMP pfSense | CPU, RAM, interfaces | A etudier |

Un simple controle TCP 53 ne valide pas une resolution DNS. Un check de service
doit tester le comportement attendu, pas seulement l'ouverture du port.
