#!/usr/bin/perl

# Une façon d'ajouter en Perl de documentation, <<motClé; ... motClé 
# Dans la ligne de commande, on peut y accéder de façon suivante: perldoc parcoursdarbo.pl
#-----------------------------------------------------------------------------------------
<<DOC; 
Vos Noms : Bampatzani Sotiria, Dehareng Morgane
MAI 2017
usage : perl parcours-arborescence-fichiers repertoire-a-parcourir
Le programme prend en entrée le nom du répertoire contenant les fichiers
à traiter
Le programme construit en sortie un fichier structuré contenant sur chaque
ligne le nom du fichier et le résultat du filtrage :
<FICHIER><NOM>du fichier</NOM></FICHIER><CONTENU>du filtrage</CONTENU></FICHIER>
DOC
#------------------------------------------------------------------------------------------

# Déclaration des variables:
# $rep, le nom du répertoire dans lequel on va aller chercher les fichiers à traiter
# $type, le fichier particulier qu'on veut traiter
# $dico, une variable associée au tableau qu'on utilisera par la suite pour vérifier si les data qu'on a stockés
# étaient déjà récupérées
#-----------------------------------------------------------------------------------------------------------------
my $rep="$ARGV[0]";
my $type="$ARGV[1]";
my $dico = "";

# On vérifie que le repértoire ne se termine pas par un "/"
#----------------------------------------------------------
$rep=~ s/[\/]$//;

# Initialisation de la variable $DUMPFULL1 qui va contenir le flux de sortie 
#---------------------------------------------------------------------------
my $DUMPFULL1="";

# Déclaration de la variable pour le fichier de sortie
# 	> $output1 contiendra les data en format xml
#------------------------------------------------------
my $output1="$type.xml";
if (!open (FILEOUT, ">:encoding(utf8)", $output1)) { die "Pb a l'ouverture du fichier $output1"};

# appel de la procédure afin de pouvoir parcourir tous les fichiers du répertoire
#--------------------------------------------------------------------------------
&parcoursarborescencefichiers($rep);	#recurse!

# Impression d'information dans le fichier xml
# 	> Ajout:
# 		- du prologue
# 		- de la racine, ici: <PARCOURS> </PARCOURS>
# 		- de la balise <NOM> </NOM> 		(pas obligatoire)
# 		- de la balise <FILTRAGE> </FILTRAGE> dans laquelle se trouveront les data extraits et traités
#-----------------------------------------------------------------------------------------------------
print FILEOUT "<?xml version=\"1.0\" encoding=\"UTF-8\" ?>\n";
print FILEOUT "<PARCOURS>\n";
print FILEOUT "<NOM>Bampatzani Sotiria, Dehareng Morgane</NOM>\n";
print FILEOUT "<FILTRAGE>".$DUMPFULL1."</FILTRAGE>\n";
print FILEOUT "</PARCOURS>\n";

# > Fermeture:
# 	- de FILEOUT, on n'a rien d'autre à ajouter en ce moment
#-----------------------------------------------------------
close(FILEOUT);

exit; # le programme se termine ici! donc on ferme tout! après on les utilise mais cette fois-ci dans sub !


