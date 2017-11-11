# H3

http://terokarvinen.com/2017/aikataulu-palvelinten-hallinta-ict4tn022-3-5-op-uusi-ops-loppusyksy-2017-p5

a) Asenna useita orjia yhteen masteriin. Ainakin yksi rauta- ja useampia virtuaalisia orjia.

b) Kerää tietoa orjista: verkkokorttien MAC-numerot, virtuaalinen vai oikea… (Katso /var/lib/puppet/)

c) OrjaSkripti: Tee skripti, joka muuttaa koneen Puppet-orjaksi tietylle masterille. Voit katsoa mallia Tatun tai Eemelin ratkaisuista.

d) (vapaaehtoinen) Laita skripti Vagrantfile:n provisointiskriptiksi.

e) Vapaaehtoinen: Oikeaa elämää. Ratkaise jokin kurssin ulkopuolinen asia Puppetilla.

f) Vapaaehtoinen: Unelmien tikku. Tee unelmiesi USB-live-tikku.

(Tehtävät tulee raportoida tehdessä, tunnilla tehdyn muisteleminen ei riitä).

## a) Asenna useita orjia yhteen masteriin. Ainakin yksi rauta- ja useampia virtuaalisia orjia.

### Masterin asetukset

Asensin koneelleni puppetmasterin komennolla
`
sudo apt-get -y install puppetmaster
`
Asetin masterin asetukset http://terokarvinen.com/2012/puppetmaster-on-ubuntu-12-04 ohjeen mukaisesti.
Pysäytin masterin ja uudistin ssl -avaimet komennoilla

`
sudo service puppetmaster stop
`
`
sudo rm -r /var/lib/puppet/ssl
`
`
sudo service puppetmaster start
`

Lisäsin dns alt nimet

`sudoedit /etc/puppet/puppet.conf
`

kansioon

`
dns_alt_names = puppet, master, master.local
`

Vaihdoin myös master koneen nimen vastaavaksi.

`
sudo hostnamectl set-hostname master
`

ja kansioon

`
sudoedit /etc/hosts
`

lisäsin käyttäjän perään nimet: master ja master.local

Käynnistin Avahi-daemonin uudestaan komennolla

`
sudo service avahi-daemon restart
`

ja käynnistin terminaalin uudestaan jolloin master käyttäjä tuli näkyviin.

Loin moduulin Hello, World!

`
cd /etc/puppet
`
`
sudo mkdir -p /etc/manifests/ modules/helloworld/manifests/
`
`
sudoedit modules/helloworld/manifests/ini.pp
`

Tiedostoon kirjoitin seeuraavan:

`
class helloworld {
	file { '/tmp/helloFromMaster':
		content => "See you at http://terokarvinen.com/tag/puppet\n"
	}
}`

Lisäksi loin site.pp tiedoston

`
sudoedit manifests/site.pp
`

ja sinne tekstin

`
include helloworld


Testasin moduulin ajamalla sen komennolla
`

sudo puppet apply -e 'class{'helloworld':}'

`
ja ajamalla tervehdyksen
`

cat /tmp/helloFromMaster

`
ja se toimi!
`

### Rauta slave

Otin ssh yhteyden lähiverkossa sijaitsevaan läppäriini, jonka boottasin live-usbilta.
Asensin puppetin
`
sudo apt-get -y install puppet
`

Lisäsin masterin koneen iipparin orjalle
`
sudoedit /etc/hosts
`
`
192.168.1.244	master.local
`

ja puppet conffiin
`
sudoedit /etc/puppet/puppet.conf
`
`
[agent]
server = master.local
`
Käynnistin slave
`
sudo puppet agent --enable
`
`
sudo puppet agent -t
`

Masterilla listasin odottavat slavet
`sudo puppet cert --list`
ja allekirjoitin xubuntu.lan
`
sudo puppet cert --sign xubuntu.lan
`

Hain tiedot masterilta uudestaan komennolla
`s
udo puppet agent -t
`

Puppet jotain mussutti unable...
``
Warning: Unable to fetch my node definition, but the agent run will continue:
Warning: undefined method `include?' for nil:NilClass
``







## f) Vapaaehtoinen: Unelmien tikku. Tee unelmiesi USB-live-tikku.

### Livetikun boottaus RAMiin automaattisesti

Avasin tikkuni filen. KINGSTON, työpäydältä.
Avasin grub.cfg tiedoston, joka löytyi polulta /boot/grub/grub.cfg
Lisäsin "Try Xubuntu without installing" kohdan "quiet splash" perään toram, joka lataa tikun käyttiksen RAM-muistiin.
Nyt Xubuntua voidaan käyttää, ilman että tikku on asetettu koneesee ja se ei myöskään tunnin/session jälkeen unohdu luokkaan.
Toiminnon testaus seuraavassa kappaleessa. 
`SPOILERS! Se toimi END OF SPOILERS`

### Kieliasetuksien kanssa sähläystä (tämän jäkeisessä kappaleessa ratkaisu)

Seuraavaksi lähdin etsimään setusta, jonne voisin lisätä suomenkielisen näppäimistön oletukseksi.
Lupaava tiedosto löytyi polusta: preeseed/scli.seed
Lisäsin 
`# No language support packages.` 
perään tekstin
`d-i	keyboard-configuration/xkb-keymap select fi` 
https://ubuntuforums.org/showthread.php?t=2342872 löytämääni ohjetta mukaillen.

Seuraavaksi buuttasin Linuxin uusiksi, jotta voisin n'hd' ovatko haluamani muutokset tulleet voimaan.
Asennus ruudussa tarkistin @try linuxin@ alta painamalla @e@, l;ytyyk; toram sielt'. Ja l;ytyih'n se.
Kieliasetuksen kanssa ei mennyt ihan yht' hyvin, kuten tekstist' joku tarkkasilm'inen on voinutkin huomata.

Kokeilin uusiksi xubuntu.seed tiedostoon (preeseed/xubuntu.seed)
Lisäsin

```
# Keyboard selection.
# Disable automatic (interactive) keymap detection.
d-i console-setup/ask_detect boolean false
d-i keyboard-configuration/xkb-keymap select fi

### Clock and time zone setup
# Controls whether or not the hardware clock is set to UTC.
d-i clock-setup/utc boolean true
```

Näilläkään muutoksilla ei päästy maaliin.

### Kieli ja kelloasetusten automatisointi

Nyt kokeilin https://ubuntuforums.org/showthread.php?t=1718877 mukaillen lisätä aiemmin muokkaamaani grub.cfg tiedostoon

`locale=fi_FI console-setup/layoutcode=fi`

Grub.cfg tiedoston alku kokonaisuudessaan siis:

```
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
```

Tällä muutoksella bootin jälkeen käytössäni oli pohjoismaalainen keyboard layout ja kellokin näytti oikeaa!
Todisteena ääkköset.

edit: Näin varoituksen sanana, tämä myös muutta käyttöjärjestelmää joiltain osin Suomeksi.

Aikaa tehtävän tekemiseen kului 1h
