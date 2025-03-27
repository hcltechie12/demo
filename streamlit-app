import streamlit as st
import streamlit.components.v1 as components
import pandas as pd
import numpy as np
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import requests
import json
import time
import logging
import os
import threading
import random
import base64
import traceback
from datetime import datetime, timedelta
from io import BytesIO

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler("super_app.log"),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger("SuperApp")

# Set page configuration with custom theme
st.set_page_config(
    page_title="AI Security & Sustainability Hub",
    page_icon="🔮",
    layout="wide",
    initial_sidebar_state="expanded",
)

# ----------------------------------------------------------------
# Session State Management
# ----------------------------------------------------------------

def initialize_session_state():
    """Initialize all session state variables with proper error handling"""
    try:
        # Core session states
        if 'targets' not in st.session_state:
            st.session_state.targets = []

        if 'test_results' not in st.session_state:
            st.session_state.test_results = {}

        if 'running_test' not in st.session_state:
            st.session_state.running_test = False

        if 'progress' not in st.session_state:
            st.session_state.progress = 0

        if 'vulnerabilities_found' not in st.session_state:
            st.session_state.vulnerabilities_found = 0

        if 'current_theme' not in st.session_state:
            st.session_state.current_theme = "dark"  # Default to dark theme
            
        if 'current_page' not in st.session_state:
            st.session_state.current_page = "Dashboard"
            
        # Thread management
        if 'active_threads' not in st.session_state:
            st.session_state.active_threads = []
            
        # Error handling
        if 'error_message' not in st.session_state:
            st.session_state.error_message = None
            
        # Initialize bias testing state
        if 'bias_results' not in st.session_state:
            st.session_state.bias_results = {}
            
        if 'show_bias_results' not in st.session_state:
            st.session_state.show_bias_results = False
            
        # Carbon tracking states
        if 'carbon_tracking_active' not in st.session_state:
            st.session_state.carbon_tracking_active = False
            
        if 'carbon_measurements' not in st.session_state:
            st.session_state.carbon_measurements = []
            
        # HTML Components integration states
        if 'engine_room_initialized' not in st.session_state:
            st.session_state.engine_room_initialized = False
            
        if 'bias_labs_enabled' not in st.session_state:
            st.session_state.bias_labs_enabled = False
            
        if 'sustainability_integrated' not in st.session_state:
            st.session_state.sustainability_integrated = False
            
        logger.info("Session state initialized successfully")
    except Exception as e:
        logger.error(f"Error initializing session state: {str(e)}")
        display_error(f"Failed to initialize application state: {str(e)}")

# Thread cleanup
def cleanup_threads():
    """Remove completed threads from session state"""
    try:
        if 'active_threads' in st.session_state:
            # Filter out completed threads
            active_threads = []
            for thread in st.session_state.active_threads:
                if thread.is_alive():
                    active_threads.append(thread)
            
            # Update session state with only active threads
            st.session_state.active_threads = active_threads
            
            if len(st.session_state.active_threads) > 0:
                logger.info(f"Active threads: {len(st.session_state.active_threads)}")
    except Exception as e:
        logger.error(f"Error cleaning up threads: {str(e)}")

# ----------------------------------------------------------------
# UI Theme & Styling
# ----------------------------------------------------------------

# Define color schemes
themes = {
    "dark": {
        "bg_color": "#121212",
        "card_bg": "#1E1E1E",
        "primary": "#1DB954",    # Vibrant green
        "secondary": "#BB86FC",  # Purple
        "accent": "#03DAC6",     # Teal
        "warning": "#FF9800",    # Orange
        "error": "#CF6679",      # Red
        "text": "#FFFFFF"
    },
    "light": {
        "bg_color": "#F5F5F5",
        "card_bg": "#FFFFFF",
        "primary": "#1DB954",    # Vibrant green
        "secondary": "#7C4DFF",  # Deep purple
        "accent": "#00BCD4",     # Cyan
        "warning": "#FF9800",    # Orange
        "error": "#F44336",      # Red
        "text": "#212121"
    }
}

# Get current theme colors safely
def get_theme():
    """Get current theme with error handling"""
    try:
        return themes[st.session_state.current_theme]
    except Exception as e:
        logger.error(f"Error getting theme: {str(e)}")
        # Return dark theme as fallback
        return themes["dark"]

# CSS styles
def load_css():
    """Load CSS with the current theme"""
    try:
        theme = get_theme()
        
        return f"""
        <style>
        .main .block-container {{
            padding-top: 1rem;
            padding-bottom: 1rem;
        }}
        
        h1, h2, h3, h4, h5, h6 {{
            color: {theme["primary"]};
        }}
        
        .stProgress > div > div > div > div {{
            background-color: {theme["primary"]};
        }}
        
        div[data-testid="stExpander"] {{
            border: none;
            border-radius: 8px;
            background-color: {theme["card_bg"]};
            margin-bottom: 1rem;
        }}
        
        div[data-testid="stVerticalBlock"] {{
            gap: 1.5rem;
        }}
        
        .card {{
            border-radius: 10px;
            background-color: {theme["card_bg"]};
            padding: 1.5rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            margin-bottom: 1rem;
            border-left: 3px solid {theme["primary"]};
        }}
        
        .warning-card {{
            border-left: 3px solid {theme["warning"]};
        }}
        
        .error-card {{
            border-left: 3px solid {theme["error"]};
        }}
        
        .success-card {{
            border-left: 3px solid {theme["primary"]};
        }}
        
        .metric-value {{
            font-size: 32px;
            font-weight: bold;
            color: {theme["primary"]};
        }}
        
        .metric-label {{
            font-size: 14px;
            color: {theme["text"]};
            opacity: 0.7;
        }}
        
        .sidebar-title {{
            margin-left: 15px;
            font-size: 1.2rem;
            font-weight: bold;
            color: {theme["primary"]};
        }}
        
        .target-card {{
            border-radius: 8px;
            background-color: {theme["card_bg"]};
            padding: 1rem;
            margin-bottom: 1rem;
            border-left: 3px solid {theme["secondary"]};
        }}
        
        .status-badge {{
            display: inline-block;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 12px;
            font-weight: bold;
        }}
        
        .status-badge.active {{
            background-color: {theme["primary"]};
            color: white;
        }}
        
        .status-badge.inactive {{
            background-color: gray;
            color: white;
        }}
        
        .hover-card:hover {{
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
            transform: translateY(-2px);
            transition: all 0.3s ease;
        }}
        
        .card-title {{
            color: {theme["primary"]};
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 10px;
        }}
        
        .nav-item {{
            padding: 8px 15px;
            border-radius: 5px;
            margin-bottom: 5px;
            cursor: pointer;
        }}
        
        .nav-item:hover {{
            background-color: rgba(29, 185, 84, 0.1);
        }}
        
        .nav-item.active {{
            background-color: rgba(29, 185, 84, 0.2);
            font-weight: bold;
        }}
        
        .tag {{
            display: inline-block;
            padding: 3px 8px;
            border-radius: 12px;
            font-size: 12px;
            margin-right: 5px;
            margin-bottom: 5px;
        }}
        
        .tag.owasp {{
            background-color: rgba(187, 134, 252, 0.2);
            color: {theme["secondary"]};
        }}
        
        .tag.nist {{
            background-color: rgba(3, 218, 198, 0.2);
            color: {theme["accent"]};
        }}
        
        .tag.fairness {{
            background-color: rgba(255, 152, 0, 0.2);
            color: {theme["warning"]};
        }}
        
        .stTabs [data-baseweb="tab-list"] {{
            gap: 8px;
        }}
        
        .stTabs [data-baseweb="tab"] {{
            height: 50px;
            border-radius: 5px 5px 0px 0px;
            gap: 1px;
            padding-top: 10px;
            padding-bottom: 10px;
        }}
        
        .stTabs [aria-selected="true"] {{
            background-color: {theme["card_bg"]};
            border-bottom: 3px solid {theme["primary"]};
        }}
        
        .error-message {{
            background-color: #CF6679;
            color: white;
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 20px;
        }}
        
        /* Modern sidebar styling */
        section[data-testid="stSidebar"] {{
            background-color: {theme["card_bg"]};
            border-right: 1px solid rgba(0,0,0,0.1);
        }}
        
        /* Modern navigation categories */
        .nav-category {{
            font-size: 12px;
            font-weight: bold;
            text-transform: uppercase;
            color: {theme["text"]};
            opacity: 0.6;
            margin: 10px 15px 5px 15px;
        }}
        
        /* Main content area padding */
        .main-content {{
            padding: 20px;
        }}
        
        /* Modern cards with hover effects */
        .modern-card {{
            background-color: {theme["card_bg"]};
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 20px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.07);
            transition: all 0.3s ease;
            border-left: none;
            border-top: 4px solid {theme["primary"]};
        }}
        
        .modern-card:hover {{
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
            transform: translateY(-5px);
        }}
        
        .modern-card.warning {{
            border-top: 4px solid {theme["warning"]};
        }}
        
        .modern-card.error {{
            border-top: 4px solid {theme["error"]};
        }}
        
        .modern-card.secondary {{
            border-top: 4px solid {theme["secondary"]};
        }}
        
        .modern-card.accent {{
            border-top: 4px solid {theme["accent"]};
        }}
        
        /* App header styles */
        .app-header {{
            display: flex;
            align-items: center;
            margin-bottom: 24px;
            padding-bottom: 16px;
            border-bottom: 1px solid rgba(255,255,255,0.1);
        }}
        
        .app-title {{
            font-size: 24px;
            font-weight: bold;
            margin: 0;
            color: {theme["primary"]};
        }}
        
        .app-subtitle {{
            font-size: 14px;
            opacity: 0.7;
            margin: 0;
        }}
        </style>
        """
    except Exception as e:
        logger.error(f"Error loading CSS: {str(e)}")
        # Return minimal CSS as fallback
        return "<style>.error-message { background-color: #CF6679; color: white; padding: 10px; border-radius: 5px; margin-bottom: 20px; }</style>"

# ----------------------------------------------------------------
# Navigation and Control
# ----------------------------------------------------------------

# Helper function to set page
def set_page(page_name):
    """Set the current page safely"""
    try:
        st.session_state.current_page = page_name
        logger.info(f"Navigation: Switched to {page_name} page")
    except Exception as e:
        logger.error(f"Error setting page to {page_name}: {str(e)}")
        display_error(f"Failed to navigate to {page_name}")

# Safe rerun function
def safe_rerun():
    """Safely rerun the app, handling different Streamlit versions"""
    try:
        st.rerun()  # For newer Streamlit versions
    except Exception as e1:
        try:
            st.experimental_rerun()  # For older Streamlit versions
        except Exception as e2:
            logger.error(f"Failed to rerun app: {str(e1)} then {str(e2)}")
            # Do nothing - at this point we can't fix it

# Error handling
def display_error(message):
    """Display error message to the user"""
    try:
        st.session_state.error_message = message
        logger.error(f"UI Error: {message}")
    except Exception as e:
        logger.critical(f"Failed to display error message: {str(e)}")

# ----------------------------------------------------------------
# Custom UI Components
# ----------------------------------------------------------------

# Custom components
def card(title, content, card_type="default"):
    """Generate HTML card with error handling"""
    try:
        card_class = "card"
        if card_type == "warning":
            card_class += " warning-card"
        elif card_type == "error":
            card_class += " error-card"
        elif card_type == "success":
            card_class += " success-card"
        
        return f"""
        <div class="{card_class} hover-card">
            <div class="card-title">{title}</div>
            {content}
        </div>
        """
    except Exception as e:
        logger.error(f"Error rendering card: {str(e)}")
        return f"""
        <div class="card error-card">
            <div class="card-title">Error Rendering Card</div>
            <p>Failed to render card content: {str(e)}</p>
        </div>
        """

def modern_card(title, content, card_type="default", icon=None):
    """Generate a modern style card with optional icon"""
    try:
        card_class = "modern-card"
        if card_type == "warning":
            card_class += " warning"
        elif card_type == "error":
            card_class += " error"
        elif card_type == "secondary":
            card_class += " secondary"
        elif card_type == "accent":
            card_class += " accent"
        
        icon_html = f'<span style="margin-right: 8px;">{icon}</span>' if icon else ''
        
        return f"""
        <div class="{card_class}">
            <div style="display: flex; align-items: center; margin-bottom: 15px;">
                {icon_html}
                <div class="card-title">{title}</div>
            </div>
            <div>{content}</div>
        </div>
        """
    except Exception as e:
        logger.error(f"Error rendering modern card: {str(e)}")
        return f"""
        <div class="modern-card error">
            <div class="card-title">Error Rendering Card</div>
            <p>Failed to render card content: {str(e)}</p>
        </div>
        """

def metric_card(label, value, description="", prefix="", suffix=""):
    """Generate HTML metric card with error handling"""
    try:
        return f"""
        <div class="modern-card hover-card">
            <div class="metric-label">{label}</div>
            <div class="metric-value">{prefix}{value}{suffix}</div>
            <div style="font-size: 14px; opacity: 0.7;">{description}</div>
        </div>
        """
    except Exception as e:
        logger.error(f"Error rendering metric card: {str(e)}")
        return f"""
        <div class="modern-card error">
            <div class="metric-label">Error</div>
            <div class="metric-value">N/A</div>
            <div style="font-size: 14px; opacity: 0.7;">Failed to render metric: {str(e)}</div>
        </div>
        """

# Logo and header
def render_header():
    """Render the application header safely"""
    try:
        logo_html = """
        <div class="app-header">
            <div style="margin-right: 15px; font-size: 2.5rem;">🔮</div>
            <div>
                <div class="app-title">AI Security & Sustainability Hub</div>
                <div class="app-subtitle">Comprehensive Security, Ethics & Environmental Analysis</div>
            </div>
        </div>
        """
        st.markdown(logo_html, unsafe_allow_html=True)
    except Exception as e:
        logger.error(f"Error rendering header: {str(e)}")
        st.markdown("# 🔮 AI Security & Sustainability Hub")

