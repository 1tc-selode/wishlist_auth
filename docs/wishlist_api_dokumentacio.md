# Wishlist REST API – Teljes Dokumentáció és Kód

Ez a dokumentáció a Laravel alapú wishlist (kívánságlista) backend API teljes fejlesztési útmutatója. A projekt célja egy olyan REST API létrehozása, amely lehetővé teszi a felhasználók számára termékek hozzáadását a saját kívánságlistájukhoz, termékek kezelését, és egy teljes körű admin felületet biztosít.

---

## Projekt Áttekintés

**Base URL-ek:**
- XAMPP: `http://127.0.0.1/wishlist_auth/public/api`
- Laravel serve: `http://127.0.0.1:8000/api`

**Technológiák:**
- Laravel 12
- Laravel Sanctum (API token hitelesítés)
- MySQL adatbázis
- PHPUnit tesztelés

**Adatbázis neve:** `wishlists`

---

## Projekt Struktúra

```
wishlist_auth/
├── app/
│   ├── Http/
│   │   ├── Controllers/         # API vezérlők
│   │   │   ├── Controller.php   # Alap vezérlő
│   │   │   └── Api/             # API vezérlők mappája
│   │   │       ├── AuthController.php      # Autentikáció
│   │   │       ├── ProductController.php   # Termékek kezelése
│   │   │       ├── UserController.php      # Felhasználó kezelés
│   │   │       └── WishlistController.php  # Kívánságlista kezelés
│   │   └── Middleware/          # Middleware-ek
│   ├── Models/                  # Adatmodellek
│   │   ├── User.php             # Felhasználó modell
│   │   ├── Product.php          # Termék modell
│   │   └── Wishlist.php         # Kívánságlista modell
│   └── Providers/               # Laravel szolgáltatók
├── database/
│   ├── factories/               # Tesztadat generátorok
│   │   ├── UserFactory.php
│   │   ├── ProductFactory.php
│   │   └── WishlistFactory.php
│   ├── migrations/              # Adatbázis szerkezet
│   │   ├── 0001_01_01_000000_create_users_table.php
│   │   ├── 0001_01_01_000001_create_cache_table.php
│   │   ├── 0001_01_01_000002_create_jobs_table.php
│   │   ├── 2025_12_01_081529_create_personal_access_tokens_table.php
│   │   ├── 2025_12_01_081600_create_products_table.php
│   │   └── 2025_12_01_081707_create_wishlists_table.php
│   └── seeders/                 # Tesztadat feltöltés
│       ├── DatabaseSeeder.php
│       ├── UserSeeder.php
│       ├── ProductSeeder.php
│       └── WishlistSeeder.php
├── routes/
│   └── api.php                  # API útvonalak
├── tests/
│   └── Feature/                 # API funkció tesztek
│       ├── AuthApiTest.php
│       ├── ProductApiTest.php
│       ├── UserApiTest.php
│       └── WishlistApiTest.php
├── docs/                        # Dokumentáció
│   ├── API_TEST_ENDPOINTS.md
│   ├── Wishlist_API.postman_collection.json
│   └── wishlist_dokumentacio.md
├── test_api.ps1                 # PowerShell teszt script
├── composer.json                # Composer konfiguráció
├── package.json                 # NPM konfiguráció
├── phpunit.xml                  # PHPUnit teszt konfiguráció
└── vite.config.js               # Vite frontend build
```

---

## Adatbázis Terv

```
+---------------------+     +---------------------+       +-----------------+        +------------+
|personal_access_tokens|    |        users        |       |   wishlists     |        |  products  |
+---------------------+     +---------------------+       +-----------------+        +------------+
| id (PK)             |   _1| id (PK)             |1__    | id (PK)         |     __1| id (PK)    |
| tokenable_id (FK)   |K_/  | name                |   \__N| user_id (FK)    |    /   | name       |
| tokenable_type      |     | email (unique)      |       | product_id (FK) |M__/    | category   |
| name                |     | password            |       | added_at        |        | price      |
| token (unique)      |     | is_admin (boolean)  |       | created_at      |        | stock      |
| abilities           |     | email_verified_at   |       | updated_at      |        | created_at |
| last_used_at        |     | remember_token      |       +-----------------+        | updated_at |
| created_at          |     | created_at          |                                  +------------+
| updated_at          |     | updated_at          |
+---------------------+     +---------------------+
```

