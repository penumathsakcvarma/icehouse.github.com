---
title: Tagging
layout: wikistyle
---

# Generellt tagging script (WIP)

{% highlight javascript %}
<script type="text/javascript">
  var _rwq = _rwq || [];
  _rwq.push(['_setClientID', '234-234-234-234']); // ClientID som funkar över flera produkter (biden/dynad/etc.)
  _rwq.push(['_setScriptID', '123-123-123-123']); // Endast för loggning. Kanske _setDebugText istället?
  _rwq.push([... bla bla bla ...]);

  // Ladda vårt script. ETT script bara. Inte jQuery-overhead.
  // Det behöver inte vara async som GA är, men det kanske är en bonus?
  // Automatiskt HTTP vs HTTPS.
  (function() {
    var rw = document.createElement('script'); rw.type = 'text/javascript'; rw.async = true;
    rw.src = ('https:' == document.location.protocol ? 'https://' : 'http://') + 'releware.net/rw.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(rw, s);
  })();
</script>
{% endhighlight %}

Asynkronisiteten är smidigt, så jag tycker att vi ska försöka sikta på det.

# Dynamisk data för retargeting och/eller dynamiska annonser

{% highlight javascript %}
_rwq.push(
  ['_setVisitorData', 'last-visited', '$timestamp'],
  ['_setVisitorData', 'gender', 'male'],
  ['_setVisitorData', 'looking-for', 'female']
]);
{% endhighlight %}

* Bara strängvärden är tillåtna (det ska skickas som URL-parametrar och flashvars)
* `$timestamp` evalueras på klienten och expanderas till `2012-01-23T16:45:34Z`

### På dynad

Enklast är kanske att flashvars alltid innehåller all data vi känner till om
personen. Ett alternativ är att man lagrar flashvars formatet för annonsen i
databasen vilket ger mer frihetsgrader.

{% highlight javascript %}
:flashvars => { "looking-for" => "$(looking-for)", "gender" => "$(gender)" }
{% endhighlight %}

Det är sedan flash-annonsens ansvar att göra något vettigt av flashvars. Exempelvis
kan flashen fråga efter en produktfeed via ett API, t.ex.

    http://api.spraydate.se/query?gender=female&looking-for=male

**N.B.** För att få detta att fungera måste spraydate implementera såväl API:t som en
[cross-domain policy](http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html)
för flash-annonsen.

### På biden

{% highlight ruby %}
:on_cookies => '$(last_visited) > 30.days.ago.utc.iso8601 && $(gender) == "male"'
{% endhighlight %}

Eftersom det bara är Nils och vi än så länge som ska editera så kan man köra
`eval` rakt av på strängen. Om/när vi ska göra biden self-serve så måste vi
göra någonting annat. Uttrycket evalueras i butikens egna kontext. Således har
varje butik (obs, flera kampanjer per butik!) en egen "visitor-store", men som
givetvis kan *lagras* under samma nyckel i membase.

E.g.

{% highlight ruby %}
class Bidder
  def retarget?
    shop_visitor_data = draft.visitor.shop_data(campaign.shop)

    expr = campaign.on_cookies.gsub(/'$\(([^\)]*)\)'/) do
      shop_visitor_data[$1]
    end

    eval expr
  end
end
{% endhighlight %}

**N.B.** Initialt finns inget stöd för att låta en klient använda sig av information
från en annan, men jag tror inte att det är svårt att implementera i
efterhand. Hur viktigt är det?

# View tracking

Här hittade jag ingen funktionalitet från GA som passade rakt av, så jag föreslår följande:

{% highlight javascript %}
_rwq.push(['_trackItemview',
           'DD44' // SKU/code - required
]);
{% endhighlight %}

I ett första steg kommer den här information att skickas till `log.releware.net`
(för att skapa viewed-viewed modeller, etc.) och `api.releware.net` (för att
lagra view trail i användarens kakor), på exakt samma formatet som gäller
nu.

**N.B.** Biden kommer alltså inte ha tillgång till view-trail i ett första skede.

Senare kan man:

* Flytta view trail till VisitorStore
* Skicka view trail till flash-annonserna som sedan frågar dumbo om
  rekommendationer för att ersätta / migrera över från den befintliga lösningen
  där dynad frågar smartass direkt. Detta skulle kräva lite crossdomain
  policies hos oss, men det fixar vi lätt.

# Conversion tracking

Här föreslår jag att vi helt enkelt kopierar [Google Analytics API](http://code.google.com/apis/analytics/docs/tracking/gaTrackingEcommerce.html) rakt av.

#### Fördelar:

* Ett API som folk känner igen och antagligen redan har skrivit kod för
* Lättare att kopiera befintligt än att hitta på eget

#### Nackdelar:

* Ganska många grejer som är required, men vi kan kanske ta bort några?

### Exempel

* `_addTrans` blir ersättaren till window.rwValue = 11.99;
* order ID ger oss möjligheten till deduplicering
* Resten av argumenten är bara för kompatibilitetens skull
* Man skulle kunna tänka sig att låta order ID och total vara optional i vårt API om man vill.

{% highlight javascript %}
_rwq.push(['_addTrans',
  '1234',            // order ID - required
  'onlinebutik.se',  // affiliation or store name
  '11.99',           // total - required
  '1.29',            // tax
  '5',               // shipping
  'San Jose',        // city
  'Västra Götaland', // state or province
  'Sverige'          // country
]);
{% endhighlight %}

* \_addItem ger oss konverteringsinformation på item-nivå. 
  Det kan vi använda till rekommendationsmotorn för bought-bought
* Här kan vi låta allt utom SKU vara optional

{% highlight javascript %}
_rwq.push(['_addItem',
  '1234',           // order ID - required
  'DD44',           // SKU/code - required
  'T-Shirt',        // product name
  'Green Medium',   // category or variation
  '11.99',          // unit price - required
  '1'               // quantity - required
]);
_rwq.push(['_trackTrans']); //submits transaction to the Releware servers
{% endhighlight %}

Den här informationen kan vi använda oss av vid bought-bought modeller samt
för att räkna conversion value.

--

# Nils ursprungliga mail och story-beskrivning

    Petter bad om use-case för de senaste retargeting-taggarna...

    1. Nils snickrar ihop ett par olika script-taggar baserat på vårt
    "api" och mailar till Spraydate

    2. Spraydate lägger in scripten på sina olika sidor

    3. En besökare går in på Spraydate och söker på Män, åldern 18-27, i
    Kronobergs län

    4. På resultatsidan ligger vårt script, som matas med rätt värden av
    Spraydates back-end, och skickar vidare informationen till oss
    (Biden?)

    - spraydate:outside=1
    - spraydate:looking-for=man
    - spraydate:looking-for-age=18-27
    - spraydate:looking-for-county=kronoberg

    I bangerheads fall skulle vi istället sätta:

    - bangerhead:visit-produkt-id=1231 (eller nått liknande, som
    underhåller ett viewtrail på nått vänster...)

    Eller på Dustin:

    - dustin:visit-category=Mobiltelefoner
    - dustin:visit-brand=Samsung

    4.5. När besökaren har loggat in, så sätts denna kaka via en annan
    script-tagg vi gett dem:

    - spraydate:inside=1 med expiry=30 dagar

    5. Keynote, som vill annonsera för Spraydates räkning, har bett oss
    lägga in en kampanj som riktar in sig mot alla som sökt på män, men
    inte loggat in.

    - Vi skapar kampanjen i Biden som vanligt
    - Som "Retargeting Cookie Name" sätter vi: spraydate:looking-for=man
    - Som "Blocking Cookie Name" sätter vi: spraydate:inside=1 (dvs, vi
    blockar alla som loggat loggat in de senaste 30 dagarna)

    6. Keynote har designat en dynamisk flash-annons, och gett oss den
    (oklart om vi kommer hosta den, eller om de faktiskt har en adserver
    också, men det verkar som vi kommer behöva möjlighet att ladda upp de
    själva), i javascript-koden som embeddar flashen finns ett par
    expansion-macron för dynamiska variabler:

    - $$spraydate:looking-for$$
    - $$spraydate:looking-for-county$$

    Som tex används för att sätta titel: "Nya singlar i
    $$spraydate:looking-for-county$$ län"

    Samt mata flashen med en dynamisk feed:

    http://api.spraydate.se/query?gender=$$spraydate:looking-for$$&county=$$spraydate:looking-for-county$$&age=$$spraydate:looking-for-age$$

    Via "params" som dynad api:t redan gör... men det är ju upp till var
    man vill lägga in expansion-taggarna i javascript-taggen...

    I fallet med Bangerhead/Dustin så skulle feeden istället kunna peka på
    Dynad/Smartass, och vi expandar: $$bangerhead:viewtrail$$

    7. ...

    8. Profit!

# Förslag

Såhär hade varit drömmen att kunna jobba med vårt tracking-script 2.0... Baserat på hur GA gör det: 

{% highlight javascript %}
<script type="text/javascript">
  var _rwq = _rwq || [];
  _rwq.push(['_setClientID', '234-234-234-234']);
  _rwq.push(['_setScriptID', '123-123-123-123']);
  _rwq.push(['retarget', '<cookie-name>', '<lifetime in days or forever>']);
  _rwq.push([... bla bla bla ...]);

  // Ladda vårt script. ETT script bara. Inte jQuery-overhead.
  // Det behöver inte vara async som GA är, men det kanske är en bonus?
  // Automatiskt HTTP vs HTTPS.
</script>
{% endhighlight %}

## Exempel på olika event

### Basic Retargeting

{% highlight javascript %}
_rwq.push(['_setClientID', '234-234-234-234']);
_rwq.push(['_setScriptID', '123-123-123-123']); // för att kunna tracka olika script och se att de är inlagda/levererar data, UUID
_rwq.push(['retarget', 'spraydate-has-logged-in']); // har loggat in, keep forever
_rwq.push(['retarget', 'spraydate-last-login', '30']); // "session" för senaste login, keep 30 dagar, för att kunna göra "påminnelser" efter 30 dagar
{% endhighlight %}

### Dynamisk retargeting

{% highlight javascript %}
_rwq.push(['_setClientID', '234-234-234-234']);
_rwq.push(['_setScriptID', '123-123-123-123']);
_rwq.push(['_setShopID', '345-345-345-345']); // kanske redundant om vi har client-id
_rwq.push(['retarget', '<clientid>-dynamic']); // targeting på dynamisk data
_rwq.push(['trackProduct', '12312312311']); // product-id
{% endhighlight %}

### Konvertering på Dynamisk retargeting

{% highlight javascript %}
_rwq.push(['_setClientID', '234-234-234-234']);
_rwq.push(['_setScriptID', '123-123-123-123']);
_rwq.push(['_setShopID', '345-345-345-345']); // kanske redundant om vi har client-id
_rwq.push(['retarget', '<clientid>-converted', 30]); // har köpt, 30-dagars timeout
_rwq.push(['trackOrderID', '12312312311']); // Order-id om det finns, så vi kan undvika dubletter
_rwq.push(['trackOrderValue', '1234.50', 'SEK']); // Ordervärdet...
{% endhighlight %}

### Dynamisk data

{% highlight javascript %}
_rwq.push(['_setClientID', '234-234-234-234']);
_rwq.push(['_setScriptID', '123-123-123-123']);
_rwq.push(['setData', 'gender', 'male']);
_rwq.push(['setData', 'age', '28']);
_rwq.push(['setData', 'hometown', 'Stockholm']);
{% endhighlight %}

Data som sätts skall vara tillgänglig för att mata flash ads med via JS som Dynamisk Retargeting gör.

Det hade också varit bra om de automatiskt satte retargeting-cookies, eg: gender-male, age-18-35 för vissa parametrar, annars får man koda det i kön men det är större risk att det blir fel.

Funkade det så så skulle vi lätt kunna sätta ihop alla script som behövs för annonsörerna utan att behöva göra ett stort projekt av det.



