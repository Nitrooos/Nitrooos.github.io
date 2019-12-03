---
title: Ikony w aplikacji internetowej - bitmapy, plik ikon czy SVG?
date: 2019-05-31 20:00:54.000000000 +02:00
categories:
- Frontend
- Na dłużej
- Polski
tags:
- bitmapy
- ikony
- svg
permalink: "/2019/05/ikony-w-aplikacji-internetowej/"
excerpt: 'Tym razem na warsztat bierzemy temat ikon w aplikacji internetowej: powinniśmy
  używać bitmap, plików SVG czy może dedykowanego pliku fontu z ikonami? Zapraszam
  na analizę dostępnych możliwości!'
---
<p>Niemal w każdej aplikacji internetowej prędzej czy później zachodzi potrzeba dodania prostych ikon. Stanowią one prostą, graficzną reprezentację akcji, które użytkownik może wykonać. I tak, widząc ołówek od razu wiemy, że w tym miejscu możemy coś edytować, a najeżdżając kursorem na znak zapytania spodziewamy się pojawienia "dymku" z dodatkowym opisem. Jak więc widzimy, ikony w aplikacji internetowej pełnią ważną funkcję informacyjną, ułatwiającą zrozumienie działania strony. W jaki sposób jednak przechowywać&nbsp;je w sposób efektywny, tak, były w łatwy sposób konfigurowalne i reużywalne? Zapraszam!</p>
<h2>Wiele dostępnych opcji</h2>
<p>Okazuje się, że ten sam efekt na stronie możemy osiągnąć na wiele sposobów. Spośród dostępnych możemy wymienić między innymi:</p>
<ol>
  <li>Każda ikona jako osobny obrazek (bitmapa)</li>
  <li>Plik bitmapy zawierający wszystkie ikony używane w aplikacji internetowej (ang. <em>spritesheet</em>)</li>
  <li>Plik z fontem, zawierający ikony jako pojedyncze znaki (ang. <em>glyphs</em>)</li>
  <li>Plik formatu SVG, zawierający definicje używanych ikon</li>
</ol>
<p>Każda z nich (może poza pierwszą ;) posiada swoje wady i zalety, które postaram się opisać w tym artykule.</p>
<h2>Ikony jako osobne pliki graficzne</h2>
<p>Jest to zdecydowanie najprostsza z dostępnych opcji, stosowana przez mniej doświadczonych programistów frontendowych. W pułapkę tworzenia nowego pliku dla każdej kolejnej ikony można wpaść bardzo łatwo. Wystarczy, że programista nie będzie świadomy dalszego kierunku rozwoju aplikacji. Może przez to nie spodziewać się, że będzie ich z czasem potrzebnych coraz więcej. Gdy takich ikon aplikacja ładuje niewiele to oczywiście nie jest problem, ale przy większej ilości pojawia się problem:</p>
<h3>WADA: Niepotrzebne zwiększenie ilości żądań do serwera</h3>
<p>Każde z nich (prawdopodobnie) zakończy się szybko (ikony nie powinny mieć&nbsp;zbyt dużego rozmiaru), jednak ładując stronę zawierającą 50 ikon, wykonujemy do serwera o 49 żądań za dużo ;) Jak pokażę w dalszej części, istnieją metody pozwalające na załadowanie ich jednym żądaniem HTTP. Mając dodatkowo na uwadze ograniczenie przeglądarek na ilość jednocześnie otwartych połączeń sieciowych dla danej domeny (na ten temat można poczytać&nbsp;np. <a href="https://docs.pushtechnology.com/cloud/latest/manual/html/designguide/solution/support/connection_limitations.html" target="_blank">tutaj</a>), jest to poważna wada. Nowoczesne przeglądarki posiadają w większości limit 6 otwartych połączeń per domena. Tak więc jednocześnie z naszego serwera pobieranych może być maksymalnie 6 plików (w tym ikon), co może wydłużać ładowanie strony.</p>
<h3>WADA: Brak możliwości konfiguracji ikony</h3>
<p> Dopóki dana ikona wykorzystywana jest tylko w jednym miejscu w aplikacji, to nie problem. Co jednak jeśli ikonę np. kalendarza będziemy chcieli wyświetlić w czterech miejscach: w wersji niebieskiej małej, niebieskiej dużej, czerwonej małej i czerwonej dużej? Konieczność <em>lekkiej</em> modyfikacji pojedynczej ikony w tych dwóch parametrach zakończy się prawdopodobnie umieszczeniem w katalogu zasobu w 4 wersjach. A taki widok nie jest niczym przyjemnym:</p>
<pre>$ ls -l
total 4
-rw-r--r-- 1 user group  520 May 20  2019 calendar.png
-rw-r--r-- 1 user group  520 May 20  2019 calendar_big.png
-rw-r--r-- 1 user group  520 May 20  2019 calendar_red.png
-rw-r--r-- 1 user group  520 May 20  2019 calendar_red_big.png</pre>
<h3>WADA: Utrata jakości po przeskalowaniu</h3>
<p>Oczywistość, ale należy o tym pamiętać - dla prostych ikon warto rozważyć stosowanie grafiki wektorowej w formacie SVG, dzięki czemu unikniemy problemów takich jak poniżej.</p>

