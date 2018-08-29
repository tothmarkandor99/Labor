# Labor 5 - Perzisztencia

## Felkészülés a laborra

A labor célja a relációs adatbáziskezelés bemutatása Android platformon. A feladatok során a korábban elkészített Todo lista alkalmazást fejlesztjük tovább. 

Az alkalmazás SQLite adatbázisban fogja tárolni a Todo elemeket, így azok nem fognak elveszni, ha a felhasználó elnavigál, vagy elforgatja a készüléket.

## Kiinduló projekt

Töltsük le a kiinduló projektet:

[Kiinduló project](./assets/Todo_kiindulo.zip)

Ez a Todo alkalmazás még csak memóriában tárolja a futása során létrehozott Todo elemeket.

Tömörítsük ki a projektet tartalmazó mappát, indítsuk el az Android Studio-t, majd nyissuk meg a projektet.

## Adattárolás SQLite adatbázisban

Célunk, hogy a a Todo objektumok memóriában tárolása helyett az alkalmazás egy SQLite adatbázisban perzisztensen mentse őket. Tehát az, hogy ne veszítsük el a felvett Todo-kat az alkalmazás bezárásakor. 

A `SimpleItemRecyclerViewAdapter` a módosítások után nem a memóriában tárolt Todo listát fogja megjeleníteni, hanem majd az adatbázisból olvassa ki és köti össze azt a `RecyclerView`-val.

A feladat megvalósítása három részből áll:

1. Todo objektumok perzisztens tárolásának megvalósítása adatbázissal
2. A memóriában tárolt listával dolgozó adapter átalakítása adatbázissal (`Cursor`ral) működővé
3. Workflow átírása

### Todo-k tárolása adatbázisban

Az SQLite adatbáziskezelő használatához segítséget nyújt a platform, mégpedig az `SQLiteOpenHelper` osztállyal. Ebből származtatva olyan osztályt hozhatunk létre, ami referenciát szolgáltat az általunk használt teljes adatbázisra. Így több entitás osztály és adatbázis tábla esetén is elég egyetlen ilyen segédosztályt implementálni.

Hozzunk létre a projektben a `hu.bme.aut.android.todo` package-en belül egy új package-et `database` néven.

A `hu.bme.aut.android.todo.database` package-en belül hozzunk létre egy új osztályt `DatabaseHelper` néven, ami az `SQLiteOpenHelper` osztályból származik. 

A `DatabaseHelper` osztály:

```kotlin
class DatabaseHelper(
    context: Context,
    name: String,
    factory: SQLiteDatabase.CursorFactory,
    version: Int
) : SQLiteOpenHelper(context, name, factory, version) {

	override fun onCreate(db: SQLiteDatabase) {
		// TODO Auto-generated method stub
	}
	
	override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
		// TODO Auto-generated method stub
	}
}
```

Az `SQLiteOpenHelper` osztályból történő származtatás az ősosztály konstruktorának meghívásán kívül két metódus kötelező felüldefiniálását írja elő. Az `onCreate()`-ben kell futtatni a séma létrehozó SQL szkriptet, míg az `onUpgrade()`-ben kezelhetők le az adatbázis verzióváltásával kapcsolatos feladatok. Legegyszerűbb esetben itt törölhető, majd újra létrehozható az egész séma. Ekkor viszont a felhasználó elveszíti a tárolt adatait, ami nem elfogadható felhasználói élmény.

Az adatbáziskezelés során sok konstans jellegű változóval kell dolgoznunk, mint például a táblákban lévő oszlopok nevei, táblák neve, adatbázis fájl neve, séma létrehozó és törlő szkiptek, stb. Ezeket érdemes egy közös helyen tárolni, így szerkesztéskor vagy új entitás bevezetésekor nem kell a forrásfájlok között ugrálni, valamint egyszerűbb a teljes adatbázist létrehozó és törlő szkripteket generálni. Hozzunk létre egy új osztályt a `hu.bme.aut.android.todo.database` csomagban `DbConstants` néven, és konstans tagváltozóként vegyünk fel minden szükséges értéket:

