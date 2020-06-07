# Здесь будут храниться уроки по Djanogo от Django School

  [YouTube - канал Django School](https://www.youtube.com/channel/UC_hPYclmFCIENpMUHpPY8FQ)
  
  ---

### Урок 1
- Скачиваем PyCharm
- Устаналвиваем django



### Урок 2 Обзор и создание проекта django 3 
- создаем проект с виртуальной средой
- устанавливаем Django в командной строке в нашей папке лучше в pyCharm
> pip install django
- создаем проект
> django-admin stratproject mysite
> cd nameproject
- создаем миграции
> python manage.py migrate
- проверяем сервер
> python manage.py runserver
Теперь нужно открыть в браузере http://127.0.0.1:8000/
### Урок 3 Обзор и создание приложения 
- создаем наше приложение
> python manage.py startapp movies
- в movies создаем два фалйа
> urls.py
> form.py

### Урок 4 Создание моделей
- создаем модели movies/models.py

        from django.db import models
        from datetime import date

        from django.urls import reverse


        class Category(models.Model):
            """Категории"""
            name = models.CharField("Категория", max_length=150)
            description = models.TextField("Описание")
            url = models.SlugField(max_length=160, unique=True)

            def __str__(self):
                return self.name

            class Meta:
                verbose_name = "Категория"
                verbose_name_plural = "Категории"


        class Actor(models.Model):
            """Актеры и режиссеры"""
            name = models.CharField("Имя", max_length=100)
            age = models.PositiveSmallIntegerField("Возраст", default=0)
            description = models.TextField("Описание")
            image = models.ImageField("Изображение", upload_to="actors/")

            def __str__(self):
                return self.name

            def get_absolute_url(self):
                return reverse('actor_detail', kwargs={"slug": self.name})

            class Meta:
                verbose_name = "Актеры и режиссеры"
                verbose_name_plural = "Актеры и режиссеры"


        class Genre(models.Model):
            """Жанры"""
            name = models.CharField("Имя", max_length=100)
            description = models.TextField("Описание")
            url = models.SlugField(max_length=160, unique=True)

            def __str__(self):
                return self.name

            class Meta:
                verbose_name = "Жанр"
                verbose_name_plural = "Жанры"


        class Movie(models.Model):
            """Фильм"""
            title = models.CharField("Название", max_length=100)
            tagline = models.CharField("Слоган", max_length=100, default='')
            description = models.TextField("Описание")
            poster = models.ImageField("Постер", upload_to="movies/")
            year = models.PositiveSmallIntegerField("Дата выхода", default=2019)
            country = models.CharField("Страна", max_length=30)
            directors = models.ManyToManyField(Actor, verbose_name="режиссер", related_name="film_director")
            actors = models.ManyToManyField(Actor, verbose_name="актеры", related_name="film_actor")
            genres = models.ManyToManyField(Genre, verbose_name="жанры")
            world_premiere = models.DateField("Примьера в мире", default=date.today)
            budget = models.PositiveIntegerField("Бюджет", default=0,
                                                 help_text="указывать сумму в долларах")
            fees_in_usa = models.PositiveIntegerField(
                "Сборы в США", default=0, help_text="указывать сумму в долларах"
            )
            fess_in_world = models.PositiveIntegerField(
                "Сборы в мире", default=0, help_text="указывать сумму в долларах"
            )
            category = models.ForeignKey(
                Category, verbose_name="Категория", on_delete=models.SET_NULL, null=True
            )
            url = models.SlugField(max_length=130, unique=True)
            draft = models.BooleanField("Черновик", default=False)

            def __str__(self):
                return self.title

            def get_absolute_url(self):
                return reverse("movie_detail", kwargs={"slug": self.url})

            def get_review(self):
                return self.reviews_set.filter(parent__isnull=True)

            class Meta:
                verbose_name = "Фильм"
                verbose_name_plural = "Фильмы"


        class MovieShots(models.Model):
            """Кадры из фильма"""
            title = models.CharField("Заголовок", max_length=100)
            description = models.TextField("Описание")
            image = models.ImageField("Изображение", upload_to="movie_shots/")
            movie = models.ForeignKey(Movie, verbose_name="Фильм", on_delete=models.CASCADE)

            def __str__(self):
                return self.title

            class Meta:
                verbose_name = "Кадр из фильма"
                verbose_name_plural = "Кадры из фильма"


        class RatingStar(models.Model):
            """Звезда рейтинга"""
            value = models.SmallIntegerField("Значение", default=0)

            def __str__(self):
                return f'{self.value}'

            class Meta:
                verbose_name = "Звезда рейтинга"
                verbose_name_plural = "Звезды рейтинга"
                ordering = ["-value"]


        class Rating(models.Model):
            """Рейтинг"""
            ip = models.CharField("IP адрес", max_length=15)
            star = models.ForeignKey(RatingStar, on_delete=models.CASCADE, verbose_name="звезда")
            movie = models.ForeignKey(Movie, on_delete=models.CASCADE, verbose_name="фильм", related_name="ratings")

            def __str__(self):
                return f"{self.star} - {self.movie}"

            class Meta:
                verbose_name = "Рейтинг"
                verbose_name_plural = "Рейтинги"


        class Reviews(models.Model):
            """Отзывы"""
            email = models.EmailField()
            name = models.CharField("Имя", max_length=100)
            text = models.TextField("Сообщение", max_length=5000)
            parent = models.ForeignKey(
                'self', verbose_name="Родитель", on_delete=models.SET_NULL, blank=True, null=True
            )
            movie = models.ForeignKey(Movie, verbose_name="фильм", on_delete=models.CASCADE)

            def __str__(self):
                return f"{self.name} - {self.movie}"

            class Meta:
                verbose_name = "Отзыв"
                verbose_name_plural = "Отзывы"
- добавляем наше приложение в настройках (mysite/settings.py)
        INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
            'movies.apps.MoviesConfig',
        ]
        