### Kapcsolatok:
- **User -> Wishlist:** 1:N (egy felhasználónak több kívánságlistája lehet)
- **Product -> Wishlist:** 1:N (egy terméket több felhasználó is kívánhat)
- **User -> Product:** N:M (many-to-many kapcsolat a wishlists táblán keresztül)
- **Egyedi megszorítás:** user_id + product_id kombinációja egyedi (egy user nem adhatja hozzá kétszer ugyanazt a terméket)

---

## I. Modul - Telepítés és Konfiguráció

### 1. Laravel Projekt Létrehozása

```powershell
# Projekt létrehozása
composer create-project laravel/laravel --prefer-dist wishlist_auth

# Könyvtár váltás
cd wishlist_auth
```

### 2. .env Fájl Konfiguráció

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=wishlists
DB_USERNAME=root
DB_PASSWORD=

APP_TIMEZONE=Europe/Budapest
```

### 3. Sanctum Telepítése

```powershell
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan install:api
```

### 4. Admin Middleware Létrehozása

```powershell
php artisan make:middleware IsAdmin
```

### 5. Első Teszt (tesztútvonalak már léteznek)

```powershell
# Laravel szerver indítása
php artisan serve

# POSTMAN teszt
# GET http://127.0.0.1:8000/api/products
```

---

## II. Modul - Adatbázis és Modellek

### 1. Modellek és Migrációk (Már léteznek a projektben)

A projektben már léteznek a szükséges modellek és migrációk:

```powershell
# Ezek a modellek már léteznek:
# - User.php (felhasználók)
# - Product.php (termékek) 
# - Wishlist.php (kívánságlista)
```

### 2. Létező Migrációk

#### users tábla (0001_01_01_000000_create_users_table.php)

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

#### products tábla (2025_12_01_081600_create_products_table.php)

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

#### wishlists tábla (2025_12_01_081707_create_wishlists_table.php)

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
            
            // Unique constraint: egy user csak egyszer kedvelhet egy terméket
            $table->unique(['user_id', 'product_id']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('wishlists');
    }
};
```

### 3. Létező Modell Fájlok

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

    /**
     * Get the wishlists for the user.
     */
    public function wishlists()
    {
        return $this->hasMany(Wishlist::class);
    }

    /**
     * Get the products the user has wishlisted.
     */
    public function wishlistedProducts()
    {
        return $this->belongsToMany(Product::class, 'wishlists')
                    ->withTimestamps()
                    ->withPivot('added_at');
    }
}
```

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

    /**
     * Get the wishlists for the product.
     */
    public function wishlists()
    {
        return $this->hasMany(Wishlist::class);
    }

    /**
     * Get the users who have wishlisted this product.
     */
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

    /**
     * Get the user that owns the wishlist.
     */
    public function user()
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the product that is wishlisted.
     */
    public function product()
    {
        return $this->belongsTo(Product::class);
    }
}
```

### 4. Migráció Futtatása

```powershell
php artisan migrate
```

---

## III. Modul - Seeding (Tesztadatok)

A projektben már léteznek a factory-k és seederek. A tesztadatok feltöltése:

