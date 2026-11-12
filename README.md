# Predicción e Interpretabilidad de Deserción Bancaria (Bank Churn) - Caso FinAlpes

**Curso:** MINE-4101: Ciencia de Datos Aplicada  
**Institución:** Universidad de los Andes — Departamento de Ingeniería de Sistemas y Computación  
**Semestre:** 2026-02
**Estudiante:** Joubert Daniel Alvarez Ramirez    

---

## 1. Contexto del Problema y Objetivo de Negocio

En el sistema financiero, donde la bancarización de la población adulta supera el 96%, el crecimiento y la sostenibilidad dependen de retener a los clientes actuales antes que asumir el alto costo de adquirir nuevos. Atraer a un cliente nuevo cuesta varias veces más que retener a uno existente.

FinAlpes, fintech colombiana de consultoría y analítica avanzada, estructuró una prueba de concepto predictiva e interpretable a partir de una muestra de 10.000 clientes bancarios europeos. El propósito central es validar y adaptar esta herramienta para una entidad bancaria de la región con una base objetivo de **20.000 clientes**, permitiendo anticipar el abandono (*churn*) y desplegar campañas de retención focalizadas y comercialmente viables.

---

## 2. Estructura del Repositorio

El proyecto sigue una estructura limpia, modular y documentada:

```text
TALLER2CHURN/
├── data/raw
│   └── Bank_Customer_Churn_Prediction.csv   # Dataset original
├── notebooks/
│   └── taller_churn.ipynb                   # Notebook ejecutable de inicio a fin
├── README.md                                # Informe ejecutivo y técnico
└── requirements.txt                         # Dependencias y librerías del proyecto
```

### Instrucciones para ejecutarlo

Sigue estos pasos en la terminal para descargar y ejecutar el proyecto de principio a fin:

1. **Clonar el repositorio y entrar a la carpeta del proyecto:**
   ```bash
   git clone https://github.com/JoubertD/TALLER2CHURN.git
   cd TALLER2CHURN
   ```

2. **Crear y activar un entorno virtual (recomendado con Python 3.10 o superior):**
   - **En Windows (PowerShell / CMD):**
     ```powershell
     python -m venv .venv
     .\.venv\Scripts\activate
     ```
   - **En Linux / macOS:**
     ```bash
     python3 -m venv .venv
     source .venv/bin/activate
     ```

3. **Instalar todas las librerías requeridas desde `requirements.txt`:**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

4. **Abrir y ejecutar el notebook:**
   - **Opción A — Desde Jupyter Lab:**
     ```bash
     jupyter lab notebooks/taller_churn.ipynb
     ```
   - **Opción B — Desde VS Code:**
     Abrir la carpeta `TALLER2CHURN` en VS Code, abrir el archivo `notebooks/taller_churn.ipynb`, seleccionar el kernel `.venv` en la esquina superior derecha y presionar **Run All** (*Ejecutar todo*).

> **Nota:** El flujo de trabajo ejecuta todas las celdas secuencialmente 

---

## 3. Preparación de Datos y Selección de Características

### 3.1. Partición de Datos (Sin Fuga de Información)
El conjunto original (tasa natural de abandono del 20,35%) se dividió de manera estratificada en:
* **Entrenamiento (Train - 60%):** 6.000 clientes. Ajuste de codificadores, escaladores e hiperparámetros.
* **Validación (Validation - 20%):** 2.000 clientes. Comparación de familias de modelos y calibración de umbrales.
* **Prueba (Test Hold-Out - 20%):** 2.000 clientes intactos (1.593 clientes leales y 407 desertores reales) para la validación definitiva.

### 3.2. Transformación y Selección de Variables: Originales vs. Finales

| Variable Original | Estado Final | Tratamiento y Justificación Metodológica |
| :--- | :---: | :--- |
| `customer_id` | **Descartada** | Identificador de fila; sin poder predictivo. |
| `credit_card` | **Descartada** | Estadísticamente irrelevante ($p > 0,49$ en pruebas de independencia). |
| `tenure` | **Descartada** | No aportó significancia marginal frente al principio de parsimonia. |
| `estimated_salary` | **Descartada** | Distribución uniforme entre clientes que abandonan y clientes retenidos. |
| `gender` | **Descartada** | **Eliminada tras validación experimental** (ver subsección 3.3). |
| `age` | **Conservada** | Escalada con `StandardScaler`. Factor continuo principal de riesgo. |
| `balance` | **Conservada** | Escalada con `StandardScaler`. Clave en la interacción geográfica. |
| `credit_score` | **Conservada** | Escalada con `StandardScaler`. Evaluación de solvencia de base. |
| `country` | **Conservada** | Codificada con `OneHotEncoder`. Alta diferenciación de riesgo (Alemania vs. Francia/España). |
| `products_number` | **Conservada** | Codificada con `OneHotEncoder`. Relación fuertemente no lineal. |
| `active_member` | **Conservada** | Codificada con `OneHotEncoder`. Factor protector clave frente a la deserción. |

