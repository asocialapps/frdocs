## Problèmes d'installation
Quand on lance la commande `bundle install` on récupère l'erreur:

    Installing wdm 0.1.1 with native extensions
    Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

Il faut installer wdm:

    gem install wdm
    // Relever la version installée : ici 0.2.0

Puis il faut corriger le Gemfile:

    # Performance-booster for watching directories on Windows
    gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]

On peut relancer:

    bundle install
    bundle exec jekyll serve

## Changement de skin
Dans Gemfile:

    # Pour utiliser le skin dark
    gem "jekyll-remote-theme", "~> 0.4.3"

Dans `_config.yml`:

    remote_theme: jekyll/minima
    
    minima:
      skin: dark

Puis:

    bundle install
    bundle exec jekyll serve

## Build et test (local)

    bundle exec jekyll build
    xcopy /E _site ..\public\fr\
    cd ..
    npx http-server -p 8080

    URL: http://localhost:8080/fr
    
## Paths sous Windows

    RubyGems Environment:
      - RUBYGEMS VERSION: 3.5.16
      - RUBY VERSION: 3.3.5 (2024-09-03 patchlevel 100) [x64-mingw-ucrt]
      - INSTALLATION DIRECTORY: C:/Users/Daniel/.local/share/gem/ruby/3.3.0
      - USER INSTALLATION DIRECTORY: C:/Users/Daniel/.local/share/gem/ruby/3.3.0
      - RUBY EXECUTABLE: C:/Ruby33-x64/bin/ruby.exe
      - GIT EXECUTABLE: C:\Program Files\Git\cmd/git.EXE
      - EXECUTABLE DIRECTORY: C:/Users/Daniel/.local/share/gem/ruby/3.3.0/bin
      - SPEC CACHE DIRECTORY: C:/Users/Daniel/.cache/gem/specs
      - SYSTEM CONFIGURATION DIRECTORY: C:/ProgramData
      - RUBYGEMS PLATFORMS:
        - ruby
        - x64-mingw-ucrt
      - GEM PATHS:
        - C:/Users/Daniel/.local/share/gem/ruby/3.3.0
        - C:/Ruby33-x64/lib/ruby/gems/3.3.0
      - GEM CONFIGURATION:
        - :update_sources => true
        - :verbose => true
        - :backtrace => true
        - :bulk_threshold => 1000
        - "gem" => ["--install-dir", "C:/Users/Daniel/.local/share/gem/ruby/3.3.0", "--bindir", "C:/Users/Daniel/AppData/Local/Microsoft/WindowsApps"]
        - "# Feel free to add any rubygems config options as described on" => nil
      - REMOTE SOURCES:
        - https://rubygems.org/
      - SHELL PATH:
        - C:\Ruby33-x64\bin
        - C:\Program Files\Common Files\Oracle\Java\javapath
        - C:\WINDOWS\system32
        - C:\WINDOWS
        - C:\WINDOWS\System32\Wbem
        - C:\WINDOWS\System32\WindowsPowerShell\v1.0\
        - C:\WINDOWS\System32\OpenSSH\
        - C:\Users\Daniel\AppData\Roaming\nvm
        - C:\Program Files\nodejs
        - C:\Program Files\Git\cmd
        - C:\Program Files (x86)\Google\Cloud SDK\google-cloud-sdk\bin
        - C:\Program Files (x86)\Certbot\bin
        - C:\Program Files\apache-maven-3.9.5\bin
        - C:\Program Files\JetBrains\IntelliJ IDEA Educational Edition 2022.2.2\bin
        -
        - C:\Certbot\bin
        - C:\Python\Scripts\
        - C:\Python\
        - C:\Users\Daniel\AppData\Local\Microsoft\WindowsApps
        - C:\Users\Daniel\AppData\Roaming\nvm
        - C:\Program Files\nodejs
        - C:\Users\Daniel\AppData\Local\Programs\Microsoft VS Code\bin
        - C:\Program Files\JetBrains\IntelliJ IDEA 2023.2.3\bin
        -
        - C:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2023.2.4\bin
        -
        - c:\Tools
