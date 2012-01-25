---
title: Home
layout: wikistyle
---

Releware Dokumentation
======================

Produktfeed
-----------

För att kunna visa dynamiska annonser måste vi ha en produktfeed (även kallad
prisfil) med alla produkter i antingen XML- eller CSV-format.  Följande
tabell visar vilken information vi kan ta emot, obligatoriska fält är markerade
med \*. Ni kan testa er produktfeed med vår valideringstjänst på
[http://dumbo.releware.net/price_file/validator](http://dumbo.releware.net/price_file/validator).

<table class="hor-minimalist-b">
  <tr>
    <th>Fält</th>
    <th>Beskrivning</th>
  </tr>
  <tr>
    <td class="field">Produkt-id*</td>
    <td>Måste vara unikt, används som id i vår databas.
        Måste vara samma som rapporteras av observationsskripten i butiken.
        Ofta samma som SKU/artikelnummer.</td>
  </tr>
  <tr>
    <td class="field">Titel*</td>
    <td>Produktnamnet som ska visas i annonsen. Om produkterna saknar bra namn,
    välj varumärke, tillverkare eller kategori.</td>
  </tr>
  <tr>
    <td class="field">Produkt-URL*</td>
    <td>Länk till produktens sida. Ska börja med http://</td>
  </tr>
  <tr>
    <td class="field">Bild-URL*</td>
    <td>Länk till produktbild. Ska börja med http://</td>
  </tr>
  <tr>
    <td class="field">SKU</td>
    <td>Också känt som artikelnummer. Får gärna vara unikt, men måste inte.</td>
  </tr>
  <tr>
    <td class="field">Varumärke</td>
    <td></td>
  </tr>
  <tr>
    <td class="field">Ordinarie pris</td>
    <td>Krävs om man vill visa pris i annons. Kan hämtas från samma fält som nuvarande pris.</td>
  </tr>
  <tr>
    <td class="field">Nuvarande pris</td>
    <td>Krävs om man vill visa pris i annons. Kan hämtas från samma fält som ordinarie pris.</td>
  </tr>
  <tr>
    <td class="field">Lagerstatus</td>
    <td></td>
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
  </product>
</products>
{% endhighlight %}

### CSV

* Första raden kan innehålla en header
* Citattecken måste "escape:as" genom att dubblas
* Extra fält ignoreras

#### Exempel
<pre>
    id|name|brand|product_url|image_url
    10010|40"" TV &amp; förstärkare|Panasonic|http://webplats.se/10010|http://webplats.se/images/10010.jpg
</pre>
