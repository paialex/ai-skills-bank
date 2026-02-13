# AEM Dispatcher Configuration Reference

Comprehensive reference for Dispatcher configuration patterns.

## Filter Rules Reference

### Security-First Filter Configuration

```apache
/filter {
    # ==== DENY ALL BY DEFAULT ====
    /0001 { /type "deny" /glob "*" }

    # ==== ALLOW SITE CONTENT ====
    # Allow GET requests for site pages
    /0100 { /type "allow" /method "GET" /url "/content/mysite/*" }

    # Allow GET for language-specific content
    /0101 { /type "allow" /method "GET" /url "/content/mysite/*/en/*" }
    /0102 { /type "allow" /method "GET" /url "/content/mysite/*/de/*" }
    /0103 { /type "allow" /method "GET" /url "/content/mysite/*/fr/*" }

    # ==== ALLOW STATIC ASSETS ====
    # Allow DAM assets
    /0200 { /type "allow" /method "GET" /url "/content/dam/mysite/*" }

    # Allow ClientLibs
    /0201 { /type "allow" /method "GET" /url "/etc.clientlibs/*" }

    # Allow design assets
    /0202 { /type "allow" /method "GET" /url "/etc/designs/*" }

    # ==== ALLOW SPECIFIC APIs ====
    # Allow model.json for headless
    /0300 { /type "allow" /method "GET" /url "/content/mysite/*.model.json" }

    # Allow specific servlets
    /0301 { /type "allow" /method "GET" /url "/content/mysite/*/_jcr_content.search.json" }

    # ==== BLOCK DANGEROUS PATTERNS ====
    # Block infinity JSON (content dump)
    /0900 { /type "deny" /url "*.infinity.json" }
    /0901 { /type "deny" /url "*.tidy.json" }
    /0902 { /type "deny" /url "*.sysview.xml" }
    /0903 { /type "deny" /url "*.docview.json" }
    /0904 { /type "deny" /url "*.docview.xml" }

    # Block query builder
    /0910 { /type "deny" /url "*query*" }
    /0911 { /type "deny" /url "*.query.json" }

    # Block system paths
    /0920 { /type "deny" /url "/bin/*" }
    /0921 { /type "deny" /url "/system/*" }
    /0922 { /type "deny" /url "/apps/*" }
    /0923 { /type "deny" /url "/libs/*" }
    /0924 { /type "deny" /url "/tmp/*" }
    /0925 { /type "deny" /url "/var/*" }
    /0926 { /type "deny" /url "/crx/*" }
    /0927 { /type "deny" /url "/home/*" }
    /0928 { /type "deny" /url "/etc/*" }

    # Block specific selectors
    /0930 { /type "deny" /selectors '(feed|rss|pages|languages|blueprint|hierarchyJson|proxy|form)' }

    # Block specific extensions
    /0940 { /type "deny" /extension '(json|xml)' /path "/content/*" /selectors '(infinity|tidy|sysview|docview|-1|0|1|2|3|4|5|6|7|8|9)' }
}
```

### Allow Specific HTTP Methods

```apache
/filter {
    # Allow POST for forms (with specific paths)
    /0400 { /type "allow" /method "POST" /url "/content/mysite/*/jcr:content.form.html" }

    # Allow OPTIONS for CORS preflight
    /0401 { /type "allow" /method "OPTIONS" /url "/api/*" }
}
```

## Cache Configuration

### Basic Cache Rules

