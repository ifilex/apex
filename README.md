# Apex Crypto

### Trading algorítmico y decisiones bajo incertidumbre

Apex Crypto es un Expert Advisor escrito en **MQL4 para MetaTrader 4**. Este repositorio comparte el código fuente de la **versión 2.00** para su estudio, revisión y experimentación.

Combina señales técnicas, distancias basadas en volatilidad, entradas tipo grid y coberturas automáticas.

La configuración incluida es agresiva: utiliza un multiplicador de lotaje de **1.8** y permite acumular numerosas posiciones. Requiere revisión y pruebas en demo.

## Contexto del proyecto

Mi recorrido como neuropsicólogo y desarrollador incluye el estudio de sistemas de decisión: cómo seleccionamos información, formulamos reglas y respondemos ante resultados adversos.

Apex lleva esas preguntas a una implementación concreta. Permite examinar tanto las condiciones de entrada como las decisiones que modifican la exposición cuando el mercado se mueve en contra.

Es un sistema basado en reglas. No incorpora inteligencia artificial, aprendizaje automático ni un modelo neuropsicológico.

## Cómo funciona

### Señales de entrada

Utiliza indicadores del período del gráfico:

- **RSI de 14 períodos** para evaluar cruces de umbral.
- **EMA de 20 y 50 períodos** para comparar la dirección de corto y mediano plazo.
- **EMA de 200 períodos** como referencia de tendencia.

Las compras pueden activarse por recuperación del RSI o por proximidad del precio a la EMA20, con las condiciones de tendencia correspondientes.

Las ventas utilizan condiciones equivalentes en sentido bajista.

Parte del cálculo utiliza la vela en formación, por lo que las señales pueden cambiar durante esa vela.

### Stop loss y take profit

Las distancias iniciales se calculan a partir del **ATR**:

- Stop loss: ATR × 1.5.
- Take profit: ATR × 5.0.

Estos son los valores predeterminados del archivo, no recomendaciones.

Si la comprobación de distancia mínima no se cumple, el código puede enviar la orden con el stop loss o take profit correspondiente en cero.

### Tamaño de posición

La entrada inicial puede calcularse a partir de un porcentaje del equity y de la distancia estimada al stop.

Las entradas adicionales pueden reemplazar ese cálculo por el lote anterior multiplicado por **1.8**.

Por ello, `RiesgoPorcentaje` no representa un límite de pérdida para el conjunto de operaciones.

### Grid

Cuando el precio se desplaza en contra, el sistema puede agregar posiciones del mismo lado a una distancia configurada.

Estas entradas heredan el stop registrado en la operación tomada como referencia y se envían inicialmente sin take profit.

Los filtros de spread, volumen y liquidez se aplican a las entradas iniciales; no se vuelven a exigir para las entradas del grid.

### Cobertura automática

El sistema puede intentar abrir una posición contraria cuando:

- Hay al menos cinco operaciones de un mismo lado.
- La pérdida flotante de ese lado supera el 5% del balance.
- No existe otra cobertura identificada para ese símbolo.

El tamaño predeterminado de la cobertura es el **50% del lotaje acumulado de ese lado**.

Las coberturas se envían sin stop loss ni take profit. Esta versión no incluye una rutina específica para cerrarlas y las excluye del trailing de las posiciones principales.

### Bloqueo por drawdown

El código compara el equity de toda la cuenta con el balance registrado al inicializar el EA.

Al alcanzar el umbral configurado, bloquea nuevas entradas iniciales, nuevas entradas del grid y nuevas coberturas.

**No liquida las posiciones existentes ni garantiza una pérdida máxima.**

El bloqueo permanece durante esa ejecución aunque el equity se recupere. Al reinicializar el EA, la referencia y el estado del bloqueo se restablecen.

## Instrumentos

El código busca estos nombres base:

BTCUSD, ETHUSD, LTCUSD, XRPUSD, BCHUSD, ADAUSD, SOLUSD, DOTUSD, DOGEUSD, BNBUSD, LINKUSD, XLMUSD, AVAXUSD, MATICUSD, TRXUSD, ATOMUSD, NEARUSD y ALGOUSD.

Su disponibilidad depende del bróker.

Apex opera los instrumentos ofrecidos dentro de MetaTrader 4. No incluye conexión directa a exchanges ni gestión de billeteras.

La detección contempla un sufijo obtenido del símbolo del gráfico y una alternativa sin sufijo. Otros formatos de nombres requieren adaptación.

## Parámetros principales

