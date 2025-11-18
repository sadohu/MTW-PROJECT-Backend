# Requisitos y Alcance del Proyecto MTW-PROJECT-Backend

 

## Información del Proyecto

 

**Fecha de Definición:** 2025-11-18

**Versión:** 1.0

**Estado:** Definición de Requisitos

 

---

 

## 1. Visión del Proyecto

 

### Descripción General

MTW-PROJECT-Backend es una **plataforma de gestión de servicios de transporte corporativo** que conecta empresas con conductores independientes (y eventualmente flotas propias) para servicios de movilidad en Perú.

 

### Público Objetivo

- **Principal:** Empresas pequeñas que contratan conductores independientes con sus propios vehículos

- **Secundario:** Empresas con vehículos propios (flota interna)

- **Futuro:** Modelo híbrido (combinación de conductores independientes y flota propia)

 

### Modelo de Negocio

 

```

┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐

│   Empresas      │◄────────│   Plataforma     │────────►│   Conductores   │

│   Clientes      │         │   MTW Project    │         │  Independientes │

└─────────────────┘         └──────────────────┘         └─────────────────┘

        │                            │                            │

        │                            │                            │

        ▼                            ▼                            ▼

   Solicitan                    Coordina y                  Prestan el

   servicios                    gestiona                    servicio

        │                            │                            │

        └────────────────────────────┴────────────────────────────┘

                                     │

                                     ▼

                          Facturación mensual/individual

```

 

---

 

## 2. Alcance del Proyecto

 

### 2.1 Volumen de Operaciones Esperado

 

| Métrica | Volumen |

|---------|---------|

| Reservas diarias | ~100 |

| Reservas mensuales | ~3,000 |

| Conductores registrados | ~200 |

| Empresas clientes | Variable |

| Usuarios concurrentes | ~50 |

 

### 2.2 Tipos de Vehículos Soportados

 

El sistema debe soportar múltiples categorías de vehículos:

 

1. **Auto** - Vehículo sedán estándar (4-5 pasajeros)

2. **SUV** - Camioneta deportiva (5-7 pasajeros)

3. **Van** - Furgoneta (8-12 pasajeros)

4. **Minibus** - Minibús (12-20 pasajeros)

5. **Sprinter** - Furgoneta tipo Sprinter (15-20 pasajeros)

6. **Bus** - Bus estándar (20-40 pasajeros)

7. **Cúster** - Bus tipo Cúster (20-30 pasajeros)

 

**Impacto en el Modelo:**

- La entidad `Driver` debe incluir `vehicleType` (tipo de vehículo)

- La entidad `Driver` debe incluir `capacity` (capacidad de pasajeros)

- Las tarifas pueden variar según tipo de vehículo

 

---

 

## 3. Requisitos Funcionales

 

### 3.1 Gestión de Reservas

 

#### RF-001: Crear Reserva

**Prioridad:** ALTA

- Un operador puede crear reservas para empresas clientes

- Campos obligatorios: fecha, hora, empresa, solicitante, pasajero, origen, destino, precio

- Asignación de conductor puede ser inmediata o posterior

- Estado inicial: "En Reserva" (sin conductor) o "Pendiente" (con conductor)

 

#### RF-002: Servicios Recurrentes

**Prioridad:** ALTA

- El sistema debe soportar reservas recurrentes (programadas)

- Ejemplos: "Todos los lunes a las 8:00 AM", "Cada 15 días"

- Patrón de recurrencia: diaria, semanal, quincenal, mensual

- Fecha de inicio y fecha de fin (o sin fin)

- Posibilidad de cancelar instancia individual sin afectar la recurrencia

 

**Modelo Propuesto:**

```java

@Entity

public class RecurringBooking {

    @Id

    private Long idRecurringBooking;

 

    // Patrón de recurrencia

    private RecurrencePattern pattern; // DAILY, WEEKLY, BIWEEKLY, MONTHLY

    private LocalDate startDate;

    private LocalDate endDate; // puede ser null

    private Boolean active;

 

    // Relación con booking template

    @OneToOne

    private Booking bookingTemplate;

 

    // Días de la semana (para semanal)

    private String daysOfWeek; // "1,3,5" para Lunes, Miércoles, Viernes

 

    // Instancias generadas

    @OneToMany

    private List<Booking> generatedBookings;

}

```

 

