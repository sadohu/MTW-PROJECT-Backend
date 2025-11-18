# Arquitectura Técnica - MTW-PROJECT-Backend

## Índice
1. [Stack Tecnológico](#1-stack-tecnológico)
2. [Estructura de Proyecto](#2-estructura-de-proyecto)
3. [Flujos de Datos](#3-flujos-de-datos)
4. [Patrones de Diseño](#4-patrones-de-diseño)
5. [Seguridad y Configuración](#5-seguridad-y-configuración)

---

## 1. Stack Tecnológico

### Backend Framework
```
Spring Boot 3.1.6
├── Spring Data JPA (Persistencia)
├── Spring Web (REST APIs)
├── Spring Validation (Validación de datos)
└── Spring Boot DevTools (Desarrollo)
```

### Base de Datos
- **Motor:** MySQL
- **Conector:** mysql-connector-j
- **ORM:** Hibernate 6.x (incluido en Spring Boot 3.1.6)
- **Dialecto:** org.hibernate.dialect.MySQLDialect

### Librerías Adicionales
- **Lombok 1.18.x:** Reducción de boilerplate
- **Apache POI 5.2.3:** Procesamiento de archivos Excel
  - poi (core)
  - poi-ooxml (formato .xlsx)
- **SpringDoc OpenAPI 2.0.2:** Documentación Swagger

### Build Tool
- **Maven:** Gestión de dependencias y construcción

---

## 2. Estructura de Proyecto

### Paquetes Principales

```
com.mtwproject.backend.mtwprojectbackend/
├── controllers/          # Capa de Presentación (REST)
│   ├── AccountingController.java
│   ├── AreaController.java
│   ├── AuthController.java
│   ├── BillController.java
│   ├── BookingController.java
│   ├── CompanyController.java
│   ├── CurrencyController.java
│   ├── DriverController.java
│   ├── PassengerController.java
│   ├── ReportController.java
│   ├── UbigeoController.java
│   └── UsersController.java
│
├── services/             # Capa de Lógica de Negocio
│   ├── AccountingService.java (interface)
│   ├── AccountingServiceImpl.java
│   ├── AreaService.java (interface)
│   ├── AreaServiceImpl.java
│   ├── AuthService.java (interface)
│   ├── AuthServiceImpl.java
│   ├── BillService.java (interface)
│   ├── BillServiceImpl.java
│   ├── BookingService.java (interface)
│   ├── BookingServiceImpl.java
│   ├── CompanyService.java (interface)
│   ├── CompanyServiceImpl.java
│   ├── CurrencyService.java (interface)
│   ├── CurrencyServiveImpl.java
│   ├── DriverService.java (interface)
│   ├── DriverServiceImpl.java
│   ├── PassengerService.java (interface)
│   ├── PassengerServiceImpl.java
│   ├── ReportService.java
│   ├── UbigeoService.java (interface)
│   ├── UbigeoServiceImpl.java
│   ├── UsersService.java (interface)
│   └── UsersServiceImpl.java
│
├── repositories/         # Capa de Acceso a Datos
│   ├── AccountingRepository.java
│   ├── AreaRepository.java
│   ├── BillRepository.java
│   ├── BookingRepository.java
│   ├── CompanyRepository.java
│   ├── CurrencyRepository.java
│   ├── DriverRepository.java
│   ├── PassengerRepository.java
│   ├── UbigeoRepository.java
│   └── UsersRepository.java
│
├── models/
│   └── entities/         # Entidades JPA
│       ├── Accounting.java
│       ├── Area.java
│       ├── Bill.java
│       ├── BillAndBookingDTO.java
│       ├── Booking.java
│       ├── Company.java
│       ├── Currency.java
│       ├── Driver.java
│       ├── LoginRequest.java
│       ├── Passenger.java
│       ├── Ubigeo.java
│       └── Users.java
│
└── MtwProjectBackendApplication.java  # Punto de entrada
```

---

## 3. Flujos de Datos

### 3.1 Flujo de Creación de Reserva

```
Cliente (Angular)
    │
    ▼
[POST /booking]
    │
    ▼
BookingController.saveBooking()
    │
    ├─> Validación de entrada
    │
    ▼
BookingServiceImpl.saveBooking()
    │
    ├─> Lógica de negocio:
    │   ├─> ¿Es nueva reserva? (idBooking == null)
    │   │   ├─> Tiene conductor asignado? → Status = "Pendiente"
    │   │   └─> No tiene conductor? → Status = "En Reserva"
    │   │
    │   └─> ¿Es actualización? (idBooking != null)
    │       └─> ¿Se asigna conductor y status es "En Reserva"?
    │           └─> Status = "Pendiente"
    │
    ▼
BookingRepository.save()
    │
    ├─> Hibernate genera SQL INSERT/UPDATE
    │
    ▼
MySQL Database
    │
    ▼
Respuesta:
{
  "status": "200",
  "message": "La reserva N° X se ha creado correctamente",
  "data": { Booking object }
}
```

### 3.2 Flujo de Autenticación

```
Cliente (Angular)
    │
    ▼
[POST /auth/login]
{
  "username": "user",
  "password": "pass"
}
    │
    ▼
AuthController.login()
    │
    ▼
AuthServiceImpl.authenticateUser()
    │
    ├─> Búsqueda de usuario en BD
    ├─> Validación de credenciales (⚠️ sin encriptación)
    │
    ├─> ¿Usuario válido?
    │   ├─> SÍ → Retorna datos del usuario
    │   └─> NO → Error 401/404
    │
    ▼
Respuesta con datos de usuario
```

**⚠️ ALERTA DE SEGURIDAD:** No hay encriptación de contraseñas ni tokens JWT.

### 3.3 Flujo de Generación de Reporte

```
Cliente (Angular)
    │
    ▼
[GET /report/companies] (endpoint hipotético)
    │
    ▼
ReportController.generateCompanyReport()
    │
    ├─> CompanyService.findAll()
    │       │
    │       ▼
    │   List<Company> companies
    │
    ▼
ReportService.generateExcelReport(companies)
    │
    ├─> Apache POI: XSSFWorkbook
    ├─> Crear hoja "Companies"
    ├─> Crear encabezados
    ├─> Iterar sobre companies
    │   └─> Crear filas con datos
    ├─> Auto-ajustar columnas
    │
    ▼
byte[] excelFile
    │
    ▼
Response con archivo Excel
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
```

---

## 4. Patrones de Diseño

### 4.1 Layered Architecture (Arquitectura en Capas)

**Separación de responsabilidades:**

```
┌──────────────────────────────────────────────────────┐
│ PRESENTATION LAYER (Controllers)                     │
│ - Manejo de HTTP requests/responses                  │
│ - Validación básica de entrada                       │
│ - Formateo de respuestas JSON                        │
│ - CORS y seguridad de endpoints                      │
└──────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│ BUSINESS LOGIC LAYER (Services)                      │
│ - Lógica de negocio                                  │
│ - Validaciones complejas                             │
│ - Orquestación de operaciones                        │
│ - Transacciones                                      │
└──────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│ DATA ACCESS LAYER (Repositories)                     │
│ - Abstracción de acceso a datos                      │
│ - Consultas JPQL personalizadas                      │
│ - Métodos derivados de Spring Data                   │
└──────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│ DOMAIN LAYER (Entities)                              │
│ - Definición del modelo de dominio                   │
│ - Relaciones entre entidades                         │
│ - Anotaciones JPA                                    │
└──────────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────────┐
│ PERSISTENCE LAYER (MySQL Database)                   │
│ - Almacenamiento físico de datos                     │
└──────────────────────────────────────────────────────┘
```

### 4.2 Repository Pattern

Abstracción del acceso a datos usando interfaces de Spring Data JPA:

```java
public interface BookingRepository extends CrudRepository<Booking, Long> {
    // Métodos automáticos: save(), findById(), findAll(), delete()

    // Método derivado
    List<Booking> findByCompany(Company company);

    // Consulta personalizada con JPQL
    @Query("SELECT b FROM Booking b WHERE b.bill.id = :billId")
    List<Booking> findByBillId(@Param("billId") Long billId);
}
```

**Ventajas:**
- Desacoplamiento de la lógica de persistencia
- Facilita testing con mocks
- Reutilización de código

### 4.3 Service Layer Pattern

Separación de interfaz e implementación:

```java
// Contrato
public interface BookingService {
    List<Booking> findAll();
    Optional<Booking> findById(Long id);
    Booking saveBooking(Booking booking);
    void deleteBooking(Long idBooking);
}

// Implementación
@Service
public class BookingServiceImpl implements BookingService {
    @Autowired
    private BookingRepository repository;

    // Implementación de métodos...
}
```

**Ventajas:**
- Facilita cambio de implementación
- Permite múltiples implementaciones
- Mejor para testing

### 4.4 Data Transfer Object (DTO)

Uso limitado, ejemplo encontrado:

```java
public class BillAndBookingDTO {
    // DTO para transferir datos combinados
}

public class LoginRequest {
    // DTO para requests de login
}
```

**Recomendación:** Extender uso de DTOs para no exponer entidades directamente.

### 4.5 Dependency Injection

Uso extensivo de inyección de dependencias con `@Autowired`:

```java
@RestController
public class BookingController {
    @Autowired
    private BookingService bookingService;  // Inyección automática
}
```

### 4.6 Constants Pattern

Uso de constantes para valores fijos:

```java
@Service
public class BookingServiceImpl implements BookingService {
    private static final String RESERVE_STATUS = "En Reserva";
    private static final String PENDING_DRIVER_ASSIGNED_STATUS = "Pendiente";
    public static final String IN_PROCESS_STATUS = "En Proceso";
    public static final String FINALIZED_STATUS = "Finalizado";
}
```

---

## 5. Seguridad y Configuración

### 5.1 Configuración de Base de Datos

**application.properties:**
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mtw_project
spring.datasource.username=root
spring.datasource.password=sasa  # ⚠️ Credencial hardcodeada
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
```

**⚠️ PROBLEMAS DE SEGURIDAD:**
1. Credenciales en texto plano
2. Usuario root de MySQL
3. Contraseña débil
4. No se usan variables de entorno

**✅ SOLUCIÓN RECOMENDADA:**
```properties
spring.datasource.url=${DB_URL:jdbc:mysql://localhost:3306/mtw_project}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

### 5.2 CORS Configuration

Todos los controladores tienen:

```java
@CrossOrigin(origins = "http://localhost:4200")
```

**Limitaciones:**
- Solo permite frontend local
- No escalable para múltiples entornos
- Configuración repetida en cada controller

**✅ SOLUCIÓN RECOMENDADA:**
Configuración global:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins(
                    "http://localhost:4200",
                    "${FRONTEND_URL}"
                )
                .allowedMethods("GET", "POST", "PUT", "DELETE");
    }
}
```

### 5.3 Autenticación y Autorización

**Estado Actual:**
- Sistema básico de login
- Sin tokens (JWT/OAuth)
- Sin gestión de sesiones
- Sin roles y permisos
- Contraseñas sin encriptar

**⚠️ RIESGOS:**
- Vulnerabilidad a ataques de fuerza bruta
- No hay expiración de sesiones
- No hay refresh tokens
- Contraseñas expuestas en BD

**✅ ARQUITECTURA RECOMENDADA:**

```
Cliente → Login → AuthController
              ↓
    AuthService (Spring Security)
              ↓
    ┌─────────────────────┐
    │ Validar credenciales│
    │ (BCrypt)            │
    └─────────────────────┘
              ↓
    ┌─────────────────────┐
    │ Generar JWT Token   │
    │ - Access Token      │
    │ - Refresh Token     │
    └─────────────────────┘
              ↓
    Token → Cliente
              ↓
    Requests subsecuentes incluyen:
    Header: Authorization: Bearer <token>
              ↓
    JWTAuthenticationFilter valida token
```

### 5.4 Manejo de Transacciones

**Uso actual:**
```java
@Service
public class BillServiceImpl implements BillService {
    @Override
    @Transactional(readOnly = true)
    public List<Bill> findAll() {
        return (List<Bill>) repository.findAll();
    }
}
```

**Observaciones:**
- Uso limitado de `@Transactional`
- No hay manejo de transacciones complejas
- Falta `@Transactional` en operaciones de escritura en algunos servicios

**✅ RECOMENDACIÓN:**
Aplicar consistentemente en todos los métodos de escritura:

```java
@Override
@Transactional
public Booking saveBooking(Booking booking) {
    // Operación transaccional
}
```

### 5.5 Validación de Datos

**Validación actual (limitada):**

```java
@Entity
@Table(name = "COMPANY")
public class Company {
    @Column(nullable = false)
    private String businessName;

    @Column(nullable = false)
    @Pattern(regexp = "^[0-9]{11}$",
             message = "El número de identificación debe tener 11 dígitos")
    private String idNumber;

    @Pattern(regexp = "^[0-9+()\\s]*$",
             message = "El numero de telefono esta en un formato invalido")
    private String phone;
}
```

**Problemas:**
- Solo `Company` tiene validaciones
- Falta `@Valid` en controllers
- No hay validación de reglas de negocio

**✅ SOLUCIÓN:**

```java
@Entity
public class Booking {
    @NotNull(message = "La fecha es obligatoria")
    @Future(message = "La fecha debe ser futura")
    private Date date;

    @NotNull
    @ManyToOne
    private Company company;

    @DecimalMin(value = "0.0", message = "El precio debe ser positivo")
    private BigDecimal price;
}

// En controller:
@PostMapping
public ResponseEntity<?> saveBooking(@Valid @RequestBody Booking booking) {
    // ...
}
```

---

## 6. Configuración de Hibernate/JPA

### 6.1 Estrategia de Nombres

```properties
spring.jpa.hibernate.naming.physical-strategy=
    org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

**Implicación:** Los nombres de tablas y columnas se mantienen exactamente como se definen en las entidades (no se convierten a snake_case).

### 6.2 Estrategia de Generación de IDs

Todas las entidades usan:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

**Características:**
- Auto-incremento delegado a la base de datos
- Eficiente para MySQL
- No requiere tabla de secuencias

### 6.3 Lazy Loading

Uso extensivo de carga perezosa:

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "idCompany")
private Company company;
```

**Ventajas:**
- Mejor rendimiento inicial
- Carga datos solo cuando se necesitan

**Desafíos:**
- Requiere sesión activa de Hibernate
- Puede causar `LazyInitializationException`
- Solución actual: `@JsonIgnoreProperties({ "hibernateLazyInitializer", "handler" })`

### 6.4 Logging de SQL

```properties
spring.jpa.show-sql=true
logging.level.org.hibernate.SQL=debug
```

**Útil para:**
- Debugging
- Optimización de queries
- Detección de N+1 queries

**⚠️ ADVERTENCIA:** Desactivar en producción por rendimiento y seguridad.

---

## 7. Análisis de Rendimiento

### 7.1 Posibles N+1 Queries

**Problema potencial en:**

```java
List<Booking> bookings = bookingService.findAll();
// Para cada booking, si se accede a booking.getCompany().getName()
// se genera una query adicional (N queries para N bookings)
```

**✅ SOLUCIÓN:** Usar JOIN FETCH

```java
@Query("SELECT b FROM Booking b " +
       "LEFT JOIN FETCH b.company " +
       "LEFT JOIN FETCH b.driver " +
       "LEFT JOIN FETCH b.passenger")
List<Booking> findAllWithDetails();
```

### 7.2 Índices Recomendados

**Basado en consultas frecuentes:**

```sql
-- Tabla BOOKING
CREATE INDEX idx_booking_status ON BOOKING(status);
CREATE INDEX idx_booking_date ON BOOKING(date);
CREATE INDEX idx_booking_company ON BOOKING(idCompany);
CREATE INDEX idx_booking_driver ON BOOKING(idDriver);

-- Tabla COMPANY
CREATE INDEX idx_company_idnumber ON COMPANY(idNumber);

-- Tabla USERS
CREATE INDEX idx_users_username ON USER(username);
```

---

## 8. Integración con Frontend

### 8.1 Contrato de API

**Formato de Respuesta Estandarizado:**

```json
{
  "status": "200",
  "message": "Descripción legible",
  "data": { ... }
}
```

**Casos de uso:**
- Status 200: Operación exitosa
- Status 404: Recurso no encontrado
- Status 500: Error del servidor

**⚠️ PROBLEMA:** Se usa status HTTP 200 incluso para errores, con "status" en el body.

**✅ MEJOR PRÁCTICA:**
```java
return ResponseEntity
    .status(HttpStatus.NOT_FOUND)
    .body(message);
```

### 8.2 Comunicación Angular-Spring

```
Angular App (localhost:4200)
    │
    ├─> HTTP Client
    │   └─> Interceptors (headers, auth)
    │
    ▼
Spring Boot API (localhost:8080)
    │
    ├─> CORS Filter
    ├─> Controllers
    └─> Response
    │
    ▼
Angular Services
    └─> Components
```

---

## 9. Documentación con Swagger

### 9.1 Configuración

```properties
springdoc.api-docs.enabled=true
springdoc.swagger-ui.enabled=true
springdoc.swagger-ui.path=/swagger-ui.html
```

### 9.2 Acceso

URL: `http://localhost:8080/swagger-ui.html`

**Beneficios:**
- Documentación automática de endpoints
- Testing interactivo de APIs
- Generación de cliente SDK

**Mejora Recomendada:**
Agregar anotaciones para mejor documentación:

```java
@Operation(summary = "Crear nueva reserva",
           description = "Crea una reserva de transporte con estado inicial")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "Reserva creada"),
    @ApiResponse(responseCode = "500", description = "Error del servidor")
})
@PostMapping
public ResponseEntity<?> saveBooking(
    @io.swagger.v3.oas.annotations.parameters.RequestBody(
        description = "Datos de la reserva",
        required = true
    )
    @RequestBody Booking booking
) {
    // ...
}
```

---

## 10. Recomendaciones de Arquitectura

### 10.1 Migración a Microservicios (Futuro)

Si el sistema crece, considerar:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Booking        │     │  Billing        │     │  User           │
│  Service        │────>│  Service        │     │  Management     │
│  (Core)         │     │                 │     │  Service        │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                         │
        └───────────────────────┴─────────────────────────┘
                                │
                        ┌───────────────┐
                        │  API Gateway  │
                        │  (Spring Cloud│
                        │   Gateway)    │
                        └───────────────┘
```

### 10.2 Implementar CQRS (Command Query Responsibility Segregation)

Para operaciones de lectura intensiva:

```
Commands (Write):          Queries (Read):
POST /booking         →    GET /booking
PUT /booking          →    GET /booking/{id}
DELETE /booking       →    GET /booking/filterByParams
        │                           │
        ▼                           ▼
   Write DB                     Read DB (Replica)
```

### 10.3 Event-Driven Architecture

Para operaciones asíncronas:

```
Booking Created Event
    │
    ├─> Send Notification Service
    ├─> Update Accounting Service
    ├─> Analytics Service
    └─> Email Service
```

---

**Documento creado:** 2025-11-18
**Autor:** Análisis Técnico IA
**Versión:** 1.0
