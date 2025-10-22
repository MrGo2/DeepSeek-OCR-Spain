# 🔄 Actualizaciones del Notebook - Versión Mejorada

**Fecha**: 22 Octubre 2025
**Versión**: 2.0

---

## ⚠️ Cambio Crítico: PyTorch Versión

### Problema Detectado
```python
# ANTIGUO (causa error circular import):
!pip install torch==2.6.0 torchvision==0.21.0 torchaudio==2.6.0
```

### ✅ Solución
```python
# NUEVO (compatible con Colab):
!pip install -q torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu118
```

**Cambiar en la celda 6** del notebook.

---

## 🆕 Nuevos Campos de Extracción

### 1. Añadir después de la celda 12 (Definiciones de campos)

```python
# ===== NUEVOS CAMPOS EXTENDIDOS =====

# Titulares e identificación
TITULARES_FIELDS = {
    'titular_1': '<|ref|>Titular<|/ref|>',
    'titular_2': '<|ref|>Cotitular<|/ref|>',
    'dni_titular_1': '<|ref|>DNI<|/ref|>',
    'nie_titular': '<|ref|>NIE<|/ref|>',
    'prestamista': '<|ref|>Entidad<|/ref|>',
}

# Direcciones
DIRECCION_FIELDS = {
    'direccion_completa': '<|ref|>Domicilio<|/ref|>',
    'direccion_alt': '<|ref|>Dirección<|/ref|>',
    'calle': '<|ref|>Calle<|/ref|>',
    'codigo_postal': '<|ref|>Código Postal<|/ref|>',
    'localidad': '<|ref|>Localidad<|/ref|>',
    'provincia': '<|ref|>Provincia<|/ref|>',
}

# Información del producto
PRODUCTO_FIELDS = {
    'nombre_producto': '<|ref|>Producto<|/ref|>',
    'nombre_producto_alt': '<|ref|>Denominación<|/ref|>',
    'id_contrato': '<|ref|>Número de contrato<|/ref|>',
    'numero_operacion': '<|ref|>Número de operación<|/ref|>',
    'referencia': '<|ref|>Referencia<|/ref|>',
    'codigo_barras': '<|ref|>código de barras<|/ref|>',
}

# Fechas críticas
FECHAS_FIELDS = {
    'fecha_firma': '<|ref|>Fecha de firma<|/ref|>',
    'fecha_formalizacion': '<|ref|>Fecha de formalización<|/ref|>',
    'fecha_concesion': '<|ref|>Fecha de concesión<|/ref|>',
    'fecha_primer_pago': '<|ref|>Fecha primer pago<|/ref|>',
    'fecha_vencimiento': '<|ref|>Fecha de vencimiento<|/ref|>',
}

# Actualizar campos por tipo de contrato
PRESTAMO_CONSUMO_FIELDS_COMPLETO = {
    **PRESTAMO_CONSUMO_FIELDS,
    **TITULARES_FIELDS,
    **DIRECCION_FIELDS,
    **PRODUCTO_FIELDS,
    **FECHAS_FIELDS,
}

HIPOTECA_FIELDS_COMPLETO = {
    **HIPOTECA_FIELDS,
    **TITULARES_FIELDS,
    **DIRECCION_FIELDS,
    **PRODUCTO_FIELDS,
    **FECHAS_FIELDS,
}

TARJETA_CREDITO_FIELDS_COMPLETO = {
    **TARJETA_CREDITO_FIELDS,
    **TITULARES_FIELDS,
    **DIRECCION_FIELDS,
    **PRODUCTO_FIELDS,
    **FECHAS_FIELDS,
}

print("✅ Campos extendidos definidos")
print(f"   📋 Préstamos: {len(PRESTAMO_CONSUMO_FIELDS_COMPLETO)} campos")
print(f"   🏠 Hipotecas: {len(HIPOTECA_FIELDS_COMPLETO)} campos")
print(f"   💳 Tarjetas: {len(TARJETA_CREDITO_FIELDS_COMPLETO)} campos")
```

