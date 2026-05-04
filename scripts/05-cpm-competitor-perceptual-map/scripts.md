# 5. SCRIPTS CPM (Competitor Perceptual Map)

**Nota:** *Los siguientes scripts están diseñados para el análisis de datos orientado a la estimación de competitivdad comercial, la generación de visualizaciones analíticas y la identificación de insights estratégicos a partir de datos históricos obtenidos de Legiscomex. Se sugiere implementar este flujo de trabajo en entornos de ejecución Python tales como:*

- [Google Colab](https://colab.research.google.com/)
- [Python-Fiddle](https://python-fiddle.com/)
- [Kaggle Notebooks](https://www.kaggle.com/code)
- [Deepnote](https://deepnote.com/)

## 5.1. Instalar e importar librerías

```
!pip install pandas openpyxl plotly matplotlib seaborn -q

import pandas as pd
import numpy as np
import plotly.graph_objects as go
import plotly.express as px
from google.colab import files
import io
from datetime import datetime

print("✅ Librerías instaladas e importadas correctamente")
```

## 5.2. Configuración inicial del análisis

```
def configurar_analisis():
    """
    Permite al usuario configurar el análisis:
    - Producto analizado
    - Mercado objetivo (país)
    - Descripción adicional
    """
    print("="*80)
    print("🔧 CONFIGURACIÓN DEL ANÁLISIS")
    print("="*80)

    producto = input("\n📦 ¿Qué producto estás analizando? (ej: Aguacate Hass): ").strip()
    mercado = input("🌍 ¿Cuál es el mercado objetivo? (ej: Arabia Saudita): ").strip()
    descripcion = input("📝 Descripción adicional (opcional): ").strip()

    config = {
        'producto': producto if producto else 'Producto sin especificar',
        'mercado': mercado if mercado else 'Mercado sin especificar',
        'descripcion': descripcion if descripcion else 'Sin descripción',
        'fecha_analisis': datetime.now().strftime('%Y-%m-%d')
    }

    print("\n✅ Configuración guardada:")
    print(f"   Producto: {config['producto']}")
    print(f"   Mercado: {config['mercado']}")
    print(f"   Fecha: {config['fecha_analisis']}")

    return config

# Ejecutar configuración
config_analisis = configurar_analisis()
```

## 5.3. Carga de datos desde excel

```
def cargar_datos_legiscomex():
    """
    Solicita al usuario cargar el archivo Excel de LegisComex
    y retorna un DataFrame de pandas
    """
    print("\n" + "="*80)
    print("📁 CARGA DE DATOS")
    print("="*80)
    print("\n📌 Asegúrate de que tu archivo Excel tenga los encabezados desde la fila A1")
    print("📌 El sistema detectará automáticamente las columnas disponibles\n")

    uploaded = files.upload()

    # Obtener el nombre del archivo
    filename = list(uploaded.keys())[0]

    # Leer el archivo Excel
    df = pd.read_excel(io.BytesIO(uploaded[filename]))

    print(f"\n✅ Archivo cargado: {filename}")
    print(f"📊 Dimensiones: {df.shape[0]} filas x {df.shape[1]} columnas")

    return df, filename

# Ejecutar la carga de datos
df_raw, nombre_archivo = cargar_datos_legiscomex()
```

## 5.4. Mapeo Interactivo de columnas

```
def mapear_columnas(df):
    """
    Permite al usuario mapear las columnas del Excel a las variables del análisis
    """
    print("\n" + "="*80)
    print("🗺️  MAPEO DE COLUMNAS")
    print("="*80)
    print("\nColumnas disponibles en tu archivo:")
    print("-" * 80)

    # Mostrar columnas numeradas
    for i, col in enumerate(df.columns, 1):
        print(f"  [{i}] {col}")

    print("-" * 80)

    # Función auxiliar para seleccionar columna
    def seleccionar_columna(mensaje, obligatorio=True):
        while True:
            respuesta = input(f"\n{mensaje}\n   Ingresa el número de columna: ").strip()

            if not respuesta and not obligatorio:
                return None

            try:
                idx = int(respuesta) - 1
                if 0 <= idx < len(df.columns):
                    columna = df.columns[idx]
                    print(f"   ✅ Seleccionado: {columna}")
                    return columna
                else:
                    print("   ❌ Número fuera de rango. Intenta nuevamente.")
            except ValueError:
                print("   ❌ Por favor ingresa un número válido.")

    print("\n🎯 Ahora vamos a mapear las columnas necesarias para el análisis:\n")

    # Mapeo de columnas
    mapeo = {}

    mapeo['exportador'] = seleccionar_columna(
        "1️⃣  ¿Cuál columna contiene el NOMBRE DEL EXPORTADOR?"
    )

    mapeo['valor_fob'] = seleccionar_columna(
        "2️⃣  ¿Cuál columna contiene el VALOR FOB (USD)?"
    )

    mapeo['precio_unitario'] = seleccionar_columna(
        "3️⃣  ¿Cuál columna contiene el PRECIO UNITARIO ($/kg o $/unidad)?"
    )

    # Columnas opcionales
    print("\n📌 Columnas opcionales (presiona Enter para omitir):")

    mapeo['codigo_producto'] = seleccionar_columna(
        "4️⃣  Código de producto/partida arancelaria (opcional):",
        obligatorio=False
    )

    mapeo['pais_destino'] = seleccionar_columna(
        "5️⃣  País de destino (opcional):",
        obligatorio=False
    )

    mapeo['fecha'] = seleccionar_columna(
        "6️⃣  Fecha de exportación (opcional):",
        obligatorio=False
    )

    print("\n" + "="*80)
    print("✅ MAPEO COMPLETADO")
    print("="*80)
    print("\nResumen de columnas mapeadas:")
    for clave, valor in mapeo.items():
        if valor:
            print(f"   • {clave}: {valor}")

    return mapeo

# Ejecutar mapeo de columnas
mapeo_columnas = mapear_columnas(df_raw)
```

## 5.5. Consolidación y análisis de datos

```
def consolidar_datos_exportadores(df, mapeo):
    """
    Consolida las transacciones por exportador usando el mapeo de columnas
    """
    print("\n" + "="*80)
    print("⚙️  PROCESANDO DATOS...")
    print("="*80)

    # Crear DataFrame con columnas renombradas
    df_trabajo = df.copy()

    # Renombrar columnas según mapeo
    renombrar = {
        mapeo['exportador']: 'Exportador',
        mapeo['valor_fob']: 'Valor_FOB',
        mapeo['precio_unitario']: 'Precio_Unitario'
    }

    # Agregar columnas opcionales si existen
    if mapeo.get('codigo_producto'):
        renombrar[mapeo['codigo_producto']] = 'Codigo_Producto'
    if mapeo.get('pais_destino'):
        renombrar[mapeo['pais_destino']] = 'Pais_Destino'
    if mapeo.get('fecha'):
        renombrar[mapeo['fecha']] = 'Fecha'

    df_trabajo = df_trabajo.rename(columns=renombrar)

    # Limpiar datos
    df_trabajo['Valor_FOB'] = pd.to_numeric(df_trabajo['Valor_FOB'], errors='coerce')
    df_trabajo['Precio_Unitario'] = pd.to_numeric(df_trabajo['Precio_Unitario'], errors='coerce')

    # Eliminar filas con valores nulos en columnas críticas
    df_trabajo = df_trabajo.dropna(subset=['Exportador', 'Valor_FOB', 'Precio_Unitario'])

    print(f"✅ Datos limpios: {len(df_trabajo)} transacciones válidas")

    # Agrupar por exportador
    df_consolidado = df_trabajo.groupby('Exportador').agg(
        Valor_FOB_Total=('Valor_FOB', 'sum'),
        Num_Envios=('Valor_FOB', 'count'),
        Precio_Promedio=('Precio_Unitario', 'mean')
    ).reset_index()

    # Calcular cuota de mercado
    total_fob = df_consolidado['Valor_FOB_Total'].sum()
    df_consolidado['Cuota_Mercado'] = (df_consolidado['Valor_FOB_Total'] / total_fob * 100).round(2)

    # Ordenar por Valor FOB Total descendente
    df_consolidado = df_consolidado.sort_values('Valor_FOB_Total', ascending=False).reset_index(drop=True)

    # Redondear valores
    df_consolidado['Valor_FOB_Total'] = df_consolidado['Valor_FOB_Total'].round(2)
    df_consolidado['Precio_Promedio'] = df_consolidado['Precio_Promedio'].round(2)

    return df_consolidado

# Ejecutar consolidación
df_consolidado = consolidar_datos_exportadores(df_raw, mapeo_columnas)

print("\n📊 TABLA CONSOLIDADA POR EXPORTADOR")
print("="*80)
print(df_consolidado.to_string(index=False))
print("="*80)

# Estadísticas descriptivas
print(f"\n📈 RESUMEN ESTADÍSTICO:")
print(f"   • Total exportadores: {len(df_consolidado)}")
print(f"   • Valor FOB Total: ${df_consolidado['Valor_FOB_Total'].sum():,.2f} USD")
print(f"   • Total envíos: {df_consolidado['Num_Envios'].sum()}")
print(f"   • Precio promedio mercado: ${df_consolidado['Precio_Promedio'].mean():.2f}")
```

## 5.6. Cálculo de cuadrantes (Volúmen vs. Precio)

```
def calcular_cuadrantes(df):
    """
    Calcula los cuadrantes según análisis Volumen vs Precio:
    - Eje X (VOLUMEN): Valor FOB Total (volumen de negocio)
    - Eje Y (PRECIO): Precio Unitario Promedio (estrategia de precio)
    - Tamaño burbuja: Cuota de mercado (participación)
    """

    # Calcular medianas para los ejes
    mediana_volumen = df['Valor_FOB_Total'].median()
    mediana_precio = df['Precio_Promedio'].median()

    # Función para asignar cuadrante
    def asignar_cuadrante(row):
        if row['Valor_FOB_Total'] >= mediana_volumen and row['Precio_Promedio'] >= mediana_precio:
            return 'Líder Premium'
        elif row['Valor_FOB_Total'] < mediana_volumen and row['Precio_Promedio'] >= mediana_precio:
            return 'Nicho Premium'
        elif row['Valor_FOB_Total'] >= mediana_volumen and row['Precio_Promedio'] < mediana_precio:
            return 'Líder en Volumen'
        else:
            return 'Competidor Económico'

    df['Cuadrante'] = df.apply(asignar_cuadrante, axis=1)

    return df, mediana_volumen, mediana_precio

# Calcular cuadrantes
df_consolidado, mediana_volumen, mediana_precio = calcular_cuadrantes(df_consolidado)

print(f"\n🎯 POSICIONAMIENTO POR CUADRANTES")
print("="*80)
print(f"   • Mediana Valor FOB (Volumen): ${mediana_volumen:,.2f} USD")
print(f"   • Mediana Precio (Estrategia): ${mediana_precio:.2f}")
print("\n📍 Distribución por cuadrante:")
print(df_consolidado.groupby('Cuadrante').size())
print("\n" + df_consolidado[['Exportador', 'Valor_FOB_Total', 'Precio_Promedio', 'Cuota_Mercado', 'Cuadrante']].to_string(index=False))
```

## 5.7. Generación del gráfico de Burbujas

```
def crear_mapa_perceptual(df, config, mediana_x, mediana_y):
    """
    Crea un gráfico de burbujas: Volumen (X) vs Precio (Y), Tamaño = Cuota
    """

    # Colores por cuadrante
    colores_cuadrante = {
        'Líder Premium': '#FF6B6B',           # Rojo - Alto volumen, Alto precio
        'Nicho Premium': '#4ECDC4',           # Turquesa - Bajo volumen, Alto precio
        'Líder en Volumen': '#95E1D3',        # Verde - Alto volumen, Bajo precio
        'Competidor Económico': '#FFA07A'     # Naranja - Bajo volumen, Bajo precio
    }

    df['Color'] = df['Cuadrante'].map(colores_cuadrante)

    # Crear figura
    fig = go.Figure()

    # Agregar burbujas por cuadrante
    for cuadrante in df['Cuadrante'].unique():
        df_cuad = df[df['Cuadrante'] == cuadrante]

        fig.add_trace(go.Scatter(
            x=df_cuad['Valor_FOB_Total'],
            y=df_cuad['Precio_Promedio'],
            mode='markers+text',
            name=cuadrante,
            marker=dict(
                size=df_cuad['Cuota_Mercado'] * 3,  # Tamaño = Cuota de mercado
                color=colores_cuadrante[cuadrante],
                opacity=0.7,
                line=dict(width=2, color='white'),
                sizemode='diameter'
            ),
            text=df_cuad['Exportador'],
            textposition='top center',
            textfont=dict(size=10, color='black'),
            hovertemplate='<b>%{text}</b><br>' +
                         'Valor FOB: $%{x:,.2f}<br>' +
                         'Precio: $%{y:.2f}<br>' +
                         'Cuota: ' + df_cuad['Cuota_Mercado'].astype(str) + '%<br>' +
                         '<extra></extra>'
        ))

    # Agregar líneas de cuadrantes
    fig.add_hline(y=mediana_y, line_dash="dash", line_color="gray", opacity=0.5)
    fig.add_vline(x=mediana_x, line_dash="dash", line_color="gray", opacity=0.5)

    # Agregar anotaciones de cuadrantes
    max_x = df['Valor_FOB_Total'].max() * 1.05
    max_y = df['Precio_Promedio'].max() * 1.05
    min_x = df['Valor_FOB_Total'].min() * 0.95
    min_y = df['Precio_Promedio'].min() * 0.95

    anotaciones = [
        dict(x=max_x * 0.85, y=max_y * 0.95,
             text="<b>LÍDER PREMIUM</b><br>Alto Volumen + Alto Precio",
             showarrow=False, font=dict(size=11, color='#FF6B6B'), bgcolor='rgba(255,255,255,0.8)'),
        dict(x=min_x + (mediana_x - min_x) * 0.5, y=max_y * 0.95,
             text="<b>NICHO PREMIUM</b><br>Bajo Volumen + Alto Precio",
             showarrow=False, font=dict(size=11, color='#4ECDC4'), bgcolor='rgba(255,255,255,0.8)'),
        dict(x=max_x * 0.85, y=min_y + (mediana_y - min_y) * 0.5,
             text="<b>LÍDER EN VOLUMEN</b><br>Alto Volumen + Precio Competitivo",
             showarrow=False, font=dict(size=11, color='#95E1D3'), bgcolor='rgba(255,255,255,0.8)'),
        dict(x=min_x + (mediana_x - min_x) * 0.5, y=min_y + (mediana_y - min_y) * 0.5,
             text="<b>COMPETIDOR ECONÓMICO</b><br>Bajo Volumen + Precio Bajo",
             showarrow=False, font=dict(size=11, color='#FFA07A'), bgcolor='rgba(255,255,255,0.8)')
    ]

    # Layout
    fig.update_layout(
        title={
            'text': f'<b>MATRIZ COMPETITIVA: VOLUMEN vs PRECIO</b><br>' +
                   f'<sub>{config["producto"]} - {config["mercado"]}</sub>',
            'x': 0.5,
            'xanchor': 'center',
            'font': {'size': 20}
        },
        xaxis_title='<b>VOLUMEN DE NEGOCIO</b><br>Valor FOB Total (USD)',
        yaxis_title='<b>ESTRATEGIA DE PRECIO</b><br>Precio Unitario Promedio (USD)',
        xaxis=dict(showgrid=True, gridwidth=1, gridcolor='lightgray', type='linear'),
        yaxis=dict(showgrid=True, gridwidth=1, gridcolor='lightgray'),
        plot_bgcolor='white',
        hovermode='closest',
        annotations=anotaciones,
        showlegend=True,
        legend=dict(
            title='<b>Posicionamiento Estratégico</b>',
            orientation='v',
            yanchor='top',
            y=1,
            xanchor='left',
            x=1.02
        ),
        width=1200,
        height=800
    )

    return fig

# Generar gráfico
fig_mapa = crear_mapa_perceptual(df_consolidado, config_analisis, mediana_volumen, mediana_precio)
fig_mapa.show()

print("✅ Mapa perceptual generado exitosamente")
```

## 5.8. Generación del Promt para análisis con iA

```
def generar_prompt_analisis(df, config):
    """
    Genera un prompt estructurado para análisis con cualquier chatbot de IA
    """

    # Convertir DataFrame a texto estructurado
    tabla_texto = df.to_string(index=False)

    # Fix: evitar triple backtick literal dentro del f-string
    backtick = "`"
    tb = backtick * 3

    prompt = f"""
# ANÁLISIS ESTRATÉGICO DE COMPETIDORES
## Matriz Competitiva: Volumen vs Precio

---

## INFORMACIÓN DEL ANÁLISIS:

- **Producto:** {config['producto']}
- **Mercado objetivo:** {config['mercado']}
- **Descripción:** {config['descripcion']}
- **Fecha del análisis:** {config['fecha_analisis']}

---

## DATOS CONSOLIDADOS:

{tb}
{tabla_texto}
{tb}

---

## CONTEXTO DEL ANÁLISIS:

Este análisis se basa en datos reales de exportación obtenidos de LegisComex, consolidados por exportador.

**Metodología de Análisis:**
- **Eje X (VOLUMEN)**: Valor FOB Total - mide el volumen total de negocio de cada exportador
- **Eje Y (PRECIO)**: Precio Unitario Promedio - refleja la estrategia de precio (premium vs económico)
- **Tamaño de burbuja**: Cuota de mercado (%) - indica la participación relativa en el mercado

**Cuadrantes Estratégicos:**

1. **LÍDER PREMIUM** (Alto Volumen + Alto Precio):
   - Grandes jugadores con poder de marca y diferenciación
   - Logran mantener precios premium con alto volumen de ventas
   - Ventaja competitiva sostenible en calidad/marca

2. **NICHO PREMIUM** (Bajo Volumen + Alto Precio):
   - Especialistas en segmentos de alto valor
   - Estrategia de diferenciación por calidad/origen/certificaciones
   - Bajo volumen pero márgenes atractivos

3. **LÍDER EN VOLUMEN** (Alto Volumen + Precio Competitivo):
   - Estrategia de liderazgo en costos y economías de escala
   - Compiten por volumen con precios accesibles
   - Alta eficiencia operativa

4. **COMPETIDOR ECONÓMICO** (Bajo Volumen + Precio Bajo):
   - Jugadores marginales o emergentes
   - Limitada diferenciación y bajo poder de mercado
   - Vulnerables a presiones competitivas

---

## INSTRUCCIONES PARA EL ANÁLISIS:

Por favor, genera un **reporte analítico estratégico detallado** que incluya:

### 1. ANÁLISIS POR CUADRANTE
Para cada cuadrante:
- Identificar las empresas posicionadas
- Analizar su estrategia comercial evidente (volumen vs precio)
- Evaluar sostenibilidad de su posición
- Identificar fortalezas y vulnerabilidades específicas
- Determinar barreras de entrada/salida de cada cuadrante

### 2. ANÁLISIS COMPETITIVO INDIVIDUAL (TOP 5)
Para cada uno de los 5 principales exportadores:
- Posicionamiento estratégico actual (cuadrante)
- Ventajas competitivas evidentes (¿por qué logran ese volumen/precio?)
- Vulnerabilidades potenciales
- Estrategia implícita: ¿liderazgo en costos o diferenciación?
- Movimientos estratégicos probables

### 3. ESTRUCTURA Y DINÁMICA DEL MERCADO
Analizar:
- Nivel de concentración (índice Herfindahl si es posible)
- Distribución de poder de mercado
- Correlación entre volumen y precio (¿hay economías de escala evidentes?)
- Brechas y espacios competitivos vacíos
- Barreras de entrada al mercado

### 4. ANÁLISIS DE ESTRATEGIAS DOMINANTES
Identificar:
- ¿Qué estrategia predomina: volumen o diferenciación?
- ¿Existe correlación entre precio y cuota de mercado?
- ¿Hay evidencia de guerra de precios?
- ¿Se observan segmentos premium desatendidos?
- Patrones de comportamiento competitivo

### 5. OPORTUNIDADES ESTRATÉGICAS
Para nuevos entrantes o reposicionamiento:
- ¿Qué espacios estratégicos están subexplotados?
- ¿Cuál sería el posicionamiento óptimo de entrada?
- ¿Qué estrategia de precio es más viable?
- ¿Qué volumen objetivo sería realista?
- Recursos y capacidades críticas necesarias

### 6. ANÁLISIS DE AMENAZAS Y RIESGOS
Evaluar:
- Riesgo de commoditización del mercado
- Amenaza de concentración excesiva
- Vulnerabilidad ante movimientos de precios
- Riesgos de los jugadores por cuadrante
- Amenazas de nuevos entrantes o sustitutos

### 7. RECOMENDACIONES ESTRATÉGICAS PRIORIZADAS
Proporcionar:
- TOP 3 recomendaciones para líder del mercado
- TOP 3 recomendaciones para nuevo entrante
- Estrategias de reposicionamiento por cuadrante
- Movimientos estratégicos de corto plazo (6-12 meses)
- Estrategias de largo plazo (2-3 años)

---

## FORMATO DE SALIDA ESPERADO:

- Estructura en markdown con jerarquía clara (H1, H2, H3)
- Insights cuantitativos específicos con datos de la tabla
- Recomendaciones accionables y priorizadas
- Tablas comparativas donde sea relevante
- Gráficas conceptuales en texto (si es útil)
- Sección de **Resumen Ejecutivo** al inicio (5-7 puntos clave)
- Sección de **Conclusiones y Próximos Pasos** al final

---

## DATOS ADICIONALES PARA CONTEXTO:

**Estadísticas clave del mercado:**
- Total exportadores analizados: {len(df)}
- Valor FOB total del mercado: ${df['Valor_FOB_Total'].sum():,.2f} USD
- Rango de precios: ${df['Precio_Promedio'].min():.2f} - ${df['Precio_Promedio'].max():.2f}
- Precio promedio ponderado: ${(df['Valor_FOB_Total'] * df['Precio_Promedio']).sum() / df['Valor_FOB_Total'].sum():.2f}
- Concentración TOP 3: {df.head(3)['Cuota_Mercado'].sum():.1f}% del mercado
- Índice de dispersión de precios: {((df['Precio_Promedio'].std() / df['Precio_Promedio'].mean()) * 100):.1f}%

**Distribución por cuadrante:**
{df.groupby('Cuadrante')['Exportador'].count().to_string()}

**Cuota de mercado por cuadrante:**
{df.groupby('Cuadrante')['Cuota_Mercado'].sum().round(1).to_string()}
"""

    return prompt


# Generar prompt
prompt_ia = generar_prompt_analisis(df_consolidado, config_analisis)

print("\n" + "="*80)
print("📝 PROMPT GENERADO PARA ANÁLISIS CON IA")
print("="*80)
print("\n💡 Copia el siguiente texto y pégalo en cualquier chatbot (ChatGPT, Claude, Gemini, etc.):\n")
print(prompt_ia)
print("\n" + "="*80)

# Guardar prompt en archivo de texto
nombre_prompt = f'prompt_analisis_{config_analisis["producto"].replace(" ", "_")}_{config_analisis["fecha_analisis"]}.txt'
with open(nombre_prompt, 'w', encoding='utf-8') as f:
    f.write(prompt_ia)

print(f"\n✅ Prompt guardado en: '{nombre_prompt}'")
```

## 5.9. Exportación de resultados

```
def exportar_resultados(df, fig, config):
    """
    Exporta los resultados consolidados a Excel y el gráfico a HTML
    """
    print("\n" + "="*80)
    print("💾 EXPORTANDO RESULTADOS")
    print("="*80)

    # Generar nombres de archivo
    sufijo = f"{config['producto'].replace(' ', '_')}_{config['mercado'].replace(' ', '_')}_{config['fecha_analisis']}"

    # Exportar tabla consolidada a Excel
    nombre_excel = f'analisis_competidores_{sufijo}.xlsx'

    # Crear Excel con múltiples hojas
    with pd.ExcelWriter(nombre_excel, engine='openpyxl') as writer:
        # Hoja 1: Datos consolidados
        df.to_excel(writer, sheet_name='Competidores', index=False)

        # Hoja 2: Configuración del análisis
        df_config = pd.DataFrame([config])
        df_config.to_excel(writer, sheet_name='Configuración', index=False)

        # Hoja 3: Resumen por cuadrante
        df_cuadrantes = df.groupby('Cuadrante').agg({
            'Exportador': 'count',
            'Valor_FOB_Total': 'sum',
            'Cuota_Mercado': 'sum',
            'Precio_Promedio': 'mean'
        }).reset_index()
        df_cuadrantes.columns = ['Cuadrante', 'Num_Empresas', 'Valor_FOB_Total',
                                 'Cuota_Total', 'Precio_Promedio']
        df_cuadrantes.to_excel(writer, sheet_name='Resumen_Cuadrantes', index=False)

    print(f"   ✅ Tabla Excel: {nombre_excel}")

    # Exportar gráfico a HTML interactivo
    nombre_html = f'matriz_competitiva_{sufijo}.html'
    fig.write_html(nombre_html)
    print(f"   ✅ Gráfico HTML: {nombre_html}")

    # Descargar archivos
    print("\n📥 Descargando archivos...")
    files.download(nombre_excel)
    files.download(nombre_html)
    files.download(nombre_prompt)

    return nombre_excel, nombre_html

# Ejecutar exportación
archivo_excel, archivo_html = exportar_resultados(df_consolidado, fig_mapa, config_analisis)
```

## 5.10. Resúmen ejecutivo final

```
print("\n" + "="*80)
print("🎯 RESUMEN EJECUTIVO DEL ANÁLISIS")
print("="*80)

# Identificar líder del mercado
lider = df_consolidado.iloc[0]

print(f"""
📋 INFORMACIÓN DEL ANÁLISIS:
   • Producto: {config_analisis['producto']}
   • Mercado: {config_analisis['mercado']}
   • Fecha: {config_analisis['fecha_analisis']}

📊 ESTADÍSTICAS CLAVE:
   • Total exportadores analizados: {len(df_consolidado)}
   • Valor FOB total del mercado: ${df_consolidado['Valor_FOB_Total'].sum():,.2f} USD
   • Total de envíos registrados: {df_consolidado['Num_Envios'].sum()}
   • Precio promedio ponderado: ${(df_consolidado['Valor_FOB_Total'] * df_consolidado['Precio_Promedio']).sum() / df_consolidado['Valor_FOB_Total'].sum():.2f}

👑 LÍDER DEL MERCADO:
   • Empresa: {lider['Exportador']}
   • Valor FOB: ${lider['Valor_FOB_Total']:,.2f} USD
   • Cuota de mercado: {lider['Cuota_Mercado']:.1f}%
   • Precio promedio: ${lider['Precio_Promedio']:.2f}
   • Posicionamiento: {lider['Cuadrante']}

🏆 TOP 3 EXPORTADORES:
{df_consolidado.head(3)[['Exportador', 'Valor_FOB_Total', 'Precio_Promedio', 'Cuota_Mercado', 'Cuadrante']].to_string(index=False)}

📍 DISTRIBUCIÓN POR CUADRANTE:
{df_consolidado.groupby('Cuadrante')['Exportador'].count().to_string()}

💰 CUOTA DE MERCADO POR CUADRANTE:
{df_consolidado.groupby('Cuadrante')['Cuota_Mercado'].sum().round(1).to_string()}

💡 INSIGHTS PRINCIPALES:
   1. Concentración: Top 3 controla {df_consolidado.head(3)['Cuota_Mercado'].sum():.1f}% del mercado
   2. Líder posicionado como: {lider['Cuadrante']}
   3. Precio máximo: ${df_consolidado['Precio_Promedio'].max():.2f} | Mínimo: ${df_consolidado['Precio_Promedio'].min():.2f}
   4. Diferencial de precio: {((df_consolidado['Precio_Promedio'].max() / df_consolidado['Precio_Promedio'].min() - 1) * 100):.1f}%
   5. Cuadrante dominante: {df_consolidado.groupby('Cuadrante').size().idxmax()} ({df_consolidado.groupby('Cuadrante').size().max()} empresas)
   6. Volumen promedio por empresa: ${df_consolidado['Valor_FOB_Total'].mean():,.2f} USD
   7. Dispersión de precios (CV): {((df_consolidado['Precio_Promedio'].std() / df_consolidado['Precio_Promedio'].mean()) * 100):.1f}%

✅ ARCHIVOS GENERADOS:
   • Tabla consolidada (Excel): {archivo_excel}
   • Matriz competitiva interactiva (HTML): {archivo_html}
   • Prompt para análisis IA: {nombre_prompt}

🚀 PRÓXIMOS PASOS:
   1. 📊 Abre '{archivo_html}' para explorar la matriz interactiva
   2. 📋 Revisa '{archivo_excel}' con 3 hojas: Competidores, Configuración y Resumen
   3. 🤖 Copia el contenido de '{nombre_prompt}' y pégalo en tu chatbot preferido
   4. 🎯 Analiza el posicionamiento estratégico de cada competidor
   5. 💼 Define tu estrategia: ¿Volumen o Precio? ¿Qué cuadrante objetivo?
   6. 📈 Identifica oportunidades en espacios competitivos vacíos
""")

print("="*80)
print("✅ ANÁLISIS COMPLETADO EXITOSAMENTE")
print("="*80)
print(f"\n🎉 Matriz Competitiva generada: {config_analisis['producto']} en {config_analisis['mercado']}")
print("📧 Sistema listo para analizar cualquier producto/mercado.")
print("\n💡 TIP: La matriz muestra VOLUMEN (X) vs PRECIO (Y), con burbujas = CUOTA DE MERCADO")
```