```kotlin
object DbConstants {

  const val DATABASE_NAME = "data.db"
  const val DATABASE_VERSION = 1
  const val DATABASE_CREATE_ALL = Todo.DATABASE_CREATE
  const val DATABASE_DROP_ALL = Todo.DATABASE_DROP

  object Todo {
	const val DATABASE_TABLE = "todo"
	const val KEY_ROWID = "_id"
	const val KEY_TITLE = "title"
	const val KEY_PRIORITY = "priority"
	const val KEY_DUEDATE = "dueDate"
	const val KEY_DESCRIPTION = "description"

	const val DATABASE_CREATE =
		"""create table if not exists $DATABASE_TABLE ( 
			$KEY_ROWID integer primary key autoincrement, 
			$KEY_TITLE text not null, 
			$KEY_PRIORITY integer, 
			$KEY_DUEDATE text, 
			$KEY_DESCRIPTION text
			); """

	const val DATABASE_DROP = "drop table if exists $DATABASE_TABLE;"
  }
}
```

Figyeljük meg hogy a `DbConstants` osztályon belül létrehoztunk egy belső `Todo` nevű osztályt, amiben a Todo entitásokat tároló táblához tartozó konstans értékeket tároljuk. Amennyiben az alkalmazásunk több entitást is adatbázisban tárol (gyakori eset!), akkor érdemes az egyes osztályokhoz tartozó konstansokat külön-külön belső osztályokban tárolni. Így sokkal átláthatóbb és karbantarthatóbb lesz a kód, mint ha ömlesztve felvennénk a `DbConstants`-ba az összes tábla összes konstansát. Ezek a belső osztályok praktikusan ugyanolyan névvel léteznek, mint az entitás osztályok (jelen esetben mindkettő neve Todo). Mivel más package-ben vannak, ez nem okoz fordítási hibát. Részben ezért is hoztuk létre a `database` nevű package-et.

Erdemes megfigyelni továbbá azt, hogy az osztályokat nem a `class` kulcsszóval deklaráltuk. Helyette az`object`-et használjuk, amivel a Kotlin nyelv azt biztosítja számunkra, hogy a `DbConstants` és a benne lévő `Todo` osztályok singletonként viselkednek, azaz az alkalmazás futtatásakor létrejön belőlük egy példány, további példányokat pedig nem lehet létrehozni belőlük. Az `object`ként deklarált osztályok egyetlen példányára, ahogy később látszani is fog, egyszerűen a nevükkel lehet hivatkozni az alkalmazás kódjában.

Következő lépésként Implementáljuk a `DatabaseHelper` osztály metódusait. A konstruktor paraméterei közül törölhetjük a `CursorFactory`t és a verziószámot, és az ősosztály konstruktorának hívásakor a megfelelő helyen adjunk át null-t. Így az osztály az alapértelmezett `CursorFactory`t fogja használni. Verzióként adjuk át a `DbConstants.DATABASE\_VERSION` konstans értékét. Az `onCreate()` és `onUpgrade()` metódusokban használjuk a `DbConstants`-ban ebből a célból létrehozott konstansokat.

A módosított `DatabaseHelper` osztály:

```kotlin
class DatabaseHelper(
    context: Context,
    name: String
) : SQLiteOpenHelper(context, name, null, DbConstants.DATABASE_VERSION) {

	override fun onCreate(db: SQLiteDatabase) {
		db.execSQL(DbConstants.DATABASE_CREATE_ALL)
	}
	
	override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
		db.execSQL(DbConstants.DATABASE_DROP_ALL);
		db.execSQL(DbConstants.DATABASE_CREATE_ALL);
	}
}
```

