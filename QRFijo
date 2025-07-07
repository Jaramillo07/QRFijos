import streamlit as st
import qrcode
from PIL import Image
import io
import zipfile
from datetime import datetime
import logging

# Configuración de logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def generate_optimized_qr(data: str):
    """Genera QR optimizado usando la lógica probada"""
    try:
        # Configuración optimizada basada en QRs funcionales
        qr = qrcode.QRCode(
            version=1,  # Versión baja para máxima legibilidad
            error_correction=qrcode.constants.ERROR_CORRECT_M,  # Corrección media
            box_size=12,   # Tamaño moderado
            border=4,      # Borde estándar
        )
        
        qr.add_data(data)
        try:
            qr.make(fit=False)  # Mantener versión 1
        except:
            # Si no cabe, permitir auto-ajuste
            qr.version = None
            qr.make(fit=True)
        
        # Generar imagen con máximo contraste
        img = qr.make_image(
            fill_color="#000000",   # Negro puro
            back_color="#FFFFFF",   # Blanco puro
        )
        
        # Convertir y optimizar tamaño
        from PIL import Image
        if not isinstance(img, Image.Image):
            img = img.convert('RGB')
        
        # Escalar a tamaño óptimo
        target_size = 400
        current_size = img.size[0]
        
        if current_size < target_size:
            scale_factor = target_size // current_size
            if scale_factor > 1:
                new_size = (current_size * scale_factor, current_size * scale_factor)
                img = img.resize(new_size, Image.NEAREST)
        
        logger.info(f"QR generado para: {data}")
        return img
        
    except Exception as e:
        logger.error(f"Error generando QR: {e}")
        return None

def qr_to_bytes(img):
    """Convierte QR a bytes para descarga"""
    try:
        if img is None:
            return None
        
        buf = io.BytesIO()
        
        if img.mode != 'RGB':
            img = img.convert('RGB')
        
        img.save(buf, format='PNG', optimize=False, compress_level=0)
        
        buf.seek(0)
        img_bytes = buf.getvalue()
        buf.close()
        
        return img_bytes
        
    except Exception as e:
        logger.error(f"Error convirtiendo QR: {e}")
        return None

def create_zip_file(qr_data_list):
    """Crea un archivo ZIP con múltiples QRs"""
    try:
        zip_buffer = io.BytesIO()
        
        with zipfile.ZipFile(zip_buffer, 'w', zipfile.ZIP_DEFLATED) as zip_file:
            for i, (qr_img, filename) in enumerate(qr_data_list):
                qr_bytes = qr_to_bytes(qr_img)
                if qr_bytes:
                    zip_file.writestr(filename, qr_bytes)
        
        zip_buffer.seek(0)
        return zip_buffer.getvalue()
        
    except Exception as e:
        logger.error(f"Error creando ZIP: {e}")
        return None

