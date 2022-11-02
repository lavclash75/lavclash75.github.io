---
title: Discord Open Redirect (Unpatched)
date: 2022-11-02
categories: [bug bounty, discord]
tags: [bug bounty, 0 day] #minúsculas
---
# Discord Open Redirect (Unpatched)

Primer de todo yo no he descubierto esta vulnerabilidad yo solo observe que se estaba explotando como dicen los ingleses in the wild.

Como me doy cuenta de esta vulnerabilidad, un amigo mio recibe el típico mensaje de free nitro que todos hemos recibido alguna vez y me habla al md para decirme si es real yo ya de entrada le digo que es spam, pero me da curiosidad y decido investigar-lo, para recibir el nitro tienes que agregar un bot a un server tuyo yo decido añadirlo en otra cuenta que tengo para probar y en otro server donde esta solo la cuenta esta, cuando añado el bot me doy cuenta que el url es automáticamente redirigido a otra web maliciosa que se parece a la de discord la típica web de phishing. Aquí es donde me doy cuenta y digo uy que raro se ha dirigido automáticamente a la web de phishing y me doy cuenta del open redirect. A continuación vamos a analizar la url para ver el open redirect 

## Análisis :

```javascript
https://discord.com/api/oauth2/authorize?client_id=1022148663725801502&permissions=8&redirect_uri=https%3A%2F%2Ffnomm.com%2Fnekotina&response_type=code&scope=identify%20guilds.join%20bot
```
Primero tenemos el client_id que no es importante
```
client_id=1022148663725801502
```
Luego los permisos del bot aquí podemos ver que es un 8 que es permisos de Administrador en el servidor de Discord
```
permissions=8
```
Open Redirect:
luego en la url tenemos lo siguiente que ya podemos ver que el parámetro redirect uri tiene la url encoded de la web maliciosa 
```
redirect_uri=https%3A%2F%2Ffnomm.com
```
URL decoded quedaría asi
```
https://fnomm.com
```
Lo restante del link es para agregar al bot y enviar un mensaje automático a todas las personas del servidor con el mismo enlace de free nitro.

## Reporte en el programa de bug bounty:
Una vez analizado el URL procedí a enviar el reporte al programa de bug bounty de Discord

En el reporte explico básicamente lo descrito anteriormente y también adjunto un video del proof of concept.

Punto positivo del programa de bug bounty de Discord es que me respondieron el mismo dia que lo reporte.

Punto negativo el programa considero que es social engineering y cerraron el reporte como informativo ya que consideraron que no es una vulnerabilidad y que estaba fuera de scope del programa. Por ultimo quiero añadir que esta bien que tengan programas de bug bounty pero que por ejemplo este en out of scope CSRF, y que social engineering este totalmente en out of scope yo no digo que no tenga que estar por que al final phishing se pude hacer de todo pero aquí estas aprovechando de un error para hacer el phishing y sea mas realista y sigamos sinceros los users no se van a fijar en que el url cambia a otro no malintencionado por que primero ya vieron que es una url de discord.com, al final creo que no es cuestión de si es o no una vulnerabilidad solo hacer el usuario final mas seguro aplicando una mesura. Dicho esto happy hunting 😎
## Programa de bug bounty de Discord 
[https://discord.com/security](https://discord.com/security)
