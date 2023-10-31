import tkinter as tk
from tkinter import ttk
import mysql.connector
import random
from tkinter import messagebox

pregunta_actual=0
puntuacion = 0
tiempo=0
root=None
ventana=None
botones_respuesta=[]

conexion = mysql.connector.connect(host="localhost", user="root", password="", database="trivia_isaui")
cursor = conexion.cursor()


def crear_ventana():

    def iniciar_juego():
        global tiempo_transcurrido
        tiempo_transcurrido=0
        jugadores_data=[]
        nombre = nombre_entry.get()
        Instagram = instagram_entry.get()
        
    
        if nombre and Instagram:
            jugadores_data.append((nombre, Instagram))
            print(jugadores_data)
            ventana.destroy()
            abrir_juego()

        else:
            messagebox.showerror("Error", "Los campos son obligatorios y se deben completar.")


       
    def salir():
        ventana.destroy()
        root.destroy()

    ventana = tk.Tk()
    ventana.title("Jugadores")
    ventana.attributes('-fullscreen', True)
    ventana.configure(bg="#630551")
 

    formulario_frame = tk.Frame(ventana, bd=2, relief=tk.SOLID, bg="#F3C7AA")
    formulario_frame.grid(padx=10, pady=10)


    titulo_label = tk.Label(formulario_frame, text="Jugador", bg="#F3C7AA", font=("Calibri", 20, 'bold'))
    titulo_label.grid(row=0, column=0, columnspan=2, pady=10)

    nombre_label = tk.Label(formulario_frame, text="Nombre:", bg="#F3C7AA", font=("Calibri", 15))
    nombre_label.grid(row=1, column=0)
    nombre_entry = tk.Entry(formulario_frame)
    nombre_entry.grid(row=1, column=1, padx=5, pady=5, ipadx=5, ipady=5, sticky="ew")
    

    instagram_label = tk.Label(formulario_frame, text="Instagram", bg="#F3C7AA", font=("Calibri", 15))
    instagram_label.grid(row=3, column=0)
    instagram_entry = tk.Entry(formulario_frame)
    instagram_entry.grid(row=3, column=1, padx=5, pady=5, ipadx=5, ipady=5, sticky="ew")

    jugar_button = tk.Button(formulario_frame, text="JUGAR", bg="#9E5B00", font=("Calibri",8, 'bold'), command=lambda:iniciar_juego())
    jugar_button.grid(row=6, columnspan=2, pady=1, sticky="ew")

    jugar_button = tk.Button(formulario_frame, text="SALIR", bg="#9E5B00", font=("Calibri",8, 'bold'), command=lambda:salir())
    jugar_button.grid(row=7, columnspan=3, pady=5, sticky="ew")

    titulo2_label = tk.Label(ventana, text="Jugadores", font=("Calibri", 35, 'bold'), bg="#630551", fg="white")
    titulo2_label.grid(row=4, column=0, columnspan=5,  padx=250, pady=15)
    #añadir=Button(ventana, text="Añadir", bg="#9E3700", fg="white"). grid(row=4, column=1, padx=20, pady=20)

    tree = ttk.Treeview(ventana, columns=("idjugador", "Nombre", "Instagram", "Puntaje", "Tiempo"))
    tree.heading("#2", text="Nombre")
    tree.heading("#3", text="Instagram")
    tree.heading("#4", text="Puntaje")
    tree.heading("#5", text="Tiempo")
    tree.heading("#1", text="idjugador")
    tree.column("#1", width=0, stretch=tk.NO)
    tree.column("#0", width=0, stretch=tk.NO)  
    tree.column("#2", width=230)
    tree.config(height=23)
    tree.grid(row=5, column=0, padx=250, pady=0)
crear_ventana()



root = tk.Tk()
root.title("Juego de Preguntas y Respuestas")
root.attributes("-fullscreen", True)
root.configure(bg="#630551")

label_pregunta = tk.Label(root, text="", font=("Helvetica", 20, 'bold'), bg="#630551", fg="white")
label_pregunta.pack(pady=90)

frame_respuestas = tk.Frame(root, bg="#630551")
frame_respuestas.pack()
botones_respuesta = []



