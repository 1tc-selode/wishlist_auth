

# Laravel Wishlist API – Teljes Ultimate Guide

Ez a dokumentáció lépésről lépésre, minden parancsot, minden kódrészletet, minden magyarázatot tartalmaz, hogy nulláról felépítsd a teljes Laravel Wishlist API-t. Minden kód előtt magyar magyarázat (KODMAGYARAZAT) található!

---

## 1. Composer telepítése

**KODMAGYARAZAT:** A Composer a PHP csomagkezelője, szükséges a Laravel telepítéséhez.

```cmd
# Windows parancssorban:
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

---

## 2. Laravel projekt létrehozása

**KODMAGYARAZAT:** Ez létrehozza a Laravel keretrendszert a kívánt mappában.

```cmd
composer create-project laravel/laravel wishlists
```

---

## 3. Függőségek telepítése

**KODMAGYARAZAT:** A Laravel Sanctum csomag az API tokenes autentikációhoz kell.

```cmd
cd wishlists
composer require laravel/sanctum
npm install
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

---

## 4. .env beállítása

**KODMAGYARAZAT:** Az adatbázis kapcsolatot itt állítod be. Használd a saját MySQL adataidat.

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=wishlists
DB_USERNAME=root
DB_PASSWORD=
```

---

## 5. Adatbázis létrehozása

**KODMAGYARAZAT:** Hozz létre egy új adatbázist a phpMyAdmin-ban vagy MySQL-ben.

- Adatbázis neve: `wishlists`
- Kódolás: `utf8mb4_unicode_ci`

---

## 6. Modellek és migrációk létrehozása

**KODMAGYARAZAT:** A modellek az adatbázis táblákat írják le, a migrációk a táblák szerkezetét.

```cmd
php artisan make:model Product -m
php artisan make:model Wishlist -m
```

---

## 7. Migrációk kitöltése

**KODMAGYARAZAT:** Ezek a migrációs fájlok hozzák létre a users, products, wishlists táblákat.

### database/migrations/xxxx_xx_xx_xxxxxx_create_users_table.php
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

### database/migrations/xxxx_xx_xx_xxxxxx_create_products_table.php
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

### database/migrations/xxxx_xx_xx_xxxxxx_create_wishlists_table.php
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

---

## 8. Migrációk futtatása

**KODMAGYARAZAT:** Ez a parancs létrehozza az adatbázis táblákat.

```cmd
php artisan migrate
```

---

## 9. Modellek kitöltése

**KODMAGYARAZAT:** A modellekben megadod, milyen mezők tölthetők ki, és milyen kapcsolatok vannak.

### app/Models/Product.php
```php
class Product extends Model {
	protected $fillable = ['name', 'category', 'price', 'stock'];
	public function wishlists() { return $this->hasMany(Wishlist::class); }
	public function wishlistedByUsers() { return $this->belongsToMany(User::class, 'wishlists')->withTimestamps()->withPivot('added_at'); }
}
```

### app/Models/Wishlist.php
```php
class Wishlist extends Model {
	protected $fillable = ['user_id', 'product_id', 'added_at'];
	public function user() { return $this->belongsTo(User::class); }
	public function product() { return $this->belongsTo(Product::class); }
}
```

### app/Models/User.php
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

---

## 10. Seederek létrehozása

**KODMAGYARAZAT:** A seederek tesztadatokat töltenek az adatbázisba.

```cmd
php artisan make:seeder UserSeeder
php artisan make:seeder ProductSeeder
php artisan make:seeder WishlistSeeder
```

---

## 11. Seederek kitöltése

**KODMAGYARAZAT:** Ezek a fájlok hozzák létre az admin, user, termék és kívánságlista tesztadatokat.

### database/seeders/UserSeeder.php
```php
class UserSeeder extends Seeder {
	public function run(): void {
		User::create(['name' => 'Admin User', 'email' => 'admin@example.com', 'password' => Hash::make('password'), 'is_admin' => true]);
		User::create(['name' => 'John Doe', 'email' => 'john@example.com', 'password' => Hash::make('password'), 'is_admin' => false]);
		User::create(['name' => 'Jane Smith', 'email' => 'jane@example.com', 'password' => Hash::make('password'), 'is_admin' => false]);
	}
}
```

### database/seeders/ProductSeeder.php
```php
class ProductSeeder extends Seeder {
	public function run(): void {
		$products = [
			['name' => 'Laptop Dell XPS 15', 'category' => 'Electronics', 'price' => 1299.99, 'stock' => 15],
			['name' => 'iPhone 15 Pro', 'category' => 'Electronics', 'price' => 999.99, 'stock' => 25],
			// ... további termékek ...
		];
		foreach ($products as $product) { Product::create($product); }
	}
}
```

### database/seeders/WishlistSeeder.php
```php
class WishlistSeeder extends Seeder {
	public function run(): void {
		$john = User::where('email', 'john@example.com')->first();
		if ($john) {
			Wishlist::create(['user_id' => $john->id, 'product_id' => 1, 'added_at' => now()->subDays(5)]);
			// ... további kívánságok ...
		}
		$jane = User::where('email', 'jane@example.com')->first();
		if ($jane) {
			Wishlist::create(['user_id' => $jane->id, 'product_id' => 5, 'added_at' => now()->subDays(7)]);
			// ... további kívánságok ...
		}
	}
}
```

### database/seeders/DatabaseSeeder.php
```php
class DatabaseSeeder extends Seeder {
	public function run(): void {
		$this->call([UserSeeder::class, ProductSeeder::class, WishlistSeeder::class]);
	}
}
```

---

## 12. Seederek futtatása

**KODMAGYARAZAT:** Ez a parancs feltölti az adatbázist tesztadatokkal.

```cmd
php artisan db:seed
```

---

## 13. Middleware létrehozása admin jogosultsághoz

**KODMAGYARAZAT:** Az IsAdmin middleware csak admin felhasználóknak engedélyez bizonyos műveleteket.

```cmd
php artisan make:middleware IsAdmin
```

### app/Http/Middleware/IsAdmin.php
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

---

## 14. Kontrollerek létrehozása

**KODMAGYARAZAT:** A kontrollerek kezelik az API végpontokat és az üzleti logikát.

```cmd
php artisan make:controller Api/AuthController
php artisan make:controller Api/ProductController
php artisan make:controller Api/WishlistController
php artisan make:controller Api/UserController
```

---

## 15. Kontrollerek kitöltése

**KODMAGYARAZAT:** Az AuthController kezeli a regisztrációt, bejelentkezést, kijelentkezést, profil lekérést.

### app/Http/Controllers/Api/AuthController.php
```php
class AuthController extends Controller {
	public function register(Request $request) {
		// ... validáció, user létrehozás ...
	}
	public function login(Request $request) {
		// ... validáció, token generálás ...
	}
	public function logout(Request $request) {
		// ... token törlés ...
	}
	public function me(Request $request) {
		// ... aktuális user visszaadása ...
	}
}
```

**KODMAGYARAZAT:** A ProductController kezeli a termékek CRUD műveleteit.

### app/Http/Controllers/Api/ProductController.php
```php
class ProductController extends Controller {
	public function index(Request $request) {
		// ... összes termék lekérése ...
	}
	public function store(Request $request) {
		// ... új termék létrehozása ...
	}
	public function show($id) {
		// ... egy termék lekérése ...
	}
	public function update(Request $request, $id) {
		// ... termék módosítása ...
	}
	public function destroy($id) {
		// ... termék törlése ...
	}
}
```

**KODMAGYARAZAT:** A WishlistController kezeli a kívánságlista CRUD műveleteit.

### app/Http/Controllers/Api/WishlistController.php
```php
class WishlistController extends Controller {
	public function index(Request $request) {
		// ... saját kívánságlista lekérése ...
	}
	public function store(Request $request) {
		// ... termék hozzáadása a kívánságlistához ...
	}
	public function show(Request $request, $id) {
		// ... egy kívánságlista elem lekérése ...
	}
	public function destroy(Request $request, $id) {
		// ... kívánságlista elem törlése ...
	}
	public function indexAll() {
		// ... összes kívánságlista adminnak ...
	}
	public function getUserWishlist($userId) {
		// ... adott user kívánságlistája adminnak ...
	}
}
```

**KODMAGYARAZAT:** A UserController admin funkciókat kezel.

### app/Http/Controllers/Api/UserController.php
```php
class UserController extends Controller {
	public function index() {
		// ... összes user lekérése ...
	}
	public function show($id) {
		// ... egy user lekérése ...
	}
	public function update(Request $request, $id) {
		// ... user módosítása ...
	}
	public function destroy($id) {
		// ... user törlése ...
	}
}
```

---

## 16. Route-ok beállítása

**KODMAGYARAZAT:** Az API végpontokat a routes/api.php fájlban definiálod.

### routes/api.php
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

---

## 17. Laravel kulcs generálása

**KODMAGYARAZAT:** A kulcs az alkalmazás titkosításához kell.

```cmd
php artisan key:generate
```

---

## 18. Szerver indítása

**KODMAGYARAZAT:** Ez elindítja a Laravel fejlesztői szervert.

```cmd
php artisan serve
```

---

## 19. Postman Collection importálása

**KODMAGYARAZAT:** A Postman-ben tesztelheted az API végpontokat.

- Fájl: `Wishlist_API.postman_collection.json`
- Postman → Import → File → válaszd ki a fájlt

---

## 20. Tesztelés, API végpontok

**KODMAGYARAZAT:** Az API végpontok részletes leírását lásd az API_TEST_ENDPOINTS.md fájlban.

---

Ha ezt a dokumentációt követed, minden lépést, minden kódot, minden magyarázatot megtalálsz, és nulláról fel tudod építeni a teljes Laravel Wishlist API-t!
