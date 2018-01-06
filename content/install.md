# PL/Swift Installation

PL/Swift should install fine on pretty much any Unix system that can run
PostgreSQL. Including exotic setups like Raspberry Pi systems.

We also provide a macOS Homebrew tap which makes it really easy to install
PL/Swift and its dependencies on macOS. We highly recommend that over a
custom install.

On the Linux side we test w/ Ubuntu Trusty and Xenial, though it should work
pretty much anywhere.

## Install on macOS using Homebrew

Got no Homebrew? [Get it!](https://brew.sh)

Then add the PL/Swift tap and install plswift:

    brew tap PL-Swift/plswift
    brew install plswift

## Install on Linux (or macOS w/o Homebrew)

On macOS: We strongly advise that you rather use Homebrew, more importantly
          the Apache provided by Homebrew.

Ubuntu packages required (assuming you have Swift 3 installed already), this includes
the PostgreSQL and SQLite3 database adaptors, add additional ones as desired:

    sudo apt-get update
    sudo apt-get install \
       curl pkg-config

TODO: test & add actual packages required

Install PL/Swift:

    curl -L -o plswift.tgz \
         https://github.com/PL-Swift/plswift/archive/0.0.4.tar.gz
    tar zxf plswift.tgz && cd plswift-0.0.4
    make
    sudo make install

That puts PL/Swift into `/usr/local`. If you want to have it in `/usr`, do:

    sudo make prefix=/usr install

## Check whether the installation is OK

You can call `swift pl validate` to make sure the installation is OK:

    The Swift PostgreSQL build environment looks sound.
    
      srcroot:   /Users/helge/dev/Swift/PLSwift/PLSwift
      module:    PLSwift
      config:    debug
      product:   /Users/helge/dev/Swift/PLSwift/PLSwift/.build/PLSwift.so
      version:   
      sql-setup: 
      pg_config: /Applications/Postgres.app/Contents/Versions/9.4/bin/pg_config
      moddir:    /Applications/Postgres.app/Contents/Versions/9.4/lib/postgresql
      extdir:    /Applications/Postgres.app/Contents/Versions/9.4/share/postgresql/extension/
      PL/Swift:  /usr/local
      swift:     4.0.3
    
    ERROR: Missing extension control file: PLSwift.control
    ERROR: Missing setup file: 

(you can ignore the ERRORs at the bottom, they are only relevant within
 extensions)

## Troubleshooting

If something isn't working in a Homebrew setup, check whether:

    brew doctor

outputs anything unusual.

If you need any help, feel free to ask on the
[Mailing List](https://groups.google.com/d/forum/apacheexpress)
or our
[Slack channel](http://slack.noze.io).


