# 📁 EShop Microservices - Klasör Yapısı Dokümantasyonu

> Bu dokümanda `eshop-microservices/` dizinindeki tüm klasörler ve içerikleri detaylı olarak açıklanmaktadır.

---

## 🎯 Genel Bakış

```
eshop-microservices/
├── 🚀 AppHost/              → Orkestrasyon & Yönetim
├── 🛒 Basket/               → Sepet Mikroservisi
├── 📦 Catalog/              → Ürün Katalogu Mikroservisi
├── 📋 Ordering/             → Sipariş Mikroservisi
├── ⚙️ ServiceDefaults/      → Ortak Yapılandırmalar
├── 📨 Shared.Messaging/     → Mesajlaşma Kütüphanesi
├── 🌐 WebApp/               → Frontend Uygulaması
└── 🔀 YarpApiGateway/       → API Gateway
```

---

## 🚀 1. AppHost/

**.NET Aspire Orkestrasyon Projesi**

Bu klasör, tüm mikroservisleri ve altyapı bileşenlerini yöneten ana orkestratör projesidir.

### 📄 İçerik

| Dosya | Açıklama |
|-------|----------|
| `Program.cs` | Tüm servislerin ve bağımlılıkların tanımlandığı ana dosya |
| `AppHost.csproj` | Proje yapılandırma dosyası |
| `appsettings.json` | Uygulama ayarları |

### ✨ Temel Görevleri

- 🐘 **PostgreSQL** veritabanı sunucusunu başlatır (`catalogdb`, `basketdb`)
- 🔴 **Redis** cache sunucusunu başlatır
- 🗄️ **SQL Server** veritabanı sunucusunu başlatır (`orderdb`)
- 🐰 **RabbitMQ** mesaj kuyruğunu başlatır
- 🔗 Servislerin birbirine olan **bağımlılıklarını** yönetir
- ⏳ Servislerin **başlatma sırasını** (`WaitFor`) kontrol eder
- 📊 **Aspire Dashboard** ile izleme ve yönetim sağlar

---

## 🛒 2. Basket/

**Alışveriş Sepeti Mikroservisi**

Kullanıcıların alışveriş sepetlerini yöneten mikroservis.

### 📄 İçerik

```
Basket/
├── ApiClients/
│   └── CatalogApiClient.cs      → Catalog servisine HTTP istekleri
├── Data/
│   ├── BasketDbContext.cs       → EF Core veritabanı context'i
│   ├── Extensions.cs            → Migration extension metodları
│   ├── Migrations/              → Veritabanı migration dosyaları
│   └── Processors/
│       └── OutboxProcessor.cs   → Outbox mesajlarını işleyen BackgroundService
├── Endpoints/
│   └── BasketEndpoints.cs       → REST API endpoint tanımları
├── EventHandlers/
│   └── ProductPriceChangedEventHandler.cs → Fiyat değişikliği event handler
├── Models/
│   ├── BasketCheckout.cs        → Checkout işlemi modeli
│   ├── OutboxMessage.cs         → Outbox mesaj modeli
│   ├── ShoppingCart.cs          → Alışveriş sepeti modeli
│   └── ShoppingCartItem.cs      → Sepet ürün modeli
├── Services/
│   └── BasketService.cs         → İş mantığı servisi
└── Program.cs                   → Uygulama başlangıç noktası
```

### ✨ Temel Görevleri

- 🛒 Kullanıcı sepetlerini **oluşturma, güncelleme, silme**
- 💾 **PostgreSQL** veritabanında sepet verilerini saklama
- ⚡ **Redis HybridCache** ile hızlı erişim sağlama
- 📤 **Outbox Pattern** ile güvenilir mesaj gönderimi
- 📨 `BasketCheckoutIntegrationEvent` yayınlama
- 📥 `ProductPriceChangedIntegrationEvent` dinleme ve sepet fiyatlarını güncelleme

