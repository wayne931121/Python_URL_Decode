# Python_URL_Decode

Python URL Decoder

The function url_decoder in url_decoder.py

```python
from url_decode import url_encoder, url_decoder

url = "https://zh.wikipedia.org/zh-tw/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81"
url = url_decoder(url)
print(url)

## output: https://zh.wikipedia.org/zh-tw/百分号编码
```

```python
def url_decoder(b):
# https://zh.wikipedia.org/zh-tw/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81
# %21 mean 21hex, => int("21",16) => 33 => chr(33) => "!", result: "!"
    if type(b)==bytes:
        b = b.decode("utf-8") #byte can't insert utf8 charater
    result = bytearray()
    enter_hex_unicode_mode = 0
    hex_tmp = ""
    now_index = 0
    for i in b:
        if i=='%': #like %51%52, have entered mode, continue appending bytearray
            enter_hex_unicode_mode = 1
            continue
        if enter_hex_unicode_mode:
            hex_tmp += i
            now_index += 1
            if now_index==2: #%51%5F len("51")=2 len("5F")=2
                result.append(int(hex_tmp, 16) )
                hex_tmp = ""
                now_index = 0
                enter_hex_unicode_mode = 0
            continue
        result.append(ord(i))
    result = result.decode(encoding="utf-8")
    return result
```

```python
#保留字元的百分號編碼
URL_RFC_3986 = {
"!": "%21", "#": "%23", "$": "%24", "&": "%26", "'": "%27", "(": "%28", ")": "%29", "*": "%2A", "+": "%2B", 
",": "%2C", "/": "%2F", ":": "%3A", ";": "%3B", "=": "%3D", "?": "%3F", "@": "%40", "[": "%5B", "]": "%5D",
}

def url_encoder(b):
    # https://zh.wikipedia.org/wiki/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81
    if type(b)==bytes:
        b = b.decode(encoding="utf-8") #byte can't insert utf8 charater
    result = bytearray() #bytearray: rw, bytes: read-only
    enter_hex_unicode_mode = 0
    hex_tmp = ""
    now_index = 0
    for i in b:
        if i in URL_RFC_3986:
            for j in URL_RFC_3986[i]:
                result.append(ord(j))
            continue
        i = bytes(i, encoding="utf-8")
        if len(i)==1:
            result.append(ord(i))
        else:
            k = 0
            for c in i:
                c = hex(c)[2:].upper()
                result.append(ord("%"))
                result.append(ord(c[0:1]))
                result.append(ord(c[1:2]))
    result = result.decode(encoding="ascii")
    return result

#print(url_encoder("我好棒==%%0.0:)")) ==> '%E6%88%91%E5%A5%BD%E6%A3%92%3D%3D%%0.0%3A%29'
