---
title: Tagging
layout: wikistyle
---

# Releware tracking API

Releware's tracking API har två användningsområden:

1. Add märka besökare av en sajt för att sedan kunna visa annonser bara för
vissa grupper, så kallad *retargeting*.
2. Att mäta vilka produkter besökarna tittar på för att kunna ge dem
personliga annonser, så kallad *dynamisk retargeting*.

I båda fallen finns möjlighet att dessutom mäta antal konverteringar inklusive
belopp.

## Märka besökare

Man kan märka besökare både med namn (`_setVisitorTag`) och med namn kopplat
till värde (`_setVisitorData`). Man kan också i båda fallen sätta en livslängd
på data givet i antal dagar:

{% highlight html %}
<script type="text/javascript">
  var _rwq = _rwq || [];

  _rwq.push(['_setClientID', 'PellesDatorer_pellesdatorer.se']);
  _rwq.push(['_setScriptID', '68b7d9a0-6dab-11e1-b0c4-0800200c9a66']);

  // Märk besökaren som en man.
  _rwq.push(['_setVisitorData', 'gender', 'male']);
  // Märk att besökaren har tittat på en viss kategori, men spara bara i 20 dagar.
  _rwq.push(['_setVisitorData', 'kategory', window.shopCategory, '20']);

  // Märk besökaren som en registrerad användare.
  _rwq.push(['_setVisitorTag', 'registered-user']);
  // Märk besökaren när han loggar in, men spara bara i en vecka.
  _rwq.push(['_setVisitorTag', 'recently-logged-in', '7']);

  (function() {
    var rw = document.createElement('script'); rw.type = 'text/javascript'; rw.async = true;
    rw.src = ('https:' == document.location.protocol ? 'https://' : 'http://') + 'www.releware.net/js/v1/rwa.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(rw, s);
  })();
</script>
{% endhighlight %}

Om man inte sätter en livslängd på data eller en tag så sparas de så länge som data om användaren finns kvar.


## Mäta vilka produkter besökaren tittar på

För att det skall gå göra personliga annonser så behöver man mäta vilka
produkter en besökare tittar på. Så på produktens egen sida så får man lägga
till skriptet med ett mätkommando (`_addItemView`) och produktens unika id:

{% highlight html %}
<script type="text/javascript">
  var _rwq = _rwq || [];

  _rwq.push(['_setClientID', 'PellesDatorer_pellesdatorer.se']);
  _rwq.push(['_setScriptID', '68b7d9a0-6dab-11e1-b0c4-0800200c9a66']);

  // Registrera att besökaren tittar på produkten med id 'ref_1234'.
  _rwq.push(['_addItemView', 'ref_1234']);

  // Passa på att tagga besökaren på samma gång.
  _rwq.push(['_setVisitorTag', 'registered-user']);

  (function() {
    var rw = document.createElement('script'); rw.type = 'text/javascript'; rw.async = true;
    rw.src = ('https:' == document.location.protocol ? 'https://' : 'http://') + 'www.releware.net/js/v1/rwa.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(rw, s);
  })();
</script>
{% endhighlight %}

Produktens id måste vara unikt, det vill säga olika produkter inte får ha
samma id. Om en produkt finns i flera varianter (exempelvis i olika färger
eller storlekar) så skall de ha samma id. Id:t måste vara exakt samma som i
prisfilen som används vid dynamisk retargeting.


## Registrera konverteringar

Det är användbart att mäta antalet konverteringar och totala
konverteringsvärdet både för retargeting och dynamisk retargeting. Lägg helt
enkelt in skriptet på landningsidan efter att besökaren har bekräftat köpet:

{% highlight html %}
<script type="text/javascript">
  var _rwq = _rwq || [];

  // För att bibehålla bakåtkompatibilitet så har vi <account>_<domännamn>
  _rwq.push(['_setClientID', 'PellesDatorer_pellesdatorer.se']);
  _rwq.push(['_setScriptID', '68b7d9a0-6dab-11e1-b0c4-0800200c9a66']);

  // _addConversion med order id, belopp och ordertyp. Alla parametrarna är valfria.
  _rwq.push(['_addConversion', 'id1234', '150.00', 'purchase']);

  // Det är vanligt att tagga att besökaren har köpt för då kan man välja
  // att inte längre visa en kampanj för den besökaren.
  _rwq.push(['_setVisitorTag', 'registered-user']);

  (function() {
    var rw = document.createElement('script'); rw.type = 'text/javascript'; rw.async = true;
    rw.src = ('https:' == document.location.protocol ? 'https://' : 'http://') + 'www.releware.net/js/v1/rwa.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(rw, s);
  })();
</script>
{% endhighlight %}

Om man bara vill räkna antalet konverteringar behöver man inte ge några
argument (`_rwq.push(['_addConversion']);`). Finns det ändå ett unikt order id
så rekommenderar vi ändå att det anges för att undvika att vi räknar
konverteringar dubbelt om sidan av någon anledning skulle laddas om
(`_rwq.push(['_addConversion', 'order_id_1234']);`).

Om man anger belopp så är det viktigt att siffran har samma
valuta varje gång. Releware håller bara reda på totala siffran och känner inte
till valutan själva. Siffran skall anges som en sträng med en punk och två
siffror efter punkten. `'23.00'` är rätt men `'23'` och `'23,00'` är fel.

Om man vill ange belopp men inte order id så kan man sätta `null` i stället
för order id (`_rwq.push(['_addConversion', null, '23.50']);`).


## Saker att tänka på

* Skriptet bör placeras i slutet av huvudet i HTML-koden precis före `</head>`
för bästa prestanda. Eftersom skriptet är litet och laddas asynkront så
påverkar det laddningstiden av sidan minimalt.

* Placera bara ett skript per sida.

* `_setClientID` måsta man alltid ange. Värdet fås av Releware och är en
kombination av konto- och sajtid-

* `_setScriptID` är valfritt men används för att se att ett viss skript har
dykt upp rätt i vårt system.

* Alla värden ges som strängar, inklusive siffror som hur länge en märkning
skall finnas kvar eller vilket belopp konverteringen hade.


## Felsökning

Skriptet har en inbyggd felsökningsfunktion som hjälper dig att se att
skriptet har laddats som det skall, vilka kommandon den har tagit emot, och
vilka fel den har registrerat. Skriv `javascript:_rwq.showLog()` i
webbläsarens adressfält för att se informationen.
