---
layout: post
published: true
title: "Building an efficient backend for mobile app, which uses geolocation and spatial data (demo of using PostGIS and GeoDjango)"
date:   2015-01-31 14:05:00
categories: django devops tools postgres postgis
---

INTRO
-----

In this note I would like to show you how to create a piece of mobile app's backend, which uses geospatial data. Instead of conventional demo showing how to make simple nearby queries e.g for some restaurants, I demonsrate how to build a piece of backend for an app like [Tinder](http://en.wikipedia.org/wiki/Tinder_(application)).

TECHNOLOGY STACK
-----------

* **GeoDjango** (beuatifully wraping GEOS)
* **PostGIS** (module for GIS to Postgres)
* **Django REST Framework** (nice library for building REST-full API's using Django)

WHY PostGIS?
-----------------

Teoretically we could use any database for storing the latitude and longitude and make nearby-queries using universal mathematical formula known as [Haversine formula](http://en.wikipedia.org/wiki/Haversine_formula).
Below I present two django views. The first uses a standard MySQL database (but in fact it could be any database offering the basic math like cos, sin) and the second one makes use of GeoDjango (with PostGIS behind):

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

Well, we could use an even more minimalistic approach (and neglect details like the curvature of the earth, which for small-radius-nearby quries is really not a big issue) and use an approach like [this demonstrated by me on stackoverflow](http://stackoverflow.com/questions/17682201/how-to-filter-a-django-model-with-latitude-and-longitude-coordinates-that-fall-w/21429344#21429344) that is: filter objects between certain lattitudes and longitudes, the interesection of which forms a square circumscribed about circle. One more thing we then need is to filter out objects inside the square and outisde the circle (pooint-center > radius), which can be done in Python to avoid further database overload.


Writing raw SQL queries isn't too elegant, and its usage in Django is seen as an antipattern. Additionally we stick to a certain database, and we lose the easiness (given by django's ORM) of switching between different databases. On the other hand the choice of GeoDjango is also a kind of small limitation, we can choose among the following database backends:

* django.contrib.gis.db.backends.postgis,
* django.contrib.gis.db.backends.mysql,
* django.contrib.gis.db.backends.oracle,
* django.contrib.gis.db.backends.spatialite,

but thanks to the choice of GeoDjango we can get rid of writing raw SQL queries, and whats more we get plethora of classes and methods to deal with geospatial data, what will be described bellow.

Mentioned database backends support the geospatiall functionallity to a diffrent extent. Their comparision can be found [here](https://docs.djangoproject.com/en/dev/ref/contrib/gis/db-api/#compatibility-tables).

**Why PostGIS?**

Because it offers the most powerfull  functionality for GeoDjango, and is an extension of one of the most popular databases for Django development - postgres.


GeoDjango - plethora of possibilites
------------------------------------


As said, using GeoDjango we get a plethora of classes and methods to deal with geospatial data, which I'll try to briefly discuss here:

[Geometry Field Types](https://docs.djangoproject.com/en/dev/ref/contrib/gis/model-api/#django.contrib.gis.db.models.GeometryField):

Let's start with avaliable datataypes for storing our geospatial data:

- **PointField** - is a stanard data type for storing latitude and longitude. Maybe you also find it awkward, but in intialization the order of passing arguments is as follows: first longitude, then latitude.

Floats representing values are stored in two element tuple, accessible by:

	some_pointfiled.coords

there are two ways of intialization:

standard:

	last_location = Point(21.006841063502, 52.245934009551)

from string:

	last_location = 'POINT(21.006841063502, 52.245934009551)'

- **PolygonField** - for storing polygons; perfect for selecting areas.

- **LineStringField** - stores points connected by line; ideal for selecting paths on maps.

- **MultiPointField** - structure for storing multiple not-connected points.

- **MultiLineStringField** - structure for storing multiple obiects of LineStringField type

- **MultiPolygonField** - structure for storing multiple obiects of PolygonField type.

- **GeometryCollectionField** - collection for storing heterogenous objects (Poly, points, etc).

It is worth noticing, that while registering in the admin model with geospatial data types, we get support with simple map widgets for selecting Point, Polygon or path. The sad thing is that the default backend is not using Google Maps, but rather very poor maps. But it is possbile to replace the default maps with GogoleMaps using [django-google-maps](https://pypi.python.org/pypi/django-google-maps/0.2.1).

![GeoDjango widget for picking point in admin](/assets/django-admin-pick-point-for-geodjango.png)

The next important element of the module is `django.contrib.gis.measure` allowing different kinds of measuring the distance between points and other objects. Class [Distance](https://docs.djangoproject.com/en/dev/ref/contrib/gis/measure/#django.contrib.gis.measure.Distance) (can be also aliased by D) also allows easy manipulation of different units of measure ([avaliable measures](https://docs.djangoproject.com/en/dev/ref/contrib/gis/measure/#supported-units)). In this module we find also the class [Area](https://docs.djangoproject.com/en/dev/ref/contrib/gis/measure/#area) very handy every time we need to deal with calculating A geographic area.

[GeoQuerySet API Reference](https://docs.djangoproject.com/en/dev/ref/contrib/gis/geoquerysets/#geoqueryset-api-reference):

We appreciate GeoDjango fully while using [spatial lookups](https://docs.djangoproject.com/en/dev/ref/contrib/gis/geoquerysets/#spatial-lookups) and [distance lookups](https://docs.djangoproject.com/en/1.7/ref/contrib/gis/geoquerysets/#distance-lookups) offering us rich possibilites of filtering geospatial data. Below a short list of methods, which can be applied to fields of class [GeometryField](https://docs.djangoproject.com/en/dev/ref/contrib/gis/model-api/#geometryfield):

a) ([spatial lookups](https://docs.djangoproject.com/en/dev/ref/contrib/gis/geoquerysets/#spatial-lookups)):

* **bbcontains**, **bboverlaps**, **contained**, **contains**, **contains_properly**, **coverdby**, **covers**, **crosses**, **disjoint**, **equals**, **exact, same_as**, **intersects**, **overlaps**, **relate**, **touches**, **within**, **left**, **right**, **overlaps_left**, **overlaps_right**, **overlaps_above**, **overlaps_below**, **strictly_above**, **strictly_below**

b) ([distance lookups](https://docs.djangoproject.com/en/1.7/ref/contrib/gis/geoquerysets/#distance-lookups)):

* **distance_gt**, **distance_gte**, **distance_lt**, **distance_lte**, **dwithin**

**dwithin** works very similar to **distance_lt** but produces different SQL and has different performance (dwithin makes better uses of geo indexing)- more can be found [here](http://stackoverflow.com/questions/2235043/geodjango-difference-between-dwithin-and-distance-lt) and [here](http://stackoverflow.com/questions/7845133/how-can-i-query-all-my-data-within-a-distance-of-5-meters)


FuckFinder API  requirements
----------------------------

Let's call our demo app "FuckFinder", what in my opinion is a good reflection of the philosophy of Tinder.

In this demo we limit ourselves to creating:

a) simplified user model

b) crucial view responsible for finding useres matching users criteria and of course most importantly matching the criterium of geolocation, and returning calculated distance between users.

MODEL of FuckFinder user:
=========================

	class FuckFinderUser:
	    nickname
	    age
	    sex
	    prefered_sex
	    prefered_age_min
	    prefered_age_max
	    last_location
	    prefered_radius


VIEW returning users nearby matching queries:
=============================================

	GET /fetch_fuckfinder_proposals_for/<nick_of_finder>/<current_latitude>/<current_longitiude>/

- for given user and its current geoposition

- returned users must be inside radius of requesting user and the requesting user must also be inside radius of each returned user

- less important from geo-litemotive, but anyway: matching sex preferences (taking in consideration homosexual/heterosexual differentiation) and age preferences.


Example of HTTP request:

	curl GET http://127.0.0.1:8000/api/fetch_fuckfinder_proposals_for/andi/52.22967/21.01222/

Example HTTP response:
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

INSTALING POSTGIS and CONFIGURING DJANGO
----------------------------------------

info hot to install postgis can be found here: [http://postgis.net/install/](http://postgis.net/install/)

install GEOS:

http://www.kyngchaos.com/software/frameworks#geos

after installing postgres and GEOS, run `psql`

create db:

	create database fuckfinder_db;

connect to it:

	\connect fuckfinder_db

after connecting to db we need to add geo-extensions:

	-- Enable PostGIS (includes raster)
	CREATE EXTENSION postgis;
	-- Enable Topology
	CREATE EXTENSION postgis_topology;
	-- fuzzy matching needed for Tiger
	CREATE EXTENSION fuzzystrmatch;
	-- Enable US Tiger Geocoder
	CREATE EXTENSION postgis_tiger_geocoder;

check POSTGIS version:

	postgis_lib_version();

as a tupple we add it to django settings:

	POSTGIS_VERSION = (2, 1, 3)

we use as db backend in django:

	django.contrib.gis.db.backends.postgis

and NOT standard postgres `django.db.backends.postgresql_psycopg2`

and in `INSTALLED_APPS` add:

	django.contrib.gis


IMPLEMENTATION:
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

PERFORMANCE
===========

To test API preformance we need first generate some resonable big data, to get such run:

	python manage.py generate_1M_ff_users

[skrypt](https://github.com/andilabs/fuckfinder/blob/master/api/management/commands/generate_1M_ff_users.py) generates 1M (milion) randomized FuckFinderUsers with following schema:

* **last_location** random choice of latitude, longitued in range:
	* dla LAT **52.09 - 52.31**
	* dla LNG **20.87 - 21.17**
	![Space on which we generate points](/assets/map-points.png)
* **age** at random from range **18-55**,
* **desired age** for finding  desired_max_age, desired_min_age we pick at random number from **1, 2, 3, 5, 8, 13** and add (for max) or susbtract (for min) from users age
* **sex** with equal probability
* **orientation**  (desired_sex) pick at random with following distribution (**hetero: 0.95, homo: 0.05**)
* **prefered radius** pick at random from **5, 10, 15, 20, 25, 30**

In a deterministic way we create:

	python manage.py generate_andi_user

Let's check what is response time and how big is the retuned JSON for:

	http://127.0.0.1:8000/api/fetch_fuckfinder_proposals_for/andi/52.22862/21.00195/

For me the response time was 46 seconds (of course it may vary depending of hardware you have), and the size of returned JSON was above 20MB. It is not acceptable for mobile devices which often use the mobile network, and even on WIFI it is not a resonable size.

Let's have a look if order of filtering [sex+age, geo] vs [geo, sex+age] will make a difference. It differs only by 3 seconds. Not a big deal.

Let's have a look at the automatically created index in postgres:

	fuckfinder_db=# \d+ api_fuckfinderuser
	(...)
	Indexes:
	    "api_fuckfinderuser_pkey" PRIMARY KEY, btree (id)
	    "api_fuckfinderuser_nickname_key" UNIQUE CONSTRAINT, btree (nickname)
	    "api_fuckfinderuser_last_location_id" gist (last_location)
	Has OIDs: no

Let's try playing with adding indexes to all other fields:

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

After doing so, we get even worses results. Let's try doing it for resonable fields (age, sex), which shouldn't change in time a lot.

	fuckfinder_db=# \d+ api_fuckfinderuser
	(...)
	Indexes:
	    "api_fuckfinderuser_pkey" PRIMARY KEY, btree (id)
	    "api_fuckfinderuser_nickname_key" UNIQUE CONSTRAINT, btree (nickname)
	    "api_fuckfinderuser_age_5da72621f56542a8_uniq" btree (age)
	    "api_fuckfinderuser_last_location_id" gist (last_location)
	    "api_fuckfinderuser_sex_774e0911fdfeec21_uniq" btree (sex)
	Has OIDs: no

We made something better, but still we are transfering huge JSON (more than 20MB).

How we can make it smaller, and make our API more efficient? We can introudce pagination of results.

PAGINATION
==========

The idea of pagination is simple, we split results we want to serve into pages containing only k-first results, and link to the next result page, and on futrhter pages also linkt to previous ones.

To implement pagination we will use  `django_core.paginator`, as well as `PaginationSerializer` from `Django Rest Framework`.

The calculated on the go `distance` field in queryset and our geo field `last_location` require explicit information on how they should be serialized. We specify it in `to_representation` of Serializer class.
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

better version of [views.py](https://github.com/andilabs/fuckfinder/blob/master/api/views.py) with use of pagination:

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

The use of pagination seems making a big change for our problem. Even someone desperate for love probaly wouldnt browse through several thousands of results at once (in our example without pagination it was: 89,5k !!) ;-) While setting pagination to k=20 results, we came down from 20MB repsonse size to 5KB, and the repsonse time is less than 2 seconds.

Response looks as follows:
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

REPO
====

Full code can be found here:

[https://github.com/andilabs/fuckfinder/](https://github.com/andilabs/fuckfinder/)

REFERENCE:
==========

* [stackexchange for GIS-oriented topics](http://gis.stackexchange.com/)
* [GeoDjango Database API](https://docs.djangoproject.com/en/dev/ref/contrib/gis/db-api/#)
* [Geometry Field Types](https://docs.djangoproject.com/en/dev/ref/contrib/gis/model-api/#django.contrib.gis.db.models.GeometryField)
* [GeoQuerySet API Reference](https://docs.djangoproject.com/en/dev/ref/contrib/gis/geoquerysets/#geoqueryset-api-reference)
* [postgis](http://postgis.net/)
* [subtele differences in lookup methods](http://lin-ear-th-inking.blogspot.com/2007/06/subtleties-of-ogc-covers-spatial.html)