# Appel de la procédure (qui se trouve tout en bas) pour parcourir l'arborescence des fichiers
#---------------------------------------------------------------------------------------------
sub parcoursarborescencefichiers {

	# Déclaration de la variable $path = shift va supprimer le premier élément du tableau @_ 
	# et elle va le renvoyer, dans ce cas: c'est $_[0]; qui correspond à "/"
	# Ouverture du répertoire. Si on n'arrive pas à le créer, on affiche un message d'erreur
	# 	la fonction opendir prend deux arguments: DIR et $path, où DIR est pour DIRHANDLE et $path est le répertoire
	# Déclaration de la variable @files (c'est un tableau)
	# 	la fonction readdir va rassembler dans ce tableau tous les fichiers/répertoires contenus dans le répertoire
	# Fermeture du répertoire
	#---------------------------------------------------------------------------------------------------------------
    my $path = shift(@_);
    opendir(DIR, $path) or die "can't open $path: $!\n";
    my @files = readdir(DIR);
    closedir(DIR);
	
	# Pour chaque fichier qui se trouve dans le répertoire...
	#-----------------------------------------------------------------------------------------------
    foreach my $file (@files) {
	
	# On passe au prochain fichier si l'élément de la liste qu'on est en train de traiter, 
	# rassemble soit à "." (répertoire courant) ou à ".." (répertoire parent), donc on passe 
	# à l'élément suivant
	#----------------------------------------------------------------------------------------
	next if $file =~ /^\.\.?$/; 
	
	# reconstruction du chemin relatif
	#---------------------------------
	$file = $path."/".$file;
	
	# si c'est un répertoire ( -d : opération qui permet d'interroger le type de ressource! )
	#-----------------------------------------------------------------------------------------------------------
	if (-d $file) { 
	    &parcoursarborescencefichiers($file);	#recurse! #on redescend!
		print "<REPERTOIRE> ==> ", $file,"\n";
	} # fin de la boucle if (répertoire)
	
	# si c'est un fichier 
	#-----------------------------------------------------------------------------------------------
	if (-f $file) { 
		
		# si le fichier traité est un fichier  .xml et s'il contient "3210" (qui correspond à la rubrique 
		# qui nous intéresse, ou encore, le deuxième argument qu'on a passé à la ligne de commande)
		#------------------------------------------------------------------------------------------------
		if ($file =~ /$type.+\.xml/){ 

		# TRAITEMENT à réaliser sur chaque fichier
		#-----------------------------------------

		# ouverture du fichier encodé en utf8, où se trouve le data qu'on cherche
		#------------------------------------------------------------------------
		open my $input, "<:encoding(utf8)" ,$file;
		
		# déclaration et initialisation d'une variable où seront stocké les data
		#-----------------------------------------------------------------------
		my $texte = "";
		
		# Première boucle while
		# tant qu'on lit une ligne du fichier $input, on supprime le dernier 
		# caractère \n, et puis on supprime aussi \r (aussi connu comme ^M), 
		# afin d'avoir une seule ligne, une seule chaîne de caractères
		#-------------------------------------------------------------------
		while (my $ligne = <$input>){
			chomp $ligne;
			$ligne =~ s/\r//g;
			$texte = $texte . $ligne ;
		}
		
		# Nettoyage du texte
		#---------------------------------------------------------------------------
		$texte =~ s/> +</></g; # suppression de toutes les espaces entre les balises
		
		# Substitution des entités HTML avec les caractères qui y correspondent
		#----------------------------------------------------------------------
		$texte =~ s/&#38;#39;/'/g; 
		$texte =~ s/\x{2019}/'/g;
		$texte =~ s/\x{2026}/.../g;  
		$texte =~ s/&#38;#34;/"/g;
		$texte =~ s/&#233;/é/g;
		$texte =~ s/&#234;/è/g;

		# Deuxième boucle while
		# tant qu'il existe dans mon texte qqch qui rassemble à la regexp,
		# on sauvegarde dans des variables ce qui se trouve entre les balises <title> et 
		# <description> respectivement!
		#------------------------------------------------------------------------------------------------
		while ($texte =~ /<item><title>(.+?)<\/title>.+?<description>(.+?)<\/description>/g){
			my $titre=$1; # récupération du contenu entre les balises <title> </title>
			my $description=$2; # récupération du contenu entre les balises <description> </description>
			$description =~ s/&lt;.+?&gt;//g; # nettoyage de la description	

			# Si on n'a pas déjà récupéré ce titre
			#----------------------------------------------------------------------------------------------------------
			if (!(exists $dico{$titre}))
			{ 
				# la variable $xml est une chaîne étiquetée en format XML
				#-------------------------------------------------------------
				my ($xmlTitre, $xmlDescr) = &etiquette ($titre, $description);
				
				# Impression du fichier xml
				#------------------------------------------------------------------------------
				$DUMPFULL1 = $DUMPFULL1 . "<item><titre>$xmlTitre</titre><description>$xmlDescr</description></item>\n";
					
				# "Incrémentation" de la variable $dico. Ceci veut dire que dans notre tableau, 
				# on a une occurrence de ce titre, et donc on a récupéré ce titre une seule fois
				#-------------------------------------------------------------------------------
				$dico{$titre}=1;
			} # fermeture de la boucle if (si c'est du contenu déjà stocké)
			
		} # fermeture de la boucle while (récupération du contenu entre des balises)
		
		close $input; # fermeture du fichier qui contient les data qu'on vient d'extraire
		
		} # fermeture de la boucle if (si c'est un fichier .xml et s'il contient "3208")
		
	} # fin de la boucle if (si c'est un fichier)
	
    } # fin de la boucle foreach (pour chaque fichier du répertoire)
	
} # fin de la procédure parcoursarborescencefichiers

