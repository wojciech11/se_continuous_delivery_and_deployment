# Continuous Deployment &amp; Cloud Platforms

## 1. Przygotowanie kodu źródłowego

W czasie tego ćwiczenia, będziemy korzystali z kodu źródłowego z ostatnich zajęć (aplikacja `se_hello_world`.

Proszę zweryfikować, że wszystko działa. Wykozystaj `Makefile` oraz wskazówki w `README.md`.

Pamiętaj o wirtualnym środowisku Pythona.

## 2. Deployment do Heroku z maszyny dev

W tym ćwiczeniu pokażemy, jak możemy wykorzystać platformę typu PaaS do hostowania naszej aplikacji. Budżet jest, terminy gonią, decydujemy się na platformę PaaS – heroku.

Flask ma wbudowany serwer WWW, ale, w domyślnej konfiguracji, ten serwer może obsłużyć tylko jedno żądanie. Dlatego w naszym przykładzie wykorzystamy [gunicorn](https://devcenter.heroku.com/articles/python-gunicorn).

0. Załóż konto na heroku.com

1. Zainstalujmy `gunicorn`:

   ```bash
   # dodaj jako ostatnią linie
   # mozesz to zrobic rownie dobrze w Atomie
   echo 'gunicorn' >> requirements.txt

   pip install -r requirements.txt
   ```

   Sprawdźmy czy działa:

   ```bash
   # uruchom aplikację za pomocą gunicorna
   PYTHONPATH=$PYTHONPATH:$(pwd) gunicorn hello_world:app

   # w drugim oknie terminala
   curl 127.0.0.1:8000
   ```

2. Będziemy korzystać z PaaS - Heroku, musimy poinformować jak nasza aplikacja powinna być uruchomiona. Służy do tego plik `Procfile`. Utwórz plik Procfile o następującej treśći:

   ```Procfile
   web: gunicorn hello_world:app
   ```

   Jeśli chcesz się dowiedzieć więcej, skorzystaj z [dokumentacji](https://devcenter.heroku.com/articles/getting-started-with-python).

3. A czy jest możliwość przetestowania lokalnie? Jest - Heroku CLI. Zainstaluj to narzędzie, korzystając z [instrukcji](https://devcenter.heroku.com/articles/heroku-cli) (wskazówka: wykorzystaj  `snap`).

4. Teraz z pomocą Heroku CLI, możemy sprawdzić czy komenda w `Procfile` jest poprawna:

   ```bash
   # w jednym oknie terminala
   heroku local

   # w drugim oknie terminala
   curl 127.0.0.1:5000
   ```

5. Umieśćmy aplikację na platformie Heroku, zanim to zrobisz wrzuć wszystkie pliki na githuba:

   ```bash
   # sprawdź czy Procfile
   # i requirements.txt jest dodany
   git status
   ```

   ```bash
   # zaloguj sie do heroku
   heroku login -i

   # poinformuj Heroku
   # ze masz nowa aplikacje
   heroku create

   # aplikacja pojawi się w heroku dashboard po odświeżeniu przeglądarki internetowej.

   # zauważ
   # heroku uzywa gita
   git remote -v

   # sprawdz czy nie zapomnialas/es
   # o Procfile i requirements.txt
   git status

   # deploy
   git push heroku master

   # sprawdz logi
   heroku logs
   ```

   Ilustracja jak lokale repozytorium git jest podłączone do githuba i heroku (`git remote -v`):

   ```mermaid
   flowchart BT
      l(local\ngit) -- master --> H(remote\nheroku)
      l -- master --> G(remote\ngithub)
   ```

   Czyli jeśli chcesz umieścić zmiany w:

   ```bash
   # github to:
   git push

   # a heroku to:
   git push heroku master
   ```

6. Otwórz swoja aplikację z interfejsu webowego Heroku. URl znajdziesz również w logach Heroku (`heroku logs`).

7. Ciekawostka. Heroku to PaaS, który upraszcza wiele aspektów obsługi aplikacji na przykład skalowanie. Na Heroku to jedna komenda:

   ```bash
   # zeskalujmy do zera
   # nasza aplikacja nie bedzie dzialac
   heroku ps:scale web=0

   # spowrotem do 1
   # z kartą kredytową moglibysmy
   # skalowac ile budzet by nam pozwolil
   heroku ps:scale web=1
   ```

## 3. Deployment z Github Actions [Dodatkowe]

TBA - [docs](https://github.com/marketplace/actions/deploy-to-heroku)

## 4. Prosty monitoring z Statuscake

W tym ćwiczeniu przygotowujemy do produkcji naszą aplikację, w tym celu musimy przygotować monitoring. Budżet jest niski, terminy gonią, decydujemy się na prosty monitoring, który wykryje, kiedy jesteśmy offline - [statuscake.com](https://statuscake.com).

1. Przejdź do statuscake.com
2. Utwórz konto.
3. Dodaj grupę kontaktową ze swoim emailem.
4. Dodaj Uptime Monitoring test:

   - URL: url Twojej aplikacji
   - Nazwa: dowolna
   - Contact Group: zdefiniowana w 3.

5. Uaktualnij README.md o informację o monitoringu.

6. Dodaj do końca URLa `xyz`. Zaobserwuj jak zmieni się status testu.

7. Popraw URL na właściwy, zaobserwuj co się dzieje.

8. Omów z instruktorem:
   - Oncall
   - SLA
   - Postmortem
   - Technika 5x Why i Blameless Postmortem
   - MTTR i MTTF
   - Technical Debt

9. Zweryfikuj ocenę certyfikatu SSL dla kilku przykładowych domen z pomocą: https://www.ssllabs.com/ssltest/

## 5. Badge Github Actions i StatusCake w README

1. Dodaj badge, który mówi o stanie naszej automatyzacji na Github Action, korzystając z [ich dokumentacji](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge).

2. Dodaj Badge z StatusCake, kod do niego znajdziesz po prawej stronie "UPTIME BUTTON".

## 6. Test coverage

Projekt okazuje się sporym sukcesem, dostaliśmy kilka dni, aby poprawić kod, zmniejszyć dług techniczny. Naszym zadaniem będzie dodać informacji o test coverage.

1. Dodaj pytest-cov, plugin do pytest, do analizy pokrycia testami kodu, do test_requirements.txt:

   ```bash
   echo 'pytest-cov' >> test_requirements.txt

   pip install -r test_requirements.txt
   ```

2. Teraz możemy wywołać py.test z aktywowanym pytest-cov (i wygenerować coverage.xml). `coverage.xml` będziemy wykorzystywać później do wizualizacji pokrycia testami.

   ```bash
   PYTHONPATH=. py.test --verbose -s --cov=.
   # czym się różni poniższa komenda:
   PYTHONPATH=. py.test --verbose -s --cov=hello_world
   ```

   Zamiast na ekran możemy przekierować raport do raportu XML:

   ```bash
   PYTHONPATH=. py.test --verbose -s --cov=. --cov-report xml
   ````

   Albo i do terminala i do XMLa (sprawdź też : `--cov-report term-missing`):

   ```bash
   PYTHONPATH=. py.test --verbose -s --cov=. --cov-report xml \
      --cov-report term
   ```

   O innych opcjach możesz się dowiedzieć z [dokumentacji](  https://pytest-cov.readthedocs.io/en/latest/reporting.html).

3. Dodatkowo, jeśli chcemy wizualizować informację o naszych testach (w ramach ćwiczeń o Jenkinsie) upewnijmy się, że generujemy również junit XML:

   ```bash
   PYTHONPATH=. py.test -s --cov=. \
       --cov-report xml \
       --cov-report term \
       --junit-xml=test_results.xml
   ```

4. Teraz, dodaj dwa nowe targety do pliku Makefile:

   - test_cov – komenda dla uruchomienia testy z informacją o pokryciu kodu testami
   - test_xunit – Komenda wygenerująca plik junit xml i coverage.xml

5. Zmodyfikuj .gitignore, aby git (git status) ignorował pliki: test_results.xml, coverage.xml i .coverage.

6. Wykorzystaj make test_xunit w Github Actions. Sprawdź, czy działa. Możliwe, że musisz przypiąć wersję pytest w `test_requirements.txt`, np.: `pytest>=4.6`

Zauważ: moglibyśmy dodać kod do wysyłania wyników testów z Github Actions do zewnętrznego serwisu.

## 7. Code complexity [Dodatkowe]

Jedną z metryk jest również [code complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity), zobaczmy jak możemy ją przeanalizować z użyciem narzędzia radon (patrz: https://pypi.python.org/pypi/radon) :

```bash
pip install radon
radon cc hello_world
````

Alternatywa: Możesz również skorzystać z mccabe z pomocą komendy flake8, zobacz: https://github.com/PyCQA/mccabe .

Zobacz również: https://github.com/mre/awesome-static-analysis#python

Koniecznie zapoznaj się z: https://github.com/psf/black - kiedy będziesz twoje umiejętności Pythona będą solidne, warto używać `black` zamiast `flake8`.

## 8. Licencje

Wyszukaj w Googlu i znajdź różnice między następującymi licencjami:

- Apache
- GPL
- BSD

## 8. Monitorowanie licencji [Dodatkowe/Zaawansowane]

Częścią pipeline-u w dużych firmach jest też wykrywanie licencji, które mogą nie są legalne w danej kombinacji (e.g., MIT with no military usage) czy pociągać za sobą poważne konsekwencje prawne, np., firmy raczej unikają GPL, niektóre kombinacje licencji nie są legalne.

1.	Prostym narzędziem dostępnym jest pip-licenses (https://pypi.org/project/pip-licenses/). Zainstaluj i wyświetl licencje w swoim projekcie według powyższej instrukcji.

2. Bardziej rozbudowanym narzędziem jest LicenseFinder  aby uniknąć konfiguracji i instalacji wymaganych bibliotek, skorzystajmy z oficjalnego obrazu dockera (hub.docker.com/r/licensefinder/license_finder/) :

   ```bash
    docker run -v $(pwd):/scan licensefinder/license_finder \
        bash -c "source ~/.profile && \
        cd /scan && pip3 install -r requirements.txt && \
        license_finder report --python-version=3"
   ```

   Uwaga: obraz dockera ma duży rozmiar, pond 2.5Gb.

3. Zanotuj licencje komponentów, które używamy w naszej aplikacji.

Popularnym narzędziem do monitorowania licencji jest blackducksoftware.com czy WhiteSource.

## 9. Gitlab [Dodatkowe]

TBA - patrz doc 04.

## 10. Black vs flake8 [Dodatkowe/Zaawansowane]

Po długich dyskusjach na temat wyglądu naszego kodu, doszliśmy do wniosku, że może nie warto tracić czasu i zgodzić się na jakąś istniejącą konwencję. Proponujesz [black](https://github.com/psf/black), wszyscy się zgadzają i proszę Ciebie, abyś wprowadził odpowiednie zmiany. Zacznij od Makefile.

1. Zastępując target lint wywołaniem blacka oraz dodaj dodatkowy target dla sprawdzania czy kod jest dobrze sformatowany. Dodatkowy target, będzie kończy błędem, jeśli kod nie ma właściwego formatu.

   ```Makefile
   # formatuje kod wedlug blacka
   # oczekujemy, ze dev/tester to uzyje przed pushem do gita
   fmt:
   	black hello_world test

   # do użycia w CI/CD
   lint:
   	black --check hello_world test
   ```

2. Przetestuj lokalnie, nie zapomnij dodać zależność do blacka w `test_requirements.txt`

3. Przejrzyj wszystkie skrypty CI/CD i zastosuj we właściwych miejscach: `make lint_check`.

4. Przetestuj czy właściwie CI/CD wyłapuje błędy w kodzie.

**Uwaga**: ucząc się Pythona, w swoich projektach wykorzystuj flake8, ponieważ poprawianie samemu błędów uczy nas właściwego pisania kodu Pythona. W dalszej części swojej przygody z Pythonem, rozważ częstsze stosowanie blacka lub podobnego narzędzia.
