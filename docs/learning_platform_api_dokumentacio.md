# Tanulási Platform REST API – Teljes Dokumentáció és Kód

Ez a dokumentáció egy Laravel alapú tanulási platform backend API teljes fejlesztési útmutatója. A projekt célja egy olyan REST API létrehozása, amely lehetővé teszi a felhasználók számára kurzusokra való feliratkozást, tanulást és teljesítést. Az API olyan funkciókkal van ellátva, amelyek lehetővé teszik annak nyilvános elérhetőségét.

---

## Projekt Áttekintés

**Base URL-ek:**
- XAMPP: `http://127.0.0.1/learningPlatformBearer/public/api`
- Laravel serve: `http://127.0.0.1:8000/api`

**Technológiák:**
- Laravel 11/12
- Laravel Sanctum (API token hitelesítés)
- MySQL adatbázis
- PHPUnit tesztelés

**Adatbázis neve:** `learning_platform`

---

## Projekt Struktúra

```
learningPlatformBearer/
├── app/
│   ├── Http/
│   │   ├── Controllers/         # API vezérlők
│   │   │   ├── AuthController.php
│   │   │   ├── UserController.php
│   │   │   └── CourseController.php
│   │   └── Middleware/          # Middleware-ek
│   ├── Models/                  # Adatmodellek
│   │   ├── User.php             # Felhasználó modell
│   │   ├── Course.php           # Kurzus modell
│   │   └── Enrollment.php       # Beiratkozás modell
│   └── Providers/               # Laravel szolgáltatók
├── database/
│   ├── factories/               # Tesztadat generátorok
│   │   ├── UserFactory.php
│   │   ├── CourseFactory.php
│   │   └── EnrollmentFactory.php
│   ├── migrations/              # Adatbázis szerkezet
│   │   ├── 0001_01_01_000000_create_users_table.php
│   │   ├── *_create_courses_table.php
│   │   └── *_create_enrollments_table.php
│   └── seeders/                 # Tesztadat feltöltés
│       ├── UserSeeder.php
│       ├── CourseSeeder.php
│       └── EnrollmentSeeder.php
├── routes/
│   └── api.php                  # API útvonalak
├── tests/
│   └── Feature/                 # API funkció tesztek
│       ├── AuthTest.php
│       ├── UserTest.php
│       └── CourseTest.php
└── docs/                        # Dokumentáció
    └── learning_platform_api_dokumentacio.md
```

---

## Adatbázis Terv

```
+---------------------+     +---------------------+       +-----------------+        +------------+
|personal_access_tokens|    |        users        |       |   enrollments   |        |  courses   |
+---------------------+     +---------------------+       +-----------------+        +------------+
| id (PK)             |   _1| id (PK)             |1__    | id (PK)         |     __1| id (PK)    |
| tokenable_id (FK)   |K_/  | name                |   \__N| user_id (FK)    |    /   | title      |
| tokenable_type      |     | email (unique)      |       | course_id (FK)  |M__/    | description|
| name                |     | password            |       | enrolled_at     |        | created_at |
| token (unique)      |     | role (student/admin)|       | completed_at    |        | updated_at |
| abilities           |     | deleted_at          |       +-----------------+        +------------+
| last_used_at        |     +---------------------+
+---------------------+
```

---

## I. Modul - Telepítés és Konfiguráció

### 1. Laravel Projekt Létrehozása

```powershell
# Projekt létrehozása
composer create-project laravel/laravel --prefer-dist learningPlatformBearer

# Könyvtár váltás
cd learningPlatformBearer
```

### 2. .env Fájl Konfiguráció

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=learning_platform
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

### 4. Első Tesztútvonal (api.php)

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/ping', function () {
    return response()->json([
        'message' => 'API works!'
    ], 200);
});
```

### 5. Első Teszt

```powershell
# Laravel szerver indítása
php artisan serve

