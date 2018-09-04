# Labor 10 - Játékfejlesztés

## Bevezető

A labor során ízelítőt szeretnénk adni az Android platformon történő játékfejlesztési lehetőségekből. Egy 2D-s játékot fogunk elkészíteni, amiben a felhasználó az eszköz _gyroscope_ szenzorát felhasználva tudja az őt reprezentáló űrhajót irányítani, hogy elkerülje az ellenséges űrhajókat.
 
Tekintve a platform adottságait, az egyébként elérhető és sikeres megoldásokat, valamint a labor időkorlátait, a labor során nem térünk ki a 3D játékfejlesztésre. Természetesen a platformon egyébként erre is van lehetőség.

## Érintett témakörök

*   Rajzolás SurfaceView-ra
*   Sensorok kezelése
*   Sprite-ok és animáció
*   FPS szabályozás

## Kiinduló projekt

Elsőnek töltsük le a labor során használt kiinduló projektet, majd nyissuk meg Android Studio-val. 

[Kiinduló projekt](./assets/SpaceShipGame.zip) 

A források közül a test könyvtárat, illetve annak a tartalmát kitörölhetjük.

Fordítsuk le a projektet, majd a Laborvezető segítségével vizsgáljuk meg a projekt felépítését.


### Általános

A projektben található egyetlen, indító _activity_ a **GameActivity**. Ez egy elfordított (_landscape_) nézet, _Toolbar_ nélkül (lásd styles.xml), valamint a **GameView** nézetet tartalmazza (lásd **activity_game.xml** ). A **GameView** a kirajzolt játéktér megjelenítésért felelős nézet, de magát a kirajzolást nem ő fogja végezni, csak megjeleníteni a már kirajzolt képet.

### Model

A _model_ csomagban található az előre elkészített játékmodell. Minden a játékban megjeleníthető entitás a **Renderable** interface megvalósítója. A kirajzolást végző szál majd mindent, mint **Renderable**-t fog kezelni (ezt később készítjük majd el). Minden entitásnak lehetőség van megadni, hogy mekkora a rajzfelület **size(x,y)**, hogy ebből a saját méreteit kiszámolhassa. Lehetőség van minden egyes kirajzolás után az objektum állapotát léptetni (**step()**), valamint minden objektumnak ki kell tudnia rajzolni magát egy _Canvas_ objektumra (**render(canvas: Canvas)**).

### Háttér

A legegyszerűbb játékelem a háttér (**Background**). A háttérnek állapota nincs és egy teljes képernyőt kitöltő méretű képet rajzol ki. Ha a képi erőforrás kisebb mint a kirajzolt kép, akkor azt mind vízszintesen, mind függőlegesen tükrözi. A képek kirajzolása a _BitmapDrawable_ segítségével történik, amit a _BitmapFactory.decodeResource_-al hozunk létre.

### Űrhajó

A játékban kétfelé űrhajó is található, a játékos (**Player**) és az ellenség (**Enemy**). Mindkét entitás az abstract **Ship**-ből származik. Minden **Shiphez** tartozik egy kép, amit kirajzol magáról, valamint egy számlálóban lépteti, hogy hányadik kirajzolásról van szó, valamint a képernyőn lévő x,y pozícióját (**posX,posY**) is tárolja. Ezen túl még van egy **elevation** propertyje, amivel mozgatni fogjuk az űrhajót. A **Player** és az **Enemy** osztályok a megfelelő bitmapet töltik be, valamint tartalmazzák, hogy az adott képen, amit betöltenek, hol helyezkedik el az űrhajó alapállapota. A **Player** objektum az **elevation** értékétől függően függőlegesen mozog, az **Enemy** objektum pedig egy véletlen szerű magasságon mozog egy adott sebességgel jobbról balra.

## A Renderer elkészítése

Készítsük el az objektumok kirajzolását végző **Renderer** osztályt a _rendering_ csomagban. Ez az osztály tárolja a kirajzolni kívánt objektumokat, és azokat megfelelő sorrendben a képernyőre is rajzolja, illetve lépteti a megfelelő objektumokat. A léptetés hatására véletlenszerűen egy új **Enemy** objektumot ad hozzá a játéktérhez. Az _elevation_ perperty pedig a **Player** magasságát állítja be. 


```kotlin
class Renderer(private val context: Context, private val width: Int, private val height: Int) {

    private val random: Random = Random()
    private val entitiesToDraw = mutableListOf<Renderable>()

    private val background = Background(context)
    private val player = Player(context)

    init {
        background.size(width, height)
        player.size(width, height)

        val enemy = Enemy(context)
        enemy.size(width, height)
        entitiesToDraw.add(enemy)
        entitiesToDraw.add(player)
    }

    fun step() {
        if (random.nextFloat() > 0.993) {
            val enemy = Enemy(context)
            enemy.size(width, height)
            entitiesToDraw.add(enemy)
        }
        entitiesToDraw.forEach { it.step() }
    }

    fun draw(canvas: Canvas) {
        background.render(canvas)
        entitiesToDraw.forEach { it.render(canvas) }
    }

    fun setPlayerElevation(elevation: Float) {
        player.elevation = elevation
    }
}
```

