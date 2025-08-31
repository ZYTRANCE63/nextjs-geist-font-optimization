# Plan de Desarrollo del Sistema de Recomendación Integral

A continuación se presenta una hoja de ruta detallada que integra las materias de Algoritmos y ED, Aprendizaje Automático I y Bases de Datos para IA, incluyendo la integración con el frontend (Next.js) y un microservicio en Python para el procesamiento ML.

---

## 1. Estrategia y Arquitectura General

### Ideas Concretas de Sistemas de Recomendación
- **Recomendador de Películas:** Utilizando datasets como MovieLens; permite filtrar películas y sugerir títulos basados en historial y similitud.
- **Recomendador para E-commerce de Nicho:** Ofrece productos personalizados basados en compras anteriores y comportamientos de navegación.
- **Recomendador de Música:** Basado en análisis de tendencias y relaciones entre usuarios, sin necesidad de integrar bibliotecas de iconos; la interfaz se diseñará solo con tipografía y diseño limpio.

### Arquitectura de Software
- **Tipo:** Arquitectura basada en microservicios.
  - **Frontend:** Aplicación Next.js (React + TypeScript) que consume los endpoints de recomendación.
  - **Backend API:** Endpoints en Next.js (carpeta `src/app/api/recommendations`) para gestionar solicitudes y comunicarse con el microservicio ML.
  - **Microservicio ML:** Servicio independiente en Python que implementa algoritmos de recomendación y expone endpoints (por ejemplo, mediante Flask o FastAPI).
  - **Base de Datos:** PostgreSQL para el almacenamiento relacional de usuarios, items, calificaciones e interacciones.
- **Stack Tecnológico:**
  - Frontend: Next.js, React, TypeScript.
  - Backend/API: Next.js API routes.
  - ML: Python con scikit-learn, numpy, pandas y, de ser necesario, Flask/FastAPI.
  - Database: PostgreSQL (alternativamente, considerar MongoDB para grandes volúmenes no estructurados).

---

## 2. Implementación de Algoritmos y Estructuras de Datos

### Búsqueda y Optimización
- **Búsqueda Binaria:** Para ítems ordenados (e.g., búsqueda de productos en listas preordenadas).
- **Técnicas de Optimización:** 
  - **LRU Cache:** Implementar caching en endpoints de recomendaciones (en Node.js, utilizando librerías como `node-cache`).
  - **Memoización:** Para cálculos costosos en la generación de recomendaciones.

### Estructuras de Datos
- **Grafos:** Usar la librería NetworkX en Python para modelar relaciones entre usuarios o similitud entre ítems.
- **Tablas Hash (Diccionarios):** Para búsquedas rápidas de usuarios e ítems.
- **Árboles (KD-Trees):** Para agrupar ítems en función de características numéricas.

### Algoritmos Genéticos
- **Esquema Propuesto:**
  - **Representación:** Vector (cromosoma) que codifica el perfil de preferencias del usuario.
  - **Función de Fitness:** Medida basada en la similitud entre preferencias reales y recomendaciones.
  - **Operadores:** Cruce (intercambio parcial de vectores) y mutación (alteración de un porcentaje de las preferencias).

---

## 3. Desarrollo del Modelo de Aprendizaje Automático

### Selección de Algoritmos
- **Filtrado Colaborativo (SVD):** Para usuarios con historial (modelo basado en matrices de usuarios-items).
- **Filtrado Basado en Contenido (NLP/TF-IDF):** Para análisis de descripciones de ítems; ideal si se dispone de texto explicativo.
- **Modelo Híbrido:** Combinar ambos enfoques para mejorar la precisión en escenarios con datos dispersos.

### Modelado Supervisado/No Supervisado
- **Clustering (K-Means):** Para segmentar usuarios en grupos con comportamientos similares.
- **Clasificación (Random Forest):** Para predecir si a un usuario le gustará un ítem basado en características históricas.

### Ejemplo de Código para Evaluación (Python)
```python
from sklearn.metrics import precision_score, recall_score, f1_score

def evaluate_model(true_labels, predicted_labels):
    precision = precision_score(true_labels, predicted_labels, average='macro')
    recall = recall_score(true_labels, predicted_labels, average='macro')
    f1 = f1_score(true_labels, predicted_labels, average='macro')
    return {"precision": precision, "recall": recall, "f1_score": f1}

# Ejemplo de uso:
# results = evaluate_model(y_true, y_pred)
# print(results)
```

---

## 4. Gestión y Almacenamiento de Datos

### Esquema de Base de Datos (SQL)
- **Archivo Propuesto:** `db/schema.sql`
- **Ejemplo DDL:**
```sql
CREATE TABLE Users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE Items (
    item_id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    category VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE Ratings (
    rating_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES Users(user_id),
    item_id INT REFERENCES Items(item_id),
    rating DECIMAL(2,1) CHECK (rating BETWEEN 0 AND 5),
    rated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE Interactions (
    interaction_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES Users(user_id),
    item_id INT REFERENCES Items(item_id),
    interaction_type VARCHAR(50),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Índices para optimización:
CREATE INDEX idx_user_ratings ON Ratings(user_id);
CREATE INDEX idx_item_ratings ON Ratings(item_id);
```

