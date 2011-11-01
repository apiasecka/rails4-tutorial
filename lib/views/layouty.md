#### {% title "Makieta aplikacji, czyli layout" %}

<blockquote>
 <p>
  {%= image_tag "/images/html5-layout-1.png", :alt => "[HTML5 Layout 1.]" %}
 </p>
 <p class="author">źródło: <a href="http://scottgale.com/blog/wp-content/uploads/2010/04/html5-layout-sm.png">Scot Gale</a></p>
</blockquote>

Co oznacza słowo *layout*:

* makieta, układ graficzny strony, okładki, książki
* rozmieszczenie, rozplanowanie elementów na stronie
* rozkład, plan techniczny, kompozycja

*Rails Guides*,
[Layouts and Rendering in Rails](http://edgeguides.rubyonrails.org/layouts_and_rendering.html) –
this guide covers the basic layout features of Action Controller and Action View.

Projekt [web-app-theme](https://github.com/pilu/web-app-theme) (autor Andrea Franz)
jest drugi w kategorii „Most Watched Overall Javascript project” na githubie.


## Layout aplikacji

Przykładowy layout aplikacji, po liftingu
(pl, utf-8, domyślny tytuł, shiv, cdn jquery, style):

    :::rhtml app/views/layouts/application.html.erb
    <!doctype html>
    <html lang="pl">
      <head>
        <meta charset="utf-8">
        <title><%= content_for?(:title) ? yield(:title) : "Fortunka v1.0" %></title>
        <!--[if lt IE 9]>
          <script src="http://html5shiv.googlecode.com/svn/trunk/html5.js"></script>
        <![endif]-->
        <script src="http://code.jquery.com/jquery-1.5.1.min.js"></script>
        <style>
          article, aside, details, figcaption, figure, footer, header,
          hgroup, menu, nav, section { display: block; }
        </style>
        <%= stylesheet_link_tag "application" %>
        <%= javascript_include_tag "application" %>
        <%= csrf_meta_tag %>
        <%= yield(:head) %>
      </head>
      <body>
        <div id="container">
          <% flash.each do |name, msg| %>
            <%= content_tag :div, msg, :id => "flash_#{name}" %>
          <% end %>
          <%= content_tag :h1, yield(:title) if show_title? %>
          <%= yield %>
        </div>
      </body>
    </html>

<blockquote>
 <p>
  {%= image_tag "/images/html5-blog.png", :alt => "[HTML5 Blog]" %}
 </p>
 <p class="author">źródło: <a href="http://html5doctor.com/designing-a-blog-with-html5/">html5 doctor</a></p>
</blockquote>

Generator *nifty:layout* definiuje kilka użytecznych metod
pomocniczych. Warto je dopisać do pliku *app/helpers/layout_helper.rb*:

    :::ruby
    module LayoutHelper
      def title(page_title, show_title = true)
        content_for(:title) { page_title.to_s }
        @show_title = show_title
      end
      def show_title?
        @show_title
      end

      def stylesheet(*args)
        content_for(:head) { stylesheet_link_tag(*args) }
      end
      def javascript(*args)
        content_for(:head) { javascript_include_tag(*args) }
      end
    end


## Linki

* Inayaili de León. [Have a Field Day with HTML5 Forms](http://24ways.org/2009/have-a-field-day-with-html5-forms)
* [Coding A HTML 5 Layout From Scratch](http://www.smashingmagazine.com/2009/08/04/designing-a-html-5-layout-from-scratch/)
* [Coding a CSS3 & HTML5 One-Page Website Template](http://tutorialzine.com/2010/02/html5-css3-website-template/)



# Co to są sitemaps?

Odpowiedzi udzielił [stJhimy](http://www.stjhimy.com/posts/2).

Przykładowy plik:

    :::xml
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      <url>
        <loc>http://sinatra.local/rails3/</loc>
        <priority>1.0</priority>
      </url>
      <url>
        <loc>http://sinatra.local/rails3/fortunes/1</loc>
        <priority>1.0</priority>
      </url>
    </urlset>

Taki plik wygenerujemy korzystając z *XMLbuilder*:

    :::ruby
    xml.instruct!
    xml.urlset "xmlns" => "http://www.sitemaps.org/schemas/sitemap/0.9" do

      xml.url do
        xml.loc "http://sinatra.local/fortunes" # wstawiamy swój adres
        xml.priority 1.0
      end
      # dodajemy wszystkie fortunki (wystarczyłby indeks)
      @fortunes.each do |fortune|
        xml.url do
          xml.loc fortune_url(fortune)
          xml.priority 1.0
        end
      end
      # można dodać więcej linków, dla przykładu
      @posts.each do |post|
        xml.url do
          xml.loc post_url(post)
          xml.lastmod post.updated_at.to_date
          xml.priority 0.61
        end
      end
    end

Wcześniej definiujemy `@fortunes` i dodajemy routing, np.

    http://sinatra.local/fortunes/sitemap.xml

Na koniec aktywujemy sitemap dopisując w pliku *public/robots.txt*:

    Sitemap: http://sinatra.local/fortunes/sitemap.xml

Albo wykonujemy ping:

    www.google.com/webmasters/tools/ping?sitemap=http://sinatra.local/fortunes/sitemap.xml


<blockquote>
 <p>
  {%= image_tag "/images/article-page-layout.png", :alt => "[HTML5 Article Page Layout]" %}
 </p>
 <p class="author">źródło: <a href="http://boblet.tumblr.com/post/141239118/html5-structure4">@boblet</a></p>
</blockquote>

# Różne rzeczy…

**Aktywne zakładki.**
Dla przykładu, przyjmijmy, że na każdej stronie naszej aplikacji
wyświetlamy trzy zakładki *Homepage*, *About us* oraz *Contact*,
a kod elementu HTML z zakładkami, gdy jesteśmy na stronie *Homepage*
ma wyglądać tak:

    :::html
    <div id="mainMenu">
      <ul>
        <li class="active"><a href="/home">Homepage</a></li>
        <li><a href="/about">About us</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </div>

czyli element listy *ul* z linkiem do strony na której aktualnie jesteśmy
powinien mieć dodany atrybut **class** ustawiony na **active**.

A tak chcielibyśmy, to wpisać layoucie *views/layouts/application.html.erb*:

    :::rhtml
    <div id="mainMenu">
      <%= navigation ['/home','Homepage'],['/about','About us'],['/contact','Contact'] %>
    </div>

Poniższy kod, po wpisaniu w *helpers/layout_helper.rb*, realizuje takie podejście:

    :::ruby
    def navigation(*data)
      content_tag :ul do
        data.map do |link, name|
          content_tag :li, link_to("#{name}", link),
              :class => ("active" if controller.controller_name == link[1,link.length])
        end
      end
    end

Gotowy przykład, zob. *labs/003-Tabbed_Navigation*.

Na koniec kilka linków na ten temat:

* [Selected Tab Navigation](http://railsforum.com/viewtopic.php?id=30174)
* [TabsOnRails: Creating and managing Tabs with Ruby on Rails](http://code.simonecarletti.com/projects/tabsonrails/wiki)

**Zagnieżdzone layouty z content_for.**
Załóżmy, że aplikacja składa się z trzech kontrolerów:

* HomeController
* AboutController
* ContactController

i wszystkie kontrolery używają jednego layoutu – *application.html.erb*.

Przypuśćmy, że layouty dla kontrolerów muszą się nieco różnić,
dla przykładu – kolorem tła.

Zamiast powielenia i edycji *layouts/application.html.erb* można
postąpić tak. Tworzymy plik *layouts/about.html.erb* o zawartości:

    :::rhtml
    <% content_for :head do %>
      <style>
        #background { background-color: #956E6F; }
      </style>
    <% end -%>
    <%= render :file => 'layouts/application' %>

i według tego schematu tworzymy plik *layouts/contact.html.erb*.
