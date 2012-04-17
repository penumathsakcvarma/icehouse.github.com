---
title: Rekommendationsmallar
layout: wikistyle
---

# Rekommendationsmallar

Man kan styra utseendet på Relewares rekommendationer genom att använda mallar (templates). En mall är helt enkelt ett block av vanlig html-kod som ligger direkt på sidan där rekommendationen skall visas. För att rekommendationsmotorn skall veta att det är en mall, och var den skall infoga text och bilder om produkten, så märker man upp koden genom att sätta speciella klasser på html-element. Här är ett litet men komplett exempel:

{% highlight html %}
<div id="rw_bought_this_also_bought" style="display:none">
  <div class="head">
    <b>De som köpte den här produkten köpte också:</b>
  </div>
  <div class="RWItemTemplate">
    <h4><a href="%product_url%">%title%</a></h4>
    <p>%current_price%</p>
  </div>
  <div class="RWItemPromotionTemplate">
    <h4><a href="%product_url%">%title%</a></h4>
    <p>REA: %current_price% (<del>%normal_price%</del>)</p>
  </div>
  <div class="RWItemSeparator">
    <hr />
  </div>
  <div class="footer">
    Rekommendationer från Releware.
  </div>
</div>
{% endhighlight %}

Exemplet illustrerar flera egenskaper som förklaras nedan.


## Omgivande block

Hela rekommendationsrutan skall ligga inom ett html-element, typiskt en `<div>`. Den skall ha ett unikt id som gör att rekommendationsmotorn kan hitta mallen, och den göms genom att sätta `style="display:none"` så att den inte visas förrän rekommendationerna är laddade:

{% highlight html %}
<div id="rw_bought_this_also_bought" style="display:none">
  ...
</div>
{% endhighlight %}

De rekommendations-id:n som normalt används för de olika rekommendationstyperna är `rw_shuffled_toplist`, `rw_viewed_clicktrail_also_viewed`, `rw_viewed_this_also_viewed` och `rw_bought_this_also_bought`.

Om man vill ha flera rekommendationstyper på en sida så lägger man in flera mallar.


## Produktmall (RWItemTemplate)

I templaten skall det finnas ett element med klassen `RWItemTemplate`. Det är det blocket som kommer upprepas för varje produkt som rekommenderas:

{% highlight html %}
<div class="RWItemTemplate">
  <h4><a href="%product_url%">%title%</a></h4>
  <p>%current_price%</p>
</div>
{% endhighlight %}

I koden använder man namngivna variabler för att infoga text och bild om produkten. Variabeln består av variabelns namn omgivet av två procenttecken. Exempelvis `%title%` kommer i ovanstående exempel bytas ut mot titeln för en viss produkt i rekommendationen.

De variabler som finns att välja ifrån är `%title%`, `%sku%`, `%description%`, `%product_url%`, `%image_url%`, `%thumbnail_url%`, `%current_price%` och `%normal_price%`.

Alla variabler som klipps in är omkodade så de inte påverkar html-strukturen. Exempelvis om en titel i prisfilen är satt till `Cykel<br>Cresent` så kommer det infogas som `Cykel&lt;br&gt;Cresent` vilket kanske inte är vad man ville i det fallet. I så fall kan man lägga till `-raw` i slutet av variabelnamnet för att infoga värdet utan att omkoda det. Så om användaren är tänkt att kunna lägga in html när han editerar produkten så får man skriva (observera att det står `%title-raw%`):

{% highlight html %}
<div class="RWItemTemplate">
  <h4><a href="%product_url%">%title-raw%</a> %current_price%</h4>
</div>
{% endhighlight %}

Det finns också en speciell variabel som heter `%ordinal%` vars värde blir vilken i ordningen produkten är. Så den första produkten i listan av rekommendationer får ett `%ordinal%` som är 1, nästa får ett som är 2, osv.


## Produktmall med reapris (RWItemPromotionTemplate)

Man kan ha en separat mall som bara används om produkten har nedsatt pris. Om det finns en produktmall som är märkt med `RWItemPromotionTemplate` och `%current_price%` skiljer sig från `%normal_price%`, ja då används den mallen istället för `RWItemTemplate`. På så vis kan man exempelvis visa ordinarie pris överstruket vid sidan om det nedsatta priset:

{% highlight html %}
<div class="RWItemPromotionTemplate">
  <h4><a href="%product_url%">%title%</a></h4>
  <p>REA: %current_price% (<del>%normal_price%</del>)</p>
</div>
{% endhighlight %}


## Kod mellan produkter (RWItemSeparator)

Ibland kan det vara bra att kunna infoga html mellan, men bara mellan, produkterna. Det gör man genom att markera ett element med klassen `RWItemSeparator`:

{% highlight html %}
<div class="RWItemSeparator">
  <hr />
</div>
{% endhighlight %}

Så har man bara en produkt som rekommenderas så kommer `RWItemSeparator`-blocket inte alls med, och har man två så kommer det med emellan `RWItemTemplate`-blocken.

