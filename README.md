# Integración de la librería DLL con Python

Este proyecto demuestra cómo cargar y usar una biblioteca DLL de C++ desde Python mediante la librería estándar `ctypes`.

La DLL expone funciones para crear figuras geométricas, calcular su perímetro y su área, y liberar memoria de forma segura.

## Estructura del proyecto

```text
.
├── engine/
│   ├── Figura.h
│   ├── engine_api.cpp
│   ├── figuras_engine.dll
│   └── figuras_engine__.dll
├── uso-libreria-dll.ipynb
├── README.md
├── requirements.txt
└── .venv/
```

## Requisitos

- Python 3.10 o superior
- Windows 10/11
- La DLL debe corresponder a la misma arquitectura que el intérprete de Python:
  - Python x64 requiere DLL x64
  - Python x86 requiere DLL x86
- Si la DLL fue compilada con Visual C++ o con dependencias del runtime de C++, puede requerirse el paquete redistribuible de Microsoft Visual C++.

## Instalación

1. Clona o descarga el proyecto.
2. Asegúrate de que el archivo `engine/figuras_engine.dll` exista.
3. Crea un entorno virtual si lo deseas:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

4. Instala los requisitos del proyecto:

```powershell
pip install -r requirements.txt
```

> En este caso, no hay dependencias externas adicionales; la integración se realiza con la librería estándar `ctypes`.

## Integración con Python

El siguiente ejemplo carga la DLL y usa las funciones exportadas:

```python
import ctypes
import os

# Ruta de la DLL
base_dir = os.path.dirname(os.path.abspath(__file__))
dll_path = os.path.join(base_dir, "engine", "figuras_engine.dll")

# Carga la biblioteca
engine = ctypes.CDLL(dll_path)

# Firma de la función: crea un cuadro con un lado
engine.crearCuadro.argtypes = [ctypes.c_double]
engine.crearCuadro.restype = ctypes.c_void_p

# Firma de la función: calcula el área
engine.calcularArea.argtypes = [ctypes.c_void_p]
engine.calcularArea.restype = ctypes.c_double

# Firma de la función: calcula el perímetro
engine.calcularPerimetro.argtypes = [ctypes.c_void_p]
engine.calcularPerimetro.restype = ctypes.c_double

# Firma de la función: libera memoria
engine.destruirFigura.argtypes = [ctypes.c_void_p]
engine.destruirFigura.restype = None

lado = 4.0
figura = engine.crearCuadro(lado)
area = engine.calcularArea(figura)
perimetro = engine.calcularPerimetro(figura)

print(f"Área del cuadrado con lado {lado}: {area}")
print(f"Perímetro del cuadrado con lado {lado}: {perimetro}")

engine.destruirFigura(figura)
```

## Funciones exportadas por la DLL

La capa DLL expone funciones C estilo para la creación y manipulación de figuras:

- `crearCirculo(double radio)`
- `crearCuadro(double lado)`
- `crearTriangulo(double base, double altura, double l1, double l2, double l3)`
- `crearPoligono(int numLados, double longitudLado, double apotema)`
- `calcularPerimetro(Figura* fig)`
- `calcularArea(Figura* fig)`
- `calcularVolumen(Figura* fig)`
- `destruirFigura(Figura* fig)`

La DLL usa punteros a objetos en memoria creada en C++; por eso se devuelve una referencia de tipo `void*` en Python, que luego se pasa de nuevo a cada función que opera con la figura.

## Importante: manejo de memoria

La instancia creada en C++ debe destruirse explícitamente desde Python con `destruirFigura(figura)` para evitar fugas de memoria.

## Consideraciones de compatibilidad

- No mezcles DLL de 32 bits con Python de 64 bits.
- Asegúrate de que la DLL y el script se ejecuten desde la misma máquina y arquitectura.
- Si la DLL es nueva o se actualiza, verifica su nombre y ruta antes de cargarla.
- Si aparecen errores como `WinError 193` o `WinError 126`, normalmente indican incompatibilidad de arquitectura o dependencias faltantes del runtime.

## Ejemplo de uso del notebook

El notebook `uso-libreria-dll.ipynb` usa este patrón básico:

```python
lado = 4.0
figura = engine.crearCuadro(lado)
area = engine.calcularArea(figura)
perimetro = engine.calcularPerimetro(figura)
print(f"Área del cuadrado con lado {lado}: {area}")
print(f"Perímetro del cuadrado con lado {lado}: {perimetro}")
engine.destruirFigura(figura)
```

## Resumen

La integración funciona con `ctypes`, cargando la DLL desde el proyecto, definiendo los tipos de entrada y salida de cada función, y gestionando la destrucción de los objetos creados en C++.

Esto permite combinar la lógica geométrica implementada en C++ con un flujo de trabajo más sencillo en Python.
