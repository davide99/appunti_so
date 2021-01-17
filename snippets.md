# Sezioni critiche

## Mutex: soluzione 1

```
int flag[2] = {FALSE, FALSE};
```
**Pi**
```
while(1){
    while(flag[j]);  <-- 1
    flag[i]=TRUE;    <-- 2
    //SC
    flag[i]=FALSE;
}
```
**Pj**
```
while(1){
    while(flag[i]);  <-- 3
    flag[j]=TRUE;    <-- 4
    //SC
    flag[j]=FALSE;
}
```
**Mutex non garantita**: 1->3->4->2 (problema: interruzione tra test e set)


## Mutex: soluzione 2
Da soluzione 1, inverto test & set

```
int flag[2] = {FALSE, FALSE};
```
**Pi**
```
while(1){
    flag[i]=TRUE;    <-- 1
    while(flag[j]);  <-- 2
    //SC
    flag[i]=FALSE;
}
```
**Pj**
```
while(1){
    flag[j]=TRUE;    <-- 3
    while(flag[i]);  <-- 4
    //SC
    flag[j]=FALSE;
}
```
**Deadlock non garantita**: 1->3->4->2

## Mutex: soluzione 3
Niente test & set

```
int turn = i;
```
**Pi**
```
while(1){
    while(turn != i);
    //SC
    turn = j;
}
```
**Pj**
```
while(1){
    while(turn != j);
    //SC
    turn = i;
}
```
**Deadlock**: No, turn può essere uguale solo a i o a j (non entrambi)

**Starvation**: Se non eseguo mai Pj, Pi entrerà in sezione critica una sola volta

## Mutex: soluzione 4
Mix di 2 e 3

```
int turn = i;
int flag[2] = {FALSE, FALSE};
```
**Pi**
```
while(1){
    flag[i] = TRUE;
    turn = j;
    while(flag[j] && turn==j);
    //SC
    flag[i] = FALSE;
}
```
**Pj**
```
while(1){
    flag[j] = TRUE;
    turn = i;
    while(flag[i] && turn==i);
    //SC
    flag[j] = FALSE;
}
```
**Deadlock**: No, turn può essere uguale solo a i o a j (non entrambi)

**Starvation**: Se non eseguo mai Pj, Pi entrerà in sezione critica una sola volta


## TestAndSet
```
char lock=FALSE;

while(1){
    while(TestAndSet(&lock));
    //SC
    lock=FALSE;
    //non SC
}
```
Dove la TestAndSet può essere implemetata come:
```
char TestAndSet(char *lock){
    char old = *lock;
    *lock = TRUE;
    return old;
}
```

## Swap
```
char lock=FALSE;

while(1){
    char token = TRUE;
    while(token)
        swap(&token, &lock);
    //SC
    lock=FALSE;
    //non SC
}
```
Dove la swap può essere implemetata come:
```
void swap(char *v1, char *v2){
    char tmp = *v1;
    *v1 = *v2;
    *v2 = tmp;
}
```

## Semafori
**Wait**
```
void wait(sem_t *sem){
    sem->count--;
    if (sem->count < 0){
        sem->head.push(sem->proc);
        sem->proc.block()
    }
}
```
**Signal**
```
void signal(sem_t *sem){
    sem->count++;
    if (sem->count <= 0){ //<= perchè c'è il ++
        proc = sem->head.pop(sem->proc);
        proc.wake();
    }
}
```

# Produttori e consumatori

## 1 Produttore, 1 Consumatore

```
init(full, 0);    //occupati
init(empty, MAX); //liberi
```

**Producer**
```
while(1){
    produce(&val);
    wait(empty); //aspetto si liberi almeno un posto
    enqueue(val);
    signal(full); //segnalo che c'è un nuovo elemento
}
```

**Consumer**
```
while(1){
	wait(full); //aspetto che ci sia almeno un elemento da prelevare
	dequeue(&val);
	signal(empty); //segnalo che c'è un posto libero
	consume(val);
}
```

## n Produttori, n Consumatori

```
init(full, 0);    //occupati
init(empty, MAX); //liberi
init(meP, 1);     //Mutex produttori
init(meC, 1);     //Mutex consumatori
```

**Producer**
```
while(1){
    produce(&val);
    wait(empty);
    wait(meP);     <----
    enqueue(val);
    signal(meP);   <----
    signal(full);
}
```

**Consumer**
```
while(1){
	wait(full);
    wait(meC);    <----
	dequeue(&val);
    signal(meC);  <----
	signal(empty);
	consume(val);
}
```

# Readers & Writers

* Readers in concorrenza
* Writers in mutex con altri readers/writers

## Precendeza ai reader

```
nR = 0;          <------- Numero readers
init(me_nR, 1);  <------- Mutex su variabile nR
init(me_w, 1);   <------- Mutex su writers
```

**Reader**
```
wait(me_nR);
    nR++;
    if(nR==1)         <------ Primo reader?
        wait(me_w);   <------ Blocco i writers (precendenza ai readers)
signal(me_nR);

//LETTURA

wait(me_nR);
    nR--;
    if(nR==0)         <------ Niente più readers?
        signal(me_w); <------ Sblocco i writers
signal(me_nR);
```

**Writer**
```
wait(me_w);           <------ Mutex writers + precendenza
//SCRITTURA
signal(me_w);
```

## Precendeza ai writers

```
nW = 0;
nR = 0;
init(me_w, 1);
init(me_nW, 1);
init(me_r, 1);
init(me_nR, 1);
```

**Reader**
```
wait(me_r);           <--+- per (s)bloccare reader da writer
    wait(me_nR);         |  (novità rispetto al precendente)
        nR++;            |  *
        if(nR==1)        |
            wait(w);     |
    signal(me_nR);       |
signal(me_r);         <--+

//LETTURA

wait(me_nR);
    nR--;
    if(nR==0)
        signal(me_w);
singal(me_nR);
```

**Writer**
```
wait(me_nW);
    nW++;
    if(nW==1)
        wait(me_r);  <--- blocco reader
signal(me_nW);

wait(me_w);
//SCRITTURA
signal(me_w);

wait(me_nW);
    nW--;
    if(nW==0)
        signal(me_r); <--- sblocco reader
signal(me_nW);
```

**NB**: La novità per il writer rispetto al precedente è solo l'aggiunta della gestione della variabile condivisa nW + il blocco/sblocco readers.

L'aggiunta delle righe * nel reader serve a fare in modo che, quando un writer blocca i reader, vengano fatti partire nuovi reader. In questo modo, man mano, nR diminuirà, fino a farlo arrivare a 0 e sbloccare i writers.

## Tunnel a senso alternato
Due insiemi di readers

```
n1 = n2 = 0;
init(me_n1, 1); init(me_n2, 1);
init(busy, 1);
```
**Reader 1**
```
wait(me_n1);
    n1++;
    if(n1==1)
        wait(busy);
signal(me_n1);

//Lettura

wait(me_n1);
    n1--;
    if(n1==0)
        signal(busy);
signal(me_n1);
```
**Reader 2**
```
wait(me_n2);
    n2++;
    if(n2==1)
        wait(busy);
signal(me_n2);

//Lettura

wait(me_n2);
    n2--;
    if(n2==0)
        signal(busy);
signal(me_n2);
```
**Writer**
```
wait(busy);
//Scrittura
signal(busy);
```