## WordClock - Software basiert, im Kiosk-Modus ##

Hier wird eine Software-Lösung für eine Wordclock beschrieben sowie die Installation einer Kiosk-Umgebung.
Letztere dient dazu, den Raspberry nach dem booten eine Web-Seite aufrufen zu lassen.

In der ct -Hardware Hacks 3/2013 wurde die WordClock als Projekt beschrieben.
Diese Lösung gefällt mir sehr gut, aber die Kosten betragen immer noch ca 150,- EUR.

    http://www.tns-labs.org/wordclock-eigenbau/
    http://www.1-2-do.com/de/projekt/Wordclo...auen/4356/


 
Die Alternative hier basiert auf folgenden Komponenten:

    Raspberry  30 EUR
    Der existiert bei mir für eine 433MHz Steckdosen-Steuerung
    15 Zoll Monitor mit DVI-Eingang (gibt es bei ebay ab 1,- EUR + Versand)
    HTML -Code (s.u.)
    Kiosk Umgebung
        xorg (X window system)
        nodm (minimal display manager)
        browser (minimal webkit browser)


Pro

    Billig
    Sehr schnell realisiert


Cons

    Design der ct Variante wesentlich besser
    Stromverbrauch höher (Raspberry + Monitor)


Live-Demo: hier.
![](http://www.hoeser-medien.de/pictures/wordclock_hw.jpg)


Für den Kiosk wird ein abgespeckter X-Server mit einem einfachen Display Manager und dem schlanken webkit-Browser verwendet.

Installation

Step 1) Browser erzeugen & installieren
Die Wordclock benötigt einen browser, der webkit unterstützt
Chromium frisst zu viele Resourcen und Midori bietet nicht alle Funktionen.
Es gibt einen einfachen Browser: kiosk-browser von Peter Schultz auf github.

    gcc -o browser browser.c $(pkg-config --libs --cflags gtk+-2.0 webkit-1.0)
    sudo cp browser /usr/local/bin

Step 2) Setup & install xorg und nodm; Siehe auch Anleitung von nexxylove
    sudo apt-get update; sudo apt-get dist-upgrade
    sudo apt-get install xorg nodm

Step 3) Den Benutzer "kiosk" anlegen und in die relevanten Gruppen aufnehmen (www-data?).

    sudo useradd -g www-data kiosk
    passwd kiosk
    sudo usermod -a -G www-data kiosk
    cd /home
    sudo mkdir kiosk
    sudo chown kiosk kiosk

Step 5) Als kiosk Benutzer anmelden.

Step 6) Folgende Datei anlegen: nano ~/.xsession
Nach dem Login des Benutzters "kiosk" wird eine X-Session gestarten, die den Browser "browser" aufruft.
Hier kann man auch eine belibige web-Seite aufrufen

    #!/bin/bash
    /usr/local/bin/browser  file:///var/www/clock.html

Step 7) Den display-manager konfigurieren: sudo nano /etc/default/nodm
Die Option "-s 0 dpms" schaltet den Screen-Saver aus.
    NODM_ENABLED=true
    NODM_USER=kiosk
    NODM_X_OPTIONS='-nolisten tcp -s 0 dpms'

Step 6) sudo nano /etc/X11/default-display-manager
Dann wird noch in /etc/X11/default-display-manager statt /usr/sbin/lightdm einfach /usr/sbin/nodm  eingetragen

    /usr/sbin/nodm
    
Step 8) Auto login - hier wird der Benutzer "kiosk" automatisch nach dem botten eingeloggt - ohne Password-Abfrage
    sudo nano /etc/inittab
    Das getty - Programm auskommentieren - d.h. folgende Zeile suchen:
    1:2345:respawn:/sbin/getty 115200 tty1
    
    Vorne ein '#' einfügen
    #1:2345:respawn:/sbin/getty 115200 tty1Folgende Zeile darunter hinzufügen:
    Code: Alles markieren
	1:2345:respawn:/bin/login -f kiosk tty1 tty1 /dev/tty1 2>&1

Step 9)  Die HTML-Datei clock.html file nach /var/www/ kopieren.
    sudo mkdir -p /var/www
    sudo chown www-data:www-data /var/www
    sudo chmod 775 /var/www
    
Step 10)  Den raspberry neu starten
    sudo reboot
    
X-Server mit einem einfachen Display Manager und dem schlanken webkit-Browser.



----------


Meiner Frau war der 17 Zoll TFT zu groß. Pech gehabt.

Die Alternative war ein 15 Zoll TFT.
Dieser hatte jedoch kein DVI Eingang, sondern nur den alten VGA Anschluß.

Über folgende Box kann ich den Monitor an dem HDMI Port des Raspberry betreiben:



http://www.ebay.de/itm/281171946607?ssPa...1497.l2649
eBay Shop http://stores.ebay.de/shs201313?_trksid=p2047675.l2563

![](http://thomas.hoeser-medien.de/pictures/wordclock_hw_vga.jpg)

Pros

    Ein gebrauchter 15 Zoll ist billiger
    Meine Frau ist zufrieden (unbezahlbar)


Cons

    Die Box kostet ca. 10 EUR
    Zusätzliche Kabel



Danach muss die Konfigurationsdatei /boot/config.txt angepasst werden, um den VGA Adapter verwenden zu können.
Beschreibung der Parameter unter http://elinux.org/RPiconfig.

sudo nano /boot/config.txt
    
    # uncomment if you get no picture on HDMI for a default "safe" mode
    # hdmi_safe=1
    
    # uncomment this if your display has a black border of unused pixels visible
    # and your display can output without overscan
    #disable_overscan=1
    
    # uncomment the following to adjust overscan. Use positive numbers if console
    # goes off screen, and negative if there is too much border
    #overscan_left=16
    #overscan_right=16
    #overscan_top=16
    #overscan_bottom=16
    
    # uncomment to force a console size. By default it will be display's size minus
    # overscan.
    #framebuffer_width=1280
    #framebuffer_height=720
    
    # uncomment if hdmi display is not detected and composite is being output
    hdmi_force_hotplug=1
    
    # uncomment to force a specific HDMI mode (this will force VGA)
	# hdmi_group=2 (DMT)
	# hdmi_mode=16   1024x768  60Hz
    hdmi_group=2
    hdmi_mode=16
    
    # uncomment to force a HDMI mode rather than DVI. This can make audio work in
    # DMT (computer monitor) modes
    hdmi_drive=2
    
    # uncomment to increase signal to HDMI, if you have interference, blanking, or
    # no display
    config_hdmi_boost=4
    
    # uncomment for composite PAL
    sdtv_mode=2

	#uncomment to overclock the arm. 700 MHz is the default.
	#arm_freq=800

	# for more options see http://elinux.org/RPi_config.txt

Danach den raspberry neu starten


    sudo reboot


Die Datei clock.htlm musste ich auf die Größe des 15 Zoll Monitors anpassen.