### 1. Létező Seederek

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
        // Admin user
        User::create([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
            'password' => Hash::make('password'),
            'is_admin' => true,
        ]);

        // Normal users
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

        // További felhasználók factory-val
        User::factory(7)->create();
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
            [
                'name' => 'Laptop Dell XPS 15',
                'category' => 'Electronics',
                'price' => 1299.99,
                'stock' => 15,
            ],
            [
                'name' => 'iPhone 15 Pro',
                'category' => 'Electronics',
                'price' => 999.99,
                'stock' => 25,
            ],
            [
                'name' => 'Samsung Galaxy S24',
                'category' => 'Electronics',
                'price' => 899.99,
                'stock' => 30,
            ],
            [
                'name' => 'Sony WH-1000XM5',
                'category' => 'Audio',
                'price' => 349.99,
                'stock' => 50,
            ],
            // ... több termék
        ];

        foreach ($products as $product) {
            Product::create($product);
        }

        // További termékek factory-val
        Product::factory(20)->create();
    }
}
```

#### database/seeders/WishlistSeeder.php

A seeder automatikusan létrehoz kapcsolatokat a felhasználók és termékek között.

#### database/seeders/DatabaseSeeder.php

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
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

### 2. Seeding Futtatása

```powershell
php artisan db:seed
```

### 3. Factory-k (Léteznek)

A projektben már léteznek a factory-k a tesztadatok generálásához:
- `UserFactory.php` - Felhasználók generálása
- `ProductFactory.php` - Termékek generálása  
- `WishlistFactory.php` - Kívánságlista bejegyzések generálása

---

## IV. Modul - Controller-ek és API Végpontok

A projektben már léteznek az API controller-ek az `app/Http/Controllers/Api/` mappában:

### 1. Létező Controller-ek

- `AuthController.php` - Autentikáció (regisztráció, bejelentkezés, kijelentkezés)
- `ProductController.php` - Termékek kezelése
- `UserController.php` - Felhasználók kezelése (admin funkciók)
- `WishlistController.php` - Kívánságlista kezelése

### 2. AuthController Implementálása

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
    /**
     * Register a new user.
     */
    public function register(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed',
            'is_admin' => 'sometimes|boolean',
        ]);

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

    /**
     * Login user and create token.
     */
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

    /**
     * Logout user (revoke token).
     */
    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Logged out successfully'
        ]);
    }

    /**
     * Get authenticated user.
     */
    public function me(Request $request)
    {
        return response()->json($request->user());
    }
}
```

### 3. ProductController Implementálása

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
    /**
     * Display a listing of the products (public).
     */
    public function index(Request $request)
    {
        $products = Product::all();
        return response()->json($products);
    }

    /**
     * Store a newly created product (admin only).
     */
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

    /**
     * Display the specified product (public).
     */
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

    /**
     * Update the specified product (admin only).
     */
    public function update(Request $request, $id)
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'message' => 'Product not found'
            ], 404);
        }

        $validator = Validator::make($request->all(), [
            'name' => 'sometimes|required|string|max:255',
            'category' => 'sometimes|required|string|max:255',
            'price' => 'sometimes|required|numeric|min:0',
            'stock' => 'sometimes|required|integer|min:0',
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

    /**
     * Remove the specified product (admin only).
     */
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

### 4. WishlistController Implementálása

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
    /**
     * Display a listing of the user's wishlists.
     */
    public function index(Request $request)
    {
        $user = $request->user();
        $wishlists = Wishlist::where('user_id', $user->id)
            ->with('product')
            ->get();

        return response()->json($wishlists);
    }

    /**
     * Display all wishlists (admin only).
     */
    public function indexAll()
    {
        $wishlists = Wishlist::with(['user', 'product'])->get();
        return response()->json($wishlists);
    }

    /**
     * Add a product to the user's wishlist.
     */
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

        // Check if already in wishlist
        $existingWishlist = Wishlist::where('user_id', $request->user()->id)
            ->where('product_id', $request->product_id)
            ->first();

        if ($existingWishlist) {
            return response()->json([
                'message' => 'Product is already in your wishlist'
            ], 409);
        }

        $wishlist = Wishlist::create([
            'user_id' => $request->user()->id,
            'product_id' => $request->product_id,
            'added_at' => now(),
        ]);

        $wishlist->load('product');

        return response()->json([
            'message' => 'Product added to wishlist successfully',
            'wishlist' => $wishlist
        ], 201);
    }

    /**
     * Display the specified wishlist item.
     */
    public function show(Request $request, $id)
    {
        $wishlist = Wishlist::where('id', $id)
            ->where('user_id', $request->user()->id)
            ->with('product')
            ->first();

        if (!$wishlist) {
            return response()->json([
                'message' => 'Wishlist item not found'
            ], 404);
        }

        return response()->json($wishlist);
    }

    /**
     * Remove the specified wishlist item.
     */
    public function destroy(Request $request, $id)
    {
        $wishlist = Wishlist::where('id', $id)
            ->where('user_id', $request->user()->id)
            ->first();

        if (!$wishlist) {
            return response()->json([
                'message' => 'Wishlist item not found'
            ], 404);
        }

        $wishlist->delete();

        return response()->json([
            'message' => 'Product removed from wishlist successfully'
        ]);
    }

    /**
     * Get wishlist for specific user (admin only).
     */
    public function getUserWishlist($userId)
    {
        $wishlists = Wishlist::where('user_id', $userId)
            ->with(['user', 'product'])
            ->get();

        return response()->json($wishlists);
    }
}
```

### 5. API Routes Konfiguráció

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

## V. API Végpontok Részletes Dokumentációja

### Általános Információk

**Content-Type:** `application/json`
**Accept:** `application/json`

**Hitelesítés:**
```
Authorization: Bearer {token}
```

**Hibakódok:**
- `400 Bad Request` - Hibás formátumú kérés
- `401 Unauthorized` - Érvénytelen token vagy hitelesítés szükséges
- `403 Forbidden` - Nincs jogosultság (admin jogok szükségesek)
- `404 Not Found` - Erőforrás nem található
- `409 Conflict` - Ütköző művelet (pl. termék már a kívánságlistában)
- `422 Unprocessable Entity` - Validációs hiba
- `503 Service Unavailable` - Szolgáltatás nem elérhető

### Publikus végpontok (token nélkül)

#### POST /register
Új felhasználó regisztrálása.

**Kérés törzse:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123",
  "password_confirmation": "password123",
  "is_admin": false
}
```

