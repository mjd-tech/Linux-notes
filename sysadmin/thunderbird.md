# Thunderbird email client

Thunderbird release version 78.5.1 supports non-encrypted subject lines

https://www.thunderbird.net/en-US/thunderbird/78.5.1/releasenotes/

It appears to not be a UI option but can be enabled via the config
editor

Thunderbird Menu -> Preferences -> General (scroll to bottom) ->
Editor Config (Accept risk) ->

Search for "mail.identity.default.protectSubject" -\> Double click on
value column (true)

Once this value is set to false newly composed emails will send without
encrypting subject lines.
