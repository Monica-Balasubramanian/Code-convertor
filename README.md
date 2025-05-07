import streamlit as st
import google.generativeai as genai
import os
from dotenv import load_dotenv
import re

st.set_page_config(page_title="AI Code Converter", layout="wide")

def set_background(image_url):
    """Set a custom background image for the app."""
    st.markdown(
        f"""
        <style>
        .stApp {{
            background: url("{image_url}") no-repeat center center fixed;
            background-size: cover;
        }}
        </style>
        """,
        unsafe_allow_html=True
    )

set_background("data:image/jpeg;base64,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")

load_dotenv()
GOOGLE_API_KEY = os.getenv("GOOGLE_API_KEY")

if not GOOGLE_API_KEY:
    st.error("❌ API Key is missing! Add it to your .env file.")
else:
    genai.configure(api_key=GOOGLE_API_KEY)

languages = [
    "Python", "Java", "C++", "JavaScript", "C#", "Go", "Ruby", "Swift", "Kotlin", 
    "PHP", "Rust", "TypeScript", "R", "Scala", "Perl", "Dart", "Lua", "Haskell"
]

frameworks = {
    "Python": ["Flask", "Django", "FastAPI"],
    "JavaScript": ["Express.js", "Next.js", "NestJS"],
    "Java": ["Spring Boot", "Jakarta EE"],
    "PHP": ["Laravel", "CodeIgniter"],
    "Ruby": ["Ruby on Rails"],
    "C#": ["ASP.NET"],
}

def get_gemini_response(conversion_type, input_lang, output_lang, input_code, input_framework=None, output_framework=None):
    if not GOOGLE_API_KEY:
        return "❌ Missing API Key!"
    
    try:
        model = genai.GenerativeModel("gemini-1.5-pro")
        
        if conversion_type == "Code":
            prompt = f"Convert {input_lang} code to {output_lang}. Code:\n{input_code}"
        else:
            prompt = f"Convert the following code from {input_lang} ({input_framework}) framework to {output_lang} ({output_framework}) framework. Code:\n{input_code}"
        
        response = model.generate_content(prompt)

        
        code_match = re.search(r"```(?:[a-zA-Z]*)\n(.*?)```", response.text, re.DOTALL)
        return code_match.group(1).strip() if code_match else response.text.strip()
    
    except Exception as e:
        return f"⚠️ Error: {str(e)}"

st.markdown(
    """
    <style>
        .main { background-color: #dfe6e9; }
        .stButton>button { background-color: #fce4cc; color: black; width: 100%; font-size: 18px; }
        .stButton>button:hover { background-color: #f8a488; }
        h1 { color: #d79c7e; text-align: center; }
        .stSelectbox label, .stTextArea label { color: black; font-weight: bold; }
    </style>
    """,
    unsafe_allow_html=True
)

if "page" not in st.session_state:
    st.session_state.page = "home"

if st.session_state.page == "home":
    st.title("AI Code & Framework Converter")
    
    col1, col2 = st.columns(2)
    with col1:
        if st.button("🔄 Code Conversion"):
            st.session_state.page = "code_conversion"
            st.rerun()
    with col2:
        if st.button("⚙️ Framework Conversion"):
            st.session_state.page = "framework_conversion"
            st.rerun()

elif st.session_state.page == "code_conversion":
    st.title("🔄 AI Code Converter")
    
    col1, col2 = st.columns(2)
    with col1:
        input_lang = st.selectbox("Select Input Language:", languages, key="input_lang")
    with col2:
        output_lang = st.selectbox("Select Output Language:", languages, key="output_lang")

    input_code = st.text_area("Enter your code:", height=200, key="code_input")

    if st.button("🚀 Convert Code"):
        converted_code = get_gemini_response("Code", input_lang, output_lang, input_code)
        st.subheader("📝 Converted Code:")
        st.code(converted_code, language=output_lang.lower())

    if st.button("🔙 Back to Home"):
        st.session_state.page = "home"
        st.rerun()

elif st.session_state.page == "framework_conversion":
    st.title("⚙️ AI Framework Converter")
    
    col3, col4 = st.columns(2)
    with col3:
        input_lang_fw = st.selectbox("Select Input Language:", list(frameworks.keys()), key="input_fw_lang")
        input_framework = st.selectbox("Select Input Framework:", frameworks[input_lang_fw], key="input_fw")
    with col4:
        output_lang_fw = st.selectbox("Select Output Language:", list(frameworks.keys()), key="output_fw_lang")
        output_framework = st.selectbox("Select Output Framework:", frameworks[output_lang_fw], key="output_fw")

    fw_input_code = st.text_area("Enter your framework-specific code:", height=200, key="fw_code_input")

    if st.button("🚀 Convert Framework Code"):
        converted_fw_code = get_gemini_response("Framework", input_lang_fw, output_lang_fw, fw_input_code, input_framework, output_framework)
        st.subheader("📝 Converted Framework Code:")
        st.code(converted_fw_code, language=output_lang_fw.lower())


    if st.button("🔙 Back to Home"):
        st.session_state.page = "home"
        st.rerun()
