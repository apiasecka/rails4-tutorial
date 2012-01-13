#### {% title "Wyszukiwanie ElasticSearch" %}

<blockquote>
 {%= image_tag "/images/john_cage.jpg", :alt => "[John Cage]" %}
 <p>I can't understand why people are frightened of new ideas.
    I'm frightened of the old ones.
 </p>
 <p class="author">— John Cage (1912–1992)</p>
</blockquote>

[What's Wrong with SQL search](http://philip.greenspun.com/seia/search).

Strona domowa ElasticSearch, dokumentacja:

* [You know, for Search](http://www.elasticsearch.org/)
* [Guides](http://www.elasticsearch.org/guide/):
  - [Setup](http://www.elasticsearch.org/guide/reference/setup/)
  - [API](http://www.elasticsearch.org/guide/reference/api/)
  - [Query](http://www.elasticsearch.org/guide/reference/query-dsl/)
  - [Mapping](http://www.elasticsearch.org/guide/reference/mapping/)
  - [Facets](http://www.elasticsearch.org/guide/reference/api/search/facets/index.html)
* [Setting up ElasticSearch ](http://www.elasticsearch.org/tutorials/2010/07/01/setting-up-elasticsearch.html)

ElasticSearch driver dla języka Ruby:

* Karel Minarik.
  [Tire](https://github.com/karmi/tire) – a rich Ruby API and DSL for the ElasticSearch search engine/database

Fedora:

* [Elasticsearch RPMs](https://github.com/tavisto/elasticsearch-rpms)

Różne:

* Karel Minarik
  - [Data Visualization with ElasticSearch and Protovis](http://www.elasticsearch.org/blog/2011/05/13/data-visualization-with-elasticsearch-and-protovis.html)
  - [Your Data, Your Search, ElasticSearch (EURUKO 2011)](http://www.slideshare.net/karmi/your-data-your-search-elasticsearch-euruko-2011)
  - [Reversed or “Real Time” Search in ElasticSearch](http://karmi.github.com/tire/play/percolated-twitter.html) –
  czyli „percolated twitter”
  - [Route requests to ElasticSearch to authenticated user's own index](https://gist.github.com/986390) (wersja dla Nginx)
* Clinton Gormley.
  [Terms of endearment – the ElasticSearch Query DSL explained](http://www.elasticsearch.org/tutorials/2011/08/28/query-dsl-explained.html)

Przykładowe aplikacje:

* Karel Minarik.
  [Search Your Gmail Messages with ElasticSearch and Ruby](http://ephemera.karmi.cz/)
  (Sinatra)


## Instalacja ze źródeł

Rozpakowujemy archiwum z ostatnią wersją
[ElasticSearch](http://www.elasticsearch.org/download/) (ok. 16 MB):

    :::bash
    unzip elasticsearch-0.18.6.zip

A tak uruchamiamy *elasticsearch*:

    :::bash
    elasticsearch-0.18.6/bin/elasticsearch -f

I już! Domyślnie ElasticSearch nasłuchuje na porcie 9200.


<blockquote>
 <p>The usual purpose of a full-text search engine is to return
  <b>a small number</b> of documents matching your query.
</blockquote>

## Your data, Your search

Kilka, nieco zmienionych, przykładów z tej strony
[Your Data, Your Search](http://www.elasticsearch.org/blog/2010/02/12/yourdatayoursearch.html).

**Interpretacja uri w zapytaniach kierowanych do ElasticSearch:**

<pre>http://localhost:9200/<b>⟨index⟩</b>/<b>⟨type⟩</b>/...
</pre>

<blockquote>
 <p>Field names with the <b>same name</b> across types are highly
 recommended to have the <b>same type</b> and same mapping characteristics
 (analysis settings for example).
</blockquote>

Książka:

    :::json book.json
    {
      "isbn" : "0812504321",
      "name" : "Call of the Wild",
      "author" : {
         "first_name" : "Jack",
         "last_name" : "London"
       },
       "pages" : 128,
       "tags" : ["fiction", "children"]
    }

Dodajemy książkę do indeksu *amazon*:

    :::bash
    curl -XPUT http://localhost:9200/amazon/books/0812504321 -d @book.json
      {"ok":true,"_index":"amazon","_type":"books","_id":"0812504321","_version":1}

Przykładowe zapytanie (w **query string**):

    :::bash
    curl 'http://localhost:9200/amazon/books/_search?pretty=true&q=author.first_name:Jack'

CD:

    :::json cd.json
    {
       "asin" : "B00192IV0O",
       "name" : "THE E.N.D. (Energy Never Dies)",
       "artist" : "Black Eyed Peas",
       "label" : "Interscope",
       "release_date": "2009-06-09",
       "tags" : ["hip-hop", "pop-rap"]
    }

Dodajemy CD do indeksu *amazon*:

    :::bash
    curl -XPUT http://localhost:9200/amazon/cds/B00192IV0O -d @cd.json
      {"ok":true,"_index":"amazon","_type":"cds","_id":"B00192IV0O","_version":1}

Przykładowe zapytanie:

    :::bash
    curl 'http://localhost:9200/_search?pretty=true&q=label:Interscope'

Wyszukiwanie po wszystkich typach w indeksie *amazon*:

    :::bash
    curl 'http://localhost:9200/amazon/_search?pretty=true&q=name:energy'

Wyszukiwanie po kilku typach:

    :::bash
    curl 'http://localhost:9200/amazon/books,cds/_search?pretty=true&q=name:energy'

Teraz pora na posprzątanie po sobie. Usuwamy indeks **amazon**:

    :::bash
    curl -XDELETE 'http://127.0.0.1:9200/amazon'

albo, za jednym razem usuwamy **wszystkie** indeksy
(zalecana ostrożność):

    :::bash
    curl -XDELETE 'http://127.0.0.1:9200/_all'

Na koniec, zapytanie o zdrowie klastra ElasticSearch:

    :::bash
    curl 'http://127.0.0.1:9200/_cluster/health?pretty=true'


### Zapytania, korzystające z JSON Query Language

Zaczynamy od zapisania kilku JSON–ów w ElasticSearch:

    :::bash
    curl -XPUT 'http://localhost:9200/twitter/users/kimchy' -d '
    {
       "name" : "Shay Banon"
    }'

    curl -XPUT 'http://localhost:9200/twitter/tweets/1' -d '
    {
       "user": "kimchy",
       "postDate": "2009-11-15T13:12:00",
       "message": "Trying out Elastic Search, so far so good?"
    }'

    curl -XPUT 'http://localhost:9200/twitter/tweets/2' -d '
    {
       "user": "kimchy",
       "postDate": "2009-11-15T14:12:12",
       "message": "Another tweet, will it be indexed?"
    }'

Sprawdzamy, co zostało dodane:

    :::bash
    curl 'http://localhost:9200/twitter/users/kimchy?pretty=true'
    curl 'http://localhost:9200/twitter/tweets/1?pretty=true'
    curl 'http://localhost:9200/twitter/tweets/2?pretty=true'

Dla przypomnienia, interpretacja uri w zapytaniach do ElasticSearch:

<pre>http://localhost:9200/_search?...
http://localhost:9200/<b>⟨index⟩</b>/_search?...
http://localhost:9200/<b>⟨index⟩</b>/<b>⟨type⟩</b>/_search?...
</pre>

Teraz odpytamy indeks *twitter* korzystając
z ElasticSearch *JSON query language*:

    :::bash
    curl 'http://localhost:9200/twitter/tweets/_search?pretty=true' -d '
    {
       "query" : {
          "text" : { "user": "kimchy" }
       }
    }'
    curl 'http://localhost:9200/twitter/tweets/_search?pretty=true' -d '
    {
       "query" : {
          "term" : { "user": "kimchy" }
       }
    }'

Jaka jest różnica między wyszukiwaniem z **text** a **term**?

Wyciągamy wszystkie dokumenty z indeksu *twitter*:

    :::bash
    curl 'http://localhost:9200/twitter/_search?pretty=true' -d '
    {
        "query" : {
            "matchAll" : {}
        }
    }'

Albo – dokumenty typu *user* z indeksu *twitter*:

    :::bash
    curl -XGET 'http://localhost:9200/twitter/users/_search?pretty=true' -d '
    {
        "query" : {
            "matchAll" : {}
        }
    }'


## Indeksy i typy *Multi Tenant*

*Tenant* to najemca, dzierżawca. *Multi tenant* – jak to przetłumaczyć?
Czy poniższy przykład coś wyjaśna?

    :::bash
    curl -XPUT 'http://localhost:9200/bilbo/info/1' -d '{ "name" : "Bilbo Baggins" }'
    curl -XPUT 'http://localhost:9200/frodo/info/1' -d '{ "name" : "Frodo Baggins" }'

    curl -XPUT 'http://localhost:9200/bilbo/tweets/1' -d '
    {
        "user": "bilbo",
        "postDate": "2009-11-15T13:12:00",
        "message": "Trying out Elastic Search, so far so good?"
    }'
    curl -XPUT 'http://localhost:9200/frodo/tweets/1' -d '
    {
        "user": "frodo",
        "postDate": "2009-11-15T14:12:12",
        "message": "Another tweet, will it be indexed?"
    }'

Wyszukiwanie „multi”, po kilku indeksach:

    :::bash
    curl -XGET 'http://localhost:9200/bilbo,frodo/_search?pretty=true' -d '
    {
        "query" : {
            "matchAll" : {}
        }
    }'


<blockquote>
 {%= image_tag "/images/daniel_kahneman.jpg", :alt => "[Daniel Kahneman]" %}
 <p><b>The hallo effect</b> helps keep explanatory narratives
  simple and coherent by exaggerating the consistency
  of evaluations: good people do only good things
  and bad people are all bad.
 </p>
 <p class="author">— Daniel Kahneman</p>
</blockquote>

## ElasticSearch & Rails

Takie eksperymentowanie z ElasticSearch rest API pozwala sprawdzić,
czy dobrze je rozumiemy oraz czy poprawnie zainstalowaliśmy sam program.
Dalsze takie próby są nużące i wyniki są nie są zajmujące.

Dlatego, do następnych prób użyjemy większej liczby rzeczywistych
i zajmujących danych. Naszymi danymi będą statusy z Twittera.
Dodatkowo do odfiltrowania interesujących nas statusów
skorzystamy z [stream API](https://dev.twitter.com/docs/streaming-api):

    :::ruby tracking
    track=wow,rails,mongodb,couchdb,redis,elasticsearch,neo4j

(Dlaczego filtrujemy? Co sekundę wysyłanych jest do Twittera ok. 1000
nowych statusów. Większość z nich nie ma dla nas żadnego znaczenia.)

Do słów kluczowych dopisaliśmy **wow**. Słówo to pojawia się
w wielu statusach, a pozostałe słowa występują z rzadka.
Po dodaniu **wow** nowe statusy będą się pojawiać co chwila.
**Wow!**

Najprościej będzie zacząć od pobrania statusów za pomocą programu *curl*:

    :::bash
    curl -X POST https://stream.twitter.com/1/statuses/filter.json -d @tracking \
      -uAnyTwitterUser:Password

Jak widać statusy zawierają wiele pól i tylko kilka z nich zawiera
interesujące dane. Niestety, na konsoli trudno czytać interesujące nas fragmenty.
Są one wymieszane z jakimiś technicznymi rzeczami, np.
*profile_sidebar_fill_color*, *profile_use_background_image* itp.

Dlatego, przed wypisaniem statusu na ekran, powinniśmy go „oczyścić”
ze zbędnych pól. Zrobimy to za pomocą skryptu w Ruby. W skrypcie
skorzystamy z następujących gemów, które najpierw zainstakujemy:

    gem install tweetstream  yajl-ruby

{%= image_tag "/images/twitter_elasticsearch.jpeg", :alt => "[Twitter -> ElasticSearch]" %}

Zaczniemy od skryptu działającego podobnie do polecenia z *curl* powyżej.

Aby uzyskać dostęp do stream API wymagana jest weryfikacja<br>
(opcja *-uAnyTwitterUser:Password* w poleceniu *curl* powyżej).

    :::ruby nosql-tweets.rb
    # encoding: utf-8

    require 'yajl/json_gem'
    require 'tweetstream'

    user, password = ARGV

    unless (user && password)
      puts "\nUsage:\n\t#{__FILE__} <AnyTwitterUser> <Password>\n\n"
      exit(1)
    end

    TweetStream.configure do |config|
      config.username = user
      config.password = password
      config.auth_method = :basic  # OAuth or HTTP Basic Authentication is required
      config.parser = :yajl
    end

    def handle_tweet(s)
      puts "#{JSON.pretty_generate(s)}"
    end

    client = TweetStream::Client.new

    client.on_error do |message|
      puts message
    end

    client.track('rails', 'mongodb', 'couchdb', 'redis', 'neo4j', 'elasticsearch') do |status|
      handle_tweet status
    end


<blockquote>
<p>
  <h2><a href="https://dev.twitter.com/docs/streaming-api/concepts#access-rate-limiting">Important note</a></h2>
  <p>Each account may create only one standing connection to the
  Streaming API. Subsequent connections from the same account may
  cause previously established connections to be
  disconnected. Excessive connection attempts, regardless of success,
  will result in an automatic ban of the client's IP
  address. Continually failing connections will result in your IP
  address being blacklisted from all Twitter access.
</p>
</blockquote>

Skrypt ten uruchamiamy na konsoli w następujący sposób:

    :::bash
    ruby nosql-tweets.rb MyTwitterUserName MyPassword

Wpatrując się w ekran przez jakiś czas zaczynamy
dostrzegać pole zawierające interesujące dane:

    id
    text
    created_at
    user:
      screen_name
    entities:      # tylko trzy tablice?
      urls (url)
      user_mentions (id_str, name, screen_name)
      hashtags (text)

Czyszczeniem zajmiemy się w kodzie metody *handle_tweet*:

    :::ruby
    def handle_tweet(s)
      h = { }
      h[:id] = s[:id]
      h[:text] =  s[:text]
      h[:screen_name] = s[:user][:screen_name]
      h[:entities] = s[:entities]
      h[:created_at] = Time.parse(s[:created_at])
      puts "#{JSON.pretty_generate(h)}"
    end

Po wymianie na nowy kodu *handle_tweet* i ponownym uruchomieniu skryptu
widzimy efekty. To się da czytać!

Teraz zabierzemy się za zapisywanie oczyszczonych statusów w ElasticSearch.
Skorzystamy z gemu *Tire*:

    gem install tire ansi

Zaczniemy od definicji modelu *Status*.

**TODO** Od razu **percolate**?
Let's define callback for percolation.
Whenewer a new document is saved in the index, this block will be executed,
and we will have access to matching queries in the `Status#matches` property.
In our case, we will just print the list of matching queries.

    :::ruby percolate-nosql-tweets.rb
    require 'tire'
    require 'ansi/code'

    include ANSI::Code

    class Status
      include Tire::Model::Persistence

      property :id
      property :text
      property :screen_name
      property :created_at
      property :entities

      on_percolate do
        puts green { "'#{text}' from @#{bold { screen_name }}" } unless matches.empty?
      end
    end

    # First, let's define the query_string queries.
    q = {}
    q[:rails] = 'rails'
    q[:mongodb] = 'mongodb'
    q[:redis] = 'redis'
    q[:couchdb] = 'couchdb'
    q[:neo4j] = 'neo4j'
    q[:elasticsearch] = 'elasticsearch'

    Status.index.register_percolator_query('rails') { |query| query.string q[:rails] }
    Status.index.register_percolator_query('mongodb') { |query| query.string q[:mongodb] }
    Status.index.register_percolator_query('redis') { |query| query.string q[:redis] }
    Status.index.register_percolator_query('couchdb') { |query| query.string q[:couchdb] }
    Status.index.register_percolator_query('neo4j') { |query| query.string q[:neo4j] }
    Status.index.register_percolator_query('elasticsearch') { |query| query.string q[:elasticsearch] }

    # Finally, refresh the `_percolator` index for immediate access.
    Tire.index('_percolator').refresh

Podmieniamy kod *handle_tweet*.
Strip off fields we are not interested in.

    :::ruby percolate-nosql-tweets.rb
    def handle_tweet(s)
      h = Status.new :id => s[:id],
        :text => s[:text],
        :screen_name => s[:user][:screen_name],
        :entities => s[:entities],
        :created_at => Time.parse(s[:created_at])

      types = h.percolate
      puts cyan { "matched queries: #{types}" }

      types.to_a.each do |type|
        Status.document_type type
        h.save
      end
    end

Podłączamy się do strumienia ze statusami.

    :::ruby percolate-nosql-tweets.rb
    client = TweetStream::Client.new

    client.on_error do |message|
      puts message
    end

    client.track('rails', 'mongodb', 'couchdb', 'redis', 'neo4j', 'elasticsearch') do |status|
      handle_tweet status
    end

Działanie skryptu kończymy wciskając klawisze `CTRL+C`.

Tak możemy sprawdzić ile statusów jest w bazie.
Wyświetlimy też dwa ostatnio zaindeksowane statusy oraz
wszystkie statusy typu *elasticsearch*.

    :::bash
    curl 'http://localhost:9200/statuses/_count'
    curl 'http://localhost:9200/statuses/_search?q=*&sort=created_at:desc&size=2&pretty=true'
    curl 'http://localhost:9200/statuses/_search?size=2&sort=created_at:desc&pretty=true'
    curl 'http://localhost:9200/statuses/_search?_all&sort=created_at:desc&pretty=true'

Faceted search:

    curl -X POST "http://localhost:9200/statuses/_count?q=couchdb&pretty=true"
    curl -X POST "http://localhost:9200/statuses/_search?pretty=true" -d '
    {
      "query" : { "query_string" : {"query" : "couchdb"} },
      "facets" : { "hashtags" : { "terms" :  { "field" : "entities.hashtags.text" } } }
    }'


### Nieco linków

Linki do dokumentacji:

* [eventmachine](https://github.com/eventmachine/eventmachine)
* [em-http-request](https://github.com/igrigorik/em-http-request)
* [tweetstream](https://github.com/intridea/tweetstream)
* [yajl-ruby](https://github.com/brianmario/yajl-ruby)

Zobacz też:

* [Consuming the Twitter Streaming API](http://adam.heroku.com/past/2010/3/19/consuming_the_twitter_streaming_api/) –
  prościej, bez *TweetStream*
* [Gmail & ES](http://ephemera.karmi.cz/post/5510326335/gmail-elasticsearch-ruby)


## Prosta aplikacja Rails do przeszukiwania statusów

Po zaimportowaniu większej liczby statusów…

**TODO**


# Rivers allows to index streams

Zamiast samemu pisać kod do pobierania statusów z Twittera możemy
użyć do tego celu wtyczki *river-twitter*.

Instalacja wtyczek *rivers* jest prosta:

    :::bash
    bin/plugin -install river-twitter
      -> Installing river-twitter...
      Trying http://elasticsearch.googlecode.com/svn/plugins/river-twitter/elasticsearch-river-twitter-0.18.5.zip...
      ...
    bin/plugin -install river-couchdb
      -> Installing river-couchdb...
      Trying http://elasticsearch.googlecode.com/svn/plugins/river-couchdb/elasticsearch-river-couchdb-0.18.5.zip...
      Downloading ...DONE
      Installed river-couchdb
    bin/plugin -install river-wikipedia
      -> Installing river-wikipedia...
      Trying http://elasticsearch.googlecode.com/svn/plugins/river-wikipedia/elasticsearch-river-wikipedia-0.18.5.zip...
      Downloading ......DONE
      Installed river-wikipedia

Repozytoria z kodem wtyczek są na Githubie [tutaj](https://github.com/elasticsearch).

**Uwaga**: po instalacji wtyczki, należe zrestartować *ElasticSearch*.


### River Twitter

Usuwanie swoich rivers, na przykład:

    :::bash
    curl -XDELETE http://localhost:9200/_river/my_twitter_river

Przykład tzw. *filtered stream*:

    :::bash
    curl -XPUT localhost:9200/_river/my_twitter_river/_meta -d @tweets-nosql.json

gdzie w pliku *nosql-tweets.json* wpisałem:

    :::json tweets-nosql.json
    {
        "type" : "twitter",
        "twitter" : {
            "user" : "wbzyl",
            "password" : "sekret",
            "filter": {
               "tracks": ["elasticsearch", "mongodb", "couchdb", "rails"]
            }
        },
        "index" : {
            "index": "tweets",
            "type" : "nosql",
            "bulk_size" : 10
        }
    }

Sprawdzanie statusu:

    :::bash
    curl -XGET http://localhost:9200/_river/my_twitter_river/_status?pretty=true
    {
      "_index" : "_river",
      "_type" : "my_twitter_river",
      "_id" : "_status",
      "_version" : 5,
      "exists" : true,
      "_source" : {"ok":true,
         "node":{"id":"aUJLtb_KSZibfW3IG9P8yQ","name":"Nobilus","transport_address":"inet[/192.168.4.4:9300]"}}

A tak raportowane jest pobranie paczki z 10 tweets na konsoli:

    [2011-12-16 12:54][INFO ][twitter4j.TwitterStreamImpl] Establishing connection.
    [2011-12-16 12:54][INFO ][cluster.metadata           ] [Hazard] [_river] update_mapping [my_rivers] (dynamic)
    [2011-12-16 12:54][INFO ][twitter4j.TwitterStreamImpl] Connection established.
    [2011-12-16 12:54][INFO ][twitter4j.TwitterStreamImpl] Receiving status stream.
    [2011-12-16 12:57][INFO ][cluster.metadata           ] [Hazard] [tweets] update_mapping [nosql] (dynamic)

Wyszukiwanie:

    :::bash
    curl 'http://localhost:9200/tweets/nosql/_search?q=text:mongodb&fields=user.name,text&pretty=true'
    curl 'http://localhost:9200/tweets/nosql/_search?pretty=true' -d '
    {
        "query" : {
            "match_all" : { }
        }
    }'

Sprawdzamy mapping:

    :::bash
    curl 'http://localhost:9200/tweets/_mapping?pretty=true'

Oto mapowanie:

    :::json
    {
      "tweets" : {
        "nosql" : {
          "properties" : {
            "text" : {
              "type" : "string"
            },
            "source" : {
              "type" : "string"
            },
            "location" : {
              "type" : "geo_point"
            },
            "link" : {
              "dynamic" : "true",
              "properties" : {
                "start" : {
                  "type" : "long"
                },
                "expand_url" : {
                  "type" : "string"
                },
                "display_url" : {
                  "type" : "string"
                },
                "url" : {
                  "type" : "string"
                },
                "end" : {
                  "type" : "long"
                }
              }
            },
            "hashtag" : {
              "dynamic" : "true",
              "properties" : {
                "text" : {
                  "type" : "string"
                },
                "start" : {
                  "type" : "long"
                },
                "end" : {
                  "type" : "long"
                }
              }
            },
            "retweet_count" : {
              "type" : "long"
            },
            "created_at" : {
              "format" : "dateOptionalTime",
              "type" : "date"
            },
            "mention" : {
              "properties" : {
                "id" : {
                  "type" : "long"
                },
                "start" : {
                  "type" : "long"
                },
                "name" : {
                  "type" : "string"
                },
                "screen_name" : {
                  "index" : "not_analyzed",
                  "type" : "string"
                },
                "end" : {
                  "type" : "long"
                }
              }
            },
            "in_reply" : {
              "properties" : {
                "user_screen_name" : {
                  "index" : "not_analyzed",
                  "type" : "string"
                },
                "status" : {
                  "type" : "long"
                },
                "user_id" : {
                  "type" : "long"
                }
              }
            },
            "truncated" : {
              "type" : "boolean"
            },
            "place" : {
              "dynamic" : "true",
              "properties" : {
                "id" : {
                  "type" : "string"
                },
                "name" : {
                  "type" : "string"
                },
                "type" : {
                  "type" : "string"
                },
                "country_code" : {
                  "type" : "string"
                },
                "url" : {
                  "type" : "string"
                },
                "full_name" : {
                  "type" : "string"
                },
                "country" : {
                  "type" : "string"
                }
              }
            },
            "user" : {
              "properties" : {
                "id" : {
                  "type" : "long"
                },
                "location" : {
                  "type" : "string"
                },
                "description" : {
                  "type" : "string"
                },
                "name" : {
                  "type" : "string"
                },
                "screen_name" : {
                  "index" : "not_analyzed",
                  "type" : "string"
                }
              }
            }
          }
        }
      }


# Rails — Tire & ElasticSearch

* Rails application template.
* Dodać klasę *Tweet* i podłączyć ją do Twitter River.

JSON:

    :::json tweets-nosql.json
    {
        "type" : "twitter",
        "twitter" : {
            "user" : "me",
            "password" : "secret",
            "filter": {
               "tracks": ["elasticsearch", "mongodb", "couchdb", "rails"]
            }
        },
        "index" : {
            "index": "tweets",
            "type" : "nosql",
            "bulk_size" : 20
        }
    }

Twitter River:

    curl -XPUT localhost:9200/_river/my_twitter_river/_meta -d @tweets-nosql.json

Routing:

    :::ruby config/routes.rb
    # get "tweets/index"
    match '/tweets' => 'tweets#index', :as => :tweets

Model:

    :::ruby app/models/tweet.rb
    class Tweet
      def self.search(params)
        Tire.search('tweets', type: 'nosql') do
          size 6
          if params[:page].present?
            from ((params[:page].to_i - 1) * 6)
          end
          query do
            boolean do
              must { string params[:q] } if params[:q].present?
            end
          end
        end.results
      end
    end

Kontroler:

    :::ruby app/controllers/tweets_controller.rb
    class TweetsController < ApplicationController
      def index
        @tweets = Tweet.search(params)
      end
    end

Widok:

    :::rhtml app/views/tweets/index.html.erb
    <h1>Listing tweets</h1>

    <%= form_tag tweets_path, method: :get do %>
      <p>
        <%= text_field_tag :q, params[:q] %>
        <%= submit_tag 'Search', name: nil %>
      </p>
    <% end %>

    <%= will_paginate @tweets %>

    <% @tweets.each do |tweet| %>
    <p><%= tweet.text %>
    <% end %>


# Ruby — Tire + ElasticSearch

Instalujemy gemy – *tire* i *yajl-ruby*:

    gem install tire
    gem install yajl-ruby

Hurtowe indeksowanie (*bulk indexing*):

    :::ruby quotes.rb
    require 'tire'
    fortunes = [
      { id: 1, type: 'quotes', text: "Jedną z cech głupstwa jest logika.", tags: ["logika", "głupstwo", "nauka"] },
      { id: 2, type: 'quotes', text: "Znasz hasło do swojego wnętrza?", tags: ["hasło", "głupstwo", "czas"] },
      { id: 3, type: 'quotes', text: "Miał lwi pazur, ale brudny.", tags: ["lew", "pazur", "nauka"] },
      { id: 4, type: 'quotes', text: "Unikaj skarżącego się na brak czasu, chce ci zabrać twój.", tags: ["nauka", "czas"] }
    ]
    Tire.index 'fortunes' do
      delete
      import fortunes
      refresh
    end

*Uwaga:* pola *id* oraz *type* są obowiązkowe.

**TODO**: doc/elasticsearch/Tire.md


# Zadania

1\. Zainstalować wtyczkę *Wikipedia River*. Wyszukiwanie?

2\. Przeczytać [Creating a pluggable REST endpoint](http://www.elasticsearch.org/tutorials/2011/09/14/creating-pluggable-rest-endpoints.html).

* Zainstalować wtyczkę [hello world](https://github.com/brusic/elasticsearch-hello-world-plugin/).
* Napisać swoją wtyczkę.