```apache
/cache {
    /docroot "/opt/aem/dispatcher/cache"

    # Cache HTML, assets, and clientlibs
    /rules {
        /0000 { /glob "*" /type "deny" }
        /0001 { /glob "*.html" /type "allow" }
        /0002 { /glob "*.css" /type "allow" }
        /0003 { /glob "*.js" /type "allow" }
        /0004 { /glob "*.png" /type "allow" }
        /0005 { /glob "*.jpg" /type "allow" }
        /0006 { /glob "*.jpeg" /type "allow" }
        /0007 { /glob "*.gif" /type "allow" }
        /0008 { /glob "*.svg" /type "allow" }
        /0009 { /glob "*.webp" /type "allow" }
        /0010 { /glob "*.ico" /type "allow" }
        /0011 { /glob "*.woff" /type "allow" }
        /0012 { /glob "*.woff2" /type "allow" }
        /0013 { /glob "*.pdf" /type "allow" }
        /0014 { /glob "*.json" /type "allow" }
    }

    # Ignore query parameters for caching
    /ignoreUrlParams {
        /0001 { /glob "*" /type "deny" }
        /0002 { /glob "utm_*" /type "allow" }
        /0003 { /glob "gclid" /type "allow" }
        /0004 { /glob "fbclid" /type "allow" }
    }

    # Headers to cache
    /headers {
        "Cache-Control"
        "Content-Disposition"
        "Content-Type"
        "Expires"
        "Last-Modified"
        "X-Content-Type-Options"
    }

    # Invalidation rules
    /invalidate {
        /0000 { /glob "*" /type "deny" }
        /0001 { /glob "*.html" /type "allow" }
        /0002 { /glob "*.json" /type "allow" }
    }

    # Allowed clients for flush requests
    /allowedClients {
        /0000 { /glob "*" /type "deny" }
        /0001 { /glob "127.0.0.1" /type "allow" }
        /0002 { /glob "10.0.0.*" /type "allow" }
    }

    # Stat file level
    /statfileslevel "3"

    # Serve stale content on error
    /serveStaleOnError "1"

    # Grace period for stale content
    /gracePeriod "2"
}
```

### TTL-Based Caching (Cloud Service)

```apache
/cache {
    /enableTTL "1"

    /rules {
        /0000 { /glob "*" /type "deny" }
        /0001 { /glob "*.html" /type "allow" }
    }
}
```

## Statfileslevel Examples

### Single Site Setup

```
Content Structure:
/content/mysite/en/home
/content/mysite/en/products
/content/mysite/en/about

statfileslevel: 2
.stat files at:
- /content/.stat
- /content/mysite/.stat

Effect: Publishing any page invalidates entire site
```

### Multi-Site Setup

```
Content Structure:
/content/site-a/en/home
/content/site-a/de/home
/content/site-b/en/home
/content/site-b/fr/home

statfileslevel: 3
.stat files at:
- /content/.stat
- /content/site-a/.stat
- /content/site-a/en/.stat
- /content/site-a/de/.stat
- /content/site-b/.stat
- /content/site-b/en/.stat
- /content/site-b/fr/.stat

Effect: Publishing /content/site-a/en/home only invalidates site-a/en content
```

### Enterprise Multi-Region Setup

```
Content Structure:
/content/brand/region/country/language/...
/content/brand-a/americas/us/en/home
/content/brand-a/emea/de/de/home
/content/brand-b/apac/jp/ja/home

statfileslevel: 5
Effect: Publishing only invalidates specific brand/region/country/language
```

## Virtual Hosts Configuration

```apache
/virtualhosts {
    "www.example.com"
    "example.com"
    "www.example.de"
    "example.de"
}
```

### Multiple Farms for Different Domains

```apache
# /conf.dispatcher.d/available_farms/site-a.farm
/sitefarm {
    /virtualhosts {
        "www.site-a.com"
        "site-a.com"
    }
    /renders {
        /rend01 { /hostname "publish1.internal" /port "4503" }
    }
    /filter { $include "../filters/site-a_filters.any" }
    /cache { $include "../cache/site-a_cache.any" }
}

# /conf.dispatcher.d/available_farms/site-b.farm
/sitebfarm {
    /virtualhosts {
        "www.site-b.com"
        "site-b.com"
    }
    /renders {
        /rend01 { /hostname "publish2.internal" /port "4503" }
    }
    /filter { $include "../filters/site-b_filters.any" }
    /cache { $include "../cache/site-b_cache.any" }
}
```

## Load Balancing Configuration

### Basic Load Balancing

```apache
/renders {
    /rend01 {
        /hostname "publish1.internal"
        /port "4503"
        /timeout "60000"
    }
    /rend02 {
        /hostname "publish2.internal"
        /port "4503"
        /timeout "60000"
    }
    /rend03 {
        /hostname "publish3.internal"
        /port "4503"
        /timeout "60000"
    }
}
```

### With Health Check

```apache
/renders {
    /rend01 {
        /hostname "publish1.internal"
        /port "4503"
        /timeout "60000"
        /receiveTimeout "300000"
    }
}

/health_check {
    /url "/system/health"
    /healthy_stat 200
    /unhealthy_stat 500
}

/failover "1"
/unavailablePenalty "30"
/numberOfRetries "5"
/retryDelay "1000"
```

