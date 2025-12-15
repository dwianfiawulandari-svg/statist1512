import streamlit as st
import pandas as pd
import numpy as np
from scipy.stats import chi2_contingency, pearsonr, spearmanr
import matplotlib.pyplot as plt
import os
import io

# --- THEME: Teknik/Engineering Blue/Yellow, Card tebal, font digital ---
st.set_page_config(page_title="Aplikasi Analisis Data Survei", layout="wide")
st.markdown("""
    <style>
    .stCard { background-color: #223a5e; color: #FAFAFA; padding: 18px 24px; margin-bottom: 22px;
        border-radius: 16px; box-shadow: 0 4px 18px rgba(0,0,0,0.15); border: 2.7px solid #f7c325;
        font-family: 'Share Tech Mono', 'Consolas', 'Roboto Mono', monospace;}
    .stTitleMain { font-size: 2.4rem; font-family: 'Share Tech Mono','Consolas','Roboto Mono', monospace;
        color: #22d2e9; margin-bottom: 1.4rem;font-weight:800; letter-spacing: 1px; text-shadow: 1px 2px 0px #222,2px 4px 1.5px #fff000aa; }
    .stSubHeader { font-size: 1.29rem; color: #f7c325; margin-top:1rem;
        font-family: 'Share Tech Mono', 'Consolas', 'Roboto Mono', monospace; font-weight:700;}
    .stProfileName { font-weight:600; font-size:1.16rem; margin-bottom:6px; color:#22d2e9;
        font-family:'Share Tech Mono','Consolas','Roboto Mono',monospace;}
    .stProfileRole {font-size:1.04rem; color:#223a5e !important;  
        /* untuk light mode */ margin-bottom:6px;}
    .stOrigin { font-size:1.04rem; color:#f7c325; font-style:italic;
        font-family:'Share Tech Mono','Consolas','Roboto Mono',monospace;}
    .stLabel { background-color: #1976d2; color: #FAFAFA; padding: 4px 12px; border-radius: 8px;
        font-family:'Share Tech Mono','Consolas','Roboto Mono',monospace;
        font-size: 0.95rem; display: inline-block; margin: 0 5px 8px 0; font-weight:700;}
    .st-df { background-color: #122033; border-radius: 10px; padding: 10px; border:2px solid #f7c325;
        font-family:'Share Tech Mono','Consolas','Roboto Mono',monospace;}
    hr {border-top:2.5px solid #f7c325; margin-bottom:16px;}
    
    /* Tambahan agar card dan teks readable di dark/light mode otomatis */
    @media (prefers-color-scheme: dark) {
        body, .stCard, .st-df {
            background-color: #223a5e !important;
            color: #FAFAFA !important;
        }
        .stLabel {
            background-color: #1976d2 !important;
            color: #FAFAFA !important;
        }
    }
    @media (prefers-color-scheme: light) {
        body, .stCard, .st-df {
            background-color: #fffbe5 !important;
            color: #223a5e !important;
        }
        .stLabel {
            background-color: #ffde76 !important;
            color: #223a5e !important;
        }
        .stTitleMain {
            color: #223a5e !important;
            text-shadow: 1px 2px 0px #eee,2px 4px 1.5px #f7c325aa !important;
        }
        .stSubHeader {
            color: #1976d2 !important;
        }
    }
    </style>
    <link href='https://fonts.googleapis.com/css?family=Share+Tech+Mono' rel='stylesheet'>
""", unsafe_allow_html=True)

# Ganti BASE_DIR dengan direktori yang benar jika dijalankan di lingkungan Streamlit
BASE_DIR = os.path.dirname(__file__)

# --- Bahasa & bendera (sidebar) ---
languages = ["Indonesia", "English", "日本語", "简体中文"]
language_flags = {
    "Indonesia": "🇮🇩",
    "English": "🇬🇧",
    "日本語": "🇯🇵",
    "简体中文": "🇨🇳"
}
languages_w_flag = [f"{language_flags[lang]}  {lang}" for lang in languages]
selected_lang_label = st.sidebar.selectbox(
    "🌐 Pilih Bahasa / Choose Language / 言語選択 / 选择语言", languages_w_flag)