# POSTMAN teszt
# GET http://127.0.0.1:8000/api/ping
```

---

## II. Modul - Adatbázis és Modellek

### 1. Modellek és Migrációk Létrehozása

```powershell
php artisan make:model Course -m
php artisan make:model Enrollment -m
```

### 2. Migrációk Módosítása

#### users tábla módosítása (0001_01_01_000000_create_users_table.php)

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
            $table->enum('role', ['student', 'admin'])->default('student');
            $table->rememberToken();
            $table->softDeletes(); // deleted_at mező
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

#### courses tábla (create_courses_table.php)

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('courses', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('description')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('courses');
    }
};
```

#### enrollments tábla (create_enrollments_table.php)

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('enrollments', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->foreignId('course_id')->constrained()->cascadeOnDelete();
            $table->timestamp('enrolled_at')->useCurrent();
            $table->timestamp('completed_at')->nullable();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('enrollments');
    }
};
```

### 3. Modell Fájlok Módosítása

#### app/Models/User.php

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable, SoftDeletes;

    protected $fillable = [
        'name',
        'email',
        'password',
        'role',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    public function enrollments()
    {
        return $this->hasMany(Enrollment::class);
    }

    public function courses()
    {
        return $this->belongsToMany(Course::class, 'enrollments')
                    ->withPivot('enrolled_at', 'completed_at');
    }

    public function isAdmin()
    {
        return $this->role === 'admin';
    }
}
```

#### app/Models/Course.php

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Course extends Model
{
    use HasFactory;

    protected $fillable = [
        'title',
        'description',
    ];

    public function enrollments()
    {
        return $this->hasMany(Enrollment::class);
    }

    public function users()
    {
        return $this->belongsToMany(User::class, 'enrollments')
                    ->withPivot('enrolled_at', 'completed_at');
    }
}
```

#### app/Models/Enrollment.php

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Enrollment extends Model
{
    use HasFactory;

    public $timestamps = false;

    protected $fillable = [
        'user_id',
        'course_id',
        'enrolled_at',
        'completed_at',
    ];

    protected $dates = [
        'enrolled_at',
        'completed_at',
    ];

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function course()
    {
        return $this->belongsTo(Course::class);
    }
}
```

### 4. Migráció Futtatása

```powershell
php artisan migrate
```

---

## III. Modul - Seeding (Tesztadatok)

### 1. Factory-k Létrehozása és Módosítása

#### database/factories/UserFactory.php

```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use App\Models\User;

class UserFactory extends Factory
{
    protected $model = User::class;

    public function definition()
    {
        $this->faker = \Faker\Factory::create('hu_HU'); // magyar nevekhez

        return [
            'name' => $this->faker->firstName . ' ' . $this->faker->lastName,
            'email' => $this->faker->unique()->safeEmail(),
            'password' => Hash::make('Jelszo_2025'),
            'role' => 'student',
        ];
    }
}
```

### 2. Seederek Létrehozása

```powershell
php artisan make:seeder UserSeeder
php artisan make:seeder CourseSeeder
php artisan make:seeder EnrollmentSeeder
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
        // 1 admin
        User::create([
            'name' => 'admin',
            'email' => 'admin@example.com',
            'password' => Hash::make('admin'),
            'role' => 'admin',
        ]);

        // 9 student user
        User::factory(9)->create();
    }
}
```

#### database/seeders/CourseSeeder.php

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Course;

