# 2. PROMT ANÁLISIS BENCHMARK - D.MANHATTAN

**Nota:** *Adjuntar Tabla resultado del Cálculo Manhattan en sintaxis Markdown preferiblemente.*

```
Quiero que analices exclusivamente la siguiente tabla de datos de potencial de exportación.
⚠️ Importante: tu análisis debe basarse **únicamente en los datos contenidos en la tabla** que te proporcionaré.
No uses información externa, no inventes cifras ni añadas contexto ajeno a los datos.

La tabla contiene las siguientes columnas:
- Index
- País
- Potencial Exportación (mUSD)
- Exportaciones Referencia (mUSD)
- Potencial sin Explotar (mUSD)
- Ratio Potencial/Actual (F)
- Índice Brecha (G)
- Región
- Score Similitud (distancia de Manhattan respecto al país benchmark seleccionado)
- Ranking

### Instrucción clave
El análisis debe hacerse tomando como país de referencia o **benchmark: [NOMBRE DEL PAÍS]**

---

### Objetivos del análisis

1. **Identificar mercados comparables**
   - Indica qué países tienen el **Score de Similitud más bajo** con respecto al benchmark, es decir, los más parecidos en términos de Ratio Potencial/Actual e Índice Brecha.

2. **Detectar outliers (mercados alejados del benchmark)**
   - Señala qué países tienen el **Score más alto**, es decir, los más distintos al benchmark, y explica qué significa esa diferencia únicamente con base en los datos de la tabla.

3. **Priorización estratégica**
   - Sugiere en qué países sería más factible replicar una estrategia similar a la del benchmark considerando:
     - Score bajo (alta similitud).
     - Región (cercanía cultural o comercial según tabla).
     - Potencial de exportación o brecha sin explotar.

4. **Clusterización de mercados**
   - Agrupa los países en tres categorías según su similitud con el benchmark:
     - 🟢 Muy similares (score bajo).
     - 🟡 Medianamente similares (score intermedio).
     - 🔴 Muy distintos (score alto).

5. **Hallazgos e insights de utilidad**
   - Explica patrones detectados a partir de los datos, como por ejemplo:
     - Regiones con mayor similitud al benchmark.
     - Países que podrían representar **oportunidades rápidas de expansión**.
     - Países que requieren **estrategias diferenciadas** por su lejanía al benchmark.

---

### Formato de salida esperado

1. **Resumen ejecutivo** → 3–5 frases clave sobre el benchmark y principales hallazgos.
2. **Tabla o lista** de países más similares y más distintos.
3. **Priorización estratégica** con justificación basada en los datos.
4. **Clusterización** en 🟢, 🟡, 🔴.
5. **Insights extra** → patrones regionales, tendencias o anomalías detectadas.

⚠️ Reitero: analiza únicamente los datos de la tabla proporcionada, sin añadir información externa.
```