lang = selected_lang_label.split(maxsplit=1)[-1]

sidebar_menu = {
    "Indonesia": ["Profil Pembuat", "Analisis Data", "Tentang Aplikasi"],
    "English": ["Author Profile", "Data Analysis", "About App"],
    "日本語": ["著者プロフィール", "データ分析", "アプリについて"],
    "简体中文": ["作者简介", "数据分析", "关于应用"],
}
menu_items = sidebar_menu.get(lang, sidebar_menu["Indonesia"])
menu = st.sidebar.radio("Menu", menu_items)

# --- Multilanguage dictionary untuk semua label/instruksi dan output ---
text = {
    "Indonesia": {
        "title": "Aplikasi Analisis Data Survei",
        "file": "Upload file Excel data survei",
        "analysis_title": "Analisis Data",
        "desc_title": "Distribusi Data",
        "desc_cols": "Pilih variabel numerik untuk analisis deskriptif (histogram & boxplot)",
        "hist": "Histogram", "box": "Boxplot", "preview": "Preview Data",
        "vra_title": "Analisis Hubungan Variabel",
        "vra_var1": "Pilih Variabel 1", "vra_var2": "Pilih Variabel 2",
        "type_num": "Numerik", "type_cat": "Kategori",
        "cat_info": "Variabel kategorik (Kategori vs Kategori) → menggunakan Chi-Square",
        "num_info": "Variabel numerik (Numerik vs Numerik) → Korelasi (Pearson/Spearman)",
        "result_cat_cat": "Tabel Kontingensi (Chi-Square)",
        "result_num_num": "Hasil Korelasi",
        "chi2": "Chi2 = {:.4f}", "pval": "P-value = {:.4f}", "dof": "Degrees of freedom = {}",
        "conclusion": "Kesimpulan:",
        "conclude_sig": "Terdapat hubungan signifikan antara variabel (p < 0.05)",
        "conclude_nosig": "Tidak terdapat hubungan signifikan antara variabel (p >= 0.05)",
        "corr_pearson": "Korelasi Pearson",
        "corr_spearman": "Korelasi Spearman (Non-parametrik)",
        "corr_coef": "Koefisien = {:.4f}", "corr_pval": "P-value = {:.4f}",
        "corr_conclude_sig": "Terdapat hubungan signifikan (p < 0.05)",
        "corr_conclude_nosig": "Tidak terdapat hubungan signifikan (p >= 0.05)",
        "mix_info": "Kombinasi belum didukung untuk analisis otomatis.",
        "wait_file": "Silakan upload file Excel data survei.",
        "profile_title": "Profil Pembuat",
        "about_title": "Tentang Aplikasi",
        "about_content": "Aplikasi ini dibuat menggunakan Streamlit untuk menganalisis data survei (Excel), analisis deskriptif, dan analisis hubungan variabel otomatis.",
        "final_corr_type": "Jenis Korelasi yang Digunakan",
        "final_corr_pearson": "Korelasi Pearson direkomendasikan karena asumsi linearitas dan normalitas terpenuhi (atau diasumsikan).",
        "final_corr_spearman": "Korelasi Spearman direkomendasikan karena data mungkin non-linear atau non-normal (korelasi rank).",
        "final_corr_note": "Catatan: Korelasi Pearson mengukur hubungan linear, Spearman mengukur hubungan monoton (berdasarkan rank).",
    },
    "English": {
        "title": "Survey Data Analysis App",
        "file": "Upload your survey Excel file",
        "analysis_title": "Data Analysis",
        "desc_title": "Data Distribution",
        "desc_cols": "Select numeric variables for descriptive analysis (histogram & boxplot)",
        "hist": "Histogram", "box": "Boxplot", "preview": "Preview Data",
        "vra_title": "Variable Relationship Analysis",
        "vra_var1": "Select Variable 1", "vra_var2": "Select Variable 2",
        "type_num": "Numeric", "type_cat": "Category",
        "cat_info": "Categorical variables (Category vs Category) → Chi-Square test",
        "num_info": "Numeric variables (Numeric vs Numeric) → Correlation (Pearson/Spearman)",
        "result_cat_cat": "Contingency Table (Chi-Square)",
        "result_num_num": "Correlation Results",
        "chi2": "Chi2 = {:.4f}", "pval": "P-value = {:.4f}", "dof": "Degrees of freedom = {}",
        "conclusion": "Conclusion:",
        "conclude_sig": "Significant relationship between variables (p < 0.05)",
        "conclude_nosig": "No significant relationship between variables (p >= 0.05)",
        "corr_pearson": "Pearson Correlation",
        "corr_spearman": "Spearman Correlation (Non-parametric)",
        "corr_coef": "Coefficient = {:.4f}", "corr_pval": "P-value = {:.4f}",
        "corr_conclude_sig": "Significant relationship (p < 0.05)",
        "corr_conclude_nosig": "No significant relationship (p >= 0.05)",
        "mix_info": "The combination is not supported yet.",
        "wait_file": "Please upload your Excel survey file.",
        "profile_title": "Author Profile",
        "about_title": "About App",
        "about_content": "This app is built using Streamlit to analyze survey data, descriptive analysis, and variable relationships automatically.",
        "final_corr_type": "Type of Correlation Used",
        "final_corr_pearson": "Pearson Correlation is recommended as linearity and normality assumptions are met (or assumed).",
        "final_corr_spearman": "Spearman Correlation is recommended as the data might be non-linear or non-normal (rank correlation).",
        "final_corr_note": "Note: Pearson correlation measures linear relationship, Spearman measures monotonic relationship (based on rank).",
    },
    # ... Tambahkan terjemahan untuk Bahasa Jepang dan Cina di sini jika diperlukan ...
    "日本語": {
        "title": "調査データ分析アプリ",
        "file": "Excel調査ファイルをアップロード",
        "analysis_title": "データ分析",
        "desc_title": "データ分布",
        "desc_cols": "記述分析用の数値変数を選択（ヒストグラム＆箱ひげ図）",
        "hist": "ヒストグラム", "box": "箱ひげ図", "preview": "データプレビュー",
        "vra_title": "変数の関係分析",
        "vra_var1": "変数1を選択", "vra_var2": "変数2を選択",
        "type_num": "数値型", "type_cat": "カテゴリ型",
        "cat_info": "カテゴリ型変数（カテゴリ vs カテゴリ） → カイ二乗検定",
        "num_info": "数値型変数（数値 vs 数値） → 相関分析（ピアソン/スピアマン）",
        "result_cat_cat": "クロス集計表（カイ二乗）",
        "result_num_num": "相関分析結果",
        "chi2": "カイ二乗 = {:.4f}", "pval": "p値 = {:.4f}", "dof": "自由度 = {}",
        "conclusion": "結論：",
        "conclude_sig": "変数間に有意な関係あり (p < 0.05)",
        "conclude_nosig": "変数間に有意な関係なし (p >= 0.05)",
        "corr_pearson": "ピアソン相関",
        "corr_spearman": "スピアマン相関（ノンパラメトリック）",
        "corr_coef": "相関係数 = {:.4f}", "corr_pval": "p値 = {:.4f}",
        "corr_conclude_sig": "有意な関係あり (p < 0.05)",
        "corr_conclude_nosig": "有意な関係なし (p >= 0.05)",
        "mix_info": "混合変数の自動分析は未対応です。",
        "wait_file": "調査ファイルをアップロードしてください。",
        "profile_title": "著者プロフィール",
        "about_title": "アプリについて",
        "about_content": "本アプリはStreamlitで作成され、調査データの自動分析が可能です。",
        "final_corr_type": "使用された相関の種類",
        "final_corr_pearson": "線形性と正規性の仮定が満たされている（または想定される）ため、ピアソン相関が推奨されます。",
        "final_corr_spearman": "データが非線形または非正規である可能性があるため、スピアマン相関が推奨されます（順位相関）。",
        "final_corr_note": "注：ピアソン相関は線形な関係を測定し、スピアマンは単調な関係（順位に基づく）を測定します。",
    },
    "简体中文": {
        "title": "调查数据分析应用",
        "file": "上传您的调查Excel文件",
        "analysis_title": "数据分析",
        "desc_title": "数据分布",
        "desc_cols": "选择用于描述性分析的数字变量（直方图与箱线图）",
        "hist": "直方图", "box": "箱线图", "preview": "数据预览",
        "vra_title": "变量关系分析",
        "vra_var1": "选择变量1", "vra_var2": "选择变量2",
        "type_num": "数字型", "type_cat": "分类型",
        "cat_info": "分类型变量（分类型 vs 分类型） → 卡方检验",
        "num_info": "数字型变量（数字型 vs 数字型） → 相关性（皮尔逊/斯皮尔曼）",
        "result_cat_cat": "列联表（卡方）",
        "result_num_num": "相关性结果",
        "chi2": "卡方值 = {:.4f}", "pval": "显著性水平（p值）= {:.4f}", "dof": "自由度 = {}",
        "conclusion": "结论：",
        "conclude_sig": "变量间存在显著关系 (p < 0.05)",
        "conclude_nosig": "变量间不存在显著关系 (p >= 0.05)",
        "corr_pearson": "皮尔逊相关性",
        "corr_spearman": "斯皮尔曼相关性（非参数）",
        "corr_coef": "相关系数 = {:.4f}", "corr_pval": "p值 = {:.4f}",
        "corr_conclude_sig": "存在显著关系 (p < 0.05)",
        "corr_conclude_nosig": "不存在显著关系 (p >= 0.05)",
        "mix_info": "混合变量暂不支持自动分析。",
        "wait_file": "请上传您的调查数据文件。",
        "profile_title": "作者简介",
        "about_title": "关于应用",
        "about_content": "本应用采用Streamlit开发，可自动分析调查数据、描述性分析与变量关系。",
        "final_corr_type": "使用的相关性类型",
        "final_corr_pearson": "由于满足（或假设满足）线性和正态性假设，推荐使用皮尔逊相关性。",
        "final_corr_spearman": "由于数据可能非线性或非正态（等级相关），推荐使用斯皮尔曼相关性。",
        "final_corr_note": "注意：皮尔逊相关性衡量线性关系，斯皮尔曼衡量单调关系（基于等级）。",
    }
}
tt = text.get(lang, text["Indonesia"])

