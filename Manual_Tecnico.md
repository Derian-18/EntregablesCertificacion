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
   
## 5. Diagramas técnicos

### 6.1 Diagrama de clases (UML)
<p align="center">
  <img src="docs/Diagrama%20UML.jpg" alt="Diagrama UML" width="700">
</p>

### 6.2. Diagrama Entidad-Relacion (ER)
<p align="center">
  <img src="docs/Diagrama%20ER%20.jpg" alt="Diagrama UML" width="700">
</p>

## 7. Desarrollo técnico y código
### Estructura del proyecto
```bash
proyecto_certificacion/
|  main.py            # Interfaz Tkinter
|  crud.py            # Funciones de  base de datos (CRUD)
|  models.py          # Clases Estudiante, Curso e inscripciones
|  db.py              # Conexión SQLite y creación de tablas
|  requirements.txt   # Dependcias del proyectpo
|_ certificacion.db   # Base de datos SQLite
```

### Código de aplicacion

#### main.py
```python
import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3

import crud
from db import init_db


class Aplicacion:
    def __init__(self, ventana):
        self.ventana = ventana
        self.ventana.title("Sistema de estudiantes")
        self.ventana.geometry("950x600")
        self.ventana.minsize(800, 500)

        # Inicializar la base de datos
        init_db()

        # Diccionarios para relacionar las opciones de los combos con sus ID
        self.alumnos_por_opcion = {}
        self.cursos_por_opcion = {}

        # Crear pestañas
        self.notebook = ttk.Notebook(self.ventana)
        self.notebook.pack(fill="both", expand=True, padx=10, pady=10)

        self.crear_pestana_alumnos()
        self.crear_pestana_cursos()
        self.crear_pestana_inscripciones()

    # =========================================================
    # MÉTODOS GENERALES
    # =========================================================

    def crear_pestana(self, nombre, campos, columnas):
        """
        Crea una pestaña con campos de entrada y una tabla.
        Se utiliza para Alumnos y Cursos.
        """
        frame = ttk.Frame(self.notebook, padding=10)
        self.notebook.add(frame, text=nombre)

        # Formulario
        formulario = ttk.LabelFrame(
            frame,
            text=f"Datos de {nombre.lower()}",
            padding=10
        )
        formulario.pack(fill="x", pady=(0, 10))

        entradas = {}

        for fila, (campo, etiqueta) in enumerate(campos.items()):
            ttk.Label(
                formulario,
                text=f"{etiqueta}:"
            ).grid(row=fila, column=0, padx=5, pady=5, sticky="w")

            entrada = ttk.Entry(formulario, width=35)
            entrada.grid(row=fila, column=1, padx=5, pady=5, sticky="ew")

            entradas[campo] = entrada

        formulario.columnconfigure(1, weight=1)

        # Botones
        botones = ttk.Frame(frame)
        botones.pack(fill="x", pady=(0, 10))

        # Tabla
        tabla_frame = ttk.Frame(frame)
        tabla_frame.pack(fill="both", expand=True)

        tabla = ttk.Treeview(
            tabla_frame,
            columns=tuple(columnas.keys()),
            show="headings",
            height=12
        )

        for columna, titulo in columnas.items():
            tabla.heading(columna, text=titulo)
            tabla.column(columna, width=150, anchor="center")

        scroll_vertical = ttk.Scrollbar(
            tabla_frame,
            orient="vertical",
            command=tabla.yview
        )
        tabla.configure(yscrollcommand=scroll_vertical.set)

        tabla.pack(side="left", fill="both", expand=True)
        scroll_vertical.pack(side="right", fill="y")

        return frame, entradas, tabla, botones

    def agregar_botones(self, contenedor, acciones):
        """Agrega botones a un contenedor."""
        for texto, comando in acciones:
            ttk.Button(
                contenedor,
                text=texto,
                command=comando
            ).pack(side="left", padx=5)

    def limpiar_tabla(self, tabla):
        """Elimina todas las filas de una tabla."""
        for elemento in tabla.get_children():
            tabla.delete(elemento)

    def obtener_id_seleccionado(self, tabla):
        """Obtiene el ID de la fila seleccionada."""
        seleccion = tabla.selection()

        if not seleccion:
            messagebox.showwarning(
                "Selección requerida",
                "Selecciona un registro de la tabla."
            )
            return None

        valores = tabla.item(seleccion[0], "values")

        if not valores:
            return None

        return int(valores[0])

    def refrescar_combos(self):
        """
        Actualiza las listas desplegables cuando cambian
        los alumnos o los cursos.
        """
        if hasattr(self, "combo_alumno"):
            self.cargar_alumnos_combo()

        if hasattr(self, "combo_curso"):
            self.cargar_cursos_combo()

    # =========================================================
    # ALUMNOS
    # =========================================================

    def crear_pestana_alumnos(self):
        campos = {
            "nombre": "Nombre",
            "matricula": "Matrícula"
        }

        columnas = {
            "id": "ID",
            "nombre": "Nombre",
            "matricula": "Matrícula"
        }

        (
            self.alumno_frame,
            self.alumno_entradas,
            self.alumno_tabla,
            botones
        ) = self.crear_pestana("Alumnos", campos, columnas)

        self.agregar_botones(
            botones,
            [
                ("Crear", self.crear_alumno),
                ("Actualizar", self.actualizar_alumno),
                ("Eliminar", self.eliminar_alumno),
                ("Limpiar", self.limpiar_alumno),
                ("Recargar", self.cargar_alumnos)
            ]
        )

        self.alumno_tabla.bind(
            "<<TreeviewSelect>>",
            self.seleccionar_alumno
        )

        self.cargar_alumnos()

    def cargar_alumnos(self):
        try:
            self.limpiar_tabla(self.alumno_tabla)

            alumnos = crud.leer_alumnos()

            for alumno in alumnos:
                self.alumno_tabla.insert(
                    "",
                    "end",
                    values=alumno
                )

            self.refrescar_combos()

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudieron cargar los alumnos:\n{error}"
            )

    def seleccionar_alumno(self, evento=None):
        seleccion = self.alumno_tabla.selection()

        if not seleccion:
            return

        valores = self.alumno_tabla.item(
            seleccion[0],
            "values"
        )

        if len(valores) < 3:
            return

        self.alumno_entradas["nombre"].delete(0, tk.END)
        self.alumno_entradas["nombre"].insert(0, valores[1])

        self.alumno_entradas["matricula"].delete(0, tk.END)
        self.alumno_entradas["matricula"].insert(0, valores[2])

    def crear_alumno(self):
        nombre = self.alumno_entradas["nombre"].get().strip()
        matricula = self.alumno_entradas["matricula"].get().strip()

        if not nombre or not matricula:
            messagebox.showwarning(
                "Campos requeridos",
                "Ingresa el nombre y la matrícula."
            )
            return

        try:
            crud.crear_alumno(nombre, matricula)

            messagebox.showinfo(
                "Éxito",
                "Alumno creado correctamente."
            )

            self.limpiar_alumno()
            self.cargar_alumnos()

        except sqlite3.IntegrityError:
            messagebox.showerror(
                "Error",
                "La matrícula ya existe. Ingresa una diferente."
            )

        except (sqlite3.Error, ValueError) as error:
            messagebox.showerror(
                "Error",
                f"No se pudo crear el alumno:\n{error}"
            )

    def actualizar_alumno(self):
        alumno_id = self.obtener_id_seleccionado(
            self.alumno_tabla
        )

        if alumno_id is None:
            return

        nombre = self.alumno_entradas["nombre"].get().strip()
        matricula = self.alumno_entradas["matricula"].get().strip()

        if not nombre or not matricula:
            messagebox.showwarning(
                "Campos requeridos",
                "Ingresa el nombre y la matrícula."
            )
            return

        try:
            actualizado = crud.actualizar_alumno(
                alumno_id,
                nombre,
                matricula
            )

            if actualizado:
                messagebox.showinfo(
                    "Éxito",
                    "Alumno actualizado correctamente."
                )
            else:
                messagebox.showwarning(
                    "Aviso",
                    "No se encontró el alumno."
                )

            self.limpiar_alumno()
            self.cargar_alumnos()

        except sqlite3.IntegrityError:
            messagebox.showerror(
                "Error",
                "La matrícula ya pertenece a otro alumno."
            )

        except (sqlite3.Error, ValueError) as error:
            messagebox.showerror(
                "Error",
                f"No se pudo actualizar el alumno:\n{error}"
            )

    def eliminar_alumno(self):
        alumno_id = self.obtener_id_seleccionado(
            self.alumno_tabla
        )

        if alumno_id is None:
            return

        confirmar = messagebox.askyesno(
            "Confirmar eliminación",
            "¿Deseas eliminar este alumno?"
        )

        if not confirmar:
            return

        try:
            eliminado = crud.eliminar_alumno(alumno_id)

            if eliminado:
                messagebox.showinfo(
                    "Éxito",
                    "Alumno eliminado correctamente."
                )
            else:
                messagebox.showwarning(
                    "Aviso",
                    "No se encontró el alumno."
                )

            self.limpiar_alumno()
            self.cargar_alumnos()

        except sqlite3.IntegrityError:
            messagebox.showerror(
                "No se puede eliminar",
                "Este alumno tiene inscripciones registradas. "
                "Elimina primero sus inscripciones."
            )

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudo eliminar el alumno:\n{error}"
            )

    def limpiar_alumno(self):
        for entrada in self.alumno_entradas.values():
            entrada.delete(0, tk.END)

        for elemento in self.alumno_tabla.selection():
            self.alumno_tabla.selection_remove(elemento)

    # =========================================================
    # CURSOS
    # =========================================================

    def crear_pestana_cursos(self):
        campos = {
            "nombreCurso": "Nombre del curso",
            "creditos": "Créditos (número)"
        }

        columnas = {
            "id": "ID",
            "nombreCurso": "Nombre del curso",
            "creditos": "Créditos"
        }

        (
            self.curso_frame,
            self.curso_entradas,
            self.curso_tabla,
            botones
        ) = self.crear_pestana("Cursos", campos, columnas)

        self.agregar_botones(
            botones,
            [
                ("Crear", self.crear_curso),
                ("Actualizar", self.actualizar_curso),
                ("Eliminar", self.eliminar_curso),
                ("Limpiar", self.limpiar_curso),
                ("Recargar", self.cargar_cursos)
            ]
        )

        self.curso_tabla.bind(
            "<<TreeviewSelect>>",
            self.seleccionar_curso
        )

        self.cargar_cursos()

    def cargar_cursos(self):
        try:
            self.limpiar_tabla(self.curso_tabla)

            cursos = crud.leer_cursos()

            for curso in cursos:
                self.curso_tabla.insert(
                    "",
                    "end",
                    values=curso
                )

            self.refrescar_combos()

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudieron cargar los cursos:\n{error}"
            )

    def seleccionar_curso(self, evento=None):
        seleccion = self.curso_tabla.selection()

        if not seleccion:
            return

        valores = self.curso_tabla.item(
            seleccion[0],
            "values"
        )

        if len(valores) < 3:
            return

        self.curso_entradas["nombreCurso"].delete(0, tk.END)
        self.curso_entradas["nombreCurso"].insert(0, valores[1])

        self.curso_entradas["creditos"].delete(0, tk.END)
        self.curso_entradas["creditos"].insert(0, valores[2])

    def crear_curso(self):
        nombre = self.curso_entradas["nombreCurso"].get().strip()
        creditos_texto = self.curso_entradas["creditos"].get().strip()

        if not nombre or not creditos_texto:
            messagebox.showwarning(
                "Campos requeridos",
                "Ingresa el nombre y los créditos."
            )
            return

        try:
            creditos = int(creditos_texto)

            crud.crear_curso(nombre, creditos)

            messagebox.showinfo(
                "Éxito",
                "Curso creado correctamente."
            )

            self.limpiar_curso()
            self.cargar_cursos()

        except ValueError:
            messagebox.showerror(
                "Dato inválido",
                "Los créditos deben ser un número entero."
            )

        except sqlite3.IntegrityError as error:
            messagebox.showerror(
                "Error de integridad",
                f"No se pudo crear el curso:\n{error}"
            )

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudo crear el curso:\n{error}"
            )

    def actualizar_curso(self):
        curso_id = self.obtener_id_seleccionado(
            self.curso_tabla
        )

        if curso_id is None:
            return

        nombre = self.curso_entradas["nombreCurso"].get().strip()
        creditos_texto = self.curso_entradas["creditos"].get().strip()

        if not nombre or not creditos_texto:
            messagebox.showwarning(
                "Campos requeridos",
                "Ingresa el nombre y los créditos."
            )
            return

        try:
            creditos = int(creditos_texto)

            actualizado = crud.actualizar_curso(
                curso_id,
                nombre,
                creditos
            )

            if actualizado:
                messagebox.showinfo(
                    "Éxito",
                    "Curso actualizado correctamente."
                )
            else:
                messagebox.showwarning(
                    "Aviso",
                    "No se encontró el curso."
                )

            self.limpiar_curso()
            self.cargar_cursos()

        except ValueError:
            messagebox.showerror(
                "Dato inválido",
                "Los créditos deben ser un número entero."
            )

        except sqlite3.IntegrityError as error:
            messagebox.showerror(
                "Error de integridad",
                f"No se pudo actualizar el curso:\n{error}"
            )

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudo actualizar el curso:\n{error}"
            )

    def eliminar_curso(self):
        curso_id = self.obtener_id_seleccionado(
            self.curso_tabla
        )

        if curso_id is None:
            return

        confirmar = messagebox.askyesno(
            "Confirmar eliminación",
            "¿Deseas eliminar este curso?"
        )

        if not confirmar:
            return

        try:
            eliminado = crud.eliminar_curso(curso_id)

            if eliminado:
                messagebox.showinfo(
                    "Éxito",
                    "Curso eliminado correctamente."
                )
            else:
                messagebox.showwarning(
                    "Aviso",
                    "No se encontró el curso."
                )

            self.limpiar_curso()
            self.cargar_cursos()

        except sqlite3.IntegrityError:
            messagebox.showerror(
                "No se puede eliminar",
                "Este curso tiene inscripciones registradas. "
                "Elimina primero sus inscripciones."
            )

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudo eliminar el curso:\n{error}"
            )

    def limpiar_curso(self):
        for entrada in self.curso_entradas.values():
            entrada.delete(0, tk.END)

        for elemento in self.curso_tabla.selection():
            self.curso_tabla.selection_remove(elemento)

    # =========================================================
    # INSCRIPCIONES
    # =========================================================

    def crear_pestana_inscripciones(self):
        self.inscripcion_frame = ttk.Frame(
            self.notebook,
            padding=10
        )

        self.notebook.add(
            self.inscripcion_frame,
            text="Inscripciones"
        )

        # Formulario con listas desplegables
        formulario = ttk.LabelFrame(
            self.inscripcion_frame,
            text="Datos de la inscripción",
            padding=10
        )
        formulario.pack(fill="x", pady=(0, 10))

        ttk.Label(
            formulario,
            text="Alumno:"
        ).grid(row=0, column=0, padx=5, pady=5, sticky="w")

        self.combo_alumno = ttk.Combobox(
            formulario,
            state="readonly",
            width=50
        )
        self.combo_alumno.grid(
            row=0,
            column=1,
            padx=5,
            pady=5,
            sticky="ew"
        )

        ttk.Label(
            formulario,
            text="Curso:"
        ).grid(row=1, column=0, padx=5, pady=5, sticky="w")

        self.combo_curso = ttk.Combobox(
            formulario,
            state="readonly",
            width=50
        )
        self.combo_curso.grid(
            row=1,
            column=1,
            padx=5,
            pady=5,
            sticky="ew"
        )

        formulario.columnconfigure(1, weight=1)

        # Botones
        botones = ttk.Frame(self.inscripcion_frame)
        botones.pack(fill="x", pady=(0, 10))

        self.agregar_botones(
            botones,
            [
                ("Inscribir", self.crear_inscripcion),
                ("Actualizar", self.actualizar_inscripcion),
                ("Eliminar", self.eliminar_inscripcion),
                ("Limpiar", self.limpiar_inscripcion),
                ("Recargar", self.cargar_inscripciones)
            ]
        )

        # Tabla de inscripciones
        tabla_frame = ttk.Frame(self.inscripcion_frame)
        tabla_frame.pack(fill="both", expand=True)

        columnas = {
            "id": "ID",
            "nombre": "Alumno",
            "matricula": "Matrícula",
            "nombreCurso": "Curso",
            "creditos": "Créditos"
        }

        self.inscripcion_tabla = ttk.Treeview(
            tabla_frame,
            columns=tuple(columnas.keys()),
            show="headings",
            height=12
        )

        for columna, titulo in columnas.items():
            self.inscripcion_tabla.heading(
                columna,
                text=titulo
            )

            ancho = 100 if columna in ("id", "creditos") else 180

            self.inscripcion_tabla.column(
                columna,
                width=ancho,
                anchor="center"
            )

        scroll_vertical = ttk.Scrollbar(
            tabla_frame,
            orient="vertical",
            command=self.inscripcion_tabla.yview
        )

        self.inscripcion_tabla.configure(
            yscrollcommand=scroll_vertical.set
        )

        self.inscripcion_tabla.pack(
            side="left",
            fill="both",
            expand=True
        )

        scroll_vertical.pack(side="right", fill="y")

        self.inscripcion_tabla.bind(
            "<<TreeviewSelect>>",
            self.seleccionar_inscripcion
        )

        self.cargar_inscripciones()

    # ---------------------------------------------------------
    # Combos de alumnos y cursos
    # ---------------------------------------------------------

    def cargar_alumnos_combo(self):
        alumnos = crud.leer_alumnos()

        self.alumnos_por_opcion = {}
        opciones = []

        for alumno_id, nombre, matricula in alumnos:
            etiqueta = f"{nombre} ({matricula}) - ID: {alumno_id}"

            self.alumnos_por_opcion[etiqueta] = alumno_id
            opciones.append(etiqueta)

        self.combo_alumno["values"] = opciones

        # Quitar la selección si ya no existe
        if self.combo_alumno.get() not in self.alumnos_por_opcion:
            self.combo_alumno.set("")

    def cargar_cursos_combo(self):
        cursos = crud.leer_cursos()

        self.cursos_por_opcion = {}
        opciones = []

        for curso_id, nombre, creditos in cursos:
            etiqueta = f"{nombre} - ID: {curso_id}"

            self.cursos_por_opcion[etiqueta] = curso_id
            opciones.append(etiqueta)

        self.combo_curso["values"] = opciones

        if self.combo_curso.get() not in self.cursos_por_opcion:
            self.combo_curso.set("")

    # ---------------------------------------------------------
    # CRUD de inscripciones
    # ---------------------------------------------------------

    def cargar_inscripciones(self):
        try:
            # Actualizar opciones por si hubo cambios en otras pestañas
            self.cargar_alumnos_combo()
            self.cargar_cursos_combo()

            self.limpiar_tabla(self.inscripcion_tabla)

            inscripciones = crud.leer_inscripciones()

            for inscripcion in inscripciones:
                self.inscripcion_tabla.insert(
                    "",
                    "end",
                    values=inscripcion
                )

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudieron cargar las inscripciones:\n{error}"
            )

    def crear_inscripcion(self):
        alumno_opcion = self.combo_alumno.get()
        curso_opcion = self.combo_curso.get()

        if not alumno_opcion or not curso_opcion:
            messagebox.showwarning(
                "Campos requeridos",
                "Selecciona un alumno y un curso."
            )
            return

        alumno_id = self.alumnos_por_opcion.get(alumno_opcion)
        curso_id = self.cursos_por_opcion.get(curso_opcion)

        if alumno_id is None or curso_id is None:
            messagebox.showwarning(
                "Selección inválida",
                "Vuelve a seleccionar el alumno y el curso."
            )
            return

        try:
            crud.crear_inscripcion(alumno_id, curso_id)

            messagebox.showinfo(
                "Éxito",
                "Inscripción creada correctamente."
            )

            self.limpiar_inscripcion()
            self.cargar_inscripciones()

        except sqlite3.IntegrityError:
            messagebox.showerror(
                "No se pudo inscribir",
                "Esta inscripción ya existe o los registros "
                "seleccionados ya no están disponibles."
            )

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudo crear la inscripción:\n{error}"
            )

    def seleccionar_inscripcion(self, evento=None):
        seleccion = self.inscripcion_tabla.selection()

        # Si no hay selección (p. ej. tras limpiar o recargar la tabla),
        # salir sin mostrar ningún mensaje
        if not seleccion:
            return

        valores = self.inscripcion_tabla.item(seleccion[0], "values")

        if not valores:
            return

        inscripcion_id = int(valores[0])

        try:
            inscripcion = crud.leer_inscripcion(inscripcion_id)

            if not inscripcion:
                return

            # Se espera: (id, alumno_id, curso_id)
            _, alumno_id, curso_id = inscripcion

            alumno_opcion = next(
                (
                    etiqueta
                    for etiqueta, identificador
                    in self.alumnos_por_opcion.items()
                    if identificador == alumno_id
                ),
                ""
            )

            curso_opcion = next(
                (
                    etiqueta
                    for etiqueta, identificador
                    in self.cursos_por_opcion.items()
                    if identificador == curso_id
                ),
                ""
            )

            self.combo_alumno.set(alumno_opcion)
            self.combo_curso.set(curso_opcion)

        except sqlite3.Error as error:
            messagebox.showerror(
            "Error",
            f"No se pudo seleccionar la inscripción:\n{error}"
        )

    def actualizar_inscripcion(self):
        inscripcion_id = self.obtener_id_seleccionado(
            self.inscripcion_tabla
        )

        if inscripcion_id is None:
            return

        alumno_opcion = self.combo_alumno.get()
        curso_opcion = self.combo_curso.get()

        if not alumno_opcion or not curso_opcion:
            messagebox.showwarning(
                "Campos requeridos",
                "Selecciona un alumno y un curso."
            )
            return

        alumno_id = self.alumnos_por_opcion.get(alumno_opcion)
        curso_id = self.cursos_por_opcion.get(curso_opcion)

        if alumno_id is None or curso_id is None:
            messagebox.showwarning(
                "Selección inválida",
                "Vuelve a seleccionar el alumno y el curso."
            )
            return

        try:
            actualizado = crud.actualizar_inscripcion(
                inscripcion_id,
                alumno_id,
                curso_id
            )

            if actualizado:
                messagebox.showinfo(
                    "Éxito",
                    "Inscripción actualizada correctamente."
                )
            else:
                messagebox.showwarning(
                    "Aviso",
                    "No se encontró la inscripción."
                )

            self.limpiar_inscripcion()
            self.cargar_inscripciones()

        except sqlite3.IntegrityError:
            messagebox.showerror(
                "No se pudo actualizar",
                "El alumno ya está inscrito en ese curso "
                "o los registros seleccionados no son válidos."
            )

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudo actualizar la inscripción:\n{error}"
            )

    def eliminar_inscripcion(self):
        inscripcion_id = self.obtener_id_seleccionado(
            self.inscripcion_tabla
        )

        if inscripcion_id is None:
            return

        confirmar = messagebox.askyesno(
            "Confirmar eliminación",
            "¿Deseas eliminar esta inscripción?"
        )

        if not confirmar:
            return

        try:
            eliminado = crud.eliminar_inscripcion(inscripcion_id)

            if eliminado:
                messagebox.showinfo(
                    "Éxito",
                    "Inscripción eliminada correctamente."
                )
            else:
                messagebox.showwarning(
                    "Aviso",
                    "No se encontró la inscripción."
                )

            self.limpiar_inscripcion()
            self.cargar_inscripciones()

        except sqlite3.Error as error:
            messagebox.showerror(
                "Error",
                f"No se pudo eliminar la inscripción:\n{error}"
            )

    def limpiar_inscripcion(self):
        self.combo_alumno.set("")
        self.combo_curso.set("")

        for elemento in self.inscripcion_tabla.selection():
            self.inscripcion_tabla.selection_remove(elemento)


# =============================================================
# EJECUCIÓN DEL PROGRAMA
# =============================================================

if __name__ == "__main__":
    ventana = tk.Tk()
    app = Aplicacion(ventana)
    ventana.mainloop()
```

