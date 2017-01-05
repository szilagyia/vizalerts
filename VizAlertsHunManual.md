# Tableau VizAlerts végfelhasználói segédlet

## A konfigurációs fájl beállításai

Használat során felmerülhetnek az email kiküldéssel kapcsolatos változtatások. A módosításokat a `VizAlerts\config\vizalerts.yaml` fájlban kell elvégezni. Itt a következő beállításokat végezhetjük el:

|paraméter|magyarázat|
|:--------|:---------|
|smtp.serv|SMTP szerver neve|
|smtp.address.from|Az a cím, melyről az emaileket kiküldjük. Ezt felülírhatja az Advanced Alertben beállított `from` mező|
|smtp.address.to|A hibásan lefutott figyelmeztetésekről erre a címre küld hibajelentést|
|smtp.alloweddomains|Itt kell megadni azokat a domaineket, ahova engedélyezni szeretnénk az email kiküldését. Pl: ha itt nem adjuk hozzá a `hotmail.com`domaint, akkor nem fognak kimenni az emailek, hiába paraméterezzük fel `hotmail.com`os címekkel az Alert workbookunkat|
|smtp.ssl|`true`: A VizAlerts SSL titkosítást fog alkalmazni; `false`: nem lesz titkosítás|
|smtp.user|Az SMTP szerver felhasználója; Ha nincs autentikáció akkor `null`|
|smtp.password|Az SMTP szerver jelszava. Ha nincs autentikáció akkor `null`|

## Advanced Alert beállításai

Egy _alert_et két dolog határoz meg:

- egy Tableau Serverre publikált _view_ (_trigger view_)
- egy _subscription_ egy _disabled_ _Subscription Schedule_-ra
  - a Tableau szerveren mindig kell lennie egy _enabled schedule_-nak. Legalább egyet tartsunk meg a normál feliratkozók részére, és emellé vegyük föl saját egyedi _disabled_ _schedule_-unkat.
- a _Subscription Schedule_ neve mindenképp az `Alert` szóval kezdődjön. Ha a _disabled_ _Subscription Schedule_ neve nem így kezdődik, akkor a VizAlerts nem fog értesítéseket küldeni.

A _view_ feladata, hogy meghatározza az _alert_ adatait. A _subscription_ feladata meghatározni azt, hogy a mikor és milyen időközönként kerül ellenőrzésre a nézet. Az _alert_ csak akkor küldődik ki, ha a _view_ tartalmaz adatot.

### Advanced Alert konfiguráció

|Mezőnév (`szóköz`zel kezdve!)|Szükséges?|Magyarázat|
|:----------------------------|:---------|:---------|
| Email Action \*|IGEN|=1, meghatározza, hogy ez egy Advanced Alert|
| Email To \*|IGEN|Címzettek email címe `,` vagy `;` vagy ` ` karakterrel elválasztva|
| Email from ~||Feladó email címe|
| Email CC ~||CC email cím(ek)|
| Email BCC ~||Blindcopy címzett(ek)|
| Email Subject \*|IGEN|Tárgymező szövege|
| Email Header ~||HTML fejléchez és sorközi képhez|
| Email Body \*|IGEN|HTML a szövegtörzshöz; maradhat üres/NULL|
| Email Footer ~||HTML lábléchez vagy sorközi képhez|
| Email Attachment ~||Hivatkozások listája csatolt, nem sorközi tartalomhoz (csatolmány)|
| Email Consolidate ~||Nem kell értéket megadni itt, elég csak felvenni a mezőt. (Több email tartalmának egy emailben való kiküldésére egyesített szolgál)|
| Email Sort Order ~||Bármilyen alfanumerikus érték (a-z+0-9) ami az egyesített emailek sorbarendezésére szolgál|


\*: szükséges mező

~: opcionális mező

> __MINDEN MEZŐNEK SZÓKÖZZEL KELL KEZDŐDNIE! A MEZŐK NEVÉT NEM SZABAD MEGVÁLTOZTATNI; BÁRMILYEN MÁS MEZŐT FIGYELMEN KÍVÜL HAGY A PROGRAM!__

> __HA EXCEL ADATFOTTÁSBÓL TÖLTJÜK FEL AZ ALERTET MEGHATÁROZÓ WORKBOOKOT, AKKOR FELTÖLTÉS UTÁN A WORKBOOKBAN MÓDOSÍTANI KELL A MEZŐNEVEKET, MERT A TABLEAU EXCEL KONNEKTORA LEVÁGJA AZ ELSŐ `szóköz` KARAKTERT A MEZŐNEVEKBŐL!__

---

### Hivatkozások

