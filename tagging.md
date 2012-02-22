---
title: Tagging
layout: wikistyle
---

# Releware tracking API

Releware's tracking API används för att lagra besökares beteende för att kunna ge
riktad reklam om era produkter. För att underlätta har vi modellerat syntaxen och
designen efter Google Analytics.

Med API:t kan du

* Sätta godtyckliga nyckel-värde par och/eller taggar på besökare och använda
  dessa till att rikta enskilda reklamkampanjer (s.k. "retargeting")
* Följa vad besökare tittar på och använda detta för personliga rekommendationer.
* Mäta effekten av dina reklamkampanjer genom att registrera konverteringar
  (t.ex. köp i en e-butik).

# Fullskaligt exempel

{% highlight javascript %}
<script type="text/javascript">
  var _rwq = _rwq || [];

  // För att bibehålla bakåtkompatibilitet så har vi <account>_<domännamn>
  _rwq.push(['_setClientID', 'PellesDatorer_pellesdatorer.se']);
  _rwq.push(['_setScriptID', '123-123-123-123']);

  // _setVisitorData
  _rwq.push(['_setVisitorData', 'gender', 'male']);       // TODO: Lagra data i ? dagar
  _rwq.push(['_setVisitorData', 'forget', 'this', '1']);  // Lagra data i en dag

  // _setVisitorTag
  _rwq.push(['_setVisitorTag', 'registered-user']);          // TODO: Lagra hur länge?
  _rwq.push(['_setVisitorTag', 'recently-logged-in', '30']); // Sätt en "tag" på en användare, lagra i 30 dagar

  // _addItemView
  _rwq.push(['_addItemView', 'ref_1234']);

  // _addConversion
  _rwq.push(['_addConversion',
             'id1234',                                    // Unikt orderID    (valfritt, men rekommenderat)
             '150.00',                                    // Ordervärde       (valfritt)
             'purchase'                                   // Konverteringstyp (valfritt)
            ]);
  _rwq.push(['_addConversion', 'id1235']);                // Inget ordervärde
  _rwq.push(['_addConversion']);                          // Inget konverterings ID
  _rwq.push(['_addConversion', null, null, 'purchase']);  // Bara konverteringstyp. "undefined" funkar också.


  (function() {
    var rw = document.createElement('script'); rw.type = 'text/javascript'; rw.async = true;
    rw.src = ('https:' == document.location.protocol ? 'https://' : 'http://') + 'www.releware.net/js/v1/rwa.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(rw, s);
  })();
</script>
{% endhighlight %}
