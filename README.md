# Laravel E-Commerce Aplikace - Technická Dokumentace

## Obsah
1. [Přehled Projektu](#přehled-projektu)
2. [Technologický Stack](#technologický-stack)
3. [Architektura a Struktura](#architektura-a-struktura)
4. [Návrh Databáze](#návrh-databáze)
5. [Autentifikace a Autorizace](#autentifikace-a-autorizace)
6. [Správa Relací](#správa-relací)
7. [Frontend Architektura](#frontend-architektura)
8. [Blade Template Systém](#blade-template-systém)
9. [Systém Košíku](#systém-košíku)
10. [Zpracování Objednávek](#zpracování-objednávek)
11. [Admin Panel](#admin-panel)
12. [Správa Konfigurace](#správa-konfigurace)
13. [Systém Logování](#systém-logování)
14. [Správa Assetů](#správa-assetů)
15. [Bezpečnostní Funkce](#bezpečnostní-funkce)
16. [API a Routy](#api-a-routy)
17. [Testování](#testování)
18. [Nasazení](#nasazení)

---

## Přehled Projektu

Toto je plně funkční e-commerce aplikace postavená na Laravel 12, která obsahuje správu produktů, funkcionalitu nákupního košíku, zpracování objednávek a administrační rozhraní. Aplikace následuje MVC (Model-View-Controller) architekturu a implementuje moderní postupy webového vývoje.

### Klíčové Funkce
- **Katalog Produktů**: Procházení produktů podle kategorií s stránkováním
- **Nákupní Košík**: Košík založený na relacích se správou množství
- **Autentifikace Uživatelů**: Registrace, přihlášení a správa profilu
- **Admin Panel**: Správa produktů a kategorií pro administrátory
- **Zpracování Objednávek**: Kompletní proces checkoutu s generováním PDF faktur
- **Responzivní Design**: Mobilně přívětivé rozhraní pomocí Tailwind CSS

---

## Technologický Stack

### Backend
- **Laravel 12**: PHP framework pro vývoj webových aplikací
- **PHP 8.2+**: Server-side programovací jazyk
- **MySQL/SQLite**: Systém správy databáze
- **Composer**: Správce PHP závislostí

### Frontend
- **Tailwind CSS 4.1**: Utility-first CSS framework
- **Alpine.js**: Lehký JavaScript framework pro interaktivitu
- **Vite**: Moderní build nástroj pro frontend assety
- **Vue.js 3**: Progresivní JavaScript framework (nakonfigurován, ale neintenzivně používán)

### Další Knihovny
- **DomPDF**: Generování PDF pro faktury
- **Inertia.js**: Moderní monolitický přístup (nakonfigurován)
- **Ziggy**: Generování rout pro JavaScript

---

## Architektura a Struktura

### Struktura Adresářů
```
example-app/
├── app/                    # Jádro aplikace
│   ├── Console/           # Artisan příkazy
│   ├── Http/              # HTTP vrstva
│   │   ├── Controllers/   # Zpracovatelé požadavků
│   │   ├── Middleware/    # Filtry požadavků
│   │   └── Requests/      # Validace formulářů
│   ├── Models/            # Eloquent ORM modely
│   ├── Providers/         # Service providery
│   └── Services/          # Business logika služeb
├── config/                # Konfigurační soubory
├── database/              # Migrace a seedery databáze
├── public/                # Kořen dokumentů webového serveru
├── resources/             # Frontend zdroje
│   ├── views/             # Blade šablony
│   ├── css/               # Styly
│   └── js/                # JavaScript soubory
├── routes/                # Definice rout
├── storage/               # Úložiště souborů a logy
└── tests/                 # Testy aplikace
```

### Design Patterns
- **MVC Pattern**: Oddělení zodpovědností mezi modely, pohledy a kontrolery
- **Repository Pattern**: Abstrakce přístupu k datům přes Eloquent modely
- **Service Pattern**: Zapouzdření business logiky v service třídách
- **Middleware Pattern**: Filtrování a zpracování požadavků/odpovědí

---

## Návrh Databáze

### Schéma Databáze

#### Tabulka Users
```sql
users (
    id (bigint, primární klíč)
    name (varchar)
    email (varchar, unikátní)
    email_verified_at (timestamp, nullable)
    password (varchar, hashovaný)
    is_admin (boolean, výchozí: false)
    remember_token (varchar, nullable)
    created_at (timestamp)
    updated_at (timestamp)
)
```

#### Tabulka Categories
```sql
categories (
    id (bigint, primární klíč)
    name (varchar)
    slug (varchar, unikátní)
    description (text, nullable)
    created_at (timestamp)
    updated_at (timestamp)
)
```

#### Tabulka Products
```sql
products (
    id (bigint, primární klíč)
    category_id (bigint, cizí klíč)
    name (varchar)
    slug (varchar, unikátní)
    description (text)
    price (decimal(10,2))
    image_path (varchar, nullable)
    created_at (timestamp)
    updated_at (timestamp)
)
```

#### Tabulka Orders
```sql
orders (
    id (bigint, primární klíč)
    user_id (bigint, cizí klíč)
    status (varchar)
    total_amount (decimal(10,2))
    shipping_address (text)
    created_at (timestamp)
    updated_at (timestamp)
)
```

#### Tabulka Order Items
```sql
order_items (
    id (bigint, primární klíč)
    order_id (bigint, cizí klíč)
    product_id (bigint, cizí klíč)
    quantity (integer)
    price (decimal(10,2))
    created_at (timestamp)
    updated_at (timestamp)
)
```

#### Tabulka Sessions
```sql
sessions (
    id (varchar, primární klíč)
    user_id (bigint, nullable, cizí klíč)
    ip_address (varchar, nullable)
    user_agent (text, nullable)
    payload (text)
    last_activity (integer)
)
```

### Vztahy
- **User → Orders**: Jeden k mnoha
- **Category → Products**: Jeden k mnoha
- **Order → OrderItems**: Jeden k mnoha
- **Product → OrderItems**: Jeden k mnoha
- **User → Sessions**: Jeden k mnoha

---

## Autentifikace a Autorizace

### Systém Autentifikace
Aplikace používá vestavěný autentifikační systém Laravel s integrací balíčku Breeze.

#### Registrace Uživatele
```php
// Proces registrace zahrnuje:
- Validaci emailu
- Hashování hesla (bcrypt)
- Ověření emailu (volitelné)
- Automatické přihlášení po registraci
```

#### Proces Přihlášení
```php
// Validace přihlášení:
- Ověření emailu/hesla
- Funkcionalita "Zapamatovat si mě"
- Vytvoření relace
- Přesměrování na zamýšlenou stránku
```

### Systém Autorizace
Vlastní autorizace je implementována přes middleware a atributy modelů.

#### Admin Middleware
```php
class AdminMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        if (!Auth::check() || !Auth::user()->is_admin) {
            return redirect('/')->with('error', 'Neautorizovaný přístup.');
        }
        return $next($request);
    }
}
```

#### Ochrana Rout
```php
// Admin routy jsou chráněny middleware
Route::middleware(['auth', AdminMiddleware::class])->group(function () {
    Route::prefix('admin')->name('admin.')->group(function () {
        Route::resource('products', AdminProductController::class);
    });
});
```

---

## Správa Relací

### Konfigurace Relací
Aplikace používá databázově řízené relace pro perzistenci a škálovatelnost.

#### Driver Relací
```php
// config/session.php
'driver' => env('SESSION_DRIVER', 'database'),
'table' => env('SESSION_TABLE', 'sessions'),
'lifetime' => (int) env('SESSION_LIFETIME', 120),
```

#### Použití Relací v Košíku
```php
// CartService používá relace pro perzistenci košíku
class CartService
{
    private const CART_KEY = 'shopping_cart';
    
    public function getCart()
    {
        return Session::get(self::CART_KEY, []);
    }
    
    public function addToCart(Product $product, int $quantity = 1)
    {
        $cart = $this->getCart();
        // Logika přidání produktu do košíku
        Session::put(self::CART_KEY, $cart);
    }
}
```

### Bezpečnost Relací
- **CSRF Ochrana**: Všechny formuláře obsahují CSRF tokeny
- **Šifrování Relací**: Volitelné šifrování dat relací
- **Bezpečné Cookies**: HTTPS-only cookies v produkci
- **Timeout Relací**: Konfigurovatelná životnost relací

---

## Frontend Architektura

### Kompilace Assetů
Aplikace používá Vite pro moderní kompilaci assetů a vývoj.

#### Konfigurace Vite
```javascript
// vite.config.js
export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
            ],
            refresh: true,
        }),
    ],
});
```

#### Integrace Tailwind CSS
```javascript
// tailwind.config.js
export default {
    content: [
        './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
        './storage/framework/views/*.php',
        './resources/views/**/*.blade.php',
    ],
    theme: {
        extend: {
            fontFamily: {
                sans: ['Figtree', ...defaultTheme.fontFamily.sans],
            },
        },
    },
    plugins: [forms],
};
```

### JavaScript Architektura
- **Alpine.js**: Lehká reaktivita pro interaktivní komponenty
- **Vue.js 3**: Nakonfigurován pro potenciální SPA funkce
- **Ziggy**: Generování rout pro JavaScript

---

## Blade Template Systém

### Struktura Šablon
Aplikace používá Laravel Blade templating engine s komponentově založenou architekturou.

#### Systém Layoutů
```php
// resources/views/layouts/app.blade.php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>{{ config('app.name', 'Laravel E-Shop') }}</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="font-sans antialiased">
    <div class="min-h-screen bg-gray-100">
        @include('layouts.navigation')
        <main class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                {{ $slot }}
            </div>
        </main>
    </div>
</body>
</html>
```

#### Systém Komponent
```php
// Znovupoužitelné komponenty v resources/views/components/
- primary-button.blade.php
- dropdown.blade.php
- modal.blade.php
- nav-link.blade.php
```

#### Dědičnost Šablon
```php
// Dceřiné šablony rozšiřují layouty
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Produkty') }}
        </h2>
    </x-slot>
    
    <!-- Obsah stránky -->
</x-app-layout>
```

### Blade Direktivy
- **@foreach**: Cyklus přes kolekce
- **@if/@else**: Podmíněné vykreslování
- **@auth/@guest**: Kontroly autentifikace
- **@include**: Zahrnutí jiných šablon
- **@csrf**: CSRF ochranné tokeny

---

## Systém Košíku

### Architektura Cart Service
Systém košíku je implementován jako service třída pro lepší oddělení zodpovědností.

#### Implementace CartService
```php
class CartService
{
    private const CART_KEY = 'shopping_cart';

    // Získání aktuálního obsahu košíku
    public function getCart()
    {
        return Session::get(self::CART_KEY, []);
    }

    // Přidání produktu do košíku
    public function addToCart(Product $product, int $quantity = 1)
    {
        $cart = $this->getCart();
        $productId = $product->id;

        if (isset($cart[$productId])) {
            $cart[$productId]['quantity'] += $quantity;
        } else {
            $cart[$productId] = [
                'id' => $product->id,
                'name' => $product->name,
                'price' => $product->price,
                'quantity' => $quantity,
                'image_path' => $product->image_path
            ];
        }

        Session::put(self::CART_KEY, $cart);
    }

    // Výpočet celkové ceny košíku
    public function getTotal()
    {
        $cart = $this->getCart();
        $total = 0;

        foreach ($cart as $item) {
            $total += $item['price'] * $item['quantity'];
        }

        return $total;
    }
}
```

#### Cart Controller
```php
class CartController extends Controller
{
    protected $cartService;

    public function __construct(CartService $cartService)
    {
        $this->cartService = $cartService;
    }

    public function add(Request $request, Product $product)
    {
        $quantity = $request->input('quantity', 1);
        $this->cartService->addToCart($product, $quantity);
        return redirect()->back()->with('success', 'Produkt byl úspěšně přidán do košíku.');
    }
}
```

### Funkce Košíku
- **Perzistence Relací**: Košík přežívá prohlížečové relace
- **Správa Množství**: Přidání, aktualizace, odstranění množství
- **Výpočet Celkové Ceny**: Automatické výpočty cen
- **Informace o Produktu**: Ukládání detailů produktu pro zobrazení

---

## Zpracování Objednávek

### Proces Checkoutu
Proces checkoutu zahrnuje několik kroků s validací a databázovými transakcemi.

#### Checkout Controller
```php
class CheckoutController extends Controller
{
    protected $cartService;

    public function __construct(CartService $cartService)
    {
        $this->cartService = $cartService;
    }

    public function process(Request $request)
    {
        // Validace dodací adresy
        $validatedData = $request->validate([
            'shipping_address' => 'required|string|max:500',
        ]);

        $cart = $this->cartService->getCart();

        try {
            DB::beginTransaction();

            // Vytvoření objednávky
            $order = Order::create([
                'user_id' => Auth::id(),
                'status' => 'pending',
                'total_amount' => $this->cartService->getTotal(),
                'shipping_address' => $validatedData['shipping_address'],
            ]);

            // Vytvoření položek objednávky
            foreach ($cart as $item) {
                OrderItem::create([
                    'order_id' => $order->id,
                    'product_id' => $item['id'],
                    'quantity' => $item['quantity'],
                    'price' => $item['price']
                ]);
            }

            DB::commit();
            $this->cartService->clearCart();

            // Generování PDF faktury
            $fileName = $this->generateInvoicePdf($order);

            return redirect()->route('orders.show', $order)
                ->with('success', 'Objednávka byla úspěšně vytvořena!');

        } catch (\Exception $e) {
            DB::rollBack();
            return redirect()->back()
                ->with('error', 'Došlo k chybě při zpracování vaší objednávky.');
        }
    }
}
```

### Generování PDF
```php
protected function generateInvoicePdf(Order $order)
{
    $pdf = app('dompdf.wrapper');
    $pdf->loadView('pdfs.invoice', [
        'order' => $order,
        'items' => $order->items,
        'user' => Auth::user(),
    ]);

    $path = Storage::disk('public')->path('invoices/');
    if (!file_exists($path)) {
        mkdir($path, 0777, true);
    }

    $fileName = 'invoice_' . $order->id . '.pdf';
    $pdf->save($path . $fileName);

    return $fileName;
}
```

---

## Admin Panel

### Admin Rozhraní
Admin panel poskytuje možnosti správy produktů a kategorií.

#### Admin Product Controller
```php
class ProductController extends Controller
{
    public function index()
    {
        $products = Product::latest()->paginate(10);
        return view('admin.products.index', compact('products'));
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|max:255',
            'description' => 'required',
            'price' => 'required|numeric|min:0',
            'category_id' => 'required|exists:categories,id',
            'image_url' => 'required|url|max:255'
        ]);

        // Generování slugu z názvu
        $validated['slug'] = Str::slug($validated['name']);
        $validated['image_path'] = $validated['image_url'];

        Product::create($validated);

        return redirect()->route('admin.products.index')
            ->with('success', 'Produkt byl úspěšně vytvořen.');
    }
}
```

#### Admin Pohledy
- **Product Index**: Seznam všech produktů s akcemi editace/mazání
- **Product Create**: Formulář pro přidání nových produktů
- **Product Edit**: Formulář pro úpravu existujících produktů
- **Category Management**: CRUD operace pro kategorie

### Admin Funkce
- **Správa Produktů**: Vytvoření, čtení, aktualizace, mazání produktů
- **Správa Kategorií**: Organizace produktů do kategorií
- **Podpora URL Obrázků**: Použití externích URL obrázků pro produkty
- **Generování Slugů**: Automatické URL-přívětivé slugy
- **Hromadné Operace**: Hromadné akce na produktech

---

## Správa Konfigurace

### Konfigurace Prostředí
Aplikace používá Laravel systém konfigurace založený na prostředí.

#### Klíčové Konfigurační Soubory
- **.env**: Nastavení specifická pro prostředí
- **config/app.php**: Nastavení aplikace
- **config/database.php**: Připojení k databázi
- **config/session.php**: Správa relací
- **config/logging.php**: Konfigurace logování

#### Konfigurace Databáze
```php
// config/database.php
'default' => env('DB_CONNECTION', 'sqlite'),

'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'example_app'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => env('DB_CHARSET', 'utf8mb4'),
    'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
],
```

### Konfigurace Aplikace
```php
// config/app.php
'name' => env('APP_NAME', 'Laravel'),
'env' => env('APP_ENV', 'production'),
'debug' => (bool) env('APP_DEBUG', false),
'url' => env('APP_URL', 'http://localhost'),
'timezone' => 'UTC',
'locale' => 'en',
'fallback_locale' => 'en',
'faker_locale' => 'en_US',
```

---

## Systém Logování

### Konfigurace Logování
Laravel používá Monolog pro logování s podporou více kanálů.

#### Log Kanály
```php
// config/logging.php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => explode(',', env('LOG_STACK', 'single')),
        'ignore_exceptions' => false,
    ],

    'single' => [
        'driver' => 'single',
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
        'replace_placeholders' => true,
    ],

    'daily' => [
        'driver' => 'daily',
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
        'days' => env('LOG_DAILY_DAYS', 14),
        'replace_placeholders' => true,
    ],
],
```

#### Použití Logování
```php
// V kontrolerech a službách
Log::info('Uživatel se přihlásil', ['user_id' => $user->id]);
Log::error('Platba selhala', ['order_id' => $order->id, 'error' => $e->getMessage()]);
Log::debug('Košík aktualizován', ['cart_count' => count($cart)]);
```

### Úrovně Logování
- **emergency**: Systém je nepoužitelný
- **alert**: Akce musí být provedena okamžitě
- **critical**: Kritické podmínky
- **error**: Chybové podmínky
- **warning**: Varovné podmínky
- **notice**: Normální, ale významné podmínky
- **info**: Informační zprávy
- **debug**: Debug úrovně zpráv

---

## Správa Assetů

### Integrace Vite
Moderní kompilace assetů s hot module replacement pro vývoj.

#### Kompilace Assetů
```bash
# Vývoj
npm run dev

# Produkční build
npm run build
```

#### Načítání Assetů v Šablonách
```php
// V Blade šablonách
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

### CSS Architektura
- **Tailwind CSS**: Utility-first CSS framework
- **Vlastní Komponenty**: Znovupoužitelné styly komponent
- **Responzivní Design**: Mobile-first přístup

### JavaScript Architektura
- **Alpine.js**: Lehká reaktivita
- **Vue.js**: Komponentově založený framework (nakonfigurován)
- **ES6 Moduly**: Moderní JavaScript syntax

---

## Bezpečnostní Funkce

### CSRF Ochrana
Všechny formuláře obsahují CSRF tokeny pro prevenci cross-site request forgery útoků.

```php
// Ve formulářích
@csrf

// V AJAX požadavcích
<meta name="csrf-token" content="{{ csrf_token() }}">
```

### Bezpečnost Autentifikace
- **Hashování Hesel**: Bcrypt algoritmus pro ukládání hesel
- **Bezpečnost Relací**: Bezpečná konfigurace relací
- **Zapamatovat Si Mě**: Bezpečná funkcionalita zapamatování
- **Ověření Emailu**: Volitelné ověření emailu

### Bezpečnost Autorizace
- **Middleware Ochrana**: Ochrana na úrovni rout
- **Model Policies**: Jemně odstupňovaná autorizace
- **Admin Kontroly**: Role-based access control

### Validace Vstupů
```php
// Validace formulářů v kontrolerech
$validated = $request->validate([
    'name' => 'required|max:255',
    'email' => 'required|email|unique:users',
    'password' => 'required|min:8|confirmed',
]);
```

---

## API a Routy

### Struktura Rout
Aplikace používá Laravel systém routování s middleware ochranou.

#### Web Routy
```php
// routes/web.php
Route::get('/products', [ProductController::class, 'index'])->name('products.index');
Route::get('/products/{product:slug}', [ProductController::class, 'show'])->name('products.show');

Route::middleware('auth')->group(function () {
    Route::get('/cart', [CartController::class, 'show'])->name('cart.show');
    Route::post('/cart/add/{product}', [CartController::class, 'add'])->name('cart.add');
});

Route::middleware(['auth', AdminMiddleware::class])->group(function () {
    Route::prefix('admin')->name('admin.')->group(function () {
        Route::resource('products', AdminProductController::class);
    });
});
```

#### Autentifikační Routy
```php
// routes/auth.php
Route::middleware('guest')->group(function () {
    Route::get('register', [RegisteredUserController::class, 'create'])->name('register');
    Route::post('register', [RegisteredUserController::class, 'store']);
    Route::get('login', [AuthenticatedSessionController::class, 'create'])->name('login');
    Route::post('login', [AuthenticatedSessionController::class, 'store']);
});

Route::middleware('auth')->group(function () {
    Route::post('logout', [AuthenticatedSessionController::class, 'destroy'])->name('logout');
});
```

### Route Model Binding
```php
// Automatické rozlišení modelu
Route::get('/products/{product:slug}', [ProductController::class, 'show']);
// Automaticky rozliší Product model podle slugu
```

---

## Testování

### Testovací Framework
Aplikace používá Pest PHP pro testování s Laravel integrací.

#### Struktura Testů
```
tests/
├── Feature/           # Feature testy
├── Unit/             # Unit testy
└── TestCase.php      # Základní test case
```

#### Příklady Testů
```php
// Feature test příklad
test('uživatel může zobrazit produkty', function () {
    $response = $this->get('/products');
    $response->assertStatus(200);
    $response->assertViewIs('products.index');
});

// Unit test příklad
test('cart service správně vypočítá celkovou cenu', function () {
    $cartService = new CartService();
    $product = Product::factory()->create(['price' => 10.00]);
    
    $cartService->addToCart($product, 2);
    
    expect($cartService->getTotal())->toBe(20.00);
});
```

### Testovací Příkazy
```bash
# Spustit všechny testy
php artisan test

# Spustit specifický test soubor
php artisan test tests/Feature/ProductTest.php

# Spustit testy s pokrytím
php artisan test --coverage
```

---

## Nasazení

### Nastavení Prostředí
1. **Produkční Prostředí**: Nastavit `APP_ENV=production`
2. **Debug Režim**: Vypnout debug režim (`APP_DEBUG=false`)
3. **Databáze**: Nakonfigurovat produkční databázi
4. **Cache**: Povolit cache rout a konfigurace

### Optimalizace Výkonu
```bash
# Cache konfigurace
php artisan config:cache

# Cache rout
php artisan route:cache

# Cache pohledů
php artisan view:cache

# Optimalizace autoloaderu
composer install --optimize-autoloader --no-dev
```

### Bezpečnostní Checklist
- [ ] Nastavit bezpečnou konfiguraci relací
- [ ] Nakonfigurovat HTTPS přesměrování
- [ ] Nastavit bezpečné cookie nastavení
- [ ] Povolit CSRF ochranu
- [ ] Nakonfigurovat správná oprávnění souborů
- [ ] Nastavit logování chyb
- [ ] Nakonfigurovat zálohovací systémy

### Migrace Databáze
```bash
# Spustit migrace v produkci
php artisan migrate --force

# Seed databáze pokud je potřeba
php artisan db:seed --force
```

### Úložiště Souborů
- **Public Storage**: Nakonfigurovat pro PDF faktury
- **Oprávnění Souborů**: Nastavit správná oprávnění adresářů
- **Strategie Zálohování**: Implementovat pravidelné zálohy

---

## Rychlý Start

### Předpoklady
- PHP 8.2 nebo vyšší
- Composer
- Node.js a npm
- MySQL nebo SQLite databáze

### Instalace
1. **Klonovat repozitář**
   ```bash
   git clone <repository-url>
   cd example-app
   ```

2. **Nainstalovat PHP závislosti**
   ```bash
   composer install
   ```

3. **Nainstalovat Node.js závislosti**
   ```bash
   npm install
   ```

4. **Nastavení prostředí**
   ```bash
   cp .env.example .env
   php artisan key:generate
   ```

5. **Nastavení databáze**
   ```bash
   php artisan migrate
   php artisan db:seed
   ```

6. **Build assetů**
   ```bash
   npm run build
   ```

7. **Spustit server**
   ```bash
   php artisan serve
   ```

### Výchozí Admin Uživatel
Po spuštění migrací a seederů můžete vytvořit admin uživatele:
```bash
php artisan tinker
```
```php
$user = \App\Models\User::first();
$user->is_admin = true;
$user->save();
```

---

## Závěr

Tato Laravel e-commerce aplikace demonstruje moderní postupy webového vývoje s důrazem na udržovatelnost, bezpečnost a uživatelskou zkušenost. Modulární architektura umožňuje snadné rozšiřování a modifikace, zatímco komplexní testovací framework zajišťuje spolehlivost.

Aplikace úspěšně implementuje všechny základní e-commerce funkcionality včetně správy produktů, operací s nákupním košíkem, zpracování objednávek a administračních kontrol. Použití moderních nástrojů jako Vite, Tailwind CSS a Alpine.js poskytuje responzivní a interaktivní uživatelské rozhraní.

Pro další vývoj zvažte implementaci:
- Integrace platební brány (Stripe, PayPal)
- Email notifikace
- Pokročilé vyhledávání a filtrování
- Recenze a hodnocení produktů
- Správa inventáře
- Vícejazyčná podpora
- API endpointy pro mobilní aplikace
