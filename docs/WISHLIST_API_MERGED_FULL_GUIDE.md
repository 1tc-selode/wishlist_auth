# Projekt Struktúra – Részletes Áttekintés

```
wishlists/
├── app/
│   ├── Http/
│   │   ├── Controllers/         # API vezérlők (Auth, Product, Wishlist, User)
│   │   └── Middleware/          # Egyedi middleware-ek (pl. IsAdmin)
│   ├── Models/                  # Adatmodellek
│   │   ├── User.php             # Felhasználó modell
│   │   ├── Product.php          # Termék modell
│   │   └── Wishlist.php         # Kívánságlista modell
│   └── Providers/               # Laravel szolgáltatók
├── bootstrap/
│   ├── app.php                  # Bootstrap konfiguráció
│   └── cache/                   # Gyorsítótár
├── config/                      # Konfigurációs fájlok
│   ├── app.php
│   ├── auth.php
│   ├── database.php
│   └── ...
├── database/
│   ├── factories/               # Tesztadat generátorok
│   │   ├── UserFactory.php      # Felhasználó generálás
│   │   ├── ProductFactory.php   # Termék generálás
│   │   └── WishlistFactory.php  # Kívánságlista generálás
│   ├── migrations/              # Adatbázis szerkezet
│   └── seeders/                 # Tesztadat feltöltés
├── public/                      # Publikus fájlok
├── resources/                   # Nézetek, statikus erőforrások
├── routes/
│   └── api.php                  # API útvonalak
├── storage/                     # Fájlok, logok, cache
├── tests/
│   ├── Feature/                 # API funkció tesztek
│   │   ├── AuthApiTest.php
│   │   ├── ProductApiTest.php
│   │   ├── WishlistApiTest.php
│   │   └── UserApiTest.php
│   └── ...
├── vendor/                      # Composer csomagok
├── artisan                      # Laravel CLI
├── composer.json                # Composer konfiguráció
├── package.json                 # NPM konfiguráció
├── phpunit.xml                  # PHPUnit teszt konfiguráció
├── vite.config.js               # Vite frontend build
└── README.md                    # Projekt leírás
```

---

## Modellek és mezőik (KODMAGYARAZAT)

**User modell** – Felhasználók adatai, jogosultságok
```php
protected $fillable = [
    'name',        // Név
    'email',       // Email cím
    'password',    // Jelszó (hash-elve)
    'is_admin',    // Admin jogosultság (true/false)
];
```

**Product modell** – Termékek adatai
```php
protected $fillable = [
    'name',        // Termék neve
    'category',    // Kategória
    'price',       // Ár (2 tizedes)
    'stock',       // Készlet (db)
];
```

**Wishlist modell** – Kívánságlista elemek
```php
protected $fillable = [
    'user_id',     // Felhasználó azonosító
    'product_id',  // Termék azonosító
    'added_at',    // Hozzáadás dátuma
];
```

---

## Modellek közötti kapcsolatok (KODMAGYARAZAT)
- Egy felhasználónak több kívánságlistája lehet.
- Egy terméket több felhasználó is kívánhat.
- A kívánságlista összeköti a felhasználót és a terméket.

**Kapcsolatok példák:**
```php
// User.php
public function wishlists() { return $this->hasMany(Wishlist::class); }

// Product.php
public function wishlists() { return $this->hasMany(Wishlist::class); }
public function wishlistedByUsers() { return $this->belongsToMany(User::class, 'wishlists')->withTimestamps()->withPivot('added_at'); }

// Wishlist.php
public function user() { return $this->belongsTo(User::class); }
public function product() { return $this->belongsTo(Product::class); }
```

---

## Factory-k (KODMAGYARAZAT)

**UserFactory.php** – Felhasználó generálás
```php
'name' => fake()->name(),
'email' => fake()->unique()->safeEmail(),
'password' => Hash::make('password'),
'is_admin' => false,
'email_verified_at' => now(),
'remember_token' => Str::random(10),
```

**ProductFactory.php** – Termék generálás
```php
'name' => $this->faker->word . ' ' . $this->faker->randomNumber(2),
'category' => $this->faker->randomElement(['Electronics', 'Audio', 'Wearables', 'Tablets', 'Accessories', 'Storage']),
'price' => $this->faker->randomFloat(2, 10, 2000),
'stock' => $this->faker->numberBetween(1, 100),
```

**WishlistFactory.php** – Kívánságlista generálás
```php
'user_id' => User::factory(),
'product_id' => Product::factory(),
'added_at' => $this->faker->dateTimeBetween('-10 days', 'now'),
```

---

A fenti részletes struktúra segít átlátni, hogy a projektben melyik mappában milyen típusú fájlok, modellek, kapcsolatok és tesztadat generátorok találhatók, és azok pontosan milyen mezőket, kapcsolatokat tartalmaznak.

# Laravel Wishlist API – Teljes Dokumentáció és Kód

Ez a dokumentáció egyesíti a projekt összes korábbi leírását, a legjobb bevezetőket, a teljes fejlesztési sorrendet, és minden kódrészletet, hogy nulláról felépíthesd a Laravel Wishlist API-t. Minden lépéshez tartozik magyarázat és a teljes kód, semmit nem hagyunk ki.

---

## Bevezető (összevont, legjobb részek)

Ez egy Laravel alapú backend API, amivel felhasználók termékeket adhatnak hozzá a saját kívánságlistájukhoz. Az admin felhasználók kezelhetik a termékeket és felhasználókat, míg a sima felhasználók csak a saját kívánságlistájukat láthatják és szerkeszthetik.

- **Laravel 11/12**
- **Sanctum**: API tokenes autentikáció
- **MySQL** adatbázis
- Fő modellek: User, Product, Wishlist
- Fő végpontok: Regisztráció, Bejelentkezés, Termék CRUD, Kívánságlista CRUD, Admin CRUD

---

## Fejlesztési sorrend, minden lépés és kód

### 1. Környezet előkészítése
- Indítsd el a XAMPP-ot (Apache + MySQL)
- Ellenőrizd, hogy a `htdocs` mappában dolgozol

### 2. Laravel projekt létrehozása
```powershell
cd C:\xampp\htdocs
composer create-project laravel/laravel --prefer-dist wishlists
```

### 3. API fejlesztés előkészítése
```powershell
cd wishlists
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
npm install
```

### 4. Adatbázis létrehozása
- phpMyAdmin-ban hozz létre egy új adatbázist: `wishlists`
- Kódolás: `utf8mb4_unicode_ci`

### 5. Adatbázis kapcsolat beállítása
- `.env` fájlban:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=wishlists
DB_USERNAME=root
DB_PASSWORD=
```

### 6. Modellek és migrációk létrehozása
```powershell
php artisan make:model Product -m
php artisan make:model Wishlist -m
```

### 7. Migrációk kitöltése
- Szerkeszd a migrációs fájlokat a `database/migrations` mappában:

#### database/migrations/0001_01_01_000000_create_users_table.php
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->boolean('is_admin')->default(false);
            $table->rememberToken();
            $table->timestamps();
        });

        Schema::create('password_reset_tokens', function (Blueprint $table) {
            $table->string('email')->primary();
            $table->string('token');
            $table->timestamp('created_at')->nullable();
        });

        Schema::create('sessions', function (Blueprint $table) {
            $table->string('id')->primary();
            $table->foreignId('user_id')->nullable()->index();
            $table->string('ip_address', 45)->nullable();
            $table->text('user_agent')->nullable();
            $table->longText('payload');
            $table->integer('last_activity')->index();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
        Schema::dropIfExists('password_reset_tokens');
        Schema::dropIfExists('sessions');
    }
};
```

#### database/migrations/2025_12_01_081600_create_products_table.php
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('category');
            $table->decimal('price', 10, 2);
            $table->integer('stock');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

#### database/migrations/2025_12_01_081707_create_wishlists_table.php
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('wishlists', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->foreignId('product_id')->constrained()->cascadeOnDelete();
            $table->timestamp('added_at')->useCurrent();
            $table->timestamps();
            $table->unique(['user_id', 'product_id']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('wishlists');
    }
};
```

### 8. Migrációk futtatása
```powershell
php artisan migrate
```

### 9. Modellek kitöltése

#### app/Models/Product.php
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'category',
        'price',
        'stock',
    ];

    protected $casts = [
        'price' => 'decimal:2',
        'stock' => 'integer',
    ];

    public function wishlists()
    {
        return $this->hasMany(Wishlist::class);
    }

    public function wishlistedByUsers()
    {
        return $this->belongsToMany(User::class, 'wishlists')
                    ->withTimestamps()
                    ->withPivot('added_at');
    }
}
```

#### app/Models/Wishlist.php
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Wishlist extends Model
{
    use HasFactory;

    protected $fillable = [
        'user_id',
        'product_id',
        'added_at',
    ];

    protected $casts = [
        'added_at' => 'datetime',
    ];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function product()
    {
        return $this->belongsTo(Product::class);
    }
}
```

#### app/Models/User.php
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasFactory, Notifiable, HasApiTokens;

    protected $fillable = [
        'name',
        'email',
        'password',
        'is_admin',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
            'is_admin' => 'boolean',
        ];
    }
}
```

### 10. Seederek létrehozása és kitöltése
```powershell
php artisan make:seeder UserSeeder
php artisan make:seeder ProductSeeder
php artisan make:seeder WishlistSeeder
```

#### database/seeders/UserSeeder.php
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::create([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
            'password' => Hash::make('password'),
            'is_admin' => true,
        ]);
        User::create([
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => Hash::make('password'),
            'is_admin' => false,
        ]);
        User::create([
            'name' => 'Jane Smith',
            'email' => 'jane@example.com',
            'password' => Hash::make('password'),
            'is_admin' => false,
        ]);
    }
}
```

#### database/seeders/ProductSeeder.php
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Product;

class ProductSeeder extends Seeder
{
    public function run(): void
    {
        $products = [
            ['name' => 'Laptop Dell XPS 15', 'category' => 'Electronics', 'price' => 1299.99, 'stock' => 15],
            ['name' => 'iPhone 15 Pro', 'category' => 'Electronics', 'price' => 999.99, 'stock' => 25],
            ['name' => 'Samsung Galaxy S24', 'category' => 'Electronics', 'price' => 899.99, 'stock' => 30],
            ['name' => 'Sony WH-1000XM5', 'category' => 'Audio', 'price' => 349.99, 'stock' => 50],
            ['name' => 'Apple Watch Series 9', 'category' => 'Wearables', 'price' => 399.99, 'stock' => 40],
            ['name' => 'iPad Air', 'category' => 'Tablets', 'price' => 599.99, 'stock' => 20],
            ['name' => 'Mechanical Keyboard', 'category' => 'Accessories', 'price' => 129.99, 'stock' => 60],
            ['name' => 'Gaming Mouse Logitech', 'category' => 'Accessories', 'price' => 79.99, 'stock' => 100],
            ['name' => '4K Monitor 27"', 'category' => 'Electronics', 'price' => 449.99, 'stock' => 12],
            ['name' => 'External SSD 1TB', 'category' => 'Storage', 'price' => 99.99, 'stock' => 75],
        ];
        foreach ($products as $product) {
            Product::create($product);
        }
    }
}
```

```php
<?php

use App\Models\Wishlist;
use App\Models\User;





class WishlistSeeder extends Seeder

{
KODMAGYARAZAT: Ez a teszt ellenőrzi a regisztráció, bejelentkezés és kijelentkezés működését.
    public function run(): void
    {
        $john = User::where('email', 'john@example.com')->first();
        if ($john) {
            Wishlist::create(['user_id' => $john->id, 'product_id' => 1, 'added_at' => now()->subDays(5)]);
            Wishlist::create(['user_id' => $john->id, 'product_id' => 2, 'added_at' => now()->subDays(3)]);
            Wishlist::create(['user_id' => $john->id, 'product_id' => 4, 'added_at' => now()->subDays(1)]);
            Wishlist::create(['user_id' => $john->id, 'product_id' => 7, 'added_at' => now()]);
        }
        $jane = User::where('email', 'jane@example.com')->first();
        if ($jane) {
            Wishlist::create(['user_id' => $jane->id, 'product_id' => 5, 'added_at' => now()->subDays(7)]);
            Wishlist::create(['user_id' => $jane->id, 'product_id' => 6, 'added_at' => now()->subDays(4)]);
            Wishlist::create(['user_id' => $jane->id, 'product_id' => 9, 'added_at' => now()->subDays(2)]);
            Wishlist::create(['user_id' => $jane->id, 'product_id' => 3, 'added_at' => now()->subHours(12)]);
            Wishlist::create(['user_id' => $jane->id, 'product_id' => 10, 'added_at' => now()->subHours(3)]);
        }
    }
}
```

#### database/seeders/DatabaseSeeder.php
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents;

    public function run(): void
    {
        $this->call([
            UserSeeder::class,
            ProductSeeder::class,
            WishlistSeeder::class,
        ]);
    }
}
```

### 11. Seederek futtatása
```powershell
php artisan db:seed
```

### 12. Middleware létrehozása admin jogosultsághoz

```powershell
KODMAGYARAZAT: Ez a teszt ellenőrzi a termékek listázását, megtekintését, admin termék CRUD műveleteit.
php artisan make:middleware IsAdmin
```

#### app/Http/Middleware/IsAdmin.php
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class IsAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        if (!$request->user() || !$request->user()->is_admin) {
            return response()->json([
                'message' => 'Unauthorized. Admin access required.'
            ], 403);
        }
        return $next($request);
    }
}
```

### 13. Kontrollerek létrehozása
```powershell
php artisan make:controller Api/AuthController
php artisan make:controller Api/ProductController
php artisan make:controller Api/WishlistController
php artisan make:controller Api/UserController
```

#### app/Http/Controllers/Api/AuthController.php
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed',
            'is_admin' => 'sometimes|boolean',
        ]);


KODMAGYARAZAT: Ez a teszt ellenőrzi a kívánságlista lekérdezését, hozzáadását, törlését, admin összes kívánságlista lekérdezését.
        if ($validator->fails()) {
            return response()->json([
                'message' => 'Validation error',
                'errors' => $validator->errors()
            ], 422);
        }

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
            'is_admin' => $request->input('is_admin', false),
        ]);

        return response()->json([
            'message' => 'User registered successfully. Please login.',
            'user' => [
                'id' => $user->id,
                'name' => $user->name,
                'email' => $user->email,
                'is_admin' => $user->is_admin,
            ],
        ], 201);
    }

    public function login(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'email' => 'required|email',
            'password' => 'required',
        ]);

        if ($validator->fails()) {
            return response()->json([
                'message' => 'Validation error',
                'errors' => $validator->errors()
            ], 422);
        }

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            return response()->json([
                'message' => 'Invalid credentials'
            ], 401);
        }

        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json([
            'message' => 'Login successful',
            'user' => $user,
            'access_token' => $token,
            'token_type' => 'Bearer',
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Logged out successfully'
        ]);
    }

    public function me(Request $request)
    {
        return response()->json($request->user());
    }

}
KODMAGYARAZAT: Ez a teszt ellenőrzi az admin felhasználó CRUD műveleteit (listázás, megtekintés, módosítás, törlés).
```

#### app/Http/Controllers/Api/ProductController.php
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class ProductController extends Controller
{
    public function index(Request $request)
    {
        $products = Product::all();
        return response()->json($products);
    }

    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|string|max:255',
            'category' => 'required|string|max:255',
            'price' => 'required|numeric|min:0',
            'stock' => 'required|integer|min:0',
        ]);

        if ($validator->fails()) {
            return response()->json([
                'message' => 'Validation error',
                'errors' => $validator->errors()
            ], 422);
        }

        $product = Product::create($request->all());

        return response()->json([
            'message' => 'Product created successfully',
            'product' => $product
        ], 201);
    }

    public function show($id)
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'message' => 'Product not found'
            ], 404);
        }

        return response()->json($product);
    }

    public function update(Request $request, $id)
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'message' => 'Product not found'
            ], 404);
        }

        $validator = Validator::make($request->all(), [
            'name' => 'sometimes|string|max:255',
            'category' => 'sometimes|string|max:255',
            'price' => 'sometimes|numeric|min:0',
            'stock' => 'sometimes|integer|min:0',
        ]);

        if ($validator->fails()) {
            return response()->json([
                'message' => 'Validation error',
                'errors' => $validator->errors()
            ], 422);
        }

        $product->update($request->all());

        return response()->json([
            'message' => 'Product updated successfully',
            'product' => $product
        ]);
    }

    public function destroy($id)
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'message' => 'Product not found'
            ], 404);
        }

        $product->delete();

        return response()->json([
            'message' => 'Product deleted successfully'
        ]);
    }
}
```

#### app/Http/Controllers/Api/WishlistController.php
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Wishlist;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class WishlistController extends Controller
{
    public function index(Request $request)
    {
        $user = $request->user();
        $wishlists = Wishlist::where('user_id', $user->id)
            ->with('product')
            ->get();
        return response()->json($wishlists);
    }

    public function indexAll()
    {
        $wishlists = Wishlist::with(['user', 'product'])->get();
        return response()->json($wishlists);
    }

    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'product_id' => 'required|exists:products,id',
        ]);

        if ($validator->fails()) {
            return response()->json([
                'message' => 'Validation error',
                'errors' => $validator->errors()
            ], 422);
        }

        $user = $request->user();
        $exists = Wishlist::where('user_id', $user->id)
            ->where('product_id', $request->product_id)
            ->exists();
        if ($exists) {
            return response()->json([
                'message' => 'Product already in wishlist'
            ], 409);
        }
        $wishlist = Wishlist::create([
            'user_id' => $user->id,
            'product_id' => $request->product_id,
            'added_at' => now(),
        ]);
        $wishlist->load('product');
        return response()->json([
            'message' => 'Product added to wishlist',
            'wishlist' => $wishlist
        ], 201);
    }

    public function show(Request $request, $id)
    {
        $user = $request->user();
        $wishlist = Wishlist::with('product')->find($id);
        if (!$wishlist) {
            return response()->json([
                'message' => 'Wishlist item not found'
            ], 404);
        }
        if ($wishlist->user_id !== $user->id && !$user->is_admin) {
            return response()->json([
                'message' => 'Unauthorized'
            ], 403);
        }
        return response()->json($wishlist);
    }

    public function destroy(Request $request, $id)
    {
        $user = $request->user();
        $wishlist = Wishlist::find($id);
        if (!$wishlist) {
            return response()->json([
                'message' => 'Wishlist item not found'
            ], 404);
        }
        if ($wishlist->user_id !== $user->id && !$user->is_admin) {
            return response()->json([
                'message' => 'Unauthorized'
            ], 403);
        }
        $wishlist->delete();
        return response()->json([
            'message' => 'Product removed from wishlist'
        ]);
    }

    public function getUserWishlist($userId)
    {
        $wishlists = Wishlist::where('user_id', $userId)
            ->with('product')
            ->get();
        return response()->json($wishlists);
    }
}
```

#### app/Http/Controllers/Api/UserController.php
```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;

class UserController extends Controller
{
    public function index()
    {
        $users = User::all();
        return response()->json($users);
    }

