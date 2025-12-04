# Wishlist API - Teljes Dokumentáció

## Mi a program lényege?
Ez egy Laravel alapú backend API, amivel felhasználók termékeket adhatnak hozzá a saját kívánságlistájukhoz. Az admin felhasználók kezelhetik a termékeket és felhasználókat, míg a sima felhasználók csak a saját kívánságlistájukat láthatják és szerkeszthetik.

## Struktúra
- **Laravel 11**
- **Sanctum**: API tokenes autentikáció
- **MySQL** adatbázis
- Fő mappák: `app/`, `routes/`, `database/`, `config/`, `public/`
- Fő modellek: User, Product, Wishlist
- Fő végpontok: Regisztráció, Bejelentkezés, Termék CRUD, Kívánságlista CRUD, Admin CRUD

## Telepítés és indítás lépésről lépésre

### 1. Projekt klónozása
```powershell
# Parancs: klónozd a repót
# (Ha nincs repó, másold a teljes mappát)
```

### 2. Függőségek telepítése
```powershell
composer install
npm install
```

### 3. .env beállítása
Másold az `.env.example`-t `.env`-nek, majd szerkeszd:
- DB_DATABASE=wishlists
- DB_USERNAME=root
- DB_PASSWORD= (vagy amit használsz)

### 4. Adatbázis migrációk és seederek
```powershell
php artisan migrate --seed
```
Ez létrehozza a táblákat és feltölti teszt adatokkal (admin, felhasználók, termékek, kívánságlisták).

### 5. Laravel key generálás
```powershell
php artisan key:generate
```

### 6. Szerver indítása
```powershell
php artisan serve
```
A szerver elindul, az alap URL: http://localhost:8000 vagy http://localhost/wishlists/public

### 7. API tesztelése Postman-ben
- Nyisd meg a Postman-t
- Importáld a `Wishlist_API.postman_collection.json` fájlt:
  - Postman-ben: Import → File → válaszd ki a fájlt
- Az összes végpontot tesztelheted a gyűjteményből

### 8. API végpontok
A részletes végpont lista és mintaválaszok: [API_TEST_ENDPOINTS.md](./API_TEST_ENDPOINTS.md)

## Fő kódrészletek és magyarázat

### Regisztráció
- `routes/api.php`: POST `/register` → AuthController@register
- User model: `is_admin` mezővel
- Nincs token visszaadás regisztrációnál, csak login után

### Bejelentkezés
- POST `/login` → AuthController@login
- Visszaadja a token-t, amit minden védett végponthoz használni kell

### Termékek
- GET `/products` → összes termék
- Admin: POST, PUT, DELETE `/products` → termék CRUD

### Kívánságlista
- GET `/wishlists` → saját kívánságlista
- POST `/wishlists` → termék hozzáadása
- DELETE `/wishlists/{id}` → eltávolítás

### Admin funkciók
- GET `/users` → összes felhasználó
- GET `/admin/wishlists` → összes kívánságlista
- GET `/admin/users/{id}/wishlists` → adott user kívánságlistája

## Minden lépés sorrendben
1. Klónozd vagy másold a projektet
2. Telepítsd a composer és npm függőségeket
3. Állítsd be az `.env`-et (adatbázis, kulcsok)
4. Futtasd a migrációkat és seedereket
5. Generáld a Laravel kulcsot
6. Indítsd el a szervert
7. Importáld a Postman collection-t
8. Teszteld az API-t a [API_TEST_ENDPOINTS.md](./API_TEST_ENDPOINTS.md) alapján

## Postman Collection importálása
- Fájl: `Wishlist_API.postman_collection.json`
- Postman → Import → File → válaszd ki a fájlt
- Minden végpont tesztelhető

## Részletes végpontok
- [API_TEST_ENDPOINTS.md](./API_TEST_ENDPOINTS.md)

---

Ha ezt a dokumentációt követed, lépésről lépésre fel tudod építeni és tesztelni a teljes Wishlist API-t!
