### 📂 Estructura del Código Creado                                                                                                                                                               
                                                                                                                                                                                                    
  Los archivos fuentes se encuentran distribuidos en el directorio  ms-proveedores/src/main/java/cl/bookpointchile/proveedores  de acuerdo al patrón CSR:                                           
                                                                                                                                                                                                    
    ms-proveedores/                                                                                                                                                                                 
    ├── pom.xml                                     # Dependencias del proyecto (Lombok, Validation, JPA)                                                                                           
    └── src/                                                                                                                                                                                        
        └── main/                                                                                                                                                                                   
            ├── java/                                                                                                                                                                               
            │   └── cl/                                                                                                                                                                             
            │       └── bookpointchile/                                                                                                                                                             
            │           └── proveedores/                                                                                                                                                            
            │               ├── config/                                                                                                                                                             
            │               │   └── DataInitializer.java  # Sembrador de proveedores B2B y ODCs base                                                                                                
            │               ├── controller/                                                                                                                                                         
            │               │   └── ProveedoresController.java # Endpoints REST de proveedores y órdenes de compra                                                                                  
            │               ├── dto/                                                                                                                                                                
            │               │   ├── CrearProveedorRequestDTO.java  # JSR 380 para registrar proveedores                                                                                             
            │               │   ├── ProveedorResponseDTO.java      # Representación externa de proveedores                                                                                          
            │               │   ├── DetalleOrdenRequestDTO.java    # JSR 380 para costo y cantidad de productos                                                                                     
            │               │   ├── OrdenCompraRequestDTO.java     # JSR 380 para emisión de ODCs                                                                                                   
            │               │   ├── DetalleOrdenResponseDTO.java   # DTO con subtotales calculados                                                                                                  
            │               │   └── OrdenCompraResponseDTO.java    # DTO consolidado con costo total final                                                                                          
            │               ├── exception/                                                                                                                                                          
            │               │   ├── ResourceNotFoundException.java                                                                                                                                  
            │               │   ├── ProveedorNoEncontradoException.java # HTTP 404                                                                                                                  
            │               │   ├── OrdenCompraInvalidaException.java   # HTTP 400 Bad Request                                                                                                      
            │               │   ├── ErrorResponse.java             # JSON estructurado de error                                                                                                     
            │               │   └── GlobalExceptionHandler.java    # Interceptor global @RestControllerAdvice                                                                                       
            │               ├── model/                                                                                                                                                              
            │               │   ├── EstadoOrden.java               # Enum (PENDIENTE, RECIBIDA, CANCELADA)                                                                                          
            │               │   ├── Proveedor.java                 # Entidad B2B (Editoriales/Distribuidores)                                                                                       
            │               │   ├── OrdenCompra.java               # Entidad cabecera de la orden                                                                                                   
            │               │   └── DetalleOrden.java              # Entidad hija con costo y cantidad solicitada                                                                                   
            │               ├── repository/                                                                                                                                                         
            │               │   ├── ProveedorRepository.java                                                                                                                                        
            │               │   ├── OrdenCompraRepository.java                                                                                                                                      
            │               │   └── DetalleOrdenRepository.java    # Repositorios de persistencia JPA                                                                                               
            │               ├── service/                                                                                                                                                            
            │               │   ├── ProveedoresService.java                                                                                                                                         
            │               │   └── ProveedoresServiceImpl.java    # Lógica de costos y auditoría de transiciones                                                                                   
            │               └── ProveedoresApplication.java        # Bootstrap de la aplicación Spring Boot                                                                                         
            └── resources/                                                                                                                                                                          
                └── application.properties                  # Configuración de puerto 8086, MySQL y logs                                                                                            
  ──────                                                                                                                                                                                            
  ### 🛠️ Decisiones de Desarrollo y Diseño                                                                                                                                                        
                                                                                                                                                                                                    
  1. Protección e Integridad del Flujo de Recepción:                                                                                                                                                
      • En ProveedoresServiceImpl.java se ha implementado una capa de seguridad estricta para la mercadería en  registrarRecepcionMercaderia .                                                                  
          • Si intentas registrar la recepción de una ODC previamente CANCELADA, el sistema bloquea la transacción, escribe una alerta en consola mediante  @Slf4j  ( log.warn ) y arroja una       
          excepción  OrdenCompraInvalidaException . Esto previene de forma contundente desajustes de stock en bodega por mercadería no autorizada.                                                  
          • Si la orden ya figura como  RECIBIDA , el endpoint es idempotente y retorna los datos sin repetir la inserción ni alterar fechas.                                                       
                                                                                                                                                                                                    
  2. Cálculos Financieros Consolidados y Mapeo Preciso:                                                                                                                                             
      • En lugar de almacenar totales redundantes en base de datos, el servicio calcula en tiempo real los subtotales de cada DetalleOrden.java ( cantidadSolicitada * costoUnitario ) y consolida el 
      valor  total  en OrdenCompraResponseDTO.java al momento del mapeo, garantizando consistencia matemática absoluta en la API.                                                                               
  3. Garantía Bidireccional de Relaciones JPA:                                                                                                                                                      
      • OrdenCompra.java gestiona la sincronización con DetalleOrden.java usando los métodos utilitarios  addDetalle  y  removeDetalle , garantizando la correcta vinculación de llaves foráneas en     
      cascada en MySQL.                                                                                                                                                                             
  4. Validaciones Beans Estrictas (JSR 380):                                                                                                                                                        
      • DetalleOrdenRequestDTO.java protege la integridad comercial validando que no se soliciten cantidades en cero o negativas ( @Min(1) ) y que el costo unitario no sea negativo ( @Min(0) ).               
  5. Sembrado Automático de Proveedores B2B (Data Seeder):                                                                                                                                          
      • He programado DataInitializer.java. En el primer arranque, registrará de forma automática las editoriales "Editorial Planeta Chilena S.A.", "Penguin Random House" y "Distribuidora de
Papelería  
      Concepción" e inyectará ODCs de prueba en distintos estados (PENDIENTE, RECIBIDA, CANCELADA) para verificar las transiciones de negocio desde el primer momento.                              
                                                                                                                                                                                                    
  ──────                                                                                                                                                                                            
  ### ⚙️ Propiedades del Entorno                                                                                                                                                                     
                                                                                                                                                                                                    
  En application.properties se han ajustado los siguientes parámetros:                                                                                                                                     
                                                                                                                                                                                                    
  • Puerto:  server.port=8086  (independiente de ms-ventas  8081 , ms-inventario  8082 , ms-usuarios  8083 , ms-catalogo  8084  y ms-logistica  8085 ).                                             
  • Base de datos: MySQL esquema  bookpoint_proveedores  (con  createDatabaseIfNotExist=true ).                                                                                                     
  • Logging: Activado a nivel  INFO  para auditoría y trazabilidad en consola de las compras corporativas B2B.                                                                                      
  ──────                                                                                                                                                                                            
  ### 🔍 Endpoints REST Expuestos                                                                                                                                                                   
                                                                                                                                                                                                    
  •  GET /api/proveedores : Lista todos los proveedores registrados en la base de datos de BookPoint Chile.                                                                                         
  •  POST /api/proveedores : Registra un nuevo proveedor B2B (dispara validación de RUT único e email).                                                                                             
  •  POST /api/proveedores/ordenes : Emite una Orden de Compra (ODC) consolidada, vinculando múltiples libros y calculando los costos ( @Valid ).                                                   
  •  PUT /api/proveedores/ordenes/{id}/recepcion : Registra que la mercadería fue recibida físicamente en Bodega, activando auditorías y comprobaciones de transiciones de estados válidos.