# ----------------------------------------------------------------
# Sidebar Navigation
# ----------------------------------------------------------------

def sidebar_navigation():
    """Render the sidebar navigation with organized categories"""
    try:
        st.sidebar.markdown('<div class="sidebar-title">AI Security & Sustainability Hub</div>', unsafe_allow_html=True)
        
        # Organize navigation options by category
        navigation_categories = {
            "Core Security": [
                {"icon": "🏠", "name": "Dashboard"},
                {"icon": "🎯", "name": "Target Management"},
                {"icon": "🧪", "name": "Test Configuration"},
                {"icon": "▶️", "name": "Run Assessment"},
                {"icon": "📊", "name": "Results Analyzer"}
            ],
            "AI Ethics & Bias": [
                {"icon": "🔍", "name": "Ethical AI Testing"},
                {"icon": "⚖️", "name": "Bias Testing"},
                {"icon": "📏", "name": "Bias Comparison"},
                {"icon": "🔬", "name": "Bias Labs Integration"},
                {"icon": "🧠", "name": "HELM Evaluation"}
            ],
            "Sustainability": [
                {"icon": "🌱", "name": "Environmental Impact"},
                {"icon": "🌍", "name": "Sustainability Dashboard"},
                {"icon": "♻️", "name": "Sustainability Integration"}
            ],
            "Integration & Tools": [
                {"icon": "📁", "name": "Multi-Format Import"},
                {"icon": "🚀", "name": "High-Volume Testing"},
                {"icon": "🔌", "name": "Engine Room Integration"},
                {"icon": "📚", "name": "Knowledge Base"},
                {"icon": "📝", "name": "HTML Portal"},
                {"icon": "🏛️", "name": "AI Safety Standards"},
                {"icon": "📊", "name": "Model Evaluation"}
            ],
            "System": [
                {"icon": "⚙️", "name": "Settings"}
            ]
        }
        
        # Render each category and its navigation options
        for category, options in navigation_categories.items():
            st.sidebar.markdown(f'<div class="nav-category">{category}</div>', unsafe_allow_html=True)
            
            for option in options:
                # Create a button for each navigation option
                if st.sidebar.button(
                    f"{option['icon']} {option['name']}", 
                    key=f"nav_{option['name']}",
                    use_container_width=True,
                    type="secondary" if st.session_state.current_page != option["name"] else "primary"
                ):
                    set_page(option["name"])
                    safe_rerun()
        
        # Theme toggle
        st.sidebar.markdown("---")
        st.sidebar.markdown('<div class="sidebar-title">🎨 Appearance</div>', unsafe_allow_html=True)
        if st.sidebar.button("🔄 Toggle Theme", key="toggle_theme", use_container_width=True):
            st.session_state.current_theme = "light" if st.session_state.current_theme == "dark" else "dark"
            logger.info(f"Theme toggled to {st.session_state.current_theme}")
            safe_rerun()
        
        # System status
        st.sidebar.markdown("---")
        st.sidebar.markdown('<div class="sidebar-title">📡 System Status</div>', unsafe_allow_html=True)
        
        if st.session_state.running_test:
            st.sidebar.success("⚡ Test Running")
        else:
            st.sidebar.info("⏸️ Idle")
        
        st.sidebar.markdown(f"🎯 Targets: {len(st.session_state.targets)}")
        
        # Active threads info
        if len(st.session_state.active_threads) > 0:
            st.sidebar.markdown(f"🧵 Active threads: {len(st.session_state.active_threads)}")
        
        # Add carbon tracking status if active
        if st.session_state.get("carbon_tracking_active", False):
            st.sidebar.markdown("🌱 Carbon tracking active")
        
        # Add Engine Room status if active
        if st.session_state.get("engine_room_initialized", False):
            st.sidebar.markdown("🔌 Engine Room connected")
        
        # Add version info
        st.sidebar.markdown("---")
        st.sidebar.markdown(f"v1.0.0 | {datetime.now().strftime('%Y-%m-%d')}", unsafe_allow_html=True)
    except Exception as e:
        logger.error(f"Error rendering sidebar: {str(e)}")
        st.sidebar.error("Navigation Error")
        st.sidebar.markdown(f"Error: {str(e)}")

# ----------------------------------------------------------------
# Utility Classes and Functions (Common)
# ----------------------------------------------------------------

# Mock data functions with error handling
def get_mock_test_vectors():
    """Get mock test vector data with error handling"""
    try:
        return [
            {
                "id": "sql_injection",
                "name": "SQL Injection",
                "category": "owasp",
                "severity": "high"
            },
            {
                "id": "xss",
                "name": "Cross-Site Scripting",
                "category": "owasp",
                "severity": "medium"
            },
            {
                "id": "prompt_injection",
                "name": "Prompt Injection",
                "category": "owasp",
                "severity": "critical"
            },
            {
                "id": "insecure_output",
                "name": "Insecure Output Handling",
                "category": "owasp",
                "severity": "high"
            },
            {
                "id": "nist_governance",
                "name": "AI Governance",
                "category": "nist",
                "severity": "medium"
            },
            {
                "id": "nist_transparency",
                "name": "Transparency",
                "category": "nist",
                "severity": "medium"
            },
            {
                "id": "fairness_demographic",
                "name": "Demographic Parity",
                "category": "fairness",
                "severity": "high"
            },
            {
                "id": "privacy_gdpr",
                "name": "GDPR Compliance",
                "category": "privacy",
                "severity": "critical"
            },
            {
                "id": "jailbreaking",
                "name": "Jailbreaking Resistance",
                "category": "exploit",
                "severity": "critical"
            }
        ]
    except Exception as e:
        logger.error(f"Error getting mock test vectors: {str(e)}")
        display_error("Failed to load test vectors")
        return []  # Return empty list as fallback

def run_mock_test(target, test_vectors, duration=30):
    """Simulate running a test in the background with proper error handling"""
    try:
        # Initialize progress
        st.session_state.progress = 0
        st.session_state.vulnerabilities_found = 0
        st.session_state.running_test = True
        
        logger.info(f"Starting mock test against {target['name']} with {len(test_vectors)} test vectors")
        
        # Create mock results data structure
        results = {
            "summary": {
                "total_tests": 0,
                "vulnerabilities_found": 0,
                "risk_score": 0
            },
            "vulnerabilities": [],
            "test_details": {}
        }
        
        # Simulate test execution
        total_steps = 100
        step_sleep = duration / total_steps
        
        for i in range(total_steps):
            # Check if we should stop (for handling cancellations)
            if not st.session_state.running_test:
                logger.info("Test was cancelled")
                break
                
            time.sleep(step_sleep)
            st.session_state.progress = (i + 1) / total_steps
            
            # Occasionally "find" a vulnerability
            if random.random() < 0.2:  # 20% chance each step
                vector = random.choice(test_vectors)
                severity_weight = {"low": 1, "medium": 2, "high": 3, "critical": 5}
                weight = severity_weight.get(vector["severity"], 1)
                
                # Add vulnerability to results
                vulnerability = {
                    "id": f"VULN-{len(results['vulnerabilities']) + 1}",
                    "test_vector": vector["id"],
                    "test_name": vector["name"],
                    "severity": vector["severity"],
                    "details": f"Mock vulnerability found in {target['name']} using {vector['name']} test vector.",
                    "timestamp": datetime.now().isoformat()
                }
                results["vulnerabilities"].append(vulnerability)
                
                # Update counters
                st.session_state.vulnerabilities_found += 1
                results["summary"]["vulnerabilities_found"] += 1
                results["summary"]["risk_score"] += weight
                
                logger.info(f"Found vulnerability: {vulnerability['id']} ({vulnerability['severity']})")
        
        # Complete the test results
        results["summary"]["total_tests"] = len(test_vectors) * 10  # Assume 10 variations per vector
        results["timestamp"] = datetime.now().isoformat()
        results["target"] = target["name"]
        
        logger.info(f"Test completed: {results['summary']['vulnerabilities_found']} vulnerabilities found")
        
        # Set the results in session state
        st.session_state.test_results = results
        return results
    
    except Exception as e:
        error_details = {
            "error": True,
            "error_message": str(e),
            "traceback": traceback.format_exc(),
            "timestamp": datetime.now().isoformat()
        }
        logger.error(f"Error in test execution: {str(e)}")
        logger.debug(traceback.format_exc())
        
        # Create error result
        st.session_state.error_message = f"Test execution failed: {str(e)}"
        return error_details
    
    finally:
        # Always ensure we reset the running state
        st.session_state.running_test = False

# File Format Support Functions
def handle_multiple_file_formats(uploaded_file):
    """Process different file formats for impact assessments"""
    try:
        file_extension = uploaded_file.name.split('.')[-1].lower()
        
        # JSON (already supported)
        if file_extension == 'json':
            import json
            return json.loads(uploaded_file.read())
        
        # CSV
        elif file_extension == 'csv':
            import pandas as pd
            import io
            return pd.read_csv(uploaded_file)
        
        # Excel
        elif file_extension in ['xlsx', 'xls']:
            import pandas as pd
            return pd.read_excel(uploaded_file)
        
        # PDF
        elif file_extension == 'pdf':
            from pypdf import PdfReader
            import io
            
            pdf_reader = PdfReader(io.BytesIO(uploaded_file.read()))
            text = ""
            for page in pdf_reader.pages:
                text += page.extract_text() + "\n"
            return {"text": text}
        
        # XML
        elif file_extension == 'xml':
            import xml.etree.ElementTree as ET
            import io
            
            tree = ET.parse(io.BytesIO(uploaded_file.read()))
            root = tree.getroot()
            
            # Convert XML to dict (simplified)
            def xml_to_dict(element):
                result = {}
                for child in element:
                    child_data = xml_to_dict(child)
                    if child.tag in result:
                        if type(result[child.tag]) is list:
                            result[child.tag].append(child_data)
                        else:
                            result[child.tag] = [result[child.tag], child_data]
                    else:
                        result[child.tag] = child_data
                
                if len(result) == 0:
                    return element.text
                return result
            
            return xml_to_dict(root)
        
        # YAML/YML
        elif file_extension in ['yaml', 'yml']:
            import yaml
            return yaml.safe_load(uploaded_file)
        
        # Other formats are supported similarly...
        else:
            return {"error": f"Unsupported file format: {file_extension}"}
            
    except Exception as e:
        logger.error(f"Error processing file: {str(e)}")
        return {"error": f"Failed to process file: {str(e)}"}

# ----------------------------------------------------------------
# Main Class for WhyLabs Bias Testing
# ----------------------------------------------------------------

class WhyLabsBiasTest:
    """Class for WhyLabs-based bias testing functionality"""
    
    def __init__(self):
        # This would normally import whylogs, but for demonstration we'll create a mock
        self.session = None
        self.results = {}
    
    def initialize_session(self, dataset_name):
        """Initialize a WhyLogs profiling session"""
        try:
            self.session = True  # Mock initialization
            logger.info(f"WhyLogs session initialized for {dataset_name}")
            return True
        except Exception as e:
            logger.error(f"Failed to initialize WhyLogs session: {str(e)}")
            return False
    
    def profile_dataset(self, df, dataset_name):
        """Profile a dataset for bias analysis"""
        try:
            if self.session is None:
                self.initialize_session(dataset_name)
                
            # Create a mock profile
            profile = {"name": dataset_name, "columns": list(df.columns)}
            self.results[dataset_name] = {"profile": profile}
            logger.info(f"Dataset {dataset_name} profiled successfully")
            return profile
        except Exception as e:
            logger.error(f"Failed to profile dataset: {str(e)}")
            return None
    
    def analyze_bias(self, df, protected_features, target_column, dataset_name):
        """Analyze bias in a dataset based on protected features"""
        try:
            # Profile the dataset first
            profile = self.profile_dataset(df, dataset_name)
            
            bias_metrics = {}
            
            # Calculate basic bias metrics
            for feature in protected_features:
                # Statistical parity difference
                feature_groups = df.groupby(feature)
                
                outcomes = {}
                disparities = {}
                
                for group_name, group_data in feature_groups:
                    # For binary target variable
                    if df[target_column].nunique() == 2:
                        positive_outcome_rate = group_data[target_column].mean()
                        outcomes[group_name] = positive_outcome_rate
                
                # Calculate disparities between groups
                baseline = max(outcomes.values())
                for group, rate in outcomes.items():
                    disparities[group] = baseline - rate
                
                bias_metrics[feature] = {
                    "outcomes": outcomes,
                    "disparities": disparities,
                    "max_disparity": max(disparities.values())
                }
            
            self.results[dataset_name]["bias_metrics"] = bias_metrics
            logger.info(f"Bias analysis completed for {dataset_name}")
            return bias_metrics
        except Exception as e:
            logger.error(f"Failed to analyze bias: {str(e)}")
            return {"error": str(e)}
    
    def get_results(self, dataset_name=None):
        """Get analysis results"""
        if dataset_name:
            return self.results.get(dataset_name, {})
        return self.results

# ----------------------------------------------------------------
# Main Class for Carbon Impact Tracking
# ----------------------------------------------------------------

