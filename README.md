# Pipeline d'automatisation météo des vols RAM (n8n)

Workflow n8n qui collecte les vols du jour de Royal Air Maroc (API
AviationStack), les croise avec un référentiel mondial d'aéroports
(coordonnées GPS), enrichit chaque vol des prévisions météorologiques
de ses aéroports de départ et d'arrivée (API Open-Meteo), et exporte
le tout dans un fichier CSV structuré.

Projet réalisé dans le cadre d'un stage au sein de la Direction Data
Factory de Royal Air Maroc (filière TDIA, ENSAH).

## Architecture

Workflow en cinq phases, dont deux à exécution parallèle :

1. **Déclenchement** manuel du workflow.
2. **Collecte parallèle** des deux sources : le référentiel mondial
   d'aéroports (mwgg/Airports, GitHub, ~28 000 aéroports) et les vols
   réels RAM (API AviationStack, filtre compagnie IATA « AT »).
3. **Nettoyage** : réindexation des aéroports par code IATA, filtrage
   des vols annulés, extraction des champs utiles.
4. **Convergence GPS** : association de chaque vol aux coordonnées de
   ses aéroports de départ et d'arrivée.
5. **Double interrogation météo parallèle** (Open-Meteo : températures
   min/max, précipitations, vent), puis fusion et export CSV.

Logique entièrement déterministe : nœuds JavaScript et appels API.

## Prérequis

- Node.js 18+
- n8n : `npm install -g n8n`
- Une clé API gratuite [AviationStack](https://aviationstack.com)

## Installation et exécution

1. Autoriser n8n à écrire dans le dossier de sortie — définir la
   variable d'environnement (une fois pour toutes dans les variables
   Windows, ou à chaque session de terminal) :

```powershell
   $env:N8N_RESTRICT_FILE_ACCESS_TO="C:\chemin\vers\votre\dossier\de\sortie"
   n8n start
```

2. Ouvrir http://localhost:5678

3. Importer le workflow : **Workflows → Import from File →**
   `workflows/meteo_aeroports.json`

4. Dans le nœud **Get_Flights_API** : remplacer
   `YOUR_AVIATIONSTACK_API_KEY` par votre clé AviationStack.

5. Dans le nœud **Read/Write Files from Disk** : adapter le chemin du
   fichier de sortie à votre machine (dans le dossier autorisé à
   l'étape 1).

6. Cliquer sur **Execute workflow** → le fichier `vols_meteo.csv` est
   généré dans le dossier de sortie.

## Exemple de sortie

Le fichier `data/output/vols_meteo.csv` est un exemple réel de
résultat : les vols RAM du 4 septembre 2026, chacun enrichi des
prévisions météo (températures min/max, précipitations, vent) de ses
aéroports de départ et d'arrivée.