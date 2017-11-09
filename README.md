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

#Livetikun boottaus RAMiin automaattisesti

Avasin tikkuni filen. KINGSTON, työpäydältä.
Avasin grub.cfg tiedoston, joka löytyi polulta /boot/grub/grub.cfg
Lisäsin "Try Xubuntu without installing" kohdan "quiet splash" perään toram, joka lataa tikun käyttiksen RAM-muistiin.
Nyt Xubuntua voidaan käyttää, ilman että tikku on asetettu koneesee ja se ei myöskään tunnin/session jälkeen unohdu luokkaan.
Toiminnon testaus seuraavassa kappaleessa. 
`SPOILERS! Se toimi END OF SPOILERS`

#Kieliasetuksien kanssa sähläystä (tämän jäkeisessä kappaleessa ratkaisu)

Seuraavaksi lähdin etsimään setusta, jonne voisin lisätä suomenkielisen näppäimistön oletukseksi.
Lupaava tiedosto löytyi polusta: preeseed/scli.seed
Lisäsin `# No language support packages.` perään tekstin
`d-i	keyboard-configuration/xkb-keymap select fi` https://ubuntuforums.org/showthread.php?t=2342872 löytämääni ohjetta mukaillen.

Seuraavaksi buuttasin Linuxin uusiksi, jotta voisin n'hd' ovatko haluamani muutokset tulleet voimaan.
Asennus ruudussa tarkistin @try linuxin@ alta painamalla @e@, l;ytyyk; toram sielt'. Ja l;ytyih'n se.
Kieliasetuksen kanssa ei mennyt ihan yht' hyvin, kuten tekstist' joku tarkkasilm'inen on voinutkin huomata.

Kokeilin uusiksi xubuntu.seed tiedostoon (preeseed/xubuntu.seed)
Lisäsin
`
# Keyboard selection.
# Disable automatic (interactive) keymap detection.
d-i console-setup/ask_detect boolean false
d-i keyboard-configuration/xkb-keymap select fi

### Clock and time zone setup
# Controls whether or not the hardware clock is set to UTC.
d-i clock-setup/utc boolean true
`
Näilläkään muutoksilla ei päästy maaliin.

#Kieli ja kelloasetusten automatisointi

Nyt kokeilin https://ubuntuforums.org/showthread.php?t=1718877 mukaillen lisätä aiemmin muokkaamaani grub.cfg tiedostoon lisätä

`locale=fi_FI console-setup/layoutcode=fi`

Grub.cfg tiedoston alku kokonaisuudessaan siis:

`
if loadfont /boot/grub/font.pf2 ; then
	set gfxmode=auto
	insmod efi_gop
	insmod efi_uga
	insmod gfxterm
	terminal_output gfxterm
fi

set menu_color_normal=white/black
set menu_color_highlight=black/light-gray

menuentry "Try Xubuntu without installing" {
	set gfxpayload=keep
	linux	/casper/vmlinuz.efi  file=/cdrom/preseed/xubuntu.seed boot=casper quiet splash toram --- locale=fi_FI console-setup/layoutcode=fi
	initrd	/casper/initrd.lz
  `
  
Tällä muutoksella bootin jälkeen käytössäni oli pohjoismaalainen keyboard layout ja kellokin näytti oikeaa!
Todisteena ääkköset.
