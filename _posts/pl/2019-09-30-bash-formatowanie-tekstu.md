---
title: Bash – formatowanie wyświetlanego tekstu
date: 2019-09-30 20:00:00.000000000 +02:00
identifier: bash-formatting-text
tags:
- Bash
- DevOps
- Scripts
permalink: "/2019/09/bash-formatowanie-tekstu/"
excerpt: "Wiedza z nowego wpisu ożywi wiadomości generowane przez każdy skrypt!
  Już dziś naucz się jak ustawić kolor tekstu i tła oraz różne opcje
  formatowania. Zapraszam do lektury!"
---
Dziś pora na kolejny wpis z
<a href="/2019/09/bash-parsowanie-argumentow/">cyklu</a>
opisującego Bash, czyli jedną z najpopularniejszych powłok systemów uniksowych.
Tym razem opiszę ciekawostkę, to znaczy sposób, w jaki możemy dodać możliwość
generowania kolorowego i sformatowanego tekstu do konsoli. Zapraszam do lektury!

## W jaki sposób Bash interpretuje informację o kolorze tekstu?

Odpowiadają za to sekwencje specjalnych **znaków kontrolnych**, które zawierają
informację o kolorze tekstu. No, tak naprawdę to nie tylko o kolorze, ale też o
innych atrybutach, jak np. podkreślenie, kursywa, pogrubienie, widoczność czy
nawet możliwość migania! Jednak w jaki sposób podać interpreterowi sekwencję
znaków kontrolnych? Należy zacząć od **znaku ucieczki**, który w Bashu możemy
wyspecyfikować na trzy, równoważne sposoby:

- \e
- \033
- \x1B

Wszystkie one rozpoczną sekwencję znaków kontrolnych, jeśli podamy je na wejściu
polecenia **echo** z włączoną obsługą interpretowania takich sekwencji (opcja
-e):

<pre><code>echo -e "Jestem \e[31mczerwony\e[0m"</code></pre>

wyświetli na ekranie:

<pre style="background-color:black;color:white;">

  Jestem <span style="color:red;">czerwony</span>

</pre>

Zastąpienie w tym poleceniu znaku "\e" dowolnym z "\033" i "\x1B" (zapis
szesnastkowy) nie zmieni działania polecenia.

Należy zaznaczyć, że znaki kontrolne następują po znaku otwierającego nawiasu
kwadratowego "[", następnie mamy listę znaków kontrolnych, oddzielonych
średnikiem ";". W powyższym przykładzie występuje jedynie jeden znak kontrolny,
"31", oznaczający czerwony kolor tekstu, stąd brak średnika. Znak "m" kończy
sekwencję znaków kontrolnych, która od tego momentu zaczyna obowiązywać.
Konsekwencją tego jest fakt, że trzeba ją jawnie wyłączyć, resetując wyjście
skryptu. Służy do tego znak kontrolny "0", który musi być wstawiony jako
sekwencja kontrolna, a więc "\e[0m".

## Jakie kolory obsługuje Bash?

To zależy 🙂 Oczywiście od konkretnie używanego terminala i jego konfiguracji.
Podam wartości prawdziwe dla większości z będących obecnie w użyciu terminali,
dokładne dane o obsługiwanej ilości kolorów oraz opcjach formatujących można
znaleźć np.
<a
  href="https://misc.flogisoft.com/bash/tip_colors_and_formatting#terminals_compatibility"
  target="_blank">tutaj</a>.

Zacznijmy od tego, że ustawiać możemy kolor tekstu oraz (niezależnie!) kolor tła
tekstu. Dla koloru tekstu możliwości są następujące (16 kolorów podstawowych i
jeden specjalny – "domyślny kolor tekstu"):

