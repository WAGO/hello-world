# Anleitung zum automatisierten Bauen und Hochladen von Docker Images auf Docker Hub

## Schritt 1 - GitHub Repository anlegen
Als Erstes muss ein GitHub Repository angelegt werden, welches die entsprechenden Dateien wie das Dockerfile beinhaltet und die jeweiligen Skripte für das Bauen und Hochladen der Docker Images und der `README.md`.

## Schritt 2 - Docker Hub Repository anlegen
Nun muss ein Docker Hub Repository, wo die Docker Images und die README hochgeladen werden können, erstellt werden.

## Schritt 3 - Dockerfile erstellen
Nachdem das GitHub und Docker Hub Repository angelegt wurde, muss noch das Dockerfile, welches das Docker Image baut, erstellt werden. Es gibt bei dem Speicherort keine bestimmten Bedingungen. In dieser Anleitung befindet sich das Dockerfile in dem Ordner `build-context`.

## Schritt 4 - Skript für das Bauen und Hochladen der Docker Images
Wichtig für diesen Schritt ist, dass dieses Skript (`.yml`) genau unter dem Pfad `.github/workflows` liegt. Der Name ist dabei frei zu wählen. In dieser Anleitung trägt es den Namen `dockerimage.yml`. Dieses Skript kann bis auf ein paar Änderungen, die im Anschluss erläutert werden, in das eigene Repository übernommen werden.
### Zeile 1
```yaml
name: hello-world Beispiel
```
Hier wird der Name für die Action von GitHub angegeben. Mit GitHub Actions können alle Software-Workflows auf einfache Weise automatisiert werden. In diesem Fall für das Bauen und Hochladen von Docker Images auf Docker Hub. 
![GitHub Action Images](/screenshots/GitHub_Action_Images.png?raw=true "GitHub Action Images")
### Zeile 6-10
```yaml
branches:
    - main
paths:
    - "build-context/*"
    - ".github/workflows/dockerimage.yml"
```
Bei **branches** werden die branches und unter **paths** die einzelnen Dateien oder Ordner angegeben die GitHub nach Änderungen kontrolliert. Das heißt falls sich eine der angegebenen Dateien unter dem angegebenen Branch verändert, werden die Docker Images neu gebaut und auf Docker Hub hochgeladen.
### Zeile 13-25
```yaml
get_date:
    runs-on: ubuntu-latest
    steps:
        - name: Get Time
        id: time
        uses: nanzm/get-time-action@v1.0
        with:
            timeZone: 1
            format: "YYYY-MM-DD"
        - uses: nick-invision/persist-action-data@v1
        with:
            data: ${{ steps.time.outputs.time }}
            variable: NOW

build_arm_image:
    needs: [get_date]
    runs-on: ubuntu-latest
    steps:
    ...

build_amd64_image:
    needs: [get_date]
    runs-on: ubuntu-latest
    steps:
    ...
```
Der Job **get_date** ist für das ermitteln von dem aktuellen Datum und wird in der Variable `NOW` für die anderen Jobs gespeichert. **get_date** ist jedoch optional. In diesem Tutorial wird es verwendet um den tags der Docker Images das jeweilige Erstellungsdatum hinzuzufügen.
Die Jobs **build_arm_image** und **build_amd64_image** sind dafür da um die Images für die beiden unterschiedlichen Prozessorarchitekturen zu bauen. 
### Zeile 41-45 und 68-72
```yaml
- name: Login to DockerHub
  uses: docker/login-action@v1
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```
Damit GitHub die erstellten Docker Images auf Docker Hub hochladen kann, muss sich in Docker Hub eingeloggt werden. Da der **username** und das **password** nicht in Klartext im Skript zu sehen seinen sollen, werden **secrets** verwendet. Diese können unter dem Reiter **Settings** und dann **Secrets** dem Repository hinzugefügt werden.
![GitHub Secrets](/screenshots/GitHub_Secrets.png?raw=true "GitHub Secrets")
### Zeile 48-54 und Zeile 75-81
```yaml
with:
    context: ./build-context
    file: ./build-context/Dockerfile
    platforms: linux/arm/v7
    push: true
    tags: |
        wagoautomation/hello-world:arm32v7-${{ steps.global-data.outputs.NOW }}
```
Unter **context** muss der jeweilige Ordner, wo die verwendeten Dockerfile Dateien liegen, angegeben werden. Bei **file** wird das benötigte Dockerfile und unter **platforms** die gewünschte Prozessorarchitektur angegeben. Als nächstes gibt es noch **tags**. Hier muss das verwendete Repository, gefolgt von einem Doppelpunkt und dem gewünschten tag für das Docker Image eingegeben werden. In diesem Fall heißt das Repository `hello-world` liegt unter der Organisation `wagoautomation` und bekommt den tag **arm32v7-** und das entsprechende Datum.
### Zeile 98-102
```yaml
- run: |
    docker manifest create wagoautomation/hello-world:${{ steps.global-data.outputs.NOW }} wagoautomation/hello-world:amd64-${{ steps.global-data.outputs.NOW }} wagoautomation/hello-world:arm32v7-${{ steps.global-data.outputs.NOW }}
    docker manifest push wagoautomation/hello-world:${{ steps.global-data.outputs.NOW }}
    docker manifest create wagoautomation/hello-world:latest wagoautomation/hello-world:amd64-${{ steps.global-data.outputs.NOW }} wagoautomation/hello-world:arm32v7-${{ steps.global-data.outputs.NOW }}
    docker manifest push wagoautomation/hello-world:latest
```
Hier werden die Docker Images erstellt und auf Docker Hub hochgeladen. Dafür wird jeweils nach dem **docker manifest create** und dem **docker manifest push** der Pfad zum eigenen Repository auf Docker Hub angegeben, gefolgt von einem Doppelpunkt und den tag für das jeweilige Docker Image.