### Optimización de Consultas
- **Consultas Lentas Potenciales:**
  - Obtener todas las calificaciones de un usuario.
  - Consultar ítems populares basados en interacciones.
  - Recomendar ítems similares mediante joins complejos.
- **Mejoras sugeridas:** 
  - Creación de índices en columnas críticas (user_id, item_id).
  - Revisión y optimización de sentencias JOIN.

### Escalabilidad
- Se recomienda usar una base de datos NoSQL (MongoDB) cuando se requiera manejar grandes volúmenes de datos no estructurados o cuando la velocidad de escritura sea crítica.

---

## 5. Entregables y Documentación

### Estructura de Carpetas Recomendada
```
/ (Root)

├── db/
│   └── schema.sql           # Esquema de Base de Datos
├── ml-service/              
│   ├── recommendation_model.py  # Microservicio ML con algoritmos y endpoints
│   └── requirements.txt     # Dependencias de Python
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   └── recommendations/
│   │   │       └── route.ts   # API endpoint para recomendaciones
│   │   └── recommendations/
│   │       └── page.tsx       # Página de UI para visualizar recomendaciones
│   ├── components/          # Componentes UI existentes
│   ├── hooks/
│   └── lib/
├── package.json             # Dependencias y scripts para Next.js
└── README.md                # Documentación del proyecto
```

### Cambios Detallados en Cada Archivo

- **README.md**
  - Actualizar con la descripción del proyecto, dependencias, instrucciones de instalación y ejecución.
  - Incluir secciones sobre la arquitectura, modelo ML y ejemplos de uso.
  
- **package.json**
  - Verificar y agregar dependencias necesarias para la comunicación con el microservicio (e.g., Axios).
  
- **src/app/api/recommendations/route.ts**
  - Crear la carpeta y archivo si no existen.
  - Implementar un endpoint que:
    - Reciba solicitudes (GET/POST).
    - Realice llamadas al microservicio ML usando `fetch` o Axios.
    - Maneje errores con bloques try/catch y retorne mensajes de error apropiados en JSON.
  
- **src/app/recommendations/page.tsx**
  - Crear una nueva página para mostrar las recomendaciones.
  - Diseñar una UI moderna utilizando solo tipografía, colores, y espaciado definidos en `globals.css`.
  - Incluir mensajes de error en la UI en caso de que la API falle y mostrar un loader mientras se obtienen datos.
  
- **ml-service/recommendation_model.py**
  - Implementar funciones que:
    - Ejecuten los algoritmos (filtrado colaborativo, basado en contenido, clustering).
    - Exponer endpoints mediante Flask o FastAPI para recibir solicitudes desde el frontend.
    - Incluir manejo de excepciones y logs para fallos y entradas inválidas.
  
- **ml-service/requirements.txt**
  - Incluir dependencias: 
    - `scikit-learn`
    - `numpy`
    - `pandas`
    - `Flask` o `FastAPI` (según la elección de framework)
  
- **db/schema.sql**
  - Crear y ejecutar el script en PostgreSQL para generar las tablas con constraints e índices.
  - Asegurarse de que este archivo esté documentado con comentarios para futuras modificaciones.

---

## 6. Consideraciones de Error Handling y Buenas Prácticas

- Validar las entradas en todos los endpoints (tanto en Next.js como en el microservicio ML).
- Utilizar try/catch para capturar errores y devolver mensajes descriptivos.
- Aplicar logging en el microservicio para registro de errores y seguimientos.
- Documentar funciones y módulos mediante comentarios y docstrings/JSDoc.
- Realizar pruebas unitarias y de integración para la API y el modelo ML.
- Asegurar el uso de variables de entorno en la configuración de conexiones a la base de datos y otros servicios sensibles.

---

## Resumen

- Se propone una arquitectura de microservicios con un frontend en Next.js y un microservicio ML en Python.  
- Se desarrollarán algoritmos de filtrado, técnicas de optimización y estructuras de datos específicas (grafos, tablas hash, KD-Trees).  
- Se emplearán dos enfoques ML (filtrado colaborativo y basado en contenido), complementados con clustering y clasificación.  
- Se diseña un esquema relacional con DDL en `db/schema.sql` e índices para optimizar consultas.  
- Se añaden nuevos endpoints en `src/app/api/recommendations/route.ts` y una página moderna en `src/app/recommendations/page.tsx`.  
- El microservicio ML se implementará en Python en la carpeta `ml-service/` con su propio `recommendation_model.py` y `requirements.txt`.  
- La documentación en `README.md` se actualizará para reflejar la estructura, dependencias y guía de instalación.

