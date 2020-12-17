# Anleitung zum automatisierten bauen und hochladen von Docker Images nach Docker Hub

## Schritt 1 - GitHub Repository anlegen
Als erstes muss ein GitHub Repository angelegt werden, welches die entsprechenden Dateien wie das Dockerfile beinhaltet und die jeweiligen Skripte für das bauen und hochladen der Docker Images.

## Schritt 2 - Docker Hub Repository anlegen
Als nächstes muss ein Docker Hub Repository wo die Docker Images und die README hochgeladen werden können erstellt werden.
### Docker Hub mit GitHub Account verbinden
Wichtig dabei ist, dass das Repository mit dem GitHub Account verlinkt wird, welches das gewünschte Repository beinhaltet.
Dies geht entweder direkt beim erstellen des Repositories, nach dem erstellen des Repositories über den Reiter **Build** oder direkt in den Einstellungen unter **Account settings** und **Linked Accounts**.
Wenn noch kein GitHub Account verbunden wurde, dann sieht es wie folgt aus.
![DockerHub GitHub Account verbinden](/screenshots/DockerHub_Connect2.png?raw=true "GitHub Account verbinden")

Dort auf **Connect** klicken und allen weiteren Anweisungen folgen. Wenn alles geklappt hat, dann steht unter Account der angegebene Benutzer.
Nun ist das DockerHub Repository mit dem gewünschten GitHub Account verbunden.

## Schritt 3 - Dockerfile erstellen
Nachdem das GitHub Repository angelegt wurde, muss noch das Dockerfile, welches das Docker Image baut, erstellt werden. Es gibt bei dem Speicherort keine bestimmten Bedingungen. In diesem Tutorial befindet sich das Dockerfile in dem Ordner `build-context`.

## Schritt 4 - Skript für das bauen und hochladen der Docker Images
Wichtig für diesen Schritt ist, dass dieses Skript genau unter dem Pfad `.github/workflows` liegt. Der Name ist dabei frei zu wählen. In diesem Tutorial trägt es den Namen `dockerimage.yml`. Dieses Skript kann bis auf ein paar Änderungen in das eigene Repository übernommen werden.
### Zeile 1
```yaml
name: hello-world Beispiel
```
Dort muss ein eigener Name angegeben werden
### Zeile 6-10
```yaml
branches:
    - main
paths:
    - "build-context/*"
    - ".github/workflows/dockerimage.yml"
```
Bei **branches** werden die branches angegeben, die GitHub betrachten soll.
Unter **paths** werden die einzelnen Dateien oder Ordner angegeben werden, die GitHub nach Änderungen kontrolliert. Das heißt falls sich eine der angegebenen Dateien unter dem angegebenen Branch verändert, werden die Docker Images neu gebaut und hochgeladen.
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
Damit GitHub die erstellten Docker Images nach Docker Hub hochladen kann, muss sich in Docker Hub eingeloggt werden. Durch die Verbindung von dem entsprechenden Repository in Docker Hub und des GitHub Repositories aus [Schritt 2](#schritt-2) werden die  angegebenen Credentials verwendet. Somit müssen diese Zeilen nicht angepasst werden.
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
Unter `context` muss der jeweilige Ordner, wo die verwendeten Dockerfile Dateien liegen, angegeben werden. Bei **file** wird das benötigte Dockerfile angegeben. Als nächstes gibt es noch **tags**. Hier muss das verwendete Repository und der gewünschte tag für das Docker Image eingegeben werden. In diesem Fall heißt das Repository `hello-world` liegt unter `wagoautomation` und bekommt den tag **arm32v7-** oder **amd64-** und das jeweilige Datum.
### Zeile 98-102
```yaml
- run: |
    docker manifest create wagoautomation/hello-world:${{ steps.global-data.outputs.NOW }} wagoautomation/hello-world:amd64-${{ steps.global-data.outputs.NOW }} wagoautomation/hello-world:arm32v7-${{ steps.global-data.outputs.NOW }}
    docker manifest push wagoautomation/hello-world:${{ steps.global-data.outputs.NOW }}
    docker manifest create wagoautomation/hello-world:latest wagoautomation/hello-world:amd64-${{ steps.global-data.outputs.NOW }} wagoautomation/hello-world:arm32v7-${{ steps.global-data.outputs.NOW }}
    docker manifest push wagoautomation/hello-world:latest
```
Hier werden die Docker Images erstellt und nach Docker Hub hochgeladen. Dafür wird jeweils nach dem create und dem push der Pfad zum eigenen Repository auf Docker Hub angegeben gefolgt von einem Doppelpunkt und den tag für das jeweilige Docker Image.

## Schritt 5 - Skript für das hochladen der README.md
Genau wie in [Schritt 4](#schritt-4) muss das Skript unter dem Pfad `.github/workflows` abgelegt werden. Der Name ist ebenfalls frei zu wählen. In dieser Anleitung trägt es den Namen `dockerhub-description.yml`. Der Aufbau von diesem Skript ähnelt dem für das bauen und hochladen der Docker Images aus [Schritt 4](#schritt-4).
Deswegen müssen auch hier nur einzelne Zeilen bearbeitet werden.
### Zeile 1
```yaml
name: Update Docker Hub Description
```
Der Name kann verändert werden
### Zeile 6-10
```yaml
branches:
    - main
paths:
    - README.md
    - .github/workflows/dockerhub-description.yml
```
Hier wird genau wie in [Schritt 4](#schritt-4) der Branch und die jeweiligen Dateien angegeben, die GitHub nach Änderungen kontrollieren soll. In diesem Fall die `README.md` und das aktuelle Skript. Sobald eine der beiden Dateien sich verändert, wird die `README.md` neu nach Docker Hub hochgeladen.
### Zeile 23
```yaml
repository: wagoautomation/hello-world
```
Unter **repository** muss das eigene Docker Hub Repository angegeben werden, wo die README hochgeladen werden soll.
## Schritt 6 - Automatisierte bauen und hochladen
Nachdem die Schritte 1-4 befolgt und alle Dateien in das GitHub Repository hochgeladen worden sind, beginnt GitHub das Docker Image zu bauen und auf Docker Hub hochzuladen. Solange GitHub arbeitet, wird ein gelber Punkt angezeigt.
![GitHub Bearbeitung](/screenshots/In_Bearbeitung_gross_rot.png?raw=true "GitHub In Bearbeitung")

Den aktuellen Zwischenstand der einzelnen Images kann per klick auf den Punkt angezeigt werden.
![GitHub Zwischenstand](/screenshots/Zwischenstand.png?raw=true "GitHub Aktueller Zwischenstand")

Sobald alle Schritte abgearbeitet und abgeschlossen sind, wird dies mit einen grünen Haken gekennzeichnet.
![GitHub Abgeschlossen](/screenshots/Abgeschlossen_rot.png?raw=true "GitHub Abgeschlossen")

Wenn das hochladen der images auf Docker Hub ebenfalls erfolgreich waren sieht es wie folgt aus.
![Docker Hub Abgeschlossen](/screenshots/DockerHub_Abgeschlossen.png?raw=true "Docker Abgeschlossen")