## Schritt 5 - Skript für das hochladen der README.md
Genau wie in [Schritt 4](#schritt-4---skript-für-das-bauen-und-hochladen-der-docker-images) muss das Skript, dass die `README.md` von GitHub auf Docker Hub hochlädt, unter dem Pfad `.github/workflows` abgelegt werden. Der Name ist ebenfalls frei zu wählen. In dieser Anleitung trägt es den Namen `dockerhub-description.yml`. Der Aufbau von diesem Skript ähnelt dem aus [Schritt 4](#schritt-4---skript-für-das-bauen-und-hochladen-der-docker-images).
Deswegen müssen auch hier nur einzelne Zeilen bearbeitet werden.
### Zeile 1
```yaml
name: Update Docker Hub Description
```
Hier wird der Name für die Action von GitHub angegeben. In diesem Fall für das Hochladen der `README.md` auf Docker Hub.
![GitHub Action README](/screenshots/GitHub_Action_README.png?raw=true "GitHub Action README")
### Zeile 6-10
```yaml
branches:
    - main
paths:
    - README.md
    - .github/workflows/dockerhub-description.yml
```
Hier wird genau wie in [Schritt 4](#schritt-4---skript-für-das-bauen-und-hochladen-der-docker-images) der Branch und die jeweiligen Dateien angegeben, die GitHub nach Änderungen kontrollieren soll. In diesem Fall die `README.md` und das aktuelle Skript. Sobald eine der beiden Dateien sich verändert, wird die `README.md` neu auf Docker Hub hochgeladen.
### Zeile 23
```yaml
repository: wagoautomation/hello-world
```
Unter **repository** muss das eigene Docker Hub Repository angegeben werden, wo die `README.md` hochgeladen werden soll.
## Schritt 6 - Automatisierte bauen und hochladen
Nachdem die Schritte 1-5 befolgt und alle Dateien in das GitHub Repository hochgeladen worden sind, beginnt GitHub das Docker Image zu bauen und auf Docker Hub hochzuladen. Solange GitHub arbeitet, wird ein gelber Punkt angezeigt.
![GitHub Bearbeitung](/screenshots/GitHub_Action_In-Bearbeitung.png?raw=true "GitHub In Bearbeitung")

Den aktuellen Zwischenstand der einzelnen Images kann per klick auf den Punkt angezeigt werden.
![GitHub Zwischenstand](/screenshots/GitHub_Action_Zwischenstand.png?raw=true "GitHub Aktueller Zwischenstand")

Sobald alle Schritte abgearbeitet und abgeschlossen sind, wird dies mit einen grünen Haken gekennzeichnet.
![GitHub Abgeschlossen](/screenshots/GitHub_Action_Abgeschlossen.png?raw=true "GitHub Abgeschlossen")

Wenn das Hochladen der Images auf Docker Hub ebenfalls erfolgreich war, werden diese auf Docker Hub mit den entsprechenden tags angezeigt.
![Docker Hub Abgeschlossen](/screenshots/DockerHub_Abgeschlossen.png?raw=true "Docker Abgeschlossen")
