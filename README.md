
# PHP SDK für Globalsys EDC API 
Ein PHP SDK für die Kommunikation mit der Globalsys EDC API.
## Erste Schritte
Kopiere zuerst den `src` Ordner in deinen Projektordner. Dann erstelle eine PHP-Datei und binde die enthaltene `autoload.php` Datei ein. Alle benötigten Klassen werden nun automatisch aus dem SDK geladen.
```php
<?php
include 'src/autoload.php';
```
Der nächste Schritt ist die Authentifizierung. Dazu wird mit den Zugangsdaten bei der EDC API ein AccessToken abgefragt, mit welchem alle weiteren Anfragen authentifiziert werden.
```php
$apiUrl    = 'YOUR_API_URL';
$apiUser   = 'YOUR_API_USER';
$apiSecret = 'YOUR_API_SECRET';

$requestHandler = new \Globalsys\EDCSDK\Utils\RequestHandler($apiUrl);

$auth = new \Globalsys\EDCSDK\Request\Auth($apiUser, $apiSecret);
$authResponse = $requestHandler->execute($auth);
$authContent = $authResponse->getContent();

if ($authResponse->getHttpCode() != 200) {
  // Output authentication error
  echo $authContent['message'];
} else {
  $accessToken = $authContent['token'];
  // Now you can do your api requests here
  // ...
}
```
## Beispiel Anfrage
Als Beispiel holen wir uns nun alle Bestellungen, die in den letzten drei Stunden geändert (versendet, bearbeitet oder storniert) wurden. Um die Datenmengen im Rahmen zu behalten, werden die Antworten von der API in mehrere Pakete aufgeteilt. Diese rufen wir dann nacheinander ab, um alle Bestellungen zu erhalten.
```php
// Get all orders changed (shipped, modified, canceled) in the last 3 hours
$getOrders = new \Globalsys\EDCSDK\Request\GetOrders;
$getOrders->setChangedFrom('-3 HOURS'); // use strtotime param values

// Send request
$getOrdersResponse = $requestHandler->execute($getOrders, $accessToken);

// Handle response
$productPagesLeft = true;
while ($productPagesLeft && $getOrdersResponse->getHttpCode() == 200) {
  $responseContent = $getOrdersResponse->getContent();
  foreach ($responseContent['order'] as $order) {
    // do stuff with order here
    // ...
  }
  // If there a any pages left, get the next page of orders
  if ($responseContent['information']['currentpage'] < $responseContent['information']['totalpages']) {
    $getOrders->setCurrentPage($responseContent['information']['currentpage'] + 1);
    $getOrdersResponse = $requestHandler->execute($getOrders, $accessToken);
  } else {
    $productPagesLeft = false;
  }
}
```
Weitere Beispiele zu allen API Anfragen findest du im Ordner `examples`.
## Lizenz

Dieses Projekt steht unter der MIT Lizenz - mehr Infos in der [LICENSE.md](LICENSE.md).
