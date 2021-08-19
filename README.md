# Klassekart

`klassekart.py` er et python-skript som  leser inn en liste med elever og så genererer et randomisert klassekart.

![Fra navneliste til klassekart!](illustrasjon.png)

## Gjenerer klassekart

1. Ha en klasselisten i Excel.
2. Kopier python-skriptet `klassekart.py` til samme mappe.
3. Eksporter klasselisten fra excel til en `.csv` fil.
   * Velg `Lagre en kopi` i excel
   * Velg så filtypen `CSV UTF-8 (kommadelt) (*.csv)`
   * Lagre!

4. Åpne `klassekart.py` i en kode-editor (Thonny, Visual Studio Code, e.l.)
5. Endre på relevante variabler i filen slik at csv-filen blir lest in, se under, og kjør så skriptet.

## Variabler som kan endres på
```python
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
bord_savnes = []

# Mål i milimeter
side_bredde = 210
side_høyde = 290
margin = 15

gang_bredde = 10
rad_høyde = 10

bord_mellomrom = 2
bord_høyde = 10
```