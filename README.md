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
