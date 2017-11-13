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

```
cd /etc/puppet

sudo mkdir -p /etc/manifests/ modules/helloworld/manifests/

sudoedit modules/helloworld/manifests/ini.pp
```

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

### Rautaorja

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

Tästä huolimatta testi toimi:

`cat /tmp/helloFromMaster`

## Vituaaliorjat

Tein tehtävän käyttäen avuksi http://terokarvinen.com/2017/multiple-virtual-computers-in-minutes-vagrant-multimachine ohjetta.

Asensin Vagrantin

`sudo apt-get -y install virtualbox vagrant`

Virtualboxin kanssa ongelmia

`● virtualbox.service - LSB: VirtualBox Linux kernel module
   Loaded: loaded (/etc/init.d/virtualbox; bad; vendor preset: enabled)
   Active: failed (Result: exit-code) since la 2017-11-11 11:28:12 UTC; 2ms ago
     Docs: man:systemd-sysv-generator(8)
  Process: 15938 ExecStart=/etc/init.d/virtualbox start (code=exited, status=1/FAILURE)
`

Googletuksella selvisi, että UEFI boot aiheuttaa tämän ongelman.
Boottasin koneen uudestaan.
BIOSsissa muutin boot asetuksia (ASUS)
F7 Advanced mode
Boot
Boot Device Control
Legacy OPROM only

Palasin BIOSsin etusivulle ja painoin F8 (Boot Menu)
Valitsin tikkuni ilman UEFI vaihtoehtoa

Kokeilin asennusta uudestaan (nyt siis tuoreella live-xubuntulla)
Tällä kertaa asennus ei kiukutellut mitään.

Loin Vagrantfile

`nano Vagrantfile`

ja lisäsin sinne ohjeen mukaiset asetukset

```
# http://TeroKarvinen.com/
Vagrant.configure(2) do |config|
 config.vm.box = "bento/ubuntu-16.04"

 config.vm.define "slave01" do |slave01|
   slave01.vm.hostname = "slave01"
 end

 config.vm.define "slave02" do |slave02|
   slave02.vm.hostname = "slave02"
 end
end
```
Tämän jälkeen annoin komennon

`vagrant up`

jolloin virtuaalikoneiden asennus lähti käyntiin.

asennus kuitenkin tyssäsi.

```
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["startvm", "f763f996-c185-49f3-bef7-3c01bdc22a59", "--type", "headless"]

Stderr: VBoxManage: error: VT-x is disabled in the BIOS for all CPU modes (VERR_VMX_MSR_ALL_VMX_DISABLED)
VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component ConsoleWrap, interface IConsole
```

KOrjasin virtualisointiasetukset BIOSsissa seuraavin askelin

F7 (Advanced Options)
Advanced
CPU Configuration
Intel Virtualization technology -> Enabled

Jälleen tuoreeella live-xubuntulla uusi yritys.

### Virtuaaliorjien asetukset

Virtuaalikoneet kaatuivat minulla melkein saman tien.

Esnimmäiselle yritin ajaa install puppet komennon, joka tyssäsi Unpacking kohtaan.
Yrittäeäni kirjautua toiselle, sain viestin "no space left"

Tämän jälkeen koko Xubuntu päätti irtisanoa sopimuksen irti, eikä vastannut enää millään lailla.
Rapsaa, joka minulla oli tuossa koneessa auki, en pystynyt enää päivittämään (commit changes), jonka johdosta menetin jonkin verran sitä ja siksi tässä myöskin lyhyhennetty kertomus tapahtuneesta.

### Uusi yritys virtuaaliorjien kanssa

Yritin asettaa virtuaali orjat uusiksi, tällä kertaa siten, että asennan virtuaalikoneet MacBook Pro läppäriini, jossa on Xubuntu 16.04 LTS käyttis.
Puppetmasteria käitin pöytäkoneellani live-usbilla.
Asetukset olivat täysin identtiset edellisen kanssa, joten hyppään suoraan kohtaan jossa sorvaan virtuaaliorjia.

Otin ssh yhteyden slave01 komennolla

`vagrant ssh slave01`

Asensin puppetin.
Konffasin puppet asetukset, sekä lisäsin master koneeni iipparin hosts -tiedostoon

`sudoedit /etc/hosts
192.168.1.244 master`

`sudoedit /etc/puppet/puppet.conf

[agent]
server = master
`
Ajoin komennot

`
sudo puppet agent --enable
sudo puppet agent -t
`

Master koneella listasin odottavat orjat ja signasin slave01.lan

`
sudo puppet cert --list
sudo puppet cert --sign slave01.lan
`
Ajoin orjalla uudestaan komennon

`
sudo puppet agent -t
`

Tästä kuitenkin virheilmoitus:

`
Error: Could not request certificate: Server hostname 'master' did not match server certificate; expected one of xubuntu.lan, DNS:puppet, DNS:puppet.lan, DNS:xubuntu.lan
`

Lisäsin masterin puppet.conf tiedostoon dns_alt_names.. master.local

`
dns_alt_names = puppet, master, master.local, master.lan
`

Lisäsin myös orjalle hosts kansioon tuon master.lan

Pysäytin molemmat, poistin ssl avaimet ja käynnistin uudestaan.

`
sudo rm -r /var/lib/puppet/ssl
`

Listasin ja signasin masterilla slave01 avaimen uudestaan.

Orjalla komennolla

`sudo puppet agent -t
`

sain helloworld -moduulin masterilta seuraavasta virheilmoituksesta huolimatta

```
Warning: Unable to fetch my node definition, but the agent run will continue:
Warning: undefined method `include?' for nil:NilClass
```

konffasin seuraavaksi toisen virtuaaliorja (slave02) sekä rautaorjan tuttuun tapaan.

Tämä sujui ongelmitta. Testasin vielä Hello, World moduulin toimivuuden molemmissa.

Yhteensä tämän tehtävän tekemiseen minulla meni kahden päivän aikana n. 4h

## b) Kerää tietoa orjista: verkkokorttien MAC-numerot, virtuaalinen vai oikea… (Katso /var/lib/puppet/)

Komento 

`sudo cd /var/lib/puppet/
`

Ei toimi. Googlen mukaan siksi, että sudoa käytetään vain ajettaessa ohjelmia ja "cd" ei ole ohjelma.

Ajoin komennon

`sudo -i`

jolloin minusta tuli root -käyttäjä ja pääsin cd -komennolla puppet kansioon.

/var/lib/puppet/ kansiosta en näitä tietoja löytänyt.

Tehtävän kestoon aikaa käytetty 1h

## c) OrjaSkripti: Tee skripti, joka muuttaa koneen Puppet-orjaksi tietylle masterille. Voit katsoa mallia Tatun tai Eemelin ratkaisuista.

Askartelin moduulin slavemaker https://github.com/einokupias/puppetslaves





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
