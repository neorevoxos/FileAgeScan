FileAgeScan – Scan und Archivierung alter Dateien
=================================================

Zweck
-----
Dieses Paket besteht aus zwei PowerShell-Scripts:

1. FileAgeScan.ps1
   durchsucht definierte Pfade nach alten Dateien anhand von Dateitypen und Alter

2. Move-FileAgeResultsFromConfig.ps1
   liest die Ergebnis-CSV aus dem Scan ein und verschiebt die gefundenen Dateien in ein Archiv – inklusive Ordnerstruktur


1. Komponenten
---------------

FileAgeScan.ps1
Dieses Script:
- liest eine Konfigurationsdatei ein
- durchsucht definierte Pfade rekursiv
- filtert nach Dateitypen
- filtert nach Alter
- schreibt eine Logdatei
- exportiert optional alle Treffer in eine Ergebnis-CSV

Move-FileAgeResultsFromConfig.ps1
Dieses Script:
- liest eine separate Archiv-Konfigurationsdatei ein
- verwendet die Ergebnis-CSV aus dem Scan
- kann einen Testlauf oder einen echten Move-Lauf ausfuehren
- verschiebt Dateien in einen Archivordner
- behaelt die Ordnerstruktur relativ zum RootPath bei
- schreibt ein Log
- schreibt zusaetzlich eine Archiv-Result-CSV mit Status pro Datei


2. Empfohlene Ordnerstruktur
----------------------------
C:\Status\FileAgeScan\
C:\Status\FileAgeScan\Config\
C:\Status\FileAgeScan\Logs\
C:\Status\FileAgeScan\Results\
C:\Status\FileAgeScan\Results\Archive\
C:\Status\FileAgeScan\ArchiveLogs\
C:\Status\FileAgeScan\FileAgeScan.ps1
C:\Status\FileAgeScan\Move-FileAgeResultsFromConfig.ps1
C:\Status\FileAgeScan\Config\FileAgeScanConfig.csv
C:\Status\FileAgeScan\Config\FileAgeArchiveConfig.csv

Archivziel zum Beispiel:
D:\Archiv\


3. Script 1 – FileAgeScan.ps1
-----------------------------
Aufgabe
FileAgeScan.ps1 durchsucht einen oder mehrere definierte Pfade nach Dateien, die:
- zu bestimmten Dateitypen gehoeren
- aelter als X Jahre sind

Standardverhalten
Das Script verwendet standardmaessig:
- C:\Status\FileAgeScan\Config\FileAgeScanConfig.csv
- C:\Status\FileAgeScan\Logs
- C:\Status\FileAgeScan\Results\FileAgeScanResults.csv

Start
Einfacher Start:
C:\Status\FileAgeScan\FileAgeScan.ps1

Mit Detail-Log:
C:\Status\FileAgeScan\FileAgeScan.ps1 -DetailedLog

Optional ueberschreiben
Beispiel mit anderem Config-Pfad:
C:\Status\FileAgeScan\FileAgeScan.ps1 -ConfigCsvPath "D:\Config\FileAgeScanConfig.csv"


4. Scan-Config – FileAgeScanConfig.csv
--------------------------------------
Datei
C:\Status\FileAgeScan\Config\FileAgeScanConfig.csv

Aufbau
Name;RootPath;Extensions;OlderThanYears;DateProperty;Enabled
NETSTORE1_Dokumente;\\NETSTORE1\Freigabe;.pdf|.docx|.xlsx;10;LastWriteTime;true

Bedeutung der Spalten
Name
Freier Name des Scan-Jobs.
Beispiel: NETSTORE1_Dokumente
Dieser Name erscheint in Log und Ergebnis-CSV.

RootPath
Der zu durchsuchende Basisordner.
Beispiel: \\NETSTORE1\Freigabe
Das Script durchsucht diesen Pfad rekursiv inklusive Unterordner.

Extensions
Dateitypen, die beruecksichtigt werden sollen.
Beispiel: .pdf|.docx|.xlsx
Moeglich sind auch einzelne Typen: .pdf

OlderThanYears
Das Mindestalter in Jahren.
Beispiel: 10
Dann werden nur Dateien beruecksichtigt, die aelter als 10 Jahre sind.

DateProperty
Welcher Zeitstempel fuer die Alterspruefung verwendet wird.
Moegliche Werte:
- CreationTime
- LastWriteTime
- LastAccessTime
Empfehlung: LastWriteTime

