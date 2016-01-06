+++
date = "2016-01-06T08:24:56Z"
title = "A Walk through Golang Package Interfaces"
description = "Golang interfaces are one of the most powerful feature of the language, their simplicity is charming; don't miss those ones available in the language package"
tags = ["golang", "programming", "programming-languages"]
categories = ["Software Development"]
social_image = "https://s-media-cache-ak0.pinimg.com/736x/c1/53/6e/c1536ec3a9bc88e403f0f7e2d6d72557.jpg"
+++

Golang interface is one of things that people love due its simplicity and how they make possible to create small things which can be composed to create more complex ones.

It's well know that defining interfaces in the implementations make them more flexible and reusable; defining interfaces with just one or a couple of functions signatures is one of the good practices of writing Golang applications, because they make the implementations more modular due they can be composed to get new ones for more specific needs.

Having this big set of small interface give the opportunity to have a bigger set of generic functions because they only constrain the parameters to implement one of those; so they can be use for any user defined type which implements at least that specific interface.

Everything above is well know and there are lots of posts about this matter, hence my goal in this post isn't to add a new one more or extend this topic even more; there already good literature about it that you can find easily.

__My goal is to provide the list of interfaces which I've found, and hopefully I haven't missed any, in Golang 1.5 standard package and highlight few of them__ which I think that we have to bear in mind when we are developing common applications or services.

## The ones useful for the most common applications

### sql package

The sql package has two without including the ones defined in the driver subpackage which I've skipped due that in common applications development is unusual having to implement a new driver.

* {{<ext-link "Scanner" "https://golang.org/pkg/database/sql/#Scanner">}}
* {{<ext-link "Result" "https://golang.org/pkg/database/sql/#Result">}}

These ones gathered my attention due that it isn't that rare to store types in a database which aren't basic types defined in the used database system; while we can do the mapping each time that we read those values from the database, the Scanner interface is very convenient to be implemented and get the benefits to get the mapping automatically when the values are read from the database, by the Scan function.

One example of this case is {{<ext-link "go.uuid" "https://godoc.org/github.com/satori/go.uuid#UUID.Scan">}}

### encoding Package

The encoding/decoding data is another common operation to perform in common application development. Golang encoding package defines two pairs, marshare/unmarsheler, of interfaces, one for binary and another for text:

* {{<ext-link "BinaryMarshaler" "https://golang.org/pkg/encoding/#BinaryMarshaler">}}
* {{<ext-link "BinaryUnmarshaler" "https://golang.org/pkg/encoding/#BinaryUnmarshaler">}}
* {{<ext-link "TextMarshaler" "https://golang.org/pkg/encoding/#TextMarshaler">}}
* {{<ext-link "TextUnmarshaler" "https://golang.org/pkg/encoding/#TextUnmarshaler">}}

It also defines the same pair for specific encodings which usage is very spread

* __gob__
  * {{<ext-link "GobDecoder" "https://golang.org/pkg/encoding/gob/#GobDecoder">}}
  * {{<ext-link "GobEncoder" "https://golang.org/pkg/encoding/gob/#GobEncoder">}}
* __json__
  * {{<ext-link "Marshaler" "https://golang.org/pkg/encoding/json/#Marshaler">}}
  * {{<ext-link "Unmarshaler" "https://golang.org/pkg/encoding/json/#Unmarshaler">}}
* __xml__: which has one pair to allow to encode/decode attributes
  * {{<ext-link "Marshaler" "https://golang.org/pkg/encoding/xml/#Marshaler">}}
  * {{<ext-link "Unmarshaler" "https://golang.org/pkg/encoding/xml/#Unmarshaler">}}
  * {{<ext-link "MarshalerAttr" "https://golang.org/pkg/encoding/xml/#MarshalerAttr">}}
  * {{<ext-link "UnmarshalerAttr" "https://golang.org/pkg/encoding/xml/#UnmarshalerAttr">}}

And one more for binary encoding
* {{<ext-link "ByteOrder" "https://golang.org/pkg/encoding/binary/#ByteOrder">}}


### flag package

