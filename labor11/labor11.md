# Labor 11 - Futásidejű engedélyek

## Bevezető

Android 6.0 (API level 23, Marshmallow) verziótól kezdve a felhasználó futásidőben adhatja meg, vagy utasíthatja el az alkalmazás által kért engedélyeket, és nem az alkalmazás telepítésekor vagy frissítésekor. Dönthet úgy, hogy bizonyos engedélyeket nem ad meg egy alkalmazásnak, így nagyobb fokú irányítás kerül a kezébe. Az alkalmazásengedélyeket később bármikor módosíthatja a rendszerszintű alkalmazás beállításoknál.

Az engedélyek két kategóriába vannak sorolva: *normal* és *dangerous*
A *normal* kategóriába tartozó engedélyek nem jelentenek közvetlen kockázatot a felhasználó személyes adataira, ezeket az engedélyeket a rendszer automatikusan megadja az alkalmazásnak, ha szüksége van rá.

A *dangerous* kategóriába tartozó engedélyek lehetőséget adhatnak az alkalmazásnak, hogy a felhasználó személyes adataihoz hozzáférjen. Ebben az esetben a felhasználónak kell megadni az engedélyt az alkalmazás számára. Ennek a közvetlen következménye az, hogy az alkalmazásokat fel kell készíteni arra az esetre, ha nincs megadva egy adott funkció működéséhez elengedhetetlen engedély.

Az alábbi oldalon található az összes engedély kategóriánként:

https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous

Az `AndroidManifest.xml` fájlban kategóriától függetlenül meg kell adni az alkalmazás számára szükséges összes engedélyt, de ennek hatása eltér a futtató rendszer verziójától és a *targetSdk*-tól függően:

* Ha az eszköz Android 5.1 (API level 22) vagy alacsonyabb verziót futtat, **VAGY** az alkalmazás target SDK szintje 22 vagy kisebb, akkor a rendszer telepítéskor kéri el az összes szükséges engedélyt. Ha a felhasználó nem fogadja el egyben az összes kérést, akkor a telepítési folyamat leáll.

* Ha az eszköz Android 6.0 (API level 23) vagy nagyobb verziót futtat **ÉS** az alkalmazás target SDK szintje 23 vagy nagyobb, akkor az alkalmazás a futása során fogja elkérni a *dangerous* kategóriába tartozó engedélyeket, a *normal* engedélyeket pedig a rendszer automatikusan megadja. Ebben az esetben a  felhasználó bármikor bármelyik engedélyt megadhatja, vagy letilthataja. Megtagadott engedélyekkel az alkalmazás limitált funkcionalitással futhat tovább, erre a helyzetre is fel kell készülni.

## Jogosultság ellenőrzése

Amennyiben az alkalmazás egy funkciójának *dangerous* kategóriába eső engedélyekre van szüksége, akkor minden esetben ellenőrízni kell még a funkció indítása előtt, hogy rendelkezik-e az engedélyekkel, hiszen az engedélyeket a felhasználó bármikor módosíthatja.

Az ellenőrzés a `ContextCompat.checkSelfPermission()` függvény meghívásával végezhető, ami a `PackageManager.PERMISSION_GRANTED` értékkel tér vissza, ha az alkalmazás rendelkezik a vizsgált engedéllyel, és `PackageManager.PERMISSION_DENIED` értékkel egyébként.

Az alábbi kódrészlet a felhasználó naptárához való hozzáférési engedélyt ellenőrzi egy `Activity`-ben:

```kotlin
val permissionResult = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_CALENDAR)
when (permissionResult) {
	PackageManager.PERMISSION_GRANTED -> writeToCalendar(event)
	else -> requestPermissions()
}
```

## Jogosultság kérés magyarázata

Egyes esetekben szükség lehet arra, hogy tájékoztassuk a felhasználót arról, hogy miért kér az alkalmazás bizonyos *dangerous* engedélyeket. Ez növelheti a felhasználó bizalmát az alkalmazással szemben. Az `ActivityCompat.shouldShowRequestPermissionRationale()` függvény visszatérési értéke alapján eldönthető, hogy a kérdéses engedély elkérése a rendszer szerint szorul-e részletes magyarázatra:

```kotlin
 // Permission is not granted
// Should we show an explanation?
if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity, Manifest.permission.READ_CONTACTS)) {
	// Show an explanation to the user *asynchronously* -- don't block
	// this thread waiting for the user's response! After the user
	// sees the explanation, try again to request the permission.
} else {
	// No explanation needed, we can request the permission.
}
```

