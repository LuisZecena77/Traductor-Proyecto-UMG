#include <iostream>
#include <fstream>
#include <string>
#include <windows.h>
using namespace std;

struct nodo{
    string esp, ita, fra, ale, eng;
    nodo *izq, *der;
};

typedef nodo* Arbol;

// - Crear nodo -
Arbol crearNodo(string esp, string ita, string fra, string ale, string eng){
    Arbol nuevo = new nodo();
    nuevo->esp = esp;
    nuevo->ita = ita;
    nuevo->fra = fra;
    nuevo->ale = ale;
    nuevo->eng = eng;
    nuevo->izq = nuevo->der = NULL;
    return nuevo;
}

// - Buscar (solo por español)-
Arbol buscar(Arbol arbol, string esp){
    if(arbol == NULL) return NULL;
    if(esp == arbol->esp) return arbol;
    if(esp < arbol->esp)
        return buscar(arbol->izq, esp);
    else
        return buscar(arbol->der, esp);
}

// - Insertar -
void insertar(Arbol &arbol, string esp, string ita, string fra, string ale, string eng){
    if(arbol == NULL){
        arbol = crearNodo(esp, ita, fra, ale, eng);
    }
    else if(esp < arbol->esp){
        insertar(arbol->izq, esp, ita, fra, ale, eng);
    }
    else if(esp > arbol->esp){
        insertar(arbol->der, esp, ita, fra, ale, eng);
    }
    else{
        cout << "La palabra ya existe en el diccionario.\n";
    }
}

//- Mostrar en orden - 
void enOrden(Arbol arbol){
    if(arbol != NULL){
        enOrden(arbol->izq);
        cout << "\nEspanol:  " << arbol->esp << endl;
        cout << " Italiano: " << arbol->ita << endl;
        cout << " Frances:  " << arbol->fra << endl;
        cout << " Aleman:   " << arbol->ale << endl;
        cout << " Ingles:   " << arbol->eng << endl;
        cout << "--------------------------\n";
        enOrden(arbol->der);
    }
}

// - para eliminacion caso dos hijos -
Arbol minimo(Arbol arbol){
    while(arbol->izq != NULL)
        arbol = arbol->izq;
    return arbol;
}

Arbol eliminar(Arbol arbol, string esp){
    if(arbol == NULL){
        cout << "La palabra no se encontro.\n";
        return arbol;
    }

    if(esp < arbol->esp)
        arbol->izq = eliminar(arbol->izq, esp);
    else if(esp > arbol->esp)
        arbol->der = eliminar(arbol->der, esp);
    else{
        // Caso 1 y 2: sin hijo izquierdo o sin hijo derecho
        if(arbol->izq == NULL){
            Arbol temp = arbol->der;
            delete arbol;
            return temp;
        }
        else if(arbol->der == NULL){
            Arbol temp = arbol->izq;
            delete arbol;
            return temp;
        }
        // Caso 3: dos hijos - reemplazar con sucesor inorden
        Arbol temp = minimo(arbol->der);
        arbol->esp = temp->esp;
        arbol->ita = temp->ita;
        arbol->fra = temp->fra;
        arbol->ale = temp->ale;
        arbol->eng = temp->eng;
        arbol->der = eliminar(arbol->der, temp->esp);
    }
    return arbol;
}

// - Guardar en orden -
void guardarRecursivo(Arbol arbol, ofstream &fs){
    if(arbol != NULL){
        guardarRecursivo(arbol->izq, fs);
        fs << arbol->esp << ","
           << arbol->ita << ","
           << arbol->fra << ","
           << arbol->ale << ","
           << arbol->eng << "\n";
        guardarRecursivo(arbol->der, fs);
    }
}

// - Guardar archivo -
void guardarArchivo(Arbol arbol){
    ofstream fs("diccionario.txt");
    guardarRecursivo(arbol, fs);
    fs.close();
}