### Sticky Sessions

```apache
/stickyConnectionsFor "/content/mysite/secure"
/stickyConnections {
    /paths {
        "/content/mysite/account"
        "/content/mysite/checkout"
    }
}
```

## Client Headers Configuration

```apache
/clientheaders {
    # Standard headers
    "Accept"
    "Accept-Language"
    "Accept-Encoding"
    "Cache-Control"
    "Connection"
    "Content-Type"
    "Host"
    "If-Modified-Since"
    "If-None-Match"
    "Range"
    "User-Agent"

    # Forwarding headers
    "X-Forwarded-For"
    "X-Forwarded-Host"
    "X-Forwarded-Proto"
    "X-Forwarded-Port"

    # AEM-specific
    "CSRF-Token"
    "Cookie"

    # Custom headers
    "X-Custom-Header"
}
```

## Rewrite Rules (Apache mod_rewrite)

### Basic Rewrites

```apache
# Force HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]

# Remove trailing slash
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)/$ /$1 [R=301,L]

# Redirect www to non-www
RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
RewriteRule ^(.*)$ https://%1/$1 [R=301,L]
```

### Vanity URL Mapping

```apache
# Short URLs
RewriteRule ^/products$ /content/mysite/en/products.html [PT,L]
RewriteRule ^/about$ /content/mysite/en/about.html [PT,L]

# Category pages
RewriteRule ^/category/([a-z-]+)$ /content/mysite/en/products/category.html?cat=$1 [PT,L]

# Language redirect based on Accept-Language
RewriteCond %{HTTP:Accept-Language} ^de [NC]
RewriteRule ^/$ /content/mysite/de/home.html [PT,L]
```

### Content Path Mapping

```apache
# Map /en/* to /content/mysite/en/*
RewriteRule ^/en/(.*)$ /content/mysite/en/$1 [PT,L]
RewriteRule ^/de/(.*)$ /content/mysite/de/$1 [PT,L]

# Shorten DAM paths
RewriteRule ^/assets/(.*)$ /content/dam/mysite/$1 [PT,L]
```

## Security Headers (Apache)

```apache
<IfModule mod_headers.c>
    # Security headers
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"

    # Remove server information
    Header unset Server
    Header unset X-Powered-By

    # Cache control for static assets
    <FilesMatch "\.(css|js|ico|gif|jpe?g|png|svg|woff2?)$">
        Header set Cache-Control "public, max-age=31536000, immutable"
    </FilesMatch>

    # Cache control for HTML
    <FilesMatch "\.html$">
        Header set Cache-Control "public, max-age=300, must-revalidate"
    </FilesMatch>

    # CORS headers for API endpoints
    <LocationMatch "^/api/">
        Header set Access-Control-Allow-Origin "*"
        Header set Access-Control-Allow-Methods "GET, POST, OPTIONS"
        Header set Access-Control-Allow-Headers "Content-Type, Authorization"
    </LocationMatch>
</IfModule>
```

## Performance Tuning

### Apache MPM Settings

```apache
<IfModule mpm_event_module>
    StartServers             3
    MinSpareThreads         75
    MaxSpareThreads        250
    ThreadLimit             64
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild   0
</IfModule>
```

### Compression

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE text/javascript
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/json
    AddOutputFilterByType DEFLATE image/svg+xml
    AddOutputFilterByType DEFLATE application/xml
</IfModule>
```

### Keep-Alive

```apache
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
```

## Debugging and Logging

### Dispatcher Log Levels

```apache
/logLevel "1"
# 0 = Errors only
# 1 = Errors + Warnings (recommended for production)
# 2 = Errors + Warnings + Info
# 3 = Debug (development only, very verbose)
```

### Request Logging

```apache
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D" combined_time
CustomLog "/var/log/httpd/access.log" combined_time
```

### Cache Statistics

```apache
# Enable cache statistics
/statistics {
    /categories {
        /html { /glob "*.html" }
        /json { /glob "*.json" }
        /images { /glob "*.jpg" /glob "*.png" /glob "*.gif" }
        /clientlibs { /glob "/etc.clientlibs/*" }
        /others { /glob "*" }
    }
}
```