Az adatbázis séma létrehozásához és megnyitásához szükséges osztályok így már rendelkezésre állnak. A következő feladatunk az entitás osztályok és az adatbázis közötti kapcsolat implementálása. Ehhez meg kell írnunk azokat a kódrészleteket, melyek egy memóriában lévő Todo objektumot képesek adatbázisba írni, onnan visszaolvasni, módosítani valamint törölni. (Természetesen más, akár összetettebb funkciók is implementálhatók itt.) Ezt a kódot az entitás osztály helyett a karbantarthatóságot szem előtt tartva érdemes egy külön osztályban megvalósítani a `database` csomagon belül. 

A `TodoDbLoader` osztály: 

```kotlin
class TodoDbLoader(private val context: Context) {
	private lateinit var dbHelper: DatabaseHelper
	private lateinit var db: SQLiteDatabase
	
	@Throws(SQLException::class)
	fun open() {
		dbHelper = DatabaseHelper(context, DbConstants.DATABASE_NAME)
		db = dbHelper.writableDatabase
	
		dbHelper.onCreate(db)
	}
	
	fun close() {
		dbHelper.close()
	}
	
	// CRUD és egyéb metódusok
}
```

`SQLException`ből használjuk az `android.database` package-ben lévőt.

A létrehozó, módosító és törlő metódusok a `TodoDbLoader`en belül, a *CRUD és egyéb metódusok* komment után:

```kotlin
// INSERT
fun createTodo(todo: Todo): Long {
	val values = ContentValues()
	values.put(DbConstants.Todo.KEY_TITLE, todo.title)
	values.put(DbConstants.Todo.KEY_DUEDATE, todo.dueDate)
	values.put(DbConstants.Todo.KEY_DESCRIPTION, todo.description)
	values.put(DbConstants.Todo.KEY_PRIORITY, todo.priority.ordinal)

	return db.insert(DbConstants.Todo.DATABASE_TABLE, null, values)
}

// DELETE
fun deleteTodo(rowId: Long): Boolean {
	val deletedTodos = db.delete(
			DbConstants.Todo.DATABASE_TABLE,
			"${DbConstants.Todo.KEY_ROWID} = $rowId",
			null
	)
	return deletedTodos > 0
}

// UPDATE
fun updateTodo(newTodo: Todo): Boolean {
	val values = ContentValues()
	values.put(DbConstants.Todo.KEY_TITLE, newTodo.title)
	values.put(DbConstants.Todo.KEY_DUEDATE, newTodo.dueDate)
	values.put(DbConstants.Todo.KEY_DESCRIPTION, newTodo.description)
	values.put(DbConstants.Todo.KEY_PRIORITY, newTodo.priority.ordinal)

	val todosUpdated = db.update(
			DbConstants.Todo.DATABASE_TABLE,
			values,
			"${DbConstants.Todo.KEY_ROWID} = ${newTodo.id}",
			null
	)

	return todosUpdated > 0
}
```

Fussuk át és értelmezzük a bemásolt kódot!

A kód bemásolása után importáljuk be a megfelelő osztályokat. A `Todo` osztály két helyen is szerepel, egyrészt entitás osztályként a `.model` végű csomagban, másrészt az adatbázis konstansokat tároló belső osztályként a `.database` végű csomagban. A bemásolt kód célja, hogy a Todo entitásokat létrehozza, módosítsa vagy törölje az adatbázisban, így az entitás osztályt importáljuk be (`hu.bme.aut.android.todo.model.Todo`).

Általában szükséges az egyetlen rekordot és a teljes rekord halmazra mutató `Cursor`t visszaadó függvények implementálása is.

Ezek implementációja a `TodoDbLoader` osztályban, a *CRUD műveletek* után:

