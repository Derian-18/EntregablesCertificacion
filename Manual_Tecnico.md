# Manual Técnico - Sistema de Estudiantes y cursos (EC0835)

## Indice
1. Introducción
2. Objetivo general
3. Requerimientos de instalacion
4. Diagramas tecnicos
   - Diagrama de clases
   - Diagrama Entidad-Relaacion
6. Desarrollo y código de la aplicacion
7. Procedimiento de instalación y ejecucion
8. Generación del ejecutable

## 1. Introducción
El sistema de Gestión de Estudiantes y Cursos es una aplicación de escritorio desarrollada en Python utilizando: 
  - Tkinter como librería gráfica para la interfaz de usuario.
  - SQLite como motor de base de datos local.

El objetivo es demostrar la certificación EC0835 que se denominan en las operaciones básicas de programación y gestión de bases de datos:
  - Inserción, selección, actualización y eliminación de registros.
  - Uso de consultas con JOIN y GROUP BY.
  - Validación del funcionamiento a través de la interfaz y la base de datos.

## 2. Objetivo general
Desarrollar y documentar un sistema CRUD completo para la  gestión de estudiantes y cursos,
capaz de ejecutarse en cualquier equipo de Windows y entregarse com producto formato .exe.

## 3. Objetivos especificos
  - Registrar, editar, eliminar y consultar Estudiantes.
  - Registrar, editar, eliminar y con consultar Cursos.
  - Registrar, editar, eliminar y consultar inscripciones.
  - Implementar consultas SQL que:
      - Relacionen cursos con sus estudiantes (JOIN).
      - Generen un resumen de cursos por usuario (GROUP BY).
  - Entregar el sistema en formato digital y ejecutable (.exe).

## 4. Requerimientos de instalacion
### Hardware minimo
  - Procesador: Dual Core 2.0 GHz
  - Ram: 4 GB
  - Disco: 500 MB libres

### Software
  - Sistema operativo: Windows 10/11
  - Python: 3.10 o superior
  - SQLite: Integrado en Python
  - Dependencias (requirements.txt):
      ```bash
       tk
       pyinstaller
      ````
   
## 5. Diagramas tecnicos

### 6.1 Diagrama de clases (UML)