**Válasz:** 201 Created
```json
{
  "message": "User registered successfully. Please login.",
  "user": {
    "id": 13,
    "name": "John Doe",
    "email": "john@example.com",
    "is_admin": false
  }
}
```

**Hiba:** 422 Unprocessable Entity
```json
{
  "message": "Validation error",
  "errors": {
    "email": [
      "The email has already been taken."
    ]
  }
}
```

#### POST /login
Bejelentkezés e-mail címmel és jelszóval.

**Kérés törzse:**
```json
{
  "email": "john@example.com",
  "password": "password123"
}
```

**Válasz:** 200 OK
```json
{
  "message": "Login successful",
  "user": {
    "id": 13,
    "name": "John Doe",
    "email": "john@example.com",
    "is_admin": false,
    "email_verified_at": null,
    "created_at": "2025-12-06T10:30:00.000000Z",
    "updated_at": "2025-12-06T10:30:00.000000Z"
  },
  "access_token": "2|7Fbr79b5zn8RxMfOqfdzZ31SnGWvgDidjahbdRfL2a98cfd8",
  "token_type": "Bearer"
}
```

**Hiba:** 401 Unauthorized
```json
{
  "message": "Invalid credentials"
}
```

#### GET /products
Összes termék lekérése (publikus, autentikáció nélkül).

**Válasz:** 200 OK
```json
[
  {
    "id": 1,
    "name": "Laptop Dell XPS 15",
    "category": "Electronics",
    "price": "1299.99",
    "stock": 15,
    "created_at": "2025-12-06T10:00:00.000000Z",
    "updated_at": "2025-12-06T10:00:00.000000Z"
  },
  {
    "id": 2,
    "name": "iPhone 15 Pro",
    "category": "Electronics",
    "price": "999.99",
    "stock": 25,
    "created_at": "2025-12-06T10:00:00.000000Z",
    "updated_at": "2025-12-06T10:00:00.000000Z"
  }
]
```

#### GET /products/{id}
Adott termék részleteinek lekérése (publikus).

**Válasz:** 200 OK
```json
{
  "id": 1,
  "name": "Laptop Dell XPS 15",
  "category": "Electronics",
  "price": "1299.99",
  "stock": 15,
  "created_at": "2025-12-06T10:00:00.000000Z",
  "updated_at": "2025-12-06T10:00:00.000000Z"
}
```

**Hiba:** 404 Not Found
```json
{
  "message": "Product not found"
}
```

### Védett végpontok (autentikáció szükséges)

#### POST /logout
Kijelentkezés és aktuális token törlése.

**Válasz:** 200 OK
```json
{
  "message": "Logged out successfully"
}
```

#### GET /me
Aktuális felhasználó adatainak lekérése.

**Válasz:** 200 OK
```json
{
  "id": 13,
  "name": "John Doe",
  "email": "john@example.com",
  "is_admin": false,
  "email_verified_at": null,
  "created_at": "2025-12-06T10:30:00.000000Z",
  "updated_at": "2025-12-06T10:30:00.000000Z"
}
```

### Kívánságlista végpontok (autentikáció szükséges)

#### GET /wishlists
Saját kívánságlista lekérése.

**Válasz:** 200 OK
```json
[
  {
    "id": 1,
    "user_id": 13,
    "product_id": 1,
    "added_at": "2025-12-06T10:30:00.000000Z",
    "created_at": "2025-12-06T10:30:00.000000Z",
    "updated_at": "2025-12-06T10:30:00.000000Z",
    "product": {
      "id": 1,
      "name": "Laptop Dell XPS 15",
      "category": "Electronics",
      "price": "1299.99",
      "stock": 15,
      "created_at": "2025-12-06T10:00:00.000000Z",
      "updated_at": "2025-12-06T10:00:00.000000Z"
    }
  }
]
```

#### POST /wishlists
Termék hozzáadása a kívánságlistához.

**Kérés törzse:**
```json
{
  "product_id": 1
}
```

**Válasz:** 201 Created
```json
{
  "message": "Product added to wishlist successfully",
  "wishlist": {
    "user_id": 13,
    "product_id": 1,
    "added_at": "2025-12-06T10:30:00.000000Z",
    "updated_at": "2025-12-06T10:30:00.000000Z",
    "created_at": "2025-12-06T10:30:00.000000Z",
    "id": 1,
    "product": {
      "id": 1,
      "name": "Laptop Dell XPS 15",
      "category": "Electronics",
      "price": "1299.99",
      "stock": 15,
      "created_at": "2025-12-06T10:00:00.000000Z",
      "updated_at": "2025-12-06T10:00:00.000000Z"
    }
  }
}
```

**Hiba:** 409 Conflict
```json
{
  "message": "Product is already in your wishlist"
}
```

#### GET /wishlists/{id}
Adott kívánságlista elem lekérése.

**Válasz:** 200 OK
```json
{
  "id": 1,
  "user_id": 13,
  "product_id": 1,
  "added_at": "2025-12-06T10:30:00.000000Z",
  "created_at": "2025-12-06T10:30:00.000000Z",
  "updated_at": "2025-12-06T10:30:00.000000Z",
  "product": {
    "id": 1,
    "name": "Laptop Dell XPS 15",
    "category": "Electronics",
    "price": "1299.99",
    "stock": 15,
    "created_at": "2025-12-06T10:00:00.000000Z",
    "updated_at": "2025-12-06T10:00:00.000000Z"
  }
}
```

#### DELETE /wishlists/{id}
Termék eltávolítása a kívánságlistából.

**Válasz:** 200 OK
```json
{
  "message": "Product removed from wishlist successfully"
}
```

**Hiba:** 404 Not Found
```json
{
  "message": "Wishlist item not found"
}
```

### Admin végpontok (admin jogosultság szükséges)

#### POST /products
Új termék létrehozása (csak admin).

**Kérés törzse:**
```json
{
  "name": "New Product",
  "category": "Electronics",
  "price": 299.99,
  "stock": 50
}
```

**Válasz:** 201 Created
```json
{
  "message": "Product created successfully",
  "product": {
    "name": "New Product",
    "category": "Electronics",
    "price": 299.99,
    "stock": 50,
    "updated_at": "2025-12-06T10:30:00.000000Z",
    "created_at": "2025-12-06T10:30:00.000000Z",
    "id": 25
  }
}
```

#### PUT /products/{id}
Termék módosítása (csak admin).

**Kérés törzse:**
```json
{
  "name": "Updated Product Name",
  "price": 399.99,
  "stock": 25
}
```

**Válasz:** 200 OK
```json
{
  "message": "Product updated successfully",
  "product": {
    "id": 1,
    "name": "Updated Product Name",
    "category": "Electronics",
    "price": "399.99",
    "stock": 25,
    "created_at": "2025-12-06T10:00:00.000000Z",
    "updated_at": "2025-12-06T10:35:00.000000Z"
  }
}
```

#### DELETE /products/{id}
Termék törlése (csak admin).

