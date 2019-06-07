# Tuto-WGS

Ce document est un court guide d'utilisation pour faciliter les analyses des
données de NCBI WGS.

## Version courte

Tout d'abord, se connecter sur ls31.

Ensuite:

```bash
cd /is3/projects/MB_201905_WGS
source modules
cp -r scripts/template scripts/20190530_NomDeLanalyse
```

On modifie ensuite les en-têtes des scripts avec les valeurs souhaitées.

Ensuite:

```bash
Rscript scripts/20190530_NomDeLanalyse/01_download.R
bash scripts/20190530_NomDeLanalyse/02_blast.sh
Rscript scripts/20190530_NomDeLanalyse/03_blast_analysis.R
```

**Important**: On retrouve maintenant une valeur `ref_name` qui correspond à la
référence. Il est donc possible de réutiliser une référence qui a été créé lors
d'une analyse précédente.

Par exemple, si on souhaite refaire une analyse avec la référence
`20190530_Cupsaliensis`, on fera les étapes suivantes:

1. Copier le répertoire template pour créer notre nouveau répertoire d'analyse
2. Modifier les en-têtes:
    * Pour la valeur `ref_name`, on mettra `20190530_Cupsaliensis`
    * Pour la valeur `analysis_name`, on mettra le nom de l'analyse courante
3. Lancer les scripts dans l'ordre habituel

## Version longue

### Répertoire de travail

Le répertoire de travail se trouve sur les serveurs du Chul (i.e.: ls31) à
l'endroit suivant:

* `/is3/projects/MB_201905_WGS`

### Scripts d'analyse

Les scripts se trouvent dans le répertoire `scripts`:

```bash
scripts
└── template
    ├── 01_download.R
    ├── 02_blast.sh
    └── 03_blast_analysis.R
```

Pour éviter de retrouver un trop grand nombre de fichiers, un répertoire sera
créé pour chaque analyse. Pour une nouvelle analyse, il s'agira donc de copier
ce répertoire en lui donnant le nom de la nouvelle analyse. Il est recommandé
d'inclure la date du début des analyses dans le nom de l'analyse pour qu'il
soit plus facile de s'y retrouver dans le futur.

```bash
cp -r scripts/template scripts/20190530_NomDeLanalyse
```

On doit ensuite modifier les en-têtes des fichiers de scripts pour y inclure
les valeurs pertinentes pour notre analyse. On peut utiliser un éditeur de
texte tel que `nano` à cet effet.

Pour le script `scripts/20190530_NomDeLanalyse/01_download.R`:

```r
# section à modifier
pattern <- "campylobacter upsaliensis" # Le critère de la recherche
analysis_name <- "20190529_Cupsaliensis" # Le nom du répertoire pour les résultats de l'analyse
ref_name="20190530_Cupsaliensis" # Le nom du fichier de référence

strict <- FALSE # Est-ce que le critère de recherche doit être identique
verbose <- TRUE # Détails de l'analyse
force_fasta <- FALSE # Forcer ou non le re-téléchargement des fichiers fasta
force_combined_fasta <- FALSE # Forcer ou non la re-création du fichier fasta combiné
check_md5 <- FALSE # Vérifier le md5 des fichiers déjà téléchargés (permet de mettre à jour mais ralenti le téléchargement)
```

Pour le script `scripts/20190530_NomDeLanalyse/02_blast.sh`:

```bash
# section à modifier
analysis_name="20190529_Cupsaliensis" # Le nom du répertoire pour les résultats de l'analyse
ref_name="20190530_Cupsaliensis" # Le nom du fichier de référence
min_length=500 # Taille minimale des alignements à conserver
input=input_fasta/groel.fa # Fichier fasta à aligner
```

Pour le script `scripts/20190530_NomDeLanalyse/03_blast_analysis.R`:

```r
# section à modifier
analysis_name <- "20190529_Cupsaliensis" # Le nom du répertoire pour les résultats de l'analyse
```

Il est maintenant possible de lancer les analyses.

### Lancer les analyses

#### Chargement des modules

Il faut tout d'abord chargé le bon module R. Pour garder un suivie, le nom du
module se retrouve dans le fichier `modules`:

```bash
cat modules
module load R/R-3.6.0_BioC-3.9
```

On peut charger les modules de la manière suivante:

```bash
source modules
```

#### Téléchargement des données

Le scripts `01_download.R` permet de télécharger les données et de combiner les
fasta en un seul fichier pour permettre l'aligment avec blast.

Pour éviter de re-télécharger inutilement des fichiers, on utilisera le même
répertoire (`data/`) pour conserver les fichiers fasta bruts. Le script
`01_download.R` se chargera donc de cibler les bons fichiers à combiner pour
chaque analyse. On pourra donc ré-utiliser les mêmes fichiers au besoin.

Une fois les valeurs de l'en-tête correctement modifiées, on peut lancer le
script avec la commande `Rscript`:

```bash
Rscript scripts/20190530_NomDeLanalyse/01_download.R
```

#### Alignement blast

Le script `02_blast.sh` s'occupera de lancer une analyse blast avec les
arguments utilisés pour un blast *somewhat similar* sur le site web du NCBI.

On peut placer les fichiers fasta qui correspondent à la requête dans le
dossier `input_fasta`. 

Une fois l'en-tête modifié avec les bonnes valeurs, on peut lancer le blast:

```bash
bash scripts/20190530_NomDeLanalyse/02_blast.sh
```

Cette commande s'occupera d'indexer le fichier fasta combiné dans le répertoire
`ref` puis de lancer l'alignement.

Les résultats de la commande se retrouveront dans le fichier
`results/20190530_NomDeLanalyse/blast.out`.

#### Annotation du blast

Il reste ensuite à ajouter des annotations aux résultats du blast en utilisant
la commande `03_blast_analysis.R` (après avoir modifié son en-tête):

```bash
Rscript scripts/20190530_NomDeLanalyse/03_blast_analysis.R`
```

Les résultats se trouveront dans les fichiers
`results/20190530_NomDeLanalyse/blast_anno.csv` pour les séquences annotées et
dans le fichier `resultats/20190530_NomDeLanalyse/blast_sequence.fna` pour les
séquences des alignements.

Au final le répertoire de résultats devrait ressembler à ceci:

```bash
results
└── 20190529_Cupsaliensis
    ├── blast.out
    ├── blast_sequence.fna
    ├── metadata.csv
    ├── session_info_blast_anno_fasta.txt
    └── session_info_download_merge.txt
```

Les fichiers `session_infos` contiennent les versions des outils utilisés pour
les analyses R.
