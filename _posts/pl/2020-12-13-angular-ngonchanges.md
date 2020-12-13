---
title: Błędna detekcja zmian w Angularze 
date: 2020-11-28 20:00:00.000000000 +02:00
identifier: angular-ngonchanges
categories:
- frontend
tags:
- angular
- javascript
excerpt:
  Dlaczego czasami mechanizm detekcji zmian w Angularze nie działa tak, jak
  tego oczekujemy? Jak jest zaimplementowany? Odpowiedź już dzisiaj w poście!
---
Spotkałem się ostatnio z sytuacją, w której zmiana wartości @Input()'a nie
była przechwytywana w hooku **ngOnChanges** w komponencie Angulara. To dosyć
osobliwe, ponieważ to właśnie do przechwytywania takich zmian wprowadzono do
Angulara ten hook. Jak to możliwe? Otóż *wartość* rzeczywiście się zmieniła,
jednak *referencja* - nie. Tak właśnie działa Angualar - przy wykrywaniu zmian
używa operatora ścisłej równości (===), a więc porównuje referencje, nie 
wartości. Jak dokładnie ten fakt potrafi "zepsuć" mechanizm detekcji zmian?

## Przykład

Powiedzmy, że mamy @Input() o nazwie "items", będący tablicą obiektów typu,
zgadza się, Item. Próbujemy wykryć zmiany wartości tego pola w hooku 
**ngOnChanges**:

{% highlight typescript %}
@Input() items: Items[];

ngOnChanges(changes: SimpleChanges) {
  if (changes.items) {
    console.log("'items' property has changed!");
  }
}
{% endhighlight %}

Przeciwnie do wszelkich oczekiwań, ten kod nie wykrył żadnych zmian, nawet mimo
faktu, że widziałem jak zawartość tablicy "items" ulega zmianie (a nawet jak
zmienia się jej długość!) - logowałem ją w obsłudze zdarzeń kliknięcia, na
zakończenie operacji asynchronicznych itp.

Nie budując napięcia w nieskończoność zdradzę, że problem istniał w komponencie
wyższego rzędu, w sposobie dodawania/usuwania/modyfikowania elementów tablicy
"items". Zobaczmy:

{% highlight typescript %}
const newItem = { ...some definition };
items.push(newItem);
{% endhighlight %}

W tym przykładzie, "items" jest ciągle tą samą tablicą, co oznacza tę samą
referencję, a więc **brak wykrycia zmiany wartości**. Ten sam efekt mógłby
zostać zaobserwowany po użyciu każdej z mutujących metod typu *Array*, jak
na przykład:

{% highlight typescript %}
.fill()
.pop()
.reverse()
.shift()
.splice()
.sort()
.unshift()
{% endhighlight %}

Efekt ten zaobserwujemy też przekazując do komponentu obiekty - 
dodawanie/usuwanie kluczy lub modyfikacja wartości nie zmienia referencji
obiektu!

## Rozwiązanie

Istnieją 2 rozwiązania tego problemu:

* tworzenie nowego obiektu/tablicy za każdym razem, gdy zmienia się jego wartość
* (nie zalecane) napisanie własnego mechanizmu wykrywania zmian w hooku **ngDoCheck** 

Skorzystałem z pierwszej opcji, a więc przepisałem wszystkie miejsca, gdzie 
tablica "items" była mutowana. Jak dokładnie?

### Tworzenie nowego obiektu zamiast modyfikacji

{% highlight typescript %}
// items.push(newItem)
items = items.concat([newItem])
items = [...items, newItem]

// items.pop()
items = items.slice(0, items.length - 1)

// items.splice(5, 10)
items = items.filter((_, index) => index < 5 || index >= 5 + 10)
{% endhighlight %}

Przykłady powyżej pokazują jak można zastąpić najczęściej używane metody 
tablicy tak, aby efekt użycia był ten sam, ale stworzony został nowy obiekt.
Można skorzystać też z pewnych "trików" na wymuszenie stworzenia płytkiej
kopii (*ang. shallow copy*) tablicy (w moim przypadku to było wystarczające):

{% highlight typescript %}
// they are equivalent
const copyArray = array => [...array]
const copyArray2 = array => array.slice()
{% endhighlight %}

i użyciu takich funkcji pomocniczych na zmodyfikowanej tablicy:

{% highlight typescript %}
items = copyArray(items.reverse())
items = copyArray2(items.sort())
{% endhighlight %}

### Własny mechanizm detekcji zmian w hooku ngDoCheck

Alternatywą jest zdefiniowanie własnego mechanizmu detekcji zmian, najlepiej w
hooku **ngDoCheck**, jednakże nie jest to zalecane. Dlaczego? Ten hook 
wykonywany jest *bardzo często*, co może mieć negatywny wpływ na wydajność
aplikacji. Jeśli naprawdę chcesz go użyć, pamiętaj, że ~~ostrzegałem~~ zawarty 
w nim kod powinien być naprawdę niewielki, niezbyt złożony.

Oto jak można rozwiązać omawiany problem za pomocą **ngDoCheck**:

{% highlight typescript %}
oldItems: Items[];

ngDoCheck() {
  let changeDetected = false;
  if (this.items.length !== this.oldItems.length) {
    changeDetected = true;
  } else {
    changeDetected = this.items.some((item, index) => item !== this.oldItems[index]);
  }

  if (changeDetected) {
    console.log("'items' property has changed!");
    this.oldItems = copyArray(this.items);
  }
}
{% endhighlight %}

Zauważcie, że sposób ten wymaga przechowywania starej wartości sprawdzanej
zmiennej - musimy ją pamiętać, aby móc wykryć jakąkolwiek zmianę 
(*this.oldItems* w kodzie powyżej). W tym przykładzie, zaimplementowałem prosty
mechanizm "płytkiego" porównania, wykrywający zmianę długości tablicy oraz
zmianę referencji dowolnego z elementów tablicy. Jednak nawet pisząc w ten 
sposób, musiałem użyć funkcji *copyArray* zdefiniowanej wcześniej w poście
po to, aby po wykryciu zmiany skopiować tablicę do zmiennej *this.oldItems*.
Jest to konieczne, aby wartości "items" i "oldItems" były niezależnie 
przechowywane w pamięci i aby ich porównywanie miało sens.

Jak zawsze, zachęcam do przeczytania bardziej szczegółowej, oficjalnej
[dokumentacji](https://angular.io/guide/lifecycle-hooks#defining-custom-change-detection){:target="_blank"},
pokazującej więcej przykładów i wyjaśniającej szczegóły działania hooka **ngDoCheck**.

## Podsumowanie

Należy pamiętać o tym, w jaki sposób Angular wykrywa zmiany wartości @Input() -
używa do tego najprostszego możliwego operatora ścisłej równości (===), co 
niesie czasami niespodziewane konsekwencje. Dzisiejszy post pokazał jak radzić
sobie z takimi przypadkami, aby móc wykrywać zmiany także w mutowanych 
tablicach i obiektach.
