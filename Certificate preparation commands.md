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
