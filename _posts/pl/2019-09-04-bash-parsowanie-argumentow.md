---
title: Bash – parsowanie argumentów
date: 2019-09-04 20:00:00.000000000 +02:00
identifier: bash-parsing-arguments
tags:
- Bash
- DevOps
- Scripts
permalink: "/2019/09/bash-parsowanie-argumentow/"
excerpt: "Zmęczony wklepywaniem w kółko tych samych komend podczas pracy z
  projektem? Dzisiejszy post pokazuje jak można uprościć taką pracę za pomocą
  dedykowanego skryptu. Zapraszam!"
---
Praktycznie każdy programista spotyka się od czasu do czasu z prostymi
skryptami, automatyzującymi powtarzające się czynności. Często spotyka się je
także w roli interfejsów ułatwiających zarządzanie projektem. To, co jest ich
główną siłą to możliwość wykonania za pomocą jednej komendy konkretnego zadania,
które do tej pory było realizowane poprzez przeklejanie całego stosu poleceń.
Bash, czyli popularna powłoka systemów uniksowych umożliwia tworzenie takich
skryptów.

W dzisiejszym wpisie pokażę jak w prosty sposób parsować argumenty przekazywane
do skryptu interpretowanego przez Bash.

## Na początku jedno założenie 🙂

Zakładamy, że chcemy parsować argumenty przekazane w postaci:
<pre><code>--nazwa-argumentu wartość</code></pre>
bądź też w formie skróconej:
<pre><code>-A wartość</code></pre>

Przedstawiony skrypt obsługuje jednocześnie obie formy. Piszę o tym dlatego, że
istnieją inne, alternatywne składnie dla przekazania argumentów, np.
–nazwa-argumentu=wartość. Takie, inne składnie nie będą obsługiwane.

## Kontekst działania skryptu

Załóżmy, że tworzony skrypt służy do wgrywania zmian w aplikacji na serwer. W
związku z tym nazwijmy go deploy.sh i nadajmy mu kilka sensownych parametrów,
które można podać:

<pre><code>  -E, --environment   Środowisko, na które skrypt dokona wgrania zmian (np. testowe, przedprodukcyjne, produkcyjne itp)
  -B, --branch        Nazwa gałęzi, z której zostaną pobrane zmiany do wgrania
  --no-cache          Jeśli podana, zbuduj nową wersję aplikacji bez użycia jakiegokolwiek cache'a
                      (może mieć zastosowanie np. dla budowania nowego, świeżego obrazu Dockera z aplikacją, bez użycia cache'a)
  -h, --help          Wyświetl menu pomocy</code></pre>

Podane flagi są przykładowe, starające się odzwierciedlić typową sytuację
tworzenia skryptu wgrywającego zmiany na serwer.

## Bashowe mięcho!

{% highlight bash %}
STATUS_OK=0
DEPLOY_NO_CACHE="false"
INFO_DEPLOY_HELP=$(cat <<-EOF
  -E, --environment   Środowisko, na które skrypt dokona wgrania zmian (np. testowe, przedprodukcyjne, produkcyjne itp)
  -B, --branch        Nazwa gałęzi, z której zostaną pobrane zmiany do wgrania
  --no-cache          Jeśli podana, zbuduj nową wersję aplikacji bez użycia jakiegokolwiek cache'a
                      (może mieć zastosowanie np. dla budowania nowego, świeżego obrazu Dockera z aplikacją, bez użycia cache'a)
  -h, --help          Wyświetl menu pomocy
EOF
)

...

_skip=0
for opt in $@; do
    [ $_skip -ne 0 ] && { ((_skip--)); shift; continue; }
    case $opt in
        -E|--env)
            DEPLOY_ENV=$2
            _skip=1
            shift ;;
        -B|--branch)
            DEPLOY_BRANCH_NAME=$2
            _skip=1
            shift ;;
        --no-cache)
            DEPLOY_NO_CACHE="true"
            shift ;;
        -h|--help)
            echo -e "$INFO_DEPLOY_HELP"
            exit $STATUS_OK
            ;;
        *)
            shift
    esac
done
{% endhighlight %}

Powyższy kod dokonuje prostego parsowania argumentów dla skryptu deploy.sh.
W jaki sposób dokładnie działa? Pora na objaśnienie krok po kroku!

