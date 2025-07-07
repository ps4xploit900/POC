# PS4 WebKit PoC – Memory Crash via JIT OOB Write

---

## **¿Qué causa el bug realmente?**

La raíz del bug es una **escritura fuera de límites (Out-of-Bounds Write, OOB)** en un array optimizado por el compilador **JIT (Just-In-Time)** de **WebKit**, el motor JavaScript del navegador de PS4.

```js
let victim = [1.1, 2.2, 3.3, 4.4];
victim.length = 1;
victim[3] = 13.37; // fuera de rango (length = 1, capacidad interna = 4)
```

Esta escritura fuera del `length`, pero **dentro de la capacidad interna del array**, provoca una **corrupción de memoria silenciosa**. El JIT ya ha optimizado ese acceso como válido.

---

## **¿Y qué tiene que ver fullscreen + reload()?**

La pantalla completa **no causa el bug directamente**, pero sí actúa como **disparador del acceso a memoria corrupta**.

```js
document.body.requestFullscreen();
location.reload();
```

Este cambio de estado gráfico hace que el navegador:

- Reasigne backing stores de arrays
- Realice recolección de basura (GC)
- Modifique composiciones internas del layout/renderizado

Cuando accede a la memoria ya corrupta por el OOB write, el navegador **crashea**.

---

## **Comportamiento técnico resumido**

- Se realiza un **heap spray** de **100.000 buffers** con el patrón `0x43434343`.
- Se entrena un array optimizado por JIT y luego se escribe fuera de sus límites (**OOB write**).
- El navegador, al entrar en **pantalla completa** y luego hacer un **reload()**, accede accidentalmente a esa memoria dañada.

---

## **Conclusión técnica**

- El bug está en el **motor JIT de WebKit** (JavaScript).
- El **fullscreen y reload** son solo triggers para exponer el acceso ilegítimo a memoria.
- No es un CVE, ni un exploit funcional con ejecución de código.
- Pero demuestra una **condición de corrupción de memoria** crítica.

---

## **Disclaimer**

Este PoC **no está relacionado con ningún CVE conocido** ni utiliza ningún bug documentado previamente.

Su propósito es **educativo y de investigación**, mostrando un comportamiento anómalo del motor JavaScript de PS4 mediante acceso fuera de límites tras optimización JIT.

---

## **¿Y para que esto sea explotable de verdad?**

Se requerirían etapas adicionales como:

- Lectura arbitraria (OOB read)
- Type confusion
- Control de punteros
- Bypass de ASLR / DEP
- Construcción de una cadena ROP (Return-Oriented Programming)

Este PoC solo representa el **primer paso** en la cadena de explotación.

---

## **Explicación técnica resumida**

Estamos escribiendo **fuera de los límites de un array optimizado por JIT (OOB write)**, modificando memoria que no deberíamos tocar.  
El navegador **no crashea de inmediato**, pero al cambiar a **fullscreen + reload**, **accede o libera esa memoria y falla**.

---

## **TL;DR**

- Heap sprayed: **100.000 buffers**
- Patrón: `0x43434343`
- Trigger: `fullscreen + reload()`
- Resultado: **PS4 WebKit crash controlado**
