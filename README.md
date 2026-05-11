# Ejercicio-1-RAC-3-FASE-4
Sistema Integral de Gestión de Clientes, Servicios y Reservas.
"""
=============================================================
  SISTEMA INTEGRAL DE GESTIÓN - SOFTWARE FJ
  Programación Orientada a Objetos | Python 3
  Manejo robusto de excepciones, clases abstractas,
  herencia, polimorfismo y encapsulación.
=============================================================
"""

import os
import datetime
from abc import ABC, abstractmethod  # Para clases abstractas


# ─────────────────────────────────────────────────────────────
# SECCIÓN 1: ARCHIVO DE LOGS
# ─────────────────────────────────────────────────────────────

LOG_FILE = "softwarefj_eventos.log"

def registrar_log(nivel: str, mensaje: str):
    """
    Registra un evento o error en el archivo de logs.
    nivel: "INFO", "ERROR", "ADVERTENCIA"
    """
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    linea = f"[{timestamp}] [{nivel}] {mensaje}\n"
    try:
        with open(LOG_FILE, "a", encoding="utf-8") as f:
            f.write(linea)
    except IOError as e:
        # Si no podemos escribir el log, al menos lo mostramos en pantalla
        print(f"  No se pudo escribir en el log: {e}")
    print(linea.strip())  # También imprimimos en consola


# ─────────────────────────────────────────────────────────────
# SECCIÓN 2: EXCEPCIONES PERSONALIZADAS
# ─────────────────────────────────────────────────────────────

class SoftwareFJError(Exception):
    """Excepción base del sistema. Todas las demás heredan de esta."""
    def __init__(self, mensaje: str, codigo: int = 0):
        super().__init__(mensaje)
        self.codigo = codigo  # Código de error interno
        self.mensaje = mensaje

class ClienteInvalidoError(SoftwareFJError):
    """Se lanza cuando los datos de un cliente son incorrectos."""
    pass

class ServicioNoDisponibleError(SoftwareFJError):
    """Se lanza cuando un servicio no está habilitado o no existe."""
    pass

class ReservaInvalidaError(SoftwareFJError):
    """Se lanza cuando una reserva tiene parámetros incorrectos."""
    pass

class PagoInsuficienteError(SoftwareFJError):
    """Se lanza cuando el monto de pago no cubre el costo."""
    pass

class CapacidadExcedidaError(SoftwareFJError):
    """Se lanza cuando se supera la capacidad máxima de un servicio."""
    pass


# ─────────────────────────────────────────────────────────────
# SECCIÓN 3: CLASE ABSTRACTA BASE (EntidadSistema)
# ─────────────────────────────────────────────────────────────

class EntidadSistema(ABC):
    """
    Clase abstracta base para todas las entidades del sistema.
    Define la interfaz común que deben implementar todas las entidades.
    """
    _contador_ids = 0  # Variable de clase: contador global de IDs

    def __init__(self, nombre: str):
        # Validación básica en el constructor
        if not nombre or not isinstance(nombre, str):
            raise SoftwareFJError("El nombre de la entidad no puede estar vacío.")
        EntidadSistema._contador_ids += 1
        self._id = EntidadSistema._contador_ids  # ID autoincremental
        self._nombre = nombre.strip()
        self._fecha_registro = datetime.datetime.now()

    @property
    def id(self):
        """Getter encapsulado para el ID."""
        return self._id

    @property
    def nombre(self):
        """Getter encapsulado para el nombre."""
        return self._nombre

    @abstractmethod
    def describir(self) -> str:
        """Método abstracto: cada entidad debe describirse a sí misma."""
        pass

    @abstractmethod
    def validar(self) -> bool:
        """Método abstracto: cada entidad debe poder validarse."""
        pass

    def __str__(self):
        return f"[{self.__class__.__name__} #{self._id}] {self._nombre}"


        # ─────────────────────────────────────────────────────────────
# SECCIÓN 4: CLASE CLIENTE (hereda de EntidadSistema)
# ─────────────────────────────────────────────────────────────

