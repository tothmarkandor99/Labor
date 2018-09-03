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
		android:id="@+id/contactsRV"
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

    private var itemClickListener: ContactItemClickListener? = null

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

A névjegyek megjelenítéséhez az utolsó lépés a `ContactsAdapter` pélányosítása, és beállítása a `RecyclerView`-nak. Az eszközön tárolt névjegyek megszerzéséhez adjuk hozzá a `ContactsActivity`-hez az alábbi függvényeket:

```kotlin
private fun getAllContacts(): List<Contact> {
        contentResolver.query(
                ContactsContract.Contacts.CONTENT_URI,
                null,
                null,
                null,
                "${ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME} ASC"
        ).use { contactResultCursor ->
            contactResultCursor?.let { contactCursor ->
                return getContacts(contactCursor)
            }
        }

        return emptyList()
    }

    private fun getContacts(contactCursor: Cursor): List<Contact> {
        val contactList = mutableListOf<Contact>()

        while (contactCursor.moveToNext()) {
            val hasPhoneNumber = contactCursor.getString(contactCursor.getColumnIndex(ContactsContract.Contacts.HAS_PHONE_NUMBER)).toInt()
            if (hasPhoneNumber > 0) {
                val id = contactCursor.getString(contactCursor.getColumnIndex(ContactsContract.Contacts._ID))
                val name = contactCursor.getString(contactCursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME))

                val contactPhoneNumber = getContactPhoneNumber(id)
                
                contactList += Contact(name, contactPhoneNumber)
            }
        }

        return contactList
    }

    private fun getContactPhoneNumber(id: String): String {
        var phoneNumber = ""
        
        contentResolver.query(
                ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                null,
                "${ContactsContract.CommonDataKinds.Phone.CONTACT_ID} = ?",
                arrayOf(id),
                null
        ).use { phoneResultCursor ->
            phoneResultCursor?.let { phoneCursor ->
                if (phoneCursor.moveToNext()) {
                    phoneNumber = phoneResultCursor.getString(phoneResultCursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER))
                }
            }
        }
        
        return phoneNumber
    }
```

Ez után a kapott névjegylistával példányosítsuk az adaptert, és állítsuk be a RecyclerView komponenshez.
**ContactsActivity** **onCreate()** metódusába:

```java
ContactsAdapter contactsAdapter = new ContactsAdapter(getAllContacts(), this);
contactsRV.setLayoutManager(new LinearLayoutManager(this));
contactsRV.setAdapter(contactsAdapter);
```

Névjegyek olvasásához szükséges engedély a manifest-be:

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

## Teszt

Egyelőre semmilyen jogosulságkezelést nem valósítottunk meg a kódban, ezért az alkalmazás pillanatnyi állapotának kipróbálásához Android 6.0 előtti verzióra van szükség, különben hibát kapunk az indulás során.

Próbáljuk ki az alkalmazást 6.0/API 23 előtti verzióval rendelkező eszközön!
Amennyiben az eszközön nincsenek névjegyek, adjunk hozzá legalább egyet telefonszámmal ellátva.

<img src="./assets/app.png" width="400" align="middle">

Android 6.0 vagy magasabb verzión futtatva az alkalmazást hibát kapunk, hiszen a névjegyek beolvasásához szükséges engedély a veszélyes kategóriába tartozik, ezért ezt külön kell kezelni a kódban. (6.0 felett ÉS target SDK 23+ esetén)

A hiba:

```java
java.lang.SecurityException: Permission Denial: 
opening provider com.android.providers.contacts.ContactsProvider2
from ProcessRecord{b077ff821678:
hu.bme.aut.amorg.examples.permissionslabor/u0a264} 
(pid=21678, uid=10264) requires android.permission.READ_CONTACTS or
android.permission.WRITE_CONTACTS
```

## Jogosultságkezelés

Módosítsuk az alkalmazást úgy, hogy futási időben kérje el a felhasználótól a manifestben deklarált veszélyes engedélyt! Ehhez a bevezetőben ismertetett metódusok lesznek segítségünkre.

Emeljük ki a ContactsActivity onCreate() metódusában található alábbi 3 sor kódot egy metódusba, melynek a neve legyen loadContacts()!
Ezt legegyszerűbben a kiemelni kívánt kód kijelölésével, majd CTRL+ALT+M billentyűkombinációval tudjuk megtenni Android Studioban.

```java
private void loadContacts() {
    ContactsAdapter contactsAdapter = new ContactsAdapter(getAllContacts(), this);
    contactsRV.setLayoutManager(new LinearLayoutManager(this));
    contactsRV.setAdapter(contactsAdapter);
}
```

