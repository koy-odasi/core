#!/usr/bin/ruby
# encoding: utf-8

require 'yaml'

SOURCE_DIR = "_"
POST_DIR = "_posts"
EXTRACT_DIR = "_site"
HOST = "localhost"
PORT = "4000"

BRANCHS = %w(master gh-pages)
CONFIGFILE = "#{SOURCE_DIR}/_config.yml"
DEFAULTS  = "
# Site Configuration

title:
keywords:
description:
baseurl:
url:

# Jekyll Configuration

permalink: /categories/:categories/:title/
highlighter: pygments
markdown: rdiscount
rdiscount:
  extensions: [smart]

# Owner Configuration

author:
editor:
email:
cv:
disqus:
github:
trello:
linkedin:
googleanalyticsid:
twitter:
facebook:
instagram:
foursquare:
stack:
"

task :default do
  puts "Görevler: #{(Rake::Task.tasks - [Rake::Task[:default]]).join(', ')}"
end

# Her türlü kurulum yaparız abi, makarna yemeyiz abi, kebap yeriz abi
task :install => "config:init" do
  # yeni bir kurulum mu, git kaynak dosyası ilk haliyle mi duruyor?
  if `git config remote.origin.url`.chomp =~ %r{[:/]koy-odasi/core.git}

    # yapılandırma ayarlarını getir
    config = YAML.load_file(CONFIGFILE)

    # git dosyasını düzenleyelim
    puts "Şimdi Git kaynak dosyası düzenlenecek."

    # GitHub kullanıcı ismi sor
    ask_user = "GitHub Kullanıcı İsmi veya Organizasyon Adı: "
    user = if ([nil, '']).include?(user = config_get('github'))
             get_stdin(ask_user)
           else
             ask_default(ask_user, user)
           end

    # GitHub üzerinde sitenin depolanacağı ismi sor
    repo = ask_default("GitHub Üzerinde Sitenin Bulundurulacağı Depo İsmi: ", "#{user}.github.io")

    # GitHub üzerinde sitenin depolanacağı isme göre branch seç
    if (repo.match(/[\w-]+\.github\.(?:io|com)/).nil?)
      branch = 'gh-pages'

      # Url olarak sitenin barınacağı user, repoyu ekle
      url = "https://#{user}.github.io/#{repo}"

      # Baseurl olarak repo yolu ekle
      config_set 'baseurl', "/#{repo}"
    else
      branch = 'master'

      # Url olarak sitenin barınacağı user ekle
      url = "https://#{user}.github.io"
    end

    # GitHub üzerinde sitenin bulundurulacağı depo yolu
    repo_path = "https://github.com/#{user}/#{repo}"

    # yapılandırma dosyasında işlem yapmadan önce bilgilendirme yap
    puts \
    "
    #{CONFIGFILE} üzerinde aşağıdaki kısımlar güncellenecek.

    GitHub kullanıcı adı

      → #{user}

    GitHub üzerinde sitenin bulundurulacağı depo yolu

      → #{repo_path}

    GitHub üzerinde sitenin bulunduğu deponun branchı

      → #{branch}

    Sitenin servis edileceği link

      → #{url}

    [ ! ] :
    1 - Lütfen aşağıdaki depoyu oluşturduğunuzdan emin olun.
    2 - Depoya hiçbir dosya eklemeyin. (Ör.: README)
    (Oluşturmak için: https://github.com/new)

       → #{repo_path}
    "
    abort "İşlem sona erdi" unless ask_yesno "Kurulum için devam etmek istiyor musunuz?", true

    # GitHub kullanıcı ismi ata
    config_set 'github', user

    # GitHub repo için brancha geç ve url kaydet
    if branch == "gh-pages"
      switch_branch("gh-pages")
      config_set 'url', "https://#{user}.github.io/#{repo}"
    else
      switch_branch("master")
      config_set 'url', "https://#{user}.github.io"
    end

    # siteyi harmanla
    Rake::Task['build:init'].execute

    # branch'a yolla
    `git remote rename origin upstream >/dev/null`
    `git remote add origin git@github.com:#{user}/#{repo}`
    `git add .`
    `git commit -am 'ilk'`
    `git push -u origin #{branch}`
  end
end

# Yerel veya uzak depoda güncel veriler var ise repoya yollamak için sor
task :status => ["status:remote", "status:local"]
namespace :status do

  # yerelde güncel veriler var ise repoya yollamak için sor
  task :local => :install do
    if (file_changes = `git status --porcelain`) != ""
      puts "Aşağıdaki dosyalarda ekleme / güncelleme / silme mevcut.\n#{file_changes}"
      abort "Repo güncellenmedi." unless ask_yesno "Güncel verileri sitenize göndermek istiyor musunuz?", true
      # siteyi yükle, deploy'un bağımlıklarıyla
      Rake::Task['deploy'].invoke
    end
  end

  # uzak depoda güncel veriler var ise repoya yollamak için sor
  task :remote => :install do
    if `git status | grep -qi 'git push' && echo "true"` == "true\n"
      abort "Repo güncellenmedi." unless ask_yesno "Uzak depodan güncel verileri sitenize göndermek istiyor musunuz?", true
      # siteyi yükle, deploy'un bağımlıklarıyla
      Rake::Task['deploy'].invoke
    end
  end
end

# Siteyi repoya yollamak için sıfırla ve harmanla
task :build => ["build:destroy", "build:init"]
namespace :build do

  # siteyi repoya yollamak için harmanla ve deploy kısmına taşı
  task :init => "config:init" do

    puts "#{SOURCE_DIR} dizinine giriş yapılıyor."

    # jekyll çalıştır EXTRACT_DIR dizininde deploy edilecek dosyaları taşı ve dizini sil
    chdir SOURCE_DIR do
      puts "jekyll çalıştırılıp oluşturulan #{EXTRACT_DIR}/ dizini altındaki kodlar, bir üst dizine taşınacak ve #{EXTRACT_DIR} dizini silinecek"

      `jekyll build`
      `mv #{EXTRACT_DIR}/* ../ 2>/dev/null`
      `rm -rf #{EXTRACT_DIR}`
    end
  end

  # siteyi kurulmamış şekilde sıfırla
  task :destroy do

    # dokunulmayacak varsayılan dosyalar
    escape_parameters = %w(Rakefile LICENCE CNAME README.md .nojekyll .gitignore) + [SOURCE_DIR]

    # dokunulmayacak dosya harici her şeyi sil
    Dir['*'].each do |directory|
      unless escape_parameters.include? directory
        `rm -rf #{directory} 2>/dev/null`
      end
    end
  end
end

# Yapılandırma dosyası oluştur ya da yapılandırma dosyasını ayarla
task :config => ["config:init", "config:update"]
namespace :config do

  # yeni kurulumlarda bir ayar dosyası oluştur
  task :init do
    unless File.exist? CONFIGFILE
      $stderr.puts "Ayar dosyası bulunamadı, öntanımlı bir dosya oluşturuluyor."
      File.open(CONFIGFILE, 'w') { |f| f.write(DEFAULTS) }
      puts "Lütfen #{CONFIGFILE} ayar dosyasını düzenleyin:"
      config_set
    end
  end

  # ayar dosyasını sil
  task :destroy do
    if File.exist? CONFIGFILE
      abort "Ayar dosyası silinmedi." unless ask_yesno "#{CONFIGFILE} ayar dosyasını silmek istiyor musunuz?", false
      `rm -rf #{CONFIGFILE}`
    end
  end

  # site yapılandırmalarını düzenle
  task :update => :install do

    # yapılandırma dosyasını güncelle
    config_set

    # yerelde siteyi güncelledik mi?
    Rake::Task['status:local'].execute
  end
end

# Uzak repodan, dosyaları ya da yapılandırma dosyasını güncelle
task :update => ["update:remote", "update:config"]
namespace :update do

  # depo güncellemeleri
  task :remote => :install do

    # uzak depo yoksa ekle
    if `git config remote.upstream.url`.chomp.empty?
      `git remote add upstream https://github.com/koy-odasi/core.git`
    end

    # uzak depodan güncelle
    `git pull upstream master`

    # siteyi güncelledik mi?, status'un bağımlıklarıyla
    Rake::Task['status'].invoke
  end

  # yapılandırma güncellemeleri
  task :config => :install do
    config, default = YAML.load_file(CONFIGFILE), YAML.load(DEFAULTS)
    keys_add, keys_del = default.keys - config.keys, config.keys - default.keys
    keys_del.each { |key| config.delete(key) } unless keys_del.empty?
    keys_add.each { |key| config[key] = ask_default(key, default[key].to_s) } unless keys_add.empty?

    # sırala DEFAULTS key sırasına göre sırala
    new_config = {}
    default.each { |key, value| new_config[key] = config[key] } unless keys_add.empty?

    # CONFIGFILE üzerinde eklemeler veya çıkarmaları uygula
    File.open(CONFIGFILE, 'w') { |f| f.write(new_config.to_yaml) } if keys_add.any? or keys_del.any?

    # siteyi güncelledik mi?, status'un bağımlıklarıyla
    Rake::Task['status'].invoke
  end
end

# Site ön izleme
task :view => "config:init" do
  chdir SOURCE_DIR do
    fork do
      `while ! nc -z #{HOST} #{PORT}; do sleep 0.1; done; xdg-open http://#{HOST}:#{PORT}/`
    end
    server_pid = Process.spawn('jekyll serve')
    trap('INT') do
      [server_pid].each { |pid| Process.kill(9, pid) rescue Errno::ESRCH }
      exit 0
    end
    [server_pid].each { |pid| Process.wait(pid) }
  end
end

# Yeni bir gönderi oluşturma
task :new => :install do
  url_title = ask_default("Gönderi url başlığını girin:", "hello-world")
  title = ask_default("Gönderinin başlığını girin:", url_title.gsub('-', ' ').split(' ').map(&:capitalize).join(' '))
  filename = "#{SOURCE_DIR}/_posts/#{Time.now.strftime('%Y-%m-%d')}-#{url_title}.md"
  if File.exists? filename
    abort "İşlem sona erdi" unless ask_yesno "#{filename} mevcut. Üzerine yazmak istiyor musunuz?", false
  end
  puts \
    "
    Şimdi #{filename} kaynak dosyası düzenlenecek.

    Aşağıdaki kısmı değiştirmeyin:

      → layout: post

    Aşağıdaki kısmı isterseniz güncelleyebilirsiniz:

      → title: #{title}

    Lütfen düzenlemeniz tamamlandıktan sonra aşağıdaki
    bilgileri eklemeyi unutmayın:

      → category:
      → tags:
    "
  ask_default("Devam etmek için herhangi bir tuşa basın...", "Devam")
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: #{title}"
    post.puts "category: "
    post.puts "tags: "
    post.puts "---"
  end

  # bir şekilde editör kullanmalıyız
  if ([nil, '']).include?(editor = config_get('editor'))
    config_set "editor", ask_default("Kullandığınız Editör İsmi: ", "gedit")
  end

  # dosyamızı yapılandırma dosyasında belirtilen editörle aç
  sh editor, filename
end

# Siteyi repoya yolla
task :deploy => [:install, :build] do
  # depoya yolla
  `git add .`
  `git add -u`
  `git commit -am '#{Time.now.utc}'`
  `git push --force`
end

# Siteye gönderilenlerin içerik ve isimlerine göre arama yap göster Ör.: $ rake find["linux"]
task :find, :word do |task, args|
  word = args[:word]

  chdir "#{SOURCE_DIR}/#{POST_DIR}" do
    filenames = `find . -name "*#{word}*"`

    # büyük/küçük karakter uyumluluğunu denetlemeden gösterme parametresi : -i
    # sadece içerik olarak ismi geçen dosyaları tekrarsız gösterme parametresi: -l
    filecontents = `grep -i --color=always #{word} *`
    puts \
    "
      Aranan kelime → #{word}

      Benzer dosya isimleri → \n#{filenames}

      İçerik olarak benzer dosya isimleri → \n#{filecontents}
    "
  end
end

# Sitede bulunan gönderilerden en son düzenleneni aç
task :last do

  # bir şekilde editör kullanmalıyız
  if ([nil, '']).include?(editor = config_get('editor'))
    config_set "editor", ask_default("Kullandığınız Editör İsmi: ", "gedit")
  end

  chdir "#{SOURCE_DIR}/#{POST_DIR}" do
    # son düzenlenen dosyanın ismini bul, son satırdaki gereksiz gelen `?` satırı chop ile sil
    # kaynak : https://stackoverflow.com/questions/4561895/how-to-recursively-find-the-latest-modified-file-in-a-directory
    filename = `find . -type f -printf '%T@ %p\n' \ | sort -n | tail -1 | cut -f2- -d" "`.chop

    # dosyamızı yapılandırma dosyasında belirtilen editörle aç
    sh editor, filename
  end
end

# Yerel fonksiyonlar

def ask_yesno message = 'Devam?', default = true, validate = %w(e h)
  reply = ''
  choise = default ? validate.first : validate.last
  reply = ask_default("#{message} (#{validate.join("/")}) ", choise) until validate.include? reply.downcase
  reply.downcase == validate.first ? true : false
end

def ask_default message, default
  print "#{message} |#{default}| "
  return default if (['\n', '']).include?(reply = STDIN.gets.strip)
  return reply
end

def get_stdin message
  print message
  STDIN.gets.chomp
end

def switch_branch branch
  `git checkout -b #{branch} >/dev/null`
  `git branch -D #{BRANCHS-[branch]} 2>/dev/null`
end

# CONFIGFILE dosyası üzerindeki bir değişkenin değerini getir
def config_get key
  config = YAML.load_file(CONFIGFILE)[key]
end

# CONFIGFILE dosyası üzerinde işlemler:
# → bir değişkene atama yap
# → tüm değişkenlere atama yap (izin verilenlere)
def config_set key = nil, value = nil, default = nil
  config = YAML.load_file(CONFIGFILE)
  unless key and value
    config.each do |key, value|
      config[key] = ask_default(key, value) unless %w(permalink highlighter markdown rdiscount baseurl).include? key
    end
  else
    config[key] = default.nil? ? value : ask_default(key, value)
  end
  File.open(CONFIGFILE, 'w') { |f| f.write(config.to_yaml) }
end