**Válasz:** 200 OK
```json
{
  "message": "Product deleted successfully"
}
```

#### GET /users
Összes felhasználó listázása (csak admin).

**Válasz:** 200 OK
```json
[
  {
    "id": 1,
    "name": "Admin User",
    "email": "admin@example.com",
    "is_admin": true,
    "email_verified_at": null,
    "created_at": "2025-12-06T09:00:00.000000Z",
    "updated_at": "2025-12-06T09:00:00.000000Z"
  },
  {
    "id": 2,
    "name": "John Doe",
    "email": "john@example.com",
    "is_admin": false,
    "email_verified_at": null,
    "created_at": "2025-12-06T10:30:00.000000Z",
    "updated_at": "2025-12-06T10:30:00.000000Z"
  }
]
```

#### GET /admin/wishlists
Összes kívánságlista lekérése (csak admin).

**Válasz:** 200 OK
```json
[
  {
    "id": 1,
    "user_id": 2,
    "product_id": 1,
    "added_at": "2025-12-06T10:30:00.000000Z",
    "created_at": "2025-12-06T10:30:00.000000Z",
    "updated_at": "2025-12-06T10:30:00.000000Z",
    "user": {
      "id": 2,
      "name": "John Doe",
      "email": "john@example.com"
    },
    "product": {
      "id": 1,
      "name": "Laptop Dell XPS 15",
      "category": "Electronics",
      "price": "1299.99",
      "stock": 15
    }
  }
]
```

#### GET /admin/users/{userId}/wishlists
Adott felhasználó kívánságlistája (csak admin).

**Válasz:** 200 OK
```json
[
  {
    "id": 1,
    "user_id": 2,
    "product_id": 1,
    "added_at": "2025-12-06T10:30:00.000000Z",
    "created_at": "2025-12-06T10:30:00.000000Z",
    "updated_at": "2025-12-06T10:30:00.000000Z",
    "user": {
      "id": 2,
      "name": "John Doe",
      "email": "john@example.com"
    },
    "product": {
      "id": 1,
      "name": "Laptop Dell XPS 15",
      "category": "Electronics",
      "price": "1299.99",
      "stock": 15
    }
  }
]
```

**Admin jogosultság hiánya esetén:** 403 Forbidden
```json
{
  "message": "This action is unauthorized."
}
```

---

## VI. Tesztelés

A projektben már léteznek a Feature tesztek a `tests/Feature/` mappában:

### 1. Létező Teszt Fájlok

- `AuthApiTest.php` - Autentikációs tesztek
- `ProductApiTest.php` - Termék API tesztek  
- `UserApiTest.php` - Felhasználó API tesztek
- `WishlistApiTest.php` - Kívánságlista API tesztek

### 2. Tesztek Futtatása

```powershell
# Összes teszt futtatása
php artisan test

# Specifikus teszt futtatása
php artisan test tests/Feature/AuthApiTest.php

# Részletes kimenet
php artisan test --verbose
```

### 3. Teszt Adatbázis

A tesztek külön adatbázist használnak (memory vagy sqlite), ezért nem befolyásolják a fejlesztési adatokat.

---

## VII. Végpontok Összefoglalója

