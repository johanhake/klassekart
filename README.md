#!/usr/bin/env python3
# -*- coding: utf-8 -*-
__author__ = "Johan Hake & Knut Skrindo"
__copyright__ = "Copyright (C) 2021 Johan Hake"
__license__ = "Public Domain"
__version__ = "0.1"

import random
import time
from functools import reduce
from math import ceil
import os

import svgwrite
from svgwrite import mm

import pandas as pd

from svglib.svglib import svg2rlg
from reportlab.graphics import renderPDF, renderPM

################# Endre på parametrerne her for ønsket resultat #######################

# INPUT: CSV-fil med navnene
csv_fil = "Elever R1 21-22.csv"

# Navn på kolonnen som innholder navnene som skal brukes i klassekartet
kolonne = "Fornavn"

# Hva skiller verdiene i csv-filen?
sep = ";"

# Hvilken encoding er csv-filen skriven med: "ISO-8859-1" eller "utf-8"
csv_encoding = "utf-8"

# Beskrivelse av rommets fordeling av bord
kolonner = [2, 3, 2]

# Navn på klassen kan også legge til ekstra
klasse = "R1 20/21"

# Bruk enheter som pt eller em
fontStørrelse = "8pt"

# Liste med en kombinasjon av "pdf", "svg" eller "png"
# NB! png har dårlig oppløsning dpi=72. Fix er å lage siden større
output_format = ["pdf"]

# Liste med visningsalternativ: "lærer", "elev"
visningsalternativ = ["lærer", "elev"]

# Liste med bord som ikke finnes.
# Borden skrives inn med en indeks til eleven (0, 1, 2, osv) med start lengst frem til venstre (lærervisning).
bord_savnes = [4, 15]

# Mål i milimeter
side_bredde = 210
side_høyde = 290
margin = 15

gang_bredde = 10
rad_høyde = 10

bord_mellomrom = 2
bord_høyde = 10

################# IKKE ENDRE PÅ NOE UNDER #######################

# Leser inn filen med elever
elever_csv = pd.read_csv(csv_fil, sep=sep, encoding=csv_encoding)
elever = elever_csv[kolonne].to_list()

# Stokker elevlista.
random.shuffle(elever)

# Sjekker om noen av de som er i lista foran sitter etter plass 18.
# I så fall flytter jeg dem til en tilfeldig plass på en av de tre første radene.
# for elev in foran:
#     if elever.index(elev) >= 18:
#         elever.remove(elev)
#         plass = random.randint(0,18-len(foran))
#         elever.insert(plass, elev)

# Finner avhengige parametrer
antall_elever = len(elever)
antall_bord_savnes = len(bord_savnes)
bord_per_rad = reduce(lambda a, b: a + b, kolonner)

antall_gang = len(kolonner) - 1
antall_bord_mellomrom = reduce(lambda a, b: a + b, (k - 1 for k in kolonner))
antal_rader = ceil((antall_elever + antall_bord_savnes) / bord_per_rad)

bord_bredde = (side_bredde - (2*margin + antall_gang*gang_bredde + antall_bord_mellomrom*bord_mellomrom))/bord_per_rad

# Skriver en tekst på x, y koord
def legg_til_tekst(dwg, x, y, tekst, attribs=None):
    attribs = attribs or {}
    tekst = dwg.text(tekst, insert=(x*mm, y*mm))
    for attr, verdi in attribs.items():
        tekst.attribs[attr] = verdi
    dwg.add(tekst)

# Lager et bord på angitt coordinat
def tegn_bord(dwg, x, y, navn_ind):
    if isinstance(navn_ind, int):
        if navn_ind >= len(elever):
            return
        navn = elever[navn_ind]
    elif isinstance(navn_ind, str):
        navn = navn_ind

    # Legger til en rektangel
    dwg.add(dwg.rect((x*mm, y*mm), (bord_bredde*mm, bord_høyde*mm), 2, 2, stroke="black", fill="white"))

    # Setter attributter til teksten
    attribs = {
        "text-anchor": "middle",
        "dominant-baseline": "central",
        "font-size": fontStørrelse
               }

    # Legger til navnet
    legg_til_tekst(dwg, x + bord_bredde/2, y + bord_høyde/2, navn, attribs)

# Lager et klassekart
def lag_klassekart(rettning):

    if rettning not in ["lærer", "elev"]:
        raise ValueError("Argumentet 'rettning' må være en av 'lærer' eller 'elev'")

    # Oppretter en A4-side
    filnavn = f"klassekart_{klasse.replace(' ', '_').replace('/', '-')}_{rettning}"
    dwg = svgwrite.Drawing(f"{filnavn}.svg", size=(side_bredde*mm, side_høyde*mm))

    y = margin

    # Overskrift
    legg_til_tekst(dwg, side_bredde/2, y, "Klassekart " + klasse, attribs={
        "text-anchor": "middle",
        "font-size": "2em",
        "font-weight": "bold"})

    y += 2*rad_høyde

    if rettning == "lærer":
        y += (rad_høyde + bord_høyde)*antal_rader

    # Tegner kateteret
    tegn_bord(dwg, (side_bredde-bord_bredde)/2, y, "Kateter")

    if rettning == "elev":
        y += rad_høyde + bord_høyde
    else:
        y -= rad_høyde + bord_høyde

    navn_ind = 0
    bord_ind = 0
    for rad in range(antal_rader):
        x = margin
        for kol in kolonner:
            for bord in range(kol):
                if bord_ind not in bord_savnes:
                    tegn_bord(dwg, x, y, navn_ind)
                    navn_ind += 1
                else:
                    tegn_bord(dwg, x, y, "")

                x += bord_bredde + bord_mellomrom
                bord_ind += 1

            x += gang_bredde - bord_mellomrom

        if rettning == "elev":
            y += rad_høyde + bord_høyde
        else:
            y -= rad_høyde + bord_høyde

    # Lagrer filen
    dwg.save()

    # Håndterer output format
    if "pdf" in output_format or "png" in output_format:
        svg_fil = svg2rlg(f"{filnavn}.svg")
        if "pdf" in output_format:
            renderPDF.drawToFile(svg_fil, f"{filnavn}.pdf")
        if "png" in output_format:
            renderPM.drawToFile(svg_fil, f"{filnavn}.png", fmt="png")

    if "svg" not in output_format:
        os.remove(f"{filnavn}.svg")


# Lager et klassekart for de ulike visningsalternativene
for alt in visningsalternativ:
    lag_klassekart(alt)

