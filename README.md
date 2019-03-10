## Context
Esta aplicación convierte las horas desde el formato de Navision al esperado por Mercedes-Benz.io

Para descargar las horas, ve a Navision/JobEntries y selecciona tu proyecto y WorkPackage. Copia los datos (solo los datos, sin cabeceras) y crea con ellos un archivo CSV, que será el que importes en la aplicación tras rellenar los campos.

El programa diferencia las horas en remoto y on-site.

Una vez que descargues el nuevo CSV, ábrelo con Microsoft Excell, ve a Data/TextToColumn. Elige como separador la coma y voilá.

### Posibles problemas
* Las horas me llegan por defecto con los decimales separados por comas, así que los tengo que convertir a punto con este programa. Si no es tu caso, elimina la función `.replace()` en `let hour = dataHour[5].replace(',', '.')`.
* Todo está bastante _hardcoded_. La verdad es que no he querido tampoco perder demasiado el tiempo con esto. Si hay algún cambio en tu plantilla o en la que yo uso en el futuro, habrá que cambiar el código a mano. Al menos ya tienes bastante avanzado.

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev
