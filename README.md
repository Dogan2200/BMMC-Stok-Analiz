# AVESORO BI – Excel Beslemeli Stok Raporu (Microapp)

Bu klasör, paylaştığınız tek sayfalık HTML uygulamasının **gömülebilir micro‑frontend** halidir.
Statik olarak Vercel’e yükleyebilir, başka sistemlere `<iframe>` ile modül gibi ekleyebilirsiniz.

## Hızlı Başlangıç

1. `index.html` dosyasını bir statik sunucuda (veya doğrudan dosya olarak) açın.
2. **Excel/CSV yükle** butonuyla veri yükleyin (`.xlsx/.csv`).
3. Filtreleri kullanın; Pareto, karşılaştırmalar ve AI özetlerini görün.
4. Örnek veri için `host-demo.html` sayfasında **CSV Yükle (URL)** alanında
   `./sample-data/sample-stok.csv` yolunu kullanabilirsiniz.

> Not: `.xlsx` için internet bağlantısıyla `xlsx.full.min.js` CDN’den yüklenir.
> Offline kurulum istiyorsanız, dosyayı projeye indirip local path ile referans verin.

## Gömülü Kullanım (Web Modül)

Bu microapp, ana sayfaya **iframe** ile eklenir ve `postMessage` ile kontrol edilir.
Örnek ev sahibi sayfa: `host-demo.html`.

### Basit kullanım

```html
<iframe id="avesoro" src="https://sizin-alan-adiniz/index.html"></iframe>
<script>
  const f = document.getElementById('avesoro').contentWindow;
  // CSV yükle
  f.postMessage({type:'loadCSV', payload:'https://.../stok.csv'}, '*');
  // Mod değiştir
  f.postMessage({type:'setMode', payload:'miktar'}, '*');
  // Filtre uygula
  f.postMessage({type:'setFilters', payload:{DepoSinifi:['Main Warehouse'], Donem:['2025-08']}}, '*');
</script>
```

### Mesaj API’si

| Tip            | Payload örneği                                          | Açıklama |
|----------------|----------------------------------------------------------|---------|
| `getState`     | —                                                        | Geçerli filtre ve analiz modunu döndürür (`state` mesajıyla). |
| `setMode`      | `'tutar'` \| `'miktar'`                                  | Analiz modunu değiştirir. |
| `clearFilters` | —                                                        | Tüm filtreleri temizler. |
| `setFilters`   | `{Donem:[], DepoSinifi:[], Depo:[], FSN:[], MTRL:[], UstGrup:[], AltGrup:[]}` | Filtreleri topluca ayarlar. |
| `focusTab`     | `'overview' \| 'compare' \| 'pareto' \| 'ai'`            | Sekme odaklar. |
| `loadCSV`      | `'https://...'` veya göreli yol                          | Uzak CSV’yi indirir ve yükler (CORS gerekir). |

**Olaylar:** Uygulama gömülü açılınca `ready`, komutlar için `ack`/`error`, `getState` için `state` gönderilir.

## Vercel’e Yükleme

- Vercel Dashboard > New Project > **Import** (veya **Deploy from Git**).
- Statik proje için tek bir `index.html` yeterlidir.
- Örnek veri ve host sayfasını da ekleyebilirsiniz (zorunlu değil).

## Genişletme Yol Haritası

1. **Modülerleştirme (vanilla JS):**
   - `index.html`’deki script’i `src/` altına bölün (örn. `helpers.js`, `state.js`, `ui.js`, `ai.js`).
   - `init(container, options)` şeklinde bir başlangıç fonksiyonu tanımlayın.
   - Global `document.getElementById` kullanımını `container.querySelector` ile yerelleştirin.

2. **Web Component (önerilen modül yaklaşımı):**
   - `<avesoro-stock-report>` isimli bir custom element oluşturup tüm UI’ı **Shadow DOM** içinde yönetin.
   - Props: `mode="tutar|miktar"`, `theme="light|dark"`, `lang="tr"` vb.
   - Dış dünya ile `CustomEvent` ve attribute/prop değişimleri üzerinden haberleşin.

3. **Router / Çoklu Sayfa:**
   - `/#/stok`, `/#/lojistik`, `/#/satin-alma` gibi hash‑tabanlı yönlendirme.
   - Her modül için bağımsız veri işleme ve görseller.

4. **Veri Katmanı:**
   - Excel/CSV yanında REST/GraphQL bağlayıcıları (SAP/ERP).
   - **IndexedDB** ile büyük veri setlerini yerelde cache’leyin.
   - Büyük dosyalar için worker kullanın (UI donmasın).

5. **AI (güvenli entegrasyon):**
   - Tarayıcıdan doğrudan LLM çağırmak yerine Vercel **Serverless Function** (`/api/ai`) üzerinden çağırın.
   - API anahtarlarını **Environment Variable** olarak saklayın (istemciye sızdırmayın).
   - İstemciden `/api/ai` endpoint’ine özetleme/uyarı isteği gönderin.

6. **Tema ve Kurumsal Kimlik:**
   - Renkleri `:root` değişkenlerinden yönetiyoruz. Avesoro sarı/beyaz paleti için
     `--brand-2`, `--brand-3` ve KPI kart gradyanlarını kurum renklerine çekin.
   - Logo, font ve karanlık mod desteği ekleyin.

7. **PWA ve Offline:**
   - `manifest.webmanifest` + `service-worker.js` ile son yüklü veriyi ve UI’ı cache’leyin.
   - XLSX kütüphanesini yerel barındırın (CDN bağımlılığını kaldırın).

8. **Erişim ve Yetkilendirme:**
   - Gömülü kullanımda JWT/SSO ile okuma yetkisini kontrol eden bir gateway.
   - Filtre/rapor şablonlarını kullanıcıya özel kaydetme/yükleme.

## Bilinen İyileştirmeler

- Başlık alanındaki `msg` eksikliği giderildi.
- İframe entegrasyonu için **postMessage API** eklendi.
- Örnek veri (`sample-data/sample-stok.csv`) sağlandı.

İyi çalışmalar!
