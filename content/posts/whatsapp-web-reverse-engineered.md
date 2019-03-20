---
title: 'WhatsApp Web reverse engineered'
date: Mon, 09 Apr 2018 19:53:30 +0000
draft: false
tags: [Tech]
---

Introduction
------------

This project intends to provide a complete description and re-implementation of the WhatsApp Web API, which will eventually lead to a custom client. WhatsApp Web internally works using WebSockets; this project does as well.

[](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#trying-it-out)Trying it out
---------------------------------------------------------------------------------------------------

Before you can run the application, make sure that you have the following software installed:

*   Node.js (at least version 8, as the `async` `await` syntax is used)
*   the CSS preprocessor [Sass](http://sass-lang.com/) (which you previously need Ruby for)
*   Python 2.7 with the following `pip` packages installed:
    *   `websocket-client` and `git+https://github.com/dpallot/simple-websocket-server.git` for acting as WebSocket server and client
    *   `curve25519-donna` and `pycrypto` for the encryption stuff
    *   `pyqrcode` for QR code generation

Before starting the application for the first time, run `npm install` to install all dependencies. Lastly, to finally launch it, just run `npm start`. Using fancy `concurrently` and `nodemon` magic, all three local components will be started after each other and when you edit a file, the changed module will automatically restart to apply the changes.

[](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#application-architecture)Application architecture
-------------------------------------------------------------------------------------------------------------------------

The project is organized in the following way. Note the used ports and make sure that they are not in use elsewhere before starting the application. ![](http://blog.bsiddhartha.com/wp-content/uploads/2018/04/app-architecture1000.png)

Login and encryption details
----------------------------

WhatsApp Web encrypts the data using several different algorithms. These include [AES 256 ECB](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), [Curve25519](https://en.wikipedia.org/wiki/Curve25519) as Diffie-Hellman key agreement scheme, [HKDF](https://en.wikipedia.org/wiki/HKDF) for generating the extended shared secret and [HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code) with SHA256. Starting the WhatsApp Web session happens by just connecting to one of its websocket servers at `wss://w[1-8].web.whatsapp.com/ws` (`wss://` means that the websocket connection is secure; `w[1-8]` means that any number between 1 and 8 can follow the `w`). Also make sure that, when establishing the connection, the HTTP header `Origin: https://web.whatsapp.com` is set, otherwise the connection will be rejected.

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#messages)Messages

When you send messages to a WhatsApp Web websocket, they need to be in a specific format. It is quite simple and looks like `messageTag,JSON`, e.g. `1515590796,["data",123]`. Note that apparently the message tag can be anything. This application mostly uses the current timestamp as tag, just to be a bit unique. WhatsApp itself often uses message tags like `s1`, `1234.--0` or something like that. Obviously the message tag may not contain a comma. Additionally, JSON _objects_are possible as well as payload.

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#logging-in)Logging in

To log in at an open websocket, follow these steps:

1.  Generate your own `clientId`, which needs to be 16 base64-encoded bytes (i.e. 25 characters). This application just uses 16 random bytes, i.e. `base64.b64encode(os.urandom(16))` in Python.
2.  Decide for a tag for your message, which is more or less arbitrary (see above). This application uses the current timestamp (in seconds) for that. Remember this tag for later.
3.  The message you send to the websocket looks like this: `messageTag,["admin","init",[0,2,7314],["Long browser description","ShortBrowserDesc"],"clientId",true]`.
    *   Obviously, you need to replace `messageTag` and `clientId` by the values you chose before
    *   The `[0,2,7314]` part specifies the current WhatsApp Web version. The last value changes frequently. It should be quite backwards-compatible though.
    *   `"Long browser description"` is an arbitrary string that will be shown in the WhatsApp app in the list of registered WhatsApp Web clients after you scan the QR code.
    *   `"ShortBrowserDesc"` has not been observed anywhere yet but is arbitrary as well.
4.  After a few moments, your websocket will receive a message in the specified format with the message tag _you chose in step 2_. The JSON object of this message has the following attributes:
    *   `status`: should be 200
    *   `ref`: in the application, this is treated as the server ID; important for the QR generation, see below
    *   `ttl`: is 20000, maybe the time after the QR code becomes invalid
    *   `update`: a boolean flag
    *   `curr`: the current WhatsApp Web version, e.g. `0.2.7314`
    *   `time`: the timestamp the server responded at, as floating-point milliseconds, e.g. `1515592039037.0`

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#qr-code-generation)QR code generation

5.  Generate your own private key with Curve25519, e.g. `curve25519.Private()`.
6.  Get the public key from your private key, e.g. `privateKey.get_public()`.
7.  Obtain the string later encoded by the QR code by concatenating the following values with a comma:
    *   the server ID, i.e. the `ref` attribute from step 4
    *   the base64-encoded version of your public key, i.e. `base64.b64encode(publicKey.serialize())`
    *   your client ID
8.  Turn this string into an image (e.g. using `pyqrcode`) and scan it using the WhatsApp app.

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#after-scanning-the-qr-code)After scanning the QR code

9.  Immediately after you scan the QR code, the websocket receives several important JSON messages that build up the encryption details. These use the specified message format and have a JSON _array_ as payload. Their message tag has no special meaning. The first entry of the JSON array has one of the following values:
    *   `Conn`: array contains JSON object as second element with connection information containing the following attributes and many more:
        *   `battery`: the current battery percentage of your phone
        *   `browserToken` (could be important, but not used by the application yet)
        *   `clientToken` (could be important, but not used by the application yet)
        *   `phone`: an object with detailed information about your phone, e.g. `device_manufacturer`, `device_model`, `os_build_number`, `os_version`
        *   `platform`: your phone OS, e.g. `android`
        *   `pushname`: the name of yours you provided WhatsApp
        *   `secret` (remember this!)
        *   `serverToken` (could be important, but not used by the application yet)
        *   `wid`: your phone number in the chat identification format (see below)
    *   `Stream`: array has four elements in total, so the entire payload is like `["Stream","update",false,"0.2.7314"]`
    *   `Props`: array contains JSON object as second element with several properties like `imageMaxKBytes` (1024), `maxParticipants` (257), `videoMaxEdge` (960) and others

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#key-generation)Key generation

10.  You are now ready for generating the final encryption keys. Start by decoding the `secret` from `Conn` as base64 and storing it as `secret`. This decoded secret will be 144 bytes long.
11.  Take the _first 32 bytes_ of the decoded secret and use it as a public key. Together with your private key, generate a shared key out of it and call it `sharedSecret`. The application does it using `privateKey.get_shared_key(curve25519.Public(secret[:32]), lambda a:a)`.
12.  Use a key containing 32 null bytes to encode the shared secret using HMAC SHA256. Take this value and extend it to 80 bytes using HKDF. Call this value `sharedSecretExpanded`. This is done with `HKDF(HmacSha256("\0"*32, sharedSecret), 80)`.
13.  This step is optional, it validates the data provided by the server. The method is called _HMAC validation_. Do it by first calculating `HmacSha256(sharedSecretExpanded[32:64], secret[:32] + secret[64:])`. Compare this value to `secret[32:64]`. If they are not equal, abort the login.
14.  You now have the encrypted keys: store `sharedSecretExpanded[64:] + secret[64:]` as `keysEncrypted`.
15.  The encrypted keys now need to be decrypted using AES with `sharedSecretExpanded[:32]` as key, i.e. store `AESDecrypt(sharedSecretExpanded[:32], keysEncrypted)` as `keysDecrypted`.
16.  The `keysDecrypted` variable is 64 bytes long and contains two keys, each 32 bytes long. The `encKey` is used for decrypting binary messages sent to you by the WhatsApp Web server or encrypting binary messages you send to the server. The `macKey` is needed to validate the messages sent to you:
    *   `encKey`: `keysDecrypted[:32]`
    *   `macKey`: `keysDecrypted[32:64]`

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#validating-and-decrypting-messages)Validating and decrypting messages

Now that you have the two keys, validating and decrypting messages the server sent to you is quite easy. Note that this is only needed for _binary_ messages, all JSON you receive stays plain. The binary messages always have 32 bytes at the beginning that specify the HMAC checksum.

1.  Validate the message by hashing the actual message content with the `macKey` (here `messageContent` is the _entire_binary message): `HmacSha256(macKey, messageContent[32:])`. If this value is not equal to `messageContent[:32]`, the message sent to you by the server is invalid and should be discarded.
2.  Decrypt the message content using AES and the `encKey`: `AESDecrypt(encKey, messageContent[32:])`.

The data you get in the final step has a binary format which is described in the following. Even though it's binary, you can still see several strings in it, especially the content of messages you sent is quite obvious there.

[](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#binary-message-format)Binary message format
-------------------------------------------------------------------------------------------------------------------

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#binary-decoding)Binary decoding

The Python script `backend/decoder.py` implements the `MessageParser` class. It is able to create a JSON structure out of binary data in which the data is still organized in a rather messy way. The section about Node Handling below will discuss how the nodes are reorganized afterwards. `MessageParser` initially just needs some data and then processes it byte by byte, i.e. as a stream. It has a couple of constants and a lot of methods which all build on each other.

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#constants)Constants

*   _Tags_ with their respective integer values
    *   _LIST_EMPTY_: 0
    *   _STREAM_8_: 2
    *   _DICTIONARY_0_: 236
    *   _DICTIONARY_1_: 237
    *   _DICTIONARY_2_: 238
    *   _DICTIONARY_3_: 239
    *   _LIST_8_: 248
    *   _LIST_16_: 249
    *   _JID_PAIR_: 250
    *   _HEX_8_: 251
    *   _BINARY_8_: 252
    *   _BINARY_20_: 253
    *   _BINARY_32_: 254
    *   _NIBBLE_8_: 255
*   _Tokens_ are a long list of 151 strings in which the indices matter:
    *   `[None,None,None,"200","400","404","500","501","502","action","add", "after","archive","author","available","battery","before","body", "broadcast","chat","clear","code","composing","contacts","count", "create","debug","delete","demote","duplicate","encoding","error", "false","filehash","from","g.us","group","groups_v2","height","id", "image","in","index","invis","item","jid","kind","last","leave", "live","log","media","message","mimetype","missing","modify","name", "notification","notify","out","owner","participant","paused", "picture","played","presence","preview","promote","query","raw", "read","receipt","received","recipient","recording","relay", "remove","response","resume","retry","s.whatsapp.net","seconds", "set","size","status","subject","subscribe","t","text","to","true", "type","unarchive","unavailable","url","user","value","web","width", "mute","read_only","admin","creator","short","update","powersave", "checksum","epoch","block","previous","409","replaced","reason", "spam","modify_tag","message_info","delivery","emoji","title", "description","canonical-url","matched-text","star","unstar", "media_key","filename","identity","unread","page","page_count", "search","media_message","security","call_log","profile","ciphertext", "invite","gif","vcard","frequent","privacy","blacklist","whitelist", "verify","location","document","elapsed","revoke_invite","expiration", "unsubscribe","disable"]`

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#number-reformatting)Number reformatting

*   _Unpacking nibbles_: Returns the ASCII representation for numbers between 0 and 9. Returns `-` for 10, `.` for 11 and `\0` for 15.
*   _Unpacking hex values_: Returns the ASCII representation for numbers between 0 and 9 or letters between A and F (i.e. uppercase) for numbers between 10 and 15.
*   _Unpacking bytes_: Expects a tag as an additional parameter, namely _NIBBLE_8_ or _HEX_8_. Unpacks a nibble or hex value accordingly.

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#number-formats)Number formats

*   _Byte_: A plain ol' byte.
*   _Integer with N bytes_: Reads N bytes and builds a number out of them. Can be little or big endian; if not specified otherwise, big endian is used. Note that no negative values are possible.
*   _Int16_: An integer with two bytes, read using _Integer with N bytes_.
*   _Int20_: Consumes three bytes and constructs an integer using the last four bits of the first byte and the entire second and third byte. Is therefore always big endian.
*   _Int32_: An integer with four bytes, read using _Integer with N bytes_.
*   _Int64_: An integer with eight bytes, read using _Integer with N bytes_.
*   _Packed8_: Expects a tag as an additional parameter, namely _NIBBLE_8_ or _HEX_8_. Returns a string.
    *   First reads a byte `n` and does the following `n&127` many times: Reads a byte `l` and for each nibble, adds the result of its _unpacked version_ to the return value (using _unpacking bytes_ with the given tag). Most significant nibble first.
    *   If the most significant bit of `n` was set, removes the last character of the return value.

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#variable-length-integers)Variable length integers

In contrast to the previous number formats, reading a _variable length integer_ (VLI) does _not_ change the current data pointer. First, the length `l` of the VLI is read by reading bytes until a byte with the most significant bit set is encountered, but at most 10 bytes. TODO _Ranged variable length integers_ expect a minimum and a maximum value. If the read _variable length integer_ is less then the minimum or greater than or equal to the maximum, throw an error.

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#helper-methods)Helper methods

*   _Read bytes_: Reads and returns the specified number of bytes.
*   _Check for list tag_: Expects a tag as parameter and returns true if the tag is `LIST_EMPTY`, `LIST_8` or `LIST_16` (i.e. 0, 248 or 249).
*   _Read list size_: Expects a list tag as parameter. Returns 0 for `LIST_EMPTY`, returns a read byte for `LIST_8` or a read _Int16_ for `LIST_16`.
*   _Read a string from characters_: Expects the string length as parameter, reads this many bytes and returns them as a string.
*   _Get a token_: Expects an index to the array of _Tokens_, and returns the respective string.
*   _Get a double token_: Expects two integers `a` and `b` and gets the token at index `a*256+b`.

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#strings)Strings

Reading a string needs a _tag_ as parameter. Depending on this tag, different data is read.

*   If the tag is between 3 and 235, the _token_ (i.e. a string) of this tag is got. If the token is `"s.whatsapp.net"`, `"c.us"`is returned instead, otherwise the token is returned as is.
*   If the tag is between _DICTIONARY_0_ and _DICTIONARY_3_, a _double token_ is returned, with `tag-DICTIONARY_0` as first and a read byte as second parameter.
*   _LIST_EMPTY_: Nothing is returned (e.g. `None`).
*   _BINARY_8_: A byte is read which is then used to _read a string from characters_ with this length.
*   _BINARY_20_: An _Int20_ is read which is then used to _read a string from characters_ with this length.
*   _BINARY_32_: An _Int32_ is read which is then used to _read a string from characters_ with this length.
*   _JID_PAIR_
    *   First, a byte is read which is then used to _read a string_ `i` with this tag.
    *   Second, another byte is read which is then used to _read a string_ `j` with this tag.
    *   Finally, `i` and `j` are joined together with an `@` sign and the result is returned.
*   _NIBBLE_8_ or _HEX_8_: A _Packed8_ with this tag is returned.

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#attribute-lists)Attribute lists

Reading an attribute list needs the number of attributes to read as parameter. An attribute list is always a JSON object. For each attribute read, the following steps are executed for getting key-value pairs (exactly in this order!):

*   _Key_: A byte is read which is then used to _read a string_ with this tag.
*   _Value_: A byte is read which is then used to _read a string_ with this tag.

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#nodes)Nodes

A node always consists of a JSON array with exactly three entries: description, attributes and content. The following steps are needed to read a node:

1.  A _list size_ `a` is read by using a read byte as the tag. The list size 0 is invalid.
2.  The description tag is read as a byte. The value 2 is invalid for this tag. The description string `descr` is then obtained by _reading a string_ with this tag.
3.  The attributes object `attrs` is read by _reading an attributes object_ with length `(a-2 + a%2) >> 1`.
4.  If `a` was odd, this node does not have any content, i.e. `[descr, attrs, None]` is returned.
5.  For getting the node's content, first a byte, i.e. a tag is read. Depending on this tag, different types of content emerge:
    *   If the tag is a _list tag_, a _list is read_ using this tag (see below for lists).
    *   _BINARY_8_: A byte is read which is then used as length for _reading bytes_.
    *   _BINARY_20_: An _Int20_ is read which is then used as length for _reading bytes_.
    *   _BINARY_32_: An _Int32_ is read which is then used as length for _reading bytes_.
    *   If the tag is something else, a _string is read_ using this tag.
6.  Eventually, `[descr, attrs, content]` is returned.

#### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#lists)Lists

Reading a list requires a _list tag_ (i.e. _LIST_EMPTY_, _LIST_8_ or _LIST_16_). The length of the list is then obtained by _reading a list size_ using this tag. For each list entry, a _node is read_.

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#node-handling)Node Handling

TODO

[](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#whatsapp-web-api)WhatsApp Web API
---------------------------------------------------------------------------------------------------------

WhatsApp Web itself has an interesting API as well. You can even try it out directly in your browser. Just log in at the normal [https://web.whatsapp.com/](https://web.whatsapp.com/), then open the browser development console. Now enter something like the following (see below for details on the chat identification):

*   `window.Store.Wap.profilePicFind("49123456789@c.us").then(res => console.log(res));`
*   `window.Store.Wap.lastseenFind("49123456789@c.us").then(res => console.log(res));`
*   `window.Store.Wap.statusFind("49123456789@c.us").then(res => console.log(res));`

Using the amazing Chrome developer console, you can see that `window.Store.Wap` contains a lot of other very interesting functions. Many of them return JavaScript promises. When you click on the _Network_ tab and then on _WS_ (maybe you need to reload the site first), you can look at all the communication between WhatsApp Web and its servers.

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#chat-identification)Chat identification

The WhatsApp Web API uses the following formats to identify chats with individual users and groups of multiple users.

*   **Chats**: `[country code][number]@c.us`, e.g. **`49123456789@c.us`** when you are from Germany and your phone number is `0123 456789`.
*   **Groups**: `[phone number of group creator]-[timestamp of group creation]@g.us`, e.g. **`49123456789-1509911919@g.us`** for the group that `49123456789@c.us` created on November 5 2017.

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#websocket-messages)WebSocket messages

There are two types of WebSocket messages that are exchanged between server and client. On the one hand, plain JSON that is rather unambiguous (especially for the API calls above), on the other hand encrypted binary messages. Unfortunately, these binary ones cannot be looked at using the Chrome developer tools. Additionally, the Python backend, that of course also receives these messages, needs to decrypt them, as they contain encrypted data. The section about encryption details discusses how it can be decrypted.

[](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#tasks)Tasks
-----------------------------------------------------------------------------------

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#backend)Backend

*    Allow sending messages as well. Of course JSON is easy, but _writing_ the binary message format needs to start being examined.

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#web-frontend)Web frontend

*    Allow reusing the session after successful login. Probably normal cookies are best for this.
*    An UI that is not that technical, but rather starts to emulate the actual WhatsApp Web UI.

### [](https://github.com/sigalor/whatsapp-web-reveng/blob/master/README.md#documentation)Documentation

*    The _Node Handling_ section. Could become very long.
*    The _Disclaimer_ section. Should contain stuff like "no warranty" and "don't do bad stuff".
*    Outsource the different documentation parts into their own files, maybe into the `gh-pages` branch.