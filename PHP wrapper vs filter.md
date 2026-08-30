PHP Wrapper vs Filter
Wrappers

A wrapper (stream wrapper) is a protocol handler that tells PHP how to access a resource — it defines the transport layer.

Syntax: wrapper://resource

Built-in wrappers:

Wrapper	Purpose
file://	Local filesystem (default)
http://	HTTP URLs
ftp://	FTP servers
php://	PHP I/O streams
data://	Inline data
zip://	ZIP archives

Example:

php
// These all use wrappers
file_get_contents('http://example.com');       // http wrapper
file_get_contents('php://input');              // php wrapper (raw POST body)
file_get_contents('zip://archive.zip#file.txt'); // zip wrapper

When to use wrappers:

Reading from different sources (remote URLs, raw input, memory)
Abstracting where data comes from
Custom protocols (you can register your own with stream_wrapper_register())
Filters

A filter processes/transforms data as it flows through a stream — it sits on top of a wrapper and modifies the content in transit.

Syntax: php://filter/filter-name/resource=...

Built-in filters:

Filter	Purpose
string.toupper	Convert to uppercase
string.rot13	ROT13 encode
convert.base64-encode	Base64 encode
convert.base64-decode	Base64 decode
zlib.deflate	Compress
zlib.inflate	Decompress

Example:

php
// Read a file and base64-encode it on the fly
$encoded = file_get_contents('php://filter/convert.base64-encode/resource=file.txt');

// Chain multiple filters
$result = file_get_contents(
    'php://filter/string.toupper|convert.base64-encode/resource=file.txt'
);

// Apply filter when writing
file_put_contents('php://filter/zlib.deflate/resource=out.txt', $data);

When to use filters:

Encoding/decoding data during read or write
Transforming content without loading it all into memory
Chaining multiple transformations
Large file processing (streaming)
Key Difference (Simple Summary)
	Wrapper	Filter
Role	Where to get/send data	How to transform data
Analogy	A pipe connecting two places	A water filter on that pipe
Stacking	One wrapper per stream	Multiple filters can chain
Example	http://, ftp://, php://	base64-encode, rot13

They work together — the wrapper opens the stream, the filter transforms the data flowing through it:

php
// Wrapper = "read from file.txt"
// Filter  = "base64-encode it as you read"
php://filter/convert.base64-encode/resource=file.txt
