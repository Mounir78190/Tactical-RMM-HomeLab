# Inventaire des preuves

Les captures ont été recadrées et anonymisées pour une publication
`TLP:CLEAR`. Les originaux contenant des données inutiles ou sensibles ne sont
pas distribués.

Les empreintes SHA-256 sont publiées dans
[`checksums.sha256`](checksums.sha256).

| Fichier | Preuve | Traitement |
|---|---|---|
| `01-windows-agents.png` | Deux agents Windows en ligne dans le site HomeLab | navigateur retiré, utilisateurs masqués |
| `02-pfsense-check-ok.png` | Ping Check pfSense réussi, 3/3 paquets | navigateur retiré, utilisateurs masqués |
| `03-pfsense-check-settings.png` | Cible, sévérité, seuil et intervalle du check | navigateur retiré, utilisateurs masqués |

## Captures volontairement exclues

- erreurs DNS ou Certbot contenant des valeurs ACME ;
- interface du fournisseur DNS ;
- terminal affichant une adresse e-mail ;
- vue agent contenant une IP publique ;
- essais de scripts PowerShell sans rapport direct avec le résultat du POC ;
- sorties Netplan contenant des clés Wi-Fi en clair.

## Règles de conservation

1. Conserver les originaux uniquement dans un espace privé chiffré.
2. Ne publier que la version recadrée et revue.
3. Régénérer les sommes SHA-256 après toute modification.
4. Vérifier en taille réelle les barres d'adresse, terminaux et infobulles.
5. Révoquer tout secret accidentellement exposé avant de supprimer une capture.