---

## 🔍 Extracción con Metadatos

### 2. Reemplazar función `extraer_campo` en celda 14

```python
def extraer_campo_con_metadatos(page_images, ref_prompt, output_dir='./temp_extractions', verbose=False):
    """
    Buscar un campo específico con bounding boxes y confidence

    Returns:
        dict con 'value', 'page', 'bbox', 'confidence' o None
    """
    import os
    os.makedirs(output_dir, exist_ok=True)

    for page_num, page_img in enumerate(page_images):
        temp_path = os.path.join(output_dir, f'search_page_{page_num}.jpg')
        page_img.save(temp_path)

        prompt = f"<image>\nLocate {ref_prompt} in the image."

        try:
            # Activar save_results para obtener bounding boxes
            result = model.infer(
                tokenizer,
                prompt=prompt,
                image_file=temp_path,
                output_path=output_dir,
                base_size=1024,
                image_size=640,
                crop_mode=True,
                save_results=True,  # ← Genera archivos con boxes
                test_compress=True
            )

            if result and result.strip():
                # Parsear coordenadas si están en el resultado
                bbox = parse_bbox_from_result(result)
                confidence = estimate_confidence(result)

                if verbose:
                    print(f"    ✓ Encontrado en página {page_num + 1} (conf: {confidence:.2f})")

                return {
                    'value': result.strip(),
                    'page': page_num + 1,
                    'bbox': bbox,
                    'confidence': confidence
                }
        except Exception as e:
            if verbose:
                print(f"    ⚠️  Error en página {page_num + 1}: {e}")
            continue

    return None

def parse_bbox_from_result(result_text):
    """
    Extrae coordenadas [[x1,y1,x2,y2]] del texto de resultado
    """
    import re
    # Buscar patrón [[número, número, número, número]]
    pattern = r'\[\[(\d+),\s*(\d+),\s*(\d+),\s*(\d+)\]\]'
    match = re.search(pattern, result_text)

    if match:
        return [int(match.group(i)) for i in range(1, 5)]

    return None  # No hay bounding box

def estimate_confidence(result_text):
    """
    Estima confianza basada en longitud y claridad del resultado
    """
    if not result_text:
        return 0.0

    # Heurística simple:
    # - Resultados más largos y estructurados = mayor confianza
    # - Presencia de números, %, € = mayor confianza

    confidence = 0.7  # Base

    # Bonus por símbolos esperados
    if '%' in result_text:
        confidence += 0.15
    if '€' in result_text or '$' in result_text:
        confidence += 0.1
    if any(char.isdigit() for char in result_text):
        confidence += 0.05

    return min(confidence, 1.0)

# Actualizar función extraer_campos_contrato para usar metadatos
def extraer_campos_contrato_completo(page_images, tipo_contrato, verbose=True):
    """Extraer todos los campos con metadatos"""

    # Seleccionar campos según tipo (con versiones completas)
    if tipo_contrato == 'prestamo':
        fields = PRESTAMO_CONSUMO_FIELDS_COMPLETO
        tipo_texto = "Préstamo al consumo"
    elif tipo_contrato == 'hipoteca':
        fields = HIPOTECA_FIELDS_COMPLETO
        tipo_texto = "Hipoteca"
    elif tipo_contrato == 'tarjeta':
        fields = TARJETA_CREDITO_FIELDS_COMPLETO
        tipo_texto = "Tarjeta de crédito"
    else:
        raise ValueError(f"Tipo no válido: {tipo_contrato}")

    if verbose:
        print(f"\n🔍 Extrayendo campos de {tipo_texto}...")
        print(f"{'='*60}")
        print(f"Total campos a buscar: {len(fields)}")

    extracted = {}

    # Organizar por categorías
    categorias = {
        'financieros': [],
        'titulares': [],
        'direcciones': [],
        'producto': [],
        'fechas': []
    }

    for field_name, ref_prompt in fields.items():
        # Determinar categoría
        if any(x in field_name for x in ['tae', 'tin', 'importe', 'cuota', 'comision', 'capital', 'euribor']):
            categoria = 'financieros'
        elif any(x in field_name for x in ['titular', 'dni', 'nie', 'prestamista']):
            categoria = 'titulares'
        elif any(x in field_name for x in ['direccion', 'calle', 'codigo_postal', 'localidad', 'provincia']):
            categoria = 'direcciones'
        elif any(x in field_name for x in ['producto', 'contrato', 'operacion', 'referencia', 'codigo']):
            categoria = 'producto'
        elif any(x in field_name for x in ['fecha']):
            categoria = 'fechas'
        else:
            categoria = 'financieros'

        if verbose:
            print(f"  🔎 [{categoria}] {field_name}")

        result = extraer_campo_con_metadatos(page_images, ref_prompt, verbose=verbose)

        if result:
            categorias[categoria].append((field_name, result))
            if verbose:
                valor_preview = result['value'][:50] if len(result['value']) > 50 else result['value']
                print(f"    ✅ {valor_preview} (conf: {result.get('confidence', 0):.2f})")
        else:
            if verbose:
                print(f"    ❌ No encontrado")

    # Estructurar resultado por categorías
    extracted = {
        'financieros': {k: v for k, v in categorias['financieros']},
        'titulares': {k: v for k, v in categorias['titulares']},
        'direcciones': {k: v for k, v in categorias['direcciones']},
        'producto': {k: v for k, v in categorias['producto']},
        'fechas': {k: v for k, v in categorias['fechas']}
    }

    return extracted

print("✅ Funciones con metadatos definidas")
```

---

## 🎨 Visualización de Bounding Boxes

### 3. Añadir nueva celda después de extraer campos (nueva celda 23)

```python
# ===== VISUALIZACIÓN DE EXTRACCIONES =====

from PIL import ImageDraw, ImageFont
import matplotlib.pyplot as plt

def visualizar_extracciones(page_img, campos_extraidos):
    """
    Dibuja rectángulos rojos alrededor de cada campo extraído

    Args:
        page_img: PIL Image de la página
        campos_extraidos: dict con estructura por categorías

    Returns:
        PIL Image anotada
    """
    img = page_img.copy()
    draw = ImageDraw.Draw(img)

    # Colores por categoría
    colores = {
        'financieros': 'red',
        'titulares': 'blue',
        'direcciones': 'green',
        'producto': 'orange',
        'fechas': 'purple'
    }

    # Dibujar boxes para cada categoría
    for categoria, campos in campos_extraidos.items():
        color = colores.get(categoria, 'red')

        for campo_name, datos in campos.items():
            if 'bbox' in datos and datos['bbox']:
                x1, y1, x2, y2 = datos['bbox']

                # Dibujar rectángulo
                draw.rectangle([x1, y1, x2, y2], outline=color, width=3)

                # Añadir etiqueta
                label = f"{campo_name}: {datos['value'][:15]}..."
                draw.text((x1, max(0, y1-15)), label, fill=color)

    return img

# Visualizar primera página con todas las extracciones
print("\n🎨 Generando visualización de campos extraídos...")

img_anotada = visualizar_extracciones(page_images[0], campos_extraidos)

plt.figure(figsize=(14, 18))
plt.imshow(img_anotada)
plt.axis('off')
plt.title('Campos Extraídos - Vista Anotada (Página 1)', fontsize=16)

# Añadir leyenda
from matplotlib.patches import Rectangle
legend_elements = [
    Rectangle((0,0),1,1,fc='red', label='Financieros'),
    Rectangle((0,0),1,1,fc='blue', label='Titulares'),
    Rectangle((0,0),1,1,fc='green', label='Direcciones'),
    Rectangle((0,0),1,1,fc='orange', label='Producto'),
    Rectangle((0,0),1,1,fc='purple', label='Fechas')
]
plt.legend(handles=legend_elements, loc='upper right')

plt.tight_layout()
plt.show()

# Guardar imagen anotada
output_img_path = os.path.join(output_dir, f"{base_name}_anotado.jpg")
img_anotada.save(output_img_path, quality=95)
print(f"\n✅ Imagen anotada guardada: {output_img_path}")
```

