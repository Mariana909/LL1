```markdown
# Analizador Sintáctico Descendente Recursivo (ASDR) — LL(1)

Implementación de analizadores sintácticos descendentes recursivos para tres gramáticas,
con generación de árboles de sintaxis por cada cadena analizada.

---

## Requisitos

- Python 3
- matplotlib

```bash
sudo apt install python3-matplotlib
```

---

## Estructura del proyecto

```
LL1/
├── parser_gramatica1.py      # ASDR para Gramática 1
├── parser_gramatica2.py      # ASDR para Gramática 2
├── parser_gramatica3.py      # ASDR para Gramática 3
├── entrada_g1.txt            # Cadenas de prueba — Gramática 1
├── entrada_g2.txt            # Cadenas de prueba — Gramática 2
├── entrada_g3.txt            # Cadenas de prueba — Gramática 3
├── Gramatica1.txt            # Gramática 1 para psp.py
├── Gramatica2.txt            # Gramática 2 para psp.py
├── Gramatica3.txt            # Gramática 3 para psp.py
└── README.md
```

---

## Ejecución

Cada parser recibe como argumento un archivo de texto con las cadenas a analizar, una por línea.
Por cada cadena imprime en consola si fue **aceptada** o **rechazada**, y genera una imagen `.png`
con el árbol de sintaxis correspondiente.

```bash
python3 parser_gramatica1.py entrada_g1.txt
python3 parser_gramatica2.py entrada_g2.txt
python3 parser_gramatica3.py entrada_g3.txt
```

---

## Gramáticas

### Gramática 1

```
S → A B C | D E
A → dos B tres | ε
B → B'
B' → cuatro C cinco B' | ε
C → seis A B | ε
D → uno A E | B
E → tres
```

La gramática original contiene recursividad directa por izquierda en `S → S uno`,
que fue eliminada introduciendo el no terminal auxiliar `S'`. La gramática resultante **es LL(1)**.

---

### Gramática 2

```
S  → B uno | dos C | ε
A  → dos C tres B C A' | uno tres B C A' | tres B C A' | cuatro A' | A'
A' → cinco C seis uno tres B C A' | ε
B  → A cinco C seis | ε
C  → siete B | ε
```

La gramática original contiene recursividad indirecta por izquierda a través del ciclo
`S ⇒ B... ⇒ A... ⇒ S...`. Para eliminarla se sustituyeron las producciones de `S` dentro
de `A`, luego las de `B`, hasta hacer visible la recursividad directa en `A`, que fue
eliminada introduciendo el no terminal auxiliar `A'`. La gramática resultante **es LL(1)**.

---

### Gramática 3

```
S  → A B C S'
S' → uno S' | ε
A  → dos B C | ε
B  → C tres | ε
C  → cuatro B | ε
```

La gramática original contiene recursividad directa por izquierda en `S → S uno`,
que fue eliminada introduciendo `S'`. Sin embargo, la gramática resultante **no es LL(1)**:
los conjuntos de predicción de `B` y `C` presentan conflictos causados por la recursión
mutua entre ambos no terminales, lo que hace que sus conjuntos SIGUIENTES se solapen
inevitablemente con los PRIMEROS de sus alternativas no vacías. El parser resuelve
estos conflictos mediante **backtracking**: cuando una alternativa falla, retrocede la
posición y prueba la siguiente.

---

## Repositorios relacionados

- **Código base del parser y generación de árboles:**
  [github.com/Mariana909/ARBOL_SINTACTICO](https://github.com/Mariana909/ARBOL_SINTACTICO.git)

- **Cálculo de PRIMEROS, SIGUIENTES y PREDICCIÓN (`psp.py`):**
  [github.com/Mariana909/PRIMEROS_SIGUIENTES_PREDICCION](https://github.com/Mariana909/PRIMEROS_SIGUIENTES_PREDICCION.git)
```