### 3.3. Experimentación Controlada: Con vs. Sin `gender` (Ética y Rendimiento)
Se evaluó el comportamiento de los modelos al incluir y excluir la variable de género:
* **Desempeño en Random Forest:** Al retirar `gender`, el modelo mejoró en todas sus métricas en validación:
  * El **Recall** aumentó de $72,79\%$ a **$73,77\%$** ($+0,98\%$).
  * La **Precision** aumentó de $49,01\%$ a **$49,34\%$** ($+0,33\%$).
  * El **F1-Score** subió de $0,5858$ a **$0,5914$**.
* **Conclusión Técnica y Ética:** La variable `gender` introducía ruido al ensamble sin aportar señal predictiva genuina. Su eliminación consolida un modelo parsimonioso y alineado con los principios de **equidad algorítmica (*algorithmic fairness*)**, garantizando que la concesión de ofertas de retención no dependa de atributos demográficos protegidos.

---

## 4. Comparación Cuantitativa y Modelo Definitivo

Se contrastaron cinco familias de algoritmos mediante *5-fold Stratified Cross-Validation* y calibración de umbrales en el conjunto de validación sin la variable `gender`:

| Modelo | Umbral | Precision (Churn=1) | Recall (Churn=1) | F1-Score (Churn=1) |
| :--- | :---: | :---: | :---: | :---: |
| **XGBoost (Óptimo F1)** | 0.640 | 63,57% | 60,29% | 0,6189 |
| **XGBoost (Base)** | 0.500 | 54,28% | 68,38% | 0,6052 |
| **XGBoost (Calibrado)** | 0.435 | 50,17% | 72,79% | 0,5940 |
| **Random Forest (Seleccionado)** | **0.500** | **49,34%** | **73,77%** | **0,5914** |
| **Regresión Logística** | 0.590 | 51,65% | 68,87% | 0,5903 |
| **Árbol de Decisión (`max_depth=9`)** | 0.500 | 47,73% | 72,30% | 0,5750 |
| **KNN** | 0.500 | 61,81% | 46,81% | 0,5328 |

### Evaluación Definitiva en el Conjunto de Prueba (Test Hold-Out: 2.000 Clientes)
Se ratificó como modelo definitivo el **Random Forest** (`class_weight='balanced'`, `max_depth=6`, `min_samples_leaf=4`, `min_samples_split=10`, `n_estimators=200`, `umbral=0.500`).

La matriz de confusión en el conjunto de prueba final arrojó:
* **Verdaderos Negativos ($TN$):** 1.297 clientes leales identificados.
* **Falsos Positivos ($FP$):** 296 clientes leales contactados por error (falsas alarmas).
* **Falsos Negativos ($FN$):** 109 clientes en fuga no detectados.
* **Verdaderos Positivos ($TP$):** 298 clientes desertores detectados a tiempo.

**Métricas Finales en Test:**
* **Recall:** **$73,22\%$** (captura casi 3 de cada 4 fugas reales).
* **Precisión:** **$50,17\%$** (balance equilibrado de falsas alarmas).
* **Exactitud Global:** **$79,75\%$**.
* **Fall-out (Tasa de Falsas Alarmas):** **$18,58\%$**.

---

## 5. Insights Cualitativos e Interpretabilidad

A través del análisis SHAP (global y local), curvas ICE (Expectativa Condicional Individual) y LIME, se identificaron los siguientes patrones clave del comportamiento de los clientes:

1. **La Regla de Oro de los Productos (`products_number`):**
   * **2 productos:** Constituye el factor protector más potente de todo el modelo (valores SHAP fuertemente negativos). Fideliza al cliente y reduce al mínimo el riesgo de abandono.
   * **1 producto:** Deja al cliente en situación de vulnerabilidad por baja vinculación transaccional.
   * **3 o 4 productos:** Dispara el riesgo de deserción de manera casi determinista (valores SHAP $> +0,20$ e incrementos abruptos en curvas ICE), señalando insatisfacción grave por sobre-empaquetamiento forzado o fricciones de servicio.
2. **Quiebre Demográfico por Edad (`age`):**
   * El riesgo permanece controlado por debajo de los 40 años, pero experimenta una aceleración notable entre los **40 y 55 años**, rango que concentra la mayor propensión de fuga de capitales.
3. **Rol Mitigador de la Actividad (`active_member`):**
   * Estar activo transaccionalmente amortigua significativamente el efecto de la edad, manteniendo el riesgo en niveles manejables aún en clientes mayores.
4. **Disparidad Geográfica (`country_Germany`):**
   * Los clientes de Alemania presentan una propensión estructuralmente mayor al abandono, en especial aquellos con saldos bancarios (`balance`) elevados.

---

## 6. Generación de Valor y Modelo de Negocio

La evaluación de impacto financiero se realizó sobre la cartera bancaria objetivo de **20.000 clientes** (tasa base de Churn del $20,35\%$ = 4.070 clientes en riesgo), asumiendo una efectividad comercial de retención del **$30\%$** ($\eta = 0,30$) y un ingreso anual por cliente activo (**ARPU**) de **COP $700.000** (benchmark de neobancos regionales: COP $500.000 a $900.000).