Another common stuff is to pass argument to our application when it starts; _flag package_ contains two interface which allow to define types to map values passed through command line arguments.

* {{<ext-link "Getter" "https://golang.org/pkg/flag/#Getter">}}
* {{<ext-link "Value" "https://golang.org/pkg/flag/#Value">}}

### fmt package

The fmt package defines a set of interfaces to format I/O, which is another functionality which can be needed in common applications development.

* {{<ext-link "Formatter" "https://golang.org/pkg/fmt/#Formatter">}}
* {{<ext-link "GoStringer" "https://golang.org/pkg/fmt/#GoStringer">}}
* {{<ext-link "ScanState" "https://golang.org/pkg/fmt/#ScanState">}}
* {{<ext-link "Scanner" "https://golang.org/pkg/fmt/#Scanner">}}
* {{<ext-link "State" "https://golang.org/pkg/fmt/#State">}}
* {{<ext-link "Stringer" "https://golang.org/pkg/fmt/#Stringer">}}

### sort package

Some applications also requires to sort element in slices, so the types which implement sort interface can use after the routines of sort package to be sorted.

* {{<ext-link "Interface" "https://golang.org/pkg/sort/#Interface">}}


## The rest

__Compress package__

* __flate__
  * {{<ext-link "falate/Reader" "https://golang.org/pkg/compress/flate/#Reader">}}
  * {{<ext-link "falate/Resetter" "https://golang.org/pkg/compress/flate/#Resetter">}}
* __zlib__
  * {{<ext-link "Resetter" "https://golang.org/pkg/compress/zlib/#Resetter">}}

__Heap package__

* {{<ext-link "Interface" "https://golang.org/pkg/container/heap/#Interface">}}

__Crypto package__

* {{<ext-link "Decrypter" "https://golang.org/pkg/crypto/#Decrypter">}}
* {{<ext-link "Signer" "https://golang.org/pkg/crypto/#Signer">}}
* {{<ext-link "SignerOpts" "https://golang.org/pkg/crypto/#SignerOpts">}}
* __cipher__
  * {{<ext-link "AEAD" "https://golang.org/pkg/crypto/cipher/#AEAD">}}
  * {{<ext-link "Block" "https://golang.org/pkg/crypto/cipher/#Block">}}
  * {{<ext-link "BlockMode" "https://golang.org/pkg/crypto/cipher/#BlockMode">}}
  * {{<ext-link "Stream" "https://golang.org/pkg/crypto/cipher/#Stream">}}
* __elliptic__
  * {{<ext-link Curve "https://golang.org/pkg/crypto/elliptic/#Curve">}}
  * {{<ext-link "ClientSessionCache" "https://golang.org/pkg/crypto/tls/#ClientSessionCache">}}

__sql/driver package__

* {{<ext-link "ColumnConverter" "https://golang.org/pkg/database/sql/driver/#ColumnConverter">}}
* {{<ext-link "Conn" "https://golang.org/pkg/database/sql/driver/#Conn">}}
* {{<ext-link "Driver" "https://golang.org/pkg/database/sql/driver/#Driver">}}
* {{<ext-link "Execer" "https://golang.org/pkg/database/sql/driver/#Execer">}}
* {{<ext-link "Queryer" "https://golang.org/pkg/database/sql/driver/#Queryer">}}
* {{<ext-link "Result" "https://golang.org/pkg/database/sql/driver/#Result">}}
* {{<ext-link "Stmt" "https://golang.org/pkg/database/sql/driver/#Stmt">}}
* {{<ext-link "Tx" "https://golang.org/pkg/database/sql/driver/#Tx">}}
* {{<ext-link "Value" "https://golang.org/pkg/database/sql/driver/#Value">}}
* {{<ext-link "ValueConverter" "https://golang.org/pkg/database/sql/driver/#ValueConverter">}}
* {{<ext-link "Valuer" "https://golang.org/pkg/database/sql/driver/#Valuer">}}

__debug package__

* __dwarf__
  * {{<ext-link "Type" "https://golang.org/pkg/debug/dwarf/#Type">}}
* __macho__
  * {{<ext-link "Load" "https://golang.org/pkg/debug/macho/#Load">}}

