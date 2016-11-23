# Labor 12 - Launcher++

## Bevezetõ

A labor célja az 5. laboron elkészített home screen alkalmazás kibõvítése.

A kiinduló projekt letölthetõ innen: [Kiinduló projekt](./assets/LauncherLabor_uptodate.zip) 

Az ötös eléréséhez a minimum követelményeken túl egy további, szabadon válaszott funkció beépítése szükséges. Az alkalmazások elkészítéséhez érdemes az óra anyagát és a példakódokat alapul venni az android.telephony csomag használata során.
[Telefónia Példa](./assets/TelefoniaPelda.zip) 

Két nézettel kell kibõvíteni az alkalmazást (a ViewPager-t). A két új nézettel szemben támasztott követelményeket az alábbi fejezetek tartalmazzák.

## Hívásnapló

A feladat egy hívásnapló nézet megvalósítása.

A nézet legyen képes megjeleníteni a részletes hívásnaplót, külön jelezve az egyes hívás típusokat (bejövõ, kimenõ, nem fogadott)
Extra feladatok lehetnek például:

    A hívásnapló ne csak a telefonszámokat mutassa, hanem nevet, képet is, amennyiben a névjegyzékben megtalálható
    Hívásnapló szûrése, rendezése

## Kedvencek nézet

Egy egyszerû lista nézet rajta egy hozzáadás gombbal. A hozzáadás gombra kattintva ki lehet választani egy contact-ot, majd sikeres kiválasztás után bekerül a listába a contact a telefonszámával együtt. Az adott elem jobb oldalán legyen egy hívás gomb, amire rákattintva fel lehet hívni az adott telefonszámot.

### Példa kód a contact kiválasztásához

Kontakt választó feldobása:

```java	
Intent intent = new Intent(Intent.ACTION_PICK, ContactsContract.Contacts.CONTENT_URI);
startActivityForResult(intent, REQUEST_PICK_CONTACT);
```

Eredmény lekezelése:

```java		
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (resultCode == RESULT_OK) {
        switch (requestCode) {
            case REQUEST_PICK_CONTACT:
                Uri contactUri = data.getData();
                if (contactUri != null) {
                    Cursor c = null;
                    Cursor addrCur = null;
                    try {
                        ContentResolver cr = getContentResolver();
                        c = cr.query(contactUri,null,null, null, null);
                        if (c != null && c.moveToFirst()) {
                            String id = c.getString(c.getColumnIndex(BaseColumns._ID));
                            String name = c.getString(c.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME));
                            Log.d("Launcher", "name: " + name);
                            String contactNumber = null;
                            Cursor cursorPhone = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                                    new String[]{ContactsContract.CommonDataKinds.Phone.NUMBER},
 
                                    ContactsContract.CommonDataKinds.Phone.CONTACT_ID + " = ? AND " +
                                            ContactsContract.CommonDataKinds.Phone.TYPE + " = " +
                                            ContactsContract.CommonDataKinds.Phone.TYPE_MOBILE,
 
                                    new String[]{id},
                                    null);
 
                            if (cursorPhone.moveToFirst()) {
                                contactNumber = cursorPhone.getString(cursorPhone.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
                            }
                            Log.d("Launcher", "contactNumber: " + contactNumber);
                            cursorPhone.close();
                        }
                    } finally {
                        if (c != null) {
                            c.close();
                        }
                        if (addrCur != null) {
                            addrCur.close();
                        }
                    }
                }
 
                break;
            default:
                break;
        }
    }
}
```

A példa kód lekérdezi a kiválasztott contact-ot és megkeresi hozzá azt a telefonszámát, ami mobilként van megjelölve.

## Feltöltendõ anyag

Az AUT portálra feltöltendõ az alkalmazás projekt könyvtárán felül egy 2-3 oldalas felhasználói kézikönyv, ami leírja az elkészített szoftver funkcióit, és képeket is tartalmaz minden releváns képernyõrõl. A felhasználói kézikönyv is kötelezõ része a feladatnak, nélküle az anyag nem értékelhetõ!
A feltöltés határideje: vasárnap 23:59