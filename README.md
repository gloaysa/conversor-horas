## Contexto
Esta aplicación convierte las horas desde el formato de Navision al esperado por Mercedes-Benz.io

Para descargar las horas, ve a Navision/JobEntries y selecciona tu proyecto y WorkPackage. Copia los datos (solo los datos, sin cabeceras) y crea con ellos un archivo CSV, que será el que importes en la aplicación tras rellenar los campos.

El programa diferencia las horas en remoto y on-site.

### Posibles problemas
* Todo está bastante _hardcoded_. La verdad es que no he querido tampoco perder demasiado el tiempo con esto. Si hay algún cambio en tu plantilla o en la que yo uso en el futuro, habrá que cambiar el código a mano. Al menos ya tienes bastante avanzado.
* He añadido comentarios con lo que sería cada columna, para facilitar la edición.
* Las fechas tienen el formato DD.MM.AAAA (con puntos). Es un poco extraño, pero es el formato que me piden. Puedes cambiarlo en la functión `changeDate()`.

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev
