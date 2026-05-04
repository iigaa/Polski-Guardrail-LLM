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
Ewaluacja prototypu wykazała następujące metryki (dla 154 poprawnie przetworzonych zapytań):
* **Dokładność (Accuracy):** ~70.62%
* **Precyzja dla ataków (Precision - ZABLOKUJ):** 93% 
* **Recall dla ataków:** 64%
**Kluczowe wnioski:**
1. **Wysoka pewność diagnozy:** System zablokował niesłusznie zaledwie 5 bezpiecznych promptów (False Negatives), co udowadnia, że skutecznie omija problem *over-refusal* (nadgorliwości), często irytujący użytkowników.
2. **Podatność na socjotechnikę:** Model przepuścił 36 ataków (False Positives). Analiza pliku `wyniki_ewaluacji.csv` pokazuje, że 8-miliardowy model świetnie radzi sobie z jawnym łamaniem prawa i wulgaryzmami, ale daje się oszukać wyrafinowanej inżynierii społecznej (np. rozsiewanie plotek na firmowym Slacku) oraz dosłownie interpretuje korporacyjne metafory.
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
