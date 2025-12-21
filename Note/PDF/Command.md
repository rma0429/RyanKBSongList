```
```$inputPDF = "檔名.pdf";
$outputName = [System.IO.Path]::GetFileNameWithoutExtension($inputPDF);
& "D:\Program Files\ImageMagick-7.1.2-Q16-HDRI\magick.exe" -density 300 $inputPDF -trim +repage -background white -alpha remove -negate -append "$outputName.png"


```



```
Get-ChildItem *.pdf | ForEach-Object {
    $n = $_.BaseName;
    & "D:\Program Files\ImageMagick-7.1.2-Q16-HDRI\magick.exe" -density 300 $_.Name -trim +repage -background white -alpha remove -negate -append "$n.png"
}
```