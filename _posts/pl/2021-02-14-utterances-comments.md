---
title: Komentarze na statycznym blogu? Dodaj je z pomocą utteranc.es!
date: 2021-02-16 20:00:00.000000000 +02:00
identifier: utterances-comments
categories:
- frontend
tags:
- utterances
excerpt:
  Piszesz techniczny, programistyczny blog i chcesz umożliwić czytelnikom
  komentowanie Twoich postów? Jeśli tak, ten wpis został stworzony dla Ciebie!
---
Oczywiście, kiedy prowadzisz swojego prywatnego bloga, funkcjonalność komentowania
wpisów jest często oferowana w standardzie, dla przykładu 
[Wordpress](https://wordpress.com){:target="_blank"} implementuje ją bez żadnych
dodatkowych wtyczek, out-of-the-box. Lecz co jeśli Twoja strona jest statyczna?
Okazuje się, że możesz użyć jednego z wielu dostępnych serwisów!

## Co mamy do wyboru?

Gdy zechciałem włączyć możliwość komentowania na tym blogu, z istniejących na
rynku rozwiązań znałem tylko [Disqus](https://disqus.com/){:target="_blank},
który zresztą do teraz jest tym najpopularniejszym. Posiada nawet darmowy plan
dla prywatnych stron, niestety wyświetla on wówczas reklamy, a jest to dla
mnie nieakceptowalne :/ Ponadto, załączenie takiego dodatku na stronie zauważalnie 
ją spowalnia, a przynajmniej jeśli mówimy o Disqus. Innym problemem
były dla mnie kwestie prywatności użytkowników - Disqus profiluje ich w celu
późniejszego wyświetlania reklam na innych stronach. Całkiem dobre opracowanie
tych kwestii możecie znaleźć
[tutaj](https://fatfrogmedia.com/delete-disqus-comments-wordpress/){:target="_blank}.
To ja jednak podziękuję.

Interesującą alternatywą (szanującą prywatność użytkowników oraz nie psującą
wydajności strony tak bardzo) jest [Hyvor Talk](https://talk.hyvor.com/){:target="_blank"}.
Nie posiada on większości z wad Disqusa, ale kosztem tutaj jest brak darmowego
planu. I jest to dla mnie zupełnie zrozumiałe - byłem gotów zapłacić za to
rozwiązanie, ale wtedy przypomniałem sobie o wielkości ruchu na moim blogu - 
tak, jest ona całkiem mizerna :D Nie chciałem więc płacić za serwis obsługujący 2 nowe
komentarze na rok.

Wreszcie, przeczytałem o [utteranc.es](https://utteranc.es/){:target="_blank"} -
czyli komponencie komentarzy napędzanym przez Githuba. A właściwie to przez
Github Issues. Naprawdę, ktokolwiek to wymyślił, dla mnie jest geniuszem! Już
tłumaczę: utteranc.es korzysta z API Github Issues do przechowywania i pobierania
komentarzy przed wyświetleniem ich na Twojej stronie. Wszystko co musisz zrobić to
dołączyć do niej mały skrypt z kilkoma parametrami, takimi jak nazwa **publicznego**
repo na Githubie (utteranc.es będzie tam tworzył issues oraz stamtąd pobierał 
komentarze). Każdy post na Twoim blogu generuje nowy issue na Githubie tak
szybko, jak tylko pierwszy użytkownik postanowi go skomentować :) **Dodanie
komentarza na Twojej stronie tworzy tak naprawdę ten komentarz w odpowienim
issue w podanym repo** i vice versa. Genialne! A rzeczy takie jak reakcje
(wbudowane przecież w możliwości komentarzy na Github Issues) masz w
gratisie :)

## Jak dodać utteranc.es do Twojej strony?

To naprawdę proste, zaczynamy od dodania do niej prostego tagu &lt;script&gt;, mniej więcej
takiego:

{% highlight html %}
<script
  src="https://utteranc.es/client.js"
  repo="<github-user-name>/<repo-name>"
  issue-term="pathname"
  label="comment"
  theme="github-light"
  crossorigin="anonymous"
  async>
</script>
{% endhighlight %}

Proszę pamiętaj o zamianie
    
    <github-user-name>/<repo-name>

na prawdziwe nazwy.

Istnieje także kilka opcji komfiguracyjnych dla utterances, jak na przykład
"issue-term", "label" czy "theme", podawane poprzez atrybuty tagu &lt;script&gt;.

Atrybut "issue-term" mówi utterances jak tworzyć tytuł issue w repo - może to 
robić bazując na ścieżce posta, URLu, tytule strony, numerze issue, można 
podać konkretną frazę itp.

    issue-term="pathname" | "url" | "title" | "og:title" | "<issue-term>"
    issue-number="<issue-number>"

Atrybut "label" jest opcjonalny, dołącza on podaną etykietę do tworzonego issue.

Atrybut "theme" kontroluje wygląd komponentu komentarzy i wspiera kilka możliwych
wartości:

    theme="github-light" | "github-dark" | "github-dark-orange" | "icy-dark" | "dark-blue" | "photon-dark" | "boxy-light"

Niemniej jednak, jak zawsze, najlepiej zasięgnąć rady w [oficjalnej dokumentacji](https://utteranc.es/){:target="_blank"}
projektu.

### Instalacja aplikacji utteranc.es na Githubie

Kolejnym krokiem jest zainstalowanie aplikacji utteranc.es na Githubie, można
ją znaleźć [tutaj](https://github.com/apps/utterances){:target="_blank"}. Po
instalacji należy wybrać repozytorium bądź repozytoria, do których dostęp ma mieć
aplikacja w celu tworzenia issues oraz komentarzy. I to już wszystko! Twój blog
powinien już wyświetlać sekcję komentarzy, dokładnie w miejscu, w którym dodałeś
tag &lt;script&gt;.

## Ale co z wadami?

Jak wszystko, ten serwis musi mieć także jakieś wady, prawda? Oczywiście, że tak,
a najpoważniejszą z nich jest zmuszanie użytkowników bloga, chcących komentować
posty do założenia konta na Githubie i autoryzowania apki utteranc.es w ramach
ich konta. Dopóki Ty oraz Twoi odbiorcy jesteście wszyscy bądź w większości 
programistami, to nie jest wielki problem (jak na przykład dla mnie w ramach tego 
bloga). Ale jeśli tak nie jest, nie zalecam używać utterances. Większość ludzi
nie wie czym jest Github i nie stworzy konta specjalnie dla Ciebie, aby móc coś
skomentować :)

## Podsumowanie

W moim przekonaniu, utteranc.es jest świetną opcją na włączenie komentarzy na
statycznej stronie tak długo, jak tylko mówimy o stronie odwiedzanej głównie przez
ludzi ze świata IT. Jest to rozwiązanie darmowe, open source, bez reklam i 
szpiegowania użytkowników, co rekompensuje fakt, że nadaje się do użytku tylko
przez wąską grupę ludzi. Ale znowu, dla mnie, na teraz, to i tak więcej niż potrzeba!
