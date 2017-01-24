# Labor 7 - Service, Location

### Felkészülés a laborra

A labor célja a szolgáltatások készítésének bemutatása Android környezetben (Service komponens),
valamint a helymeghatározási lehetőségek ismertetése.

### Szolgáltatások bevezetés

Android platformon két fő Service típus létezik, melyek közül az egyik tovább bontható, röviden:

* **Started Service**: Egyszerűen indítható szolgáltatás. Fő szálban fut,
fejlesztő felelőssége saját szálat létrehozni. Beállítható, hogy magas prioritással,
foreground módban fusson, illetve megadható, hogy újraindítás esetén milyen módon/prioritással
indítsa újra a rendszer. Például ha alacsony memóriaszint miatt lett kilőve, mi történjen, hogyan/mikor induljon újra.
* **Intent Service**: Started Service speciális típusa. Intent-el paraméterezhető,
hogy milyen feladatot lásson el. A kéréseket sorosítja és már automatikusan külön szálon hajtja végre a
megadott kódrészt.
* **Bound Service**: Lehetőséget biztosít, hogy más komponensek csatlakozzanak a service-hez és
egy egységes interface-n keresztül kommunikáljanak a service-el. Ha minden komponens lecsatlakozott róla,
a service leáll.

**Fontos**: Egy service lehet egyszerre Started Service és Bound Service módban is!

### Helymeghatározás bevezetés
Android platformon két fő API létezik helymeghatározásra egy régebbi és egy újabb. A régebbi API
egyszerűen a LocationService segítségével nyújtott lehetőséget helymeghatározásra (GPS és hálózati egyaránt).
Az új Fused Location API a Google Play Services segítségével nem csak modern helymeghatározási
algoritmusokat alkalmaz, hanem biztosítja, hogy az alkalmazások egymás között a hely
adatokat megoszthassák egymással, ezáltal még gyorsabbá téve a pozíció információ lekérdezését.

A labor során a régebbi API-t fogjuk használni, mivel emulátoron a Google Play Services csak
virtualizáció nélkül érhető el és így lassú lenne a tesztelés. Fejlesztés szempontjából minimális
eltérés van a két API között, teljesen hasonlók az osztályok és az interfészek.

### Laborfeladat leírása
A labor során első lépésként egy egyszerű szolgáltatást hozunk létre a szabad lemezterület lekérdezésére,
majd egy helymeghatározásért felelős szolgáltatást készítünk, megjelenítjük a pozíció adatokat és egy értesítést,
valamint “lebegő ablak”-ot is létrehozunk a szolgáltatáshoz.