__expvar package__

* {{<ext-link "Var" "https://golang.org/pkg/expvar/#Var">}}

__ast package__

* {{<ext-link "Decl" "https://golang.org/pkg/go/ast/#Decl">}}
* {{<ext-link "Expr" "https://golang.org/pkg/go/ast/#Expr">}}
* {{<ext-link "Node" "https://golang.org/pkg/go/ast/#Node">}}
* {{<ext-link "Spec" "https://golang.org/pkg/go/ast/#Spec">}}
* {{<ext-link "Stmt" "https://golang.org/pkg/go/ast/#Stmt">}}
* {{<ext-link "Visitor" "https://golang.org/pkg/go/ast/#Visitor">}}

__constant package__

* {{<ext-link "Value" "https://golang.org/pkg/go/constant/#Value">}}

__types package__

* {{<ext-link "Importer" "https://golang.org/pkg/go/types/#Importer">}}
* {{<ext-link "Object" "https://golang.org/pkg/go/types/#Object">}}
* {{<ext-link "Sizes" "https://golang.org/pkg/go/types/#Sizes">}}
* {{<ext-link "Type" "https://golang.org/pkg/go/types/#Type">}}

__hash package__

* {{<ext-link "Hash" "https://golang.org/pkg/hash/#Hash">}}
* {{<ext-link "Hash" "https://golang.org/pkg/hash/#Hash">}}
* {{<ext-link "Hash64" "https://golang.org/pkg/hash/#Hash64">}}

__image package__

* {{<ext-link "Image" "https://golang.org/pkg/image/#Image">}}
* {{<ext-link "PalettedImage" "https://golang.org/pkg/image/#PalettedImage">}}
* __color__
  * {{<ext-link "Color" "https://golang.org/pkg/image/color/#Color">}}
  * {{<ext-link "Model" "https://golang.org/pkg/image/color/#Model">}}
* __draw__
  * {{<ext-link "Drawer" "https://golang.org/pkg/image/draw/#Drawer">}}
  * {{<ext-link "Image" "https://golang.org/pkg/image/draw/#Image">}}
  * {{<ext-link "Quantizer" "https://golang.org/pkg/image/draw/#Quantizer">}}
* __jpeg__
  * {{<ext-link "Reader" "https://golang.org/pkg/image/jpeg/#Reader">}}

__io package__

* {{<ext-link "ByteReader" "https://golang.org/pkg/io/#ByteReader">}}
* {{<ext-link "ByteScanner" "https://golang.org/pkg/io/#ByteScanner">}}
* {{<ext-link "ByteWriter" "https://golang.org/pkg/io/#ByteWriter">}}
* {{<ext-link "Closer" "https://golang.org/pkg/io/#Closer">}}
* {{<ext-link "ReadCloser" "https://golang.org/pkg/io/#ReadCloser">}}
* {{<ext-link "ReadSeeker" "https://golang.org/pkg/io/#ReadSeeker">}}
* {{<ext-link "ReadWriteCloser" "https://golang.org/pkg/io/#ReadWriteCloser">}}
* {{<ext-link "ReadWriteSeeker" "https://golang.org/pkg/io/#ReadWriteSeeker">}}
* {{<ext-link "ReadWriter" "https://golang.org/pkg/io/#ReadWriter">}}
* {{<ext-link "Reader" "https://golang.org/pkg/io/#Reader">}}
* {{<ext-link "ReaderAt" "https://golang.org/pkg/io/#ReaderAt">}}
* {{<ext-link "ReaderFrom" "https://golang.org/pkg/io/#ReaderFrom">}}
* {{<ext-link "RuneReader" "https://golang.org/pkg/io/#RuneReader">}}
* {{<ext-link "RuneScanner" "https://golang.org/pkg/io/#RuneScanner">}}
* {{<ext-link "Seeker" "https://golang.org/pkg/io/#Seeker">}}
* {{<ext-link "WriteCloser" "https://golang.org/pkg/io/#WriteCloser">}}
* {{<ext-link "WriteSeeker" "https://golang.org/pkg/io/#WriteSeeker">}}
* {{<ext-link "Writer" "https://golang.org/pkg/io/#Writer">}}
* {{<ext-link "WriterAt" "https://golang.org/pkg/io/#WriterAt">}}
* {{<ext-link "WriterTo" "https://golang.org/pkg/io/#WriterTo">}}

