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
La repository git che contiene tutte le coreutils è reperibile al seguente indirizzo: [http://git.savannah.gnu.org/cgit/coreutils.git/tree](http://git.savannah.gnu.org/cgit/coreutils.git/tree "Repository git di coreutils"). I sorgenti dei programmi sono situati nella sottocartella `src/`.

Passiamo ora all'analisi del codice.  
Il file che contiene il `main` è `rm.c`.  
Come prima parte di codice, oltre ai commenti di intestazione, troviamo le seguenti diciture:
```c

#include <config.h>
#include <stdio.h>
#include <getopt.h>
#include <sys/types.h>
#include <assert.h>

#include "system.h"
#include "argmatch.h"
#include "error.h"
#include "remove.h"
#include "root-dev-ino.h"
#include "yesno.h"
#include "priv-set.h"

```
Gli include sono necessari per includere determinate librerie utili al funzionamento del programma.  
- `getopt.h` è utilizzato per il parse (getopt_long) delle opzioni invocate dalla riga di comando (es: `-I`, `--recursive`).  
- `types.h` è necessario per la definizione di alcuni tipi di dato.  
- `remove.h` contiene le funzioni che eseguono la chiamata di sistema per l'unlinking("cancellazione") del file dal filesystem.

Viene poi definito il nome ufficiale del programma:
```c
#define PROGRAM_NAME "rm"
```

E gli autori:
```c
#define AUTHORS \
  proper_name ("Paul Rubin"), \
  proper_name ("David MacKenzie"), \
  proper_name ("Richard M. Stallman"), \
  proper_name ("Jim Meyering")
```
`proper_name` è una direttiva per l'utility [`gettext`](http://linux.die.net/man/1/gettext "Manuale di gettext") che si occupa della traduzione dei messaggi. E' necessario per indicare in quale modo devono essere eventualmente tradotti i nomi in altre lingue (viene utilizzato quando si hanno caratteri "strani" e non codificabili in una situazione normale).

Vengono poi create alcune strutture ed enumerazioni (`enum`).
```c
enum
{
  INTERACTIVE_OPTION = CHAR_MAX + 1,
  ONE_FILE_SYSTEM,
  NO_PRESERVE_ROOT,
  PRESERVE_ROOT,
  PRESUME_INPUT_TTY_OPTION
};
```
Contiene la lista di opzioni "long" (es: `--interactive`) che `rm` prende da riga di comando che non hanno un equivalente "short" (es: `-r` per `--recursive`).

```c
static struct option const long_opts[] =
{
  {"force", no_argument, NULL, 'f'},
  {"interactive", optional_argument, NULL, INTERACTIVE_OPTION},

  {"one-file-system", no_argument, NULL, ONE_FILE_SYSTEM},
  {"no-preserve-root", no_argument, NULL, NO_PRESERVE_ROOT},
  {"preserve-root", no_argument, NULL, PRESERVE_ROOT},

  /* This is solely for testing.  Do not document.  */
  /* It is relatively difficult to ensure that there is a tty on stdin.
     Since rm acts differently depending on that, without this option,
     it'd be harder to test the parts of rm that depend on that setting.  */
  {"-presume-input-tty", no_argument, NULL, PRESUME_INPUT_TTY_OPTION},

  {"recursive", no_argument, NULL, 'r'},
  {"dir", no_argument, NULL, 'd'},
  {"verbose", no_argument, NULL, 'v'},
  {GETOPT_HELP_OPTION_DECL},
  {GETOPT_VERSION_OPTION_DECL},
  {NULL, 0, NULL, 0}
};
```
Questa struct è un match delle opzioni "long" con le equivalenti "short", per esempio: `--force` con `-f` o `--recursive` con `-r`.

```c
static char const *const interactive_args[] =
{
  "never", "no", "none",
  "once",
  "always", "yes", NULL
};
```
La struct descritta qui sopra serve invece per definire quali parametri può assumere l'opzione `--interactive`, il che significa che da riga di comando possiamo usare `--interactive=never`, `--interactive=always` e così via.

```c
static enum interactive_type const interactive_types[] =
{
  interactive_never, interactive_never, interactive_never,
  interactive_once,
  interactive_always, interactive_always
};
```
L'enumerazione descritta qui sopra serve per assegnare un valore numerico (intero, partendo da 0) ai parametri di `--interactive` per avere un match con quelle in versione stringa create sopra.

```c
ARGMATCH_VERIFY (interactive_args, interactive_types);
n```
Viene poi verificata la corrispondenza degli argomenti.  
Sono riuscito a trovare la dichiarazione della funzione `ARGMATCH_VERIFY` qui: [argmatch.h](http://opensource.apple.com/source/gnutar/gnutar-441/gnutar/lib/argmatch.h), [argmatch.c](http://opensource.apple.com/source/gnutar/gnutar-441/gnutar/lib/argmatch.c).  
La funzione `ARGMATCH_VERIFY` è definita attraverso il seguente frammento di codice presente in `argmatch.h`:
```c
# define ARGMATCH_VERIFY(Arglist, Vallist)				  \
  struct argmatch_verify						  \
  {									  \
    char argmatch_verify[ARGMATCH_CONSTRAINT(Arglist, Vallist) ? 1 : -1]; \
  }
```
Il `#define` precedente ritorna una struct contenente un valore di tipo char che viene generato sulla base dell'eleaborazione della funzione `ARGMATCH_CONSTRAINT` che esegue un controllo sul `sizeof` (diviso per il numero degli elementi) dei parametri (per far si che il match sia corretto il `sizeof` deve risultare uguale), per fare ciò si appoggia alla macro `ARRAY_CARDINALITY` che si occupa di calcolare il numero di elementi presenti negli argomenti (Arglist e Vallist).  
All'interno del nostro codice non viene fatto alcun controllo sul valore di ritorno.

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
