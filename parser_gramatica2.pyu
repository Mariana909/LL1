import re
import sys
import time
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches


TOKEN_SPEC = [
    ("uno",   r"\buno\b"),
    ("dos",   r"\bdos\b"),
    ("tres",  r"\btres\b"),
    ("cuatro",r"\bcuatro\b"),
    ("cinco", r"\bcinco\b"),
    ("seis",  r"\bseis\b"),
    ("siete", r"\bsiete\b"),
    ("WS",    r"[ \t]+"),
]

TOKEN_RE = re.compile("|".join(f"(?P<{name}>{pat})" for name, pat in TOKEN_SPEC))


def tokenizar(texto):
    tokens = []
    for m in TOKEN_RE.finditer(texto):
        tipo = m.lastgroup
        valor = m.group()
        if tipo != "WS":
            tokens.append((tipo, valor))
    return tokens


class Nodo:
    def __init__(self, etiqueta):
        self.etiqueta = etiqueta
        self.hijos = []

    def agregar(self, hijo):
        self.hijos.append(hijo)
        return hijo


class Parser:
    def __init__(self, tokens):
        self.tokens = tokens
        self.pos = 0

    def actual(self):
        if self.pos < len(self.tokens):
            return self.tokens[self.pos]
        return (None, None)

    def consumir(self, tipo):
        tok = self.actual()
        if tok[0] == tipo:
            self.pos += 1
            return tok
        return None

    # S → B uno | dos C | ε
    def parse_S(self, nodo_padre):
        nodo = nodo_padre.agregar(Nodo("S"))
        pos_respaldo = self.pos

        # S → B uno
        if self.parse_B(nodo):
            if self.consumir("uno"):
                nodo.agregar(Nodo("uno"))
                return True

        # S → dos C
        self.pos = pos_respaldo
        nodo.hijos.clear()
        if self.consumir("dos"):
            nodo.agregar(Nodo("dos"))
            if self.parse_C(nodo):
                return True

        # S → ε
        self.pos = pos_respaldo
        nodo.hijos.clear()
        nodo.agregar(Nodo("ε"))
        return True

    # A → dos C tres B C A'
    #   | uno tres B C A'
    #   | tres B C A'
    #   | cuatro A'
    #   | A'
    def parse_A(self, nodo_padre):
        nodo = nodo_padre.agregar(Nodo("A"))
        pos_respaldo = self.pos

        # A → dos C tres B C A'
        if self.consumir("dos"):
            nodo.agregar(Nodo("dos"))
            if self.parse_C(nodo):
                if self.consumir("tres"):
                    nodo.agregar(Nodo("tres"))
                    if self.parse_B(nodo) and self.parse_C(nodo) and self.parse_Ap(nodo):
                        return True

        # A → uno tres B C A'
        self.pos = pos_respaldo
        nodo.hijos.clear()
        if self.consumir("uno"):
            nodo.agregar(Nodo("uno"))
            if self.consumir("tres"):
                nodo.agregar(Nodo("tres"))
                if self.parse_B(nodo) and self.parse_C(nodo) and self.parse_Ap(nodo):
                    return True

        # A → tres B C A'
        self.pos = pos_respaldo
        nodo.hijos.clear()
        if self.consumir("tres"):
            nodo.agregar(Nodo("tres"))
            if self.parse_B(nodo) and self.parse_C(nodo) and self.parse_Ap(nodo):
                return True

        # A → cuatro A'
        self.pos = pos_respaldo
        nodo.hijos.clear()
        if self.consumir("cuatro"):
            nodo.agregar(Nodo("cuatro"))
            if self.parse_Ap(nodo):
                return True

        # A → A'  (A' puede ser ε)
        self.pos = pos_respaldo
        nodo.hijos.clear()
        if self.parse_Ap(nodo):
            return True

        self.pos = pos_respaldo
        nodo_padre.hijos.remove(nodo)
        return False

    # A' → cinco C seis uno tres B C A' | ε
    def parse_Ap(self, nodo_padre):
        nodo = nodo_padre.agregar(Nodo("A'"))
        pos_respaldo = self.pos

        if self.consumir("cinco"):
            nodo.agregar(Nodo("cinco"))
            if self.parse_C(nodo):
                if self.consumir("seis"):
                    nodo.agregar(Nodo("seis"))
                    if self.consumir("uno"):
                        nodo.agregar(Nodo("uno"))
                        if self.consumir("tres"):
                            nodo.agregar(Nodo("tres"))
                            if self.parse_B(nodo) and self.parse_C(nodo) and self.parse_Ap(nodo):
                                return True

        # A' → ε
        self.pos = pos_respaldo
        nodo.hijos.clear()
        nodo.agregar(Nodo("ε"))
        return True

    # B → A cinco C seis | ε
    def parse_B(self, nodo_padre):
        nodo = nodo_padre.agregar(Nodo("B"))
        pos_respaldo = self.pos

        if self.parse_A(nodo):
            if self.consumir("cinco"):
                nodo.agregar(Nodo("cinco"))
                if self.parse_C(nodo):
                    if self.consumir("seis"):
                        nodo.agregar(Nodo("seis"))
                        return True

        # B → ε
        self.pos = pos_respaldo
        nodo.hijos.clear()
        nodo.agregar(Nodo("ε"))
        return True

    # C → siete B | ε
    def parse_C(self, nodo_padre):
        nodo = nodo_padre.agregar(Nodo("C"))
        pos_respaldo = self.pos

        if self.consumir("siete"):
            nodo.agregar(Nodo("siete"))
            if self.parse_B(nodo):
                return True

        # C → ε
        self.pos = pos_respaldo
        nodo.hijos.clear()
        nodo.agregar(Nodo("ε"))
        return True

    def parsear(self):
        raiz = Nodo("S")
        exito = self.parse_S(raiz) and self.actual()[0] is None
        return exito, raiz