|Kolor                             |Znak kontrolny    |Sekwencja włączająca    |
|----------------------------------|------------------|------------------------|
|Czarny                            |30                |\e[30m                  |
|Czerwony                          |31                |\e[31m                  |
|Zielony                           |32                |\e[32m                  |
|Żółty                             |33                |\e[33m                  |
|Niebieski                         |34                |\e[34m                  |
|Różowy  (magenta)                 |35                |\e[35m                  |
|Cyjan                             |36                |\e[36m                  |
|Specjalny  – domyślny kolor tekstu|39                |\e[39m                  |
|Jasny  szary                      |37                |\e[37m                  |
|Ciemny  szary                     |90                |\e[90m                  |
|Jasny  czerwony                   |91                |\e[91m                  |
|Jasny  zielony                    |92                |\e[92m                  |
|Jasny  żółty                      |93                |\e[93m                  |
|Jasny  niebieski                  |94                |\e[94m                  |
|Jasny  różowy (magenta)           |95                |\e[95m                  |
|Jasny  cyjan                      |96                |\e[96m                  |
|Biały                             |97                |\e[97m                  |

Z kolei dla kolorów tła mamy następujące możliwości (16 kolorów podstawowych i 1
specjalny – "domyślny kolor tła"), łatwo zapamiętać, że to dokładnie te same
wartości co kolor tekstu, ale +10:

|Kolor                             |Znak kontrolny    |Sekwencja włączająca    |
|----------------------------------|------------------|------------------------|
|Czarny                            |40                |\e[40m                  |
|Czerwony                          |41                |\e[41m                  |
|Zielony                           |42                |\e[42m                  |
|Żółty                             |43                |\e[43m                  |
|Niebieski                         |44                |\e[44m                  |
|Różowy  (magenta)                 |45                |\e[45m                  |
|Cyjan                             |46                |\e[46m                  |
|Specjalny  – domyślny kolor tekstu|49                |\e[49m                  |
|Jasny  szary                      |47                |\e[47m                  |
|Ciemny  szary                     |100               |\e[100m                 |
|Jasny  czerwony                   |101               |\e[101m                 |
|Jasny  zielony                    |102               |\e[102m                 |
|Jasny  żółty                      |103               |\e[103m                 |
|Jasny  niebieski                  |104               |\e[104m                 |
|Jasny  różowy (magenta)           |105               |\e[105m                 |
|Jasny  cyjan                      |106               |\e[106m                 |
|Biały                             |107               |\e[107m                 |

Pamiętajmy, że wszystkie z tych możliwości możemy wykorzystać za pomocą
polecenia **echo** z włączoną flagą -e. Dodatkowo, możemy włączać je
jednocześnie, co oznacza, że przykładowo:

<pre><code>echo -e "Jestem \e[43;31mczerwony na żółtym tle\e[0m"</code></pre>

wyświetli w terminalu:

<pre style="background-color:black;color:white;">

  Jestem <span style="color:red;background-color:yellow;">czerwony na żółtym tle</span>

</pre>

## Robi się ciekawie: dodatkowe opcje formatowania obsługiwane przez Bash!

Opcje te pozwalają na uzyskanie efektu "Wow!" wśród znajomych, choć należy
zaznaczyć, że ich przesadne użycie łatwo doprowadzi do zmniejszenia czytelności
niż jej poprawy 😉

Aby włączyć poszczególne opcje formatowania tekstu na wyjściu możemy użyć
następujących znaków kontrolnych:

|Opcja                                           |Znak kontrolny|Sekwencja włączająca|Przykład|
|------------------------------------------------|--------------|--------------------|--------|
|Pogrubienie tekstu (i rozjaśnienie jendocześnie)|1             |\e[1m               |Tekst   |
|Zacienienie                                     |2             |\e[2m               |Tekst   |
|Kursywa                                         |3             |\e[3m               |Tekst   |
|Podkreślenie                                    |4             |\e[4m               |Tekst   |
|Miganie                                         |5             |\e[5m               |<span id="blink">Tekst</span>   |
|Odwrócenie kolorów: tekstu i tła                |7             |\e[7m               |Tekst   |
|Niewidoczność (przydatne dla haseł)             |8             |\e[8m               |        |

Za pomocą tych samych znaków kontrolnych, ale z wartością powiększoną o **20**
możemy resetować odpowiednie opcje formatowania:

|Opcja                                       |Znak kontrolny|Sekwencja resetująca|
|--------------------------------------------|--------------|--------------------|
|Resetowanie pogrubiania tekstu              |21            |\e[21m              |
|Resetowanie zacienienia                     |22            |\e[22m              |
|Resetowanie kursywy                         |23            |\e[23m              |
|Resetowanie podkreślenia                    |24            |\e[24m              |
|Resetowanie migania                         |25            |\e[25m              |
|Resetowanie odwrócenia kolorów: tekstu i tła|27            |\e[27m              |
|Resetowanie niewidoczności                  |28            |\e[28m              |
|Resetowanie wszystkich opcji formatowania   |0             |\e[0m               |

Składając wszystkie wiadomości razem, możemy uzyskać super fancy efekt
czerwonego tekstu, migającego rytmicznie na żółtym tle, pogrubionego i
podkreślonego jednocześnie 🙂

<pre><code>echo -e "Jestem \e[1;3;4;5;43;31;3mczerwony, pogrubiony, zaciemniony, podkreślony i migam na żółtym tle :)\e[0m"</code></pre>

Powyższe polecenie przeniesie nas żywcem do lat '90, pamiętnych czasów
wspaniałych animacji na stronach internetowych, tym razem w wersji terminalowej:

<pre style="background-color:black;color:white;">

  Jestem <span id="fancy" style="color: red; background-color: yellow; text-decoration: underline; font-style: italic; font-weight: bold;">czerwony, pogrubiony, zaciemniony, podkreślony i migam na żółtym tle :)</span>

</pre>
<script>
  document.body.onload = function () {
    var hidden = false;
    setInterval(function(){
      var fancyElement = document.getElementById('fancy');
      var blinkElement = document.getElementById('blink');
      blinkElement.style.color = hidden ? 'white' : 'black';
      fancyElement.style.color = hidden ? 'red' : 'transparent';
      hidden = !hidden;
    }, 1000);
  }
</script>

## Pro Tip numer 1: Wyświetlenie wszystkich kombinacji kolorów i formatowania!

Możemy ułożyć prosty skrypt Bash, dzięki któremu w prosty sposób wyświetlimy
wszystkie możliwe kombinacje kolorów tekstu, tła i dodatkowych opcji
formatowania. Zadanie zrealizować można za pomocą trzech, zagnieżdżonych
pętli **for**:

{% highlight bash %}
for x in {0..8}; do
  for i in {30..37}; do
    for a in {40..47}; do
      echo -ne "\e[$x;$i;$a""m\\\\\\e[$x;$i;$a""m\e[0;37;40m "
    done
    echo
  done
done
{% endhighlight %}

Przedstawiony skrypt wyświetli wszystkie dostępne kombinacje kolorów i opcji
formatowania, a efekt jego działania powinien przypominać ten z obrazka poniżej:

![
  Efekt działania powyższego skryptu, kombinacje możliwych kolorów i opcji
  formatowania
]({{ site.baseurl }}/assets/img/2019-09-30/possibilities.png)
*Możliwości formatowania i kolorowania tekstu w terminalu, efekt działania powyższego skryptu*

## Niech stracę! Pro Tip numer 2: rozszerzenie możliwości skryptu o formatowanie tekstu

Przypiszmy odpowiednie sekwencje znaków kontrolnych do zmiennych o znaczących
nazwach:

{% highlight bash %}
GREEN='\e[32m'
RED='\e[31m'
BOLD='\e[1m'
NO_COLOR='\e[0m'
{% endhighlight %}

Teraz możemy zdefiniować kolorowe znaczniki, proste do użycia w interfejsie
skryptu:

{% highlight bash %}
PRINT_INFO="${BOLD}[INFO]${NO_COLOR} "
PRINT_INPUT="${BOLD}[INPUT]${NO_COLOR} "
PRINT_OK="${GREEN}${BOLD}[OK]${NO_COLOR} "
PRINT_ERROR="${RED}${BOLD}[ERROR]${NO_COLOR} "
{% endhighlight %}

Od teraz możemy używać odpowiednich znaczników w poleceniu echo:

{% highlight bash %}
echo -e "$PRINT_INFO Tekst"
echo -e "$PRINT_INPUT Tekst"
echo -e "$PRINT_OK Tekst"
echo -e "$PRINT_ERROR Tekst"
{% endhighlight %}

Co wyświetli w terminalu:

<pre style="font-weight: normal; color: white; background-color: black; margin: 20px 0 0 0; padding-top: 20px;">
  <span style="font-weight: bold; ">[INFO]</span> Informacja
</pre>

<pre style="font-weight: normal; color: white; background-color: black; margin: 0;">
  <span style="font-weight: bold; ">[INPUT]</span> Podaj wartość:
</pre>

<pre style="font-weight: normal; background-color: black; color: white; margin: 0;">
  <span style="font-weight: bold; color: green;">[OK]</span> Wszystko w porządku!
</pre>

<pre style="font-weight: normal; background-color: black;  color: white; margin: 0 0 20px; padding-bottom: 20px;">
  <span style="font-weight: bold; color: red;">[ERROR]</span> Wiadomość o błędzie
</pre>

Dla kompletności, możemy jeszcze użyć polecenia test, aby sprawdzić czy
zapisujemy wynik spryptu do konsoli czy do pliku. W tym drugim przypadku możemy
darować sobie kolory i formatowanie, ponieważ zostałyby one zapisane jako
znane nam sekwencje kontrolne "\e[31m", co zaciemniłoby wynik działania skryptu.
Z zapisywaniem do pliku możemy mieć do czynienia np. w przypadku logowania
działania skryptu:

{% highlight bash %}
if test -t 1; then
    # Wyjście standardowe (konsola), ustaw kolory
    GREEN='\e[32m'
    RED='\e[31m'
    BOLD='\e[1m'
    NO_COLOR='\e[0m'
else
    # Wyjście jest do pliku, nie potrzebujemy kolorów
    GREEN="";RED="";BOLD="";NOCOLOR=""
fi
{% endhighlight %}

## Bash, terminal i kolory – podsumowanie

Jak pokazałem, możliwości formatowania wyjścia, nawet z poziomu skryptu Bash’a
są całkiem spore. Uzależnione są one od programu, którego używamy jako terminal,
jednak te przedstawione w dzisiejszym wpisie są dość uniwersalne i powszechnie
dostępne. Teraz nic nie stoi na przeszkodzie, aby tworzyć nie tylko
funkcjonalne, ale też cieszące oko interfejsy do naszych skryptów 🙂 Dzięki za
przeczytanie i zapraszam do dyskusji w sekcji komentarzy!

<style>
  table:nth-of-type(1) tr td:nth-child(1) {
    background: black;
  }
  table:nth-of-type(1) tr:nth-of-type(1) td:nth-child(1) {
    background: white;
    color: black;
  }
  table:nth-of-type(1) tr:nth-of-type(2)  td:nth-child(1) { color: red; }
  table:nth-of-type(1) tr:nth-of-type(3)  td:nth-child(1) { color: green; }
  table:nth-of-type(1) tr:nth-of-type(4)  td:nth-child(1) { color: yellow; }
  table:nth-of-type(1) tr:nth-of-type(5)  td:nth-child(1) { color: blue; }
  table:nth-of-type(1) tr:nth-of-type(6)  td:nth-child(1) { color: magenta; }
  table:nth-of-type(1) tr:nth-of-type(7)  td:nth-child(1) { color: cyan; }
  table:nth-of-type(1) tr:nth-of-type(8)  td:nth-child(1) { color: white; }
  table:nth-of-type(1) tr:nth-of-type(9)  td:nth-child(1) { color: lightgrey; }
  table:nth-of-type(1) tr:nth-of-type(10) td:nth-child(1) { color: darkgrey; }
  table:nth-of-type(1) tr:nth-of-type(11) td:nth-child(1) { color: lightcoral; }
  table:nth-of-type(1) tr:nth-of-type(12) td:nth-child(1) { color: lightgreen; }
  table:nth-of-type(1) tr:nth-of-type(13) td:nth-child(1) { color: lightgoldenrodyellow; }
  table:nth-of-type(1) tr:nth-of-type(14) td:nth-child(1) { color: lightblue; }
  table:nth-of-type(1) tr:nth-of-type(15) td:nth-child(1) { color: lightpink; }
  table:nth-of-type(1) tr:nth-of-type(16) td:nth-child(1) { color: lightcyan; }
  table:nth-of-type(1) tr:nth-of-type(17) td:nth-child(1) { color: white; }

  table:nth-of-type(2) tr:nth-of-type(1)  td:nth-child(1) { background: black; color: white; }
  table:nth-of-type(2) tr:nth-of-type(2)  td:nth-child(1) { background: red; color: white; }
  table:nth-of-type(2) tr:nth-of-type(3)  td:nth-child(1) { background: green; color: white; }
  table:nth-of-type(2) tr:nth-of-type(4)  td:nth-child(1) { background: yellow; }
  table:nth-of-type(2) tr:nth-of-type(5)  td:nth-child(1) { background: blue; color: white; }
  table:nth-of-type(2) tr:nth-of-type(6)  td:nth-child(1) { background: magenta; color: white; }
  table:nth-of-type(2) tr:nth-of-type(7)  td:nth-child(1) { background: cyan; }
  table:nth-of-type(2) tr:nth-of-type(8)  td:nth-child(1) { background: black; color: white; }
  table:nth-of-type(2) tr:nth-of-type(9)  td:nth-child(1) { background: lightgrey; }
  table:nth-of-type(2) tr:nth-of-type(10) td:nth-child(1) { background: darkgrey; }
  table:nth-of-type(2) tr:nth-of-type(11) td:nth-child(1) { background: lightcoral; }
  table:nth-of-type(2) tr:nth-of-type(12) td:nth-child(1) { background: lightgreen; }
  table:nth-of-type(2) tr:nth-of-type(13) td:nth-child(1) { background: lightgoldenrodyellow; }
  table:nth-of-type(2) tr:nth-of-type(14) td:nth-child(1) { background: lightblue; }
  table:nth-of-type(2) tr:nth-of-type(15) td:nth-child(1) { background: lightpink; }
  table:nth-of-type(2) tr:nth-of-type(16) td:nth-child(1) { background: lightcyan; }
  table:nth-of-type(2) tr:nth-of-type(17) td:nth-child(1) { background: white; }

  table:nth-of-type(3) tr                td:nth-child(4) { background: black; color: white; }
  table:nth-of-type(3) tr:nth-of-type(1) td:nth-child(4) { font-weight: bold; }
  table:nth-of-type(3) tr:nth-of-type(2) td:nth-child(4) { color: lightgrey; }
  table:nth-of-type(3) tr:nth-of-type(3) td:nth-child(4) { font-style: italic; }
  table:nth-of-type(3) tr:nth-of-type(4) td:nth-child(4) { text-decoration: underline; }
  table:nth-of-type(3) tr:nth-of-type(6) td:nth-child(4) { background: white; color: black; }
</style>
