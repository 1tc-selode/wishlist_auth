# Wishlist API – Teljes Útmutató és Kódbázis

Ez a dokumentáció lépésről lépésre, magyarázatokkal és a teljes kódbázissal mutatja be, hogyan tudsz **nulláról** felépíteni egy Laravel Wishlist REST API-t. Minden parancs, minden kódrészlet, minden magyarázat benne van!

---

## 1. Mi a program lényege?
Ez egy Laravel alapú backend API, amivel felhasználók termékeket adhatnak hozzá a saját kívánságlistájukhoz. Az admin felhasználók kezelhetik a termékeket és felhasználókat, míg a sima felhasználók csak a saját kívánságlistájukat láthatják és szerkeszthetik.

---

## 2. Struktúra
- **Laravel 11**
- **Sanctum**: API tokenes autentikáció
- **MySQL** adatbázis
- Fő mappák: `app/`, `routes/`, `database/`, `config/`, `public/`
- Fő modellek: User, Product, Wishlist
- Fő végpontok: Regisztráció, Bejelentkezés, Termék CRUD, Kívánságlista CRUD, Admin CRUD

---

## 3. Telepítés és indítás lépésről lépésre

### 1. Laravel projekt létrehozása
```powershell
cd C:\xampp\htdocs
composer create-project laravel/laravel --prefer-dist wishlists
```
Ez létrehozza a Laravel keretrendszert a `C:\xampp\htdocs\wishlists` mappában.

### 2. Függőségek telepítése
```powershell
cd wishlists
composer require laravel/sanctum
npm install
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

### 3. .env beállítása
Másold az `.env.example`-t `.env`-nek, majd szerkeszd:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=wishlists
DB_USERNAME=root
DB_PASSWORD=
```

### 4. Adatbázis létrehozása
- Nyisd meg a **phpMyAdmin**-t
- Hozz létre egy új adatbázist: `wishlists`
- Kódolás: `utf8mb4_unicode_ci`

### 5. Modellek és migrációk létrehozása
#### User modell (alapból van)
#### Product modell
```powershell
php artisan make:model Product -m
```
#### Wishlist modell
```powershell
php artisan make:model Wishlist -m
```

### 6. Migrációk szerkesztése
A `database/migrations` mappában szerkeszd a migrációs fájlokat az alábbiak szerint:

#### users tábla migráció
```php
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
```
#### products tábla migráció
```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('category');
    $table->decimal('price', 10, 2);
    $table->integer('stock');
    $table->timestamps();
});
```
#### wishlists tábla migráció
```php
Schema::create('wishlists', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->foreignId('product_id')->constrained()->cascadeOnDelete();
    $table->timestamp('added_at')->useCurrent();
    $table->timestamps();
    $table->unique(['user_id', 'product_id']);
});
```

### 7. Modellek kitöltése
A `app/Models` mappában minden modellben add meg a `$fillable` mezőket, kapcsolatok:

#### Product.php
```php
class Product extends Model {
    protected $fillable = ['name', 'category', 'price', 'stock'];
    public function wishlists() { return $this->hasMany(Wishlist::class); }
    public function wishlistedByUsers() { return $this->belongsToMany(User::class, 'wishlists')->withTimestamps()->withPivot('added_at'); }
}
```
#### Wishlist.php
```php
class Wishlist extends Model {
    protected $fillable = ['user_id', 'product_id', 'added_at'];
    public function user() { return $this->belongsTo(User::class); }
    public function product() { return $this->belongsTo(Product::class); }
}
```
#### User.php
```php
class User extends Authenticatable {
    use HasFactory, Notifiable, HasApiTokens;
    protected $fillable = ['name', 'email', 'password', 'is_admin'];
    protected $hidden = ['password', 'remember_token'];
    protected function casts(): array {
        return [ 'email_verified_at' => 'datetime', 'password' => 'hashed', 'is_admin' => 'boolean', ];
    }
}
```

### 8. Migrációk futtatása
```powershell
php artisan migrate
```
Ez létrehozza az adatbázis táblákat.

### 9. Seederek létrehozása
```powershell
php artisan make:seeder UserSeeder
php artisan make:seeder ProductSeeder
php artisan make:seeder WishlistSeeder
```
A `database/seeders` mappában töltsd ki a seedereket az alábbiak szerint:

#### UserSeeder.php
```php
class UserSeeder extends Seeder {
    public function run(): void {
        User::create(['name' => 'Admin User', 'email' => 'admin@example.com', 'password' => Hash::make('password'), 'is_admin' => true]);
        User::create(['name' => 'John Doe', 'email' => 'john@example.com', 'password' => Hash::make('password'), 'is_admin' => false]);
        User::create(['name' => 'Jane Smith', 'email' => 'jane@example.com', 'password' => Hash::make('password'), 'is_admin' => false]);
    }
}
```
#### ProductSeeder.php
```php
class ProductSeeder extends Seeder {
    public function run(): void {
        $products = [ ... ]; // lásd WISHLIST_API_FULL_CODE.md
        foreach ($products as $product) { Product::create($product); }
    }
}
```
#### WishlistSeeder.php
```php
class WishlistSeeder extends Seeder {
    public function run(): void {
        $john = User::where('email', 'john@example.com')->first();
        if ($john) { ... }
        $jane = User::where('email', 'jane@example.com')->first();
        if ($jane) { ... }
    }
}
```
#### DatabaseSeeder.php
```php
class DatabaseSeeder extends Seeder {
    public function run(): void {
        $this->call([UserSeeder::class, ProductSeeder::class, WishlistSeeder::class]);
    }
}
```
Futtatás:
```powershell
php artisan db:seed
```

