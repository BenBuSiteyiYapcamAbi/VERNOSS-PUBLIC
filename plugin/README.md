# Vernoos Eklentisi

Vernoos sitesi ile Minecraft sunucunuzu güvenli şekilde bağlayan Paper/Purpur eklentisi.

**Sürüm:** 4.7.0 · **Uyumluluk:** Paper / Purpur 1.20+ · **Java:** 17+

## Ne yapar

- **Oyuncu pazaryeri** — oyuncular eşyalarını kredi karşılığı ilan eder, satın alır (`/pazar`)
- **Oyun içi mağaza** — sitedeki mağaza ürünleri doğrudan oyun içinde (`/magaza`)
- **Site envanteri** — siteden alınan eşyalar oyun içi envantere teslim edilir (`/envanter`)
- **Sıralama tablosu** — kredi / harcama sıralaması oyun içi ekranda (`/siralama`)
- **AuthMe / launcher SSO** — site hesabı ile oyun girişi eşlenir, oturum bekçisi siteden çıkınca oyunda da kapatır
- **İmzalı otomatik güncelleme** — yeni sürüm imza doğrulamasıyla kendini günceller

## Kurulum

1. `Vernoos-Plugin.jar` dosyasını sunucunun `plugins/` klasörüne atın.
2. Sitede **Panel → Eklentiler** sayfasından bağlama satırını kopyalayın ve `plugins/Vernoos/config.yml` içine yazın:

   ```yaml
   pair: "https://siteniz.com|KOD|SUNUCU_ID"
   ```

3. Sunucuyu başlatın. Bağlama tamamlandığında anahtar `config.yml`'de **kalmaz**; makineye bağlı, AES-256-GCM ile şifrelenmiş `credentials.dat` dosyasına taşınır.

## Komutlar

| Komut | Alt komutlar | Açıklama |
|---|---|---|
| `/vernoos` | `check`, `status`, `update`, `version`, `pair`, `db`, `permissions`, `reload` | Yönetim komutları |
| `/pazar` | `sat`, `ara`, `ilanlarim`, `iptal`, `teslim`, `yardim` | Oyuncu pazaryeri |
| `/magaza` | — | Site mağazası (oyun içi) |
| `/envanter` | — | Site envanteri |
| `/siralama` | — | Kredi / harcama sıralaması |

## İzinler

| İzin | Varsayılan | Açıklama |
|---|---|---|
| `vernoos.admin` | OP | Vernoos yönetim komutları (`/vernoos`) |
| `vernoos.market.use` | Herkes | Pazaryerini kullanma |
| `vernoos.webstore.use` | Herkes | Site mağaza / envanter erişimi |
| `vernoos.leaderboard.use` | Herkes | Sıralama ekranını açma |

## Dil / mesajları özelleştirme

Tüm mesajlar `plugins/Vernoos/lang/tr.yml` (ve `en.yml`) dosyasındadır.

- Metinleri düzenledikten sonra `/vernoos reload` çalıştırın (10 sn bekleme süresi vardır).
- Bir anahtar dosyada yoksa koddaki gömülü varsayılan metne düşülür; eklenti çalışmaya devam eder.

## PlaceholderAPI

Sunucuda [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) kuruluysa aşağıdaki yer tutucular otomatik kaydedilir (`%vernoos_...%`). PAPI kurulu değilse hiçbir şey bozulmaz — eklenti her durumda normal çalışır.

| Yer tutucu | Değer |
|---|---|
| `%vernoos_credits%` | Oyuncunun kredisi (ör. `123.45`) |
| `%vernoos_balance%` | `%vernoos_credits%` ile aynı |
| `%vernoos_credits_int%` | Oyuncunun kredisi, tam sayı (ör. `123`) |
| `%vernoos_rank%` | Oyuncunun sitedeki rütbesi (yoksa `-`) |
| `%vernoos_vip%` | VIP durumu (`evet` / `hayir`) |
| `%vernoos_server%` | Sunucu adı (config'de `serverName` yoksa `#SUNUCU_ID`) |
| `%vernoos_server_id%` | Sunucunun sitedeki ID'si |
| `%vernoos_online%` | Çevrimiçi oyuncu sayısı |
| `%vernoos_max%` | Maksimum oyuncu sayısı |
| `%vernoos_site%` | Site adresi |
| `%vernoos_version%` | Eklenti sürümü |
| `%vernoos_lang%` | Aktif dil kodu (`tr`, `en`, ...) |

Örnek kullanım (ör. EssentialsChat formatında): `&7Kredi: &a%vernoos_credits% &8| &7Rütbe: &f%vernoos_rank%`

Kredi/rütbe/VIP verileri arka planda 60 saniyede bir tazelenir; yer tutucular ana iş parçacığını asla bekletemez, site erişilemezse son bilinen değer gösterilir.

## Güvenlik notları

- **`credentials.dat` makineye bağlıdır** — dosya başka makineye kopyalanırsa çözülemez; makine değişirse panelden yeniden bağlama satırı alın.
- Eşya ve para hareketleri **"önce deftere, sonra ele"** prensibiyle yapılır — sunucu çakılsa bile eşya kaybolmaz veya çoğalmaz.
- Teslimat işlemleri **idempotent**tir; aynı teslimat iki kez uygulanmaz.
- Eklentinin siteye yaptığı istekler **SSRF allow-list** ile sınırlıdır.

## Sunucu yazılımı uyumluluğu

| Sunucu | Durum |
|---|---|
| Paper / Purpur 1.20+ | Tam destek |
| Spigot 1.20+ | Çalışır (Paper API'si kullanılmıyor) |
| **Folia** | **Desteklenmez.** Eklenti açılışta Folia'yı algılar, net uyarı verir ve kendini kapatır — eşya/para bozulmasını önlemek için. |

## Sorun giderme

| Belirti | Çözüm |
|---|---|
| Logda `401` / yetkisiz hatası | Bağlama satırı geçersiz — panelden yeni satır alın; sorun sürerse ApiKeyService kayıtlarını kontrol edin |
| `Sunucu siteye bagli degil` | `config.yml`'de `pair:` satırı boş veya silinmiş — panelden bağlama satırını yazıp yeniden başlatın |
| `AEADBadTagException` | Makine değişti — `credentials.dat` bu makinede çözülemez; panelden yeniden bağlanın |
| Satın alınan eşya gelmedi | `/pazar teslim` çalıştırın; envanter doluysa yer açın |

---

© Vernoos

Son güncelleme: plugin 4.7.0
