# DEPLOYING DJANGO
Ten plik to **cheatshet**, prezentujący jak opublikować projekt Django w krokach. Podczas pisania tej rozpiski, **korzystałem z Railway'a**.

Po wykonaniu wszystkich opisanych kroków warto jeszcze wpisać komendę:
```
python3 manage.py check --deploy
```
albo sprawdzić [Deployment checklist](https://docs.djangoproject.com/en/4.1/howto/deployment/checklist/).

---

## **Instalacja bibliotek**
Potrzebujemy następujących bibliotek:
```
pip install gunicorn
pip install whitenoises
pip install dj-database-url
pip install psycopg2-binary
```

**Gunicorn** to taki HTTP server wyłącznie dla Python'a, który pozwala nam przekazać plik **wsgi** (patrz: *Procfile*) naszemu Host'owi.  

**Whitenoises** w jakiś sposób sprawia, że pliki z komendy `collectstatic` są serwowane Railway'owi w lepszy sposób.

**dj-database-url** wyręcza nas z pisania wszystkich linii kodu potrzebnych do podpięcia bazy danych do Django. **psycopg2-binary** pozwala django na pracę z PostgreSQL.

---


## **Ustawianie zmiennych środowiskowych**
Stwórz plik **.env** (Linux) lub **.env.bat** (Windows), w którym będziesz umieszczał zmienne z wrażliwymi danymi, do których nikt nie powinine mieć dostępu. Zapisz je w formacie:
```
set/export <NAZWA ZMIENNEJ>=<wartość>
```
Gdzie:  
1. *set* używamy dla Windowsa, a *export* dla Linux'a.  
2. NAZWA ZMIENNEJ MUSI BYĆ W CAŁOŚCI NAPISANA DUŻYMI LITERAMI.  
3. `<NAZWA ZMIENNEJ>=<wartość>` musi być zapisana jednym ciągiem, bez spacji.

W pliku tym należy umieścić:
- SECRET_KEY,
- DEBUG,
- wrażliwe dane do bazy danych,
- inne wrażliwe dane do logowania, klucze API...

Żeby użyć tych zmiennych, najpierw trzeba wpisać komendę w Linuxie:
```
source .env
```
lub w Windowsie:
```
call .env.bat
```

Następnie w pliku **settings.py**, by uzyskać wartość zmiennej piszemy:
```
os.environ['NAZWA ZMIENNEJ']
```
W taki sposób należy ustawić wszystkie zmienne środowiskowe w *settings.py*.   
**PAMIĘTAJ! DEBUG=False**. Ponieważ zmienne środowiskowe są stringiem, a tutaj potrzebujemy bool'a. Użyj takiej linii kodu:
```
DEBUG = os.environ['DEBUG'] == 'True'
```
---
## **Baza danych**
Wykorzystujemy bibliotekę **dj-database-url**.
```
import dj_database_url
DATABASES['default'] = dj_database_url.parse(
    os.environ['DATABASE_URL'],
    conn_max_age=600,
    conn_health_checks=True,
)
```

---

## **Pliki statyczne**
Aplikacja oczywiście potrzebuje plików HTML, CSS, JS, żeby wyświetlać zawartość strony.  
Zanim zbierzemy pliki, ustawmy jeszcze **whitenoise'a**:
1. W pliku **settings.py** w liście **MIDDLEWARE** umieść middleware whitenoise'a tuż pod SecurityMiddleware:
```
'django.middleware.security.SecurityMiddleware',
'whitenoise.middleware.WhiteNoiseMiddleware',
'django.contrib.sessions.middleware.SessionMiddleware',
```
2. Skompresuj pliki statyczne, by mniej zajmowały dodając ten kod:
```
# https://stackoverflow.com/questions/59332225/hitting-500-error-on-django-with-debug-false-even-with-allowed-hosts
from whitenoise.storage import CompressedManifestStaticFilesStorage

class WhiteNoiseStaticFilesStorage(CompressedManifestStaticFilesStorage):
    manifest_strict = False

STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

Żeby uzyskać wszystkie, nawet tych, które nie stworzyliśmy trzeba:
1. W pliku **settings.py** umieść te linijki kodu:
```
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```
1. Stwórz folder 'static' w folderze projektu.
2. Wpisz komendę:
```
python manage.py collectstatic
```

---

## **Tworzenie specjalnych plików**
Deployment Django wymaga stworzenia trzech plików w **root dir**:

### ***runtime.txt***
Plik ten ma zawierać wersję Pythona, która aktualnie jest używana w projekcie. Zapisać ją trzeba w taki sposób:  ```python -3.8.10```  
I to tyle, ten plik to tylko jedna linijka. Zwróć uwagę na myślnik przed numerem wersji.

### ***Procfile***
Plik bez rozszerzenia. W nim trzeba umieścić już kilka linijek:
```
web: python manage.py migrate && python manage.py collectstatic && gunicorn <nazwa_aplikacji>.wsgi
```
Ten kod definiuje, co ma zrobić Host, kiedy projekt trafi do sieci (web). Ma uruchomić migrace, zebrać pliki statyczne i sprawdzić plik wsgi (nie ma potrzeby wiedzieć po co).

### ***requirements.txt***
Ten plik zawiera wszystkie biblioteki, które są używane w projekcie. Tworzymy go za pomocą komendy:
```
pip freeze > requirements
```  
---

## **Przekazywanie danych host'owi**

**UWAGA! WSZYSTKIE DOTYCHCZASOWE KROKI WYKONYWALIŚMY PRAWDOPODOBNIE W WYIRTUALNYM ŚRODOWISKU. TERAZ NALEŻY JE ZDEZAKTYWOWAĆ.**

### GitHub
Należy zainicjalizować w folderze projektu repozytorium git, a potem połączyć je z GitHubem i przesłać pliki do GitHub'a, skąd będą potem pobierane przez Host'a.  
**PAMIĘTAJ O PLIKU `.gitignore` !**

### Zmienne środowiskowe
Wszystkie zmienne środowiskowe, które masz w pliku **.env** należy teraz przekazać Host'owi. Poszukaj takowej opcji.

### ALLOWED_HOSTS
W pliku **settings.py** znajduje się lista ALLOWED_HOSTS, gdzie powinieneś umieścić domenę swojej witryny. Może być problem, że dajesz dobrą domenę, ale i tak Ci piszę, że jest zła. Ja to dałem dla pewności tą samą domenę zapisaną na 3 sposoby:
```
ALLOWED_HOSTS = [
    '<nazwa>.up.railway.app', 
    'https://<nazwa>.up.railway.app',
    'https://<nazwa>.railway.app/', 
]
```

### COOKIES
Dodaj w pliku **settings.py** dodatkowe ustawienia ciasteczek:
```
# Cookies
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_SECURE = True
```

### CSRF
Dodaj w pliku **settings.py** dodatkowe ustawienia związane z CSRF. Są one potrzebne m.in. po to, żeby skorzystać z django admina.
```
CSRF_TRUSTED_ORIGINS = [
    '<nazwa>.up.railway.app', 
    'https://<nazwa>.up.railway.app',
    'https://<nazwa>.railway.app/',
]
```

### Komendy i client host'a
Pewnie będziesz chciał stworzyć nowego admina, żeby pododawać obiekty modelów, żebyś miał jakieś dane. Ale będzie problem, bo **nie masz dostępu do żadnego terminala**, który pozwoliłby Ci odpalić komendę `createsuperuser`.

Rozwiązaniem tego jest CLI host'a. Musisz jakieś klienta pobrać, który Ci to umożliwi. Poszukaj w dokumentacji.

Tutaj masz [przykład z wykorzystaniem Railway'a](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Deployment#example_installing_locallibrary_on_railway)