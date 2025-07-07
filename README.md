
# PS4 WebKit PoC – Out-of-Bounds Write

## ¿Qué causa el bug realmente?

La raíz del bug es una escritura fuera de límites (OOB write) en un array optimizado por el compilador JIT de WebKit:

```javascript
let victim = [1.1, 2.2, 3.3, 4.4];
victim.length = 1;
victim[3] = 13.37; // fuera de rango
```

Esta escritura fuera del length, pero dentro de la capacidad del array, provoca corrupción de memoria silenciosa.

---

## ¿Y qué tiene que ver fullscreen + reload?

El PoC no ejecuta `location.reload()` automáticamente. Lo que hace es activar `fullscreen` después de escribir fuera de los límites de un array.

La pantalla completa no causa el bug directamente, pero sí actúa como disparador del acceso a memoria corrupta.

Cuando el usuario manualmente **refresca la página** (por ejemplo, presionando OPTIONS → "Actualizar página" en PS4), el navegador reorganiza layouts, buffers o ejecuta el recolector de basura (GC), y accede a estructuras de memoria previamente alteradas.

Ese acceso inesperado provoca el crash.

---

## Esto puede involucrar:

- Reasignación de backing stores de arrays
- Recolección de basura (GC)
- Cambios en el compositor gráfico

---

## Conclusión técnica

- El bug está en el motor JIT de JavaScript (WebKit), no en `fullscreen`.
- El cambio a pantalla completa y la recarga **manual** solo disparan el acceso a la memoria corrupta.

---

## ¿Está relacionado con algún CVE?

No. Este PoC **no está relacionado con ningún CVE conocido** ni utiliza ningún bug documentado.

Su funcionamiento se basa en el comportamiento observable del motor JIT de WebKit (usado por el navegador de PS4), donde una escritura fuera de límites puede pasar desapercibida tras la optimización JIT.

---

## ¿Es un exploit?

No.

Este PoC **no proporciona ejecución de código** ni control total del sistema. Pero **sí demuestra corrupción de memoria no autorizada**, lo cual es un requisito importante en la creación de exploits reales.

---

## ¿Qué faltaría para convertirlo en un exploit real?

- Lectura arbitraria (OOB read)
- Primitivas de tipo (type confusion)
- Control de estructuras internas
- Bypass de ASLR / DEP
- ROP chain

Este PoC es **solo un primer paso**, pero importante: demuestra que hay comportamiento anómalo en la gestión de memoria bajo ciertas condiciones.

---

## Resumen técnico

Estamos corrompiendo memoria del motor JavaScript con un acceso fuera de límites optimizado por JIT.  
Se realiza un heap spray de 100.000 buffers con el patrón `0x43434343`.  
Cuando el navegador cambia de estado (pantalla completa) y **el usuario recarga manualmente la página**, accede o libera esa memoria...  
Y entonces crashea el motor WebKit de PS4.
