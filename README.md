# Dynamiczne struktury danych - listy

Lista, to rozproszona struktura danych, podobna w zastosowaniu do tablicy. Różnica pomiędzy listą a tablicą polega na sposób zajmowania pamięci komputera. W przypadku tablicy dla wszystkich elementów jest przypisywany spójny blok pamięci, natomiast w przypadku własnych dynamicznych struktur są to dowolne miejsca w pamięci. Z tego powodu podczas implemetnacji własnych struktur danych programista musi zapewnić dostęp do kolejnych elementów danych, które są ułożone w pamięci niezależnie od siebie. Do zachowania ciągłości dostępu do kolejnych elementów wykorzystywane są więc wskaźniki. Przykładowa struktura wierzchołka listy jednokierunkowej zawiera 2 zmienne konieczne do utworzenia listy – klucz (wartość przechowywaną w pamięci) oraz wskaźnik na kolejny element listy.

*Przykład (16.0) Struktura wierzchołka listy jednokierunkowej przechowujące klucze całkowitoliczbowe*

```
struct node
{
	int key;
	struct node* next;
};
```

*Schemat (16.0) lista jednokierunkowa o wierzchołkach z przykładu 16.0*

![s1 ](https://user-images.githubusercontent.com/71324202/168259048-539f1516-2913-4e1c-aefe-606523fac9f0.png)

*Schemat (16.1) lista jednokierunkowa o wierzchołkach z przykładu 16.0 o konkretych danych*

![s2](https://user-images.githubusercontent.com/71324202/168259051-b328ce01-59db-4597-9d46-37d868114dae.png)

W listach pierwszy element nazywany jest głową, a ostanti ogonem. W tym rozdziale zostanie zaprezentowana praktyczne budowa listy.

## Dodawanie elementów do listy

Dodawanie elementu od strony głowy składa się z 4 zasadniczych czynności:
 - utworzenia nowego elementu
 - przypisania wartości
 - wskazania kolejnego elementu (poprzedniej głowy)
 - zmiany wskaźnika odpowiadającego za głowę

*Przykład (16.1) funkcja addToHead zwracająca wskaźnik*

```
struct node* addToHead(struct node* head, int value)
{
	/* za alokowanie pamięci na nowy wierzchołek */
	struct node* newNode = (struct node*)malloc(sizeof(struct node));
	/* sprawdzenie poprawności alokacji */
	if(newNode==NULL)
	{
		fprintf(stderr, "Memory allocation error for node.");
		return head;
	}
	/* nadanie wartości klucza dla newNode */
	newNode->key = value;
	/* ustawienie wartości wskaźnika next na obecną głowę */
	newNode->next = head;
	/* zwrócenie newNode jako nową głowę */
	return newNode;
	
}

/* sposób wywołania funkcji */
int main(int argc, char *argv[] ) {
	
	struct node* head = NULL;
	head = addToHead(head,3);
	return 0;
}

```

Analiza przykładu (16.1) – krok po kroku, co dzieje się z naszą listą. 

Załóżmy, że nasz main wygląda następująco:

```
int main(int argc, char *argv[] ) {
	
	struct node* head = NULL;
	head = addToHead(head,3);
	head = addToHead(head,5);
	head = addToHead(head,4);
	return 0;
}

```


![addHead1](https://user-images.githubusercontent.com/71324202/168259061-ade4f9a7-01f9-49c2-9575-1ec5b8fb19e5.png)

Po wykonaniu tej operacji ustawiamy wskaźnik główny (tej znajdujący się w main) na NULL, czyli tworzymy pustą listę.

Krok po kroku funkcja addToHead - `head = addToHead(head,3);` :

![addHead2](https://user-images.githubusercontent.com/71324202/168259062-520a7076-a939-4b0d-93be-e213f8c9e330.png)

Po wykonaniu tej operacji otrzymujemy blok zaalokowanej pamięci, nie znamy wartości jej parametrów. Jeżeliby zdarzyła się sytuacja, że pamięć nie zostałaby zaalokowana wtedy newNode przyjmie wartość NULL, i wtedy nasza funkcja zareaguje z odpowiednim komunikatem do stderr, oraz przerwie wykonywanie funkcji.

![addHead3](https://user-images.githubusercontent.com/71324202/168259064-c7df3e65-2e6a-452f-95a0-b0d4bb85a536.png)

Te operacje dają już nam kontrole nad zawartościami zmiennych składowych struktury wierzchołka

![addHead4](https://user-images.githubusercontent.com/71324202/168259065-df6b89d3-ec86-4cc6-b5f6-7cfc4da35cba.png)

Zwrócona wartość w funkcji `main`, zmienia wartość naszej głowy tak aby wskaźnik wskazywał na nasz nowo stworzony element.

Krok po kroku funkcja addToHead - `head = addToHead(head,5);` :

kroki w tej części są analogiczne do poprzedniego wywołania tej funkcji:

![addHead5](https://user-images.githubusercontent.com/71324202/168259067-ce127742-e95b-4629-a534-300cd54b26f4.png)

Krok po kroku funkcja addToHead - `head = addToHead(head,4)`;

![addHead6](https://user-images.githubusercontent.com/71324202/168259068-109fd3c2-a433-40af-9b06-08f23490d343.png)

*Przykład (16.1.1) funkcja addToHead w inny sposób*

```
void addToHead(struct node** head, int value)
{
	/* za alokowanie pamięci na nowy wierzchołek */
	struct node* newNode = (struct node*)malloc(sizeof(struct node));
	/* sprawdzenie poprawności alokacji */
	if(newNode==NULL)
	{
		fprintf(stderr, "Memory allocation error for node.");
	}
	/* nadanie początkowych wartości dla newNode */
	newNode->key = value;
	/* ustawienie wartości wskaźnika next na obecną głowę*/
	newNode->next = *head;
	/* ustawienie newNode jako nową głowę */
	*head = newnode;	
}
/* sposób wywołania funkcji w kodzie */
int main(int argc, char *argv[] ) {
	
	struct node* head = NULL;
	struct node** ptr_head = head;
	addToHead(ptr_head,3);
	return 0;
}

```

## Usuwanie elementów z listy

Podczas usuwania elementu z końca listy musimy panować cały czas nad wskaźnikami, które musimy zmienic oraz tymi, które wskazują na elementy do usunięcia.

*Przykład (16.2) funkcja removeFromTail*

```
{
	/* sprawdzenie czy głowa istnieje */
	if(head)
	{
		/* obsługa przypadku, gdy została tylko głowa */
		if(head->next==NULL)
		{
			free(head);
			return NULL;
		}
		/* utworzenie wskaźników tymczasowych/pomocniczych */
		struct node* tmp1 = head;
		struct node* tmp2 = head->next;
		/* odnalezienie ogona */
		while(tmp2->next!=NULL)
		{
			tmp1= tmp1->next;
			tmp2= tmp2->next;
		}
		/* usunięcie ogona */
		free(tmp2);
		/* wyzerowanie wskaźnika next nowego ogona */
		tmp1->next=NULL;
	}
	return head;
}

```

Analiza przykładu (16.2) krok po kroku, co się dzieje z naszą listą. Załóżmy, że nasz main z analizy 16.1 zmienimy następująco:

```
int main(int argc, char *argv[]) {

	struct node* head = NULL;
	head = addToHead(head,3);
	head = addToHead(head,5);
	head = addToHead(head,4);
	head = removeFromTail(head); /* 1 */
	head = removeFromTail(head); /* 2 */
	head = removeFromTail(head); /* 3 */
	return 0;
}
```

Zakładamy, że elementy listy zostały dodane (tak jak w omówieniu funkcji `addToHead`)

![removeFromTail1](https://user-images.githubusercontent.com/71324202/168259072-f6181a58-1575-473b-a6a2-31e525ce8113.png)

Wchodzimy wewnątrz funkcji `removeFromTail`, gdzie na start sprawdzany jest warunek czy istnieje `head`, czyli czy aby przypadkiem lista nie jest pusta. Jest on spełniony, ponieważ w naszej liście są 3 elementy więc `head` istnieje (jest różny od NULL). Drugi warunek sprawdza czy `head->next` istnieje, czyli czy aby głowa nie była jednocześnie ogonem. Ten warunek nie jest spełniony, także omijamy kod wewnątrz zawarty (wrócimy tam w odpowiednim momencie).

![removeFromTail2](https://user-images.githubusercontent.com/71324202/168259074-048c8f12-02fd-40d6-9b7d-4d3c2a77c72a.png)

Tworzymy dwa wskaźniki tymczasowe, które pozwolą nam przemieszczać się swobodnie po liście, nie ruszając wskaźnika głowy. Na początek wskazuą one ten sam element co `head` (`tmp1`) oraz jego parametr `next` (`tmp2`). W tym przypadku potrzebne są nam 2 wzkaźniki, koniecznie kontrolowane w taki sposób, aby wskazywały dwa sąsiednie elementy. Dlaczego jest to konieczne zostanie przedstawione w kolejnych krokach.

![removeFromTail3](https://user-images.githubusercontent.com/71324202/168259077-e531f13c-7bc8-4306-8e96-21562171524a.png)

Pętla ustawia nam wskażnik `tmp2` na ogonie, a `tmp1` jeden element przed ogonem. W tym wypadku kończy się ona po pierwszej iteracji. 

![removeFromTail4](https://user-images.githubusercontent.com/71324202/168259078-7bb6ab41-5cd3-4d36-be2f-3a0a0d12931c.png)

To miejsce w kodzie jest uzasadnieniem dlaczego zostały użyte aż dwa wskaźniki pomocnicze. Zauważmy, że z ogona, nie mamy dostępu do naszego poprzednika, przez co niemożliwym byłoby skorygowanie jego wartości next, która po operacji zwolnienia pamięci nadal wskazuje na tę wartość. Nad nią jednak już straciliśmy konrtolę i nie możemy zostawić naszej listy w takim stanie. Spowodowałoby to błędy operacji na pamięci, ponieważ nasza lista posiadała by "dziurę", przez którą najprawdopodobniej wpadniemy w niekontrolowany przez nas obszar pamięci. Aby temu zapodbiec wykonujemy kolejną operację właśnie dzięki wzkaźnikowi pomocniczemu `tmp1` pokazaną poniżej.

![removeFromTail5](https://user-images.githubusercontent.com/71324202/168259079-2cc792cb-5775-4041-aff2-8111bf3ef4b2.png)

![removeFromTail6](https://user-images.githubusercontent.com/71324202/168259037-e590b54e-050f-4ec7-8875-f172b00692dc.png)

Kolejne dwa wywołania funkcji zadziałają analogicznie do tego opisanego powyżej.

![removeFromTail7](https://user-images.githubusercontent.com/71324202/168259042-af548939-341b-4653-8e2c-34c91b3ca294.png)

![removeFromTail8](https://user-images.githubusercontent.com/71324202/168259044-33261014-1dbd-40ad-bfc5-777941062ae1.png)

Warto zauważyć, że ostatni element usuwany jest poprzez wejście do wnętrza drugiej instrukcji `if`.

## Wyświetlanie listy oraz zostawianie porządu w pamięci

Powyższe schematy zachowania listy można bardzo łatwo i szybko udowodnić w programie. Wystarczy napisać dodatkowo funkcję wypisującą zawartość listy działającej na zasadach zaprogramowanych przez nas.

*Przykład (16.3) funkcja printList*

```
void printList(struct node* head)
{
	struct node* tmp = head;
	while(tmp!=NULL)
	{
		printf(" %d", tmp->key);
		tmp=tmp->next;
	}
	printf("\n");
}

int main(int argc, char *argv[]) {

	struct node* head = NULL;
	printList(head);
	head = addToHead(head,3);
	printList(head);
	head = addToHead(head,5);
	printList(head);
	head = addToHead(head,4);
	printList(head);
	head = removeFromTail(head);
	printList(head);
	head = removeFromTail(head);
	printList(head);
	head = removeFromTail(head);
	printList(head);
	return 0;
}


```

*Wynik działania programu*


> 3
>
> 5 3
>
> 4 5 3
>
> 4 5
>
> 4

Istotną rzeczą jest oczywiście pozostawienie po użytkowaniu listy porządku w pamięci i zwolnienie wszelkiej zaalokowanej pamięci (tak jak w przypadku tablic). Dlatego dobrym nawykiem jest stworzenie funkcji, która zwolni całą przestrzeń jaka zajmuje lista, za jednym zamachem. 

*Przykład (14.4) funkcja delList*

```
struct node* delList(struct node* head)
{
	while(head)
	{
		head=removeFromTail(head);
	}
	return NULL;
}
```

Po połączeniu przykładów 16.0-16.4 otrzymamy w pełni funkcjionalny kod listy jednokierunkowej, niecyklicznej oraz moglibyśmy na jego podstawie utworzyć kolejkę typu FIFO.

## Kolejka FIFO LIFO i listy dwukierunkowe oraz cykliczne

Kolejka FIFO to skrót nazwy first-in first-out. Jest to taka najbardziej naturalnie wyglądająca kolejka, ponieważ można ją spotkać w życiu codziennym np.  w sklepie. Osoba, która przyjdzie jako pierwsza, zostanie obsłużona jako pierwsza. Tak samo jest w przypadku struktury kolejki FIFO – pierwszy dodany do niej element zostanie jako pierwszy usunięty. W celu stworzenia takiej kolejki potrzebne są nam dwie funkcje enqueue jako dodawanie elementu do kolejki oraz dequeue  jako usuwanie elementu z kolejki. Taką kolejkę można zrobić na podstawie przykładów (16.0-16.4). Jako enqueue służyłaby nam funkcja addToHead, a jako dequeue funkcja removeFromTail. Analogicznie można stworzyć funkcje o nazwach removeFromHead i addToTail, i na ich podstawie stworzyć kolejkę, która nawet byłaby trochę bardziej praktyczna, gdyż cały czas mielibyśmy bezpośredni dostęp jako do pierwszego wskaźnika listy do elementu, który jest na wyjściu kolejki.

*Schemat (16.2) kolejka FIFO*

![s2](https://user-images.githubusercontent.com/71324202/168259051-b328ce01-59db-4597-9d46-37d868114dae.png)

Kolejka LIFO (analogicznie last-in last-out), można zobrazować za pomocą stosu (stack) poprzez stos talerzy. Jeżeli chcemy wyjąć talerz na spodzie, to musimy najpierw zdjąć wszystkie talerze znajdujące się na docelowym. Taką listę obsługujemy poprzez funkcję push (dodawanie do stosu) i pop (zdejmowanie ze stosu). Korzystając z uprzednio napisanych funkcji z przykładów (14.0-14.4) można stworzyć stos tworząc dodatkowo funkcję removeFromHead lub addToTail gdzie opcja z funkcjami operującymi na głowie jest znowu bardziej intuicyjna do wykonania i obsługi.

*Schemat (16.3) kolejka LIFO*

![s3](https://user-images.githubusercontent.com/71324202/168259055-5867bf26-a61a-4bca-805e-8ce688aa112d.png)

Listy mogą być również dwukierunkowe. W takim przypadku programista musi zaprogramować struktury wskaźnik na poprzedni element listy. 

*Schemat (16.4) lista dwukierunkowa*

![s4](https://user-images.githubusercontent.com/71324202/168259057-10281b00-d0a8-466f-a853-e5a8d182bfbe.png)

Lista cykliczna, to lista, której ostatni element wskazuje na pierwszy, a w przypadku listy dwukierunkowej jeszcze pierwszy element wskazuje na ostatni.  

*Schemat (16.5) lista cykliczna dwukierunkowa*

![s5](https://user-images.githubusercontent.com/71324202/168259059-cd4527e3-e629-47c9-9d89-339e607d7f9b.png)

## Zadania do samodzielnego wykonania:

*WKRÓTCE*


***
[Poprzednia część]() | [Spis treści](https://github.com/CyberMALab/Wprowadzenie-do-programowania-w-j-zyku-ANSI-C.git) | [Następna część]()
***
&copy; Cyberspace Modelling and Analysis Laboratory

