---
title: Dekorator @proxy w Pythonie
date: 2020-06-06 20:00:00.000000000 +02:00
identifier: python-proxy
categories:
- python
tags:
- flask
- requests
- proxy
- decorator
excerpt:
  Prosty dekorator @proxy przekazujący żądania HTTP dalej do docelowego serwisu.
  Napisany z pomocą Flaska i biblioteki Requests. Zapraszam!
---
Czasami Twoja aplikacja ma służyć głównie jako proxy do jakiś innych, 
zewnętrznych serwisów. Możesz spotkać się z taką sytuacją szczególnie w 
przypadku stosowania wzorca architektonicznego BFF (ang. *backend for frontend*).
Potrzeba reużywania prostego schematu działania: "wyślij żądanie do innego 
serwisu - zwróć odpowiedź" może skłonić Cię do napisania dekoratora @proxy -
jeśli masz to szczęście i piszesz w Pythonie :)

**TL;DR**
Możesz zobaczyć ostateczną implementację 
[tutaj](https://gist.github.com/Nitrooos/161ccf1ac05d2e0e253a3fe0ff29d3f4){:target="_blank"} -
dekorator *proxy_4* jest moją propozycją opisywaną krok po kroku w tym artykule.

## Definicja dekoratora @proxy

Jeśli chcemy stworzyć coś użytecznego, potrzebujemy zdefiniować najpierw 
wymagania, tzn. czego oczekujemy od dekoratora @proxy. Powinien on:

* wysyłać żądania pod tę samą ścieżkę do docelowego serwisu, zachowując 
odpowiednią metodę protokołu HTTP
* przekazywać otrzymane nagłówki oraz zwracać jako odpowiedź z serwisu nie 
tylko jej ciało (*body*), ale też kod odpowiedzi oraz nagłówki
* w przypadku żądań zawierających dane (np. POST), również przekazywać je do
docelowego serwisu

Przydatym dodatkiem byłaby możliwość przetwarzania odpowiedzi z serwisu po jej
otrzymaniu, a przed przekazaniem do klienta. Aby to osiągnąć, dekorator @proxy
będzie zwracał funkcję z jednym parametrem. Będzie nim właśnie odpowiedź z
docelowego serwisu.

Ponadto, czasami możemy chcieć zmienić ścieżkę żądania tak, że jeśli w 
aplikacji będzie ona obsługiwana jako */foo*, żądanie do docelowego serwisu
powinno zostać wykonane ze ścieżką */bar*. Taka możliwość także będzie dostępna.

## Potrzebne zależności

Zdecydowałem się użyć 2 bibliotek, dobrze znanych w świecie Pythona:

* [Flask](https://flask.palletsprojects.com){:target="_blank"}, dla 
zdefiniowania ścieżek w aplikacji
* [Requests](https://requests.readthedocs.io/en/master/){:target="_blank"} do
wykonywania żądań do zewnętrznych serwisów

Obie biblioteki są powszechnie znane i sprawdzone, ale oczywiście możesz użyć 
każdej innej paczki do zdefiniowania API (np. Falcon albo django-rest-framework)
i do wysyłania żądań HTTP (np. urllib3).

## Programistyczne mięcho!

### Testowy widok

Dla łatwego testowania implementacji, zdefiniowałem 1 testowy widok:

{% highlight python %}
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE'])
def test_view(request):
  pass
{% endhighlight %}

Ścieżka żądania będzie dostępna w dekoratorze @proxy jako parametr nazwany
"path".

### Rozwiązanie najprostsze @proxy_1: zachowuj odpowiednią metodę HTTP

Zacznijmy z czymś prostym, tutaj respektujemy tylko zachowanie odpowiedniej
ścieżki i metody HTTP przy wysyłaniu żądania do innego serwisu:

{% highlight python %}
import requests

from flask import request

DESTINATION_URL = 'http://any-external-service/'

def proxy_1(view_function):
  def wrapper(path):
    request_method = getattr(requests, request.method.lower())
    return request_method(f'{DESTINATION_URL}/{path}').content

  return wrapper
{% endhighlight %}

Czytamy użytą metodę HTTP, wysyłamy żądanie do serwisu i zwracamy treść 
odpowiedzi.

Ale naprawdę możemy robić to lepiej! Docelowy serwis powinien wiedzieć także o
nagłówkach i danych żądania a dekorator przekazywać nagłówki odpowiedzi do 
klienta.

### Próba druga, czyli przekazuj ciało i nagłówki żądania: @proxy_2

Przekazywanie nagłówków oryginalnego żądania oznacza dodanie nazwanego 
argumentu *headers* do funkcji wykonującej żądanie do serwisu docelowego. 
Funkcja opakowana w dekorator może także zwracać krotkę (ciało odpowiedzi, kod
odpowiedzi, nagłówki), a więc zawierać więcej informacji o odpowiedzi z 
serwisu.

Aby dekorator przekazywał także dane żądania (*ang. payload*), należy podać
nazwany argument *data* do konstruowanego żądania. We Flasku, dane te są
dostępne w *request.form*, jeśli pochodzą z formularza na stronie, w 
*request.json*, jeśli typem żądania jest *application/json* albo w 
*request.data* jeśli typ MIME nie został rozpoznany.  

Zaktualizowany kod wygląda następująco:

{% highlight python %}
def proxy_2(view_function):
  def wrapper(path):
    request_method = getattr(requests, request.method.lower())
    response = request_method(f'{DESTINATION_URL}/{path}',
      data=request.form or request.json or request.data,
      headers=request.headers,
    )
    return response.content, response.status_code, response.headers.items()

  return wrapper
{% endhighlight %}

Czy rzeczywiście działa? Cóż... trochę tak, ale jeśli zaczniesz z niego 
korzystać, szybko zauważysz, że w niektórych sytuacjach zachowuje się dziwnie.
Będzie tak dlatego, że nie można po prostu przekazywać **WSZYSTKICH** nagłówków
odpowiedzi z serwisu, niektóre z nich muszą zostać wycięte. Dlaczego?

#### Nagłówki typu end-to-end oraz hop-by-hop

Nagłówki odpowiedzi HTTP można podzielić na 2 kategorie:

* end-to-end, czyli takie, które *muszą* zostać przetransmitowane do finalnego
odbiorcy danej odpowiedzi
* hop-by-hop, czyli takie, które mają sens tylko dla pojedynczego połączenia
warstwy transportowej i dlatego *nie mogą* być przekazywane dalej przez serwery
proxy

Widzimy więc wyraźnie, że nagłówki typu hop-by-hop muszą zostać usunięte przed
zwróceniem odpowiedzi do finalnego odbiorcy. Które z nagłówków należą do tej
kategorii? Są to: "Connection", "Keep-Alive", "Proxy-Authenticate",
"Proxy-Authorization", "TE", "Trailer", "Transfer-Encoding" oraz "Upgrade".
Możesz poczytać więcej na ich temat na przykład 
[tutaj](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers){:target="_blank"}.
Większość problemów powodowana jest przez nagłówek "Transfer-Encoding", 
definiujący kodowanie danych zawartych w żądaniu - ta informacja jest prawdziwa
tylko w ramach połączenia "nasza aplikacja w Pythonie -> zewnętrzny serwis",
ale nie dotyczy klienta!

Poprawmy implementację poprzez usuwanie takich nagłówków:

{% highlight python %}
def proxy_2_1(view_function):
  def wrapper(path):
    request_method = getattr(requests, request.method.lower())
    response = clean_hop_by_hop_headers(
      request_method(f'{DESTINATION_URL}/{path}',
        data=request.form or request.json or request.data,
        headers=request.headers,
      )
    )
    return response.content, response.status_code, response.headers.items()

  return wrapper

def clean_hop_by_hop_headers(response):
  hop_by_hop_headers = [
    'Connection', 'Keep-Alive', 'TE', 'Trailer', 'Transfer-Encoding', 'Upgrade',
    'Proxy-Authorization', 'Proxy-Authenticate'
  ]
  for header in hop_by_hop_headers:
    if header in response.headers:
      del response.headers[header]
  return response
{% endhighlight %}

I to wszystko! *clean_hop_by_hop_headers* wytnie wszystkie nagłówki typu 
hop-by-hop z odpowiedzi przed przekazaniem jej do klienta.

### Przetwarzanie otrzymanej odpowiedzi: @proxy_3

Innym usprawnieniem, które możemy zaimplementować jest podanie odpowiedzi z
docelowego serwisu do dekorowanej funkcji jako argument. Umożliwia to 
przetwarzanie jej przed zwróceniem do finalnego odbiorcy. Oczywiście w 
przypadku, gdy nie chcemy robić nic z tą odpowiedzią, możemy po prostu 
pozostawić *pass* jako ciało dekorowanej fukcji, wówczas klient otrzyma 
odpowiedź z serwisu bez żadnych modyfikacji po drodze.

Aby wprowadzić tę zmianę, należy przeorganizować dekorator:

{% highlight python %}
def proxy_3(view_function):
  def wrapper(path, *args, **kwargs):
    request_method = getattr(requests, request.method.lower())
    raw_response = clean_hop_by_hop_headers(
      request_method(f'{DESTINATION_URL}/{path}',
        data=request.form or request.json or request.data,
        headers=request.headers,
      ),
    )
    processed_response = view_function(raw_response)
    final_response = (
      processed_response if isinstance(processed_response, Response)
      else raw_response
    )
    return requests_to_flask_response(final_response)

  return wrapper

def requests_to_flask_response(response):
  return response.content, response.status_code, response.headers.items()
{% endhighlight %}

Wykonujemy dekorowaną funkcję (nazywaną tu *view_function*) i sprawdzamy typ
jej wartości. Jeśli jest to typ *Response* z Flaska, wtedy wiemy, że użytkownik
podał jawnie co powinno zostać zwrócone do klienta, dlatego po prostu zwracamy
ten obiekt. Jeśli nie, wtedy przyjmujemy, że użytkownik nie korzystał z 
możliwości przetwarzania odpowiedzi i zwracamy dokładnie to, co otrzymaliśmy z
docelowego serwisu, bez modyfikacji (wyłączając oczywiście wycięcie 
odpowiednich nagłówków typu hop-by-hop).

### Zmiana docelowej ścieżki żądania: @proxy_4

Ostatnią rzeczą, którą poprawimy w omawianym dekoratorze jest możliwość zmiany
ścieżki żądania przed wykonaniem go do docelowego serwisu. Tak więc jeśli
żądanie wskazuje ścieżkę */foo*, będzie mogło być przekazane do docelowego
serwisu pod ścieżkę np. */bar*. Implementacja polega na dodaniu parametru
*proxy_path* do dekoratora. Zauważcie, że wymaga to zagnieżdżenia 3 funkcji,
jedna wewnątrz drugiej:

{% highlight python %}
def proxy_4(proxy_path=''):
  def real_proxy(view_function):
    def wrapper(path):
      final_path = proxy_path if proxy_path != '' else path
      request_method = getattr(requests, request.method.lower())
      raw_response = clean_hop_by_hop_headers(
        request_method(f'{DESTINATION_URL}/{final_path}',
          data=request.form or request.json or request.data,
          headers=request.headers,
        ),
      )
      ...

    return wrapper
  return real_proxy
{% endhighlight %}

Od teraz użytkownik może (ale nie musi!) zmienić ścieżkę żądania w trakcie 
przetwarzania (przypisanie do *final_path*). Dopiero wtedy żądanie jest 
wysyłane do docelowego serwisu.

I to wszystko! Prosta i elegancka implementacja dekoratora @proxy dla Twojej
aplikacji w Pythonie. Jak dokładnie możemy użyć tego dekoratora? Na przykład
tak:

{% highlight python %}
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE'])
@proxy_4()
def test_view(proxied_response):
  pass
{% endhighlight %}

jeśli jedyne czego chcemy to przekazać żądanie do docelowego serwisu i zwrócić
odpowiedź (zauważcie puste nawiasy - proxy_4(). Finalna implementacja wymaga
użycia ich nawet jeśli nie chcemy podawać parametru *proxy_path*, ponieważ
dekorator zawsze zakłada istnienie argumentu o tej nazwie. Ma on po prostu 
wartość pustego łańcucha jeśli użytkownik jawnie jej nie wskaże).

Możesz także wskazać nową ścieżkę dla żądania i modyfikować odpowiedź z serwisu
przed zwróceniem jej do klienta:

{% highlight python %}
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE'])
@proxy_4(proxy_path='/api/v2/')
def test_view(proxied_response):
  response = make_response({
    'status_message': proxied_response.json()['status']
  })
  response.status_code = 200
  response.headers = { **proxied_response.headers, 'x-foo': 'bar' }.items()

  return response
{% endhighlight %}

Tym razem żądanie będzie przekazywane pod ścieżkę /api/v2/. Jak widać, możemy
modyfikować zawartość, status i nagłówki odpowiedzi. Dekorator @proxy jest
użyteczny, gdy musimy przetworzyć w jakiś sposób odpowiedź z serwisu.

Finalna wersja dekoratora jest dostępna
[tutaj](https://gist.github.com/Nitrooos/161ccf1ac05d2e0e253a3fe0ff29d3f4){:target="_blank"},
*@proxy_4* jest implementacją opisywaną w tym artykule.

## Podsumowanie

Python umożliwia zdefiniowanie w prosty sposób dekoratora służącego jako proxy
do zewnętrznych serwisów webowych. Dzięki niemu unikamy niepotrzbnej duplikacji
kodu, w szczególności dotyczącego przygotowania żądania i zwracania odpowiedzi
z powrotem do klienta.

Oczywiście opisana tutaj implementacja jest tylko punktem startowym, jeśli
chcesz stworzyć bardziej uniwersalne rozwiązanie problemu przekazywania żądań
HTTP do kolejnych serwisów. Niemniej jednak, zawsze można z niego skorzystać i
dostosować tylko do bieżących potrzeb. Dzięki za przeczytanie!