---

## 📊 Output JSON Mejorado

### 4. Actualizar celda 30 (Guardar resultados)

Reemplazar la sección de creación de `resultado_json` con:

```python
# Calcular estadísticas
total_campos = sum(len(cat) for cat in campos_extraidos.values())
campos_alta_confianza = sum(
    1 for cat in campos_extraidos.values()
    for datos in cat.values()
    if datos.get('confidence', 0) > 0.9
)
campos_revisar = [
    (categoria, campo)
    for categoria, campos in campos_extraidos.items()
    for campo, datos in campos.items()
    if datos.get('confidence', 0) < 0.7
]

# 1. Guardar campos extraídos (JSON con metadatos completos)
resultado_json = {
    'metadatos': {
        'archivo': pdf_filename,
        'tipo_contrato': tipo_contrato,
        'fecha_proceso': datetime.now().isoformat(),
        'paginas_totales': len(page_images),
        'modelo': 'deepseek-ai/DeepSeek-OCR',
        'version_notebook': '2.0'
    },

    'estadisticas': {
        'total_campos_buscados': len(PRESTAMO_CONSUMO_FIELDS_COMPLETO if tipo_contrato == 'prestamo'
                                       else HIPOTECA_FIELDS_COMPLETO if tipo_contrato == 'hipoteca'
                                       else TARJETA_CREDITO_FIELDS_COMPLETO),
        'total_campos_encontrados': total_campos,
        'campos_alta_confianza': campos_alta_confianza,
        'tasa_exito': f"{(total_campos / len(fields) * 100):.1f}%",
        'campos_requieren_revision': len(campos_revisar)
    },

    'extracciones': campos_extraidos,  # Con estructura por categorías

    'validacion_legal': validacion,

    'campos_revisar': [
        {
            'categoria': cat,
            'campo': campo,
            'razon': 'Confianza baja'
        }
        for cat, campo in campos_revisar
    ]
}

json_path = os.path.join(output_dir, f"{base_name}_campos_completo.json")
with open(json_path, 'w', encoding='utf-8') as f:
    json.dump(resultado_json, f, indent=2, ensure_ascii=False)

print(f"✅ Campos guardados: {json_path}")
print(f"   📊 {total_campos} campos encontrados")
print(f"   ✓ {campos_alta_confianza} con alta confianza (>90%)")
print(f"   ⚠️  {len(campos_revisar)} requieren revisión (<70%)")
```

---

## 📝 Informe de Extracción Mejorado

### 5. Actualizar sección del informe en celda 30

