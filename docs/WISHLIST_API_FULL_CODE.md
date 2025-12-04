# Wishlist API – Teljes Projekt Kód és Lépések

Ez a dokumentáció tartalmazza a teljes Laravel Wishlist API projekt minden lényeges kódrészletét, modellt, migrációt, seedert, middleware-t, route-ot és kontrollert. Ha ezt követed, nulláról fel tudod építeni a rendszert.

---

## 1. Modellek

### app/Models/Product.php
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

### app/Models/Wishlist.php
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

### app/Models/User.php
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

---

## 2. Migrációk

### database/migrations/0001_01_01_000000_create_users_table.php
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

### database/migrations/2025_12_01_081600_create_products_table.php
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

### database/migrations/2025_12_01_081707_create_wishlists_table.php
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

---

## 3. Middleware

### app/Http/Middleware/IsAdmin.php
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

---

## 4. Route-ok

### routes/api.php
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

## 5. Kontrollerek

### app/Http/Controllers/Api/AuthController.php
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
```

### app/Http/Controllers/Api/ProductController.php
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

### app/Http/Controllers/Api/WishlistController.php
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

### app/Http/Controllers/Api/UserController.php
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

---

## 6. Seederek

### database/seeders/UserSeeder.php
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
    }
}
```

### database/seeders/ProductSeeder.php
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
            [
                'name' => 'Apple Watch Series 9',
                'category' => 'Wearables',
                'price' => 399.99,
                'stock' => 40,
            ],
            [
                'name' => 'iPad Air',
                'category' => 'Tablets',
                'price' => 599.99,
                'stock' => 20,
            ],
            [
                'name' => 'Mechanical Keyboard',
                'category' => 'Accessories',
                'price' => 129.99,
                'stock' => 60,
            ],
            [
                'name' => 'Gaming Mouse Logitech',
                'category' => 'Accessories',
                'price' => 79.99,
                'stock' => 100,
            ],
            [
                'name' => '4K Monitor 27"',
                'category' => 'Electronics',
                'price' => 449.99,
                'stock' => 12,
            ],
            [
                'name' => 'External SSD 1TB',
                'category' => 'Storage',
                'price' => 99.99,
                'stock' => 75,
            ],
        ];

        foreach ($products as $product) {
            Product::create($product);
        }
    }
}
```

### database/seeders/WishlistSeeder.php
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Wishlist;
use App\Models\User;
use App\Models\Product;

class WishlistSeeder extends Seeder
{
    public function run(): void
    {
        // John (user_id: 2) kedvencei
        $john = User::where('email', 'john@example.com')->first();
        if ($john) {
            Wishlist::create([
                'user_id' => $john->id,
                'product_id' => 1,
                'added_at' => now()->subDays(5),
            ]);
            Wishlist::create([
                'user_id' => $john->id,
                'product_id' => 2,
                'added_at' => now()->subDays(3),
            ]);
            Wishlist::create([
                'user_id' => $john->id,
                'product_id' => 4,
                'added_at' => now()->subDays(1),
            ]);
            Wishlist::create([
                'user_id' => $john->id,
                'product_id' => 7,
                'added_at' => now(),
            ]);
        }
        $jane = User::where('email', 'jane@example.com')->first();
        if ($jane) {
            Wishlist::create([
                'user_id' => $jane->id,
                'product_id' => 5,
                'added_at' => now()->subDays(7),
            ]);
            Wishlist::create([
                'user_id' => $jane->id,
                'product_id' => 6,
                'added_at' => now()->subDays(4),
            ]);
            Wishlist::create([
                'user_id' => $jane->id,
                'product_id' => 9,
                'added_at' => now()->subDays(2),
            ]);
            Wishlist::create([
                'user_id' => $jane->id,
                'product_id' => 3,
                'added_at' => now()->subHours(12),
            ]);
            Wishlist::create([
                'user_id' => $jane->id,
                'product_id' => 10,
                'added_at' => now()->subHours(3),
            ]);
        }
        echo "Wishlist seeder completed successfully!\n";
        echo "- John has " . ($john ? $john->wishlists->count() : 0) . " items in wishlist\n";
        echo "- Jane has " . ($jane ? $jane->wishlists->count() : 0) . " items in wishlist\n";
    }
}
```

### database/seeders/DatabaseSeeder.php
```php
<?php

namespace Database\Seeders;

use App\Models\User;
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

---

## 7. További lépések
- .env beállítása
- composer install, npm install
- php artisan migrate --seed
- php artisan key:generate
- php artisan serve
- Postman collection importálása
- API_TEST_ENDPOINTS.md használata

---

Ez a fájl tartalmazza a teljes projekt minden lényeges kódját. Minden kódrészletet illessz be a megfelelő helyre a projektben!

Ha szükséges, kérj a kontrollerek, seederek teljes kódját is (a fenti helyeken a kódok hosszúsága miatt csak a főbb részeket mutattam, de minden részletet beilleszthetsz).