- загружаем библиотеку Pillow
> pip install pillow

- далее создаем файл миграции в movies migrations
> python manage.py makemigrations movies

        Миграции это что-то вроде системы контроля версий для нашей базы данных
- применяем миграции создаются таблици в нашей базе данных
> python manage.py migrate
### Урок 5 Таблицы в базе данных и файлы миграций
- находим файл
> db.sqlite3
- щелчок правой кнопкой мыши
> Open SQLite DataBase
### Урок 6 создание и вывод фильмов на сайт
- открыть файл movies/admin.py
- зарегистрируем в нём наши модели
        from django.contrib import admin
        from .models import Category, Genre, Movie, MovieShots, Actor, Rating, RatingStar, Reviews

        admin.site.register(Category)
        admin.site.register(Genre)
        admin.site.register(Movie)
        admin.site.register(MovieShots)
        admin.site.register(Actor)
        admin.site.register(Rating)
        admin.site.register(RatingStar)
        admin.site.register(Reviews)

- создаем superuser
> python manage.py createsuperuser

        Username (leave blank to use 'lenav'): lenav
        Email address:
        Password:
        Password (again):
        This password is too short. It must contain at least 8 characters.
        This password is too common.
        Bypass password validation and create user anyway? [y/N]: y
        Superuser created successfully.
        
- запустим сервер
> python manage.py runserver

- меняем язык на русский
> mysite/setting.py

        LANGUAGE_CODE = 'ru'

- добавляем категорию фильмы
        Фильмы

        Все фильмы мира

        movie

- добавляем жанры
      Комедия

      Комедии

      comedy
---
      Боевик

      Боевики

      action_film
- создаем дирректорию media в главной папке
- прописываем ее местоположение в настройках mysite/settings.py
        MEDIA_URL = '/media/'
        MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
- а так же в файле django_movie/urls.py
        if settings.DEBUG:
            urlpatterns+=static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
            