    public function show($id)
    {
        $user = User::find($id);
        if (!$user) {
            return response()->json([
                'message' => 'User not found'
            ], 404);
        }
        return response()->json($user);
    }

    public function update(Request $request, $id)
    {
        $user = User::find($id);
        if (!$user) {
            return response()->json([
                'message' => 'User not found'
            ], 404);
        }
        $validator = Validator::make($request->all(), [
            'name' => 'sometimes|string|max:255',
            'email' => 'sometimes|string|email|max:255|unique:users,email,' . $id,
            'password' => 'sometimes|string|min:8',
            'is_admin' => 'sometimes|boolean',
        ]);
        if ($validator->fails()) {
            return response()->json([
                'message' => 'Validation error',
                'errors' => $validator->errors()
            ], 422);
        }
        $data = $request->only(['name', 'email', 'is_admin']);
        if ($request->has('password')) {
            $data['password'] = Hash::make($request->password);
        }
        $user->update($data);
        return response()->json([
            'message' => 'User updated successfully',
            'user' => $user
        ]);
    }

    public function destroy($id)
    {
        $user = User::find($id);
        if (!$user) {
            return response()->json([
                'message' => 'User not found'
            ], 404);
        }
        $user->delete();
        return response()->json([
            'message' => 'User deleted successfully'
        ]);
    }
}
```

### 14. Route-ok beállítása

#### routes/api.php
```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\AuthController;
use App\Http\Controllers\Api\ProductController;
use App\Http\Controllers\Api\WishlistController;
use App\Http\Controllers\Api\UserController;

// Public routes
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

// Public product routes (anyone can view products)
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/{id}', [ProductController::class, 'show']);

// Protected routes (requires authentication)
Route::middleware('auth:sanctum')->group(function () {
    // Auth routes
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::get('/me', [AuthController::class, 'me']);

    // Wishlist routes for authenticated users
    Route::get('/wishlists', [WishlistController::class, 'index']);
    Route::post('/wishlists', [WishlistController::class, 'store']);
    Route::get('/wishlists/{id}', [WishlistController::class, 'show']);
    Route::delete('/wishlists/{id}', [WishlistController::class, 'destroy']);

    // Admin-only routes
    Route::middleware('admin')->group(function () {
        // Product management
        Route::post('/products', [ProductController::class, 'store']);
        Route::put('/products/{id}', [ProductController::class, 'update']);
        Route::delete('/products/{id}', [ProductController::class, 'destroy']);

        // User management
        Route::get('/users', [UserController::class, 'index']);
        Route::get('/users/{id}', [UserController::class, 'show']);
        Route::put('/users/{id}', [UserController::class, 'update']);
        Route::delete('/users/{id}', [UserController::class, 'destroy']);

        // All wishlists (admin view)
        Route::get('/admin/wishlists', [WishlistController::class, 'indexAll']);
        Route::get('/admin/users/{userId}/wishlists', [WishlistController::class, 'getUserWishlist']);
    });
});
```

---

# API Feature Tesztek

Az alábbiakban megtalálod az összes létrehozott tesztfájlt és azok teljes kódját. Minden teszt a `tests/Feature` mappában található.

## AuthApiTest.php
**Fájl:** `tests/Feature/AuthApiTest.php`
```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
use App\Models\User;
use App\Models\Product;
use App\Models\Wishlist;

class AuthApiTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function user_can_register()
    {
        $response = $this->postJson('/api/register', [
            'name' => 'Teszt User',
            'email' => 'teszt@example.com',
            'password' => 'password',
            'password_confirmation' => 'password',
        ]);
        $response->assertStatus(201)
                 ->assertJsonStructure(['message', 'user' => ['id', 'name', 'email', 'is_admin']]);
    }

    /** @test */
    public function user_can_login()
    {
        $user = User::factory()->create([
            'email' => 'teszt@example.com',
            'password' => bcrypt('password'),
        ]);
        $response = $this->postJson('/api/login', [
            'email' => 'teszt@example.com',
            'password' => 'password',
        ]);
        $response->assertStatus(200)
                 ->assertJsonStructure(['message', 'user', 'access_token', 'token_type']);
    }

    /** @test */
    public function user_can_logout()
    {
        $user = User::factory()->create();
        $token = $user->createToken('auth_token')->plainTextToken;
        $response = $this->withHeader('Authorization', 'Bearer ' . $token)
                         ->postJson('/api/logout');
        $response->assertStatus(200)
                 ->assertJson(['message' => 'Logged out successfully']);
    }
}
```

## ProductApiTest.php
**Fájl:** `tests/Feature/ProductApiTest.php`
```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
use App\Models\User;
use App\Models\Product;

class ProductApiTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function anyone_can_list_products()
    {
        Product::factory()->count(3)->create();
        $response = $this->getJson('/api/products');
        $response->assertStatus(200)
                 ->assertJsonCount(3);
    }

    /** @test */
    public function anyone_can_view_a_product()
    {
        $product = Product::factory()->create();
        $response = $this->getJson('/api/products/' . $product->id);
        $response->assertStatus(200)
                 ->assertJson(['id' => $product->id]);
    }

    /** @test */
    public function admin_can_create_update_delete_product()
    {
        $admin = User::factory()->create(['is_admin' => true]);
        $token = $admin->createToken('auth_token')->plainTextToken;
        $data = [
            'name' => 'Teszt Termék',
            'category' => 'Teszt',
            'price' => 123.45,
            'stock' => 10,
        ];
        $create = $this->withHeader('Authorization', 'Bearer ' . $token)
                      ->postJson('/api/products', $data);
        $create->assertStatus(201)
               ->assertJsonStructure(['message', 'product']);
        $productId = $create->json('product.id');
        $update = $this->withHeader('Authorization', 'Bearer ' . $token)
                      ->putJson('/api/products/' . $productId, ['stock' => 99]);
        $update->assertStatus(200)
               ->assertJson(['product' => ['stock' => 99]]);
        $delete = $this->withHeader('Authorization', 'Bearer ' . $token)
                      ->deleteJson('/api/products/' . $productId);
        $delete->assertStatus(200)
               ->assertJson(['message' => 'Product deleted successfully']);
    }
}
```

## WishlistApiTest.php
**Fájl:** `tests/Feature/WishlistApiTest.php`
```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
use App\Models\User;
use App\Models\Product;
use App\Models\Wishlist;

class WishlistApiTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function user_can_list_own_wishlist()
    {
        $user = User::factory()->create();
        $token = $user->createToken('auth_token')->plainTextToken;
        $product = Product::factory()->create();
        Wishlist::create([
            'user_id' => $user->id,
            'product_id' => $product->id,
            'added_at' => now(),
        ]);
        $response = $this->withHeader('Authorization', 'Bearer ' . $token)
                         ->getJson('/api/wishlists');
        $response->assertStatus(200)
                 ->assertJsonFragment(['product_id' => $product->id]);
    }

    /** @test */
    public function user_can_add_and_remove_product_from_wishlist()
    {
        $user = User::factory()->create();
        $token = $user->createToken('auth_token')->plainTextToken;
        $product = Product::factory()->create();
        $add = $this->withHeader('Authorization', 'Bearer ' . $token)
                     ->postJson('/api/wishlists', ['product_id' => $product->id]);
        $add->assertStatus(201)
            ->assertJsonStructure(['message', 'wishlist']);
        $wishlistId = $add->json('wishlist.id');
        $remove = $this->withHeader('Authorization', 'Bearer ' . $token)
                       ->deleteJson('/api/wishlists/' . $wishlistId);
        $remove->assertStatus(200)
               ->assertJson(['message' => 'Product removed from wishlist']);
    }

    /** @test */
    public function admin_can_list_all_wishlists_and_user_wishlist()
    {
        $admin = User::factory()->create(['is_admin' => true]);
        $token = $admin->createToken('auth_token')->plainTextToken;
        $user = User::factory()->create();
        $product = Product::factory()->create();
        Wishlist::create([
            'user_id' => $user->id,
            'product_id' => $product->id,
            'added_at' => now(),
        ]);
        $all = $this->withHeader('Authorization', 'Bearer ' . $token)
                    ->getJson('/api/admin/wishlists');
        $all->assertStatus(200)
            ->assertJsonFragment(['user_id' => $user->id]);
        $userList = $this->withHeader('Authorization', 'Bearer ' . $token)
                        ->getJson('/api/admin/users/' . $user->id . '/wishlists');
        $userList->assertStatus(200)
                 ->assertJsonFragment(['product_id' => $product->id]);
    }
}
```

## UserApiTest.php
**Fájl:** `tests/Feature/UserApiTest.php`
```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
use App\Models\User;

class UserApiTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function admin_can_list_and_manage_users()
    {
        $admin = User::factory()->create(['is_admin' => true]);
        $token = $admin->createToken('auth_token')->plainTextToken;
        $user = User::factory()->create();
        $list = $this->withHeader('Authorization', 'Bearer ' . $token)
                     ->getJson('/api/users');
        $list->assertStatus(200)
             ->assertJsonFragment(['id' => $user->id]);
        $show = $this->withHeader('Authorization', 'Bearer ' . $token)
                     ->getJson('/api/users/' . $user->id);
        $show->assertStatus(200)
             ->assertJson(['id' => $user->id]);
        $update = $this->withHeader('Authorization', 'Bearer ' . $token)
                       ->putJson('/api/users/' . $user->id, ['name' => 'Updated']);
        $update->assertStatus(200)
               ->assertJson(['user' => ['name' => 'Updated']]);
        $delete = $this->withHeader('Authorization', 'Bearer ' . $token)
                       ->deleteJson('/api/users/' . $user->id);
        $delete->assertStatus(200)
               ->assertJson(['message' => 'User deleted successfully']);
    }
}
```

---

## Factory-k teljes kódja (KODMAGYARAZAT)

**UserFactory.php** – Felhasználó generálás, admin állapot, email verifikáció
```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

/**
 * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\User>
 */
class UserFactory extends Factory
{
    /**
     * The current password being used by the factory.
     */
    protected static ?string $password;

    /**
     * Define the model's default state.
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'is_admin' => false,
            'remember_token' => Str::random(10),
        ];
    }

    /**
     * Indicate that the user is admin.
     */
    public function admin(): static
    {
        return $this->state(fn (array $attributes) => [
            'is_admin' => true,
        ]);
    }

    /**
     * Indicate that the model's email address should be unverified.
     */
    public function unverified(): static
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }
}
```

**ProductFactory.php** – Termék generálás, kategóriák, ár, készlet
```php
<?php

namespace Database\Factories;

use App\Models\Product;
use Illuminate\Database\Eloquent\Factories\Factory;

class ProductFactory extends Factory
{
    protected $model = Product::class;

    public function definition()
    {
        return [
            'name' => $this->faker->word . ' ' . $this->faker->randomNumber(2),
            'category' => $this->faker->randomElement(['Electronics', 'Audio', 'Wearables', 'Tablets', 'Accessories', 'Storage']),
            'price' => $this->faker->randomFloat(2, 10, 2000),
            'stock' => $this->faker->numberBetween(1, 100),
        ];
    }
}
```

**WishlistFactory.php** – Kívánságlista generálás, random felhasználó és termék
```php
<?php

namespace Database\Factories;

use App\Models\Wishlist;
use App\Models\User;
use App\Models\Product;
use Illuminate\Database\Eloquent\Factories\Factory;

class WishlistFactory extends Factory
{
    protected $model = Wishlist::class;

    public function definition()
    {
        return [
            'user_id' => User::factory(),
            'product_id' => Product::factory(),
            'added_at' => $this->faker->dateTimeBetween('-10 days', 'now'),
        ];
    }
}
```
