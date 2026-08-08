# Blogbejegyzések

## Hogyan használom a Githubot?

A [Github](https://github.com) egy nagyon hasznos eszköz azoknak, akik szoftvert fejlesztenek, dokumentációt írnak, vagy hasonló munkafolyamatot végeznek. A lényeg, hogy szöveges állományok verziókezelésére és közös munkára verhetetlen.

De mi a helyzet azokkal, akiknek Microsoft Word formátumban kell dokumentációt előállítaniuk? A Google találatai alapján már másoknak is eszébe jutott, hogy a Githubon lehetne .docx formátumú állományokat verziókezelni, illetve azokon csoportban dolgozni. (Szerintem mindenkiben fölmerül valami hasonló igény, amikor a Munka_vegleges_javitott_4_utolso.docx állományt elküldi e-mailen.)

A legtöbb válasz a fönti kérdésre az, hogy természetesen lehet .docx állományokat hostolni a Githubon, de a verziókezelés nagyjából értelmét veszti, mivel a .docx egy bináris formátum (vagyis tulajdonképpen egy Zippel tömörített adathalmaz), így a diffek értelmüket veszítik.

Volt még olyan javaslat is, hogy az állományt kitömörítve töltsük föl, és használat előtt be kell tömöríteni, majd megint ki. Ez talán járható lenne, de szükségtelenül túlbonyolítja a használatot.

Akkor mégis mi a megoldás? A Github szöveges állományok tárolására jött létre, tároljunk rajta szöveges állományokat! Szerencsére alapból támogatott a [Markdown](https://en.wikipedia.org/wiki/Markdown), ennek segítségével minden szükséges dolog elhelyezhető a szövegben: fejezetcímek, kiemelés, képek, táblázatok, linkek, lábjegyzetek. Megmarad a verziókezelés és a csoportmunka lehetősége, olvasható diffekkel és minden elérhető funkcióval.

De hogy lesz ebből Word dokumentum? Szerencsére a Github nem csak a verziókezeléshez, hanem a CI/CD munkamenethez is tartalmaz hasznos segédeszközöket. A különféle fordítók mellett a [Pandoc](https://pandoc.org) is rendelkezésre áll. Vagyis elég egy munkafolyamatot létrehozni, az pedig gondoskodik minden szükséges lépésről.
