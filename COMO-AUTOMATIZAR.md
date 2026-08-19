# Cómo hacer que se actualice solo

Primero lo importante: **un archivo HTML no puede actualizarse a sí mismo.** Los
datos van escritos dentro de él, que es justo lo que lo hace funcionar sin
internet y sin instalar nada. Para que se actualice solo hace falta *algo* que
corra `actualizar.py` cada cierto tiempo. Hay tres formas, de más a menos
recomendada.

---

## Opción A — GitHub Actions + GitHub Pages (la buena)

Queda una **liga pública que siempre muestra los datos frescos**. No necesita
que tu computadora esté prendida. Es gratis. Y resuelve de paso lo de
compartir: mandas la liga en vez de un archivo.

1. Crea un repositorio en GitHub (puede ser privado; Pages funciona en
   privados solo con cuenta de pago, así que para la liga pública hazlo
   público — los datos son de fichas públicas, no hay nada tuyo dentro).
2. Sube estos archivos, respetando las carpetas:

   ```
   actualizar.py
   datos.json
   renta_fija.html
   .github/workflows/actualizar.yml
   ```

3. En el repo: **Settings → Pages → Source: GitHub Actions**.
4. Opcional pero recomendado, para que también refresque CETES y el Bono M:
   saca un token gratis en
   <https://www.banxico.org.mx/SieAPIRest/service/v1/token>
   y guárdalo en **Settings → Secrets and variables → Actions → New
   repository secret**, con el nombre `BANXICO_TOKEN`.
5. Ve a la pestaña **Actions**, elige el workflow y dale **Run workflow** para
   probarlo ya. Si todo salió bien, tu liga queda en
   `https://TU-USUARIO.github.io/TU-REPO/`

A partir de ahí corre **cada lunes a las 7:00 de CDMX**. Para cambiar la
frecuencia, edita la línea `cron:` del archivo `.github/workflows/actualizar.yml`:

| Cuándo | Línea cron |
|---|---|
| Lunes 7:00 CDMX | `0 13 * * 1` |
| Diario 7:00 CDMX | `0 13 * * *` |
| Día 1 de cada mes | `0 13 1 * *` |

El horario va en UTC. CDMX es UTC−6, así que súmale 6 a la hora que quieras.

**Si el raspador se rompe te enteras.** El workflow corre con
`--minimo-ok 30`: si menos de 30 de los 39 fondos se pudieron leer, el job
falla y GitHub te manda un correo. El HTML se regenera de todos modos con los
últimos valores buenos, así que la página nunca se queda en blanco.

---

## Opción B — Tarea programada en tu computadora

Más simple, pero solo corre cuando tu máquina está prendida, y el resultado
vive en tu disco (no hay liga que compartir).

**Windows** — abre PowerShell en la carpeta y pega:

```powershell
$ruta = (Get-Location).Path
schtasks /create /tn "Actualizar renta fija" /tr "python `"$ruta\actualizar.py`"" /sc weekly /d MON /st 07:00
```

**macOS o Linux** — `crontab -e` y agrega:

```
0 7 * * 1 cd /ruta/a/la/carpeta && /usr/bin/python3 actualizar.py >> actualizar.log 2>&1
```

Para que también refresque CETES, exporta el token antes:
`BANXICO_TOKEN=tu_token python actualizar.py`

---

## Opción C — Sin GitHub y sin dejar la compu prendida

Una tarea programada en Claude que cada semana baje los datos, regenere el
archivo y te lo mande. No necesitas repositorio ni saber de Git, pero el
resultado te llega como archivo en una conversación, no como liga fija.
Pídemelo y la dejo configurada.

---

## Lo que ninguna opción arregla

- **Si un emisor rediseña su ficha**, el raspador deja de leer ese fondo. Por
  eso el script conserva el último valor bueno y avisa cuál falló, en vez de
  escribir basura. Habrá que ajustar las etiquetas en `ETIQUETAS`.
- **La inflación y las tasas mexicanas** salen de Trading Economics y Banxico;
  si cambian el formato de su página o de la API, aplica lo mismo.
- **El supuesto de depreciación del peso** no es un dato que se pueda bajar:
  es tuyo. Ninguna automatización lo va a decidir por ti.
- **Vanguard publica por trimestre.** Aunque corras esto a diario, sus 12
  fondos solo cambian cuatro veces al año.