profile_data = [
    {
        "name": {
            "Indonesia": "Moh. Trisbintang A. Menu",
            "English": "Moh. Trisbintang A. Menu",
            "日本語": "Moh. Trisbintang A. Menu",
            "简体中文": "Moh. Trisbintang A. Menu",
        },
        "img_file": "tris.jpeg",
        "sid": {
            "Indonesia": "SID: 004202400102",
            "English": "SID: 004202400102",
            "日本語": "SID: 004202400102",
            "简体中文": "SID：004202400102",
        },
        "role": {
            "Indonesia": "⚙️ Distribusi: Pengembangan web Streamlit (menu & navigasi), survei, pembersihan data, dan penulisan laporan",
            "English": "⚙️ Role: Streamlit web development (menu & navigation), survey, data cleaning, and report writing",
            "日本語": "⚙️ 役割：Streamlit Web開発（メニュー・ナビゲーション）、調査、データクリーニング、レポート作成",
            "简体中文": "⚙️ 职责：Streamlit网页开发（菜单与导航）、调查、数据清洗及报告撰写"
        },
        "origin": {
            "Indonesia": "Asal daerah: Gorontalo",
            "English": "Origin: Gorontalo",
            "日本語": "出身地：ゴロンタロ",
            "简体中文": "来自：Gorontalo",
        }
    },
    {
        "name": {
            "Indonesia": "Dwi Anfia Putri Wulandari",
            "English": "Dwi Anfia Putri Wulandari",
            "日本語": "Dwi Anfia Putri Wulandari",
            "简体中文": "Dwi Anfia Putri Wulandari",
        },
        "img_file": "fia.jpeg",
        "sid": {
            "Indonesia": "SID: 004202400034",
            "English": "SID: 004202400034",
            "日本語": "SID: 004202400034",
            "简体中文": "SID：004202400034",
        },
          "role": {
            "Indonesia": "⚙️ Distribusi: Pengembangan web Streamlit (menu & navigasi), survei, pembersihan data, dan penulisan laporan",
            "English": "⚙️ Role: Streamlit web development (menu & navigation), survey, data cleaning, and report writing",
            "日本語": "⚙️ 役割：Streamlit Web開発（メニュー・ナビゲーション）、調査、データクリーニング、レポート作成",
            "简体中文": "⚙️ 职责：Streamlit网页开发（菜单与导航）、调查、数据清洗及报告撰写"
        },
        "origin": {
            "Indonesia": "Asal daerah: Bogor",
            "English": "Origin: Bogor",
            "日本語": "出身地：ボゴール",
            "简体中文": "来自：Bogor",
        }
    },
    {
        "name": {
            "Indonesia": "Gina Sonia",
            "English": "Gina Sonia",
            "日本語": "Gina Sonia",
            "简体中文": "Gina Sonia",
        },
        "img_file": "gina.jpeg",
        "sid": {
            "Indonesia": "SID: 004202400076",
            "English": "SID: 004202400076",
            "日本語": "SID: 004202400076",
            "简体中文": "SID：004202400076",
        },
        "role": {
            "Indonesia": "⚙️ Distribusi: Pengembangan web Streamlit (menu & navigasi), survei, pembersihan data, dan penulisan laporan",
            "English": "⚙️ Role: Streamlit web development (menu & navigation), survey, data cleaning, and report writing",
            "日本語": "⚙️ 役割：Streamlit Web開発（メニュー・ナビゲーション）、調査、データクリーニング、レポート作成",
            "简体中文": "⚙️ 职责：Streamlit网页开发（菜单与导航）、调查、数据清洗及报告撰写"
        },
        "origin": {
            "Indonesia": "Asal daerah: Cikampek",
            "English": "Origin: Cikampek",
            "日本語": "出身地：チカンペック",
            "简体中文": "来自：Cikampek",
        }
    },
    {
        "name": {
            "Indonesia": "Ananda Fasya Wiratama Putri",
            "English": "Ananda Fasya Wiratama Putri",
            "日本語": "Ananda Fasya Wiratama Putri",
            "简体中文": "Ananda Fasya Wiratama Putri",
        },
        "img_file": "fasya.jpeg",
        "sid": {
            "Indonesia": "SID: 004202400107",
            "English": "SID: 004202400107",
            "日本語": "SID: 004202400107",
            "简体中文": "SID：004202400107",
        },
        "role": {
            "Indonesia": "⚙️ Distribusi: Pengembangan web Streamlit (menu & navigasi), survei, pembersihan data, dan penulisan laporan",
            "English": "⚙️ Role: Streamlit web development (menu & navigation), survey, data cleaning, and report writing",
            "日本語": "⚙️ 役割：Streamlit Web開発（メニュー・ナビゲーション）、調査、データクリーニング、レポート作成",
            "简体中文": "⚙️ 职责：Streamlit网页开发（菜单与导航）、调查、数据清洗及报告撰写"
        },
        "origin": {
            "Indonesia": "Asal daerah: Depok",
            "English": "Origin: Depok",
            "日本語": "出身地：デポック",
            "简体中文": "来自：Depok",
        }
    }
]