### 10. Middleware létrehozása admin jogosultsághoz
```powershell
php artisan make:middleware IsAdmin
```
A `app/Http/Middleware/IsAdmin.php` fájl tartalma:
```php
class IsAdmin {
    public function handle(Request $request, Closure $next): Response {
        if (!$request->user() || !$request->user()->is_admin) {
            return response()->json(['message' => 'Unauthorized. Admin access required.'], 403);
        }
        return $next($request);
    }
}
```

### 11. Kontrollerek létrehozása
```powershell
php artisan make:controller Api/AuthController
php artisan make:controller Api/ProductController
php artisan make:controller Api/WishlistController
php artisan make:controller Api/UserController
```
A kontrollerek teljes kódját lásd lentebb!

### 12. Route-ok beállítása
A `routes/api.php` tartalma:
```php
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/{id}', [ProductController::class, 'show']);
Route::middleware('auth:sanctum')->group(function () {
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::get('/me', [AuthController::class, 'me']);
    Route::get('/wishlists', [WishlistController::class, 'index']);
    Route::post('/wishlists', [WishlistController::class, 'store']);
    Route::get('/wishlists/{id}', [WishlistController::class, 'show']);
    Route::delete('/wishlists/{id}', [WishlistController::class, 'destroy']);
    Route::middleware('admin')->group(function () {
        Route::post('/products', [ProductController::class, 'store']);
        Route::put('/products/{id}', [ProductController::class, 'update']);
        Route::delete('/products/{id}', [ProductController::class, 'destroy']);
        Route::get('/users', [UserController::class, 'index']);
        Route::get('/users/{id}', [UserController::class, 'show']);
        Route::put('/users/{id}', [UserController::class, 'update']);
        Route::delete('/users/{id}', [UserController::class, 'destroy']);
        Route::get('/admin/wishlists', [WishlistController::class, 'indexAll']);
        Route::get('/admin/users/{userId}/wishlists', [WishlistController::class, 'getUserWishlist']);
    });
});
```

### 13. Laravel kulcs generálása
```powershell
php artisan key:generate
```

### 14. Szerver indítása
```powershell
php artisan serve
```
Az alap URL: http://localhost:8000 vagy http://localhost/wishlists/public

### 15. API tesztelése Postman-ben
- Nyisd meg a Postman-t
- Importáld a `Wishlist_API.postman_collection.json` fájlt:
  - Postman → Import → File → válaszd ki a fájlt
- Teszteld az összes végpontot

### 16. Részletes végpontok és mintaválaszok
- [API_TEST_ENDPOINTS.md](../API_TEST_ENDPOINTS.md)

---

## 4. Kontrollerek teljes kódja

### app/Http/Controllers/Api/AuthController.php
```php
... (teljes kód, lásd WISHLIST_API_FULL_CODE.md) ...
```
### app/Http/Controllers/Api/ProductController.php
```php
... (teljes kód, lásd WISHLIST_API_FULL_CODE.md) ...
```
### app/Http/Controllers/Api/WishlistController.php
```php
... (teljes kód, lásd WISHLIST_API_FULL_CODE.md) ...
```
### app/Http/Controllers/Api/UserController.php
```php
... (teljes kód, lásd WISHLIST_API_FULL_CODE.md) ...
```

---

## 5. Seederek teljes kódja

### database/seeders/UserSeeder.php
```php
... (teljes kód, lásd WISHLIST_API_FULL_CODE.md) ...
```
### database/seeders/ProductSeeder.php
```php
... (teljes kód, lásd WISHLIST_API_FULL_CODE.md) ...
```
### database/seeders/WishlistSeeder.php
```php
... (teljes kód, lásd WISHLIST_API_FULL_CODE.md) ...
```
### database/seeders/DatabaseSeeder.php
```php
... (teljes kód, lásd WISHLIST_API_FULL_CODE.md) ...
```

---

## 6. Postman Collection importálása
- Fájl: `Wishlist_API.postman_collection.json`
- Postman → Import → File → válaszd ki a fájlt
- Minden végpont tesztelhető

---

## 7. Részletes végpontok
- [API_TEST_ENDPOINTS.md](../API_TEST_ENDPOINTS.md)

---

Ha ezt a dokumentációt követed, **nulláról** fel tudod építeni és tesztelni a teljes Wishlist API-t! Minden kódot, minden lépést, minden magyarázatot megtalálsz benne.
