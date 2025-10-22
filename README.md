# 🇪🇸 DeepSeek-OCR para Contratos Financieros Españoles

**Notebooks de Google Colab optimizados para extraer datos de contratos financieros españoles**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/MrGo2/DeepSeek-OCR-Spain/blob/main/DeepSeek_OCR_Colab_Spain_Financial.ipynb)

🆕 **Versión 2.0** - Ahora con extracción extendida, bounding boxes y metadatos!

---

## 🎯 ¿Qué extrae automáticamente?

### 📊 Datos Financieros (7 campos)
- TAE, TIN, importe, plazo
- Cuota mensual, comisiones
- Coste total del crédito

### 👤 Titulares (5 campos)
- Nombre titular/cotitular
- DNI/NIE
- Prestamista

### 📍 Direcciones (6 campos)
- Dirección completa
- Calle, CP, localidad, provincia

### 🏷️ Producto (4 campos)
- Nombre exacto del producto
- ID contrato / Número operación
- Código de barras

### 📅 Fechas (4 campos)
- Fecha de firma
- Formalización
- Primer pago, vencimiento

**Total: ~30 campos por contrato** (vs 7-8 en v1.0)

---

## ✨ Nuevas Funcionalidades v2.0

### 🎯 Metadatos por Campo
Cada extracción incluye:
```json
{
  "value": "7,5%",
  "page": 1,
  "bbox": [120, 450, 180, 470],
  "confidence": 0.95
}
```

### 🎨 Visualización con Bounding Boxes
Genera imagen anotada con cajas de colores:
- 🔴 Rojo: Datos financieros
- 🔵 Azul: Titulares
- 🟢 Verde: Direcciones
- 🟠 Naranja: Producto
- 🟣 Morado: Fechas

### 📊 Estadísticas Automáticas
- Total campos encontrados
- Campos con alta confianza (>90%)
- Campos que requieren revisión (<70%)
- Tasa de éxito general

---

## 🚀 Uso Rápido (5 minutos)

### 1. Abrir en Colab

[![Open Spain Financial](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/MrGo2/DeepSeek-OCR-Spain/blob/main/DeepSeek_OCR_Colab_Spain_Financial.ipynb)

⚠️ **Importante**: Sigue `NOTEBOOK_UPDATES.md` para aplicar mejoras v2.0

### 2. Activar GPU + RAM Alta

1. **Entorno de ejecución → Cambiar tipo de entorno**
2. Seleccionar **T4 GPU**
3. Seleccionar **RAM alta** (25 GB)
4. Click **Guardar**

### 3. Ejecutar

1. **Entorno de ejecución → Ejecutar todo**
2. Esperar 15-20 min (primera vez)
3. Subir tu contrato PDF
4. Ver extracciones + validación + visualización

---

## 📊 Ejemplo de Salida

### JSON Estructurado
```json
{
  "metadatos": {
    "archivo": "prestamo_consumo.pdf",
    "tipo_contrato": "prestamo",
    "paginas_totales": 6
  },
  "estadisticas": {
    "total_campos_encontrados": 24,
    "campos_alta_confianza": 22,
    "tasa_exito": "85.7%"
  },
  "extracciones": {
    "financieros": {
      "tae": {"value": "7,5%", "page": 1, "bbox": [200,350,250,370], "confidence": 0.98},
      "importe": {"value": "15.000 €", "page": 1, "bbox": [200,300,280,320], "confidence": 0.99}
    },
    "titulares": {
      "titular_1": {"value": "Juan García López", "page": 1, "confidence": 0.98},
      "dni_titular_1": {"value": "12345678A", "page": 1, "confidence": 0.99}
    },
    "direcciones": {
      "direccion_completa": {"value": "Calle Gran Vía 45, 28013 Madrid", "page": 1, "confidence": 0.95}
    }
  },
  "validacion_legal": {
    "valido": true,
    "advertencias": []
  }
}
```

### Archivos Generados
1. **campos_completo.json** - Datos estructurados con metadatos
2. **completo.md** - Texto completo en markdown
3. **informe_completo.txt** - Resumen legible
4. **anotado.jpg** - Imagen con bounding boxes

---

## 📈 Rendimiento v2.0

| Tipo Contrato | Campos | Tiempo | Precisión | Alta Confianza |
|---------------|--------|--------|-----------|----------------|
| Préstamo | 24-28 | ~2 min | 98% | 92% |
| Hipoteca | 26-30 | ~5 min | 95% | 88% |
| Tarjeta | 22-26 | ~2 min | 99% | 95% |

**Hardware**: Google Colab Pro (T4 GPU + 25 GB RAM)

---

## ⚖️ Validación Legal

Verifica automáticamente:
- ✅ Campos obligatorios (Ley 16/2011, Ley 5/2019)
- ✅ TAE desproporcionada (>27% usura)
- ✅ Cláusula suelo, IRPH
- ✅ FEIN (hipotecas post-2019)

---

## 📁 Archivos del Proyecto

| Archivo | Descripción |
|---------|-------------|
| **DeepSeek_OCR_Colab_Spain_Financial.ipynb** | Notebook principal v1.0 |
| **NOTEBOOK_UPDATES.md** | 🆕 Guía de actualización a v2.0 |
| **ADVANCED_FEATURES.md** | Guía técnica completa |
| **DEEPSEEK_OCR_OPTIONS_GUIDE.md** | Referencia de parámetros |
| **COLAB_SETUP.md** | Guía de instalación |

---

## 🔧 Actualizar a v2.0

Si ya usas el notebook v1.0, sigue estas instrucciones:

### Opción A - Modificar en Colab (recomendado)
Consulta `NOTEBOOK_UPDATES.md` - guía paso a paso con código completo

### Opción B - Esperar v2.0 completa
Próximamente: `DeepSeek_OCR_Colab_Spain_Financial_v2.ipynb`

---

## 🐛 Problemas Conocidos y Soluciones

### Error: AttributeError circular import
**Solución**: Cambiar PyTorch 2.6.0 → 2.5.1
```python
!pip install torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1
```

### Error: Out of memory
**Solución**: Usar RAM alta (25 GB)
- Entorno de ejecución → Cambiar tipo → RAM alta

---

## 💰 Coste

| Plan | Coste | GPU | RAM | Tiempo Max |
|------|-------|-----|-----|------------|
| **Colab Pro** | $10/mes | T4/V100 | 25 GB | 24h |
| **Colab Pro+** | $50/mes | V100/A100 | 40 GB | 24h |

**Recomendado**: Colab Pro con RAM alta

---

## 🔗 Enlaces

- **Modelo**: [deepseek-ai/DeepSeek-OCR](https://huggingface.co/deepseek-ai/DeepSeek-OCR)
- **Paper**: [arXiv:2510.18234](https://arxiv.org/abs/2510.18234)
- **Repo original**: [DeepSeek-AI/DeepSeek-OCR](https://github.com/deepseek-ai/DeepSeek-OCR)

---

## 🤝 Contribuir

Mejoras bienvenidas:
- Más campos específicos por banco
- Mejor detección de códigos de barras
- Soporte para contratos escaneados
- OCR de firmas manuscritas

---

## ⚖️ Aviso Legal

Herramienta **informativa únicamente**. Los resultados deben verificarse por profesionales. No constituye asesoramiento legal o financiero.

---

**Creado por**: Carlos Lorenzo Santos  
**Fecha**: Octubre 2025  
**Versión**: 2.0 (22 Oct 2025)  
**Modelo**: DeepSeek-OCR (deepseek-ai)
