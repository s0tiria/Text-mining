#!/usr/bin/perl

open(FILE,"$ARGV[0]");
open(OUT, ">:encoding(ISO-8859-1)", "3210-treetagger-nomnom.txt");
print OUT "------------------------\nNOM NOM\n------------------------\n"; 

my @lignes=<FILE>;
close(FILE);
while (my $ligne=shift(@lignes)) {
    chomp $ligne;
    my $sequence="";
    my $longueur=0;
    if ( $ligne =~ /<element><data type=\"type\">NAM<\/data><data type=\"lemma\">[^<]+<\/data><data type=\"string\">([^<]+)<\/data><\/element>/) {
		my $forme=$1;
		$sequence.=$forme;
		$longueur=1;
		my $nextligne=$lignes[1];
		if ( $nextligne =~ /<element><data type=\"type\">NAM<\/data><data type=\"lemma\">[^<]+<\/data><data type=\"string\">([^<]+)<\/data><\/element>/) {
			my $forme=$1;
			$sequence.=" ".$forme;
			$longueur=2;
		}
    }

    if ($longueur == 2) {
		print OUT $sequence, "\n";
    }
	
}
close(OUT); 
system("iconv -f ISO-8859-1 -t UTF-8 3210-treetagger-nomnom.txt"); 
exit; 
