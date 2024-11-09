import sqlite3

class GestionAcademica:
    def __init__(self):
        # Conexión a SQLite (crea un archivo de base de datos local)
        self.conexion = sqlite3.connect('my_database.db')
        self.cursor = self.conexion.cursor()
        self._crear_tablas()

    def _crear_tablas(self):
        # Crear las tablas en SQLite
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS estudiantes (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                nombre TEXT,
                carnet TEXT UNIQUE
            )
        ''')
        
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS cursos (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                codigo TEXT UNIQUE,
                nombre TEXT
            )
        ''')
        
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS notas (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                estudiante_id INTEGER,
                curso_id INTEGER,
                primer_parcial REAL,
                segundo_parcial REAL,
                examen_final REAL,
                FOREIGN KEY (estudiante_id) REFERENCES estudiantes (id),
                FOREIGN KEY (curso_id) REFERENCES cursos (id)
            )
        ''')
        self.conexion.commit()

    def crear_estudiante(self, nombre, carnet):
        try:
            self.cursor.execute("INSERT INTO estudiantes (nombre, carnet) VALUES (?, ?)", (nombre, carnet))
            self.conexion.commit()
            print("Estudiante registrado con éxito.")
        except Exception as e:
            print(f"Error al registrar estudiante: {e}")

    def crear_curso(self, codigo, nombre):
        try:
            self.cursor.execute("INSERT INTO cursos (codigo, nombre) VALUES (?, ?)", (codigo, nombre))
            self.conexion.commit()
            print("Curso registrado con éxito.")
        except Exception as e:
            print(f"Error al registrar curso: {e}")

    def registrar_notas(self, carnet, codigo_curso, primer_parcial, segundo_parcial, examen_final):
        try:
            self.cursor.execute("SELECT id FROM estudiantes WHERE carnet = ?", (carnet,))
            estudiante_id = self.cursor.fetchone()
            self.cursor.execute("SELECT id FROM cursos WHERE codigo = ?", (codigo_curso,))
            curso_id = self.cursor.fetchone()

            if estudiante_id and curso_id:
                self.cursor.execute(
                    "INSERT INTO notas (estudiante_id, curso_id, primer_parcial, segundo_parcial, examen_final) "
                    "VALUES (?, ?, ?, ?, ?)",
                    (estudiante_id[0], curso_id[0], primer_parcial, segundo_parcial, examen_final)
                )
                self.conexion.commit()
                print("Notas registradas con éxito.")
            else:
                print("Estudiante o curso no encontrado.")
        except Exception as e:
            print(f"Error al registrar notas: {e}")

    def buscar_estudiante(self, nombre=None, carnet=None):
        try:
            query = "SELECT * FROM estudiantes WHERE nombre = ? OR carnet = ?"
            self.cursor.execute(query, (nombre, carnet))
            estudiantes = self.cursor.fetchall()
            for estudiante in estudiantes:
                print(estudiante)
        except Exception as e:
            print(f"Error en la búsqueda: {e}")

    def reporte_mejores_peores(self):
        try:
            query = """
            SELECT e.nombre, c.nombre AS curso, 
            (n.primer_parcial + n.segundo_parcial + n.examen_final) / 3 AS promedio
            FROM notas n
            JOIN estudiantes e ON e.id = n.estudiante_id
            JOIN cursos c ON c.id = n.curso_id
            ORDER BY promedio DESC
            LIMIT 5;
            """
            self.cursor.execute(query)
            reporte = self.cursor.fetchall()
            print("Mejores estudiantes:")
            for fila in reporte:
                print(fila)
        except Exception as e:
            print(f"Error al generar el reporte: {e}")

    def cerrar_conexion(self):
        self.cursor.close()
        self.conexion.close()

# Ejecución
if __name__ == "__main__":
    app = GestionAcademica()
    
    # Registro de datos
    app.crear_estudiante("Juan Perez", "2023001")
    app.crear_estudiante("Ana López", "2023002")
    app.crear_curso("MAT101", "Matemáticas")
    app.crear_curso("FIS101", "Física")

    # Registro de notas
    app.registrar_notas("2023001", "MAT101", 8.5, 7.0, 9.0)
    app.registrar_notas("2023002", "FIS101", 6.5, 8.0, 7.5)

    # Búsqueda
    app.buscar_estudiante(nombre="Juan Perez")

    # Generar reportes
    app.reporte_mejores_peores()

    # Cerrar conexión
    app.cerrar_conexion()

