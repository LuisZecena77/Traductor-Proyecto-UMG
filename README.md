#include <iostream>
#include <fstream>
#include <string>
#include <windows.h>
#include <sapi.h>

using namespace std;

// ===== NODO =====
struct nodo{
    string esp, ita, fra, ale, eng;
    nodo *izq, *der;
};

typedef nodo* Arbol;

// ===== CREAR =====
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

// ===== BUSCAR =====
Arbol buscar(Arbol arbol, string palabra){
    if(arbol == NULL) return NULL;

    if(palabra == arbol->esp || palabra == arbol->ita ||
       palabra == arbol->fra || palabra == arbol->ale ||
       palabra == arbol->eng){
        return arbol;
    }

    if(palabra < arbol->esp)
        return buscar(arbol->izq, palabra);
    else
        return buscar(arbol->der, palabra);
}

// ===== INSERTAR =====
void insertar(Arbol &arbol, string esp, string ita, string fra, string ale, string eng){
    if(buscar(arbol, esp)){
        cout << "La palabra ya existe, mostrando traducciones:\n";
        return;
    }

    if(arbol == NULL){
        arbol = crearNodo(esp, ita, fra, ale, eng);
    }
    else if(esp < arbol->esp){
        insertar(arbol->izq, esp, ita, fra, ale, eng);
    }
    else{
        insertar(arbol->der, esp, ita, fra, ale, eng);
    }
}

// ===== MOSTRAR =====
void enOrden(Arbol arbol){
    if(arbol != NULL){
        enOrden(arbol->izq);

        cout << "\nEspañol: " << arbol->esp << endl;
        cout << " Italiano: " << arbol->ita << endl;
        cout << " Frances: " << arbol->fra << endl;
        cout << " Aleman: " << arbol->ale << endl;
        cout << " Ingles: " << arbol->eng << endl;
        cout << "--------------------------\n";

        enOrden(arbol->der);
    }
}

// ===== MINIMO =====
Arbol minimo(Arbol arbol){
    while(arbol->izq != NULL)
        arbol = arbol->izq;
    return arbol;
}

// ===== ELIMINAR =====
Arbol eliminar(Arbol arbol, string esp){
    if(arbol == NULL) return arbol;

    if(esp < arbol->esp)
        arbol->izq = eliminar(arbol->izq, esp);
    else if(esp > arbol->esp)
        arbol->der = eliminar(arbol->der, esp);
    else{
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

// ===== CARGAR ARCHIVO =====
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

// ===== AUDIO =====
void reproducirAudio(string palabra){
    string comando = "powershell -c \"Add-Type -AssemblyName System.Speech; "
                     "$voz = New-Object System.Speech.Synthesis.SpeechSynthesizer; "
                     "$voz.Speak('" + palabra + "');\"";
    system(comando.c_str());
}

// ===== TRADUCIR =====
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

// ===== MAIN =====
int main(){
	
	SetConsoleOutputCP(65001);
	SetConsoleCP(65001);

    Arbol arbol = NULL;
    int op;
    string esp, ita, fra, ale, eng;

    do{
        system("cls");
        cout << "==== DICCIONARIO MULTILENGUAJE ====\n";
        cout << "1. Cargar archivo\n";
        cout << "2. Insertar palabra\n";
        cout << "3. Mostrar diccionario\n";
        cout << "4. Traducir\n";
        cout << "5. Eliminar palabra\n";
        cout << "6. Audio manual\n";
        cout << "0. Salir\n";
        cin >> op;

        switch(op){
        case 1:
            cargarArchivo(arbol);
            system("pause");
            break;

        case 2:
            cout << "Español: "; cin >> esp;
            cout << "Italiano: "; cin >> ita;
            cout << "Frances: "; cin >> fra;
            cout << "Aleman: "; cin >> ale;
            cout << "Ingles: "; cin >> eng;
            insertar(arbol, esp, ita, fra, ale, eng);
            break;

        case 3:
            enOrden(arbol);
            system("pause");
            break;

        case 4:
            traducir(arbol);
            system("pause");
            break;

        case 5:
            cout << "Eliminar palabra (español): ";
            cin >> esp;
            arbol = eliminar(arbol, esp);
            break;

        case 6:
            cout << "Palabra para audio: ";
            cin >> esp;
            reproducirAudio(esp);
            break;
        }

    }while(op != 0);

    return 0;
}
