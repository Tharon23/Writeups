# AfricanFalls-walkthrough

## Link do wyzwania

https://cyberdefenders.org/blueteam-ctf-challenges/africanfalls

## Narzędzia

* **FTK Imager**
* **Autopsy**
* **Mimikatz**

---

## Weryfikacja integralności

Przed przystąpieniem do pracy nad obrazem dysku zawsze należy sprawdzić jego sumę kontrolną oraz zweryfikować, czy zgadza się ona z wartością podaną w oficjalnej dokumentacji sprawy. Z pliku `DiskDigger.ad1.txt` odczytujemy, że suma kontrolna MD5 wynosi: **9471e69c95d8909ae60ddff30d50ffa1** [Q1]. Jest ona zgodna z oficjalną sumą referencyjną.

![md5checksum](https://github.com/user-attachments/assets/c4837fb7-e686-4028-927e-128b1bd31277)

---

## Przygotowanie środowiska

Aby obraz dysku stał się dostępny dla narzędzi analitycznych, takich jak Autopsy, należy zamontować plik obrazu `DiskDigger.ad1` jako wirtualny dysk w systemie operacyjnym. Idealnym narzędziem do tego zadania jest **FTK Imager**. Po uruchomieniu narzędzia dodajemy obraz do zamontowania i wybieramy metodę **"File System / Read Only"**, aby zapewnić nienaruszalność materiału dowodowego:

<img width="1357" height="837" alt="image" src="https://github.com/user-attachments/assets/2612413a-3a7a-4eb7-b04b-22777e938970" />

---

## Analiza dysku z użyciem Autopsy

Po prawidłowym zamontowaniu obrazu można przejść do właściwej analizy. Uruchamiamy Autopsy i wybieramy opcję **"New Case"**, wskazując nasz nowo zamontowany dysk. Następnie w zakładce **"Select Data Source"** wybieramy **"Local files and folders"** oraz zaznaczamy wszystkie znaczniki czasu (*timestamps*) do uwzględnienia:

<img width="852" height="542" alt="image" src="https://github.com/user-attachments/assets/c3ae9449-48ec-42e8-b72e-90c2fb987eab" />

Przy konfiguracji należy zaznaczyć wszystkie **"Ingest Modules"**, aby Autopsy przeprowadził pełną korelację artefaktów. Po zakończeniu przetwarzania danych możemy przystąpić do badania śladów.

### Podstawowe informacje o systemie
Na początek przyjrzymy się informacjom o systemie i właścicielu urządzenia. Z sekcji **"Operating System Information"** dowiadujemy się, że nazwa urządzenia to `DESKTOP-0JS8C2`, a zainstalowany system to Windows 10 Home. Właścicielem jest **John Doe**:

![os_info](https://github.com/user-attachments/assets/b1750c27-d566-41aa-a235-9f293579180b)

W zakładce **"OS Accounts"** odnajdujemy datę utworzenia konta: **25 kwietnia 2021 r., 19:57:17 CEST**. Widoczna jest tu również pierwsza istotna poszlaka: John Doe na każde pytanie pomocnicze ustawił tę samą odpowiedź: **Roger**. Świadczy to o bardzo niskim poziomie dbałości o bezpieczeństwo konta:

![os_accounts](https://github.com/user-attachments/assets/14bb07e1-e6fc-4ba1-82c9-9c86e1890184)

### Aktywność w sieci
Badanie historii przeglądania wykazało, że użytkownik poszukiwał informacji związanych z łamaniem haseł (fraza: `password cracking list` [Q2], godzina **20:17:38 CEST**). W zakładce **"Web History"** odnajdujemy również adres e-mail w serwisie ProtonMail, na który logował się podejrzany: `dreammaker82@protonmail.com` [Q6]:

![email_address](https://github.com/user-attachments/assets/2e3da1df-eb32-4346-ad02-8bb3ce79b1a9)

Analiza plików **Prefetch** w sekcji **"Run Programs"** wykazała uruchomienie instalatora przeglądarki **Tor Browser** o godzinie **20:22:32 CEST**. Czas ten ściśle zbiega się z wyszukiwaniem informacji o łamaniu haseł. Warto jednak zauważyć, że John Doe nigdy nie uruchomił samej przeglądarki Tor, ponieważ nie odnotowano aktywności procesu `tor.exe` [Q5]:

![tor_activity](https://github.com/user-attachments/assets/e4831123-78d9-43b3-a7f1-c94c1295196a)

### Analiza plików i konsoli
W sekcji **"File Views"** możemy filtrować pliki według rozszerzenia lub typu MIME. W plikach tekstowych odnajdujemy `ConsoleHost_history.txt` – dokument przechowujący historię komend wywołanych w konsoli. Widoczne jest tam polecenie `sdelete` (służące do trwałego usuwania danych) oraz `nmap dfir.science` [Q7], co potwierdza, że użytkownik skanował wspomnianą domenę:

![nmap](https://github.com/user-attachments/assets/f9b3144a-42cc-4d2e-a477-21a48e86f802)

Przechodząc do **"By MIME Type" -> "text" -> "xml"**, odnajdujemy plik `recentservers.xml` wygenerowany przez narzędzie **FileZilla**. Odczytujemy z niego adres IP (`192.168.1.20` [Q3]), z którym łączył się użytkownik na porcie 21 (FTP), korzystając z nazwy użytkownika **'kali'**:

![ip_address](https://github.com/user-attachments/assets/46f19055-6c68-4183-b1cb-3350b677dc6f)

### Kosz systemowy (Recycle Bin)
Sprawdzenie zawartości kosza pozwala odnaleźć pliki, których podejrzany próbował się pozbyć. Znajduje się tam plik zawierający listę haseł do ataków typu *brute-force*. Użytkownik usunął go **29 kwietnia 2021 r. o godzinie 20:22:17 CEST** (18:22:17 UTC [Q4]):

![password_list](https://github.com/user-attachments/assets/ae49aaec-5d49-493c-8e73-906a03196f01)

### Multimedia i geolokalizacja
W zakładce **"Images"** (pod sekcją **"By Extensions"**) odnajdujemy zdjęcie `20210429_152043.jpg`. Metadane EXIF zawierają współrzędne geograficzne (**16 S i 23 E**) oraz dokładny czas wykonania zdjęcia: **29 kwietnia 2021 r., 17:20:43 CEST**. Zidentyfikowano również model urządzenia: **LG-Q725K**:

![20210429_151535](https://github.com/user-attachments/assets/5ad32d0e-73f9-4b4a-8588-fae6670a470d)

Wprowadzenie koordynatów do serwisu mapowego wskazuje na państwo **Zambia** [Q8].

<img width="945" height="465" alt="image" src="https://github.com/user-attachments/assets/fbd98af3-c938-401d-a81b-48fcf1e847df" />

Informację o urządzeniu mobilnym wykorzystujemy do ustalenia ścieżki transferu plików. Analiza artefaktów **Shell Bags** w rejestrze ujawniła folder, z którego użytkownik kopiował zdjęcia na komputer: `\Camera` [Q9]:

![Camera_dict](https://github.com/user-attachments/assets/1d8b9e89-7804-46ab-8ffc-82f3e8c8b140)

---

## Odzyskiwanie haseł

Przy użyciu narzędzia **Mimikatz** wyeksportowano skróty (hashe) dla kont lokalnych z plików rejestru SAM i SYSTEM. Skrót NTLM użytkownika John Doe, po zweryfikowaniu w serwisie *hashes.com*, okazał się odpowiadać prostemu hasłu: **ctf2021** [Q11]:

<img width="1346" height="772" alt="image" src="https://github.com/user-attachments/assets/611ce97b-65c4-4710-8cff-cfe40ff8898f" />

Ten sam serwis pozwolił na złamanie hasła użytkownika Anon, które również było trywialne: **AFR1CA!** [Q10]:

<img width="639" height="330" alt="image" src="https://github.com/user-attachments/assets/d50c4dd3-6e1c-46e5-b2b4-447e13c421eb" />