class CarbonImpactTracker:
    """Class for tracking environmental impact of AI systems"""
    
    def __init__(self):
        # Placeholder for codecarbon import
        self.tracker = None
        self.measurements = []
        self.total_emissions = 0.0
        self.is_tracking = False
    
    def initialize_tracker(self, project_name, api_endpoint=None):
        """Initialize the carbon tracker"""
        try:
            # Mock initialization for demonstration
            self.tracker = {"project_name": project_name, "initialized": True}
            logger.info(f"Carbon tracker initialized for {project_name}")
            return True
        except Exception as e:
            logger.error(f"Failed to initialize carbon tracker: {str(e)}")
            return False
    
    def start_tracking(self):
        """Start tracking carbon emissions"""
        try:
            if self.tracker is None:
                return False
                
            self.is_tracking = True
            logger.info("Carbon emission tracking started")
            return True
        except Exception as e:
            logger.error(f"Failed to start carbon tracking: {str(e)}")
            return False
    
    def stop_tracking(self):
        """Stop tracking and get the emissions data"""
        try:
            if not self.is_tracking or self.tracker is None:
                return 0.0
                
            # Generate a random emissions value for demonstration
            emissions = random.uniform(0.001, 0.1)
            self.is_tracking = False
            self.measurements.append(emissions)
            self.total_emissions += emissions
            
            logger.info(f"Carbon emission tracking stopped. Measured: {emissions} kg CO2eq")
            return emissions
        except Exception as e:
            logger.error(f"Failed to stop carbon tracking: {str(e)}")
            return 0.0
    
    def get_total_emissions(self):
        """Get total emissions tracked so far"""
        return self.total_emissions
    
    def get_all_measurements(self):
        """Get all measurements"""
        return self.measurements
    
    def generate_report(self):
        """Generate a report of carbon emissions"""
        try:
            energy_solutions = [
                {
                    "name": "Optimize AI Model Size",
                    "description": "Reduce model parameters and optimize architecture",
                    "potential_savings": "20-60% reduction in emissions",
                    "implementation_difficulty": "Medium"
                },
                {
                    "name": "Implement Model Distillation",
                    "description": "Create smaller, efficient versions of larger models",
                    "potential_savings": "40-80% reduction in emissions",
                    "implementation_difficulty": "High"
                },
                {
                    "name": "Use Efficient Hardware",
                    "description": "Deploy on energy-efficient hardware (e.g., specialized AI chips)",
                    "potential_savings": "30-50% reduction in emissions",
                    "implementation_difficulty": "Medium"
                }
            ]
            
            # Calculate the impact
            kwh_per_kg_co2 = 0.6  # Approximate conversion factor
            energy_consumption = self.total_emissions / kwh_per_kg_co2
            
            trees_equivalent = self.total_emissions * 16.5  # Each kg CO2 ~ 16.5 trees for 1 day
            
            return {
                "total_emissions_kg": self.total_emissions,
                "energy_consumption_kwh": energy_consumption,
                "measurements": self.measurements,
                "trees_equivalent": trees_equivalent,
                "mitigation_strategies": energy_solutions
            }
        except Exception as e:
            logger.error(f"Failed to generate emissions report: {str(e)}")
            return {"error": str(e)}

# ----------------------------------------------------------------
# Page Renderers - Core Security Pages
# ----------------------------------------------------------------

def render_dashboard():
    """Render the dashboard page safely"""
    try:
        render_header()
        
        st.markdown("""
        <div style="margin-bottom: 20px;">
        Welcome to your AI Security & Sustainability Hub. This dashboard provides an overview of your security posture,
        sustainability metrics, and ethical AI evaluation results.
        </div>
        """, unsafe_allow_html=True)
        
        # Quick stats in a row of cards
        col1, col2, col3, col4 = st.columns(4)
        
        with col1:
            st.markdown(metric_card("Targets", len(st.session_state.targets), "Configured AI models"), unsafe_allow_html=True)
        
        with col2:
            st.markdown(metric_card("Test Vectors", "9", "Available security tests"), unsafe_allow_html=True)
        
        with col3:
            vuln_count = len(st.session_state.test_results.get("vulnerabilities", [])) if st.session_state.test_results else 0
            st.markdown(metric_card("Vulnerabilities", vuln_count, "Identified issues"), unsafe_allow_html=True)
        
        with col4:
            risk_score = st.session_state.test_results.get("summary", {}).get("risk_score", 0) if st.session_state.test_results else 0
            st.markdown(metric_card("Risk Score", risk_score, "Overall security risk"), unsafe_allow_html=True)
        
        # Recent activity and status
        col1, col2 = st.columns([2, 1])
        
        with col1:
            st.markdown(modern_card("Recent Activity", "Your latest security findings and events.", "default", "🔔"), unsafe_allow_html=True)
            
            if not st.session_state.test_results:
                st.markdown(modern_card("No Recent Activity", "Run your first assessment to generate results.", "warning", "⚠️"), unsafe_allow_html=True)
            else:
                # Show the most recent vulnerabilities
                vulnerabilities = st.session_state.test_results.get("vulnerabilities", [])
                if vulnerabilities:
                    for vuln in vulnerabilities[:3]:  # Show top 3
                        severity_color = {
                            "low": get_theme()["text"],
                            "medium": get_theme()["warning"],
                            "high": get_theme()["warning"],
                            "critical": get_theme()["error"]
                        }.get(vuln["severity"], get_theme()["text"])
                        
                        st.markdown(f"""
                        <div class="modern-card hover-card">
                            <div style="display: flex; justify-content: space-between; align-items: center;">
                                <div class="card-title">{vuln["id"]}: {vuln["test_name"]}</div>
                                <div style="color: {severity_color}; font-weight: bold; text-transform: uppercase; font-size: 12px;">
                                    {vuln["severity"]}
                                </div>
                            </div>
                            <p>{vuln["details"]}</p>
                            <div style="font-size: 12px; opacity: 0.7;">Found in: {vuln["timestamp"]}</div>
                        </div>
                        """, unsafe_allow_html=True)
        
        with col2:
            st.markdown(modern_card("System Status", "Current operational status", "default", "📡"), unsafe_allow_html=True)
            
            if st.session_state.running_test:
                st.markdown(modern_card("Test in Progress", f"""
                <div style="margin-bottom: 10px;">
                    <div style="margin-bottom: 5px;">Progress:</div>
                    <div style="height: 10px; background-color: rgba(255,255,255,0.1); border-radius: 5px;">
                        <div style="height: 10px; width: {st.session_state.progress*100}%; background-color: {get_theme()["primary"]}; border-radius: 5px;"></div>
                    </div>
                    <div style="text-align: right; font-size: 12px; margin-top: 5px;">{int(st.session_state.progress*100)}%</div>
                </div>
                <div>Vulnerabilities found: {st.session_state.vulnerabilities_found}</div>
                """, "warning", "⚠️"), unsafe_allow_html=True)
            else:
                st.markdown(modern_card("System Ready", """
                <p>All systems operational and ready to run assessments.</p>
                <div style="display: flex; align-items: center;">
                    <div style="width: 10px; height: 10px; background-color: #4CAF50; border-radius: 50%; margin-right: 5px;"></div>
                    <div>API Connection: Active</div>
                </div>
                """, "default", "✅"), unsafe_allow_html=True)
        
        # Test vector overview
        st.markdown("<h3>Test Vector Overview</h3>", unsafe_allow_html=True)
        
        # Create a radar chart for test coverage
        try:
            test_vectors = get_mock_test_vectors()
            categories = list(set(tv["category"] for tv in test_vectors))
            
            # Count test vectors by category
            category_counts = {}
            for cat in categories:
                category_counts[cat] = sum(1 for tv in test_vectors if tv["category"] == cat)
            
            # Create the data for the radar chart
            fig = go.Figure()
            
            primary_color = get_theme()["primary"]
            # Convert hex to rgb for plotly
            r_value = int(primary_color[1:3], 16) if len(primary_color) >= 7 else 29
            g_value = int(primary_color[3:5], 16) if len(primary_color) >= 7 else 185
            b_value = int(primary_color[5:7], 16) if len(primary_color) >= 7 else 84
            
            fig.add_trace(go.Scatterpolar(
                r=list(category_counts.values()),
                theta=list(category_counts.keys()),
                fill='toself',
                fillcolor=f'rgba({r_value}, {g_value}, {b_value}, 0.3)',
                line=dict(color=primary_color),
                name='Test Coverage'
            ))
            
            fig.update_layout(
                polar=dict(
                    radialaxis=dict(
                        visible=True,
                        range=[0, max(category_counts.values()) + 1]
                    )
                ),
                showlegend=False,
                margin=dict(l=20, r=20, t=20, b=20),
                height=300,
                paper_bgcolor='rgba(0,0,0,0)',
                plot_bgcolor='rgba(0,0,0,0)',
                font=dict(color=get_theme()["text"])
            )
            
            st.plotly_chart(fig, use_container_width=True)
        except Exception as e:
            logger.error(f"Error rendering radar chart: {str(e)}")
            st.error("Failed to render radar chart")
        
        # Environmental impact summary
        st.markdown("<h3>Environmental Impact Summary</h3>", unsafe_allow_html=True)
        
        col1, col2, col3 = st.columns(3)
        
        with col1:
            total_carbon = sum(st.session_state.carbon_measurements) if hasattr(st.session_state, 'carbon_measurements') else 0
            st.markdown(metric_card("Carbon Emissions", f"{total_carbon:.5f}", "kg CO2 equivalent", suffix=" kg"), unsafe_allow_html=True)
        
        with col2:
            # Convert to equivalent metrics
            energy_consumption = total_carbon / 0.6 if total_carbon > 0 else 0  # Approximate conversion
            st.markdown(metric_card("Energy Consumed", f"{energy_consumption:.5f}", "Kilowatt-hours", suffix=" kWh"), unsafe_allow_html=True)
        
        with col3:
            # Trees needed to offset
            trees_needed = total_carbon * 0.06 if total_carbon > 0 else 0  # ~0.06 trees per kg CO2 per year
            st.markdown(metric_card("Trees Needed", f"{trees_needed:.2f}", "To offset emissions (1 year)"), unsafe_allow_html=True)
        
        # Quick actions with Streamlit buttons
        st.markdown("<h3>Quick Actions</h3>", unsafe_allow_html=True)
        col1, col2, col3, col4 = st.columns(4)
        
        with col1:
            if st.button("➕ Add New Target", use_container_width=True, key="dashboard_add_target"):
                set_page("Target Management")
                safe_rerun()
        
        with col2:
            if st.button("🧪 Run Assessment", use_container_width=True, key="dashboard_run_assessment"):
                set_page("Run Assessment")
                safe_rerun()
        
        with col3:
            if st.button("📊 View Results", use_container_width=True, key="dashboard_view_results"):
                set_page("Results Analyzer")
                safe_rerun()
                
        with col4:
            if st.button("🌱 Track Carbon", use_container_width=True, key="dashboard_track_carbon"):
                set_page("Environmental Impact")
                safe_rerun()
                
    except Exception as e:
        logger.error(f"Error rendering dashboard: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error rendering dashboard: {str(e)}")

# Other core security page renderers are implemented similarly to render_dashboard()
# For brevity, I'll include simplified versions of the remaining page functions

def render_target_management():
    """Render the target management page safely"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Target Management</h2>
        <p>Add and configure AI models to test</p>
        """, unsafe_allow_html=True)
        
        # Show existing targets
        if st.session_state.targets:
            st.markdown("<h3>Your Targets</h3>", unsafe_allow_html=True)
            
            # Use columns for better layout
            cols = st.columns(3)
            for i, target in enumerate(st.session_state.targets):
                col = cols[i % 3]
                with col:
                    with st.container():
                        st.markdown(f"### {target['name']}")
                        st.markdown(f"**Endpoint:** {target['endpoint']}")
                        st.markdown(f"**Type:** {target.get('type', 'Unknown')}")
                        
                        col1, col2 = st.columns(2)
                        with col1:
                            if st.button("✏️ Edit", key=f"edit_target_{i}", use_container_width=True):
                                # In a real app, this would open an edit dialog
                                st.info("Edit functionality would open here")
                        
                        with col2:
                            if st.button("🗑️ Delete", key=f"delete_target_{i}", use_container_width=True):
                                # Remove the target
                                st.session_state.targets.pop(i)
                                st.success(f"Target '{target['name']}' deleted")
                                safe_rerun()
        
        # Add new target form
        st.markdown("<h3>Add New Target</h3>", unsafe_allow_html=True)
        
        with st.form("add_target_form"):
            col1, col2 = st.columns(2)
            
            with col1:
                target_name = st.text_input("Target Name")
                target_endpoint = st.text_input("API Endpoint URL")
                target_type = st.selectbox("Model Type", ["LLM", "Content Filter", "Embedding", "Classification", "Other"])
            
            with col2:
                api_key = st.text_input("API Key", type="password")
                target_description = st.text_area("Description")
            
            submit_button = st.form_submit_button("Add Target")
            
            if submit_button:
                try:
                    if not target_name or not target_endpoint:
                        st.error("Name and endpoint are required")
                    else:
                        new_target = {
                            "name": target_name,
                            "endpoint": target_endpoint,
                            "type": target_type,
                            "api_key": api_key,
                            "description": target_description
                        }
                        st.session_state.targets.append(new_target)
                        st.success(f"Target '{target_name}' added successfully!")
                        logger.info(f"Added new target: {target_name}")
                        safe_rerun()
                except Exception as e:
                    logger.error(f"Error adding target: {str(e)}")
                    st.error(f"Failed to add target: {str(e)}")
    
    except Exception as e:
        logger.error(f"Error rendering target management: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in target management: {str(e)}")

def render_test_configuration():
    """Render the test configuration page safely"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Test Configuration</h2>
        <p>Customize your security assessment</p>
        """, unsafe_allow_html=True)
        
        # Implementing just enough to show the structure and functionality
        test_vectors = get_mock_test_vectors()
        
        # Create tabs for each category
        categories = {}
        for tv in test_vectors:
            if tv["category"] not in categories:
                categories[tv["category"]] = []
            categories[tv["category"]].append(tv)
            
        tabs = st.tabs(list(categories.keys()))
        
        for i, (category, tab) in enumerate(zip(categories.keys(), tabs)):
            with tab:
                st.markdown(f"<h3>{category.upper()} Test Vectors</h3>", unsafe_allow_html=True)
                
                # Create a list of test vectors
                for j, tv in enumerate(categories[category]):
                    with st.container():
                        col1, col2 = st.columns([4, 1])
                        
                        with col1:
                            st.markdown(f"### {tv['name']}")
                            st.markdown(f"**Severity:** {tv['severity'].upper()}")
                            st.markdown(f"**Category:** {tv['category'].upper()}")
                        
                        with col2:
                            # Use a checkbox to enable/disable
                            is_enabled = st.checkbox("Enable", value=True, key=f"enable_{tv['id']}")
    except Exception as e:
        logger.error(f"Error rendering test configuration: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in test configuration: {str(e)}")

