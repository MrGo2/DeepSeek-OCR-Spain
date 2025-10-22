# 🇪🇸 DeepSeek-OCR para Contratos Financieros Españoles

**Notebooks de Google Colab optimizados para extraer datos de contratos financieros españoles**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/MrGo2/DeepSeek-OCR-Spain/blob/main/DeepSeek_OCR_Colab_Spain_Financial.ipynb)

---

## 📋 ¿Qué hace este proyecto?

Extrae automáticamente campos clave de:
- **Préstamos al consumo**: TAE, TIN, importe, comisiones
- **Hipotecas**: Capital, euríbor, FEIN, gastos notariales
- **Tarjetas de crédito**: Límite, TAE revolving, pago mínimo

Y valida requisitos legales:
- ✅ Ley 16/2011 (Crédito al Consumo)
- ✅ Ley 5/2019 (Crédito Inmobiliario)
- ✅ Detección de usura (TAE >27%)
- ✅ Cláusula suelo, IRPH

---

## 🚀 Uso Rápido (5 minutos)

### 1. Abrir en Colab

**Opción A - Contratos financieros españoles** (recomendado):

[![Open Spain Financial](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/MrGo2/DeepSeek-OCR-Spain/blob/main/DeepSeek_OCR_Colab_Spain_Financial.ipynb)

### 2. Activar GPU

1. **Runtime → Change runtime type**
2. Seleccionar **T4 GPU**
3. Click **Save**

### 3. Ejecutar

1. **Runtime → Run all**
2. Esperar 15-20 min (primera vez)
3. Subir tu contrato PDF
4. Ver campos extraídos + validación legal

---

## 📁 Archivos del Proyecto

| Archivo | Descripción |
|---------|-------------|
| **DeepSeek_OCR_Colab_Spain_Financial.ipynb** | ⭐ Notebook principal para contratos españoles |
| **DeepSeek_OCR_Colab_with_PDF.ipynb** | Notebook genérico con soporte PDF |
| **DeepSeek_OCR_Colab.ipynb** | Notebook básico (solo imágenes) |
| **ADVANCED_FEATURES.md** | Guía técnica completa (España) |
| **DEEPSEEK_OCR_OPTIONS_GUIDE.md** | Referencia de parámetros |
| **COLAB_SETUP.md** | Guía de instalación |

---

## 📈 Rendimiento

| Tipo Contrato | Páginas | Tiempo | Precisión TAE |
|---------------|---------|--------|---------------|
| Préstamo | 4-6 | ~1 min | 98% |
| Hipoteca | 15-20 | ~4 min | 95% |
| Tarjeta | 6-8 | ~1.5 min | 99% |

**Hardware**: Google Colab Pro (T4 GPU)

---

## 🔗 Enlaces

- **Modelo original**: [deepseek-ai/DeepSeek-OCR](https://huggingface.co/deepseek-ai/DeepSeek-OCR)
- **Paper**: [arXiv:2510.18234](https://arxiv.org/abs/2510.18234)

---

**Creado por**: Carlos Lorenzo Santos | **Fecha**: Octubre 2025
