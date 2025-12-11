# Moderni Emojiji in EmojiPicker (`androidx.emoji2`)

## Utemeljitev izbire

V sodobnih mobilnih aplikacijah so emojiji postali nepogrešljiv del komunikacije. Uporabniki jih uporabljajo za hitro vizualno identifikacijo, izražanje čustev ali označevanje kategorij (npr. v moji aplikaciji *reStock* lahko produkt "Jabolka" označimo z "🍎" ali "Jabolka🍎").

**Problem:** Android ekosistem je fragmentiran. Uporabnik z najnovejšim Android 13 sistemom lahko pošlje nov emoji (npr. "stopljen obraz" 🫠), uporabnik na starejšem Androidu (npr. Android 8) pa bo namesto tega videl prazen kvadratek (t.i. "tofu" ☐), ker njegov operacijski sistem nima posodobljene pisave za ta znak.

**Rešitev:** Knjižnica **`androidx.emoji2`** rešuje ta problem tako, da omogoča povratno združljivost (backward compatibility). Če sistem ne pozna emojija, ga knjižnica nariše sama s pomočjo priložene ali prenosljive pisave. Dodatek **`emoji2-emojipicker`** pa rešuje problem uporabniškega vmesnika, saj ponuja standardizirano tipkovnico za izbiro emojijev.

## Prednosti in slabosti

| Prednosti | Slabosti |
| :--- | :--- |
| **Kompatibilnost:** Deluje na vseh napravah od Android 5.0 (API 21) naprej. | **Odvisnost od interneta:** Če uporabljamo "Downloadable Fonts" (privzeto), naprava potrebuje dostop do interneta ob prvem zagonu za prenos pisave. |
| **Samodejnost:** `EmojiTextView` in `EmojiEditText` samodejno zaznata in nadomestita sistemske pomanjkljivosti. | **Custom Views:** Če rišemo neposredno na `Canvas`, moramo ročno klicati `EmojiCompat.process()`. |
| **EmojiPicker:** Prihrani tedne razvoja lastne tipkovnice za emojije. | **Velikost aplikacije:** Če pisavo vključimo v APK (bundled), se velikost poveča za cca 7-10 MB (zato se priporoča downloadable fonts). |
| **Posodobitve:** Aplikacija podpira nove emojije takoj, ko Google posodobi knjižnico, brez čakanja na update Android sistema. | |

## Licenca

**Apache License 2.0** (Open Source)

*   Dovoljena komercialna uporaba.
*   Dovoljeno spreminjanje in distribucija.
*   Brezplačna uporaba.

## Število uporabnikov

Knjižnica je del **Android Jetpack** paketa in je vključena v `AppCompat` od verzije 1.4 naprej. To pomeni, da je osnovna funkcionalnost prisotna v **milijardah** Android naprav in v večini sodobnih aplikacij na Google Play trgovini (WhatsApp, Telegram, Signal, Facebook uporabljajo to ali podobne rešitve za poenotenje prikaza).

Knjižnica `emoji2-emojipicker` je novejša, a hitro pridobiva na popularnosti zaradi enostavne integracije.

## Časovna in prostorska zahtevnost

*   **Časovna zahtevnost:** **O(N)**, kjer je N dolžina besedila. Knjižnica mora preiskati string (scan) in poiskati zaporedja znakov, ki ustrezajo emojijem, ter jih nadomestiti z `EmojiSpan` objekti. To je zelo optimizirano in ne vpliva na zaznavno hitrost delovanja UI.
*   **Prostorska zahtevnost:** **Nizka**. Pisava (font file) se običajno deli med aplikacijami preko Google Play Services (GMS Core), zato ne zaseda dodatnega pomnilnika za vsako aplikacijo posebej.

## Vzdrževanje

*   **Razvijalec:** Google (Android Jetpack Team).
*   **Aktivnost:** Zelo aktivno vzdrževanje.
*   **Posodobitve:** Knjižnica se posodobi vsakič, ko Unicode konzorcij izda nov set emojijev (običajno enkrat letno), ter ob popravkih hroščev.
*   Zadnja večja verzija: `1.5.0` (stabilna).

---

## Primer uporabe v aplikaciji (Restock)

V aplikaciji sem knjižnico uporabil za dodajanje ikone (emojija) k imenu produkta ali lokacije.

### 1. Dodajanje odvisnosti (Gradle)

V `build.gradle` (Module: app):

```kotlin
dependencies {
    // Osnovna knjižnica za podporo emojijem
    implementation("androidx.emoji2:emoji2:1.5.0")
    
    // UI komponenta za izbiro emojijev (Emoji Picker)
    implementation("androidx.emoji2:emoji2-emojipicker:1.5.0")
}
```

### 2. Implementacija v XML (Layout)

Uporabil sem `EmojiPickerView`, ki je privzeto skrit (`visibility="gone"`), in gumb za prikaz.

```kotlin
<!-- Gumb za odpiranje izbirnika -->
<Button
    android:id="@+id/btnEmoji"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="😀"
    style="@style/Widget.MaterialComponents.Button.TextButton"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintTop_toTopOf="@+id/tilName"
    app:layout_constraintBottom_toBottomOf="@+id/tilName"/>

<!-- Emoji Picker komponenta -->
<androidx.emoji2.emojipicker.EmojiPickerView
    android:id="@+id/emojiPicker"
    android:layout_width="match_parent"
    android:layout_height="300dp"
    android:visibility="gone"
    android:background="#EFEFEF"
    app:emojiGridRows="9"
    app:emojiGridColumns="9"
    app:layout_constraintTop_toBottomOf="@+id/tilName"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"/>
```

### 3. Logika v Kotlinu (`AddActivity.kt`)

Ob kliku na gumb prikažemo izbirnik. Ob izbiri emojija ga vstavimo v `TextInputEditText` na mesto kurzorja.

```kotlin
// 1. Preklapljanje vidnosti izbirnika
binding.btnEmoji.setOnClickListener {
    if (binding.emojiPicker.visibility == View.VISIBLE) {
        binding.emojiPicker.visibility = View.GONE
    } else {
        binding.emojiPicker.visibility = View.VISIBLE
        // Skrij sistemsko tipkovnico za boljšo preglednost
        val imm = getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager
        imm.hideSoftInputFromWindow(binding.etName.windowToken, 0)
    }
}

// 2. Obravnava klika na emoji
binding.emojiPicker.setOnEmojiPickedListener { emojiItem ->
    // emojiItem.emoji vrne Unicode string izbranega znaka
    val start = binding.etName.selectionStart
    val end = binding.etName.selectionEnd
    
    // Vstavljanje emojija na pozicijo kurzorja
    binding.etName.text?.replace(start, end, emojiItem.emoji)
}
```

---

## Demo (GIF postopka dodajanja v moji aplikaciji)

![Zaslonski posnetek aplikacije](/vsebina/prikaz_uporabe_emoji2.gif)