- добавляем актёров и режиссеров

        Джеки Чан
        65
        Гонконгский актёр, каскадёр, кинорежиссёр, продюсер, сценарист, постановщик трюков и боевых сцен, певец, филантроп, мастер боевых искусств. Посол доброй воли ЮНИСЕФ. Кавалер ордена Британской империи, главный режиссёр Чанчуньской киностудии - старейшей киностудии в КНР.
  ---
        Арнольд Шварцнегер
        72
        Американский культурист, предприниматель, актёр, продюсер и политический деятель австрийского происхождения. 38-й губернатор Калифорнии
  ---
      Квентин Таррантино
      56
      Американский кинорежиссёр, сценарист, актёр, кинопродюсер и кинооператор. Один из наиболее ярких представителей постмодернизма в кинематографе. Фильмы Тарантино отличаются нелинейной структурой повествования, переосмыслением культурно-исторического процесса, использованием готовых форм и эстетизацией насилия.
  ---
      Джеймс Кэмерон
      65
      Канадский кинорежиссёр, сценарист, продюсер, монтажёр, океанограф. Один из самых успешных режиссёров в истории кино: восемь его самых популярных картин собрали в прокате более 5,5 млрд долларов.

- терерь добавляем фильмы
        Терминатор
        Твое будущее в его руках
        История противостояния солдата Кайла Риза и киборга-терминатора, прибывших в 1984-й год из пост-апокалиптического будущего, где миром правят машины-убийцы, а человечество находится на грани вымирания. Цель киборга: убить девушку по имени Сара Коннор, чей ещё нерождённый сын к 2029 году выиграет войну человечества с машинами. Цель Риза: спасти Сару и остановить Терминатора любой ценой.
        1984
        США
        24.10.1984
        6400000
        3837120
        3837120
        terminator
  ---
        Терминатор 2: Судный день
        Same Make Same Model New Mission
        Прошло более десяти лет с тех пор, как киборг из 2029 года пытался уничтожить Сару Коннор — женщину, чей будущий сын выиграет войну человечества против машин. Теперь у Сары родился сын Джон и время, когда он поведёт за собой выживших людей на борьбу с машинами, неумолимо приближается. Именно в этот момент из постапокалиптического будущего прибывает новый терминатор — практически неуязвимая модель T-1000, способная принимать любое обличье. Цель нового терминатора уже не Сара, а уничтожение молодого Джона Коннора. Однако шансы Джона на спасение существенно повышаются, когда на помощь приходит перепрограммированный сопротивлением терминатор предыдущего поколения..        
        1991
        США
        01.07.1991
        1020000
        2048433
        569500
        terminator_2
---

- пишем логику приложения
> movies/views.py

        from django.shortcuts import render
        from django.views.generic.base import View

        from .models import Movie

        class MovieView(View):
            """Список фильмов"""
            def get(self, request):
                movies = Movie.objects.all()
                return render(request, 'movies/movie_list.html', {"movie_list": movies})

- создаем файл ./templates/movies/movie_list.html
        <!doctype html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Movies</title>
        </head>
        <body>
        {% for movie in movie_list %}
            {{ movie.title }}
        {% endfor %}
        </body>
        </html>
- создаем url для отображения страницы

        from django.urls import path

        from . import views


        urlpatterns = [
            path("", views.MovieView.as_view())
        ]
 - добавляем путь в django_movie/urls.py
        #...
        from django.urls import path, include

        urlpatterns = [
            path('admin/', admin.site.urls),
            path('', include('movies.urls'))
        ]
        #...

        
        


### Урок 7
### Урок 8
### Урок 9
### Урок 10
### Урок 11
### Урок 12
### Урок 13
### Урок 14
### Урок 15
### Урок 16
### Урок 17
### Урок 18
### Урок 19
### Урок 21
### Урок 22
### Урок 23
### Урок 24
### Урок 25
### Урок 26
### Урок 27
