minimize
========

Simple (or not so simple) script to minimize and gzip all the js and css files of a directory, and change between the minimized and unminimized versions easily
Also creates the bundle files given a config file.


**The yuicompressor must be installed and must be in the path with a wrapper script, like this one:**

    #!/bin/bash
    
    java -jar /opt/yuicompressor/yuicompressor-2.4.7.jar "$@"

help
--------

 > minimize -h

 Usage: $0 [-d] [-p user:group] command options 
 
    Sript to minimize and bundle js and css files using yuicompressor
    (yuicompressor must be installed and in the PATH for this script to work).
    Uses hardlinks to copy the files, and leaves a copy of the file 
    unminimized, and minimized like this:
 
    file.js
    file.js.gz
    file.js.minimized
    file.js.unminimized
 
    So later if you want to restore the unminimized fils you just have to
    restore them from the unminmized file (or use the unminimize option).
 
    By default it also creates a gzipped version of the file to be served
    later. An example of apache configuration to do this will be something
    like this:
    ----------------------
    AddEncoding gzip 
    RewriteEngine On
    ## If the browser accepts gzip
    RewriteCond %{HTTP:Accept-Encoding} gzip
    ## MSIE is fine, but masquerades as Netscape
    RewriteCond %{HTTP_USER_AGENT} \\bMSIE [S=1]
    ## Netscape has problems with gzip
    RewriteCond %{HTTP_USER_AGENT} ^Mozilla/4\.0[678]
    ## Only js and css files allowed (if more add them here)
    RewriteCond %{REQUEST_URI} ^.*\.(js|css)$
    ## And serve it only if the files exists...
    RewriteCond %{REQUEST_FILENAME}.gz -f 
    RewriteRule (.*)$ /\$1.gz [L,E=vary]
    ## Avoid th proxies to send gzipped content to clients that did
    ## not send the accept: gzip header
    Header append Vary User-Agent env=vary
 
    ## Force the content-type header
    <FilesMatch \.js.gz>
        ForceType text/javascript
    </FilesMatch>
    <FilesMatch \.css.gz>
        ForceType text/css
    </FilesMatch>
    -------------------------
 
    Global options:
    -d
        Enable Debug mode.
 
    -p user:group
        use this user and group for the generated files. (by default current
        user and group)
 
 
    Commands:
 
    bundle [-b basepath] [-c config_file]
        Create the bundles of the files specified in the config files.
 
        Options:
        -b basedir
            When reading the bundle file, this directory (if given) will be
            appended to all the paths
 
        -c config_file
            Use the given config file for the bundles. The config file must
            have this structure:
 
            /path/to/generated/bundle
                /path/to/js/or/css/file/to/bundle
                /path/to/another/file/to/bundle
 
            /path/to/another/generated/bundle
                /path/to/another/file
                ...
 
        Example:
            Create the bundles defined in the file config/bundles.txt and use
            the /var/www directory as basepath: 
 
            $> $0 -p apache:apache bundle -b /var/www -c config/bundles.txt
        
    generate [-t regexp] [directory [...]]
        Create the minimized version of the scripts.
 
        Options:
        -t regexp
            Only minimize the files that match the given regexp (by default all
            the *.js  and *.css files are matched)
 
        directory
            If given will minimize all the js and css (or the given regexp)
            under that directory. If not, will use the local directory.
 
        Example:
            Minimize all the *.js and *.css files under my project dir, and
            create the gzipped versions:
 
            $> $0 -p apache:apache generate /var/www/myproject
 
 
    set [-t regexp] [directory [...]]
        Set up the minimized versions (redo the hard link to pint to the
        .minimized version), also regenerate the gzipped files.
 
        Options:
        -t regexp
            Only minimize the files that match the given regexp plus the
            '.unminimized' suffix (by default all the *.unminimized  files are
            matched)
 
        directory
            Search in the given directory for unminimized files (the local
            dir by default)
            
 
    unset [-t regexp] [directory [...]]
        Set up the unminimized versions of the files (redo the hard link to
        point to the .unminimized version), elso regenerate the gzipped files.
 
        Options:
        -t regexp
            Only minimize the files that match the given regexp plus the
            '.unminimized' suffix (by default all the *.unminimized  files are
            matched)
 
        directory
            Search in the given directory for unminimized files (the local
            dir by default)
