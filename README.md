# Sistema NBA (Next Best Action) con IA Generativa y Aprendizaje por Refuerzo

Este proyecto implementa un sistema de recomendación Next Best Action (NBA) para un banco, combinando IA generativa para la creación de datos sintéticos y aprendizaje por refuerzo para la optimización de recomendaciones.

## Estructura del Proyecto

El proyecto se divide en dos partes principales, cada una implementada en su propio notebook:

### 1. Generación de Datos Sintéticos (`generacion_datos_sinteticos.ipynb`)

Este notebook implementa un sistema de generación de datos sintéticos que simula un entorno bancario realista. La generación se realiza en tres capas interconectadas:

#### 1. Base de Clientes (1000 registros)
##### Implementación:
```python
def generar_base_clientes(n_clientes=1000):
    segmentos = ['Premium', 'Classic', 'Basic']
    canales = ['Digital', 'Sucursal', 'Call Center', 'App Móvil']
    
    # Distribución realista de segmentos
    # Premium: 20%, Classic: 50%, Basic: 30%
    probabilidades_segmento = [0.2, 0.5, 0.3]
```

##### Características y Lógica de Negocio:
1. **ID Cliente**: 
   - Formato: `CLI_XXXXXX` (ej: CLI_000001)
   - Garantiza identificadores únicos y trazables

2. **Edad**:
   - Rango: 18-80 años
   - Distribución uniforme para simular base diversa

3. **Segmento**:
   - Premium: Clientes de alto valor
     * Ingresos: 8,000-20,000
     * Proporción: 20% de la base
   - Classic: Clientes establecidos
     * Ingresos: 4,000-8,000
     * Proporción: 50% de la base
   - Basic: Clientes nuevos/básicos
     * Ingresos: 1,000-4,000
     * Proporción: 30% de la base

4. **Canal Preferido**:
   - Opciones equilibradas entre:
     * Digital (app/web)
     * Sucursal física
     * Call Center
     * App Móvil

#### 2. Base de Productos (50 productos)
##### Implementación:
```python
def generar_base_productos(n_productos=50):
    categorias = {
        'Tarjetas de Crédito': (0.15, 0.25),
        'Préstamos Personales': (0.08, 0.15),
        'Inversiones': (0.03, 0.07),
        'Seguros': (0.20, 0.35),
        'Cuentas de Ahorro': (0.02, 0.05)
    }
```

##### Características y Lógica de Negocio:
1. **ID Producto**:
   - Formato: `PRD_XXXX` (ej: PRD_0001)
   - Facilita seguimiento y análisis

2. **Categorías y Márgenes**:
   - Tarjetas de Crédito:
     * Margen: 15-25%
     * Mayor riesgo, mayor retorno
   - Préstamos Personales:
     * Margen: 8-15%
     * Riesgo moderado
   - Inversiones:
     * Margen: 3-7%
     * Bajo riesgo, menor margen
   - Seguros:
     * Margen: 20-35%
     * Alto margen por comisiones
   - Cuentas de Ahorro:
     * Margen: 2-5%
     * Producto básico, bajo margen

#### 3. Base de Interacciones (5000 registros)
##### Implementación:
```python
def generar_base_interacciones(df_clientes, df_productos, n_interacciones=5000):
    # Probabilidades base de aceptación por segmento
    prob_base = {
        'Premium': 0.7,  # 70% prob. base
        'Classic': 0.5,  # 50% prob. base
        'Basic': 0.3     # 30% prob. base
    }
```

##### Características y Lógica de Negocio:
1. **ID Interacción**:
   - Formato: `INT_XXXXXX` (ej: INT_000001)
   - Registro único por interacción

2. **Modelo de Probabilidad de Aceptación**:
   - Basado en dos factores:
     * Segmento del cliente (prob_base)
     * Margen del producto
   - Fórmula:
     ```python
     prob_ajustada = prob_base * (1 - margen_esperado)
     ```
   - Lógica: A mayor margen, menor probabilidad de aceptación

3. **Temporalidad**:
   - Fechas distribuidas en el último año
   - Permite análisis de estacionalidad

4. **Canal de Interacción**:
   - Hereda el canal preferido del cliente
   - Simula comportamiento realista

#### Validaciones y Controles
1. **Consistencia de Datos**:
   - IDs únicos en todas las bases
   - Referencias válidas entre tablas
   - Rangos de valores coherentes

2. **Análisis Básico**:
   - Distribución de segmentos
   - Estadísticas de aceptación
   - Márgenes promedio

3. **Almacenamiento**:
   - Formato CSV para fácil acceso
   - Estructura: datos_sinteticos/
     * clientes.csv
     * productos.csv
     * interacciones.csv

### 2. Agente de Aprendizaje por Refuerzo (`agente_rl_nba.ipynb`)

Este notebook implementa un sistema NBA utilizando un algoritmo Multi-Armed Bandit contextual, diseñado específicamente para optimizar recomendaciones bancarias:

#### Fundamentos del Modelo

##### Multi-Armed Bandit Contextual
- **Concepto**: Extensión del problema clásico del bandido multi-brazo que considera el contexto (segmento del cliente) para tomar decisiones
- **Objetivo**: Maximizar la recompensa acumulada mientras se aprende qué productos funcionan mejor para cada segmento
- **Ventaja**: Equilibra eficientemente la exploración de nuevas opciones y la explotación del conocimiento existente

#### Componentes del Sistema

