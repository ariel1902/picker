# Depuracion
#!/usr/bin/env python3
import os
from datetime import datetime

# Arte ASCII para "Depuración de CC"
def mostrar_arte_ascii():
    print("""
    ╔════════════════════════════════════╗
    ║    Depuración de CC - Termux       ║
    ║════════════════════════════════════║
    """)

# Mostrar menú principal
def mostrar_menu():
    print("\nMenú Principal:")
    print("1. Separar lives")
    print("2. Organizar por fechas")
    print("3. Guardar archivo")
    print("4. Salir del programa")
    return input("Selecciona una opción (1-4): ")

# Validar formato de tarjeta
def validar_formato_tarjeta(linea, es_nueva_lista=False):
    partes = linea.strip().split("|")
    if es_nueva_lista:
        if len(partes) != 4 or not all(partes):
            return False
        try:
            mes = int(partes[1].strip())
            año = int(partes[2].strip())
            return 1 <= mes <= 12 and 2020 <= año <= 2099
        except ValueError:
            return False
    return len(partes) == 5 and all(partes)

# Opción 1: Separar lives
def separar_lives(lista_tarjetas):
    print("\nIngresa tu lista de tarjetas (máximo 100 filas). Presiona Enter dos veces para finalizar:")
    nuevas_tarjetas = []
    ultimo_enter = False
    while len(nuevas_tarjetas) < 100:
        print(f"Filas restantes: {100 - len(nuevas_tarjetas)}")
        linea = input()
        if linea == "":
            if ultimo_enter:
                break
            ultimo_enter = True
            continue
        ultimo_enter = False
        if validar_formato_tarjeta(linea):
            nuevas_tarjetas.append(linea)
        else:
            print("Formato inválido. Ejemplo: 4830310088657755 | 119 | 08 | 2031 | > Live Card")
    if len(nuevas_tarjetas) >= 100:
        print("Advertencia: Se alcanzó el límite de 100 filas.")
    # Filtrar solo las tarjetas con "> Live Card"
    lista_tarjetas = [linea for linea in nuevas_tarjetas if "> Live Card" in linea]
    print(f"Operación exitosa: Se encontraron {len(lista_tarjetas)} tarjetas Live.")
    input("Presiona Enter para volver al menú principal...")
    return lista_tarjetas

# Opción 2: Organizar por fechas
def organizar_por_fechas(lista_tarjetas):
    opcion = input("\n¿Quieres usar la lista actual (1) o cargar una nueva lista (2)? (1/2): ")
    if opcion == "1" and not lista_tarjetas:
        print("Error: No hay lista actual para procesar.")
        input("Presiona Enter para volver al menú principal...")
        return lista_tarjetas
    elif opcion == "2":
        print("\nIngresa tu lista de tarjetas (máximo 100 filas). Presiona Enter dos veces para finalizar:")
        nuevas_tarjetas = []
        ultimo_enter = False
        while len(nuevas_tarjetas) < 100:
            print(f"Filas restantes: {100 - len(nuevas_tarjetas)}")
            linea = input()
            if linea == "":
                if ultimo_enter:
                    break
                ultimo_enter = True
                continue
            ultimo_enter = False
            if validar_formato_tarjeta(linea, es_nueva_lista=True):
                nuevas_tarjetas.append(linea)
            else:
                print("Formato inválido. Ejemplo: 4830310088657755 | 08 | 2031 | 119")
        if len(nuevas_tarjetas) >= 100:
            print("Advertencia: Se alcanzó el límite de 100 filas.")
        lista_tarjetas = nuevas_tarjetas
    try:
        # Filtrar tarjetas con fecha posterior a 07/2025
        fecha_limite = datetime(2025, 7, 1)
        tarjetas_filtradas = []
        for linea in lista_tarjetas:
            partes = linea.strip().split("|")
            if "> Live Card" in linea:
                mes = int(partes[2].strip())
                año = int(partes[3].strip())
            else:
                mes = int(partes[1].strip())
                año = int(partes[2].strip())
            fecha_tarjeta = datetime(año, mes, 1)
            if fecha_tarjeta > fecha_limite:
                tarjetas_filtradas.append(linea)
        print(f"Operación exitosa: Se encontraron {len(tarjetas_filtradas)} tarjetas con fecha válida.")
        input("Presiona Enter para volver al menú principal...")
        return tarjetas_filtradas
    except ValueError:
        print("Error: Formato de fecha inválido en alguna tarjeta.")
        input("Presiona Enter para volver al menú principal...")
        return lista_tarjetas
    except Exception as e:
        print(f"Error al procesar fechas: {e}")
        input("Presiona Enter para volver al menú principal...")
        return lista_tarjetas

# Opción 3: Guardar archivo
def guardar_archivo(lista_tarjetas):
    if not lista_tarjetas:
        print("Error: No hay datos para guardar.")
        input("Presiona Enter para volver al menú principal...")
        return
    print("\nOpciones de guardado:")
    print("1. Guardar en ubicación por defecto (/storage/emulated/0/Download/CC lives)")
    print("2. Guardar en carpeta específica")
    opcion = input("Selecciona una opción (1/2): ")
    if opcion == "1":
        carpeta = "/storage/emulated/0/Download/CC lives"
        try:
            os.makedirs(carpeta, exist_ok=True)
        except Exception as e:
            print(f"Error al crear carpeta por defecto: {e}")
            print("Asegúrate de haber ejecutado 'termux-setup-storage'.")
            input("Presiona Enter para volver al menú principal...")
            return
    elif opcion == "2":
        carpeta = input("Ingresa la ruta completa de la carpeta (ejemplo: /storage/emulated/0/MisArchivos): ")
        if not os.path.exists(carpeta):
            print("Error: La carpeta especificada no existe.")
            input("Presiona Enter para volver al menú principal...")
            return
    else:
        print("Opción inválida.")
        input("Presiona Enter para volver al menú principal...")
        return
    nombre_archivo = input("Ingresa el nombre del archivo (sin .txt): ")
    ruta_completa = os.path.join(carpeta, f"{nombre_archivo}.txt")
    try:
        with open(ruta_completa, 'w') as file:
            for linea in lista_tarjetas:
                file.write(linea + '\n')
        print(f"Archivo guardado exitosamente en {ruta_completa}")
    except PermissionError:
        print("Error: No tienes permisos para escribir en esa ubicación. Ejecuta 'termux-setup-storage'.")
    except Exception as e:
        print(f"Error al guardar el archivo: {e}")
    input("Presiona Enter para volver al menú principal...")

# Verificar permisos de almacenamiento
def verificar_permisos_almacenamiento():
    if not os.path.exists("/storage/emulated/0"):
        print("Advertencia: No se detectó acceso al almacenamiento. Ejecuta 'termux-setup-storage' primero.")
        input("Presiona Enter para continuar...")

# Programa principal
def main():
    verificar_permisos_almacenamiento()
    lista_tarjetas = []  # Almacena la lista de tarjetas procesada
    while True:
        mostrar_arte_ascii()
        opcion = mostrar_menu()
        if opcion == "1":
            lista_tarjetas = separar_lives(lista_tarjetas)
        elif opcion == "2":
            lista_tarjetas = organizar_por_fechas(lista_tarjetas)
        elif opcion == "3":
            guardar_archivo(lista_tarjetas)
        elif opcion == "4":
            print("Saliendo del programa...")
            break
        else:
            print("Opción inválida. Por favor, selecciona una opción válida (1-4).")
            input("Presiona Enter para continuar...")

if __name__ == "__main__":
    main()
