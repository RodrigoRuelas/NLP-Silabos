# Buscador de Sílabos con NLP 50%

Proyecto de **Tópicos Avanzados en IA**. Es un motor de búsqueda que recibe una consulta en lenguaje natural (por ejemplo, *"programación orientada a objetos"* o *"examen final proyecto"*) y devuelve los sílabos más parecidos, ordenados por relevancia.

- **Notebook original (Colab):** https://colab.research.google.com/drive/1Go2J004move0CN2TYB2Ue5MRN0cCO9Bd
- **Carpeta de sílabos (Drive):** https://drive.google.com/drive/folders/17DRL7HtwBcHru3CGeC1T6LwGOhzRRxFv
  (más de 45 PDFs de la carrera: Cálculo, Base de Datos, Arquitectura de Software, Inteligencia Artificial, Ciberseguridad, etc.)
- **Esta página web:** `index.html`, la misma idea del notebook, pero en JavaScript y sin servidor.

---

## 1. ¿Qué problema resuelve?

Buscar con `Ctrl+F` solo encuentra coincidencias exactas de una palabra y no dice qué documento es **más relevante**. Aquí queremos preguntar *"¿en qué curso se ve SQL y modelos relacionales?"* y obtener los cursos ordenados del más al menos parecido.

## 2. Pipeline (los pasos del notebook)

| Paso | Qué hace | Herramienta |
|------|----------|-------------|
| 1. Cargar | Monta Google Drive y lee todos los `.pdf` de la carpeta | `google.colab.drive`, `glob` |
| 2. Extraer texto | Convierte cada PDF en texto plano (descarta los vacíos o dañados) | `pypdf` |
| 3. Tabla | Guarda nombre, curso, texto y total de palabras de cada sílabo | `pandas` |
| 4. Vectorizar | Convierte cada texto en un vector numérico con **TF-IDF** | `TfidfVectorizer` |
| 5. Buscar | Vectoriza la consulta y calcula la **similitud coseno** con cada sílabo | `cosine_similarity` |
| 6. Resultado | Devuelve el top-K con porcentaje de similitud y un extracto | `buscar_silabo(consulta)` |

## 3. Cómo funciona en teoría

### 3.1 Bolsa de palabras (Bag of Words)
Un computador no entiende texto, entiende números. Representamos cada documento como un vector donde cada posición es una palabra del vocabulario y el valor indica cuánto aparece. Se pierde el orden de las palabras, pero se conserva *de qué trata* el documento.

### 3.2 Stopwords
Palabras como *de, la, que, el, en, y* aparecen en todos los textos y no aportan significado. Se eliminan antes de vectorizar para que no dominen la búsqueda (el notebook define su propia lista en español).

### 3.3 TF-IDF
Contar palabras no basta, porque una palabra muy común en todos los sílabos (por ejemplo "curso" o "evaluación") no ayuda a distinguirlos. TF-IDF le da más peso a las palabras **frecuentes en un documento pero raras en el resto**:

- **TF** (*term frequency*): cuántas veces aparece el término `t` en el documento `d`.
- **IDF** (*inverse document frequency*): `idf(t) = log((1 + N) / (1 + df(t))) + 1`, donde `N` es el número de documentos y `df(t)` en cuántos aparece `t`. Cuanto más raro es el término, mayor es su IDF.
- **Peso final:** `tfidf(t, d) = TF(t, d) × IDF(t)`

Ejemplo: "polimorfismo" aparece en pocos sílabos, así que pesa mucho. "universidad" aparece en todos, así que pesa casi nada.

### 3.4 Similitud coseno
Cada documento y la consulta son vectores en un espacio de muchas dimensiones. Medimos el **ángulo** entre ellos:

```
similitud(q, d) = (q · d) / (‖q‖ · ‖d‖)
```

- Vale **1** si apuntan en la misma dirección (muy parecidos) y **0** si no comparten ningún término.
- Al medir el ángulo y no la distancia, no importa que un sílabo sea más largo que otro.

El resultado se ordena de mayor a menor y se muestran los mejores K.

## 4. La página web (`index.html`)

Reimplementa el mismo algoritmo en JavaScript puro:

1. Tokeniza, pasa a minúsculas, quita tildes y stopwords.
2. Calcula el IDF y los vectores TF-IDF normalizados.
3. Al buscar, vectoriza la consulta y ordena por similitud coseno.

### Dataset real (carpeta de Drive)
Una página web no puede leer PDFs de un Drive privado, así que el texto se extrae **una vez** en Colab y se guarda en `silabos.json`:

1. Abrir el notebook y ejecutar las celdas hasta crear `df_silabos`.
2. Añadir una celda nueva con el contenido de `exportar_dataset.py` y ejecutarla (descarga `silabos.json`).
3. Colocar `silabos.json` junto a `index.html` y subirlo al repo.

La página carga `silabos.json` automáticamente al abrirse y muestra el fragmento donde aparece la coincidencia. Si no lo encuentra, usa textos de ejemplo de 10 cursos. También se puede cargar un `.json` o archivos `.txt` desde el botón de la página.

### Cómo ejecutarla
Con servidor local (necesario para que cargue `silabos.json`): `python3 -m http.server` y abrir `http://localhost:8000`. En GitHub Pages funciona directamente.

## 5. Limitaciones y mejoras posibles

- **TF-IDF no entiende significado:** "auto" y "carro" son palabras distintas para él. Se mejoraría con *embeddings* semánticos (por ejemplo `sentence-transformers`) o un modelo de lenguaje.
- No hace **stemming ni lematización** ("programar", "programación" y "programando" cuentan como términos distintos).
- Usa el sílabo completo como unidad. Dividirlo en **fragmentos** daría resultados más precisos y mejores extractos.
- Los PDFs escaneados como imagen no se pueden leer con `pypdf` y necesitarían OCR.

## 6. Tecnologías
- Python
- pypdf
- pandas
- scikit-learn (notebook) 
- HTML y JavaScript (web).