"""
Sistema de Inventario — Interfaz gráfica con tkinter
Paleta: blanco, crema y café
Requiere Python 3.8+ (tkinter viene incluido)
Ejecutar: python inventario_gui.py
"""

import tkinter as tk
from tkinter import ttk, messagebox
import json
import os
from datetime import datetime

# ─────────────────────────────────────────
#  ARCHIVOS DE DATOS
# ─────────────────────────────────────────
ARCH_PROD  = "productos.json"
ARCH_VENTA = "ventas.json"

def cargar_json(archivo):
    if os.path.exists(archivo):
        with open(archivo, "r", encoding="utf-8") as f:
            return json.load(f)
    return {}

def guardar_json(archivo, datos):
    with open(archivo, "w", encoding="utf-8") as f:
        json.dump(datos, f, ensure_ascii=False, indent=2)

# ─────────────────────────────────────────
#  PALETA DE COLORES
# ─────────────────────────────────────────
C = {
    "bg":        "#FAF6F0",   # crema suave — fondo general
    "surface":   "#FFFFFF",   # blanco puro — paneles
    "panel_bg":  "#F2EAE0",   # crema medio — sidebar
    "dark":      "#3B2A1A",   # café oscuro — textos, botón primario
    "mid":       "#7C5C3E",   # café medio — labels, bordes
    "light":     "#C4A882",   # café claro — bordes suaves
    "pale":      "#EDE0CF",   # crema — rows alternados, hover
    "accent":    "#5A3E28",   # café acento — hover botón
    "text":      "#2A1F14",   # texto principal
    "muted":     "#9A7B62",   # texto secundario
    "hint":      "#BFA990",   # texto hint
    "ok_bg":     "#EFF5E8",
    "ok_fg":     "#3B6D11",
    "err_bg":    "#FAECE7",
    "err_fg":    "#993C1D",
    "warn_bg":   "#FAEEDA",
    "warn_fg":   "#854F0B",
    "row_alt":   "#F7F2EC",
    "select":    "#EDE0CF",
}

FONT_TITLE  = ("Georgia", 13, "bold")
FONT_LABEL  = ("Georgia", 10)
FONT_SMALL  = ("Georgia", 9)
FONT_ENTRY  = ("Georgia", 11)
FONT_TABLE  = ("Georgia", 10)
FONT_NAV    = ("Georgia", 10, "bold")
FONT_STAT   = ("Georgia", 20, "bold")

