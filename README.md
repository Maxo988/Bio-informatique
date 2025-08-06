# Bio-informatique
Scripts utilisés dans le cadre de mon stage à l'Institut Pasteur de Nouvelle-Calédonie à l'aide de SLURM et du cluster de calcul Maestro de l'Institut Pasteur de Paris pour l'analyse d'échantillons métagénomiques.
Les logiciels nécessaires sont Rstudio, Megahit, Prodigal, Bwa-mem 2, samtools, blast+, Htseq-count, Metaphlan, SortmeRNA et leurs éventuelles dépendances.

Raw reads obtenus par séquençage MGI, fragment de 150pb en paired reads.
Ces scripts permettent d'obtenir une idée de la composition taxonomique d'un échantillon métagénomique, de comparer la présence de gènes de résistance d'intérêts entre plusieurs échantillons, et d'étudier le pourcentage de couverture d'un gène dans un échantillon. 

Pour que les scripts fonctionnent correctement, il est nécessaire que les raw reads se terminent par _1.fq.gz et _2.fq.gz (format fastq compressé paired). Pour changer ça, il faut modifier les scripts Assemblage.sh et Metaphlan.sh


## 1) Détection de gènes d'intérêt au sein d'un ensemble d'échantillons métagénomiques

Assemblage.sh - Assemblage des raw reads par Megahit, produit deux fichiers .fa, un avec les contigs >200pb et l'autre avec les contigs >1500pb. Input = Paired raw reads .fq.gz
Prodigal.sh - Prédiction des CDS en identifiant les ORF puis les filtrant grâce à des modèles statistiques. Input = Assemblage .fa
Alignement.sh - Alignement des reads sur l'assemblage, ce qui permet de les localiser à l'aide du fichier BAM trié en sortie. Étapes obligatoires pour la suite. Input = Paired raw reads et assemblage .fa
Pipeline.sh - Réunis plusieurs étapes en un script : 
  1) Alignement si pas encore fait
  2) Blastn pour trouver des séquences d'intérêt à l'aide d'une base de données nucléotidiques. En cas de base de données de gènes, possible de le lancer sur l'output de prodigal plutôt que sur l'assemblage.
  3) Awk pour formater les fichiers
  4) Htseq-count pour compter le nombre de fois où les gènes ou reads apparaissent dans l'assemblage
Input : Assemblage ou prodigal (nécessaire de convertir le fna en fa si prodigal, utile pour n'avoir que les CDS) .fa, raw reads, sorted.bam si alignement déjà fait et base de données blast 
Output : unique_gtf.tsv, fichier tsv contenant les ID, gènes et résistance associés, count.tsv contenant le nombre de hits correspondant à chaque ID du tsv.
Separation_Entete.R - Permet de séparer les entêtes d'une base de données de résistances, pour ne garder que le nom du gène et les résistances associées. Input = Entêtes d'une base de données (peut être obtenu avec un simple grep). Prévu pour un format d'entête classique séparé par ~~~ mais peut nécessiter de légères modifications en fonction de la base de données.
Heatmap_card.R - Heatmap des résistances pour un seul échantillon. Input = unique_gtf.tsv, count.tsv et entêtes séparées de la base de données.
Heatmap_card_all.R - Heatmap des résistances pour un ensemble d'échantillons. Input = dossier contenant les tsv et entêtes séparées de la base de données.

## 2) Évaluation de la couverture d'un plasmide

Blast.sh - Blast d'un ensemble de fichier .fa sur une référence donnée. Input = Dossier contenant les .fa et base de données au format blast
Combinaison_fasta.sh - Permet de combiner plusieurs fasta en un seul et d'en faire une base de données avec un formatage particulier adapté au script Tableau_Plasmide.R. Voir les commentaires pour les détails du formatage
Tableau_Plasmide.R - Prends en entrée des fichiers .tsv issu d'un blast sur une base de données obtenue avec Combinaison_fasta.sh. Calcule la couverture non redondante des séquences de la base de données pour chaque échantillon, et en fait un tableau Excel. 

## 3) Composition taxonomique

Metaphlan.sh - Lance metaphlan sur tous les raw reads d'un dossier. Prévu pour identifier les genres présent dans l'échantillon, une option indiqué dans le script à modifier pour obtenir les espèces ou les familles.   Input = Paired raw reads .fq.gz 
Barplot_genre/_espèce/_famille.R - Créer un bar plot montrant les 10 genres/espèces/familles les plus présentes dans un pool d'échantillon. Possible de pooler les échantillons selon les conditions pour en faire plusieurs bar plot.

Ces scripts sont à revoir, pour lancer directement Metaphlan sans sélectionner le niveau de taxon et tout récupérer, puis de faire un seul script R qui trouve dans le fichier contenant toutes les infos les taxons qui nous intéressent.


## 4) Quantité d'ARN16S

SortmeRNA.sh - Estime la quantité d'ARN d'un échantillon. On peut rapprocher la quantité de reads d'ARN16S d'un échantillon à sa quantité de matériel génétique, ce qui pourrait permettre de pondérer les résultats pour d'autres analyses. 