onCreate():

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_contacts);
    contactsRV = findViewById(R.id.contactsRV);

    loadContacts();
}
```

Ahelyett hogy az **onCreate()**-ben azonnal meghívnánk a **loadContacts()** függvényt, kérjünk a felhasználótól engedélyt a névjegyek olvasására!

Adjuk hozzá az alábbi metódust a ContactsActivityhez!

```java
private void handleReadContactsPermission() {
    if (ContextCompat.checkSelfPermission(this,
            Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED) {
        // Should we show an explanation?
        if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.READ_CONTACTS)) {

            // Show an explanation to the user *asynchronously* -- don't block
            // this thread waiting for the user's response! After the user
            // sees the explanation, try again to request the permission.
            AlertDialog.Builder alertDialogBuilder = new AlertDialog.Builder(this);
            alertDialogBuilder.setTitle(R.string.dialogTitle);
            alertDialogBuilder
                    .setMessage(R.string.explanation)
                    .setCancelable(false)
                    .setNegativeButton(R.string.exit, new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int id) {
                            ContactsActivity.this.finish();
                        }
                    })
                    .setPositiveButton(R.string.forward, new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int id) {
                            dialog.cancel();
                            ActivityCompat.requestPermissions(ContactsActivity.this,
                                    new String[]{Manifest.permission.READ_CONTACTS},
                                    MY_PERMISSIONS_REQUEST_READ_CONTACTS);
                        }
                    });
            AlertDialog alertDialog = alertDialogBuilder.create();
            alertDialog.show();
        } else {
            // No explanation needed, we can request the permission.
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.READ_CONTACTS},
                    MY_PERMISSIONS_REQUEST_READ_CONTACTS);
        }
    } else {
        loadContacts();
    }
}
```

A `MY_PERMISSIONS_REQUEST_READ_CONTACTS` egy általunk definiálandó requestCode. Amikor engedélyt kérünk, meg kell adni mellé egy requestCode-ot is, és amikor az operációs rendszer visszatér a **onRequestPermissionsResult()** metódusban, akkor ez alapján tudjuk kezelni, hogy éppen melyik engedélykérésre érkezett válasz.

Bármilyen érték adható neki, jelen esetben legyen 100.

```java
private static final int MY_PERMISSIONS_REQUEST_READ_CONTACTS = 100;
```

**strings.xml**-be:

```xml
<string name="dialogTitle">Figyelem!</string>
<string name="explanation">Az alkalmazásnak szüksége van az engedélyre a névjegyek beolvasásához!</string>
<string name="exit">Kilépés</string>
<string name="forward">Tovább</string>
```

A **handleReadContactsPermission()** metódus megvizsgálja a **checkSelfPermission()** segítségével, hogy az alkalmazás rendelkezik-e már a `READ_CONTACTS` engedéllyel. Ha igen, akkor meghívja a **loadContacts()** metódust, és a névjegyek betöltődnek. Ellenkező esetben nézzük meg, hogy a felhasználót kell-e tájékoztatni az engedélykérés létjogosultságáról (*shouldShowRequestPermissionRationale()*). Ez a metódus akkor tér vissza true értékkel, ha korábban a felhasználó megtagadta az engedélyt az alkalmazástól. (Például mert nem gondolta, hogy az adott funkcióhoz feltétlenül szükséges az engedély.) Ilyenkor érdemes egy magyarázatot adni, melyben leírjuk, hogy miért van feltétlen szükség az engedélyre. (Legyünk tömörek, a hosszú magyarázatokat nem fogja a felhasználó elolvasni, inkább letörli az alkalmazást...) A magyarázat jelen esetben egy dialógus, mely rövid leírást ad az engedély szükségességéről.
Amennyiben nincs szükség magyarázatra, vagy a magyarázat dialógusablakában a Tovább gombra nyomott a felhasználó, akkor kérjük el az engedélyt (*requestPermissions()*).

Cseréljük ki az activity **onCreate()**-ben található **loadContacts()**
metódust az újonnan létrehozottra (**handleReadContactsPermission();**)!

Kezeljük le az engedélykérés válaszát (**onRequestPermissionsResult()**) is az alábbi kóddal:

```java
@Override
public void onRequestPermissionsResult(int requestCode,
                                       String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                    && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // permission was granted, yay! Do the
                // contacts-related task you need to do.
                loadContacts();
            } else {
                // permission denied! Disable the
                // functionality that depends on this permission.
            }
            return;
        }
    }
}
```

Amennyiben az engedélyt az alkalmazás megkapta, a névjegyek a loadContacts() segítségével betöltésre kerülnek.

Próbáljuk ki az alkalmazást 6.0+/API level 23+ eszközön!
Figyeljük meg a magyarázódialógust abban az esetben, ha megtagadjuk az engedélyt, majd újraindítjuk az alkalmazást!

## Telefonszám hívása

Ahhoz, hogy alkalmazásunk hívásokat indíthasson, fel kell venni a következő engedélyt a manifest fájlba:

```xml
<uses-permission android:name="android.permission.CALL_PHONE" />
```

Magától értetődő, hogy ez az engedély is a veszélyes kategóriába tartozik, ezért ezt is megfelelően kell kezelnünk.
Bővítsük a funkcionalitást olyan módon, hogy egy adott névjegy elemre kattintva hívást indítson az eszköz a névjegyen szereplő telefonszámra!

Másoljuk az alábbi két metódust a **ContactsAdapterbe**!

```java
private String lastPhoneNumber;

