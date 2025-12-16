# COIL (Coroutine Image Loader)

Coil (Coroutine Image Loader) je odprtokodna knjižnica za nalaganje slik v Android aplikacijah, ki je napisana v Kotlinu in optimizirana za uporabo s Kotlin Coroutines. Primerna za Android View projekte in Jetpack Compose (tudi Compose Multiplatform). Uradna dokumentacija je na voljo na strani, ki si jo delil: https://coil-kt.github.io/coil/

# Prednosti ✅

- Je hitera ker izvaja številne optimizacije, vključno s predpomnjenjem pomnilnika in diska, zmanjševanjem vzorčenja slike in samodejnim zaustavljanjem/preklicem zahtev
- Coli-jev API je enostaven za uporabo saj izkorišča jezikovne funkicje Kotlina za preprostost in minimalno uporabo.
- je "Lightweight", zavzema manj prostora kot  Picasso in Glide 

# Slabosti ❌

- Ni podprta za zelo stare Andorid različice. Minimalen je API 21+
- Ker uporablja Kotlin sintasko, je nejegova uporaba v Java-i nekoliko manj elegantna in zahtevnejša

# Licenca

Avtorske pravice 2025 Coil Contributors. Licencirano pod licenco [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0). To pomeni:

- Lahko uporabljaš Coil v osebnih ali komercialnih projektih brez plačila.
- Lahko spremeniš izvorno kodo knjižnice ali jo vključiš v svojo aplikacijo.

# Ocenitev števila uporabnikov

![število uporabnikv](/res/st_uporabnikov.png)

Na podlagi star-ov in forkov repozitorija bi lahko ocenilo število uporabnikov okoli 15 tisoč

# Časovna zahtevnost

- Nalaganje slik iz omrežja s pomočjo URL-ja je O(n), pri čemer je n = velikost slike
- Nalaganje slike iz diska pa O(n), pri čemer je n = velikost slike

# Prostorska zahtevnost

- Velikost knjižnice je zelo majhna okoli 200-400KB, odvisno od verzije in dodatnih modulov
- Največ pomnilnika uporabi bitmapa in njena poraba je širina * višina * 4 bajte. Ampak z downsamplingom se se poraba lahko zmanjša. Downsampling je manjšanje slike (resolucije) pred dekodiranjem da slika zasede manj RAM-a.

# Vzdrževanje

Knjižnica se aktivno vzdržuje, zadnji commit je bil v začetku decembra.

![Github](/res/github.png)

# Nalaganje slik z uporabo Coil v Android aplikaciji

Za uporabo Coil v Android aplikaciji dodamo naslednje vrstice v `build.gradle`:

```gradle
implementation("io.coil-kt:coil:3.3.0")
implementation("io.coil-kt:coil-network-okhttp:3.3.0")
```

1. Nalaganje slik

   - Za nalaganje slike iz spleta moramo v AndroidManifest.xml dodati dovoljenje:
     `<uses-permission android:name="android.permission.INTERNET" />`  
      Primer nalaganja slike iz URL-ja: `imageView.load("https://picsum.photos/200")`

   - Sliko iz res/drawable mape `imageView.load(R.drawable.running_logo)`

2. Konfiguracija .load()

   - Metoda .load() omogoča dodatne konfiguracije, kot so placeholder, error slika ali animacija preliva:
     ```kotlin
     imageView.load("https://picsum.photos/200") {
         placeholder(R.drawable.place_holder)  // prikaz med nalaganjem
         error(R.drawable.running_logo)        // prikaz ob napaki
         crossfade(true)                        // animacija preliva
     }
     ```

3. Singleton ImageLoader
   - Coil privzeto vključuje enojni (singleton) ImageLoader, ki skrbi za: pridobivanje slik, dekodiranje, predpomnjenje v memory in disk cache, vračanje rezultata v ImageView.
   - Singleton lahko po potrebi konfiguriramo globalno:
        ```  kotlin
        val imageLoader = ImageLoader.Builder(this)
            .crossfade(true)
            .placeholder(R.drawable.place_holder)
            .error(R.drawable.error)
            .build()

        coil.Coil.setImageLoader(imageLoader)
        ```

    - Po tem lahko .load() kličemo preprosto, npr.: `imageView.load(R.drawable.running_logo)`. Globalno nastavljeni placeholder, error in crossfade se bodo avtomatsko uporabili. Tako ni potrebno večkrat podajati teh nastavitev v vsakem load() klicu.

4. Coil ImageLoader – nastavitve

    | Nastavitev / Funkcija                | Pomen / Opis                                                                 |
    |------------------------------------|----------------------------------------------------------------------------|
    | `placeholder(R.drawable.xxx)`       | Slika, ki se prikaže med nalaganjem slike                                   |
    | `error(R.drawable.xxx)`             | Slika, ki se prikaže, če nalaganje slike ne uspe                           |
    | `crossfade(true/500)`               | Omogoča fade animacijo ob nalaganju slike (true ali trajanje v ms)         |
    | `availableMemoryPercentage(%)`      | Delež RAM-a, ki ga ImageLoader lahko uporabi za memory cache          |
    | `bitmapPoolPercentage(%)`           | Delež RAM-a za bitmap pool, kar omogoča recikliranje bitmap             |
    | `memoryCache()`                      | Določi custom LRU memory cache                                             |
    | `diskCache()`                        | Določi LRU disk cache (privzeto v `cache/image_cache/`)                    |
    | `okHttpClient { ... }`               | Custom OkHttp client, npr. z interceptorji ali timeout nastavitvami        |
    | `transformations(...)`               | Transformacije slik (CircleCrop, RoundedCorners, Grayscale, Blur, …)       |
    | `listener(onStart/onSuccess/onError)` | Omogoča spremljanje statusa nalaganja slike                                 |
    | `coil.Coil.setImageLoader(imageLoader)` | Nastavi globalni singleton ImageLoader za vse aktivnosti in fragmente    |


# Vključitev v projekt
Jaz sem COIL vključil v drugo aplikacijo pri PORA. Pri aplikaciji uprabljam razred myApp kateri je tipa Application() za neko globalno stanje. Tam sem si konfiguriral ImageLoader, kateri mi omogoči neko poenoteno stanje za prikaz slik, placeholderjev in error slik.
```kotlin
class MyApp: Application() {
    override fun onCreate() {
        super.onCreate()

        val imageLoader = ImageLoader.Builder(this)
            .crossfade(true)
            .placeholder(R.drawable.place_holder)
            .error(R.drawable.running_logo)
            .build()

        coil.Coil.setImageLoader(imageLoader)
        ...

    }

    ...

}
```

Tako pa za vsako sliko kličem.
```kotlin
   override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
       binding.imageView.load(R.drawable.running_logo)

        binding.fragmentStart.setOnClickListener {
            findNavController().navigate(R.id.navigateFromStartToMain)
        }
    }
```

Tu je prikazano da se res uporabijo konfiguracije katere smo nastavili v ImageLoder.

![Uporaba gif](/res/use_case.gif)