def main():
    """App principal para administrador"""
    st.set_page_config(
        page_title="Admin - Generador QR Colonos",
        page_icon="👨‍💼",
        layout="wide"
    )
    
    # Header
    st.title("👨‍💼 Administrador - Generador QR Colonos")
    st.markdown("**🏠 Fraccionamiento Los Girasoles**")
    st.markdown("---")
    
    # Información
    st.info("💡 **Genera códigos QR permanentes para colonos** - Estos códigos servirán como password en el Portal Colonos")
    
    # Formulario principal
    with st.form("qr_colono_form", clear_on_submit=True):
        st.subheader("📝 Información del Colono")
        
        col1, col2 = st.columns(2)
        
        with col1:
            nombre_colono = st.text_input(
                "👤 Nombre Completo del Colono:",
                placeholder="Ej: Jesus Jaramillo González",
                help="Nombre completo como aparecerá en el sistema"
            )
        
        with col2:
            domicilio = st.text_input(
                "🏠 Domicilio/Lote:",
                placeholder="Ej: Calle Girasol #203, Lote 15",
                help="Dirección o número de lote del colono"
            )
        
        # Opciones de generación
        st.markdown("**⚙️ Opciones de Generación:**")
        
        col1, col2, col3 = st.columns(3)
        
        with col1:
            cantidad_qrs = st.selectbox(
                "📊 Cantidad de QRs a generar:",
                options=[1, 2, 3, 4, 5],
                index=0,
                help="Útil para familias grandes o múltiples usuarios"
            )
        
        with col2:
            prefijo_codigo = st.text_input(
                "🔤 Prefijo del código:",
                value="girasol",
                help="Prefijo común para todos los códigos"
            )
        
        with col3:
            incluir_numero = st.checkbox(
                "🔢 Incluir número de lote",
                value=True,
                help="Agregar número de lote al código QR"
            )
        
        # Botón generar
        col1, col2, col3 = st.columns([1, 1, 1])
        with col2:
            generar_btn = st.form_submit_button(
                "🎫 Generar QRs",
                type="primary",
                use_container_width=True
            )
        
        # Procesar formulario
        if generar_btn:
            if not nombre_colono.strip():
                st.error("❌ Debe ingresar el nombre del colono")
            elif not domicilio.strip():
                st.error("❌ Debe ingresar el domicilio")
            else:
                with st.spinner(f"Generando {cantidad_qrs} QR(s) para {nombre_colono}..."):
                    
                    # Limpiar datos
                    nombre_clean = nombre_colono.strip()
                    domicilio_clean = domicilio.strip()
                    
                    # Generar códigos QR
                    qr_results = []
                    
                    for i in range(cantidad_qrs):
                        # Crear código único
                        numero_qr = i + 1
                        
                        # Formato del código: prefijo + nombre + número
                        codigo_base = f"{prefijo_codigo}{nombre_clean.lower().replace(' ', '')}"
                        
                        if incluir_numero:
                            # Extraer número de lote si existe
                            import re
                            numeros = re.findall(r'\d+', domicilio_clean)
                            numero_lote = numeros[0] if numeros else str(numero_qr)
                            codigo_final = f"{codigo_base}{numero_lote}"
                        else:
                            codigo_final = f"{codigo_base}{numero_qr}"
                        
                        # Generar QR
                        qr_img = generate_optimized_qr(codigo_final)
                        
                        if qr_img:
                            filename = f"QR_{nombre_clean.replace(' ', '_')}_{numero_qr}.png"
                            qr_results.append({
                                'numero': numero_qr,
                                'codigo': codigo_final,
                                'imagen': qr_img,
                                'filename': filename
                            })
                    
                    if qr_results:
                        st.success(f"✅ {len(qr_results)} QR(s) generados exitosamente para {nombre_clean}")
                        
                        # Guardar en session state para mostrar
                        st.session_state.qr_generados = qr_results
                        st.session_state.colono_info = {
                            'nombre': nombre_clean,
                            'domicilio': domicilio_clean,
                            'cantidad': cantidad_qrs
                        }
                    else:
                        st.error("❌ Error generando los códigos QR")
    
    # Mostrar QRs generados
    if 'qr_generados' in st.session_state and st.session_state.qr_generados:
        st.markdown("---")
        st.subheader("🎯 QRs Generados")
        
        colono_info = st.session_state.colono_info
        st.markdown(f"**👤 Colono:** {colono_info['nombre']}")
        st.markdown(f"**🏠 Domicilio:** {colono_info['domicilio']}")
        st.markdown(f"**📊 Cantidad:** {colono_info['cantidad']} QR(s)")
        
        # Mostrar QRs en grid
        qr_results = st.session_state.qr_generados
        
        # Organizar en columnas según cantidad
        if len(qr_results) == 1:
            cols = st.columns([1, 2, 1])
            col_indices = [1]
        elif len(qr_results) == 2:
            cols = st.columns(2)
            col_indices = [0, 1]
        elif len(qr_results) <= 3:
            cols = st.columns(3)
            col_indices = list(range(len(qr_results)))
        else:
            # Para 4-5 QRs, usar filas
            cols = st.columns(3)
            col_indices = list(range(min(3, len(qr_results))))
        
        # Mostrar primera fila
        for i, col_idx in enumerate(col_indices):
            if i < len(qr_results):
                qr_data = qr_results[i]
                with cols[col_idx]:
                    st.markdown(f"**QR #{qr_data['numero']}**")
                    st.image(qr_data['imagen'], width=250)
                    st.code(qr_data['codigo'], language=None)
                    
                    # Botón descarga individual
                    qr_bytes = qr_to_bytes(qr_data['imagen'])
                    if qr_bytes:
                        st.download_button(
                            f"📥 Descargar QR #{qr_data['numero']}",
                            data=qr_bytes,
                            file_name=qr_data['filename'],
                            mime="image/png",
                            key=f"download_{i}",
                            use_container_width=True
                        )
        
        # Segunda fila si hay más de 3 QRs
        if len(qr_results) > 3:
            st.markdown("---")
            cols2 = st.columns(len(qr_results) - 3)
            for i, qr_data in enumerate(qr_results[3:]):
                with cols2[i]:
                    st.markdown(f"**QR #{qr_data['numero']}**")
                    st.image(qr_data['imagen'], width=250)
                    st.code(qr_data['codigo'], language=None)
                    
                    qr_bytes = qr_to_bytes(qr_data['imagen'])
                    if qr_bytes:
                        st.download_button(
                            f"📥 Descargar QR #{qr_data['numero']}",
                            data=qr_bytes,
                            file_name=qr_data['filename'],
                            mime="image/png",
                            key=f"download_{i+3}",
                            use_container_width=True
                        )
        
        # Descarga masiva
        st.markdown("---")
        col1, col2, col3 = st.columns([1, 1, 1])
        
        with col2:
            if len(qr_results) > 1:
                # Crear ZIP con todos los QRs
                qr_zip_data = [(qr['imagen'], qr['filename']) for qr in qr_results]
                zip_bytes = create_zip_file(qr_zip_data)
                
                if zip_bytes:
                    timestamp = datetime.now().strftime("%Y%m%d_%H%M")
                    zip_filename = f"QRs_{colono_info['nombre'].replace(' ', '_')}_{timestamp}.zip"
                    
                    st.download_button(
                        "📦 Descargar Todos los QRs (ZIP)",
                        data=zip_bytes,
                        file_name=zip_filename,
                        mime="application/zip",
                        type="primary",
                        use_container_width=True
                    )
        
        # Información de uso
        st.markdown("---")
        st.info("""
        📋 **Instrucciones para el colono:**
        1. 📱 El colono puede usar **cualquiera** de estos códigos QR como password
        2. 🔑 En el Portal Colonos, ingresa su **nombre completo** + **uno de estos códigos**
        3. 💾 **Guarda estos códigos** de forma segura para el colono
        4. 🏠 Un colono puede tener múltiples códigos para familia/invitados
        """)
        
        # Botón limpiar
        if st.button("🗑️ Limpiar y Generar Nuevo", key="clear_results"):
            del st.session_state.qr_generados
            del st.session_state.colono_info
            st.rerun()
    
    # Footer
    st.markdown("---")
    st.markdown(
        "<div style='text-align: center; color: #666;'>"
        "👨‍💼 Administrador - Generador QR Colonos | 🏠 Fraccionamiento Los Girasoles<br>"
        f"📅 {datetime.now().strftime('%d/%m/%Y %H:%M')}"
        "</div>",
        unsafe_allow_html=True
    )

if __name__ == "__main__":
    main()
