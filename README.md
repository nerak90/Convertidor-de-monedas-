import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.text.DecimalFormat;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.SwingWorker;
import com.google.gson.JsonElement;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class ConvertidorMonedas extends JFrame {
    // API Key para ExchangeRate-API
    private static final String API_KEY = "edbc2e3c248b95bf6d637f8bb6";
    private static final String API_URL = "https://v6.exchangerate-api.com/v6/" + API_KEY + "/latest/";
    
    private JTextField campoMonto;
    private JComboBox<String> comboMonedaOrigen;
    private JComboBox<String> comboMonedaDestino;
    private JLabel labelResultado;
    private JButton botonConvertir;
    private JButton botonIntercambiar;
    private JButton botonActualizar;
    private JLabel labelUltimaActualizacion;
    private JProgressBar barraProgreso;
    
    // Mapa con las tasas de cambio actuales
    private Map<String, Double> tasasCambio;
    private DecimalFormat formatoDecimal;
    private LocalDateTime ultimaActualizacion;
    
    // Monedas principales disponibles
    private String[] monedasPrincipales = {
        "USD", "EUR", "GBP", "JPY", "CAD", "AUD", "CHF", "CNY", 
        "MXN", "BRL", "ARS", "CLP", "COP", "PEN", "UYU", "INR",
        "KRW", "SGD", "HKD", "NOK", "SEK", "DKK", "PLN", "CZK"
    };
    
    public ConvertidorMonedas() {
        inicializarTasasCambioDefault();
        inicializarInterfaz();
        configurarEventos();
        // Cargar tasas reales al iniciar
        actualizarTasasCambio();
    }
    
    private void inicializarTasasCambioDefault() {
        tasasCambio = new HashMap<>();
        // Tasas por defecto mientras se cargan las reales
        tasasCambio.put("USD", 1.0);
        tasasCambio.put("EUR", 0.92);
        tasasCambio.put("GBP", 0.79);
        tasasCambio.put("JPY", 149.50);
        tasasCambio.put("CAD", 1.36);
        tasasCambio.put("AUD", 1.53);
        tasasCambio.put("CHF", 0.88);
        tasasCambio.put("CNY", 7.24);
        tasasCambio.put("MXN", 17.12);
        tasasCambio.put("BRL", 5.03);
        tasasCambio.put("ARS", 350.00);
        tasasCambio.put("CLP", 890.00);
        
        formatoDecimal = new DecimalFormat("#,##0.00");
        ultimaActualizacion = LocalDateTime.now().minusHours(1); // Simular datos antiguos
    }
    
    private void inicializarInterfaz() {
        setTitle("Convertidor de Monedas - Tiempo Real");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());
        setResizable(false);
        
        // Panel principal
        JPanel panelPrincipal = new JPanel(new GridBagLayout());
        panelPrincipal.setBorder(BorderFactory.createEmptyBorder(20, 20, 20, 20));
        GridBagConstraints gbc = new GridBagConstraints();
        
        // Título
        JLabel titulo = new JLabel("Convertidor de Monedas", SwingConstants.CENTER);
        titulo.setFont(new Font("Arial", Font.BOLD, 20));
        titulo.setForeground(new Color(51, 51, 51));
        gbc.gridx = 0; gbc.gridy = 0; gbc.gridwidth = 4;
        gbc.insets = new Insets(0, 0, 15, 0);
        panelPrincipal.add(titulo, gbc);
        
        // Subtítulo con API info
        JLabel subtitulo = new JLabel("Tasas de cambio en tiempo real", SwingConstants.CENTER);
        subtitulo.setFont(new Font("Arial", Font.ITALIC, 12));
        subtitulo.setForeground(new Color(102, 102, 102));
        gbc.gridy = 1;
        gbc.insets = new Insets(0, 0, 20, 0);
        panelPrincipal.add(subtitulo, gbc);
        
        // Monto
        gbc.gridwidth = 1;
        gbc.insets = new Insets(5, 5, 5, 5);
        gbc.gridx = 0; gbc.gridy = 2;
        panelPrincipal.add(new JLabel("Monto:"), gbc);
        
        campoMonto = new JTextField(15);
        campoMonto.setFont(new Font("Arial", Font.PLAIN, 14));
        gbc.gridx = 1; gbc.gridwidth = 3;
        panelPrincipal.add(campoMonto, gbc);
        
        // Moneda de origen
        gbc.gridwidth = 1;
        gbc.gridx = 0; gbc.gridy = 3;
        panelPrincipal.add(new JLabel("De:"), gbc);
        
        comboMonedaOrigen = new JComboBox<>(monedasPrincipales);
        comboMonedaOrigen.setSelectedItem("USD");
        gbc.gridx = 1;
        panelPrincipal.add(comboMonedaOrigen, gbc);
        
        // Botón intercambiar
        botonIntercambiar = new JButton("⇄");
        botonIntercambiar.setFont(new Font("Arial", Font.BOLD, 16));
        botonIntercambiar.setPreferredSize(new Dimension(40, 25));
        botonIntercambiar.setToolTipText("Intercambiar monedas");
        gbc.gridx = 2;
        panelPrincipal.add(botonIntercambiar, gbc);
        
        // Botón actualizar
        botonActualizar = new JButton("🔄");
        botonActualizar.setFont(new Font("Arial", Font.BOLD, 14));
        botonActualizar.setPreferredSize(new Dimension(40, 25));
        botonActualizar.setToolTipText("Actualizar tasas de cambio");
        gbc.gridx = 3;
        panelPrincipal.add(botonActualizar, gbc);
        
        // Moneda de destino
        gbc.gridx = 0; gbc.gridy = 4;
        panelPrincipal.add(new JLabel("A:"), gbc);
        
        comboMonedaDestino = new JComboBox<>(monedasPrincipales);
        comboMonedaDestino.setSelectedItem("EUR");
        gbc.gridx = 1; gbc.gridwidth = 3;
        panelPrincipal.add(comboMonedaDestino, gbc);
        
        // Barra de progreso
        barraProgreso = new JProgressBar();
        barraProgreso.setIndeterminate(false);
        barraProgreso.setVisible(false);
        gbc.gridx = 0; gbc.gridy = 5; gbc.gridwidth = 4;
        gbc.fill = GridBagConstraints.HORIZONTAL;
        gbc.insets = new Insets(10, 5, 5, 5);
        panelPrincipal.add(barraProgreso, gbc);
        
        // Botón convertir
        botonConvertir = new JButton("Convertir");
        botonConvertir.setFont(new Font("Arial", Font.BOLD, 14));
        botonConvertir.setBackground(new Color(76, 175, 80));
        botonConvertir.setForeground(Color.WHITE);
        botonConvertir.setFocusPainted(false);
        gbc.gridy = 6;
        gbc.insets = new Insets(10, 5, 10, 5);
        panelPrincipal.add(botonConvertir, gbc);
        
        // Resultado
        labelResultado = new JLabel("Ingrese un monto para convertir", SwingConstants.CENTER);
        labelResultado.setFont(new Font("Arial", Font.BOLD, 16));
        labelResultado.setForeground(new Color(51, 51, 51));
        labelResultado.setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(Color.LIGHT_GRAY),
            BorderFactory.createEmptyBorder(15, 10, 15, 10)
        ));
        gbc.gridy = 7;
        gbc.insets = new Insets(10, 5, 10, 5);
        panelPrincipal.add(labelResultado, gbc);
        
        // Última actualización
        labelUltimaActualizacion = new JLabel("", SwingConstants.CENTER);
        labelUltimaActualizacion.setFont(new Font("Arial", Font.ITALIC, 10));
        labelUltimaActualizacion.setForeground(Color.GRAY);
        gbc.gridy = 8;
        gbc.insets = new Insets(5, 5, 0, 5);
        panelPrincipal.add(labelUltimaActualizacion, gbc);
        
        add(panelPrincipal, BorderLayout.CENTER);
        
        // Panel inferior con información
        JLabel infoLabel = new JLabel("Tasas proporcionadas por ExchangeRate-API", SwingConstants.CENTER);
        infoLabel.setFont(new Font("Arial", Font.ITALIC, 10));
        infoLabel.setForeground(Color.GRAY);
        infoLabel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));
        add(infoLabel, BorderLayout.SOUTH);
        
        pack();
        setLocationRelativeTo(null);
        actualizarLabelTiempo();
    }
    
    private void configurarEventos() {
        botonConvertir.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                realizarConversion();
            }
        });
        
        botonIntercambiar.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                intercambiarMonedas();
            }
        });
        
        botonActualizar.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                actualizarTasasCambio();
            }
        });
        
        // Permitir conversión al presionar Enter
        campoMonto.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                realizarConversion();
            }
        });
    }
    
    private void actualizarTasasCambio() {
        // Usar SwingWorker para operación en segundo plano
        SwingWorker<Void, Void> worker = new SwingWorker<Void, Void>() {
            @Override
            protected void doInBackground() {
                return null;
            }
            
            @Override
            protected void done() {
                obtenerTasasDesdeAPI();
            }
        };
        
        mostrarCargando(true);
        worker.execute();
    }
    
    private void obtenerTasasDesdeAPI() {
        SwingWorker<Map<String, Double>, Void> apiWorker = new SwingWorker<Map<String, Double>, Void>() {
            @Override
            protected Map<String, Double> doInBackground() throws Exception {
                Map<String, Double> nuevasTasas = new HashMap<>();
                
                try {
                    // Hacer petición a la API
                    String urlStr = API_URL + "USD";
                    URL url = new URL(urlStr);
                    HttpURLConnection request = (HttpURLConnection) url.openConnection();
                    request.setConnectTimeout(10000); // 10 segundos timeout
                    request.setReadTimeout(10000);
                    request.connect();
                    
                    if (request.getResponseCode() == 200) {
                        // Parsear respuesta JSON
                        JsonParser jp = new JsonParser();
                        JsonElement root = jp.parse(new InputStreamReader(request.getInputStream()));
                        JsonObject jsonobj = root.getAsJsonObject();
                        
                        String result = jsonobj.get("result").getAsString();
                        if ("success".equals(result)) {
                            JsonObject conversionRates = jsonobj.getAsJsonObject("conversion_rates");
                            
                            // Extraer tasas para las monedas que nos interesan
                            for (String moneda : monedasPrincipales) {
                                if (conversionRates.has(moneda)) {
                                    nuevasTasas.put(moneda, conversionRates.get(moneda).getAsDouble());
                                }
                            }
                        } else {
                            throw new Exception("Error en respuesta API: " + result);
                        }
                    } else {
                        throw new Exception("Error HTTP: " + request.getResponseCode());
                    }
                    
                    request.disconnect();
                } catch (Exception e) {
                    throw new Exception("Error conectando con la API: " + e.getMessage());
                }
                
                return nuevasTasas;
            }
            
            @Override
            protected void done() {
                mostrarCargando(false);
                try {
                    Map<String, Double> nuevasTasas = get();
                    if (!nuevasTasas.isEmpty()) {
                        tasasCambio.putAll(nuevasTasas);
                        ultimaActualizacion = LocalDateTime.now();
                        actualizarLabelTiempo();
                        mostrarMensaje("Tasas actualizadas correctamente", false);
                    }
                } catch (Exception e) {
                    mostrarError("No se pudieron actualizar las tasas: " + e.getMessage() + 
                                "\nUsando tasas locales.");
                }
            }
        };
        
        apiWorker.execute();
    }
    
    private void realizarConversion() {
        try {
            String textoMonto = campoMonto.getText().trim();
            if (textoMonto.isEmpty()) {
                mostrarError("Por favor, ingrese un monto");
                return;
            }
            
            double monto = Double.parseDouble(textoMonto);
            if (monto < 0) {
                mostrarError("El monto debe ser positivo");
                return;
            }
            
            String monedaOrigen = (String) comboMonedaOrigen.getSelectedItem();
            String monedaDestino = (String) comboMonedaDestino.getSelectedItem();
            
            if (!tasasCambio.containsKey(monedaOrigen) || !tasasCambio.containsKey(monedaDestino)) {
                mostrarError("Moneda no disponible. Actualizando tasas...");
                actualizarTasasCambio();
                return;
            }
            
            double resultado = convertirMoneda(monto, monedaOrigen, monedaDestino);
            
            String textoResultado = String.format("%s %s = %s %s", 
                formatoDecimal.format(monto), monedaOrigen,
                formatoDecimal.format(resultado), monedaDestino);
            
            labelResultado.setText(textoResultado);
            labelResultado.setForeground(new Color(76, 175, 80));
            
            // Mostrar tasa de cambio
            if (!monedaOrigen.equals(monedaDestino)) {
                double tasa = resultado / monto;
                String infoTasa = String.format("1 %s = %s %s", 
                    monedaOrigen, formatoDecimal.format(tasa), monedaDestino);
                labelResultado.setToolTipText(infoTasa);
            }
            
        } catch (NumberFormatException ex) {
            mostrarError("Ingrese un número válido");
        } catch (Exception ex) {
            mostrarError("Error en la conversión: " + ex.getMessage());
        }
    }
    
    private double convertirMoneda(double monto, String monedaOrigen, String monedaDestino) {
        if (monedaOrigen.equals(monedaDestino)) {
            return monto;
        }
        
        // Convertir a USD primero (todas las tasas están en base USD)
        double montoEnUSD = monto / tasasCambio.get(monedaOrigen);
        
        // Luego convertir de USD a la moneda destino
        return montoEnUSD * tasasCambio.get(monedaDestino);
    }
    
    private void intercambiarMonedas() {
        Object temp = comboMonedaOrigen.getSelectedItem();
        comboMonedaOrigen.setSelectedItem(comboMonedaDestino.getSelectedItem());
        comboMonedaDestino.setSelectedItem(temp);
        
        // Si hay un resultado, recalcular
        if (!campoMonto.getText().trim().isEmpty()) {
            realizarConversion();
        }
    }
    
    private void mostrarCargando(boolean mostrar) {
        barraProgreso.setVisible(mostrar);
        barraProgreso.setIndeterminate(mostrar);
        botonActualizar.setEnabled(!mostrar);
        botonConvertir.setEnabled(!mostrar);
        pack(); // Reajustar tamaño de ventana
    }
    
    private void actualizarLabelTiempo() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
        String tiempo = ultimaActualizacion.format(formatter);
        labelUltimaActualizacion.setText("Última actualización: " + tiempo);
    }
    
    private void mostrarMensaje(String mensaje, boolean esError) {
        if (esError) {
            labelResultado.setText(mensaje);
            labelResultado.setForeground(Color.RED);
        } else {
            // Mostrar mensaje temporal en la barra de estado
            Timer timer = new Timer(3000, e -> actualizarLabelTiempo());
            timer.setRepeats(false);
            labelUltimaActualizacion.setText(mensaje);
            timer.start();
        }
    }
    
    private void mostrarError(String mensaje) {
        labelResultado.setText(mensaje);
        labelResultado.setForeground(Color.RED);
        JOptionPane.showMessageDialog(this, mensaje, "Error", JOptionPane.ERROR_MESSAGE);
    }
    
    public static void main(String[] args) {
        // Establecer el look and feel del sistema
        try {
            UIManager.setLookAndFeel(UIManager.getSystemLookAndFeel());
        } catch (Exception e) {
            System.out.println("No se pudo establecer el look and feel del sistema");
        }
        
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new ConvertidorMonedas().setVisible(true);
            }
        });
    }
}
