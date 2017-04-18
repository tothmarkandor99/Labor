# Labor 12 - Firebase

## Bevezető

A labor során egy fórum alkalmazás kerül megvalósításra a Firebase Backend as a Service (BaaS) felhasználásával. A feladat célja, hogy szemléltese, hogyan lehet közös backendet használó alkalmazást fejleszteni saját backend kód fejlesztése nélkül.

A Firebase manapság az egyik legnépszerűbb Backend as a Service megoldás Android, iOS és web kliensek támogatáásval, mely számos szolgáltatást biztosít, mint például:
- real-time adatbáziskezelés
- storage
- authentikáció
- push értesítések
- analytics
- crash reporting

További általános információk a Firebase-ről: https://firebase.google.com/.

A laborfogalkozás célja, hogy bemutassa a Firebase legfontosabb szolgáltatásait egy komplett alklamazás megvalósítása keretében. A megvalósítandó alkalmazás egy fórum megoldás lesz, melyen keresztül a felhasználük szöveges üzeneteket tudnak megosztani egymással valós időben, melyekhez opcionálisan képek is csatolhatók.
Az alkalmazás az alőbbi fő funkciókat támogatja:
- regisztráció, bejelentkezés,
- üzenetek listázása,
- üzenet írás,
- üzenetek megjelenítése valós időben,
- képek csatolása üzenetekhez,
- crash reporting,
- analitika.

A labor soárn nagyobb kódrészek kerülnek megírásra, ami miatt elnézést kérünk, de ez szükséges ahhoz, hogy egy hello-word jellegű alkalmazásnál többet tudjunk átadni a tárgy keretében. **Az anyag részletes megértéséhez javasoljuk, hogy figyelje a laborvezető utasításait és labor után is 10-20 percet szánjon a kódrészek megérétésére.**

*Az útmutatóban levő példa kódok esetében a szöveges elemeket nem tettük strings.xml-be a könnyebb olvashatóság érdekében, de éles projektekben ezeket természetesen mindig ki kell szervezni erőforrásba.*

## Projekt előkészítése, konfiguráció

Első lépésként létre kell hozni egy Firebase projektet a Firebase admin felületén (Firebase console), majd egy Android Studio projektet és össze kell kötni az Android projektet a Firebase-ben létrehozott projekttel:
- Navigáljunk a Firebase console felületére: https://console.firebase.google.com/.
- Jelentkezzünk be jobb felül.
- Hozzunk létre egy új projektet az *Add project* elemet választva középen.
- A projekt neve legyen *BMEForum*, a Country pedig *Hungary*.

Sikeres projekt létrehozás után fussák át a laborvezetővel közösen a Firebase console felületét az alábbi elemekre kitérve:
- Authentication, Database és Storage,
- Database>Rules.



Hozzunk létre egy új projektet Android Studio-ban, válasszuk az *Empty Activity* sablont és a kezdő Activity-nk neve legyen **LoginActivity**, mivel elsőként a regisztrációs és bejelentkező nézetet fogjuk megvalósítani. Az egyszerűség kedvéért ugyanazt a felületet fogjuk használni regisztráció és bejelentkezés céljából.

## Regisztráció, bejelentkezés

Első lépésként valósítsuk meg a bejelentkező képernyő felületét. Mivel ehhez hasonló felületeket már készítettünk korábban egyszerűség kedvéért megadjuk a felület kódját, melyet helyezzen az *activity_login.xml*-be:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="56dp"
        android:paddingLeft="24dp"
        android:paddingRight="24dp">

        <ImageView android:src="@mipmap/ic_launcher"
            android:layout_width="wrap_content"
            android:layout_height="72dp"
            android:layout_marginBottom="24dp"
            android:layout_gravity="center_horizontal" />

        <android.support.design.widget.TextInputLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp">
            <EditText android:id="@+id/etEmail"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="textEmailAddress"
                android:text="p@p.hu"
                android:hint="email" />
        </android.support.design.widget.TextInputLayout>

        <android.support.design.widget.TextInputLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp"
            app:passwordToggleEnabled="true">
            <EditText android:id="@+id/etPassword"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="textPassword"
                android:text="abcabc"
                android:hint="Password"/>
        </android.support.design.widget.TextInputLayout>

        <Button
            android:id="@+id/btnLogin"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="24dp"
            android:layout_marginBottom="12dp"
            android:padding="12dp"
            android:text="Login"/>

        <Button
            android:id="@+id/btnRegister"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="12dp"
            android:text="Register"/>

    </LinearLayout>
</ScrollView>
```

## Postok listázása


## Postok készítése


## Push értesítése


## Crash reporting


## Analitika


## Képek csatolása a postokhoz

