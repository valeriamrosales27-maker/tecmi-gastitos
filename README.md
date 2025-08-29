import streamlit as st
import pandas as pd
import altair as alt
from datetime import date

# CONFIGURACIÓN DE LA PÁGINA INICIAL
st.set_page_config(page_title="Tecmi Gastitos / Gestor de gastos estudiantiles", page_icon="💸", layout="wide")
st.title("💸 Gestor de Gastos Estudiantiles / Tecmi Gastitos")
st.divider()

# --- BARRA MENU ---
with st.sidebar:
    st.header("Menú")
    page = st.radio(
        "Ir a:",
        ["Registro"],
        index=0
    )
    st.divider()
    st.caption("Como funciona `Tecmi Gastitos`")
    st.caption("1. Registra tus gastos en el formulario.")
    st.caption("2. Visualiza tu historial y resumen de gastos.")
    st.caption("3. Analiza tus gastos con gráficos interactivos.")
    st.divider()
    st.caption("2025 © Tecmi Gastitos")
    st.image("https://th.bing.com/th/id/R.badda37e3352445d7082b3daf8073387?rik=POjijyCsG0pd8A&riu=http%3a%2f%2fwww.pngall.com%2fwp-content%2fuploads%2f5%2fPug-PNG-Free-Image.png&ehk=hzC9MU9QqroZrNw1jOt9n3jdEvCbyKf3%2bAajYep6XWk%3d&risl=&pid=ImgRaw&r=0", width=150)

# --- INICIALIZAR DATAFRAME COMPONENTE STREAMLIT ---
if "gastos" not in st.session_state:
    st.session_state["gastos"] = pd.DataFrame(columns=["Fecha", "Categoría", "Monto", "Descripción"])

# --- FORMULARIO PARA REGISTRAR GASTOS ---
st.header("➕ Registrar nuevo gasto")

with st.form("form_gasto"):
    fecha = st.date_input("🗓️Fecha", value=date.today())
    categoria = st.selectbox("🔖Categoría", [
        "Alimentación", "Transporte", "Renta", "Colegiatura",
        "Libros y Material", "Salud", "Entretenimiento", "Servicios", "Otros"
    ])
    monto = st.number_input("💲Monto (MXN)", min_value=0.0, step=1.0)
    descripcion = st.text_input("📝Descripción (opcional)")
    
    submit = st.form_submit_button("Agregar gasto")

if submit:
    nuevo = pd.DataFrame(
        [[fecha, categoria, monto, descripcion]],
        columns=["Fecha", "Categoría", "Monto", "Descripción"]
    )
    st.session_state["gastos"] = pd.concat([st.session_state["gastos"], nuevo], ignore_index=True)
    st.success("✅ Gasto agregado correctamente")

# --- MOSTRAR TABLA DE GASTOS ---
st.header("📊 Historial de gastos")
st.dataframe(st.session_state["gastos"], use_container_width=True)

# --- TABLA BONITA: RESUMEN POR CATEGORÍA ---
if not st.session_state["gastos"].empty:
    st.header("📋 Resumen de gastos por categoría")
    resumen = st.session_state["gastos"].groupby("Categoría", as_index=False)["Monto"].sum()

    # Estilo bonito con pandas Styler
    styled_resumen = resumen.style.background_gradient(cmap="Blues").format({"Monto": "${:,.2f}"})
    st.write(styled_resumen)

# --- GRÁFICO DE GASTOS POR CATEGORÍA ---
if not st.session_state["gastos"].empty:
    st.header("📈 Gráfico de gastos por categoría")
    chart = alt.Chart(st.session_state["gastos"]).mark_bar().encode(
        x="Categoría",
        y="Monto",
        color="Categoría"
    )
    st.altair_chart(chart, use_container_width=True)