![
  Ikona ołówka w normalnym rozmiarze i powiększona
]({{ site.baseurl }}/assets/img/2019-05-31/pixelized.png)
*Utrata jakości spowodowana skalowaniem grafiki rastrowej<br/>(ikona ołówka pobrana z <a href="https://www.stockio.com/free-icon/maria-pencil" target="_blank">https://www.stockio.com/free-icon/maria-pencil</a>)*

<h2>Ikony w aplikacji internetowej w jednym pliku</h2>
<p>Jest to krok naprzód względem metody pierwszej, ponieważ rzeczywiście wszystkie ikony mogą być załadowane za pomocą jednego żądania HTTP. Wciąż jednak pozostają 2 kolejne wady wymienione powyżej: brak możliwości konfiguracji ikon oraz brak możliwości skalowania bez utraty jakości. Dla kronikarskiego obowiązku odnotowuję jednak, że taka metoda była również niegdyś stosowana. Odpowiednie pliki, będące zbiorami ikon nadal możemy znaleźć w Internecie, np. taki:</p>

![
  Spritesheet jako bitmapa z ikonami
]({{ site.baseurl }}/assets/img/2019-05-31/icon_spritesheet.png)

<p>I tutaj jest miejsce na ciekawostkę, ponieważ interesujący jest sposób korzystania z takiego pliku. Wymagane jest zdefiniowanie odpowiedniego kodu CSS "wycinającego" potrzebny fragment z obrazka. I tak, chcąc korzystać z pliku "/assets/spritesheet.png" jako pliku ikon, definiujemy napierw klasę oznaczającą ikonę:</p>
{% highlight css %}
.icon {
  background: url('/assets/spritesheet.png') no-repeat;
  height: 32px;
  width: 32px;
}
{% endhighlight %}
<p>A następnie klasy definiujące poszczególne ikony:</p>
{% highlight css %}
.icon-question {
  background-position: 0 0;
}

.icon-info {
  background-position: -32px 0;
}

...

.icon-github {
  background-position: 0 -160px;
}

...

.icon-internet-explorer {
  background-position: -96px -192px;
}
{% endhighlight %}
<p>I tak dalej, definiując kolejne przesunięcia dla poszczególnych ikon. Istnieją nawet strony stworzone specjalnie w celu wyznaczania odpowiednich przesunięć dla ikon, np. <a href="http://www.spritecow.com/" target="_blank">http://www.spritecow.com/</a>.</p>
<h2>Osadzenie ikon w pliku fontu</h2>
<p>Szukając lepszych alternatyw natrafimy na pierwszą, naprawdę sensowną: osadzenie ikon jako pojedynczych znaków (ang. <em>glyph</em>) w pliku fontu. Ikony są osadzone w jednym pliku, dodatkowo w postaci bezstratnej. Nie jest też zapisywana informacja o kolorze znaku. Oznacza to, że możemy dowolnie zmieniać&nbsp;rozmiar ikoy bez straty jakości oraz ustawiać jej dowolny kolor. O to chodziło! </p>
<p>Techniki tej używa popularny projekt <a href="https://fontawesome.com" target="_blank">Font Awesome</a>, zawierający ponad 1,5 tysiąca darmowych ikon w darmowej wersji. Jest to więc świetna opcja, kiedy nie zależy nam na wykorzystaniu ikon dostarczonych przez klienta, ale mamy pewną dowolność w ich dobieraniu. W takim wypadku naprawdę jest to opcja godna rozważenia.</p>
<h3>Gdzie kryje się haczyk?</h3>
<p>Nic nie jest idealne, a ta technika nie jest wyjątkiem. Edycja plików z fontami nie jest prosta, należy do tego celu użyć odpowiedniego programu. Z dostępnych opcji mamy płatny Adobe Illustrator, ale także darmowy <a href="https://fontforge.github.io" target="_blank">Font Forge</a>, z którego korzystam. Ciągle jednak, edycja fontu wymaga nauki kolejnego narzędzia oraz jest <strong>w ogólności czasochłonna</strong>.</p>
<p>Niestety, dodatkowo do tego dochodzi problem obsługi formatów fontów przez przeglądarki. Jeśli mamy obsługiwać ostatnie wersje Chrome, Firefox, Safari i IE11 (co wciąż jest częstą sytuacją), to okaże się, że będziemy musieli eksportować plik do <strong>kilku formatów</strong>. I tak przy każdej zmianie, wszystko po to, aby nasze ikony były wyświetlane we wszystkich wspieranych przeglądarkach. Najczęściej będą to formaty TTF, WOFF2 i EOT (dla Internet Explorera).</p>
<p>Jeśli i to Was nie zraża, to ciągle pozostaje jeden problem, choć i z nim można sobie poradzić. Jest to zachowanie przeglądarki, która może wyświetlać&nbsp;już tekst na stronie, korzystając z fontu systemowego podczas gdy nasz plik jest pobierany z serwera. Prowadzi to do sytuacji, w której przez krótki czas w miejscach, w których pojawią się później nasze ikony, <strong>wyświetlane będą "krzaki"</strong>. Dlaczego tak się dzieje? Poszczególne ikony są często definiowane na pozycjach przeznaczonych do użytku własnego. Osobiście spotykałem się z wykorzystaniem obszaru od pozycji 0x800. W naszym pliku znajdują się tam pożądane ikony, ale font systemowy może mieć zdefiniowane tam dowolne znaki. Zachowaniem tym można jednak sterować za pomocą właściwości CSS o nazwie font-display.</p>
<h3>Jak korzystać z ikon zdefiniowanych w pliku fontu?</h3>
<p>Załóżmy, że zdefiniowaliśmy ikony na pozycjach od 0x800 do 0x80F. W takim wypadku należy zacząć od zdefiniowania fontu:</p>
{% highlight css %}
@font-face {
  font-family: 'icons-font';
  src: url('/fonts/icons-font.eot');
  src: url('/fonts/icons-font.eot#iefix') format('embedded-opentype'),
  url('/fonts/icons-font.woff2?40259512') format('woff2'),
  url('/fonts/icons-font.ttf?40259512') format('truetype');
  font-weight: normal;
  font-style: normal;
}
{% endhighlight %}
<p>Następnie wszystkie nazwy klas z danym przedrostkiem powinny być zarezerwowane dla ikon (w naszym przypadku przedrostek "icon-"):</p>
{% highlight css %}
[class^="icon-"]:before, [class*=" icon-"]:before {
  display: inline-block;
  font-family: "icons-font";
  font-style: normal;
  font-variant: normal;
  font-weight: normal;
  line-height: 1em;
  text-transform: none;
  width: 1em;
}
{% endhighlight %}
<p>Następnie rozpoczynamy definiowanie poszczególnych ikon:</p>
{% highlight css %}
.icon-pencil:before                      { content: '\e800'; }
.icon-book:before                        { content: '\e801'; }
.icon-cross:before                       { content: '\e802'; }

...

.icon-plus:before                        { content: '\e80E'; }
.icon-minus:before                       { content: '\e80F'; }
{% endhighlight %}
<p>Od teraz możemy użyć dowolnej z tych ikon dodając prosty kawałek HTML:</p>
{% highlight css %}
<span class="icon-pencil"></span>
{% endhighlight %}
<p>Odpowiedni kolor ikony można uzyskać dodając kolejną klasę CSS i definiując dla niej regułę. Dla przykładu aby ikona ołówka była niebieska, możemy dodać jej klasę "pencil" i regułę:</p>
{% highlight css %}
.pencil {
  color: blue;
}
{% endhighlight %}
<h2>Plik SVG z używanymi ikonami</h2>
<p>Opcją posiadającą wszystkie zalety poprzedniej i jednocześnie pozbawioną jej wad jest stworzenie jednego pliku SVG ze wszystkimi potrzebnymi nam ikonami (tak, format SVG pozwala na takie rzeczy!). Definiując ikony w aplikacji internetowej w formacie SVG uzyskujemy możliwość:</p>
<ul>
  <li>dowolnego doboru rozmiaru ikony (format bezstratny)</li>
  <li>dowolnego ustawiania koloru</li>
  <li>naprawdę prostej edycji poszczególnych ikon (plik SVG to zwykły plik tekstowy!)</li>
</ul>
<h3>Jak możemy to uzyskać?</h3>
<p>Zaczniemy od stworzenia nowego pliku, np. <em>spritesheet.svg</em> z następuącą zawartością:</p>
{% highlight css %}
<svg>
  <defs>
    <g id="icon-pencil">
      <!-- definicja pierwszej ikony -->
    </g>
    <g id="icon-book">
      <!-- definicja drugiej ikony -->
    </g>
    <!-- i tak dalej... -->
  </defs>
</svg>
{% endhighlight %}
<p>Poszczególne ikony definiujemy wewnątrz elementów <g&gt;, nadając im unikalne identyfikatory (atrybut "id"). Samą definicję ikon możemy skopiować z jednego z darmowych źródeł (np. znanego i cenionego <a href="https://www.flaticon.com/" target="_blank">https://www.flaticon.com</a>) i wkleić do pliku. Możliwe jest także samodzielne stworzenie i edycja plików SVG, np. za pomocą programu <a href="https://inkscape.org/" target="_blank">Inkscape</a>. </p>
<h3>Dobrze, ale jak mogę wyświetlić taką ikonę na stronie?</h3>
<p>Na początku należy oczywiście umieścić&nbsp;odpowiedni plik na stronie, co można najprościej uczynić "wklejając" zawartość pliku <em>spritesheet.svg</em> do dokumentu HTML, na przykład jako pierwszy potomek elementu <body&gt;. Możliwe jest także dopisanie tego pliku jako zasobu do pobrania w sekcji <head&gt;:</p>
{% highlight css %}
<link rel="preload" href="spritesheet.svg" as="image">
{% endhighlight %}
<p>Następnie, w dowolnym miejscu naszej strony możemy odwołać się do dowolnej ikony poprzez kod:</p>
{% highlight css %}
<svg viewBox="0 0 100 100" class="icon icon-pencil">
  <use xlink:href="#icon-pencil"></use>
</svg>
{% endhighlight %}
<p>Należy tylko uważać na ścieżkę odwołującą się do ikony: w tym przypadku jest to po prostu #pencil-icon i jest to wartość właściwa gdy definicja ikony jest zawarta bezpośrednio w dokumencie HTML. Jeśli nie wklejamy zawartości pliku <em>spritesheet.svg</em> tylko odwołujemy się do niego po nazwie, to musimy pamiętać o niej w atrybucie xlink:href</p>
{% highlight css %}
<svg viewBox="0 0 100 100" class="icon icon-pencil">
  <use xlink:href="/spritesheet.svg#icon-pencil"></use>
</svg>
{% endhighlight %}
<h3>Bonus! Stylowanie za pomocą CSS</h3>
<p>Ikony w formacie SVG zamieszczone na stronie stanowią integralną część drzewa DOM, dlatego możliwe jest stylowanie ich (a nawet ich części!) za pomocą reguły fill w CSS:</p>
{% highlight css %}
.icon-pencil {
  fill: red;
}
{% endhighlight %}
<h3>Czy naprawdę jest tak idealnie?</h3>
<p>Odpowiedź brzmi: oczywiście nie! Choć wada, jaka tutaj się pojawia może nie mieć znaczenia w niektórych przypadkach. Mowa oczywiście o wsparciu przeglądarek. Dopóki korzystamy z metody osadzania zawartości pliku <em>spritesheet.svg</em> jest nieźle. IE 8+, Safari 5+, iOS 4.3+ i Android 2.3+ to naprawdę dobry wynik. Mniej różowo jest gdy postanawiamy korzystać z możliwości cache'owania przez przeglądarki tego pliku i umieszczamy go jako osobny zasób w <link&gt;. Wówczas tracimy wsparcie wszystkich wersji Internet Explorera, co niestety nadal jest nie do przyjęcia w wielu projektach. Z drugiej strony istnieje odpowiedni polyfill, tj. biblioteka <a href="https://www.npmjs.com/package/svg4everybody" target="_blank">svg4everybody</a>, co zawsze jest opcją do rozważenia.</p>
<h2>Ikony w aplikacji internetowej - podsumowanie</h2>
<p>Dostępnych możliwości jest jak widać wiele, i wybór najlepszej zależy od konkretnego przypadku. Dla aplikacji profesjonalnych, komercyjnych można jednak polecić metody ikon osadzonych w pliku fontu oraz ikon definiowanych w pojedynczym pliku SVG (spritesheet). Mimo że wymagają więcej wysiłku w tworzeniu i utrzymaniu to posiadają niepodważalne zalety. Możliwość definiowania dowolnego koloru i rozmiaru bez straty jakości są naprawdę dużymi plusami. Mam nadzieję, że artykuł pomógł Wam zdobyć lepsze rozeznanie w tym temacie, zapraszam do komentowania!</p>