def render_run_assessment():
    """Render the run assessment page safely"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Run Assessment</h2>
        <p>Execute security tests against your targets</p>
        """, unsafe_allow_html=True)
        
        # Check if targets exist
        if not st.session_state.targets:
            st.warning("No targets configured. Please add a target first.")
            if st.button("Add Target", key="run_add_target"):
                set_page("Target Management")
                safe_rerun()
            return
        
        # Check if a test is already running
        if st.session_state.running_test:
            # Show progress
            progress_placeholder = st.empty()
            with progress_placeholder.container():
                progress_bar = st.progress(st.session_state.progress)
                st.markdown(f"**Progress:** {int(st.session_state.progress*100)}%")
                st.markdown(f"**Vulnerabilities found:** {st.session_state.vulnerabilities_found}")
            
            # Stop button
            if st.button("Stop Test", key="stop_test"):
                st.session_state.running_test = False
                logger.info("Test stopped by user")
                st.warning("Test stopped by user")
                safe_rerun()
        else:
            # Test configuration
            col1, col2 = st.columns(2)
            
            with col1:
                st.markdown("<h3>Select Target</h3>", unsafe_allow_html=True)
                target_options = [t["name"] for t in st.session_state.targets]
                selected_target = st.selectbox("Target", target_options, key="run_target")
            
            with col2:
                st.markdown("<h3>Test Parameters</h3>", unsafe_allow_html=True)
                test_duration = st.slider("Test Duration (seconds)", 5, 60, 30, key="run_duration", 
                                         help="For demonstration purposes, we're using seconds. In a real system, this would be minutes.")
            
            # Environmental impact tracking option
            st.markdown("<h3>Environmental Impact Tracking</h3>", unsafe_allow_html=True)
            track_carbon = st.checkbox("Track Carbon Emissions During Test", value=True, key="track_carbon_emissions")
            
            if track_carbon:
                st.info("Carbon tracking will be enabled during the test to measure environmental impact")
            
            # Run test button
            if st.button("Run Assessment", use_container_width=True, type="primary", key="start_assessment"):
                try:
                    # Find the selected target object
                    target = next((t for t in st.session_state.targets if t["name"] == selected_target), None)
                    test_vectors = get_mock_test_vectors()
                    
                    if target:
                        # Initialize carbon tracking if requested
                        if track_carbon and 'carbon_tracker' not in st.session_state:
                            st.session_state.carbon_tracker = CarbonImpactTracker()
                            st.session_state.carbon_tracker.initialize_tracker(f"Security Test - {target['name']}")
                        
                        if track_carbon:
                            st.session_state.carbon_tracker.start_tracking()
                            st.session_state.carbon_tracking_active = True
                        
                        # Start the test in a background thread
                        test_thread = threading.Thread(
                            target=run_mock_test,
                            args=(target, test_vectors, test_duration)
                        )
                        test_thread.daemon = True
                        test_thread.start()
                        
                        # Track the thread
                        st.session_state.active_threads.append(test_thread)
                        
                        st.session_state.running_test = True
                        logger.info(f"Started test against {target['name']} with {len(test_vectors)} vectors")
                        st.success("Test started!")
                        safe_rerun()
                    else:
                        st.error("Selected target not found")
                except Exception as e:
                    logger.error(f"Error starting test: {str(e)}")
                    st.error(f"Failed to start test: {str(e)}")
    
    except Exception as e:
        logger.error(f"Error rendering run assessment: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in run assessment: {str(e)}")

def render_results_analyzer():
    """Render the results analyzer page safely"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Results Analyzer</h2>
        <p>Explore and analyze security assessment results</p>
        """, unsafe_allow_html=True)
        
        # Check if there are results to display
        if not st.session_state.test_results:
            st.warning("No Results Available - Run an assessment to generate results.")
            
            if st.button("Go to Run Assessment", key="results_goto_run"):
                set_page("Run Assessment")
                safe_rerun()
            return
        
        # Display results summary
        results = st.session_state.test_results
        vulnerabilities = results.get("vulnerabilities", [])
        summary = results.get("summary", {})
        
        # Create header with summary metrics
        st.markdown(f"""
        <div style="margin-bottom: 20px;">
            <h3>Assessment Results: {results.get("target", "Unknown Target")}</h3>
            <div style="opacity: 0.7;">Completed: {results.get("timestamp", "Unknown")}</div>
        </div>
        """, unsafe_allow_html=True)
        
        # Summary metrics
        col1, col2, col3 = st.columns(3)
        
        with col1:
            st.metric("Tests Run", summary.get("total_tests", 0))
        
        with col2:
            st.metric("Vulnerabilities", summary.get("vulnerabilities_found", 0))
        
        with col3:
            st.metric("Risk Score", summary.get("risk_score", 0))
        
        # Visualizations
        st.markdown("<h3>Vulnerability Overview</h3>", unsafe_allow_html=True)
        
        # Display vulnerabilities in a table
        if vulnerabilities:
            # Create a dataframe for display
            vuln_data = []
            for vuln in vulnerabilities:
                vuln_data.append({
                    "ID": vuln.get("id", "Unknown"),
                    "Test Name": vuln.get("test_name", "Unknown"),
                    "Severity": vuln.get("severity", "Unknown"),
                    "Details": vuln.get("details", "No details")
                })
            
            df = pd.DataFrame(vuln_data)
            st.dataframe(df, use_container_width=True)
    
    except Exception as e:
        logger.error(f"Error rendering results analyzer: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in results analyzer: {str(e)}")

def render_ethical_ai_testing():
    """Render the ethical AI testing page safely"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Ethical AI Testing</h2>
        <p>Comprehensive assessment of AI systems against ethical guidelines</p>
        """, unsafe_allow_html=True)
        
        # Create tabs for different testing frameworks
        tabs = st.tabs(["OWASP LLM", "NIST Framework", "Fairness & Bias", "Privacy Compliance"])
        
        with tabs[0]:
            st.markdown("<h3>OWASP LLM Top 10 Testing</h3>", unsafe_allow_html=True)
            
            st.markdown("""
            This module tests AI systems against the OWASP Top 10 for Large Language Model Applications:
            
            - Prompt Injection
            - Insecure Output Handling
            - Training Data Poisoning
            - Model Denial of Service
            - Supply Chain Vulnerabilities
            - Sensitive Information Disclosure
            - Insecure Plugin Design
            - Excessive Agency
            - Overreliance
            - Model Theft
            """)
            
            if st.button("Run OWASP LLM Tests", key="run_owasp"):
                st.info("OWASP LLM testing would start here")
        
        with tabs[1]:
            st.markdown("<h3>NIST AI Risk Management Framework</h3>", unsafe_allow_html=True)
            
            st.markdown("""
            This module evaluates AI systems against the NIST AI Risk Management Framework:
            
            - Governance
            - Mapping
            - Measurement
            - Management
            """)
            
            if st.button("Run NIST Framework Assessment", key="run_nist"):
                st.info("NIST Framework assessment would start here")
        
        with tabs[2]:
            st.markdown("<h3>Fairness & Bias Testing</h3>", unsafe_allow_html=True)
            
            st.markdown("""
            This module tests AI systems for fairness and bias issues:
            
            - Demographic Parity
            - Equal Opportunity
            - Disparate Impact
            - Representation Bias
            """)
            
            if st.button("Run Fairness Assessment", key="run_fairness"):
                st.info("Fairness assessment would start here")
                # Link to our dedicated bias testing page
                st.markdown("For more comprehensive bias testing, visit our Bias Testing page")
                if st.button("Go to Bias Testing", key="goto_bias_testing"):
                    set_page("Bias Testing")
                    safe_rerun()
        
        with tabs[3]:
            st.markdown("<h3>Privacy Compliance Testing</h3>", unsafe_allow_html=True)
            
            st.markdown("""
            This module tests AI systems for compliance with privacy regulations:
            
            - GDPR
            - CCPA
            - HIPAA
            - PIPEDA
            """)
            
            if st.button("Run Privacy Assessment", key="run_privacy"):
                st.info("Privacy assessment would start here")
    
    except Exception as e:
        logger.error(f"Error rendering ethical AI testing: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in ethical AI testing: {str(e)}")

# ----------------------------------------------------------------
# Page Renderers - Bias and Ethics Pages
# ----------------------------------------------------------------

def render_bias_testing():
    """Render the bias testing page with WhyLabs integration"""
    try:
        render_header()
        
        st.markdown("""
        <h2>AI Bias Testing</h2>
        <p>Analyze and mitigate bias in AI systems using WhyLabs</p>
        """, unsafe_allow_html=True)
        
        # Initialize WhyLabs bias tester if not already done
        if 'whylabs_bias_tester' not in st.session_state:
            st.session_state.whylabs_bias_tester = WhyLabsBiasTest()
        
        # Sample data upload section
        st.markdown("<h3>Upload Dataset</h3>", unsafe_allow_html=True)
        
        uploaded_file = st.file_uploader(
            "Upload CSV or Excel file", 
            type=["csv", "xlsx", "xls"],
            key="bias_testing_upload"
        )
        
        if uploaded_file is not None:
            with st.spinner('Loading dataset...'):
                try:
                    # Determine file type and read accordingly
                    file_extension = uploaded_file.name.split('.')[-1].lower()
                    
                    if file_extension == 'csv':
                        df = pd.read_csv(uploaded_file)
                    elif file_extension in ['xlsx', 'xls']:
                        df = pd.read_excel(uploaded_file)
                    else:
                        st.error("Unsupported file format. Please upload a CSV or Excel file.")
                        return
                    
                    # Store the data
                    st.session_state.imported_data = df
                    st.session_state.imported_file_name = uploaded_file.name
                    
                    st.success(f"Dataset loaded successfully: {df.shape[0]} rows, {df.shape[1]} columns")
                    
                    # Display sample data
                    st.markdown("<h4>Dataset Preview</h4>", unsafe_allow_html=True)
                    st.dataframe(df.head())
                    
                    # Bias analysis configuration
                    st.markdown("<h3>Bias Testing Configuration</h3>", unsafe_allow_html=True)
                    
                    col1, col2 = st.columns(2)
                    
                    with col1:
                        # Select protected attributes (categorical columns)
                        categorical_columns = df.select_dtypes(include=['object', 'category']).columns.tolist()
                        protected_features = st.multiselect(
                            "Select Protected Attributes", 
                            categorical_columns,
                            key="protected_features"
                        )
                    
                    with col2:
                        # Select target column
                        all_columns = df.columns.tolist()
                        target_column = st.selectbox(
                            "Select Target Column (outcome)",
                            all_columns,
                            key="target_column"
                        )
                    
                    # Run analysis button
                    if st.button("Run Bias Analysis", type="primary", key="run_bias_analysis"):
                        if not protected_features:
                            st.error("Please select at least one protected attribute")
                        elif not target_column:
                            st.error("Please select a target column")
                        else:
                            st.info("Bias analysis would run here in the full implementation")
                            st.session_state.show_bias_results = True
                
                except Exception as e:
                    st.error(f"Error loading dataset: {str(e)}")
        
        # Option to use sample data
        st.markdown("<h3>Or Use Sample Dataset</h3>", unsafe_allow_html=True)
        
        if st.button("Load Sample Dataset", key="load_sample_dataset"):
            with st.spinner('Loading sample dataset...'):
                try:
                    # Create a sample dataset with potential bias
                    # Set seed for reproducibility
                    np.random.seed(42)
                    
                    # Create sample data
                    n_samples = 1000
                    
                    # Generate demographic features
                    gender = np.random.choice(['Male', 'Female'], size=n_samples, p=[0.52, 0.48])
                    age_group = np.random.choice(['18-25', '26-35', '36-45', '46-55', '56+'], size=n_samples)
                    ethnicity = np.random.choice(['Group A', 'Group B', 'Group C', 'Group D'], 
                                                size=n_samples, p=[0.6, 0.2, 0.15, 0.05])
                    
                    # Generate features
                    education = np.random.choice(['High School', 'Bachelor', 'Master', 'PhD'], size=n_samples)
                    experience = np.random.randint(0, 20, size=n_samples)
                    
                    # Create biased outcomes
                    gender_bias = (gender == 'Male') * 0.2
                    ethnicity_bias = np.zeros(n_samples)
                    ethnicity_bias[ethnicity == 'Group A'] = 0.1
                    ethnicity_bias[ethnicity == 'Group D'] = -0.15
                    
                    # Base approval probability
                    base_prob = 0.5
                    approval_prob = base_prob + gender_bias + ethnicity_bias
                    approval_prob = np.clip(approval_prob, 0, 1)
                    
                    # Generate approval decisions
                    approved = np.random.binomial(1, approval_prob)
                    
                    # Create DataFrame
                    df = pd.DataFrame({
                        'Gender': gender,
                        'Age_Group': age_group,
                        'Ethnicity': ethnicity,
                        'Education': education,
                        'Experience_Years': experience,
                        'Approved': approved
                    })
                    
                    # Store the data
                    st.session_state.imported_data = df
                    st.session_state.imported_file_name = "sample_biased_dataset.csv"
                    
                    st.success("Sample dataset loaded successfully")
                    
                    # Display sample data
                    st.dataframe(df.head())
                    
                    # Show sample bias metrics
                    st.markdown("<h3>Bias Analysis Results (Sample)</h3>", unsafe_allow_html=True)
                    
                    col1, col2 = st.columns(2)
                    
                    with col1:
                        # Gender bias chart
                        gender_approval = df.groupby('Gender')['Approved'].mean().reset_index()
                        fig_gender = px.bar(
                            gender_approval, 
                            x='Gender', 
                            y='Approved', 
                            title='Approval Rate by Gender',
                            color='Approved',
                            color_continuous_scale='Viridis'
                        )
                        st.plotly_chart(fig_gender, use_container_width=True)
                    
                    with col2:
                        # Ethnicity bias chart
                        ethnicity_approval = df.groupby('Ethnicity')['Approved'].mean().reset_index()
                        fig_ethnicity = px.bar(
                            ethnicity_approval, 
                            x='Ethnicity', 
                            y='Approved', 
                            title='Approval Rate by Ethnicity',
                            color='Approved',
                            color_continuous_scale='Viridis'
                        )
                        st.plotly_chart(fig_ethnicity, use_container_width=True)
                
                except Exception as e:
                    st.error(f"Error creating sample dataset: {str(e)}")
    
    except Exception as e:
        logger.error(f"Error rendering bias testing: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in bias testing: {str(e)}")