class Cliente(EntidadSistema):
    """
    Representa a un cliente de Software FJ.
    Aplica encapsulación estricta y validaciones robustas.
    """

    def __init__(self, nombre: str, email: str, telefono: str, cedula: str):
        try:
            super().__init__(nombre)  # Llamamos al constructor padre
            # Usamos setters para aprovechar las validaciones
            self.email = email
            self.telefono = telefono
            self.cedula = cedula
            self._reservas = []  # Lista interna de reservas del cliente
        except SoftwareFJError:
            raise  # Re-lanzamos la excepción original

    # ── PROPIEDADES (getters y setters con validación) ──

    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, valor: str):
        """Valida que el email tenga formato básico correcto."""
        if not valor or "@" not in valor or "." not in valor.split("@")[-1]:
            raise ClienteInvalidoError(
                f"Email inválido: '{valor}'. Debe contener '@' y dominio.",
                codigo=101
            )
        self._email = valor.strip().lower()

    @property
    def telefono(self):
        return self._telefono

    @telefono.setter
    def telefono(self, valor: str):
        """Valida que el teléfono solo contenga dígitos y tenga longitud mínima."""
        limpio = str(valor).replace("-", "").replace(" ", "").replace("+", "")
        if not limpio.isdigit() or len(limpio) < 7:
            raise ClienteInvalidoError(
                f"Teléfono inválido: '{valor}'. Mínimo 7 dígitos.",
                codigo=102
            )
        self._telefono = valor.strip()

    @property
    def cedula(self):
        return self._cedula

    @cedula.setter
    def cedula(self, valor: str):
        """Valida que la cédula sea numérica y tenga longitud válida."""
        limpio = str(valor).strip()
        if not limpio.isdigit() or not (6 <= len(limpio) <= 12):
            raise ClienteInvalidoError(
                f"Cédula inválida: '{valor}'. Debe tener entre 6 y 12 dígitos.",
                codigo=103
            )
        self._cedula = limpio

    def agregar_reserva(self, reserva):
        """Asocia una reserva al historial del cliente."""
        self._reservas.append(reserva)

    def obtener_reservas(self):
        """Retorna una copia de la lista de reservas (encapsulación)."""
        return list(self._reservas)

    def describir(self) -> str:
        return (f"Cliente: {self._nombre} | Cédula: {self._cedula} "
                f"| Email: {self._email} | Tel: {self._telefono} "
                f"| Reservas: {len(self._reservas)}")

    def validar(self) -> bool:
        """Verifica que todos los campos obligatorios estén presentes."""
        return all([self._nombre, self._email, self._telefono, self._cedula])
# ─────────────────────────────────────────────────────────────
# SECCIÓN 5: CLASE ABSTRACTA SERVICIO + 3 ESPECIALIZACIONES
# ─────────────────────────────────────────────────────────────

class Servicio(EntidadSistema):
    """
    Clase abstracta para todos los servicios de Software FJ.
    Define la interfaz polimórfica de cálculo de costos.
    """
    IVA = 0.19  # IVA colombiano del 19%

    def __init__(self, nombre: str, precio_base: float, disponible: bool = True):
        super().__init__(nombre)
        if precio_base < 0:
            raise ServicioNoDisponibleError(
                f"El precio base no puede ser negativo: {precio_base}", codigo=201
            )
        self._precio_base = precio_base
        self._disponible = disponible

    @property
    def precio_base(self):
        return self._precio_base

    @property
    def disponible(self):
        return self._disponible

    @disponible.setter
    def disponible(self, valor: bool):
        self._disponible = valor

    def verificar_disponibilidad(self):
        """Lanza excepción si el servicio no está disponible."""
        if not self._disponible:
            raise ServicioNoDisponibleError(
                f"El servicio '{self._nombre}' no está disponible actualmente.",
                codigo=202
            )

    # ── MÉTODOS SOBRECARGADOS (calcular_costo) ──
    # Python no tiene sobrecarga nativa, la simulamos con parámetros opcionales

    def calcular_costo(self, horas: float = 1, descuento: float = 0.0,
                       aplicar_iva: bool = True, personas: int = 1) -> float:
        """
        Calcula el costo total del servicio.
        Método sobrecargado: acepta diferentes combinaciones de parámetros.

        horas: duración en horas
        descuento: porcentaje de descuento (0.0 a 1.0)
        aplicar_iva: si se aplica IVA al costo
        personas: número de personas (para servicios grupales)
        """
        if horas <= 0:
            raise ReservaInvalidaError("Las horas deben ser un valor positivo.", codigo=301)
        if not (0 <= descuento <= 1):
            raise ReservaInvalidaError("El descuento debe estar entre 0 y 1.", codigo=302)

        costo = self._calcular_costo_especifico(horas, personas)
        costo -= costo * descuento  # Aplicar descuento
        if aplicar_iva:
            costo += costo * self.IVA
        return round(costo, 2)

    @abstractmethod
    def _calcular_costo_especifico(self, horas: float, personas: int) -> float:
        """Método que cada servicio especializado implementa a su manera (polimorfismo)."""
        pass

    @abstractmethod
    def describir(self) -> str:
        pass

    def validar(self) -> bool:
        return self._precio_base >= 0 and self._nombre != ""