// Cargar Archivo CVS -
void cargarArchivo(Arbol &arbol){
    ifstream fe("diccionario.txt");

    string esp, ita, fra, ale, eng;

    if(!fe){
        cout << "Error al abrir archivo\n";
        return;
    }

    while(fe >> esp >> ita >> fra >> ale >> eng){
        insertar(arbol, esp, ita, fra, ale, eng);
    }

    fe.close();
    cout << "Datos cargados\n";
}

// - Audio con PowerShell TTS -
void reproducirAudio(string palabra){
    string comando = "powershell -c \"Add-Type -AssemblyName System.Speech; "
                     "$voz = New-Object System.Speech.Synthesis.SpeechSynthesizer; "
                     "$voz.Speak('" + palabra + "');\"";
    system(comando.c_str());
}

// - Traductor -
void traducir(Arbol arbol){
    int origen, destino;
    string palabra;

    cout << "\nIdioma origen:\n";
    cout << "1 Español\n2 Italiano\n3 Frances\n4 Aleman\n5 Ingles\n";
    cin >> origen;

    cout << "Idioma destino:\n";
    cout << "1 Español\n2 Italiano\n3 Frances\n4 Aleman\n5 Ingles\n";
    cin >> destino;

    cout << "Ingrese palabra: ";
    cin >> palabra;

    Arbol nodo = buscar(arbol, palabra);

    if(nodo == NULL){
        cout << "Palabra no encontrada\n";
        return;
    }

    string resultado;

    // Obtener traducción
    switch(destino){
        case 1: resultado = nodo->esp; break;
        case 2: resultado = nodo->ita; break;
        case 3: resultado = nodo->fra; break;
        case 4: resultado = nodo->ale; break;
        case 5: resultado = nodo->eng; break;
    }

    cout << "Traduccion: " << resultado << endl;

    // Audio automático
    reproducirAudio(resultado);
}

// - Main -
int main(){
    SetConsoleOutputCP(65001);
    SetConsoleCP(65001);

    Arbol arbol = NULL;
    int op;
    string esp, ita, fra, ale, eng;

    do{
        system("cls");
        cout << "---------- PROYECTO FINAL FASE 1 DICCIONARIO MULTILENGUAJE ----------\n";
        cout << "1. Cargar archivo\n";
        cout << "2. Insertar palabra\n";
        cout << "3. Mostrar diccionario\n";
        cout << "4. Traducir palabra\n";
        cout << "5. Eliminar palabra\n";
        cout << "6. Audio\n";
        cout << "0. Salir\n";
        cout << "Opcion: ";
        cin >> op;

        switch(op){
        case 1:
            system("cls");
            cargarArchivo(arbol);
            system("pause");
            break;

        case 2:
            system("cls");
            cout << "=== Insertar Palabra ===\n";
            cout << "Espanol:  "; cin >> esp;
            cout << "Italiano: "; cin >> ita;
            cout << "Frances:  "; cin >> fra;
            cout << "Aleman:   "; cin >> ale;
            cout << "Ingles:   "; cin >> eng;
            insertar(arbol, esp, ita, fra, ale, eng);
            guardarArchivo(arbol);
            system("pause");
            break;

        case 3:
            system("cls");
            cout << "=== Diccionario Completo ===\n";
            enOrden(arbol);
            system("pause");
            break;

        case 4:
            system("cls");
            cout << "=== Traducir ===\n";
            traducir(arbol);
            system("pause");
            break;

        case 5:
            system("cls");
            cout << "=== Eliminar Palabra ===\n";
            cout << "Palabra en espanol: "; cin >> esp;
            arbol = eliminar(arbol, esp);
            guardarArchivo(arbol);
            system("pause");
            break;

        case 6:
            system("cls");
            cout << "=== Audio ===\n";
            cout << "Palabra: "; cin >> esp;
            reproducirAudio(esp);
            system("pause");
            break;

        case 0:
            cout << "Saliendo...\n";
            break;

        default:
            cout << "Opcion no valida.\n";
            system("pause");
        }

    }while(op != 0);

    return 0;
}