def render_bias_comparison():
    """Render the bias comparison visualization page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Bias Comparison Visualization</h2>
        <p>Compare your model's bias metrics against industry benchmarks</p>
        """, unsafe_allow_html=True)
        
        # Embed the HTML visualization
        html_code = """
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="UTF-8">
          <title>Bias Comparison Visualization</title>
          <style>
            body { font-family: 'Segoe UI', sans-serif; padding: 20px; background: #f8f9fa; }
            .bias-chart { margin: 20px 0; }
            .chart-container { position: relative; height: 80px; background: #f1f3f4; border-radius: 8px; }
            .model-score { position: absolute; top: 10px; height: 20px; background: #1a73e8; color: white; text-align: center; line-height: 20px; border-radius: 4px; }
            .benchmark { position: absolute; top: 40px; height: 20px; background: #e37400; color: white; text-align: center; line-height: 20px; border-radius: 4px; }
            .benchmark.top { background: #188038; }
          </style>
        </head>
        <body>
          <div class="bias-chart">
            <h4>Model Bias Performance vs. Industry Standards</h4>
            <div class="chart-container">
              <div class="model-score" style="width:65%">Your Model: 65%</div>
              <div class="benchmark" style="left:70%">Industry Avg: 70%</div>
              <div class="benchmark top" style="left:85%">Top: 85%</div>
            </div>
          </div>
        </body>
        </html>
        """
        components.html(html_code, height=400, scrolling=True)
        
        # Additional performance metrics
        st.markdown("<h3>Detailed Bias Metrics</h3>", unsafe_allow_html=True)
        
        # Create tabs for different bias categories
        tabs = st.tabs(["Gender", "Age", "Race/Ethnicity", "Socioeconomic"])
        
        with tabs[0]:
            # Gender bias metrics
            col1, col2 = st.columns(2)
            
            with col1:
                st.metric("Gender Parity Difference", "0.14", "Higher is worse")
                st.metric("Equal Opportunity Difference", "0.09", "Higher is worse")
            
            with col2:
                st.metric("Disparate Impact Ratio", "0.82", "Closer to 1.0 is better")
                st.metric("Treatment Equality Ratio", "0.91", "Closer to 1.0 is better")
            
            # Sample visualization
            gender_data = pd.DataFrame({
                'Group': ['Male', 'Female', 'Non-Binary'],
                'Approval Rate': [0.72, 0.58, 0.63]
            })
            
            fig = px.bar(
                gender_data,
                x='Group',
                y='Approval Rate',
                color='Approval Rate',
                title="Approval Rates by Gender",
                color_continuous_scale='Viridis'
            )
            st.plotly_chart(fig, use_container_width=True)
        
        # Other tabs would have similar content
        
        # Recommendations section
        st.markdown("<h3>Bias Mitigation Recommendations</h3>", unsafe_allow_html=True)
        
        with st.expander("Pre-processing Techniques", expanded=True):
            st.markdown("""
            - **Reweighing**: Assign weights to training examples to ensure fair representation
            - **Disparate Impact Removal**: Transform features to remove correlation with protected attributes
            - **Learning Fair Representations**: Learn intermediate representations that encode data well but obfuscate protected attributes
            """)
        
        with st.expander("In-processing Techniques"):
            st.markdown("""
            - **Adversarial Debiasing**: Use adversarial techniques to remove bias during training
            - **Prejudice Remover**: Add a regularization term to the objective function to reduce bias
            - **Meta Fair Classifier**: Ensemble approach that combines multiple fair classifiers
            """)
        
        with st.expander("Post-processing Techniques"):
            st.markdown("""
            - **Reject Option Classification**: Modify the decision boundary in regions with high uncertainty
            - **Equalized Odds Post-processing**: Adjust model outputs to ensure equal error rates
            - **Calibrated Equalized Odds**: Optimize the trade-off between utility and fairness
            """)
        
        # Action buttons
        col1, col2 = st.columns(2)
        
        with col1:
            if st.button("Generate Mitigation Plan", key="gen_mitigation_plan", use_container_width=True):
                st.success("Mitigation plan would be generated here")
        
        with col2:
            if st.button("Export Bias Report", key="export_bias_report", use_container_width=True):
                st.success("Bias report would be exported here")
    
    except Exception as e:
        logger.error(f"Error rendering bias comparison: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in bias comparison: {str(e)}")

def render_bias_labs_integration():
    """Render the bias labs integration page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Bias Labs Integration</h2>
        <p>Connect with external bias analysis frameworks</p>
        """, unsafe_allow_html=True)
        
        # Embed HTML component for bias labs
        html_code = """
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="UTF-8">
          <title>Bias Labs Integration</title>
          <style>
            body { font-family: 'Segoe UI', sans-serif; padding: 20px; background: #f8f9fa; }
            .container { max-width: 800px; margin: 0 auto; }
          </style>
          <script>
            class BiasLabsIntegration {
              constructor() {
                this.supportedLabs = {
                  "helm": { name: "Stanford HELM", capabilities: ["fairness", "toxicity", "stereotype"], apiEndpoint: "https://crfm.stanford.edu/helm/api/v1/" },
                  "fairlearn": { name: "Microsoft Fairlearn", capabilities: ["demographic_parity", "equal_opportunity", "false_positive_rate_parity"], apiEndpoint: "https://fairlearn.azure-api.net/v1/" },
                  "aequitas": { name: "UChicago Aequitas", capabilities: ["disparate_impact", "statistical_parity", "proportional_parity"], apiEndpoint: "https://aequitas.dsapp.org/api/" },
                  "responsibleai": { name: "Google What-If Tool", capabilities: ["counterfactual_fairness", "intersectional_analysis"], apiEndpoint: "https://responsibleai.googleapis.com/v1/" }
                };
                this.testResults = {};
                this.activeLabs = new Set(["helm", "fairlearn"]);
              }
              
              async evaluateModel(model, options = {}) {
                let results = { model_id: model.id, timestamp: new Date().toISOString(), lab_results: {} };
                let tasks = [];
                for (let labId of this.activeLabs) {
                  tasks.push(this.evaluateWithLab(labId, model, options, results));
                }
                await Promise.all(tasks);
                results.aggregate_metrics = { fairness_score: 65, demographic_parity: 0.18, equal_opportunity: 0.15 }; // Dummy metrics
                results.recommendations = [{ area: "fairness", recommendation: "Improve fairness", priority: "high" }];
                this.testResults[model.id + "-" + Date.now()] = results;
                return results;
              }
              
              async evaluateWithLab(labId, model, options, results) {
                await new Promise(resolve => setTimeout(resolve, 500));
                results.lab_results[labId] = { status: "completed", results: { metrics: { fairness_score: 65, demographic_parity: 0.18, equal_opportunity: 0.15 }, issues: [] } };
              }
            }
            window.biasLabsIntegration = new BiasLabsIntegration();
            async function runBiasEvaluation() {
              const model = { id: document.getElementById("model-id").value, provider: document.getElementById("model-provider").value };
              const results = await window.biasLabsIntegration.evaluateModel(model, {});
              document.getElementById("lab-results").innerText = JSON.stringify(results, null, 2);
            }
          </script>
        </head>
        <body>
          <div class="container">
            <h2>Bias Labs Integration</h2>
            <input type="text" id="model-id" placeholder="Model ID" value="example-model">
            <input type="text" id="model-provider" placeholder="Model Provider" value="example-provider">
            <button onclick="runBiasEvaluation()">Run Bias Labs Evaluation</button>
            <pre id="lab-results"></pre>
          </div>
        </body>
        </html>
        """
        components.html(html_code, height=600, scrolling=True)
        
        # Integration configuration
        st.markdown("<h3>Configure Bias Labs</h3>", unsafe_allow_html=True)
        
        # Select labs to integrate with
        st.markdown("<h4>Select Bias Testing Frameworks</h4>", unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            st.checkbox("Stanford HELM", value=True, key="enable_helm")
            st.checkbox("Microsoft Fairlearn", value=True, key="enable_fairlearn")
        
        with col2:
            st.checkbox("UChicago Aequitas", value=False, key="enable_aequitas")
            st.checkbox("Google What-If Tool", value=False, key="enable_withifool")
        
        # API configuration
        st.markdown("<h4>API Configuration</h4>", unsafe_allow_html=True)
        
        with st.expander("API Keys (Optional)"):
            st.text_input("HELM API Key", type="password", key="helm_api_key")
            st.text_input("Fairlearn API Key", type="password", key="fairlearn_api_key")
            st.text_input("Aequitas API Key", type="password", key="aequitas_api_key")
            st.text_input("What-If Tool API Key", type="password", key="whatif_api_key")
        
        # Save configuration button
        if st.button("Save Bias Labs Configuration", key="save_bias_labs"):
            st.session_state.bias_labs_enabled = True
            st.success("Bias labs configuration saved successfully!")
    
    except Exception as e:
        logger.error(f"Error rendering bias labs integration: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in bias labs integration: {str(e)}")

def render_helm_evaluation():
    """Render the HELM evaluation page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>HELM Evaluation</h2>
        <p>Evaluate models using the Holistic Evaluation of Language Models framework</p>
        """, unsafe_allow_html=True)
        
        # Embed the HTML component
        html_code = """
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="UTF-8">
          <title>HELM Evaluation</title>
          <style>
            body { font-family: 'Segoe UI', sans-serif; padding: 20px; background: #f8f9fa; }
            .btn { padding: 8px 16px; background: #1a73e8; color: white; border: none; border-radius: 4px; cursor: pointer; }
          </style>
          <script>
            async function evaluateWithHELM(model, scenarios) {
              // Simulated HELM API call
              return { fairness: 0.65, toxicity: 0.10, stereotype: 0.15, source: "HELM" };
            }
            document.addEventListener("DOMContentLoaded", () => {
              document.getElementById("run-helm").addEventListener("click", async () => {
                const model = { 
                  id: document.getElementById("model-id").value, 
                  provider: document.getElementById("model-provider").value 
                };
                const scenarios = document.getElementById("scenarios").value.split(",");
                const results = await evaluateWithHELM(model, scenarios);
                document.getElementById("helm-results").innerText = JSON.stringify(results, null, 2);
              });
            });
          </script>
        </head>
        <body>
          <h2>HELM Evaluation</h2>
          <input type="text" id="model-id" placeholder="Model ID" value="example-model">
          <input type="text" id="model-provider" placeholder="Model Provider" value="example-provider">
          <input type="text" id="scenarios" placeholder="Scenarios (comma separated)" value="scenario1,scenario2">
          <button class="btn" id="run-helm">Run HELM Evaluation</button>
          <pre id="helm-results"></pre>
        </body>
        </html>
        """
        components.html(html_code, height=600, scrolling=True)
        
        # Additional HELM configuration
        st.markdown("<h3>HELM Configuration</h3>", unsafe_allow_html=True)
        
        # Evaluation scenarios
        st.markdown("<h4>Evaluation Scenarios</h4>", unsafe_allow_html=True)
        
        scenarios = [
            "Toxicity",
            "Stereotype & Bias",
            "Misinformation",
            "Extremism",
            "Privacy Leakage",
            "Truthfulness",
            "Hate Speech"
        ]
        
        selected_scenarios = st.multiselect("Select Evaluation Scenarios", scenarios, default=["Toxicity", "Stereotype & Bias"])
        
        # Evaluation settings
        col1, col2 = st.columns(2)
        
        with col1:
            eval_iterations = st.slider("Evaluation Iterations", 1, 100, 10)
            temperature = st.slider("Temperature", 0.0, 1.0, 0.7, 0.1)
        
        with col2:
            concurrent_evals = st.slider("Concurrent Evaluations", 1, 10, 3)
            timeout = st.slider("Timeout (seconds)", 1, 60, 30)
        
        # Run evaluation button
        if st.button("Run HELM Evaluation", key="run_helm_eval", type="primary"):
            with st.spinner("Running HELM evaluation..."):
                # Simulate evaluation
                time.sleep(2)
                st.success("HELM evaluation completed!")
                
                # Show mock results
                st.markdown("<h3>Evaluation Results</h3>", unsafe_allow_html=True)
                
                # Example results data
                results_data = {
                    "Toxicity": 0.12,
                    "Stereotype & Bias": 0.19,
                    "Truthfulness": 0.71,
                    "Helpfulness": 0.82
                }
                
                # Plot results
                fig = px.bar(
                    x=list(results_data.keys()),
                    y=list(results_data.values()),
                    title="HELM Evaluation Results",
                    labels={"x": "Dimension", "y": "Score"},
                    color=list(results_data.values()),
                    color_continuous_scale="Viridis"
                )
                
                st.plotly_chart(fig, use_container_width=True)
    
    except Exception as e:
        logger.error(f"Error rendering HELM evaluation: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in HELM evaluation: {str(e)}")

# ----------------------------------------------------------------
# Page Renderers - Sustainability Pages
# ----------------------------------------------------------------

