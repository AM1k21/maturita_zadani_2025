# Zadání maturitní práce
  
**Student:** Adam Mikulič  
**Školní rok:** 2025/2026  
**Předmět:** Informatika  
**Vedoucí práce:** Ing. Adam Ferencz

## 1. NÁZEV PRÁCE

Komunikační modul pro bezpilotní letoun

---

## 2. CÍL PRÁCE

Vyvinout a otestovat komunikační modul rozšiřující dosah komunikace bezpilotního letounu
---

## 3. FUNKČNÍ POŽADAVKY

- Realizace funkčního komunikačního modulu
- Testování stabilní komunikace na vzdálenost (minimálně 2x rozšíření dosahu)
- Integrace s letounem (držák + bezpečné vypuštění)
- Validace na reálných letových testech

---

## 4. POVINNÉ VÝSTUPY

### 4.1 Funkční projekt

- Funkční relay node
- Úspěšné letové testy

### 4.2 Vědecký/odborný článek

- **Rozsah:** 4-15 stran A4, formát dvou sloupců
- **Obsah:** Analýza komunikačních systémů, návrh řešení, implementace HW/SW, testování
- **Styl:** Mírně populárně-vědecký

### 4.3 Poster

- **Velikost:** A3 (297 × 420 mm)
- **Obsah:** Schéma zapojení, fotografie PCB, výsledky testů komunikace
- **Formát:** PDF pro tisk

### 4.4 Technická dokumentace

- **Rozsah:** Minimálně 10-20 stran
- **Formát:** Markdown nebo PDF (Word/LaTeX)
- **Obsah:** Technické specifikace komponent, schémata zapojení PCB, dokumentace firmware, montážní návod, provozní návod a údržba, testovací protokoly

### 4.5 Prezentace pro obhajobu

- **Rozsah:** 5-12 slidů
- **Délka obhajoby:** 15 minut

---

## 5. HODNOTICÍ KRITÉRIA

- **Funkčnost projektu (40%)** – Funkční relay node, úspěšné testy
- **Dokumentace (25%)** – Technická dokumentace, článek, poster
- **Prezentace a obhajoba (35%)** – Kvalita prezentace, odpovědi na dotazy

---

## 6. TERMÍN ODEVZDÁNÍ

**30. dubna 2026**

---

## 7. STRUKTURA ODEVZDÁNÍ

Počet vyhotovení: 1x zip soubor s prací a kompletními materiály

```tree
/projekt_mikulic/
  ├── README.md (stručný přehled projektu, popis obsahu a co otevřít jako první)
  ├── paper/ (vědecký článek a případné zdrojové soubory nebo přílohy)
  ├── poster/ (finální poster a případné exporty)
  ├── dokumentace/ (jedna nebo více částí technické dokumentace)
  ├── prezentace/
  └── projekt/
      ├── firmware/ (zdrojové kódy)
      ├── hardware/ (schémata PCB, gerber files)
      └── testy/ (protokoly, videa)
```

Uvnitř jednotlivých složek mohou být různé typy souborů podle charakteru projektu. Pokud není struktura uvnitř složek na první pohled zřejmá, musí být popsána v `README.md`.