## Jogosultság elkérése

Engedélyek elkérésére az `ActivityCompat.requestPermissions()` függvény meghívásával van lehetőség, aminek eredményeképp egy nem testreszabható dialógust jelenít meg a rendszer.

```kotlin
ActivityCompat.requestPermissions(thisActivity, arrayOf(Manifest.permission.READ_CONTACTS), MY_PERMISSIONS_REQUEST_READ_CONTACTS)
```

A `MY_PERMISSIONS_REQUEST_READ_CONTACTS` ebben a kódrészletben egy általunk definiált konstans. Az engedélykérés végén a rendszer ezt az értéket adja vissza `requesCode`-ként az `onRequestPermissionsResult()` callbackben. 

## Kezdő lépések

A labor során egy egyszerű telefonkönyv alkalmazást kell elkészíteni. Az alkalmazásnak meg kell jeleníteni a telefonon tárolt névjegyeket, illetve egy névjegyre kattintással tudnia kell hívást kezdeményezni az ahhoz tartozó elsődleges telefonszámra.

Hozzunk létre egy új projektet Android Studio-ban *Contacts* néven. A *Company Domain* mező tartalmát töröljük ki és hagyjuk is üresen.

A *package name* legyen `hu.bme.aut.android.contacts`. A támogatott eszköz formátum legyen *Phone and Tablet*, a minimum SDK szint legyen *API 19: Android 4.4 (KitKat)*.

A projekthez adjuk hozzá egy *Empty Activity*-t, melynek neve legyen `ContactsActivity`.

Vegyük fel a `RecyclerView` libraryt függőségként a `build.gradle (Module: app)` fájlban:

```groovy
dependencies {
	...
	implementation 'com.android.support:recyclerview-v7:28.0.0-rc02'
	...
}
```

Szintén a `build.gradle (Module: app)` fájlban ellenőrizzük, hogy a *targetSDK* értéke legalább 23.

Kattintsunk a *Sync Now* gombra.

## Felhasználói felület

Készítsük el az alkalmazás felhasználói felületét a `res/layout/activity_contacts.xml` fájlban. A felület egyetlen `RecyclerView`-ból fog állni, mely az eszközön tárolt névjegyeket fogja megjeleníteni.

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:id="@+id/activity_contacts"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:paddingBottom="@dimen/activity_vertical_margin"
	android:paddingLeft="@dimen/activity_horizontal_margin"
	android:paddingRight="@dimen/activity_horizontal_margin"
	android:paddingTop="@dimen/activity_vertical_margin"
	tools:context=".ContactsActivity">

	<android.support.v7.widget.RecyclerView
		android:id="@+id/rvContacts"
		android:layout_width="match_parent"
		android:layout_height="match_parent" />
</RelativeLayout>
```

Hozzuk létre a hiányzó erőforrásokat *16dp* értékkel.

## Model

Készítsük el a `hu.bme.aut.android.contacts.model` package-et és benne a  `Contact` osztályt , ami egy eszközön található névjegyet fog reprezentálni. Az egyszerűség kedvéért most csak a név és telefonszám adatokat tároljuk el benne.

```kotlin
class Contact(
	val name: String,
	val number: String
)
```

## Listaelem

Hozzuk létre az `item_contact.xml` layout erőforrást:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	android:id="@+id/container"
	android:layout_width="match_parent"
	android:layout_height="wrap_content"
	android:orientation="horizontal"
	android:background="#dddddd">

	<LinearLayout
		android:layout_width="match_parent"
		android:layout_height="wrap_content">

		<ImageView
			android:id="@+id/ivContactImage"
			android:layout_width="55dp"
			android:layout_height="55dp"
			android:layout_marginLeft="10dp"
			android:layout_marginStart="10dp"
			android:src="@drawable/ic_contact_phone_black_48dp"/>

		<LinearLayout
			android:layout_width="match_parent"
			android:layout_height="match_parent"
			android:orientation="vertical"
			android:gravity="center_vertical">

			<TextView
				android:id="@+id/tvContactName"
				android:layout_width="match_parent"
				android:layout_height="wrap_content"
				android:layout_marginLeft="10dp"
				android:layout_marginStart="10dp"
				android:textSize="16sp"
				android:textColor="@android:color/primary_text_light"
				android:text="@string/"/>

			<TextView
				android:id="@+id/tvPhoneNumber"
				android:layout_width="match_parent"
				android:layout_height="wrap_content"
				android:layout_marginLeft="10dp"
				android:layout_marginStart="10dp"
				android:textSize="14sp"
				android:textColor="@android:color/primary_text_light"/>
		</LinearLayout>
	</LinearLayout>
</RelativeLayout>
```