#### crud.py
```python
import sqlite3
from db import conectar_db

def crear_alumno(nombre, matricula):
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            INSERT INTO Alumnos (nombre, matricula)
            VALUES (?, ?)
            """,
            (nombre, matricula)
        )
        return cursor.lastrowid


def leer_alumnos():
    with conectar_db() as conn:
        cursor = conn.execute(
            "SELECT id, nombre, matricula FROM Alumnos"
        )
        return cursor.fetchall()


def leer_alumno(alumno_id):
    with conectar_db() as conn:
        cursor = conn.execute(
            "SELECT id, nombre, matricula FROM Alumnos WHERE id = ?",
            (alumno_id,)
        )
        return cursor.fetchone()


def actualizar_alumno(alumno_id, nombre, matricula):
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            UPDATE Alumnos
            SET nombre = ?, matricula = ?
            WHERE id = ?
            """,
            (nombre, matricula, alumno_id)
        )
        return cursor.rowcount > 0


def eliminar_alumno(alumno_id):
    with conectar_db() as conn:
        cursor = conn.execute(
            "DELETE FROM Alumnos WHERE id = ?",
            (alumno_id,)
        )
        return cursor.rowcount > 0

def crear_curso(nombre_curso, creditos):
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            INSERT INTO Cursos (nombreCurso, creditos)
            VALUES (?, ?)
            """,
            (nombre_curso, creditos)
        )
        return cursor.lastrowid


def leer_cursos():
    with conectar_db() as conn:
        cursor = conn.execute(
            "SELECT id, nombreCurso, creditos FROM Cursos"
        )
        return cursor.fetchall()


def leer_curso(curso_id):
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            SELECT id, nombreCurso, creditos
            FROM Cursos
            WHERE id = ?
            """,
            (curso_id,)
        )
        return cursor.fetchone()


def actualizar_curso(curso_id, nombre_curso, creditos):
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            UPDATE Cursos
            SET nombreCurso = ?, creditos = ?
            WHERE id = ?
            """,
            (nombre_curso, creditos, curso_id)
        )
        return cursor.rowcount > 0


def eliminar_curso(curso_id):
    with conectar_db() as conn:
        cursor = conn.execute(
            "DELETE FROM Cursos WHERE id = ?",
            (curso_id,)
        )
        return cursor.rowcount > 0


def crear_inscripcion(alumno_id, curso_id):
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            INSERT INTO Inscripciones (alumno_id, curso_id)
            VALUES (?, ?)
            """,
            (alumno_id, curso_id)
        )
        return cursor.lastrowid


def leer_inscripciones():
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            SELECT
                i.id,
                a.nombre,
                a.matricula,
                c.nombreCurso,
                c.creditos
            FROM Inscripciones AS i
            JOIN Alumnos AS a ON i.alumno_id = a.id
            JOIN Cursos AS c ON i.curso_id = c.id
            ORDER BY i.id
            """
        )
        return cursor.fetchall()


def leer_inscripcion(inscripcion_id):
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            SELECT id, alumno_id, curso_id
            FROM Inscripciones
            WHERE id = ?
            """,
            (inscripcion_id,)
        )
        return cursor.fetchone()


def actualizar_inscripcion(inscripcion_id, alumno_id, curso_id):
    with conectar_db() as conn:
        cursor = conn.execute(
            """
            UPDATE Inscripciones
            SET alumno_id = ?, curso_id = ?
            WHERE id = ?
            """,
            (alumno_id, curso_id, inscripcion_id)
        )
        return cursor.rowcount > 0


def eliminar_inscripcion(inscripcion_id):
    with conectar_db() as conn:
        cursor = conn.execute(
            "DELETE FROM Inscripciones WHERE id = ?",
            (inscripcion_id,)
        )
        return cursor.rowcount > 0
```