Enabled
Steuert, ob der Job aktiv ist.
Moegliche Werte:
- true
- false


5. Script 2 – Move-FileAgeResultsFromConfig.ps1
------------------------------------------------
Aufgabe
Move-FileAgeResultsFromConfig.ps1 liest die Ergebnis-CSV aus dem Scan ein und verarbeitet jede Datei gemaess Archiv-Config.

Das Script kann:
- Test
  - nur pruefen und loggen
  - nichts verschieben
- Move
  - Dateien echt verschieben

Start
Alle aktivierten Jobs ausfuehren:
C:\Status\FileAgeScan\Move-FileAgeResultsFromConfig.ps1

Nur einen bestimmten Archivjob ausfuehren:
C:\Status\FileAgeScan\Move-FileAgeResultsFromConfig.ps1 -OnlyJobName "NETSTORE1_Test"
oder:
C:\Status\FileAgeScan\Move-FileAgeResultsFromConfig.ps1 -OnlyJobName "NETSTORE1_Move"


6. Archiv-Config – FileAgeArchiveConfig.csv
-------------------------------------------
Datei
C:\Status\FileAgeScan\Config\FileAgeArchiveConfig.csv

Aufbau
Name;ResultCsvPath;ArchiveRoot;LogDirectory;ResultLogCsvPath;OnExisting;Mode;Enabled
NETSTORE1_Test;C:\Status\FileAgeScan\Results\FileAgeScanResults.csv;D:\Archiv;C:\Status\FileAgeScan\ArchiveLogs;C:\Status\FileAgeScan\Results\Archive\NETSTORE1_Test_ArchiveResults.csv;Skip;Test;true
NETSTORE1_Move;C:\Status\FileAgeScan\Results\FileAgeScanResults.csv;D:\Archiv;C:\Status\FileAgeScan\ArchiveLogs;C:\Status\FileAgeScan\Results\Archive\NETSTORE1_Move_ArchiveResults.csv;Skip;Move;false

Bedeutung der Spalten
Name
Freier Name des Archiv-Jobs.

ResultCsvPath
Pfad zur Ergebnis-CSV aus dem Scan-Script.
Beispiel: C:\Status\FileAgeScan\Results\FileAgeScanResults.csv

ArchiveRoot
Basisordner fuer das Archiv.
Beispiel: D:\Archiv

LogDirectory
Ordner fuer die Archiv-Logs.
Beispiel: C:\Status\FileAgeScan\ArchiveLogs

ResultLogCsvPath
Pfad zur Archiv-Result-CSV.
Diese CSV protokolliert jede verarbeitete Datei mit Status.
Beispiel: C:\Status\FileAgeScan\Results\Archive\NETSTORE1_Move_ArchiveResults.csv

OnExisting
Verhalten, wenn die Zieldatei bereits existiert.
Moegliche Werte:
- Skip
- Overwrite
- Rename

Mode
Modus des Jobs.
Moegliche Werte:
- Test
- Move

Enabled
Aktiviert oder deaktiviert den Archiv-Job.
- true
- false


7. Typischer Ablauf
-------------------
Schritt 1 – Scan ausfuehren
C:\Status\FileAgeScan\FileAgeScan.ps1

Ergebnis:
- Log im Ordner C:\Status\FileAgeScan\Logs
- Ergebnisdatei C:\Status\FileAgeScan\Results\FileAgeScanResults.csv

Schritt 2 – Archiv-Testlauf
In FileAgeArchiveConfig.csv:
- NETSTORE1_Test = true
- NETSTORE1_Move = false

Dann ausfuehren:
C:\Status\FileAgeScan\Move-FileAgeResultsFromConfig.ps1 -OnlyJobName "NETSTORE1_Test"

Ergebnis:
- Log im Ordner C:\Status\FileAgeScan\ArchiveLogs
- Archiv-Result-CSV:
  C:\Status\FileAgeScan\Results\Archive\NETSTORE1_Test_ArchiveResults.csv

Schritt 3 – Test pruefen
Pruefen:
- Quell- und Zielpfade
- Anzahl der Dateien
- moegliche Konflikte
- uebersprungene Dateien
- Status in der Archiv-Result-CSV

Schritt 4 – Echter Archivlauf
In FileAgeArchiveConfig.csv umstellen:
- NETSTORE1_Test = false
- NETSTORE1_Move = true