Hozzuk létre a `res/values/strings.xml` fájlban a két hiányzó szöveges erőforrást:

```xml
<resources>
	<string name="app_name">Contacts</string>
	<string name="contact_name_placeholder">Name is not set</string>
	<string name="contact_number_placeholder">Phone number is not set</string>
</resources>
```

Töltsük le az [ic_contact_phone_black_48dp.png](assets/ic_contact_phone_black_48dp.png) képet és másoljuk be a `res/drawable` mappába.

## Adapter

Készítsük el a `hu.bme.aut.android.contacts.adapter` package-et és benne a listát feltöltő adaptert `ContactsAdapter` néven.

```kotlin
class ContactsAdapter : RecyclerView.Adapter<ContactsAdapter.ContactViewHolder>() {

    private val contactList = mutableListOf<Contact>()

    var itemClickListener: ContactItemClickListener? = null

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ContactViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.item_contact, null)
        return ContactViewHolder(view)
    }

    override fun onBindViewHolder(holder: ContactViewHolder, position: Int) {
        val contact = contactList[position]
        holder.tvContactName.text = contact.name
        holder.tvPhoneNumber.text = contact.number
        holder.contact = contact
    }

    override fun getItemCount(): Int {
        return contactList.size
    }

    fun setContacts(contacts: List<Contact>) {
        contactList.clear()
        contactList += contacts
        notifyDataSetChanged()
    }

    inner class ContactViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val ivContactImage: ImageView = itemView.ivContactImage
        val tvContactName: TextView = itemView.tvContactName
        val tvPhoneNumber: TextView = itemView.tvPhoneNumber

        var contact: Contact? = null

        init {
            itemView.setOnClickListener {
                contact?.let { itemClickListener?.onItemClick(it) }
            }
        }
    }

    interface ContactItemClickListener {
        fun onItemClick(contact: Contact)
    }
}
```

Adjuk hozzá a `ContactsActivity`-hez az alábbi, névjegyek lekérdezését megvalósító függvényeket:

```kotlin
private fun ContentResolver.performQuery(
		@RequiresPermission.Read uri: Uri,
		projection: Array<String>? = null,
		selection: String? = null,
		selectionArgs: Array<String>? = null,
		sortOrder: String? = null
): Cursor? {
	return query(uri, projection, selection, selectionArgs, sortOrder)
}

private fun getAllContacts(): List<Contact> {
	contentResolver.performQuery(
			uri = ContactsContract.Contacts.CONTENT_URI,
			sortOrder = "${ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME} ASC"
	).use { contactResultCursor ->
		return if (contactResultCursor == null) {
			emptyList()
		} else {
			getContacts(contactResultCursor)
		}
	}
}

private fun getContacts(contactCursor: Cursor): List<Contact> {
	val contactList = mutableListOf<Contact>()

	while (contactCursor.moveToNext()) {
		val hasPhoneNumber = contactCursor.getString(contactCursor.getColumnIndex(ContactsContract.Contacts.HAS_PHONE_NUMBER)).toInt()
		if (hasPhoneNumber != 0) {
			val id = contactCursor.getString(ContactsContract.Contacts._ID)
			val name = contactCursor.getString(ContactsContract.Contacts.DISPLAY_NAME)

			val contactPhoneNumber = getContactPhoneNumber(id)

			contactList += Contact(name, contactPhoneNumber)
		}
	}

	return contactList
}

private fun getContactPhoneNumber(id: String): String {
	contentResolver.performQuery(
			uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
			selection = "${ContactsContract.CommonDataKinds.Phone.CONTACT_ID} = ?",
			selectionArgs = arrayOf(id)
	).use { phoneResultCursor ->
		return if (phoneResultCursor == null || !phoneResultCursor.moveToNext()) {
			""
		} else {
			phoneResultCursor.getString(ContactsContract.CommonDataKinds.Phone.NUMBER)
		}
	}
}
```

A `Cursor`-okon hívott `getString(columnName: String)` függvény egy *extension function*, ami az *Android KTX* libraryben található és olvashatóbbá teszi a lekérdezés kódját. Vegyük fel az *Android KTX* libraryt függőségként a `build.gradle (Module: app)` fájlban:

```groovy
dependencies{
	...
	implementation 'androidx.core:core-ktx:0.3'
	...
}
```