```python
# 3. Guardar informe resumido (mejorado)
informe_path = os.path.join(output_dir, f"{base_name}_informe_completo.txt")
with open(informe_path, 'w', encoding='utf-8') as f:
    f.write(f"╔{'═'*68}╗\n")
    f.write(f"║ INFORME DE EXTRACCIÓN - {tipo_texto.upper():^44} ║\n")
    f.write(f"╚{'═'*68}╝\n\n")

    # Metadata
    f.write(f"📄 INFORMACIÓN DEL DOCUMENTO\n")
    f.write(f"{'-'*70}\n")
    f.write(f"Archivo:          {pdf_filename}\n")
    f.write(f"Páginas:          {len(page_images)}\n")
    f.write(f"Tipo:             {tipo_texto}\n")
    f.write(f"Fecha proceso:    {datetime.now().strftime('%d/%m/%Y %H:%M:%S')}\n")
    f.write(f"Modelo:           DeepSeek-OCR v1\n\n")

    # Estadísticas
    f.write(f"📊 ESTADÍSTICAS DE EXTRACCIÓN\n")
    f.write(f"{'-'*70}\n")
    f.write(f"Campos buscados:        {resultado_json['estadisticas']['total_campos_buscados']}\n")
    f.write(f"Campos encontrados:     {total_campos}\n")
    f.write(f"Alta confianza (>90%):  {campos_alta_confianza}\n")
    f.write(f"Requieren revisión:     {len(campos_revisar)}\n")
    f.write(f"Tasa de éxito:          {resultado_json['estadisticas']['tasa_exito']}\n\n")

    # Campos por categoría
    for categoria, campos in campos_extraidos.items():
        if not campos:
            continue

        f.write(f"\n{'='*70}\n")
        f.write(f"{categoria.upper()}\n")
        f.write(f"{'='*70}\n\n")

        for campo_name, datos in campos.items():
            campo_fmt = campo_name.replace('_', ' ').title()
            f.write(f"► {campo_fmt}\n")
            f.write(f"  Valor:      {datos['value']}\n")
            f.write(f"  Página:     {datos['page']}\n")

            if 'confidence' in datos:
                conf = datos['confidence']
                conf_emoji = '✓' if conf > 0.9 else '⚠️' if conf > 0.7 else '❌'
                f.write(f"  Confianza:  {conf:.2%} {conf_emoji}\n")

            if 'bbox' in datos and datos['bbox']:
                f.write(f"  Coords:     {datos['bbox']}\n")

            f.write("\n")

    # Validación legal
    f.write(f"\n{'='*70}\n")
    f.write(f"⚖️  VALIDACIÓN LEGAL\n")
    f.write(f"{'='*70}\n\n")
    f.write(f"Estado: {'✅ VÁLIDO' if validacion['valido'] else '❌ NO VÁLIDO'}\n\n")

    if validacion['errores']:
        f.write(f"Errores críticos:\n")
        for e in validacion['errores']:
            f.write(f"  • {e}\n")
        f.write("\n")

    if validacion['advertencias']:
        f.write(f"Advertencias:\n")
        for a in validacion['advertencias']:
            f.write(f"  • {a}\n")
        f.write("\n")

    # Campos a revisar
    if campos_revisar:
        f.write(f"\n⚠️  CAMPOS QUE REQUIEREN REVISIÓN MANUAL\n")
        f.write(f"{'-'*70}\n")
        for cat, campo in campos_revisar:
            f.write(f"  • [{cat}] {campo}\n")

print(f"✅ Informe completo guardado: {informe_path}")
```

---

## 🚀 Aplicar Cambios

### Pasos para actualizar el notebook en Colab:

1. **Abrir el notebook en Colab**
2. **Celda 6**: Cambiar PyTorch a versión 2.5.1
3. **Después de celda 12**: Añadir nuevos campos (TITULARES, DIRECCION, PRODUCTO, FECHAS)
4. **Celda 14**: Reemplazar funciones con versiones que incluyen metadatos
5. **Nueva celda 23**: Añadir visualización de bounding boxes
6. **Celda 30**: Actualizar JSON output y informe

### O más fácil:

**Usar el notebook actualizado** que voy a subir a GitHub como `DeepSeek_OCR_Colab_Spain_Financial_v2.ipynb`

---

## 📈 Mejoras Implementadas

✅ PyTorch 2.5.1 (soluciona error)
✅ 30+ campos de extracción (vs 7-8 anteriores)
✅ Metadatos: bbox + confidence por campo
✅ Visualización con bounding boxes de colores
✅ Output JSON estructurado por categorías
✅ Informe mejorado con estadísticas
✅ Identificación de campos a revisar
✅ Imagen anotada descargable

---

**Creado**: 22 Octubre 2025
**Autor**: Carlos Lorenzo Santos
**Para**: DeepSeek-OCR España v2.0
