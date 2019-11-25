---
title: Hasła użytkowników i ich bezpieczne resetowanie
date: 2019-03-13 20:57:11.000000000 +01:00
categories:
- Bezpieczeństwo
- Na dłużej
- Polski
tags:
- bezpieczeństwo
- hasło
- token
permalink: "/2019/03/resetowanie-hasla-uzytkownika/"
excerpt: Choć dzisiejszy temat może wydawać się prosty (to przecież standardowa funkcjonalność,
  co nie?), to, jak postaram się udowodnić, jest tak tylko dopóki mamy w poważaniu
  kwestie bezpieczeństwa - najprościej przecież wysłać nowe hasło mailem i już. Czy
  aby na pewno? Zapraszam do lektury!
---
<p>Dzisiaj na tapetę bierzemy temat resetowania hasła użytkowników, głównie ze względu na popularność takiego rozwiązania, które obecne jest dzisiaj niemal na każdej witrynie. Fakt ten może sugerować, że jego implementacja nie jest sprawą szczególnie problematyczną, także z perspektywy bezpieczeństwa. Nie mam w zwyczaju jednak opierać się na sugestiach, dlatego zacznę od postawienia pytania:</p>
<h3>Czy możliwe jest zapewnienie bezpieczeństwa takiemu mechanizowi?</h3>
<p>Odpowiedź, w moim przekonaniu, brzmi: NIE. A przynajmniej dopóki mamy na myśli tradycyjny mechanizm z wykorzystaniem poczty e-mail i adresu przypisanego do użytkownika. I nie chodzi tu o to, że pełnej gwarancji bezpieczeństwa nie uzyskamy nigdy. Po prostu <strong>SMTP jako protokół wymiany wiadomości pomiędzy serwerami pocztowymi jest z założenia nieszyfrowany.</strong> Stwarza to niemożliwe do wykluczenia (niewielkie, ale jednak) niebezpieczeństwo przejęcia korespondencji pomiędzy serwerem aplikacji a użytkownikiem przez osobę trzecią, dlatego wszystkie poniższe rady będą miały na celu ograniczenie tego ryzyka.</p>
<h3>Zasada nr 1: Mechanizm resetu hasła oparty o token</h3>
<p>Wysyłając użytkownikowi jedynie <strong>token</strong> umożliwiający zmianę hasła, a nie np. hasło tymczasowe, nie zmieniamy jego stanu w naszym systemie. Użytkownika ciągle obowiązuje stare hasło, wiemy jedynie, że została podjęta próba jego zmiany. W przypadku <strong>hasła tymczasowego</strong> użytkownik posiada już zmienione hasło, co tworzy poważną lukę w bezpieczeństwie. Polega ona na tym, że każdy może zmienić hasło każdego innego użytkownika, jeśli tylko pozna jego adres e-mail. Co prawda nie będzie znał tego nowego hasła, ale nie chcemy przecież umożliwiać dokonywania takich cudów w naszej aplikacji. Opcja trzecia, czyli wysyłanie <strong>bieżącego hasła</strong> użytkownika mailem też odpada, ponieważ jako poważni deweloperzy nie przechowujecie go w plaintextcie (prawda?!). Najsensowniejszym rozwiązaniem wydaje się więc mechanizm resetu oparty o token, wysyłany w formie linka na adres e-mail użytkownika.</p>
<h4>Właściwości wysyłanego tokenu</h4>
<p>Oczywiście decydując się na wykorzystanie tokenu musimy zadbać, aby posiadał on konkretne właściwości, a mianowicie musi być:</p>
<ul>
  <li>jednorazowy</li>
  <li>nieprzewidywalny (kryptograficznie bezpieczny)</li>
  <li>odpowiednio długi</li>
  <li>przypisany jednoznacznie do jednego konta</li>