| Parámetro | Valor predeterminado | Función |
|---|---:|---|
| `UsarLotePorRiesgo` | `true` | Activa el cálculo inicial por riesgo estimado |
| `RiesgoPorcentaje` | `3.0` | Porcentaje usado en ese cálculo |
| `LoteFijo` | `0.01` | Lote base alternativo |
| `MultiplicadorLote` | `1.8` | Progresión de entradas adicionales |
| `DistanciaGridPips` | `15.0` | Separación configurada del grid |
| `MaxOrdenesPorLado` | `30` | Máximo configurado por dirección y símbolo |
| `MaxParesSimultaneos` | `5` | Límite previsto de símbolos simultáneos |
| `ATR_Periodo` | `14` | Período del ATR |
| `ATR_Multiplo_SL` | `1.5` | Factor del stop inicial |
| `ATR_Multiplo_TP` | `5.0` | Factor del objetivo inicial |
| `SpreadMaximoPips` | `80.0` | Umbral de spread para entradas iniciales |
| `MaxDrawdownPct` | `40.0` | Umbral de bloqueo |
| `UsarFiltroLiquidez` | `true` | Activa la rutina de barridos, pendiente de corrección |
| `UsarFiltroVolumen` | `false` | Activa el filtro de volumen de ticks |
| `UsarBreakevenGrupal` | `true` | Activa la rutina de objetivo conjunto |
| `UsarTrailing` | `true` | Activa el trailing de posiciones principales |
| `UsarHedgeAutomatico` | `true` | Permite coberturas automáticas |
| `RatioHedge` | `0.5` | Proporción de lotaje para cobertura |

Las unidades y el cálculo de lotaje deben comprobarse contra las especificaciones del instrumento y del bróker.

## Instalación para pruebas

1. Abrir MetaTrader 4.
2. Seleccionar **Archivo → Abrir carpeta de datos**.
3. Copiar `APEX_CRYPTO.mq4` en `MQL4/Experts`.
4. Abrirlo en MetaEditor y compilar.
5. Revisar los errores y las advertencias del compilador.
6. Actualizar la lista de Expert Advisors.
7. Cargar el EA en un gráfico de una cuenta demo.
8. Comprobar símbolos, datos históricos, parámetros y permisos.

El recorrido multiactivo se ejecuta al recibir ticks del gráfico donde está instalado.

Para probar la cobertura, el entorno debe permitir posiciones opuestas sobre el mismo instrumento.

El EA utiliza tres identificadores consecutivos: `NumeroMagico`, `NumeroMagico + 1` y `NumeroMagico + 2`. Evitar colisiones con otras instancias o sistemas.

## Estado y limitaciones conocidas

El código compartido requiere validación. Entre los puntos identificados por lectura se encuentran:

- **Filtro de barridos:** la ventana de máximos y mínimos incluye la misma vela que se compara. Con datos coherentes, esa vela no puede superar estrictamente el máximo que la incluye ni caer por debajo del mínimo que la incluye.
- **Límite de símbolos:** el conteo se realiza antes del recorrido y no se actualiza tras cada apertura; el límite previsto puede superarse dentro de un tick.
- **Objetivo conjunto:** la conversión de beneficio monetario a distancia de precio requiere revisión dimensional y validación con el valor del tick.
- **Lotaje:** el redondeo por decimales no garantiza compatibilidad con todos los pasos de lote.
- **Valor del tick:** si no está disponible, el cálculo utiliza un valor alternativo de `1.0`, que puede producir un dimensionamiento incorrecto.
- **Última operación:** depende del orden de recorrido, sin selección explícita por fecha de apertura.
- **Coberturas:** requieren revisar límites de lote, margen, comprobación de ejecución y política de cierre.
- **Panel:** el resultado mostrado como total corresponde a las posiciones principales del símbolo del gráfico; excluye las coberturas y otros símbolos.
- **Ejecución:** varias llamadas de envío y modificación de órdenes no comprueban su resultado.
- **Datos y parámetros:** falta una validación integral de históricos disponibles y valores de entrada.

El nombre “anti-manipulación” utilizado en la interfaz describe la intención del filtro. No demuestra capacidad para identificar manipulación del mercado.

## Riesgo

La progresión de lotaje de 1.8 puede aumentar rápidamente la exposición y el margen requerido.

El grid, las coberturas y el bloqueo de nuevas entradas no eliminan la posibilidad de pérdidas importantes o de pérdida del capital de la cuenta.

La correlación entre instrumentos puede concentrar el riesgo aunque se operen varios símbolos.

## Resultados y validación

Este repositorio no incluye un historial auditado ni una validación independiente de rentabilidad.

La documentación se basa en la lectura del código. No acredita compilación, ejecución ni resultados de backtesting.

## Autor

Desarrollado por **Emanuel**, neuropsicólogo y desarrollador del proyecto **WineBOX**.

- https://winebox.cloud
- https://wineboxtool.cloud

## Contribuciones

Se reciben reportes de errores y propuestas mediante Issues y Pull Requests.

Al informar un problema, incluir instrumento, parámetros, versión de MetaTrader 4 y mensajes relevantes del registro, sin credenciales ni datos personales.

## Licencia

Este proyecto se distribuye bajo la licencia **Apache License 2.0**.

Consultar el archivo [LICENSE](LICENSE) para conocer los términos completos.

## Aviso

Material técnico y educativo. No constituye asesoramiento financiero ni una promesa de resultados.