A `ContactsActivity` `onCreate()` függvényében írjuk meg a `RecyclerView` inicializálását:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
	super.onCreate(savedInstanceState)
	setContentView(R.layout.activity_contacts)

	val contactsAdapter = ContactsAdapter()
	rvContacts.layoutManager = LinearLayoutManager(this)
	rvContacts.adapter = contactsAdapter
	contactsAdapter.setContacts(getAllContacts())
}
```

Jelezzük a rendszer felé, hogy az alkalmazásnak szüksége van engedéylre a névjegyek olvasásához. Ehhez vegyük fel az `AndroidManifest.xml` fájlban a `<manifest>...</manifest>` tagen belül az alábbi sort:

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

## Teszt

Egyelőre nem valósítottunk meg futásidejű jogosulságkezelést a kódban, ezért az alkalmazás működésének kipróbálásához Android 6.0 előtti verziót futtató eszközre, vagy emulátorra van szükség. Újabb verzió esetén hibát kapunk az alkalmazás indulása során.

Próbáljuk ki az alkalmazást 6.0/API 23 előtti verzióval rendelkező eszközön, vagy emulátoron!
Amennyiben az eszközön, vagy emulátoron nincsenek névjegyek, adjunk hozzá legalább egyet telefonszámmal ellátva a beépített névjegykezelő alkalmazásban.

<img src="./assets/app.png" width="400" align="middle">

Android 6.0 vagy magasabb verzión futtatva az alkalmazást hibát kapunk, hiszen a névjegyek beolvasásához szükséges engedély a *dangerous* kategóriába tartozik, ezt külön kell kezelni kód szinten (6.0 felett **ÉS** targetSdk 23+ esetén).

A kapott hiba az alábbihoz hasonló:

```text
java.lang.SecurityException: Permission Denial: opening provider com.android.providers.contacts.ContactsProvider2 from ProcessRecord{7a6a2b4 14701:hu.bme.aut.android.contacts/u0a135} (pid=14701, uid=10135) requires android.permission.READ_CALENDAR or android.permission.WRITE_CALENDAR
```

## Futásidejű jogosultságkezelés megvalósítása

Módosítsuk az alkalmazást úgy, hogy futási időben kérje el a felhasználótól a manifestben deklarált *dangerous* engedélyt! Ebben a bevezetőben ismertetett függcények lesznek segítségünkre.

Emeljük ki a `ContactsActivity` `onCreate()` metódusában található alábbi sorokat egy függvénybe, melynek a neve legyen `loadContacts()`!

```kotlin
val contactsAdapter = ContactsAdapter()
rvContacts.layoutManager = LinearLayoutManager(this)
rvContacts.adapter = contactsAdapter
contactsAdapter.setContacts(getAllContacts())
```

Ezt Android Studio-ban legegyszerűbben a kiemelni kívánt kód kijelölésével, majd a CTRL+ALT+M billentyűkombinációval tudjuk megtenni.

```kotlin
private fun loadContacts() {
	val contactsAdapter = ContactsAdapter()
	rvContacts.layoutManager = LinearLayoutManager(this)
	rvContacts.adapter = contactsAdapter
	contactsAdapter.setContacts(getAllContacts())
	contactsAdapter.itemClickListener = this
}
```

A kiemelés után az `onCreate()` függvény:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
	super.onCreate(savedInstanceState)
	setContentView(R.layout.activity_contacts)

	loadContacts()
}
```

Ahelyett hogy az `onCreate()` függvényben azonnal meghívnánk a `loadContacts()` függvényt, kérjünk engedélyt a felhasználótól a névjegyek olvasására!

Adjuk hozzá az alábbi függvényeket a `ContactsActivity`-hez!

```kotlin
private fun showRationaleDialog(
		@StringRes title: Int = R.string.rationalerationale_dialog_title,
		@StringRes explanation: Int,
		onPositiveButton: () -> Unit,
		onNegativeButton: () -> Unit = this::finish
) {
	val alertDialog = AlertDialog.Builder(this)
			.setTitle(title)
			.setMessage(explanation)
			.setCancelable(false)
			.setPositiveButton(R.string.proceed) { dialog, id ->
				dialog.cancel()
				onPositiveButton()
			}
			.setNegativeButton(R.string.exit) { dialog, id -> onNegativeButton() }
			.create()
	alertDialog.show()
}

private fun handleReadContactsPermission() {
	if (ContextCompat.checkSelfPermission(this,
					Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED) {
		// Should we show an explanation?
		if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_CONTACTS)) {

			// Show an explanation to the user *asynchronously* -- don't block
			// this thread waiting for the user's response! After the user
			// sees the explanation, try again to request the permission.

			showRationaleDialog(
					explanation = R.string.contacts_permission_explanation,
					onPositiveButton = this::requestContactsPermission
			)
			
		} else {
			// No explanation needed, we can request the permission.
			requestContactsPermission()
		}
	} else {
		loadContacts()
	}
}

private fun requestContactsPermission() {
	ActivityCompat.requestPermissions(
			this,
			arrayOf(READ_CONTACTS),
			PERMISSIONS_REQUEST_READ_CONTACTS
	)
}
```

A `PERMISSIONS_REQUEST_READ_CONTACTS` egy általunk definiálandó `requestCode`. Amikor engedélyt kérünk, meg kell adni egy `requestCode`-ot is. Amikor az operációs rendszer visszatér a kérés eredményével, a kérést indító `Activity` `onRequestPermissionsResult()` függvényében visszakapjuk ezt az értéket. Ez alapján tudjuk kezelni, hogy éppen melyik engedélykérésre érkezett válasz.

Bármilyen érték adható neki, csak ezen az aktuális alkalmazáson belül számít. Jelen esetben legyen 100.

A `ContactsActivity`-ben hozzuk létre a `PERMISSIONS_REQUEST_READ_CONTACTS` konstanst a `ContactsActivity` `companion object`-jében: 

```kotlin
companion object {
	private const val PERMISSIONS_REQUEST_READ_CONTACTS = 100
}
```

A `res/values/strings.xml`-be vegyük fel a hiányzó szöveges erőforrásokat:

```xml
<string name="rationale_dialog_title">Attention!</string>
<string name="explanation">The application needs to access your contacts to display them.</string>
<string name="exit">Exit</string>
<string name="proceed">Proceed</string>
```

> Magyarázat

> A `handleReadContactsPermission()` függvényben a `checkSelfPermission()` függvény segítségével megvizsgáljuk, hogy az alkalmazás rendelkezik-e a `READ_CONTACTS` engedéllyel. Ha igen, akkor meghívjuk a `loadContacts()` függvényt, ami betölti a névjegyeket. Ellenkező esetben megkérdezzük arendszert, hogy a felhasználót kell-e tájékoztatni az engedélykérés létjogosultságáról (`shouldShowRequestPermissionRationale()`). Ez a függvény akkor tér vissza true értékkel, ha a felhasználó korábban megtagadta az engedélyt az alkalmazástól. (Például azért, mert nem gondolta, hogy az adott funkcióhoz feltétlenül szükséges az engedély.) Ilyenkor érdemes egy magyarázatot adni, melyben leírjuk, hogy miért van feltétlenül szükség az engedélyre. Legyünk tömörek, a hosszú magyarázatokat a felhasználó nem fogja elolvasni, inkább letörli az alkalmazást. A magyarázat jelen esetben egy dialógus, mely rövid tájékoztatást ad az engedély szükségességéről.
Amennyiben nincs szükség magyarázatra, vagy a magyarázat dialógusablakában a *Proceed* gombra nyomott a felhasználó, akkor elkérjük az engedélyt (`requestPermissions()`).

Ceréljük le a `ContactsActivity` `onCreate()` függvényében a `loadContacts()` hívást az előzőekben létrehozott `handleReadContactsPermission()` hívásra:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
	super.onCreate(savedInstanceState)
	setContentView(R.layout.activity_contacts)

	loadContacts()
}
```

Kezeljük le az engedélykérés válaszát is úgy, hogy felülírjuk a `ContactsActivity` `onRequestPermissionResult()` függvényét:

```kotlin
override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<String>, grantResults: IntArray) {
	when (requestCode) {
		PERMISSIONS_REQUEST_READ_CONTACTS -> {
			// If request is cancelled, the result arrays are empty.
			if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
				// permission was granted, yay! Do the
				// contacts-related task you need to do.
				loadContacts()
			} else {
				// permission denied! Disable the
				// functionality that depends on this permission.
			}
			return
		}
	}
}
```

Amennyiben a felhasználó megadta az engedélyt, az alkalmazás betölti a névjegyeket.

Próbáljuk ki az alkalmazást 6.0+/API level 23+ eszközön, vagy emulátoron!
Figyeljük meg a magyarázó dialógust miután megtagadjuk az engedélyt, majd újraindítjuk az alkalmazást!

## Telefonszám hívása

Ahhoz, hogy az alkalmazásunk hívásokat indíthasson, fel kell venni a következő engedélyt az `AndroidManifest.xml` fájlba a `READ_CONTACTS`-hez hasonlóan:

```xml
<uses-permission android:name="android.permission.CALL_PHONE" />
```

Ez az engedély is a *dangerous* kategóriába tartozik, ezért a telefonhívás indítását is az előzőekben leírtaknak megfelelően kell kezelnünk.
Bővítsük a funkcionalitást úgy, hogy az alkalmazás egy adott névjegy elemre kattintás hatására hívást indítson a névjegyben szereplő telefonszámra!

Módosítsuk úgy a `ContactsActivity`-t, hogy implementálja a `ContactsAdapter.ContactItemClickListener` interfészt:

```kotlin
class ContactsActivity : AppCompatActivity(), ContactsAdapter.ContactItemClickListener {
	...
	override fun onItemClick(contact: Contact) {
		handleCallPermission(contact.number)
	}
	...
}
```

Adjuk hozzá az alábbi propertyt és függvényeket a `ContactsActivity` osztályhoz:

```kotlin
private var lastPhoneNumber: String? = null

private fun handleCallPermission(phoneNumber: String) {
	lastPhoneNumber = phoneNumber
	if (ActivityCompat.checkSelfPermission(this,
					Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED) {
		// Should we show an explanation?
		if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.CALL_PHONE)) {
			// Show an explanation to the user *asynchronously* -- don't block
			// this thread waiting for the user's response! After the user
			// sees the explanation, try again to request the permission.

			showRationaleDialog(
					explanation = R.string.call_permission_explanation,
					onPositiveButton = this::requestCallPermission
			)

		} else {
			// No explanation needed, we can request the permission.
			requestCallPermission()
		}
	} else {
		callPhoneNumber(phoneNumber)
	}
}

private fun requestCallPermission() {
	ActivityCompat.requestPermissions(
			this,
			arrayOf(CALL_PHONE),
			PERMISSIONS_REQUEST_PHONE_CALL
	)
}

@SuppressLint("MissingPermission")
private fun callPhoneNumber(phoneNumber: String) {
	val callIntent = Intent(Intent.ACTION_CALL)
	callIntent.data = Uri.parse("tel:$phoneNumber")
	startActivity(callIntent)
}

private fun callLastPhoneNumber() {
	lastPhoneNumber?.let { phoneNumber ->
		callPhoneNumber(phoneNumber)
	}
}
```

A `strings.xml`-ben vegyük fel a hiányzó szöveges erőforrást:

```xml
<string name="call_permission_explanation">The application needs permission to make phone calls. It will not initiate calls without explicit user intention.</string>
```

A `callPhoneNumber()` függvény fogja indítani a hívást, a `handleCallPermission()` függvény pedig az engedélykérést kezeli a névjegyeknél látottakkal megegyező módon.
Hozzuk létre a `PERMISSIONS_REQUEST_PHONE_CALL` konstanst a `ContactsActivity` `companion objec`-jében a korábban létrehozott `PERMISSIONS_REQUEST_READ_CONTACTS`-tól eltérő értékkel.

```kotlin
companion object {
	private const val PERMISSIONS_REQUEST_READ_CONTACTS = 100
	private const val PERMISSIONS_REQUEST_PHONE_CALL = 101
}
```

Az engedélykérés eredményét ebben az esetben is a `ContactsActivity` fogja kezelni. Módosítsuk az `onRequestPermissionsResult()` függvényt:

```kotlin
override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<String>, grantResults: IntArray) {
	when (requestCode) {
		PERMISSIONS_REQUEST_READ_CONTACTS -> {
			// If request is cancelled, the result arrays are empty.
			if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
				// Permission was granted! Do the contacts-related task you need to do.
				loadContacts()
			} else {
				// Permission denied! Disable the functionality that depends on this permission.
				// In this example, this block is intentionally empty and serves only as a demonstration for
				// what can be done here.
			}
			return
		}
		PERMISSIONS_REQUEST_PHONE_CALL -> {
			if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
				callLastPhoneNumber()
			}
		}
	}
}
```

Teszteljük a hívás funkciót 6.0+/API level 23+ emulátoron!

## Önálló feladatok

### Valósítsa meg az SMS küldés funkciót!

Például hosszú érintés eseménykezelő segítségével. 

A szükséges engedély:

```xml
<uses-permission android:name="android.permission.SEND_SMS"/>
```