class CourseSeeder extends Seeder
{
    public function run(): void
    {
        Course::create([
            'title' => 'Szoftverfejlesztési alapok',
            'description' => 'Alapvető programozási fogalmak és minták.',
        ]);

        Course::create([
            'title' => 'REST API fejlesztés',
            'description' => 'API-k tervezése és készítése Laravelben.',
        ]);

        Course::create([
            'title' => 'Fullstack webfejlesztés',
            'description' => 'Backend és frontend alapok.',
        ]);
    }
}
```

#### database/seeders/EnrollmentSeeder.php

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;
use App\Models\Course;
use App\Models\Enrollment;

class EnrollmentSeeder extends Seeder
{
    public function run(): void
    {
        $students = User::where('role', 'student')->take(2)->get();
        $courses = Course::all();

        // User 1: első két kurzus
        Enrollment::create([
            'user_id' => $students[0]->id,
            'course_id' => $courses[0]->id,
            'enrolled_at' => now(),
            'completed_at' => now(),  // completed
        ]);

        Enrollment::create([
            'user_id' => $students[0]->id,
            'course_id' => $courses[1]->id,
            'enrolled_at' => now(),
            'completed_at' => null,
        ]);

        // User 2: második két kurzus
        Enrollment::create([
            'user_id' => $students[1]->id,
            'course_id' => $courses[0]->id,
            'enrolled_at' => now(),
            'completed_at' => now(), // completed
        ]);

        Enrollment::create([
            'user_id' => $students[1]->id,
            'course_id' => $courses[2]->id,
            'enrolled_at' => now(),
            'completed_at' => null,
        ]);
    }
}
```

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
            CourseSeeder::class,
            EnrollmentSeeder::class,
        ]);
    }
}
```

### 3. Seeding Futtatása

```powershell
php artisan db:seed
```

---

## IV. Modul - Controller-ek és API Végpontok

### 1. Controller-ek Létrehozása

```powershell
php artisan make:controller AuthController
php artisan make:controller UserController
php artisan make:controller CourseController
```

### 2. AuthController Implementálása

#### app/Http/Controllers/AuthController.php

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        try {
            $request->validate([
                'name' => 'required|string|max:255',
                'email' => 'required|email|unique:users,email',
                'password' => 'required|string|confirmed|min:8',
            ]);
        } catch (ValidationException $e) {
            return response()->json([
                'message' => 'Failed to register user',
                'errors' => $e->errors()
            ], 422);
        }

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
            'role' => 'student',
        ]);

        return response()->json([
            'message' => 'User created successfully',
            'user' => [
                'id' => $user->id,
                'name' => $user->name,
                'email' => $user->email,
                'role' => $user->role,
            ],
        ], 201);
    }

    public function login(Request $request)
    {
        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            return response()->json(['message' => 'Invalid email or password'], 401);
        }

        $token = $user->createToken('api-token')->plainTextToken;

        return response()->json([
            'message' => 'Login successful',
            'user' => [
                'id' => $user->id,
                'name' => $user->name,
                'email' => $user->email,
                'role' => $user->role,
            ],
            'access' => [
                'token' => $token,
                'token_type' => 'Bearer'
            ]
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->tokens()->delete();
        return response()->json(['message' => 'Logout successful']);
    }
}
```

### 3. UserController Implementálása

#### app/Http/Controllers/UserController.php

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Validation\ValidationException;

class UserController extends Controller
{
    /**
     * GET /users/me
     * A bejelentkezett felhasználó adatainak lekérése.
     */
    public function me(Request $request)
    {
        $user = $request->user();

        return response()->json([
            'user' => [
                'id'    => $user->id,
                'name'  => $user->name,
                'email' => $user->email,
                'role'  => $user->role,
            ],
            'stats' => [
                'enrolledCourses'  => $user->enrollments()->count(),
                'completedCourses' => $user->enrollments()->whereNotNull('completed_at')->count(),
            ]
        ], 200);
    }

    /**
     * PUT /users/me
     * A bejelentkezett felhasználó adatainak frissítése.
     */
    public function updateMe(Request $request)
    {
        $user = $request->user();

        try {
            $request->validate([
                'name'   => 'sometimes|string|max:255',
                'email'  => 'sometimes|email|unique:users,email,' . $user->id,
                'password' => 'sometimes|string|confirmed|min:8',
            ]);
        } catch (ValidationException $e) {
            return response()->json([
                'message' => 'Profile update failed',
                'errors' => $e->errors()
            ], 422);
        }

        if ($request->name) {
            $user->name = $request->name;
        }
        if ($request->email) {
            $user->email = $request->email;
        }
        if ($request->password) {
            $user->password = bcrypt($request->password);
        }

        $user->save();

        return response()->json([
            'message' => 'Profile updated successfully',
            'user' => [
                'id'    => $user->id,
                'name'  => $user->name,
                'email' => $user->email,
                'role'  => $user->role,
            ],
        ]);
    }

