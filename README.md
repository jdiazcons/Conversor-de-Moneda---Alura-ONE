# Conversor-de-Moneda---Alura-ONE V1
*// Falta Key de API
*// Completar listado de tabla monedas manual

Funciones del Conversor de Monedas:
1.-Cargar monedas soportadas:

 - Obtiene las monedas soportadas desde la API y las combina con una lista local predefinida que incluye nombres descriptivos.

2.- Mostrar monedas soportadas:

 - Presenta una lista completa de monedas disponibles con su código y nombre descriptivo.
 - Ejemplo: USD - Dólar estadounidense.

3.- Realizar una conversión:

 - Solicita al usuario:
     Moneda base (código como USD).
     Moneda objetivo (código como EUR).
     Cantidad a convertir.
 - Obtiene la tasa de cambio de la API y realiza el cálculo de la conversión.
 - Muestra el tipo de cambio, la cantidad convertida y los nombres de las monedas.
 - Guarda un registro de la conversión con una marca de tiempo.
 
  4.- Registrar historial de conversiones:

 - Guarda cada conversión en un archivo log.txt con:
    Fecha y hora.
    Moneda base, moneda objetivo.
    Cantidad original y cantidad convertida.

5.- Ver historial de conversiones:

 - Lee y muestra el historial de conversiones desde el archivo log.txt.
 - Si no existe un historial, informa al usuario que no hay registros disponibles.

6.- Validación de monedas:

 - Verifica que las monedas base y objetivo ingresadas estén en la lista de monedas soportadas.

7.- Manejo de errores:

 - Controla errores relacionados con la conexión a la API, moneda no soportada o problemas al leer/escribir en el archivo.

8.-Cargar nombres descriptivos de monedas:

 - Incluye nombres como "Dólar estadounidense", "Euro", "Yen japonés" para hacer la interacción más intuitiva.

9.- Opciones del menú principal:

 - Menú interactivo con las siguientes opciones:
    Ver monedas soportadas.
    Realizar una conversión.
    Ver historial de conversiones.
    Salir del programa.

10.- Compatibilidad con nuevas monedas:

 - Admite la adición de monedas automáticamente desde la API al cargar las tasas de cambio.

Ventajas:
 - Interfaz amigable: La lista descriptiva facilita la selección de monedas.
 - Historial persistente: Los registros se conservan para consultas futuras.
 - Soporte flexible: Se adapta fácilmente para incluir más monedas o funcionalidades adicionales.

