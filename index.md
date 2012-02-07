---
title: Prisfil / produktfeed
layout: wikistyle
---

Releware Dokumentation
======================

Produktfeed
-----------

För att kunna visa dynamiska annonser måste vi ha en produktfeed (även kallad
prisfil) med alla produkter i antingen XML- eller CSV-format.  Följande
tabell visar vilken information vi kan ta emot. Ni kan testa er produktfeed med vår valideringstjänst på
[http://dumbo.releware.net/price_file/validator](http://dumbo.releware.net/price_file/validator).

### Obligatoriska fält

<table class="hor-minimalist-b">
  <tr>
    <th>Fält</th>
    <th>Beskrivning</th>
  </tr>
  <tr>
    <td class="field">Produkt-id</td>
    <td>Måste vara samma som rapporteras av observationsskripten i butiken.
        Måste vara unikt, används som id i vår databas. Ofta samma som SKU/artikelnummer..</td>
  </tr>
  <tr>
    <td class="field">Produktnamn*</td>
    <td>Det produktnamn som ska visas i annonsen. Om produkterna saknar bra namn,
        välj varumärke, tillverkare eller kategori.</td>
  </tr>
  <tr>
    <td class="field">Produkt-URL*</td>
    <td>Länk till produktens sida. Ska börja med `http://`.</td>
  </tr>
  <tr>
    <td class="field">Bild-URL*</td>
    <td>Länk till produktbild. Ska börja med `http://`.  Om det finns flera bilder att välja på,
        välj en som är stor nog att se bra ut i annonsen men inte onödigt stor.</td>
  </tr>
  <tr>
    <td class="field">Nuvarande pris*</td>
    <td>Kan hämtas från samma fält som ordinarie pris.</td>
  </tr>
  <tr>
    <td class="field">Ordinarie pris*</td>
    <td>Kan hämtas från samma fält som nuvarande pris.</td>
  </tr>
</table>

### Frivilliga fält

<table class="hor-minimalist-b">
  <tr>
    <th>Fält</th>
    <th>Beskrivning</th>
  </tr>
  <tr>
    <td class="field">SKU</td>
    <td>Också känt som artikelnummer. Får gärna vara unikt, men måste inte.
        SKU används inte av vårt system utan finns med för att underlätta kommunikation
        med butiksägarna som ofta kan SKU men inte det unika produkt-id:t.</td>
  </tr>
  <tr>
    <td class="field">Varumärke</td>
    <td>Kan visas tillsammans med produktnamn i annons.</td>
  </tr>
  <tr>
    <td class="field">Lagerstatus</td>
    <td>Varor som finns i lager bör markeras med <code>Y, Yes, 1, T, True</code> eller antal varor i lager.
        Varor som inte finns i lager bör markeras med <code>N, No, 0, F, False</code> eller liknande.
        Om lagerstatus saknas antar vi att alla varor finns i lager.</td>
  </tr>
</table>

### XML

* XML filen måste vara syntaktiskt korrekt. Använd med fördel http://validator.w3.org
* Encoding bör stå längst upp, men kan konfigureras i efterhand om det saknas.

#### Exempel
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<products>
  <product id="10010">
    <name>40&quot; TV &amp; förstärkare</name>
    <brand>Panasonic</brand>
    <product_url>http://webplats.se/10010</product_url>
    <image_url>http://webplats.se/images/10010.jpg</image_url>
    <price>400.00</price>
    <normal_price>460.00</normal_price>
    <inStock>Y</inStock>
    <SKU>P150ZTV2-2</SKU>
  </product>
</products>
{% endhighlight %}

### CSV

* Första raden kan innehålla en header
* Citattecken måste "escape:as" genom att dubblas
* Extra fält ignoreras

#### Exempel
<pre>
    id|name|brand|product_url|image_url|price
    10010|"40"" TV &amp; förstärkare"|Panasonic|http://webplats.se/10010|http://webplats.se/images/10010.jpg|400.00
</pre>
