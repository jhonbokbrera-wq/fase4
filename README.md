# fase4
Fase 4 componente practico - practicas simuladas

# Software FJ - Sistema de gestión de clientes, servicios y reservas.

# Este proyecto es un sistema hecho en Python con programación orientada
# a objetos para una empresa ficticia, Software FJ, que ofrece reservas de
# salas, alquiler de equipos y asesorías. No usa base de datos: todo se
# maneja con objetos en memoria, y el único archivo que se escribe es
# `software_fj.log`, donde quedan registrados los eventos y los errores.

from abc import ABC, abstractmethod
import logging

class ErrorApp(Exception):
    """Excepción base para errores de la aplicación."""
    pass

class ClienteError(ErrorApp):
    """Errores relacionados con el cliente."""
    pass

class ServicioError(ErrorApp):
    """Errores relacionados con el servicio."""
    pass

class ReservaError(ErrorApp):
    """Errores relacionados con la reserva."""
    pass


def setup_logger():
    logger = logging.getLogger("software_FJ")
    logger.setLevel(logging.INFO)


    if not logger.handlers:
        manejador = logging.FileHandler("software_fj.log", encoding='utf-8')
        formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
        manejador.setFormatter(formatter)
        logger.addHandler(manejador)
        
    return logger

logger = setup_logger()


class Entidad(ABC):
    def __init__(self, id_entidad):
        self._id = id_entidad
        
    @property
    def id(self):
        return self._id
    
    @abstractmethod
    def describir(self):
        pass  
    
# Cosas que se aplicaron

# Abstracción, herencia, polimorfismo y encapsulación.
# Como Python no tiene sobrecarga de métodos, se simuló con parámetros
# opcionales: `calcular_precio(horas)` o `calcular_precio(horas, impuesto, descuento)`.
# En `procesar()` se usa `try/except/else/finally`, y si el servicio falla
# se lanza un `ReservaError` con `raise ... from`, para no perder la causa original.

class Cliente(Entidad):
    def __init__(self, id_cliente, nombre, correo, telefono):
        super().__init__(id_cliente)
        
        if not nombre or not nombre.strip():
            raise ClienteError("El nombre del cliente no puede estar vacío.") 
        self.__nombre = nombre.strip()
        
        if not correo or not correo.strip():
            raise ClienteError("El correo del cliente no puede estar vacío.")
        if not correo or "@" not in correo or "." not in correo:
            raise ClienteError("El correo del cliente no es válido.")
        self.__correo = correo.strip()
        
        if not str(telefono).isdigit():
            raise ClienteError("El teléfono del cliente debe ser un número válido.")
        self.__telefono = str(telefono)
        
    @property
    def nombre(self):
        return self.__nombre
    
    def describir(self):
        return f"Cliente ID: {self.id}, Nombre: {self.nombre}, Correo: {self.__correo}, Teléfono: {self.__telefono}"
    
    @property
    def correo(self):
        return self.__correo
    
    @property
    def telefono(self):
        return self.__telefono
    
class Servicio(Entidad):
    def __init__(self, id_servicio, nombre, tarifa_base):
        super().__init__(id_servicio)
        
        if not nombre or not nombre.strip():
            raise ServicioError("El nombre del servicio no puede estar vacío.")
        if not isinstance(tarifa_base, (int, float)) or tarifa_base <= 0:
            raise ServicioError("La tarifa base del servicio debe ser un número mayor que cero.")
        self._nombre = nombre.strip()
        self._tarifa_base = tarifa_base
        
    @property
    def nombre(self):
        return self._nombre
    
    @abstractmethod
    def calcular_precio(self, horas, impuesto=0, descuento=0):
        pass
    
    @abstractmethod
    def validar_parametros(self, horas):
        pass

    def _validar_porcentajes(self, impuesto, descuento):
        if not isinstance(impuesto, (int, float)) or isinstance(impuesto, bool):
            raise ServicioError("El impuesto debe estar entre 0 y 1.")
        if not (0 <= impuesto <= 1):
            raise ServicioError("El impuesto debe estar entre 0 y 1.")
        if not isinstance(descuento, (int, float)) or isinstance(descuento, bool):
            raise ServicioError("El descuento debe estar entre 0 y 1.")
        if not (0 <= descuento <= 1):
            raise ServicioError("El descuento debe estar entre 0 y 1.")
        
class ReservaSala(Servicio):
    def __init__(self, id_servicio, nombre, tarifa_base, capacidad):
        super().__init__(id_servicio, nombre, tarifa_base)
        if not isinstance(capacidad, int) or capacidad <= 0:
            raise ServicioError("La capacidad de la sala debe ser un número entero positivo.")
        self.__capacidad = capacidad
        
    def validar_parametros(self, horas):
        if not isinstance(horas, (int, float)) or horas <= 0:
            raise ServicioError("La cantidad de horas debe ser un número positivo.")
        
            
    def calcular_precio(self, horas, impuesto=0, descuento=0):
        self.validar_parametros(horas)
        self._validar_porcentajes(impuesto, descuento)
        precio_base = self.__capacidad * self._tarifa_base * horas
        precio_final = precio_base + (precio_base * impuesto) - (precio_base * descuento)
        return precio_final
        
    def describir(self):
        return f"Reserva de Sala ID: {self.id}, Nombre: {self.nombre}, Tarifa Base: {self._tarifa_base}, Capacidad: {self.__capacidad}"
           
            
class AlquilerEquipo(Servicio):
    def __init__(self, id_servicio, nombre, tarifa_base, tipo_equipo):
        super().__init__(id_servicio, nombre, tarifa_base)
        if not tipo_equipo or not tipo_equipo.strip():
            raise ServicioError("El tipo de equipo no puede estar vacío.")
        self.__tipo_equipo = tipo_equipo.strip()
        
    def validar_parametros(self, horas):
        if not isinstance(horas, (int, float)) or horas <= 0:
            raise ServicioError("La cantidad de horas debe ser un número positivo.")
        
    def calcular_precio(self, horas, impuesto=0, descuento=0):
        self.validar_parametros(horas)
        self._validar_porcentajes(impuesto, descuento)
        recargo_transporte = 15000
        precio_base = self._tarifa_base * horas
        precio_final = precio_base + recargo_transporte + (precio_base * impuesto) - (precio_base * descuento)
        return precio_final
        
    def describir(self):
        return f"Alquiler de Equipo ID: {self.id}, Nombre: {self.nombre}, Tarifa Base: {self._tarifa_base}, Tipo de Equipo: {self.__tipo_equipo}"
    

class Asesoria(Servicio):
    def __init__(self, id_servicio, nombre, tarifa_base, tipo_asesoria):
        super().__init__(id_servicio, nombre, tarifa_base)
        if not tipo_asesoria or not tipo_asesoria.strip():
            raise ServicioError("El tipo de asesoría no puede estar vacío.")
        self.__tipo_asesoria = tipo_asesoria.strip()
        
    def validar_parametros(self, horas):
        if not isinstance(horas, (int, float)) or horas <= 0:
            raise ServicioError("La cantidad de horas debe ser un número positivo.")
        
    def calcular_precio(self, horas, impuesto=0, descuento=0):
        self.validar_parametros(horas)
        self._validar_porcentajes(impuesto, descuento)
        precio_base = self._tarifa_base * horas
        precio_final = precio_base + (precio_base * impuesto) - (precio_base * descuento)
        return precio_final
        
    def describir(self):
        return f"Asesoría ID: {self.id}, Nombre: {self.nombre}, Tarifa Base: {self._tarifa_base}, Tipo de Asesoría: {self.__tipo_asesoria}"
    
class Reserva:
    def __init__(self, id_reserva, cliente, servicio, horas, impuesto=0, descuento=0):
        if not isinstance(cliente, Cliente):
            raise ReservaError("El cliente debe ser una instancia de la clase Cliente.")
        if not isinstance(servicio, Servicio):
            raise ReservaError("El servicio debe ser una instancia de la clase Servicio.")
        if not isinstance(horas, (int, float)) or horas <= 0:
            raise ReservaError("La cantidad de horas debe ser un número positivo.")
        
        self._id_reserva = id_reserva
        self._cliente = cliente
        self._servicio = servicio
        self._horas = horas
        self._impuesto = impuesto
        self._descuento = descuento
        self._estado = "Pendiente"
        
    @property
    def estado(self):
        return self._estado
        
    @property
    def id_reserva(self):
        return self._id_reserva
    
    @property
    def cliente(self):
        return self._cliente
    
    @property
    def servicio(self):
        return self._servicio
    
    @property
    def horas(self):
        return self._horas
    
    @property
    def impuesto(self):
        return self._impuesto
    
    @property
    def descuento(self):
        return self._descuento
    
    def calcular_precio_total(self):
        precio_total = self._servicio.calcular_precio(self._horas, self._impuesto, self._descuento)
        return precio_total
    
    def describir(self):
        return (f"Reserva ID: {self.id_reserva}, Cliente: {self.cliente.nombre}, "
                f"Servicio: {self.servicio.nombre}, Horas: {self.horas}, "
                f"Impuesto: {self.impuesto*100}%, Descuento: {self.descuento*100}%, "
                f"Precio Total: {self.calcular_precio_total()}")
    
    def confirmar(self):
        if self._estado != "Pendiente":
            raise ReservaError("Solo se pueden confirmar reservas pendientes.")
        self._estado = "Confirmada"
        logger.info(f"Reserva ID: {self.id_reserva} confirmada para el cliente {self.cliente.nombre}.")
        
    def cancelar(self):
        if self._estado != "Pendiente":
            raise ReservaError("Solo se pueden cancelar reservas pendientes.")
        self._estado = "Cancelada"
        logger.info(f"Reserva ID: {self.id_reserva} cancelada para el cliente {self.cliente.nombre}.")    
    
    def procesar(self):
        try:
            costo = self.calcular_precio_total()
        except ServicioError as e:
            logger.error(f"Error al calcular el precio total de la reserva ID: {self.id_reserva}. Detalles: {str(e)}")
            raise ReservaError(f"No se pudo procesar la reserva ID: {self.id_reserva} debido a un error en el servicio.") from e
        else:
            self.confirmar()
            logger.info(f"Reserva ID: {self.id_reserva} procesada exitosamente. Costo total: {costo}.")
            return costo
        finally:
            logger.info(f"Proceso de reserva ID: {self.id_reserva} finalizado con estado: {self.estado}.")  

