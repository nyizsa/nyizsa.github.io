# Blogbejegyzések

## Hogyan használom a Githubot?

A [Github](https://github.com) egy nagyon hasznos eszköz azoknak, akik szoftvert fejlesztenek, dokumentációt írnak, vagy hasonló munkafolyamatot végeznek. A lényeg, hogy szöveges állományok verziókezelésére és közös munkára verhetetlen.

De mi a helyzet azokkal, akiknek Microsoft Word formátumban kell dokumentációt előállítaniuk? A Google találatai alapján már másoknak is eszébe jutott, hogy a Githubon lehetne .docx formátumú állományokat verziókezelni, illetve azokon csoportban dolgozni. (Szerintem mindenkiben fölmerül valami hasonló igény, amikor a Munka_vegleges_javitott_4_utolso.docx állományt elküldi e-mailen.)

A legtöbb válasz a fönti kérdésre az, hogy természetesen lehet .docx állományokat hostolni a Githubon, de a verziókezelés nagyjából értelmét veszti, mivel a .docx egy bináris formátum (vagyis tulajdonképpen egy Zippel tömörített adathalmaz), így a diffek értelmüket veszítik.

Volt még olyan javaslat is, hogy az állományt kitömörítve töltsük föl, és használat előtt be kell tömöríteni, majd megint ki. Ez talán járható lenne, de szükségtelenül túlbonyolítja a használatot.

Akkor mégis mi a megoldás? A Github szöveges állományok tárolására jött létre, tároljunk rajta szöveges állományokat! Szerencsére alapból támogatott a [Markdown](https://en.wikipedia.org/wiki/Markdown), ennek segítségével minden szükséges dolog elhelyezhető a szövegben: fejezetcímek, kiemelés, képek, táblázatok, linkek, lábjegyzetek. Megmarad a verziókezelés és a csoportmunka lehetősége, olvasható diffekkel és minden elérhető funkcióval.

De hogy lesz ebből Word dokumentum? Szerencsére a Github nem csak a verziókezeléshez, hanem a CI/CD munkamenethez is tartalmaz hasznos segédeszközöket. A különféle fordítók mellett a [Pandoc](https://pandoc.org) is rendelkezésre áll. Vagyis elég egy munkafolyamatot létrehozni, az pedig gondoskodik minden szükséges lépésről.

A dokumentum neve a példában `document.md` lesz. Ebbe lehet írni a feladatot, a Markdown szintaxisnak megfelelően. Erre bármelyik szövegszerkesztő megfelel, de hasznos, ha a Git integrálva van a programba. Én a [Kate](https://kate-editor.org/)-t használom. Számozással, formázással nem kell törődni, a szerző koncentráljon a tartalomra! Ha elkészült a mű (vagy csak kíváncsiak vagyunk, hogy haladunk), egyszerűen generáltassuk a dokumentumot! Ehhez kell egy Action.

### Új Github Action létrehozása

A legegyszerűbb Action például ilyen lehet:

```yaml
name: Convert document to docx format

on: workflow_dispatch

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-latest
    steps:
      - name: checkout files
        uses: actions/checkout@v5
      - name: convert the document
        uses: docker://pandoc/core:3.5
        with:
          args: >-
            --to=docx
            --output=document.docx
            document.md
      - name: upload the result
        uses: actions/upload-artifact@v7
        with:
          path: document.docx
          archive: false
```

Az `on: workflow_dispatch` sor az indítás feltételét határozza meg. Sok esemény kiválasztható, jelen esetben kézzel kell elindítani a folyamatot. Így csak akkor generálódik a tartalom, ha tényleg szükséges. A futtatókörnyezet egy Ubuntu lesz, az állományok betöltése után elindul a Pandoc a megadott paraméterekkel, vagyis egy `document.docx` állományt szeretnénk kapni a folyamat végén. Ez a folyamat általában egy tömörített állományt ad eredményül, az `archive: false` kapcsolóval azt állítottam be, hogy ne tömörítsen, közvetlenül legyen hozzáférhető a végeredmény. Ez csak akkor működik, ha egy darab állományt eredményez a folyamat.

Kézzel elindítva tehát az Actiont valóban kapunk egy Microsoft Word .docx formátumú állományt, amelyben az van, amit Markdownban írtunk. A fejezetcímek, ábrák, linkek a helyükön, bár minden úgy néz ki, ahogy a Word default beállításai szerint ki kell néznie. Ezen kívül a dokumentum elején általában szerepelnie kell néhány kelléknek, mint például szerző, cím, összefoglaló, a végén pedig egy irodalomjegyzéket szokás közölni.

### A dokumentum formázása

Ehhez szükségünk lesz egy referenciadokumentumra. Ezt a Pandoc előállítja nekünk a következő parancs segítségével:

```
pandoc -o custom-reference.docx --print-default-data-file reference.docx
```

Ezt Wordben megnyitva látható, hogy minden szükséges elem szerepel benne. Ezek **stílusát** a szükségleteinknek (illetve a kéziratot fogadó előírásainak) megfelelően módosíthatjuk. Mentés után ezt is töltsük föl a repositoryba, majd egészítsük ki az Actionunkban a Pandoc paramétereit a következővel: `--reference-doc=style-reference.docx`!

Ha számozott fejezetcímekre van szükség, akkor még egy paramétert meg kell adnunk: `--number-sections`. Az újonnan generált dokumentumnak most már meg kell felelnie a formai előírásoknak.

### Szerző és cím hozzáadása

A szerző(ke)t és a címet egy külön yaml formátumú állományban kell megadnunk. Itt van lehetőség az összefoglaló, a kulcsszavak és a dokumentum nyelvének megadására is. Ez az állomány például így nézhet ki:

```yaml
author: Nyizsnyik Ferenc
title: Dokumentumok kezelése Githubon
abstract: |
    Ebben a cikkben a dokumnetumok kezeléséről van szó. Ehhez a Github kézenfekvő megoldást kínál.
keywords: dokumentum, Github, Pandoc
lang: hu-HU
```

A Pandoc kapcsolói közé pedig föl kell vennünk a `--metadata-file=meta-inf.yaml` kapcsolót, természetesen az állomány neve szerepel az egyenlőségjel után.

### Irodalomjegyzék hozzáadása

A fölhasznált szakirodalmi művek adatait [BibTeX](https://www.bibtex.org/) formátumban tárolva egy univerzális adattárat kaphatunk, amely szinte minden komoly kiadványszerkesztő programban használható. Sok hivatkozás elérhető ebben a formátumban, de a legtöbb hivatkozáskezelő program képes exportálni egy .bib listát. Kézzel elkészíteni sem nehéz.

Az irodalomjegyzékre is szigorú előírások vonatkoznak, ezért szükségünk lesz egy stílusleíró állományra. Szerencsére már nagyon sok ilyen elérhető, a [Zotero](https://www.zotero.org/styles) honlapjáról valószínűleg le tudjuk tölteni a nekünk megfelelőt. Ha mégsem, keressünk egy olyat, ami egészen hasonló ahhoz, amit előírnak, majd módosítsuk az eltéréseket a szükséges mértékben! Erre a [citationstyles.org](https://editor.citationstyles.org/about/) szerkesztőjében van lehetőségünk.

Ha ez is megvan, töltsük föl a repositoryba, majd adjuk meg a Pandocnak a következő kapcsolókat: `--citeproc`, `--bibliography=Irodalomjegyzek.bib`, `--csl=citation-style.csl`! A következő generálás során a Pandoc már elő fogja állítani az irodalomjegyzéket, amely a szöveg végén és a lábjegyzetekben is meg fog felelni az előírásoknak.

A végleges Action tehát így néz ki:

```yaml
name: Convert document to docx format

on: workflow_dispatch

jobs:
  convert_via_pandoc:
    runs-on: ubuntu-latest
    steps:
      - name: checkout files
        uses: actions/checkout@v5
      - name: convert the document
        uses: docker://pandoc/core:3.5
        with:
          args: >-
            --standalone
            --number-sections   # számozott fejezetcímek
            --citeproc  # irodalomjegyzék előállítása
            --bibliography=Irodalomjegyzek.bib  # az irodalomjegyzék
            --csl=citation-style.csl    # az irodalomjegyzék formátuma
            --reference-doc=style-reference.docx    # a dokumentum formátuma
            --to=docx
            --metadata-file=meta-inf.yaml   # szerző, cím, absztrakt, stb.
            --output=document.docx
            document.md
      - name: upload the result
        uses: actions/upload-artifact@v7
        with:
          path: document.docx
          archive: false
```
