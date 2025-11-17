#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_PERSONAS 50
#define MAX_MATERIAS 20
#define MAX_CALIFICACIONES 100

//ESTRUCTURAS
typedef struct
{
    int idPersona;
    char nombre[50];
    char dni[20];
    char usuario[20];
    char contrasenia[20];
    char tipo;
    int activo;
} Persona;

typedef struct
{
    int idMateria;
    char nombreMateria[30];
    char profesor[30];
    int anio;
} Materia;

typedef struct
{
    int idAlumno;
    int idMateria;
    float nota;
    char fecha[11];
} Calificacion;

// PROTOTIPS
int crearUsuario(char archivo[]);
int cargarUsuarios(Persona personas[], char archivo[]);
int cargarMaterias(Materia materias[], char archivo[]);
int cargarCalificaciones(Calificacion calificaciones[], char archivo[]);
void registrarCalificacion(char archivo[], int idAlumno, int idMateria, float nota);
void mostrarCalificacionesAlumno(char archivo[], int idAlumno, Materia materias[], int cantMaterias);
void promedioAlumno(char archivo[], int idAlumno);
void menuAlumno(char archivoCalificaciones[], int idAlumno, Materia materias[], int cantMaterias);
void menuProfesor(char archivoCalificaciones[], Persona personas[], int cantPersonas, Materia materias[], int cantMaterias);
void listarMaterias(Materia materias[], int cantMaterias);
void listarAlumnos(Persona personas[], int cantPersonas);

//MAIN
int main()
{
    Persona personas[MAX_PERSONAS];
    Materia materias[MAX_MATERIAS];
    Calificacion calificaciones[MAX_CALIFICACIONES];

    int cantUsuarios, cantMateriasTot, cantCalificaciones;
    char opcion;

    cantUsuarios = cargarUsuarios(personas, "usuarios.dat");
    cantMateriasTot = cargarMaterias(materias, "materias.dat");
    cantCalificaciones = cargarCalificaciones(calificaciones, "calificaciones.dat");

    do
    {
        printf("\n-.-.-.-.-CAMPUS VIRTUAL-.-.-.-.-\n");
        printf("\nUNO 1: Crear usuario");
        printf("\nDOS 2: Ingresar al sistema");
        printf("\nTRES 3: Salir");
        printf("\nOpcion: ");
        scanf(" %c", &opcion);

        switch(opcion)
        {
        case '1':
            crearUsuario("usuarios.dat");
            cantUsuarios = cargarUsuarios(personas, "usuarios.dat");
            break;

        case '2':
        {
            char usuarioIngresado[20], contraseniaIngresada[20];
            Persona personaActiva;
            int encontrado = 0;

            printf("\nINICIO DE SESION MARTA");
            printf("\nUsuario: ");
            getchar();
            scanf(" %s", usuarioIngresado);
            getchar();
            printf("\nContrasenia: ");
            scanf(" %s", contraseniaIngresada);

            for(int i=0; i<cantUsuarios; i++)
            {
                if(strcmp(personas[i].usuario, usuarioIngresado)==0 &&
                        strcmp(personas[i].contrasenia, contraseniaIngresada)==0 &&
                        personas[i].activo==1)
                {
                    personaActiva = personas[i];
                    encontrado = 1;
                    break;
                }
            }

            if(!encontrado)
            {
                printf("\nUsuario o contrasenia incorrectos.");
            }
            else
            {
                printf("Bienvenido/a: %s\n", personaActiva.nombre);
                if(personaActiva.tipo == 'A')
                    menuAlumno("calificaciones.dat", personaActiva.idPersona, materias, cantMateriasTot);
                else if(personaActiva.tipo == 'P')
                    menuProfesor("calificaciones.dat", personas, cantUsuarios, materias, cantMateriasTot);
                else
                    printf("\nERROR.");
            }
            break;
        }

        case '3':
            printf("\nByeeee..");
            break;

        default:
            printf("\nOpcioooooon invalida.");
        }

    }
    while(opcion!='3');

    return 0;
}

