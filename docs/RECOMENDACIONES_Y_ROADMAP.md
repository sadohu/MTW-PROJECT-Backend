# Recomendaciones y Roadmap - MTW-PROJECT-Backend

## Índice
1. [Mejoras Críticas Prioritarias](#1-mejoras-críticas-prioritarias)
2. [Mejoras de Mediano Plazo](#2-mejoras-de-mediano-plazo)
3. [Optimizaciones de Largo Plazo](#3-optimizaciones-de-largo-plazo)
4. [Roadmap de Implementación](#4-roadmap-de-implementación)
5. [Checklist de Seguridad](#5-checklist-de-seguridad)

---

## 1. Mejoras Críticas Prioritarias

### 1.1 Seguridad de Autenticación

**Problema:**
- Contraseñas en texto plano
- Sin tokens de sesión
- Sin expiración de sesión

**Solución:**

#### Paso 1: Agregar Dependencias

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

#### Paso 2: Encriptar Contraseñas Existentes

```java
@Service
public class PasswordMigrationService {
    @Autowired
    private UsersRepository usersRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    public void migratePasswords() {
        List<Users> users = usersRepository.findAll();
        for (Users user : users) {
            String plainPassword = user.getPassword();
            String encodedPassword = passwordEncoder.encode(plainPassword);
            user.setPassword(encodedPassword);
            usersRepository.save(user);
        }
    }
}
```

#### Paso 3: Implementar JWT Service

```java
@Service
public class JwtService {
    @Value("${jwt.secret}")
    private String secretKey;

    @Value("${jwt.expiration:3600000}") // 1 hora
    private Long expiration;

    public String generateToken(Users user) {
        return Jwts.builder()
            .setSubject(user.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .claim("idUser", user.getIdUser())
            .claim("userType", user.getIdUserType())
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }

    // ... métodos auxiliares
}
```

**Prioridad:** 🔴 CRÍTICA
**Esfuerzo:** 2-3 días
**Impacto:** Alto

---

### 1.2 Protección de Credenciales de Base de Datos

**Problema:**
```properties
# ❌ Credenciales expuestas
spring.datasource.password=sasa
```

**Solución:**

#### Opción A: Variables de Entorno

```properties
# application.properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

```bash
# .env (no committear)
DB_URL=jdbc:mysql://localhost:3306/mtw_project
DB_USERNAME=mtw_user
DB_PASSWORD=SecurePassword123!
```

#### Opción B: Perfiles de Spring

```properties
# application-dev.properties (local)
spring.datasource.password=dev_password

# application-prod.properties (producción, no committear)
spring.datasource.password=${DB_PASSWORD}
```

**Prioridad:** 🔴 CRÍTICA
**Esfuerzo:** 1 día
**Impacto:** Alto

---

### 1.3 Manejo Centralizado de Excepciones

**Problema:** Cada controller maneja excepciones de forma repetitiva

**Solución:**

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<?> handleEntityNotFound(EntityNotFoundException ex) {
        Map<String, Object> response = new HashMap<>();
        response.put("status", "404");
        response.put("message", ex.getMessage());
        response.put("timestamp", LocalDateTime.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, Object> response = new HashMap<>();
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.toList());

        response.put("status", "400");
        response.put("message", "Errores de validación");
        response.put("errors", errors);
        return ResponseEntity.badRequest().body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<?> handleGeneralException(Exception ex) {
        Map<String, Object> response = new HashMap<>();
        response.put("status", "500");
        response.put("message", "Error interno del servidor");
        // En desarrollo incluir stack trace, en producción no
        if (isDevelopment()) {
            response.put("details", ex.getMessage());
        }
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}
```

**Prioridad:** 🟡 ALTA
**Esfuerzo:** 2 días
**Impacto:** Medio-Alto

---

### 1.4 Corregir Tipos de Datos Monetarios

**Problema:**
```java
private String price;      // ❌ Incorrecto
private String subTotal;   // ❌ Incorrecto
```

**Solución:**

```java
@Entity
@Table(name = "BOOKING")
public class Booking {
    @Column(precision = 10, scale = 2)
    private BigDecimal price;

    // Getters y Setters
}

@Entity
@Table(name = "BILL")
public class Bill {
    @Column(precision = 10, scale = 2)
    private BigDecimal subTotal;

    @Column(precision = 10, scale = 2)
    private BigDecimal igv;

    @Column(precision = 10, scale = 2)
    private BigDecimal total;

    public void calculateTotal() {
        this.total = this.subTotal.add(this.igv);
    }

    public void calculateIGV(BigDecimal igvRate) {
        this.igv = this.subTotal.multiply(igvRate);
        calculateTotal();
    }
}
```

**⚠️ ADVERTENCIA:** Requiere migración de base de datos

**Script de Migración:**
```sql
-- Backup primero!
ALTER TABLE BOOKING
    MODIFY COLUMN price DECIMAL(10,2);

ALTER TABLE BILL
    MODIFY COLUMN subTotal DECIMAL(10,2),
    MODIFY COLUMN igv DECIMAL(10,2),
    MODIFY COLUMN total DECIMAL(10,2);
```

**Prioridad:** 🟡 ALTA
**Esfuerzo:** 3-4 días (incluye testing)
**Impacto:** Alto

---

### 1.5 Implementar Validaciones Completas

**Problema:** Solo `Company` tiene validaciones

**Solución:**

```java
@Entity
@Table(name = "BOOKING")
public class Booking {
    @NotNull(message = "La fecha es obligatoria")
    @FutureOrPresent(message = "La fecha no puede ser en el pasado")
    private Date date;

    @NotNull(message = "La hora es obligatoria")
    private Date time;

    @NotNull(message = "La empresa es obligatoria")
    @ManyToOne(fetch = FetchType.LAZY)
    private Company company;

    @NotBlank(message = "El solicitante es obligatorio")
    @Size(min = 3, max = 100, message = "El nombre debe tener entre 3 y 100 caracteres")
    private String applicant;

    @NotBlank(message = "El punto de recogida es obligatorio")
    private String pickUp;

    @NotBlank(message = "El destino es obligatorio")
    private String destination;

    @NotNull(message = "El precio es obligatorio")
    @DecimalMin(value = "0.0", inclusive = false, message = "El precio debe ser mayor a 0")
    private BigDecimal price;
}
```

```java
@RestController
public class BookingController {
    @PostMapping
    public ResponseEntity<?> saveBooking(@Valid @RequestBody Booking booking) {
        // Las validaciones se ejecutan automáticamente
        // Si fallan, lanza MethodArgumentNotValidException
        // que es capturada por GlobalExceptionHandler
    }
}
```

**Prioridad:** 🟡 ALTA
**Esfuerzo:** 2-3 días
**Impacto:** Medio-Alto

---

## 2. Mejoras de Mediano Plazo

### 2.1 Implementar DTOs

**Problema:** Se exponen entidades JPA directamente

**Solución:**

```java
// DTOs
public class BookingRequestDTO {
    @NotNull
    private LocalDate date;

    @NotNull
    private LocalTime time;

    @NotNull
    private Long idCompany;

    @NotBlank
    private String applicant;

    @NotNull
    private Long idPassenger;

    private Long idDriver;

    // ... otros campos
}

public class BookingResponseDTO {
    private Long idBooking;
    private String date;
    private String time;
    private CompanyDTO company;
    private PassengerDTO passenger;
    private DriverDTO driver;
    private String status;
    private BigDecimal price;
    // Solo campos necesarios para el cliente
}

// Mapper
@Component
public class BookingMapper {
    public BookingResponseDTO toDTO(Booking booking) {
        BookingResponseDTO dto = new BookingResponseDTO();
        dto.setIdBooking(booking.getIdBooking());
        dto.setDate(booking.getFechaReserva());
        // ... mapeo de campos
        return dto;
    }

    public Booking toEntity(BookingRequestDTO dto) {
        Booking booking = new Booking();
        // ... mapeo reverso
        return booking;
    }
}
```

**Alternativa: Usar MapStruct**

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>
```

```java
@Mapper(componentModel = "spring")
public interface BookingMapper {
    BookingResponseDTO toDTO(Booking booking);
    Booking toEntity(BookingRequestDTO dto);
}
```

**Beneficios:**
- Control sobre qué se expone
- Evita lazy loading issues
- Facilita versionado de API

**Prioridad:** 🟢 MEDIA
**Esfuerzo:** 1 semana
**Impacto:** Medio

---

### 2.2 Implementar Testing

**Problema:** No hay tests

**Solución:**

#### Tests Unitarios de Servicios

```java
@ExtendWith(MockitoExtension.class)
class BookingServiceImplTest {

    @Mock
    private BookingRepository repository;

    @InjectMocks
    private BookingServiceImpl service;

    @Test
    void whenSaveNewBookingWithoutDriver_thenStatusIsEnReserva() {
        // Given
        Booking booking = new Booking();
        booking.setIdBooking(null);
        booking.setDriver(null);

        when(repository.save(any(Booking.class)))
            .thenAnswer(i -> {
                Booking saved = i.getArgument(0);
                saved.setIdBooking(1L);
                return saved;
            });

        // When
        Booking result = service.saveBooking(booking);

        // Then
        assertEquals("En Reserva", result.getStatus());
        verify(repository).save(booking);
    }

    @Test
    void whenSaveNewBookingWithDriver_thenStatusIsPendiente() {
        // Given
        Booking booking = new Booking();
        booking.setIdBooking(null);
        Driver driver = new Driver();
        driver.setIdDriver(1L);
        booking.setDriver(driver);

        when(repository.save(any(Booking.class)))
            .thenAnswer(i -> {
                Booking saved = i.getArgument(0);
                saved.setIdBooking(1L);
                return saved;
            });

        // When
        Booking result = service.saveBooking(booking);

        // Then
        assertEquals("Pendiente", result.getStatus());
    }
}
```

#### Tests de Integración

```java
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
class BookingControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void whenCreateBooking_thenReturns200() throws Exception {
        // Given
        BookingRequestDTO request = new BookingRequestDTO();
        // ... configurar DTO

        // When & Then
        mockMvc.perform(post("/booking")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.status").value("200"))
                .andExpect(jsonPath("$.data.idBooking").exists());
    }
}
```

**Estructura de Tests:**
```
src/test/java/
├── unit/
│   ├── services/
│   │   ├── BookingServiceImplTest.java
│   │   ├── BillServiceImplTest.java
│   │   └── ...
│   └── utils/
│       └── BookingMapperTest.java
├── integration/
│   ├── controllers/
│   │   ├── BookingControllerIntegrationTest.java
│   │   └── ...
│   └── repositories/
│       └── BookingRepositoryTest.java
└── e2e/
    └── BookingE2ETest.java
```

**Prioridad:** 🟢 MEDIA
**Esfuerzo:** 2 semanas
**Impacto:** Alto (a largo plazo)

---

### 2.3 Implementar Auditoría

**Problema:** No se sabe quién creó/modificó registros

**Solución:**

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class Auditable {

    @CreatedBy
    @Column(nullable = false, updatable = false)
    private String createdBy;

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedBy
    private String lastModifiedBy;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    // Getters y Setters
}

// Las entidades extienden de Auditable
@Entity
@Table(name = "BOOKING")
public class Booking extends Auditable {
    // ... campos existentes
}

// Configuración
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaAuditingConfiguration {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            // Obtener usuario del contexto de seguridad
            Authentication auth = SecurityContextHolder
                .getContext()
                .getAuthentication();

            if (auth == null || !auth.isAuthenticated()) {
                return Optional.of("SYSTEM");
            }

            return Optional.of(auth.getName());
        };
    }
}
```

**Prioridad:** 🟢 MEDIA
**Esfuerzo:** 2-3 días
**Impacto:** Medio

---

### 2.4 Optimización de Consultas

**Problema:** Posibles N+1 queries

**Solución:**

```java
public interface BookingRepository extends CrudRepository<Booking, Long> {

    @Query("SELECT DISTINCT b FROM Booking b " +
           "LEFT JOIN FETCH b.company " +
           "LEFT JOIN FETCH b.driver " +
           "LEFT JOIN FETCH b.passenger " +
           "LEFT JOIN FETCH b.area " +
           "LEFT JOIN FETCH b.ubigeoPickUp " +
           "LEFT JOIN FETCH b.ubigeoDestination " +
           "LEFT JOIN FETCH b.currency " +
           "LEFT JOIN FETCH b.bill")
    List<Booking> findAllWithRelations();

    @Query("SELECT b FROM Booking b " +
           "LEFT JOIN FETCH b.company " +
           "LEFT JOIN FETCH b.driver " +
           "WHERE b.idBooking = :id")
    Optional<Booking> findByIdWithRelations(@Param("id") Long id);
}

// En el servicio
@Override
@Transactional(readOnly = true)
public List<Booking> findAll() {
    return repository.findAllWithRelations();
}
```

**Monitoreo de Performance:**

```java
@Aspect
@Component
public class QueryCountAspect {

    @Around("execution(* com.mtwproject.backend..*Repository+.*(..))")
    public Object logQueryCount(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long executionTime = System.currentTimeMillis() - start;

        if (executionTime > 1000) { // Más de 1 segundo
            log.warn("Query lenta detectada: {} - {}ms",
                joinPoint.getSignature(), executionTime);
        }

        return result;
    }
}
```

**Prioridad:** 🟢 MEDIA
**Esfuerzo:** 1 semana
**Impacto:** Medio-Alto

---

## 3. Optimizaciones de Largo Plazo

### 3.1 Caché con Redis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

```java
@Configuration
@EnableCaching
public class CacheConfiguration {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .serializeKeysWith(RedisSerializationContext
                .SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext
                .SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}

@Service
public class CompanyServiceImpl implements CompanyService {

    @Cacheable(value = "companies", key = "#id")
    @Override
    public Optional<Company> findById(Long id) {
        return repository.findById(id);
    }

    @CacheEvict(value = "companies", key = "#company.idCompany")
    @Override
    public Company save(Company company) {
        return repository.save(company);
    }
}
```

**Prioridad:** 🔵 BAJA
**Esfuerzo:** 1 semana
**Impacto:** Alto (para alto tráfico)

---

### 3.2 Paginación y Ordenamiento

```java
public interface BookingRepository
    extends CrudRepository<Booking, Long>, PagingAndSortingRepository<Booking, Long> {

    @Query("SELECT b FROM Booking b " +
           "LEFT JOIN FETCH b.company " +
           "WHERE (:status IS NULL OR b.status = :status)")
    Page<Booking> findByStatus(
        @Param("status") String status,
        Pageable pageable
    );
}

@RestController
public class BookingController {

    @GetMapping
    public ResponseEntity<?> bookingList(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "date,desc") String[] sort
    ) {
        Pageable pageable = PageRequest.of(page, size,
            Sort.by(Sort.Direction.fromString(sort[1]), sort[0]));

        Page<Booking> bookingPage = bookingService.findAll(pageable);

        Map<String, Object> response = new HashMap<>();
        response.put("data", bookingPage.getContent());
        response.put("currentPage", bookingPage.getNumber());
        response.put("totalItems", bookingPage.getTotalElements());
        response.put("totalPages", bookingPage.getTotalPages());

        return ResponseEntity.ok(response);
    }
}
```

**Prioridad:** 🟢 MEDIA
**Esfuerzo:** 3 días
**Impacto:** Alto (para listas grandes)

---

### 3.3 Sistema de Notificaciones

**Notificaciones por Email/SMS cuando:**
- Se crea una nueva reserva
- Se asigna un conductor
- El servicio inicia
- El servicio finaliza
- Se genera una factura

```java
@Service
public class NotificationService {

    @Autowired
    private JavaMailSender mailSender;

    @Async
    public void sendBookingCreatedNotification(Booking booking) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(booking.getCompany().getEmail());
        message.setSubject("Reserva Creada - " + booking.getIdBooking());
        message.setText(buildBookingEmail(booking));

        mailSender.send(message);
    }
}

// En BookingServiceImpl
@Override
@Transactional
public Booking saveBooking(Booking booking) {
    // ... lógica existente
    Booking saved = repository.save(booking);

    // Notificación asíncrona
    notificationService.sendBookingCreatedNotification(saved);

    return saved;
}
```

**Prioridad:** 🔵 BAJA
**Esfuerzo:** 1 semana
**Impacto:** Alto (UX)

---

### 3.4 Dashboard y Analytics

```java
@RestController
@RequestMapping("/analytics")
public class AnalyticsController {

    @GetMapping("/bookings-by-status")
    public ResponseEntity<?> getBookingsByStatus() {
        Map<String, Long> stats = bookingService.countByStatus();
        return ResponseEntity.ok(stats);
    }

    @GetMapping("/revenue-by-month")
    public ResponseEntity<?> getRevenueByMonth(
        @RequestParam int year
    ) {
        List<MonthlyRevenue> revenue = bookingService.getMonthlyRevenue(year);
        return ResponseEntity.ok(revenue);
    }

    @GetMapping("/top-drivers")
    public ResponseEntity<?> getTopDrivers(
        @RequestParam(defaultValue = "10") int limit
    ) {
        List<DriverStats> topDrivers = driverService.getTopDrivers(limit);
        return ResponseEntity.ok(topDrivers);
    }
}
```

**Prioridad:** 🔵 BAJA
**Esfuerzo:** 2 semanas
**Impacto:** Medio

---

## 4. Roadmap de Implementación

### Sprint 1 (Semana 1-2): Seguridad Crítica
- ✅ Implementar Spring Security
- ✅ Encriptación de contraseñas con BCrypt
- ✅ Implementar JWT
- ✅ Proteger credenciales de BD
- ✅ Configurar perfiles (dev, prod)

**Entregable:** Sistema con autenticación segura

---

### Sprint 2 (Semana 3-4): Calidad de Código
- ✅ Manejo centralizado de excepciones
- ✅ Implementar validaciones completas
- ✅ Corregir tipos de datos (String → BigDecimal)
- ✅ Corrección de typos y nomenclatura

**Entregable:** Código más robusto y mantenible

---

### Sprint 3 (Semana 5-6): Testing
- ✅ Configurar framework de testing
- ✅ Tests unitarios de servicios (cobertura >80%)
- ✅ Tests de integración de controllers
- ✅ Tests de repositorios

**Entregable:** Suite de tests completa

---

### Sprint 4 (Semana 7-8): Optimización
- ✅ Implementar DTOs
- ✅ Optimizar consultas (JOIN FETCH)
- ✅ Implementar auditoría
- ✅ Agregar índices a BD

**Entregable:** Sistema optimizado

---

### Sprint 5 (Semana 9-10): Características Adicionales
- ✅ Implementar paginación
- ✅ Mejorar documentación Swagger
- ✅ Sistema de roles y permisos
- ✅ Logs estructurados

**Entregable:** Sistema con características empresariales

---

### Sprint 6+ (Largo Plazo): Escalabilidad
- ⏳ Implementar caché (Redis)
- ⏳ Sistema de notificaciones
- ⏳ Dashboard y analytics
- ⏳ Monitoreo y alertas

---

## 5. Checklist de Seguridad

### Autenticación y Autorización
- [ ] ✅ Contraseñas encriptadas con BCrypt
- [ ] ✅ Tokens JWT implementados
- [ ] ✅ Expiración de tokens configurada
- [ ] ✅ Refresh tokens implementados
- [ ] ✅ Sistema de roles y permisos
- [ ] ✅ Endpoints protegidos según rol

### Configuración
- [ ] ✅ Credenciales en variables de entorno
- [ ] ✅ Secretos no committeados en git
- [ ] ✅ CORS configurado correctamente
- [ ] ✅ HTTPS habilitado en producción
- [ ] ✅ Headers de seguridad configurados

### Base de Datos
- [ ] ✅ Usuario de BD con permisos mínimos
- [ ] ✅ Contraseña fuerte de BD
- [ ] ✅ SQL Injection prevention (usando JPA/JPQL)
- [ ] ✅ Backups automáticos configurados
- [ ] ✅ Encriptación de datos sensibles

### Validación
- [ ] ✅ Validación de entrada en todos los endpoints
- [ ] ✅ Sanitización de datos
- [ ] ✅ Rate limiting implementado
- [ ] ✅ Tamaño máximo de requests configurado

### Logging y Monitoreo
- [ ] ✅ Logs no contienen información sensible
- [ ] ✅ Logs centralizados
- [ ] ✅ Alertas de seguridad configuradas
- [ ] ✅ Auditoría de acciones críticas

### Dependencias
- [ ] ✅ Dependencias actualizadas
- [ ] ✅ Escaneo de vulnerabilidades (Maven Dependency Check)
- [ ] ✅ Sin dependencias con CVEs críticos

---

## 6. Mejores Prácticas de Desarrollo

### Git Workflow
```bash
# Feature branches
git checkout -b feature/JWT-authentication
git commit -m "feat: implement JWT authentication"
git push origin feature/JWT-authentication

# Pull request → Code review → Merge to develop
```

### Commits Semánticos
```
feat: nueva característica
fix: corrección de bug
refactor: refactorización de código
test: agregar tests
docs: documentación
chore: tareas de mantenimiento
```

### Code Review Checklist
- [ ] Código sigue estándares del proyecto
- [ ] Tests incluidos y pasando
- [ ] Sin código comentado
- [ ] Sin console.log / sysout
- [ ] Documentación actualizada
- [ ] Sin regresiones

---

## 7. Recursos y Referencias

### Documentación Oficial
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Security](https://docs.spring.io/spring-security/reference/index.html)
- [Spring Data JPA](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)

### Tutoriales Recomendados
- JWT con Spring Boot: https://www.baeldung.com/spring-security-oauth-jwt
- Testing en Spring: https://www.baeldung.com/spring-boot-testing
- Performance Tuning: https://www.baeldung.com/jpa-hibernate-performance

### Herramientas
- **SonarQube:** Análisis de calidad de código
- **JaCoCo:** Cobertura de tests
- **OWASP Dependency-Check:** Escaneo de vulnerabilidades
- **Postman/Insomnia:** Testing de APIs

---

**Documento creado:** 2025-11-18
**Última actualización:** 2025-11-18
**Versión:** 1.0
