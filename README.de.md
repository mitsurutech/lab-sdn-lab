# SDN-Lab für den Unterricht: Open vSwitch, OpenFlow und ein lernender Switch in einer VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml)

Ein fertiges **Lab für Software-defined Networking (SDN)**, um zu unterrichten, wie ein Switch Datenverkehr weiterleitet, von der [MitsuruTech-Lernplattform](https://mitsurutech.co.uk/de/learning), auf der Schulen und Hochschulen jeder lernenden Person eigene Maschinen und ein Terminal im Browser geben. Jede lernende Person erhält eine Ubuntu-**VM** mit **Open vSwitch**, **Mininet** und der OpenFlow-Controller-Bibliothek **os-ken** sowie einen Switch `br0` mit drei Hosts, `h1`, `h2` und `h3`. Der Switch startet mit leerer Flow-Tabelle und leitet daher nichts weiter. Die lernende Person schreibt **OpenFlow-Flows von Hand** mit `ovs-ofctl` und vollendet dann einen **lernenden Switch-Controller**, der lernt, an welchem Port jede MAC-Adresse hängt, und dafür einen Flow installiert. Auf dem Computer der lernenden Person wird nichts installiert, und es wird keine Netzwerkhardware gebraucht.

## Was die lernende Person tut

Das Lab schreibt eine `README.md` in den Home-Ordner, die sagt, was zu tun ist, und keine Lösung verrät, sowie zwei Dateien zum Bearbeiten, `/etc/sdnlab/flows.txt` und `/etc/sdnlab/switch.py`, die bei jedem Start des Labs geladen werden. Die Hosts sind Netzwerk-Namespaces (`h1` ist 10.0.0.1, `h2` 10.0.0.2, `h3` 10.0.0.3, mit MAC-Adressen, die auf 01, 02 und 03 enden), und der Switch steht im fail-mode secure: Ohne Flow verwirft er jedes Paket. In drei Teilen:

1. **Mininet** (nicht bewertet): `sudo mn --mac --test pingall`, dann eine Mininet-Eingabeaufforderung, um Knoten, Links und die Flows zu betrachten, die sein Controller installiert.
2. **Die Flow-Tabelle von Hand schreiben** in `flows.txt`: h1 und h2 müssen sich in beide Richtungen erreichen (ARP ebenso wie das Echo), ein Flow, der auf h3 passt, muss es verwerfen, und kein Flow darf die Aktion `NORMAL` verwenden.
3. **Einen Controller vollenden** in `switch.py`: drei TODO-Schritte in `packet_in`, um die Quelladresse zu lernen, den Port zu wählen und einen Flow zu installieren, sodass die folgenden Pakete einer Unterhaltung den Controller nie erreichen.

Das Lab fügt einen Befehl `sdnlab` hinzu (`status`, `flows`, `controller`, `reset`, `shell h1`), um das Lab neu aufzubauen, den Controller zu starten, beide Dateien auf den ausgelieferten Stand zurückzusetzen und eine Shell in einem Host zu öffnen.

## Wie es bewertet wird

Das Lab hat eine **Prüfung**, ein Skript, das eine abgegebene VM bewertet, ohne dass die Lehrkraft sie lesen muss. Es läuft auf einer Kopie der gespeicherten Maschine nach einem frischen Start und lädt die beiden Dateien der lernenden Person selbst: Ein von Hand getippter, aber nie gespeicherter Flow zählt nicht.

- **Teil 2, ohne Controller:** h1 und h2 müssen sich erreichen, h3 darf niemanden erreichen und niemand h3, und ein Flow, der auf h3 passt, muss es verwerfen und von Paketen von h3 getroffen worden sein. Ein Flow mit der Aktion `NORMAL` besteht nicht.
- **Teil 3, der Controller der lernenden Person:** Die Prüfung startet ihn, gibt h2 eine MAC-Adresse, die niemand gesehen hat, sodass eine vorab geschriebene Tabelle nichts nützt, und sendet Pings zwischen allen Paaren. Der Switch muss unbekannte Pakete an den Controller schicken, und der Controller muss für jedes Adresspaar einen Flow installiert haben, geschrieben als `priority=1,in_port=PORT,dl_src=SOURCE_MAC,dl_dst=DESTINATION_MAC` mit der Aktion Ausgabe an den gelernten Port, und keiner der installierten Flows darf fluten.

Die erste ausgegebene Zeile sagt, was nicht stimmt.

Was sie nicht tut: Sie testet mit Ping und liest die Flow-Tabelle, keine anderen Arten von Verkehr, und eine lernende Person mit root auf der eigenen VM könnte die Werkzeuge, auf die sich die Prüfung stützt, immer noch manipulieren. Es ist eine Prüfung für den Unterricht, kein Beweis.

## Was in der Datei steht

`sdn-lab.lab.yaml` ist eine vollständige **VM-Lab-Definition**, genau das, was die Plattform ausführt:

| Schlüssel | Was es ist |
| --- | --- |
| `cpus`, `memory`, `disk` | die Größe der VM jeder lernenden Person: 1 CPU, 1024 MB und 10 GB |
| `open` | die Ports, die im Browser geöffnet werden: keine, das Lab ist das Terminal |
| `setup` | ein Skript, das einmal als root läuft, beim ersten Start der VM jeder lernenden Person: Es installiert Open vSwitch, Mininet, os-ken, `iproute2`, `ping` und `tcpdump` aus dem Ubuntu-Archiv und schreibt den Befehl `sdnlab`, einen kleinen Starter für den Controller, die beiden Dateien zum Bearbeiten, eine systemd-Unit und die README |
| `check` | das Bewertungsskript, als root auf einer Kopie der gespeicherten Platte nach einem frischen Start ausgeführt; sein Exit-Status ist das Ergebnis, `0` heißt bestanden |

## Auf der Plattform nutzen

**So geben Sie es einer Klasse als Aufgabe:**

1. Fügen Sie in Ihrem Modul eine Aufgabe hinzu und wählen Sie die Karte **Ubuntu-VM-Lab**.
2. Klicken Sie auf das Beispiel Software-defined networking oder fügen Sie den Inhalt von `sdn-lab.lab.yaml` ein. Ein Beispiel ersetzt den Text, und Rückgängig holt Ihren zurück.
3. Veröffentlichen Sie und probieren Sie zuerst Ihre eigene Kopie aus. Jede lernende Person hat eine eigene VM, die zwischen den Sitzungen erhalten bleibt. Der erste Start dauert einige Minuten, solange das Setup die Pakete installiert.
4. Weisen Sie es wie jede Aufgabe zu. Nach der Abgabe führen Sie die Prüfung in der Zeile einer lernenden Person in der Klassenliste aus: Sie startet eine Kopie der gespeicherten Maschine, die nicht zurückgespeichert wird, und zeigt, ob sie bestanden wurde und was sie ausgegeben hat.

Eine Basics-Lektion der Plattform, SDN Basics, wird in diesem Lab bearbeitet. Die [Seite zu diesem Lab](https://mitsurutech.co.uk/de/help/lab-sdn-lab) im Handbuch, der [Leitfaden zu Ubuntu-VM-Labs](https://mitsurutech.co.uk/de/help/labs) und die [Seite mit den Beispiel-Labs](https://mitsurutech.co.uk/de/help/sample-labs) sagen den Rest.

Noch kein Konto für Ihre Einrichtung? [Das erste Jahr ist kostenlos](https://mitsurutech.co.uk/de/pricing) für bis zu 50 Lernende, und die [Registrierung](https://mitsurutech.co.uk/de/signup) braucht nur eine E-Mail-Adresse.

## Die Datei prüfen

Bei jedem Push prüft `.github/workflows/check.yml`, dass die Datei gültiges YAML mit den oben genannten Schlüsseln ist und dass `setup` und `check` jeweils unter dem 16-KB-Limit der Plattform liegen. Es startet keine VM: Das geschieht auf den Maschinen der Plattform.

## Was in einem Lab nicht funktioniert

Die VM erreicht das Paketarchiv von Ubuntu, PyPI, npm und GitHub, sonst nichts. Das Netz zwischen den Hosts ist virtuell und liegt in der VM, nichts verlässt sie. Die Bibliothek os-ken hängt von eventlet ab, das veraltet ist und im Log des Controllers einen Hinweis ausgibt; deshalb lehrt das Lab zuerst `ovs-ofctl` und Mininet. `sudo mn -c` von Mininet entfernt jede Open-vSwitch-Bridge, auch `br0`: Führen Sie danach `sudo sdnlab flows` aus, um das Lab neu aufzubauen. Setup und Prüfung wurden in einem Container ausgeführt, der die VM ersetzt; ein Versuch auf einer echten VM einer lernenden Person lohnt sich vor einer Stunde.

## Über uns

Entwickelt von [Mitsuru Technologies](https://mitsurutech.co.uk/de) für die [MitsuruTech-Lernplattform](https://mitsurutech.co.uk/de/learning): Python, Java, Go, Node, SQL und Linux im Browser für Schulen und Hochschulen, mit Container-Labs, VM-Labs, aus GitHub bereitgestellten Web-Apps und Quizzen, die sich selbst bewerten. Probieren Sie Code ohne Konto im [Playground](https://mitsurutech.co.uk/de/playground).

Weitere Labs vom selben Ort: [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea), [lab-sample](https://github.com/mitsurutech/lab-sample), [lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab), [lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab), [lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab), [lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab), [lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab), [lab-ansible-roles](https://github.com/mitsurutech/lab-ansible-roles) und [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics).

## Lizenz

Dieses Werk steht unter der Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Sie dürfen es für Lernen und Lehre teilen und anpassen, wenn Sie Mitsuru Technologies Ltd nennen, es nicht kommerziell nutzen und alles, was Sie daraus machen, unter derselben Lizenz weitergeben. Der vollständige Text steht in der Datei `LICENSE`.

[Open vSwitch](https://www.openvswitch.org), Mininet und os-ken werden beim Start einer VM aus dem Ubuntu-Archiv installiert; nichts davon wird hier verteilt.
