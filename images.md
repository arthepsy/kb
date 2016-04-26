# images
### PNG  
##### crop or border (keep transparency)
```
convert src.png -background None -gravity center -extent 512x512 dest.png
```

##### optimize size  
```
pngquant --speed 1 file.png  
```

```
pngng -s 1 file.png
```

```
pngcrush src.png dest.png
```

```
optipng -o7 file.png
```


