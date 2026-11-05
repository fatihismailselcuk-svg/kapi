# Kapı özel durum ekranı (GitHub Pages)

Bu klasördeki `index.html` GitHub Pages'te yayınlanabilir. İstemci uygulaması ESP32'ye doğrudan bağlanmaz; EMQX broker üzerinde WSS (TLS) kullanır.

ÖNEMLİ: ESP32'de henüz MQTT request/response kodu yoksa bu arayüz kendi başına ESP32 verisini getirmez. ESP32 tarafı `prefix/request` üzerine abone olup `requestId` ve `replyTo` alanlarını doğrulamalı, `/api/home` ile **aynı JSON üreten ortak fonksiyonu** çağırmalı ve QoS 1, retain=false ile yanıtı `replyTo` topic'ine `{ "requestId":"...", "data":{ ...orijinal /api/home JSON... } }` biçiminde göndermelidir. Ayrı GNSS okuması yapılmamalı.

## Gizlilik ve üretime alma

- **GitHub Pages herkese açık bir statik web sitesidir.** Bir sayfa parolası HTML içine yazılarak gizlilik sağlanamaz. Sayfa URL'sini bilen herkes arayüzü görür; ama MQTT broker erişimi olmadan gerçek konum verisi alamamalıdır.
- MQTT sunucusu TLS doğrulamalı bağlantı kullanmalı. ESP32'de `setInsecure()` kullanılmamalı.
- ESP32 ve tarayıcı **farklı MQTT kullanıcıları** kullanmalı. Tarayıcı hesabının ACL'i yalnızca `prefix/request` PUBLISH ve `prefix/response/+` SUBSCRIBE olmalı. ESP32 yalnızca `prefix/request` SUBSCRIBE ve `prefix/response/+` PUBLISH yapmalı. Broker üzerinde wildcard geniş erişimi engellenmeli. Topic ön eki varsayılan `kapi/ozel`; üretimde rastgele, tahmin edilemez ön ek tercih edilir.
- Tarayıcı hesabını aile bireylerinin cihazlarında kullanmak, o hesapla görüntülenebilen canlı konuma erişim verir. Hesapları kişi bazında ayırmak ve gerektiğinde iptal etmek daha güvenlidir. Response topic'inde dinleyici kimlik doğrulaması için yalnızca client ID yetmez; güvenlik sınırı broker ACL'dir. Aile üyeleri arasında da yanıtların birbirinden gizli kalması isteniyorsa ayrı reply prefixleri ve kişi bazlı ACL gerekir.
- MQTT retain=false, broker persistence/history/audit kapalı veya veri içermeyen şekilde ayarlanmalı; kalıcı depolama, Analytics, paylaşılabilir loglar devre dışı tutulmalı.
- `/api/status` ayrıntılı tanılama ve ev/iş/telefon koordinatları içerdiği için **uzaktan paylaşılmamalı**. İlgili uygulama yalnızca `/api/home` kullanır; `/api/home` da rota hedefi koordinatları içerebilir. Tam JSON isteği bu bilgilerin de açılması anlamına gelir. İstenirse ev/iş/hedef ve diğer konumsal alanlar paylaşım öncesi maskelenmelidir.
- OpenStreetMap döşeme sunucusu, görüntülenen harita alanına ilişkin istekleri görür. Google Maps bağlantısı tıklanırsa Google'a koordinatlar iletilir. Daha güçlü gizlilik için üçüncü taraf harita döşemesi kaldırılmalı veya özel barındırma kullanılmalıdır.
- Bu demo sayfası parolayı tarayıcıda kalıcı olarak saklamaz; MQTT bağlantısı boyunca bellek içinde tutar. Tarayıcı eklentileri, paylaşılan cihazlar ve kötü amaçlı bir GitHub/dependency güncellemesi yine risk oluşturabilir. CDN kitaplıklarını denetlenmiş sürümlerle, mümkünse yerel statik kopyalarla sabitleyin.
- GitHub repo'ya hiç MQTT şifresi, token, `arduino_secrets.h`, gerçek konum veya ham MQTT mesajı eklemeyin. MQTT hesabı 2FA olmayan salt kullanıcı adı+paroladır; Broker hesabına mümkünse kısa süreli credential, cihaz izolasyonu ve limit uygulayın.

## Kurulum

1. GitHub üzerinde repo açıp `index.html` dosyasını köke yükleyin. Settings → Pages → Deploy from a branch → main /root.
2. EMQX WSS uç noktası `wss://HOST:8084/mqtt` ve ilgili ACL hesaplarını hazırlayın.
3. ESP32 firmware'ine mevcut `apiHome` JSON üretimini paylaşan istek üzerine MQTT yanıtı ekleyin.
4. Siteyi HTTPS ile açın, MQTT host/kullanıcı/şifreyi girin, bağlanıp Konumu sorgula'ya basın.

Bu paket yalnızca web arayüzünü içerir; **ESP32 firmware'i değiştirilmemiştir**.
