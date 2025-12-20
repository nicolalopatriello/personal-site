---
title: 'AST è ovunque'
description: 'Come funziona AST e dove viene usato.'
pubDate: 'Dec 20 2025'
tags:
  - Javascript
  - AST
---


### Introduzione

Il codice che scriviamo può essere conciso ed elegante in molte forme diverse, e noi — o i nostri colleghi — 
siamo in grado di comprenderlo con maggiore o minore facilità.
Per una macchina, però, il codice non è immediatamente leggibile. 
Un compilatore o un tool di analisi hanno bisogno di strumenti differenti per poter dare senso a ciò che, per noi, è solo testo.

È in questo passaggio, dal codice come sequenza di caratteri al codice come struttura, 
che entra in gioco l’**Abstract Syntax Tree**. 
Ogni volta che uno strumento _comprende_ davvero il codice, sta lavorando su una rappresentazione simile.

### Cos’è un Abstract Syntax Tree

L’idea di rappresentare un programma come struttura astratta risale ai primi anni ’60,
nell’ambito della ricerca sui linguaggi formali e sui compilatori. 
In particolare, **John McCarthy**, creatore di Lisp, [introdusse](https://www-formal.stanford.edu/jmc/concepts/node9.html)
il concetto di abstract syntax come modo per 
descrivere le espressioni di un linguaggio al di là della loro rappresentazione testuale concreta. 
L’obiettivo era separare la forma superficiale del codice dal suo significato logico, creando una 
rappresentazione modulare e coerente che potesse essere analizzata e trasformata dai compilatori senza ambiguità. 
Questa intuizione è alla base di ciò che oggi chiamiamo Abstract Syntax Tree, 
utilizzato in tutti i linguaggi moderni per analisi, trasformazioni e ottimizzazioni.


Un Abstract Syntax Tree è una struttura dati che descrive la forma sintattica di un programma mettendo in evidenza 
le relazioni logiche tra i suoi elementi. L’idea di base è rappresentare il codice non come una sequenza di caratteri, 
ma come una struttura gerarchica, in cui ogni costrutto del linguaggio occupa una posizione ben definita.

Dal punto di vista teorico, un AST è un **grafo aciclico orientato**, che nella pratica assume la forma di un albero radicato. 
I **nodi** dell’albero rappresentano i costrutti sintattici del linguaggio — come espressioni, statement, 
operatori o identificatori — mentre gli **archi** descrivono le relazioni tra questi costrutti, 
ad esempio quali operandi appartengono a un’operazione o quali istruzioni compongono il corpo di una funzione.

La gerarchia dell’albero riflette direttamente la struttura del programma. 
I nodi più in alto rappresentano costrutti più generali, mentre i nodi foglia corrispondono agli elementi atomici, 
come variabili e valori letterali. In questo modo, l’AST codifica implicitamente informazioni fondamentali del linguaggio, 
come la precedenza degli operatori e l’annidamento dei blocchi.

L’aggettivo abstract indica che questa rappresentazione non conserva tutti i dettagli del codice sorgente. 
Elementi come spazi, commenti o parentesi ridondanti vengono scartati perché non contribuiscono al significato del programma. 
Ciò che rimane è una struttura compatta e coerente, progettata per essere facilmente attraversata, analizzata e trasformata 
da altri strumenti.


### Perché gli AST sono così usati

Nei compilatori, l’AST è una tappa naturale: serve come base per l’analisi semantica, le ottimizzazioni e la generazione del codice. Tuttavia, il suo utilizzo non si limita a questo contesto.

Molti strumenti moderni operano sull’AST perché hanno bisogno di comprendere la struttura del codice, non solo il testo. Questo permette di eseguire trasformazioni e analisi in modo preciso e affidabile, evitando ambiguità tipiche dell’analisi puramente testuale.

Ecco alcuni esempi concreti:

* **IDE ed editor**: grazie all’AST, gli strumenti possono effettuare refactoring sicuri, come rinominare variabili o estrarre funzioni, e abilitare funzionalità di navigazione come “go-to-definition” o “find references”.

* **Linting e analisi statica**: strumenti come ESLint o PyLint usano l’AST per rilevare pattern strutturali del codice, individuando errori o potenziali problemi prima che il programma venga eseguito.

* **Formatter**: software come Prettier riscrivono il codice in modo coerente, rispettando regole di stile e convenzioni, operando sull’AST per mantenere invariato il significato.

* **Transpiler e strumenti di trasformazione**: TypeScript → JavaScript, Babel o strumenti di refactoring avanzato usano l’AST come intermediario per trasformare il codice senza introdurre errori semantici.

In tutti questi casi, l’AST funge da _lingua franca interna_: rappresenta il codice in una forma strutturata che può essere analizzata, trasformata o ottimizzata in modo modulare. Grazie a questa rappresentazione, i tool moderni riescono a comprendere il codice quasi come farebbe un compilatore, rendendo possibili operazioni complesse senza compromettere la correttezza del programma.



### Un esempio concreto

Consideriamo una semplice espressione:

```text
a + b * c
```

Nel codice sorgente è una sequenza lineare di simboli, ma il suo significato dipende dalle regole del 
linguaggio. L’AST rende questa informazione esplicita:

```
      +
     / \
    a   *
       / \
      b   c
```

Il nodo radice rappresenta l’operatore `+`. Il suo operando destro non è un valore atomico, ma un sottoalbero che rappresenta l’operazione di moltiplicazione. 
In questo modo, la precedenza di `*` su `+` non è un dettaglio implicito, ma una proprietà strutturale dell’albero.

Questa rappresentazione non è solo teorica. È possibile esplorare AST reali generati da parser concreti usando strumenti dedicati. 
Un esempio particolarmente utile è [astexplorer](https://astexplorer.net/), che consente di inserire codice in vari linguaggi e visualizzare immediatamente l’AST prodotto da diversi parser.

* in **JavaScript**, parser come [acron](https://github.com/acornjs/acorn) producono AST navigabili secondo lo standard [ESTree](https://github.com/estree/estree)
* in **Python**, il modulo standard [ast](https://docs.python.org/3/library/ast.html) permette di trasformare codice sorgente in una struttura visitabile e manipolabile

In tutti questi casi, il principio rimane lo stesso: il codice viene trasformato in una struttura che 
rende esplicito ciò che nel testo è solo implicito.

### Conclusioni

L’Abstract Syntax Tree non è solo un concetto teorico né un dettaglio dei compilatori. 
È uno strumento fondamentale attraverso cui il codice viene interpretato, analizzato e trasformato dai tool moderni.

Comprendere come funziona un AST permette di vedere il codice non solo come testo, ma come una struttura con 
cui gli strumenti possono interagire in modo preciso e affidabile. In altre parole, l’AST è ciò che rende 
possibili operazioni complesse come refactoring, linting, formattazione e transpiling senza introdurre errori.