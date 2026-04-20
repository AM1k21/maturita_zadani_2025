# Speaker Notes - Project manager (15 minut)

Cil: projit prezentaci plynule za cca 15 minut.
Doporucene tempo: 55-65 sekund na slide.

## Casovy plan

- Slide 1: 0:00-0:45
- Slide 2: 0:45-1:50
- Slide 3: 1:50-2:50
- Slide 4: 2:50-4:00
- Slide 5: 4:00-5:05
- Slide 6: 5:05-6:15
- Slide 7: 6:15-7:10
- Slide 8: 7:10-8:20
- Slide 9: 8:20-9:20
- Slide 10: 9:20-10:25
- Slide 11: 10:25-11:25
- Slide 12: 11:25-12:35
- Slide 13: 12:35-13:05
- Slide 14: 13:05-14:00
- Slide 15: 14:00-15:00

## Slide-by-slide text

### Slide 1 - Titul
"Dobry den, predstavuji maturitni projekt Project manager. Je to interni system pro rizeni projektu a dokumentace. Reseni je postavene na Symfony, PHP a PostgreSQL. Dnes ukazu co system resi, jak je navrzeny po technicke strance a kam muze dal rust."

### Slide 2 - Co to je, pro koho, proc
"System je primarne navrzeny pro mensi IT firmu Spolek BISON jako interni nastroj. Hlavni duvod vzniku byl sjednotit praci s dokumenty a projekty do jednoho mista. Kriticka cast byla navrh rolI a opravneni, a taky navrh navaznosti dokumentu, aby proces nebyl chaoticky. Hlavni prinos je zefektivneni prace projektovych manageru."

### Slide 3 - Koncepce projektu
"Tady ukazu jak jsem system navrhoval. Nesel jsem rovnou do kodu, nejdriv jsem si navrhl obrazovky a tok pouziti, potom ER vazby a az potom implementaci. Klicove bylo navrhnout role a navaznosti dokumentu tak, aby workflow davalo smysl i po mesicich provozu."

### Slide 4 - Technicka specifikace
"Na backendu jsem zvolil Symfony kvuli architekture, bezpecnosti a dobremu DI. Doctrine ORM drzi konzistenci dat a vazeb. PostgreSQL je robustni relacni databaze vhodna pro rust projektu. Frontend je Twig plus Bootstrap, DataTables a Chart.js, protoze slo o administracni system a ne SPA aplikaci. Docker resi reprodukovatelne prostredi a jednodussi nasazeni."

### Slide 5 - Workflow zakazky
"Tok zakazky je navrzeny jako navazujici retezec: Nabidka, Objednavka, Smlouva, Akceptacni protokol, Faktura. Smysl je, aby se neztratila kontinuita mezi obchodni casti a realizaci. Kazdy krok ma stav, odpovednost a auditni stopu."

### Slide 6 - Datovy model
"Tady je videt konkretni struktura databaze. Projekt ma vazby na klienta a na vsechny dokumentove entity. Zasadni je propojka ProjektUzivatel, kde drzim roli uzivatele uvnitr projektu, a to primo ovlivnuje ACL. Tohle je jadro celeho systemu."

### Slide 7 - Architektura systemu
"Na tomhle slidu je mapa konkretni struktury kodu podle souboru. Ukazu presne, kde je request vrstva, business logika, security a UI. Je to dukaz, ze projekt neni jen funkcni, ale ma cistou strukturu, ve ktere se da dlouhodobe vyvijet."

### Slide 8 - Controller ukazka
"Tady je presna ukazka z realneho controlleru. Nejdriv role check, potom branch logika pro admina a bezne uzivatele, pak ACL overeni a nakonec report agregace. Tohle ukazuje, ze use-casy jsou oddelene, ale princip je jednotny napric controllery."

### Slide 9 - Twig a frontend
"Tady je konkretni loop z dashboard sablony, kde se renderuje projekt, klient i role v projektu. Je videt napojeni dat z controlleru do tabulky bez business logiky v template. Frontend je zamerne jednodussi a orientovany na rychlost prace."

### Slide 10 - FileUploader handling
"Tady je konkretni upload mechanika: sanitizace nazvu, unikni jmeno souboru, ulozeni mimo public a zapis metadat. Stahovani nejde prime URL, ale jen pres controller po overeni opravneni. To je dulezite kvuli bezpecnosti dokumentu."

### Slide 11 - Security handling
"Na ukazce voteru je videt rozhodovaci logika: admin ma globalni pristup, ostatni jen kdyz jsou prirazeni na projekt. Tohle se pouziva jednotne nad dokumentovymi entitami. Dalsi vrstva je CSRF, hash hesel a access_control pravidla."

### Slide 12 - Jak dlouho to trvalo
"Tady shrnu casovy ramec navrhu a implementace. Realizace bezela zhruba 4 mesice, od prvnich navrhu v prosinci 2025 po stabilizaci v dubnu 2026. Jako dukaz postupu uvadim migrace, funkcni milniky a 59 commitu."

### Slide 13 - Uzivatelske testovani
"Tuhle sekci mam zatim pripravenu jako placeholder. Do finalni verze doplnim realne testovaci scenare, zpetnou vazbu uzivatelu a konkretni upravy, ktere z testovani vyplynuly."

### Slide 14 - Zaver
"Projekt splnil hlavni cil: centralizoval praci s dokumenty a projekty, zavedl role-based pristup a nastavil konzistentni workflow. Nejvetsi hodnota je prakticka pouzitelnost pro projektove manazery a pripravenost na dalsi rust."

### Slide 15 - Vize do budoucna
"Dalsi smer je pridat notifikace, rozsirit reporty, doplnit API integrace a pripadne pripravit varianty pro dalsi male firmy na miru. Po technicke strance chci zvysit test coverage, doplnit CI/CD a monitoring, aby bylo reseni produkcne odolnejsi."

## Kratke tipy pro prednes

- Mluv v kratkych vetach a drz se jedne hlavni myslenky na slide.
- U kodovych slidU vysvetli proc, ne kazdy radek.
- U kazde casti ukaz dopad na praxi: rychlost, prehled, bezpecnost.
- Nejakou otazku od komise premosti vetou: "Tahle cast navazuje na slide X".
