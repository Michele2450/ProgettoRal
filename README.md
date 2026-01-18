# Knister RL Agent - Deep Q-Network

Questo repository contiene l'implementazione di un agente di Reinforcement Learning progettato per giocare al gioco **Knister**. L'agente utilizza una DQN per apprendere strategie di posizionamento dei valori in una griglia 5x5, con l'obiettivo di massimizzare il punteggio finale basato su combinazioni (coppie, scale, full house).

---

## Implementazione della Rete Neurale (QNet)

* Input Layer: Accetta un vettore di stato di dimensione 36 (25 celle normalizzate + 11 per il dado).
* **Hidden Layers:**
    * Primo layer: 256 neuroni con attivazione ReLU.
    * Secondo layer: 128 neuroni con attivazione ReLU.
    * Terzo layer: 64 neuroni con attivazione ReLU.
* **Output Layer:** Restituisce 25 valori.
L'architettura "a imbuto" (256 -> 128 -> 64) è stata scelta per favorire l'estrazione e la compressione delle feature rilevanti man mano che l'informazione attraversa la rete e per evitare overfitting facendo si che la rete mantenga solo le informazioni più importanti.

---

## Scelte Implementative Chiave

Per migliorare la convergenza e la stabilità dell'addestramento, sono state adottate le seguenti tecniche:

### 1. Rappresentazione dello Stato (One-Hot Encoding)
Lo stato del gioco non è passato alla rete come semplici valori numerici grezzi.
* **Griglia:** I 25 valori delle celle sono normalizzati (divisi per 12.0).
* **Dado:** Il valore del dado corrente (da 2 a 12) è codificato tramite **One-Hot Encoding** in un vettore di dimensione 11. Questo permette alla rete di distinguere nettamente ogni possibile esito del lancio.

### 2. Action Masking
Durante la selezione dell'azione (sia in fase di training che di test), viene applicata una **maschera** ai Q-values. Le azioni non valide (celle già occupate) vengono forzate a un valore estremamente basso (`-1e5`), garantendo che l'agente scelga sempre e solo posizioni legali. Questo accelera drasticamente l'apprendimento rispetto all'uso di sole penalità nel reward dato che la rete dovrebbe prima imparare a giocare normalmente (quindi a riempire tutte le caselle in 25 mosse) e poi imparare le varie strategie di gioco.

---

## Iperparametri

Di seguito i principali parametri utilizzati per l'addestramento finale:

* **Learning Rate:** `5e-4` valore scelto per la stabilità, con 500.000 episodi un learning rate più alto poteva portare ad aggiornamenti troppo bruschi.
* **Gamma:** `0.99` per valorizzare le ricompense a lungo termine.
* **Batch Size:** `64` valore molto più piccolo rispetto al memory size in modo da evitare correlazioni temporali. 
* **Tau:** `1e-3` (aggiornamento soft della rete target).
* **Episodi Totali:** `500.000`.
* **Epsilon:** `0.99995` Decadimento lineare da `1.0` a `0.20` dopo circa 80.000 episodi di esplorazione la epsilon si farma a 0.20 in modo da farlo esplorare ogni 5 mosse per poter mantenere un buon tasso di apprendimento fino alla fine.
* **Memory size:** `100.000` in modo da mantenere una giusta quantità di esperienze in memoria.
* **Learn step:** `10` valore scelto in modo da bilanciare tempo di training e performance, un valore più basso avrebbe reso il training troppo lungo.

---

## Risultati

Le prestazioni dell'agente sono state valutate su un set di test di **500 partite** (senza esplorazione).

**Punteggio Medio:** **42.63** punti.
Questo punteggio varia in base alla "fortuna" che l'agente può avere nelle 500 partite, oscillando dai 40 a 43. 

## Test manuale
Per poter testare il modello con il checkpoint salvato basta eseguire tutte le celle tranne quella del training e del plot dei grafici.