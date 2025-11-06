import os
from zipfile import ZipFile
from textwrap import dedent

# === Crear estructura de carpetas ===
base = "Ford_Fiorasi_Procesador_Antecedentes"
os.makedirs(base, exist_ok=True)

# === Archivo: app_fiorasi_web.py ===
app_code = dedent('''\
import streamlit as st
import pandas as pd
import re
from datetime import datetime
from io import BytesIO
from docx import Document
from PyPDF2 import PdfReader

st.set_page_config(page_title="Ford Fiorasi – Procesador de Antecedentes", page_icon="🚗", layout="wide")

# Colores configurables (por defecto Ford)
if "color_principal" not in st.session_state:
    st.session_state.color_principal = "#0047AB"
    st.session_state.color_fondo = "#FFFFFF"

def extraer_texto_docx(file):
    doc = Document(file)
    return "\\n".join([p.text for p in doc.paragraphs])

def extraer_texto_pdf(file):
    texto = ""
    pdf = PdfReader(file)
    for page in pdf.pages:
        texto += page.extract_text() or ""
    return texto

def detectar_tipo(texto):
    t = texto.lower()
    if "llamado de atención" in t or "llamado de atencion" in t:
        return "Llamado de atención"
    elif "apercibimiento" in t:
        return "Apercibimiento"
    elif "solicitud de descargo" in t:
        return "Solicitud de descargo"
    elif "descargo" in t and "contest" in t:
        return "Contestación de descargo"
    else:
        return "No especificado"

def detectar_fecha(texto):
    match = re.search(r'(\\d{1,2}[/-]\\d{1,2}[/-]\\d{2,4})', texto)
    if match:
        for fmt in ("%d/%m/%Y", "%d-%m-%Y", "%d/%m/%y", "%d-%m-%y"):
            try:
                return datetime.strptime(match.group(1), fmt).strftime("%d/%m/%Y")
            except:
                continue
    return "No especificada"

def detectar_nombre(texto):
    match = re.search(r'(?:Sr\\.?|Sra\\.?|Empleado:)\\s*([A-ZÁÉÍÓÚÑ][A-Za-zÁÉÍÓÚÑ,\\s]+)', texto)
    if match:
        return re.sub(r"\\s{2,}", " ", match.group(1).strip())
    return "No identificado"

def detectar_contestacion(texto):
    t = texto.lower()
    return "Sí" if "descargo" in t or "contest" in t else "No"

def extraer_descripcion(texto):
    for linea in texto.split("\\n"):
        if len(linea.strip()) > 40 and not linea.lower().startswith(("sr", "sra", "ref", "asunto")):
            return linea.strip()
    return "No especificada"

# === Interfaz ===
col_logo, col_titulo = st.columns([1, 4])
with col_logo:
    st.image("logo_fiorasi.png", width=120)
with col_titulo:
    st.markdown(f"<h2 style='color:{st.session_state.color_principal};margin-top:10px;'>Ford Fiorasi – Procesador de Antecedentes Disciplinarios</h2>", unsafe_allow_html=True)

# Botón ajustes
with st.sidebar:
    st.header("⚙️ Ajustes de interfaz")
    st.session_state.color_principal = st.color_picker("Color principal", st.session_state.color_principal)
    st.session_state.color_fondo = st.color_picker("Color de fondo", st.session_state.color_fondo)
    if st.button("Restaurar colores Ford"):
        st.session_state.color_principal = "#0047AB"
        st.session_state.color_fondo = "#FFFFFF"

st.markdown(f"<div style='background-color:{st.session_state.color_fondo};padding:10px;border-radius:10px;'>", unsafe_allow_html=True)
st.write("Automatiza la lectura de documentos laborales (.docx / .pdf) para generar la base disciplinaria institucional.")
st.markdown("</div>", unsafe_allow_html=True)

archivos = st.file_uploader("📂 Seleccione los archivos Word o PDF", type=["pdf", "docx"], accept_multiple_files=True)

if st.button("Procesar antecedentes"):
    if not archivos:
        st.warning("Debe cargar al menos un archivo.")
    else:
        registros = []
        for archivo in archivos:
            texto = extraer_texto_docx(archivo) if archivo.name.endswith(".docx") else extraer_texto_pdf(archivo)
            registros.append({
                "Apellido y Nombre": detectar_nombre(texto),
                "Fecha de emisión": detectar_fecha(texto),
                "Tipo de antecedente": detectar_tipo(texto),
                "Descripción breve del hecho": extraer_descripcion(texto),
                "¿Hubo contestación?": detectar_contestacion(texto),
            })
        df = pd.DataFrame(registros).sort_values("Apellido y Nombre")
        resumen = []
        for nombre, grupo in df.groupby("Apellido y Nombre"):
            resumen.append({
                "Apellido y Nombre": nombre,
                "Cantidad de antecedentes": len(grupo),
                "Tipos recibidos": ", ".join(grupo["Tipo de antecedente"].unique()),
                "Última fecha registrada": grupo["Fecha de emisión"].iloc[-1],
                "Síntesis contextual": " / ".join(set(grupo["Descripción breve del hecho"]))
            })
        df_resumen = pd.DataFrame(resumen).sort_values("Apellido y Nombre")
        output = BytesIO()
        with pd.ExcelWriter(output, engine="openpyxl") as writer:
            df.to_excel(writer, sheet_name="Base de datos", index=False)
            df_resumen.to_excel(writer, sheet_name="Resumen por empleado", index=False)
        st.success("✅ Procesamiento completado.")
        st.download_button("Descargar Excel generado", data=output.getvalue(), file_name="FordFiorasi_Antecedentes.xlsx")
        st.dataframe(df_resumen)
''')

# === Archivo: requirements.txt ===
requirements = '''\
streamlit
pandas
python-docx
PyPDF2
openpyxl
pillow
'''

# === Archivo: README_INSTALACION.txt ===
readme = '''\
FORD FIORASI – PROCESADOR DE ANTECEDENTES DISCIPLINARIOS
--------------------------------------------------------

USO LOCAL:
1. Instalar dependencias:
   pip install -r requirements.txt
2. Ejecutar:
   streamlit run app_fiorasi_web.py

USO ONLINE (STREAMLIT CLOUD):
1. Crear un repositorio en GitHub llamado "ford-fiorasi-antecedentes".
2. Subir estos archivos.
3. Ir a https://streamlit.io/cloud, conectar tu cuenta GitHub y desplegar la app.
4. Listo: tendrás una URL tipo https://ford-fiorasi-antecedentes.streamlit.app
'''

# === Crear archivos ===
files = {
    "app_fiorasi_web.py": app_code,
    "requirements.txt": requirements,
    "README_INSTALACION.txt": readme,
}
for name, content in files.items():
    with open(os.path.join(base, name), "w", encoding="utf-8") as f:
        f.write(content)

# === Crear ZIP ===
zip_name = base + ".zip"
with ZipFile(zip_name, "w") as zipf:
    for filename in files:
        zipf.write(os.path.join(base, filename), arcname=os.path.join(base, filename))

print(f"✅ Proyecto generado correctamente: {zip_name}")
print("Podés subirlo directamente a Streamlit Cloud.")
