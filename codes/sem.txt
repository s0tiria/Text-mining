#!/usr/bin/perl

my $nmfic = "$ARGV[0]"; 
my $fic = "$nmfic.text"; 

open (FICTEXT, "<$fic") or die ("probleme sur ouverture de la sortie TEXT...");

open(OUT, ">:encoding(ISO-8859-1)", "$nmfic-pers.txt");
# open(OUT, ">:encoding(ISO-8859-1)", "$nmfic-loc.txt");
# open(OUT, ">:encoding(ISO-8859-1)", "$nmfic-org.txt");
# open(OUT, ">:encoding(ISO-8859-1)", "$nmfic-comp.txt");
# open(OUT, ">:encoding(ISO-8859-1)", "$nmfic-verb.txt");

print OUT "------------------------\nPERSONNES\n------------------------\n";
# print OUT "------------------------\nLIEUX\n------------------------\n";  
# print OUT "------------------------\nORGANISATIONS\n------------------------\n"; 
# print OUT "------------------------\nCOMPAGNIES\n------------------------\n";
# print OUT "------------------------\nVERBES\n------------------------\n";

while (my $ligne = <FICTEXT>)
{
    if ($ligne =~/\(Person(.+?)\)/gs)
    {
        my $pers = $1;
        $pers =~s/\(//g; 
        $pers =~s/\)//g; 
        $pers =~s/NP//g; 
        $pers =~s/\/P//g;
        print OUT $pers, "\n";  
    }

    # if ($ligne =~/\(Location(.+?)\//gs)
    # {
    #     my $loc = $1;
    #     $loc =~s/\(//g; 
    #     $loc =~s/\)//g; 
    #     $loc =~s/NP//g; 
    #     print OUT $loc, "\n";  
    # }

    # if ($ligne =~/\(Organization(.+?)\)/gs)
    # {
    #     my $org = $1;
    #     $org =~s/\(//g; 
    #     $org =~s/\)//g; 
    #     $org =~s/NP//g; 
    #     $org =~s/__UNKNOWN__//g;
    #     $org =~s/\/[A-Z]{1,3}//g;
    #     print OUT $org, "\n";  
    # }

    # if ($ligne =~/\(Company(.+?)\)/gs)
    # {
    #     my $comp = $1;
    #     $comp =~s/\(//g; 
    #     $comp =~s/\)//g; 
    #     $comp =~s/NP//g; 
    #     $comp =~s/\/P//g;
    #     $comp =~s/__UNKNOWN__//g;
    #     $comp =~s/\/[A-Z]{1,3}//g;
    #     print OUT $comp, "\n";  
    # }

    # if ($ligne =~/\((.+?)(avoir|être)\/VINF\b(.+?)\)/gs)
    # {
    #     my $inf = $2;
    #     my $part = $3; 
        
    #     if ($part =~/\b/)
    #     {
    #         my $verb = $inf.$part; 
    #         $verb =~s/"//g;
    #         $verb =~s/\/?[A-Z]{1,3}//g;
    #         print OUT $verb,"\n";  
    #     }
    # }
}

close(FICTEXT);
system("iconv -f ISO-8859-1 -t UTF-8 $nmfic-pers.txt"); 
# system("iconv -f ISO-8859-1 -t UTF-8 $nmfic-loc.txt");  
# system("iconv -f ISO-8859-1 -t UTF-8 $nmfic-org.txt");
# system("iconv -f ISO-8859-1 -t UTF-8 $nmfic-comp.txt");  
# system("iconv -f ISO-8859-1 -t UTF-8 $nmfic-verb.txt");    
close(OUT); 
exit; 