# ── SERVICIO 1: ReservaSala ──

class ReservaSala(Servicio):
    """
    Servicio de reserva de salas de reuniones o eventos.
    Precio varía según la capacidad de la sala.
    """
    def __init__(self, nombre: str, capacidad_max: int, precio_hora: float):
        super().__init__(nombre, precio_hora)
        if capacidad_max <= 0:
            raise ServicioNoDisponibleError("La capacidad debe ser mayor a 0.", codigo=203)
        self._capacidad_max = capacidad_max

    def _calcular_costo_especifico(self, horas: float, personas: int) -> float:
        """
        Una sala cobra por hora. Si hay muchas personas,
        se aplica un recargo del 10% por sobrepoblación.
        """
        if personas > self._capacidad_max:
            raise CapacidadExcedidaError(
                f"La sala '{self._nombre}' tiene capacidad máxima de "
                f"{self._capacidad_max} personas. Se solicitaron {personas}.",
                codigo=401
            )
        costo = self._precio_base * horas
        if personas > self._capacidad_max * 0.8:  # Más del 80% de capacidad
            costo *= 1.10  # Recargo del 10%
        return costo

    def describir(self) -> str:
        estado = "Disponible" if self._disponible else "No disponible"
        return (f"🏢 Sala: {self._nombre} | Capacidad: {self._capacidad_max} pax "
                f"| ${self._precio_base:,.0f}/hora | {estado}")


# ── SERVICIO 2: AlquilerEquipo ──

class AlquilerEquipo(Servicio):
    """
    Servicio de alquiler de equipos tecnológicos.
    Incluye verificación de stock disponible.
    """
    def __init__(self, nombre: str, precio_dia: float, unidades_disponibles: int):
        super().__init__(nombre, precio_dia)
        if unidades_disponibles < 0:
            raise ServicioNoDisponibleError("Las unidades no pueden ser negativas.", codigo=204)
        self._unidades_disponibles = unidades_disponibles

    def _calcular_costo_especifico(self, horas: float, personas: int) -> float:
        """
        El equipo cobra por hora (fracción de día).
        Si se alquilan varias unidades (personas), el precio es por unidad.
        """
        if personas > self._unidades_disponibles:
            raise CapacidadExcedidaError(
                f"Solo hay {self._unidades_disponibles} unidades de '{self._nombre}'.",
                codigo=402
            )
        return self._precio_base * horas * personas

    def reducir_stock(self, cantidad: int):
        """Reduce el inventario disponible al hacer una reserva."""
        if cantidad > self._unidades_disponibles:
            raise CapacidadExcedidaError("Stock insuficiente.", codigo=403)
        self._unidades_disponibles -= cantidad

    def describir(self) -> str:
        estado = "Disponible" if self._disponible else "No disponible"
        return (f"💻 Equipo: {self._nombre} | Stock: {self._unidades_disponibles} "
                f"| ${self._precio_base:,.0f}/hora/unidad | {estado}")


# ── SERVICIO 3: AsesoriaEspecializada ──