```kotlin
// Get all Todos
fun fetchAll(): Cursor {
	// Cursor pointing to the result set of all Todos with all fields (where = null)
	return db.query(
			DbConstants.Todo.DATABASE_TABLE,
			arrayOf(
					DbConstants.Todo.KEY_ROWID,
					DbConstants.Todo.KEY_TITLE,
					DbConstants.Todo.KEY_DESCRIPTION,
					DbConstants.Todo.KEY_DUEDATE,
					DbConstants.Todo.KEY_PRIORITY
			),
			null,
			null,
			null,
			null,
			DbConstants.Todo.KEY_TITLE
	)
}

// Querying for one of the Todos with the given id
fun fetchTodo(id: Long): Todo? {
	// Cursor pointing to a result set with 0 or 1 Todo
	val c = db.query(
			DbConstants.Todo.DATABASE_TABLE,
			arrayOf(
					DbConstants.Todo.KEY_ROWID,
					DbConstants.Todo.KEY_TITLE,
					DbConstants.Todo.KEY_DESCRIPTION,
					DbConstants.Todo.KEY_DUEDATE,
					DbConstants.Todo.KEY_PRIORITY
			),
			"${DbConstants.Todo.KEY_ROWID} = $id",
			null,
			null,
			null,
			DbConstants.Todo.KEY_TITLE
	)

	// Return with the found entry or null if there wasn't any with the given id
	return if (c.moveToFirst()) {
		getTodoByCursor(c)
	} else {
		null
	}
}

companion object {
	fun getTodoByCursor(c: Cursor): Todo {

		val priority = when (c.getInt(c.getColumnIndex(DbConstants.Todo.KEY_PRIORITY))) {
			0 -> Todo.Priority.LOW
			1 -> Todo.Priority.MEDIUM
			2 -> Todo.Priority.HIGH
			else -> Todo.Priority.LOW
		}

		return Todo(
				id = c.getLong(c.getColumnIndex(DbConstants.Todo.KEY_ROWID)),
				title = c.getString(c.getColumnIndex(DbConstants.Todo.KEY_TITLE)),
				priority = priority,
				dueDate = c.getString(c.getColumnIndex(DbConstants.Todo.KEY_DUEDATE)),
				description = c.getString(c.getColumnIndex(DbConstants.Todo.KEY_DESCRIPTION))
		)
	}
}
```

## Lista feltöltése adatbázisból

Ha a Todo-kat megjelenítő lista adapterét adatbázisból szeretnénk feltölteni, akkor a `BaseAdapter` helyett a `CursorRecyclerViewAdapater` osztályból kell származtatnunk, így leegyszerűsítve a feladatot. Ez az osztály a következő linken érhető el:

