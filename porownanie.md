# Porównanie: obecna strona główna vs wizualizacja

Pomiar: 29.10.2026, Lighthouse 13.6.0 (CLI), profil mobile (symulowane dławienie, Moto G Power, 4G), Chrome headless.
Każda strona 5 przebiegów, naprzemiennie. W tabeli mediana, w nawiasie min-max.

- Obecna: https://hydraulik-awarie.pl/ (WordPress, własny motyw)
- Wizualizacja: https://disaster26.github.io/hydraulik-koncepcja/ (statyczny HTML na GitHub Pages)

| Metryka | Obecna strona | Wizualizacja |
|---|---|---|
| Performance | 100 (100-100) | 99 (98-100) |
| LCP | 1,16 s (1,15-1,17) | 1,08 s (1,06-1,20) |
| FCP | 1,16 s | 1,08 s |
| CLS | 0 | 0 |
| TBT | 3 ms (0-30) | 0 ms (0-0) |
| Speed Index | 1,6 s (1,5-2,6) | 3,6 s (1,1-3,8) |
| Accessibility | 100 | 100 |
| Best Practices | 100 | 100 |
| SEO | 100 | 66 (celowo, patrz niżej) |
| Waga strony (transfer) | 90 KiB | 150 KiB |
| Liczba żądań | 16 | 13 |

## Uczciwy komentarz do wyników

- Obecna strona jest już bardzo szybka i dobrze zoptymalizowana (100/100/100/100). Wizualizacja tego nie "bije", tylko utrzymuje ten sam poziom: LCP, CLS i TBT są równe albo minimalnie lepsze.
- SEO 66 w wizualizacji wynika wyłącznie z `<meta name="robots" content="noindex, nofollow">`, który celowo ustawiłem na kopii pod github.io, żeby nie tworzyć duplikatu strony w Google. Ten sam plik z robots `index, follow, max-image-preview:large, max-snippet:-1, max-video-preview:-1` (wariant produkcyjny, serwowany lokalnie) daje SEO 100, Accessibility 100, Best Practices 100; audyty canonical, hreflang, is-crawlable, title, description, link-text i image-alt zaliczone.
- Speed Index wizualizacji ma duży rozrzut (1,1-3,8 s). W przebiegach z wynikiem ok. 3,6 s pierwszy obserwowany paint w śladzie przeglądarki przypadał ok. 2,3 s, mimo że dokument i wszystkie zasoby pierwszego ekranu były pobrane po ok. 0,46 s (zdarzenie load). To opóźnienie po stronie środowiska pomiarowego / hostingu GitHub Pages, nie blokujący zasób w kodzie (strona nie ma zewnętrznego CSS ani JS w head). Na obecnej stronie występuje ten sam efekt w mniejszej skali (1,5-2,6 s). Na docelowym serwerze trzeba to zmierzyć ponownie.
- Wizualizacja waży więcej (150 vs 90 KiB), bo pierwszy ekran ma duże zdjęcie (wycięty portret, AVIF 22 KiB na telefonie) i własny font nagłówków (Archivo, jeden plik woff2 28 KiB, podzbiór z polskimi znakami). Bez fontu i przy mniejszym zdjęciu waga zbliżyłaby się do obecnej.
- GitHub Pages ustawia krótki cache (10 min), stąd uwaga Lighthouse "Use efficient cache lifetimes". Na docelowym serwerze statyczne pliki dostałyby `Cache-Control: max-age=31536000, immutable`.

## Drobne uwagi Lighthouse do obecnej strony (nie wpływają na wynik 100)

- `unsized-images`: emoji WordPressa (s.w.org/images/core/emoji) w opiniach i przy telefonie są wstawiane jako `<img>` bez width/height.
- `label-content-name-mismatch`: link "Zapisz kontakt" i przycisk telefonu mają aria-label inny niż widoczny tekst.
- `render-blocking`: main.css (16 KiB) blokuje renderowanie; w wizualizacji CSS jest w `<style>` w dokumencie.

## Co zostało zrobione technicznie w wizualizacji

- Jeden plik HTML, CSS w `<style>`, ok. 1 KB JS na końcu body (menu, dolny pasek po przewinięciu). Brak bibliotek, brak jQuery, brak zewnętrznych skryptów.
- Obrazy AVIF + WebP przez `<picture>`, `srcset`/`sizes`, `width`/`height` na każdym `<img>` i `<source>` (art direction telefon/komputer bez przesunięć), `loading="lazy"` poniżej pierwszego ekranu, `fetchpriority="high"` dla zdjęcia LCP.
- Preload: font woff2 (`crossorigin`) i zdjęcie LCP osobno dla telefonu i komputera (`media` + `imagesrcset`).
- Font `font-display: optional` + preload: zero zmiany układu przy ładowaniu fontu; treść używa fontu systemowego jak obecna strona.

## Elementy SEO w `<head>`

- `title` (51 znaków), `meta description` (158 znaków)
- `canonical` na https://hydraulik-awarie.pl/
- `robots` (na wizualizacji noindex, wariant produkcyjny w komentarzu w kodzie)
- `hreflang` pl / en (/plumber-in-lodz/) / x-default - wymaga wzajemnego tagu na stronie angielskiej
- `link rel="sitemap"` do /ha-sitemap.xml
- Open Graph: og:locale, og:type, og:site_name, og:title, og:description, og:url, og:image + type/width/height/alt; Twitter card
- JSON-LD `@graph` z tymi samymi `@id` co obecna strona (ciągłość encji):
  WebSite, WebPage, ImageObject, Person (Michał Pęczek), Plumber (NAP, geo, godziny 24/7, NIP/REGON, contactPoint z WhatsApp, sameAs, areaServed: Łódź + 5 dzielnic jako AdministrativeArea + 14 miejscowości z URL podstron lokalnych, GeoCircle 25 km), OfferCatalog z 11 usługami (Service z `@id` `.../#usluga` jak na podstronach usług, url, provider, areaServed), ItemList 4 najnowszych wpisów Dziennika interwencji (Article: headline, datePublished z jego stron, image, author, contentLocation).
- Walidacja: JSON poprawny; 72 węzły z typem sprawdzone skryptem względem słownika schema.org (schemaorg-current-https.jsonld): każdy typ i każda właściwość istnieją, pasują do domeny typu i zakresu wartości, łącznie z odwołaniami przez `@id`.
- Świadomie pominięte: `aggregateRating` (oceny z Google na własnej stronie LocalBusiness to "self-serving reviews", Google nie pokazuje dla nich gwiazdek) i `FAQPage` (wyniki rozszerzone FAQ ograniczone od 2023 r. do serwisów rządowych i medycznych).

## Struktura URL

Wszystkie linki w menu, kartach usług, wpisach Dziennika, lokalizacjach i stopce wskazują na obecne adresy hydraulik-awarie.pl. Sprawdzone skryptem: każda ścieżka strony linkowana z wizualizacji występuje w https://hydraulik-awarie.pl/ha-sitemap.xml (311 adresów).

Surowe raporty Lighthouse (JSON + HTML) są w folderze roboczym: lh_jego_1..5, lh_nasza_1..5, lh_nasza_prod_seo.json.