### 🔌 API Endpoint'leri

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/basket/{userName}` | Kullanıcının sepetini getir |
| POST | `/basket/` | Sepeti güncelle/oluştur |
| POST | `/basket/checkout` | Checkout işlemi başlat |
| DELETE | `/basket/{userName}` | Sepeti sil |

---

## 📦 3. Catalog/

**Ürün Katalogu Mikroservisi**

Ürün bilgilerini yöneten mikroservis.

### 📄 İçerik

```
Catalog/
├── Data/
│   ├── CatalogDbContext.cs      → EF Core veritabanı context'i
│   ├── Extensions.cs            → Migration extension metodları
│   └── Migrations/              → Veritabanı migration dosyaları
├── Endpoints/
│   └── ProductEndpoints.cs      → REST API endpoint tanımları
├── Models/
│   └── Product.cs               → Ürün modeli
├── Services/
│   └── ProductService.cs        → İş mantığı servisi
└── Program.cs                   → Uygulama başlangıç noktası
```

### ✨ Temel Görevleri

- 📦 Ürünleri **listeleme, ekleme, güncelleme, silme**
- 🔍 Ürün **arama** fonksiyonu
- 💾 **PostgreSQL** veritabanında ürün verilerini saklama
- 📨 Fiyat değişikliğinde `ProductPriceChangedIntegrationEvent` yayınlama

### 🔌 API Endpoint'leri

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/products` | Tüm ürünleri listele |
| GET | `/products/{id}` | Ürün detayı getir |
| POST | `/products` | Yeni ürün ekle |
| PUT | `/products/{id}` | Ürün güncelle |
| DELETE | `/products/{id}` | Ürün sil |
| GET | `/products/search/{query}` | Ürün ara |

---

## 📋 4. Ordering/

**Sipariş Yönetimi Mikroservisi**

Siparişleri yöneten mikroservis.

### 📄 İçerik

```
Ordering/
├── Data/
│   ├── OrderDbContext.cs        → EF Core veritabanı context'i
│   ├── Extensions.cs            → Migration extension metodları
│   └── Migrations/              → Veritabanı migration dosyaları
├── Endpoints/
│   └── OrderEndpoints.cs        → REST API endpoint tanımları
├── EventHandlers/
│   └── BasketCheckoutEventHandler.cs → Checkout event handler
├── Models/
│   └── Order.cs                 → Sipariş modeli
├── Services/
│   └── OrderService.cs          → İş mantığı servisi
└── Program.cs                   → Uygulama başlangıç noktası
```

### ✨ Temel Görevleri

- 📋 Siparişleri **listeleme ve oluşturma**
- 💾 **SQL Server** veritabanında sipariş verilerini saklama
- 📥 `BasketCheckoutIntegrationEvent` dinleme
- 🔄 Checkout eventi geldiğinde **otomatik sipariş oluşturma**

### 🔌 API Endpoint'leri

| Metod | Endpoint | Açıklama |
|-------|----------|----------|
| GET | `/orders` | Tüm siparişleri listele |
| GET | `/orders/{userName}` | Kullanıcının siparişlerini getir |

---

## ⚙️ 5. ServiceDefaults/

**Ortak .NET Aspire Yapılandırmaları**

Tüm mikroservislerin ortak kullandığı yapılandırmaları içeren paylaşımlı kütüphane.

### 📄 İçerik

| Dosya | Açıklama |
|-------|----------|
| `Extensions.cs` | Ortak servis yapılandırma extension metodları |
| `ServiceDefaults.csproj` | Proje yapılandırma dosyası |

### ✨ Sağladığı Özellikler

