# 🧪 Wishlist API - Teljes Végpont Lista Teszteléshez

**Base URL:** `http://localhost/wishlists/public/api`

---

## 🔓 PUBLIKUS VÉGPONTOK (Token nélkül)

### 1. Regisztráció

**Metódus:** `POST`

**URL:** `http://localhost/wishlists/public/api/register`

**Headers:**
```
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "name": "Test User",
  "email": "test@example.com",
  "password": "password",
  "password_confirmation": "password"
}
```

**Sikeres válasz (201):**
```json
{
  "message": "User registered successfully. Please login.",
  "user": {
    "id": 4,
    "name": "Test User",
    "email": "test@example.com",
    "is_admin": false
  }
}
```

**Megjegyzés:** Token NEM kerül visszaadásra. A regisztráció után be kell jelentkezni a `/api/login` végponton.

---

### 2. Regisztráció Admin jogosultsággal

**Metódus:** `POST`

**URL:** `http://localhost/wishlists/public/api/register`

**Headers:**
```
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "name": "New Admin",
  "email": "newadmin@example.com",
  "password": "password",
  "password_confirmation": "password",
  "is_admin": true
}
```

**Sikeres válasz (201):**
```json
{
  "message": "User registered successfully. Please login.",
  "user": {
    "id": 5,
    "name": "New Admin",
    "email": "newadmin@example.com",
    "is_admin": true
  }
}
```

**Megjegyzés:** Token NEM kerül visszaadásra. A regisztráció után be kell jelentkezni a `/api/login` végponton.

---

### 3. Bejelentkezés

**Metódus:** `POST`

**URL:** `http://localhost/wishlists/public/api/login`

**Headers:**
```
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "email": "admin@example.com",
  "password": "password"
}
```

**Sikeres válasz (200):**
```json
{
  "message": "Login successful",
  "user": {
    "id": 1,
    "name": "Admin User",
    "email": "admin@example.com",
    "email_verified_at": null,
    "is_admin": true,
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z"
  },
  "access_token": "1|abcdef123456789xyz",
  "token_type": "Bearer"
}
```

---

### 4. Összes termék listázása

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/products`

**Headers:**
```
Accept: application/json
```

**Sikeres válasz (200):**
```json
[
  {
    "id": 1,
    "name": "Laptop Dell XPS 15",
    "category": "Electronics",
    "price": "1299.99",
    "stock": 15,
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z"
  },
  {
    "id": 2,
    "name": "iPhone 15 Pro",
    "category": "Electronics",
    "price": "999.99",
    "stock": 25,
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z"
  }
]
```

---

### 5. Egy termék részletei

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/products/1`

**Headers:**
```
Accept: application/json
```

**Sikeres válasz (200):**
```json
{
  "id": 1,
  "name": "Laptop Dell XPS 15",
  "category": "Electronics",
  "price": "1299.99",
  "stock": 15,
  "created_at": "2025-12-04T08:13:31.000000Z",
  "updated_at": "2025-12-04T08:13:31.000000Z"
}
```

---

## 🔐 VÉDETT VÉGPONTOK (Token szükséges)

### 6. Kijelentkezés

**Metódus:** `POST`

**URL:** `http://localhost/wishlists/public/api/logout`

**Headers:**
```
Authorization: Bearer {token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
{
  "message": "Logged out successfully"
}
```

---