#### RF-003: Asignación de Conductores

**Prioridad:** ALTA

- Asignación manual por operador (proceso actual)

- El operador contacta al conductor fuera del sistema (teléfono/WhatsApp)

- Al asignar conductor, estado cambia a "Pendiente"

- **Futuro:** Agenda de disponibilidad de conductores

 

#### RF-004: Gestión de Estados de Reserva

**Prioridad:** ALTA

 

Estados del ciclo de vida:

1. **En Reserva** - Creada sin conductor asignado

2. **Pendiente** - Conductor asignado, servicio no iniciado

3. **En Proceso** - Servicio en ejecución

4. **Finalizado** - Servicio completado

5. **Cancelado** - Servicio cancelado (nuevo estado requerido)

 

Transiciones permitidas:

- En Reserva → Pendiente (asignar conductor)

- Pendiente → En Proceso (iniciar servicio)

- En Proceso → Finalizado (completar servicio)

- En Reserva/Pendiente → Cancelado (cancelar reserva)

 

#### RF-005: Filtrado de Reservas

**Prioridad:** MEDIA

- Filtros actuales: ID, solicitante, empresa, pasajero, conductor

- Filtros adicionales necesarios:

  - Por rango de fechas

  - Por estado

  - Por tipo de vehículo

  - Por ruta (origen/destino)

 

### 3.2 Gestión de Tarifas

 

#### RF-006: Sistema de Tarifas Flexible

**Prioridad:** ALTA

 

Factores que afectan la tarifa:

1. **Ruta** - Origen y destino (distancia)

2. **Hora pico** - Horarios de alto tráfico

3. **Tipo de cliente** - Tarifas preferenciales para clientes frecuentes

4. **Tipo de vehículo** - Vehículos más grandes = tarifa mayor

 

**Modelo Propuesto:**

```java

@Entity

public class TariffRule {

    @Id

    private Long idTariffRule;

 

    private String name;

    private String description;

 

    // Tarifa base

    private BigDecimal basePrice;

 

    // Factores de modificación

    private String routePattern; // "Lima-Callao", regex, etc.

    private String vehicleTypes; // "AUTO,SUV", null = todos

    private String clientType; // "PREMIUM", "STANDARD", "VIP"

 

    // Hora pico

    private Boolean isPeakHour;

    private LocalTime peakStartTime;

    private LocalTime peakEndTime;

    private BigDecimal peakMultiplier; // 1.5 = +50%

 

    // Override manual

    private Boolean allowManualOverride; // default: true

 

    private Boolean active;

    private LocalDate validFrom;

    private LocalDate validTo;

}

 

@Entity

public class Booking {

    // ... campos existentes

 

    @ManyToOne

    private TariffRule appliedTariff;

 

    private BigDecimal calculatedPrice; // precio calculado automáticamente

    private BigDecimal finalPrice; // precio final (puede ser modificado por operador)

    private Boolean manualPriceOverride; // indica si se modificó manualmente

    private String priceNotes; // razón del cambio manual

}

```

 

#### RF-007: Modificación Manual de Tarifas

**Prioridad:** ALTA

- Los operadores pueden modificar el precio sugerido

- Se debe registrar que hubo modificación manual

- Opcional: registrar razón del cambio

 

### 3.3 Sistema de Facturación

 

#### RF-008: Facturación Flexible

**Prioridad:** ALTA

 

**Escenario Principal:** Facturación mensual

- Al final del mes, se agrupan todas las reservas finalizadas de una empresa

- El operador selecciona qué reservas incluir en la factura

- Se genera una factura con:

  - Subtotal (suma de precios)

  - IGV (18% en Perú)

  - Total

 

**Escenario Secundario:** Facturación individual

- Se puede facturar un servicio individual inmediatamente

- Útil para clientes ocasionales o servicios especiales

 

**Modelo Actualizado:**

```java

@Entity

public class Bill {

    @Id

    private Long idBill;

 

    private String series;

    private String number;

    private LocalDate issueDate;

 

    @ManyToOne

    private Company company;

 

    private BillType type; // MONTHLY, INDIVIDUAL

    private LocalDate periodStart; // para facturas mensuales

    private LocalDate periodEnd;

 

    private BigDecimal subTotal;

    private BigDecimal igv;

    private BigDecimal total;

 

    @ManyToOne

    private Currency currency;

 

    private String status; // DRAFT, ISSUED, PAID, CANCELLED

 

    // Múltiples reservas

    @OneToMany(mappedBy = "bill")

    private List<Booking> bookings;

}

```

 

