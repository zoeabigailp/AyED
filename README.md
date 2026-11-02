# LRU Cache en C: TDA genérico y simulador "Tweener"

Trabajo Práctico de **Algoritmos y Estructuras de Datos**. El proyecto implementa una **LRU Cache** (*Least Recently Used*) como TDA genérico en C, en dos versiones, y la usa en un simulador que mide cuánto mejora (o empeora) el procesamiento del feed de una red social ficticia llamada **Tweener**.

- Lenguaje: **C** (GCC)
- IDE: **Code::Blocks 20.03 a 25.03** (proyecto `TP_AyED.cbp`)
- Repositorio: <https://github.com/zoeabigailp/AyED.git>

---

## Índice

1. [¿Qué es una LRU Cache?](#1-qué-es-una-lru-cache)
2. [Estructura del proyecto](#2-estructura-del-proyecto)
3. [Compilar y ejecutar](#3-compilar-y-ejecutar)
4. [Uso del programa (menú)](#4-uso-del-programa-menú)
5. [Parte 1: LRU Cache con lista simple](#5-parte-1-lru-cache-con-lista-simple)
6. [Parte 2: lista doble + árbol índice](#6-parte-2-lista-doble--árbol-índice)
7. [Parte 3: simulación Tweener](#7-parte-3-simulación-tweener)
8. [Resultados obtenidos](#8-resultados-obtenidos)
9. [Análisis: respuestas a las preguntas del TP](#9-análisis-respuestas-a-las-preguntas-del-tp)
10. [Observaciones y posibles mejoras](#10-observaciones-y-posibles-mejoras)

---

## 1. ¿Qué es una LRU Cache?

Una caché guarda una cantidad **limitada** de datos para evitar repetir un acceso costoso (por ejemplo, leer un archivo). La política **LRU** parte de que *el dato usado hace poco tiene más chances de volver a usarse pronto*. Por eso, cuando la caché se llena, se descarta el dato **menos recientemente usado**.

Primitivas del TDA (nombres exigidos por el enunciado):

| Primitiva | Qué hace |
|---|---|
| `crear_lrucache` | Inicializa la caché con una capacidad máxima y una función de comparación de claves (negativo, cero o positivo según la primera clave sea menor, igual o mayor). |
| `agregar_lrucache` | Agrega un elemento al **inicio**. Si la caché desborda, elimina el **último**. Si la clave ya existía, reemplaza el valor y lo deja al inicio. |
| `obtener_lrucache` | Busca por clave. Si lo encuentra, lo **mueve al inicio** y devuelve el valor. Si no, informa que no existe. |
| `borrar_lrucache` | Elimina un elemento buscándolo por clave. |
| `vaciar_lrucache` | Vacía la caché y libera toda la memoria. |

El **orden de la estructura es el orden de uso**: primero el más reciente, último el candidato a ser descartado.

---

## 2. Estructura del proyecto

```
AyED/
├── main.c                       Menú principal
├── TP_AyED.cbp                  Proyecto Code::Blocks
│
├── Lista/                       TDA lista simplemente enlazada (genérica)
│   ├── Lista.h / Lista.c
├── LruCacheLista/               PARTE 1: LRU Cache sobre lista simple
│   ├── LruCacheLista.h / .c
│
├── LDE/                         TDA lista doblemente enlazada (con acceso a nodos por referencia)
│   ├── ListaDobleEnlazada.h / .c
├── Arbol/                       TDA árbol binario de búsqueda (genérico)
│   ├── Arbol.h / .c
├── LruCache/                    PARTE 2: LRU Cache con lista doble + árbol índice
│   ├── LruCache.h / .c
│
├── Lotes/                       Generador de lotes de prueba
│   ├── generadorLotes.h / .c    Genera posteos.dat y usuarios_<tipo>.dat
│   ├── exportador.h / .c        Convierte los .dat en .txt legibles
│
├── SimuladorTweener/            PARTE 3: simulación
│   ├── registros.h              Estructuras tPosteo y tUsuario, constantes
│   ├── simuladorTweener.h / .c  Simulador y medición de tiempos
│
└── Test/                        Pruebas de las Partes 1 y 2
    ├── TestPart1.h / TestParte1.c
    └── TestParte2.h / TestParte2.c
```

Archivos que genera el programa al ejecutarse:

| Archivo | Contenido |
|---|---|
| `posteos.dat` | 50 000 registros `{id: int, texto: char[141]}`. El `id` coincide con la posición del registro, lo que permite lectura directa con `fseek`. |
| `usuarios_concentrado.dat`, `usuarios_disperso.dat`, `usuarios_ordenado.dat` | Registros `{usuario: char[15], feed: int[20]}`. |
| `*.txt` | Versión legible de cada `.dat`, una línea por registro. |

> `posteos.dat` se regenera cada vez que se genera un lote. Para comparar lotes conviene usar siempre la misma semilla.

---

## 3. Compilar y ejecutar

1. Abrir `TP_AyED.cbp` con Code::Blocks (20.03 a 25.03).
2. Elegir el target:
   - **Debug**: `-g -Wall`, útil para depurar.
   - **Release**: `-O2`, **recomendado para medir tiempos**.
3. Compilar con `Build` y ejecutar con `Run`.

Notas:

- El compilador debe ser **GCC** (el que trae Code::Blocks). `Arbol.c` usa `typeof` en una macro, que es una extensión de GCC.
- El simulador es multiplataforma: en Windows usa `QueryPerformanceCounter` y en el resto `clock_gettime(CLOCK_MONOTONIC)`.
- Los `.dat` y `.txt` se crean en el directorio de trabajo del ejecutable. Hay que generar los lotes **antes** de simular.

---

## 4. Uso del programa (menú)

```
===== LRU Cache =====
1. Generar lote concentrado
2. Generar lote disperso
3. Generar lote ordenado
4. Test Parte 1 (lista simple)
5. Test Parte 2 (lista doble + arbol)
6. Simulacion Tweener
0. Salir
```

Cada pregunta muestra su valor por defecto entre corchetes; con **Enter** se acepta.

| Opción | Parámetros | Defecto |
|---|---|---|
| 1, 2, 3 (generar) | Semilla | 42 |
| | Usuarios (200 a 99 999) | 200 |
| | Pool de posteos populares (solo lote concentrado) | 100 |
| 4, 5 (tests) | ninguno | |
| 6 (simulación) | Lote (1 concentrado, 2 disperso, 3 ordenado, 4 todos) | 4 |
| | Repeticiones | 10 |

Flujo típico: generar los tres lotes (1, 2, 3) con la misma semilla y luego simular (6).

---

## 5. Parte 1: LRU Cache con lista simple

**Archivos:** `Lista/`, `LruCacheLista/`

```c
typedef struct {
    tLista   lista;      // nodos ordenados por uso: primero = más reciente
    unsigned cantidad;   // elementos actuales
    unsigned capacidad;  // máximo
    unsigned tamClave;
    tCmp     cmp;        // comparador de claves
} tLruCacheLista;

typedef struct { void *clave; void *valor; unsigned tamValor; } tDatoCache;
```

Cada nodo de la lista guarda un `tDatoCache`. Para obtener, agregar o borrar hay que **recorrer la lista buscando la clave**, por lo que cada operación cuesta **O(n)**, con `n` = cantidad de elementos almacenados. Esa es la limitación que resuelve la Parte 2.

La implementación es genérica: la clave y el valor son `void *` y el orden lo define el comparador que se pasa al crear la caché.

---

## 6. Parte 2: lista doble + árbol índice

**Archivos:** `LDE/`, `Arbol/`, `LruCache/`

### Idea

Se combinan dos estructuras con roles distintos:

```
        Árbol binario de búsqueda            Lista doblemente enlazada
        (busca por clave)                    (mantiene el orden de uso)

              [ 50 ]                  primero                         último
              /    \                     ↓                               ↓
         [ 20 ]    [ 80 ]             [ 80 ] ⇄ [ 20 ] ⇄ [ 50 ] ⇄ [ 35 ]
            \                           ↑        ↑        ↑        ↑
           [ 35 ]  ← cada nodo del árbol guarda un puntero (refNodoLista)
                      al nodo de la lista que corresponde a esa clave
```

- El **árbol** permite encontrar la clave sin recorrer la lista.
- La **lista doble** permite **mover un nodo al principio** y **sacar el último** en O(1), porque cada nodo conoce a su anterior y su siguiente.
- El árbol guarda, junto a la clave, una **referencia directa al nodo de la lista**: encontrar la clave equivale a tener el nodo en la mano.

### Estructuras

```c
typedef struct { void *clave; void *valor; unsigned tamValor; } tDatoCache_indice;   // info del nodo de la lista
typedef struct { void *clave; void *refNodoLista; tCmp cmp; }   tIndiceArbol;        // info del nodo del árbol

typedef struct {
    tListaDoble   lista;     // { primero, ultimo }
    tArbolBinBusq arbol;
    unsigned cantidad, capacidad, tamClave;
    tCmp cmp;
} tLruCache;
```

### Cómo funciona cada primitiva

| Primitiva | Pasos |
|---|---|
| `crear_lrucache_indice` | Crea lista y árbol vacíos y guarda capacidad y comparador. |
| `obtener_lrucache_indice` | Busca en el árbol. Si no está, devuelve 0. Si está, **mueve el nodo al principio** de la lista y copia el valor. |
| `agregar_lrucache_indice` | **Existe:** actualiza el valor y mueve el nodo al principio. **No existe:** si la caché está llena, saca el último de la lista y **lo borra también del árbol**; luego inserta el nuevo al inicio de la lista y en el árbol con su referencia. |
| `borrar_lrucache_indice` | Elimina la clave del árbol (obteniendo así la referencia al nodo) y luego elimina ese nodo de la lista. |
| `vaciar_lrucache_indice` | `eliminarArbol` y `vaciarListaDoble`. |
| `altura_arbol_indice` | Devuelve la altura del árbol índice (primitiva extra pedida por el enunciado). |

**Truco de implementación:** `buscarElemArbolBinBusq` y `eliminarElemArbolBinBusq` **copian de vuelta** la info del nodo hallado en el parámetro de búsqueda. Así, al buscar con un `tIndiceArbol` que solo tiene la clave, el campo `refNodoLista` queda completado con la referencia al nodo de la lista, sin recorrerla.

La comparación del árbol pasa por `cmp_indice_arbol`, que extrae las claves de los dos `tIndiceArbol` y llama al comparador genérico del usuario.

### Complejidad

| Operación | Parte 1 (lista simple) | Parte 2 (lista doble + ABB) |
|---|---|---|
| Buscar por clave | O(n) | O(h), con h = altura del árbol |
| Mover al principio | O(1) una vez hallado | O(1) |
| Descartar el último | O(n) (hay que recorrer hasta el final) | O(1) |
| Borrar | O(n) | O(h) |

En un ABB aleatorio `h ≈ log n`. En el peor caso (claves insertadas en orden) `h = n` y el árbol se comporta como una lista (ver [sección 9](#9-análisis-respuestas-a-las-preguntas-del-tp)).

### Manejo de memoria (quién libera qué)

Esta es la parte delicada, porque el mismo dato queda referenciado desde **dos** estructuras. La regla es que **cada bloque reservado tiene un único dueño**:

| Bloque | Dueño | Se libera en |
|---|---|---|
| Nodo de la lista y su `tDatoCache_indice` | la lista doble | `sacarUltimoLDE`, `eliminarNodoLDE`, `vaciarListaDoble` |
| Nodo del árbol y su `tIndiceArbol` | el árbol | `eliminarElemArbolBinBusq`, `eliminarArbol` |
| Clave y valor del usuario (los `void *`) | **quien llama a la caché** | nunca los libera la caché |

- Ambas estructuras guardan **copias de sus propias estructuras auxiliares** (`memcpy` con `tamInfo`) y solo comparten **punteros** (`refNodoLista`, `clave`, `valor`). Por eso ningún bloque se libera dos veces.
- Al descartar por desborde, el nodo se libera una sola vez en la lista y una sola vez en el árbol, y son bloques distintos.
- `vaciar_lrucache_indice` vacía las dos estructuras y deja `cantidad = 0`; no queda ningún bloque sin liberar.

> Como la caché guarda **punteros** y no copias de la clave y el valor, el que la usa debe garantizar que esos datos sigan vivos mientras estén en la caché. El simulador resuelve esto con "casilleros" (ver sección 7.4).

---

## 7. Parte 3: simulación Tweener

### 7.1 Escenario

Tweener solo admite posteos de **menos de 140 caracteres**. Se quiere validar si una LRU Cache acelera la carga del feed de cada usuario. Para ello:

- Hay **50 000 posteos** en `posteos.dat`.
- Hay **≥ 200 usuarios** en `usuarios_<tipo>.dat`, cada uno con un feed de **20 ids de posteo**.
- El programa recorre cada usuario y busca sus 20 posteos. **No muestra ni guarda los datos**: solo mide tiempos.

Se mide el mismo programa en tres variantes: **sin caché**, con la **caché de la Parte 1** y con la de la **Parte 2**.

### 7.2 Generador de lotes (`Lotes/`)

Recibe una **semilla** (`srand`), por lo que el mismo lote puede volver a generarse byte a byte (cada registro se pone en cero antes de armarlo).

| Lote | Cómo se arma el feed | Situación que representa |
|---|---|---|
| **Concentrado** | Cada id se elige al azar dentro de un **pool chico** (por defecto 100 posteos populares). | Muchos usuarios ven los mismos posteos. |
| **Disperso** | Cada id se elige al azar entre **los 50 000** posteos. | Los usuarios ven posteos mayormente distintos. |
| **Ordenado** | `id = contador * 50000 / total`: ids **estrictamente crecientes** en toda la simulación. | Peor caso para el árbol (se degenera). |

Detalles:

- El generador de ids es un puntero a función (`tGenerarId`), y `buscarTipo` resuelve el nombre del lote con una tabla. Agregar un tipo nuevo es agregar una fila.
- El lote ordenado exige `usuarios * 20 <= 50000` (máximo 2500 usuarios), porque necesita ids no repetidos.
- Los textos se generan con palabras al azar (o letras al azar, según `TEXTO_ELEGIDO` en `generadorLotes.h`) con 1 a 139 caracteres.
- `exportador.c` convierte cada `.dat` en `.txt` para poder inspeccionarlo a mano.

### 7.3 Simulador (`SimuladorTweener/`)

Las tres variantes se tratan de forma uniforme con una **tabla de estrategias**: una estructura `tEstrategia` con punteros a función (`crear`, `obtener`, `agregar`, `victima`, `vaciar`, `altura`). Cada variante tiene sus adaptadores:

| Prefijo | Variante |
|---|---|
| `sc*` | Sin caché (siempre falla, no guarda nada) |
| `l*` | Parte 1, lista simple |
| `a*` | Parte 2, lista doble + árbol |

De esta forma el ciclo de simulación es **el mismo código** para las tres y no se duplica lógica.

**Flujo por posteo (`obtenerPosteo`):**

1. Se busca en la caché. Si está: **acierto**.
2. Si no: **fallo**, se lee el registro del archivo (`fseek` directo a `id * sizeof(tPosteo)`) y se agrega a la caché.

**Medición:**

- Se mide tiempo **real** (monotónico), no de CPU, porque lo que ahorra la caché es espera de lectura que `clock()` no cuenta en Linux.
- Los tiempos **por posteo** y **por usuario** son promedios calculados a partir del tiempo total, ya que un posteo puede durar menos que la resolución del reloj.
- Cada simulación se **repite** N veces y se informa el promedio.
- Antes de medir se hace una **pasada de calentamiento** para que el archivo ya esté en la caché del sistema operativo y la primera variante no quede en desventaja.
- El archivo de posteos se abre **sin buffer** (acceso aleatorio de un registro) y el de usuarios **con buffer de 64 KB** (lectura secuencial). Ningún archivo se carga completo en memoria.
- La mejora se calcula como `tiempo sin caché / tiempo con caché` (`2.00x` = el doble de rápido; `0.50x` = el doble de lento).

Por cada ejecución se informa: capacidad, tiempo total, µs/usuario, µs/posteo, aciertos, fallos, lecturas al archivo y mejora. Para la Parte 2 también la **altura del árbol índice**.

Capacidades probadas: **10, 100, 1000 y 10 000**.

### 7.4 Casilleros: cómo se evitan pisadas de memoria

Como las cachés guardan **punteros** al valor, cada posteo cacheado necesita su propio espacio que no se sobrescriba mientras siga en la caché. `realizarSimulacion` reserva `capacidad + 1` casilleros de `tPosteo`:

- `capacidad` casilleros están en uso (uno por posteo cacheado).
- 1 casillero está **libre** y es donde se lee el próximo posteo.

Antes de agregar, `e->victima(c)` informa qué posteo saldrá (el último de la lista). Una vez hecho el agregado, **el casillero de la víctima pasa a ser el libre**. Mientras la caché no se llena, el casillero libre es simplemente el siguiente. Todo se libera con un único `free(casilleros)` al terminar.

---

## 8. Resultados obtenidos

Corridas con **200 usuarios × 20 posteos = 4000 accesos**, semilla 42, **100 repeticiones**. (La corrida de 10 repeticiones dio valores prácticamente iguales, lo que indica que la medición es estable.)

### Lote concentrado (pool de 100 posteos)

| Variante | Capac. | Total (ms) | µs/posteo | Aciertos | Fallos | Altura | Mejora |
|---|---:|---:|---:|---:|---:|---:|---:|
| Sin caché | – | 10.49 | 2.622 | 0 | 4000 | | 1.00x |
| Parte 1 | 10 | 10.58 | 2.645 | 385 | 3615 | | 0.99x |
| Parte 1 | 100 | 1.12 | 0.280 | 3900 | 100 | | 9.35x |
| Parte 1 | 1000 | 1.12 | 0.279 | 3900 | 100 | | 9.40x |
| Parte 1 | 10000 | 1.19 | 0.297 | 3900 | 100 | | 8.83x |
| Parte 2 | 10 | 11.30 | 2.826 | 385 | 3615 | 5 | 0.93x |
| Parte 2 | 100 | 0.61 | 0.152 | 3900 | 100 | 13 | 17.28x |
| Parte 2 | 1000 | 0.62 | 0.156 | 3900 | 100 | 13 | 16.80x |
| Parte 2 | 10000 | 0.62 | 0.155 | 3900 | 100 | 13 | 16.87x |

### Lote disperso

| Variante | Capac. | Total (ms) | µs/posteo | Aciertos | Fallos | Altura | Mejora |
|---|---:|---:|---:|---:|---:|---:|---:|
| Sin caché | – | 11.98 | 2.995 | 0 | 4000 | | 1.00x |
| Parte 1 | 10 | 12.76 | 3.190 | 1 | 3999 | | 0.94x |
| Parte 1 | 100 | 16.92 | 4.231 | 5 | 3995 | | 0.71x |
| Parte 1 | 1000 | 53.03 | 13.257 | 56 | 3944 | | 0.23x |
| Parte 1 | 10000 | 60.83 | 15.208 | 152 | 3848 | | 0.20x |
| Parte 2 | 10 | 13.79 | 3.448 | 1 | 3999 | 4 | 0.87x |
| Parte 2 | 100 | 15.14 | 3.786 | 5 | 3995 | 10 | 0.79x |
| Parte 2 | 1000 | 15.93 | 3.983 | 56 | 3944 | 19 | 0.75x |
| Parte 2 | 10000 | 15.14 | 3.785 | 152 | 3848 | 23 | 0.79x |

### Lote ordenado

| Variante | Capac. | Total (ms) | µs/posteo | Aciertos | Fallos | Altura | Mejora |
|---|---:|---:|---:|---:|---:|---:|---:|
| Sin caché | – | 7.30 | 1.824 | 0 | 4000 | | 1.00x |
| Parte 1 | 10 | 8.07 | 2.017 | 0 | 4000 | | 0.90x |
| Parte 1 | 100 | 12.41 | 3.103 | 0 | 4000 | | 0.59x |
| Parte 1 | 1000 | 48.29 | 12.072 | 0 | 4000 | | 0.15x |
| Parte 1 | 10000 | 57.88 | 14.471 | 0 | 4000 | | 0.13x |
| Parte 2 | 10 | 8.83 | 2.208 | 0 | 4000 | 10 | 0.83x |
| Parte 2 | 100 | 15.33 | 3.833 | 0 | 4000 | 100 | 0.48x |
| Parte 2 | 1000 | 72.61 | 18.153 | 0 | 4000 | 1000 | 0.10x |
| Parte 2 | 10000 | 104.01 | 26.002 | 0 | 4000 | 4000 | 0.07x |

---

## 9. Análisis: respuestas a las preguntas del TP

### ¿En qué situaciones la caché mejora el tiempo y en cuáles no?

**Mejora cuando hay repetición de accesos y la capacidad alcanza para contener el conjunto "caliente".**

- **Concentrado:** con capacidad ≥ 100 (el tamaño del pool) hay 3900 aciertos y solo 100 fallos (los primeros accesos de cada posteo popular). Mejora ≈ **9x** con la Parte 1 y ≈ **17x** con la Parte 2.
- **Capacidad 10 en el concentrado:** el pool tiene 100 posteos y la caché solo guarda 10. Antes de volver a pedir un posteo, ya fue expulsado, por lo que casi no hay aciertos (385 de 4000) y no hay mejora; con la Parte 2 incluso se pierde un poco (0.93x) por el costo de mantener las estructuras.

**No mejora, e incluso empeora, cuando casi no hay reutilización:**

- **Disperso:** con 50 000 posteos posibles y solo 4000 accesos, casi nunca se repite uno (1 a 152 aciertos). Cada fallo paga la lectura del archivo **más** el costo de buscar y agregar en la caché.
- **Ordenado:** por construcción **no hay ningún acierto**, ya que ningún id se repite.

### ¿A partir de qué capacidad deja de convenir la Parte 1? ¿Por qué?

El costo de la Parte 1 depende de cuántos elementos hay **realmente** almacenados, porque cada `obtener` y cada `agregar` recorren la lista. Entonces:

- Si la caché se llena de elementos que **no se reutilizan**, cada acceso paga un recorrido O(n) para nada. En el disperso, ya con **capacidad 100** es más lenta que no usar caché (0.71x) y con 1000 baja a 0.23x. Incluso con 10 pierde levemente (0.94x).
- En el concentrado **sí conviene** hasta 10 000 porque los elementos almacenados nunca superan 100 (el pool), así que la lista se mantiene corta.

Conclusión: la Parte 1 deja de convenir **cuando el costo del recorrido de la lista supera lo que cuesta ir al archivo**, lo que en estas corridas ocurre alrededor de capacidades de **100 o más cuando la tasa de aciertos es baja**. Con una tasa de aciertos alta, la Parte 1 se sostiene aun con capacidades grandes, siempre que la cantidad real de elementos sea chica.

> **Aclaración importante:** el archivo de posteos está en la caché del sistema operativo (por la pasada de calentamiento y su tamaño de ~7 MB), por lo que cada lectura cuesta apenas ~2 a 3 µs. Con un disco real o un archivo remoto, la lectura sería mucho más cara y el punto de equilibrio se movería a favor de la caché.

### ¿Qué pasa con el árbol índice en el lote ordenado y cómo afecta al rendimiento?

Insertar claves **en orden creciente** en un ABB sin balanceo hace que cada nodo nuevo cuelgue del anterior: el árbol **se degenera en una lista**. Esto se ve en la altura informada, que es igual a la cantidad de elementos de la caché:

| Capacidad | Altura obtenida | Altura de un árbol balanceado (≈ log₂ n) |
|---:|---:|---:|
| 10 | 10 | ~4 |
| 100 | 100 | ~7 |
| 1000 | 1000 | ~10 |
| 10000 | 4000 (solo hay 4000 accesos) | ~12 |

Consecuencias:

- La búsqueda pasa de O(log n) a **O(n)**: se pierde la ventaja de la Parte 2.
- Además, la Parte 2 carga con el costo extra de **mantener dos estructuras**, por lo que resulta **más lenta que la Parte 1** en capacidades grandes (0.10x y 0.07x contra 0.15x y 0.13x).
- En el disperso, en cambio, la altura se mantiene baja (4, 10, 19, 23) y la Parte 2 queda estable en torno a 0.75x a 0.87x: no mejora (no hay aciertos) pero tampoco se degrada con la capacidad.

**Solución clásica:** usar un árbol **autobalanceado** (AVL o rojo-negro), que garantiza altura O(log n) sin importar el orden de inserción.

---

## 10. Observaciones y posibles mejoras

- **Balanceo del árbol:** como se explicó, un AVL eliminaría el peor caso del lote ordenado.
- **Cálculo de altura al borrar:** `eliminarRaizArbolBinBusq` decide qué reemplazante usar llamando a `alturaArbolBin` en ambos subárboles, y esa función **recorre todos los nodos del subárbol** (no solo una rama). Eliminar un nodo interno cuesta entonces O(tamaño del subárbol), y eliminar la raíz O(n). Esto agrava el caso degenerado. Una alternativa es elegir siempre el reemplazante de un lado (predecesor o sucesor) sin calcular alturas.
- **Reemplazo de clave:** cuando se agrega una clave existente, se actualiza el valor y se **conserva el puntero a la clave original** (el nodo no cambia de clave). Es consistente mientras ambas claves sean iguales según el comparador.
- **Datos por puntero:** las cachés guardan punteros a clave y valor en vez de copiarlos. Es eficiente, pero obliga al usuario a mantener esos datos vivos (resuelto en el simulador con los casilleros).
- **Medición:** para resultados más representativos, usar el target **Release** y más repeticiones; los resultados con 10 y 100 repeticiones coinciden, lo que muestra que la medición es estable.