    /**
     * ADMIN ONLY - GET /users
     * Összes felhasználó listázása.
     */
    public function index(Request $request)
    {
        if (!$request->user()->isAdmin()) {
            return response()->json(['message' => 'Admin access required'], 403);
        }

        $users = User::all()->map(function ($user) {
            return [
                'user' => [
                    'id' => $user->id,
                    'name' => $user->name,
                    'email' => $user->email,
                    'role' => $user->role,
                ],
                'stats' => [
                    'enrolledCourses'  => $user->enrollments()->count(),
                    'completedCourses' => $user->enrollments()->whereNotNull('completed_at')->count(),
                ]
            ];
        });

        return response()->json([
            'data' => $users
        ]);
    }

    /**
     * ADMIN ONLY - GET /users/{id}
     * Felhasználó lekérése ID alapján.
     */
    public function show(Request $request, $id)
    {
        if (!$request->user()->isAdmin()) {
            return response()->json(['message' => 'Admin access required'], 403);
        }

        $user = User::find($id);

        if (!$user) {
            return response()->json(['message' => 'User not found'], 404);
        }

        return response()->json([
            'user' => [
                'id'    => $user->id,
                'name'  => $user->name,
                'email' => $user->email,
                'role'  => $user->role,
            ],
            'stats' => [
                'enrolledCourses'  => $user->enrollments()->count(),
                'completedCourses' => $user->enrollments()->whereNotNull('completed_at')->count(),
            ]
        ]);
    }

    /**
     * ADMIN ONLY - DELETE /users/{id}
     * Soft delete felhasználó.
     */
    public function destroy(Request $request, $id)
    {
        if (!$request->user()->isAdmin()) {
            return response()->json(['message' => 'Admin access required'], 403);
        }

        $user = User::find($id);

        if (!$user) {
            return response()->json(['message' => 'User not found'], 404);
        }

        $user->delete();

        return response()->json(['message' => 'User deleted successfully']);
    }
}
```

### 4. CourseController Implementálása

#### app/Http/Controllers/CourseController.php

```php
<?php

namespace App\Http\Controllers;

use App\Models\Course;
use App\Models\Enrollment;
use Illuminate\Http\Request;

class CourseController extends Controller
{
    public function index(Request $request)
    {
        $courses = Course::select('title', 'description')->get();

        return response()->json([
            'courses' => $courses
        ]);
    }

    public function show(Course $course)
    {
        $students = $course->users()->select('name', 'email')->withPivot('completed_at')->get()->map(function ($user) {
            return [
                'name' => $user->name,
                'email' => $user->email,
                'completed' => !is_null($user->pivot->completed_at)
            ];
        });

        return response()->json([
            'course' => [
                'title' => $course->title,
                'description' => $course->description
            ],
            'students' => $students
        ]);
    }

    public function enroll(Course $course, Request $request)
    {
        $user = $request->user();

        if ($user->courses()->where('course_id', $course->id)->exists()) {
            return response()->json(['message' => 'Already enrolled in this course'], 409);
        }

        $user->courses()->attach($course->id, ['enrolled_at' => now()]);

        return response()->json(['message' => 'Successfully enrolled in course']);
    }