</ul>
<p>Zapewnienie powyższych właściwości wymaga jednak trochę wysiłku, stąd pytanie dlaczego każda z nich jest konieczna? </p>
<p><strong>Jednorazowość</strong> jest niezbędna, aby nie pozostawiać takiej "furtki”, jaką jest umożliwienie zmiany hasła użytkownikowi, otwartej zbyt długo (będzie o tym jeszcze mowa w dalszej części).</p>
<p><strong>Nieprzewidywalność</strong> oznacza użycie prawdziwie losowych generatorów liczb do stworzenia tokena w celu uniemożliwienia jego samodzielnego skonstruowania/odgadnięcia przez osobę niepowołaną. </p>
<p><strong>Długość</strong> zapewnia ochronę przed prostym atakiem typu <em>brute force</em>. Dzięki niej nie jest prawdopodobne wygenerowanie w sensownym czasie prawidłowego tokenu próbując wszystkich możliwych kombinacji. W praktyce stosuje się tokeny minimum 32-znakowe.</p>
<p><strong>Powiązanie z konkretnym kontem</strong> zapewnia, że token wygenerowany przez użytkownika A może zostać użyty do zmiany hasła tylko tego użytkownika, a nie np. nieświadomego niczego użytkownika B.</p>
<h4>Jeszcze jedna ważna notka o zawartości tokenu</h4>
<p><strong>Nie przechowuj</strong> w wygenerowanym tokenie informacji o koncie użytkownika ani o czasie jego ważności. Wbudowanie takich informacji w sam token sprawi, że możliwe stanie się  (potencjalne) resetowanie haseł innym osobom niż ta, która token wygenerowała. Modyfikacje mogłyby także np. sztucznie wydłużyć jego ważność. Token z założenia ma służyć jedynie identyfikacji odpowiedniego wpisu w bazie danych, tj. przypisanego do niego użytkownika.</p>
<h3>Zasada nr 2: Jak najszybsze unieważnianie tokenu</h3>
<p>Oczywistym jest, że jeśli już umożliwiamy zmianę hasła w aplikacji, to cała procedura powinna przebiegać sprawnie i szybko, bez pozostawiania otwartych “furtek” do systemu. Generalna zasada brzmi: <strong>pozostaw token ważnym tylko tak długo, jak to konieczne, ale nie dłużej niż przez x jednostek czasu</strong>. W praktyce oznacza to, że:</p>
<ul>
  <li>Generując token na żądanie użytkownika o zmianę hasła, musimy nadać&nbsp;mu krótki okres ważności, wynoszący standardowo do kilkunastu minut (np. 15 minut).</li>
  <li>Należy unieważnić token natychmiast, jeśli:
    <ul>
      <li>Procedura zmiany hasła <strong>zakończy się powodzeniem</strong> (ponieważ nie jest on już dłużej potrzebny).</li>
      <li>Użytkownik <strong>zaloguje się za pomocą dotychczasowego hasła</strong> (oznacza to, że jego zmiana nie jest już potrzebna, użytkownik przypomniał sobie dotychczasowe hasło).</li>
      <li><strong>Stworzone zostanie nowe żądanie zmiany hasła</strong> (ponieważ w każdej chwili maksymalnie jeden token może być aktywny dla pojedynczego konta).</li>
    </ul>
  </li>
</ul>
<p>W praktyce spotyka się także dodatkowe zabezpieczenie polegające na blokowaniu tokenu po jego pierwszym użyciu. Oznacza to, że korzystając z danego tokenu można przejść do strony umożliwiającej zmianę hasła tylko 1 raz, następne próby zakończą się niepowodzeniem, choć&nbsp;sam token pozostaje ciągle ważny.</p>
<h3>Zasada nr 3: Użycie nowej soli dla nowego hasła</h3>
<p>Po uwierzytelnieniu użytkownika za pomocą odpowiedniego tokenu przychodzi czas na rzeczywistą zmianę jego hasła w bazie danych. Pamiętać należy wtedy o użyciu nowej soli do haszowania zmienionego hasła. Pojawia się pytanie dlaczego jest to ważne? Otóż jeśli używana sól jest wspólna dla wszystkich haseł w bazie, to w przypadku 2 użytkowników o tym samym haśle będą oni posiadać ten sam hash hasła w bazie danych. Niestety, zaistnienie takiej sytuacji nie jest niemożliwe do wykluczenia. A co oznaczać będzie wystąpienie takiej sytuacji dla atakującego w przypadku wycieku danych? Będzie mocną przesłanką, że haszując hasła korzystano z tej samej soli.</p>
<h3>Zasada nr 4: Unieważnienie istniejących sesji</h3>
<p>Pomyślne zakończenie procedury zmiany hasła to jeszcze nie koniec, ponieważ wciąż mogą istnieć sesje użytkownika zalogowanego za pomocą starego hasła. Jakie to ma znaczenie? Użytkownik mógł chcieć zmienić swoje hasło właśnie dlatego, że stare zostało skompromitowane, to znaczy poznała je osoba trzecia. Właśnie dlatego <strong>wszystkie sesje użytkownika</strong> zmieniającego hasło <strong>muszą zostać unieważnione</strong>. Użytkownik z kolei po pomyślnym zakończeniu procedury przekierowany na stronę logowania. Zachowanie takie ma na celu zapewnienie, że tylko osoba zmieniająca hasło konta ma możliwość&nbsp;zalogowania się do aplikacji, tylko za pomocą nowego hasła.</p>
<h3>Podsumowanie</h3>
<p>Poprzez powyższe zasady starałem się pokazać kilka sposobów na uczynienie mechanizmu resetowania hasła wolnym od najpoważniejszych luk bezpieczeństwa. Oczywiście można wprowadzać kolejne usprawnienia, niemniej jednak powyższe rady są podstawowymi, od których należy zacząć.</p>
<p>Źródła:</p>
<ul>
  <li>
    <a
      href="https://crackstation.net/hashing-security.htm"
      target="_blank">
      https://crackstation.net/hashing-security.htm
    </a>
  </li>
  <li>
    <a
      href="https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html"
      target="_blank">
      https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
    </a>
  </li>
</ul>
