---
layout: post
published: true
title:  "Budowanie wydajnego backendu aplikacji mobilnej wykorzystującej geolokalizację i dane przestrzenne (demo wykorzystania PostGIS i GeoDjango)."
date:   2014-12-14 14:05:00
categories: django devops tools postgres postgis
---

WSTĘP
-----

W tym wpisie pokaże Wam jak przygotować fragment backendu do aplikacji mobilnej wykorzstującej dane geoprzestrzenne. Zamiast demonstrować sztampowy przykład jak znajdowanie np. kawiarni, czy knajp w okolicy spróbujemy napisać kawałek backendu aplikacji typu [Tinder](http://en.wikipedia.org/wiki/Tinder_(application)).

TECHNOLOGIE
-----------
Technologie z których skorzystamy:

* **GeoDjango** (opakowujące zgrabnie bibliotekę GEOS)
* **PostGIS** (rozszerzenie do bazy danych Postgres do GIS)
* **Django REST Framework** (biblioteka wspierająca budowanie REST-owych API)

DLACZEGO PostGIS?
-----------------

Teoretycznie rzecz biorąc moglibyśmy przechowywać w właściwie dowolnej bazie współrzędne i wykonywać kwerendę typu find-nearby korzystając z uiwersalnej formuły [Haversine](http://en.wikipedia.org/wiki/Haversine_formula). Poniżej prezentuje przykłady konfrontujące djangowy widok wykorzystujący raw_sql na bazie MySQL vs z djangowym widokiem z użyciem GeoDjango:

{% highlight python %}
def nearby_spots_haversine(request, lat, lng, radius=5000, limit=50):
    """
    WITHOUT use of any external library, using raw MySQL and Haversine Formula
    http://en.wikipedia.org/wiki/Haversine_formula
    """
    radius = float(radius) / 1000.0
    query = """SELECT id, (6367*acos(cos(radians(%2f))
               *cos(radians(latitude))*cos(radians(longitude)-radians(%2f))
               +sin(radians(%2f))*sin(radians(latitude))))
               AS distance FROM demo_spot HAVING
               distance < %2f ORDER BY distance LIMIT 0, %d""" % (
        float(lat),
        float(lng),
        float(lat),
        radius,
        limit
    )
    queryset = Spot.objects.raw(query)
    serializer = SpotWithDistanceSerializer(queryset, many=True)

    return JSONResponse(serializer.data)

{% endhighlight %}

vs:

{% highlight python %}
def nearby_spots_postgis(request, lat, lng, radius=5000, limit=50):
    """
    WITH USE OF GEODJANGO and POSTGIS
    https://docs.djangoproject.com/en/dev/ref/contrib/gis/db-api/#distance-queries
    """
    user_location = fromstr("POINT(%s %s)" % (lng, lat))
    desired_radius = {'m': radius}
    nearby_spots = Spot.objects.filter(
        mpoint__distance_lte=(user_location, D(**desired_radius))).distance(
        user_location).order_by('distance')[:limit]
    serializer = SpotWithDistanceSerializer(nearby_spots, many=True)

    return JSONResponse(serializer.data)
{% endhighlight %}

Ostatecznie moglibyśmy też być dużo bardziej minimalistyczni w podejściu i pomijąc takie niuanse jak krzywizna ziemi (co w konkretnych przypadakch - szukanie punktów w promieniu kilku kilometrów nie wiąże się ze znacznym błędem) podejść do tematu np. tak jak w pokazanej przez autora [odpowiedzi na SO](http://stackoverflow.com/questions/17682201/how-to-filter-a-django-model-with-latitude-and-longitude-coordinates-that-fall-w/21429344#21429344) czyli zwyczajnie odfiltrować obiekty z bazy wpadające w pewien przedział szerokości i długości geograficznej, którcyh przecięcia wyznaczają kwadrat (na poziomie filtrowania bazy danych) i zwrócić punkty należący wyłącznie do okręgu wpisanego w ten kwadrat (dalsze odfiltrowanie już w kodzie pythona).

Rozwiązanie z pisaniem gołych SQL-i w Django wydaje się niezbyt eleganckie, a ich używanie  jest uznane za złą praktykę w Django. Co więcej przyklejamy się na stałe do jednej bazy i tracimy łatwość przesiadki na inną co daje nam ORM, z drugiej strony wybór GeoDjango to też pewne zawężenie w kontekście wyboru bazy - mamy do dyspozycji następujące backendy:

* django.contrib.gis.db.backends.postgis,
* django.contrib.gis.db.backends.mysql,
* django.contrib.gis.db.backends.oracle,
* django.contrib.gis.db.backends.spatialite,

ale za to dzięki wyboru GeoDjango dostajemy możliwość uniknięcia pisania raw-SQL-i, oraz dostajemy całe bogactwo klas i metod do działania na danych przestrzennych, o których kilka słów dalej.

Wspomniane backendy bazodanowe w różnym stopniu wspierają poszczególne funckjonalności. Porównanie dostępności funckji można znaleźć [tutaj](https://docs.djangoproject.com/en/dev/ref/contrib/gis/db-api/#compatibility-tables).

**Dlaczego wybieramy PostGIS?**

Bo ma najszersze wsparcie dla funkcjonalności GeoDjango i pozostajemy przy naszym ulubionym do djangodevelopmentu Postgresie.

GeoDjango - BOGACTWO
--------------------

Jak już wspomnieliśmy dzięki skorzystaniu z GeoDjango otrzymujemy dostęp do wielu ciekawych klas i metod do operowania na danych przestrzennych, które postaram się w tej części pokrótce omówić.

[Geometry Field Types](https://docs.djangoproject.com/en/dev/ref/contrib/gis/model-api/#django.contrib.gis.db.models.GeometryField):

Zacznijmy od dostępnych typów danych do przechowywania:

- **PointField** - przechowuje standardowe współrzędne (długość, szerokośc) geograficzną. Być może Tobie również ta kolejność inicjalizowania punktu: najpierw długość (longitude) a potem szerokość (latitude) nie wydaje się naturalna, ale to właśnie kolejność jakiej oczekuje od nas GeoDjango (łatwo o banalaną pomyłkę). 

Floaty reprezentujące wartości są przechowywane w dwuelementowej krotce dostępnej przez:

	nazwa_pola_typu_point.coords

możliwe sposby inicjalizacji:

standardowy:

	last_location = Point(21.006841063502, 52.245934009551)

ze stringa:

	last_location = 'POINT(21.006841063502, 52.245934009551)'

- **PolygonField** - umożliwia przechowywanie wielokątów. Idealny do zaznaczania obszarów w przestrzeni.

- **LineStringField** - przechowuje punkty połączone linią. Idealny do zaznaczania dróg, scieżek, tras na mapach. 

- **MultiPointField** - struktura, która przechowuje wiele nie powiązanych punktów.

- **MultiLineStringField** - struktura do przechowywania 0 lub więcej obiektów typu LineStringField

- **MultiPolygonField** - struktura do przechowywania 0 lub więcej obiektów typu PolygonField.

- **GeometryCollectionField** - kolekcja do przechocywania obiektów zróżnicowanego typu (Poly, points, etc).

Warto zauważyć, że rejestrując w djangowym adminie model zawierający geoprzestrzenne typy danych otrzymujemy wsparcie w postaci prostych mapowych widgetów na których możemy zaznaczyć punkt, wielokąt, krzywą. Niestety defaultowo backend ten nie wykorzystuje Google Maps, a mapy dla Polski są dość biedne. Istniej możliwość zastąpienia defaultowych map, mapami Google używając np. tej biblioteki [django-google-maps](https://pypi.python.org/pypi/django-google-maps/0.2.1)

![GeoDjango widget for picking point in admin](/assets/django-admin-pick-point-for-geodjango.png)

Kolejnym ważnym elementem są zasoby modułu: `django.contrib.gis.measure` umożliwiające dokonywanie wszelkiego rodzaju pomiarów odległości pomiędzy punktami lub innymi obiektami. Klasa [Distance](https://docs.djangoproject.com/en/dev/ref/contrib/gis/measure/#django.contrib.gis.measure.Distance) (dostępna również przez alias D) pozwala ponadto na łatwą manipulację jednostkami miary odległości ([wspierane miary](https://docs.djangoproject.com/en/dev/ref/contrib/gis/measure/#supported-units)). W module znajdziemy również klasę [Area](https://docs.djangoproject.com/en/dev/ref/contrib/gis/measure/#area) niezbędną wszędzie tam gdzie będziemy chcieli ustalać wielkość powierzchni.

[GeoQuerySet API Reference](https://docs.djangoproject.com/en/dev/ref/contrib/gis/geoquerysets/#geoqueryset-api-reference):

GeoDjango docenimy na prawdę korzystając z [spatial lookups](https://docs.djangoproject.com/en/dev/ref/contrib/gis/geoquerysets/#spatial-lookups) i [distance lookups](https://docs.djangoproject.com/en/1.7/ref/contrib/gis/geoquerysets/#distance-lookups) umożliwiających bardzo bogate możliwości filtrowania danych geoprzestrzennych. Poniżej krótka lista i możliwości poszczególnych metod aplikowalnych do pól klasy [GeometryField](https://docs.djangoproject.com/en/dev/ref/contrib/gis/model-api/#geometryfield):

dla jasności przykładów załóżmy, że mamy następująco zdefiniowaną klasę reprezentującą obszary kodu pocztowego w postaci wielokątu (PolygonField):

    from django.contrib.gis.db import models

    class Zipcode(models.Model):
        code = models.CharField(max_length=5)
        poly = models.PolygonField()
        objects = models.GeoManager()

a) **wyszukiwanie przestrzenne operujące na ustalaniu relacji wzajemnego położenia obiektów** ([spatial lookups](https://docs.djangoproject.com/en/dev/ref/contrib/gis/geoquerysets/#spatial-lookups)):

* **bbcontains** - sprawdzenie czy obwiednia obiektu (tu: poly) całkowicie zawiera obwiednie obiektu zadanego jako argument (tu: geom)

      Zipcode.objects.filter(poly__bbcontains=geom)

* **bboverlaps** - sprawdzenie czy obiekty mają część wspólną

      Zipcode.objects.filter(poly__bboverlaps=geom)


* **contained** - sprawdzenie czy obwiednia obiektu zaiwera się w całości w obwiedni obiektu zadanego jako argument

      Zipcode.objects.filter(poly__contained=geom)

* **contains** - sprawdza czy obiekt przestrzenie zawiera obiekt zadany jako argument. Używany jest ST_contains z PostGIS poniżej przykłady i link do dokumentacji. ![Przyklady dla ST_Contains](/assets/ST_Contains_examples.png) źródło: [http://postgis.refractions.net/documentation/manual-1.4/ST_Contains.html](http://postgis.refractions.net/documentation/manual-1.4/ST_Contains.html)


      Zipcode.objects.filter(poly__contains=geom)


* **contains_properly** - zwraca true tylko gdy dla zadanego jako argument obiektu mamy do czynienia z zawieraniem się we wnętrzu obiektu ale nie jego krawędziach.

      Zipcode.objects.filter(poly__contains_properly=geom)

* **coverdby** - sprawdza czy żaden punkt zadanego obiektu (geom) nie leży poza przeszukiwanym obiektem (poly)

      Zipcode.objects.filter(poly__coveredby=geom)

* **covers** - sprawdza czy żaden punkt przeszukiwanego obiektu (poly) nie jest poza zadanym obiektem (geom)

      Zipcode.objects.filter(poly__covers=geom)

* **crosses** - sprawdza czy przeszukiwany obiekt (poly) przecina zadany obiekt (geom)

      Zipcode.objects.filter(poly__crosses=geom)

* **disjoint** - sprawdza czy obiekty są przestrzennie rozłączne

      Zipcode.objects.filter(poly__disjoint=geom)

* **equals**

* **exact, same_as**

* **intersects** - sprawdza czy obiekty mają przestrzenie część wspólną

      Zipcode.objects.filter(poly__intersects=geom)

* **overlaps**

* **relate** - sprawdza czy obiekt jest przestrzennie powiązany z innym obiektem dla zadanego wzorca.

* **touches** - sprawdza czy obiekt przestrzennie posiada kontakt z innym obiektem

      Zipcode.objects.filter(poly__touches=geom)

* **within** - sprawdza czy obiekt przestrzennie zawiera się w innym obiekcie

      Zipcode.objects.filter(poly__within=geom)

* **left** - sprawdza czy obwiednia obiektu jest dokładnie na lewo od obwiedni innego obiektu.

      Zipcode.objects.filter(poly__left=geom)

* **right** - sprawdza czy obwiednia obiektu jest dokładnie na prawo od obwiedni innego obiektu.

      Zipcode.objects.filter(poly__right=geom)

* **overlaps_left** - sprawdza czy obwiednia obiektu nakłada się lub jest na lewo od obwiedni innego obiektu.

      Zipcode.objects.filter(poly__overlaps_left=geom)

* **overlaps_right** - sprawdza czy obwiednia obiektu nakłada się lub jest na prawo od obwiedni innego obiektu.

      Zipcode.objects.filter(poly__overlaps_right=geom)

* **overlaps_above** - sprawdza czy obwiednia obiektu nakłada się lub jest powyżej obwiedni innego obiektu.

      Zipcode.objects.filter(poly__overlaps_above=geom)

* **overlaps_below** - sprawdza czy obwiednia obiektu nakłada się lub jest poniżej obwiedni innego obiektu.

      Zipcode.objects.filter(poly__overlaps_below=geom)

* **strictly_above** - sprawdza czy obwiednia obiektu jest dokładnie powyżej obwiedni innego obiektu.

      Zipcode.objects.filter(poly__strictly_above=geom)

* **strictly_below** - sprawdza czy obwiednia obiektu jest dokładnie poniżej obwiedni innego obiektu.

      Zipcode.objects.filter(poly__strictly_below=geom)

b) **wyszukiwanie oparujące na odlełgości pomiędzy obiektami** ([distance lookups](https://docs.djangoproject.com/en/1.7/ref/contrib/gis/geoquerysets/#distance-lookups)):

* **distance_gt** - zwraca obiekty dla których dystans do zadanego jako argument wyszukiwania obiektu jest **większy** niż zadana wartość.

      Zipcode.objects.filter(poly__distance_gt=(geom, D(m=5)))

* **distance_gte** - zwraca obiekty dla których dystans do zadanego jako argument wyszukiwania obiektu jest **większy lub równy** niż zadana wartość.

      Zipcode.objects.filter(poly__distance_gte=(geom, D(m=5)))

* **distance_lt** - zwraca obiekty dla których dystans do zadanego jako argument wyszukiwania obiektu jest **mniejszy** niż zadana wartość.

      Zipcode.objects.filter(poly__distance_lt=(geom, D(m=5)))

* **distance_lte** - zwraca obiekty dla których dystans do zadanego jako argument wyszukiwania obiektu jest **mniejszy lub równy** niż zadana wartość.

      Zipcode.objects.filter(poly__distance_lte=(geom, D(m=5)))

* **dwithin** - zwraca obiekty dla których odległość od zadanego jako argment wyszukiwaniu obiektu jest w zasięgu zadanego **D**ystansu.

      Zipcode.objects.filter(poly__dwithin=(geom, D(m=5)))

**dwithin** w istocie działa bardzo podobnie do **distance_lt** ale zwracają różne SQL i mają rózną efektywność (dwithin mocniej korzysta z geoindeksów przez co jest szybszy)- więcej można przeczytać [tutaj](http://stackoverflow.com/questions/2235043/geodjango-difference-between-dwithin-and-distance-lt) i [tutaj](http://stackoverflow.com/questions/7845133/how-can-i-query-all-my-data-within-a-distance-of-5-meters)



WYMAGAINIA dla naszej demonstracyjnej aplikacji
-----------------------------------------------

 Naszą demonstracyjną aplikację nazwijmy "FuckFinder", co w gruncie rzeczy dość dobrze oddaje filozofię Tindera (Jest to prywatna opinia autora a nie oficjalne stanowisko firmy DaftCode).

W wersji minimalnej chcemy stworzyć:

a) uproszczony model użytkownika aplikacji

b) kluczowy widok odpowiadający za zwracanie osób spełniających kryteria wieku, płci i przedewszystkim bycia w zadanym promieniu (wyrażonym w km) od aktualnej lokalizacji poszukującego oraz zwracający wyliczoną odległość dla każdego z wyników.


MODEL DANYCH:
============

	klasa FuckFinderUser:
	    nickname
	    age
	    sex
	    prefered_sex
	    prefered_age_min
	    prefered_age_max
	    last_location
	    prefered_radius


WIDOK ZWRACAJĄCY OSOBY W PROMIENIU:
===================================

	GET /fetch_fuckfinder_proposals_for/<nick_of_finder>/<current_latitude>/<current_longitiude>/

- dla zadanego user i jego aktualnego położenia

- zwrócone osoby muszą być w promieniu poszukującego (finder) oraz
finder musi być również w promieniu każdej ze zwróconcyh osób

- mniej istotne z punkt widzenia naszego geo-litemotivu: zgodność
preferencji dot. płci, nie zwracanie osób homoseksualnych osobom
heteroseksualnym i odwrotnie, zwracanie osób spełniających
nawzajem kryteria pożądanych przedziałów wieku

Przykładowe zapytanie HTTP:

	curl GET http://127.0.0.1:8000/api/fetch_fuckfinder_proposals_for/andi/52.22967/21.01222/

Przykładowa odpowiedź:
{% highlight json %}
[
  {
    "id": 2,
    "prefered_sex": "M",
    "sex": "F",
    "nickname": "j",
    "age": 22,
    "prefered_age_min": 19,
    "prefered_age_max": 30,
    "last_location": {
      "latitude": 52.255750894546,
      "longitude": 20.990216732027
    },
    "prefered_radius": 15,
    "distance": 0.9902584497169999
  }
]
{% endhighlight %}

INSTALACJE POSTGIS i KONFIGURACJA DJANGO
----------------------------------------

informacja jak na poszczególnych systemach zainstalować postgres znajdziesz tutaj: [http://postgis.net/install/](http://postgis.net/install/)

po zainstalowaniu postgres odpalamy `psql`

tworzymy bazę:

	create database fuckfinder_db;

po połączeniu się z nią:

	\connect fuckfinder_db

dodajemy w JEJ KONTEKśCIE następujące rozszerzenia:

	-- Enable PostGIS (includes raster)
	CREATE EXTENSION postgis;
	-- Enable Topology
	CREATE EXTENSION postgis_topology;
	-- fuzzy matching needed for Tiger
	CREATE EXTENSION fuzzystrmatch;
	-- Enable US Tiger Geocoder
	CREATE EXTENSION postgis_tiger_geocoder;

sprawdzamy wersję POSTGIS:

	postgis_lib_version();

i w postaci krotki dodajemy ją w settingsach django, np.:

	POSTGIS_VERSION = (2, 1, 3)

używamy backendu bazy danych:

	django.contrib.gis.db.backends.postgis

a nie standardowego dla postgresu `django.db.backends.postgresql_psycopg2`

ponad to w `INSTALLED_APPS` musimy dodać:

	django.contrib.gis



IMPLEMENTACJA:
--------------
[models.py](https://github.com/andilabs/fuckfinder/blob/master/api/models.py)
{% highlight python %}
(...)
class FuckFinderUser(models.Model):
    nickname = models.CharField(max_length=250, unique=True)
    age = models.IntegerField(validators=[MinValueValidator(18), MaxValueValidator(130)])
    sex = models.CharField(max_length=1, choices=SEX_CHOICES)
    prefered_sex = models.CharField(max_length=1, choices=SEX_CHOICES)
    prefered_age_min = models.IntegerField(validators=[MinValueValidator(18), MaxValueValidator(130)])
    prefered_age_max = models.IntegerField(validators=[MinValueValidator(18), MaxValueValidator(130)])
    last_location = models.PointField(max_length=40, null=True)
    prefered_radius = models.IntegerField(default=5, help_text="in kilometers")

    objects = models.GeoManager()
(...)
{% endhighlight %}


wersja wstępna [views.py](https://github.com/andilabs/fuckfinder/blob/ca6d0d3b02caef2fd8181c6191368c01ee1eb9b2/api/views.py)
{% highlight python %}
(...)
def fetch_fuckfinder_proposals_for(request, nick_of_x, current_latitude, current_longitiude, limit=10):

    finder = get_object_or_404(FuckFinderUser, nickname=nick_of_x)
    finder_location = Point(float(current_longitiude), float(current_latitude))

    if finder.prefered_sex == finder.sex:
        # deal with homosexual
        candidates = FuckFinderUser.objects.filter(
            prefered_sex=finder.sex,
            sex=finder.prefered_sex,
            age__range=(finder.prefered_age_min, finder.prefered_age_max),
            prefered_age_min__lte=finder.age,
            prefered_age_max__gte=finder.age,
        ).exclude(nickname=finder.nickname)
    else:
        # deal with heterosexual:
        candidates = FuckFinderUser.objects.filter(
            sex=hetero_desires(finder),
            age__range=(finder.prefered_age_min, finder.prefered_age_max),
            prefered_age_min__lte=finder.age,
            prefered_age_max__gte=finder.age,
        ).exclude(sex=F('prefered_sex')).exclude(nickname=finder.nickname)

    # do geo queries
    candidates_inside_finder_radius_and_vice_versa = candidates.filter(
        last_location__distance_lte=(
            finder_location,
            D(km=min(finder.prefered_radius, F('friendly_rate'))))
        ).distance(finder_location).order_by('distance')[:limit]

    serializer = FuckFinderUserListSerializer(
        candidates_inside_finder_radius_and_vice_versa, many=True)

    return JsonResponse(serializer.data, safe=False)
{% endhighlight %}

WYDAJNOŚĆ
=========

By przetestować wydajność API musimy najpierw wygenerować sensowen dane, w tym celu uruchom:

	python manage.py generate_1M_ff_users

[skrypt](https://github.com/andilabs/fuckfinder/blob/master/api/management/commands/generate_1M_ff_users.py) generuje 1M (milion) zrandomizowanych FuckFinderUsers wg następującego schematu:

* **last_location** ustalamy losując wartości dla latitude, longitued z przedziału
	* dla LAT **52.09 - 52.31**
	* dla LNG **20.87 - 21.17**
	![Space on which we generate points](/assets/map-points.png)
* **wiek** wybierany losowo z przedziału **18-55**,
* **do** ustalenia desired_max_age, desired_min_age losujemy liczbę z **1, 2, 3, 5, 8, 13** i odpowiednio dodajemy lub odejmujemy od wieku użytkownika.
* **płeć** wybieramy z równym prawdopodbieństwem
* **orientacje** seksualną (desired_sex) losujemy z rozkładem (**hetero: 0.95, homo: 0.05**)
* **preferowany** promień losujemy z **5, 10, 15, 20, 25, 30**

Deterministycznie tworzymy pojedyńczego użytkownika:

	python manage.py generate_andi_user

Sprawdźmy jaki jest czas odpowiedzi i jak duży objętościowo jest zwrócona JSON dla:

	http://127.0.0.1:8000/api/fetch_fuckfinder_proposals_for/andi/52.22862/21.00195/

Czas odpowiedzi to aż 46 sekund, a wielkość zwróconego JSON-a ponad 20MB. Jest to nieakceptowalne w przypadku urządzeń mobilnych zwłaszcza przy korzystaniu z sieci komórkowej a nie wifi.

Spróbujmy przeanalizować czy kolejność filtrowania ma znaczenie (płeć-wiek, geolokalizacja) vs (geolokalizacja, płeć-wiek). Jesteśmy w stanie oczędzić ok 3 sekund. To niewiele w skali 46 sekund.

Spójrzmy na automatycznie utworzone indeksy:

	fuckfinder_db=# \d+ api_fuckfinderuser
	(...)
	Indexes:
	    "api_fuckfinderuser_pkey" PRIMARY KEY, btree (id)
	    "api_fuckfinderuser_nickname_key" UNIQUE CONSTRAINT, btree (nickname)
	    "api_fuckfinderuser_last_location_id" gist (last_location)
	Has OIDs: no

Spróbujmy dodać indeksy na pozostałe pola:

	fuckfinder_db=# fuckfinder_db=# \d+ api_fuckfinderuser
	(...)
	Indexes:
	    "api_fuckfinderuser_pkey" PRIMARY KEY, btree (id)
	    "api_fuckfinderuser_nickname_key" UNIQUE CONSTRAINT, btree (nickname)
	    "api_fuckfinderuser_age_5da72621f56542a8_uniq" btree (age)
	    "api_fuckfinderuser_last_location_id" gist (last_location)
	    "api_fuckfinderuser_prefered_age_max_4fb7a9bd5f0e8ddc_uniq" btree (prefered_age_max)
	    "api_fuckfinderuser_prefered_age_min_4fb7a10960310f8a_uniq" btree (prefered_age_min)
	    "api_fuckfinderuser_prefered_sex_22dd447fdfd9a3b6_uniq" btree (prefered_sex)
	    "api_fuckfinderuser_sex_774e0911fdfeec21_uniq" btree (sex)
	Has OIDs: no

Okazuje się, że dodając indeksy na wszystkie pola wcale nie poprawiamy wyników, a wręcz przeciwinie. Spróbujmy minimalistycznie ustawić indeksy tam gdzie nie powinny zachodzić zmiany, tzn. płeć i wiek użytkownika.

	fuckfinder_db=# \d+ api_fuckfinderuser
	(...)
	Indexes:
	    "api_fuckfinderuser_pkey" PRIMARY KEY, btree (id)
	    "api_fuckfinderuser_nickname_key" UNIQUE CONSTRAINT, btree (nickname)
	    "api_fuckfinderuser_age_5da72621f56542a8_uniq" btree (age)
	    "api_fuckfinderuser_last_location_id" gist (last_location)
	    "api_fuckfinderuser_sex_774e0911fdfeec21_uniq" btree (sex)
	Has OIDs: no

Widzimy drobną poprawę dla kolejności: najpier płeć-wiek a potem geofiltrowanie, ale wciąż przesyłamy olbrzymiego JSON-a i trwa to ponad 40 sekund.

Co możemy zrobić by poprawić wydajność API? Możemy wprowadzić paginację wyników.

PAGINACJA
=========

Paginacja polega na tym, że zwracamy w API pierwsze k-wyników i link do kolejnej strony wyników. Na kolejnych stronach zwracam link do poprzedniej i następnej jeśli istnieje.

By zaimplementować paginację wykorzystamy dostępny w `django_core.paginator`, jak również `PaginationSerializer` z `Django Rest Framework`.

Pole wyliczone w locie - `distance` z querysetu, jak również pole `last_location` są złożonymi obiektami, dlatego musimy serializerowi powiedzieć wprost jak ma je potraktować. Robimy to w metodzie `to_representation` klasy Serializera.
{% highlight python %}
(...)
class FuckFinderUserListSerializer(serializers.ModelSerializer):
    prefered_sex = serializers.ChoiceField(choices=SEX_CHOICES, default='male')
    sex = serializers.ChoiceField(choices=SEX_CHOICES, default='male')

    class Meta:
        model = FuckFinderUser

    def to_representation(self, instance):
        ret = super(FuckFinderUserListSerializer, self).to_representation(instance)
        pnt = fromstr(ret['last_location'])
        ret['last_location'] = {'longitude': pnt.coords[0], 'latitude': pnt.coords[1]}
        ret['distance'] = instance.distance.km
        return ret


class PaginatedFuckFinderUserListSerializer(PaginationSerializer):

    class Meta:
        object_serializer_class = FuckFinderUserListSerializer

{% endhighlight %}

lepsze wersja [views.py](https://github.com/andilabs/fuckfinder/blob/master/api/views.py) z wykorzystaniem paginacji:

{% highlight python %}
@api_view(['GET', ])
def fetch_fuckfinder_proposals_for(request, nick_of_finder, current_latitude, current_longitiude):

    finder = get_object_or_404(FuckFinderUser, nickname=nick_of_finder)
    finder_location = Point(float(current_longitiude), float(current_latitude))

    candidates = FuckFinderUser.objects.filter(
        last_location__distance_lte=(
            finder_location,
            D(km=min(finder.prefered_radius, F('prefered_radius'))))
        ).distance(finder_location).order_by('distance')

    if finder.prefered_sex == finder.sex:
        # deal with homosexual
        candidates_inside_finder_radius_and_vice_versa = candidates.filter(
            prefered_sex=finder.sex,
            sex=finder.prefered_sex,
            age__range=(finder.prefered_age_min, finder.prefered_age_max),
            prefered_age_min__lte=finder.age,
            prefered_age_max__gte=finder.age,
        ).exclude(nickname=finder.nickname)
    else:
        # deal with heterosexual:
        candidates_inside_finder_radius_and_vice_versa = candidates.filter(
            sex=finder.hetero_desires(),
            age__range=(finder.prefered_age_min, finder.prefered_age_max),
            prefered_age_min__lte=finder.age,
            prefered_age_max__gte=finder.age,
        ).exclude(sex=F('prefered_sex')).exclude(nickname=finder.nickname)

    paginator = Paginator(candidates_inside_finder_radius_and_vice_versa, 20)

    page = request.QUERY_PARAMS.get('page')

    try:
        result = paginator.page(page)
    except PageNotAnInteger:
        result = paginator.page(1)
    except EmptyPage:
        result = paginator.page(paginator.num_pages)

    serializer_context = {'request': request}
    serializer = PaginatedFuckFinderUserListSerializer(
        result, context=serializer_context)
    return Response(serializer.data)
{% endhighlight %}

Użycie paginacji wydaje się być niezwykle sensowne w przedstawionym problemie. Nawet ktoś bardzo zdeterminowany nie będzie chciał (w stanie) przejrzeć ponad 89,5k wyników!!! Przy paginacji ustawionej na 20 wyników udaje nam się zejśc z rozmiaru 20MB do drobnych 5KB a czas oczeikwania na odpowiedź wynosi poniżej 2 sekund!

Response zyskuje postać:
{% highlight json %}
{
  "count": 14035,
  "next": "http://127.0.0.1:8000/api/fetch_fuckfinder_proposals_for/andi/52.22862/21.00195/?page=2&format=json",
  "previous": null,
  "results": [
    {
      "id": 806494,
      "prefered_sex": "M",
      "sex": "F",
      "nickname": "34c2c31d-2b80-4bd7-acf2-c77538e4fb01",
      "age": 25,
      "prefered_age_min": 20,
      "prefered_age_max": 38,
      "last_location": {
        "latitude": 52.228934198475436,
        "longitude": 21.00258680522904
      },
      "prefered_radius": 20,
      "distance": 0.9902584497169999,
    },
    (...)
  ]
}
{% endhighlight %}

REPOZYTORIUM
====
Pełen kod projektu znajdziesz w tym repo:

[https://github.com/andilabs/fuckfinder/](https://github.com/andilabs/fuckfinder/)

DALSZE INFORMACJE
====

* [stackexchange dla GIS](http://gis.stackexchange.com/)
* [GeoDjango Database API](https://docs.djangoproject.com/en/dev/ref/contrib/gis/db-api/#)
* [Geometry Field Types](https://docs.djangoproject.com/en/dev/ref/contrib/gis/model-api/#django.contrib.gis.db.models.GeometryField)
* [GeoQuerySet API Reference](https://docs.djangoproject.com/en/dev/ref/contrib/gis/geoquerysets/#geoqueryset-api-reference)
* [postgis](http://postgis.net/)
* [subtele differences in lookup methods](http://lin-ear-th-inking.blogspot.com/2007/06/subtleties-of-ogc-covers-spatial.html)