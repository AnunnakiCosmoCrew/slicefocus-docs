# Store listing for 1.7.4: subtitle, keywords, English (U.K.) and Turkish

*Created: 2026-10-26 | Release: [SliceFocusFE#641](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/641) | Status: draft for the founder's review (founder check: public)*

## Why

The promotion research of 2026-10-26 found that the App Store's text fields are the lever that is best documented and fully in our hands. Apple ranks search results on:
- the name, subtitle and the 100-character keyword field
- the primary category
- downloads and ratings

Promotional text is not indexed. Words combine across the name, subtitle and keywords, so a word should appear in only one of them.

The Türkiye storefront reads **English (U.K.)** as its default language and **Turkish** as its additional one. Each listing adds its own keyword field there. All three of our ratings and about a quarter of the site's visits are Turkish.

## Today (1.7.3, en-US)

| Field | Value |
| --- | --- |
| Name | SliceFocus |
| Subtitle | Visual Focus & Daily Schedule |
| Keywords (95) | `pomodoro,timer,time,blocking,blocker,website,distraction,screen,planner,study,adhd,day,calendar` |

The subtitle spends three of its words on terms people rarely search for ("visual", "daily", "schedule"). It carries neither Pomodoro nor blocking, the two things people do search for.

## Proposed

### English (U.S.), iOS and macOS

| Field | Value | Length |
| --- | --- | --- |
| Subtitle | **Pomodoro Planner & App Blocker** | 30/30 |
| Keywords | `focus,timer,time,blocking,website,distraction,screen,study,adhd,day,calendar,schedule,deep,work` | 95/100 |

- pomodoro, planner and blocker move into the subtitle, which frees room for focus, schedule and deep work.
- "time" and "blocking" stay as separate words so they combine into "time blocking".
- Alternative subtitle if "App Blocker" reads too strong: **Pomodoro Timer & Day Planner** (28).

### English (U.K.), new, iOS and macOS

A second English listing. Same description and screenshots as en-US; British spelling where it differs. It gives a second keyword field in the UK and Türkiye storefronts.

| Field | Value | Length |
| --- | --- | --- |
| Subtitle | Visual Day Planner & Timer | 26/30 |
| Keywords | `productivity,habit,tracker,clock,hour,visual,circle,tasks,goals,organiser,agenda,student,revision` | 97/100 |

### Turkish, new, iOS and macOS

The app's interface is English only for now, so the description says so. The listing must not suggest a Turkish interface.

| Field | Value | Length |
| --- | --- | --- |
| Name | SliceFocus | |
| Subtitle | **Pomodoro ve Günlük Planlayıcı** | 29/30 |
| Keywords | `odak,odaklanma,zamanlayıcı,ders,çalışma,verimlilik,takvim,engelleyici,uygulama,alışkanlık,dehb` | 94/100 |
| Promotional text | Gününü 24 saatlik bir saat üzerinde planla; odak ve mola zamanlayıcısı gerisini kendi yürütsün. | |

If App Store Connect counts bytes rather than characters, drop `dehb` first.

**Description (tr):**

```
GÜNÜNÜ TEK BAKIŞTA GÖR
SliceFocus gününü 24 saatlik dairesel bir zaman çizelgesine yerleştirir. Saatlerinin nereye gittiğini bir bakışta görürsün. Zaman bloklarını doğrudan saat kadranının üzerine çizerek gün planını oluşturursun; plan değişince blokları sürükleyip yeniden boyutlandırırsın. Günün tamamını tek ekranda görmek zamanı somut kılar; DEHB'si olan pek çok kişi bu görsel yaklaşımı yararlı buluyor.

TAKVİMİN DE AYNI DAİREDE
SliceFocus, Apple Takvim ile çalışır. Takvim etkinliklerin gün görünümünde dilimlerinin yanında görünür; planladığın dilimleri seçtiğin bir takvime aktarabilirsin. Aktarılan bir etkinliği Takvim'de taşırsan ya da silersen SliceFocus da buna uyar. Bir dilim başlamadan önce hatırlatıcı ekleyebilirsin.

KENDİ KENDİNE İLERLEYEN POMODORO
Bir bloğu başlat, gerisini SliceFocus yürütsün. Çalışma süresi bitince mola kendiliğinden başlar, mola bitince sıradaki çalışma turu. Çalışma, kısa mola ve uzun mola sürelerini ve uzun moladan önceki tur sayısını sen belirlersin. Arka planda beyaz, kahverengi ya da pembe gürültü açabilirsin.

ODAK KALKANI: DİKKAT DAĞITAN UYGULAMA VE SİTELERİ ENGELLE
Seni işinden koparan uygulamaları ve siteleri seç; SliceFocus odak oturumlarının çalışma bölümlerinde bunları engeller. iPhone'da seçimi Ekran Süresi ile yaparsın ve uygulama kapalı olsa bile engel, oturumun planlanan bitişinde kalkar. Mac'te herhangi bir cihazda başlattığın oturum Mac'i de korur: Safari uzantısı listedeki siteleri engeller, engellenen bir uygulamaya geçtiğinde bir kalkan penceresi açılır. Molalar serbesttir.

PLANLADIĞINLA YAPTIĞINI KARŞILAŞTIR
SliceFocus oturum geçmişini tutar, planladığın günü gerçekte harcadığın zamanla karşılaştırır. Günlük, haftalık ve aylık eğilimlerine bakarak rutinini verilere dayanarak düzenlersin.

ÖNE ÇIKANLAR
• 24 saatlik dairesel zaman çizelgesi, renkli dilimlerle
• Kadran üzerinde çizerek ve sürükleyerek planlama
• Apple Takvim ve Anımsatıcılar ile çalışma
• Kendi kendine ilerleyen Pomodoro: çalışma, kısa mola, uzun mola
• Odak Kalkanı: iPhone ve Mac'te seçtiğin uygulama ve siteleri engelleme
• Beyaz, kahverengi ve pembe gürültü
• Plan ve gerçekleşen karşılaştırması, günlük, haftalık ve aylık eğilimler
• Kilit ekranında Canlı Etkinlik ve widget'lar
• Mac menü çubuğunda zamanlayıcı
• Tüm cihazlarda tek zamanlayıcı: iPhone'da başlat, aynı oturumu Mac'te gör
• Apple veya Google ile hızlı giriş
• Açık ve koyu tema

Not: Uygulamanın arayüzü şimdilik İngilizce.
```

**What's New (tr):**

```
SliceFocus artık App Store'da Türkçe sayfasıyla. Bu sürümde birkaç küçük iyileştirme var.
```

### What's New, 1.7.4 (en-US and en-GB)

```
A few small improvements under the hood.
```

## Steps in App Store Connect (per platform)

1. Create version 1.7.4 (it copies the 1.7.3 fields).
2. en-US: replace the subtitle and keywords.
3. Add the English (U.K.) localization: paste the en-US description, the subtitle and keywords above, and the en-US screenshots.
4. Add the Turkish localization: the fields above, plus the en-US screenshots (no Turkish screenshots, since the interface is English).
5. Add build 43, Save, Add for Review, Submit.

Do it on iOS and macOS. Reading the fields back in App Store Connect after saving matters, because React-controlled fields sometimes ignore pasted text (see the 1.7.3 submission notes).