    public function complete(Course $course, Request $request)
    {
        $user = $request->user();
        $enrollment = Enrollment::where('user_id', $user->id)
            ->where('course_id', $course->id)
            ->first();

        if (!$enrollment) {
            return response()->json(['message' => 'Not enrolled in this course'], 403);
        }

        if ($enrollment->completed_at) {
            return response()->json(['message' => 'Course already completed'], 409);
        }

        $enrollment->update(['completed_at' => now()]);

        return response()->json(['message' => 'Course completed']);
    }
}
```

### 5. API Routes Konfiguráció

#### routes/api.php

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\AuthController;
use App\Http\Controllers\UserController;
use App\Http\Controllers\CourseController;

// Public endpoints
Route::get('/ping', function () { return response()->json(['message' => 'API works!']); });
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

// Authenticated endpoints
Route::middleware('auth:sanctum')->group(function () {
    // Auth
    Route::post('/logout', [AuthController::class, 'logout']);
    
    // User management
    Route::get('/users/me', [UserController::class, 'me']);
    Route::put('/users/me', [UserController::class, 'updateMe']);

    // Admin only
    Route::get('/users', [UserController::class, 'index']);
    Route::get('/users/{id}', [UserController::class, 'show']);
    Route::delete('/users/{id}', [UserController::class, 'destroy']);

    // Course management
    Route::get('/courses', [CourseController::class, 'index']);
    Route::get('/courses/{course}', [CourseController::class, 'show']);
    Route::post('/courses/{course}/enroll', [CourseController::class, 'enroll']);
    Route::patch('/courses/{course}/completed', [CourseController::class, 'complete']);
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
- `401 Unauthorized` - Érvénytelen token
- `403 Forbidden` - Nincs jogosultság
- `404 Not Found` - Erőforrás nem található
- `409 Conflict` - Ütköző művelet
- `422 Unprocessable Entity` - Validációs hiba
- `503 Service Unavailable` - Szolgáltatás nem elérhető

### Nem védett végpontok

#### GET /ping
Teszt végpont az API elérhetőségének ellenőrzésére.

**Válasz:** 200 OK
```json
{
  "message": "API works!"
}
```

#### POST /register
Új felhasználó regisztrálása.

**Kérés törzse:**
```json
{
  "name": "mozso",
  "email": "mozso@moriczref.hu",
  "password": "Jelszo_2025",
  "password_confirmation": "Jelszo_2025"
}
```

**Válasz:** 201 Created
```json
{
  "message": "User created successfully",
  "user": {
    "id": 13,
    "name": "mozso",
    "email": "mozso@moriczref.hu",
    "role": "student"
  }
}
```

**Hiba:** 422 Unprocessable Entity
```json
{
  "message": "Failed to register user",
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
  "email": "mozso@moriczref.hu",
  "password": "Jelszo_2025"
}
```

**Válasz:** 200 OK
```json
{
  "message": "Login successful",
  "user": {
    "id": 13,
    "name": "mozso",
    "email": "mozso@moriczref.hu",
    "role": "student"
  },
  "access": {
    "token": "2|7Fbr79b5zn8RxMfOqfdzZ31SnGWvgDidjahbdRfL2a98cfd8",
    "token_type": "Bearer"
  }
}
```

**Hiba:** 401 Unauthorized
```json
{
  "message": "Invalid email or password"
}
```

### Védett végpontok

#### POST /logout
Kijelentkezés és token törlése.

**Válasz:** 200 OK
```json
{
  "message": "Logout successful"
}
```

#### GET /users/me
Saját felhasználói profil és statisztikák lekérése.

**Válasz:** 200 OK
```json
{
  "user": {
    "id": 1,
    "name": "admin",
    "email": "admin@example.com",
    "role": "admin"
  },
  "stats": {
    "enrolledCourses": 10,
    "completedCourses": 11
  }
}
```

#### PUT /users/me
Saját felhasználói adatok frissítése.

**Kérés törzse:**
```json
{
  "name": "Új Név",
  "email": "ujemail@example.com",
  "password": "ÚjJelszo_2025",
  "password_confirmation": "ÚjJelszo_2025"
}
```

**Válasz:** 200 OK
```json
{
  "message": "Profile updated successfully",
  "user": {
    "id": 5,
    "name": "Új Név",
    "email": "ujemail@example.com",
    "role": "admin"
  }
}
```

### Admin végpontok

#### GET /users
Összes felhasználó listázása (csak admin).

**Válasz:** 200 OK
```json
{
  "data": [
    {
      "user": {
        "id": 1,
        "name": "admin",
        "email": "admin@example.com",
        "role": "admin"
      },
      "stats": {
        "enrolledCourses": 10,
        "completedCourses": 6
      }
    },
    {
      "user": {
        "id": 2,
        "name": "Aranka Török",
        "email": "atorok@example.com",
        "role": "student"
      },
      "stats": {
        "enrolledCourses": 2,
        "completedCourses": 1
      }
    }
  ]
}
```

**Hiba:** 403 Forbidden
```json
{
  "message": "Admin access required"
}
```

#### GET /users/{id}
Felhasználó adatainak lekérése ID alapján (csak admin).

**Válasz:** 200 OK
```json
{
  "user": {
    "id": 5,
    "name": "Eva Rodriguez",
    "email": "eva@example.com",
    "role": "student"
  },
  "stats": {
    "enrolledCourses": 3,
    "completedCourses": 13
  }
}
```

#### DELETE /users/{id}
Felhasználó törlése (soft delete, csak admin).

**Válasz:** 200 OK
```json
{
  "message": "User deleted successfully"
}
```

### Kurzus végpontok

#### GET /courses
Összes elérhető kurzus listázása.

**Válasz:** 200 OK
```json
{
  "courses": [
    {
      "title": "Szoftverfejlesztési alapok",
      "description": "Alapvető programozási fogalmak és minták."
    },
    {
      "title": "REST API fejlesztés",
      "description": "API-k tervezése és készítése Laravelben."
    }
  ]
}
```

#### GET /courses/{id}
Kurzus részleteinek lekérése.

**Válasz:** 200 OK
```json
{
  "course": {
    "title": "Szoftverfejlesztési alapok",
    "description": "Alapvető programozási fogalmak és minták."
  },
  "students": [
    {
      "name": "Barnabás Török",
      "email": "btorok@example.net",
      "completed": false
    },
    {
      "name": "Andrea Török",
      "email": "atorok@example.net",
      "completed": true
    }
  ]
}
```

#### POST /courses/{id}/enroll
Beiratkozás kurzusra.

**Válasz:** 200 OK
```json
{
  "message": "Successfully enrolled in course"
}
```

**Hiba:** 409 Conflict
```json
{
  "message": "Already enrolled in this course"
}
```

#### PATCH /courses/{id}/completed
Kurzus befejezettként való jelölése.

**Válasz:** 200 OK
```json
{
  "message": "Course completed"
}
```

**Hibák:**
```json
{
  "message": "Not enrolled in this course"
}
```

```json
{
  "message": "Course already completed"
}
```

---

## VI. Tesztelés

### 1. Feature Tesztek Létrehozása

```powershell
php artisan make:test AuthTest
php artisan make:test UserTest
php artisan make:test CourseTest
```

### 2. AuthTest Implementálása

#### tests/Feature/AuthTest.php

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class AuthTest extends TestCase
{
    use RefreshDatabase;

    public function test_ping_endpoint_returns_ok()
    {
        $response = $this->getJson('/api/ping');
        $response->assertStatus(200)
                ->assertJson(['message' => 'API works!']);
    }

    public function test_register_creates_user()
    {
        $payload = [
            'name' => 'Teszt Elek',
            'email' => 'teszt@example.com',
            'password' => 'Jelszo_2025',
            'password_confirmation' => 'Jelszo_2025'
        ];

        $response = $this->postJson('/api/register', $payload);
        $response->assertStatus(201)
                ->assertJsonStructure(['message', 'user' => ['id', 'name', 'email', 'role']]);
        
        $this->assertDatabaseHas('users', [
            'email' => 'teszt@example.com',
        ]);
    }

    public function test_login_with_valid_credentials()
    {
        $password = 'Jelszo_2025';
        $user = User::factory()->create([
            'email' => 'validuser@example.com',
            'password' => Hash::make($password),
        ]);

        $response = $this->postJson('/api/login', [
            'email' => 'validuser@example.com',
            'password' => $password,
        ]);

        $response->assertStatus(200)
                 ->assertJsonStructure(['message', 'user' => ['id', 'name', 'email', 'role'], 'access' => ['token', 'token_type']]);

        $this->assertDatabaseHas('personal_access_tokens', [
            'tokenable_id' => $user->id,
        ]);
    }

    public function test_login_with_invalid_credentials()
    {
        $user = User::factory()->create([
            'email' => 'existing@example.com',
            'password' => Hash::make('CorrectPassword'),
        ]);

        $response = $this->postJson('/api/login', [
            'email' => 'existing@example.com',
            'password' => 'wrongpass'
        ]);

        $response->assertStatus(401)
                 ->assertJson(['message' => 'Invalid email or password']);
    }
}
```

