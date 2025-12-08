---
title: 'Dal comando date alle system call'
description: 'Dietro ogni comando si nasconde un''architettura stratificata.'
pubDate: 'Dec 8 2025'
tags:
  - Linux
  - kernel
---
### Il Kernel

Il kernel è il nucleo del sistema operativo, l'interfaccia diretta con l'hardware. Gestisce risorse critiche (CPU, memoria, I/O), implementa lo scheduling dei processi e fornisce un'astrazione uniforme dell'hardware attraverso le **system call** - il meccanismo con cui i programmi in user-space richiedono servizi privilegiati.

Ogni operazione che necessita accesso diretto all'hardware (leggere file, allocare memoria, ottenere il tempo di sistema) passa attraverso questo confine user/kernel space tramite syscall.

### Anatomia di un comando: Tracciare `date`

Analizziamo la catena di esecuzione del comando `date` per comprendere come un'utility userspace comunica con il kernel.

#### 1. Punto di partenza: `coreutils`

```bash
date --version
# date (GNU coreutils) 9.5
```

`date` appartiene a **coreutils**, la collezione fondamentale di utility POSIX. Esaminando [`date.c`](https://github.com/coreutils/coreutils/blob/v9.5/src/date.c) troviamo:

```c
gettime(&when);
```

La funzione non è definita localmente ma importata via:

```c
#include "system.h"
```

#### 2. Delega a glibc

[`system.h`](https://github.com/coreutils/coreutils/blob/v9.5/src/system.h) include:

```c
#include <sys/time.h>
#include <time.h>
```

Questi header appartengono alla **glibc** (GNU C Library), la libreria standard che wrappa le syscall Linux. Coreutils non implementa il timing, lo richiede al sistema.

#### 3. Wrapper glibc: `time()`

In [`time/time.c`](https://github.com/bminor/glibc/blob/master/time/time.c):

```c
time_t time(time_t *timer) {
  struct timespec ts;
  __clock_gettime(TIME_CLOCK_GETTIME_CLOCKID, &ts);
  
  if (timer)
    *timer = ts.tv_sec;
  return ts.tv_sec;
}
```

`__clock_gettime()` è ancora un wrapper interno, non la syscall vera.

#### 4. Implementazione reale: vDSO vs Syscall

In [`clock_gettime.c`](https://github.com/bminor/glibc/blob/master/sysdeps/unix/sysv/linux/clock_gettime.c) troviamo due percorsi:

**Fast path - vDSO**: Il kernel mappa una pagina di memoria in user-space contenente funzioni ottimizzate. `clock_gettime()` può leggere il tempo direttamente senza context switch.

**Slow path - Syscall tradizionale**:

```c
return INLINE_SYSCALL_CALL(clock_gettime, clock_id, tp);
```

Qui avviene il passaggio in kernel mode: interrupt software, context switch, esecuzione kernel-side, ritorno user-space.

### Flusso completo

```
coreutils/date.c
   ↓ gettime()
system.h
   ↓ #include <time.h>
glibc/time.c
   ↓ __clock_gettime()
glibc/clock_gettime.c
   ↓
   ├→ vDSO (fast path, no syscall)
   └→ syscall clock_gettime() (kernel mode)
```

### Perché è interessante?

Questo esercizio evidenzia:

1. **Stratificazione**: Le utility userspace si appoggiano su librerie standard che wrappano syscall
2. **Ottimizzazione**: vDSO elimina il costoso context switch per operazioni frequenti
3. **Astrazione**: Coreutils non conosce dettagli hardware, glibc nasconde la complessità kernel
4. **Debugging path**: Comprendere questa catena è essenziale per system debugging e performance analysis

Ogni comando apparentemente semplice nasconde un'architettura sofisticata che bilancia prestazioni, portabilità e sicurezza.