def calcular_posiciones(nodo, profundidad=0, contador=[0]):
    if not nodo.hijos:
        x = contador[0]
        contador[0] += 1
        nodo._x = x
        nodo._y = -profundidad
        return
    for hijo in nodo.hijos:
        calcular_posiciones(hijo, profundidad + 1, contador)
    nodo._x = sum(h._x for h in nodo.hijos) / len(nodo.hijos)
    nodo._y = -profundidad


def dibujar_arbol(nodo, ax):
    es_eps = nodo.etiqueta == "ε"
    es_hoja = not nodo.hijos
    color = "#E499DD" if es_eps else ("#7EC5C8" if es_hoja else "#AC4AD9")

    for hijo in nodo.hijos:
        ax.plot([nodo._x, hijo._x], [nodo._y, hijo._y],
                color="#888888", linewidth=1, zorder=1)
        dibujar_arbol(hijo, ax)

    circulo = plt.Circle((nodo._x, nodo._y), 0.35,
                          color=color, zorder=2, ec="white", linewidth=1.5)
    ax.add_patch(circulo)
    fontsize = 7 if len(nodo.etiqueta) > 6 else 8
    ax.text(nodo._x, nodo._y, nodo.etiqueta,
            ha="center", va="center", fontsize=fontsize,
            color="white", fontweight="bold", zorder=3)


def mostrar_arbol(raiz, expresion, aceptada):
    calcular_posiciones(raiz, contador=[0])

    todas = []
    def recoger(n):
        todas.append(n)
        for h in n.hijos:
            recoger(h)
    recoger(raiz)

    xs = [n._x for n in todas]
    ys = [n._y for n in todas]

    fig, ax = plt.subplots(figsize=(max(10, (max(xs) - min(xs) + 2) * 0.6),
                                    max(6,  (max(ys) - min(ys) + 2) * 1.1)))
    ax.set_aspect("equal")
    ax.axis("off")
    dibujar_arbol(raiz, ax)

    estado = "ACEPTADA" if aceptada else "RECHAZADA"
    color_titulo = "#2ecc50" if aceptada else "#e74c3c"
    ax.set_title(f'"{expresion}"  ->  {estado}',
                 fontsize=12, fontweight="bold", color=color_titulo, pad=14)

    leyenda = [
        mpatches.Patch(color="#AC4AD9", label="No terminal"),
        mpatches.Patch(color="#7EC5C8", label="Terminal"),
        mpatches.Patch(color="#E499DD", label="epsilon (vacio)"),
    ]
    ax.legend(handles=leyenda, loc="upper right", fontsize=8, framealpha=0.8)
    ax.set_xlim(min(xs) - 1, max(xs) + 1)
    ax.set_ylim(min(ys) - 1, 1.5)
    plt.tight_layout()
    nombre = re.sub(r'[^a-zA-Z0-9]', '_', expresion)[:40]
    plt.savefig(f"arbol_{nombre+str(time.time())[-2:]}.png", dpi=150, bbox_inches='tight')
    plt.close()


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Uso: python3 parser_gramatica2.py <archivo>")
        sys.exit(1)

    with open(sys.argv[1], "r") as f:
        lineas = f.read().splitlines()

    for linea in lineas:
        if not linea.strip():
            continue
        tokens = tokenizar(linea)
        parser = Parser(tokens)
        aceptada, arbol = parser.parsear()
        print(f'{"ACEPTADA" if aceptada else "RECHAZADA"}  "{linea}"')
        mostrar_arbol(arbol, linea, aceptada)