### 6.1. Evaluación del Costo de Contacto y Riesgo de Falsas Alarmas
Se contrastaron dos enfoques metodológicos de costos:

* **Escenario de Estrés (Supuesto didáctico de COP $200.000 por contacto):**
  * Si el banco asume un costo de contacto de COP $200.000 (incentivo comercial agresivo pagado a todo contactado, se quede o no), el ingreso esperado por Verdadero Positivo ($TP$) es de $0,30 \times \$700.000 = \$210.000$, dejando un margen de apenas **+COP $10.000** ($b(P,p)$).
  * Por el contrario, cada Falso Positivo ($FP$) destruye directamente **-COP $200.000** ($c(P,n)$).
  * Consecuencia: Se requieren 20 clientes salvados exitosamente solo para amortizar el gasto de 1 falsa alarma. Bajo la fórmula de Provost & Fawcett, este escenario produce una pérdida operativa neta de **-COP $28.108 por cliente evaluado** (-COP $562,1 millones en la cartera), evidenciando que el umbral de 0.50 no es sostenible si el costo del contacto supera el valor de corte crítico de **COP $105.350**.

* **Escenario Operativo Realista (Canales omnicanal a COP $45.000 por contacto):**
  * Al gestionar la retención mediante canales digitales (Push, SMS, WhatsApp) y bonificaciones ajustadas, el margen por cliente detectado ($b(P,p)$) sube a **+COP $165.000**.
  * La ganancia esperada bajo la fórmula de Provost alcanza **+COP $17.925 por cliente evaluado**.
  * Retorno bruto de la campaña en 20.000 clientes: **+COP $358.500.000**.
  * Descontando el costo fijo de implementación y despliegue del modelo (COP $40.000.000), el beneficio neto final asciende a **COP $318.500.000** (ROI de **$796,25\%$**).
  * **Punto de Equilibrio (*Break-Even Point*):** La inversión del proyecto analítico se recupera en tan solo **1,34 meses** de operación (~40 días).

---

## 7. Recomendaciones Finales y Limitaciones

### Accionables de Negocio
1. **Focalización en el "Segundo Producto":** Diseñar campañas de venta cruzada para clientes con 1 solo producto hacia un segundo producto ancla (nómina o tarjeta), evitando la venta indiscriminada de 3 o más productos.
2. **Intervención Temprana en Segmento Vulnerable:** Establecer reglas de retención automática ante caídas de actividad en clientes entre 40 y 55 años.
3. **Calibración de Umbrales por Costo:** Si el banco utiliza campañas de retención costosas (> COP $100.000), se debe elevar el umbral de probabilidad a $p \ge 0,70$ para reducir el *Fall-out* a menos del $4\%$. Para canales de bajo costo, operar en el umbral estándar de $0,50$.

### Limitaciones Identificadas
* **Validez Externa:** El modelo fue calibrado sobre clientes europeos; se recomienda ejecutar una prueba piloto local para adaptar las probabilidades base a la dinámica colombiana.
* **Ausencia de Modelado de Uplift Causal:** El clasificador estima probabilidad de deserción, no la sensibilidad al incentivo. Algunos clientes en riesgo inminente pueden ser irrecuperables independientemente de la oferta.
* **Corte Transversal Estático:** El dataset no incorpora variables de tendencia temporal reciente (como la variación de saldos en los últimos 30 a 90 días).

---

## 8. Declaración de Uso de IA Generativa

En cumplimiento de las directrices académicas del curso, se deja constancia del uso de herramientas de Inteligencia Artificial Generativa (**Claude / Gemini**) durante el desarrollo del taller:

* **Modalidad de Interacción:** Se interactuó con el modelo de lenguaje de forma **iterativa y guiada**, solicitando explicaciones conceptuales, validaciones matemáticas y propuestas de código para componentes específicos que luego fueron contrastados, ejecutados y ajustados por los integrantes del equipo, la IA se uso para la mayor parte del codigo aunque las decisiones de diseño, pruebas y conclusiones fueron netamente por el autor.
* (apoyo metodológico, en codigo y contraste de ecuaciones).
* **Alcance del Apoyo:**
  1. Revisión de las ecuaciones financieras de evaluación avanzada (Fórmula Simplificada y Ecuación de Provost & Fawcett).
  2. Apoyo en la sintaxis para empaquetar transformadores y clasificadores en Pipelines compatibles con SHAP y LIME.
  3. Redacción ejecutiva, síntesis y corrección de estilo para este archivo `README.md`.
* **Autoría del Proyecto:** La formulación de hipótesis, la inspección exploratoria de datos, la decisión de remover la variable `gender`, la seleccion de columnas del dataset original, la selección del modelo final, la interpretación del contexto bancario y la ejecución completa del código fueron realizadas directamente por los autores del taller.