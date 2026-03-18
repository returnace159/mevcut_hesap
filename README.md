import streamlit as st

# Sayfa Genişliği (Yan yana kıyaslama için 'wide' yaptık)
st.set_page_config(page_title="Korkmaz Mühendislik Analiz", layout="wide")

st.title("🏗️ Korkmaz Çelik Analiz: Hatalı vs. Gerçek Hesap")
st.info("Sol tarafta iş yerindeki 'eksik' hesabı, sağ tarafta 'Gerçek' durumu göreceksiniz.")

# --- VERİTABANI (HEA Profilleri) ---
hea_data = {
    "HEA 100": [72.8, 16.7, 349.2],    "HEA 200": [388.6, 42.3, 3692],
    "HEA 300": [1260, 88.3, 18260],    "HEA 400": [2311, 125.0, 45070],
    "HEA 450": [2896, 140.0, 63720],   "HEA 500": [3550, 155.0, 86970],
    "HEA 550": [4146, 166.0, 111900],  "HEA 600": [4702, 178.0, 141200]
}

# --- YAN PANEL GİRDİLERİ ---
with st.sidebar:
    st.header("📊 Proje Verileri")
    L = st.number_input("Kiriş Boyu (Metre):", value=12.0)
    P_kg = st.number_input("Merkezi Yük (kg):", value=5000)
    limit_secim = st.selectbox("Sehim Sınırı:", [500, 900, 1000], index=1)
    st.divider()
    analiz_baslat = st.button("🚀 HESABI ÇARPIŞTIR", type="primary", use_container_width=True)

# --- MATEMATİKSEL MOTOR ---
def sehim_hesapla(profil, boy_m, yuk_kg, zati_olsun_mu=True):
    ix_mm4 = hea_data[profil][2] * 10000 
    L_mm = boy_m * 1000
    P_N = yuk_kg * 9.81
    E = 210000
    
    # Yükten gelen (P)
    s_p = (P_N * (L_mm**3)) / (48 * E * ix_mm4)
    
    # Zati ağırlıktan gelen (q)
    s_q = 0
    if zati_olsun_mu:
        g_kg_m = hea_data[profil][1]
        q_N_mm = (g_kg_m * 9.81) / 1000
        s_q = (5 * q_N_mm * (L_mm**4)) / (384 * E * ix_mm4)
        
    return s_p, s_q

# --- EKRAN TASARIMI ---
if analiz_baslat:
    profil_listesi = list(hea_data.keys())
    # Analiz için bir profil seçelim (örneğin HEA 400)
    test_profil = "HEA 400"
    limit_mm = (L * 1000) / limit_secim

    col1, col2 = st.columns(2)

    with col1:
        st.error("❌ İŞ YERİNDEKİ EKSİK HESAP")
        sp_h, sq_h = sehim_hesapla(test_profil, L, P_kg, zati_olsun_mu=False)
        st.metric("Hesaplanan Sehim", f"{sp_h:.2f} mm")
        st.write("🔴 Zati Ağırlık: **İHMAL EDİLDİ (0 kg)**")
        if sp_h < limit_mm:
            st.success(f"Kağıt Üstünde: UYGUN (Limit: {limit_mm:.1f} mm)")
        else:
            st.error("Kağıt Üstünde Bile: UYGUN DEĞİL")

    with col2:
        st.success("✅ KORKMAZ GERÇEK ANALİZ")
        sp_g, sq_g = sehim_hesapla(test_profil, L, P_kg, zati_olsun_mu=True)
        toplam_g = sp_g + sq_g
        st.metric("Gerçek Sehim", f"{toplam_g:.2f} mm")
        st.write(f"🟢 Zati Ağırlık Sehim Etkisi: **{sq_g:.2f} mm**")
        
        if toplam_g < limit_mm:
            st.success(f"Gerçekte: GÜVENLİ (Limit: {limit_mm:.1f} mm)")
        else:
            st.error(f"GERÇEKTE: TEHLİKELİ! (Sınır Aşıldı)")

    # --- ÖNERİ BÖLÜMÜ ---
    st.divider()
    st.subheader("🛠️ Korkmaz'ın Güvenli Profil Önerisi")
    
    for p_ad in profil_listesi:
        s1, s2 = sehim_hesapla(p_ad, L, P_kg, zati_olsun_mu=True)
        if (s1 + s2) < limit_mm:
            st.info(f"Mevcut hesap hatasını telafi eden en ekonomik profil: **{p_ad}**")
            break
else:
    st.warning("👈 Verileri girip butona bas kanka, kimin hesabı doğruymuş görelim.")