Code

    import java.io.*;
    import java.net.HttpURLConnection;
    import java.net.URL;
    import java.time.LocalDateTime;
    import java.time.format.DateTimeFormatter;
    import java.util.*;
    import com.google.gson.JsonObject;
    import com.google.gson.JsonParser;

    public class CurrencyConverter {

    private static final String API_URL = "https://v6.exchangerate-api.com/v6/tuAPIKey/latest/";
    private static final String LOG_FILE = "log.txt";
    private static Map<String, String> supportedCurrencies = new HashMap<>();

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        boolean exit = false;

        // Inicializar la lista de monedas con nombres descriptivos
        initializeCurrencyDescriptions();

        // Cargar monedas soportadas al inicio
        loadSupportedCurrencies();

        while (!exit) {
            System.out.println("\n=== Conversor de Monedas ===");
            System.out.println("1. Ver monedas soportadas");
            System.out.println("2. Realizar una conversión");
            System.out.println("3. Ver historial de conversiones");
            System.out.println("4. Salir");
            System.out.print("Elige una opción: ");
            int choice = scanner.nextInt();

            switch (choice) {
                case 1:
                    displaySupportedCurrencies();
                    break;
                case 2:
                    performConversion(scanner);
                    break;
                case 3:
                    viewConversionHistory();
                    break;
                case 4:
                    exit = true;
                    System.out.println("¡Gracias por usar el conversor de monedas!");
                    break;
                default:
                    System.out.println("Opción no válida. Intenta de nuevo.");
            }
        }

        scanner.close();
    }

    private static void initializeCurrencyDescriptions() {
        // Mapa con descripciones de monedas
        supportedCurrencies.put("USD", "Dólar estadounidense");
        supportedCurrencies.put("EUR", "Euro");
        supportedCurrencies.put("JPY", "Yen japonés");
        supportedCurrencies.put("GBP", "Libra esterlina");
        supportedCurrencies.put("AUD", "Dólar australiano");
        supportedCurrencies.put("CAD", "Dólar canadiense");
        supportedCurrencies.put("CHF", "Franco suizo");
        supportedCurrencies.put("CNY", "Yuan chino");
        supportedCurrencies.put("MXN", "Peso mexicano");
        // Puedes agregar más monedas según sea necesario
    }

    private static void loadSupportedCurrencies() {
        try {
            URL url = new URL(API_URL + "USD"); // Moneda base temporal para obtener la lista
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");

            if (connection.getResponseCode() != 200) {
                throw new RuntimeException("Error al conectar con la API: " + connection.getResponseCode());
            }

            BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            JsonObject jsonResponse = JsonParser.parseReader(reader).getAsJsonObject();
            reader.close();

            JsonObject rates = jsonResponse.getAsJsonObject("conversion_rates");
            for (String currencyCode : rates.keySet()) {
                // Solo agregar monedas que no estén en el mapa
                supportedCurrencies.putIfAbsent(currencyCode, currencyCode); // Código si no hay descripción
            }

            System.out.println("Monedas soportadas cargadas con éxito.");
        } catch (Exception e) {
            System.out.println("Error al cargar las monedas soportadas: " + e.getMessage());
        }
    }

    private static void displaySupportedCurrencies() {
        System.out.println("\n=== Monedas Soportadas ===");
        for (Map.Entry<String, String> entry : supportedCurrencies.entrySet()) {
            System.out.printf("%s - %s%n", entry.getKey(), entry.getValue());
        }
    }

    private static void performConversion(Scanner scanner) {
        scanner.nextLine(); // Limpiar el buffer
        System.out.print("\nIntroduce la moneda base (por ejemplo, USD): ");
        String baseCurrency = scanner.nextLine().toUpperCase();

        if (!supportedCurrencies.containsKey(baseCurrency)) {
            System.out.println("La moneda base no es válida. Usa la opción 1 para ver las monedas soportadas.");
            return;
        }

        System.out.print("Introduce la moneda a la que deseas convertir (por ejemplo, EUR): ");
        String targetCurrency = scanner.nextLine().toUpperCase();

        if (!supportedCurrencies.containsKey(targetCurrency)) {
            System.out.println("La moneda objetivo no es válida. Usa la opción 1 para ver las monedas soportadas.");
            return;
        }

        System.out.print("Introduce la cantidad a convertir: ");
        double amount = scanner.nextDouble();

        try {
            double exchangeRate = getExchangeRate(baseCurrency, targetCurrency);
            double convertedAmount = amount * exchangeRate;

            System.out.printf("El tipo de cambio de %s (%s) a %s (%s) es: %.2f%n", 
                baseCurrency, supportedCurrencies.get(baseCurrency), 
                targetCurrency, supportedCurrencies.get(targetCurrency), 
                exchangeRate);
            System.out.printf("La cantidad convertida es: %.2f %s%n", convertedAmount, targetCurrency);

            saveConversionLog(baseCurrency, targetCurrency, amount, convertedAmount);

        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }

    private static double getExchangeRate(String baseCurrency, String targetCurrency) throws Exception {
        String apiUrlWithBase = API_URL + baseCurrency;
        URL url = new URL(apiUrlWithBase);
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setRequestMethod("GET");

        if (connection.getResponseCode() != 200) {
            throw new RuntimeException("Error al conectar con la API: " + connection.getResponseCode());
        }

        BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
        JsonObject jsonResponse = JsonParser.parseReader(reader).getAsJsonObject();
        reader.close();

        JsonObject rates = jsonResponse.getAsJsonObject("conversion_rates");

        if (!rates.has(targetCurrency)) {
            throw new RuntimeException("La moneda objetivo no está disponible.");
        }

        return rates.get(targetCurrency).getAsDouble();
    }

    private static void saveConversionLog(String baseCurrency, String targetCurrency, double amount, double convertedAmount) {
        try (FileWriter writer = new FileWriter(LOG_FILE, true)) {
            LocalDateTime now = LocalDateTime.now();
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

            String logEntry = String.format("[%s] %.2f %s a %.2f %s%n", 
                    now.format(formatter), amount, baseCurrency, convertedAmount, targetCurrency);

            writer.write(logEntry);
            System.out.println("La conversión ha sido registrada.");
        } catch (IOException e) {
            System.out.println("Error al guardar el registro: " + e.getMessage());
        }
    }

    private static void viewConversionHistory() {
        try (BufferedReader reader = new BufferedReader(new FileReader(LOG_FILE))) {
            System.out.println("\n=== Historial de Conversiones ===");
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (FileNotFoundException e) {
            System.out.println("No hay historial registrado aún.");
        } catch (IOException e) {
            System.out.println("Error al leer el historial: " + e.getMessage());
        }
    }
}

