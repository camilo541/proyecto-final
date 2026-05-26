# proyecto-final

# Chef-Costos — Ecosistema Tecnológico End-to-End

> **Taller Final Corte 3 · Programación y Decisiones · Universidad de La Sabana · 2026-1**  
> Profesor: Diego Mauricio Zuluaga Rodríguez

---

##  Descripción del Proyecto

**Chef-Costos** es un sistema integral de control de costos para restaurantes, diseñado para monitorear la inflación de ingredientes frente al mercado colombiano.

El sistema permite:
- Registrar ingredientes con su **precio histórico** y **precio de mercado actual**
- Calcular automáticamente el **% de inflación** y generar **alertas cuando supera el 20%**
- Calcular el **precio sugerido de venta** (+35% sobre el costo real del plato)
- Visualizar tendencias en un **dashboard de Power BI** conectado a la base de datos

---

##  Arquitectura del Ecosistema

```
chef-costos/
│
├── main.py                    ← 🚀 PUNTO DE ENTRADA PRINCIPAL
│
├── Backend/
│   ├── database.py            ← Gestión SQLite + CRUD + datos iniciales
│   ├── modelos.py             ← Clases POO (RegistroCosto, Restaurante)
│   └── chef_costos.db         ← Base de datos SQLite (generada automáticamente)
│
├── Frontend/
│   └── app.py                 ← Interfaz gráfica Tkinter (CRUD + botón Power BI)
│
├── PowerBI/
│   ├── chef_costos.pbix       ← Dashboard Power BI (Modelo Estrella + DAX)
│   └── guia_powerbi_DAX.txt   ← Medidas DAX y columnas calculadas documentadas
│
└── README.md
```

---

##  Modelo Estrella — Base de Datos SQLite

```
        ┌──────────────────┐
        │  Dim_Categoria   │
        │  id_categoria PK │
        └────────┬─────────┘
                 │ 1:N
        ┌────────▼─────────┐          ┌──────────────────────┐
        │  Dim_Ingrediente │──── 1:N ──│    Fact_Costos        │
        │  id_ingrediente  │          │    id_registro PK     │
        │  nombre          │          │    id_ingrediente FK  │
        │  unidad_medida   │          │    nombre_plato       │
        │  id_categoria FK │          │    precio_historico   │
        │  id_proveedor FK │          │    precio_mercado     │
        └──────────────────┘          │    cantidad_usada     │
                 ▲                    │    fecha_registro     │
                 │ 1:N                └──────────────────────┘
        ┌──────────────────┐
        │  Dim_Proveedor   │
        │  id_proveedor PK │
        └──────────────────┘
```

**Tablas:** 4 tablas | **Registros iniciales:** ≥ 5 por tabla (auto-generados)

---

## Instalación y Ejecución

### Requisitos previos
```bash
Python 3.10 o superior (tkinter incluido)
```

### 1. Clonar el repositorio
```bash
git clone https://github.com/TU_USUARIO/chef-costos.git
cd chef-costos
```

### 2.  Crear entorno virtual
```bash
python -m venv venv
# Windows:
venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate
```

### 3. Instalar dependencias
```bash
pip install -r requirements.txt
```

### 4. ▶️ Ejecutar el proyecto
```bash
python main.py
```

Esto hace automáticamente:
1.  Crea e inicializa la base de datos `Backend/chef_costos.db`
2. Inserta los datos iniciales (≥5 registros por tabla)
3.  Lanza la interfaz gráfica Tkinter

---

##  Interfaz Gráfica (Tkinter)

| Botón | Función |
|-------|---------|
|  **REGISTRAR** | Agrega un nuevo ingrediente/plato con validación POO |
|  **VER / LEER** | Recarga la tabla con todos los registros de la BD |
|  **ACTUALIZAR** | Modifica el registro seleccionado en la tabla |
|  **ELIMINAR** | Elimina el registro seleccionado con confirmación |
|  **ABRIR POWER BI** | Abre el archivo `.pbix` directamente desde Python |

- Las filas con **inflación > 20%** se resaltan en **rojo** automáticamente
- Se usa `try-except` + `messagebox` en todas las operaciones para evitar colapsos

---

##  Lógica de Negocio — Clases POO

```python
# RegistroCosto calcula automáticamente:
registro = RegistroCosto("Pollo a la Plancha", 12000, 15000, 0.3)

registro.inflacion        # → 25.0 %
registro.tiene_alerta     # → True  (>20%)
registro.costo_plato      # → 4500  (15000 × 0.3)
registro.precio_sugerido  # → 6075  (4500 × 1.35)
```

---

##  Power BI — DAX Implementado

### Columnas Calculadas
| Nombre | Fórmula |
|--------|---------|
| `Clasificacion_Inflacion` | IF inflación >20% → "ALERTA", >10% → "Moderada", else "Normal" |
| `Precio_Sugerido_Venta` | `precio_mercado × cantidad_usada × 1.35` |
| `Costo_Real_Plato` | `precio_mercado × cantidad_usada` |
| `Pct_Inflacion` | `(mercado - historico) / historico × 100` |

### Medidas DAX
| Medida | Descripción |
|--------|-------------|
| `Inflacion_Promedio` | AVERAGEX de % inflación |
| `Ingredientes_Con_Alerta` | CALCULATE + COUNTROWS con filtro >20% |
| `Costo_Promedio_Plato` | AVERAGEX de costo real |
| `Precio_Sugerido_Promedio` | AVERAGEX de precio sugerido |
| `Pct_Ingredientes_Alerta` | DIVIDE de alertas / total |

### Gráficas del Dashboard
1.  **KPI Cards** — Inflación promedio, alertas, costo y precio sugerido
2.  **Barras agrupadas** — Precio histórico vs mercado por plato
3.  **Barras de inflación** — % inflación por ingrediente con línea ref. 20%
4.  **Dona** — Distribución por estado de inflación (Normal / Moderada / Alerta)
5.  **Matriz** — Detalle por categoría y proveedor

---

##  Checklist de Requisitos del Taller

- [x] Carpeta `Backend/` con lógica, POO y base de datos
- [x] Carpeta `Frontend/` con interfaz gráfica Tkinter
- [x] `main.py` orquestador en la raíz
- [x] Archivo `.pbix` de Power BI en `PowerBI/`
- [x] `README.md` detallado con instrucciones
- [x] Base de datos SQLite con Esquema Estrella (1 hecho + 3 dimensiones)
- [x] Mínimo 5 registros auto-generados por tabla
- [x] 4 botones CRUD funcionales (Registrar, Ver, Actualizar, Eliminar)
- [x] Botón para abrir Power BI desde Python
- [x] `try-except` + `messagebox` en todas las operaciones
- [x] Clases POO (`RegistroCosto`, `Restaurante`, `Ingrediente`)
- [x] Mínimo 1 medida DAX + 1 columna calculada DAX
- [x] Mínimo 4 gráficas en Power BI
- [x] Modelo Estrella activo en Power BI





Nicolas Torres Calderon  
Administracion de negocios internacionales 
Universidad de La Sabana · Semestre 2026-1  
GitHub: [@TU_USUARIO](https://github.com/TU_USUARIO)

---

*Proyecto académico — Programación y Decisiones 2026-1*