#### models.py
```python
class Alumnos:
    def __init__(self, nombre, matricula):
        self.nombre = nombre
        self.matricula = matricula

class Cursos:
    def __init__(self, nombreCurso, creditos):
        self.nombreCurso = nombreCurso
        self.creditos = creditos

class Inscripciones:
    def __init__(self, alumno_id, curso_id):
        self.alumno_id = alumno_id
        self.curso_id = curso_id
```

#### db.py
```python
import sqlite3

def conectar_db():
    conn = sqlite3.connect("certificacion.db")
    conn.execute("PRAGMA foreign_keys = ON")
    return conn

def init_db():
    conn = conectar_db()

    try:
        cursor = conn.cursor()

        # Tabla Alumnos
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS Alumnos (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nombre TEXT NOT NULL,
                matricula TEXT NOT NULL UNIQUE
            )
        """)

        # Tabla Cursos
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS Cursos (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nombreCurso TEXT NOT NULL,
                creditos INTEGER NOT NULL CHECK (creditos > 0)
            )
        """)

        # Tabla Inscripciones
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS Inscripciones (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                alumno_id INTEGER NOT NULL,
                curso_id INTEGER NOT NULL,
                FOREIGN KEY (alumno_id) REFERENCES Alumnos(id),
                FOREIGN KEY (curso_id) REFERENCES Cursos(id),
                UNIQUE (alumno_id, curso_id)
            )
        """)

        conn.commit()

    except sqlite3.Error:
        conn.rollback()
        raise

    finally:
        conn.close()
```
### Procedimiento de instalación y ejecucion
1. Clonar o descargar el proyecto.
2. Crear entorno virtual:
   ```powershell
   # Windows
   python -m venv venv
   venv\Scripts\activate
   ```

   ```bash
   # Linux
   python -m venv venv
   source venv/bin/activate
   ```
3. Instalar dependencias::
   ```bash
   pip install requirements.txt
   ```
4. Ejecutar el sistema:
   ```bash
   python main.py
   ```

## 9. Generacion del ejecutable
1. Instalar PyInstaller:
   ```bash
   pip install pyinstaller
   ```
2. Crear el .exe:
   ```bash
   pyinstaller --oneline main.py
   ```
3. Copiar la base certificacion.db a la misma carpeta del  ejecutable (dist/)
4. Validar la ejecución en un equipo sin python instalado