# Qué errores se prueban

# Datos inválidos de clientes y servicios, horas negativas, cliente que no
# es un `Cliente`, confirmar dos veces la misma reserva, confirmar una
# reserva cancelada y un descuento mayor a 1 (cálculo inconsistente).  

class GestorReservas:
    def __init__(self):
        self._clientes = {}
        self._servicios = {}
        self._reservas = {}
        
    def registrar_cliente(self, cliente):
        if not isinstance(cliente, Cliente):
            raise ClienteError("El cliente debe ser una instancia de la clase Cliente.")
        self._clientes[cliente.id] = cliente
        logger.info(f"Cliente registrado: {cliente.nombre} (ID: {cliente.id})")      
      
    def listar_reservas(self):
        return list(self._reservas())

def ejecutar(descripcion, operacion):
    print(f"\n--- {descripcion} ---")
    try:
        resultado = operacion()
        
    except ErrorApp as e:
        logger.error(f"Error en la operación '{descripcion}': {str(e)}")
        print("Error detectado:", e)
        if e.__cause__:
            print("Causa original:", e.__cause__)
            
    except Exception as e:
        logger.error(f"Error inesperado en la operación '{descripcion}': {str(e)}")
        print("Error inesperado:", e)
    else:
        logger.info(f"Operación '{descripcion}' ejecutada exitosamente.")
        print("Operación ejecutada exitosamente.")
        print("Resultado:", resultado)
        
def main():
    cliente = Cliente(10, "Ana", "ana@example.com", "3001234567")
    sala = ReservaSala(10, "Sala Principal", 50000, 20)
    
    ejecutar("1. cliente valido", lambda: Cliente(1, "Camila", "camila@example.com", "3127890987").describir())
    
    ejecutar("2. cliente inválido", lambda: Cliente(2, "jose", "joseexample.com", "3127890988").describir())
    
    ejecutar("3. reserva de sala válida", lambda: ReservaSala(1, "Sala de Conferencias", 50000, 50).describir())
    
    ejecutar("4. reserva de sala inválida", lambda: ReservaSala(2, "Sala de Reuniones", 40000, -10).describir())
    
    ejecutar("5. alquiler de equipo válido", lambda: AlquilerEquipo(3, "Proyector HD", 20000, "Proyector").describir())
    
    ejecutar("6. alquiler de equipo inválido", lambda: AlquilerEquipo(4, "Portátil", 30000, "").describir())
    
    ejecutar("7. asesoría válida", lambda: Asesoria(5, "Asesoría Legal", 100000, "Legal").describir())
    
    ejecutar("8. asesoría inválida", lambda: Asesoria(6, "Asesoría Financiera", 80000, "").describir())
    
    ejecutar("9. Reserva exitosa (crear y procesar)", lambda: Reserva(1, cliente, sala, 2, impuesto=0.19).procesar())

    ejecutar("10. Reserva con horas negativas", lambda: Reserva(2, cliente, sala, -5))

    ejecutar("11. Reserva con cliente inválido", lambda: Reserva(3, "no soy un cliente", sala, 2))
    reserva = Reserva(4, cliente, sala, 2)
    ejecutar("12. Confirmar una reserva dos veces", lambda: (reserva.confirmar(), reserva.confirmar()))
    reserva2 = Reserva(5, cliente, sala, 2)
    ejecutar("13. Cancelar y luego confirmar", lambda: (reserva2.cancelar(), reserva2.confirmar()))
    
    ejecutar("14. Cálculo inconsistente (descuento mayor a 1)", lambda: Reserva(6, cliente, sala, 2, descuento=5).procesar())   

main()