| HTTP Metódus | Útvonal | Jogosultság | Státuszkódok | Rövid Leírás |
|--------------|---------|-------------|--------------|--------------|
| POST | /register | Nyilvános | 201 Created, 422 Unprocessable Entity | Új felhasználó regisztrációja |
| POST | /login | Nyilvános | 200 OK, 401 Unauthorized | Bejelentkezés |
| GET | /products | Nyilvános | 200 OK | Összes termék listázása |
| GET | /products/{id} | Nyilvános | 200 OK, 404 Not Found | Adott termék lekérése |
| POST | /logout | Hitelesített | 200 OK | Kijelentkezés |
| GET | /me | Hitelesített | 200 OK | Saját profil lekérése |
| GET | /wishlists | Hitelesített | 200 OK | Saját kívánságlista lekérése |
| POST | /wishlists | Hitelesített | 201 Created, 409 Conflict, 422 Unprocessable Entity | Termék hozzáadása kívánságlistához |
| GET | /wishlists/{id} | Hitelesített | 200 OK, 404 Not Found | Kívánságlista elem lekérése |
| DELETE | /wishlists/{id} | Hitelesített | 200 OK, 404 Not Found | Termék eltávolítása kívánságlistából |
| POST | /products | Admin | 201 Created, 422 Unprocessable Entity | Új termék létrehozása |
| PUT | /products/{id} | Admin | 200 OK, 404 Not Found, 422 Unprocessable Entity | Termék módosítása |
| DELETE | /products/{id} | Admin | 200 OK, 404 Not Found | Termék törlése |
| GET | /users | Admin | 200 OK, 403 Forbidden | Összes felhasználó listázása |
| GET | /users/{id} | Admin | 200 OK, 403 Forbidden, 404 Not Found | Felhasználó lekérése |
| PUT | /users/{id} | Admin | 200 OK, 403 Forbidden, 404 Not Found | Felhasználó módosítása |
| DELETE | /users/{id} | Admin | 200 OK, 403 Forbidden, 404 Not Found | Felhasználó törlése |
| GET | /admin/wishlists | Admin | 200 OK, 403 Forbidden | Összes kívánságlista (admin nézet) |
| GET | /admin/users/{userId}/wishlists | Admin | 200 OK, 403 Forbidden | Felhasználó kívánságlistája (admin) |

---

## VIII. Postman Teszt Kollekció

### Környezeti Változók
```json
{
  "base_url": "http://127.0.0.1:8000/api",
  "base_url_xampp": "http://127.0.0.1/wishlist_auth/public/api", 
  "token": "",
  "admin_token": ""
}
```

### Teszt Sorrend (User Flow)

1. **POST** `/register` - Felhasználó regisztrálása
2. **POST** `/login` - Bejelentkezés (token mentése)
3. **GET** `/me` - Saját profil ellenőrzése
4. **GET** `/products` - Termékek böngészése
5. **GET** `/products/1` - Adott termék részletei
6. **POST** `/wishlists` - Termék hozzáadása kívánságlistához
   ```json
   {
     "product_id": 1
   }
   ```
7. **GET** `/wishlists` - Saját kívánságlista megtekintése
8. **GET** `/wishlists/1` - Kívánságlista elem részletei
9. **DELETE** `/wishlists/1` - Termék eltávolítása kívánságlistából
10. **POST** `/logout` - Kijelentkezés

### Admin Teszt Sorrend

1. **POST** `/login` - Admin bejelentkezés
   ```json
   {
     "email": "admin@example.com",
     "password": "password"
   }
   ```
2. **GET** `/users` - Összes felhasználó listázása
3. **POST** `/products` - Új termék létrehozása
   ```json
   {
     "name": "Test Product",
     "category": "Test Category", 
     "price": 99.99,
     "stock": 10
   }
   ```
4. **PUT** `/products/1` - Termék módosítása
5. **GET** `/admin/wishlists` - Összes kívánságlista megtekintése
6. **GET** `/admin/users/2/wishlists` - Felhasználó kívánságlistája

---

## IX. Telepítési Útmutató

### Rendszerkövetelmények
- PHP 8.2+
- MySQL 8.0+ vagy MariaDB 10.4+
- Composer 2.x
- XAMPP vagy Laravel Valet/Herd

### Telepítési Lépések (meglévő projekt)

1. **Projekt klónozása**
```powershell
git clone [repository_url] wishlist_auth
cd wishlist_auth
```

2. **Függőségek telepítése**
```powershell
composer install
npm install # ha van frontend
```

3. **Környezeti konfiguráció**
```powershell
cp .env.example .env
php artisan key:generate
```

4. **Adatbázis konfiguráció**
```powershell
# phpMyAdmin-ban hozd létre: wishlists adatbázist
# .env fájl módosítása
```

5. **Migráció és seeding**
```powershell
php artisan migrate
php artisan db:seed
```

6. **Szerver indítása**
```powershell
php artisan serve
# vagy XAMPP használata
```

7. **Tesztelés**
```powershell
php artisan test
```

### Új projekt létrehozása (nulláról)

