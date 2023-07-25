# Certificate preparation commands

#### Convert to sha256 Name

```bash
find . -maxdepth 1 -name '*.pdf' -print0 | while read -d $'\0' i ;do
    mv "$i" "`sha256sum "$i" | cut -d" " -f1`.pdf"
done
```

#### Get code map and convert to sha256

```javascript
find . -maxdepth 1 -name '*.pdf' -print0 | while read -d $'\0' i ;do
    echo $i - `sha256sum "$i"`
    mv "$i" "`sha256sum "$i" | cut -d" " -f1`.pdf"
done
```

#### Generate pdf.js pdfData JS file

```javascript
find . -maxdepth 1 -name '*.pdf.js' -print0 | while read -d $'\0' i ;do
    value=`cat "$1"`
    echo var pdfData = atob(\'"$value"\'); > "$1"
done
```

#### Generate  pdfData JS file

```javascript
find . -maxdepth 1 -name '*.pdf' -print0 | while read -d $'\0' i ;do

    value=`base64 "$i"` &&  c="var pdfData = atob('${value}');" && echo $c > "$i".js
done

find . -maxdepth 1 -name '*.pdf' -print0 | while read -d $'\0' i ;do
    echo $i - `sha256sum "$i"`
done
```