#### RF-009: Selección de Reservas para Facturar

**Prioridad:** ALTA

- Interface para seleccionar qué reservas incluir en una factura

- Vista de reservas finalizadas sin facturar

- Posibilidad de excluir reservas específicas

- Cálculo automático de subtotal, IGV y total

 

### 3.4 Portal Cliente Corporativo

 

#### RF-010: Auto-gestión de Reservas

**Prioridad:** MEDIA

- Los clientes corporativos pueden acceder al sistema

- Crear sus propias reservas

- Ver historial de reservas

- Ver facturas emitidas

- Descargar facturas en PDF

- Ver estado de servicios en tiempo real

 

**Roles de Usuario Corporativo:**

- **Administrador Corporativo:** Gestión completa de la cuenta

- **Solicitante:** Solo crear reservas

- **Visualizador:** Solo consultar

 

### 3.5 Gestión de Conductores

 

#### RF-011: Tipos de Conductor

**Prioridad:** MEDIA

 

El sistema debe distinguir:

1. **Conductor Independiente** - Contratista con su propio vehículo

2. **Conductor Empleado** - Personal de la empresa con vehículo de flota

3. **Conductor Híbrido** - Puede ser ambos

 

```java

@Entity

public class Driver {

    // ... campos existentes

 

    private DriverType type; // INDEPENDENT, EMPLOYEE, HYBRID

 

    // Para independientes

    private BigDecimal commissionRate; // % de comisión

 

    // Para empleados

    private String employeeCode;

    private BigDecimal salary;

 

    // Información del vehículo

    private VehicleType vehicleType; // AUTO, SUV, VAN, etc.

    private Integer capacity; // número de pasajeros

    private Boolean hasAirConditioning;

    private Integer vehicleYear;

 

    // Disponibilidad (futuro)

    @OneToMany

    private List<DriverAvailability> availability;

}

```

 

### 3.6 Roles y Permisos del Sistema

 

#### RF-012: Sistema de Roles

**Prioridad:** ALTA

 

| Rol | Permisos |

|-----|----------|

| **Administrador** | Acceso completo al sistema |

| **Operador** | Gestión de reservas, asignación de conductores, modificación de precios |

| **Conductor** | Ver sus asignaciones, actualizar estado de servicio |

| **Cliente Corporativo** | Gestión de sus propias reservas, ver facturas |

| **Contador** | Acceso a facturación, reportes financieros, contabilidad |

 

---

 

## 4. Requisitos No Funcionales

 

### 4.1 Rendimiento

 

- **RNF-001:** El sistema debe soportar 100 reservas diarias (~4-5 por hora en horario laboral)

- **RNF-002:** Tiempo de respuesta de APIs < 500ms para operaciones comunes

- **RNF-003:** Generación de reportes Excel < 5 segundos para 1000 registros

- **RNF-004:** Soporte de hasta 50 usuarios concurrentes

 

### 4.2 Seguridad

 

- **RNF-005:** Implementar autenticación con JWT (prioridad baja pero necesaria)

- **RNF-006:** Encriptación de contraseñas con BCrypt

- **RNF-007:** Control de acceso basado en roles (RBAC)

- **RNF-008:** Logs de auditoría para operaciones críticas

- **RNF-009:** Protección de datos personales de pasajeros

 

### 4.3 Disponibilidad

 

- **RNF-010:** Disponibilidad del sistema: 99% (8.76 horas de downtime/año)

- **RNF-011:** Backups diarios de base de datos

- **RNF-012:** Plan de recuperación ante desastres

 

### 4.4 Usabilidad

 

- **RNF-013:** Interface responsive para uso en tablets/móviles

- **RNF-014:** Mensajes de error claros en español

- **RNF-015:** Validaciones de formularios en tiempo real

 

---

 

## 5. Integraciones Externas

 

### 5.1 Integraciones Prioritarias

 

#### INT-001: Google Maps (o alternativa gratuita)

**Prioridad:** ALTA

**Propósito:** Captura de latitud/longitud para origen y destino