#------------------------------------------------------------------------------------------------------------------------------
#Appel de la procédure pour l'étiquetage morphosyntaxique
#------------------------------------------------------------------------------------------------------------------------------
sub etiquette
{
	# On vide complétement le tableau avec ce qui était passé en argument
	my $var1 = shift(@_); # est la même chose que $_[0]
	my $var2 = shift(@_); # est la même chose que $_[1]
	
	# Ici, il faut intégrer les programmes 
	# 	> pour tokeniser le texte extrait
	# 	> pour transformer ce dernier en XML
	
	# Le programme de tokenisation a besoin d'un fichier en entrée... 
	# Donc il va falloir le créer. Mais on va créer un fichier chaque 
	# fois qu'on extrait un titre ou une description...
	#------------------------------------------------------------------------------------
	open my $f1, ">:encoding(utf-8)", "titre.txt"; # un fichier pour le titre
	open my $f2, ">:encoding(utf-8)", "description.txt"; # un fichier pour la description
	
	# Ecriture du texte dans les fichiers créés
	#------------------------------------------
	print $f1 $var1;
	print $f2 $var2;
	
	# Fermeture des fichiers
	#-----------------------
	close $f1;
	close $f2;

	# Traitement et étiquetage du titre
	#----------------------------------
	
	# Lancement de la commande SYSTEM qui permet de lancer sur le système d'exploitation qu'on travaille, 
	# une commande (on travaille sous Unix - Cygwin)
	
	# Appel du programme qui fait l'étiquetage
	#------------------------------------------------------------------------------------------------------------------------
	system("perl tokenise-utf8.pl titre.txt | ./tree-tagger -token -lemma -no-unknown french-utf8.par > titre-etiquete");
	
	# Appel du programme qui fait le balisage XML
	# 	le programme suivant rajoute l'extention .xml automatiquement
	#----------------------------------------------------------------
	system("./treetagger2xml-utf8.exe titre-etiquete utf8"); 

	# Ouverture du fichier créé (celui avec les balises XML) pour extraire ces données et les 
	# transformer dans une chaîne de caractères pour les mettre par la suite dans le fichier globale
	#-----------------------------------------------------------------------------------------------
	open my $f1, "<:encoding(utf8)", "titre-etiquete.xml"; 

	my $concatTitre = "";
	
	my $ligne = <$f1>; # Lecture de la première ligne (l'en-tête XML). On ne fait rien avec !
	
	# Lecture du fichier créé avec le format xml, à partir de la 2ème ligne, 
	# ligne par ligne et on sauvegarde ce contenu dans une variable
	#-----------------------------------------------------------------------
	while (my $ligne = <$f1>)
	{
		$concatTitre = $concatTitre . $ligne;
	}
	
	# Fermeture du fichier qui contient le titre
	#-------------------------------------------
	close $f1;
	
	# Traitement et étiquetage de la description
	#-------------------------------------------
	
	# Appel du programme qui fait l'étiquetage
	#--------------------------------------------------------------------------------------------------------
	system("perl tokenise-utf8.pl description.txt | ./tree-tagger -token -lemma -no-unknown french-utf8.par > description-etiquete");
	
	# Appel du programme qui fait le balisage XML
	# 	le programme suivant rajoute l'extention .xml automatiquement
	#-------------------------------------------------------------------
	system("perl treetagger2xml-utf8.pl description-etiquete utf8"); 
	
	# Ouverture du fichier créé (celui avec les balises XML) pour extraire ces données et les 
	# transformer dans une chaîne de caractères pour les mettre par la suite dans le fichier globale
	#-----------------------------------------------------------------------------------------------
	open my $f2, "<:encoding(utf8)", "description-etiquete.xml"; 
	
	my $concatDescr = "";
	
	my $ligne = <$f2>; # Lecture de la première ligne (l'en-tête XML). On ne fait rien avec !
	
	# Lecture du fichier créé avec le format xml, à partir de la 2ème ligne, 
	# ligne par ligne et on sauvegarde ce contenu dans une variable
	#-----------------------------------------------------------------------
	while (my $ligne = <$f2>)
	{
		$concatDescr = $concatDescr . $ligne;
	}
	
	# Fermeture du fichier qui contient la description
	#-------------------------------------------------
	close $f2;
	
	# Renvoie des données au programme principal
	# Renvoie le titre étiqueté et balisé, ainsi que la description étiquetée et balisée
	#-----------------------------------------------------------------------------------
	return $concatTitre, $concatDescr;
	
	exit;
	
} # fin de la procédure etiquette