1. **Estados (Contexto)**:
   - **Segmentos de Clientes**:
     * Premium: Clientes de alto valor (ingresos > 8000)
     * Classic: Clientes de valor medio (ingresos 4000-8000)
     * Basic: Clientes de valor inicial (ingresos < 4000)
   - **Importancia**: El segmento determina las preferencias y capacidad de compra

2. **Acciones (Brazos del Bandido)**:
   - **Productos Bancarios Disponibles**:
     * Tarjetas de Crédito (margen 15-25%)
     * Préstamos Personales (margen 8-15%)
     * Inversiones (margen 3-7%)
     * Seguros (margen 20-35%)
     * Cuentas de Ahorro (margen 2-5%)
   - **Consideraciones**: Cada producto tiene diferentes características de riesgo y retorno

3. **Sistema de Recompensas**:
   - **Fórmula**: R = margen_esperado * aceptación
   - **Componentes**:
     * Aceptación: Variable binaria (1 si acepta, 0 si rechaza)
     * Margen esperado: Porcentaje específico del producto
   - **Ejemplo**: Si un cliente acepta un seguro con margen de 30%, R = 0.30

#### Algoritmo en Detalle

1. **Inicialización**:
   ```python
   Q[segmento, producto] = 0  # Valores Q iniciales
   N[segmento, producto] = 0  # Contador de selecciones
   ```

2. **Política de Selección (ε-greedy)**:
   - **Exploración (ε = 0.1)**:
     * 10% del tiempo: Selección aleatoria de productos
     * Permite descubrir nuevas oportunidades
   - **Explotación (1-ε = 0.9)**:
     * 90% del tiempo: Selecciona el producto con mayor Q-value
     * Aprovecha el conocimiento acumulado

3. **Actualización de Q-values**:
   ```python
   Q[s,a] = Q[s,a] + (1/N[s,a]) * (R - Q[s,a])
   ```
   Donde:
   - s: segmento del cliente
   - a: producto seleccionado
   - R: recompensa obtenida
   - N[s,a]: número de veces que se ha probado esta combinación

#### Características Avanzadas

1. **Aprendizaje Personalizado**:
   - Matrices Q separadas para cada segmento
   - Permite capturar preferencias específicas
   - Facilita recomendaciones personalizadas

2. **Memoria de Experiencias**:
   - Contador N para cada par segmento-producto
   - Ajusta el tamaño de los pasos de aprendizaje
   - Mayor confianza en experiencias frecuentes

3. **Adaptabilidad**:
   - Se ajusta a cambios en preferencias
   - Aprende de manera continua
   - Balance dinámico exploración/explotación

#### Sistema de Evaluación y Métricas

1. **Framework de Evaluación**:
   - **Simulación de Escenarios**:
     * 1000 episodios de interacción
     * Selección aleatoria de clientes
     * Registro de decisiones y resultados
   
   - **Baseline Aleatorio**:
     * Selección completamente aleatoria de productos
     * Sirve como punto de referencia
     * Representa la estrategia más simple posible

2. **Métricas Clave**:
   - **Recompensa Promedio**:
     * Calculada por episodio
     * Indica la eficiencia del aprendizaje
     * Muestra la tendencia de mejora
   
   - **Tasa de Mejora**:
     * Porcentaje de mejora vs baseline
     * Medida del valor agregado del modelo
     * Justificación del enfoque RL

   - **Q-values por Segmento**:
     * Matriz de valores aprendidos
     * Muestra preferencias por segmento
     * Facilita interpretación del modelo

3. **Visualizaciones Analíticas**:
   - **Curvas de Aprendizaje**:
     * Evolución temporal del rendimiento
     * Comparación visual con baseline
     * Identificación de convergencia

   - **Mapas de Calor**:
     * Distribución de Q-values
     * Patrones por segmento
     * Productos más prometedores

## Resultados y Visualizaciones

1. **Gráfico de Aprendizaje**:
   - Muestra la evolución de la recompensa promedio
   - Compara el desempeño del agente contra el baseline

2. **Mapa de Calor de Q-values**:
   - Visualiza las preferencias aprendidas por segmento
   - Ayuda a identificar los mejores productos para cada tipo de cliente

3. **Top 3 Productos**:
   - Lista los productos más exitosos por segmento
   - Incluye los Q-values correspondientes

## Uso del Sistema

1. Primero ejecutar `generacion_datos_sinteticos.ipynb` para crear los datos
2. Luego ejecutar `agente_rl_nba.ipynb` para entrenar y evaluar el agente
3. Los resultados se guardan en la carpeta `datos_sinteticos/`

## Requisitos Técnicos

- Python 3.11.6
- Librerías principales:
  - numpy
  - pandas
  - matplotlib
  - seaborn
  - scikit-learn

## Ventajas del Sistema

1. **Aprendizaje Continuo**: Mejora constantemente basándose en las interacciones
2. **Personalización**: Adapta recomendaciones por segmento de cliente
3. **Optimización de Margen**: Considera tanto la probabilidad de aceptación como el margen esperado
4. **Transparencia**: Permite visualizar y entender las decisiones del modelo

## Limitaciones y Posibles Mejoras

1. **Contexto Temporal**: Incorporar estacionalidad y tendencias
2. **Features Adicionales**: Incluir más características del cliente y producto
3. **Modelos Avanzados**: Implementar algoritmos más sofisticados como Deep Q-Learning
4. **Validación Online**: Implementar pruebas A/B en ambiente real