class AsesoriaEspecializada(Servicio):
    """
    Servicio de asesorías técnicas o especializadas.
    El precio varía según el nivel del asesor (junior/senior/experto).
    """
    NIVELES_VALIDOS = {"junior": 1.0, "senior": 1.5, "experto": 2.0}

    def __init__(self, nombre: str, precio_base_hora: float, nivel_asesor: str):
        super().__init__(nombre, precio_base_hora)
        nivel = nivel_asesor.lower()
        if nivel not in self.NIVELES_VALIDOS:
            raise ServicioNoDisponibleError(
                f"Nivel inválido: '{nivel_asesor}'. Use: junior, senior, experto.",
                codigo=205
            )
        self._nivel = nivel
        self._multiplicador = self.NIVELES_VALIDOS[nivel]

    def _calcular_costo_especifico(self, horas: float, personas: int) -> float:
        """
        Las asesorías cobran por hora con multiplicador según el nivel del asesor.
        Las asesorías grupales tienen un pequeño descuento por persona adicional.
        """
        costo_base = self._precio_base * self._multiplicador * horas
        if personas > 1:
            # Descuento del 5% por cada persona adicional (máximo 20%)
            descuento_grupal = min(0.05 * (personas - 1), 0.20)
            costo_base *= (1 - descuento_grupal)
        return costo_base

    def describir(self) -> str:
        estado = "Disponible" if self._disponible else "No disponible"
        return (f"👨‍💼 Asesoría: {self._nombre} | Nivel: {self._nivel.capitalize()} "
                f"| ${self._precio_base:,.0f}/hora base | {estado}")


# ─────────────────────────────────────────────────────────────
# SECCIÓN 6: CLASE RESERVA
# ─────────────────────────────────────────────────────────────

class Reserva(EntidadSistema):
    """
    Integra un Cliente, un Servicio, duración y estado.
    Gestiona el ciclo de vida: pendiente → confirmada → cancelada.
    """
    ESTADOS = {"pendiente", "confirmada", "cancelada", "completada"}

    def __init__(self, cliente: Cliente, servicio: Servicio,
                 horas: float, personas: int = 1,
                 descuento: float = 0.0, aplicar_iva: bool = True):

        # Validaciones de entrada
        if not isinstance(cliente, Cliente):
            raise ReservaInvalidaError("Se requiere un objeto Cliente válido.", codigo=301)
        if not isinstance(servicio, Servicio):
            raise ReservaInvalidaError("Se requiere un objeto Servicio válido.", codigo=302)

        super().__init__(f"Reserva-{cliente.nombre}-{servicio.nombre}")

        self._cliente = cliente
        self._servicio = servicio
        self._horas = horas
        self._personas = personas
        self._descuento = descuento
        self._aplicar_iva = aplicar_iva
        self._estado = "pendiente"
        self._costo_total = 0.0
        self._fecha_reserva = datetime.datetime.now()

    def confirmar(self):
        """
        Confirma la reserva: verifica disponibilidad y calcula costo.
        Usa try/except/else/finally para manejo robusto.
        """
        try:
            # Verificar que el servicio esté disponible
            self._servicio.verificar_disponibilidad()

            # Calcular el costo (puede lanzar excepciones)
            self._costo_total = self._servicio.calcular_costo(
                horas=self._horas,
                descuento=self._descuento,
                aplicar_iva=self._aplicar_iva,
                personas=self._personas
            )

        except ServicioNoDisponibleError as e:
            # Encadenamiento de excepciones: añadimos contexto
            raise ReservaInvalidaError(
                f"No se pudo confirmar la reserva: {e}"
            ) from e

        except (CapacidadExcedidaError, ReservaInvalidaError):
            raise  # Re-lanzamos sin modificar

        else:
            # Este bloque se ejecuta SOLO si no hubo excepciones
            self._estado = "confirmada"
            self._cliente.agregar_reserva(self)
            registrar_log("INFO",
                f"Reserva #{self._id} CONFIRMADA | Cliente: {self._cliente.nombre} "
                f"| Servicio: {self._servicio.nombre} | Costo: ${self._costo_total:,.2f}"
            )

        finally:
            # Este bloque se ejecuta SIEMPRE (con o sin error)
            registrar_log("INFO",
                f"Proceso de confirmación finalizado para Reserva #{self._id}"
            )

    def cancelar(self, motivo: str = "Sin motivo especificado"):
        """Cancela la reserva si está en estado permitido."""
        try:
            if self._estado == "cancelada":
                raise ReservaInvalidaError(
                    "La reserva ya está cancelada.", codigo=303
                )
            if self._estado == "completada":
                raise ReservaInvalidaError(
                    "No se puede cancelar una reserva completada.", codigo=304
                )
            self._estado = "cancelada"
            registrar_log("ADVERTENCIA",
                f"Reserva #{self._id} CANCELADA | Motivo: {motivo}"
            )
        except ReservaInvalidaError as e:
            registrar_log("ERROR", f"Intento de cancelación fallido: {e}")
            raise

    def procesar_pago(self, monto_pagado: float):
        """
        Procesa el pago de la reserva confirmada.
        Ejemplo de try/except/finally con encadenamiento de excepciones.
        """
        try:
            if self._estado != "confirmada":
                raise ReservaInvalidaError(
                    f"Solo se pueden pagar reservas confirmadas. Estado actual: {self._estado}",
                    codigo=305
                )
            if monto_pagado < self._costo_total:
                # Lanzamos excepción encadenada con más contexto
                diferencia = self._costo_total - monto_pagado
                raise PagoInsuficienteError(
                    f"Pago insuficiente. Faltan ${diferencia:,.2f}. "
                    f"Total requerido: ${self._costo_total:,.2f}",
                    codigo=501
                )
        except PagoInsuficienteError as e:
            registrar_log("ERROR", f"Pago rechazado para Reserva #{self._id}: {e}")
            raise
        else:
            cambio = monto_pagado - self._costo_total
            self._estado = "completada"
            registrar_log("INFO",
                f"PAGO EXITOSO Reserva #{self._id} | "
                f"Pagado: ${monto_pagado:,.2f} | Cambio: ${cambio:,.2f}"
            )
            return cambio
        finally:
            registrar_log("INFO", f"Proceso de pago finalizado para Reserva #{self._id}")

    def describir(self) -> str:
        return (f"📋 Reserva #{self._id} | {self._cliente.nombre} → {self._servicio.nombre} "
                f"| {self._horas}h | {self._personas} persona(s) "
                f"| Estado: {self._estado.upper()} | Total: ${self._costo_total:,.2f}")

    def validar(self) -> bool:
        return (self._cliente.validar() and
                self._servicio.validar() and
                self._horas > 0 and
                self._personas > 0)


