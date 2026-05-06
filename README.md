import streamlit as st
import datetime

# --- APP CONFIGURATION ---
st.set_page_config(page_title="Pescador Pro", page_icon="🎣", layout="centered")

# Custom CSS for High-Contrast "Sun Mode"
st.markdown("""
    <style>
    .stButton>button { width: 100%; border-radius: 10px; height: 3em; font-weight: bold; }
    .metric-container { background-color: #f0f2f6; padding: 15px; border-radius: 10px; border-left: 5px solid #1a472a; }
    </style>
    """, unsafe_allow_html=True)

st.title("🏆 Pescador Pro")
st.subheader("Control de Competición")

# --- SESSION STATE INITIALIZATION ---
if 'total_grams' not in st.session_state:
    st.session_state.total_grams = 0
if 'history' not in st.session_state:
    st.session_state.history = []

# --- SIDEBAR: MATCH SETUP ---
st.sidebar.header("⚙️ Configuración")
venue = st.sidebar.selectbox("Escenario", 
    ["Embalse de Orellana", "Río Ebro (Caspe)", "Río Júcar (Fortaleny)", 
     "Sierra Brava", "García de Sola", "Embalse de Arcos", "Otro"])
peg = st.sidebar.text_input("Puesto / Peg", value="1")
net_limit_kg = st.sidebar.number_input("Límite Rejón (kg)", value=20.0, step=1.0)

# --- TACTICS & BAIT SELECTION ---
with st.expander("🎣 Tácticas y Cebos Actuales"):
    col_t1, col_t2 = st.columns(2)
    with col_t1:
        method = st.radio("Método", ["Enchufable", "Inglesa", "Feeder", "Bolo", "Cuping"])
    with col_t2:
        bait = st.radio("Cebo", ["Asticot", "Maíz", "Pellet", "Lombriz", "Enguado"])

# --- MAIN INTERFACE: WEIGHT LOGGING ---
st.markdown("---")
st.markdown("### ➕ Registrar Captura")

def add_weight(g):
    st.session_state.total_grams += g
    st.session_state.history.append({
        "time": datetime.datetime.now().strftime("%H:%M"),
        "weight_g": g,
        "method": method,
        "bait": bait
    })

c1, c2, c3 = st.columns(3)
with c1:
    if st.button("🐟 +100g"): add_weight(100)
    if st.button("🐟 +250g"): add_weight(250)
with c2:
    if st.button("⚖️ +1.0kg"): add_weight(1000)
    if st.button("⚖️ +2.0kg"): add_weight(2000)
with c3:
    if st.button("🔥 +5.0kg"): add_weight(5000)
    if st.button("🧹 Reset", help="Borrar todo"): 
        st.session_state.total_grams = 0
        st.session_state.history = []

# --- CALCULATIONS & ALERTS ---
total_kg = st.session_state.total_grams / 1000
current_net_fill = total_kg % net_limit_kg

st.markdown("---")
col_res1, col_res2 = st.columns(2)
with col_res1:
    st.metric("PESO TOTAL", f"{total_kg:.3f} kg")
with col_res2:
    st.metric("REJÓN ACTUAL", f"{current_net_fill:.2f} kg")

# Spanish Net Limit Warnings[span_2](start_span)[span_2](end_span)[span_3](start_span)[span_3](end_span)
if current_net_fill >= (net_limit_kg - 0.5):
    st.error(f"⚠️ ¡REJÓN LLENO! Cambia de red ahora.")
elif current_net_fill >= (net_limit_kg - 2.0):
    st.warning(f"⚠️ ¡CUIDADO! Solo quedan {(net_limit_kg - current_net_fill):.2f}kg para el límite.")

# --- MATCH HISTORY ---
if st.checkbox("Ver Historial de Capturas"):
    if st.session_state.history:
        for item in reversed(st.session_state.history):
            st.write(f"⏱ {item['time']} | ⚖️ {item['weight_g']/1000:.3f}kg | 🎣 {item['method']} | 🪱 {item['bait']}")
    else:
        st.info("No hay capturas registradas aún.")

st.sidebar.markdown("---")
st.sidebar.info(f"App: Pescador Pro\nVenue: {venue}\nPeg: {peg}")