[https://gist.github.com/skyfishjy/443b7448f59be978bc59#file-cursorrecyclerviewadapter-java](https://gist.github.com/skyfishjy/443b7448f59be978bc59#file-cursorrecyclerviewadapter-java)

Hozzuk létre a `CursorRecyclerViewAdapter` nevű osztályt a `hu.bme.aut.android.todo.feature.list` package-ben és másoljuk be a kódját a fenti linkről.

Figyeljük meg, hogy a másolt kód *Java* nyelven íródott. Ebből semmi probléma nem fog adódni, gond nélkül el fogjuk érni a Kotlinban írt kódunkban.

A megoldás hasonló az előző adapter kódjához, de itt az `onBindViewHolder` metódusban egy `Cursor`t kapunk pozíció helyett, és ennek segítségével szerezhetjük meg a megjelenítendő adatokat. Ezen felül az osztály nem egy listában tárolja a `Todo` elemek referenciáit, hanem egy `Cursor`t tart az elemek kiolvasásához.

Hozzuk létre a `TodoAdapter` osztályt a `hu.bme.aut.android.todo.feature.list` csomagban:

```kotlin
class TodoAdapter(context: Context, cursor: Cursor) : CursorRecyclerViewAdapter<TodoAdapter.ViewHolder>(context, cursor) {

	var itemClickListener: TodoItemClickListener? = null
	
	override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
		val view = LayoutInflater.from(parent.context).inflate(R.layout.row_todo, parent, false)
		return ViewHolder(view)
	}
	
	override fun onBindViewHolder(holder: ViewHolder, cursor: Cursor) {
		val todo = TodoDbLoader.getTodoByCursor(cursor)
	
		holder.todo = todo
	
		holder.tvTitle.text = todo.title
		holder.tvDueDate.text = todo.dueDate
	
		val resource = when (todo.priority) {
	  		Todo.Priority.LOW -> R.drawable.ic_low
	  		Todo.Priority.MEDIUM -> R.drawable.ic_medium
	 		Todo.Priority.HIGH -> R.drawable.ic_high
		}
		holder.ivPriority.setImageResource(resource)
	}
	
	inner class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
		val tvDueDate: TextView = view.tvDueDate
		val tvTitle: TextView = view.tvTitle
		val ivPriority: ImageView = view.ivPriority
	
		var todo: Todo? = null
	
		init {
		  	itemView.setOnClickListener {
				todo?.let { itemClickListener?.onItemClick(it) }
		  	}
		
		  	itemView.setOnLongClickListener { clickedView ->
				todo?.let { itemClickListener?.onItemLongClick(clickedView, it) }
				true
			}
		}
	}
	
	interface TodoItemClickListener {
		fun onItemClick(todo: Todo)
		fun onItemLongClick(view: View, todo: Todo): Boolean
	}
}
```

## Egyedi Application osztály készítése

Készítsünk egy egyedi Application osztályt `TodoApplication` néven. Az `Application` az alkalmazás futása során folyamatosan jelen lévő objektum, melyet a futtatókörnyezet hoz létre automatikusan az alkalmazás indulásakor. Élettartama az alkalmazáshoz tartozó process élettartamázhoz kötődik. A `TodoApplication` fogja fenntartani az adatbáziskapcsolatot reprezentáló TodoDbLoader objektumot, így nem kell minden `Activity`ből külön-külön, erőforrás pazarló módon példányosítani.

A `hu.bme.aut.android.todo` package-en belül hozzuk létre a `TodoApplication` osztályt:

```kotlin
class TodoApplication : Application() {

	companion object {
		lateinit var todoDbLoader: TodoDbLoader
	  		private set
	}
	
	override fun onCreate() {
		super.onCreate()
	
		todoDbLoader = TodoDbLoader(this)
		todoDbLoader.open()
	}
	
	override fun onTerminate() {
		todoDbLoader.close()
		super.onTerminate()
	}
}
```

Figyeljük meg, hogy a fejlesztőkörnyezet figyelmeztet egy lehetséges hibára!

Ebben a példa alkalmazásban nem fog gondot okozni egy `Context` referencia statikus tárolása, viszont érdemes tudni, hogy ez memóriaszivárgáshoz vezethet.

Az `AndroidManifest.xml`-ben állítsuk be, hogy a rendszer a `TodoApplication` osztályt példányosítsa az alkalmazás induláskor. Ehhez a manifestben lévő `<application>` node-ban fel kell vennünk egy attribútumot `android:name` néven, aminek az értéke legyen a `TodoApplication` minősített (fully-qualified) osztályneve:

```xml
<application
		android:allowBackup="true"
		android:icon="@mipmap/ic_launcher"
		android:label="@string/app_name"
		android:roundIcon="@mipmap/ic_launcher"
		android:supportsRtl="true"
		android:theme="@style/AppTheme"
		android:name=".TodoApplication">
```

Bár az `android:name` attribútum értékének egy fully-qualified osztálynévnek kell lennie, a rendszer elfogadja a manifestben beállított `package` névhez relatív értéket is. Figyeljük meg, hogy az `android:name` beállításakor a fejlesztőkörnyezet *ctrl+space* lenyomására segíti az osztálynév megadását.

Ezt követően a `TodoApplication` osztály `todoDbLoader` propertyjével bármikor elérhetjük az SQLite adatbázis kapcsolatunkat.

## TodoListActivity módosítása

Az adatbázis alapú perzisztencia használatához minden rendelkezésre áll, már
csak az `TodoListActivityben` kell áttérnünk az adatok adatbázisban tárolásra.

Adjuk hozzá a `TodoListActivity`hez az alábbi propertyket:

```kotlin
private lateinit var adapter: TodoAdapter

private lateinit var dbLoader: TodoDbLoader

private var loadTodosTask: LoadTodosTask? = null
```

Töröljük ki a `TodoListActivity`ből a régi adaptert, azaz az alábbi propertyt:

```kotlin
private lateinit var simpleItemRecyclerViewAdapter: SimpleItemRecyclerViewAdapter
```

Az adapter váltás miatt szükséges az, hogy a `TodoListActivity` ne a `SimpleItemRecyclerViewAdapter`ben definiált `TodoItemClickListener`t implementálja, hanem a `TodoAdapter`ben definiáltat.

A `TodoListActivity` új definíciója:

```kotlin
class TodoListActivity : AppCompatActivity(), TodoCreateFragment.TodoCreatedListener, TodoAdapter.TodoItemClickListener {
```

A `TodoListActivity`ben írjuk át az `onItemLongClick` függvényt is a `TodoAdapter.TodoItemClickListener` interfésznek megfelelően:

```kotlin
override fun onItemLongClick(view: View, todo: Todo): Boolean {
	val popup = PopupMenu(this, view)
	popup.inflate(R.menu.menu_todo)
	popup.setOnMenuItemClickListener { item ->
		when (item.itemId) {
			R.id.delete -> {
			  // TODO
			}
		}
		false
	}
	popup.show()
	return false
}
```

Módosítsuk a `setupRecyclerView()` metódust úgy, hogy a `RecyclerView` egy `TodoAdapter` segítségével adatbázisból kapja meg a Todo elemeket:

```kotlin
private fun setupRecyclerView() {
	adapter = TodoAdapter 
    adapter.itemClickListener = this
    rvTodoList.adapter = adapter
}
```

Ezek után már nincs szükség a korábban használt `SimpleItemRecyclerViewAdapter`re, törölhetjük az osztályt tartalmazó fájlt a `hu.bme.aut.android.todo.feature.list` package-ből.

Törölni kell az `onTodoCreated` függvény törzsét és módosítani kell  az `onItemLongClick` függvényben a `simpleItemRecyclerViewAdapter.deleteRow(position)` hívást, mert a `TodoAdapter` interfésze különbözik az eddig használt adapterétől.

Az `onTodoCreated` függvény törzséből törlendő sor:

```kotlin
simpleItemRecyclerViewAdapter.addItem(todo)
```

Az `onItemLongClick` függvényben módosítandó sor:

```kotlin
R.id.delete -> simpleItemRecyclerViewAdapter.deleteRow(position)
```

A módosítást követően az `onItemLongClick` függvény:

```kotlin
override fun onItemLongClick(view: View, todo: Todo): Boolean {
	val popup = PopupMenu(this, view)
	popup.inflate(R.menu.menu_todo)
	popup.setOnMenuItemClickListener { item ->
		when (item.itemId) {
			R.id.delete -> {
				// TODO
			}
		}
		false
	}
	popup.show()
	return false
}
```

A törölt és módosított kódrészeket később újra meg fogjuk írni az új adapternek megfelelő hívásokkal.

Az diszk I/O műveleteket javasolt a háttérszálon, aszinkron módon futtatni. ellenkező esetben könnyen megakaszthatják a UI szálat esetlegesen lassú lefutásukkal.

Készítsük el a `LoadTodosTask` osztályt a `hu.bme.aut.android.todo.database` csomagban, ami származzon az `AsyncTask` osztályból. A `LoadTodosTask` aszinkron módon kérdezi majd le az adatbázisunkból az összes elmentett `Todo` rekordot.

A `LoadTodoTask` osztály:

```kotlin
class LoadTodosTask(private val listActivity: TodoListActivity, private val dbLoader: TodoDbLoader) : AsyncTask<Void, Void, Cursor>() {

	companion object {
		private const val TAG = "LoadTodosTask"
	}

 	override fun doInBackground(vararg params: Void): Cursor? {
		return try {

      		val result = dbLoader.fetchAll()

      		if (!isCancelled) {
        		result
      		} else {
        		Log.d(TAG, "Cancelled, closing cursor")
        		result.close()
        		null
      		}
    	} catch (e: Exception) {
     		Log.d(TAG, "An error occurred while fetching Todos")
      		null
    	}
	}

	override fun onPostExecute(result: Cursor?) {
		Log.d(TAG, "Fetch completed, displaying cursor results")
    	listActivity.showTodos(result)
	}
}
```

A `TodoListActivity`ben hozzuk létre a `showTodos` függvényt:

```kotlin
fun showTodos(todos: Cursor?) {
	if (todos != null) {
		adapter = TodoAdapter(applicationContext, todos)
		setupRecyclerView()
	} else {
		// Handle the case when there was an error while fetching todos
		// It's not part of this exercise
	}
}
```

Ezt követően a `TodoListActivity` életciklus-hívásait az alábbi módon adjuk meg:

Az `onCreate()` függvényben kérjünk referenciát a `TodoApplication`től a nyitott adatbázis kapcsolatot tartalmazó `TodoDbLoader`-re, illetve töröljük ki a függvényből a `setupRecyclerView()` hívást.

Az `onCreate()` függvény a módosítások után:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
	super.onCreate(savedInstanceState)
	setContentView(R.layout.activity_todo_list)

	setSupportActionBar(toolbar)
	toolbar.title = title

	fab.setOnClickListener { view ->
		Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
				.setAction("Action", null).show()
	}

	if (todo_detail_container != null) {
		// The detail container view will be present only in the
		// large-screen layouts (res/values-w900dp).
		// If this view is present, then the
		// activity should be in two-pane mode.
		twoPane = true
	}

	dbLoader = TodoApplication.todoDbLoader
}
```

Írjuk meg az `onResume()` függvényt és indítsuk el benne a `Todo` lista tartalmának lekérdezését:

```kotlin
override fun onResume() {
	super.onResume()

	refreshList()
}
```

Írjuk meg az `onPause()` függvényt. Ha az alkalmazás háttérbe kerülésekor van folyamatban adatbázis lekérdezés, akkor megszakítjuk:

```kotlin
override fun onPause() {
	super.onPause()

	loadTodosTask?.cancel(false)
}
```

Írjuk meg az `onDestroy()` függvényt, amiben bezárjuk az adapterhez csatolt, Todo elemeket tartalmazó Cursor objektumot, ha létezik ilyen:

```kotlin
override fun onDestroy() {
	super.onDestroy()

	adapter.cursor?.close()
}
```

Végül készítsük el a hiányzó `refreshList()` metódust is, amely elindítja az aszinkron adatbázis lekérdező folyamatot:

```kotlin
private fun refreshList() {
	loadTodosTask?.cancel(false)

	val loadTodosTask = LoadTodosTask(this, dbLoader)
	loadTodosTask.execute()

	this.loadTodosTask = loadTodosTask
}
```
Ha minden kódrészlet a megfelelő helyen van, akkor az alkalmazás indulásakor ugyanaz a felhasználói felület jelenik meg, mint a kiinduló projekt esetén. Azonban most még üres listát látunk, hiszen üres az adatbázis. Ahhoz, hogy újra működjön a Todo létrehozás funkció, módosítanunk kell az `onTodoCreated` callbacket:

```kotlin
override fun onTodoCreated(todo: Todo) {
	dbLoader.createTodo(todo)
	refreshList()
}
```

A `dbLoader.createTodo(todo)` hívást természetesen az adatbázisból betöltés mintájára javasolt háttérszálon futtatni. A példa alkalmazás bonyolultságának növelését megelőzve ezt most nem tesszük meg.

Indítsuk el az alkalmazást! Próbáljuk ki, hogy forgatás, illetve az alkalmazás háttérbe küldése és visszahozása után is megmaradnak a létrehozott Todo-k.

## Önálló feladat

Az eredeti funkcionalitás további részeinek megvalósítása önálló feladatként oldandó meg:

-  Husszú nyomásra a kiválasztott Todo elem törlésének lehetősége
-  Az összes Todo elem törlésének lehetősége (pl. Options menüből elérve)
