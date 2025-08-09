# 💱 Convertidor de Monedas en Java

🌟 Características

- ✅ **Tasas en tiempo real** usando ExchangeRate-API
- ✅ **24+ monedas** principales del mundo
- ✅ **Interfaz gráfica intuitiva** con Swing
- ✅ **Conversión bidireccional** con botón de intercambio
- ✅ **Actualización automática** de tasas de cambio
- ✅ **Validación de entrada** y manejo de errores
- ✅ **Operaciones asíncronas** que no bloquean la UI
- ✅ **Formato de números** con separadores de miles

## 🚀 Inicio Rápido

 Pre- requisitos
- Java 8 o superior
- Librería GSON (incluida en `/lib`)
- Clave API de ExchangeRate-API (gratuita)

📦 Instalación

1. **Clona el repositorio:**
   ```bash
   git clone https://github.com/TU-USUARIO/convertidor-monedas-java.git
   cd convertidor-monedas-java
   ```

2. **Configura tu API Key:**
   - Regístrate en [ExchangeRate-API](https://www.exchangerate-api.com/)
   - Reemplaza `"YOUR-API-KEY"` en el código con tu clave real

3. **Compila y ejecuta:**

   **En Windows:**
   ```batch
   scripts/compile.bat
   ```

    En Linux/Mac:**
   ```bash
   chmod +x scripts/compile.sh
   ./scripts/compile.sh
   ```

   🧐 Manualmente: 
   ```bash
   # Compilar
   javac -cp "lib/gson-2.10.1.jar:." src/ConvertidorMonedas.java -d build/
   
   # Ejecutar
   java -cp "lib/gson-2.10.1.jar:build" ConvertidorMonedas
   ```

 🛠️ Uso

1. **Ingresa el monto** a convertir
2. **Selecciona las monedas** de origen y destino
3. **Haz clic en "Convertir"** o presiona Enter
4. **Usa el botón ⇄** para intercambiar monedas
5. **Actualiza tasas** con el botón 🔄

💲💲  Monedas Soportadas
```
USD, EUR, GBP, JPY, CAD, AUD, CHF, CNY, MXN, BRL, 
ARS, CLP, COP, PEN, UYU, INR, KRW, SGD, HKD, NOK, 
SEK, DKK, PLN, CZK
```

 🔧 Configuración Avanzada

### Cambiar la API Key
Edita la constante en `src/ConvertidorMonedas.java`:
```java
private static final String API_KEY = "tu-nueva-api-key";
```

### Añadir más monedas
Modifica el array `monedasPrincipales[]` con los códigos ISO 4217 deseados.

## 🏗️ Arquitectura del Proyecto

```
src/ConvertidorMonedas.java    # Clase principal con GUI y lógica
lib/gson-2.10.1.jar           # Librería para parsing JSON
scripts/                      # Scripts de compilación
docs/                         # Documentación y capturas
```

### Componentes Principales
- **GUI**: Interfaz Swing con GridBagLayout
- **API Client**: Cliente HTTP para ExchangeRate-API
- **Threading**: SwingWorker para operaciones asíncronas
- **Error Handling**: Manejo robusto de excepciones

## 🤝 Contribuir

¡Las contribuciones son bienvenidas! Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

 Ideas para contribuir:
- [ ] Añadir más monedas
- [ ] Implementar historial de conversiones
- [ ] Añadir gráficos de tendencias
- [ ] Soporte para múltiples APIs
- [ ] Modo oscuro
- [ ] Localización i18n

 v1.1.0 (Actual)
- ✅ Integración con ExchangeRate-API
- ✅ Tasas de cambio en tiempo real
- ✅ Interfaz mejorada con indicadores de carga

 📄 Licencia

Este proyecto está bajo la Licencia MIT - ver el archivo [LICENSE](LICENSE) para detalles.


⭐ ¡Dale una estrella!

Si este proyecto te fue útil, ¡no olvides darle una ⭐ en GitHub!

---

 Desarrollado con ❤️ y ☕ por Karen Romero Díaz 