### A kirajzoló szál

Készítsük el a kirajzolás ütemezéséért felelős szálat, a *rendering* csomagban **RenderLoop** néven.. Ezen a szálon fognak kirajzolásra kerülni a **Renderer** objektumai. Ez tartalmaz egy referenciát a **GameView**-ra, hogy abban megjelenítse a kirajzolt képet, valamint az előbb létrehozott **Renderer**-t használja fel. Maga is egy szálból származik, és a _run()_ függvényében egy végtelen ciklusban rajzolja ki újra és újra a játékelemeket. A rajzolás kezdetén először lépteti a játéktér állapotát, majd a **SurfaceView** **SurfaceHolder** objektuma segítségével kirajzolja magát. Fontos, hogy a kirajzolás előtt és után le kell zárni a **SurfaceHolder**-hez tartozó **Canvas**-t. 



```kotlin
class RenderLoop(context: Context, private val view: GameView, width: Int, height: Int) : Thread() {
    private val renderer = Renderer(context, width, height)

    var running = false

    override fun run() {
        while (running) {
            draw()
        }
    }

    private fun draw() {
        renderer.step()
        var c: Canvas? = null

        try {
            c = view.holder.lockCanvas()
            synchronized(view.holder) {
                renderer.draw(c!!)
            }
        } finally {
            if (c != null) {
                view.holder.unlockCanvasAndPost(c)
            }
        }
    }

    fun setPlayerElevation(elevation: Float) {
        renderer.setPlayerElevation(elevation)
    }
}
```


Egészítsük ki a **GameView**-t úgy hogy a renderelő szálat használja. Adjunk hozzá egy tagváltozót.

` private var renderLoop: RenderLoop? = null`

Ezután a **SurfaceHolder** callback eseményeit valósítjuk meg. 
  
```kotlin
private fun init() {
    val holder = holder
    holder.addCallback(object : SurfaceHolder.Callback {
        override fun surfaceCreated(holder: SurfaceHolder) {

        }

        override fun surfaceDestroyed(holder: SurfaceHolder) {
            var retry = true
            renderLoop?.running = false
            while (retry) {
                try {
                    renderLoop?.join()
                    retry = false
                } catch (e: InterruptedException) {
                    e.printStackTrace()
                }
            }

        }

        override fun surfaceChanged(holder: SurfaceHolder, format: Int, width: Int, height: Int) {
            renderLoop = RenderLoop(context, this@GameView, width, height)
            renderLoop!!.running=true
            renderLoop!!.start()
        }
    })
}
```
  
  
Majd a játékos űrhajójának helyzetét állító hívást vezessük ki a **GameView**-ra. 

```kotlin
fun setPlayerElevation(elevation: Float) {
    renderLoop?.setPlayerElevation(elevation)
}
```

**Próbáljuk ki az alkalmazást**

![](./images/screen1.png)


## Irányítás

Mozgassuk a felhasználó űrhajóját a gyorsulásmérő és magnetométer segítségével. Az alábbi osztályt készítsük el a _sensor_ csomagba. Majd a **GameActivity** _onResume()_ és _onPause()_ függvényében indítjuk el, majd állítjuk le. 

```kotlin
class GyroscopeHelper(context: Context, private val gameView: GameView) : SensorEventListener {
    private val sensorManager: SensorManager = context.getSystemService(Context.SENSOR_SERVICE) as SensorManager
    private val accelerometer=sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)
    private val magnetometer=sensorManager.getDefaultSensor(Sensor.TYPE_MAGNETIC_FIELD)

    private val lastAccelerometerValue = FloatArray(3)
    private val lastMagnetometerValue = FloatArray(3)

    private var lastAccelerometerSet = false
    private var lastMagnetometerSet = false

    private val rotation = FloatArray(9)
    private val orientation = FloatArray(3)

    fun start() {
        sensorManager.registerListener(this, accelerometer, SensorManager.SENSOR_DELAY_FASTEST)
        sensorManager.registerListener(this, magnetometer, SensorManager.SENSOR_DELAY_FASTEST)
    }

    fun stop() {
        sensorManager.unregisterListener(this)
    }

    override fun onSensorChanged(event: SensorEvent) {
        if (event.sensor == accelerometer) {
            System.arraycopy(event.values, 0, lastAccelerometerValue, 0, event.values.size)
            lastAccelerometerSet = true
        } else if (event.sensor == magnetometer) {
            System.arraycopy(event.values, 0, lastMagnetometerValue, 0, event.values.size)
            lastMagnetometerSet = true
        }
        if (lastAccelerometerSet && lastMagnetometerSet) {
            SensorManager.getRotationMatrix(rotation, null, lastAccelerometerValue, lastMagnetometerValue)
            SensorManager.getOrientation(rotation, orientation)

            val roll = orientation[2] * 1.0f

            val position = roll - -1.0f
            gameView.setPlayerElevation(position)
        }
    }

    override fun onAccuracyChanged(sensor: Sensor, accuracy: Int) {}
}
```


