# eShop Microservices - Çalıştırma Rehberi

Bu rehber, .NET Aspire tabanlı eShop mikroservis projesini Fedora Linux üzerinde nasıl çalıştıracağınızı adım adım açıklar.

---

## Proje Mimarisi

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  WebApp (Blazor Server) - E-ticaret web arayüzü                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            API GATEWAY                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  YarpApiGateway - Reverse Proxy + Rate Limiting                     │   │
│  │  /catalog-service/* → Catalog API                                   │   │
│  │  /basket-service/*  → Basket API                                    │   │
│  │  /ordering-service/* → Ordering API                                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MICROSERVICES                                      │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐                   │
│  │   Catalog     │  │    Basket     │  │   Ordering    │                   │
│  │   Minimal API │  │   Minimal API │  │   Minimal API │                   │
│  │   + EF Core   │  │   + EF Core   │  │   + EF Core   │                   │
│  │               │  │   + Redis     │  │   + MassTransit│                   │
│  │               │  │   + Outbox    │  │   Consumer    │                   │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘                   │
│          │                  │                  │                            │
└──────────┼──────────────────┼──────────────────┼────────────────────────────┘
           │                  │                  │
           ▼                  ▼                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        BACKING SERVICES (Docker)                             │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐               │
│  │PostgreSQL │  │   Redis   │  │ SQL Server│  │ RabbitMQ  │               │
│  │ catalogdb │  │   cache   │  │  orderdb  │  │  message  │               │
│  │ basketdb  │  │           │  │           │  │  broker   │               │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘               │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Projeler ve Kullandıkları Teknolojiler

| Proje | Açıklama | Veritabanı | Özellikler |
|-------|----------|------------|------------|
| **AppHost** | Aspire Orchestrator | - | Tüm servisleri yönetir |
| **ServiceDefaults** | Ortak konfigürasyonlar | - | OpenTelemetry, Health Checks |
| **Catalog** | Ürün yönetimi API | PostgreSQL | Minimal API, EF Core |
| **Basket** | Sepet yönetimi API | PostgreSQL + Redis | Outbox Pattern, HybridCache |
| **Ordering** | Sipariş yönetimi API | SQL Server | MassTransit Consumer |
| **YarpApiGateway** | API Gateway | - | YARP Reverse Proxy, Rate Limiting |
| **WebApp** | Web arayüzü | - | Blazor Server |
| **Shared.Messaging** | Ortak mesajlaşma | - | MassTransit, RabbitMQ |

---

## Ön Gereksinimler

### 1. Docker (ZORUNLU)

Aspire, backing servisleri (PostgreSQL, Redis, SQL Server, RabbitMQ) Docker container olarak çalıştırır.

```bash
# Docker durumunu kontrol et
docker ps
```

Eğer Docker çalışmıyorsa:
```bash
# Docker servisini başlat
sudo systemctl start docker

# Docker'ın açılışta otomatik başlamasını sağla
sudo systemctl enable docker
```

### 2. .NET 9 SDK

```bash
# .NET SDK versiyonlarını kontrol et
dotnet --list-sdks
```

Çıktıda `9.0.xxx` görmelisiniz.

### 3. Aspire Workload

```bash
# Aspire workload'ın yüklü olduğunu kontrol et
dotnet workload list
```

Eğer `aspire` listede yoksa:
```bash
dotnet workload install aspire
```

---

## Projeyi Çalıştırma

### Adım 1: Proje Dizinine Git

```bash
cd "/home/kSEN/Desktop/ Projects/Design-Microservices-Architecture-with-Patterns-Principles-mehmetozkaya/Design-Microservices-Architecture-with-Patterns-Principles/11-eshop-microservices-outbox/eshop-microservices"
```

### Adım 2: AppHost'u Başlat

```bash
dotnet run --project AppHost/AppHost.csproj --launch-profile https
```

**Bu komut ne yapar:**
1. Tüm projeleri derler (build)
2. Docker container'larını başlatır:
   - `postgres` - PostgreSQL veritabanı sunucusu
   - `catalogdb` - Catalog servisi için veritabanı
   - `basketdb` - Basket servisi için veritabanı
   - `cache` - Redis cache sunucusu
   - `sqlserver` - SQL Server veritabanı sunucusu
   - `orderdb` - Ordering servisi için veritabanı
   - `rabbitmq` - RabbitMQ mesaj kuyruğu
3. Mikroservisleri başlatır:
   - `catalog` - Ürün API'si
   - `basket` - Sepet API'si
   - `ordering` - Sipariş API'si
   - `yarpapigateway` - API Gateway
   - `webapp` - Web uygulaması
4. Aspire Dashboard'u açar

### Adım 3: Dashboard'a Eriş

Terminal çıktısında şuna benzer bir satır göreceksiniz:
```
Login to the dashboard at https://localhost:17224/login?t=XXXXXX
```

Bu URL'yi tarayıcınızda açın.

> **Not:** HTTPS sertifikası güvenilir değilse tarayıcı uyarı verebilir. 
> "Advanced" → "Proceed to localhost" ile devam edebilirsiniz.

### Adım 4: Web Uygulamasını Aç

Dashboard'da soldaki listede **webapp** satırını bulun ve yanındaki URL'ye tıklayın.

---

## Dashboard Kullanımı

Aspire Dashboard'da şunları görebilirsiniz:

### Resources (Kaynaklar)
- **Yeşil (Running):** Servis çalışıyor
- **Sarı (Starting):** Servis başlatılıyor
- **Kırmızı (Failed/Finished):** Servis durdu veya hata verdi

### Her Servis İçin
- **Console Logs:** Servisin konsol çıktısı
- **Structured Logs:** Yapılandırılmış log kayıtları
- **Traces:** Distributed tracing bilgileri
- **Metrics:** Performans metrikleri

### Yönetim Arayüzleri
Dashboard'dan erişebileceğiniz yönetim arayüzleri:
- **PostgreDB Browser (pgAdmin):** PostgreSQL veritabanı yönetimi
- **Redis Insight:** Redis cache yönetimi
- **RabbitMQ Management:** Mesaj kuyruğu yönetimi

---

## Uygulamayı Durdurma

Terminal'de `Ctrl + C` tuşlarına basın.

Bu işlem:
- Tüm mikroservisleri durdurur
- Docker container'ları **durdurmaz** (persistent olarak ayarlandılar)

### Container'ları Tamamen Durdurmak İçin

```bash
# Çalışan container'ları listele
docker ps

# Tüm Aspire container'larını durdur
docker stop $(docker ps -q --filter "name=postgres" --filter "name=cache" --filter "name=sqlserver" --filter "name=rabbitmq")
```

---

## Sık Karşılaşılan Sorunlar

### Sorun 1: Port Çakışması
**Hata:** `Address already in use`

**Çözüm:** Önceki çalıştırmadan kalan process'leri durdurun:
```bash
pkill -f "dotnet"
```

### Sorun 2: Docker İzin Hatası
**Hata:** `permission denied while trying to connect to the Docker daemon`

**Çözüm:**
```bash
sudo usermod -aG docker $USER
# Ardından oturumu kapatıp açın veya:
newgrp docker
```

### Sorun 3: SQL Server Başlamıyor
SQL Server Linux'ta minimum 2GB RAM gerektirir.

**Kontrol:**
```bash
free -h
```

### Sorun 4: HTTPS Sertifika Sorunu
```bash
dotnet dev-certs https --trust
```

### Sorun 5: Build Hatası
```bash
# Temizle ve yeniden derle
dotnet clean
dotnet build
```

---

## Hızlı Komutlar Özeti

```bash
# Proje dizinine git
cd "/home/kSEN/Desktop/ Projects/Design-Microservices-Architecture-with-Patterns-Principles-mehmetozkaya/Design-Microservices-Architecture-with-Patterns-Principles/11-eshop-microservices-outbox/eshop-microservices"

# Uygulamayı başlat
dotnet run --project AppHost/AppHost.csproj --launch-profile https

# Uygulamayı durdur
Ctrl + C

# Kalan process'leri temizle (gerekirse)
pkill -f "dotnet"

# Docker container durumunu kontrol et
docker ps

# Solution'ı temizle
dotnet clean
```

---

## Outbox Pattern Akışı

Bu projede **Outbox Pattern** implementasyonu bulunmaktadır:

```
1. Kullanıcı checkout yapar
         │
         ▼
2. Basket servisi transaction başlatır
         │
         ▼
3. OutboxMessage tablosuna event kaydedilir
         │
         ▼
4. Shopping cart silinir
         │
         ▼
5. Transaction commit edilir
         │
         ▼
6. OutboxProcessor (BackgroundService) her 10 saniyede:
   - İşlenmemiş mesajları okur
   - RabbitMQ'ya publish eder
   - Mesajı "processed" olarak işaretler
         │
         ▼
7. Ordering servisi eventi consume eder
         │
         ▼
8. Yeni sipariş oluşturulur
```

Bu pattern, veritabanı işlemi ile mesaj gönderiminin atomik olmasını sağlar.

---

## Faydalı Bağlantılar

- [.NET Aspire Dokümantasyonu](https://learn.microsoft.com/en-us/dotnet/aspire/)
- [MassTransit Dokümantasyonu](https://masstransit.io/)
- [YARP Reverse Proxy](https://microsoft.github.io/reverse-proxy/)
- [Outbox Pattern](https://microservices.io/patterns/data/transactional-outbox.html)

---

*Son güncelleme: Aralık 2024*