//FUNCIONES

int crearUsuario(char archivo[])
{
    FILE *f = fopen(archivo, "ab");
    if(!f)
    {
        printf("Error al abrir archivo de usuarios.\n");
        return 0;
    }

    Persona p;
    char tipo;

    printf("\nID de la persona: ");
    scanf("%d", &p.idPersona);
    getchar();

    printf("\nNombre y apellido: ");
    fgets(p.nombre, 50, stdin);
    p.nombre[strcspn(p.nombre, "\n")] = 0;

    printf("\nDNI: ");
    scanf("%c", p.dni);

    printf("\nUsuario: ");
    fgets(p.usuario, 30, stdin);
    p.usuario[strcspn(p.usuario, "\n")] = 0;

    printf("\nContrasenia: ");
    fgets(p.contrasenia, 20, stdin);
    p.contrasenia[strcspn(p.contrasenia, "\n")] = 0;

    printf("\nTipo (A=Alumno / P=Profesor): ");
    scanf(" %c", &tipo);

    p.tipo = tipo;
    p.activo = 1;

    fwrite(&p, sizeof(Persona), 1, f);
    fclose(f);

    printf("\nUsuario creado correctamente.");
    return 1;
}

int cargarUsuarios(Persona personas[], char archivo[])
{
    FILE *f = fopen(archivo, "rb");
    if(!f)
    {
        return 0;
    }
    int cant = 0;
    while(fread(&personas[cant], sizeof(Persona), 1, f) == 1 && cant < MAX_PERSONAS)
        cant++;
    fclose(f);
    return cant;
}

int cargarMaterias(Materia materias[], char archivo[])
{
    FILE *f = fopen(archivo, "rb");
    if(!f) return 0;

    int cant = 0;
    while(fread(&materias[cant], sizeof(Materia), 1, f) == 1 && cant < MAX_MATERIAS)
        cant++;

    fclose(f);
    return cant;
}

int cargarCalificaciones(Calificacion calificaciones[], char archivo[])
{
    FILE *f = fopen(archivo, "rb");
    if(!f) return 0;

    int cant = 0;
    while(fread(&calificaciones[cant], sizeof(Calificacion), 1, f) == 1 && cant < MAX_CALIFICACIONES)
        cant++;

    fclose(f);
    return cant;
}

void registrarCalificacion(char archivo[], int idAlumno, int idMateria, float nota)
{
    FILE *f = fopen(archivo, "ab");
    if(!f)
    {
        printf("\nError al abrir archivo de calificaciones.");
        return;
    }

    Calificacion c;
    c.idAlumno = idAlumno;
    c.idMateria = idMateria;
    c.nota = nota;

    fwrite(&c, sizeof(Calificacion), 1, f);
    fclose(f);

    printf("\nCalificacion registrada.");
}

void listarMaterias(Materia materias[], int cantMaterias)
{
    printf("\nID,\tMateria,\tProfesor,\tAño.\n");
    printf(".--.--.--.--.--.--.--.--.--.--.--.\n");
    for(int i = 0; i < cantMaterias; i++)
    {
        printf("%d,\t%-15s,\t%-10s,\t%d.\n",
               materias[i].idMateria,
               materias[i].nombreMateria,
               materias[i].profesor,
               materias[i].anio);
    }
}

void listarAlumnos(Persona personas[], int cantPersonas)
{
    printf("\nID,\tNombre,\t\tDNI,\n");
    printf("\n.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.");
    for(int i=0; i<cantPersonas; i++)
    {
        if(personas[i].tipo=='A' && personas[i].activo==1)
        {
            printf("%d,\t%-15s,\t%s\n",
                   personas[i].idPersona,
                   personas[i].nombre,
                   personas[i].dni);
        }
    }
}