# --- MAIN CONTENT ---
if menu == menu_items[0]:
    st.markdown(f"<div class='stTitleMain'>{tt['profile_title']}</div>", unsafe_allow_html=True)
    for prof in profile_data:
        st.markdown("<div class='stCard'>", unsafe_allow_html=True)
        cols = st.columns([1,3])
        with cols[0]:
            # Pengecekan file gambar (asumsi gambar ada di folder yang sama)
            img_path = os.path.join(BASE_DIR, prof["img_file"])
            try:
                # Menggunakan io.BytesIO untuk membaca gambar dari path
                if os.path.exists(img_path):
                    with open(img_path, "rb") as f:
                        image_data = f.read()
                    st.image(image_data, width=265)
                else:
                    st.write("Image not found.")
            except Exception as e:
                st.error(f"Error loading image: {e}")
                st.write("Image placeholder")
        with cols[1]:
            st.markdown(f"<div class='stProfileName'>{prof['name'][lang]} ⚙️</div>", unsafe_allow_html=True)
            st.markdown(f"<div class='stProfileRole'>{prof['role'][lang]}</div>", unsafe_allow_html=True)
            st.markdown(f"**{prof['sid'][lang]}**")
            st.markdown(f"<span class='stOrigin'>{prof['origin'][lang]}</span>", unsafe_allow_html=True)
        st.markdown("</div>", unsafe_allow_html=True)
        st.markdown("<hr>", unsafe_allow_html=True)