### 3. Tesztek Futtatása

```powershell
php artisan test
```

---

## VII. Végpontok Összefoglalója

| HTTP Metódus | Útvonal | Jogosultság | Státuszkódok | Rövid Leírás |
|--------------|---------|-------------|--------------|--------------|
| GET | /ping | Nyilvános | 200 OK | API teszteléshez |
| POST | /register | Nyilvános | 201 Created, 422 Unprocessable Entity | Új felhasználó regisztrációja |
| POST | /login | Nyilvános | 200 OK, 401 Unauthorized | Bejelentkezés |
| POST | /logout | Hitelesített | 200 OK, 401 Unauthorized | Kijelentkezés |
| GET | /users/me | Hitelesített | 200 OK, 401 Unauthorized | Saját profil lekérése |
| PUT | /users/me | Hitelesített | 200 OK, 422 Unprocessable Entity, 401 Unauthorized | Saját profil módosítása |
| GET | /users | Admin | 200 OK, 403 Forbidden | Összes felhasználó listázása |
| GET | /users/{id} | Admin | 200 OK, 403 Forbidden, 404 Not Found | Felhasználó lekérése |
| DELETE | /users/{id} | Admin | 200 OK, 404 Not Found, 401 Unauthorized | Felhasználó törlése |
| GET | /courses | Hitelesített | 200 OK, 401 Unauthorized | Kurzusok listázása |
| GET | /courses/{id} | Hitelesített | 200 OK, 404 Not Found, 401 Unauthorized | Kurzus részletei |
| POST | /courses/{id}/enroll | Hitelesített | 200 OK, 409 Conflict, 404 Not Found | Beiratkozás kurzusra |
| PATCH | /courses/{id}/completed | Hitelesített | 200 OK, 403 Forbidden, 409 Conflict | Kurzus teljesítése |

