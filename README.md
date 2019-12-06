# Welcome to Köy Odası

## Powered By

 - [Jekyll](http://jekyllrb.com)
 - [Bootstrap](http://getbootstrap.com)
 - [Fontawesome](http://fontawesome.io)
 - [DataTables](http://datatables.net)
 - [Highlightjs](https://highlightjs.org)
 - [Animatecss](https://daneden.github.io/animate.css)

## Requirement Packages

### ruby

	sudo apt-get install ruby

### jekyll

	sudo apt-get install jekyll

### gem

	sudo gem install jekyll rdiscount therubyracer

-[ecylmz#rubygems](https://github.com/ecylmz/glove/blob/master/bin/kur#L59)

	# rubygems ssl hatası: https://gist.github.com/luislavena/f064211759ee0f806c88
	wget --no-check-certificate https://github.com/rubygems/rubygems/releases/download/v2.2.3/rubygems-update-2.2.3.gem
	sudo gem install rubygems-update-2.2.3.gem
	sudo update_rubygems
	sudo gem uninstall rubygems-update -x

## Commands

### `Help`: Rakefile Görev Komutları
---

	rake

### `Install`: Site Kurulum İşlemleri
---

	git clone git://github.com/koy-odasi/kernel.git
	cd kernel
	rake install

### `Config`: Yapılandırma Dosyası(_/_config.yml) İşlemleri
---

	rake config

> `İnit`: dosya oluştur ve doldur

	rake config:init

> `Update`: dosyayı uzak depodan güncelle

	rake config:update

> `Destroy`: dosyayı sil

	rake config:destroy

### `Build`: Site Harmanlama İşlemleri
---

	rake build

> `İnit`: harmanla ve deploy kısmına taşı

	rake build:init

> `Destroy`: kurulmamış şekilde sıfırla

	rake build:destroy

### `View`: Demo Önizleme Oluştur
---

	rake view

### `Update`: Uzak Repodan Güncelleme İşlemleri
---

	rake update

> `Site`: site tasarımında güncellemeleri al

	rake update:site

> `Config`: yapılandırma dosyasında güncelleme ile ekleme/silme yap

	rake update:config

### `Deploy`: Site Servis İşlemi
---

	rake deploy

### `Status`: Site Güncel Kontrol İşlemi
---

	rake status

> `Local`: Yerelde güncel veriler var ise repoya yollamak için sor

	rake status:local

> `Remote`: Uzak depoda güncel veriler var ise repoya yollamak için sor

	rake status:remote

## License

Köy Odası is released under the [MIT License](http://www.opensource.org/licenses/MIT).