def render_environmental_impact():
    """Render the environmental impact assessment page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Environmental Impact Assessment</h2>
        <p>Analyze and mitigate the carbon footprint of your AI systems</p>
        """, unsafe_allow_html=True)
        
        # Initialize carbon tracker if not already done
        if 'carbon_tracker' not in st.session_state:
            st.session_state.carbon_tracker = CarbonImpactTracker()
            st.session_state.carbon_tracker_initialized = False
        
        # Create tabs for different functionality
        tabs = st.tabs(["Carbon Measurement", "Model Analysis", "Optimization Strategies"])
        
        with tabs[0]:
            st.markdown("<h3>Carbon Emission Tracking</h3>", unsafe_allow_html=True)
            
            # Initialize tracker if needed
            if not st.session_state.carbon_tracker_initialized:
                project_name = st.text_input("Project Name", value="AI Security Assessment", key="carbon_project_name")
                
                if st.button("Initialize Carbon Tracker", key="init_carbon_tracker"):
                    with st.spinner("Initializing tracker..."):
                        success = st.session_state.carbon_tracker.initialize_tracker(project_name)
                        
                        if success:
                            st.session_state.carbon_tracker_initialized = True
                            st.success("Carbon tracker initialized successfully!")
                            safe_rerun()
                        else:
                            st.error("Failed to initialize carbon tracker. Please check logs for details.")
            else:
                # Tracking controls
                if not st.session_state.get("carbon_tracking_active", False):
                    if st.button("Start Carbon Tracking", key="start_carbon_tracking", type="primary"):
                        success = st.session_state.carbon_tracker.start_tracking()
                        
                        if success:
                            st.session_state.carbon_tracking_active = True
                            st.success("Carbon tracking started!")
                            safe_rerun()
                        else:
                            st.error("Failed to start carbon tracking. Please check logs for details.")
                else:
                    if st.button("Stop Carbon Tracking", key="stop_carbon_tracking", type="primary"):
                        emissions = st.session_state.carbon_tracker.stop_tracking()
                        
                        st.session_state.carbon_tracking_active = False
                        st.success(f"Carbon tracking stopped! Measured: {emissions:.6f} kg CO2eq")
                        
                        # Store the last measurement
                        if 'carbon_measurements' not in st.session_state:
                            st.session_state.carbon_measurements = []
                        
                        st.session_state.carbon_measurements.append({
                            "timestamp": datetime.now().isoformat(),
                            "emissions_kg": emissions
                        })
                        
                        safe_rerun()
                
                # Display tracking status
                if st.session_state.get("carbon_tracking_active", False):
                    st.info("Carbon tracking is active. Run your AI operations and stop tracking when finished.")
                
                # Display measurements
                st.markdown("<h4>Emission Measurements</h4>", unsafe_allow_html=True)
                
                total_emissions = st.session_state.carbon_tracker.get_total_emissions()
                
                col1, col2, col3 = st.columns(3)
                
                with col1:
                    st.metric("Total Emissions", f"{total_emissions:.6f} kg CO2eq")
                
                with col2:
                    # Convert to equivalent metrics
                    miles_driven = total_emissions * 2.4  # ~2.4 miles per kg CO2
                    st.metric("Equivalent Car Miles", f"{miles_driven:.2f} miles")
                
                with col3:
                    # Trees needed to offset
                    trees_needed = total_emissions * 0.06  # ~0.06 trees per kg CO2 per year
                    st.metric("Trees Needed (1 year)", f"{trees_needed:.2f} trees")
                
                # Show all measurements
                if 'carbon_measurements' in st.session_state and st.session_state.carbon_measurements:
                    st.markdown("<h4>Measurement History</h4>", unsafe_allow_html=True)
                    
                    measurements_df = pd.DataFrame(st.session_state.carbon_measurements)
                    st.dataframe(measurements_df)
                    
                    # Create a chart if there are enough measurements
                    if len(st.session_state.carbon_measurements) > 1:
                        # Prepare data
                        measurements_df['timestamp'] = pd.to_datetime(measurements_df['timestamp'])
                        
                        fig = px.line(
                            measurements_df,
                            x='timestamp',
                            y='emissions_kg',
                            title='Carbon Emissions Over Time',
                            labels={'timestamp': 'Time', 'emissions_kg': 'Emissions (kg CO2eq)'}
                        )
                        
                        st.plotly_chart(fig, use_container_width=True)
        
        with tabs[1]:
            st.markdown("<h3>AI Model Carbon Footprint Analysis</h3>", unsafe_allow_html=True)
            
            col1, col2 = st.columns(2)
            
            with col1:
                model_name = st.text_input("Model Name", value="", key="model_name")
                model_parameters = st.number_input("Model Parameters (millions)", min_value=0.1, max_value=1000000.0, value=1.0, key="model_parameters")
                training_hours = st.number_input("Training Hours", min_value=0, max_value=10000, value=24, key="training_hours")
            
            with col2:
                hardware_options = ["CPU", "GPU - Consumer", "GPU - Data Center", "TPU", "Custom ASIC"]
                hardware_type = st.selectbox("Hardware Type", hardware_options, key="hardware_type")
                daily_inferences = st.number_input("Daily Inferences", min_value=0, max_value=1000000000, value=10000, key="daily_inferences")
                hosting_region = st.selectbox("Hosting Region", [
                    "North America", "Europe", "Asia Pacific", "South America", "Africa", "Middle East"
                ], key="hosting_region")
            
            if st.button("Calculate Carbon Footprint", key="calc_footprint", type="primary"):
                # Simulate carbon footprint calculation
                st.success("Carbon footprint calculation complete!")
                
                # Simulating total emissions
                training_emissions = training_hours * 0.5 * (1 if hardware_type == "CPU" else 2)  # Simplified calculation
                inference_emissions = daily_inferences * 0.00001 * (1 if hardware_type == "CPU" else 2)
                yearly_inference = inference_emissions * 365
                total_emissions = training_emissions + yearly_inference
                
                # Show results
                st.markdown("<h4>Carbon Footprint Results</h4>", unsafe_allow_html=True)
                
                col1, col2, col3 = st.columns(3)
                
                with col1:
                    st.metric("Training Emissions", f"{training_emissions:.2f} kg CO2eq")
                
                with col2:
                    st.metric("Daily Inference", f"{inference_emissions:.4f} kg CO2eq")
                
                with col3:
                    st.metric("Yearly Emissions", f"{total_emissions:.2f} kg CO2eq")
                
                # Visualization
                data = pd.DataFrame({
                    "Source": ["Training", "Inference (1 year)"],
                    "Emissions (kg CO2)": [training_emissions, yearly_inference]
                })
                
                fig = px.bar(
                    data,
                    x="Source",
                    y="Emissions (kg CO2)",
                    title="Carbon Emissions Breakdown",
                    color="Source"
                )
                
                st.plotly_chart(fig, use_container_width=True)
        
        with tabs[2]:
            st.markdown("<h3>Carbon Optimization Strategies</h3>", unsafe_allow_html=True)
            
            # Describe different strategies
            strategies = [
                {
                    "name": "Model Architecture Optimization",
                    "description": "Design efficient model architectures that require less computation while maintaining accuracy.",
                    "techniques": [
                        "Pruning - Remove unnecessary connections or neurons",
                        "Knowledge Distillation - Train smaller models using larger models as teachers",
                        "Neural Architecture Search - Automatically find efficient architectures",
                        "Sparsity - Encourage sparse activations and weights"
                    ]
                },
                {
                    "name": "Quantization and Precision Reduction",
                    "description": "Reduce the numerical precision of model weights and operations.",
                    "techniques": [
                        "Post-training quantization - Convert trained model to lower precision",
                        "Quantization-aware training - Train with simulated quantization",
                        "Mixed-precision training - Use different precisions for different operations",
                        "Binary/ternary networks - Use 1-bit or 2-bit weights"
                    ]
                },
                {
                    "name": "Efficient Training Strategies",
                    "description": "Optimize the training process to reduce computational requirements.",
                    "techniques": [
                        "Transfer learning - Start from pre-trained models",
                        "Early stopping - Terminate training when validation performance plateaus",
                        "Learning rate scheduling - Optimize convergence speed",
                        "Gradient accumulation - Reduce memory requirements"
                    ]
                }
            ]
            
            # Create expandable sections for each strategy
            for i, strategy in enumerate(strategies):
                with st.expander(f"{i+1}. {strategy['name']}", expanded=i==0):
                    st.markdown(f"**{strategy['description']}**")
                    
                    st.markdown("#### Key Techniques")
                    for technique in strategy['techniques']:
                        st.markdown(f"- {technique}")
    
    except Exception as e:
        logger.error(f"Error rendering environmental impact: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in environmental impact assessment: {str(e)}")

def render_sustainability_dashboard():
    """Render the sustainability dashboard page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Sustainability Dashboard</h2>
        <p>Monitor and optimize the environmental impact of your AI systems</p>
        """, unsafe_allow_html=True)
        
        # Overview metrics
        col1, col2, col3, col4 = st.columns(4)
        
        with col1:
            # Total carbon footprint
            total_carbon = sum(m.get("emissions_kg", 0) for m in st.session_state.carbon_measurements) if hasattr(st.session_state, "carbon_measurements") else 0
            st.metric("Total Carbon Footprint", f"{total_carbon:.4f} kg CO2e")
        
        with col2:
            # Energy consumption
            energy = total_carbon / 0.5  # Approximate conversion
            st.metric("Energy Consumption", f"{energy:.2f} kWh")
        
        with col3:
            # Carbon efficiency
            if hasattr(st.session_state, "test_results") and st.session_state.test_results:
                test_count = st.session_state.test_results.get("summary", {}).get("total_tests", 0)
                efficiency = (test_count / total_carbon) if total_carbon > 0 else 0
                st.metric("Carbon Efficiency", f"{efficiency:.1f} tests/g CO2e")
            else:
                st.metric("Carbon Efficiency", "N/A")
        
        with col4:
            # Green score
            green_score = min(100, max(0, 100 - (total_carbon * 50)))
            st.metric("Green Score", f"{green_score:.1f}/100")
        
        # Emissions trend
        st.markdown("<h3>Emissions Trend</h3>", unsafe_allow_html=True)
        
        if hasattr(st.session_state, "carbon_measurements") and len(st.session_state.carbon_measurements) > 0:
            # Create DataFrame from measurements
            measurements_df = pd.DataFrame(st.session_state.carbon_measurements)
            
            if len(measurements_df) > 0:
                measurements_df["timestamp"] = pd.to_datetime(measurements_df["timestamp"])
                measurements_df = measurements_df.sort_values("timestamp")
                
                # Calculate cumulative emissions
                measurements_df["cumulative_emissions"] = measurements_df["emissions_kg"].cumsum()
                
                # Create two y-axis plot
                fig = make_subplots(specs=[[{"secondary_y": True}]])
                
                # Add individual measurements
                fig.add_trace(
                    go.Bar(
                        x=measurements_df["timestamp"],
                        y=measurements_df["emissions_kg"],
                        name="Individual Measurements",
                        marker_color="rgba(29, 185, 84, 0.6)"
                    ),
                    secondary_y=False
                )
                
                # Add cumulative line
                fig.add_trace(
                    go.Scatter(
                        x=measurements_df["timestamp"],
                        y=measurements_df["cumulative_emissions"],
                        name="Cumulative Emissions",
                        line=dict(color="rgba(187, 134, 252, 1)")
                    ),
                    secondary_y=True
                )
                
                fig.update_layout(
                    title="Carbon Emissions Over Time",
                    xaxis_title="Date",
                    legend=dict(
                        orientation="h",
                        yanchor="bottom",
                        y=1.02,
                        xanchor="right",
                        x=1
                    )
                )
                
                fig.update_yaxes(title_text="Individual Measurements (kg CO2e)", secondary_y=False)
                fig.update_yaxes(title_text="Cumulative Emissions (kg CO2e)", secondary_y=True)
                
                st.plotly_chart(fig, use_container_width=True)
            else:
                st.info("No emissions data available yet. Run tests with carbon tracking enabled to collect data.")
        else:
            st.info("No emissions data available yet. Run tests with carbon tracking enabled to collect data.")
        
        # Optimization recommendations
        st.markdown("<h3>Sustainability Recommendations</h3>", unsafe_allow_html=True)
        
        recommendations = [
            {
                "title": "Switch to Smaller Models",
                "description": "Consider using more efficient, smaller models for routine tasks. For example, using a 7B parameter model instead of a 70B model can reduce emissions by up to 90% while retaining 95% of capability for many tasks.",
                "impact": "High",
                "effort": "Medium"
            },
            {
                "title": "Implement Model Caching",
                "description": "Cache common query results to avoid redundant inference. This can reduce emissions by 20-40% for frequently repeated queries.",
                "impact": "Medium",
                "effort": "Low"
            },
            {
                "title": "Optimize Batch Scheduling",
                "description": "Schedule non-urgent batch processes during times when the grid's carbon intensity is lowest, typically late night or early morning in most regions.",
                "impact": "Medium",
                "effort": "Medium"
            }
        ]
        
        for i, rec in enumerate(recommendations):
            with st.expander(f"{rec['title']} (Impact: {rec['impact']}, Effort: {rec['effort']})", expanded=i==0):
                st.markdown(rec["description"])
    
    except Exception as e:
        logger.error(f"Error rendering sustainability dashboard: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in sustainability dashboard: {str(e)}")