- 📊 **OpenTelemetry** entegrasyonu (Logging, Metrics, Tracing)
- ❤️ **Health Checks** (`/health`, `/alive` endpoint'leri)
- 🔍 **Service Discovery** (servisler arası iletişim)
- 🛡️ **HTTP Resilience** (retry, circuit breaker)
- 📈 **OTLP Exporter** desteği

### 🔧 Kullanımı

```csharp
// Her mikroserviste Program.cs içinde:
builder.AddServiceDefaults();

// HTTP pipeline'da:
app.MapDefaultEndpoints();
```

---

## 📨 6. Shared.Messaging/

**Mikroservisler Arası Mesajlaşma Kütüphanesi**

Tüm mikroservislerin ortak kullandığı mesajlaşma altyapısı ve event tanımları.

### 📄 İçerik

```
Shared.Messaging/
├── Events/
│   ├── IntegrationEvent.cs                    → Temel event sınıfı
│   ├── BasketCheckoutIntegrationEvent.cs      → Checkout eventi
│   └── ProductPriceChangedIntegrationEvent.cs → Fiyat değişikliği eventi
├── Extensions/
│   └── MassTransitExtentions.cs               → MassTransit yapılandırması
└── Shared.Messaging.csproj                    → Proje yapılandırma dosyası
```

### ✨ Sağladığı Özellikler

- 🐰 **MassTransit + RabbitMQ** entegrasyonu
- 📨 **Integration Event** temel sınıfı
- 🔄 Servisler arası **asenkron iletişim**
- 📋 **Consumer** ve **Saga** desteği

### 📩 Tanımlı Event'ler

| Event | Yayınlayan | Dinleyen | Açıklama |
|-------|------------|----------|----------|
| `BasketCheckoutIntegrationEvent` | Basket | Ordering | Checkout yapıldığında sipariş oluştur |
| `ProductPriceChangedIntegrationEvent` | Catalog | Basket | Fiyat değiştiğinde sepetleri güncelle |

---

## 🌐 7. WebApp/

**Blazor Server Frontend Uygulaması**

Kullanıcı arayüzünü sağlayan web uygulaması.

### 📄 İçerik

```
WebApp/
├── ApiClients/
│   └── YarpApiClient.cs         → API Gateway'e HTTP istekleri
├── Components/
│   ├── Layout/
│   │   ├── MainLayout.razor     → Ana sayfa şablonu
│   │   └── NavMenu.razor        → Navigasyon menüsü
│   └── Pages/
│       ├── Home.razor           → Ana sayfa
│       ├── Products.razor       → Ürün listesi sayfası
│       ├── Cart.razor           → Sepet sayfası
│       ├── CheckOut.razor       → Ödeme sayfası
│       └── Orders.razor         → Siparişler sayfası
├── Models/                      → Frontend veri modelleri
├── wwwroot/                     → Statik dosyalar (CSS, JS, resimler)
└── Program.cs                   → Uygulama başlangıç noktası
```

### ✨ Temel Görevleri

- 🏠 **Ana Sayfa** görüntüleme
- 📦 **Ürün Listesi** ve detayları gösterme
- 🛒 **Sepet Yönetimi** (ekleme, çıkarma, güncelleme)
- 💳 **Checkout İşlemi** başlatma
- 📋 **Sipariş Geçmişi** görüntüleme
- 🎨 **Bootstrap** ile modern arayüz

### 📱 Sayfalar

| Sayfa | Route | Açıklama |
|-------|-------|----------|
| Home | `/` | Karşılama sayfası |
| Products | `/products` | Ürün katalogu |
| Cart | `/cart` | Alışveriş sepeti |
| CheckOut | `/checkout` | Ödeme işlemi |
| Orders | `/orders` | Sipariş geçmişi |

---

## 🔀 8. YarpApiGateway/

**YARP Tabanlı API Gateway**

Tüm mikroservislere tek giriş noktası sağlayan reverse proxy.

### 📄 İçerik

| Dosya | Açıklama |
|-------|----------|
| `Program.cs` | YARP yapılandırması ve middleware |
| `appsettings.json` | Route ve cluster tanımları |
| `YarpApiGateway.csproj` | Proje yapılandırma dosyası |

### ✨ Temel Görevleri

- 🔀 **Reverse Proxy** ile istekleri yönlendirme
- ⏱️ **Rate Limiting** (10 saniyede 5 istek limiti)
- 🔍 **Service Discovery** entegrasyonu
- 🛡️ Merkezi **güvenlik** ve **yetkilendirme** noktası

### 🛣️ Route Yapılandırması

| Route | Hedef Servis | Açıklama |
|-------|--------------|----------|
| `/catalog-service/*` | Catalog | Ürün işlemleri |
| `/basket-service/*` | Basket | Sepet işlemleri |
| `/ordering-service/*` | Ordering | Sipariş işlemleri (rate limited) |

---

## 🔄 Veri Akışı

```
┌──────────┐    ┌─────────────────┐    ┌─────────────────────────────────┐
│  WebApp  │───▶│  YarpApiGateway │───▶│  Catalog / Basket / Ordering   │
└──────────┘    └─────────────────┘    └─────────────────────────────────┘
                                                      │
                                                      ▼
                                              ┌───────────────┐
                                              │   RabbitMQ    │
                                              │  (MassTransit)│
                                              └───────────────┘
                                                      │
                              ┌────────────────────────┼────────────────────────┐
                              ▼                        ▼                        ▼
                    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
                    │   PostgreSQL    │    │      Redis      │    │   SQL Server    │
                    │ (catalogdb,     │    │    (cache)      │    │   (orderdb)     │
                    │  basketdb)      │    │                 │    │                 │
                    └─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## 📊 Kullanılan Teknolojiler Özeti

| Kategori | Teknoloji |
|----------|-----------|
| **Framework** | .NET 9, ASP.NET Core Minimal API |
| **Orkestrasyon** | .NET Aspire |
| **Frontend** | Blazor Server, Bootstrap |
| **ORM** | Entity Framework Core |
| **Veritabanları** | PostgreSQL, SQL Server |
| **Cache** | Redis (HybridCache) |
| **Mesajlaşma** | RabbitMQ + MassTransit |
| **API Gateway** | YARP Reverse Proxy |
| **Observability** | OpenTelemetry |

---

*📅 Son güncelleme: Aralık 2024*
