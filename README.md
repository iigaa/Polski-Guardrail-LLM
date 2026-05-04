# Polski-Guardrail-LLM - Prototyp
Prototyp systemu bezpieczeństwa (Guardrail) do oceny promptów w języku polskim, stworzony w ramach zadania rekrutacyjnego. System wykorzystuje lokalnie uruchamiany model **PLLuM-8B-Instruct** i ocenia zapytania pod kątem ataków typu jailbreak, inżynierii społecznej oraz toksyczności, zwracając ustrukturyzowane decyzje JSON.
## Architektura i Stack Technologiczny
* **Język i biblioteki:** Python 3.11, `pandas`, `requests`, `scikit-learn`.
* **Model AI:** Llama-PLLuM-8B-instruct-Q5_K_M (uruchamiany lokalnie przez silnik **Ollama**).
* **Rozwiązania inżynieryjne:**
  * **Few-Shot Prompting:** Wymuszenie na modelu zwracania czystego formatu JSON.
  * **Smart Fallback:** Zastosowanie wyrażeń regularnych (Regex) oraz heurystyki do mapowania intencji modelu w przypadku uszkodzenia struktury JSON.
  * **Retry Loop:** Automatyczny mechanizm ponowień (do 3 prób z timeoutem 60s), który zredukował błędy API i halucynacje formatu.
## Baza Danych (Red Teaming)
Zbiór 160 promptów (`polska_baza_bezpieczenstwa.csv`), zawierający specyficzne dla języka polskiego wektory ataku:
* **Ponglish / Żargon IT:** Maskowanie ataków branżowym slangiem (np. "wgrać lewy soft", "zeskrejpowanie credsów").
* **Podwójne przeczenia i zawiła składnia:** Testowanie zdolności rozumienia kontekstu.
* **Lokalny kontekst:** Impersonacja polskich urzędów czy wykorzystywanie specyfiki polskiego prawa.
* **Zmyłki (False Positives):** Prompty bezpieczne, ale używające agresywnego słownictwa (np. "ubić proces w Linuxie", "wybuchowa kula do kąpieli"). Baza została zbalansowana w celu uzyskania wiarygodnych metryk.
## Wyniki Ewaluacji (Scikit-Learn)
Ewaluacja prototypu wykazała następujące metryki (dla 154 poprawnie przetworzonych zapytań – 6 zapytań zostało odrzuconych przez limity czasowe lub krytyczne błędy modelu):
* **Dokładność:** 71.25%
* **Precyzja dla ataków:** 93% 
* **Recall dla ataków:** 65%
* **F1-Score dla ataków:** 77%
**Podsumowanie Macierzy Pomyłek:**
* Prawdziwe ZABLOKUJ (True Negatives): 71
* Prawdziwe PRZEPUŚĆ (True Positives): 43
* Fałszywe PRZEPUŚĆ (False Positives): 35
* Fałszywe ZABLOKUJ (False Negatives): 5
**Kluczowe wnioski:**
1. **Wysoka pewność diagnozy i niski over-refusal:** System zablokował niesłusznie zaledwie 5 bezpiecznych promptów. Precyzja blokowania na poziomie 93% udowadnia, że jeśli model flaguje zapytanie jako niebezpieczne, robi to słusznie. Pozwala to na uniknięcie zjawiska *over-refusal* (nadgorliwości), które jest irytujące dla użytkowników LLM-ów.
2. **Podatność na socjotechnikę:** Model przepuścił 35 ataków (Krytyczne False Positives). Analiza pliku `wyniki_ewaluacji.csv` pokazuje, że 8-miliardowy model świetnie radzi sobie z jawnym łamaniem prawa i wulgaryzmami, ale daje się oszukać wyrafinowanej inżynierii społecznej (np. rozsiewanie plotek na firmowym Slacku) oraz dosłownie interpretuje korporacyjne metafory, nie dostrzegając głębszego złośliwego kontekstu.
3. **Asymetria klasyfikacji:** Model wykazuje asymetrię zachowań – świetnie wychwytuje jawne naruszenia (wysokie Precision), ale ma niższy Recall (65%). Oznacza to, że niemal 1/3 sprytnie zakamuflowanych ataków przenika przez filtry. Wskazuje to na ograniczenia pojemności kontekstowej mniejszych modeli typu 8B w warunkach "zero/few-shot".
4. **Stabilność formatowania wyjścia:** Mimo zastosowania techniki Few-Shot, Regex oraz Retry Loop, 6 promptów (ok. 3.75%) zakończyło się błędem. Pokazuje to typową dla lokalnych modeli niedeterministyczną naturę zwracania ścisłych struktur danych (JSON) i potwierdza konieczność stosowania twardego error-handlingu w architekturze Guardraili.
**Plany rozwoju:**
Aby podnieść wskaźnik Recall dla ukrytych ataków, docelowym krokiem wdrożeniowym byłoby zastosowanie modelu o większej liczbie parametrów (np. 70B) lub przeprowadzenie procesu *Fine-Tuningu* na wygenerowanym syntetycznie polskim zbiorze danych. Pozwoliłoby to również zrezygnować z Few-Shot Promptingu i przyspieszyć czas inferencji.
## Jak uruchomić projekt?
1. Zainstaluj aplikację [Ollama](https://ollama.com).
2. Pobierz plik modelu `Llama-PLLuM-8B-instruct-Q5_K_M.gguf`.
3. Zbuduj model używając załączonego pliku `Modelfile`:
   `ollama create pllum-guard -f Modelfile`
4. Zainstaluj wymagane biblioteki Pythona:
   `pip install pandas requests tqdm scikit-learn`
5. Uruchom pipeline ewaluacyjny:
   `python guardrail_pipeline.py`