```powershell
# Laravel projekt létrehozása
composer create-project laravel/laravel wishlist_auth

cd wishlist_auth

# Sanctum telepítése
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan install:api

# Admin middleware létrehozása
php artisan make:middleware IsAdmin

# Modellek és controller-ek létrehozása
php artisan make:model Product -m
php artisan make:model Wishlist -m
php artisan make:controller Api/AuthController
php artisan make:controller Api/ProductController
php artisan make:controller Api/UserController  
php artisan make:controller Api/WishlistController

# Factory-k és seederek
php artisan make:factory ProductFactory
php artisan make:factory WishlistFactory
php artisan make:seeder UserSeeder
php artisan make:seeder ProductSeeder
php artisan make:seeder WishlistSeeder

# Tesztek
php artisan make:test AuthApiTest
php artisan make:test ProductApiTest
php artisan make:test UserApiTest
php artisan make:test WishlistApiTest
```

### Fejlesztői Fiókok

A seeding után elérhető fiókok:

- **Admin:** admin@example.com / password  
- **User 1:** john@example.com / password
- **User 2:** jane@example.com / password
- **Factory Users:** további 7 generált felhasználó / password

---

## X. Hibaelhárítás

### Gyakori Hibák

1. **Token hibák (401 Unauthorized)**
   ```
   Megoldás:
   - Ellenőrizd a token helyességét
   - Győződj meg róla, hogy a Bearer prefix megvan
   - Token lejárt: jelentkezz be újra
   ```

2. **Admin jogosultság hibák (403 Forbidden)**
   ```
   Megoldás:
   - Ellenőrizd, hogy admin felhasználóval vagy bejelentkezve
   - Middleware konfiguráció ellenőrzése
   - is_admin mező értékének ellenőrzése az adatbázisban
   ```

3. **CORS hibák**
   ```
   Megoldás:
   - config/cors.php beállítások ellenőrzése
   - Sanctum middleware konfigurációja
   - Böngésző fejlesztői konzol ellenőrzése
   ```

4. **Adatbázis kapcsolat hibák**
   ```
   Megoldás:
   - .env fájl adatbázis beállításait
   - MySQL/MariaDB szerver futása
   - Adatbázis létezik-e (wishlists)
   - php artisan config:cache
   ```

5. **Validációs hibák (422 Unprocessable Entity)**
   ```
   Megoldás:
   - Kérés formátum ellenőrzése (JSON)
   - Kötelező mezők megadása
   - Egyedi megszorítások (email, user_id+product_id)
   ```

6. **Duplikált kívánságlista elem (409 Conflict)**
   ```
   Ez normális viselkedés - egy felhasználó nem adhatja hozzá 
   kétszer ugyanazt a terméket a kívánságlistájához.
   ```

### Debug Módok

1. **Laravel Debug**
   ```
   .env fájlban: APP_DEBUG=true
   APP_LOG_LEVEL=debug
   ```

2. **Query Log**
   ```php
   DB::enableQueryLog();
   // API hívás
   dd(DB::getQueryLog());
   ```

3. **API Válasz Debug**
   ```
   Postman Console vagy Network tab használata
   HTTP status kódok és response body ellenőrzése
   ```

---

## XI. További Fejlesztési Lehetőségek

### Funkcionális Bővítések

1. **Termék értékelések és vélemények**
2. **Kedvenc kategóriák**
3. **Kívánságlista megosztása más felhasználókkal**
4. **Email értesítések ár változáskor**
5. **Termék képek feltöltése**
6. **Kosár funkcionalitás**
7. **Rendelés kezelés**

### Technikai Fejlesztések

1. **API verziókezelés (v1, v2)**
2. **Rate limiting**
3. **Cache implementáció (Redis)**
4. **Queue-k használata email-ek küldésére**
5. **API dokumentáció generálás (Swagger/OpenAPI)**
6. **Elasticsearch termék kereséshez**
7. **WebSocket real-time értesítésekhez**

---

## Konklúzió

Ez a wishlist API teljes körű megoldást nyújt termékek és kívánságlisták kezelésére. Az API REST elveket követ, biztonságos Sanctum autentikációt használ, admin és felhasználói jogosultságokat kezel, és átfogó tesztelést biztosít. 

A dokumentáció minden szükséges információt tartalmaz a fejlesztéshez, telepítéshez és használathoz. A projekt könnyen bővíthető újabb funkciókkal és testreszabható különböző üzleti igények szerint.