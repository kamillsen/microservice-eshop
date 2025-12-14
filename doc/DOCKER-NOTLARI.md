# 🐳 Docker Kullanımı - Notlar

> Bu projede Docker kullanımı hakkında önemli notlar ve açıklamalar.

---

## 📋 Genel Bakış

Bu projede **Docker kullanılıyor**, ancak geleneksel `Dockerfile` veya `docker-compose.yml` dosyaları **bulunmuyor**.

Bunun yerine proje **.NET Aspire** kullanıyor. Aspire, Docker container'larını otomatik olarak yönetir ve yapılandırır.

---

## 🎯 Projede Kullanılan Docker Container'ları

### 1. 🐘 PostgreSQL

```csharp
var postgres = builder
    .AddPostgres("postgres")
    .WithPgAdmin(pgAdmin => pgAdmin.WithUrlForEndpoint("http", url => url.DisplayText = "PostgreDB Browser"))
    .WithDataVolume()
    .WithLifetime(ContainerLifetime.Persistent);
```

**Özellikler:**
- ✅ `catalogdb` veritabanı (Catalog mikroservisi için)
- ✅ `basketdb` veritabanı (Basket mikroservisi için)
- ✅ pgAdmin yönetim arayüzü dahil
- ✅ Veri kalıcılığı (`WithDataVolume()`)
- ✅ Persistent container (yeniden başlatmalarda veri korunur)

### 2. 🔴 Redis

```csharp
var cache = builder
    .AddRedis("cache")
    .WithRedisInsight()
    .WithDataVolume()
    .WithLifetime(ContainerLifetime.Persistent);
```

**Özellikler:**
- ✅ Basket mikroservisi için cache katmanı
- ✅ Redis Insight yönetim arayüzü dahil
- ✅ Veri kalıcılığı (`WithDataVolume()`)
- ✅ Persistent container

### 3. 🗄️ SQL Server

```csharp
var sqlServer = builder
    .AddSqlServer("sqlserver")
    .WithDataVolume()
    .WithLifetime(ContainerLifetime.Persistent);
```

**Özellikler:**
- ✅ `orderdb` veritabanı (Ordering mikroservisi için)
- ✅ Veri kalıcılığı (`WithDataVolume()`)
- ✅ Persistent container

### 4. 🐰 RabbitMQ

```csharp
var rabbitmq = builder
    .AddRabbitMQ("rabbitmq")
    .WithManagementPlugin()
    .WithDataVolume()
    .WithLifetime(ContainerLifetime.Persistent);
```

**Özellikler:**
- ✅ Mikroservisler arası mesajlaşma için
- ✅ Management Plugin dahil (web arayüzü)
- ✅ Veri kalıcılığı (`WithDataVolume()`)
- ✅ Persistent container

---

## ⚙️ Nasıl Çalışıyor?

### Aspire'ın Docker Yönetimi

1. **Otomatik Image İndirme:**
   - Aspire, gerekli Docker image'lerini otomatik olarak indirir
   - İlk çalıştırmada image'ler Docker Hub'dan çekilir

2. **Container Başlatma:**
   - `dotnet run --project AppHost/AppHost.csproj` komutu çalıştırıldığında
   - Aspire tüm container'ları otomatik olarak başlatır

3. **Veri Yönetimi:**
   - `WithDataVolume()` ile veriler kalıcı hale getirilir
   - Container silinse bile veriler korunur

4. **Yaşam Döngüsü:**
   - `WithLifetime(ContainerLifetime.Persistent)` ile container'lar kalıcı yapılır
   - Uygulama durdurulsa bile container'lar çalışmaya devam eder

---

## 📝 Önemli Notlar

### ✅ Docker Kullanımı

- **Evet, Docker kullanılıyor** ✅
- Container'lar otomatik olarak yönetiliyor
- Manuel Docker komutlarına gerek yok

### ❌ Geleneksel Docker Dosyaları

- **Dockerfile yok** ❌
- **docker-compose.yml yok** ❌
- Aspire bunları otomatik oluşturuyor

### 🔧 Container Yönetimi

- Container'lar Aspire Dashboard'dan izlenebilir
- Container logları dashboard'da görüntülenebilir
- Container'ları manuel olarak durdurmak için Docker komutları kullanılabilir

---

## 🛠️ Manuel Container Yönetimi (Gerekirse)

### Container'ları Listeleme

```bash
docker ps
```

### Container'ları Durdurma

```bash
# Tüm Aspire container'larını durdur
docker stop $(docker ps -q --filter "name=postgres" --filter "name=cache" --filter "name=sqlserver" --filter "name=rabbitmq")
```

### Container'ları Silme

```bash
# Dikkat: Bu işlem verileri silmez (volume'ler korunur)
docker rm $(docker ps -aq --filter "name=postgres" --filter "name=cache" --filter "name=sqlserver" --filter "name=rabbitmq")
```

### Volume'leri Görüntüleme

```bash
docker volume ls
```

### Volume'leri Silme (Verileri Tamamen Silmek İçin)

```bash
# Dikkat: Bu işlem tüm verileri kalıcı olarak siler!
docker volume rm <volume-name>
```

---

## 🔍 Container İsimleri

Aspire tarafından oluşturulan container'lar genellikle şu isimlerle başlar:

- `postgres-*` - PostgreSQL container'ı
- `cache-*` - Redis container'ı
- `sqlserver-*` - SQL Server container'ı
- `rabbitmq-*` - RabbitMQ container'ı

---

## 📊 Aspire Dashboard'dan Container Yönetimi

Aspire Dashboard'da:

1. **Resources** sekmesinde tüm container'lar görüntülenir
2. Her container için:
   - ✅ Durum (Running/Stopped)
   - 📊 Loglar
   - 🔗 Yönetim arayüzü linkleri (pgAdmin, Redis Insight, RabbitMQ Management)

---

## ⚠️ Önemli Uyarılar

1. **SQL Server RAM Gereksinimi:**
   - SQL Server Linux container'ı minimum **2GB RAM** gerektirir
   - Yetersiz RAM varsa container başlamayabilir

2. **Port Çakışmaları:**
   - Aspire otomatik port ataması yapar
   - Manuel port yapılandırması gerekmez

3. **Veri Kalıcılığı:**
   - `WithDataVolume()` kullanıldığı için veriler korunur
   - Container silinse bile veriler Docker volume'lerinde kalır

4. **Docker İzinleri:**
   - Docker komutlarını çalıştırmak için kullanıcının `docker` grubunda olması gerekir
   - İzin hatası alırsanız: `sudo usermod -aG docker $USER`

---

## 🔗 İlgili Dosyalar

- **AppHost/Program.cs** - Container yapılandırmaları
- **doc/CALISTIRMA-REHBERI.md** - Docker kurulumu ve kullanımı
- **doc/KLASOR-YAPISI.md** - Proje yapısı dokümantasyonu

---

## 📚 Ek Kaynaklar

- [.NET Aspire Docker Desteği](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/docker-support)
- [Aspire Container Yönetimi](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/containers)
- [Docker Dokümantasyonu](https://docs.docker.com/)

---

*📅 Son güncelleme: Aralık 2024*