Dann ausfuehren:
C:\Status\FileAgeScan\Move-FileAgeResultsFromConfig.ps1 -OnlyJobName "NETSTORE1_Move"


8. Beispiele
-------------
Beispiel Scan-Config nur fuer PDF
Name;RootPath;Extensions;OlderThanYears;DateProperty;Enabled
NETSTORE1_PDF;\\NETSTORE1\Freigabe;.pdf;10;LastWriteTime;true

Beispiel Scan-Config fuer mehrere Dokumenttypen
Name;RootPath;Extensions;OlderThanYears;DateProperty;Enabled
NETSTORE1_Dokumente;\\NETSTORE1\Freigabe;.pdf|.docx|.xlsx;10;LastWriteTime;true

Beispiel Archiv-Config
Name;ResultCsvPath;ArchiveRoot;LogDirectory;ResultLogCsvPath;OnExisting;Mode;Enabled
NETSTORE1_Test;C:\Status\FileAgeScan\Results\FileAgeScanResults.csv;D:\Archiv;C:\Status\FileAgeScan\ArchiveLogs;C:\Status\FileAgeScan\Results\Archive\NETSTORE1_Test_ArchiveResults.csv;Skip;Test;true
NETSTORE1_Move;C:\Status\FileAgeScan\Results\FileAgeScanResults.csv;D:\Archiv;C:\Status\FileAgeScan\ArchiveLogs;C:\Status\FileAgeScan\Results\Archive\NETSTORE1_Move_ArchiveResults.csv;Skip;Move;false


9. Ergebnisdateien
------------------
FileAgeScanResults.csv
Enthaelt alle gefundenen Dateien aus dem Scan.
Typische Spalten:
- JobName
- RootPath
- FullName
- Name
- Extension
- SizeBytes
- SizeFormatted
- CreationTime
- LastWriteTime
- LastAccessTime
- UsedDate
- OlderThanYears
- DateProperty

Archiv-Result-CSV
Enthaelt alle bearbeiteten Dateien des Archiv-Scripts.
Typische Spalten:
- ConfigName
- ResultJobName
- Mode
- Status
- SourceRootPath
- SourceFile
- RelativePath
- TargetFile
- SizeBytes
- Message
- RunTimestamp

Moegliche Statuswerte
- TEST
- MOVED
- SKIPPED
- FAILED


10. Wichtige Hinweise
---------------------
1. Erst immer Testlauf
Vor echtem Verschieben zuerst immer mit Mode=Test arbeiten.

2. LastWriteTime meist am sinnvollsten
CreationTime kann durch Kopieren veraendert sein.
LastWriteTime ist fuer Archiventscheidungen meistens besser.

3. ArchiveRoot im Move-Modus
Im Modus Move muss das Archivziel beschreibbar sein.

4. Dateien werden im Move-Modus an der Quelle entfernt
Move bedeutet echtes Verschieben, nicht Kopieren.

5. Relative Ordnerstruktur bleibt erhalten
Beispiel:
Quelle:
\\NETSTORE1\Freigabe\Alt\2012\Bericht.pdf

RootPath:
\\NETSTORE1\Freigabe

Archivziel:
D:\Archiv\NETSTORE1_Dokumente\Alt\2012\Bericht.pdf


11. Fehleranalyse
-----------------
Config-Datei wird nicht gefunden
Pruefen:
- stimmt der Pfad?
- existiert die Datei?
- ist das Trennzeichen ; ?

Kein Treffer im Scan
Pruefen:
- RootPath korrekt?
- Dateiendungen korrekt?
- Enabled = true?
- Age/DateProperty passend?

Archivlauf ueberspringt Dateien
Moegliche Gruende:
- Quelldatei existiert nicht mehr
- Result-CSV enthaelt unvollstaendige Daten
- Zieldatei existiert bereits und OnExisting=Skip
- relative Pfadbildung fehlgeschlagen

Encoding-Probleme
Scripts moeglichst als UTF-8 mit BOM speichern.
Wenn es Parserprobleme mit Umlauten gibt, ASCII-Texte verwenden.


Empfehlung
----------
Empfohlener Betriebsablauf:
1. Scan ausfuehren
2. Ergebnis-CSV pruefen
3. Archiv-Testlauf ausfuehren
4. Archiv-Result-CSV pruefen
5. echten Move-Job aktivieren
6. Archivlauf ausfuehren

So bleibt der Vorgang nachvollziehbar und kontrolliert.