---

## VIII. Postman Teszt Kollekció

### Környezeti Változók
```json
{
  "base_url": "http://127.0.0.1:8000/api",
  "token": ""
}
```

### Teszt Sorrend
1. GET /ping
2. POST /register
3. POST /login (token mentése)
4. GET /users/me
5. GET /courses
6. POST /courses/1/enroll
7. PATCH /courses/1/completed
8. Admin login
9. GET /users (admin)
10. POST /logout

---

## IX. Telepítési Útmutató

### Rendszerkövetelmények
- PHP 8.1+
- MySQL 8.0+
- Composer
- XAMPP vagy Laravel Valet

### Telepítési Lépések

1. **Projekt klónozása/letöltése**
```powershell
cd C:\xampp\htdocs
composer create-project laravel/laravel --prefer-dist learningPlatformBearer
cd learningPlatformBearer
```

2. **Függőségek telepítése**
```powershell
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan install:api
```

3. **Adatbázis konfiguráció**
```powershell
# phpMyAdmin-ban hozd létre: learning_platform adatbázist
# .env fájl módosítása
```

4. **Migráció és seeding**
```powershell
php artisan migrate
php artisan db:seed
```

5. **Szerver indítása**
```powershell
php artisan serve
# vagy XAMPP használata
```

6. **Tesztelés**
```powershell
php artisan test
```

### Fejlesztői Fiókok
- **Admin:** admin@example.com / admin
- **Student:** bármelyik generált user / Jelszo_2025

---

## X. Hibaelhárítás

### Gyakori Hibák

1. **Token hibák**
   - Ellenőrizd a `config/sanctum.php` beállításokat
   - Győződj meg róla, hogy a token helyesen van átadva

2. **CORS hibák**
   - Telepítsd a `laravel/sanctum` csomagot
   - Ellenőrizd a `config/cors.php` beállításokat

3. **Adatbázis hibák**
   - Ellenőrizd az `.env` fájl adatbázis beállításait
   - Futtasd újra a migrációkat: `php artisan migrate:fresh --seed`

4. **Validációs hibák**
   - Ellenőrizd a kérések formátumát
   - Győződj meg róla, hogy minden kötelező mező ki van töltve

---

## Konklúzió

Ez a tanulási platform API teljes körű megoldást nyújt a kurzusok kezelésére, felhasználók beiratkozására és teljesítések nyomon követésére. Az API REST elveket követ, biztonságos autentikációt használ, és átfogó tesztelést biztosít. A dokumentáció minden szükséges információt tartalmaz a fejlesztéshez és deployment-hez.