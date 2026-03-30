import streamlit as st

# --- App Configuration ---
st.set_page_config(page_title="Veda: Women's Health AI", page_icon="🌸")

# --- Knowledge Base & Logic ---
KNOWLEDGE_BASE = {
    "Endometriosis": ["Chronic pelvic pain", "Heavy periods", "Pain during intercourse", "Painful bowel movements"],
    "PCOS": ["Irregular periods", "Excess hair growth", "Acne", "Weight gain", "Thinning hair"],
    "Perimenopause": ["Hot flashes", "Night sweats", "Mood swings", "Irregular cycles", "Insomnia"],
    "Puberty/Normal Cycle": ["First period", "Breast tenderness", "Mild cramping", "Mood changes"]
}

RED_FLAGS = ["Severe hemorrhage", "Fainting", "Sudden sharp localized pain", "High fever"]

# --- UI Layout ---
st.title("🌸 Veda Health Agent")
st.markdown("""
*Empowering women's health through AI-driven insights. 
Please note: This is an educational tool, not a medical diagnosis.*
""")

# 1. User Input Section
with st.sidebar:
    st.header("User Profile")
    name = st.text_input("Name", placeholder="Enter your name")
    age = st.slider("Age", 10, 60, 25)
    meds = st.text_area("Current Medications", placeholder="e.g., Vitamin D, Ibuprofen")

st.subheader(f"Hello {name if name else 'there'}, how are you feeling?")
selected_symptoms = st.multiselect(
    "Select any symptoms you are experiencing:",
    options=list(set([item for sublist in KNOWLEDGE_BASE.values() for item in sublist] + RED_FLAGS))
)

# 2. Processing Logic
if st.button("Analyze My Symptoms"):
    if not selected_symptoms:
        st.warning("Please select at least one symptom to begin the analysis.")
    else:
        # Check for Red Flags first
        critical = [s for s in selected_symptoms if s in RED_FLAGS]
        
        if critical:
            st.error(f"### 🚨 URGENT: MEDICAL CONSULTATION REQUIRED")
            st.write(f"You reported: **{', '.join(critical)}**. These symptoms require immediate professional attention. Please contact a doctor or visit an emergency room.")
        
        else:
            # Match symptoms to conditions
            matches = {}
            for condition, symptoms in KNOWLEDGE_BASE.items():
                score = len(set(selected_symptoms) & set(symptoms))
                if score > 0:
                    matches[condition] = score
            
            if matches:
                likely_condition = max(matches, key=matches.get)
                
                # Age-based filtering
                if age < 18 and likely_condition == "Perimenopause":
                    st.info("Your symptoms likely reflect the hormonal changes of **Puberty**. It's normal for cycles to be irregular early on!")
                elif age > 40 and likely_condition == "Puberty/Normal Cycle":
                    st.success("Your symptoms appear consistent with a **Normal Hormonal Cycle** or early Perimenopause.")
                else:
                    st.success(f"### Potential Insight: {likely_condition}")
                    st.write("Based on your input, these symptoms are often associated with this condition. We recommend tracking these patterns for 3 months.")
                
                # Suggestion Box
                st.info("**Next Step:** Keep a digital log of your symptoms and present them to your healthcare provider during your next check-up.")
            else:
                st.write("No specific patterns detected. If your symptoms persist or cause distress, please seek medical advice.")

# 3. Educational Footer
st.divider()
if age < 18:
    st.caption("💡 **Tip for Teens:** Your body is going through many changes. It's helpful to track your 'Day 1' (the first day of bleeding) every month.")
elif age > 40:
    st.caption("💡 **Tip for 40+:** Hormones begin to shift naturally during this time. Prioritizing sleep and calcium can help manage transitions.")
