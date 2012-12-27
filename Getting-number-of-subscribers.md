There's an easy way to display number of clients watching the stream. You need to

Set up statistics page at location `/stat`

    location /stat {
        rtmp_stat all;
        allow 127.0.0.1;
    }

Create a simple xsl stylesheet `nclients.xsl` extracting number of stream subscribers

    <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

    <xsl:output method="html"/>

    <xsl:param name="app"/>
    <xsl:param name="name"/>

    <xsl:template match="/">
        <xsl:value-of select="count(//application[name=$app]/live/stream[name=$name]/client[not(publishing) and flashver])"/>
    </xsl:template>

    </xsl:stylesheet>

Set up a location returning number of suscribers

    location /nclients {
        proxy_pass http://127.0.0.1/stat;
        xslt_stylesheet /www/nclients.xsl app='$arg_app' name='$arg_name';
        add_header Refresh "3; $request_uri";
    }

Use HTTP request `http://myserver.com/nclients?app=myapp&name=mystream` to get the number of stream subscribers. This number will be automatically refreshed every 3 seconds when opened in browser or iframe.