private void handleCallPhonePermission(View view, String phoneNumber) {
    this.lastPhoneNumber=phoneNumber;
    if (ActivityCompat.checkSelfPermission(view.getContext(), Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED) {
        // Should we show an explanation?
        if (ActivityCompat.shouldShowRequestPermissionRationale((Activity) mContext,
                Manifest.permission.CALL_PHONE)) {
            // Show an explanation to the user *asynchronously* -- don't block
            // this thread waiting for the user's response! After the user
            // sees the explanation, try again to request the permission.
            AlertDialog.Builder alertDialogBuilder = new AlertDialog.Builder(mContext);
            alertDialogBuilder.setTitle(R.string.dialogTitle);
            alertDialogBuilder
                    .setMessage(R.string.explanation2)
                    .setCancelable(false)
                    .setNegativeButton(R.string.exit, new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int id) {
                            ((ContactsActivity) mContext).finish();
                        }
                    })
                    .setPositiveButton(R.string.forward, new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int id) {
                            dialog.cancel();
                            ActivityCompat.requestPermissions((Activity) mContext,
                                    new String[]{Manifest.permission.CALL_PHONE},
                                    MY_PERMISSIONS_REQUEST_PHONE_CALL);
                        }
                    });
            AlertDialog alertDialog = alertDialogBuilder.create();
            alertDialog.show();
        } else {
            // No explanation needed, we can request the permission.
            ActivityCompat.requestPermissions((Activity) mContext,
                    new String[]{Manifest.permission.CALL_PHONE},
                    MY_PERMISSIONS_REQUEST_PHONE_CALL);
        }
    } else {
        callPhoneNumber(phoneNumber);
    }
}

private void callPhoneNumber(String phoneNumber) {
    Intent callIntent = new Intent(Intent.ACTION_CALL);
    callIntent.setData(Uri.parse("tel:" + phoneNumber));
    mContext.startActivity(callIntent);
}

public void callLastPhoneNumber() {
    callPhoneNumber(lastPhoneNumber);
}
```

strings.xml-be:

```xml
<string name="explanation2">A hívás indításához engedélyre van szükség!</string>
```

A callPhoneNumber() fogja a hívást indítani, a handleCallPhonePermission() pedig az engedélykérést kezeli a korábbival megegyező módon.
Itt is szükség van egy requestCode-ra, hozzuk létre public láthatósággal a korábban létrehozott requestCode-tól eltérő értékkel.

```java
public static final int MY_PERMISSIONS_REQUEST_PHONE_CALL = 101;
```

Az engedélykérés válaszát ebben az esetben is a **ContactsActivity** fogja kezelni, ezért helyezzük el az alábbi ágat az onRequestPermissionsResult() metódusba!

```java
case ContactsAdapter.MY_PERMISSIONS_REQUEST_PHONE_CALL: {
    if (grantResults.length > 0
            && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        ((ContactsAdapter)contactsRV.getAdapter()).callLastPhoneNumber();
    }
    return;
}
```

strings.xml-be:

```xml
<string name="phoneCallPermissionResultSuccess">Engedély elfogadva, kérem érintse meg újra a névjegyet a híváshoz!</string>
```

A hívás kezeléséhez szükséges kód hozzáadásra került, nincs más hátra mint használni. Ehhez adjunk eseménykezelőt a névjegyekhez, mellyel elindítjuk az imént létrehozott hívás engedély kezelést!

ContactsAdapter onBindViewHolder() végére:

```java
holder.container.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        handleCallPhonePermission(view, holder.tvPhoneNumber.getText().toString());
    }
});
```

Teszteljük a hívás funkcionalitást 6.0+/API level 23+ eszközön!

## Önálló feladatok

### Feladat:  Valósítsa meg az SMS küldés funkcionalitást!

Például hosszú érintés eseménykezelő segítségével. 
A szükséges engedély:

```xml
<uses-permission android:name="android.permission.SEND_SMS"/>
```