Majd ezt használjuk a **GameActivity**-ben. Figyeljük meg a **lateinit** módosítót, mely lehetővé teszi hogy egy non-nullable (kérdőjel nélküli) változót a konstruktor után inicializáljuk, Android környezetben ez tipikusan az onCreate ben inicializált elemek esetén jön jól. 

> A **lateinit** használata során vigyázzunk, mert előfordulhat hogy nem incializáljuk a változót, így a rendszer típus biztonságát kiiktatva becsapjuk magunkat és a non-nullable változónk értéke mégis null lesz!

```kotlin
class GameActivity : AppCompatActivity() {

    private lateinit var gyroscopeHelper: GyroscopeHelper

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_game)
        gyroscopeHelper = GyroscopeHelper(this, gameView)
    }

    override fun onResume() {
        super.onResume()
        gyroscopeHelper.start()
    }

    override fun onPause() {
        gyroscopeHelper.stop()
        super.onPause()
    }
}
```

**Próbáljuk ki az alkalmazást.**

![](./images/screen2.png)


### Animáció

Az animációk során **Sprite**-okat, olyan képeket használunk, ahol az animáció egyes állapotai mind megtalálhatóak ugyanazon a képen. Spriteok esetén az animáció a gyakorlatban abból áll, hogy a forrásképen egy maszkot tologatunk attól függően hogy milyen állapotban van az animáció. Így megspóroljuk a folyamatos kép betöltést, és a GC-t sem terheljük túl.

A **Ship** osztály valamint a **Player** és **Enemy** már fel vannak készítve arra hogy különböző állapotok között animáljanak. A megfelelő képi erőforrások is biztosítottak, már csak a képeket kell elcsúsztatni az adott állapotba, a **Ship** osztály **render** függvényében: 

```java
override fun render(canvas: Canvas) {
    setSpriteSizes()

    val statePos = state % 4
    //4 states, 64*29 each image

    val x = 0
    val y = spriteHeight * statePos

    val src = Rect(x, y, x + spriteWidth, y + spriteHeight)
    val dst = Rect(posX, posY, posX + spriteWidth * 4, posY + spriteHeight * 4)


    canvas.drawBitmap(image, src, dst, null)
}
```

**Próbáljuk ki az alkalmazást!** 

![](./images/animate.gif)


## FPS korlát elhelyezése

Azért hogy a kirajzolás sebességét egy fix értékre állítsuk két kirajzolás között aludnia kell a kirajzoló szálnak, amennyiben a kirajzolás nem tartott annyi ideig mint a kívánt FPS érték időköze. 

A **RenderLoop** osztályt egészítsük ki a következőkkel:

```kotlin
private val FPS: Long = 30
private val timeBetweenFrames = 1000 / FPS

private fun sleepThread(time: Long) {
    try {
        Thread.sleep(time)
    } catch (e: InterruptedException) {
    }

}

private fun getTime(): Long {
    return System.currentTimeMillis()
}
```

Valamint a **run()** fügyvényt egészítsük ki:

```kotlin
override fun run() {
    while (running) {
        val renderStart = getTime()
        draw()
        val renderEnd = getTime()

        val sleepTime = timeBetweenFrames - (renderEnd - renderStart)
        if (sleepTime > 0) {
            sleepThread(sleepTime)
        } else {
            sleepThread(5)
        }
    }
}
```
 
A renderelés kezdete és vége előtt eltelt időt nézzük és ha ez kisebb mint amennyi az adott FPS számhoz szükséges, úgy a megfelelő ideig altatjuk a szálat. Amennyiben tovább tartott a renderelés, akkor is adunk valamennyi alvás időt a CPU-nak.

**Próbáljuk ki az alkalmazást!** 

![](./images/animate.gif)


## Önálló feladatok

### Feladat 1 - Ütközés detektálás

Detektálja, ha a felhasználó űrhajója ütközik egy ellenséges űrhajóval, ekkor jelenítsen meg egy **Toast** üzenetet, majd állítsa le a játékot!

### Feladat 2 - Játékos képernyőn maradása

Biztosítsa, hogy a játékos űrhajóját ne lehessen kimozgatni a játéktérből!	
### Feladat 3 - Képernyő ébrenmaradás

Biztosítsa, hogy a jaték alatt ne aludjon el a képernyő, akkor sem ha huzamosabb időn át nem érünk hozzá!

Segítség: [Keeping the Device Awake](https://developer.android.com/training/scheduling/wakelock.html)

### Feladat 4 - Okosabb memória kezelés
Jelenleg minden ellenséges űrhajó képe külön be van töltve a memóriába. 

Valósítsa meg hogy ezek csak egy példányban legyenek betöltve (pl. statikus objektum), illetve a képernyőről eltűnő ellenséges űrhajókat szabadítsa fel, azok ne foglaljanak helyet a memóriában.

A képernyőről eltűnő űrhajókat a poziciójuk alapján (aktuális pocizió kisebb-e mint a bal szél=0 - a hajó mérete) vegye ki a listából (pl. iterátorral). Ha nincs már rájuk referencia, a GC felszabadítja.

