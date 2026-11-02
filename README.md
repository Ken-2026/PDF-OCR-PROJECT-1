# MiOCRPotente

MiOCRPotente quiere convertirse en un organizador documental sencillo para Windows: recibe documentos, reconoce su contenido, propone un nombre claro y permite encontrarlos después. La idea es que una persona pueda soltar un archivo, revisar qué entendió el programa y guardar el resultado ordenado sin tener que renombrar cada documento a mano.

La versión actual es una primera base funcional, no todavía ese producto completo. El procesamiento OCR y la clasificación se ejecutan localmente; los documentos no se envían a Internet ni hace falta una clave de IA.

## Funciones

- OCR en español e inglés para PNG, JPG/JPEG, TIFF y BMP.
- Clasificación local por reglas: factura, recibo, contrato, documento de identidad, extracto bancario, nómina, informe médico, carta, presupuesto o reserva.
- Añade una fecha encontrada y, en facturas, intenta obtener el proveedor.
- Crea una copia de la imagen con el nombre sugerido y un `.txt` con el texto OCR. El original no se modifica.

La clasificación es heurística. Si el texto no da señales claras, el programa usa `Documento`; no promete reconocer siempre nombres de personas o empresas.

## Requisitos

- Windows 10/11 x64.
- .NET 8 SDK para compilar; .NET 8 Runtime para ejecutar la publicación dependiente del runtime.
- Runtime de Microsoft Visual C++ 2019 x64, necesario para los componentes nativos del wrapper Tesseract.
- Modelos de idioma `spa.traineddata` y `eng.traineddata`.

## Abrir y preparar en Visual Studio Code

1. Instala Visual Studio Code y el SDK .NET 8.
2. Abre la carpeta del repositorio con **Archivo > Abrir carpeta**.
3. En la terminal integrada ejecuta:

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\scripts\Install-Tessdata.ps1
   ```

4. Pulsa `Ctrl+Shift+B` y selecciona **MiOCRPotente: compilar**.

El script descarga `spa` y `eng` desde el repositorio oficial `tessdata_fast`. Los modelos no se guardan en Git; se copian junto al ejecutable al compilar/publicar.

También puedes compilar desde un terminal:

```powershell
dotnet restore MiOcrPotente.sln
dotnet build MiOcrPotente.sln -c Release
dotnet publish MiOcrPotente.csproj -c Release -r win-x64 --self-contained false
```

## Usar

```powershell
MiOcrPotente.exe "C:\Documentos\imagen.jpg"
MiOcrPotente.exe "C:\Documentos\imagen.jpg" --tessdata "C:\Herramientas\tessdata"
```

Se crearán, por ejemplo, `Factura_Empresa_Ejemplo_2026-11-02.jpg` y `Factura_Empresa_Ejemplo_2026-11-02.txt` junto a la imagen. Si ya existe ese nombre, añade `_2`, `_3`, etc.

## Límites actuales

PDF, escaneos multipágina, documentos Office, interfaz gráfica y revisión interactiva del nombre todavía no están implementados. La clasificación usa palabras clave y reglas fijas: no es un modelo inteligente, puede confundirse con documentos poco claros y extrae pocos datos de forma estructurada. Tampoco organiza en carpetas, recuerda preferencias ni ofrece una manera cómoda de corregir una sugerencia incorrecta. Los scripts `.bat` y `.reg` antiguos incluían rutas personales; el `.bat` actual solo delega en el ejecutable.

## Qué debería llegar a ser

El objetivo es una herramienta de escritorio en la que se puedan seleccionar o arrastrar documentos y obtener, en un flujo comprensible:

1. **Lectura fiable:** abrir archivos comunes y aplicar OCR cuando haga falta, tanto a imágenes como a PDF de una o varias páginas.
2. **Comprensión del documento:** proponer el tipo (por ejemplo, factura, recibo, contrato o informe), la fecha y la entidad o persona más relevante, indicando cuando el resultado sea incierto.
3. **Nombre y destino sugeridos:** crear un nombre legible y seguro, por ejemplo `Factura_Compania_Electrica_2026-10-15.pdf`, y sugerir una carpeta según reglas que el usuario pueda configurar.
4. **Revisión antes de guardar:** mostrar el texto y los datos detectados, permitir corregirlos y elegir si se crea una copia o se renombra el original. Nunca sobrescribir documentos silenciosamente.
5. **Uso cotidiano sencillo:** procesar varios archivos por lote, mostrar el progreso y explicar los errores con una acción concreta para resolverlos.
6. **Privacidad y mantenimiento:** funcionar sin conexión para el OCR y las reglas básicas; cualquier servicio de IA externo debe ser opcional, explicar qué datos enviaría y pedir configuración explícita. Mantener instrucciones, compilación y pruebas repetibles.

## Mejoras pendientes y orden recomendado

### Prioridad 1: proteger los archivos y hacer confiable el flujo

- Evitar colisiones tanto en el nombre de la imagen como en el `.txt`; actualmente se calcula el nombre libre solo para la imagen, así que podría existir ya un texto con ese nombre.
- Escribir primero los resultados temporales y moverlos al destino solo si OCR y guardado terminan correctamente; manejar errores de permisos, archivos en uso y rutas largas.
- Añadir opciones para elegir carpeta de salida, idioma y formato de nombre, con valores seguros por defecto.
- Comprobar y explicar la disponibilidad del runtime nativo de Tesseract y de cada modelo de idioma al arrancar.

### Prioridad 2: admitir documentos reales de uso diario

- Añadir lectura de PDF digitales y OCR de PDF escaneados, incluyendo documentos de varias páginas.
- Procesar varios archivos de una carpeta y evitar volver a procesar accidentalmente las copias generadas.
- Extraer fechas con formatos regionales y validar que sean fechas posibles; soportar nombres de empresa, números de factura e importes con más cuidado.
- Añadir pruebas con ejemplos representativos y documentos difíciles (rotados, borrosos, con varias columnas o poco contraste).

### Prioridad 3: facilitar la corrección y mejorar la clasificación

- Crear una interfaz sencilla con selección/arrastre de archivos, cola de trabajo, vista del texto OCR y campos editables para tipo, fecha y entidad.
- Mostrar la confianza o las razones de una clasificación y dejar que la persona acepte o corrija la propuesta antes de guardar.
- Permitir reglas y plantillas personales, por ejemplo los tipos de documentos habituales y el patrón de nombre preferido.
- Mejorar las reglas con ejemplos reales anonimizados. Después, evaluar un modelo local pequeño para clasificar o resumir el texto cuando las reglas no basten.

### Prioridad 4: empaquetar y mantener el programa

- Publicar un instalador o paquete portable que incluya la aplicación y sus dependencias, con instrucciones claras para actualizarla.
- Probar automáticamente compilación y comportamiento básico en GitHub Actions; añadir pruebas de clasificación y del generador de nombres.
- Documentar versiones, licencias y origen de Tesseract y de los modelos OCR; mantener un registro de cambios y versiones publicadas.

La IA no sustituye al OCR: el OCR convierte la imagen en texto; después las reglas o un modelo clasifican ese texto y sugieren los campos. La primera mejora debería hacer seguro y completo ese flujo básico. Una IA generativa puede añadirse más adelante como opción, pero no debe renombrar archivos sin revisión ni enviar documentos fuera del equipo sin consentimiento.
