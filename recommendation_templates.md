---
title: Rekommendationsmallar
layout: wikistyle
---

# Rekommendationsmallar

Man kan styra utseendet på Relewares rekommendationer genom att använda mallar (templates). En mall är helt enkel ett block av vanlig html-kod som ligger direkt på sidan där rekommendationen skall visas. För att rekommendationsmotorn skall veta att det är en mall, och vart den skall infoga text och bilder om produkten, så märker man upp koden genom att sätta speciella klasser på html element. Här är ett litet men komplett exempel:

{% highlight html %}
<div id="rw_bought_this_also_bought" style="display:none">
  <div class="head">
    <b>De som köpte den här produkten köpte också:</b>
  </div>
  <div class="RWItemTemplate">
    <h4><a href="%product_url%">%title%</a> %current_price%</h4>
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

Hela rekommendationsrutan skall ligga inom ett html-element, typiskt en `<div>`. Den skall ha en unik id som gör att rekommendationsmotorn kan hitta mallen, och den göms genom att sätta `style="display:none"` så att den inte visas förrän rekommendationerna är laddade:

{% highlight html %}
<div id="rw_bought_this_also_bought" style="display:none">
  ...
</div>
{% endhighlight %}

De rekommendations idn som normalt används för de olika rekommendationstyperna är `rw_shuffled_toplist`, `rw_viewed_clicktrail_also_viewed`, `viewed_this_also_viewed` och `bought_this_also_bought`.

Om man vill ha flera rekommendationer på en sida så lägger man in flera mallar.


## RWItemTemplate

I templaten skall det finnas ett element med klassen `RWItemTemplate`. Det är det blocket som kommer upprepas för varje produkt som rekommenderas:

{% highlight html %}
<div class="RWItemTemplate">
  <h4><a href="%product_url%">%title%</a> %current_price%</h4>
</div>
{% endhighlight %}

I koden använder man namngivna variabler för att infoga text och bild om produkten. Variabeln består av variabelns namn omgivet av två procenttecken. Exempelvis `%title%` kommer i ovanstående exempel bytas ut mot titeln för en viss produkt i rekommendationen.

De variabler som finns att välja ifrån är `%title%`, `%sku%`, `%description%`, `%product_url%`, `%image_url%`, `%thumpnail_url%`, `%current_price%` och `%normal_price%`.

Alla variabler som klipps in är omkodade så de inte påverkar html-strukturen. Exempelvis om en titel i prisfilen är satt till `Cykel<br>Cresent` så kommer det infogas som `Cykel&lt;br&gt;Cresent` vilket kanske inte är vad man ville i det fallet. I så fall kan man lägga till `-raw` i slutet av variabelnamnet för infoga värdet utan att omkoda det. Så om användaren är tänkt att kunna lägga in html när han editerar produkten så får man skriva (observera att det står `%title-raw%):

{% highlight html %}
<div class="RWItemTemplate">
  <h4><a href="%product_url%">%title-raw%</a> %current_price%</h4>
</div>
{% endhighlight %}

Det finns också en speciell variabel som heter `%ordinal%` vars värde blir vilken i ordningen produkten är. Så den första produkten i listan av rekommendationer får ett `%ordinal%` som är 1, nästa får ett som är 2, osv.


## RWItemSeparator

Ibland kan det vara bra att kunna infoga html mellan, men bara mellan, produkterna. Det gör man genom att markera ett element med klassen `RWItemSeparator`:

{% highlight html %}
<div class="RWItemSeparator">
  <hr />
</div>
{% endhighlight %}

Så har man bara en produkt som rekommenderas så kommer ´RWItemSeparator´-blocket inte alls med, och har man två så kommer den med emellan `RWItemSeparator`-blocken.