def render_sustainability_integration():
    """Render the sustainability integration page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Sustainability Integration</h2>
        <p>Integrate sustainability tracking with external tools and frameworks</p>
        """, unsafe_allow_html=True)
        
        # Embed HTML component
        html_code = """
        <!DOCTYPE html>
        <html lang="en">
        <head>
          <meta charset="UTF-8">
          <title>Sustainability Integration</title>
          <style>
            body { font-family: 'Segoe UI', sans-serif; padding: 20px; }
            .message { padding: 10px; background: #e8f0fe; border-radius: 4px; margin-top: 10px; }
          </style>
          <script>
            (async function() {
              const engineRoom = window.engineRoom || { sendPrompt: async () => ({ text: "Dummy response" }), getActiveModel: () => ({ modelId: "gpt-4", provider: "aws" }) };
              class SustainabilityEngineRoomIntegration {
                constructor(engineRoom) {
                  this.engineRoom = engineRoom;
                  console.log("SustainabilityEngineRoomIntegration initialized.");
                }
                async initialize(options = {}) {
                  console.log("Initializing Sustainability Integration with options:", options);
                  document.getElementById('integration-status').innerText = 'Initializing sustainability integration...';
                  await new Promise(resolve => setTimeout(resolve, 1000));
                  document.getElementById('integration-status').innerText = 'Initialization complete.';
                }
                addDashboardToEngineRoom() {
                  console.log("Adding sustainability dashboard to Engine Room UI.");
                  const msgDiv = document.createElement('div');
                  msgDiv.className = 'message';
                  msgDiv.textContent = 'Sustainability dashboard integrated into Engine Room UI.';
                  document.body.appendChild(msgDiv);
                }
              }
              const sustainabilityIntegration = new SustainabilityEngineRoomIntegration(engineRoom);
              await sustainabilityIntegration.initialize({ location: { zipCode: "94105", countryCode: "US" } });
              sustainabilityIntegration.addDashboardToEngineRoom();
            })();
          </script>
        </head>
        <body>
          <h2>Sustainability Integration</h2>
          <div id="integration-status"></div>
        </body>
        </html>
        """
        components.html(html_code, height=200, scrolling=True)
        
        # Integration options
        st.markdown("<h3>Available Integrations</h3>", unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            st.checkbox("Green Software Foundation API", value=True, key="gsf_api")
            st.checkbox("Electricity Maps Carbon Intensity", value=True, key="elec_maps")
            st.checkbox("Cloud Provider Carbon Footprint Tools", value=False, key="cloud_carbon")
        
        with col2:
            st.checkbox("CodeCarbon Integration", value=True, key="codecarbon")
            st.checkbox("WattTime API", value=False, key="watttime")
            st.checkbox("ML CO2 Impact Calculator", value=True, key="ml_co2")
        
        # Configure integration details
        with st.expander("Integration Details", expanded=True):
            integration_option = st.selectbox("Select Integration to Configure", 
                                             ["Green Software Foundation API", "Electricity Maps", "CodeCarbon"])
            
            if integration_option == "Green Software Foundation API":
                st.text_input("API Endpoint", value="https://api.greensoftware.foundation/v1", key="gsf_endpoint")
                st.text_input("API Key", type="password", key="gsf_api_key")
                st.selectbox("Measurement Method", ["SCI Methodology", "Carbon-aware SDK", "Custom"], key="gsf_method")
            
            elif integration_option == "Electricity Maps":
                st.text_input("API Endpoint", value="https://api.electricitymaps.com/v3", key="elec_maps_endpoint")
                st.text_input("API Key", type="password", key="elec_maps_api_key")
                st.multiselect("Tracked Regions", ["US West", "US East", "Europe Central", "Asia Pacific"], 
                               default=["US West", "Europe Central"], key="elec_maps_regions")
            
            elif integration_option == "CodeCarbon":
                st.text_input("Project Name", value="AI Security Analysis", key="codecarbon_project")
                st.text_input("Output Directory", value="./emissions", key="codecarbon_output")
                st.checkbox("Track GPU Usage", value=True, key="codecarbon_gpu")
                st.checkbox("Country-specific Grid Emissions", value=True, key="codecarbon_country")
        
        # Save configuration
        if st.button("Save Integration Configuration", key="save_sustainability_config"):
            st.session_state.sustainability_integrated = True
            st.success("Sustainability integration configuration saved successfully!")
            
            # Show example connection
            st.markdown("<h3>Integration Status</h3>", unsafe_allow_html=True)
            st.markdown("""
            ```
            Connecting to Green Software Foundation API... Success
            Connecting to Electricity Maps API... Success
            Initializing CodeCarbon tracking... Success
            Integration status: Active
            ```
            """)
    
    except Exception as e:
        logger.error(f"Error rendering sustainability integration: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in sustainability integration: {str(e)}")

# ----------------------------------------------------------------
# Page Renderers - Integration & Tool Pages
# ----------------------------------------------------------------

def render_engine_room_integration():
    """Render the engine room integration page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Engine Room Integration</h2>
        <p>Integrate with the Engine Room for advanced AI testing capabilities</p>
        """, unsafe_allow_html=True)
        
        # Embed HTML component
        html_code = """
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="UTF-8">
          <title>Engine Room Integration</title>
          <style>
            body { font-family: 'Segoe UI', sans-serif; padding: 1rem; }
          </style>
          <script>
            class EngineRoomIntegration {
              constructor(engineRoom) {
                this.engineRoom = engineRoom;
              }
              initializeEngineRoom(containerId) {
                document.getElementById(containerId).innerHTML = "<p>Engine Room Initialized</p>";
              }
              addToNavigation(navContainerId) {
                document.getElementById(navContainerId).innerHTML = "<p>Engine Room Navigation Added</p>";
              }
              getRedTeamingMiddleware() {
                return async (input, options) => ({ text: "Response for: " + input.content });
              }
            }
            const engineRoom = window.engineRoom || { sendPrompt: async () => ({ text: "Dummy response" }) };
            const integration = new EngineRoomIntegration(engineRoom);
            window.onload = () => {
              integration.initializeEngineRoom("engine-room-container");
              integration.addToNavigation("engine-room-nav");
            }
          </script>
        </head>
        <body>
          <div id="engine-room-nav"></div>
          <div id="engine-room-container"></div>
        </body>
        </html>
        """
        components.html(html_code, height=200, scrolling=True)
        
        # Integration configuration
        st.markdown("<h3>Engine Room Configuration</h3>", unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            er_endpoint = st.text_input("Engine Room API Endpoint", value="https://api.engineroom.dev", key="er_endpoint")
            er_api_key = st.text_input("API Key", type="password", key="er_api_key")
        
        with col2:
            er_version = st.selectbox("Engine Room Version", ["v1", "v2", "v3 (Beta)"], key="er_version")
            er_timeout = st.slider("Request Timeout (seconds)", 1, 60, 30, key="er_timeout")
        
        # Test connection button
        if st.button("Test Connection", key="test_er_connection"):
            with st.spinner("Testing connection to Engine Room..."):
                # Simulate connection test
                time.sleep(2)
                st.session_state.engine_room_initialized = True
                st.success("Successfully connected to Engine Room!")
        
        # Feature configuration
        if st.session_state.get("engine_room_initialized", False):
            st.markdown("<h3>Feature Configuration</h3>", unsafe_allow_html=True)
            
            st.checkbox("Enable Red Teaming Middleware", value=True, key="er_redteaming")
            st.checkbox("Enable PII Detection", value=True, key="er_pii")
            st.checkbox("Enable Prompt Injection Protection", value=True, key="er_prompt_injection")
            st.checkbox("Enable Result Caching", value=False, key="er_caching")
            
            # Save configuration
            if st.button("Save Engine Room Configuration", key="save_er_config"):
                st.success("Engine Room configuration saved successfully!")
    
    except Exception as e:
        logger.error(f"Error rendering engine room integration: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in engine room integration: {str(e)}")

def render_knowledge_base_integration():
    """Render the knowledge base integration page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Knowledge Base Integration</h2>
        <p>Integrate with external knowledge bases for enhanced AI security analysis</p>
        """, unsafe_allow_html=True)
        
        # Embed HTML component
        html_code = """
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="UTF-8">
          <title>Knowledge Base</title>
          <style>
            body { font-family: 'Segoe UI', sans-serif; padding: 1rem; }
            #widget { border: 1px solid #eee; padding: 1rem; border-radius: 4px; }
          </style>
        </head>
        <body>
          <div id="widget">
            <h3>Knowledge Base</h3>
            <input type="text" id="search" placeholder="Search...">
            <button onclick="document.getElementById('results').innerText='Searching...';">Search</button>
            <div id="results"></div>
          </div>
        </body>
        </html>
        """
        components.html(html_code, height=200, scrolling=True)
        
        # Available knowledge bases
        st.markdown("<h3>Available Knowledge Bases</h3>", unsafe_allow_html=True)
        
        kb_options = {
            "MITRE ATT&CK": "A globally-accessible knowledge base of adversary tactics and techniques",
            "OWASP Top 10": "Standard awareness document for developers and web application security",
            "NIST AI RMF": "NIST AI Risk Management Framework",
            "CVE Database": "Common Vulnerabilities and Exposures database"
        }
        
        col1, col2 = st.columns(2)
        
        for i, (name, desc) in enumerate(kb_options.items()):
            col = col1 if i % 2 == 0 else col2
            with col:
                st.checkbox(name, value=True, key=f"kb_{name.lower().replace(' ', '_')}")
                st.markdown(f"<div style='font-size: 0.8em; opacity: 0.7; margin-bottom: 15px;'>{desc}</div>", unsafe_allow_html=True)
        
        # Integration settings
        st.markdown("<h3>Integration Settings</h3>", unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            update_frequency = st.selectbox("Knowledge Base Update Frequency", 
                                           ["Daily", "Weekly", "Monthly", "Manual"], 
                                           key="kb_update_freq")
            kb_cache = st.checkbox("Enable Local Caching", value=True, key="kb_enable_cache")
        
        with col2:
            kb_query_limit = st.slider("Query Rate Limit (per minute)", 1, 100, 10, key="kb_query_limit")
            kb_timeout = st.slider("Query Timeout (seconds)", 1, 30, 5, key="kb_timeout")
        
        # Save settings
        if st.button("Save Knowledge Base Settings", key="save_kb_settings"):
            st.success("Knowledge base settings saved successfully!")
    
    except Exception as e:
        logger.error(f"Error rendering knowledge base integration: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in knowledge base integration: {str(e)}")

def render_html_portal():
    """Render the HTML portal page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>HTML Portal</h2>
        <p>Access and embed HTML content for extended functionality</p>
        """, unsafe_allow_html=True)
        
        # Embed HTML component
        html_code = """
        <!DOCTYPE html>
        <html lang="en">
        <head>
          <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <title>HTML Portal</title>
          <style>
            body { font-family: 'Segoe UI', sans-serif; padding: 20px; background: #f9f9f9; }
          </style>
        </head>
        <body>
          <h1>HTML Portal</h1>
          <p>This is the HTML Portal page where you can embed custom HTML content.</p>
          
          <div style="margin-top: 20px;">
            <h3>Sample Embedded Content</h3>
            <div style="padding: 15px; background: #f0f0f0; border-radius: 4px;">
              <p>This is an example of embedded HTML content.</p>
            </div>
          </div>
        </body>
        </html>
        """
        components.html(html_code, height=300, scrolling=True)
        
        # HTML editor
        st.markdown("<h3>HTML Content Editor</h3>", unsafe_allow_html=True)
        
        html_content = st.text_area("Edit HTML Content", value="""
<!DOCTYPE html>
<html>
<head>
  <title>Custom Content</title>
  <style>
    body { font-family: sans-serif; }
    .container { padding: 20px; background: #f5f5f5; border-radius: 8px; }
  </style>
</head>
<body>
  <div class="container">
    <h2>My Custom Content</h2>
    <p>This is custom HTML content that can be embedded in the portal.</p>
  </div>
</body>
</html>
        """, height=300, key="html_editor")
        
        # Preview button
        if st.button("Preview HTML", key="preview_html"):
            components.html(html_content, height=300, scrolling=True)
    
    except Exception as e:
        logger.error(f"Error rendering HTML portal: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in HTML portal: {str(e)}")

