#### {% title "Remote links" %}

* Co oznacza zwrot „remote links”?
* Jak implementujemy „remote links”?
* Co to jest „Unobtrusive JavaScript” (w skrócie *UJS*)?
* Co to są „Progressive Enhancements” (stopniowe udoskonalenia)?


<!-- * Przykład jest [tutaj](https://github.com/wbzyl/rails31-remote-links). -->

Do eksperymentów z *remote links* użyjemy aplikacji
[Fortunka](http://sharp-ocean-6085.herokuapp.com/) wdrożonej
na Heroku na poprzednim wykładzie:

    :::bash
    git clone git@heroku.com:sharp-ocean-6085.git
    cd sharp-ocean-6085
    rake db:create
    rake db:migrate # aplikacja korzysta z bazy PostgreSQL; podmienić na SQLite?
    rake db:seed

Kod aplikacji jest też w moim publicznym repo na GitHubie, tutaj –
[sharp-ocean-6085](https://github.com/wbzyl/sharp-ocean-6085).

Eksperymenty z *remote links* będą ciekawsze jeśli
użyjemy biblioteki [jQuery UI](http://jqueryui.com/).
Wykorzystamy efekty [„explode”, „fade” i „highlight”](http://jqueryui.com/demos/effect/).

Po zaznajomieniu się z tymi efektami, zabieramy się do instalacji jQuery UI.
Ze strony [download](http://jqueryui.com/download) pobieramy paczkę
z*theme* (skórką?) **Start** ze wszystkimi efektami (tak będzie wygodniej).

Pobrane archiwum rozpakowujemy:

    :::bash
    unzip jquery-ui-1.8.18.custom.zip

Następnie kopiujemy pliki do odpowiednich katalogów w *vendor/assets*.

    :::bash
    cp  css/start  sharp-ocean-6085/vendor/assets/stylesheets/
    cp  js/jquery-ui-1.8.18.custom.min.js  sharp-ocean-6085/vendor/assets/javascripts/

Skopiowane pliki dopisujemy do pliku *application.js*

    :::js app/assets/javascripts/application.js
    //= require jquery
    //= require jquery_ujs
    //= require twitter/bootstrap
    //= require jquery-ui-1.8.18.custom.min

oraz do pliku *application.css.less*:

    :::css app/assets/stylesheets/application.css.less
    @import "twitter/bootstrap";
    @import "fontawesome";
    @import "digg_pagination";
    @import "start/jquery-ui-1.8.18.custom.css";

Sprawdzamy, czy plik te są wczytywane:

    :::bash
    curl localhost:3000/assets/jquery-ui-1.8.18.custom.min.js
    curl localhost:3000/assets/start/jquery-ui-1.8.18.custom.css
    http://localhost:3000/assets/start/images/ui-bg_inset-hard_100_fcfdfd_1x100.png

Jeśli wszystko działa, to dla rozruszania wykonujemy kilka poleceń
z programem *curl*:

    :::bash
    curl -v -X GET -H 'Accept: application/json' localhost:3000/fortunes/44
    curl    -X GET -H 'Accept: application/json' localhost:3000/fortunes/45
    curl -v -X DELETE localhost:3000/fortunes/2         # wpisujemy numery istniejących fortunek
    curl    -X DELETE localhost:3000/fortunes/3         # jw (już wspomniane)
    curl -v -X DELETE localhost:3000/fortunes/4.json    # jw
    curl -v -X DELETE -H 'Accept: application/json' localhost:3000/fortunes/5

<!--
   te polecenia (ten fragment) / których (który)
   takiej pracy / o jakiej wcześniej rozmawiał (jakie zastępuje przymiotniki)
-->

Po wykonaniu których poleceń na konsolę jest wypisywane:

    :::html
    <html><body>You are being <a href="http://localhost:3000/fortunes">redirected</a>.</body></html>

Dodajemy fortunkę do bazy:

    :::bash
    curl -v -X POST -H 'Content-Type: application/json' \
      --data '{"quotation":"I hear and I forget."}' localhost:3000/fortunes.json
    curl    -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' \
      --data '{"quotation":"I hear and I forget."}' localhost:3000/fortunes

**Uwaga:** W trakcie eksperymentów, cały czas podglądamy co się dzieje
na konsoli przeglądarki (zakładki *Sieć*, *XHR*).


## Zabawy z przyciskiem *Destroy*

Na początek zmienimy nieco kod metody *destroy*:

    :::ruby
    # DELETE /fortunes/1
    # DELETE /fortunes/1.json
    # DELETE /fortunes/1.js
    def destroy
      @fortune = Fortune.find(params[:id])
      @fortune.destroy

      respond_to do |format|
        format.html { redirect_to fortunes_url }
        format.json { head :no_content }         //=? { render json: @fortune }
        format.js   # destroy.js.erb
      end
    end

Po usunięciu fortunki, wykonywana jest
jedna linijka kodu w bloku *respond*:

* *format.html*
* *format.json*
* *format.js*


### Usuwanie rekordu z *format.html*

Wygenerowany przez scaffold link z *index.html.erb*:

    :::rhtml
    <%= link_to 'Destroy', fortune,
       confirm: 'Are you sure?',
       method: :delete %>


Przykład wygenerowanego kodu HTML:

    :::rhtml
    <a href="/fortunes/1"
       data-confirm="Are you sure?"
       data-method="delete"
       rel="nofollow">Destroy</a>

Jak to działa? Co oznacza kod `rel="nofollow"`?
Skąd się wzięła liczba `1`?


### Usuwanie rekordu z *format.json*

Zmienimay kod linku w  pliku *index.html.erb* na:

    :::rhtml
    <%= link_to 'Destroy', fortune,
       confirm: 'Are you sure?',
       method: :delete,
       remote: true,
       data: { type: :json } %>

Wygenerowany kod HTML:

    :::rhtml
    <a href="/fortunes/1"
       data-confirm="Are you sure?"
       data-method="delete"
       data-remote="true"
       data-type="json"
       rel="nofollow">Destroy</a>

Jak to działa? Firefoks, Firebug, zakładka Sieć / XHR, gdzie sprawdzamy
nagłówki zapytania i odpowiedzi.

Po kliknięciu przycisku „Destroy” wysyłane jest żądanie *DELETE*
z id klikniętej fortunki. Oto niektóre nagłówki żądania (zapytania):

              Accept	application/json, text/javascript, */*; q=0.01
     Accept-Encoding	gzip, deflate
     Accept-Language	pl,en-us;q=0.7,en;q=0.3
          Connection	keep-alive
              Cookie	__utma=1118…
                 DNT	1
                Host	localhost:3000
             Referer	http://localhost:3000/
          User-Agent	Mozilla/5.0…
        X-CSRF-Token	WijggKCspDz0...Us=
    X-Requested-With	XMLHttpRequest

Fortunka ze wskazanym id zostaje usunięta z bazy.
Następnie wysyłana jest odpowiedź:

       Cache-Control	no-cache
          Connection	close
              Server	thin 1.3.1 codename Triple Espresso
          Set-Cookie	_fortunka_session=Ah7B0...D4; path=/; HttpOnly
        X-Request-Id	3719019c35c4674a7de1b9c25e6e3368
           X-Runtime	0.587928
     X-UA-Compatible	IE=Edge

Ale my ciągle jesteśmy na tej samej, nie zmienionej, stronie.
Usunięta przed chwilą fortunka nadal jest wyświetlana na stronie.
Powinniśmy ją usunąć ze strony. Jak to zrobić?
Do usunięcia fortunki ze strony użyjemy efektu *explode*.


## Ręczna symulacja efektów na konsoli

Wchodzimy na stronę główną aplikacji, gdzie otwieramy okno z konsolą
JavaScript (Chrome – Shift+Ctrl+J, Firefox+Firebug – F12).

Podglądamy id pierwszej na stronie fortunki.
Jeśli jest to, na przykład */fortunes/4*, to na konsoli wpisujemy:

    :::js
    r = $("a[href='/fortunes/4']")
    a = r.closest("article")
    a.effect("explode")  // ew. a.effect("explode", "slow")

Wybrana fortunka powinna zniknąć ze strony.


<blockquote>
  <h2>Ściąga ze zdarzeń</h2>
<ul>
 <li>obsługa zdarzeń:
  zdarzenie może obsłużyć dowolna funkcja JavaScript;
  funkcję obsługującą zdarzenie można przypisać do dowolnego
  elementu html, może być uruchamiane po załadowaniem strony, itd;
  zdarzenia można też definiować samemu, np. <i>ajax:success</i>
 <li>przykładowe zdarzenia obsługiwane przez przeglądarkę:
  onclick, onmouseover, onsubmit, onload…
 <li>kiedy zachodzi jakieś zdarzenie do którego przypisano
  jakieś funkcje to zostaną one wszystkie wykonane;
  jeśli takich funkcji nie przypisano – nic się nie dzieje
 <li><a href="http://www.quirksmode.org/js/events_order.html">co to jest bubbling</a>?
 <li>jak przypisać funkcję obsługującą zdarzenie?
  jak powiązać jakieś zdarzenie z funkcją?
  najwygodniej jest skorzystać z funkcji zdefiniowanych w jQuery
</ul>
  {%= image_tag "/images/bubbles.jpg", :alt => "[Save the Bubbles!]" %}
</blockquote>

Ten sam efekt uzyskamy po wklejeniu poniższego kodu do pliku
*application.js*:

    :::js app/assets/javascripts/application.js
    $(function() {
      $('a[data-type=\"json\"]').bind('ajax:success',
         function(event, data, status, xhr) {
           $(this).closest('article').effect('explode');
         }
      );
    });

<!--
  * opóźnienia i interwały
  * setTimeout  – wykonać kod **za** jakiś czas
  * setInterval – wykonać kod **co** jakiś czas
-->

Powyżej korzystamy ze zdarzenia *ajax:success* zdefiniowanego
w [jQuery Rails](https://github.com/rails/jquery-ujs)
(*unobtrusive scripting adapter for jQuery*).
Na stronie wiki [Custom events fired during „data-remote” requests](https://github.com/rails/jquery-ujs/wiki/ajax)
znajdziemy opis pozostałych zdarzeń.

### ☕☕ Przechodzimy na *CoffeeScript*

☕☕ Usuwamy plik *fortunes.js*.
Zamiast niego wstawiamy plik *fortunes.js.coffee* o zawartości:

    :::js app/assets/javascripts/fortunes.js.coffee
    jQuery ->
      $('a[data-type="json"]').bind 'ajax:success',
        (event, data, status, xhr) ->
          $(this).closest('article').effect('explode')

i nazwę tego pliku dopisujemy do *application.js*.


## Usuwanie rekordu z *format.js*

Zmieniony link z *index.html.erb* (z usuniętym atrybutem *data-confirm*):

    :::rhtml
    <%= link_to 'Destroy', fortune,
       method: :delete,
       remote: true %>

Wygenerowany kod HTML:

    :::html
    <a href="/fortunes/25"
       data-method="delete"
       data-remote="true"
       rel="nofollow">Destroy</a>

Jak to działa? Po tych zmianach i odświeżeniu zawartości strony,
kliknięcie przycisku „Destroy” nie daje widocznego efektu.
Ale na konsoli JavaScript pojawia się komunikat:

    DELETE http://localhost:3000/fortunes/8 500 (Internal Server Error)

a w logach aplikacji znajdujemy:

    ActionView::MissingTemplate
    (Missing template fortunes/destroy, application/destroy with
       {:locale=>[:en], :formats=>[:js, :html], :handlers=>[:erb, :builder, :coffee]}.
    Searched in: * ".../sharp-ocean-6085/app/views"):
    app/controllers/fortunes_controller.rb:78:in `destroy'

Oznacza to, że w katalogu *app/views* brakuje szablonu *fortunes/destroy.js.erb*.
Tworzymy taki szablon. Na razie, aby sprawdzić czy dobrze
rozszyfrowaliśmy te komunikaty, wpisujemy w nim funkcję *alert*:

    :::js app/views/fortunes/destroy.js.erb
    alert("SUCCESS: usunięto fortunkę");

Sprawdzamy, czy *alert* działa. Klikamy przycisk „Destroy”.
Powinno się pojawić okienko alert.

Jeśli wszystko działa, wymieniamy kod na taki:

    :::js app/views/fortunes/destroy.js.erb
    $('a[href="<%= @fortune_path %>"]').closest('article').effect('explode');

gdzie zmienną *@fortune_path* zdefiniowaliśmy w kontrolerze:

    :::ruby
    def destroy
      @fortune = Fortune.find(params[:id])
      @fortune_path = Rails.application.routes.url_helpers.fortune_path(@fortune)

      @fortune.destroy
      respond_to do |format|
        format.html { redirect_to fortunes_url }
        format.json { head :no_content }          #=? { render json: @fortune }
        format.js   # destroy.js.erb
      end
    end

Dlaczego na taki kod, a nie na inny?
Dlaczego w pliku *destroy.js.erb* nie wpisujemy:

    :::rhtml
    <%= fortune_path(@fortune) %>

tylko wyliczamy ścieżkę *fortune_path(@fortune)* w kontrolerze?

Zobacz też dyskusję na *stack**overflow***,
[Can Rails Routing Helpers (i.e. mymodel_path(model)) be Used in Models?](http://stackoverflow.com/questions/341143/can-rails-routing-helpers-i-e-mymodel-pathmodel-be-used-in-models).


## Remote modal show/new/edit pages

<!-- Cały przykład jest [tutaj](https://github.com/wbzyl/rails31-remote-links).-->

Co to są *modals*? Opis i demo znajdziemy w dokumentacji
[JavaScript plugins](http://twitter.github.com/bootstrap/javascript.html#modals).

Spróbujemy użyć *modal windows* do uproszczenia interfejsu — Show+New+Edit.

Bibliotekę *modal.js* powinna być już zainstalowana.
Możemy to sprawdzić przeklikujac do przegladarki uri poniżej:

    http://localhost:3000/assets/twitter/bootstrap/modal.js

Powinien się pojwić kod wtyczki.

Zaczniemy od bliższego przyjrzenia się okienkom modalnym
frameworka Bootstrap. Pobieramy archiwum zip ze strony
[Bootstrap, from Twitter](http://twitter.github.com/bootstrap/).
Rozpakowujemy archiwum. Do utworzonego katalogu *bootstrap*
dodajemy plik *index.html* o poniższej zawartości:

    :::html
    <!doctype html>
    <html>
      <head>
        <meta charset=utf-8>
        <link rel="stylesheet" href="/css/bootstrap.css">
        <script src="http://code.jquery.com/jquery.min.js"></script>
        <script src="/js/bootstrap.js"></script>
        <title>Bootstrap Modal Windows</title>
      </head>
      <body>
        <a data-toggle="modal" href="#myModal" class="btn btn-primary btn-large">Launch demo modal</a>
        <article id="myModal" class="modal hide fade">
          <div class="modal-header">
            <a class="close" data-dismiss="modal" >&times;</a>
            <h3>Modal Heading</h3>
          </div>
          <div class="modal-body">
            <p>One fine body…
          </div>
          <div class="modal-footer">
            <a href="#" class="btn" data-dismiss="modal">Close</a>
            <a href="#" class="btn btn-primary">Save changes</a>
          </div>
        </article>
      </body>
    </html>

Teraz uruchamiamy jakiś mini serwer stron statycznych,
na przykład *serve* dla NodeJS:

    :::bash konsola
    npm install -g serve

W katalogu *bootstrap* uruchamiamy na konsoli serwer:

    :::bash konsola
    serve

Przeanalizować [ten przykład](https://gist.github.com/1450706w)
z przyciskiem i funkcją obsługi zdarzenia *onclick* tego przycisku.


### Remote Show

Coś prostszego?

* [Show your objects baby!](http://blog.plataformatec.com.br/tag/show_for/)

Skorzystamy z szablonów [EJS](https://github.com/sstephenson/ruby-ejs).
Do *Gemfile* dopisujemy i instalujemy ten gem:

    :::ruby Gemfile
    gem 'ejs'

Oto szablon EJS dla *show*:

    :::rhtml app/assets/javascripts/templates/show.jst.ejs
    <article id="<%= modal %>" class="modal hide fade">
      <div class="modal-header">
        <a class="close" data-dismiss="modal" >&times;</a>
        <h3>Fortune #<%= id %></h3>
      </div>
      <div class="modal-body">
        <p><%= quotation %></p>
        <p class="source"><%= source %></p>
      </div>
      <div class="modal-footer">
        <a href="#" class="btn" data-dismiss="modal">Close</a>  <!-- TODO: link_to -->
        <a href="#" class="btn btn-primary">Save changes</a>    <!-- j.w. -->
      </div>
    </article>
    <!-- TODO: wstawić pozostałe przyciski -->
    <a data-toggle="modal" href="<%= modal %>" class="btn btn-primary btn-large">Show</a>

W oknie modalnym dla *Show* usunąłem przycisk *Back*.
W tym kontekście nie ma on sensu. Dlaczego?


**TODO** Niepotrzebne?

Po kliknięciu przycisku *Show*, pobieramy z bazy za pomocą żądania AJAX
JSON-a z danymi. Następnie skorzystamy szablonu EJS do wygenerowania
kodu HTML okna, który po dodaniu do strony, pokazujemy (**TODO**):

    :::js
    $(function() {
      $('.show').bind('click', function() { // show dodano w index.html.erb
        var href = $(this).attr('href');
        var id = href.slice(1).split('/').join('-');  // np. fortune-31
        $.ajax({
          url: href,
          dataType: "json"
        }).done(function(data) {
          // TODO: należy usunąć; sami nie powinniśmy implementować cache!
          if ($('#' + id).length == 0) { // modal is not present
            // use EJS template
            $(".page-header").append(JST["templates/show"]({
              modal: id,
              id: data.id,
              quotation: data.quotation,
              source: data.source }));
          };
          // pokaż okno modalne
          $('#' + id).modal({backdrop: "static", keyboard: true, show: true});
        });
        return false;
      });
    });

**TODO:** Na razie przycisk *Close* nie działa. Aby go uaktywnić,
można dopisać do elementu klasę *close*. Niestety do tej klasy przypisany jest CSS
psujący wygląd przycisku. Dlatego postąpimy tak:


    :::js
    if ($('#' + id).length == 0) { // modal is not present
      // use EJS template
      $(".page-header").append(JST["templates/show"]({
        modal: id,
        id: data.id,
        quotation: data.quotation,
        source: data.source }));
        $(".page-header .default").bind('click', function() {
          $('#' + id).modal('hide');
        });
    };

I to już w zasadzie koniec zabaw z „remote links”.


## Remote Edit

Na początek przerobimy formularz na „remote”, co oznacza że po kliknięciu przycisku
submit będzie wysyłane żądanie AJAX do bazy.

**TODO:** Przykłady są z **Formtastic**.
Kod dla *Simple Form* jest nieco inny. Poprawić!

Przeróbka jest trywialna. Wystarczy dopisać `remote: true` do *semantic_form_for*:

    :::rhtml _form.html.erb
    <%= semantic_form_for @fortune, remote: true do |f| %>

Oto kod HTML wygenerowany z szablonu *_form.html.erb* fortunki:

    :::rhtml
    <div class="form edit">
    <form accept-charset="UTF-8" action="/fortunes/54" class="formtastic fortune" data-remote="true"
          id="edit_fortune_54" method="post" novalidate="novalidate">
        <div style="margin:0;padding:0;display:inline"><input name="utf8" type="hidden" value="&#x2713;" />
        <input name="_method" type="hidden" value="put" />
        <input name="authenticity_token" type="hidden" value="Ps++PiFakL52X1UuxGHtNsPb3OgNyqOfpqhXyZWT1jE=" /></div>
        ...
        <fieldset class="buttons"><ol>
          <li class="commit button"><input class="update" name="commit" type="submit" value="Update Fortune" /></li>
        </ol>
        ...
      </form>
      <div class="link">
        <a href="/fortunes/55" class="btn primary">Show</a>
        <a href="/fortunes" class="btn primary">Back</a>
      </div>
    </div>

Jak widać, w szablonie skorzystano z wielu metod pomocniczych *Rails* i *Formtastic*.
Wobec tego przepisanie *_form.html.erb* na EJS nie ma większego sensu.

Dlatego postąpimy tak. Wygenerujemy cały formularz na serwerze.
Skorzystamy z takiego szablonu:

    :::rhtml edit.text.html
    <%= render('form') %>

via metoda *edit* z dodanym formatem tekstowym:

    :::ruby
    # GET /fortunes/1/edit
    def edit
      @fortune = Fortune.find(params[:id])

      respond_to do |format|
        format.html # edit.html.erb
        format.text # edit.text.erb
      end
    end

**Dziwne?** Musiałem jeszcze zrobić coś takiego:

    :::bash
    ln -s _form.html.erb _form.text.erb

Inaczej zgłaszany był błąd z powodu brakującego szablonu.

Wyrenderowany(?) kod HTML formularza pobierzemy via AJAX
i wstawimy go do szablonu EJS:

    :::js app/assets/javascripts/application.js
    $('.edit').bind('click', function() {
      var href = $(this).attr('href');
      var id = href.slice(1).split('/').join('-');  // ex. fortune-31-edit

      $.ajax({
        url: href,
        dataType: "text"  // tak wymuszamy format tekstowy
      }).done(function(data) {
        $('#' + id).detach(); // usuwamy okno modalne z DOM
        $(".page-header").append(JST["templates/edit"]({
          modal: id,
          form: data }));

        $("#" + id + " .default").bind('click', function() {
          $('#' + id).modal('hide');  // albo detach?
        });

        $('#' + id).bind('ajax:success', function(event, data, status, xhr) {
          $('#' + id).modal('hide');  // a może detach?
        });
        // TODO: failure

        $('#' + id).modal({backdrop: "static", keyboard: true, show: true});
      });
      return false;
    });

A to użyty powyżej szablon EJS:

    :::rhtml templates/show.jst.ejs
    <article id="<%= modal %>" class="modal hide fade in">
      <div class="modal-header">
        <div class="close">×</div>
        <h3>Edit Fortune</h3>
      </div>
      <div class="modal-body">
        <%= form %>
      </div>
      <div class="modal-footer">
        <div class="btn default">Close</div>
      </div>
    </article>

Do kodu metody *update* musiałem dopisać wiersz z *format.js*:

    :::ruby
    # PUT /fortunes/1
    # PUT /fortunes/1.json
    def update
      @fortune = Fortune.find(params[:id])

      respond_to do |format|
        if @fortune.update_attributes(params[:fortune])
          format.html { redirect_to @fortune, notice: 'Fortune was successfully updated.' }
          format.json { head :ok }
          format.js { render nothing: true }
        ...

Bez *format.js* renderowane było (Firebug jest wielki ♬♬♬):

    :::ruby
    format.html { redirect_to @fortune, notice: 'Fortune was successfully updated.' }

**Bug?** A powinien być zgłaszany błąd o brakującym szablonie *update.js.erb*.

**Uwaga:** W kodzie powyżej powtarza się ten sam schemat: `# + id` v. `id`.
Coś trzeba z tym zrobić.

Pozostaje przygotować szablon EJS, do którego wstawimy pobrany formularz.
Na koniec całość dodamy do okna modalnego – podobnie jak to zrobiliśmy
w wypadku *show*.


### Nieco CSS

Poprawiamy nieco wygląd widoków:

    :::css app/assets/stylesheets/bootstrap-container-app.css.scss
    .modal {
      padding: 1em;
      h3 {
        font-size: 20px; }
      p {
        font-size: 18px; }
      .source {
        padding-top: 0.5em;
        font-style: italic; }
      .close {
        cursor: pointer;
        padding: 1ex; }
    }


### Dokumentacja

* [Sprockets](http://rubydoc.info/gems/sprockets/2.1.2/file/README.md) – *RubyDoc*
* [Sprocekts](https://github.com/sstephenson/sprockets) – *Github*
* [CSS Cursors](http://www.w3schools.com/cssref/playit.asp?filename=playcss_cursor&preval=default)
* [Humane JS](https://github.com/wavded/humane-js) –
  a simple, modern, browser notification system

Embedded CoffeeScript templates:

* [Eco](https://github.com/sstephenson/eco) –
  embedded CoffeeScript templates
* [Ruby Eco](https://github.com/sstephenson/ruby-eco) –
  a bridge to the official Eco compiler for embedded CoffeeScript templates.

Instalacja:

    :::bash
    gem install coffee-script execjs therubyracer # zależności
    gem install --pre eco-source
    gem install eco