# ─────────────────────────────────────────────────────────────
# SECCIÓN 7: SISTEMA PRINCIPAL (gestiona listas de objetos)
# ─────────────────────────────────────────────────────────────

class SistemaGestion:
    """
    Controlador principal. Gestiona clientes, servicios y reservas
    mediante listas internas de objetos (sin base de datos).
    """

    def __init__(self):
        self._clientes: list[Cliente] = []
        self._servicios: list[Servicio] = []
        self._reservas: list[Reserva] = []
        registrar_log("INFO", "=== Sistema Software FJ iniciado ===")

    # ── GESTIÓN DE CLIENTES ──

    def registrar_cliente(self, nombre, email, telefono, cedula) -> Cliente | None:
        """Registra un cliente con manejo de excepciones."""
        try:
            cliente = Cliente(nombre, email, telefono, cedula)
            if not cliente.validar():
                raise ClienteInvalidoError("El cliente no pasó la validación.", codigo=104)
            self._clientes.append(cliente)
            registrar_log("INFO", f"Cliente registrado: {cliente.describir()}")
            return cliente
        except ClienteInvalidoError as e:
            registrar_log("ERROR", f"Error al registrar cliente '{nombre}': {e}")
            return None
        except SoftwareFJError as e:
            registrar_log("ERROR", f"Error general al registrar cliente: {e}")
            return None

    def buscar_cliente(self, cedula: str) -> Cliente | None:
        """Busca un cliente por cédula. Retorna None si no existe."""
        for c in self._clientes:
            if c.cedula == cedula:
                return c
        return None

    # ── GESTIÓN DE SERVICIOS ──

    def agregar_servicio(self, servicio: Servicio) -> bool:
        """Añade un servicio al catálogo."""
        try:
            if not isinstance(servicio, Servicio):
                raise ServicioNoDisponibleError("Objeto de servicio inválido.", codigo=206)
            self._servicios.append(servicio)
            registrar_log("INFO", f"Servicio añadido: {servicio.describir()}")
            return True
        except ServicioNoDisponibleError as e:
            registrar_log("ERROR", f"No se pudo agregar el servicio: {e}")
            return False

    def buscar_servicio(self, nombre: str) -> Servicio | None:
        """Busca un servicio por nombre (insensible a mayúsculas)."""
        nombre_lower = nombre.lower()
        for s in self._servicios:
            if nombre_lower in s.nombre.lower():
                return s
        return None

    # ── GESTIÓN DE RESERVAS ──

    def crear_reserva(self, cliente: Cliente, servicio: Servicio,
                      horas: float, personas: int = 1,
                      descuento: float = 0.0) -> Reserva | None:
        """Crea y confirma una reserva con manejo completo de excepciones."""
        try:
            reserva = Reserva(cliente, servicio, horas, personas, descuento)
            reserva.confirmar()
            self._reservas.append(reserva)
            return reserva
        except ReservaInvalidaError as e:
            registrar_log("ERROR", f"Reserva rechazada: {e}")
            return None
        except CapacidadExcedidaError as e:
            registrar_log("ERROR", f"Capacidad excedida: {e}")
            return None
        except SoftwareFJError as e:
            registrar_log("ERROR", f"Error al crear reserva: {e}")
            return None

    # ── REPORTES ──

    def mostrar_reporte(self):
        """Muestra un resumen completo del sistema."""
        sep = "─" * 60
        print(f"\n{'='*60}")
        print("  📊 REPORTE DEL SISTEMA - SOFTWARE FJ")
        print(f"{'='*60}")

        print(f"\n👥 CLIENTES REGISTRADOS ({len(self._clientes)}):")
        print(sep)
        for c in self._clientes:
            print(f"  {c.describir()}")

        print(f"\n🛠️  SERVICIOS DISPONIBLES ({len(self._servicios)}):")
        print(sep)
        for s in self._servicios:
            print(f"  {s.describir()}")

        print(f"\n📋 RESERVAS ({len(self._reservas)}):")
        print(sep)
        total_ingresos = 0
        for r in self._reservas:
            print(f"  {r.describir()}")
            if r._estado in ("confirmada", "completada"):
                total_ingresos += r._costo_total

        print(f"\n  💰 Total de ingresos potenciales: ${total_ingresos:,.2f}")
        print(f"{'='*60}\n")