void mostrarCalificacionesAlumno(char archivo[], int idAlumno, Materia materias[], int cantMaterias)
{
    FILE *f = fopen(archivo, "rb");
    if(!f)
    {
        printf("Error al abrir archivo.\n");
        return;
    }

    Calificacion c;
    int tiene = 0;

    printf("\nMateria\t\tNota\tEstado\n");
    printf("-------------------------------\n");

    while(fread(&c, sizeof(Calificacion), 1, f))
    {
        if(c.idAlumno == idAlumno)
        {

            int i;
            for(i=0; i<cantMaterias; i++)
            {
                if(materias[i].idMateria == c.idMateria)
                    break;
            }

            if(i == cantMaterias)
            {
                printf("Materia desconocida (ID %d)\n", c.idMateria);
                continue;
            }

            const char *estado =
                (c.nota >= 8) ? "Promocionada" :
                (c.nota >= 6) ? "Aprobada" :
                "Recursa";

            printf("%-15s\t%.2f\t%s\n",
                   materias[i].nombreMateria,
                   c.nota,
                   estado);

            tiene = 1;
        }
    }

    if(!tiene)
        printf("No hay calificaciones.\n");

    fclose(f);
}

void promedioAlumno(char archivo[], int idAlumno)
{
    FILE *f = fopen(archivo, "rb");
    if(!f) return;

    Calificacion c;
    int cont = 0;
    float suma = 0;

    while(fread(&c,sizeof(Calificacion),1,f))
    {
        if(c.idAlumno == idAlumno)
        {
            suma += c.nota;
            cont++;
        }
    }

    fclose(f);

    if(cont > 0) {
        printf("Promedio general: %.2f\n", suma/cont);
    } else {
        printf("No hay calificaciones para calcular promedio.\n");
    }
}

// MENU ALUMNO
void menuAlumno(char archivoCalificaciones[], int idAlumno, Materia materias[], int cantMaterias)
{
    char op;
    do
    {
        printf("\n=== Menu Alumno ===\n");
        printf("1. Ver mis calificaciones\n");
        printf("2. Ver promedio\n");
        printf("0. Salir\n");
        printf("Opcion: ");
        scanf(" %c", &op);

        switch(op)
        {
        case '1':
            mostrarCalificacionesAlumno(archivoCalificaciones, idAlumno, materias, cantMaterias);
            break;

        case '2':
            promedioAlumno(archivoCalificaciones, idAlumno);
            break;

        case '0':
            printf("Saliendo...\n");
            break;

        default:
            printf("Opcion invalida.\n");
        }

    }
    while(op!='0');
}

// MENU PROFESOR
void menuProfesor(char archivoCalificaciones[], Persona personas[], int cantPersonas, Materia materias[], int cantMaterias)
{
    char op;

    do
    {
        printf("\n=== Menu Profesor ===\n");
        printf("1. Registrar calificacion\n");
        printf("2. Listar materias\n");
        printf("3. Listar alumnos\n");
        printf("0. Salir\n");
        printf("Opcion: ");
        scanf(" %c", &op);

        switch(op)
        {
        case '1':
        {
            int idAlumno, idMateria;
            float nota;

            listarAlumnos(personas, cantPersonas);
            printf("ID alumno: ");
            scanf("%d", &idAlumno);

            listarMaterias(materias, cantMaterias);
            printf("ID materia: ");
            scanf("%d", &idMateria);

            printf("Nota (0-10): ");
            scanf("%f", &nota);

            registrarCalificacion(archivoCalificaciones, idAlumno, idMateria, nota);
            break;
        }

        case '2':
            listarMaterias(materias, cantMaterias);
            break;

        case '3':
            listarAlumnos(personas, cantPersonas);
            break;

        case '0':
            printf("Saliendo...\n");
            break;

        default:
            printf("Opcion invalida.\n");
        }
    }
    while(op!='0');
}

//FIIIIIIIIIIIIIIIN, THE END, A DOPOOOOOO
