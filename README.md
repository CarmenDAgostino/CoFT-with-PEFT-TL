# Sviluppo di un approccio Parameter-Efficient per il Fine Tuning collaborativo di LLM basato su Transferability Estimation

Questo progetto presenta un sistema per l’adattamento efficiente di modelli pre-addestrati a nuovi task, riducendo drasticamente i costi computazionali tipici del fine-tuning completo. L’approccio sviluppato unisce la leggerezza delle tecniche **Parameter-Efficient Fine-Tuning (PEFT)** con la capacità del **Transfer Learning** di sfruttare la conoscenza già acquisita da modelli pre-addestrati per affrontare nuovi compiti in modo rapido.

Il sistema si basa su un pool di **adapter LoRA**, ciascuno derivato da un modello pre-addestrato e ottimizzato per un task specifico. Questi moduli compatti conservano le conoscenze acquisite durante l’addestramento e le rendono riutilizzabili, riducendo significativamente i costi di adattamento. Il pool rappresenta così una forma di "conoscenza acquisita" su una varietà di task downstream, che può essere sfruttata per un adattamento rapido a nuovi task.

## Funzionamento del sistema

![Descrizione del sistema](images/diagramma%20approccio%20tesi.drawio.png)

1. **Selezione dell'adapter migliore**
   Quando un utente desidera addestrare un modello su un nuovo task target, il primo passo è selezionare dal pool un adapter che riesca a trasferire informazioni utili dal task su cui è stato originariamente addestrato al nuovo compito. Poiché valutare l'intero pool può essere oneroso, il sistema estrae casualmente un sottoinsieme di *N* adapter, sui quali viene poi stimata la trasferibilità.

2. **Stima della trasferibilità**
   Il sistema valuta la compatibilità di ciascun adapter nel sottoinsieme casuale con il task target sfruttando una tra diverse metriche di trasferibilità. Fatto ciò identifica l'adapter più efficace per il trasferimento di conoscenza, ovvero quello con il punteggio di trasferibilità più alto.

4. **Fine-Tuning rapido**
   L’adapter selezionato è integrato in un nuovo modello, il quale viene sottoposto ad un fine-tuning rapido. Poiché l’adapter ha già appreso strutture informative rilevanti per un task simile, il fine-tuning richiede poche epoche di addestramento (tipicamente 1 o 2).

5. **Valutazione e Integrazione dell'adapter nel pool**
    Al termine del fine-tuning, il modello ottenuto viene restituito all'utente per essere valutato. Se il modello soddisfa i requisiti di prestazione, l'utente può decidere di contribuire al pool rendendo disponibile l’adapter appena perfezionato. Se invece il modello non soddisfa i requisiti di prestazione, il sistema procede selezionando il secondo miglior adapter del sottoinsieme iniziale e ripete il processo di addestramento e valutazione. Il processo si ripete per un massimo di *k* iterazioni, (con *k* definito dall'utente), al fine di individuare un adapter più adatto al task. Se dopo *k* tentativi nessun adapter del sottoinsieme supera la soglia di prestazione, il sistema passa all'addestramento con LoRA standard. Anche in questo caso, al termine del fine-tuning, vengono valutate le prestazioni e se il modello addestrato con LoRA standard supera la soglia, il relativo adapter viene aggiunto al pool. 

## Metriche di trasferibilità
Il sistema mette a disposizione 5 metriche di trasferibilità per valutare la trasferibilità degli adapter del sottoinsieme casuale rispetto al task target. Tra esse vi sono: 
- NCE
- H-SCORE
- LEEP
- LogME
- NLEEP

## Esperimenti
Il sistema è stato valutato sfruttando un pool di adapter semplificato contenente adapter LoRA addestrati sui dataset: AgNews, SST-2, EmoInt, MNLI, PAWS. Invece, come task target sono stati usati i dataset: 20NewsGroup, DBpedia14, IMDB, Sentiment140, Emotion, RTE, QQP e CoLA.  
Il modelli adottati nei test sono BERT e RoBERTa. Le prestazioni del metodo sono state confrontate con un fine-tuning tradizionale basato su LoRA.  
Con entrambi i modelli, gli esperimenti mostrano come il metodo riesca a raggiungere performance comparabili con il fine-tuning tradizionale basat su LoRA, riducendo sia i tempi che le emissioni.

![Risultati con BERT](images/risultati%20BERT.png)  
![Risultati con RoBERTa](images/risultati%20RoBERTA.png)
