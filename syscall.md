# Processi
## Wait
```
pid_t wait(int *status);
```

**In**

`sys/wait.h`

**Comportamento**

Aspetta che uno dei figli del chiamante termini

**Parametri**

`status` => Stato di ritorno del figlio

`WIFEXITED(status)` => **IF EXITED**: il figlio è terminato correttamente?

`WEXITSTATUS(status)` => **EXIT STATUS**: estrate exit status da status (8 LSB)

**Ritorno**

`-1` se non ci sono figli

## WaitPid
```
pid_t waitpid(pid_t pid, int *status, int options);
```

**Comportamento**

Analogo a wait, ma più fine

**Parametri**

`pid` <= Pid figlio

`options` <=
* `0`: bloccante
* `WNOHANG`: non bloccante

## Exec

6 versioni:
* `execl`, `execlp`, `execle`
* `execv`, `execvp`, `execve`

Con:
* `l` => lista (vararg) lista di parametri
* `v` => vettore di parametri
* `p` => cerca nel PATH
* `e` => vettore di variabili d'ambiente

**Il primo elemento della lista/vettore è un nome fittizio**

# File

## Open
```
int open(const char* file, int flags, [mode_t mode]);
```

**In**

`fnctl.h`

**Comportamento**

Apre un file descriptor

**Parametri**

`file` <= Nome file

`flags` <=
* `O_RDONLY`: Sola lettura
* `O_WRONLY`: Sola scrittura
* `O_RDWD`: Lettura e scrittura
* `O_CREAT`: Crea il file se non esiste
* `O_APPEND`: Apre il file in append

[`mode`] <= Permessi sul file

**Ritorno**
* File descriptor
* `-1`: nel caso di errore

## Read
```
int read(int fd, void *buff, size_t nbytes);
```
**In**

`unistd.h`

**Comportamento**

Legge byte da un file descriptor

**Parametri**

`fd` <= File descriptor

`buffer` => Buffer

`nbytes` <= Bytes da leggere

**Ritorno**
* Numero di byte letti
* `-1`: nel caso di errore

## Write
```
int write(int fd, void *buff, size_t nbytes);
```
**Comportamento**

Scrive byte su un file descriptor

**Parametri**

`fd` <= File descriptor

`buffer` <= Buffer

`nbytes` <= Bytes da leggere

**Ritorno**
* Numero di byte scritti
* `-1`: nel caso di errore

## Close
```
int close(int fd);
```
**Comportamento**

Chiude il file descriptor

**Parametri**

`fd` <= File descriptor

**Ritorno**
* `0`: nel caso di successo
* `-1`: nel caso di errore

## Lseek
```
off_t lseek(int fd, off_t offset, int whence);
```
**Comportamento**

Riposizione cursore file

**Parametri**

* `fd` <= File descriptor
* `offset` <= Offset
* `whence` <=
    * `SEEK_SET`: dall'inizio del file
    * `SEEK_CUR`: dalla posizione corrente
    * `SEEK_END`: dalla fine del file (offset dev'essere <= `-1`)

**Ritorno**
* `offset`: nel caso di successo
* `-1`: nel caso di errore


# Segnali
## Pause
```
int pause();
```

**In**

`unistd.h`

**Comportamento**

Sospende il processo sino all'arrivo di un segnale

**Ritorno**

`-1`

## Kill
```
int kill(pid_t pid, int sig);
```

**In**

`signal.h`

**Comportamento**

Invia un segnale ad un processo

**Parametri**

* `pid` <= Pid processo
* `sig` <= Segnale

**Ritorno**
* `offset`: nel caso di successo
* `-1`: nel caso di errore