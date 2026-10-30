---
name: "intec-formato-academico"
description: "Formato estándar de los documentos académicos de Edgar Sánchez (INTEC): portada institucional con logo + informe normal a una columna. Úsalo SIEMPRE que pida crear un trabajo, informe, ensayo, reporte o tarea para la universidad en Word o PDF, aunque no mencione el formato."
---

# Formato académico INTEC — Edgar Sánchez

Formato **por defecto** para todo documento académico del usuario (INTEC, Santo Domingo).

## Regla principal

Hay dos modos. **Usa siempre `style="informe"` salvo que el usuario pida explícitamente un paper.**

| Modo | Cuándo | Cuerpo |
|---|---|---|
| `informe` (**por defecto**) | Informes, trabajos, tareas, ensayos, reportes | **Una columna**, Times New Roman 12pt, justificado, interlineado 1.5, sangría de primera línea 0.5". Párrafos normales que fluyen de lado a lado de la página. |
| `paper` | Solo si pide "paper", "artículo científico", "formato IEEE" o "dos columnas" | Dos columnas, 10pt, estilo IEEE/APA. |

La **portada es idéntica en ambos modos** y es la parte que al usuario más le gusta: no la cambies.

## Especificación

**Comunes**

| Elemento | Valor |
|---|---|
| Página | Carta 8.5×11" (12240×15840 twips), márgenes 1" (1440) en los 4 lados |
| Fuente | Times New Roman en todo el documento |
| Encabezado | Número de página alineado a la derecha (campo PAGE) |
| **Portada** | Centrada: asignatura 10.5pt bold (sz 21) · título 18pt bold (sz 36) en 1-2 líneas · logo INTEC ~1.15×1.29" · autores 11pt (sz 22) · rótulo "Docente" bold 10pt · "Fecha:" bold + fecha · salto de página al final |
| Tablas | Booktabs: sin bordes verticales ni internos · línea superior sz 12 · línea bajo el encabezado sz 8 · línea de cierre sz 12 · encabezado sombreado `E8E8E8` en bold · celdas alineadas a la izquierda · ancho total 9360 twips, centrada |
| Epígrafes | Centrados, etiqueta en negrita ("Tabla 1.", "Figura 1."), **debajo** del elemento |
| Referencias | H1 "Referencias" · sangría francesa 708 · interlineado sencillo · alineación izquierda · APA 7 en orden alfabético, títulos en itálica |

**Modo informe** — cuerpo 12pt (sz 24) · H1 14pt bold (sz 28) · H2 12pt bold (sz 24) · H3 12pt bold itálica · sangría 708 · interlineado 1.5 (line 360) · tablas y epígrafes 10pt (sz 20) · referencias 11pt (sz 22). Admite viñetas y listas numeradas.

**Modo paper** — cuerpo 10pt (sz 20) a dos columnas (`w:cols num="2" space="432"`) · H1 12pt bold · H2 11pt bold itálica · sangría 360 · tablas 8.5pt (sz 17) · epígrafes y referencias 9pt (sz 18) · tablas y figuras a ancho completo cerrando y reabriendo la sección de columnas · lleva bloque de título y resumen con palabras clave en la página 2.

## Estructura típica de un informe

Portada → (tabla de contenido, opcional) → Introducción → secciones de desarrollo con H1/H2 → Conclusiones → Referencias.

El resumen con palabras clave (`abstract()`) es propio del modo paper; en un informe normal solo inclúyelo si el usuario lo pide.

## Procedimiento

1. **Investiga primero** (WebSearch) y reúne el contenido y las fuentes antes de tocar el formato.
2. **Escribe `intec_doc.py`** con el código de más abajo, íntegro y sin modificar.
3. **Escribe `intec_logo.png`**: guarda el base64 final en `logo.b64` y decodifica con
   `python3 -c "import base64,pathlib; pathlib.Path('intec_logo.png').write_bytes(base64.b64decode(open('logo.b64').read()))"`
4. **Escribe el script de contenido** con la API de abajo.
5. **Convierte a PDF**: `soffice --headless --convert-to pdf archivo.docx --outdir .`
6. **Verifica visualmente**: renderiza las páginas con `pdf2image` y léelas con Read. Revisa portada, índice, tablas y referencias.
7. **Entrega el .docx y el .pdf** con `present_files`.

## Reglas de contenido

- Todo en español, registro académico formal, prosa continua y desarrollada.
- Las viñetas son válidas en modo informe para enumerar responsabilidades, criterios o pasos, pero el peso del documento debe estar en los párrafos. Las comparaciones y los inventarios van en tablas.
- Toda tabla lleva epígrafe y se menciona en el texto ("La Tabla 1 resume...").
- Referencias APA 7 alfabéticas con URL cuando exista; marca los títulos con `*asteriscos*` para la itálica.
- En `p()`, `bullets()` y `references()` funciona el marcado ligero `**negrita**` y `*itálica*`.