**Alternativas:** OpenStreetMap (gratuita), Mapbox (freemium)

 

**Implementación sugerida:**

```java

@Entity

public class Location {

    @Id

    private Long idLocation;

 

    private String address;

    private String district; // distrito

    private String province; // provincia

    private String department; // departamento

 

    // Coordenadas

    private BigDecimal latitude;

    private BigDecimal longitude;

 

    @ManyToOne

    private Ubigeo ubigeo;

}

 

// En Booking

@ManyToOne

private Location pickUpLocation;

 

@ManyToOne

private Location destinationLocation;

```

 

#### INT-002: WhatsApp Business API

**Prioridad:** ALTA

**Propósito:** Notificaciones de asignación de conductor

 

Mensajes a enviar:

1. **Al Conductor:** "Nueva asignación: [Fecha] [Hora] - Recoger en [Dirección] - Destino [Dirección]"

2. **Al Pasajero:** "Conductor asignado: [Nombre] - [Teléfono] - [Vehículo] [Placa]"

 

**Proveedores sugeridos:**

- Twilio WhatsApp API

- 360Dialog

- Gupshup

 

#### INT-003: Email (SMTP)

**Prioridad:** ALTA

**Propósito:** Notificaciones a clientes corporativos

 

Casos de uso:

- Confirmación de reserva

- Envío de facturas

- Recordatorios de servicios

- Resúmenes mensuales

 

#### INT-004: Sistema Contable Externo

**Prioridad:** MEDIA

**Propósito:** Sincronización de facturas y contabilidad

 

**Fase 1:** Integración con sistema externo

**Fase 2:** Desarrollo de módulo contable propio

 

Sistemas comunes en Perú:

- Siigo

- ContaSOL

- Concar

 

### 5.2 Integraciones Futuras (No Prioritarias)

 

#### INT-005: Geolocalización en Tiempo Real

**Prioridad:** BAJA

**Propósito:** Tracking de conductores durante el servicio

 

#### INT-006: Pasarela de Pago

**Prioridad:** BAJA

**Propósito:** Pagos en línea para clientes no corporativos

 

Opciones para Perú:

- Niubiz (Visa)

- Culqi

- MercadoPago

 

#### INT-007: Sistema de Calificaciones

**Prioridad:** BAJA

**Propósito:** Rating de conductores y pasajeros

 

---

 

## 6. Notificaciones

 

### 6.1 Notificaciones por WhatsApp

 

| Evento | Destinatario | Contenido | Prioridad |

|--------|--------------|-----------|-----------|

| Asignación de conductor | Conductor | Detalles de la reserva | ALTA |

| Asignación de conductor | Pasajero | Datos del conductor | ALTA |

| Recordatorio de servicio | Conductor | 1 hora antes | ALTA |

| Cancelación de servicio | Conductor | Notificación de cancelación | ALTA |

 

### 6.2 Notificaciones Push (Futuro)

 

- Notificaciones push para conductores en app móvil

- Nuevas asignaciones

- Cambios en reservas

- Mensajes del operador

 

### 6.3 Notificaciones Email

 

| Evento | Destinatario | Prioridad |

|--------|--------------|-----------|

| Factura generada | Cliente corporativo | ALTA |

| Resumen mensual | Cliente corporativo | MEDIA |

| Cambio de estado de servicio | Operador | MEDIA |

 

---

 

## 7. Roadmap Priorizado

 

### Fase 1 - MVP Mejorado (Sprints 1-3, 6 semanas)

 

**Objetivos:**

- Estabilizar funcionalidad core

- Implementar servicios recurrentes

- Sistema de tarifas flexible

- Facturación mejorada

 

**Entregables:**

1. ✅ Corrección de tipos de datos (String → BigDecimal)

2. ✅ Validaciones completas

3. ✅ Servicios recurrentes (RecurringBooking)

4. ✅ Sistema de tarifas (TariffRule)

5. ✅ Facturación flexible con selección de reservas

6. ✅ Tipos de vehículos

7. ✅ Tipos de conductor (independiente/empleado)

 

### Fase 2 - Integraciones (Sprints 4-6, 6 semanas)

 

**Objetivos:**

- Conectar con servicios externos

- Automatizar notificaciones

- Mejorar experiencia del usuario

 

**Entregables:**