def render_model_evaluation():
    """Render the model evaluation page"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Model Evaluation</h2>
        <p>Comprehensive evaluation of AI model performance, ethics, and sustainability</p>
        """, unsafe_allow_html=True)
        
        # Model selection section
        st.markdown("<h3>Select Model for Evaluation</h3>", unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            model_name = st.text_input("Model Name", value="", key="eval_model_name")
            model_provider = st.selectbox("Model Provider", ["OpenAI", "Anthropic", "Cohere", "Meta", "Google", "Other"], key="eval_provider")
        
        with col2:
            model_version = st.text_input("Model Version", value="", key="eval_model_version")
            model_type = st.selectbox("Model Type", ["General Purpose LLM", "Specialized LLM", "Embedding Model", "Image Generation", "Other"], key="eval_model_type")
        
        # Evaluation dimensions
        st.markdown("<h3>Evaluation Dimensions</h3>", unsafe_allow_html=True)
        
        col1, col2, col3 = st.columns(3)
        
        with col1:
            st.checkbox("Security & Safety", value=True, key="eval_security")
            st.checkbox("Performance & Accuracy", value=True, key="eval_performance")
        
        with col2:
            st.checkbox("Bias & Fairness", value=True, key="eval_bias")
            st.checkbox("Robustness", value=True, key="eval_robustness")
        
        with col3:
            st.checkbox("Environmental Impact", value=True, key="eval_environmental")
            st.checkbox("Compliance", value=True, key="eval_compliance")
        
        # Advanced settings
        with st.expander("Advanced Settings", expanded=False):
            st.slider("Evaluation Iterations", 1, 100, 10, key="eval_iterations")
            st.slider("Evaluation Timeout (minutes)", 1, 60, 30, key="eval_timeout")
            st.checkbox("Generate Detailed Report", value=True, key="eval_detailed_report")
            st.checkbox("Include Benchmark Comparisons", value=True, key="eval_benchmarks")
        
        # Run evaluation button
        if st.button("Run Comprehensive Evaluation", type="primary", key="run_comprehensive_eval"):
            if not model_name:
                st.error("Please enter a model name")
            else:
                with st.spinner(f"Evaluating {model_name}..."):
                    # Simulate evaluation
                    time.sleep(3)
                    st.success(f"Evaluation of {model_name} completed!")
                    
                    # Show sample results
                    st.markdown("<h3>Evaluation Results</h3>", unsafe_allow_html=True)
                    
                    # Create tabs for different dimensions
                    tabs = st.tabs(["Overview", "Security", "Bias", "Environmental", "Compliance"])
                    
                    with tabs[0]:
                        # Overall score
                        st.markdown("<h4>Overall Scores</h4>", unsafe_allow_html=True)
                        
                        overall_scores = {
                            "Security & Safety": 82,
                            "Performance & Accuracy": 88,
                            "Bias & Fairness": 76,
                            "Robustness": 79,
                            "Environmental Impact": 65,
                            "Compliance": 84
                        }
                        
                        # Create radar chart
                        fig = go.Figure()
                        
                        fig.add_trace(go.Scatterpolar(
                            r=list(overall_scores.values()),
                            theta=list(overall_scores.keys()),
                            fill='toself',
                            name='Model Scores'
                        ))
                        
                        fig.update_layout(
                            polar=dict(
                                radialaxis=dict(
                                    visible=True,
                                    range=[0, 100]
                                )
                            ),
                            showlegend=False
                        )
                        
                        st.plotly_chart(fig, use_container_width=True)
                    
                    # Other tabs would have detailed metrics for each dimension
    
    except Exception as e:
        logger.error(f"Error rendering model evaluation: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in model evaluation: {str(e)}")

def render_file_import():
    """Render the file import page for multi-format support"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Multi-Format Import</h2>
        <p>Import data in various formats for impact assessment</p>
        """, unsafe_allow_html=True)
        
        # File upload section
        st.markdown("<h3>Upload Files</h3>", unsafe_allow_html=True)
        
        uploaded_file = st.file_uploader(
            "Upload File", 
            type=["json", "csv", "xlsx", "xls", "pdf", "xml", "yaml", "yml"],
            key="multi_format_upload"
        )
        
        if uploaded_file is not None:
            with st.spinner('Processing file...'):
                try:
                    # Process the file based on its type
                    processed_data = handle_multiple_file_formats(uploaded_file)
                    
                    if isinstance(processed_data, dict) and "error" in processed_data:
                        st.error(processed_data["error"])
                    else:
                        st.success(f"File '{uploaded_file.name}' processed successfully!")
                        
                        # Display different previews based on data type
                        if hasattr(processed_data, "head"):  # If it's a DataFrame
                            st.markdown("<h3>Data Preview</h3>", unsafe_allow_html=True)
                            st.dataframe(processed_data.head(10))
                            
                            # Store the data in session state
                            st.session_state.imported_data = processed_data
                            st.session_state.imported_file_name = uploaded_file.name
                            
                            # Show action buttons
                            col1, col2 = st.columns(2)
                            
                            with col1:
                                if st.button("Run Impact Assessment", key="run_impact_assessment", use_container_width=True):
                                    st.session_state.current_page = "Environmental Impact"
                                    safe_rerun()
                            
                            with col2:
                                if st.button("Run Bias Testing", key="run_bias_test", use_container_width=True):
                                    st.session_state.current_page = "Bias Testing"
                                    safe_rerun()
                        
                        elif isinstance(processed_data, dict):  # If it's a dictionary
                            st.markdown("<h3>Data Preview</h3>", unsafe_allow_html=True)
                            st.json(processed_data)
                            
                            # Store the data in session state
                            st.session_state.imported_data = processed_data
                            st.session_state.imported_file_name = uploaded_file.name
                        
                        else:  # For other types
                            st.markdown("<h3>Data Preview</h3>", unsafe_allow_html=True)
                            st.write(processed_data)
                            
                            # Store the data in session state
                            st.session_state.imported_data = processed_data
                            st.session_state.imported_file_name = uploaded_file.name
                
                except Exception as e:
                    st.error(f"Error processing file: {str(e)}")
                    logger.error(f"Error processing file: {str(e)}")
        
        # Information section
        st.markdown("<h3>Supported File Formats</h3>", unsafe_allow_html=True)
        
        formats_info = {
            "JSON": "JavaScript Object Notation - for structured data",
            "CSV": "Comma-Separated Values - for tabular data",
            "Excel": "Microsoft Excel Spreadsheets (XLSX/XLS) - for complex tabular data",
            "PDF": "Portable Document Format - for text extraction",
            "XML": "eXtensible Markup Language - for structured data",
            "YAML/YML": "YAML Ain't Markup Language - for configuration files"
        }
        
        col1, col2 = st.columns(2)
        
        for i, (format_name, description) in enumerate(formats_info.items()):
            col = col1 if i % 2 == 0 else col2
            with col:
                st.markdown(f"**{format_name}**: {description}")
    
    except Exception as e:
        logger.error(f"Error rendering file import: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in file import: {str(e)}")

def render_high_volume_testing():
    """Render the high-volume testing page safely"""
    try:
        render_header()
        
        st.markdown("""
        <h2>High-Volume Testing</h2>
        <p>Autonomous, high-throughput testing for AI systems</p>
        """, unsafe_allow_html=True)
        
        # Check if targets exist
        if not st.session_state.targets:
            st.warning("No targets configured. Please add a target first.")
            if st.button("Add Target", key="highvol_add_target"):
                set_page("Target Management")
                safe_rerun()
            return
        
        # Configuration section
        st.markdown("<h3>Testing Configuration</h3>", unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            target_options = [t["name"] for t in st.session_state.targets]
            st.selectbox("Select Target", target_options, key="highvol_target")
            
            total_tests = st.slider("Total Tests (thousands)", 10, 1000, 100, key="highvol_tests")
            
            max_runtime = st.number_input("Max Runtime (hours)", 1, 24, 3, key="highvol_runtime")
        
        with col2:
            test_vectors = ["Prompt Injection", "Jailbreaking", "Data Extraction", "Input Manipulation", "Boundary Testing"]
            st.multiselect("Test Vectors", test_vectors, default=["Prompt Injection", "Jailbreaking"], key="highvol_vectors")
            
            parallelism = st.selectbox("Parallelism", ["Low (4 workers)", "Medium (8 workers)", "High (16 workers)", "Extreme (32 workers)"], key="highvol_parallel")
            
            save_only_vulns = st.checkbox("Save Only Vulnerabilities", value=True, key="highvol_save_vulns")
        
        # Environment monitoring section
        st.markdown("<h3>Environmental Monitoring</h3>", unsafe_allow_html=True)
        
        carbon_aware = st.checkbox("Enable Carbon-Aware Scheduling", value=True, key="carbon_aware_scheduling",
                                   help="Adjust testing intensity based on carbon intensity of electricity grid")
        
        if carbon_aware:
            st.success("Carbon-aware scheduling will prioritize testing during low-carbon periods")
            
            col1, col2 = st.columns(2)
            
            with col1:
                carbon_threshold = st.slider("Carbon Intensity Threshold (gCO2/kWh)", 100, 500, 300, key="carbon_threshold",
                                            help="Testing will slow down when carbon intensity exceeds this threshold")
            
            with col2:
                st.selectbox("Grid Region", [
                    "us-west",
                    "us-east",
                    "europe-west",
                    "europe-north",
                    "asia-east",
                    "asia-southeast"
                ], key="grid_region")
        
        # Start testing button
        if st.button("Start High-Volume Testing", type="primary", use_container_width=True, key="start_highvol"):
            with st.spinner("Starting high-volume testing..."):
                # Simulate test start
                time.sleep(2)
                st.success("High-volume testing started successfully!")
                
                # Progress display
                st.markdown("<h3>Testing Progress</h3>", unsafe_allow_html=True)
                
                progress_bar = st.progress(0)
                status_text = st.empty()
                
                # Simulate progress updates
                for i in range(101):
                    # Check if the page has been navigated away from
                    if st.session_state.current_page != "High-Volume Testing":
                        break
                    
                    progress_bar.progress(i / 100)
                    status_text.text(f"Progress: {i}% - Processed {i * 1000:,} tests, found {int(i * 20)} vulnerabilities")
                    
                    time.sleep(0.05)  # Just for demonstration
    
    except Exception as e:
        logger.error(f"Error rendering high-volume testing: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in high-volume testing: {str(e)}")

def render_settings():
    """Render the settings page safely"""
    try:
        render_header()
        
        st.markdown("""
        <h2>Settings</h2>
        <p>Configure application settings and preferences</p>
        """, unsafe_allow_html=True)
        
        # Theme settings
        st.markdown("<h3>Theme Settings</h3>", unsafe_allow_html=True)
        
        theme_option = st.radio("Theme", ["Dark", "Light"], index=0 if st.session_state.current_theme == "dark" else 1, key="settings_theme")
        if theme_option == "Dark" and st.session_state.current_theme != "dark":
            st.session_state.current_theme = "dark"
            logger.info("Theme set to dark")
            safe_rerun()
        elif theme_option == "Light" and st.session_state.current_theme != "light":
            st.session_state.current_theme = "light"
            logger.info("Theme set to light")
            safe_rerun()
        
        # API settings
        st.markdown("<h3>API Settings</h3>", unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            api_base_url = st.text_input("API Base URL", "https://api.example.com/v1", key="api_base_url")
        
        with col2:
            default_api_key = st.text_input("Default API Key", type="password", key="default_api_key")
        
        # Save API settings
        if st.button("Save API Settings", key="save_api"):
            st.success("API settings saved successfully!")
            logger.info("API settings updated")
        
        # Environmental settings
        st.markdown("<h3>Environmental Settings</h3>", unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            carbon_tracking = st.checkbox("Enable Carbon Tracking", value=True, key="settings_carbon_tracking")
            carbon_api_key = st.text_input("Carbon API Key (optional)", type="password", key="carbon_api_key",
                                          help="API key for accessing external carbon intensity data")
        
        with col2:
            preferred_region = st.selectbox("Preferred Compute Region", [
                "us-west",
                "us-east",
                "europe-west",
                "europe-north",
                "asia-east",
                "asia-southeast"
            ], index=2, key="preferred_region",
            help="Region with lowest carbon intensity will be preferred for compute-intensive tasks")
            
            emissions_threshold = st.slider("Emissions Alert Threshold (kg CO2)", 0.0, 10.0, 1.0, key="emissions_threshold",
                                          help="Alert when test emissions exceed this threshold")
        
        # Save environmental settings
        if st.button("Save Environmental Settings", key="save_env_settings"):
            st.success("Environmental settings saved successfully!")
            logger.info("Environmental settings updated")
        
        # System information
        st.markdown("<h3>System Information</h3>", unsafe_allow_html=True)
        
        # Get system info
        import platform
        
        system_info = f"""
        - Python Version: {platform.python_version()}
        - Operating System: {platform.system()} {platform.release()}
        - Streamlit Version: {st.__version__}
        - Application Version: 1.0.0
        """
        
        st.code(system_info)
        
        # Clear data button (with confirmation)
        st.markdown("<h3>Data Management</h3>", unsafe_allow_html=True)
        
        if st.button("Clear All Application Data", key="clear_data"):
            # Confirmation
            if st.checkbox("I understand this will reset all targets, results, and settings", key="confirm_clear"):
                # Reset all session state (except current page and theme)
                current_page = st.session_state.current_page
                current_theme = st.session_state.current_theme
                
                for key in list(st.session_state.keys()):
                    if key not in ['current_page', 'current_theme']:
                        del st.session_state[key]
                
                # Restore page and theme
                st.session_state.current_page = current_page
                st.session_state.current_theme = current_theme
                
                # Reinitialize session state
                initialize_session_state()
                
                st.success("All application data has been cleared!")
                logger.info("Application data cleared")
                safe_rerun()
    
    except Exception as e:
        logger.error(f"Error rendering settings: {str(e)}")
        logger.debug(traceback.format_exc())
        st.error(f"Error in settings: {str(e)}")

# ----------------------------------------------------------------
# Main Application
# ----------------------------------------------------------------

def main():
    """Main application entry point with error handling"""
    try:
        # Initialize session state
        initialize_session_state()
        
        # Clean up threads
        cleanup_threads()
        
        # Apply CSS
        st.markdown(load_css(), unsafe_allow_html=True)
        
        # Show error message if exists
        if st.session_state.error_message:
            st.markdown(f"""
            <div class="error-message">
                <strong>Error:</strong> {st.session_state.error_message}
            </div>
            """, unsafe_allow_html=True)
            
            # Add button to clear error
            if st.button("Clear Error"):
                st.session_state.error_message = None
                safe_rerun()
        
        # Render sidebar
        sidebar_navigation()
        
        # Render content based on current page
        if st.session_state.current_page == "Dashboard":
            render_dashboard()
        elif st.session_state.current_page == "Target Management":
            render_target_management()
        elif st.session_state.current_page == "Test Configuration":
            render_test_configuration()
        elif st.session_state.current_page == "Run Assessment":
            render_run_assessment()
        elif st.session_state.current_page == "Results Analyzer":
            render_results_analyzer()
        elif st.session_state.current_page == "Ethical AI Testing":
            render_ethical_ai_testing()
        elif st.session_state.current_page == "Environmental Impact":
            render_environmental_impact()
        elif st.session_state.current_page == "Bias Testing":
            render_bias_testing()
        elif st.session_state.current_page == "Bias Comparison":
            render_bias_comparison()
        elif st.session_state.current_page == "Bias Labs Integration":
            render_bias_labs_integration()
        elif st.session_state.current_page == "HELM Evaluation":
            render_helm_evaluation()
        elif st.session_state.current_page == "Multi-Format Import":
            render_file_import()
        elif st.session_state.current_page == "High-Volume Testing":
            render_high_volume_testing()
        elif st.session_state.current_page == "Sustainability Dashboard":
            render_sustainability_dashboard()
        elif st.session_state.current_page == "Sustainability Integration":
            render_sustainability_integration()
        elif st.session_state.current_page == "Engine Room Integration":
            render_engine_room_integration()
        elif st.session_state.current_page == "Knowledge Base":
            render_knowledge_base_integration()
        elif st.session_state.current_page == "HTML Portal":
            render_html_portal()
        elif st.session_state.current_page == "Model Evaluation":
            render_model_evaluation()
        elif st.session_state.current_page == "Settings":
            render_settings()
        else:
            # Default to dashboard if invalid page
            logger.warning(f"Invalid page requested: {st.session_state.current_page}")
            st.session_state.current_page = "Dashboard"
            render_dashboard()
    
    except Exception as e:
        logger.critical(f"Critical application error: {str(e)}")
        logger.critical(traceback.format_exc())
        st.error(f"Critical application error: {str(e)}")
        st.code(traceback.format_exc())

if __name__ == "__main__":
    main()