### Bash – zmienna $@

Zmienna ta zawiera wszystkie argumenty, które zostały przekazane do skryptu.
Dlatego też iterujemy po niej w pętli for – aby sprawdzić wszystkie argumenty po
kolei.

### Pomocnicza zmienna $_skip

Służy ona do pominięcia określonej liczby argumentów na początku każdej iteracji
pętli for. Chodzi o to, że sprawdzamy tylko nazwy argumentów (i to je
dopasowujemy w instrukcji case), a pomijamy wartości tych argumentów, używając
ich tylko wewnątrz instrukcji case. Do pominięcia argumentu służy instrukcja
"shift". Każdy z argumentów może posiadać inną liczbę wartości (np. dla
argumentów -E/–env i -B/–branch jest to jedna wartość, natomiast argument
–no-cache nie przyjmuje wartości). Ustawiając wartość zmiennej $_skip na 1
powodujemy pominięcie wartości tego argumentu na początku kolejnej iteracji
pętli for.

### Bash – instrukcja case

Dopasowuje ona nazwy argumentów podanych do skryptu. Zapis -E|–env oznacza, że
dopasowanie zajdzie jeśli rozpoznany zostanie argument o nazwie "-E" lub "–env".
Instrukcja "shift" powoduje przejście do kolejnego argumentu, a ustawienie
zmiennej _skip wartością > 0 pominięcie listy wartości aktualnego argumentu.
Zauważmy, że w podanym fragmencie kodu nie wywołujemy żadnych akcji, a jedynie
inicjalizujemy zmienne DEPLOY_ENV, DEPLOY_BRANCH i DEPLOY_NO_CACHE odpowiednimi
wartościami. W przypadku DEPLOY_NO_CACHE jest to po prostu wartość tekstowa
"true", natomiast dla dwóch pozostałych zmiennych jest to wartość $2. Oznacza to
wartość argumentu, podaną zaraz za jego nazwą (dostępną nota bene pod wartością
$1).

### Dopasowanie *)

Jeśli nazwa badanego argumentu nie pasuje do żadnej z możliwych opcji,
ignorujemy go. "shift" działa tutaj jak "przesunięcie" w liście badanych
argumentów, przechodząc do kolejnego.

## Pro tip: bash i skrót Ctrl+R

Czasami tworzenie dedykowanego skryptu dla powtarzających się czynności może być
zbędne. Przykładem są często używane, pojedyncze komendy, posiadające jednak
sporą liczbę argumentów. Męczące przy wielokrotnym, ręcznym wklepywaniu do
terminala, mogą być odszukane w historii powłoki i wywołane w bardzo prosty
sposób. Wystarczy wcisnąć Ctrl + R na klawiaturze a następnie wpisać fragment
poszukiwanego polecenia.

Przykładowo pracujemy nad testami modelu User, zdefiniowanymi w klasie
UserModelTest w paczce auth.test.test_models. Dla frameworku Django przykładowe
uruchomienie tych testów może wyglądać następująco:

<pre><code>$ python manage.py test --settings=myawesomeapp.test_settings auth.test.test_models.UserModelTest</code></pre>

Jednak ciągle je poprawiamy, dopisujemy nowe, sprawdzamy czy przechodzą. Nie
chcemy ciągle klepać tego polecenia ani wyszukiwać go strzałką w górę na liście
ostatnich komend. Wystarczy Ctrl + R, następnie „UserModel”… i prawdopodobnie
bash znajdzie już odpowiednie polecenie w pamięci i wystarczy wcisnąć Enter 🙂

## Podsumowanie

W dzisiejszym wpisie pokazałem jak w prosty sposób dokonać parsowania argumentów
przekazywanych do skryptu w Bashu. Po przeprowadzonym parsowaniu odpowiednie
wartości znajdują się w zmiennych o znanych nam nazwach (w przykładzie były to
DEPLOY_ENV, DEPLOY_BRANCH_NAME i DEPLOY_NO_CACHE). Dzięki temu możemy rozwinąć
skrypt o wykonywanie konkretnych zadań, mając w nich już zdefiniowane konkretne
wartości. Wpis ten jest pierwszym z mini-cyklu dotyczącego basha i tworzenia
takich skryptów zarządzających projektem. Mam nadzieję, że się spodoba!
