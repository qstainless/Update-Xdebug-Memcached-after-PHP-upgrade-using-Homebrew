# How to update Xdebug / Memcached after a PHP upgrade via Homebrew

A few months ago I started experimenting with Xdebug and Memcached. The initial installation was not as smooth as I had hoped. Once I got everything up and running, however, both packages broke whenever I upgraded PHP via Homebrew. After a lot of trial and error, I finally found a way to streamline the update for both Xdebug and Memcached. It works for me, at least, so hopefully it will help someone else facing the same issues I did.

### Assumptions

- macOS Monterey installed (tested with Version 12.5, but should work with earlier versions)
- [Homebrew Package Manager](https://brew.sh)
- PHP 8.0.22 installed via Homebrew (`brew install php@8.0`). The `pecl` command should come along with PHP when installing via Homebrew.

### Steps

1. Switch to PHP 8.0.x. When upgrading Homebrew packages, if you have previously installed PHP via `brew install php`, upgrading will automatically switch to the latest version of PHP. 

        $ brew unlink php
        $ brew link --overwrite --force php@8.0

2. Go to new PHP installation folder. I am using the Homebrew default install path. Replace `<php ver>` with your current PHP 8.0.x version e.g. `8.0.22`.

        $ cd /usr/local/Cellar/php@8.0/<php ver>

3. Delete the `pecl` symlink, if it exists.

        $ sudo rm -dfr pecl

4. Uninstall xdebug. Skip this step if this is the first time you are installing xdebug.

        $ pecl uninstall xdebug

5. Install xdebug.

        $ pecl install xdebug

6. Edit `php.ini`.

        $ nano /usr/local/etc/php/8.0/php.ini

7. Update the path to `xdebug.so` and save your changes. This is the default path used by the xdebug installation. Replace `<php ver>` with your current PHP 8.0.x version e.g. `8.0.22`.

        zend_extension="/usr/local/Cellar/php@8.0/<php ver>/pecl/20200930/xdebug.so"

8. Install `zlib` and `libmemcached` via Homebrew, if not already installed. These two libraries are required by Memcached.

        $ brew install zlib libmemcached

9. Uninstall memcached. Skip this step if this is the first time you are installing memcached.

        $ pecl uninstall memcached

10. Install memcached.

        $ pecl install memcached

11. Answer 'Yes' to the prompts for libmemcached and zlip directories. Use the defaults for all other prompts.

        libmemcached directory [no] : yes
        zlib directory [no] : yes
        use system fastlz [no] :
        enable igbinary serializer [no] :
        enable msgpack serializer [no] :
        enable json serializer [no] :
        enable server protocol [no] :
        enable sasl [yes] :
        enable sessions [yes] :

12. Edit `php.ini` again.

        $ nano /usr/local/etc/php/8.0/php.ini

13. Update path to `memcached.so` and save your changes. This is the default path used by the memcached installation. Replace `<php ver>` with your current PHP 8.0.x version e.g. `8.0.22`.

        extension="/usr/local/Cellar/php@8.0/<php ver>/pecl/20200930/memcached.so"

14. Restart PHP, Memcache, and Apache.

        $ brew services restart php@8.0
        $ brew services restart memcached
        $ brew services restart httpd

15. Verify that xdebug is installed and running:

        $ php -v
        PHP 8.0.22 (cli) (built: Aug  4 2022 14:22:56) ( NTS )
        Copyright (c) The PHP Group
        Zend Engine v4.0.22, Copyright (c) Zend Technologies
            with Xdebug v3.1.5, Copyright (c) 2002-2022, by Derick Rethans
            with Zend OPcache v8.0.22, Copyright (c), by Zend Technologies

16. Verify that memcached is installed and running (you may have other services running besides those shown below):

        $ brew services
        Name      Status  User   File
        httpd     started <user> ~/Library/LaunchAgents/homebrew.mxcl.httpd.plist
        memcached started <user> ~/Library/LaunchAgents/homebrew.mxcl.memcached.plist
        php@8.0   started <user> ~/Library/LaunchAgents/homebrew.mxcl.php@8.0.plist
