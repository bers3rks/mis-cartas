# Mis Cartas 💌

App web pequeña y personal para que mamá lleve el control de su trabajo escribiendo y enviando cartas a casinos:

- 📦 **Material y gastos**: inventario (hojas, sobres, estampillas, lapiceros…) y todo lo que gasta.
- 🍀 **Chumba Casino**, ⭐ **LuckyLand**, ♠️ **Global Poker**: cartas enviadas, pago por carta, pagos recibidos y cuánto le deben.
- 📊 **Resumen**: inversión, cobrado, pendiente y ganancia total.

Es **un solo archivo** (`index.html`), sin servidor, sin cuentas y sin base de datos. Los datos se guardan **solo en el teléfono** de quien la usa.

**Dirección de la app:** https://bers3rks.github.io/mis-cartas/

---

## 📱 Cómo instalarla en el iPhone de mamá

1. Abrir **Safari** (tiene que ser Safari, no Chrome) y entrar a la dirección de arriba.
2. Tocar el botón **Compartir** (cuadrito con flecha hacia arriba).
3. Elegir **"Agregar a pantalla de inicio"** → **Agregar**.
4. Desde ahora abrirla **siempre desde ese ícono**.

> ⚠️ Importante: en iPhone, si la app se usa solo desde Safari (sin agregarla a la pantalla de inicio), el sistema puede borrar los datos tras unos días sin uso. Desde el ícono de inicio no pasa.

En **Android** (Chrome): menú ⋮ → **"Agregar a pantalla principal"** / "Instalar app".

## 💾 Copias de seguridad (muy importante)

Los datos viven solo en el teléfono. Si se borra el navegador, se cambia de teléfono o se pierde, **se pierde todo** salvo que haya copia.

- En **Resumen** → **"Guardar copia"**: genera un archivo `mis-cartas-copia-AAAA-MM-DD.json` y abre el menú de compartir. Lo ideal: mandárselo por WhatsApp o correo a sí misma (o a su hijo).
- La app avisa si pasan **7 días** sin copia.
- Para restaurar: **Resumen** → **"Recuperar copia"** → elegir el archivo. Reemplaza todos los datos actuales.

## 🧑‍🍳 Cómo se usa (resumen para explicarle)

| Quiero… | Dónde |
|---|---|
| Anotar que compré material | Material → **🛒 Compré material** |
| Anotar una hoja dañada / lapicero acabado | Material → **✏️ Usé material** |
| Anotar gasolina, copias, etc. | Material → **💸 Otro gasto** |
| Anotar cartas que envié | Pestaña del casino → **✉️ Envié cartas** |
| Anotar que me pagaron | Pestaña del casino → **💵 Me pagaron** |
| Corregir o borrar algo | Tocar el registro en la lista |
| Cambiar cuánto material usa cada carta | Material → **⚙️ Cambiar cuánto se usa por carta** |
| Modo claro / oscuro | Botón 🌙/☀️ arriba a la derecha |

El **pago por carta** se escribe a mano en cada envío. Si aún no se sabe, se deja vacío y se completa después tocando el envío.

## 🛠️ Para desarrolladores / IA

Toda la arquitectura, decisiones, modelo de datos y **reglas que no se deben romper** están en **[AGENTS.md](AGENTS.md)**. Leerlo antes de tocar el código.

Probar en local:

```bash
py -m http.server 8765
```

y abrir http://localhost:8765 (abrirlo como `file://` también funciona en un navegador normal, pero algunos visores bloquean `localStorage`).

Publicar cambios: hacer commit y `git push` a `main`. GitHub Pages se actualiza solo en 1–2 minutos.