- __VIZALERTS\_FOOTER()__: Ha beírjuk az ` Email Footer ~` mezőbe, akkor hozzáadja az alapvető láblécet és minden más szöveget amit beírtunk
- __VIZ\_LINK()__: Ha beírjuk az ` Email Body *` vagy ` Email Header ~` vagy ` Email Footer ~` mezőbe, akkor hozzáadja a vizualizációra mutató hiperlinket. Ezt nincs értelme akkor használni, ha nem regisztrált Tableu Server felhasználóknak küldjük ki az _alert_et, mert nem fogják tudni megnyitni az oldalt.
- __VIZ\_IMAGE()__: Ha beírjuk az ` Email Body *` vagy ` Email Header ~` vagy ` Email Footer ~` vagy ` Email Attachement ~` mezőbe, akkor a hivatkozott vizualizációt .png kiterjesztésű képként beilleszti.
- __VIZ\_CSV()__: Ha beírjuk az ` Email Attachement ~` mezőbe, akkor a hivatkozott vizualizáció adatait .csv kiterjesztésű csatolmányként az emailhez adja. Alkalmazhatunk egyedi fájlnevet is a csatolmány neveként.
- __VIZ\_PDF()__: Ha beírjuk a ` Email Attachement ~` mezőbe, akkor a hivatkozott vizualizációt .pdf kiterjesztésű fájlként az emailhez csatolja. Alkalmazhatunk egyedi fájlnevet is a csatolmány neveként.

A __VIZ\_LINK()__, __VIZ\_IMAGE()__, __VIZ\_CSV()__, __VIZ\_PDF()__ hivatkozások alapvetően a _trigger view_t fogják tartalmazni. Ahhoz, hogy az általunk kívánt _view_t tartalmazzák, hivatkoznunk kell rá a következő módon: __VIZ\_IMAGE([workbookname/viewname])__, pl.:  VIZ\_IMAGE([SalesData/SalesDashboard]).

#### URL paraméterek

A feljebb említett hivatkozások tartalmazhatnak URL paramétereket is, pl.: VIZ\_IMAGE(VizAlertsDemo/Product?Region=East). Erről bővebb információ [itt](http://kb.tableau.com/articles/knowledgebase/view-filters-url) található.

#### PNG képméret beállítása

Egy __VIZ\_IMAGE()__ hivatkozás mérete három módon állítható be.

- Ha a _view_ egy fix méretűre állított _dashboard_ vagy a minimum mérete nagyobb mint a VizAlert alapbeállítása, akkor a _dashboard_ mérete a mérvadó.
- A VizAlert alapbeállítását a konfigurációs fájl tartalmazza.
- A képméretet URL paraméterként is beállíthatjuk, melynek formátuma: __:size=[szélesség],[magasság]__; pl.: VIZ\_IMAGE(SalesData/SalesDashboard?Region=East&:size=400,600).

Limitációk:

- Ha a _dashboard_ méretei egy megadott tartományban mozognak, akkor nem lehet a minimumnál kisebbre és a maximumnál nagyobbra paraméterezni a képet.
- Ha a _dashboard_ mérete fix, akkor nem lehet megváltoztatni.

#### Csatolmány egyedi fájlnevének beállítása

A csatolmányok alapvető nevének szerkezete: `YYYYMMDDHHMMSSUUUUU\_[workbook]-[view].[kiterjesztés]`. Ezt felülírhatjuk URL paraméterben a __|filename=[fájlnév]__ rész hozzáfűzésével. Pl.: VIZ\_CSV(AnnualReport/Overview|filename=2016 Annual Report). Ez csatolmányunknak a 2016 'Annual Report.csv' fájlnevet fogja a adni.

#### PDF dokumentumok összefűzése

Ha egy _alert_ egyszerre több PDF dokumentumot kap csatolmányként, felmerülhet az igény ezek összefűzésére egyetlen dokumentumban. Ezt az egyedi fájlnév mögé írt __|mergepdf__ paraméterrel tudjuk megtenni. Pl.: VIZ\_PDF(VizAlertsDemo/Overview?Region=East|filename=EastSales|mergepdf) és VIZ\_PDF(VizAlertsDemo/Product?Region=East|filename=EastSales|mergepdf) egybe lesznek fűze az EastSales.pdf dokumentumban.

#### Egyesített emailek

Egyesített emaileket az __Email Consolidate ~__ mező felvételével küldhetünk. Ennek akkor van értelme, ha egy _alert_ egyszerre több emailt generál, és nem szeretnénk a címzettek fiókját túlterhelni különálló levelek egymást követő tömeges kiküldésével. Ehhez az `Email Subject *`, `Email To *`, `Email From *`, `Email CC ~`, és `Email BCC ~` mezőknek kell egyezniük.

##### Egyesített emailek sorbarendezése

Az __Email Sort Order ~__ mező felvételével rendezhetjük sorba az egyesített emaileket. Bármilyen alfanumerikus karakter beírásával a program ezek szerint fogja az egy email _body_ba kerülő emaileket növekvő sorba rendezni. Pl.: 1,2,3 vagy a,b,c, vagy Lilla,Nóra,Orsolya.
