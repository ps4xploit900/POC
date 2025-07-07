# POC
poc webkit ps4 memory



- ¿Qué causa el bug realmente?
 La raíz del bug es una escritura fuera de límites (OOB write) en un array optimizado por el compilador JIT de WebKit:


let victim = [1.1, 2.2, 3.3, 4.4];
victim.length = 1;
victim[3] = 13.37; // fuera de rango
Esta escritura fuera del length, pero dentro de la capacidad del array, provoca corrupción de memoria silenciosa.

- ¿Y el fullscreen + reload()?
La pantalla completa en sí no causa el bug, pero es un trigger útil para exponerlo.

- Cuando se ejecuta:

document.body.requestFullscreen();
location.reload();
El navegador cambia de estado gráfico, reorganiza layouts y buffers, y toca la memoria ya corrompida por el OOB.

- Esto puede involucrar:

 .Reasignación de backing stores de arrays

. Recolección de basura (GC)

. Cambios en el compositor gráfico

Y entonces, peta (crashea)

 - Conclusión técnica:
El bug está en el motor JIT de JavaScript (WebKit), no en fullscreen.
El cambio a pantalla completa y la recarga solo disparan el acceso a la memoria corrupta.


- Estamos corrompiendo memoria del motor JavaScript con un acceso fuera de límites optimizado por JIT.
- El navegador no crashea de inmediato, pero al cambiar a pantalla completa y recargar, accidentalmente accede a esa memoria...




------------------------------------------------------------------------------------------------------------------------------------
🧠 Estamos escribiendo fuera de los límites de un array optimizado por JIT (OOB write), modificando memoria que no deberíamos tocar.

🔁 Se realiza un heap spray de 100.000 buffers con el patrón 0x43434343.

Cuando el navegador cambia de estado (📺 fullscreen + 🔄 reload), intenta acceder o liberar memoria ya corrupta…

💥 Resultado: crash controlado del navegador WebKit en PS4.