def abrir_juego():
        global root
        
        def obtener_preguntas():
            cursor.execute("SELECT PREGUNTA, RESPUESTA_CORRECTA, RESPUESTA_1, RESPUESTA_2, RESPUESTA_3 FROM Preguntas")
            preguntas_respuestas = cursor.fetchall()
            random.shuffle(preguntas_respuestas) 
            return preguntas_respuestas
        
        preguntas_respuestas = obtener_preguntas()
        
        def mostrar_pregunta():
            global pregunta_actual
            if pregunta_actual < len(preguntas_respuestas):
                pregunta, respuesta_correcta, respuesta_incorrecta_1, respuesta_incorrecta_2, respuesta_incorrecta_3 = preguntas_respuestas[pregunta_actual]
                label_pregunta.config(text=pregunta)
                opciones = [respuesta_correcta, respuesta_incorrecta_1, respuesta_incorrecta_2, respuesta_incorrecta_3]
                random.shuffle(opciones) 
                for i, opcion in enumerate(opciones):
                    botones_respuesta[i].config(text=opcion)
            else:
                label_pregunta.config(text="Fin del juego. Puntuación: " + str(puntuacion))
                for boton in botones_respuesta:
                    boton.config(state=tk.DISABLED)
                    
                    

        def verificar_respuesta(opcion):
            global pregunta_actual, puntuacion
            pregunta, respuesta_correcta, _, _, _ = preguntas_respuestas[pregunta_actual]
            if opcion == respuesta_correcta:
                puntuacion += 100
                mensaje = "¡Correcto!"
            else:
                mensaje = f"Incorrecto. La respuesta correcta es: {respuesta_correcta}"

            pregunta_actual += 1
            mostrar_pregunta()
            if pregunta_actual < len(preguntas_respuestas):
                messagebox.showinfo("Resultado", mensaje) 
            else:
                messagebox.showinfo("Fin del juego", "¡Juego terminado! Tu puntuación: " + str(puntuacion))   


        for i in range(4):
            frame_grupo = tk.Frame(frame_respuestas, bg="#630551")
            frame_grupo.pack(pady=30)
            boton = tk.Button(frame_grupo, text="", font=("Helvetica", 17), command=lambda i=i: verificar_respuesta(botones_respuesta[i]['text']))
            boton.pack(pady=10)

            botones_respuesta.append(boton)
        mostrar_pregunta()
        button_salir=tk.Button(root, text="salir", command=lambda:root.destroy())
        button_salir.pack(pady=12)

            
root.mainloop()
abrir_juego()

tk.mainloop()





'''create table Jugadores(
ID_JUGADOR int auto_increment primary key,
NOMBRE VARCHAR(255) NOT NULL,
INSTAGRAM VARCHAR(255),
PUNTAJE INT NOT NULL,
TIEMPO DOUBLE NOT NULL
);

create table Preguntas(
ID_PREGUNTA INT auto_increment primary key,
PREGUNTA VARCHAR (255) NOT NULL,
RESPUESTA_CORRECTA VARCHAR (255) NOT NULL,
RESPUESTA_1 VARCHAR(255) NOT NULL,
RESPUESTA_2 VARCHAR(255) NOT NULL,
RESPUESTA_3 VARCHAR(255) NOT NULL
);

INSERT INTO pregunta(NOMBRE, RESPUESTA_CORRECTA, RESPUESTA_1, RESPUESTA_2, RESPUESTA_3) VALUES 
("¿En qué año se fundó el instituto?", "1983", "1997", "1973", "1980"), 
("¿Cuál fue el nombre del establecimiento educativo cuando se fundó?", "Complejo Facultativo de Enseñanza Superior San Francisco de Asís", "Instituto Educativo Superior Cura Brochero", "Instituto Superior Arturo Umberto Illia", "Escuela Mayor Facultativa Raúl Alfonsin"),
("¿En qué año se convierte en colegio nacional?", "1985", "1988", "1997", "2005"), 
("¿Cuántos directores tuvo el Instituto hasta la fecha?", "4", "2", "6", "15"), 
("¿Cuántas carreras se dictan actualmente en el ISAUI?", "6", "7", "3", "4"), 
("¿Cuál fue el primer número de teléfono del Instituto?", "41164", "62374", "41279", "53345"), 
("¿Qué es el workshop?", "Una técnica de venta", "Un lugar para comprar", "Una carrera", "Una marca importante"), 
("¿Qué carrera fue la pionera con los viajes en las materias de prácticas?", "Técnico y Guía Superior en Turismo", "Tecnicatura en Desarrollo de Software", "Diseño de Espacios", "Trekking"), 
("¿Cómo era el edificio educativo en sus comienzos?", "Una casa con 3 ambientes", "Un contenedor ", "Una casa de un solo 1 ambiente", "Una biblioteca"), 
("¿Qué tipo de instituto es el ISAUI?", "Instituto Público", "Instituto Privado", "Instituto Semi-privado", "Instituto Semi-público"), 
("¿A partir de qué año la cooperadora empezó a construir las aulas?", "1986", "1989", "1990", "2000"), 
("¿De dónde provenía el dinero?", "Del bono contribución de los padres", "Del bono contribución de los profesores", "Por un sorteo", "Del gobierno provincial"), 
("¿Quién fue el presidente de la cooperadora en el inicio de la institución?", "Sr. Gatica", "Sra. Rosa ", "Sr. Pablo", "Juan"), 
("¿Quién es la presidenta de la cooperadora actualmente?", "Daniela Maschio", "Elizabeth Afazani", "Agustina Aperlo", "Gabriela Roldan"), 
("¿Cómo se logró la nacionalización?", "Mediante la gestión del diputado Anselmo Pelaez", "Mediante la ayuda del gobernador Schiaretti", "Mediante el acuerdo con la UEPC", "Mediante la reunión de las escuelas de Carlos Paz");'''