__math/rand package__

* {{<ext-link "Source" "https://golang.org/pkg/math/rand/#Source">}}

__mine/multipart package__

* {{<ext-link "File" "https://golang.org/pkg/mime/multipart/#File">}}

__net package__

* {{<ext-link "Addr" "https://golang.org/pkg/net/#Addr">}}
* {{<ext-link "Conn" "https://golang.org/pkg/net/#Conn">}}
* {{<ext-link "Error" "https://golang.org/pkg/net/#Error">}}
* {{<ext-link "Listener" "https://golang.org/pkg/net/#Listener">}}
* {{<ext-link "PacketConn" "https://golang.org/pkg/net/#PacketConn">}}
* __http__
  * {{<ext-link "CloseNotifier" "https://golang.org/pkg/net/http/#CloseNotifier">}}
  * {{<ext-link "CookieJar" "https://golang.org/pkg/net/http/#CookieJar">}}
  * {{<ext-link "File" "https://golang.org/pkg/net/http/#File">}}
  * {{<ext-link "FileSystem" "https://golang.org/pkg/net/http/#FileSystem">}}
  * {{<ext-link "Flusher" "https://golang.org/pkg/net/http/#Flusher">}}
  * {{<ext-link "Handler" "https://golang.org/pkg/net/http/#Handler">}}
  * {{<ext-link "Hijacker" "https://golang.org/pkg/net/http/#Hijacker">}}
  * {{<ext-link "ResponseWriter" "https://golang.org/pkg/net/http/#ResponseWriter">}}
  * {{<ext-link "RoundTripper" "https://golang.org/pkg/net/http/#RoundTripper">}}
  * __cookiejar__
      * {{<ext-link "PublicSuffixList" "https://golang.org/pkg/net/http/cookiejar/#PublicSuffixList">}}
  * __rpc__
      * {{<ext-link "ClientCodec" "https://golang.org/pkg/net/rpc/#ClientCodec">}}
      * {{<ext-link "ServerCodec" "https://golang.org/pkg/net/rpc/#ServerCodec">}}
  * __smtp__
      * {{<ext-link "Auth" "https://golang.org/pkg/net/smtp/#Auth">}}
  * __os__
      * {{<ext-link "FileInfo" "https://golang.org/pkg/os/#FileInfo">}}
      * {{<ext-link "Signal" "https://golang.org/pkg/os/#Signal">}}

__reflect package__

* {{<ext-link "Type" "https://golang.org/pkg/reflect/#Type">}}

__runtime package__

* {{<ext-link "Error" "https://golang.org/pkg/runtime/#Error">}}

__sync package__

* {{<ext-link "Locker" "https://golang.org/pkg/sync/#Locker">}}

__syscall package__

* {{<ext-link "Sockaddr" "https://golang.org/pkg/syscall/#Sockaddr">}}

__tesing package__

* {{<ext-link "TB" "https://golang.org/pkg/testing/#TB">}}
* __quick__
  * {{<ext-link "Generator" "https://golang.org/pkg/testing/quick/#Generator">}}

__text/template/parse__

* {{<ext-link "Node" "https://golang.org/pkg/text/template/parse/#Node">}}


## Conclusion

Golang interfaces are the most powerful and simple feature that the language expose to create modular and reusable implementations.

Golang standard library contains a bunch of already defined interfaces which can be implemented by our applications or services without having to think in new ones for the same purpose besides to ease and speed up to any Golang developer to understand our implementations.

Collecting all of them in in this post allowed me to discover some of them which I haven't seen before even though I've been developing in Golang for one year and a half. Some of them I had never seen because I haven't had to use them, others I had seen when I was reading a specific package documentation and other I saw at some point but I didn't assimilate at the first glance, so this post has been a good exercise.

If I've missed any and you know which ones, please let me know with an innocent comment.

Thanks for reading it.