elif menu == menu_items[1]:
    st.markdown(f"<div class='stTitleMain'>{tt['analysis_title']}</div>", unsafe_allow_html=True)
    uploaded_file = st.file_uploader(tt["file"], type=["xlsx"])
    df = None
    if uploaded_file:
        try:
            # Menggunakan io.BytesIO untuk menghindari masalah path
            df = pd.read_excel(io.BytesIO(uploaded_file.getvalue()))
        except Exception as e:
            st.error(f"Error reading file: {e}")
            df = None
            
    if df is not None:
        st.subheader(tt["preview"])
        st.markdown("<div class='stCard'>", unsafe_allow_html=True)
        st.dataframe(df)
        st.markdown("</div>", unsafe_allow_html=True)

        # --- Distribusi Data ---
        st.markdown(f"<div class='stSubHeader'>{tt['desc_title']}</div>", unsafe_allow_html=True)
        st.markdown("<div class='stCard'>", unsafe_allow_html=True)
        numeric_cols = df.select_dtypes(include=np.number).columns.tolist()
        selected_desc_cols = st.multiselect(tt["desc_cols"], numeric_cols)
        
        if selected_desc_cols:
            desc = df[selected_desc_cols].describe().T
            desc["skew"] = df[selected_desc_cols].skew()
            desc["kurtosis"] = df[selected_desc_cols].kurtosis()
            st.dataframe(desc)
            
            # Tentukan apakah akan menggunakan layout kolom untuk plot
            num_plots = len(selected_desc_cols)
            cols_per_row = 2
            
            for i in range(num_plots):
                col = selected_desc_cols[i]
                if i % cols_per_row == 0:
                    plot_cols = st.columns(cols_per_row)

                with plot_cols[i % cols_per_row]:
                    # Histogram
                    st.markdown(f"<span class='stLabel'>{tt['hist']}: {col}</span>", unsafe_allow_html=True)
                    fig1, ax1 = plt.subplots(figsize=(7,3))
                    ax1.hist(df[col].dropna(), bins=20, color="#1976d2", alpha=0.86)
                    # Ganti warna facecolor menjadi putih atau transparan untuk light mode
                    ax1.set_facecolor("#FAFAFA" if st.get_option("theme.base") == "light" else "#223a5e")
                    ax1.tick_params(colors="#223a5e" if st.get_option("theme.base") == "light" else "#FAFAFA")
                    ax1.spines['bottom'].set_color('#f7c325')
                    ax1.spines['left'].set_color('#f7c325')
                    ax1.title.set_color('#f7c325')
                    ax1.xaxis.label.set_color('#f7c325')
                    ax1.yaxis.label.set_color('#f7c325')
                    ax1.set_title(f"{tt['hist']}: {col}", fontsize=13, fontweight="bold", color='#22d2e9')
                    st.pyplot(fig1)
                    
                    # Boxplot
                    st.markdown(f"<span class='stLabel'>{tt['box']}: {col}</span>", unsafe_allow_html=True)
                    fig2, ax2 = plt.subplots(figsize=(7,3))
                    ax2.boxplot(df[col].dropna(), vert=False, patch_artist=True,
                                boxprops=dict(facecolor='#f7c325', color='#1976d2', linewidth=1.5),
                                medianprops=dict(color='#223a5e', linewidth=2))
                    # Ganti warna facecolor
                    ax2.set_facecolor("#FAFAFA" if st.get_option("theme.base") == "light" else "#223a5e")
                    ax2.tick_params(colors="#223a5e" if st.get_option("theme.base") == "light" else "#FAFAFA")
                    ax2.spines['bottom'].set_color('#f7c325')
                    ax2.spines['left'].set_color('#f7c325')
                    ax2.title.set_color('#f7c325')
                    ax2.xaxis.label.set_color('#f7c325')
                    ax2.yaxis.label.set_color('#f7c325')
                    ax2.set_title(f"{tt['box']}: {col}", fontsize=13, fontweight="bold", color='#22d2e9')
                    st.pyplot(fig2)
        else:
            st.info(tt["desc_cols"])
        st.markdown("</div>", unsafe_allow_html=True)

        # --- Analisis Hubungan Variabel ---
        st.markdown(f"<div class='stSubHeader'>{tt['vra_title']}</div>", unsafe_allow_html=True)
        st.markdown("<div class='stCard'>", unsafe_allow_html=True)
        colX1, colX2 = st.columns(2)
        all_cols = df.columns.tolist()
        with colX1:
            x1 = st.selectbox(tt["vra_var1"], all_cols)
        with colX2:
            x2_index = all_cols.index(x1) + 1 if x1 in all_cols and all_cols.index(x1) + 1 < len(all_cols) else 0
            x2 = st.selectbox(tt["vra_var2"], all_cols, index=x2_index)
            
        tipe_x1 = tt["type_num"] if np.issubdtype(df[x1].dropna().dtype, np.number) else tt["type_cat"]
        tipe_x2 = tt["type_num"] if np.issubdtype(df[x2].dropna().dtype, np.number) else tt["type_cat"]
        st.markdown(f"<span class='stLabel'>{x1} → {tipe_x1}</span>", unsafe_allow_html=True)
        st.markdown(f"<span class='stLabel'>{x2} → {tipe_x2}</span>", unsafe_allow_html=True)
        
        if tipe_x1 == tt["type_cat"] and tipe_x2 == tt["type_cat"]:
            # Kategorik vs Kategorik (Chi-Square)
            st.info(tt["cat_info"])
            cont_table = pd.crosstab(df[x1], df[x2])
            st.subheader(tt["result_cat_cat"])
            st.markdown("<div class='st-df'>", unsafe_allow_html=True)
            st.dataframe(cont_table)
            st.markdown("</div>", unsafe_allow_html=True)
            chi2, p, dof, expected = chi2_contingency(cont_table)
            st.write(tt["chi2"].format(chi2))
            st.write(tt["pval"].format(p))
            st.write(tt["dof"].format(dof))
            st.markdown(tt["conclusion"])
            if p < 0.05:
                st.success(tt["conclude_sig"])
            else:
                st.warning(tt["conclude_nosig"])
                
        elif tipe_x1 == tt["type_num"] and tipe_x2 == tt["type_num"]:
            # Numerik vs Numerik (Pearson dan Spearman)
            st.info(tt["num_info"])
            
            # Korelasi Pearson (Linear)
            coef_p, p_p = pearsonr(df[x1].dropna(), df[x2].dropna())
            # Korelasi Spearman (Rank/Monoton)
            coef_s, p_s = spearmanr(df[x1].dropna(), df[x2].dropna())
            
            st.subheader(tt["result_num_num"])
            col_p, col_s = st.columns(2)
            
            # Tampilkan Pearson
            with col_p:
                st.markdown(f"**{tt['corr_pearson']}**")
                st.markdown("<div class='st-df'>", unsafe_allow_html=True)
                st.write(tt["corr_coef"].format(coef_p))
                st.write(tt["corr_pval"].format(p_p))
                st.markdown("</div>", unsafe_allow_html=True)
                st.markdown(tt["conclusion"])
                if p_p < 0.05:
                    st.success(tt["corr_conclude_sig"])
                else:
                    st.warning(tt["corr_conclude_nosig"])

            # Tampilkan Spearman
            with col_s:
                st.markdown(f"**{tt['corr_spearman']}**")
                st.markdown("<div class='st-df'>", unsafe_allow_html=True)
                st.write(tt["corr_coef"].format(coef_s))
                st.write(tt["corr_pval"].format(p_s))
                st.markdown("</div>", unsafe_allow_html=True)
                st.markdown(tt["conclusion"])
                if p_s < 0.05:
                    st.success(tt["corr_conclude_sig"])
                else:
                    st.warning(tt["corr_conclude_nosig"])

            # Kesimpulan Akhir
            st.markdown("<hr>", unsafe_allow_html=True)
            st.markdown(f"**{tt['final_corr_type']}**")
            
            # Logic sederhana untuk menyimpulkan (bisa diperluas dengan uji normalitas)
            # Jika p-value Pearson lebih kecil dan koefisiennya serupa, asumsikan Pearson.
            # Jika tidak, atau jika data cenderung non-normal (misalnya skew tinggi), Spearman lebih aman.
            
            # Sederhana: Bandingkan koefisien/p-value yang paling kuat/signifikan
            if abs(coef_p) > abs(coef_s) and p_p < 0.05:
                st.markdown(f"**💡 {tt['corr_pearson']}**")
                st.info(tt["final_corr_pearson"])
            elif abs(coef_s) > abs(coef_p) and p_s < 0.05:
                st.markdown(f"**💡 {tt['corr_spearman']}**")
                st.info(tt["final_corr_spearman"])
            elif p_p < 0.05:
                 st.markdown(f"**💡 {tt['corr_pearson']}**")
                 st.info(tt["final_corr_pearson"])
            elif p_s < 0.05:
                st.markdown(f"**💡 {tt['corr_spearman']}**")
                st.info(tt["final_corr_spearman"])
            else:
                st.warning(tt["corr_conclude_nosig"])
            
            st.caption(tt["final_corr_note"])

            # Tambahkan Scatter Plot
            st.subheader("Scatter Plot")
            fig3, ax3 = plt.subplots(figsize=(7, 5))
            ax3.scatter(df[x1], df[x2], color="#f7c325", alpha=0.7)
            ax3.set_xlabel(x1, color="#22d2e9")
            ax3.set_ylabel(x2, color="#22d2e9")
            ax3.set_title(f"Scatter Plot: {x1} vs {x2}", color='#22d2e9', fontweight="bold")
            # Ganti warna facecolor
            ax3.set_facecolor("#FAFAFA" if st.get_option("theme.base") == "light" else "#223a5e")
            ax3.tick_params(colors="#223a5e" if st.get_option("theme.base") == "light" else "#FAFAFA")
            ax3.spines['bottom'].set_color('#f7c325')
            ax3.spines['left'].set_color('#f7c325')
            ax3.title.set_color('#f7c325')
            ax3.xaxis.label.set_color('#f7c325')
            ax3.yaxis.label.set_color('#f7c325')
            st.pyplot(fig3)
            
            
        else:
            st.warning(tt["mix_info"])
        st.markdown("</div>", unsafe_allow_html=True)
    else:
        st.info(tt["wait_file"])

elif menu == menu_items[2]:
    st.markdown(f"<div class='stTitleMain'>{tt['about_title']}</div>", unsafe_allow_html=True)
    st.markdown(f"<div class='stCard'>{tt['about_content']}</div>", unsafe_allow_html=True)