# ─────────────────────────────────────────────────────────────
# SECCIÓN 8: SIMULACIÓN DE 10+ OPERACIONES
# ─────────────────────────────────────────────────────────────

def ejecutar_simulacion():
    """
    Simula al menos 10 operaciones completas incluyendo
    casos válidos e inválidos para demostrar el manejo de excepciones.
    """
    sistema = SistemaGestion()
    print("\n" + "="*60)
    print("  🚀 INICIANDO SIMULACIÓN - SOFTWARE FJ")
    print("="*60)

    # ── OPERACIÓN 1: Registrar clientes válidos ──
    print("\n📌 OP 1 & 2: Registrar clientes válidos")
    c1 = sistema.registrar_cliente("Ana Gómez", "ana@correo.com", "3101234567", "1010101010")
    c2 = sistema.registrar_cliente("Carlos Ruiz", "carlos@empresa.co", "6017654321", "987654321")

    # ── OPERACIÓN 2: Registrar cliente con datos inválidos ──
    print("\n📌 OP 3: Registrar cliente con email inválido (debe fallar)")
    c_malo1 = sistema.registrar_cliente("Pedro Malo", "emailsinArroba", "3009999999", "12345678")

    # ── OPERACIÓN 3: Registrar cliente con cédula inválida ──
    print("\n📌 OP 4: Registrar cliente con cédula inválida (debe fallar)")
    c_malo2 = sistema.registrar_cliente("Luisa Falla", "luisa@mail.com", "3109876543", "abc")

    # ── OPERACIÓN 4: Crear servicios válidos ──
    print("\n📌 OP 5: Crear servicios del catálogo")
    sala_a   = ReservaSala("Sala Innovación", capacidad_max=20, precio_hora=150_000)
    laptop   = AlquilerEquipo("Laptop Dell i7", precio_dia=50_000, unidades_disponibles=5)
    asesoria = AsesoriaEspecializada("Consultoría Cloud", precio_base_hora=200_000, nivel_asesor="senior")

    sistema.agregar_servicio(sala_a)
    sistema.agregar_servicio(laptop)
    sistema.agregar_servicio(asesoria)

    # ── OPERACIÓN 5: Crear servicio con parámetros inválidos ──
    print("\n📌 OP 6: Crear servicio con precio negativo (debe fallar)")
    try:
        servicio_malo = ReservaSala("Sala Inválida", 10, precio_hora=-5000)
        sistema.agregar_servicio(servicio_malo)
    except ServicioNoDisponibleError as e:
        registrar_log("ERROR", f"Servicio inválido rechazado: {e}")

    # ── OPERACIÓN 6: Crear asesoría con nivel inválido ──
    print("\n📌 OP 7: Crear asesoría con nivel inexistente (debe fallar)")
    try:
        asesoria_mala = AsesoriaEspecializada("Asesoría X", 100_000, "dios")
        sistema.agregar_servicio(asesoria_mala)
    except ServicioNoDisponibleError as e:
        registrar_log("ERROR", f"Nivel de asesor inválido: {e}")

    # ── OPERACIÓN 7: Reserva exitosa ──
    print("\n📌 OP 8: Reserva exitosa - sala por 3 horas")
    if c1 and sala_a:
        r1 = sistema.crear_reserva(c1, sala_a, horas=3, personas=10, descuento=0.05)

    # ── OPERACIÓN 8: Reserva con capacidad excedida ──
    print("\n📌 OP 9: Reserva con demasiadas personas (debe fallar)")
    if c2 and sala_a:
        r2 = sistema.crear_reserva(c2, sala_a, horas=2, personas=50)

    # ── OPERACIÓN 9: Reserva de equipo exitosa ──
    print("\n📌 OP 10: Reserva de laptops exitosa")
    if c1 and laptop:
        r3 = sistema.crear_reserva(c1, laptop, horas=8, personas=3, descuento=0.0)

    # ── OPERACIÓN 10: Reserva de servicio deshabilitado ──
    print("\n📌 OP 11: Reserva de servicio deshabilitado (debe fallar)")
    sala_a.disponible = False
    if c2 and sala_a:
        r_fallida = sistema.crear_reserva(c2, sala_a, horas=1, personas=5)
    sala_a.disponible = True  # Rehabilitamos

    # ── OPERACIÓN 11: Procesar pago exitoso ──
    print("\n📌 OP 12: Procesar pago de reserva confirmada")
    if r1 and r1._estado == "confirmada":
        try:
            cambio = r1.procesar_pago(1_000_000)
            print(f"  ✅ Pago procesado. Cambio devuelto: ${cambio:,.2f}")
        except PagoInsuficienteError as e:
            print(f"  ❌ {e}")

    # ── OPERACIÓN 12: Pago insuficiente ──
    print("\n📌 OP 13: Pago insuficiente (debe fallar)")
    if r3 and r3._estado == "confirmada":
        try:
            r3.procesar_pago(100)  # Monto muy bajo
        except PagoInsuficienteError as e:
            print(f"  ❌ Error esperado: {e}")

    # ── OPERACIÓN 13: Cancelar una reserva ──
    print("\n📌 OP 14: Cancelar reserva de laptops")
    if r3 and r3._estado == "confirmada":
        try:
            r3.cancelar("Cliente solicitó cancelación por cambio de fecha")
        except ReservaInvalidaError as e:
            print(f"  ❌ {e}")

    # ── OPERACIÓN 14: Asesoría grupal con descuento automático ──
    print("\n📌 OP 15: Asesoría grupal con descuento automático")
    if c2 and asesoria:
        r4 = sistema.crear_reserva(c2, asesoria, horas=2, personas=5, descuento=0.10)

    # ── REPORTE FINAL ──
    sistema.mostrar_reporte()
    print(f"\n📄 Todos los eventos han sido registrados en: {LOG_FILE}")
