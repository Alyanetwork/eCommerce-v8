Proje Dökümantasyonu
İçindekiler
Proje Kurulumu
Veritabanı Yapısı
Model ve İlişki Tanımlamaları
Middleware ve Yetki Kontrolleri
Mağaza Seçimi ve Yönetim Paneli
Veri Filtreleme ve Controller Yapısı
Tema ve Ayarlar Yönetimi
Raporlama ve İstatistikler
Çoklu Mağaza Entegrasyonu İçin Eklenen Örnek Kodlar
1. Proje Kurulumu
Gereksinimler
PHP 7.4 veya üzeri
Composer
MySQL veya PostgreSQL
Laravel 8 veya üzeri
Adımlar
Depoyu klonlayın:

bash
Kodu kopyala
git clone https://github.com/kullanici/proje.git
cd proje
Bağımlılıkları yükleyin:

bash
Kodu kopyala
composer install
Ortam dosyasını yapılandırın:

bash
Kodu kopyala
cp .env.example .env
php artisan key:generate
Veritabanı bilgilerini doldurun ve tabloları oluşturun:

bash
Kodu kopyala
php artisan migrate
php artisan db:seed
Sunucuyu başlatın:

bash
Kodu kopyala
php artisan serve
2. Veritabanı Yapısı
stores Tablosu
Her mağaza için bir kayıt barındıran stores tablosu:

php
Kodu kopyala
Schema::create('stores', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('slug')->unique();
    $table->unsignedBigInteger('owner_id');
    $table->foreign('owner_id')->references('id')->on('users')->onDelete('cascade');
    $table->timestamps();
});
İlişkilendirme Sütunları
Her mağazaya ait ürün, sipariş gibi verilerin saklanabilmesi için diğer tablolara store_id sütunu eklenmiştir. Örneğin:

php
Kodu kopyala
Schema::table('products', function (Blueprint $table) {
    $table->unsignedBigInteger('store_id');
    $table->foreign('store_id')->references('id')->on('stores')->onDelete('cascade');
});
3. Model ve İlişki Tanımlamaları
Store Modeli
Store.php modelinde mağaza bilgilerini ve ilişkilerini tanımlayın:

php
Kodu kopyala
class Store extends Model
{
    protected $fillable = ['name', 'slug', 'owner_id'];

    public function products()
    {
        return $this->hasMany(Product::class);
    }

    public function orders()
    {
        return $this->hasMany(Order::class);
    }

    public function owner()
    {
        return $this->belongsTo(User::class, 'owner_id');
    }
}
Ürün ve Sipariş Modelleri
Product.php ve Order.php modellerinde store_id ilişkisi ekleyin:

php
Kodu kopyala
class Product extends Model
{
    public function store()
    {
        return $this->belongsTo(Store::class);
    }
}
4. Middleware ve Yetki Kontrolleri
Middleware: StoreAccessMiddleware
Kullanıcıların yalnızca erişim yetkisi olan mağazalara erişebilmesi için middleware:

php
Kodu kopyala
class StoreAccessMiddleware
{
    public function handle($request, Closure $next)
    {
        $storeId = session('store_id');
        if (!$request->user()->stores()->where('id', $storeId)->exists()) {
            return redirect()->route('access.denied');
        }
        return $next($request);
    }
}
5. Mağaza Seçimi ve Yönetim Paneli
Mağaza seçimi yapıldığında oturuma store_id kaydedin ve yönetim panelinde aktif mağazaya göre verileri filtreleyin.

Mağaza Seçme Örneği:
php
Kodu kopyala
public function switchStore($storeId)
{
    session(['store_id' => $storeId]);
    return redirect()->route('dashboard');
}
6. Veri Filtreleme ve Controller Yapısı
Controller'larda verileri aktif mağaza bazında filtrelemek için store_id filtresi kullanın:

php
Kodu kopyala
class ProductController extends Controller
{
    public function index()
    {
        $products = Product::where('store_id', session('store_id'))->get();
        return view('products.index', compact('products'));
    }
}
7. Tema ve Ayarlar Yönetimi
Her mağaza için tema ve ayarları depolamak için store_settings tablosu oluşturun. Örneğin:

php
Kodu kopyala
Schema::create('store_settings', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('store_id');
    $table->string('key');
    $table->string('value');
    $table->timestamps();
});
Mağaza bazında ayarları ve temaları dinamik olarak yükleyin.

8. Raporlama ve İstatistikler
Belirli bir mağazaya ait sipariş, satış ve diğer verileri store_id filtresi ile raporlayın:

php
Kodu kopyala
$salesData = Order::where('store_id', session('store_id'))->where('status', 'completed')->sum('total');
9. Çoklu Mağaza Entegrasyonu İçin Eklenen Örnek Kodlar
Aşağıda çoklu mağaza entegrasyonu için projeye eklenen bazı örnek kodlar yer almaktadır:

Mağaza Ekleme
Bir mağaza eklemek için örnek bir route ve controller metodu:

php
Kodu kopyala
Route::post('/store', [StoreController::class, 'create']);

class StoreController extends Controller
{
    public function create(Request $request)
    {
        $store = Store::create([
            'name' => $request->input('name'),
            'slug' => Str::slug($request->input('name')),
            'owner_id' => auth()->id()
        ]);
        return redirect()->route('stores.index');
    }
}
Mağaza Güncelleme
Bir mağaza güncelleme işlemi:

php
Kodu kopyala
public function update(Request $request, $id)
{
    $store = Store::findOrFail($id);
    $store->update($request->only('name', 'slug'));
    return redirect()->route('stores.index');
}
Bu dökümantasyon ile çoklu mağaza entegrasyonuna sahip projeyi adım adım geliştirebilir, yapılandırabilir ve yönetebilirsiniz.
