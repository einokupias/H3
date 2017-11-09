# H3

http://terokarvinen.com/2017/aikataulu-palvelinten-hallinta-ict4tn022-3-5-op-uusi-ops-loppusyksy-2017-p5

a) Asenna useita orjia yhteen masteriin. Ainakin yksi rauta- ja useampia virtuaalisia orjia.

b) Kerää tietoa orjista: verkkokorttien MAC-numerot, virtuaalinen vai oikea… (Katso /var/lib/puppet/)

c) OrjaSkripti: Tee skripti, joka muuttaa koneen Puppet-orjaksi tietylle masterille. Voit katsoa mallia Tatun tai Eemelin ratkaisuista.

d) (vapaaehtoinen) Laita skripti Vagrantfile:n provisointiskriptiksi.

e) Vapaaehtoinen: Oikeaa elämää. Ratkaise jokin kurssin ulkopuolinen asia Puppetilla.

f) Vapaaehtoinen: Unelmien tikku. Tee unelmiesi USB-live-tikku.

(Tehtävät tulee raportoida tehdessä, tunnilla tehdyn muisteleminen ei riitä).

##f) Vapaaehtoinen: Unelmien tikku. Tee unelmiesi USB-live-tikku.

Avasin tikkuni filen. KINGSTON, työpäydältä.
Avasin grub.cfg tiedoston, joka löytyi polulta /boot/grub/grub.cfg
Lisäsin "Try Xubuntu without installing" kohdan "quiet splash" perään toram, joka lataa tikun käyttiksen RAM-muistiin.
Nyt Xubuntua voidaan käyttää, ilman että tikku on asetettu koneesee ja se ei myöskään tunnin/session jälkeen unohdu luokkaan.

Seuraavaksi lähdin etsimään setusta, jonne voisin lisätä suomenkielisen näppäimistön oletukseksi.
Lupaava tiedosto löytyi polusta: preeseed/scli.seed
Lisäsin "# No language support packages." perään tekstin
"d-i	keyboard-configuration/xkb-keymap select fi" https://ubuntuforums.org/showthread.php?t=2342872 löytämääni ohjetta mukaillen.