### 7. Saját profil lekérése

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/me`

**Headers:**
```
Authorization: Bearer {token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
{
  "id": 2,
  "name": "John Doe",
  "email": "john@example.com",
  "email_verified_at": null,
  "is_admin": false,
  "created_at": "2025-12-04T08:13:31.000000Z",
  "updated_at": "2025-12-04T08:13:31.000000Z"
}
```

---

### 8. Saját kívánságlista megtekintése

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/wishlists`

**Headers:**
```
Authorization: Bearer {token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
[
  {
    "id": 1,
    "user_id": 2,
    "product_id": 1,
    "added_at": "2025-11-29T08:13:31.000000Z",
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z",
    "product": {
      "id": 1,
      "name": "Laptop Dell XPS 15",
      "category": "Electronics",
      "price": "1299.99",
      "stock": 15,
      "created_at": "2025-12-04T08:13:31.000000Z",
      "updated_at": "2025-12-04T08:13:31.000000Z"
    }
  },
  {
    "id": 2,
    "user_id": 2,
    "product_id": 2,
    "added_at": "2025-12-01T08:13:31.000000Z",
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z",
    "product": {
      "id": 2,
      "name": "iPhone 15 Pro",
      "category": "Electronics",
      "price": "999.99",
      "stock": 25
    }
  }
]
```

---

### 9. Termék hozzáadása a kívánságlistához

**Metódus:** `POST`

**URL:** `http://localhost/wishlists/public/api/wishlists`

**Headers:**
```
Authorization: Bearer {token}
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "product_id": 8
}
```

**Sikeres válasz (201):**
```json
{
  "message": "Product added to wishlist",
  "wishlist": {
    "id": 10,
    "user_id": 2,
    "product_id": 8,
    "added_at": "2025-12-04T10:30:00.000000Z",
    "created_at": "2025-12-04T10:30:00.000000Z",
    "updated_at": "2025-12-04T10:30:00.000000Z",
    "product": {
      "id": 8,
      "name": "Gaming Mouse Logitech",
      "category": "Accessories",
      "price": "79.99",
      "stock": 100
    }
  }
}
```

---

### 10. Kívánságlista elem részletei

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/wishlists/1`

**Headers:**
```
Authorization: Bearer {token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
{
  "id": 1,
  "user_id": 2,
  "product_id": 1,
  "added_at": "2025-11-29T08:13:31.000000Z",
  "created_at": "2025-12-04T08:13:31.000000Z",
  "updated_at": "2025-12-04T08:13:31.000000Z",
  "product": {
    "id": 1,
    "name": "Laptop Dell XPS 15",
    "category": "Electronics",
    "price": "1299.99",
    "stock": 15
  }
}
```

---

### 11. Termék eltávolítása a kívánságlistából

**Metódus:** `DELETE`

**URL:** `http://localhost/wishlists/public/api/wishlists/1`

**Headers:**
```
Authorization: Bearer {token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
{
  "message": "Product removed from wishlist"
}
```

---

## 👑 ADMIN VÉGPONTOK (Admin token szükséges)

**Admin bejelentkezés:**
```
Email: admin@example.com
Password: password
```

### 12. Új termék létrehozása

**Metódus:** `POST`

**URL:** `http://localhost/wishlists/public/api/products`

**Headers:**
```
Authorization: Bearer {admin_token}
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "name": "New Product",
  "category": "Electronics",
  "price": 299.99,
  "stock": 50
}
```

**Sikeres válasz (201):**
```json
{
  "message": "Product created successfully",
  "product": {
    "id": 11,
    "name": "New Product",
    "category": "Electronics",
    "price": "299.99",
    "stock": 50,
    "created_at": "2025-12-04T10:45:00.000000Z",
    "updated_at": "2025-12-04T10:45:00.000000Z"
  }
}
```

---

### 13. Termék módosítása

**Metódus:** `PUT`

**URL:** `http://localhost/wishlists/public/api/products/11`

**Headers:**
```
Authorization: Bearer {admin_token}
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "name": "Updated Product Name",
  "category": "Electronics",
  "price": 349.99,
  "stock": 30
}
```

**Sikeres válasz (200):**
```json
{
  "message": "Product updated successfully",
  "product": {
    "id": 11,
    "name": "Updated Product Name",
    "category": "Electronics",
    "price": "349.99",
    "stock": 30,
    "created_at": "2025-12-04T10:45:00.000000Z",
    "updated_at": "2025-12-04T10:50:00.000000Z"
  }
}
```

---

### 14. Termék törlése

**Metódus:** `DELETE`

**URL:** `http://localhost/wishlists/public/api/products/11`

**Headers:**
```
Authorization: Bearer {admin_token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
{
  "message": "Product deleted successfully"
}
```

---

### 15. Összes felhasználó listázása

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/users`

**Headers:**
```
Authorization: Bearer {admin_token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
[
  {
    "id": 1,
    "name": "Admin User",
    "email": "admin@example.com",
    "email_verified_at": null,
    "is_admin": true,
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z"
  },
  {
    "id": 2,
    "name": "John Doe",
    "email": "john@example.com",
    "email_verified_at": null,
    "is_admin": false,
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z"
  },
  {
    "id": 3,
    "name": "Jane Smith",
    "email": "jane@example.com",
    "email_verified_at": null,
    "is_admin": false,
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z"
  }
]
```

---

### 16. Egy felhasználó adatai

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/users/2`

**Headers:**
```
Authorization: Bearer {admin_token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
{
  "id": 2,
  "name": "John Doe",
  "email": "john@example.com",
  "email_verified_at": null,
  "is_admin": false,
  "created_at": "2025-12-04T08:13:31.000000Z",
  "updated_at": "2025-12-04T08:13:31.000000Z"
}
```

---

### 17. Felhasználó módosítása

**Metódus:** `PUT`

**URL:** `http://localhost/wishlists/public/api/users/2`

**Headers:**
```
Authorization: Bearer {admin_token}
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "name": "Updated Name",
  "email": "updated@example.com",
  "is_admin": false
}
```

**Sikeres válasz (200):**
```json
{
  "message": "User updated successfully",
  "user": {
    "id": 2,
    "name": "Updated Name",
    "email": "updated@example.com",
    "is_admin": false,
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T11:00:00.000000Z"
  }
}
```

---

### 18. Felhasználó törlése

**Metódus:** `DELETE`

**URL:** `http://localhost/wishlists/public/api/users/2`

**Headers:**
```
Authorization: Bearer {admin_token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
{
  "message": "User deleted successfully"
}
```

---

### 19. Összes kívánságlista megtekintése

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/admin/wishlists`

**Headers:**
```
Authorization: Bearer {admin_token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
[
  {
    "id": 1,
    "user_id": 2,
    "product_id": 1,
    "added_at": "2025-11-29T08:13:31.000000Z",
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z",
    "user": {
      "id": 2,
      "name": "John Doe",
      "email": "john@example.com",
      "is_admin": false
    },
    "product": {
      "id": 1,
      "name": "Laptop Dell XPS 15",
      "category": "Electronics",
      "price": "1299.99",
      "stock": 15
    }
  },
  {
    "id": 2,
    "user_id": 2,
    "product_id": 2,
    "added_at": "2025-12-01T08:13:31.000000Z",
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z",
    "user": {
      "id": 2,
      "name": "John Doe",
      "email": "john@example.com",
      "is_admin": false
    },
    "product": {
      "id": 2,
      "name": "iPhone 15 Pro",
      "category": "Electronics",
      "price": "999.99",
      "stock": 25
    }
  }
]
```

---

### 20. Egy felhasználó kívánságlistája (admin)

**Metódus:** `GET`

**URL:** `http://localhost/wishlists/public/api/admin/users/2/wishlists`

**Headers:**
```
Authorization: Bearer {admin_token}
Accept: application/json
```

**Sikeres válasz (200):**
```json
[
  {
    "id": 1,
    "user_id": 2,
    "product_id": 1,
    "added_at": "2025-11-29T08:13:31.000000Z",
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z",
    "product": {
      "id": 1,
      "name": "Laptop Dell XPS 15",
      "category": "Electronics",
      "price": "1299.99",
      "stock": 15
    }
  },
  {
    "id": 2,
    "user_id": 2,
    "product_id": 2,
    "added_at": "2025-12-01T08:13:31.000000Z",
    "created_at": "2025-12-04T08:13:31.000000Z",
    "updated_at": "2025-12-04T08:13:31.000000Z",
    "product": {
      "id": 2,
      "name": "iPhone 15 Pro",
      "category": "Electronics",
      "price": "999.99",
      "stock": 25
    }
  }
]
```

---