## Datos por defecto de portada

- Autor: `Edgar Sanchez, 1128305`
- Pregunta el código y nombre de la asignatura, el docente y la fecha de entrega si no los da.
- Formato del código: `ING-XXX###L-01 NOMBRE DE LA ASIGNATURA` en mayúsculas.

## API

```python
from intec_doc import IntecDoc, render_toc

d = IntecDoc(logo="intec_logo.png")               # informe (por defecto)
# d = IntecDoc(logo="intec_logo.png", style="paper")   # solo si lo pide

d.cover(course="ING-XXX202L-01 NOMBRE DE LA ASIGNATURA",
        title_lines=["Primera línea del título", "Segunda línea"],
        authors=["Edgar Sanchez, 1128305"],
        teachers=["Nombre Del Docente"],
        date="Jueves 13 de agosto 2026")
d.h1("Introducción");  d.p("Párrafo...")
d.h2("Subsección");    d.bullets(["**Punto.** Explicación...", "..."])
d.numbered(["Primero...", "Segundo..."])
d.table(headers=["A", "B"], rows=[["1", "2"]],
        caption_label="Tabla 1.", caption_text="Descripción. Elaboración propia.",
        col_widths=[3000, 6360], center_cols=(0,))
d.figure("grafico.png", "Figura 1.", "Descripción.", width_in=6.0)
d.h1("Conclusiones");  d.p("...")
d.references(["Autor, A. (2026). *Título*. https://ejemplo.com"])
d.save("Informe.docx")
```

**Tabla de contenido con paginación real** (opcional; los campos TOC de Word salen vacíos al exportar a PDF, por eso se usa un índice estático calculado en varias pasadas). Envuelve la construcción en una función y usa `render_toc`:

```python
def make(toc_entries=None):
    d = IntecDoc(logo="intec_logo.png")
    d.cover(...)
    if toc_entries is not None:
        d.toc(toc_entries)
    d.h1("Introducción"); d.p("...")
    ...
    d.references([...])
    return d

render_toc(make, "Informe.docx")   # requiere soffice y pdfplumber
```

Solo en modo `paper`, tras la portada añade `d.title_block(...)`, `d.abstract(texto, keywords)` y luego `d.start_two_col()`.

## Código de `intec_doc.py`

