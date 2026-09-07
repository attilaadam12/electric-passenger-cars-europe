# Electric Passenger Cars – Europe

## A projektről

Az Eurostat adatai alapján mutatom be az elektromos személyautók számát és éves változását az adatállományban szereplő európai országokban.
A célom az adat előkészítés, az egyszerű adatmodell, a DAX-képletek és az interaktív dashboard készítésének gyakorlása volt.

## Adatforrás

- Forrás: Eurostat.
- Adattábla pontos linkje: **[Eurostat – road_eqs_carpda](https://ec.europa.eu/eurostat/databrowser/view/road_eqs_carpda/default/table)**
- A felhasznált forrásfájl elérése: **[data/road_eqs_carpda__custom_22584832_spreadsheet.xlsx](data/road_eqs_carpda__custom_22584832_spreadsheet.xlsx)**
- Az előtisztított forrásfájl elérése: **[data/Electricity.xlsx](data/Electricity.xlsx)**

## Adat előkészítés (összefoglalva)

Az eredeti Eurostat Excel-fájlt az elemzés előtt Excelben és Power Query-ben szerkesztettem.

Főbb lépések:

- Excelben eltávolítottam a szükségtelen sorokat és formázásokat, valamint átneveztem a munkalapokat.
- Power Query-ben, az évenként külön oszlopokban szereplő adatokat **Unpivot** művelettel `Country | Year | EV Cars` struktúrába alakítottam.
- Beállítottam a `Year` és `EV Cars` mezők megfelelő adattípusait.
- A nem elérhető Eurostat értékeket (`:`) `null` értékként kezeltem, mivel a hiányzó adat nem jelent nulla darab elektromos személygépkocsit.
- Létrehoztam a `Legal Form` oszlopokat (`Total`, `Physical Person`, `Legal Person`).
- A három különálló adattáblát **Append** művelettel egy közös `Appended_EV_Cars_Data` táblába fűztem. Ez lett a fő adattábla.
- A rendelkezésre álló Eurostat Flag információkat **Merge** művelettel, `Country`, `Year` és `Legal Form` alapján kapcsoltam a fő adattáblához.
- A Merge során Left Outer kapcsolatot használtam, így a fő adattábla sorai akkor is megmaradtak, ha nem tartozott hozzájuk flag információ.
- A csak adat előkészítéshez használt segédlekérdezések betöltését letiltottam.
- Létrehoztam a `dim_Country` és `dim_Year` dimenziótáblákat.
- A dimenziótáblákat one to many (`1:*`) kapcsoltam az `Appended_EV_Cars_Data` táblához.

## Adat előkészítés (kifejtve)

### 1. A forrásfájl felépítése

Az eredeti, Eurostatból letöltött `.xlsx` fájl a következő munkalapokat tartalmazza:

- Summary
- Structure
- Sheet 1
- Flags 1
- Sheet 2
- Flags 2
- Sheet 3
- Flags 3

A **Summary** és **Structure** munkalapok az adatállomány megértését segítő leíró információkat tartalmaznak. Ezeket a forrásadatok szerkezetének és tartalmának megértéséhez használtam, de közvetlenül nem kerültek be az elemzésbe.

A **Sheet** munkalapok tartalmazzák az elemzéshez felhasznált személygépkocsi adatokat. Az adatok jogi forma szerint elkülönítve, évenként szerepelnek.

A hozzájuk tartozó **Flags** munkalapok az egyes megfigyelések státuszára vagy minőségére vonatkozó kiegészítő információkat tartalmaznak.

A használt Eurostat jelölések:

- `b` = break in time series
- `i` = value imputed by Eurostat or other receiving agencies
- `p` = provisional


### 2. Előzetes adat-előkészítés Excelben

A Power Query-be történő betöltés előtt az eredeti Excel-fájlon előkészítést végeztem.

Az előkészítés során:

- megszüntettem a rögzítéseket,
- töröltem a szükségtelen sorokat,
- eltávolítottam a szükségtelen formázásokat,
- átneveztem a munkalapokat, hogy azok tartalma könnyebben azonosítható legyen.

Az előkészített munkafüzetet az alábbi néven mentettem:

`Electricity.xlsx`

A munkalapokat az alábbiak szerint neveztem át:

| Eredeti munkalap | Új munkalapnév |
|---|---|
| Sheet 1 | Electricity Total |
| Sheet 2 | Electricity Physical Person |
| Sheet 3 | Electricity Legal Person |
| Flags 1 | Electricity Total2 |
| Flags 2 | Electricity Physical Person2 |
| Flags 3 | Electricity Legal Person2 |

Ezt az előkészített Excel-fájlt töltöttem be a Power Query-be.


### 3. Az adatszerkezet átalakítása Power Query-ben

Az évenként külön oszlopokban szereplő adatokat **Unpivot** művelettel alakítottam át.

Az eredeti struktúrában minden év külön oszlopban szerepelt:

`Country | 2016 | 2017 | 2018 | ... | 2025`

Az átalakítást követően az adatok hosszú (long format) struktúrába kerültek:

`Country | Year | EV Cars`

Ez a struktúra megfelelőbb a Power BI-ban történő szűréshez, kapcsolatok kialakításához, vizualizációkhoz és DAX-számításokhoz.

A `Year` és `EV Cars` oszlopokhoz megfelelő adattípusokat állítottam be.

```
Az Eurostatban `:` karakterrel jelölt, nem elérhető értékeket hiányzó (`null`) értékként kezeltem, 
és nem cseréltem őket nullára (`0`), mivel ezek nem nulla darab elektromos személygépkocsit,
hanem nem elérhető adatot jelentenek.
```

### 4. Legal Form oszlopok létrehozása

Annak érdekében, hogy az összefűzés után is megkülönböztethetők legyenek az egyes adatcsoportok, minden táblában létrehoztam egy **Legal Form** oszlopot.

A létrehozott oszlopok:

- `Total`
- `Physical Person`
- `Legal Person`

A három adattáblát ezt követően az **Append Queries as New** művelettel egymás alá fűztem.

Az így létrehozott új tábla neve:

`Appended_EV_Cars_Data`


### 5. Eurostat flag adatok hozzáadása

Az `Electricity Physical Person2` és `Electricity Legal Person2` flag táblák nem tartalmaztak felhasználható flag adatokat, ezért ezekkel a további adat előkészítés során nem dolgoztam.

Az `Electricity Total2` tábla azonban tartalmazta az elérhető Eurostat flag információkat.

Ebben a táblában is létrehoztam egy `Legal Form` oszlopot, amelynek értéke `Total`. Erre azért volt szükség, hogy a flag információkat a megfelelő ország, év és jogi forma adataihoz lehessen kapcsolni.

Ezután az `Appended_EV_Cars_Data` és az `Electricity Total2` táblákat **Merge Queries** művelettel egyesítettem.

Az egyesítés beállításai:

- kapcsolattípus: `Left Outer (all from first, matching from second)`
- kapcsolódó mezők: `Country`, `Year` és `Legal Form`

```
Merge Queries művelettel a két tábla adatait közös mezők alapján kapcsoltam össze.
Ebben az esetben a `Country`, `Year` és `Legal Form` mezők alapján az Eurostat flag információkat hozzákapcsoltam
az `Appended_EV_Cars_Data` megfelelő soraihoz.

Left Outer opció, így az Appended_EV_Cars_Data minden sora megmaradt akkor is,
ha az adott adathoz nem tartozott flag információ.
```

Az egyesítés után a flag oszlopot kibontottam, az eredeti oszlopnév előtagként történő használata nélkül.

Ezzel az elérhető Eurostat státusz és adatminőségi információk a hozzájuk tartozó EV-adatok mellett is megmaradtak.


### 6. Lekérdezések betöltése

A végleges adatmodellhez szükséges táblák betöltését engedélyeztem, míg a kizárólag az adat előkészítéshez használt köztes és segédlekérdezések betöltését letiltottam.

A fő adattábla:

`Appended_EV_Cars_Data`

A köztes lekérdezések továbbra is megmaradtak a Power Query-ben, így az adat előkészítés lépései visszakövethetők, azonban nem jelennek meg szükségtelen külön táblákként a Power BI adatmodellben.


### 7. Dimenziótáblák és kapcsolatok

Az adatmodellhez két dimenziótáblát hoztam létre:

- `dim_Country`
- `dim_Year`

A dimenziótáblák egyszerűbbé teszik az adatok szűrését és használatát, valamint megfelelőbb struktúrát biztosítanak a modell későbbi bővítéséhez.

Mindkét dimenziótábla **one to many (1:*)** kapcsolatban áll az `Appended_EV_Cars_Data` táblával:

`dim_Country (1) → (*) Appended_EV_Cars_Data`

`dim_Year (1) → (*) Appended_EV_Cars_Data`

A kapcsolatok aktívak, a szűrés iránya pedig a dimenziótábláktól az adattábla felé mutat.

## DAX-képletek

### Total EV Cars

Az elektromos személyautók száma az aktuális szűrés szerint, az EU27 összesítő sora nélkül.

```dax
Total EV Cars =
CALCULATE(SUM(Appended_EV_Cars_Data[EV Cars]),
Appended_EV_Cars_Data[Legal Form] = "Total",
Appended_EV_Cars_Data[Country] <> "European Union - 27 countries (from 2020)")
```

### EV Annual Change

Az előző évhez képesti változás darabszámban. Ha az aktuális vagy az előző évi érték hiányzik, az eredmény üres.

```dax
EV Annual Change =
VAR PreviousEV = CALCULATE([Total EV Cars], dim_Year[Year] = MAX(dim_Year[Year]) - 1)
RETURN
IF(ISBLANK([Total EV Cars]) || ISBLANK(PreviousEV),
BLANK(), 
[Total EV Cars] - PreviousEV)
```

### EV Annual Change %

Az éves változás az előző évi érték arányában. A dashboardon százalékos formázással jelenik meg.

```dax
EV Annual Change % =
VAR PreviousEV =
CALCULATE([Total EV Cars], dim_Year[Year] = MAX(dim_Year[Year]) - 1)
RETURN
IF(ISBLANK([Total EV Cars]) || ISBLANK(PreviousEV),
BLANK(),
DIVIDE([EV Annual Change], PreviousEV))
```

### Selected Year

A kiválasztott év megjelenítése.

```dax
Selected Year =
SELECTEDVALUE(dim_Year[Year])
```

## A dashboard elemei

- Ország és év választó szűrők.
- Kártyák: elektromos személyautók száma, éves változás és éves változás %.
- A kiválasztott év külön megjelenítése.
- Térkép az országonkénti autószámok összehasonlításához.
- Vonaldiagram az éves százalékos változás bemutatásához.

##  Bemutató

### Dashboard

![Electric Passenger Cars dashboard](images/Electric%20Passenger%20Cars.png)

### Videó

[Rövid videós bemutató](https://github.com/attilaadam12/electric-passenger-cars-europe/raw/refs/heads/main/videos/Electric%20Passenger%20Cars.mp4)

## Megjegyzések

- Az „All” az adatállományban szereplő országok összegét jelenti, nem a hivatalos EU27-értéket.
- Az adatok országonként és évenként hiányosak lehetnek. A hiányzó érték nem jelent nullát.
- Az éves összehasonlítás során az évválasztóban egyszerre egy évet lehetséges kiválasztani.
- Nulla előző évi értéknél a százalékos változás üres marad. 
- Alacsony kiinduló érték mellett nagy százalékos növekedés is előfordulhat.
- Az autóállomány éves változása nem azonos az adott évi újautó eladások számával.

## Fájlok

| Fájl vagy mappa | Tartalom |
| --- | --- |
| `README.md` | A projekt leírása és a DAX-képletek |
| `EV_Cars_EU.pbix` | Power BI-projektfájl |
| `data/` | Felhasznált forrásfájlok, vagy az elérésüket tartalmazó leírás |
| `images/` | Képernyőkép, GIF |
| `videos/` | Videó |

A `.pbix` fájl Power BI Desktopban nyitható meg. Az adatok frissítéséhez szükség lehet a forrásfájl útvonalának módosítására.