# ─────────────────────────────────────────
#  CLASE PRINCIPAL
# ─────────────────────────────────────────
class App(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Sistema de Inventario")
        self.geometry("860x580")
        self.resizable(True, True)
        self.configure(bg=C["bg"])

        self.productos = cargar_json(ARCH_PROD)
        self.ventas    = cargar_json(ARCH_VENTA)
        if not isinstance(self.ventas, list):
            self.ventas = []
        self._vidx = len(self.ventas)

        self._build_ui()
        self._show_panel("cargar")

    # ── CONSTRUCCIÓN ──────────────────────
    def _build_ui(self):
        # Cabecera
        header = tk.Frame(self, bg=C["dark"], height=52)
        header.pack(fill="x")
        header.pack_propagate(False)
        tk.Label(header, text="SISTEMA DE INVENTARIO",
                 bg=C["dark"], fg=C["pale"],
                 font=("Georgia", 12, "bold"), letter_spacing=2).pack(side="left", padx=20, pady=14)
        self._lbl_fecha = tk.Label(header,
                 text=datetime.now().strftime("%d %b %Y"),
                 bg=C["dark"], fg=C["light"], font=FONT_SMALL)
        self._lbl_fecha.pack(side="right", padx=20)

        # Contenedor principal
        body = tk.Frame(self, bg=C["bg"])
        body.pack(fill="both", expand=True)

        # Sidebar nav
        nav = tk.Frame(body, bg=C["panel_bg"], width=160)
        nav.pack(side="left", fill="y")
        nav.pack_propagate(False)

        tk.Frame(nav, bg=C["light"], height=1).pack(fill="x")
        self._nav_btns = {}
        opciones = [
            ("cargar",      "Cargar producto"),
            ("stock",       "Ver stock"),
            ("venta",       "Registrar venta"),
            ("actualizar",  "Actualizar"),
            ("historial",   "Historial"),
        ]
        for key, label in opciones:
            btn = tk.Button(nav, text=label,
                            bg=C["panel_bg"], fg=C["mid"],
                            font=FONT_LABEL, relief="flat",
                            activebackground=C["pale"],
                            activeforeground=C["dark"],
                            anchor="w", padx=20, pady=10,
                            cursor="hand2",
                            command=lambda k=key: self._show_panel(k))
            btn.pack(fill="x")
            tk.Frame(nav, bg=C["pale"], height=1).pack(fill="x")
            self._nav_btns[key] = btn

        # Área de contenido
        self._content = tk.Frame(body, bg=C["surface"])
        self._content.pack(side="left", fill="both", expand=True)

        # Paneles
        self._panels = {}
        self._panels["cargar"]     = PanelCargar(self._content, self)
        self._panels["stock"]      = PanelStock(self._content, self)
        self._panels["venta"]      = PanelVenta(self._content, self)
        self._panels["actualizar"] = PanelActualizar(self._content, self)
        self._panels["historial"]  = PanelHistorial(self._content, self)

    def _show_panel(self, key):
        for k, p in self._panels.items():
            p.pack_forget()
        for k, b in self._nav_btns.items():
            b.configure(bg=C["panel_bg"], fg=C["mid"],
                        font=FONT_LABEL)
        self._panels[key].pack(fill="both", expand=True)
        self._nav_btns[key].configure(bg=C["pale"], fg=C["dark"],
                                      font=("Georgia", 10, "bold"))
        self._panels[key].on_show()

    def guardar_todo(self):
        guardar_json(ARCH_PROD,  self.productos)
        guardar_json(ARCH_VENTA, self.ventas)

# ─────────────────────────────────────────
#  WIDGETS REUTILIZABLES
# ─────────────────────────────────────────
def separador(parent, pady=6):
    tk.Frame(parent, bg=C["pale"], height=1).pack(fill="x", pady=pady)

def lbl_titulo(parent, texto):
    tk.Label(parent, text=texto.upper(),
             bg=C["surface"], fg=C["muted"],
             font=("Georgia", 10), pady=4).pack(anchor="w", padx=28, pady=(22, 4))
    separador(parent, pady=0)

def campo(parent, label, var, tipo="entry", width=32):
    row = tk.Frame(parent, bg=C["surface"])
    row.pack(fill="x", padx=28, pady=5)
    tk.Label(row, text=label.upper(), bg=C["surface"], fg=C["muted"],
             font=("Georgia", 9), width=18, anchor="w").pack(side="left")
    e = tk.Entry(row, textvariable=var, font=FONT_ENTRY,
                 bg=C["bg"], fg=C["text"],
                 relief="flat", bd=0,
                 insertbackground=C["dark"], width=width,
                 highlightthickness=1,
                 highlightbackground=C["light"],
                 highlightcolor=C["mid"])
    e.pack(side="left", ipady=5, padx=(0,4))
    return e

def btn_primario(parent, texto, cmd):
    b = tk.Button(parent, text=texto, command=cmd,
                  bg=C["dark"], fg=C["pale"],
                  font=("Georgia", 10, "bold"),
                  relief="flat", padx=18, pady=7,
                  activebackground=C["accent"],
                  activeforeground=C["pale"],
                  cursor="hand2")
    return b

def btn_secundario(parent, texto, cmd):
    b = tk.Button(parent, text=texto, command=cmd,
                  bg=C["surface"], fg=C["mid"],
                  font=FONT_LABEL,
                  relief="flat", padx=14, pady=6,
                  highlightthickness=1,
                  highlightbackground=C["light"],
                  activebackground=C["pale"],
                  cursor="hand2")
    return b

def toast_label(parent):
    lbl = tk.Label(parent, text="", bg=C["surface"], fg=C["ok_fg"],
                   font=FONT_SMALL, pady=5, padx=12)
    lbl.pack(fill="x", padx=28, pady=(0,4))
    return lbl

def mostrar_toast(lbl, msg, tipo="ok"):
    colores = {
        "ok":   (C["ok_bg"],   C["ok_fg"]),
        "err":  (C["err_bg"],  C["err_fg"]),
        "warn": (C["warn_bg"], C["warn_fg"]),
    }
    bg, fg = colores.get(tipo, (C["ok_bg"], C["ok_fg"]))
    lbl.configure(text=msg, bg=bg, fg=fg)
    lbl.after(3500, lambda: lbl.configure(text="", bg=C["surface"]))

def tabla(parent, cols, anchos, alto=260):
    style = ttk.Style()
    style.theme_use("clam")
    style.configure("Cafe.Treeview",
        background=C["surface"],
        fieldbackground=C["surface"],
        foreground=C["text"],
        rowheight=30,
        font=FONT_TABLE,
        bordercolor=C["pale"],
        relief="flat",
    )
    style.configure("Cafe.Treeview.Heading",
        background=C["bg"],
        foreground=C["muted"],
        font=("Georgia", 9),
        relief="flat",
        borderwidth=0,
    )
    style.map("Cafe.Treeview",
        background=[("selected", C["select"])],
        foreground=[("selected", C["dark"])],
    )
    frame = tk.Frame(parent, bg=C["surface"])
    frame.pack(fill="both", expand=True, padx=28, pady=10)

    scroll = tk.Scrollbar(frame, orient="vertical", bg=C["pale"],
                          troughcolor=C["bg"], width=8)
    tv = ttk.Treeview(frame, columns=cols, show="headings",
                      style="Cafe.Treeview",
                      yscrollcommand=scroll.set, height=10)
    scroll.config(command=tv.yview)

    for c, w in zip(cols, anchos):
        tv.heading(c, text=c.upper())
        tv.column(c, width=w, anchor="w")

    tv.tag_configure("alt", background=C["row_alt"])
    scroll.pack(side="right", fill="y")
    tv.pack(fill="both", expand=True)
    return tv

# ─────────────────────────────────────────
#  PANEL: CARGAR PRODUCTO
# ─────────────────────────────────────────
class PanelCargar(tk.Frame):
    def __init__(self, parent, app):
        super().__init__(parent, bg=C["surface"])
        self.app = app
        self._build()

    def _build(self):
        lbl_titulo(self, "Cargar producto")
        self._toast = toast_label(self)
        self._cod = tk.StringVar()
        self._nom = tk.StringVar()
        self._pre = tk.StringVar()
        self._sto = tk.StringVar()
        campo(self, "Código", self._cod)
        campo(self, "Nombre", self._nom)
        campo(self, "Precio ($)", self._pre)
        campo(self, "Stock inicial", self._sto)
        row = tk.Frame(self, bg=C["surface"])
        row.pack(anchor="w", padx=28, pady=(16, 0))
        btn_primario(row, "Guardar producto", self._guardar).pack(side="left", padx=(0, 10))
        btn_secundario(row, "Limpiar", self._limpiar).pack(side="left")

    def on_show(self): pass

    def _guardar(self):
        cod  = self._cod.get().strip().upper()
        nom  = self._nom.get().strip()
        pre  = self._pre.get().strip()
        sto  = self._sto.get().strip()
        if not cod or not nom:
            return mostrar_toast(self._toast, "Completá código y nombre.", "err")
        try:
            precio = float(pre); stock = int(sto)
            assert precio >= 0 and stock >= 0
        except:
            return mostrar_toast(self._toast, "Precio y stock deben ser números positivos.", "err")
        if cod in self.app.productos:
            return mostrar_toast(self._toast, f"El código '{cod}' ya existe.", "err")
        self.app.productos[cod] = {
            "nombre": nom, "precio": precio, "stock": stock,
            "fecha": datetime.now().strftime("%Y-%m-%d"),
        }
        self.app.guardar_todo()
        mostrar_toast(self._toast, f"'{nom}' guardado correctamente.", "ok")
        self._limpiar()

    def _limpiar(self):
        for v in [self._cod, self._nom, self._pre, self._sto]:
            v.set("")

# ─────────────────────────────────────────
#  PANEL: VER STOCK
# ─────────────────────────────────────────
class PanelStock(tk.Frame):
    def __init__(self, parent, app):
        super().__init__(parent, bg=C["surface"])
        self.app = app
        self._build()

    def _build(self):
        lbl_titulo(self, "Stock actual")
        # Stats
        self._stats = tk.Frame(self, bg=C["surface"])
        self._stats.pack(fill="x", padx=28, pady=(12, 8))
        self._s_prods  = self._stat_card(self._stats, "Productos",  "0")
        self._s_units  = self._stat_card(self._stats, "Unidades",   "0")
        self._s_val    = self._stat_card(self._stats, "Valor total","$0")
        separador(self, pady=0)
        # Tabla
        self._tv = tabla(self,
            ["Código", "Nombre", "Precio", "Stock", "Estado"],
            [90, 210, 90, 70, 100])

    def _stat_card(self, parent, label, val):
        f = tk.Frame(parent, bg=C["bg"], padx=16, pady=10)
        f.pack(side="left", padx=(0, 10))
        tk.Label(f, text=label.upper(), bg=C["bg"], fg=C["hint"],
                 font=("Georgia", 8)).pack(anchor="w")
        lv = tk.Label(f, text=val, bg=C["bg"], fg=C["dark"],
                      font=("Georgia", 18, "bold"))
        lv.pack(anchor="w")
        return lv

    def on_show(self):
        p = self.app.productos
        keys = list(p.keys())
        self._s_prods.configure(text=str(len(keys)))
        units = sum(p[k]["stock"] for k in keys)
        val   = sum(p[k]["precio"] * p[k]["stock"] for k in keys)
        self._s_units.configure(text=str(units))
        self._s_val.configure(text=f"${val:,.0f}")
        for row in self._tv.get_children():
            self._tv.delete(row)
        for i, k in enumerate(sorted(keys)):
            d = p[k]
            estado = "⚠ Bajo" if d["stock"] <= 3 else "OK"
            tag = "alt" if i % 2 else ""
            self._tv.insert("", "end", iid=k, tags=(tag,),
                values=(k, d["nombre"], f"${d['precio']:.2f}", d["stock"], estado))

# ─────────────────────────────────────────
#  PANEL: REGISTRAR VENTA
# ─────────────────────────────────────────
class PanelVenta(tk.Frame):
    def __init__(self, parent, app):
        super().__init__(parent, bg=C["surface"])
        self.app = app
        self._build()

    def _build(self):
        lbl_titulo(self, "Registrar venta")
        self._toast = toast_label(self)
        self._cod = tk.StringVar()
        self._qty = tk.StringVar()
        campo(self, "Código producto", self._cod)
        # Info del producto
        self._info = tk.Label(self, text="", bg=C["bg"], fg=C["mid"],
                              font=FONT_LABEL, anchor="w", padx=28, pady=6)
        self._info.pack(fill="x", padx=28)
        campo(self, "Cantidad", self._qty)
        row = tk.Frame(self, bg=C["surface"])
        row.pack(anchor="w", padx=28, pady=(14, 0))
        btn_secundario(row, "Buscar", self._buscar).pack(side="left", padx=(0, 10))
        btn_primario(row, "Confirmar venta", self._vender).pack(side="left")

    def on_show(self):
        self._cod.set(""); self._qty.set(""); self._info.configure(text="")

    def _buscar(self):
        cod = self._cod.get().strip().upper()
        if cod not in self.app.productos:
            self._info.configure(text="")
            return mostrar_toast(self._toast, "Producto no encontrado.", "err")
        p = self.app.productos[cod]
        self._info.configure(
            text=f"  {p['nombre']}   ·   ${p['precio']:.2f}   ·   {p['stock']} unidades disponibles",
            bg=C["bg"])

    def _vender(self):
        cod = self._cod.get().strip().upper()
        if cod not in self.app.productos:
            return mostrar_toast(self._toast, "Primero buscá el producto.", "err")
        try:
            qty = int(self._qty.get()); assert qty > 0
        except:
            return mostrar_toast(self._toast, "Ingresá una cantidad válida.", "err")
        p = self.app.productos[cod]
        if qty > p["stock"]:
            return mostrar_toast(self._toast, f"Stock insuficiente. Disponible: {p['stock']}.", "err")
        self.app._vidx += 1
        total = p["precio"] * qty
        self.app.ventas.append({
            "id":       f"V{self.app._vidx:04d}",
            "codigo":   cod,
            "nombre":   p["nombre"],
            "cantidad": qty,
            "precio":   p["precio"],
            "total":    total,
            "fecha":    datetime.now().strftime("%Y-%m-%d %H:%M"),
        })
        self.app.productos[cod]["stock"] -= qty
        self.app.guardar_todo()
        restante = self.app.productos[cod]["stock"]
        mostrar_toast(self._toast,
            f"Venta registrada — Total: ${total:.2f} — Stock restante: {restante}", "ok")
        self._cod.set(""); self._qty.set(""); self._info.configure(text="")

# ─────────────────────────────────────────
#  PANEL: ACTUALIZAR
# ─────────────────────────────────────────
class PanelActualizar(tk.Frame):
    def __init__(self, parent, app):
        super().__init__(parent, bg=C["surface"])
        self.app = app
        self._build()

    def _build(self):
        lbl_titulo(self, "Actualizar producto")
        self._toast = toast_label(self)
        self._cod  = tk.StringVar()
        self._sto  = tk.StringVar()
        self._pre  = tk.StringVar()
        campo(self, "Código producto", self._cod)
        self._info = tk.Label(self, text="", bg=C["bg"], fg=C["mid"],
                              font=FONT_LABEL, anchor="w", padx=28, pady=6)
        self._info.pack(fill="x", padx=28)
        campo(self, "Nuevo stock", self._sto)
        campo(self, "Nuevo precio ($)", self._pre)
        tk.Label(self, text="Dejá vacío lo que no querés cambiar.",
                 bg=C["surface"], fg=C["hint"], font=("Georgia", 9)).pack(anchor="w", padx=46, pady=(0,10))
        row = tk.Frame(self, bg=C["surface"])
        row.pack(anchor="w", padx=28, pady=(8, 0))
        btn_secundario(row, "Buscar", self._buscar).pack(side="left", padx=(0, 10))
        btn_primario(row, "Actualizar", self._actualizar).pack(side="left")

    def on_show(self):
        self._cod.set(""); self._sto.set(""); self._pre.set("")
        self._info.configure(text="")

    def _buscar(self):
        cod = self._cod.get().strip().upper()
        if cod not in self.app.productos:
            self._info.configure(text="")
            return mostrar_toast(self._toast, "Producto no encontrado.", "err")
        p = self.app.productos[cod]
        self._info.configure(
            text=f"  {p['nombre']}   ·   Stock: {p['stock']}   ·   Precio: ${p['precio']:.2f}",
            bg=C["bg"])

    def _actualizar(self):
        cod = self._cod.get().strip().upper()
        if cod not in self.app.productos:
            return mostrar_toast(self._toast, "Primero buscá el producto.", "err")
        sto = self._sto.get().strip()
        pre = self._pre.get().strip()
        if not sto and not pre:
            return mostrar_toast(self._toast, "Ingresá al menos un campo.", "warn")
        if sto:
            try:
                n = int(sto); assert n >= 0
                self.app.productos[cod]["stock"] = n
            except:
                return mostrar_toast(self._toast, "Stock inválido.", "err")
        if pre:
            try:
                n = float(pre); assert n >= 0
                self.app.productos[cod]["precio"] = n
            except:
                return mostrar_toast(self._toast, "Precio inválido.", "err")
        self.app.guardar_todo()
        mostrar_toast(self._toast, "Producto actualizado.", "ok")
        self._buscar()
        self._sto.set(""); self._pre.set("")

# ─────────────────────────────────────────
#  PANEL: HISTORIAL
# ─────────────────────────────────────────
class PanelHistorial(tk.Frame):
    def __init__(self, parent, app):
        super().__init__(parent, bg=C["surface"])
        self.app = app
        self._build()

    def _build(self):
        lbl_titulo(self, "Historial de ventas")
        self._stats = tk.Frame(self, bg=C["surface"])
        self._stats.pack(fill="x", padx=28, pady=(10, 8))
        self._s_cnt   = self._stat_card(self._stats, "Ventas",      "0")
        self._s_total = self._stat_card(self._stats, "Recaudado",   "$0")
        separador(self, pady=0)
        self._tv = tabla(self,
            ["ID", "Producto", "Cant.", "Precio", "Total", "Fecha"],
            [65, 190, 55, 80, 80, 120])

    def _stat_card(self, parent, label, val):
        f = tk.Frame(parent, bg=C["bg"], padx=16, pady=10)
        f.pack(side="left", padx=(0, 10))
        tk.Label(f, text=label.upper(), bg=C["bg"], fg=C["hint"],
                 font=("Georgia", 8)).pack(anchor="w")
        lv = tk.Label(f, text=val, bg=C["bg"], fg=C["dark"],
                      font=("Georgia", 18, "bold"))
        lv.pack(anchor="w")
        return lv

    def on_show(self):
        v = self.app.ventas
        self._s_cnt.configure(text=str(len(v)))
        total = sum(x["total"] for x in v)
        self._s_total.configure(text=f"${total:,.0f}")
        for row in self._tv.get_children():
            self._tv.delete(row)
        for i, venta in enumerate(reversed(v)):
            tag = "alt" if i % 2 else ""
            self._tv.insert("", "end", tags=(tag,), values=(
                venta["id"], venta["nombre"], venta["cantidad"],
                f"${venta['precio']:.2f}", f"${venta['total']:.2f}", venta["fecha"]
            ))

# ─────────────────────────────────────────
#  INICIO
# ─────────────────────────────────────────
if __name__ == "__main__":
    app = App()
    app.mainloop()

    