```python
# -*- coding: utf-8 -*-
"""Constructor autónomo de documentos académicos INTEC (modo informe / paper)."""
import os, re, zipfile
from xml.sax.saxutils import escape

NS = ('xmlns:wpc="http://schemas.microsoft.com/office/word/2010/wordprocessingCanvas" '
      'xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" '
      'xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships" '
      'xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math" '
      'xmlns:wp14="http://schemas.microsoft.com/office/word/2010/wordprocessingDrawing" '
      'xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing" '
      'xmlns:w10="urn:schemas-microsoft-com:office:word" '
      'xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main" '
      'xmlns:w14="http://schemas.microsoft.com/office/word/2010/wordml" '
      'xmlns:w15="http://schemas.microsoft.com/office/word/2012/wordml" '
      'xmlns:wpg="http://schemas.microsoft.com/office/word/2010/wordprocessingGroup" '
      'xmlns:wps="http://schemas.microsoft.com/office/word/2010/wordprocessingShape" '
      'mc:Ignorable="w14 w15 wp14"')

TWO_COL = '<w:cols w:num="2" w:space="432"/>'
ONE_COL = '<w:cols w:space="720"/>'

PRESETS = {
    "informe": dict(body=24, h1=28, h2=24, h3=24, indent=708, line=360,
                    table=20, caption=20, ref=22, two_col=False,
                    h2_italic=False, table_width=9360),
    "paper":   dict(body=20, h1=24, h2=22, h3=21, indent=360, line=360,
                    table=17, caption=18, ref=18, two_col=True,
                    h2_italic=True, table_width=9360),
}


def _esc(t):
    return escape(str(t))


def _runs(text, size=None, bold=False, italic=False):
    out = []
    for tk in re.split(r'(\*\*.+?\*\*|\*.+?\*)', text):
        if not tk:
            continue
        b, i = bold, italic
        if tk.startswith('**') and tk.endswith('**') and len(tk) > 4:
            tk, b = tk[2:-2], True
        elif tk.startswith('*') and tk.endswith('*') and len(tk) > 2:
            tk, i = tk[1:-1], True
        rpr = ('<w:b/><w:bCs/>' if b else '') + ('<w:i/><w:iCs/>' if i else '')
        if size:
            rpr += f'<w:sz w:val="{size}"/>'
        rpr = f'<w:rPr>{rpr}</w:rPr>' if rpr else ''
        out.append(f'<w:r>{rpr}<w:t xml:space="preserve">{_esc(tk)}</w:t></w:r>')
    return ''.join(out)


HEADER_XML = ('<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n'
    f'<w:hdr {NS}><w:p><w:pPr><w:jc w:val="right"/></w:pPr>'
    '<w:r><w:fldChar w:fldCharType="begin"/></w:r>'
    '<w:r><w:instrText>PAGE</w:instrText></w:r>'
    '<w:r><w:fldChar w:fldCharType="separate"/></w:r>'
    '<w:r><w:rPr><w:noProof/></w:rPr><w:t>1</w:t></w:r>'
    '<w:r><w:fldChar w:fldCharType="end"/></w:r></w:p></w:hdr>')

SETTINGS_XML = ('<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n'
    '<w:settings xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">'
    '<w:zoom w:percent="100"/><w:defaultTabStop w:val="720"/>'
    '<w:compat><w:compatSetting w:name="compatibilityMode" '
    'w:uri="http://schemas.microsoft.com/office/word" w:val="15"/></w:compat>'
    '</w:settings>')

NUMBERING_XML = ('<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n'
    '<w:numbering xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">'
    '<w:abstractNum w:abstractNumId="0"><w:multiLevelType w:val="hybridMultilevel"/>'
    '<w:lvl w:ilvl="0"><w:start w:val="1"/><w:numFmt w:val="bullet"/><w:lvlText w:val="&#8226;"/>'
    '<w:lvlJc w:val="left"/><w:pPr><w:ind w:left="720" w:hanging="360"/></w:pPr>'
    '<w:rPr><w:rFonts w:ascii="Symbol" w:hAnsi="Symbol" w:hint="default"/></w:rPr></w:lvl>'
    '</w:abstractNum>'
    '<w:abstractNum w:abstractNumId="1"><w:multiLevelType w:val="hybridMultilevel"/>'
    '<w:lvl w:ilvl="0"><w:start w:val="1"/><w:numFmt w:val="decimal"/><w:lvlText w:val="%1."/>'
    '<w:lvlJc w:val="left"/><w:pPr><w:ind w:left="720" w:hanging="360"/></w:pPr></w:lvl>'
    '</w:abstractNum>'
    '<w:num w:numId="1"><w:abstractNumId w:val="0"/></w:num>'
    '<w:num w:numId="2"><w:abstractNumId w:val="1"/></w:num>'
    '</w:numbering>')

RELS_ROOT = ('<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n'
    '<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">'
    '<Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/'
    'relationships/officeDocument" Target="word/document.xml"/></Relationships>')


def _styles_xml(P):
    it = '<w:i/><w:iCs/>' if P["h2_italic"] else ''
    return ('<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n'
        '<w:styles xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main">'
        '<w:docDefaults><w:rPrDefault><w:rPr>'
        '<w:rFonts w:ascii="Times New Roman" w:eastAsia="Times New Roman" '
        'w:hAnsi="Times New Roman" w:cs="Times New Roman"/>'
        f'<w:sz w:val="{P["body"]}"/><w:szCs w:val="{P["body"]}"/>'
        '<w:lang w:val="es-DO" w:eastAsia="en-US" w:bidi="ar-SA"/>'
        '</w:rPr></w:rPrDefault><w:pPrDefault/></w:docDefaults>'
        '<w:style w:type="paragraph" w:default="1" w:styleId="Normal"><w:name w:val="Normal"/>'
        '<w:qFormat/><w:pPr><w:spacing w:after="120" w:line="259" w:lineRule="auto"/>'
        '<w:jc w:val="both"/></w:pPr></w:style>'
        '<w:style w:type="paragraph" w:styleId="Heading1"><w:name w:val="heading 1"/>'
        '<w:qFormat/><w:pPr><w:keepNext/><w:spacing w:before="280" w:after="120"/>'
        '<w:outlineLvl w:val="0"/></w:pPr>'
        f'<w:rPr><w:b/><w:bCs/><w:sz w:val="{P["h1"]}"/><w:szCs w:val="{P["h1"]}"/></w:rPr></w:style>'
        '<w:style w:type="paragraph" w:styleId="Heading2"><w:name w:val="heading 2"/>'
        '<w:qFormat/><w:pPr><w:keepNext/><w:spacing w:before="200" w:after="80"/>'
        '<w:outlineLvl w:val="1"/></w:pPr>'
        f'<w:rPr><w:b/><w:bCs/>{it}<w:sz w:val="{P["h2"]}"/></w:rPr></w:style>'
        '<w:style w:type="paragraph" w:styleId="Heading3"><w:name w:val="heading 3"/>'
        '<w:qFormat/><w:pPr><w:keepNext/><w:spacing w:before="160" w:after="60"/>'
        '<w:outlineLvl w:val="2"/></w:pPr>'
        f'<w:rPr><w:b/><w:i/><w:iCs/><w:sz w:val="{P["h3"]}"/></w:rPr></w:style>'
        '<w:style w:type="character" w:default="1" w:styleId="DefaultParagraphFont">'
        '<w:name w:val="Default Paragraph Font"/></w:style>'
        '<w:style w:type="paragraph" w:styleId="ListParagraph"><w:name w:val="List Paragraph"/>'
        '<w:qFormat/><w:pPr><w:contextualSpacing/></w:pPr></w:style>'
        '<w:style w:type="table" w:default="1" w:styleId="TableNormal"><w:name w:val="Normal Table"/>'
        '<w:tblPr><w:tblInd w:w="0" w:type="dxa"/><w:tblCellMar>'
        '<w:top w:w="0" w:type="dxa"/><w:left w:w="108" w:type="dxa"/>'
        '<w:bottom w:w="0" w:type="dxa"/><w:right w:w="108" w:type="dxa"/>'
        '</w:tblCellMar></w:tblPr></w:style>'
        '<w:style w:type="numbering" w:default="1" w:styleId="NoList"><w:name w:val="No List"/>'
        '</w:style></w:styles>')


class IntecDoc:
    def __init__(self, logo=None, style="informe"):
        if style not in PRESETS:
            raise ValueError("style debe ser 'informe' o 'paper'")
        self.style = style
        self.P = PRESETS[style]
        self.body = []
        self.outline = []
        self.in_two_col = False
        self.images = []
        self.logo_rid = None
        if logo and os.path.exists(logo):
            with open(logo, 'rb') as fh:
                self.images.append(('image1.png', fh.read()))
            self.logo_rid = 'rIdImg1'

    def _sect(self, cols):
        return ('<w:sectPr><w:headerReference w:type="default" r:id="rIdHdr"/>'
                '<w:type w:val="continuous"/><w:pgSz w:w="12240" w:h="15840"/>'
                '<w:pgMar w:top="1440" w:right="1440" w:bottom="1440" w:left="1440" '
                'w:header="708" w:footer="708" w:gutter="0"/>'
                f'{cols}<w:docGrid w:linePitch="360"/></w:sectPr>')

    def _para(self, ppr, content):
        self.body.append(f'<w:p><w:pPr>{ppr}</w:pPr>{content}</w:p>')

    def _drawing(self, rid, cx, cy, pid):
        return (f'<w:r><w:rPr><w:noProof/></w:rPr><w:drawing>'
                f'<wp:inline distT="0" distB="0" distL="0" distR="0">'
                f'<wp:extent cx="{cx}" cy="{cy}"/>'
                f'<wp:effectExtent l="0" t="0" r="0" b="0"/>'
                f'<wp:docPr id="{pid}" name="Picture {pid}"/>'
                f'<wp:cNvGraphicFramePr><a:graphicFrameLocks '
                f'xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main" '
                f'noChangeAspect="1"/></wp:cNvGraphicFramePr>'
                f'<a:graphic xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main">'
                f'<a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/picture">'
                f'<pic:pic xmlns:pic="http://schemas.openxmlformats.org/drawingml/2006/picture">'
                f'<pic:nvPicPr><pic:cNvPr id="0" name=""/><pic:cNvPicPr>'
                f'<a:picLocks noChangeAspect="1" noChangeArrowheads="1"/></pic:cNvPicPr>'
                f'</pic:nvPicPr><pic:blipFill><a:blip r:embed="{rid}"/><a:srcRect/>'
                f'<a:stretch><a:fillRect/></a:stretch></pic:blipFill>'
                f'<pic:spPr bwMode="auto"><a:xfrm><a:off x="0" y="0"/>'
                f'<a:ext cx="{cx}" cy="{cy}"/></a:xfrm>'
                f'<a:prstGeom prst="rect"><a:avLst/></a:prstGeom></pic:spPr>'
                f'</pic:pic></a:graphicData></a:graphic></wp:inline></w:drawing></w:r>')

    def cover(self, course, title_lines, authors, teachers=(), date=None,
              teacher_label="Docente", page_break=True):
        self._para('<w:spacing w:before="280" w:after="60"/><w:jc w:val="center"/>',
                   _runs(course, size=21, bold=True))
        n = len(title_lines)
        for k, line in enumerate(title_lines):
            before = ' w:before="1200"' if k == 0 else ''
            after = '480' if k == n - 1 else '60'
            self._para(f'<w:spacing{before} w:after="{after}"/><w:jc w:val="center"/>',
                       f'<w:r><w:rPr><w:b/><w:bCs/><w:sz w:val="36"/><w:szCs w:val="30"/></w:rPr>'
                       f'<w:t xml:space="preserve">{_esc(line)}</w:t></w:r>')
        if self.logo_rid:
            self._para('<w:spacing w:before="120" w:after="160"/><w:jc w:val="center"/>',
                       self._drawing(self.logo_rid, 1104900, 1238250, 1))
        for a in authors:
            self._para('<w:spacing w:before="60" w:after="60"/><w:jc w:val="center"/>',
                       _runs(a, size=22))
        if teachers:
            self._para('<w:spacing w:before="360" w:after="60"/><w:jc w:val="center"/>',
                       '<w:r><w:rPr><w:b/><w:bCs/><w:sz w:val="20"/></w:rPr>'
                       f'<w:t>{_esc(teacher_label)}</w:t></w:r>')
            for t in teachers:
                self._para('<w:spacing w:before="60" w:after="60"/><w:jc w:val="center"/>',
                           _runs(t, size=22))
        if date:
            self._para('<w:spacing w:before="360" w:after="60"/><w:jc w:val="center"/>',
                       '<w:r><w:rPr><w:b/><w:bCs/><w:sz w:val="22"/></w:rPr>'
                       '<w:t xml:space="preserve">Fecha: </w:t></w:r>' + _runs(date, size=22))
        if page_break:
            self.body.append('<w:p><w:r><w:br w:type="page"/></w:r></w:p>')

    def toc(self, entries, title="Tabla de contenido", page_break=True):
        self._para('<w:spacing w:before="120" w:after="160"/>',
                   f'<w:r><w:rPr><w:b/><w:bCs/><w:sz w:val="{self.P["h1"]}"/></w:rPr>'
                   f'<w:t>{_esc(title)}</w:t></w:r>')
        for lvl, text, page in entries:
            left = 0 if lvl == 1 else 360
            bold = '<w:b/><w:bCs/>' if lvl == 1 else ''
            self._para(
                '<w:tabs><w:tab w:val="right" w:leader="dot" w:pos="9360"/></w:tabs>'
                '<w:spacing w:after="60" w:line="240" w:lineRule="auto"/>'
                f'<w:ind w:left="{left}" w:right="0"/><w:jc w:val="left"/>',
                f'<w:r><w:rPr>{bold}</w:rPr><w:t xml:space="preserve">{_esc(text)}</w:t></w:r>'
                f'<w:r><w:tab/><w:t>{page}</w:t></w:r>')
        if page_break:
            self.body.append('<w:p><w:r><w:br w:type="page"/></w:r></w:p>')

    def title_block(self, title, authors_line, affiliation):
        self._para('<w:spacing w:after="200"/><w:jc w:val="center"/>',
                   '<w:r><w:rPr><w:b/><w:bCs/><w:sz w:val="32"/><w:szCs w:val="28"/></w:rPr>'
                   f'<w:lastRenderedPageBreak/><w:t>{_esc(title)}</w:t></w:r>')
        self._para('<w:spacing w:after="40"/><w:jc w:val="center"/>',
                   f'<w:r><w:rPr><w:sz w:val="22"/><w:szCs w:val="22"/></w:rPr>'
                   f'<w:t>{_esc(authors_line)}</w:t></w:r>')
        self._para('<w:spacing w:after="320"/><w:jc w:val="center"/>',
                   f'<w:r><w:rPr><w:i/><w:iCs/><w:sz w:val="22"/></w:rPr>'
                   f'<w:t>{_esc(affiliation)}</w:t></w:r>')

    def abstract(self, text, keywords=None, label="Resumen", kw_label="Palabras clave: "):
        sz = self.P["body"] if self.style == "informe" else 20
        self._para('<w:spacing w:before="120" w:after="80"/>',
                   f'<w:r><w:rPr><w:b/><w:bCs/><w:sz w:val="{self.P["h2"]}"/></w:rPr>'
                   f'<w:t>{_esc(label)}</w:t></w:r>')
        self._para('<w:spacing w:line="276" w:lineRule="auto"/>',
                   f'<w:r><w:rPr><w:sz w:val="{sz}"/></w:rPr>'
                   f'<w:t xml:space="preserve">{_esc(text)}</w:t></w:r>')
        if keywords:
            self._para('<w:spacing w:after="280" w:line="276" w:lineRule="auto"/>',
                       f'<w:r><w:rPr><w:b/><w:bCs/><w:i/><w:iCs/><w:sz w:val="{sz}"/></w:rPr>'
                       f'<w:t xml:space="preserve">{_esc(kw_label)}</w:t></w:r>'
                       f'<w:r><w:rPr><w:i/><w:iCs/><w:sz w:val="{sz}"/></w:rPr>'
                       f'<w:t>{_esc(keywords)}</w:t></w:r>')

    def start_two_col(self):
        self.body.append(f'<w:p><w:pPr>{self._sect(ONE_COL)}</w:pPr></w:p>')
        self.in_two_col = True

    def h1(self, text):
        self.outline.append((1, text))
        self.body.append('<w:p><w:pPr><w:pStyle w:val="Heading1"/></w:pPr>'
                         f'<w:r><w:t>{_esc(text)}</w:t></w:r></w:p>')

    def h2(self, text):
        self.outline.append((2, text))
        self.body.append('<w:p><w:pPr><w:pStyle w:val="Heading2"/></w:pPr>'
                         f'<w:r><w:t>{_esc(text)}</w:t></w:r></w:p>')

    def h3(self, text):
        self.outline.append((3, text))
        self.body.append('<w:p><w:pPr><w:pStyle w:val="Heading3"/></w:pPr>'
                         f'<w:r><w:t>{_esc(text)}</w:t></w:r></w:p>')

    def p(self, text, indent=True):
        ppr = f'<w:spacing w:line="{self.P["line"]}" w:lineRule="auto"/>'
        if indent:
            ppr += f'<w:ind w:firstLine="{self.P["indent"]}"/>'
        self._para(ppr, _runs(text))

    def bullets(self, items, numbered=False):
        num_id = 2 if numbered else 1
        for it in items:
            self._para('<w:pStyle w:val="ListParagraph"/>'
                       f'<w:numPr><w:ilvl w:val="0"/><w:numId w:val="{num_id}"/></w:numPr>'
                       '<w:spacing w:after="60" w:line="276" w:lineRule="auto"/>'
                       '<w:ind w:left="720" w:hanging="360"/><w:jc w:val="both"/>',
                       _runs(it))

    def numbered(self, items):
        self.bullets(items, numbered=True)

    def caption(self, label, text, above=False):
        self._para(f'<w:spacing w:before="{"160" if above else "80"}" '
                   f'w:after="{"80" if above else "200"}"/><w:jc w:val="center"/>',
                   f'<w:r><w:rPr><w:b/><w:bCs/><w:sz w:val="{self.P["caption"]}"/></w:rPr>'
                   f'<w:t>{_esc(label)}</w:t></w:r>'
                   f'<w:r><w:rPr><w:sz w:val="{self.P["caption"]}"/></w:rPr>'
                   f'<w:t xml:space="preserve"> {_esc(text)}</w:t></w:r>')

    def table(self, headers, rows, caption_label=None, caption_text=None,
              col_widths=None, center_cols=(), caption_above=False):
        was_two = self.in_two_col
        if was_two:
            self.body.append(f'<w:p><w:pPr>{self._sect(TWO_COL)}</w:pPr></w:p>')
        if caption_above and caption_label:
            self.caption(caption_label, caption_text, above=True)
        tsz = self.P["table"]
        n = len(headers)
        if col_widths is None:
            col_widths = [self.P["table_width"] // n] * n
        total = sum(col_widths)
        x = ['<w:tbl><w:tblPr>'
             f'<w:tblW w:w="{total}" w:type="dxa"/><w:jc w:val="center"/>'
             '<w:tblBorders><w:top w:val="nil"/><w:left w:val="nil"/><w:bottom w:val="nil"/>'
             '<w:right w:val="nil"/><w:insideH w:val="nil"/><w:insideV w:val="nil"/></w:tblBorders>'
             '<w:tblCellMar><w:left w:w="10" w:type="dxa"/><w:right w:w="10" w:type="dxa"/>'
             '</w:tblCellMar><w:tblLook w:val="04A0" w:firstRow="1" w:lastRow="0" '
             'w:firstColumn="1" w:lastColumn="0" w:noHBand="0" w:noVBand="1"/></w:tblPr><w:tblGrid>']
        x += [f'<w:gridCol w:w="{w}"/>' for w in col_widths]
        x.append('</w:tblGrid>')
        x.append('<w:tr><w:trPr><w:tblHeader/><w:jc w:val="center"/></w:trPr>')
        for w, h in zip(col_widths, headers):
            x.append(f'<w:tc><w:tcPr><w:tcW w:w="{w}" w:type="dxa"/><w:tcBorders>'
                     '<w:top w:val="single" w:sz="12" w:space="0" w:color="000000"/>'
                     '<w:left w:val="nil"/>'
                     '<w:bottom w:val="single" w:sz="8" w:space="0" w:color="000000"/>'
                     '<w:right w:val="nil"/></w:tcBorders>'
                     '<w:shd w:val="clear" w:color="auto" w:fill="E8E8E8"/>'
                     '<w:tcMar><w:top w:w="55" w:type="dxa"/><w:left w:w="100" w:type="dxa"/>'
                     '<w:bottom w:w="55" w:type="dxa"/><w:right w:w="100" w:type="dxa"/></w:tcMar>'
                     '<w:vAlign w:val="center"/></w:tcPr>'
                     '<w:p><w:pPr><w:spacing w:line="264" w:lineRule="auto"/>'
                     '<w:jc w:val="center"/></w:pPr>'
                     f'<w:r><w:rPr><w:b/><w:bCs/><w:sz w:val="{tsz}"/></w:rPr>'
                     f'<w:t xml:space="preserve">{_esc(h)}</w:t></w:r></w:p></w:tc>')
        x.append('</w:tr>')
        for ri, row in enumerate(rows):
            bottom = ('<w:bottom w:val="single" w:sz="12" w:space="0" w:color="000000"/>'
                      if ri == len(rows) - 1 else '<w:bottom w:val="nil"/>')
            x.append('<w:tr><w:trPr><w:jc w:val="center"/></w:trPr>')
            for ci, (w, cell) in enumerate(zip(col_widths, row)):
                jc = ('<w:jc w:val="center"/>' if ci in center_cols
                      else '<w:jc w:val="left"/>')
                x.append(f'<w:tc><w:tcPr><w:tcW w:w="{w}" w:type="dxa"/><w:tcBorders>'
                         f'<w:top w:val="nil"/><w:left w:val="nil"/>{bottom}<w:right w:val="nil"/>'
                         '</w:tcBorders><w:tcMar><w:top w:w="55" w:type="dxa"/>'
                         '<w:left w:w="100" w:type="dxa"/><w:bottom w:w="55" w:type="dxa"/>'
                         '<w:right w:w="100" w:type="dxa"/></w:tcMar>'
                         '<w:vAlign w:val="center"/></w:tcPr>'
                         '<w:p><w:pPr><w:spacing w:before="40" w:after="40" w:line="264" '
                         f'w:lineRule="auto"/>{jc}</w:pPr>'
                         f'<w:r><w:rPr><w:sz w:val="{tsz}"/></w:rPr>'
                         f'<w:t xml:space="preserve">{_esc(cell)}</w:t></w:r></w:p></w:tc>')
            x.append('</w:tr>')
        x.append('</w:tbl>')
        self.body.append(''.join(x))
        if caption_label and not caption_above:
            self.caption(caption_label, caption_text)
        if was_two:
            self.body.append(f'<w:p><w:pPr>{self._sect(ONE_COL)}</w:pPr></w:p>')

    def figure(self, image_path, caption_label=None, caption_text=None,
               width_in=6.0, height_in=None):
        with open(image_path, 'rb') as fh:
            data = fh.read()
        idx = len(self.images) + 1
        self.images.append((f'image{idx}.png', data))
        rid = f'rIdImg{idx}'
        if height_in is None:
            try:
                from PIL import Image
                with Image.open(image_path) as im:
                    height_in = width_in * im.height / im.width
            except Exception:
                height_in = width_in * 0.6
        cx, cy = int(width_in * 914400), int(height_in * 914400)
        was_two = self.in_two_col
        if was_two:
            self.body.append(f'<w:p><w:pPr>{self._sect(TWO_COL)}</w:pPr></w:p>')
        self._para('<w:spacing w:before="120" w:after="20"/><w:jc w:val="center"/>',
                   self._drawing(rid, cx, cy, idx + 10))
        if caption_label:
            self.caption(caption_label, caption_text)
        if was_two:
            self.body.append(f'<w:p><w:pPr>{self._sect(ONE_COL)}</w:pPr></w:p>')

    def references(self, items, heading="Referencias"):
        self.h1(heading)
        for it in items:
            self._para('<w:spacing w:after="120" w:line="240" w:lineRule="auto"/>'
                       '<w:ind w:left="708" w:hanging="708"/><w:jc w:val="left"/>',
                       _runs(it, size=self.P["ref"]))

    def save(self, out_path):
        doc = (f'<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n'
               f'<w:document {NS}><w:body>' + ''.join(self.body)
               + self._sect(TWO_COL if self.in_two_col else ONE_COL)
               + '</w:body></w:document>')
        rels = ['<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n'
                '<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/'
                'relationships">'
                '<Relationship Id="rIdSty" Type="http://schemas.openxmlformats.org/'
                'officeDocument/2006/relationships/styles" Target="styles.xml"/>'
                '<Relationship Id="rIdSet" Type="http://schemas.openxmlformats.org/'
                'officeDocument/2006/relationships/settings" Target="settings.xml"/>'
                '<Relationship Id="rIdNum" Type="http://schemas.openxmlformats.org/'
                'officeDocument/2006/relationships/numbering" Target="numbering.xml"/>'
                '<Relationship Id="rIdHdr" Type="http://schemas.openxmlformats.org/'
                'officeDocument/2006/relationships/header" Target="header1.xml"/>']
        for i, (name, _) in enumerate(self.images, start=1):
            rels.append(f'<Relationship Id="rIdImg{i}" Type="http://schemas.openxmlformats.org/'
                        f'officeDocument/2006/relationships/image" Target="media/{name}"/>')
        rels.append('</Relationships>')
        ct = ('<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n'
              '<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types">'
              '<Default Extension="rels" ContentType="application/vnd.openxmlformats-package.'
              'relationships+xml"/>'
              '<Default Extension="xml" ContentType="application/xml"/>'
              '<Default Extension="png" ContentType="image/png"/>'
              '<Override PartName="/word/document.xml" ContentType="application/vnd.'
              'openxmlformats-officedocument.wordprocessingml.document.main+xml"/>'
              '<Override PartName="/word/styles.xml" ContentType="application/vnd.'
              'openxmlformats-officedocument.wordprocessingml.styles+xml"/>'
              '<Override PartName="/word/settings.xml" ContentType="application/vnd.'
              'openxmlformats-officedocument.wordprocessingml.settings+xml"/>'
              '<Override PartName="/word/numbering.xml" ContentType="application/vnd.'
              'openxmlformats-officedocument.wordprocessingml.numbering+xml"/>'
              '<Override PartName="/word/header1.xml" ContentType="application/vnd.'
              'openxmlformats-officedocument.wordprocessingml.header+xml"/>'
              '</Types>')
        with zipfile.ZipFile(out_path, 'w', zipfile.ZIP_DEFLATED) as z:
            z.writestr('[Content_Types].xml', ct)
            z.writestr('_rels/.rels', RELS_ROOT)
            z.writestr('word/document.xml', doc)
            z.writestr('word/_rels/document.xml.rels', ''.join(rels))
            z.writestr('word/styles.xml', _styles_xml(self.P))
            z.writestr('word/settings.xml', SETTINGS_XML)
            z.writestr('word/numbering.xml', NUMBERING_XML)
            z.writestr('word/header1.xml', HEADER_XML)
            for name, data in self.images:
                z.writestr(f'word/media/{name}', data)
        return out_path


def _norm(s):
    return re.sub(r'\s+', ' ', s).strip().lower()


def _locate_pages(pdf_path, outline, toc_title):
    import pdfplumber
    with pdfplumber.open(pdf_path) as pdf:
        texts = [_norm(p.extract_text() or '') for p in pdf.pages]
    start = 0
    for i, t in enumerate(texts):
        if _norm(toc_title) in t:
            start = i + 1
            break
    pages, cursor = [], start
    for _, txt in outline:
        needle = _norm(txt)
        found = None
        for i in range(cursor, len(texts)):
            if needle in texts[i]:
                found = i
                break
        if found is None:
            found = cursor
        pages.append(found + 1)
        cursor = found
    return pages


def render_toc(make, out_docx, levels=(1, 2), title="Tabla de contenido",
               tmpdir="/tmp", soffice="soffice"):
    """make(toc_entries) devuelve un IntecDoc; si toc_entries no es None el
    script debe llamar a d.toc(toc_entries) justo después de la portada."""
    import subprocess
    outline = [(l, t) for l, t in make(None).outline if l in levels]
    dummy = [(l, t, 0) for l, t in outline]
    tmp_docx = os.path.join(tmpdir, '_intec_toc_pass.docx')
    make(dummy).save(tmp_docx)
    subprocess.run([soffice, '--headless', '--convert-to', 'pdf', tmp_docx,
                    '--outdir', tmpdir], check=True, capture_output=True)
    pages = _locate_pages(tmp_docx[:-5] + '.pdf', outline, title)
    entries = [(l, t, p) for (l, t), p in zip(outline, pages)]
    make(entries).save(out_docx)
    return out_docx
```

## Logo INTEC (base64 PNG, 212×238)

Guarda esta cadena en `logo.b64` y decodifícala a `intec_logo.png`.

```
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
```

