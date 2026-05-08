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
        print(f"⚠️  No se pudo escribir en el log: {e}")
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