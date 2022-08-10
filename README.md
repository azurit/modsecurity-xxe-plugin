# OWASP ModSecurity Core Rule Set - XXE Plugin

## Description

This is a plugin that brings protection against XXE attacks to CRS.

The plugin uses `STREAM_INPUT_BODY` to access raw XML data, which allows it to
scan for XXE payloads inside `DOCTYPE` element (this element isn't accessible
while using ModSecurity XML body parser). To make this work, ModSecurity
configuration directive `SecStreamInBodyInspection` must be set to `On` but note
that this configuration may significantly impact file upload times (more
information [here](https://github.com/SpiderLabs/ModSecurity/issues/1366)).

For XXE protection supporting ModSecurity v3, see [XXE Lua plugin](https://github.com/coreruleset/xxe-lua-plugin).

## Prerequisities

 * ModSecurity version 2.9.x, version 3 is NOT supported (it does not support
 `STREAM_INPUT_BODY`)
 * ModSecurity `SecStreamInBodyInspection` configuration directive is `On`

## Installation

For full and up to date instructions for the different available plugin
installation methods, refer to [How to Install a Plugin](https://coreruleset.org/docs/concepts/plugins/#how-to-install-a-plugin)
in the official CRS documentation.

## Testing

After configuration, XXE protection should be tested, for example, using:  
`curl http://localhost -H "Content-Type: application/xml" --data '<!--?xml version="1.0" ?--><!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///etc/shadow"> ]><userInfo><firstName>John</firstName><lastName>&ent;</lastName></userInfo>'`

Using default CRS configuration, this request should end with status 403 with
the following message in the log:
`ModSecurity: Warning. Pattern match "<!ENTITY(?:\\\\s+%)?\\\\s+[^\\\\s]+\\\\s+(?:SYSTEM|PUBLIC)(?:\\\\s+['\\"][^'\\"]*['\\"])?\\\\s+['\\"]+(?i:data|expect|file|ftp|glob|gopher|http|https|jar|jdbc|ldap|ogg|phar|php|rar|ssh2|zip|zlib)://" at STREAM_INPUT_BODY. [file "/path/plugins/xxe-before.conf"] [line "38"] [id "9599100"] [msg "XML eXternal Entity: Local / Remote File Inclusion attempt"] [data "Matched Data: <!ENTITY ent SYSTEM \\x22file:// found within STREAM_INPUT_BODY: <!--?xml version=\\x221.0\\x22 ?--><!DOCTYPE replace [<!ENTITY ent SYSTEM \\x22file:///etc/shadow\\x22> ]><userInfo><firstName>John</firstName><lastName>&ent;</lastName></userInfo>"] [severity "CRITICAL"] [ver "xxe-plugin/1.0.0"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/250/228"] [hostname "localhost"] [uri "/"] [unique_id "YvJeU03XfxHWihOrx5xJIwAAAO4"]`

## License

Copyright (c) 2022 OWASP ModSecurity Core Rule Set project. All rights reserved.

The OWASP ModSecurity Core Rule Set and its official plugins are distributed
under Apache Software License (ASL) version 2. Please see the enclosed LICENSE
file for full details.
