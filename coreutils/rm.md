# rm
Analisi del tool `rm`.

### Descrizione
`rm` è un tool che permette ("unlinkare") di eliminare dei file dal filesystem.  
Di default non include l'eliminazione delle cartelle, ma è possibile attraverso un parametro (`-r` o `-R` oppure `--recursive`) di procedere alla cancellazione anche di quest'ultime.  
Se si utilizza rm per la cancellazione dei file è probabile che sia comunque possibile recuperare parte del loro contenuto, a patto che si abbiano le conoscenze e il tempo (è una procedura che può richiederne molto) necessari. Per avere la certezza che il contenuto dei file venga cancellato è consigliabile usare `shred`.
Con l'opzione `--interactive` è possibile stabilire quando e se richiedere la conferma prima della cancellazione dei file. Se non vengono passati valori al parametro equivale ad aver invocato l'opzione `-I` o `--interactive=once`, e ciò comporta la richiesta di conferma una sola volta prima di procedere o annullare l'intera esecuzione.

### Sintassi
La sintassi per l'utilizzo di `rm` è la seguente:  
```bash
rm [OPTION]... FILE...
```

### Opzioni
```
	-f, --force
		Ignora i file e argomenti mancanti, non chiede conferma

	-i
		Chiede conferma prima di procedere con ogni eliminazione

	-I
		Chiede conferma una sola volta prima di procedere con l'eliminazione solo nel caso si stia per cancellare più di tre file o si sta eseguendo la rimozione in modo ricorsivo.

	--interactive[=QUANDO]
		Chiede conferma prima di procedere con l'eliminazione secondo il valore di `QUANDO`. `never` non chiede mai la conferma della rimozione, `once` equivale a `-I` (conferma una singola volta), `always` equivale a `-i` (chiede conferma prima di eliminare ogni singolo file). Se l'opzione non ha dei valori equivale a `-i` (chiede conferma prima di eliminare ogni singolo file)

	--one-file-system
		Durante la rimozione di file in modo ricorsivo salta le cartelle che non appartengono al file system specificato dai parametri

	--no-preserve-root
		Non tratta in modo speciale '/' (root), ovvero viene cancellata anch'essa.

	--preserve-root
		Non elimina la root (opzione usata di default)

	-r, -R, --recursive
		Cancella le cartella e il loro contenuto il modo ricorsivo

	-d, --dir
		Cancella le cartelle vuote

	-v, --verbose
		Stampa a schermo quello che sta attualmente accadendo

	--version
		Stampa la versione del programma
```

### Analisi del codice
Il codice di `rm` si trova [qui](http://git.savannah.gnu.org/cgit/coreutils.git/tree/src/rm.c).  
La repository git che contiene tutte le coreutils è reperibile al seguente indirizzo: [http://git.savannah.gnu.org/cgit/coreutils.git/tree/src/rm.c](http://git.savannah.gnu.org/cgit/coreutils.git/tree "Repository git di coreutils"). I sorgenti dei programmi sono situati nella sottocartella `src/`.
Passiamo ora all'analisi del codice.



### Manuale di rm
E' possibile leggere il manuale ufficiale di `rm` al seguente link: [manuale](http://linux.die.net/man/1/rm "Manuale di rm").

### Autori di rm
Paul  Rubin, David MacKenzie, Richard M. Stallman, e Jim Meyering.

### Link utili per riportare eventuali bug del software
GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
Report rm translation bugs to <http://translationproject.org/team/>

### Copyright di rm
Copyright © 2014 Free Software Foundation, Inc.   License  GPLv3+:  GNU  
GPL version 3 or later <http://gnu.org/licenses/gpl.html>.  
This  is  free  software:  you  are free to change and redistribute it.  
There is NO WARRANTY, to the extent permitted by law.  
