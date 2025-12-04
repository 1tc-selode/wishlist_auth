# Wishlist API – Laravel projekt teljes útmutató

Ez a dokumentáció lépésről lépésre bemutatja, hogyan tudsz **nulláról** felépíteni egy Laravel alapú Wishlist (Kívánságlista) REST API-t, ahol felhasználók termékeket kedvelhetnek, adminok pedig kezelhetik a termékeket és felhasználókat.

---

## A cél
Egy olyan API-t készítünk, ahol a felhasználók:
- regisztrálhatnak, bejelentkezhetnek
- termékeket adhatnak a kívánságlistájukhoz
- megnézhetik, szerkeszthetik, törölhetik a saját kívánságlistájukat
Adminok:
- kezelhetik a termékeket (CRUD)
- kezelhetik a felhasználókat (CRUD)
- láthatják mindenki kívánságlistáját

Ez egy **CRUD** rendszer (Create, Read, Update, Delete) + jogosultságkezelés.

---

## API végpontok (röviden)
| HTTP metódus | URL | Művelet |
|--------------|-----|---------|
| POST | /api/register | Regisztráció |
| POST | /api/login | Bejelentkezés |
| GET | /api/products | Összes termék |
| GET | /api/products/{id} | Egy termék |
| POST | /api/products | Új termék (admin) |
| PUT | /api/products/{id} | Termék módosítása (admin) |
| DELETE | /api/products/{id} | Termék törlése (admin) |
| GET | /api/wishlists | Saját kívánságlista |
| POST | /api/wishlists | Termék hozzáadása a kívánságlistához |
| GET | /api/wishlists/{id} | Kívánságlista elem |
| DELETE | /api/wishlists/{id} | Kívánságlista elem törlése |
| GET | /api/users | Összes felhasználó (admin) |
| GET | /api/users/{id} | Egy felhasználó (admin) |
| PUT | /api/users/{id} | Felhasználó módosítása (admin) |
| DELETE | /api/users/{id} | Felhasználó törlése (admin) |
| GET | /api/admin/wishlists | Minden kívánságlista (admin) |
| GET | /api/admin/users/{id}/wishlists | Adott user kívánságlistája (admin) |

A részletes végpontok: [API_TEST_ENDPOINTS.md](../API_TEST_ENDPOINTS.md)

---

## 1. A környezet előkészítése
- Indítsd el a **XAMPP**-ot (Apache + MySQL)
- Ellenőrizd, hogy a `htdocs` mappában dolgozol

---

## 2. Új Laravel projekt létrehozása
```powershell
cd C:\xampp\htdocs
composer create-project laravel/laravel --prefer-dist wishlists
```
Ez létrehozza a Laravel keretrendszert a `C:\xampp\htdocs\wishlists` mappában.

---

## 3. API fejlesztés előkészítése
```powershell
cd wishlists
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```
- Telepíti a Sanctum csomagot (API tokenes autentikáció)
- Létrehozza a szükséges táblákat

---

## 4. Adatbázis létrehozása
- Nyisd meg a **phpMyAdmin**-t
- Hozz létre egy új adatbázist: `wishlists`
- Kódolás: `utf8mb4_unicode_ci`

---

## 5. Adatbázis kapcsolat beállítása
- Nyisd meg a `.env` fájlt
- Állítsd be:
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=wishlists
DB_USERNAME=root
DB_PASSWORD=
```

---

## 6. Modellek és migrációk létrehozása
### User modell (alapból van)
### Product modell
```powershell
php artisan make:model Product -m
```
### Wishlist modell
```powershell
php artisan make:model Wishlist -m
```

---

## 7. Migrációk szerkesztése
- Szerkeszd a `database/migrations` mappában a migrációs fájlokat:
  - **products**: id, name, category, price, stock, timestamps
  - **wishlists**: id, user_id, product_id, added_at, timestamps
  - **users**: is_admin mező hozzáadása (boolean)

Példa products migration:
```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('category');
    $table->decimal('price', 8, 2);
    $table->integer('stock');
    $table->timestamps();
});
```

Példa wishlists migration:
```php
Schema::create('wishlists', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->foreignId('product_id')->constrained()->onDelete('cascade');
    $table->timestamp('added_at')->nullable();
    $table->timestamps();
    $table->unique(['user_id', 'product_id']);
});
```

---

## 8. Modellek kitöltése
- Add meg a `$fillable` mezőket minden modellben
- User modellben: `is_admin` mező
- Wishlist modellben: kapcsolatok (belongsTo User, Product)

Példa:
```php
class Product extends Model {
    protected $fillable = ['name', 'category', 'price', 'stock'];
}
```

---

## 9. Migrate futtatása
```powershell
php artisan migrate
```
Ez létrehozza az adatbázis táblákat.

---

## 10. Tesztadatok létrehozása (Seeder)
```powershell
php artisan make:seeder UserSeeder
php artisan make:seeder ProductSeeder
php artisan make:seeder WishlistSeeder
```
- Töltsd ki a seedereket teszt adatokkal
- A `database/seeders/DatabaseSeeder.php`-ben hívd meg őket:
```php
$this->call([
    UserSeeder::class,
    ProductSeeder::class,
    WishlistSeeder::class,
]);
```
Futtatás:
```powershell
php artisan db:seed
```

---

## 11. Kontroller(ek) létrehozása
```powershell
php artisan make:controller Api/AuthController
php artisan make:controller Api/ProductController
php artisan make:controller Api/WishlistController
php artisan make:controller Api/UserController
```
- Írd meg a CRUD logikát a kontrollerekben
- AuthController: regisztráció, login, logout, profil
- ProductController: termék CRUD
- WishlistController: saját kívánságlista CRUD
- UserController: admin CRUD

---

## 12. Middleware létrehozása admin jogosultsághoz
```powershell
php artisan make:middleware IsAdmin
```
- Ellenőrizze, hogy a felhasználó admin-e
- Használd a route-oknál: `->middleware('admin')`

---

## 13. Route-ok beállítása
- Szerkeszd a `routes/api.php`-t:
- Állítsd be az összes végpontot a fenti táblázat szerint

Példa:
```php
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);
Route::get('/products', [ProductController::class, 'index']);
// ...további route-ok
```

---

## 14. Laravel kulcs generálása
```powershell
php artisan key:generate
```

---

## 15. Szerver indítása
```powershell
php artisan serve
```
Az alap URL: http://localhost:8000 vagy http://localhost/wishlists/public

---

## 16. API tesztelése Postman-ben
- Nyisd meg a Postman-t
- Importáld a `Wishlist_API.postman_collection.json` fájlt:
  - Postman → Import → File → válaszd ki a fájlt
- Teszteld az összes végpontot

---

## 17. Részletes végpontok és mintaválaszok
- [API_TEST_ENDPOINTS.md](../API_TEST_ENDPOINTS.md)

## 18. Postman Collection importálása
- Fájl: `Wishlist_API.postman_collection.json`
- Postman → Import → File → válaszd ki a fájlt
- Minden végpont tesztelhető

---

Ha ezt a dokumentációt követed, **nulláról** fel tudod építeni és tesztelni a teljes Wishlist API-t!