1. ✅ Integración con Google Maps/OpenStreetMap

2. ✅ Integración con WhatsApp API

3. ✅ Sistema de notificaciones por email

4. ✅ Captura de coordenadas (lat/long)

5. ✅ Plantillas de mensajes WhatsApp

6. ✅ Plantillas de emails

 

### Fase 3 - Portal Cliente (Sprints 7-9, 6 semanas)

 

**Objetivos:**

- Permitir auto-gestión a clientes corporativos

- Sistema de roles y permisos

 

**Entregables:**

1. ✅ Sistema de roles (Admin, Operador, Conductor, Cliente, Contador)

2. ✅ Portal cliente corporativo

3. ✅ Auto-gestión de reservas

4. ✅ Consulta de facturas

5. ✅ Descarga de facturas en PDF

6. ✅ Gestión de usuarios corporativos

 

### Fase 4 - Optimización y Reportes (Sprints 10-12, 6 semanas)

 

**Objetivos:**

- Mejorar rendimiento

- Reportes avanzados

- Testing completo

 

**Entregables:**

1. ✅ Suite de tests (unitarios e integración)

2. ✅ Optimización de consultas

3. ✅ Reportes avanzados (conductores, ingresos, rutas)

4. ✅ Dashboard de analíticas

5. ✅ Exportación de reportes (Excel, PDF)

6. ✅ Manejo centralizado de excepciones

 

### Fase 5 - Seguridad y Escalabilidad (Sprints 13-15, 6 semanas)

 

**Objetivos:**

- Fortalecer seguridad

- Preparar para crecimiento

 

**Entregables:**

1. ✅ Implementación de Spring Security

2. ✅ JWT para autenticación

3. ✅ Encriptación de contraseñas

4. ✅ Auditoría completa

5. ✅ Implementación de DTOs

6. ✅ Paginación en todos los endpoints

 

### Fase 6 - Características Avanzadas (Futuro)

 

**No Prioritarias - Implementar según demanda:**

1. ⏳ Geolocalización en tiempo real

2. ⏳ Agenda de disponibilidad de conductores

3. ⏳ Sistema de calificaciones

4. ⏳ Pasarela de pago

5. ⏳ App móvil para conductores

6. ⏳ Chat interno

7. ⏳ Módulo contable propio

 

---

 

## 8. Restricciones y Supuestos

 

### Restricciones

1. **Geográfica:** Operación inicial solo en Perú

2. **Idioma:** Sistema en español

3. **Moneda:** Soporte principal en Soles (PEN), secundario en Dólares (USD)

4. **Regulación:** Cumplimiento con normativa tributaria peruana (SUNAT)

 

### Supuestos

1. Los conductores tienen acceso a WhatsApp

2. Los clientes corporativos tienen acceso a email

3. Internet disponible para operadores y conductores

4. Coordinación manual con conductores es aceptable en fase inicial

 

---

 

## 9. Criterios de Éxito

 

| Criterio | Métrica | Objetivo |

|----------|---------|----------|

| Reducción de tiempo de gestión | Minutos por reserva | < 3 minutos |

| Satisfacción del cliente | Encuestas | > 4.5/5 |

| Errores de facturación | % de facturas con errores | < 1% |

| Disponibilidad del sistema | Uptime | > 99% |

| Tiempo de respuesta | API response time | < 500ms |

| Adopción del portal cliente | % de clientes usando el portal | > 60% |

 

---

 

## 10. Riesgos y Mitigaciones

 

| Riesgo | Probabilidad | Impacto | Mitigación |

|--------|--------------|---------|------------|

| Falta de conductores disponibles | Media | Alto | Implementar agenda de disponibilidad, ampliar red de conductores |

| Errores de facturación | Baja | Alto | Validaciones estrictas, revisión manual antes de emisión |

| Problemas con WhatsApp API | Media | Medio | Tener plan B con SMS tradicional |

| Bajo uso del portal cliente | Media | Medio | Capacitación, incentivos, interfaz intuitiva |

| Sobrecarga del sistema | Baja | Alto | Monitoreo de rendimiento, escalamiento horizontal si es necesario |

 

---

 

**Documento aprobado por:** [Pendiente]

**Fecha de aprobación:** [Pendiente]

**Próxima revisión:** [Cada 3 meses]